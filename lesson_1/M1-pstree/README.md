# NJUOSLAB M1:打印进程树（pstree功能实现）

把系统中的进程按照父亲-孩子的树状结构打印到终端。

-p 或 --show-pids: 打印每个进程的进程号。
-n 或 --numeric-sort: 按照pid的数值从小到大顺序输出一个进程的直接孩子。
-V 或 --version: 打印版本信息。

/proc/pid/stat 文件中有name和ppid

TODO：
1. 当进程名称出现空格 “ ” 时，如：（tmux：sever），存在bug 因为读取/proc/1234(对应进程pid)/stat文件出现空格读取错位存在bug需要解决。
2. 考虑更贴合实际pstree的输出： 对于叶子节点 /proc/1234/task/ 文件夹下会存在叶子节点开启的子进程( 还会有个该进程的副本文件夹 )文件夹，里面stat文件记录pid和进程名称，且这种进程不会在/proc/ 下出现，pstree中这种进程输出样式为 [{   ***  }}]

思考：
1. 万一我得到进程号以后，进去发现文件没了 (进程终止了)，怎么办？会不会有这种情况？万一有我的程序会不会 crash……？
2. 进程的信息一直在变，文件的内容也一直在变 (两次 cat 的结果不同)。那我会不会读到不一致的信息(前一半是旧信息、新一半是新信息)？这两个问题都是 race condition 导致的；我们将会在并发部分回到这个话题。
3. 如果我不确信这些事会不会发生，我有没有写一个程序，至少在压力环境下测试一下它们有没有可能发生？嗯，如果我同时运行很多程序，每个程序都不断扫描目录、读取文件，也观察不到这个问题，至少应该可以放点心。
