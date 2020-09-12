# jstat参数详细配置

### 类加载统计

```text
C:\Users\Administrator>jstat -class 12568
Loaded  Bytes  Unloaded  Bytes     Time
3526474 10356679.2  3369929 9923980.7 1574995.32
```

* Loaded:加载class的数量
* Bytes：所占用空间大小
* Unloaded：未加载数量
* Bytes:未加载占用空间
* Time：时间

### 编译统计

```text
C:\Users\Administrator>jstat -compiler 12568
Compiled Failed Invalid   Time   FailedType FailedMethod
   34811      2       0  2529.01          1 net/sourceforge/htmlunit/corejs/javascript/Parser nameOrLabel
```

* Compiled：编译数量。
* Failed：失败数量
* Invalid：不可用数量
* Time：时间
* FailedType：失败类型
* FailedMethod：失败的方法

### 垃圾回收统计

```text
C:\Users\Administrator>jstat -gc 12568
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
 0.0   53248.0  0.0   53248.0 997376.0 690176.0 3143680.0   517777.9  1180204.0 976772.7 353068.0 264585.4   4093  684.360   5      5.520  689.880
```

* S0C：第一个幸存区的大小
* S1C：第二个幸存区的大小
* S0U：第一个幸存区的使用大小
* S1U：第二个幸存区的使用大小
* EC：伊甸园区的大小
* EU：伊甸园区的使用大小
* OC：老年代大小（单位kb）
* OU：老年代使用大小
* MC：元空间大小
* MU：元空间使用大小
* CCSC:压缩类空间大小
* CCSU:压缩类空间使用大小
* YGC：年轻代垃圾回收次数
* YGCT：年轻代垃圾回收消耗时间
* FGC：老年代垃圾回收次数
* FGCT：老年代垃圾回收消耗时间
* GCT：垃圾回收消耗总时间

### 堆内存统计

```text
C:\Users\Administrator>jstat -gccapacity 12568
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
     0.0 4194304.0 1323008.0    0.0 57344.0 1265664.0        0.0  4194304.0  2871296.0  2871296.0      0.0 1902592.0 1206828.0      0.0 1048576.0 353068.0   4095     5
```

* NGCMN：新生代最小容量
* NGCMX：新生代最大容量
* NGC：当前新生代容量
* S0C：第一个幸存区大小
* S1C：第二个幸存区的大小
* EC：伊甸园区的大小
* OGCMN：老年代最小容量
* OGCMX：老年代最大容量
* OGC：当前老年代大小
* OC: 当前老年代大小（单位Kb）
* MCMN:最小元数据容量
* MCMX：最大元数据容量
* MC：当前元数据空间大小
* CCSMN：最小压缩类空间大小
* CCSMX：最大压缩类空间大小
* CCSC：当前压缩类空间大小
* YGC：年轻代gc次数
* FGC：老年代GC次数

### 新生代垃圾回收统计

```text
C:\Users\Administrator>jstat -gcnew 12568
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
   0.0 57344.0    0.0 57344.0 15  15 51200.0 1265664.0 942080.0   4095  684.772
```

* S0C：第一个幸存区大小
* S1C：第二个幸存区的大小
* S0U：第一个幸存区的使用大小
* S1U：第二个幸存区的使用大小
* TT: 对象在新生代存活的次数
* MTT: 对象在新生代存活的最大次数
* DSS: 期望的幸存区大小
* EC：伊甸园区的大小
* EU：伊甸园区的使用大小
* YGC：年轻代垃圾回收次数
* YGCT：年轻代垃圾回收消耗时间

### 新生代内存统计

```text
C:\Users\Administrator>jstat -gcnewcapacity 12568
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
       0.0  4194304.0  1056768.0      0.0      0.0 4194304.0 104448.0  4194304.0   952320.0  4097     5
```

* NGCMN：新生代最小容量
* NGCMX：新生代最大容量
* NGC：当前新生代容量
* S0CMX：最大幸存1区大小
* S0C：当前幸存1区大小
* S1CMX：最大幸存2区大小
* S1C：当前幸存2区大小
* ECMX：最大伊甸园区大小
* EC：当前伊甸园区大小
* YGC：年轻代垃圾回收次数
* FGC：老年代回收次数

### 老年代垃圾回收统计

```text
C:\Users\Administrator>jstat -gcold 12568
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
1264684.0 1075406.7 353068.0 293886.6   3090432.0    516754.0   4098     5    5.520  691.236
```

* MC：方法区大小
* MU：方法区使用大小
* CCSC:压缩类空间大小
* CCSU:压缩类空间使用大小
* OC：老年代大小
* OU：老年代使用大小
* YGC：年轻代垃圾回收次数
* FGC：老年代垃圾回收次数
* FGCT：老年代垃圾回收消耗时间
* GCT：垃圾回收消耗总时间

### 老年代内存统计

```text
C:\Users\Administrator>jstat -gcoldcapacity 12568
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
        0.0   4194304.0   3393536.0   3393536.0  4099     5    5.520  691.566
```

* OGCMN：老年代最小容量
* OGCMX：老年代最大容量
* OGC：当前老年代大小
* OC：老年代大小
* YGC：年轻代垃圾回收次数
* FGC：老年代垃圾回收次数
* FGCT：老年代垃圾回收消耗时间
* GCT：垃圾回收消耗总时间

### 元数据空间统计

```text
C:\Users\Administrator>jstat -gcmetacapacity 12568
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT
       0.0  1990656.0  1294892.0        0.0  1048576.0   353068.0  4100     5    5.520  691.883
```

* MCMN: 最小元数据容量
* MCMX：最大元数据容量
* MC：当前元数据空间大小
* CCSMN：最小压缩类空间大小
* CCSMX：最大压缩类空间大小
* CCSC：当前压缩类空间大小
* YGC：年轻代垃圾回收次数
* FGC：老年代垃圾回收次数
* FGCT：老年代垃圾回收消耗时间
* GCT：垃圾回收消耗总时间

### 总结垃圾回收统计

```text
C:\Users\Administrator>jstat -gcutil 12568
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00 100.00  31.67  14.19  86.28  87.28   4101  686.661     5    5.520  692.181
```

* S0：幸存1区当前使用比例
* S1：幸存2区当前使用比例
* E：伊甸园区使用比例
* O：老年代使用比例
* M：元数据区使用比例
* CCS：压缩使用比例
* YGC：年轻代垃圾回收次数
* FGC：老年代垃圾回收次数
* FGCT：老年代垃圾回收消耗时间
* GCT：垃圾回收消耗总时间

### JVM编译方法统计

```text
C:\Users\Administrator>jstat -printcompilation 12568
Compiled  Size  Type Method
   34827   3384    1 com/gargoylesoftware/htmlunit/Cache getCacheEntry
```

* Compiled：最近编译方法的数量
* Size：最近编译方法的字节码数量
* Type：最近编译方法的编译类型。
* Method：方法名标识。

