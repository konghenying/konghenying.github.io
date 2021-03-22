---
title: jvm
date: 2021年2月28日17:08:26
tags: jvm
category: jvm
---

# JVM

## 类加载过程

1. calss loading 加载

2. calss linking连接
   1. verification 验证:验证文件是否符合jvm规范 
   2. perparation准备 :静态成员变量赋初始值
   3. resloution解析:将类、方法、属性等符号引用解析为直接引用 常量池中的各种符号引用解析为指针、偏移量等内存地址的直接引用

3. class initializing 初始化: 将类的静态变量设置初始值,调用静态代码块

4. 申请对象内存.

5.  成员变量赋默认值 

6. 调用构造方法<init>

   6.1 成员变量顺序赋初始值

   6.2 执行构造方法语句

## 对象大小(64位)

### 观察虚拟机配置

```java
C:\Users\60512>java -XX:+PrintCommandLineFlags -version
-XX:InitialHeapSize=266593216 -XX:MaxHeapSize=4265491456 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
java version "1.8.0_72"
Java(TM) SE Runtime Environment (build 1.8.0_72-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.72-b15, mixed mode)
```

### 普通对象

1. 对象头:mardword 8

2. ClassPointer指针 -XX:+UserCompressedClassPointers(压缩指针指令)后为4字节  不开启为8字节

3. 示例数据 

   1. 引用类型: -XX:+UserCompressedOops(压缩实例数据中的引用类型)为4字节 不开启为8字节

      ​					Oops:Ordinary Object Pointers

4. Padding对齐 8的倍数

### 数组对象

1. 对象头:markword 8
2. ClassPointer指正
3. 数组长度:4字节
4. 数据数组
5. 对齐 8的倍数



注: markword结构

![img](/images/markword32.jpeg) 

### 验证对象大小步骤

1. 新建项目ObjectSize

2. 创建文件ObjectSizeAgent

   ```java
   package com.mashibing.jvm.agent;
   
   import java.lang.instrument.Instrumentation;
   
   public class ObjectSizeAgent {
       private static Instrumentation inst;
   
       public static void premain(String agentArgs, Instrumentation _inst) {
           inst = _inst;
       }
   
       public static long sizeOf(Object o) {
           return inst.getObjectSize(o);
       }
   }
   ```

3. src目录下创建META-INF/MANIFEST.MF

   ```java
   Manifest-Version: 1.0
   Created-By: konghenying
   Premain-Class: com.mashibing.jvm.agent.ObjectSizeAgent
   ```

4. 打jar包

5. 在项目中引入 agent jar包.

6. 在运行环境 vm options中重新制定javaagent  -javaagent:xx路径\agenct.jar

7. 启动测试类

   ```java
   public class T03_SizeOfAnObject {
          public static void main(String[] args) {
              System.out.println(ObjectSizeAgent.sizeOf(new Object()));
              System.out.println(ObjectSizeAgent.sizeOf(new int[] {}));
              System.out.println(ObjectSizeAgent.sizeOf(new MyObject()));
          }
      
          private static class MyObject {
                              //8 _markword
                              //4 _oop指针
              int id;         //4
              String name;    //4
              int age;        //4
              byte b2;        //1
              Object o;       //4
      
          }
      }
   Hotspot开
   ```

### 虚拟接hotspot开启内存压缩规则

1. 4G以内直接砍掉高32位
2. 4G~32G默认开启内存压缩ClassPointers Oops
3. 32G 不压缩

## Runtime Data Area

PC  程序计数器

> 存放指正位置
>
> 虚拟机的运行伪代码
>
> while(not end){
>
> 取PC中的位置,找到对应位置的指令;
>
> 执行指令;
>
> PC++;
>
> }

JVM Stack  栈

1. frame 栈针

   ​	一个方法就是一个栈针

   1.   Local Variables table:局部变量表
   2. Operand Stacks:操作栈
   3. Dynamic Linking :动态链接是一个将符号引用解析为直接引用的过程
   4. Return Address 返回一个操作栈的栈针 eg:a()->b() 方法a调用了方法b,b方法的返回值入栈.

**例子:**

```java
public static void main(String[] args) {
    int i = 4;
    i=i++;
    System.out.println(i);
}
结果为:4
查看命令:
 0 iconst_4 		//将4放入操作栈,
 1 istore_1			//出栈保存数据4到局部变量表中Nr为1的记录中.
 2 iload_1			//将局部变量Nr为1的记录4再入栈.
 3 iinc 1 by 1      //将局部变量Nr为1的记录增1.即局部变量表Nr.为1的位置数据为5
 6 istore_1         //出栈保存数据4到局部变量表Nr.为1的位置.
 7 getstatic #2 <java/lang/System.out>  //调用静态方法
10 iload_1    		// 将局部变量表中Nr.为1的数据入栈.
11 invokevirtual #3 <java/io/PrintStream.println>  //调用对象方法,并出栈方法个数个数据传递到下一个方法中,并保存在局部变量表中.
14 return
```



Heap 堆

Method Area 方法区

 1. Perm Space (<1.8)

    字符串常量位于PermSpace

    FGC不会清理

    大小启动的时候指定,不可改变

2. Meta Space (>=1.8)

   字符串常量位于堆中

   会触发FGC清理

   不设置则为最大物理内存

Runtime Constant Pool

Native Method Stack 

Direct Memory

> JVM可以直接访问的内核空间的内存(OS管理的内存)
>
> NIO,提高效率,实现Zero copy

验证: 字符串常量在1.7时在Perm,而1.8位于Heap?

> 一直创建字符串常量,观察堆与matespace的情况.或者等待报错.

### 常用指令

store   出栈,保存到局部变量表 eg:istore_1  将栈顶的int类型的数据出栈,保存到局部变量表中Nr.为1的位置.

load    入栈,从局部变量表中寻找记录再入栈 eg:iload_1 从局部变量表中Nr.为1的位置取一个int类型的数据并入栈.

invoke   调用后会将返回值入栈.

1. invokestatic  #2   调用静态方法
2. invokeVirtual         调用对象方法
3. invokeInterface
4. invokeSpecial:可以直接定位，不需要多态的方法 private 方法 ， 构造方法
5. invokeDynamic 

pop  弹出栈顶  当执行方法后,不接受返回数据.需要将invoke调用后入栈的数据再出栈 舍弃.

add,sub,mul,div  增减乘除   eg: iadd 将栈顶两个int型数据出栈,相加后的结果入栈.

# CG和CG Tuning

## CG基础知识

### 1.垃圾的定义:

​	没有任何引用指向的对象.

### 2.定位垃圾

1. 引用计数(ReferenceCount)

2. 根可达算法(RootSearching)

   >  whice instances are roots?
   >
   > JVM stack,
   >
   > nativce method stack,
   >
   > runtime constant pool,
   >
   > static references in method area,
   >
   >  Clazz
   >
   > 即:线程栈变量,静态变量,常量池,JNI指针;

### 3.GC Algorithms

1. mark sweep(标记清除) 位置不连续,产生碎片,两遍扫描(第一次标记,第二次清除),效率偏低.

   优点:算法相对简单,存活对象比较多的情况下效率较高.

2. copy(拷贝算法) 没有碎片,浪费空间,需要多一倍的内存用于拷贝.移动复制对象,需要调整对象引用.

   优点:适用于存活对象较少的情况,只扫描一次,效率较高,没有碎片.

3. mark compact(标记压缩) 没有碎片,效率偏低,扫描两次,需要移动对象.

   优点: 不会产生碎片,方便对象分配,不会产生内存减半.

### 4.JVM内存分代模型

1. 部分垃圾回收期使用的模型.

   除了 Epsilon ZGC Shenandoad之外的GC都是使用逻辑分代模型.

   G1是逻辑分代,物理不分代.

2. 新生代+老年代+永久代(1.7)Perm Generation/元空间(1.8)Meta Space

   ​	新生代(eden 8 : s1 1 : s2 1) 1 : 老年代 2     

   ```bash
   [root@iZbp1idck6s26toxblo39bZ ~]# java -XX:+PrintFlagsFinal -version|grep NewRatio
       uintx NewRatio                                  = 2                                   {product}
   java version "1.8.0_231"
   Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
   Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
   ```

   

   

   1. 永久代 元空间 保存 class
   2. 永久代必须指定大小限制.元空间可以设置,若不设置为物理内存大小.
   3. 字符串常量 1.7保存于永久代中.1.8保存在堆中.

3. 新生代 = Eden + 2个suvivor区
   1. YGC回收之后,大多数的对象会被回收.或者的进入S0
   2. 再次YGC,活着的对象eden+s0->s1
   3. 再次YGC,活着的对象eden+s1->s0
   4. 年龄足够->老年代
   5. s区装不下 ->老年代

>  对象何时进入老年代
>
> 1. 超过XX:MaxTenuringThreshold指定次数(YGC)
>
>    - Parallel Scavenge 15
>    - CMS 6
>    - G1 15
>
>    15的由来:markword中分代年龄占4位.
>
> 2. 动态年龄
>    - s1+eden->s2超过50%, 把年龄最大的放入老年代

4. 老年代

   1. 顽固分子,内存过大元素
   2. 老年代满了 FGC(Full GC) 

5. GC Tuning(Generation)

   1. 尽量减少FGC
   2. MinorGC = YGC


### 5.常见的垃圾回收器

![GC](/images/GC.png)

-- jdk诞生Serial追随提高效率,诞生了PS,为了配合CMS,

1. Serial: a stop-the-word(STW),copying collector which uses a single GC thread.(单线程清理垃圾) 年轻代 串行回收

   > -------->|				|-------->|				|
   >
   > -------->|<font color=#FF8C00>-----></font>|-------->|<font color=#FF8C00>-----></font>|
   >
   > -------->|				|-------->|				|

2. Parallel Scavenge: a stop-the-word,copying collector which users multiple GC threads.(多线程清理垃圾)年轻代 并行回收

   >-------->|			    |-------->|				|
   >
   >-------->|<font color=#FF8C00>-----></font>|-------->|<font color=#FF8C00>-----></font>|
   >
   >-------->|<font color=#FF8C00>-----></font>|-------->|<font color=#FF8C00>-----></font>|
   >
   >-------->|				|-------->|				|

3. ParNew: 

4. CMS: concurrent mark sweep.a mostly concurrent,low-pause collector. 老年代 

   >-------->|				|-------->|				|-------->|
   >
   >-------->|<font color=#FF8C00>-----></font>|-------->|<font color=#FF8C00>-----></font>|-------->|
   >
   >-------->|				|-------->|<font color=#FF8C00>-----></font>|-------->|
   >
   >​						|				|<font color=#FF8C00>-----></font>        |				|<font color=#FF8C00>-----></font>        |
   >
   > ​         				初始标记  并发标记 		重新标记   并发清理
   >
   >1. 初始标记: 标记根对象
   >
   >2. 并发标记: 与工作线程并发进行,根据根查找法,找出根对象下的其他对象并标记
   >
   >3. 重新标记: 在并发标记的同时,工作线程制造了新垃圾或者将垃圾重新关联上,使得标记状态的改变,需要进行重新标记
   >
   >4. 并发清理:与工作线程并发进行,将没有标记的对象清理.会产生浮动垃圾.(在并发清理阶段工作线程产生新的垃圾). 
   >
   >CMS问题:
   >
   >1. Memory Fragmentation 内存碎片化
   >
   >   -XX:+UseCMSCompactAtFullCollection -XX:+CMSFullGCsBeforCompaction  默认为0,指经过多少次FGC才进行压缩
   >
   >2. Floating Garbage 
   >
   >   PromotionFailed
   >
   >   在进行Minor GC时，Survivor Space放不下，对象只能放入老年代，而此时老年代也放不下造成的。
   >
   >   Concurrent Mode Failure
   >
   >   在执行CMS GC的过程中同时业务线程将对象放入老年代，而此时老年代空间不足，或者在做Minor GC的时候，新生代Survivor空间放不下，需要放入老年代，而老年代也放不下而产生的。stop-the-wold 降级为GC-Serail Old
   >
   >   解决方案: 降低触发CMS的阈值
   >
   >   -XX:CMSInitiatingOccupancyFraction 92%  值当老年代已用空间占比达到阈值时,触发CMS GC.调小这个值,可以保证老年代有更多足够的空间.
   >
   >Remark阶段的算法
   >
   >三色扫描算法
   
5. **G1**:分而治之

   1. 特点:

      1. 并发收集
      2. 压缩空闲空间不会延长GC的暂停时间
      3. 更易预测的GC暂停时间
      4. 适用不需要实现很高的吞吐量的场景.

   2. 产生FGC(串行回收)要怎么做:

      1. 扩内存
      2. 提高cpu性能,加快回收
      3. 降低MixedGC触发阈值,让MixedGC提交发生(默认45%)

      

## 常见垃圾回收器组合参数设定:(1.8)

- -XX:+UseSerialGC = Serial New(DefNew) + Serial Old

  小型程序,默认情况下不会是这种选项.HotSpot会根据计算及配置和jdk版本自动选择收集器.

- -XX:UseParNewGC = ParNew +SerialOld

  这个组合很少用,某些版本已经废弃.

- -XX:+UseConcMarkSweepGC = ParNew + CMS + Serial Old

- -XX:+UseParallelGC = Parallel Scavenge + Paralled Old(1.8默认)

- -XX:+UseParallelOldGC = Parallel Scavenge + Paralled Old

- -XX:+UseG1GC = G1

- Linux下1.8版本磨人的垃圾回收器到底是什么?

  - 1.8.0_181 默认 Copy MarkCompact
  - 1.8.0_222 默认 PS+PO

  如何查看版本信息:

  - java +XX:+PrintCommandLineFlags -version   windows中打印,linux没有找到默认GC的查看方法.
  - 通过GC的日志分析

### JVM常用命令行参数

HotSpot参数分类:

>cmd中敲java
>
>跳出来很多参数
>
>其中:
>
>1. 以 - 开头的为标准参数,所有版本都支持
>
>2. 以 -X开发非标参数,有些版本有有些版本没有
>
>   eg:
>
>   -Xms<size> set initial Java heap size  设置初始堆的大小
>
>   -Xmx<size> set maximum Java heap size  设置堆大小的最大值
>
>3. 以-XX开头为不稳定参数,各个版本的命名可能不同. 下个版本可能取消
>
>   查看所有参数 : java -XX:+PrintFlagsFinal -version| [grep xxxx]

Java -version

java -X

### 调优前的基础概念

1. 吞吐量: 用户代码时间/(用户代码执行时间 + 垃圾回收时间)
2. 响应时间: SWT越短,响应时间越好

调优: 确定追求啥?优先吞吐量还是优先响应时间?或在满足一定的响应时间的情况下,要求达到多大的吞吐量.

吞吐量优先: (PS+PO) 科学计算,数据挖掘.

响应时间:(1.8G1) 网站GUI API

### 什么是调优?

1. 根据需求进行JVM规划和预调优
2. 优化运行JVM环境
3. 解决JVM运行过程中出现的各种问题

### 调优规范

- 调优,从业务场景开始

- 无监控,不调优

- 步骤:

  1. 熟悉业务场景(没有最好的垃圾收集器,只有最适合的垃圾回收器)

  2. 选择回收器组合

  3. 计算内存需求

  4. 设定年代大小,升级年龄

  5. 设定日志参数

     1. -Xloggc:/opt/xxx/logs/xxx-gc-%t.log  按照系统时间生成日志文件名.

        -XX:+UserGCLogFileRotation      循环使用GC日志文件

        -XX:NumberOfGCLogFiles=5       最大GC文件数5个

        -XX:GCLogFileSize=20M				每个GC日志文件20M

        -XX:+PrintGCDetails						打印GC详情

        -XX:+PrintGCDateStamps			打印GC

        -XX:+PrintGCCause						打印GC原因

     2. 每天生成一个日志文件

  6. 观察日志情况

### 命令

1. top查找出cpu占比问题进程

2. top -Hp pid 列出问题进程的所有线程,观察cup占比. 列表中的pid为十进制的线程id.

3. jstack pid 列出进程的所有线程信息,比如现成的状态线程号码 #56,nid = 0x671  十六进制的线程id.终点关注:WAITING BLOCKED

   eg: -waiting on <0x0000000088c3310>(a java.lang.Object)

   当很多线程都外waiting on <xxx>, 一定要找到是哪个线程持有这把锁.

   搜索 jstack dump



4. jps列出所有的java进程

5. jinfo pid 列出java进程的运行环境

6. jstat -gc [pid]  [xx(ms)] 动态观察gc情况,每xx毫秒打印GC情况.

7. jmap -histo pid | head -20 ,查找有多少对象产生,列出前20条()

8. jmap -dump:format=b,file=xxx pid

   线上系统,内存才能特别大,jmap执行期间会对进程产生很大影响.甚至卡顿.

   1：设定了参数HeapDump，OOM的时候会自动产生堆转储文件 

   2：很多服务器备份（高可用），停掉这台服务器对其他服务器不影响 

   3：在线定位(一般小点儿公司用不到)

9. java -Xms20M -Xmx20M -XX:+UseParallelGC -XX:+HeapDumpOnOutOfMemoryError xxx.jar     用于差生dump文件
10. 使用MAT/jhat/jvisualvm进行dump文件分析
    1. jhat:
       1. heapdump 创建出堆文件
       2. jhat分析后启动web服务用于查询.  jhat -J-mx512M xxx.dump   -> http://ip:7000
       3. 拉到最后:找到对应链接,可以使用OQL查找特定问题对象.
    2. jvisualvm:
       1. 导出hprof文件
       2. 装入jvisualvm,即可进行图形化操作
       3. 查看类,实例数,OQL查询,查看线程状况,查看最大的对象.

### arthas

- jvm观察jvm信息
- thread定位线程问题
- dashboard 观察系统情况
- heapdump + jhat分析
- jad 反编译 
  - 动态代理生成类的问题定位
  - 第三方类的代码观察
  - 版本问题,确定代码时最新
- redefine 热替换
  - 限制条件:只能改方法实现,不能改方法名与属性.
- sc -search calss  找到类的特定信息
- watch  观察入参与结果
- 没有jmap

## 调优案例

OOM产生的原因多种多样,有些程序未必产生OOM,但会不断FGC.

## GC算法的基础概念

**card table:**

**CSet:** Collection Set 一组可以被回收的分区的集合.

**RSet:** RememberSet 记录了其他Region中的对象到本Region的引用.

**humongus object:** 大对象. 超过单个region的50%

**三色标记法:**

1. 白色:未被标记的对象
2. 灰色:自身被标记,成员变量未被标记
3. 黑色:自身与成员变量均已标记完成.

**漏标:**

 	1. 黑色对象指向了白色对象.
 	2. 指向白色对象的引用被删除了.

   打破漏标:

1. incremental update -- 增量更新,关注引用的增肌,把黑色重新标记为灰色,下次重新扫描属性.(CMS使用)

2. STAB snapshot at the beginning -- 关注引用的删除,当B->D的引用消失时,把这个引用推到GC的堆栈,保证D还能被GC扫描到.(G1使用)

   G1由于有Rset存在,不需要扫描真个堆去查找指向白色的引用,效率比较高.







GC常用参数

- -Xmn -Xms -Xmx -Xss

  年轻代  最小堆 最大堆  栈空间

  

CMS常用参数

- -XX:+userConcMarkSweepGC



