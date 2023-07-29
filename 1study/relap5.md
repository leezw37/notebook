

# 目录

[TOC]

# 简介

1. RELAP-5是轻水堆冷却系统事故工况的瞬态行为最佳估算程序，初始由爱达荷国家实验室（INL，前身INEEL）为美国核管会（NRC）开发，用于规则制定，许可审查计算，事故减缓措施评估，操作员规程评价，和实验计划分析。
2. 集两流体、六方程水力学、一维热传导、点堆中子动力学于一体的系统分析程序。
3. 该程序除拥有泵、导管、阀门、喷射泵、透平、汽水分离器和控制系统部件等通用部件模型外，还包括再淹没传热、气隙导热、壅塞流、非凝气体等特殊过程模型。
4. 该程序可用来模拟轻水堆系统的冷却剂丧失事故 (LOCA) 、丧失流量事故(LOFA)、蒸汽发生器传热管破裂事故（SGTR）、主蒸汽管道破裂事故（MSLB）等事故及某些非核的水蒸气系统的热工水力瞬态性状，是目前核电安全中物理模型较完善的大型程序之一。

## 参考文档

1. input manual，学习输入卡编写（参看 volume 2）
2. volume 1+4，学习程序算法、数学模型、本构模型
3. 其他：v2 部件模型，v3 模型评价，v5建模指导，v6数值计算方法验证，v7程序整体验证，v8程序结构

## 程序文件内容

运行程序必须有indta、Relap.exe、tpfh2onew，其他文件运行时生成。

1. indta， 输入卡，input data
2. Relap.exe， 可执行程序
3. outdta， 输出文件，output data，包含：
   - 输入编辑，包括输入卡原文、填充解读、高度检查、初始水力部件编辑，初始热构件编辑
     - 原文，`listing of input data for case  `
     - 填充解读， `0Processing type is `
     - 控制体和接管的高度检查，`Reference volumes and positions for each loop and segment for each hydrodynamic system for z coordinate direction`
     - 处理输入后的水力部件编辑，像大编辑一样输出所有变量，相当于初始条件。`Edit of hydrodynamic components after input processing`
     - 处理输入后的热构件编辑，像大编辑一样输出所有变量，相当于初始条件。`Edit of heat structures after input processing`
     - 成功处理了输入卡的提示，`0$*8 Input processing completed successfully.`
   - 小编辑
   - 大编辑，包括整体信息、水力控制体、水力接管、热构件、再启动编号、再淹没等
     - 占据outdata文件内容的大部分，基本囊括了所有需要用到的信息和变量数据。`0MAJOR EDIT !!!time=    0.00000     sec`
     - 整体信息，包括时间步长、运行时间、Trip卡为真时刻、堆芯功率信息等
     - 水力部件，参数从左到右，构件编号从小上到大。`0   Vol.no.  pressure`，`0   Jun.no.   from vol.`。
     - 热构件，`0 HEAT STRUCTURE OUTPUT +++time=     0.00000     sec`
     - 再启动编号和再启动总结，`0---Restart no.       0 written, block no.     0, at time=     0.00000    ---`，以及`0---Restart Summary:       41 blocks written in restart file  ---`
   - 报错诊断编辑
4. rstplt， 再启动（和绘图）文件，restart and plot，包含所有指定时刻运行的原始非文本数据，作为再启动工况或者绘图时的输入
5. tpfh2onew， 水物性文件，thermodynamic property file of new light water，不带new为旧版轻水物性。
6. screen，运行日志，运行时shell框体输出的内容
7. read_steam_comment.o，运行时读取物性生成的



#  编写输入卡indta

## 基本框架&小贴士

```idl
= 问题标题;  title

控制选项;  Control options (cards 100-199)
时间步选项;  Time step options (cards 200-299)

* 小编辑;  Minor edits (301-399)
* 绘图请求;  Plot requests (20300XXY)

动作信号; Trips (400-799 or 20600000-20620000)

水力部件;  Hydrodynamic components (CCCXXXX)

热构件;  Heat structures (1CCCGXNN)

热构件物性; Heat structure thermal property data（201MMMNN）

通用表;  General tables（202TTTNN）

* 控制系统;  Control system (205CCCNN or 205CCCCN); 可以算术计算、泵控制、蒸汽控制 、给水控制
* 反应堆中子动力学;  reactor kinetics (30000000-30399999)
* 控制绘图变量;  Expanded plot variables (2080XXXX)

.End of input.
```



1. 类控制体部件输入顺序：

   - 面积; 长度; 体积; 方位角; 垂直倾斜角; 高度变化; 壁面粗糙度; 水力直径; 控制体控制标记 tlpvbfe; 控制体初始条件控制标记 εbt。
   - 此处指 snglvol, tmdpvol, pipe, annulus

2. 四种常见控制标记

   - 控制体控制标记 tlpvbfe
       - t0不用热前沿追踪跟踪模型，l0不用mixture level混合物水平面追踪模型，p0使用water packing scheme水填充方案，v0不使用垂直分层模型，b0使用管道相间摩擦模型，f0沿体积的x坐标计算壁摩擦效应，e0使用非平衡(不相等温度)计算
   
   
   - 接管控制标记 jefvcahs
   
        - j0不是jet喷放接管，e0不用能量方程PV修正项，f0不用CCFL模型，v0不用水平分层夹带模型，c0开启壅塞choking模型（临界流？无1卡/1卡无50选项则为Henry-Fauske模型，否则是relap5模型），a0光滑变截面，h0非均相流，s0同时使用来/去向部件的动量通量
   
        - 跟随3个临界流参数
   
        - 临界流常数1（取决于参数6：Henry-Fausk耗散常数discharge默认1.0，relap5模型过冷耗散常数默认1.0）; 
   
        - 临界流常数2（Henry-Fausk未使用此值0.0，Relap5过热耗散常数默认1.0）; 
   
        - 临界流常数3（Henry-Fausk热力非平衡常数默认0.14，Relap5模型默认1.0）; 
   
   - 控制体初始条件控制标记 εbt
   
        - （3表示单组分水/蒸汽，用后面两参数压力、温度。其中t=0-3单组分，4-6双组分有不凝性气体。）
   
   
      - 接管初始条件控制标记 0/1
          - （1表示后面某两个参数是水、汽质量流量kg/s，0则为速度）
   



## 标题-控制-时间100~299

写在输入卡前面的杂项

```idl
* title card 题目卡; A1.3
= Bennett 5253

* 问题类型、选项（new、restart类型有瞬态transnt/稳态STDY-ST， strip有格式化输出fmtout/二进制binary）
100 new  transnt

* input / output units,A2.4,国际单位si， 英制单位british
102 si   si

* 不凝性气体种类：氩ARGON,氦HELIUM,氢HYDROGEN,氮NITROGEN,氙XENON,氪KRYPTON,空气AIR,六氟化硫SF6.
110    nitrogen

* time step control card; A3.2
* 运行结束时间; 最小时间步长（>1e-6不会生效）; 最大时间步长（申请）; 控制选项（一般3即可，半隐）; 
 		*小编辑和绘图输出频率（每‘多少步*最大时间步长’输出一次）; 大编辑输出频率; 再启动文件（rstplt）输出频率
201   40.0  1.0e-7 0.01 3 1 100 100 

*卡1，控制选项A2.1：50在所有启用了choking模型的接管上激活原始的RELAP5临界流模型，
		*而不是默认的Henry-Fauske临界流critical flow模型。
1    50

```



## Trips动作信号卡400-799

```idl
*变量Trips卡A5.3: 401-599 or 20600010-20610000

*变量; 参数; 关系运算符(lt,le,eq,ne,gt,ge，即less,than,equal,not,greater); 
		*变量; 参数; 额外内容; 开关指示（L一开常开，N始终判断（每个时间步））
*此处定义布尔值401：时间大于等于1000s后永远为真1。
401 time     0         ge  null       0       1000.0      l
*定义布尔值450、451：每当控制体变量265小于等于14.0时450为真1，大于等于67.7时451为真1
450 cntrlvar      265 le null       0       14.0      n   * 265稳压器低水位
451 cntrlvar      265 ge null       0       67.7      n   * 稳压器高水位

*逻辑Trip卡A5.4: 601-799 or 20610010-20620000
*变量Trip卡号; 逻辑运算符（and和，or或，xor异或）; 变量Trip卡号; 开关指示（L一开常开，N始终判断（每个时间步））
*此处定义布尔值689，实则=非401（例如，此处用于trip阀门开关信号，小于1000s阀门打开）
689 -401     or       -401    n
*此处非450是>14%,非451是<67.7%，两者和运算结果为布尔值650，即14%<稳压器水位百分比<67.7%时为真1。
650   -450   and    -451   n    *14%<pzr-l<67.7%
```



## 水力部件CCCXXXX(7位)

水力部件卡号形如CCCXXXX，CCC即部件编号。

### snglvol单控制体

“snglvol”单控制体，最简单的流体流通通道，在不需要分支部件和管道（可以看做单控制体和单接管的组合）部件时使用。

```idl
1450000   upplnm2      snglvol
*流通面积; 长度; 体积; 方位角; 倾斜角
1450101   7.2707       1.08208       0.0         0.0     90.0
*高度变化; 壁面粗糙度; 水力直径; 控制体控制标记 tlpvbfe
1450102   1.08208      6.3e-6        0.4265      00
*控制体初始条件标记（3单组分水/蒸汽，后面是压力和温度）
1450200   3            1.55790e7     600.466
```

### sngljun单接管

“sngljun”单接管，用来接通两个部件。

```idl
1270000  jieguan2  sngljun
*来向部件; 去向部件; 面积; 正向流动能量损失系数; 逆流能量损失系数; 接管控制标记 jefvcahs; 
		*789取决于6字段c标记（临界流模型是用的Henry-Fauske√还是relap5×，此处HF不使用9字段）
1270101  125010000  130000000  0.00012469  0.0  0.0  0  1.0  1.0
*接管初始条件标记（0则23变量是水、汽速度，1为质量流量）; 水质量流量; 汽质量流量; 界面速度
1270201  1  0.0  0.0  0.0
```

### tmdpvol时间控制体
“tmdpvol”时间相关控制体，定义边界条件，温度、压力等参数，与snglvol的不同在于其压力温度可随时间等参数变化

```idl
1100000  inlet   tmdpvol
*横截面积; 长度; 体积; 水平方位角; 竖直倾斜角; 高度变化; 壁面粗糙度; 水力直径; 控制体控制标记
1100101  0.00012469 0.048 0.0 0.0 90.0  0.048  0.0 0.0 0
*控制体初始条件控制标记; εbt格式; 3表示CCC0201第二三参数为压力和温度。
1100200  3
* 第一参数为搜索变量（如时间）; 二三由上指定为压力和温度
1100201  0.0  6.9e6  538.98
```
### tmdpjun时间接管
“tmdpjun”时间相关接管，定义流量边界条件，与sngljun的不同在于其流量可随时间等参数变化

```idl
1200000  jieguan1  tmdpjun
* 来向部件; 去向部件; 接管面积
1200101  110010000  125000000  0.00012469
* CCC0200卡省略或第一个参数为0则CCC0201的二三参数为液、汽速度，1则为液、汽质量流量。(参数二为Trip卡号)（参数三为？？？）
1200200  1
* 搜索变量; 此二三参数为液、汽质量流量kg/s = 质量流密度 * 流通面积; 界面速度0.0
1200201  0.0  0.1696  0.0  0.0

*举例CCC0200有第二个参数的TDJ
5700200   0         506
*搜索变量（此为506Trip卡的真假）; 液质量流量; 汽质量流量; 界面速度0.0
*理解：<=0.0（Trip假）时流量均为零; (0.0, 1.0]（Trip真）时液相流量设为0.2983 m^3。大于1.0沿用
5700201   0.0       0.0     0.0    0.0    
5700202   1.0    0.2983    0.0    0.0

*再举例CCC0200有第二个参数的TDJ？？？
7650200   1        760     
7650201   -1.0     0.0     0.0       0.0
7650202   0.0      551.0  0.0       0.0

*举例CCC0200有第三、四个参数的TDJ？？？

8120200   0  519    p            245010000
8120201  -1.0       0.0    0.0 0.0  
8120202   0.0       0.0    0.0 0.0
8120203   0.11743e6 1.6027 0.0 0.0    
8120204   0.49908e6 1.3555 0.0 0.0
```

### pipe管道
“pipe”管道，含有1~99个子控制体，具体个数由自己定义，各子控制体之间有内部接管相连，相当于一定数量的snglvol和sngljun的集合体

```idl
1250000  guanzi pipe
* 控制体数量<100
1250001  40
* 控制体面积
1250101  0.00012469  40
* 内部接管面积，201-299
* 控制体长度，控制体编号，301
1250301  0.1385     40
* 体积401
* 方位角501
* 垂直倾角601（90.0向上，0.0水平，-90.0向下），
1250601  90.0      40
*高度变化
1250701  0.1385     40
* 壁面粗糙度、水力直径、编号
1250801  0.0  0      40
* 内部接管顺流能量损失系数; 逆流能量损失系数; 接管编号。
1250901  0.0  0.0  39
* 控制体控制标记 tlpvbfe
1251001  00        40
*接管控制标记
1251101  00000000    39
* 控制体初始条件控制标记 εbt 格式; 参数二到六初始条件数据，此为压力温度; 参数7控制体编号
1251201  3  6.9e6  538.98  0  0  0  40
* 内部接管初始条件控制编号（0则1301为液相/气相/相界面速度0.0/编号，1则前两项为质量流量）
1251300  1
1251301  0.0   0.0   0.0  39
*接管水力直径; 淹没模型（0使用Wallis CCFL，1Kutateladze，0-1则是两者的Bankoff权重因子）; 
		*CCFL关系式中气体截距为1（默认）; CCFL中斜率为1（默认）; 接管编号。
1251401   0.0126   0.0    1.0     1.0     39
```

### mtpljun多接管

“mtpljun”多接管，连接多个控制体，A9.18。包含的单接管编号为CCCJJNN00

```idl
1010000   dn1-1-2     mtpljun
*接管数量; 初始条件控制字1（表示CCC1NNM中为水、汽速度，NN组号，M卡号，此处1011011）
1010001   2           1
*来向控制体; 去向控制体; 面积; 顺流能量损失系数; 逆流能量损失系数; 接管控制标记 jefvcahs，多接管jv=0
1010011   100010006   102010005   0.4893   5.0   5.0   01003
1010021   100020006   102020005   0.33     5.0   5.0   01003
*接上面CCC0NNM参数7-13，其中临界流常数789取决于上面控制字6
*临界流常数1; 常数2; 常数3; 来向控制体编号增量（用于批量定义多个接管）; 去向编号增量; 0; 接管编号。
1010012   1.0         1.0         1.0      0     0     0     1
1010022   1.0         1.0         1.0      0     0     0     2
*CCC1NNM:水、汽速度（视1010001规定）
1011011   0.0         0.0         2
```

### annulus环管

“annulus”特殊的管道部件，必须在竖直方向流动

```idl
1000000   dwcom1-1   annulus
*控制体数
1000001   2
*控制体面积; 编号
1000101   0.64436    2
*接管面积; 编号
1000201   0.64436    1
*控制体长度、编号（此处为1、2两个控制体长度不一样）
1000301   1.08208    1         1.00932    2
*体积
1000401   0.         1         0.         2
*水平方位角501
*竖直倾斜角
1000601   90.0       2
*高度变化
1000701   1.08208    1         1.00932    2
*壁面粗糙度、水力直径
1000801   6.3e-6     0.3304    2
*顺、逆流能量损失系数
1000901   0.0        0.0       1
*控制体标记 tlpvbfe
1001001   00         2
*接管控制标记 jefvcahs; 环jv=0
1001101   00         1
*控制体初始条件控制标记 εbt（3单组分水/蒸汽）; 压力; 温度; （参数未使用填充0.）; 控制体编号
1001201   3    1.58162e7    565.971    0.0    0.0    0.0    1
1001202   3    1.58086e7    565.968    0.0    0.0    0.0    2
*接管初始条件控制标记 0/1（1表示301卡是水、汽质量流量kg/s，0则为速度）
1001300   1
*接管初始条件：水质量流量; 汽质量流量; 相间速度未实现0; 接管编号
1001301   120.34     0.0     0.0    1
*接管水力直径; CCFL关系式（默认0使用Wallis模型，1则Kutateladze，0-1插值）; 气体截距1.0; 斜率1.0； 接管编号
1001401   0.3304     0.0     1.0    1.0    1
```

### branch分支

“branch”分支，最多可以有10个接管，相当于带有多个接管sngljun的单一控制体snglvol。

```idl
1060000   clgin-1     branch
*接管数4; 接管初始条件控制标记（决定CCCN201：0速度，1质量流量）
1060001   4           1
*面积; 长度; 体积; 方位角; 垂直倾斜角; 高度变化; 壁面粗糙度; 水力直径; 控制体控制标记
1060101   0.0         1.0439       0.8202      0.0    -90.0
1060102   -1.0439     6.3e-6       0.4008      00
*工质初始条件εbt=003单组分蒸汽/水（后面两个参数是PT）; 压力; 温度。
1060200   3           1.57999e7    565.965
*入口面; 出口面; 分支面积; 顺流能量损失系数AF; 逆流能量损失系数AR; 接管控制标记 jefvcahs分支j=0
1061101   250020002   106010001    0.3832     0.28    0.28   0000
1062101   106010001   100010001    0.0        0.0     0.0    0000
1063101   106010002   112010001    0.0        0.0     0.0    0000
1064101   106010006   108010005    0.5065     5.0     5.0    0000
*接管水力直径; CCFL关系式（默认0使用Wallis模型，1则Kutateladze，0-1插值）; 气体截距1; 斜率1
1061110   0.6985     0.0    1.0    1.0
1062110   0.4522     0.0    1.0    1.0
1063110   0.4902     0.0    1.0    1.0
1064110   0.0        0.0    1.0    1.0
*接管初始条件：液、气相质量流量（取决于CCC0001）; 界面速度。
1061201   4813.0     0.0    0.0
1062201   120.40     0.0    0.0
1063201   4692.6     0.0    0.0
1064201   0.0        0.0    0.0
```

### separator汽水分离器

“separatr”汽水分离器，特殊分支，模拟蒸汽发生器里的汽水分离器

```idl
5250000   separat1   separatr
*接管数=3; 接管初始条件控制标记（决定201参数：0速度，1质量流量。N即3个接管：1出口蒸汽2出口水3入口）
5250001   3          0
*面积; 长度; 体积; 方位角; 垂直倾斜角; 高度变化; 壁面粗糙度; 水力直径; 控制体控制标记
5250101   3.2973     2.3302     0.0      0.0     90.0     2.3302
5250102   6.3e-6     0.532      0000
*控制体初始条件εbt=000单组分蒸汽/水（后面4个参数是P,Uf,Ug,αg）; 压力; 液相内能; 气相内能; 空泡份额。
5250200   0    6.727480e6    1.24432e+06  2.58406e+06    0.75684

*出口蒸汽接管CCC1
*入口面; 出口面; 分支面积（0取相邻控制体的最小面积）; 顺流能量损失系数(AF); 逆流能量损失系数(AR); 
		*接管控制标记 jefvcahs 汽水分离器jefv=0; 
		*空泡份额限制（默认蒸汽接管0.5，水0.15，入口无。入口(?)大于此值则出口为纯蒸汽、纯水）
5251101   525010000  530000000  0.0      10.0     10.0     1100    0.50
*初始水、汽、相间速度0（取决于CCC0001）
5251201   -.28341       4.6902     0.0

*出口水接管CCC2、入口接管CCC3同上
5252101   525000000  505000000  0.0      5.0       5.0     1100
5252201   .52712      -4.9013       0.0
5253101   522010000  525000000  0.0      17.5       17.5     1100
5253201   1.7818       6.6357     0.0
```

### valve阀门

“valve”阀门，特殊接管，模拟六种不同类型的阀门及其动作

```idl
1360000  blpval  valve
*阀门来向组件; 去向组件; 面积; 前向能量损失系数A_F; 逆流能量损失系数A_R; 接管控制标记 jefvcahs 阀门j=0
1360101  260010001  212000000  1.60055e-3   0.0  0.0  0100
*接管初始条件控制标记（此处1，此后两参数为流量，0则为速度）; 初始液相流量; 初始气相流量; 界面速度0.
1360201  1  0.00  0.00  0.
*阀门类型（CHKVLV 止回阀Check; TRPVLV 跳闸阀Trip; INRVLV 惯性旋启式止回阀Inertial swing check; 
		*MTRVLV 电动阀Motor; SRVVLV 伺服阀Servo; RLFVLV安全阀Relief）
1360300  trpvlv
*Trip阀的开关信号卡编号（在Trips卡中定义为布尔值，例如小于1000s阀门打开）
1360301  689

*其他类型：电动阀，如稳压器安全阀
2770300    mtrvlv
*开启的Trip信号卡编号; 关闭的Trip卡号; 阀门开（关）变换速率（s^-1，此处无参数5，指归一化面积变化率，
		*（有参数5则指阀杆位置; 若输入参数6指开启时归一化面积变化率）; 
		*初始位置（无5指面积，有5指阀杆位置）; 
		*（参数5是阀门表编号，参数6阀门关闭速率）
2770301    683         685         0.6667     0.0
```

### pump泵

“pump”泵，模拟泵及其动作

```idl
2350000   rcppump2     pump
*面积; 长度; 体积; 水平方位角; 竖直倾斜角; 高度变化; 控制体控制标记 tlpvbfe
2350101   0.0          1.685         3.039    0.0    0.0
2350102   0.0          00000
*入口、出口控制体面; 面积; 正向能量损失系数; 逆流能量损失系数; 接管控制标记 jefvcahs
2350108   230060002    0.486945     0.0       0.0    0000
2350109   240010001    0.3832       0.146000       0.118000    0000
*控制体初始条件控制标记 εbt（3表示单组分水/蒸汽，用后面两参数压力、温度; 压力; 温度
2350200   3            1.55025e7    565.852
*入口、出口接管初始条件控制标记（0速度1流量）; 水流量; 汽流量; 相间速度未实现0
2350201   1            4813.00      0.0       0.0
2350202   1            4813.00      0.0       0.0
*泵单相数据表标记（0本组件给出单相表，正整数使用该编号代表的泵数据，-1Bingham泵，-2西屋泵）; 
		*两相系数表索引（-1无两相，>=0同上）; 两相扬程差difference表索引（-3无，>=0同上）; 
		*电机扭矩表索引（-1无，>=0同上）; 时间相关速度索引（-1无，>=0同上）; 
		*泵Trip卡编号（trip为真时断电！）; 逆流标记（0不允许，1允许）
2350301   0     0    0     -1   -1    411    0
*1额定转速rated pump velocity; 2初始转速因子（初值除以额定）; 3额定流量; 4额定扬程head; 5额定扭矩torque; 
2350302   157.0795    1.0        6.344444     96.228         37300.0
*6转动惯量moment of inertia; 7额定密度; 8额定电机扭矩（非零输入泵电机扭矩表，0由初始转速和泵扭矩计算）; 
		*9TF2摩擦扭矩系数（将真实转速与额定的比值乘以second power）; 10TF0摩擦扭矩系数（定值）; 
2350303   3800.0     742.0      0.0         120.0        0.0
*11TF1摩擦扭矩系数; *12TF3摩擦扭矩系数; （将真实转速与额定的比值乘以first、third power）
2350304   2500.0     0.0

*单相扬程曲线：曲线类型（1扬程2扭矩）; 曲线区域; 自变量（-1~0或0~1）; 因变量
2351100   1   1   0.0   1.84   0.1   1.74    0.2   1.64   0.3   1.55
2351101           0.4   1.46   0.5   1.38    0.6   1.32   0.7   1.26
2351102           0.8   1.19   0.9   1.1     0.95  1.05   0.98  1.02
2351103           1.0   1.0
*内容太多写不下曲线区域2~6。

*更多曲线与之类似，参考输入手册A9.17.16, page 208：单相扭矩曲线、两相扬程因子曲线、
		*两相扭矩因子曲线、两相水头差（head扬程,difference）曲线、两相扭矩差曲线。

```



## 热构件1CCCGXNN(以下均为8位)

热构件（A10，1CCCGXNN）：CCC是热构件编号，一般与水力部件相同，G几何编号用来区分接触同一水力控制体的不同热构件，X卡类型，NN该类型的编号

```idl
* 常见数据1CCCG000：轴向控制体数量（一般=水力部件控制体数）; 径向网格点数; 
		*几何类型（1矩形体2圆柱3球形）; 稳态初始化标记（0温度由1CCCG401-499给定，1程序计算）; 
		*左边界坐标（模拟管则为内径）; 再淹没条件标记（0无，1/2/trip卡号则需填写78字段）
11250000  40  5  2  0  0.0063  0
* 网格位置标记（0则在1CCCG101~201填网格信息，否则同该编号热构件且以下不用填); 
		*网格格式标记(1/2为卡1CCCG101-G199格式一/二，参数一是0时填写)
11250100  0  1
* 此为格式一：间隔interval数（网格点数-1）; 右坐标（模拟管则为外径）。格式二：网格间隔长度; 间隔编号
11250101  4  0.0083
* 构件成分201：材料编号（绝对值是201MMM00卡中的MMM，此处为20100500）; 间隔编号
11250201  5 4
* 热构件源项分布301：源项分布相对值（网格全1即平均分布）; 网格间隔编号
11250301  1.0  4
* 初始温度; 网格点编号
11250401  538.98  5
* 左、右边界条件501、601：边界相邻水力控制体/通用表编号的负值; 增量（一般10000，对应控制体12501/2/3/.../40）; 
		*边界条件类型（0对称/绝热，1/1nn对应多种对流边界，1xxx查总表xxx左/右边界温度，2xxx热流密度，3xxx对流系数随时间变化表，4xxx对流系数随壁面温度变化表）; 
		*表面积标记（决定后面参数5，0左表面积，1矩形体表面积/圆柱体高度/球形份额）; 
		*表面积（热构件控制体高度）/系数; 热构件编号
11250501  125010000  10000  1  1  0.1385  40
11250601  0  0  0  1  0.1385  40
* 热源类型（0无，1~999查总表，1000~1002中子动力学，10001~14095热源是控制变量，序号减10000); 
		*内热源乘子(轴向峰值因子可在该处输入，作用于参数一指明的功率); 
		*左边界慢化剂直接加热乘子（卷2节3.3）; 右边界慢化剂直接加热乘子; 热构件编号
11250701  975  1.0  0.0  0.0  40
* 其他左、右边界条件801、901：热当量直径; 向前加热长度（加热起始到板中心的距离，大于10入口段影响小）; 
		*回流加热长度（同理出口）; 向前定位架长度; 回流定位架长度; 定位架向前损失系数; 定位架回流损失系数; 
		*局部沸腾因子（=局部热流密度/沸腾开始后的平均热流密度，若无热源/局部平衡态含汽量<0时填1）; 
		*热构件编号
11250801  0.0  1.0  1.0  0.0  0.0  0.0  0.0  1.0  40
```

## 热构件物性201MMMNN

热构件热物性，A12，201MMMNN

```idl
* 热构件热物性，A12，201MMMNN
* 热构件材料成分类型，201MMM00 卡，内置碳钢C-STEEL，不锈钢S-STEEL，二氧化铀UO2和锆ZR
20100500   c-steel

* 也可以使用用户自定义表/函数TBL/FCTN，此时需要参数二三和201MMM01-49; 
		*热导率格式标记/gap 摩尔份额标记（1输入表，2函数，3包含气体组分名和摩尔份额的表）; 
		*体积热容标记（-1表只有热容，温度可以从热导率表对应，1输入包含温度、体积热容的表，2函数）
*20100500  TBL/FCTN  1  1
** AISI 304 e/L 不锈钢：温度（K）; 热导率（W/m•K）
*20100501 0.3 14.88
*20100502 343. 14.88
*20100503 373. 15.05
** AISI 304 e/L 不锈钢：温度（K）; 体积热容 (J/m^3/k)
*20100551 0.3    8.51e6
*20100552 343.   8.51e6
*20100553 373.   8.53e6
```


## 总表202TTTNN

总表数据A13，202TTTxx，TTT为查询编号。

```idl
* 表类型（POWER功率随时间变化，HTRNRATE热流密度-t，HTC-T传热系数-t，HTC-TEMP传热系数温度-t，
		*TEMP温度-t，reac-T 反应性随着时间/控制系统变量，NORMAREA 阀门归一化面积随阀杆位置）
20297500  power
* 时间; 功率（W，英制单位则为MW）（注意！！！功率是给热构件的这个控制体的，=功率密度*pi*内径*控制体高度）
20297501  0.0  4.973e3
20297502  5000.0   4.973e3
```

## 控制系统205CCCNN



```idl
*名称; 控制类型（SUM, MULT, DIV等）; 缩放系数S; 初值; 初值标记（0无初值计算用参数4; 1计算初值条件）
20501600  avgtemp     sum      0.5      0.0          1

*sum和类型：Y=S*(A0+A1*V1+A2*V2+...+Aj*Vj)。
*其中Y是最终结果，S是缩放系数; A是自定义常数; V是变量值。此处计算了平均温度。
*常数A0; 常数A1; 变量V1的名称; 变量V1的控制体编号
20501601  0.0         1.0      tempf    215010000
*常数A2; 变量V2的名称; 变量V2的控制体编号
20501602  1.0         tempf    225010000

*tripunit类型：Y=S*T1。T1是 ±(Trip卡号代表的布尔值)，负号在前则取括号内的非值)。
*同上
20573000   msrv    tripunit    1.0         0.0   0
*±Trip卡编号
20573001   730

*mult乘类型：Y=S*V1*V2*...*Vj
20551000    sm500        mult      22.7026       0.0     1
20551001    voidg        500010000           rhog    500010000

*div除类型：Y=S/V1 或者 Y=S/V1*V2
20549500  sgcr-c   div         1.0     0.0   1
20549501  mflowj   737000000   mflowj  715000000

*function函数类型：Y = S*f(V1)。f()函数在总表中定义，用于使用查表、插值函数、设置限值
20546200    sglev1    function   1.0    0.0     1
*变量V1的名称; 变量V1的控制体编号; 函数f()的总表卡号
20546201    cntrlvar  460        460

*stdfunc标准函数类型：Y = S*f(V1,V2,...)。如ABS,MIN,MAX,SQRT,EXP,LOG,SIN,COS,TAN,ATAN(即arctan)。
20540000 PCT1  stdfnctn   1.0  0.0     0     0
*函数名; 变量V1的名称; 变量V1的控制体编号; （V2名称; V2控制体编号; ...）
20540001           max      httemp   130000109
+                           httemp   130000209
+                           httemp   130000309
```


## 中子动力学30000000-30399999

用于模拟反应堆的裂变、衰变等核反应。

```idl
*中子动力学类型（只有点堆）; 反应性反馈类型（默认seperabl，独立看待慢化剂密度、空泡权重的慢化剂温度、燃料温度的反馈）
30000000  point       separabl
*裂变产物的衰变类型（no-gamma不计算，gamma标准裂变产物衰变计算，gamma-ac裂变产物衰变+锕系元素衰变）; 
		*总反应堆功率（裂变+裂变衰变+锕系元素衰变）; 初始反应性; 
		*缓发中子份额over 瞬发中子generation time (s-1).; 
		*裂变产物yield因子; U239 yield因子; 
30000001  gamma-ac    2.9529e9   0.0  241.9354  1.2   1.0  1.0  200.  day
*裂变产物类型ANS73,ANS79-1,ANS79-3; 每次裂变反应释放的能量（默认200 MeV/fission）; （其他类型见手册）
30000002  ans73       200.0
*缓发中子precursor yield ratio(1~50组); 缓发中子衰变常数（s^-1）
30000101  0.030530    0.0125
30000102  0.205070    0.0308
30000103  0.189070    0.1147
30000104  0.395470    0.3102
30000105  0.134530    1.2316
30000106  0.045330    3.2855
*反应性曲线总表卡号（0~999）/控制变量的卡号（>10000，查找时应该减去10000）
30000011  904
*慢化剂密度反馈：密度（线性插值，若范围超出最后数据点用最后的数据）; 反应性（dollar）。
*(此卡需要反应性反馈类型为seperabl，且30000701~709卡输入)
30000501   100. -26.95906667   
30000502   300. -6.137333333
30000503   500. -0.4472     
30000504   600. 0.164533333 
30000505   700. 0.           

*慢化剂温度反馈（多普勒反应性表）：温度; 反应性。(此卡需要反应性反馈类型为seperabl，且30000801~899卡输入)
30000601  473.15    3.44756 
30000602  973.15    0.34756 
30000603  1073.15  -0.27244
30000604  2873.15  -11.43244

*水力部件控制体编号; 编号增量; 密度反馈权重因子; 水温度系数(dollars/K, dollars/℉)
30000701  130010000   10000         0.100000    0.0

*热构件编号; 编号增量; 多普勒反馈权重因子; 燃料温度系数(dollars/K, dollars/℉)
30000801  1300001     10000         0.100000    0.0
```



## 结束

以英文句号`.`结束，后面的内容其实不会读入

```idl
.End of input.
```



# 处理报错

如果成功运行，会在outdta最后输出`0Transient terminated by end of time step cards.`，否则会输出`0******** Errors detected during input processing.`。

在outdta文件中搜索`0********`，找到第一个报错的位置。

常见报错如，

- 水力部件名超过8 个字符：`0******** Unrecognizable card number. `，`0******** Component type 9O on card 1100000 is illegal.`  ，`0******** Data for component 110 cannot be processed.`  ，水力部件名不得超过8 个字符。
- 无法识别Tab键：`0******** Unable to determine processing type on card 100, no further processing possible.`，RELAP输入卡中的tab符号无法被识别，使用空格替代。
- 环路高度不闭合：`******** Closure by junction 100030000 shown below is incorrect, position from loop is 0.00000 0.00000 0.00000 .`，如果闭合环路中的同一个控制体计算的高度相差大于0.0001m，则程序会提示环路高度不闭合。
  - outdta文件的输入编辑部分也有输出高度检查信息，形如`1         Reference volumes and positions for each loop and segment for each hydrodynamic system for z coordinate direction`。

# 运行程序

## 新程序运行

  1. 开始新的计算前需要删除已计算的数据，outdta 和 rstplt，然后双击`relap5.exe`。运行结束后会生成新的outdta 和 rstplt文件。

## 再启动运行

  1. 开始再启动计算（即，在之前的计算结果的基础上继续计算）前需要先备份outdta 和 rstplt文件，然后删除indta和outdta，但保留rstplt文件，再新建一张名为indta的再启动输入卡文件。
  2. 再启动卡填写：在100卡参数一填restart，103卡填再启动编号（-1表示最终时刻）。每个再启动编号都对应着一个时刻，再启动计算在此时刻之后继续。需要在原来的new问题输入卡中的201-299卡的最后一个（第七个）参数中设置再启动的输出频率。
  3. 再启动编号：可以在outdta最后找到再启动汇总信息，`0---Restart Summary:       41 blocks written in restart file  ---`。也通过在outdta文件中搜索restart在每个时刻大编辑输出的最后看到。如 `0---Restart no.    1467 written, block no.     4, at time=     4.00131    ---`，表示4.00131 s 的再启动编号为1467。



```idl
*问题类型、选项（new、restart类型有瞬态transnt/稳态STDY-ST，strip有格式化输出fmtout/二进制binary）
100   restart      transnt

*输入检查（run直接运行，inp-chk检查并处理完输入卡就停止）
101   run

*再启动编号（new类型不能用; -1表示最终时刻; 其他编号从outdta中找）
* 103  -1 

*结束时间，最小时间步，最大时间步，控制字，小编辑频率，大编辑频率，再启动频率
0000201  1500.0   1.0e-10     0.01   3   100   1000   100

.end of input


```



## strip后处理运行

参考vol 2，sec 8.4。

开始strip 卡计算输出所需参数前，类似再启动，也需要先备份outdta 和 rstplt文件，然后删除indta和outdta，但保留rstplt文件（从中读数据后处理），再新建一张名为indta的strip后处理输入卡文件。运行`relap.exe`后输出stripf文件，其输出频率就是小编辑和绘图频率。需要原来的new问题输入卡中的201-299卡中的第5个参数中设置。

```idl
= Strip from SFP
100 strip fmtout
* 对于strip输入卡，103卡再启动号必为0
103 0
*==================Parameters=============
* 变量代码，参数。例如p表示控制体压力; cntrlvar表示后面控制体的控制变量，在控制系统205CCCNN中定义
1001  p  260010000  *稳压器压力，260稳压器部件的第一个控制体的压力变量
1002  cntrlvar  260  *控制变量的编号，260在输入卡中定义为稳压器液位
.end of input
```



# 重难点及杂项

需要注意的点（从新到旧，从难到易）：

1. 
2. 动量方程的控制体
   - relap5的动量方程的控制体是相邻控制体各占一半，与质量、能量方程不同。
   - 所以可以看到大编辑中速度、阻力系数等出现在接管变量部分，而空泡份额、压力、内能等则在控制体变量部分。
3. 一般边界条件是流量入口和压力出口：
   - 入口一般设置为tmdpvol+tmdpjun。其中，tdv提供由压力和温度计算的密度，tdj提供速度，相乘得到流量入口条件。两相tdv还提供空泡份额。
   - 出口设置为sngljun+tmdpvol 。sj没用，tdv提供
   - 入口tmdpvol+sngljun应该是压力入口。
4. 时间步控制卡201中的输出频率，是指每隔多少时间步输出一次。
5. 参考控制体：
   - 用于程序中高度计算的基准，在 120 -129卡指定将其控制体中心设置为某一高度。
   - 如`120 280010000 0.0 h2o `，指定28001控制体为参考控制体，其中心高度为0，系统工质为轻水。
   - 没有指定则默认控制体编号最小的为参考控制体。
6. 接管连接控制体的两种写法：
   - （输入手册A9，P51第二段）传统是偷懒只写出待连接组件的01出口/00进口（注意面的区别）。
   - 建议写法是详细给出待连接组件的控制体的面，更好理解。
   - 如接管要连接110010000  125000000，建议写成 110400002  125010001。即，传统的写法：连接110组件和125组件的出口面01入口面00; 建议写法：连接110组件第40控制体的出口面2和125组件01控制体的入口面1。
7. 控制体和接管的位置坐标：
   - 控制体是RELAP5的基本计算单元，是用于搭建具体装置的基本单元。控制体的坐标设置在控制体的几何中心。
   - 接管用于连接控制体。而接管的坐标设置在两个控制体的连接处，实际上无实体。
8. 控制体的方向与面：

   - x方向始终是流动方向，面1、2始终是流体的进、出口

   - 水平控制体：x、y、z方向分别对应笛卡尔坐标系的x、-y、z方向。face 1左2右（3前4后5下6上）。
   - 竖直控制体：x、y、z方向分别对应笛卡尔坐标系的z、-x、-y方向。face 1下2上（3前4后5右6左）。
9. 热构件的左右边界：
   - 左/右边界表示”内/外侧“。例如常用的圆柱/圆环柱，可以分别模拟加热棒和管壁，其左边界就是加热棒的中轴线、管子的内壁面。
10. 0和0.0代表默认值。
11. 注释用`*`或者`$`，终止符为点号`.`。
    - 由于输出文件利用`0********`报错和`0$*8`输出控制体高度检查信息，且输出文件会复制输入卡，因此应该避免连续使用`*`或者`$`，以免混淆。
12. 字符数限制
    - 输入卡每行不得超过80字符，但是在下一行的行首使用加号 `+` 就可以继续写。
13. 数据类型要对应。
    - 特别是要求输入real浮点数类型的时候不要输入整数（`50.`正确，`50`错误）。
    - 输入卡的数据类型分三种：整数integer，浮点数real和含有字母数字的alphanumeric。
14. 连续扩充格式简化输入。
    - 例如，125的pipe部件有40个控制体，`1250101  0.1  20`，`1250102  0.2  40`中的参数分别代指`卡号、控制体面积、控制体编号`，可以表示前20个控制体的面积都是0.1，21~40的控制体面积都是0.2。



# 实例——单管加热

单管加热Bennett 干涸后传热工况简介：

1. 计算40s，步长0.01s，每步一组绘图数据点，每秒输出一次（大编辑），每秒保留一次再启动数据。
2. 水经过一段长为5.54m、内径6.3e-3 m，面积1.2469e-4  m^2的管道加热后竖直向上流动，加热功率4.973e3 W。
3. 管内水力构件和管壁热构件都沿轴向均分为40个控制体。
4. 边界条件：入口6.9 MPa，538.98 K，速度0.1696 m/s; 出口6.9 MPa，538.98 K。
5. 初值条件：管道内6.9 MPa，538.98 K，管壁538.98 K。



输入卡

```idl
= Bennett 5253

* 问题类型、选项（new、restart类型有瞬态transnt/稳态STDY-ST， strip有格式化输出fmtout/二进制binary）
100 new  transnt


* input / output units,A2.4,国际单位si， 英制单位british
102 si   si

* time step control card; A3.2
* 运行结束时间; 最小时间步长（>1e-6不会生效）; 最大时间步长（申请）; 控制选项（一般3即可，半隐）; 
*小编辑和绘图输出频率（每多少步输出一次）; 大编辑输出频率; 再启动文件（rstplt）输出频率
201   40.0  1.0e-7 0.01 3 1 100 100 

*  minor edits *

* boundary volume CCC=110
1100000  inlet   tmdpvol
*横截面积; 长度; 体积; 水平方位角; 竖直倾斜角; 高度变化; 壁面粗糙度; 水力直径; 控制体控制标记
1100101  0.00012469 0.048 0.0 0.0 90.0  0.048  0.0 0.0 0
*控制体初始条件控制标记; εbt格式; 3表示CCC0201第二三参数为压力和温度。
1100200  3
* 第一参数为搜索变量（如时间）; 二三由上指定为压力和温度
1100201  0.0  6.9e6  538.98

* tdj ccc = 120
1200000  jieguan1  tmdpjun
* 来向部件; 去向部件; 接管面积
1200101  110010000  125000000  0.00012469
* CCC0200卡省略或第一个参数为0则CCC0201的二三参数为液、汽速度，1则为液、汽质量流量。(参数二为Trip卡号)（参数三为？？？）
1200200  1
* 搜索变量; 此二三参数为液、汽质量流量kg/s = 质量流密度 * 流通面积; 界面速度0.0
1200201  0.0  0.1696  0.0  0.0


* pipe CCC=125
1250000  guanzi pipe
* 控制体数量<100
1250001  40
* 控制体面积
1250101  0.00012469  40
* 内部接管面积，201-299
* 控制体长度，控制体编号，301
1250301  0.1385     40
* 体积401
* 方位角501
* 垂直倾角601（90.0向上，0.0水平，-90.0向下），
1250601  90.0      40
*高度变化
1250701  0.1385     40
* 壁面粗糙度、水力直径、编号
1250801  0.0  0      40
* 内部接管顺流能量损失系数; 逆流能量损失系数; 接管编号。
1250901  0.0  0.0  39
* 控制体控制标记 tlpvbfe
1251001  00        40
*接管控制标记
1251101  00000000    39
* 控制体初始条件控制标记 εbt 格式; 参数二到六初始条件数据，此为压力温度; 参数7控制体编号
1251201  3  6.9e6  538.98  0  0  0  40
* 内部接管初始条件控制编号（0则1301为液相/气相/相界面速度0.0/编号，1则前两项为质量流量）
1251300  1
1251301  0.0   0.0   0.0  39
*接管水力直径; 淹没模型（0使用Wallis CCFL，1Kutateladze，0-1则是两者的Bankoff权重因子）;
*CCFL关系式中气体截距为1（默认）; CCFL中斜率为1（默认）; 接管编号。
1251401   0.0126   0.0    1.0     1.0     39


* tdj ccc = 127
1270000  jieguan2  sngljun
1270101  125010000  130000000  0.00012469  0.0  0.0  0  1.0  1.0
1270201  1  0.0  0.0  0.0

* boundary volume CCC=130
1300000  outlet   tmdpvol
1300101  0.00012469 0.048 0.0 0.0 90.0  0.048  0.0 0.0 0
1300200  3
1300201  0.0  6.9e6  538.98

* 常见数据1CCCG000：轴向控制体数量（一般=水力部件控制体数）; 径向网格点数;
*几何类型（1矩形体2圆柱3球形）; 稳态初始化标记（0温度由1CCCG401-499给定，1程序计算）;
*左边界坐标（模拟管则为内径）; 再淹没条件标记（0无，1/2/trip卡号则需填写78字段）
11250000  40  5  2  0  0.0063  0
* 网格位置标记（0则在1CCCG101~201填网格信息，否则同该编号热构件且以下不用填);
*网格格式标记(1/2为卡1CCCG101-G199格式一/二，参数一是0时填写)
11250100  0  1
* 此为格式一：间隔interval数（网格点数-1）; 右坐标（模拟管则为外径）。格式二：网格间隔长度; 间隔编号
11250101  4  0.0083
* 构件成分201：材料编号（绝对值是201MMM00卡中的MMM，此处为20100500）; 间隔编号
11250201  5 4
* 热构件源项分布301：源项分布相对值（网格全1即平均分布）; 网格间隔编号
11250301  1.0  4
* 初始温度; 网格点编号
11250401  538.98  5
* 左、右边界条件501、601：边界相邻水力控制体/通用表编号; 增量（一般10000，对应控制体12501/2/3/.../40）;
*边界条件类型（0对称/绝热，1/1nn对应多种对流边界，nxxx其他类型）;
*表面积标记（决定后面参数5，0左表面积，1矩形体表面积/圆柱体高度/球形份额）;
*表面积（热构件控制体高度）/系数; 热构件编号
11250501  125010000  10000  1  1  0.1385  40
11250601  0  0  0  1  0.1385  40
* 热源类型（0无，1~999查总表，1000~1002中子动力学，10001~14095热源是控制变量，序号减10000);
*内热源乘子(轴向峰值因子可在该处输入，作用于参数一指明的功率);
*左边界慢化剂直接加热乘子（卷2节3.3）; 右边界慢化剂直接加热乘子; 热构件编号
11250701  975  1.0  0.0  0.0  40
* 其他左、右边界条件801、901：热当量直径; 向前加热长度（加热起始到板中心的距离，大于10入口段影响小）;
*回流加热长度（同理出口）; 向前定位架长度; 回流定位架长度; 定位架向前损失系数; 定位架回流损失系数;
*局部沸腾因子（=局部热流密度/沸腾开始后的平均热流密度，若无热源/局部平衡态含汽量<0时填1）;
*热构件编号
11250801  0.0  1.0  1.0  0.0  0.0  0.0  0.0  1.0  40

* 热构件热物性，A12，201MMMNN
* 热构件材料成分类型，201MMM00 卡，内置碳钢C-STEEL，不锈钢S-STEEL，二氧化铀UO2和锆ZR
20100500   c-steel
* 也可以使用用户自定义表/函数TBL/FCTN，此时需要参数二三和201MMM01-49;
*热导率格式标记/gap 摩尔份额标记（1输入表，2函数，3包含气体组分名和摩尔份额的表）;
*体积热容标记（-1表只有热容，温度可以从热导率表对应，1输入包含温度、体积热容的表，2函数）
*20100500  TBL/FCTN  1  1
** AISI 304 e/L 不锈钢：温度（K）; 热导率（W/m•K）
*20100501 0.3 14.88
*20100502 343. 14.88
*20100503 373. 15.05
** AISI 304 e/L 不锈钢：温度（K）; 体积热容 (J/m^3/k)
*20100551 0.3    8.51e6
*20100552 343.   8.51e6
*20100553 373.   8.53e6
*20100500  TBL/FCTN  1  1      *stainless steel
*20100501  311.15   14.88   366.15   15.58   477.15   16.96


* 表类型（POWER功率随时间变化，HTRNRATE热流密度-t，HTC-T传热系数-t，HTC-TEMP传热系数温度-t，
*TEMP温度-t，reac-T 反应性随着时间/控制系统变量，NORMAREA 阀门归一化面积随阀杆位置）
20297500  power
* 时间; 功率（W，英制单位则为MW）（注意！！！功率是给热构件的这个控制体的，=功率密度*pi*内径*控制体高度）
20297501  0.0  4.973e3
20297502  5000.0   4.973e3

*以下仅供‘敏感性分析的修改版程序’使用
151  1.0
152  1.0
153  1.0
154  1.0
155  1.0
156  1.0
157  1.0
158  1.0
159  1.0
160  1.0
161  1.0
162  1.0
163  1.0
164  1.0
165  1.0
166  1.0
167  1.0
168  1.0
169  1.0
170  1.0
171  1.0
172  1.0
173  1.0
174  1.0
175  1.0
176  1.0
177  1.0
178  1.0
179  1.0
180  1.0
181  1.0
182  1.0
183  1.0
184  1.0
185  1.0
186  1.0
187  1.0
188  1.0
189  1.0
190  1.0
191  1.0

.End of input.
```



# rstplt&outdta变量

参见input manual A4小编辑部分


| 变量缩写                    | 解释.............................  | 备注                                                         |
| --------------------------- | ---------------------------------- | :----------------------------------------------------------- |
| acqtank                     | 安注箱内向气相传热总能             | Total energy transport to the gas by heat and mass transfer in the accumulator (W, Btu/s). |
| acrhon                      | 安注箱不可凝气体密度               | Accumulator noncondensable density (kg/m^3 , lb/ft^3 ).      |
| acttank                     | 安注箱平均壁温                     | Mean accumulator tank wall metal temperature (K, ℉).         |
| acvdm                       | 安注箱* 气相体积                   | Gas volume in the accumulator tank, standpipe, and surge line (m^3 , ft^3 ). |
| acvliq                      | 安注箱* 液相体积                   | Liquid volume in the accumulator tank, standipipe, and surge line (m^3, ft^3) |
| boron                       | 硼浓度                             | Spatial boron density, ρb (kg/m^3, lb/ft^3).                 |
| count                       | 时间步计数                         | Current attempted advancement count number.                  |
| cntrlvar                    | 控制变量                           |                                                              |
| cputime                     | CPU 时间                           | Current CPU time for this problem (s).                       |
| dt                          | 当前时间步长                       | Current time step (s).                                       |
| dtcrnt                      | 当前库朗特数时间步长               | Current Courant time step (s).                               |
| emass                       | 质量误差                           | Estimate of mass error in all the systems (kg, lb).          |
|                             |                                    |                                                              |
| fij                         | 相间摩擦的系数                     | (N-s2/m5)                                                    |
| floreg<br />flow regi       | 流型1                              | Flow regime number; the parameter is the volume number. A chart showing the meaning of each number is shown in Volume II, page 14<br /><br />High mixing bubbly,  CTB,  1<br/>High mixing bubbly/mist transition CTT 2<br/>High mixing mist CTM 3<br/>**Bubbly BBY 4<br/>Slug SLG 5<br/>Annular mist ANM 6<br/>Mist pre-CHF MPR 7<br/>**Inverted annular IAN 8<br/>Inverted slug ISL 9<br/>**Mist MST 10<br/>**Mist post-CHF MPO 11<br/>Horizontal stratified HST 12<br/>Vertical stratified VST 13<br/>Level tracking LEV 14<br/>Jet junction JET 15<br/><br />ECC mixer wavy MWY 16<br/>ECC mixer wavy/annular mist MWA 17<br/>ECC mixer annular mist MAM 18<br/>ECC mixer mist MMS 19<br/>ECC mixer wavy/slug transition MWS 20<br/>ECC mixer wavy-plug-slug transition MWP 21<br/>ECC mixer plug MPL 22<br/>ECC mixer plug-slug transition MPS 23<br/>ECC mixer slug MSL 24<br/>ECC mixer plug-bubbly transition MPB 25<br/>ECC mixer bubbly MBB 26<br /> |
| fjunft                      | 顺流不可逆的总形状损失系数         | Total form loss coefficient for irreversible losses, forward |
| fjunrt                      | 逆流不可逆的总形状损失系数         | Total form loss coefficient for irreversible losses, reverse |
| formfj                      | 液相形状损失系数                   |                                                              |
| formgj                      | 气相形状损失系数                   |                                                              |
| fwalfj                      | 液相壁面摩擦的系数                 |                                                              |
| fwalgj                      | 汽相壁面摩擦的系数                 |                                                              |
| Gamma.boil                  | 蒸发总速率                         |                                                              |
| Hif.liq.int                 | 衡量相间传热的系数，液相           | (Watts/m3-K)                                                 |
| Hig.vap.int                 | 衡量相间传热的系数，汽相           | (Watts/m3-K)                                                 |
| htchf                       | 临界热流密度                       | Critical (maximum) heat flux (W/m^2, Btu/s•ft^2).            |
| htchfr                      | 临界热流密度比率                   | Critical heat flux ratio (ratio of HTCHF to HTRNR).          |
| hthtc                       | 传热系数                           | Heat transfer coefficient (W/m^2•K, Btu/s-ft^2•℉).           |
| htrnr                       | 热流密度（热通量)                  | Heat flux (W/m^2, Btu/s•ft^2).                               |
| httemp                      | 网格点温度                         | Mesh point temperature (K, ℉).                               |
| htvat                       | 热构件体积平均温度                 | Volume averaged temperature in the heat structure (K, ℉).    |
| jun.area                    | 接管面积                           | junction area                                                |
| jun-flag                    | 接管控制标记                       | junction control flag (jefvcahs) <br /> j0不是jet喷放接管，<br />e0不用能量方程PV修正项，<br />f0不用CCFL模型，<br />v0不用水平分层夹带模型，<br />c0开启壅塞choking模型（临界流？无1卡/1卡无50选项则为Henry-Fauske模型，否则是relap5模型），<br />a0光滑变截面，<br />h0非均相流，<br />s0同时使用来/去向部件的动量通量 跟随3个临界流参数 临界流常数1（取决于参数6：Henry-Fausk耗散常数discharge默认1.0，relap5模型过冷耗散常数默认1.0）;  临界流常数2（Henry-Fausk未使用此值0.0，Relap5过热耗散常数默认1.0）;  临界流常数3（Henry-Fausk热力非平衡常数默认0.14，Relap5模型默认1.0）; |
| Mass.flux                   | 质量流密度                         | (kg/sec-m2)                                                  |
| mflowj<br />mass flow       | 总质量流率                         | Combined liquid and vapor flow rate (kg/s, lb/s).            |
| p                           | 压力                               | Volume pressure (Pa, lbf/in2).                               |
| pmphead                     | 泵扬程（水头）                     | Pump head in the pump component (Pa, lbf/ in2).              |
| pmptrq                      | 泵扭矩                             | Pump torque in the pump component (N•m, lbf•ft).             |
| pmpvel                      | 泵转速                             | Pump rotational velocity in the pump component (rad/s, rev/min) |
| q                           | 气液壁面总热源                     | Total volume heat source from the wall and direct moderator ( 慢化剂 )  heating to liquid and vapor (W, Btu/s). This variable request is the same as “Q.wall.tot.” in the major edits. |
| quala<br />quality non-cond | 不可凝气体质量份额                 | Volume noncondensable mass fraction. The ratio of the mass of the noncondensable gas to the total mass of the vapor phase. |
| qualaj                      | 接管不可凝气体质量份额             | Junction noncondensable mass fraction.                       |
| quale<br />quality mix-cup  | 热平衡含气率                       | Volume equilibrium quality. This quality uses phasic enthalpies and mixture quality, with the mixture enthalpy calculated using the flow quality. |
|                             |                                    |                                                              |
|                             |                                    |                                                              |
| quals<br />quality static   | 静态含气率                         | Volume static quality.                                       |
| qwg<br />Qwg.wall.gas       | 气相壁面总热源                     | Volume heat source from the wall and direct moderator heating to vapor (W, Btu/s). This variable request is the same as “Qwg.wall.gas.” in the major edits. |
| Qwg.wall.total              | 壁面总热源                         |                                                              |
| reac                        | 总反应性反馈                       | reactivity feedback total (dollars). This is the sum of reacm, reacrb, reacs, and reactf. |
| reacm                       | 慢化剂密度总* 反应性反馈=密度+温度 | reactivity feedback total from moderator density changes (dollars). This is the sum of reacrm and reactm. |
| reacrb                      | 硼浓度反应性反馈                   | reactivity feedback from boron density changes (dollars).    |
| reacrm                      | 慢化剂密度反应性反馈               | reactivity feedback from moderator density changes (dollars). |
| reacs                       | 紧急停堆曲线反应性反馈             | reactivity feedback from scram curve (dollars).              |
| reactf                      | 燃料温度反应性反馈                 | reactivity feedback from fuel temperature changes (dollars). |
| reactm                      | 慢化剂温度反应性反馈               | reactivity feedback from moderator temperature (spectral) changes (dollars) |
| Reynolds  liquid / vapor    | 液/汽雷诺数                        |                                                              |
| rho                         | 混合密度                           | Total density (kg/m^3, lb/ft^3).                             |
| rho-boron                   | 硼浓度                             |                                                              |
| rhof                        | 液相密度                           | Liquid density ρf (kg/m^3, lb/ft^3).                         |
| rhofj                       | 接管液相密度                       | Junction liquid density (kg/m^3, lb/ft^3).                   |
| rhog                        | 气相密度                           | Vapor density ρg (kg/m^3, lb/ft^3).                          |
| rhogj                       | 接管气相密度                       | Junction vapor density (kg/m^3, lb/ft^3).                    |
| rho-mix                     | 混合平均密度                       |                                                              |
| rkfipow                     | 裂变功率                           | Reactor power from fission (W).                              |
| rkgapow                     | 总衰变功率=锕系+裂变产物           | Reactor power from decay of fission products and actinides (W). |
| rkpowa                      | 锕系元素衰变功率                   | Reactor power from decay of actinides (W).                   |
| rkpowk                      | 裂变产物衰变功率                   | Reactor power from decay of fission products (W).            |
| rkreac                      | 反应性                             | Reactivity (dollars).                                        |
| rkrecper                    | 反应堆周期的倒数                   | Reciprocal period (s-1).<br />period：反应堆周期，即堆内中子密度变化e倍所需时间。 |
| rktpow                      | 反应堆总(热)功率=裂变+衰变         | Total reactor power, i.e., sum of fission and decay powers (W). |
| sattemp<br />satt-part      | 饱和温度（基于蒸汽分压）           | Volume saturation temperature based on the partial pressure of steam (K, ℉). |
| sounde                      | 声速                               | Volume sonic velocity (m/s, ft/s).                           |
| tempf                       | 液相温度                           | Volume liquid temperature Tf (K, ℉).                         |
| tempg                       | 气相温度                           | Volume vapor temperature Tg (K, ℉).                          |
| throat ratio                | 喉部接管面积比                     | 喉部面积/接管面积（v2p25, 26）                               |
| time                        | 模拟物理时间                       | Time (s).                                                    |
| tmass                       | 系统总质量                         | Total mass of water, steam, and noncondensables in all the systems (kg, lb). |
| uf                          | 液相比内能                         | Liquid specific internal energy (J/kg, Btu/lb).              |
| ufj                         | 接管液相比内能                     | Junction liquid specific internal energy (J/kg, Btu/lb).     |
| ug                          | 气相比内能                         | Vapor specific internal energy (J/kg, Btu/lb).               |
| ugj                         | 接管气相比内能                     | Junction vapor specific internal energy (J/kg, Btu/lb).      |
| vapgen<br />Vapor.gen       | 蒸发净总速率                       | Total mass transfer rate per unit volume at the vapor/liquid interface in the bulk fluid for  vapor generation/condensation and in the boundary layer near the wall for vapor  generation/condensation (kg/m^3  •s, lb/ft^3  •s). |
| velf<br />vel-liquid        | 液相速度                           | Volume oriented liquid velocity (m/s, ft/s); the parameter is the volume number plus F. |
| velfj<br />liq.j.vel        | 接管液相速度                       | Junction liquid velocity (m/s, ft/s). This velocity is based on the junction area Aj, which is  discussed in Volume II, Section 2.4. |
| velg<br />vel-vapor         | 气相速度                           | Volume oriented vapor velocity (m/s, ft/s); the parameter is the volume number plus F. |
| velgj<br />vap.j.vel        | 接管气相速度                       | Junction vapor velocity (m/s, ft/s). This velocity is based on the junction area Aj, which is  discussed in Volume II, Section 2.4. |
| vlvarea                     | 阀门与接管面积比                   | Ratio of the current valve physical area to the junction area. |
| vlvstem                     | 阀门面积开启比例<br />阀杆位置比   | 对于开启了归一化阀杆位置选项的电动阀和伺服阀，是当前阀杆位置与全开时的比值; 否则是面积比 |
| voidf                       | 液相截面份额                       | Volume liquid fraction.                                      |
| voidfj                      | 接管液相截面份额                   | Junction liquid fraction.                                    |
| voidg                       | 空泡份额                           | Volume vapor fraction (void fraction).                       |
| voidgj                      | 接管空泡份额                       | Junction vapor fraction (void fraction).                     |
| voidg                       | 空泡份额(前一时间步)               | Vapor void fraction previous time step (n)                   |
| vol-flag                    | 控制体控制标记                     | volume control flag (tlpvbfe)<br />t0不用热前沿追踪跟踪模型，<br />l0不用mixture level混合物水平面追踪模型，<br />p0使用water packing scheme水填充方案，<br />v0不使用垂直分层模型，<br />b0使用管道相间摩擦模型，<br />f0沿体积的x坐标计算壁摩擦效应，<br />e0使用非平衡(不相等温度)计算 |
|                             |                                    |                                                              |



# 其他



## 增加终端默认宽度

问题：relap5计算时终端输出总是由于宽度不够而自动换行。

解决：右键终端上方框体，默认值，布局，窗口大小，设置为140，确定。重新运行。

## 为无后缀文件设置默认打开程序

- 管理员运行cmd，`assoc  .`，查看无拓展的关联类型，应该显示没有找到文件关联.。
- `assoc .=No Extension`，自定义设置无后缀文件为`No Extension`类型
- `ftype "No Extension"="F:\pro\UltraEdit\uedit64.exe""%1"`，设置默认打开程序为ultraedit。
- `ftype`，查看所有后缀格式的默认打开程序，在no extention一行确认。

## 一键删除并运行.bat

新建文本文件，重命名为`0一键删除outdta和rstplt，运行relap5.bat`，编辑以下内容，双击即可运行。

其中/p提示确认，/f强制删除只读，/S删除所有子目录中的指定的文件，  /Q 安静模式，删除全局通配符时，不要求确认。  /A 根据属性选择要删除的文件。（属性: R  只读文件, S  系统文件, H  隐藏文件, A  存档文件, I  无内容索引文件,   L  重分析点, `-`表示“否”的前缀）


```bat
del /f /q outdta
del /f /q rstplt
.\Relap.exe

```



## AptPlot快速绘图rstplt

1. 安装jdk运行环境：双击 jdk-7u51-windows-x64.exe
2. 安装aptplot：右键`AptPlotInstaller.jar`——打开方式选择java (TM) Plateform SE binary——install or update from a local folder——选择当前文件夹提取安装包（含有文件AptPlot_6.5.2）——选择安装目录——按步骤操作完成安装
3. 处理relap再启动文件rstplot：打开aptplot的file目录——read——relap5 data——选择rstplt再启动文件——在弹出的select relap channels窗口选中关心的数据channel（单击/ +ctrl/ +shift）——plot绘制——可以在data sets中查看已选数据集，继续选中channel增加数据，或者在data sets中右键显示show/隐藏hide/删除kill数据，或者clear sets清空数据集。
4. 为了简化操作，各种绘图框的设置可以通过保存apf文件的形式保留下来。先read一个不需要的rstplt文件，然后read需要的。这样在需要重新运行时，只需要点击`select RELAP channels`窗口的删除按钮就能解除文件占用，然后去文件夹运行一键删除和运行脚本并读入rstplt文件就能快速查看第二次。



其他注意事项/使用技巧

1. 不懂就去查文档：Help目录——help contents
2. 查看、编辑数据表：双击data sets——双击单元格修改
3. 解决Data Sets不显示变量名：右键Data Sets框体——selector operations——勾选view sets comments
4. 导出到Origin：select relap channels窗口——data sets框——选择数据右键Edits——复制数据（如何大批量复制：选中第一行第二列单元格，拖动滚动条到最后，按住Shift时单击最后一行最后一列单元格，ctrl +c）
5. 调整坐标轴范围：Plot菜单进入Axes设置（或双击四周框线弹出）——general的Edit设置编辑对象——start、stop设置范围
6. 调整绘图框大小位置：双击四角的黑点不松手，拖动即可调整。也可以去Graph Appearance——Main——viewport调整XY范围。
7. 调整图例框位置Legends Box：双击图例框或点击Plot菜单弹出Graph Appearance——编辑Leg. Box Location中的X Y（分别是左、上边的位置）
8. 选择数据操作麻烦，如果关闭select relap channels窗口后还想继续选择数据，只能重新去file目录——read——relap5 data。

## UltraEdit / Notepad++ 列编辑outdta

Notepad++ 列编辑时只需按住`alt`键选择。



1. 双击 ue_chinese_64.exe 64位版官方安装程序。
2. 安装完成后关闭程序，复制 IDM_Universal_Patch_v6.0.exe 到安装目录，以管理员身份运行。
3. 选择 ultra edit v27.x -(x64) , 点击patch。
4. 若提示未找到安装路径，点击是，手动选择安装目录下的 ProtectionPlusDLL.dll 打开，再次patch。

使用：

1. 列编辑：主菜单——编辑菜单——列模式，打开即可对某一列数据进行操作，对outdta很实用。
2. 空格处理：格式菜单，制表符/空格，转换制表符为空格（全部）。删除行首空格。删除行尾空格。
3. 

## Origin严肃画图outdta

安装：

- 先修改系统时间为20210701（设置，时间与语言，关闭自动设置时间， 手动设置日期，否则提示试用期已过无法安装）。
- 双击setup.exe安装，安装origin pro试用，用户和公司名`yuzhuyi`，下一步直到安装完毕，稍后重启。
- 复制crack文件夹所有文件至安装目录下，替换同名文件。
- 重新设置系统时间。

AptPlot功能少、操作繁琐，只能绘制某个控制体的某个变量随时间变化的曲线。如果想绘制该变量随所有控制体的变化曲线，则需要使用Origin和Ultraedit来处理outdta。此外，Origin的图也被视为学术规范。




