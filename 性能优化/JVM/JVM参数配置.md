---
title: JVM参数配置
---



```
-Xms：堆的起始内存（start）
-Xmx：堆的最大内存（max）
-Xmn：堆的新生代内存（new）

// 查看本地堆空间配置大小和实际大小
jps // 找到对应的线程ID，比如：18484
jmap -heap 18484 // 查看本地堆空间配置大小和实际大小

```




```
[root@iZbp1ftqbb7jrcdxek68zzZ ~]# jps
28211 Jps
18484 jar
14295 Bootstrap
32168 Bootstrap
15918 Bootstrap
[root@iZbp1ftqbb7jrcdxek68zzZ ~]# jmap -heap 18484
Attaching to process ID 18484, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.191-b12
using thread-local object allocation.
Parallel GC with 4 thread(s)
Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2051014656 (1956.0MB)
   NewSize                  = 42991616 (41.0MB)
   MaxNewSize               = 683671552 (652.0MB)
   OldSize                  = 87031808 (83.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)
Heap Usage:
PS Young Generation
Eden Space:
   capacity = 479723520 (457.5MB)
   used     = 248229248 (236.7298583984375MB)
   free     = 231494272 (220.7701416015625MB)
   51.74423134392077% used
From Space:
   capacity = 9437184 (9.0MB)
   used     = 8692896 (8.290191650390625MB)
   free     = 744288 (0.709808349609375MB)
   92.11324055989583% used
To Space:
   capacity = 10485760 (10.0MB)
   used     = 0 (0.0MB)
   free     = 10485760 (10.0MB)
   0.0% used
PS Old Generation
   capacity = 127926272 (122.0MB)
   used     = 23292304 (22.213272094726562MB)
   free     = 104633968 (99.78672790527344MB)
   18.207600077644724% used
```