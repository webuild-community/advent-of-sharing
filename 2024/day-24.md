**Author:** Son Hoang

---

# Cache Problem – Cache Penetration

Cache penetration is one of problems we should consider first when utilizing cache in our application.

![favicon](https://blogs.sonht.io.vn/favicon.png)

https://blogs.sonht.io.vn/posts/240702-cache-penetration

![preview](https://blogs.sonht.io.vn/large-preview.png)

---

Cache is one of the most important part when building highly available application. It provides a high-performance way to get frequently accessed data and reduce traffic to our database.

Cache penetration is one of very first problem we should consider when utilizing cache in our application.

### What is cache penetration?

This is a scenario where data to be searched doesn't exist in our database and those empty data is not cached as well, so **database always gets hit on every search on same resource** and even cause it to crash.

### Example

This is one simple way we often cache in our application.

![cache-penetration-1](https://blogs.sonht.io.vn/images/cache-penetration-1.png)

Whenever request comes, we will search in cache first. If existed, we return data. Otherwise, we will find in our database, and then cache it and return data. So every following request on the same resource will hit cache instead of our database.

But let's imagine that when data is not found in cache, and data is not found in database as well.

![cache-penetration-2](https://blogs.sonht.io.vn/images/cache-penetration-2.png)

Since the queried data is empty, we may think that "Well no data? Nothing to cache" and we just simply return data to client, then every following request on the same resource will hit our database and yep, that's the problem we have mentioned above.

### Solution

Easiest solution here is we will cache those **empty data** as well. Remember to set ttl properly.

![cache-penetration-3](https://blogs.sonht.io.vn/images/cache-penetration-3.png)

A typical pseudo-code implementation might look like:

```ts
async function getUser(id: string) {
  const cacheKey = `user:${id}`;

  const cached = await redis.get(cacheKey);
  if (cached !== null) {
    // Note: could be an empty marker
    return JSON.parse(cached);
  }

  const user = await db.user.findById(id);

  if (!user) {
    // Cache empty result with short TTL
    await redis.set(cacheKey, JSON.stringify(null), "EX", 60);
    return null;
  }

  // Cache real data with normal TTL
  await redis.set(cacheKey, JSON.stringify(user), "EX", 3600);
  return user;
}
```

### Conclusion

This problem seems simple but we often don't care about it in real cases. Be aware of cache problems will help us to build a better application. So hope this post will help.

### Reference

[Tips Javascript](https://www.youtube.com/watch?v=_15A-fkBP7o)