**Author:** Son Hoang

---

# My Lesson Learned with Distributed Lock

Distributed lock really helps me to solve my issues, but it doesn't mean the issue will be solved immediately if I use distributed lock.

[https://blogs.sonht.io.vn/posts/240902-my-lesson-learned-with-distributed-lock](https://blogs.sonht.io.vn/posts/240902-my-lesson-learned-with-distributed-lock)

![](https://blogs.sonht.io.vn/favicon.png)

![](https://blogs.sonht.io.vn/large-preview.png)

---

Distributed lock really helps me to solve my issues, but it doesn't mean the issue will be solved immediately if I use distributed lock.

Distributed lock really helps me to solve my issues, but it doesn't mean the issue will be solved immediately if I use distributed lock.

### System Overview

Okay. Imagine that I'm working on the feature that distributes voucher to user. Like, if user signs up, they will receive the voucher item. The flow is simplified as the picture below.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-1.png)

1. Mobile calls `signup` API to **Auth Service**.  
2. **Auth Service** does its stuff then pushes message to queue. The message will be:

   ```json
   {"user_id": <user_id>}
   ```

3. The message then will be consumed by **Voucher Service** to distribute the voucher to user.
4. Finally, send notification to user to tell that their voucher is ready to use.

So the input is `user_id` and the output is voucher item distributed to user.

Let's see how **Voucher Service** distributes voucher item to user. In real world, this service is so complicated that none of us really knows how it works hahaha, but just imagine that we have one table called `voucher_item`.

#### `voucher_item` Table

| id | user_id | code     |
|----|---------|----------|
| 1  | 1       | BV1LN2KJ |
| 2  | null    | ASXKKQRG |

If `user_id` is not null (eg: 1), it means **user 1** already claims voucher item, in this case **user 1** has **voucher item 1**. Otherwise, the voucher is waiting for being claimed.

So when **Voucher Service** consumes message, it will query the **first** valid voucher which has `{user_id: null}` and then updates that voucher with corresponding `user_id`. Done.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-2.png)

### Scenario

I receive my task related to voucher, but it's not about signup. It's about employee import, like, admin will import list of employees into system and then give all of them vouchers. Actually, it's quite the same as the signup feature, but the input now will be array of users instead of one user.

But, as I said, the **Voucher Service** is too complicated, it's not smart to implement another function to handle that. The easiest way that I come up with is, I will push a bunch of message into queue. Let's say, admin imports 10 users from 1 to 10, so we will push 10 separate messages into queue to utilize available logic.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-3.png)

Okay, let's keep other steps (3), (4) as now, no need to change anything. We have queue, messages will be handled sequentially. I test that on my local machine and it works perfectly, let's ask QA team to test. Hahaha easy easy.

But then, "Nah, please fix that. It doesn't work on dev env", QA team said.

> dev env is the env that QA uses for testing.

### What's wrong here?

Based on QA team's actual result, they test with only 2 employees:

#### `employees` Table

| id |
|----|
| 1  |
| 2  |

- **user 1** doesn't receive voucher, but receives notification.
- **user 2** receives voucher, but doesn't receive notification.

Are you kidding me? Sounds magic here. But I tested and it worked perfectly on my local machine, right.

Let's see. If **user 1** already receives notification, it means, the voucher is already claimed by **user 1** (see the logic flow above). But somehow, **user 2** claims the voucher of **user 1**.

Then I realize that, the biggest differences between my local machine and dev env is that dev env has **2 pods of Voucher Service** (auto scaling). Okay, let's update the diagram.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-4.png)

Much clearer now. 2 messages can be handled almost at the same time, and 2 message handlers can point to the same voucher item.

1. 1st message is consumed, it gets `voucher_item 1` from database.
2. 2nd message is consumed right after that, it gets `voucher_item 1` from database too since 1st message is still in progress.
3. 1st message finishes handling, update `voucher_item 1` with **user 1**. Then send notification to **user 1**.
4. 2nd message finishes handling, update `voucher_item 1` with **user 2**. But I'm still not sure why **user 2** doesn't receive notification.

Okay, now our issue is, 2 processes both get access to the same resource. Definitely, **distributed lock** will save us in this case.

Actually, I intended to implement distributed lock by myself using **Redis `setnx`** but then I was recommended using [Redlock](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/) which works perfectly with multiple Redis nodes.

Let's apply lock into **Voucher Service**. We can easily find [`nodejs redlock` package](https://www.npmjs.com/package/redlock) and install.

But where should I put the lock? Hmm, it should be placed right after getting voucher item from database, since we can use `voucher_item.id` as key of redlock.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-5.png)

1. 1st message is consumed, it gets `voucher_item 1`. It tries to acquire the lock `voucher_item:1` and hold it.
2. 2nd message is consumed, it gets `voucher_item 1`. It tries to acquire the lock `voucher_item:1` but gets failed since it's being held by 1st. Okay, that's our expected result. We can choose retry message or abort it, it's up to each scenario.
3. After 1st message finishes, let's release the lock. Sounds good. Commit code and assign to QA team again.

One more time, QA said "Nah sir, the issue is still happening. Please fix that."

### What's wrong again?

Is redlock lying me? Nah. Let's see what happens. Since 2 messages can be handled almost at the same time, I come up with this theory.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-6.png)

1. 1st message is consumed, it gets `voucher_item 1` and hold lock.
2. 2nd message is consumed, it gets `voucher_item 1` **right before** 1st finishes updating `voucher_item 1`.
3. 1st finishes handling, it releases the lock.
4. 2nd then acquires lock and then updates `voucher_item 1`.

Seems reasonable. It's because the process that acquires lock and update voucher item happens too fast.

Okay. let's give it some extra time before releasing the lock and the issue should be solved now.

![](https://blogs.sonht.io.vn/images/my-lesson-learned-with-distributed-lock-7.png)

I change the lock release time from `t1` to `t2`. But you may wonder, what should the gap (`t2 - t1`) be? I think it should be at least equal to the execution time of logic between acquiring lock and releasing lock. For example, the logic above takes 10ms to acquired lock + update voucher item + release lock, then `t2 - t1` should `>= 10ms`.

### Conclusion

That's all I want to share. The lesson learned here is, should choose the lock release time carefully, it will save your life.

### Reference

- [Redlock](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/)