**Author:** kien

---

# kiennt26's home | Linux Magic System Request Key Hacks

Author: Kien Nguyen <kiennt2609@gmail.com>  
A minimal Hugo theme with Tailwind CSS

![](https://ntk148v.github.io/favicon_io/favicon.ico)

https://ntk148v.github.io/posts/linux-sysrq/

---

### Giới thiệu

Trong quá trình vận hành, bạn đã bao giờ gặp tình trạng hệ thống Linux của mình bị “treo” hoặc không phản hồi? Khi đó, hãy sử dụng Magic System Request Key (SysRq) để được cứu rỗi. Vậy nó là gì và có thể làm gì?

SysRq là một tính năng của Linux, cho phép người dùng gửi “tín hiệu cầu cứu” trực tiếp đến kernel của hệ điều hành.

![](https://framerusercontent.com/images/wjLSwytVCtdnGhq2xK8m6qSo4.png)

sysrq - from trufflesecurity.com

---

### Cấu hình SysRq

Để cấu hình SysRq, bạn có thể sử dụng command sau:

```bash
echo "number" > /proc/sys/kernel/sysrq
```

Giá trị của `number` có thể nằm trong các trường hợp sau:

```text
0   - disable sysrq completely
1   - enable all functions of sysrq
>1  – bitmask to allow specific sysrq functions

2   = 0x2   - enable control of console logging level
4   = 0x4   - enable control of keyboard (SAK, unraw)
8   = 0x8   - enable debugging dumps of processes etc.
16  = 0x10  - enable sync command
32  = 0x20  - enable remount read-only
64  = 0x40  - enable signalling of processes (term, kill, oom-kill)
128 = 0x80  - allow reboot/poweroff
256 = 0x100 - allow nicing of all RT tasks
```

Kiểm tra giá trị hiện tại của sysrq:

```bash
root@vm1:/home/kien# cat /proc/sys/kernel/sysrq
176

# 176 không match với giá trị nào, well, thực ra 176 ở đây là 16+32+128 = 176
#
# 16  = 0x10  - enable sync command
# 32  = 0x20  - enable remount read-only
# 128 = 0x80  - allow reboot/poweroff

# Để test, bật hết lên cho đơn giản
root@vm1:/home/kien# echo 1 > /proc/sys/kernel/sysrq
```

---

### Cách sử dụng

Bạn có thể sử dụng tính năng bằng cách ấn tổ hợp phím (tùy thuộc hệ điều hành, đối với x86 là:

```text
ALT-SysRq-<command key>
```

Bàn phím của bạn thường sẽ có SysRq keyboard, để ý nhé) hoặc `echo` ký tự commands vào:

```bash
echo <command key> > /proc/sysrq-trigger
```

Danh sách command key (ấn để show all)

---

### Một số command keys hữu ích

> Để dễ hình dung, ví dụ dưới đây sẽ sử dụng phương án echo ký tự vào `/proc/sysrq-trigger`

#### Poweroff

```bash
root@vm1:/home/kien# echo o > /proc/sysrq-trigger
```

#### Reboot

```bash
root@vm1:/home/kien# echo b > /proc/sysrq-trigger
```

#### Crash

Trigger crashdump thủ công nếu hệ thống bị treo. Đây cũng là một cách hay để giả lập kernel crashdump.

```bash
root@vm1:/home/kien# echo c > /proc/sysrq-trigger
```

#### Đồng bộ filesystems

```bash
root@vm1:/home/kien# echo s > /proc/sysrq-trigger
```

#### Remount filesystem read-only

```bash
root@vm1:/home/kien# echo u > /proc/sysrq-trigger
root@vm1:/home/kien# touch abc
touch: cannot touch 'abc': Read-only file system
```

#### Kill tất cả processes trừ tiến trình init

Command key này đặc biệt hữu ích nếu bạn có tiến trình không thể kill, đặc biệt nếu tiến trình đó liên tục spawning ra các tiến trình khác.

```bash
# Linux gửi SIGTERM đến tất cả các processes, trừ init
root@vm1:/home/kien# echo e > /proc/sysrq-trigger

# Linux gửi SIGKILL đến tất cả các processes, trừ init
root@vm1:/home/kien# echo i > /proc/sysrq-trigger
```

#### Gọi OOM Killer

OOM Killer được gọi và hoàn thành nhiệm vụ của nó - kill tiến trình gây high memory usage.

```bash
root@vm1:/home/kien# echo f > /proc/sysrq-trigger

# Check kern.log để kiểm tra log
3585:Dec 5 03:31:40 vm1 kernel: [ 195.899186] sysrq: Manual OOM execution
3586:Dec 5 03:31:40 vm1 kernel: [ 195.901176] kworker/0:1 invoked oom-killer: gfp_mask=0xcc0(GFP_KERNEL), order=-1, oom_score_adj=0
```

#### Xem danh sách blocked state processes

```bash
root@vm1:/home/kien# echo w > /proc/sysrq-trigger

# Check kern.log để kiểm tra
Dec 5 03:33:02 vm1 kernel: [ 277.781446] sysrq: Show Blocked State
```