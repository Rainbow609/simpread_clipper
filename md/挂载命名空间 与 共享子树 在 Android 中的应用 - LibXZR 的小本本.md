> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.xzr.moe](https://blog.xzr.moe/archives/194/)

> 前言之所以有这篇文章，还是因为在写别的内容时一不小心写多了，那就作为独立的文章发出来罢。本文中存在一些未经验证的直觉与推断，如有不准确的地方还望包容和指正。挂载命名空间什么是挂载命名空间？关于 ...

之所以有这篇文章，还是因为在写别的内容时一不小心写多了，那就作为独立的文章发出来罢。  
本文中存在一些未经验证的直觉与推断，如有不准确的地方还望包容和指正。

什么是挂载命名空间？
----------

关于 Linux 内核提供的命名空间机制，想要详细的了解还请参考 [比较官方的文档](https://man7.org/linux/man-pages/man7/namespaces.7.html) 。

总的来说，命名空间是类似 Docker 的容器技术的基石，能够对进程实现一定程度上的隔离，使其仿佛置身于 VM 之中：

```
➜  /home/libxzr docker run -it ubuntu:latest
root@8720a9012bb3:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 16:12 pts/0    00:00:00 bash
root           9       1  0 16:12 pts/0    00:00:00 ps -ef

```

在 Docker 中 ps 一下所看到的重新编号的 PID，就是 pid namespace 的效果。  
（Btw，同一个进程从不同命名空间的视角看可能具有不同的 PID，比如从容器外看容器内进程时）  
（容器内的进程是可以在容器外被看到的，这和 VM 完全不同，是一种更软的资源隔离）

命名空间有许多的类型，包括 cgroup namespace、ipc namespace、network namespace、mount namespace、time namespace、user namespace、uts namespace 等以及上面提及的 pid namespace。

Android 所使用的命名空间主要是 “挂载命名空间” （mount namespaces），其作用是在不同的进程间实现挂载隔离：  
比如，在应用 A 的命名空间内挂载了一个 tmpfs，在应用 B 的命名空间内却是看不到的，可以用一个小实验来验证一下：

```
# 首先，创建一个挂载点
windows_x86_64:/ # mkdir /mnt/testmount
# 进入某个应用进程的挂载命名空间
windows_x86_64:/ # nsenter -t 4728 -m sh
# 把一个 tmpfs 挂载到上述挂载点上
windows_x86_64:/ # mount -t tmpfs test /mnt/testmount/
# 确保挂载是成功的，可以在 mount 里看到
windows_x86_64:/ # mount | grep /mnt/testmount
test on /mnt/testmount type tmpfs (rw,seclabel,relatime)
# 切换到另一个应用的挂载命名空间
windows_x86_64:/ # nsenter -t 7341 -m sh
# 再看看刚刚的挂载还在不在，诶，没了
windows_x86_64:/ # mount | grep /mnt/testmount/
1|windows_x86_64:/ #

```

（别在意`windows_x86_64`，图方便用了 [WSA](https://learn.microsoft.com/en-us/windows/android/wsa/)）  
（`nsenter` 命令将会在下面进行更加详细的介绍）

那么我们该如何知道当前进程运行在哪个命名空间中呢？  
procfs 为我们提供了接口，我们可以通过查看 `/proc/<pid>/ns` 目录来列出进程所使用的命名空间，但，列出来的只是命名空间类型的文件名，我们需要对这些文件使用 `readlink` 或者干脆在目录下 `ls -l` 才能真正看到对应的命名空间编号：

```
windows_x86_64:/ $ ls /proc/self/ns
cgroup  ipc  mnt  net  pid  pid_for_children  time  time_for_children  uts
windows_x86_64:/ $ readlink /proc/self/ns/mnt
mnt:[4026532359]
windows_x86_64:/ $ ls -l /proc/self/ns
total 0
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 ipc -> ipc:[4026532330]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 mnt -> mnt:[4026532359]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 net -> net:[4026531999]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 pid -> pid:[4026532331]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 pid_for_children -> pid:[4026532331]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 time -> time:[4026531834]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 time_for_children -> time:[4026531834]
lrwxrwxrwx 1 shell shell 0 2022-09-28 12:34 uts -> uts:[4026532329]

```

我们可以尝试切换一个命名空间试试：

```
windows_x86_64:/ # readlink /proc/self/ns/mnt
mnt:[4026532359]
# 在新的挂载命名空间中启动一个新的 sh 进程
windows_x86_64:/ # unshare -m sh
windows_x86_64:/ # readlink /proc/self/ns/mnt
mnt:[4026532513]

```

（`unshare` 命令将会在下面进行更加详细的介绍）

可以看到，在切换命名空间后命名空间的编号确实发生了变化，说明切换很成功。

所以，`unshare` 和 `nsenter` 命令是如何进行命名空间切换的呢？

操作命名空间
------

Linux 内核提供了 [`setns` syscall](https://man7.org/linux/man-pages/man2/setns.2.html) 来方便用户空间将调用者线程切换到指定的已存在命名空间，还提供了 [`unshare` syscall](https://man7.org/linux/man-pages/man2/unshare.2.html) 用于将调用者切换到指定类型的独立新命名空间，此外我们还可以通过在使用 [`clone` syscall](https://man7.org/linux/man-pages/man2/clone.2.html) 创建新进程时传入对应的 flag 来使新进程运行在新的 namespace 中。

对于 “挂载命名空间” 来说，新命名空间在刚创建时的内容和旧命名空间是完全一致的，但它们是独立的（有 `fork()` 的味道，也可以理解为新空间会自动把旧空间的内容复制一份）。除非特别指定（下面会介绍的共享子树），挂载发生的更改不会在新旧命名空间之间传播。

将 syscall 当成方法来看，那么：  
`setns(fd, nstype)` 需要传入两个参数，一是由 `open()` 上面命名空间文件得到的 file descriptor，二是命名空间的类型。参数二的唯一作用是用来校验第一个参数是不是指定类型的命名空间，因此也可以直接传入 0。

`unshare(flags)` 则只需要传入一个参数，那就是新命名空间的类型。

命名空间的类型在 kernel headers 中有提供：

include/linux/syscalls.h

```
......
#define CLONE_NEWNS    0x00020000    /* New mount namespace group */
......
#define CLONE_NEWCGROUP        0x02000000    /* New cgroup namespace */
#define CLONE_NEWUTS        0x04000000    /* New utsname namespace */
#define CLONE_NEWIPC        0x08000000    /* New ipc namespace */
#define CLONE_NEWUSER        0x10000000    /* New user namespace */
#define CLONE_NEWPID        0x20000000    /* New pid namespace */
#define CLONE_NEWNET        0x40000000    /* New network namespace */
......

```

像是对于我们上面说到的 “挂载命名空间”，传入的便是 `CLONE_NEWNS`。  
当然，如果使用的是 libc 提供的 wrapper ，那可能会有一点点区别。

那，接下来就来看看上面用到的两条指令吧。

插一句，在 Android 平台上，类 Linux 指令往往都是由 [toybox](https://android.googlesource.com/platform/external/toybox/) 提供，它们和正统 GNU/Linux 上的 coreutils + util-linux 组合提供的命令用法虽然大体相同但在细节上往往有一些区别（后面会看到）。toybox 存在的最大意义我想就是通过 rewrite 来规避 GPL 了。

回到正题，首先是 `nsenter`：

```
    usage: nsenter [-t pid] [-F] [-i] [-m] [-n] [-p] [-u] [-U] COMMAND...

    Run COMMAND in an existing (set of) namespace(s).

    -a    Enter all supported namespaces (--all)
    -F    don't fork, even if -p is used (--no-fork)
    -t    PID to take namespaces from    (--target)

    The namespaces to switch are:

    -C    Control groups (--cgroup)
    -i    SysV IPC: message queues, semaphores, shared memory (--ipc)
    -m    Mount/unmount tree (--mount)
    -n    Network address, sockets, routing, iptables (--net)
    -p    Process IDs and init, will fork unless -F is used (--pid)
    -u    Host and domain names (--uts)
    -U    UIDs, GIDs, capabilities (--user)

    If -t isn't specified, each namespace argument must provide a path
    to a namespace file, ala "-i=/proc/$PID/ns/ipc"

```

进入某个 “挂载命名空间” 的典型用例：`nsenter -t <pid> -m <command>`。  
其中 pid 是进程 id，指你想要进入哪个进程所在的命名空间，`-m` 代表想要进入其 “挂载命名空间”（当然你也可以按需替换或增加其它命名空间类型），command 则是你想要在这个命名空间中执行的指令。

`nsenter` 所做的事情，便是调用 `setns` syscall 改变其自身所在的命名空间（当然，这中间需要一些处理，比如根据 pid 取得命名空间的 file descriptor），接着调用 `exec` 家族的函数执行 command 替换当前进程的代码，于是就起到了在指定进程的命名空间中执行指定命令的效果。

接下来是 `unshare` 指令：

```
    usage: unshare [-imnpuUr] COMMAND...

    Create new container namespace(s) for this process and its children, allowing
    the new set of processes to have a different view of the system than the
    parent process.

    -a    Unshare all supported namespaces
    -f    Fork command in the background (--fork)
    -r    Become root (map current euid/egid to 0/0, implies -U) (--map-root-user)

    Available namespaces:
    -C    Control groups (--cgroup)
    -i    SysV IPC (message queues, semaphores, shared memory) (--ipc)
    -m    Mount/unmount tree (--mount)
    -n    Network address, sockets, routing, iptables (--net)
    -p    Process IDs and init (--pid)
    -u    Host and domain names (--uts)
    -U    UIDs, GIDs, capabilities (--user)

    Each namespace can take an optional argument, a persistent mountpoint usable
    by the nsenter command to add new processes to that the namespace. (Specify
    multiple namespaces to unshare separately, ala -c -i -m because -cim is -c
    with persistent mount "im".)

```

对于新建一个 “挂载命名空间”，其典型用例是：`unshare -m <command>`。  
其中 `-m` 代表要新建的是 “挂载命名空间”（当然你也可以按需替换或增加其它命名空间类型），command 则是你想要在这个命名空间中执行的指令。

它所完成的事情便是——调用 `unshare` syscall 将自身切换到独立的新命名空间，接着调用 `exec` 家族的函数执行 command 替换当前进程的代码，于是就起到了在新的命名空间中执行指定命令的效果。

（对了，不要产生一个进程只能加入一个命名空间的误解，上面只演示了 mount namespace 是因为接下来只用到了这个。一个进程完全可以加入**不同类型**的多个命名空间，比如说 `unshare -m -p` 就能既新建一个 mount namespace 又新建一个 pid namespace 还同时加入这俩，毕竟不同类型的命名空间负责着不同的功能，是不会相互冲突的）

此外，这类命令存在的最大意义就是为调试提供方便，不要觉得系统就是靠着执行这些命令运行起来的。系统在命名空间需要改变时会去走对应的 syscall 而不是启动一个新进程来执行这些破指令，来回启动新进程的开销可一点也不小。所以，永远不要用脚本小子的思维来看待 OS。

“挂载命名空间”解决了 “隔离” 问题，但是没有解决 “共享” 问题。我们想要新挂载一个分区，难道需要把这个分区一个一个的塞进各个 “挂载命名空间” 吗？这也太麻烦了吧。  
于是 “共享子树” 就诞生了，它为事件在 “挂载命名空间” 之间传播提供了通道。想要详细的了解“共享子树”，请参阅[官方文档](https://www.kernel.org/doc/html/latest/filesystems/sharedsubtree.html) 。在这里，我只能按照我的理解进行有限的概述。

想要传播挂载事件，首先要确定传播的对象，于是我们引入了 “对等组”（peer group）的概念。

那么什么是对等组？  
首先它是一个组，也就意味着它可以有多个成员。

那么它的成员是什么？  
是挂载点。

怎样的挂载点能位于同一个 “对等组” 中呢？  
这些挂载点之间得有亲缘关系：至少它们指向的 inode 会是相同的。但是，这只是必要不充分条件，因为**我们没法手动将挂载点加入 “对等组” 中**，加入对等组的行为是自动的，但是会参照挂载点具有的**属性**进行。

那么这是什么属性呢？  
这个属性被称为 “传播类型”（propagation types），是每个挂载点都具有的属性（“挂载命名空间” 也会隔离此属性）（是挂载点才具有此属性，普通文件夹可没有这个属性）。  
一共有四种 “传播类型”：

*   shared：本挂载点愿意和 “对等组” 内的其它成员完全共享（接收 + 发送）挂载事件。
*   slave：本挂载点只愿意接受挂载事件，但是挂载事件不应该被传播给 “对等组” 内的其它成员。
*   private：本挂载点不分享挂载事件，别把我拉进对等组。
*   unbindable：本挂载点不仅不分享挂载事件（相当于 private），还不想被 bind mount 到别的地方，相当于是一个加强版的 private 类型。（接下来不再讨论这个，提及 private 类型的时候多半它也被包含了进去）

那么在什么时候，挂载点会加入到 “对等组” 中呢？

*   1.  一个挂载点的 “传播类型” 被从别的类型**修改**成了 shared 时。此时，一个 “对等组” 会被新建出来，然后这个挂载点被加了进去，当然此时它是这个组的唯一成员。
*   2.  当一个挂载点被 bind mount 时。比如 A 被 bind mount 到了 B，那么 B 将会自动加入 A 所在的 “对等组”（如果有的话，即 A 不是 private 的）。
*   3.  当一个挂载点因创建新的 “挂载命名空间” 而被 “克隆” 时。比如 A 命名空间中有挂载点 `/test`，然后我们使用 `unshare()` 创建了一个新的 “挂载命名空间” B ，那么 B 中的 `/test` 会自动加入 A 中 `/test` 所在的 “对等组”（如果有的话，即 A 不是 private 的）。

所以，“传播挂载事件” 是如何体现的呢？  
就以上述的 3 为例吧。假如 A 命名空间中的 `/test` 是 **share 类型** (1) 的，那么它必然位于一个 “对等组” 中，我们姑且称之为“组 1”。于是我们得到的 B 命名空间中的 `/test` 会自动成为 “组 1” 的成员（此时 B 中 `/test` 的默认类型也是 share ）。于是，接下来我们在 A 中的 `/test` **下** (2) 挂载的东西，也会自动挂载在 B 中的 `/test` 下，这便实现了挂载事件的传播。

*   (1) 假如 A 中的 `/test` 是 slave 类型的，那么 B 中的 `/test` 也将会是 slave 类型，但是它们仍然属于同一个 “对等组”，只不过它们都只能从同组的 share 类型挂载点接受挂载事件，而不能将自己的挂载事件传播出去。像这样的偏 edge case 不是很想在这里讨论，还是参考官方文档为妙。
*   (2) 只有属于 `/test` 直接下级的挂载事件才会被其传播，这里是指挂载位置与 `/test` 之间不能间隔一个挂载点。举点例子：挂载在 `/test/a` 显然能被其传播；挂载在 `/test/a/b/c` 能被其传播的前提是 `a` 和 `b` 都只是普通文件夹而不是挂载点。假如 `/test/a` 是一个挂载点那么 `/test/a/b/c` 的传播将由 `/test/a` 负责而非 `/test`。换言之，一个挂载的传播只由它的 “直接上级挂载点” 决定，隔了一层就完全管不着了。这里的 “直接上级挂载点” 是指层层向上找，遇到的第一个挂载点，不是说上级文件夹要是挂载点，如果这一路都没有遇到挂载点，那么它的直接上级挂载点将会是 `/`（这必然是一个挂载点）。

所以，我该如何改变一个挂载点的 “传播类型” 呢？  
在 GNU/Linux 中，我们可以使用 util-linux 提供的 `mount` 命令自带的 `--make-shared`、`--make-slave`、`--make-private` 等选项来改变一个挂载点的 “传播类型”。  
比如说：`sudo mount --make-shared /`。

但遗憾的是，Android 平台的 `mount` 由 toybox 提供，而它并没有提供这些选项。也就是说，我们并没有办法直接使用命令来调试共享子树。  
但也很显然，命令的背后一定是对应的 syscall ，Android 平台想要调用这项功能只需要在调用 `mount` 的 bionic wrapper 时传入对应的 flag 即可。  
[于是顺手写了个 `chsst`](https://github.com/libxzr/chsst) 来弥补 Android 平台上 `mount` 所缺失的功能，有需要的可以自取。

还有一些比较 tricky 的点，比如说你想实现挂载事件从 A 到 B 的单向传播，此处以 bind mount 为例（多个命名空间是类似的）。那么，流程大概是这样的：

*   首先有一个挂载点 A。
*   把 A 的 “传播类型” 修改为 shared （除非它本身就是 shared），以确保它以 shared 身份在一个 “对等组” 内。
*   将 A bind mount 到 B，此时 A 与 B 在同一个 “对等组” 内，且类型都是 shared。
*   将 B 的 “传播类型” 修改为 slave （`--make-slave`），此时它们仍然在一个 “对等组” 内，但是类型限制了事件的传播方向。
*   此时挂载事件只能由 A 向 B 单向传播，完成了。  
    第一次接触的话可能感觉有点怪，但由于我们不能手动指定谁属于哪个组，所以，也只能这么办了....

还有一些更 tricky 的点，比如把 slave 类型变回 shared 类型时同样会新建一个 “对等组”（于是它就同时属于两个对等组了）之类的，这里就不聊了，需要的建议参考官方文档。

这块内容的最后，我们该如何知道哪几个挂载点属于同一个 “对等组” 呢？  
很简单，都写在 mountinfo 里了。

```
➜  /home/libxzr cat /proc/self/mountinfo | grep /mnt
97 28 8:1 / /mnt/data rw,relatime shared:49 - ext4 /dev/sda1 rw

```

这里的 `shared:49` 代表这个挂载点是 shared 传播类型的，属于组号为 49 的 “对等组”。

然后我们来 bind mount 一下：

```
➜  /home/libxzr sudo mkdir /mnt/data2
➜  /home/libxzr sudo mount --bind /mnt/data /mnt/data2
➜  /home/libxzr cat /proc/self/mountinfo | grep /mnt
97 28 8:1 / /mnt/data rw,relatime shared:49 - ext4 /dev/sda1 rw
368 28 8:1 / /mnt/data2 rw,relatime shared:49 - ext4 /dev/sda1 rw

```

符合预期，它们都是 shared 且属于同一个 “对等组”。

最后改变一下类型看看？

```
➜  /home/libxzr sudo mount --make-slave /mnt/data2
➜  /home/libxzr cat /proc/self/mountinfo | grep /mnt
97 28 8:1 / /mnt/data rw,relatime shared:49 - ext4 /dev/sda1 rw
368 28 8:1 / /mnt/data2 rw,relatime master:49 - ext4 /dev/sda1 rw

```

data2 变成了 slave 类型，显示的内容变成了 `master:49`。这表示，它的 master 是 “对等组” 49 （太怪了，为什么我觉得改成 slave:49 比较合适😂。。。）  
于是现在挂载事件只能由 `data` 向 `data2` 单向传播了。  
对了，对于 private 类型，它压根就不会显示类似 shared / master 的字段，也没有组号。

对 Linux 内核而言，根挂载点的默认 “传播类型” 是 private (1) 。在 Android 平台上，在 second stage init 中根挂载点的 “传播类型” 被手动修改为了 shared ，这便是第一个 “对等组” 的诞生：

system/core/init/mount_namespace.cpp

```
bool SetupMountNamespaces() {
    // Set the propagation type of / as shared so that any mounting event (e.g.
    // /data) is by default visible to all processes. When private mounting is
    // needed for /foo/bar, then we will make /foo/bar as a mount point (by
    // bind-mounting by to itself) and set the propagation type of the mount
    // point to private.
    if (!ChangeMount("/", MS_SHARED | MS_REC)) return false;
    ......
}

```

于是，后来新挂载在 `/` 下面的挂载点也会自动具有 share 的类型 (1) ，比如 `/dev`、`/proc` 还有关键的 `/mnt`。

*   (1) 新挂载点的默认类型由父挂载点类型决定，没有父挂载点时默认为 private，详见相关文档。当然，这里的 “新挂载点” 不能来自其它挂载点的 bind mount，不然就是之前描述的情况了。

之后，在 Zygote 初始化时，会创建新的 “挂载命名空间”，并且将该命名空间中的所有挂载点递归的设置为 slave 类型。此时 Zygote 所在的“挂载命名空间” 与“默认命名空间”脱离，挂载事件只能从 “默认命名空间” 向 Zygote 单向传递：

core/jni/com_android_internal_os_Zygote.cpp

```
static void UnmountStorageOnInit(JNIEnv* env) {
  ......
  if (unshare(CLONE_NEWNS) == -1) {
    RuntimeAbort(env, __LINE__, "Failed to unshare()");
    return;
  }
  ......
  if (mount("rootfs", "/", nullptr, (MS_SLAVE | MS_REC), nullptr) == -1) {
    RuntimeAbort(env, __LINE__, "Failed to mount() rootfs as MS_SLAVE");
    return;
  }
  ......

```

Android 应用启动时，Zygote 会 `fork()` ，之后会再调用 `unshare()` ，使得每个应用都位于自己的 独立 (1) “挂载命名空间” 中：

core/jni/com_android_internal_os_Zygote.cpp

```
static void ensureInAppMountNamespace(fail_fn_t fail_fn) {
  ......
  if (unshare(CLONE_NEWNS) == -1) {
    fail_fn(CREATE_ERROR("Failed to unshare(): %s", strerror(errno)));
  }
  ......
}

```

于是，所有 Android 应用程序都运行在自己的独立 “挂载命名空间” 中，且只能从 “默认命名空间” 单向的接收挂载事件。  
这对于 Android 的存储结构有重大的意义，留到另一篇文章里再聊。

*   (1) 由于 Zygote 已经递归的将 所有 (2) 挂载点设为了 slave 类型，因此 `unshare()` 出来的新命名空间中所有的挂载点也将会是 slave 类型且和 Zygote 所在命名空间位于一个 “对等组” 中。虽然它们位于一个 “对等组” 里，但是由于都是 slave 类型所以无法互相传播挂载事件，只能从 “默认命名空间” 接收事件。所以不同应用的命名空间是完全隔离的，也和 Zygote 所在命名空间隔离。
*   (2) “所有” 并不准确。其实一些挂载点（如 `/apex`）会被 make-private。但是 private + make-slave = private 所以不会产生影响（参见官方文档）。

现在，应该很容易理解 SuperSU 和 Magisk 里的 “挂载命名空间” 是在说什么了吧：

[![](https://blog.xzr.moe/usr/uploads/2022/10/2506146501.png)](https://blog.xzr.moe/usr/uploads/2022/10/2506146501.png)

“全局命名空间”指的就是上面的 “默认命名空间”，“继承命名空间” 指的就是应用所在的独立命名空间，而 “独立命名空间” 是指为 root 进程在应用命名空间的基础上再 `unshare()` 一个新的出来。

哦对了，adb shell 是运行在 “默认命名空间” 的。  
这就是为什么，当这个设置项选择为 “继承命名空间” 时，应用调用 `su` 创建的挂载并不能在 `adb shell` 中被看到。但反过来，adb shell + su 创建的挂载，应用一定看得到（传播到了应用所在命名空间）。  
当选择为 “独立命名空间” 时应该更有趣，应用调用 `su` 创建的挂载只有 `su` 自己能看到，应用都看不到（笑

就到这里吧，下篇文章见。