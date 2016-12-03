jcpu
=======

jcpu is a java cpu debug tool.

The example below will dump the top 5 stack of using the most CPU time.

<pre><code>

#./jcpu.sh [pid]
.
.
.
.

</code></pre>


Enjoy it!

jcpu2
========
原理，以及说明下类似脚本的适用范围。

步骤1：dump当前JVM线程，保存现场
$JAVA_HOME/bin/jstack $pid > $tmp_file
保存现场是相当的重要，因为问题转瞬之间就会从手中溜走（但其实LOAD的统计机制也决定了，事实也并不是那么严格）

步骤2：找到当前CPU使用占比高的线程
ps H -eo user,pid,ppid,tid,time,%cpu --sort=%cpu
列说明
USER：进程归属用户
PID：进程号
PPID：父进程号
TID：线程号
%CPU：线程使用CPU占比（这里要提醒下各位，这个CPU占比是通过/proc计算得到，存在时间差）

步骤3：合并相关信息
我们需要关注的大概是3列：PID、TID、%CPU，我们通过PS拿到了TID，可以通过进制换算10-16得到jstack出来的JVM线程号

typeset nid="0x"$(echo "$line"|awk '{print $1}'|xargs -I{} echo "obase=16;{}"|bc|tr 'A-Z' 'a-z')
最后再将ps和jstack出来的信息进行一个匹配与合并。终于，得到我们最想要的信息

适用范围说明
看似这个脚本很牛X的样子，能直接定位到最耗费CPU的线程，开发再也不用担心找不到线上最有问题的代码～但，且慢，姑且注意下输出的结果，State: WAITING 这是这个啥节奏～
这是因为ps中的%CPU数据统计来自于/proc/stat，这个份数据并非实时的，而是取决于OS对其更新的频率，一般为1S。所以你看到的数据统计会和jstack出来的信息不一致也就是这个原因～但这份信息对持续LOAD由少数几个线程导致的问题排查还是非常给力的，因为这些固定少数几个线程会持续消耗CPU的资源，即使存在时间差，反正也都是这几个线程所导致。

https://yq.aliyun.com/articles/55?spm=5176.100239.blogcont65107.19.7A6ND5
