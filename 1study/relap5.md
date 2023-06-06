
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
   - 大编辑，包括水力控制体、水力接管、热构件、再启动编号、再淹没等
     - 占据outdata文件内容的大部分，基本囊括了所有需要用到的信息和变量数据。`0MAJOR EDIT !!!time=    0.00000     sec`
     - 水力部件，参数从左到右，构件编号从小上到大。`0   Vol.no.  pressure`，`0   Jun.no.   from vol.`。
     - 热构件，`0 HEAT STRUCTURE OUTPUT +++time=     0.00000     sec`
     - 再启动编号和再启动总结，`0---Restart no.       0 written, block no.     0, at time=     0.00000    ---`，`0---Restart Summary:       41 blocks written in restart file  ---`
     -
   - 报错诊断编辑
4. rstplt， 再启动（和绘图）文件，restart and plot，包含所有指定时刻运行的原始非文本数据，作为再启动工况或者绘图时的输入
5. tpfh2onew， 水物性文件，thermodynamic property file of new light water，不带new为旧版轻水物性。



#  编写输入卡indta

## 基本框架&小贴士

```idl
= 问题标题， title

控制选项， Control options (cards 100-199)
时间步选项， Time step options (cards 200-299)

* 小编辑， Minor edits (301-399)
* 绘图请求， Plot requests (20300XXY)

动作信号，Trips (400-799 or 20600000-20620000)

水力部件， Hydrodynamic components (CCCXXXX)
热构件， Heat structures (1CCCGXNN)

热构件物性，Heat structure thermal property data（201MMMNN）
通用表， General tables（202TTTNN）

* 反应堆中子动力学， Reactor kinetics (30000000-30399999)
* 控制系统， Control system (205CCCNN or 205CCCCN)，可以算术计算、泵控制、蒸汽控制 、给水控制
* 控制绘图变量， Expanded plot variables (2080XXXX)

.End of input.
```



1. 类控制体部件输入顺序：

   - 面积；长度；体积；方位角；垂直倾斜角；高度变化；壁面粗糙度；水力直径；控制体控制标记 tlpvbfe；控制体初始条件控制标记 εbt。
   - 此处指 snglvol, tmdpvol, pipe, annulus

2. 四种常见控制标记

   - 控制体控制标记 tlpvbfe
       - t0不用热前沿追踪跟踪模型，l0不用mixture level混合物水平面追踪模型，p0使用water packing scheme水填充方案，v0不使用垂直分层模型，b0使用管道相间摩擦模型，f0沿体积的x坐标计算壁摩擦效应，e0使用非平衡(不相等温度)计算


   - 接管控制标记 jefvcahs

       - j0不是jet喷放接管，e0不用能量方程PV修正项，f0不用CCFL模型，v0不用水平分层夹带模型，c0开启壅塞choking模型（临界流？无1卡/1卡无50选项则为Henry-Fauske模型，否则是relap5模型），a0光滑变截面，h0非均相流，s0同时使用来/去向部件的动量通量

       - 跟随3个临界流参数

       - 临界流常数1（取决于参数6：Henry-Fausk耗散常数discharge默认1.0，relap5模型过冷耗散常数默认1.0）；

       - 临界流常数2（Henry-Fausk未使用此值0.0，Relap5过热耗散常数默认1.0）；

       - 临界流常数3（Henry-Fausk热力非平衡常数默认0.14，Relap5模型默认1.0）；

   - 控制体初始条件控制标记 εbt

       - （3表示单组分水/蒸汽，用后面两参数压力、温度。其中t=0-3单组分，4-6双组分有不凝性气体。）

   - 接管初始条件控制标记 0/1
       - （1表示后面某两个参数是水、汽质量流量kg/s，0则为速度）




## 标题-控制-时间

写在输入卡前面的杂项

```idl
* title card 题目卡，A1.3
= Bennett 5253

* problem type and option,A2.2，一般的new、restart等类型有瞬态transnt/稳态STDY-ST选项，strip后处理类型有格式化输出fmtout/二进制binary选项。
100 new  transnt

* input / output units,A2.4,英制单位BRITISH
102 si   si

*不凝性气体种类：氩气ARGON, 氦气HELIUM, 氢气HYDROGEN, 氮气NITROGEN, 氙气XENON, 氪气KRYPTON, 空气AIR，六氟化硫SF6.
110    nitrogen

* time step control card，A3.2
* 运行结束时间，最小时间步长（>1e-6不会生效），最大时间步长（申请），控制选项（一般3即可，半隐），小编辑和绘图输出频率（每多少步输出一次），大编辑输出频率，再启动文件（rstplt）输出频率
201   40.0  1.0e-7 0.01 3 1 100 100 *时间控制!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

*卡1，控制选项A2.1：50在所有启用了choking模型的接管上激活原始的RELAP5临界流模型，而不是默认的Henry-Fauske临界流critical flow模型。
1    50

```



## Trips动作信号卡

```idl
*变量Trips卡A5.3: 401-599 or 20600010-20610000

*变量variable；参数parameter；关系运算符（lt,le,eq,ne,gt,ge，其中有less,than,equal,not,greater）；变量；参数；额外内容；开关指示（L一开常开，N始终判断（每个时间步））
*此处定义布尔值401：时间大于等于1000s后永远为真1。
401 time     0         ge  null       0       1000.0      l
*此处定义布尔值450、451：每当控制体变量265（被定义为稳压器水位百分比）小于等于14.0时450为真1，大于等于67.7时451为真1
450 cntrlvar      265 le null       0       14.0      n   * 低水位low pzr-l heaters off
451 cntrlvar      265 ge null       0       67.7      n   * hi pzr-l all heaters on

*逻辑Trips卡A5.4: 601-799 or 20610010-20620000
*变量Trip卡号；逻辑运算符（and和，or或，xor异或）；变量Trip卡号；开关指示（L一开常开，N始终判断（每个时间步））
*此处定义布尔值689，实则=非401（例如，此处用于trip阀门开关信号，小于1000s阀门打开）
689 -401     or       -401    n
*此处非450是>14%,非451是<67.7%，两者和运算结果为布尔值650，即14%<稳压器水位百分比<67.7%时为真1。
650   -450   and    -451   n    *14%<pzr-l<67.7%
```



## 水力部件

水力部件卡号形如CCCXXXX，CCC即部件编号。

### snglvol单控制体

“snglvol”单控制体，最简单的流体流通通道，在不需要分支部件和管道（可以看做单控制体和单接管的组合）部件时使用。

```idl
1450000   upplnm2      snglvol
*流通面积；长度；体积；方位角；倾斜角
1450101   7.2707       1.08208       0.0         0.0     90.0
*高度变化；壁面粗糙度；水力直径；控制体控制标记 tlpvbfe
1450102   1.08208      6.3e-6        0.4265      00
*控制体初始条件标记（3单组分水/蒸汽，后面是压力和温度）
1450200   3            1.55790e7     600.466
```

### sngljun单接管

“sngljun”单接管，用来接通两个部件。

```idl
1270000  jieguan2  sngljun
*来向部件，去向部件，面积，正向流动能量损失系数，逆流能量损失系数，接管控制标记 jefvcahs，789字段取决于6字段c标记（临界流模型是用的Henry-Fauske模型√还是relap5模型×，此处应该是discharge coefficient, thermal nonequilibrium constant, 不使用9字段）
1270101  125010000  130000000  0.00012469  0.0  0.0  0  1.0  1.0
*接管初始条件标记（0则23变量是水、汽速度，1为质量流量），水质量流量，汽质量流量，界面速度
1270201  1  0.0  0.0  0.0
```

### tmdpvol时间控制体
“tmdpvol”时间相关控制体，定义边界条件，温度、压力等参数，与snglvol的不同在于其压力温度可随时间等参数变化

```idl
1100000  inlet   tmdpvol
*横截面积，长度，体积，水平方位角，竖直倾斜角，高度变化，壁面粗糙度，水力直径，控制体控制标记
1100101  0.00012469 0.048 0.0 0.0 90.0  0.048  0.0 0.0 0
*控制体初始条件控制标记，εbt格式（工质类型水/120-9卡指定，无有硼，如何处理CCC0201-0299中的数据），省略前0。其中t=0-3单组分，4-6双组分有不凝性气体。
* 3表示CCC0201第二三参数为压力和温度。
1100200  3
* 第一参数为搜索变量（如时间），二三由上指定为压力和温度
1100201  0.0  6.9e6  538.98
```
### tmdpjun时间接管
“tmdpjun”时间相关接管，定义流量边界条件，与sngljun的不同在于其流量可随时间等参数变化

```idl
1200000  jieguan1  tmdpjun
* 来向部件，去向部件，接管面积
1200101  110010000  125000000  0.00012469
* CCC0200卡省略则二三参数为液、汽速度。第一个参数为1，则0201等的二三参数为液、汽质量流量。
1200200  1
* 此为质量流量kg/s = 质量流密度 * 流通面积
1200201  0.0  0.1696  0.0  0.0
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
* 内部接管顺流能量损失系数；逆流能量损失系数；接管编号。
1250901  0.0  0.0  39
* 控制体控制标记 tlpvbfe
1251001  00        40
*接管控制标记
1251101  00000000    39
* 控制体初始条件控制标记 εbt 格式；参数二到六初始条件数据，此为压力温度；参数7控制体编号
1251201  3  6.9e6  538.98  0  0  0  40
* 内部接管初始条件控制编号（0则1301为液相/气相/相界面速度（未实现，0）/编号，1则1301为液相/气相质量流量/相界面速度（0）/编号）
1251300  1
1251301  0.0   0.0   0.0  39
*接管水力直径，淹没模型选择（0使用Wallis CCFL模型，1Kutateladze，0-1则是两者的Bankoff权重因子），CCFL关系式中使用的气体截距为1（默认）， CCFL中使用的斜率为1（默认）；接管编号。
1251401   0.0126   0.0    1.0     1.0     39
```

### mtpljun多接管

“mtpljun”多接管，连接多个控制体，A9.18。包含的单接管编号为CCCJJNN00

```idl
1010000   dn1-1-2     mtpljun
*接管数量；初始条件控制字1（表示CCC1NNM中为水、汽速度，NN组号，M卡号，此处1011011）
1010001   2           1
*CCC0NNM:来向控制体；去向控制体；面积；顺流能量损失系数；逆流能量损失系数；接管控制标记 jefvcahs，多接管jv=0
1010011   100010006   102010005   0.4893   5.0   5.0   01003
1010021   100020006   102020005   0.33     5.0   5.0   01003
*接上面CCC0NNM参数7-13，其中临界流常数789取决于上面控制字6
*临界流常数1；常数2；常数3；来向控制体编号增量（难以理解一般不用：用于批量定义多个接管，加上参数1得到连接新控制体编号的接管，继承参数3-9）；去向控制体编号增量；未实现0；接管编号。A9.18.2，page 213。
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
*控制体面积，编号
1000101   0.64436    2
*接管面积，编号
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
*接管控制标记 jefvcahs，环jv=0
1001101   00         1
*控制体初始条件控制标记 εbt（3单组分水/蒸汽，只用后面两参数），压力，温度，（几个参数未使用填充0.0），控制体编号
1001201   3    1.58162e7    565.971    0.0    0.0    0.0    1
1001202   3    1.58086e7    565.968    0.0    0.0    0.0    2
*接管初始条件控制标记 0/1（1表示301卡是水、汽质量流量kg/s，0则为速度）
1001300   1
*接管初始条件：水质量流量；汽质量流量；相间速度未实现0；接管编号
1001301   120.34     0.0     0.0    1
*接管水力直径，淹没模型（0使用Wallis CCFL模型，1Kutateladze，0-1则是两者的Bankoff权重因子），CCFL关系式中使用的气体截距为1（默认）， CCFL中使用的斜率为1（默认）；接管编号。
1001401   0.3304     0.0     1.0    1.0    1
```

### branch分支

“branch”分支，最多可以有10个接管，相当于带有多个接管sngljun的单一控制体snglvol。

```idl
1060000   clgin-1     branch
*接管数4；接管初始条件控制标记（决定CCCN201：0速度，1质量流量）
1060001   4           1
*面积；长度；体积；方位角；垂直倾斜角；高度变化；壁面粗糙度；水力直径；控制体控制标记
1060101   0.0         1.0439       0.8202      0.0    -90.0
1060102   -1.0439     6.3e-6       0.4008      00
*工质初始条件εbt=003单组分蒸汽/水（后面两个参数是PT）；压力；温度。
1060200   3           1.57999e7    565.965
*入口面；出口面；分支面积；顺流能量损失系数(AF)；逆流能量损失系数(AR)；接管控制标记 jefvcahs分支j=0
1061101   250020002   106010001    0.3832     0.28    0.28   0000
1062101   106010001   100010001    0.0        0.0     0.0    0000
1063101   106010002   112010001    0.0        0.0     0.0    0000
1064101   106010006   108010005    0.5065     5.0     5.0    0000
*水力直径；淹没flood关系式（默认0使用Wallis CCFL模型，1则Kutateladze模型，0-1两者插值）；气体截距c = 1；斜率m = 1。
1061110   0.6985     0.0    1.0    1.0
1062110   0.4522     0.0    1.0    1.0
1063110   0.4902     0.0    1.0    1.0
1064110   0.0        0.0    1.0    1.0
*接管初始条件：液、气相质量流量（取决于CCC0001）；界面速度。
1061201   4813.0     0.0    0.0
1062201   120.40     0.0    0.0
1063201   4692.6     0.0    0.0
1064201   0.0        0.0    0.0
```

### separator汽水分离器

“separatr”汽水分离器，特殊分支，模拟蒸汽发生器里的汽水分离器

```idl
5250000   separat1   separatr
*接管数=3；接管初始条件控制标记（决定CCCN201：0速度，1质量流量。其中N即汽水分离器的3个接管：1出口蒸汽2出口水3入口）
5250001   3          0
*面积；长度；体积；方位角；垂直倾斜角；高度变化；壁面粗糙度；水力直径；控制体控制标记
5250101   3.2973     2.3302     0.0      0.0     90.0     2.3302
5250102   6.3e-6     0.532      0000
*控制体初始条件εbt=000单组分蒸汽/水（后面两个参数是P,Uf,Ug,αg）；压力；液相内能；气相内能；空泡份额。
5250200   0    6.727480e6    1.24432e+06  2.58406e+06    0.75684

*出口蒸汽接管CCC1
*入口面；出口面；分支面积（0取相邻控制体的最小面积）；顺流能量损失系数(AF)；逆流能量损失系数(AR)；接管控制标记 jefvcahs 汽水分离器jefv=0；空泡份额限制（蒸汽接管不写默认0.5，水接管不写默认0.15，入口接管无此参数。入口(?)大于此值则出口为纯蒸汽、纯水）
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
*阀门来向组件；去向组件；面积；前向能量损失系数A_F；逆流能量损失系数A_R；接管控制标记 jefvcahs 阀门j=0
1360101  260010001  212000000  1.60055e-3   0.0  0.0  0100
*接管初始条件控制标记（此处1，此后两参数为流量，0则为速度）；初始液相流量；初始气相流量；界面速度（未实现输入0.）
1360201  1  0.00  0.00  0.
*阀门类型（CHKVLV 止回阀Check；TRPVLV 跳闸阀Trip；INRVLV 惯性旋启式止回阀Inertial swing check；MTRVLV 电动阀Motor；SRVVLV 伺服阀Servo；RLFVLV安全阀Relief）
1360300  trpvlv
*Trip阀的开关信号卡编号（在Trips卡中定义为布尔值，例如小于1000s阀门打开）
1360301  689

*其他类型：电动阀，如稳压器安全阀
2770300    mtrvlv
*开启的Trip信号卡编号；关闭的Trip卡编号；阀门开（关）变换速率（s^-1。具体而言：此处无参数5，指归一化面积变化率，有参数5则指阀杆位置；若输入参数6指开启时归一化面积变化率）；初始位置（无5指面积，有5指阀杆位置）；（参数5是阀门表编号，参数6阀门关闭速率）
2770301    683         685         0.6667     0.0
```

### pump泵

“pump”泵，模拟泵及其动作

```idl
2350000   rcppump2     pump
*面积；长度；体积；水平方位角；竖直倾斜角；高度变化；控制体控制标记 tlpvbfe
2350101   0.0          1.685         3.039    0.0    0.0
2350102   0.0          00000
*入口、出口控制体面；面积；正向能量损失系数；逆流能量损失系数；接管控制标记 jefvcahs
2350108   230060002    0.486945     0.0       0.0    0000
2350109   240010001    0.3832       0.146000       0.118000    0000
*控制体初始条件控制标记 εbt（3表示单组分水/蒸汽，用后面两参数压力、温度；压力；温度
2350200   3            1.55025e7    565.852
*入口、出口接管初始条件控制标记（0速度1流量）；水流量；汽流量；相间速度未实现0
2350201   1            4813.00      0.0       0.0
2350202   1            4813.00      0.0       0.0
*泵单相数据表标记（0本组件给出单相表，正整数使用该编号代表的泵数据，-1Bingham泵，-2西屋泵）；两相系数表索引（-1无两相，>=0同上）；两相扬程差difference表索引（-3无，>=0同上）；电机扭矩表索引（-1无，>=0同上）；时间相关速度索引（-1无，>=0同上）；泵Trip卡编号（trip为真时断电！）；逆流标记（0不允许，1允许）
2350301   0     0    0     -1   -1    411    0
*1额定转速rated pump velocity；2初始转速因子（初值除以额定）；3额定流量；4额定扬程head；5额定扭矩torque；
2350302   157.0795    1.0        6.344444     96.228         37300.0
*6转动惯量moment of inertia；7额定密度；8额定电机扭矩（0由初始转速和泵扭矩计算，其中泵扭矩由初始转速、初始控制体条件和对应曲线计算。非零输入了泵电机扭矩表）；9TF2摩擦扭矩系数（将真实转速与额定的比值乘以second power）；10TF0摩擦扭矩系数（定值）；
2350303   3800.0     742.0      0.0         120.0        0.0
*11TF1摩擦扭矩系数；*12TF3摩擦扭矩系数；（将真实转速与额定的比值乘以first、third power）
2350304   2500.0     0.0

*单相扬程曲线：曲线类型（1扬程2扭矩）；曲线区域；自变量（-1~0或0~1）；因变量
2351100   1   1   0.0   1.84   0.1   1.74    0.2   1.64   0.3   1.55
2351101           0.4   1.46   0.5   1.38    0.6   1.32   0.7   1.26
2351102           0.8   1.19   0.9   1.1     0.95  1.05   0.98  1.02
2351103           1.0   1.0
*内容太多写不下曲线区域2~6。

*更多曲线与之类似，参考输入手册A9.17.16, page 208：单相扭矩曲线、两相扬程因子曲线、两相扭矩因子曲线、两相水头差（head扬程,difference）曲线、两相扭矩差曲线。

```



## 热构件

热构件（A10，1CCCGXNN）：CCC是热构件编号，一般与水力部件相同，G几何编号用来区分接触同一水力控制体的不同热构件，X卡类型，NN该类型的编号

```idl
* 常见数据1CCCG000：轴向热构件数量（1~99,一般=水力部件控制体数），周向网格点数（1~99），几何类型（1矩形体2圆柱3球形），稳态初始化标记（0温度由1CCCG401-499给定，1程序计算），左边界坐标（模拟管则为内径），再淹没条件标记（0无，1/2/trip卡号则需填写78字段）
11250000  40  5  2  0  0.0063  0
* 网格位置标记（0在则在1CCCG101~201填网格信息，否则同该编号热构件且以下不用填)，网格格式标记(1/2为卡1CCCG101-G199格式一/二，参数一是0时填写)
11250100  0  1
* 此为格式一：间隔interval数（网格点数-1），右坐标（模拟管则为外径）。格式二：网格间隔长度，间隔编号
11250101  4  0.0083
* 构件成分201：材料（绝对值是201MMM00卡中的MMM，此处为20100500），间隔编号
11250201  5 4
* 热构件源项分布301：源项分布相对值（网格全1即平均分布），网格间隔编号
11250301  1.0  4
* 初始温度，网格点编号
11250401  538.98  5
* 左、右边界条件501、601：边界控制体编号/通用表编号，增量（一般都是10000，对应控制体12501/2/3/.../40），边界条件类型（0对称/绝热，1/1nn对应多种对流边界，nxxx其他类型），表面积代码标记（决定后面参数5，0左表面积，1矩形体表面积/圆柱体高度/球形份额），表面积（圆柱高度）/系数，热构件编号
11250501  125010000  10000  1  1  0.1385  40
11250601  0  0  0  1  0.1385  40
* 热源数据：热源类型（此处为20297500。0无，1~999总表general table中对应功率，1000~1002反应堆中子动力学计算，10001~14095热源是控制变量，序号减10000),内热源乘子(轴向峰值因子可在该处输入，作用于参数一指明的功率)，左边界Direct moderator加热乘子（卷2节3.3），右边界Direct moderator加热乘子，热构件编号
11250701  975  1.0  0.0  0.0  40
* 其他左、右边界条件801、901：加热当量直径，向前加热长度（从加热入口到板中心的距离，衡量入口段影响，大于10后影响较小），回流加热长度（同理，加热出口到板中心），向前定位架长度，回流定位架长度，定位架向前损失系数，定位架回流损失系数，局部沸腾因子（等于局部热流密度除以从沸腾开始后的平均热流密度，若无热源/局部平衡态含汽量小于零时填1），热构件编号
11250801  0.0  1.0  1.0  0.0  0.0  0.0  0.0  1.0  40
```

## 热构件物性

热构件热物性，A12，201MMMNN

```idl
* 热构件热物性，A12，201MMMNN
* 热构件材料成分类型和数据格式，201MMM00 卡，程序内置了碳钢（C-STEEL），不锈钢（S-STEEL），二氧化铀（UO2）和锆（ZR）
20100500   c-steel

* 201MMM00 卡也可以使用用户自定义表/函数TBL/FCTN，此时需要参数二三和201MMM01-49），热导率格式标记/gap mole fraction flag（1输入表，2函数，3包含气体组分名和摩尔份额的表），体积热容标记（-1表只有热容，温度可以从热导率表对应，1输入包含温度、体积热容的表，2函数）
*20100500  TBL/FCTN  1  1
** AISI 304 e/L 不锈钢：温度（K），热导率（W/m•K）
*20100501 0.3 14.88
*20100502 343. 14.88
*20100503 373. 15.05
** AISI 304 e/L 不锈钢：温度（K），体积热容 (J/m^3/k)
*20100551 0.3    8.51e6
*20100552 343.   8.51e6
*20100553 373.   8.53e6
```

## 中子动力学

用于模拟反应堆的裂变、衰变等核反应。

```idl
*中子动力学类型（只有点堆模型）；反应性反馈类型（默认seperabl，数据在30000501，指将来自慢化剂密度、空泡权重的慢化剂温度、燃料温度的反馈看做相互独立的）
30000000  point       separabl
*裂变产物的衰变类型（no-gamma不计算，gamma标准裂变产物衰变计算，gamma-ac裂变产物衰变+锕系元素衰变计算）；总反应堆功率（裂变+裂变衰变+锕系元素衰变）；初始反应性；缓发中子份额over 瞬发中子generation time (s-1).；裂变产物yield因子；U239 yield因子；
30000001  gamma-ac    2.9529e9   0.0  268.81  1.0   1.0
*裂变产物类型（ANS73,ANS79-1,or ANS79-3）；每次裂变反应释放的能量（默认200 MeV/fission）；（其他类型需要更多参数参见输入手册）
30000002  ans73       200.0
*缓发中子precursor yield ratio(1~50组)；缓发中子衰变常数（s^-1）
30000101  0.030530    0.0125
30000102  0.205070    0.0308
30000103  0.189070    0.1147
30000104  0.395470    0.3102
30000105  0.134530    1.2316
30000106  0.045330    3.2855
*反应性曲线总表卡号（0~999）/控制变量的卡号（>10000，查找时应该减去10000）
30000011  904
*慢化剂密度反馈：密度（线性插值，若范围超出最后数据点用最后的数据）；反应性（dollar）。(此卡需要反应性反馈类型为seperabl，且30000701~709卡输入)
30000501  180.0  0.0
*慢化剂温度反馈（多普勒反应性表）：温度；反应性。(此卡需要反应性反馈类型为seperabl，且30000801~899卡输入)
30000601  473.15 0.0
```

## 总表

总表数据，A13,202TTTxx

```idl
* 表类型，202TTT00（POWER功率随时间变化，HTRNRATE热流密度，HTC-T传热系数，HTC-TEMP传热系数温度，TEMP温度，REAC-T 反应性reactivity？/ 控制系统变量，NORMAREA Normalized area versus normalized stem position）
20297500  power
* 时间，功率（W，若102卡选择british英制单位则为MW）（注意！！！是给热构件该控制体的功率，=功率密度*pi*内径*控制体高度）
20297501  0.0  4.973e3
20297502  5000.0   4.973e3
```

## 控制系统

205CCCNN

```idl
*名称；控制类型（SUM, MULT, DIV等）；缩放系数S；初值；初值标记（0无初值计算用参数4；1计算初值条件）
20501600  avgtemp     sum      0.5      0.0          1

*sum和类型：Y=S*(A0+A1*V1+A2*V2+...+Aj*Vj)。其中Y是最终结果，S是缩放系数；A是自定义常数；V是变量值。此处计算了平均温度。
*常数A0；常数A1；变量V1的名称；变量V1的控制体编号
20501601  0.0         1.0      tempf    215010000
*常数A2；变量V2的名称；变量V2的控制体编号
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
*变量V1的名称；变量V1的控制体编号；函数f()的总表卡号
20546201    cntrlvar  460        460

*stdfunc标准函数类型：Y = S*f(V1,V2,...)。例如ABS,SQRT,EXP ,LOG,SIN,COS,TAN,ATAN(arc tangent),MIN,MAX。
20540000 PCT1  stdfnctn   1.0  0.0     0     0
*函数名；变量V1的名称；变量V1的控制体编号；（V2名称；V2控制体编号；...）
20540001           max      httemp   130000109
+                           httemp   130000209
+                           httemp   130000309
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

## strip后处理运行

参考vol 2，sec 8.4。

开始strip 卡计算输出所需参数前，类似再启动，也需要先备份outdta 和 rstplt文件，然后删除indta和outdta，但保留rstplt文件（从中读数据后处理），再新建一张名为indta的strip后处理输入卡文件。运行`relap.exe`后输出stripf文件，其输出频率就是小编辑和绘图频率。需要原来的new问题输入卡中的201-299卡中的第5个参数中设置。

```idl
= Strip from SFP
100 strip fmtout
* 对于strip输入卡，103卡再启动号必为0
103 0
*==================Parameters=============
* 变量代码，参数。变量代码可以参考input manual，A4。例如p表示控制体压力，A4.3；cntrlvar表示后一个参数是控制变量的编号，在A15控制系统205CCCNN中定义
1001  p  260010000  *稳压器压力，260稳压器部件的第一个控制体的压力变量
1002  cntrlvar  260  *控制变量的编号，260在输入卡中定义为稳压器液位
.end of input
```



# 重难点及杂项

需要注意的点（从新到旧，从难到易）：



1.
2. relap5的动量方程的控制体是相邻控制体各占一半，与质量、能量方程不同。所以可以看到大编辑中速度、阻力系数等出现在在接管变量部分，而空泡份额、压力、内能等则在控制体变量部分。
3. 一般边界条件是流量入口和压力出口：
   - 入口一般设置为tmdpvol+tmdpjun。其中，tdv提供由压力和温度计算的密度，tdj提供速度，相乘得到流量入口条件。两相tdv还提供空泡份额。
   - 出口设置为sngljun+tmdpvol 。sj没用，tdv提供
   - 入口tmdpvol+sngljun应该是压力入口。
4. 时间步控制卡201中的几个输出频率，是指每隔多少时间步输出一次：
   - 小编辑输出频率、绘图输出频率、strip输出频率：用来自定义一些（特别关注的、希望在outdta中特别输出的）变量的输出频率，也是绘图时数据点的频率。这些变量的设置可以参考输入手册中的A4小编辑章节， MINOR EDITS, 301-399。
   - 大编辑输出频率：vol 2，A8.3.2。outdata文件中的大部分内容都是大编辑输出。基本囊括了所有需要用到的信息和变量数据——水力控制体、水力接管、热构件、再启动编号、再淹没等。可以看到类似字样`0MAJOR EDIT !!!time=    0.00000     sec`。
   - 再启动输出频率：会在rstplt文件中保存所有指定时刻的计算结果，用于再启动工况计算的候选的起始点。一般可以设置成>=大编辑频率。可以看到类似字样`0---Restart no.       0 written, block no.     0, at time=     0.00000    ---`。
5. 参考控制体：参考控制体在 120 -129卡指定，将其控制体中心设置为某一高度。没有指定则默认控制体编号最小的为参考控制体。如`120 280010000 0.0 h2o `，指定28001控制体为参考控制体，其中心高度为0，系统工质为轻水。
6. 接管连接控制体有两种写法：（输入手册A9，P51第二段）传统是偷懒只写出待连接组件的01出口/00进口（注意面的区别），建议写法是详细给出待连接组件的控制体的面，更好理解。如接管要连接 110400002  125010001。可以写成110010000  125000000。即，传统的写法：连接110组件和125组件的出口面01入口面00；建议写法：连接110组件第40控制体的出口面2和125组件01控制体的入口面1。
7. 控制体和接管的位置坐标：控制体是RELAP5的基本计算单元，是用于搭建具体装置的基本单元。接管用于连接控制体。控制体的坐标设置在控制体的几何中心，而接管的坐标设置在两个控制体的连接处，实际上无实体。
8. 控制体的方向与面：

   - x方向始终是流动方向，面1、2始终是流体的进、出口

   - 水平控制体：x、y、z方向分别对应笛卡尔坐标系的x、-y、z方向。face 1左2右（3前4后5下6上）。
   - 竖直控制体：x、y、z方向分别对应笛卡尔坐标系的z、-x、-y方向。face 1下2上（3前4后5右6左）。
9. 热构件的左右边界：左/右边界表示”内/外侧“。例如常用的圆柱/圆环柱，可以分别模拟加热棒和管壁，其左边界就是加热棒的中轴线、管子的内壁面。输出信息在 `0 HEAT STRUCTURE OUTPUT +++time=     0.00000     sec`
10. 0和0.0代表默认值。
11. 注释用`*`或者`$`，终止符为点号`.`。由于输出文件利用`0********`报错和`0$*8`输出控制体高度检查信息，且输出文件会复制输入卡，因此应该避免连续使用`*`或者`$`，以免混淆。
12. 字符数限制。输入卡每行不得超过80字符，但是在下一行的行首使用加号 `+` 就可以继续写。
13. 数据类型要对应。特别是要求输入real浮点数类型的时候不要输入整数（`50.`正确，`50`错误）。输入卡的数据类型分三种：整数integer，浮点数real和含有字母数字的alphanumeric。
14. 连续扩充格式简化输入。例如，125的pipe部件有40个控制体，`1250101  0.1  20`，`1250102  0.2  40`中的参数分别代指`卡号、控制体面积、控制体编号`，可以表示前20个控制体的面积都是0.1，21~40的控制体面积都是0.2。



# 实例

## 单管加热Bennett 干涸后传热



1. 计算40s，步长0.01s，每步一组绘图数据点，每秒输出一次（大编辑），每秒保留一次再启动数据。
2. 水经过一段长为5.54m、内径6.3e-3 m，面积1.2469e-4  m^2的管道加热后竖直向上流动，加热功率4.973e3 W。
3. 管内水力构件和管壁热构件都沿轴向均分为40个控制体。
4. 边界条件：入口6.9 MPa，538.98 K，速度0.1696 m/s；出口6.9 MPa，538.98 K。
5. 初值条件：管道内6.9 MPa，538.98 K，管壁538.98 K。



输入卡



```idl
* 星号/美元号注释,参考relap5的input manual，A1.4

* title card 题目卡，A1.3
= Bennett 5253

* problem type and option,A2.2，一般的new、restart等类型有瞬态transnt/稳态STDY-ST选项，strip后处理类型有格式化输出fmtout/二进制binary选项。
100 new  transnt

* input / output units,A2.4,英制单位BRITISH
102 si   si

* time step control card，A3.2
* 运行结束时间，最小时间步长（>1e-6不会生效），最大时间步长（申请），控制选项（一般3即可，半隐），小编辑和绘图输出频率（每多少步输出一次），大编辑输出频率，再启动文件（rstplt）输出频率
201   40.0  1.0e-7 0.01 3 1 100 100 *时间控制!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

*  minor edits  *  A4, 输出感兴趣的变量，此处无
```



水力部件，CCCXXNN，A9，部件的编号+卡的类型+该类型卡的编号

```idl
* boundary volume，部件名（<=8字符）和部件类型TDV，A9.4
1100000  inlet   tmdpvol
* 根据上述类型查， CCC0101-0109为TDV几何：横截面积，长度，体积，水平方位角，垂直倾斜角（[-90.0,90.0]），垂直提升高度，壁面粗糙度，水力直径，控制标记（0）
1100101  0.00012469 0.048 0.0 0.0 90.0  0.048  0.0 0.0 0
* CCC0200,TDV初始条件εbt格式（工质类型水/120-9卡指定，无有硼，如何处理CCC0201-0299中的数据），省略前0。其中t=0-3单组分，4-6双组分有不凝性气体。
* 3表示CCC0201第二三参数为压力和温度。
1100200  3
* 第一参数为搜索变量（如时间），二三由上指定为压力和温度
1100201  0.0  6.9e6  538.98  *入口温度，决定速度（压力不准）!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

* tdj ccc = 120，TDJ，A9.6
1200000  jieguan1  tmdpjun
* 来向部件，去向部件，接管面积
1200101  110010000  125000000  0.00012469
* CCC0200卡省略则二三参数为液、汽速度。第一个参数为1，则0201等的二三参数为液、汽质量流量。
1200200  1
* 此为质量流量kg/s = 质量流密度 * 流通面积
1200201  0.0  0.1696  0.0  0.0  *流量!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

* pipe CCC=125，pipe,A9.7
1250000  guanzi pipe
* 控制体数量<100，CCC0001
1250001  40  *控制体数量!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
* 控制体面积、编号，CCC0101-0199，内部接管面积0201-0299
1250101  0.00012469  40   *面积!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
* 长度   控制体编号0301，体积0401，方位角0501
1250301  0.1385     40   *控制体长度!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
* 垂直倾角601，垂直提升高度701，
1250601  90.0      40  *流动方向!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
1250701  0.1385     40
* 壁面粗糙度、水力直径、编号，801，内部接管能量损失系数901
1250801  0.0  0      40
* 控制体控制标记1001，接管控制标记1101
1251001  00        40
1251101  00000000    39
* 控制体初始条件1201：参数一控制εbt格式，二到六初始条件数据，七控制体编号（工质类型水/120-9卡指定，无有硼，如何处理后面的数据），省略前0。其中t=0-3单组分，4-6双组分有不凝性气体。（3，后面的俩参数为压力温度）
1251201  3  6.9e6  538.98  0  0  0  40
* 内部接管初始条件控制1300，0则1301为液相/气相/相界面速度（未实现，0）/编号，1则1301为液相/气相质量流量/相界面速度（0）/编号
1251300  1
1251301  0.0   0.0   0.0  39

* sj单接管 ccc = 127，A9.6
1270000  jieguan2  sngljun
* 来向部件，去向部件，面积，雷诺数独立的正向流动能量损失系数， 雷诺数独立的反向流动能量损失系数，接管控制标记 jefvcahs（j0不是喷放jet接管，e0不用能量方程PV修正项，f0不用CCFL模型，v0，c默认0开启壅塞choking模型（临界流？无1卡/1卡无50选项则为Henry-Fauske模型，否则是relap5模型），a0光滑变截面，h0非均相流两个动量方程，s0同时运用动量通量flux到来/去向部件），789字段取决于6字段c标记（临界流模型是用的Henry-Fauske模型√还是relap5模型×，此处应该是discharge coefficient, thermal nonequilibrium constant, 不使用9字段）
1270101  125010000  130000000  0.00012469  0.0  0.0  0  1.0  1.0
1270201  1  0.0  0.0  0.0

* boundary volume CCC=130，TDV
1300000  outlet   tmdpvol
1300101  0.00012469 0.048 0.0 0.0 90.0  0.048  0.0 0.0 0
1300200  3
1300201  0.0  6.9e6  538.98  *出口压力（温度不准）!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```



热构件A10，1CCCGXNN及其物性A12，201MMMNN，CCC热构件编号一般与水力部件相同，G几何编号用来区分同一水力控制体的不同热构件，X卡类型，NN该类型的编号

```idl
* 常见数据1CCCG000：轴向热构件数量（1~99）/delete（再启动时删除热构件），周向网格点数（1~99），几何类型（1矩形体2圆柱3球形），稳态初始化标记（0温度由1CCCG401-499给定，1程序计算），左边界坐标，再淹没条件标记（0无，1/2/trip卡号则需填写78字段）
11250000  40  5  2  0  0.0063  0 *圆管内径!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

* 网格位置标记（0在则在1CCCG101~201填网格信息，否则同CCCG且以下不用填)，网格格式标记(参数一是0填写，1/2则卡1CCCG101-G199格式一/二)
11250100  0  1
* 此为格式一：间隔interval数（网格点数-1），右坐标。格式二：网格间隔长度，间隔编号
11250101  4  0.0083 *圆管外径!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
* 构件成分201：材料（绝对值是201MMM00卡中的MMM，此处为20100500），间隔编号
11250201  5 4
* 热构件源项分布301：源项分布相对值（网格全1即平均分布），网格间隔编号
11250301  1.0  4
* 初始温度，网格点编号
11250401  538.98  5
* 左、右边界条件501、601：边界控制体编号/通用表编号，增量（一般都是10000，对应控制体12501/2/3/.../40），边界条件类型（0对称/绝热，1/1nn对应多种对流边界，nxxx其他类型），表面积代码标记（决定后面参数5，0左表面积，1矩形体表面积/圆柱体高度/球形份额），表面积（圆柱高度）/系数，热构件编号
11250501  125010000  10000  1  1  0.1385  40
11250601  0  0  0  1  0.1385  40
* 热源数据：热源类型（此处为20297500。0无，1~999总表general table中对应功率，1000~1002反应堆中子动力学计算，10001~14095热源是控制变量，序号减10000),内热源乘子(轴向峰值因子可在该处输入，作用于参数一指明的功率)，左边界Direct moderator加热乘子（卷2节3.3），右边界Direct moderator加热乘子，热构件编号
11250701  975  1.0  0.0  0.0  40
* 其他左边界条件801：加热当量直径，向前加热长度（从加热入口到板中心的距离，衡量入口段影响，大于10后影响较小），回流加热长度（同理，加热出口到板中心），向前定位架长度，回流定位架长度，定位架向前损失系数，定位架回流损失系数，局部沸腾因子（等于局部热流密度除以从沸腾开始后的平均热流密度，若无热源/局部平衡态含汽量小于零时填1），热构件编号
11250801  0.0  1.0  1.0  0.0  0.0  0.0  0.0  1.0  40

* 热构件热物性，A12，201MMMNN
* 热构件材料成分类型和数据格式，201MMM00 卡，程序内置了碳钢（C-STEEL），不锈钢（S-STEEL），二氧化铀（UO2）和锆（ZR）
20100500   c-steel

* 201MMM00 卡也可以使用用户自定义表/函数TBL/FCTN，此时需要参数二三和201MMM01-49），热导率格式标记/gap mole fraction flag（1输入表，2函数，3包含气体组分名和摩尔份额的表），体积热容标记（-1表只有热容，温度可以从热导率表对应，1输入包含温度、体积热容的表，2函数）
*20100500  TBL/FCTN  1  1
** AISI 304 e/L 不锈钢：温度（K），热导率（W/m•K）
*20100501 0.3 14.88
*20100502 343. 14.88
*20100503 373. 15.05
** AISI 304 e/L 不锈钢：温度（K），体积热容 (J/m^3/k)
*20100551 0.3    8.51e6
*20100552 343.   8.51e6
*20100553 373.   8.53e6
```

总表数据，A13,202TTTxx
```idl

* 表类型，202TTT00（POWER功率随时间变化，HTRNRATE热流密度，HTC-T传热系数，HTC-TEMP传热系数温度，TEMP温度，REAC-T 反应性reactivity？/ 控制系统变量，NORMAREA Normalized area versus normalized stem position）
20297500  power
* 时间，功率（W，若102卡选择british英制单位则为MW）（注意！！！是给该热构件控制体的功率，=功率密度*pi*内径*控制体高度）
20297501  0.0  4.973e3
20297502  5000.0   4.973e3  *加热功率（=功率密度*pi*内径*控制体高度）!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

* 终止卡
.End of input.
```



## 单相自然循环



## CPR1000大破口失水事故





# 其他

## 专业英语




| 英文 | 中文............... | 备注                                   |
| ---- | --------------------------------- | :------------------------------------- |
|  |  |  |
| ACC<br />accumulator | 安注水箱 | volume for safety injection |
|  |  |  |
| baffle | 堆芯围板 | 置于吊篮内侧 |
| barrel | 堆芯吊篮 | Core (support) barrel属于堆芯下部支撑结构，装载着反应堆。其他的下栅格板、热中子屏蔽、堆芯围板等都依附于其上。 |
| bypass | 旁路 | bypass flow 旁流 |
|  |  |  |
|  |  |  |
| CEDM、CRDM | 控制棒驱动机构 | control element/rod drive mechanism |
| cold leg | 冷管段（冷腿） |  |
| containment | 安全壳 | |
| Core (support) barrel | 堆芯吊篮 |属于堆芯下部支撑结构，装载着反应堆。其他的下栅格板（下侧）、热中子屏蔽（外侧）、堆芯围板（内侧）等都依附于其上。|
| core support column |  ||
| core support plate |  ||
| core baffle | 堆芯围板 |置于吊篮内侧|
| criticality | 临界状态 |堆内中子增值系数达到1，无需外界中子源就能维持功率水平|
|  |  ||
| background radiation | 本底辐射 ||
| data deck | 数据卡 |relap5输入卡|
| decay heat | 衰变热 ||
| downcomer | 下降段 ||
| dose equivalent | 剂量当量 |用来衡量辐射的生物效应，单位西弗Sv（Sievert，希沃特），=吸收剂量*品质因数（衡量某种辐射引起的生物学损伤能力）。|
|                        |                            |                                                              |
| ECCS                   | 应急堆芯冷却系统           | emergency core cooling system                                |
|  |  ||
| feedwater | 给水 |e.g. SG feedwater 蒸汽发生器给水，和堆芯流动类似也是先下降段再上升段|
| FP | 满功率 |full power|
| fuel assembly/bundles | 燃料组件，燃料棒 ||
|  |  ||
| GNPS | 大亚湾核电站（广东核电站）| Guangdong Nuclear Power Station (GNPS) |
| guide tubes | 控制棒导向管 |  |
|  | |  |
| hot leg | 热管段（热腿） |  |
| HPSI | 高压安注 | high pressure safety injection，高压安全注射 |
|  |  |  |
| inner radiation hazard | 内照射危害 | 体内放射性物质造成的危害 |
| | | |
| | | |
| latent heat | 潜热 | |
| lattice | 栅格 |  |
| LNPS | 岭澳核电站 | Lingao Nuclear Power Station (LNPS)|
| LOCA | 失水事故 | loss of coolant  accident |
| loop seal | 环路水封 | 环路水封清除（LSC，loop seal clearing）：在冷管段小破口时，如果一回路自然循环驱动力不足以克服过渡段U型管内的环路水封时，自然循环被中断，导致堆芯上腔室内蒸汽不断聚积，引起堆芯坍塌液位下降和燃料包壳峰值温度上升 |
| lower core plate | 堆芯下栅格板 |  |
| lower head | 下封头 |   |
| lower plenum | 下腔室 | |
| lower support plate | 堆芯支撑板 | 吊篮组件的底板 |
|  | | |
| moderator | 慢化剂 | |
| MSIV | 主蒸汽隔离阀 | main steam isolation valve |
| MSLB | 主蒸汽管道破裂事故 | main steam large break accident |
| | | |
| nominal | 名义值；归一化值 | 例如：nominal core power；<br />Nominalization |
| NSSS | 核蒸汽供应系统                    | nuclear steam supply system            |
|  | | |
| PCT | 燃料包壳峰值温度 | peak cladding temperature |
| power density | 功率密度 | J/(m^3*s) |
| power grid | 电网 | |
| PRA  | 概率风险评估                      | PRA (probabilistic risk assessment).   |
| prompt critical | 瞬发临界 | 反应堆仅靠瞬发中子prompt neutron（不靠缓发中子decay neutron）就能达到临界状态。此时反应堆很难控制 |
| PWR | 压水堆 | pressurized water reactor |
| PZR | 稳压器 | pressurizer |
|  | | |
| radiator | 散热器 | |
| RCS | 反应堆冷却剂系统 | reactor coolant system |
| reactor coolant pump | 主泵 |  |
| reactor (pressure) vessel | 反应堆（压力）容器 |  |
| reflector | 反射层 | 放在堆芯周围，反射中子减少损耗，使得反应堆体积可以减小 |
| relief valve | 释放阀 |  |
| RPV | 反应堆压力容器 | reactor pressure vessel |
|  |  |  |
| safety injection system | 安注系统 | 也称应急堆芯注水系统，失水事故、主蒸汽管道破裂事故时紧急注水 |
| safety valve | 安全阀 | 蒸汽发生器上，弹簧加载，超压开启 |
| SBLOCA | 小破口失水事故 | small break loss of coolant accident |
| scram | 反应堆紧急停堆 | 据说是safety control rod ax man，配备斧头的人砍断控制棒的绳索 |
| SG | 蒸汽发生器 | steam generator tube 蒸汽发生器传热管 |
| spacer grid | 定位格架 | |
| spent fuel | 乏燃料 | |
| spiders | 星形架 | 控制棒组件上控制棒的支撑结构 |
| surge line | 波动管 | pressurizer surge line |
|  | | |
| thermocouple columns | 热电偶套管 | |
| thermal shield | 热屏蔽 | 置于压力容器内壁 |
| | | |
| upper core plate | 堆芯上栅格板 | |
| upper head | 上封头 | |
| upper plenum | 上腔室 | |
|  |  | |

## rstplt&outdta变量

参见input manual A4小编辑部分


| 变量缩写 | 解释............................. | 备注                                                         |
| -------- | --------------------------------- | :----------------------------------------------------------- |
| boron    | 硼浓度                            | Spatial boron density, ρb (kg/m^3, lb/ft^3).                 |
| count    | 时间步计数                        | Current attempted advancement count number.                  |
| cputime  | CPU 时间                          | Current CPU time for this problem (s).                       |
| dt       | 当前时间步长                      | Current time step (s).                                       |
| dtcrnt   | 当前库朗特数时间步长              | Current Courant time step (s).                               |
| emass    | 质量误差                          | Estimate of mass error in all the systems (kg, lb).          |
| floreg   | 流型                              | 4泡状流bby，5弹状流slg，6环雾流anm，10雾状流mst。Flow regime number; the parameter is the volume number. A chart showing the meaning of each number is shown in Volume II, Section 2，page 14 |
| htchf    | l临界热流密度                     | Critical (maximum) heat flux (W/m^2, Btu/s•ft^2).            |
| htchfr   | 临界热流密度比率                  | Critical heat flux ratio (ratio of HTCHF to HTRNR).          |
| hthtc    | 传热系数                          | Heat transfer coefficient (W/m^2•K, Btu/s-ft^2•℉).           |
| htrnr    | 热流密度（热通量）                | Heat flux (W/m^2, Btu/s•ft^2).                               |
| httemp   | 网格点温度                        | Mesh point temperature (K, ℉).                               |
| htvat    | 热构件体积平均温度                | Volume averaged temperature in the heat structure (K, ℉).    |
| mflowj   | 总质量流率                        | Combined liquid and vapor flow rate (kg/s, lb/s).            |
| p        | 压力                              | Volume pressure (Pa, lbf/in2).                               |
| q        | 气液壁面总热源                    | Total volume heat source from the wall and direct moderator ( 慢化剂 )  heating to liquid and vapor (W, Btu/s). This variable request is the same as “Q.wall.tot.” in the major edits. |
| quala    | 不可凝气体质量份额                | Volume noncondensable mass fraction. The ratio of the mass of the noncondensable gas to the total mass of the vapor phase. |
| qualaj   | 热平衡含气率                      | Volume equilibrium quality. This quality uses phasic enthalpies and mixture quality, with the mixture enthalpy calculated using the flow quality. |
| quale    | 接管不可凝气体质量份额            | Junction noncondensable mass fraction.                       |
| quals    | 静态含气率                        | Volume static quality.                                       |
| qwg      | 气相壁面总热源                    | Volume heat source from the wall and direct moderator heating to vapor (W, Btu/s). This variable request is the same as “Qwg.wall.gas.” in the major edits. |
| rho      | 混合密度                          | Total density (kg/m^3, lb/ft^3).                             |
| rhof     | 液相密度                          | Liquid density ρf (kg/m^3, lb/ft^3).                         |
| rhofj    | 接管液相密度                      | Junction liquid density (kg/m^3, lb/ft^3).                   |
| rhog     | 气相密度                          | Vapor density ρg (kg/m^3, lb/ft^3).                          |
| rhogj    | 接管气相密度                      | Junction vapor density (kg/m^3, lb/ft^3).                    |
| sattemp  | 饱和温度（基于蒸汽分压）          | Volume saturation temperature based on the partial pressure of steam (K, ℉). |
| sounde   | 声速                              | Volume sonic velocity (m/s, ft/s).                           |
| tempf    | 液相温度                          | Volume liquid temperature Tf (K, ℉).                         |
| tempg    | 气相温度                          | Volume vapor temperature Tg (K, ℉).                          |
| time     | 模拟物理时间                      | Time (s).                                                    |
| tmass    | 系统总质量                        | Total mass of water, steam, and noncondensables in all the systems (kg, lb). |
| uf       | 液相比内能                        | Liquid specific internal energy (J/kg, Btu/lb).              |
| ufj      | 接管液相比内能                    | Junction liquid specific internal energy (J/kg, Btu/lb).     |
| ug       | 气相比内能                        | Vapor specific internal energy (J/kg, Btu/lb).               |
| ugj      | 接管气相比内能                    | Junction vapor specific internal energy (J/kg, Btu/lb).      |
| vapgen   | 蒸发速率                          | Total mass transfer rate per unit volume at the vapor/liquid interface in the bulk fluid for  vapor generation/condensation and in the boundary layer near the wall for vapor  generation/condensation (kg/m^3  •s, lb/ft^3  •s). |
| velf     | 液相速度                          | Volume oriented liquid velocity (m/s, ft/s); the parameter is the volume number plus F. |
| velfj    | 接管液相速度                      | Junction liquid velocity (m/s, ft/s). This velocity is based on the junction area Aj, which is  discussed in Volume II, Section 2.4. |
| velg     | 气相速度                          | Volume oriented vapor velocity (m/s, ft/s); the parameter is the volume number plus F. |
| velgj    | 接管气相速度                      | Junction vapor velocity (m/s, ft/s). This velocity is based on the junction area Aj, which is  discussed in Volume II, Section 2.4. |
| voidf    | 液相截面份额                      | Volume liquid fraction.                                      |
| voidfj   | 接管液相截面份额                  | Junction liquid fraction.                                    |
| voidg    | 空泡份额                          | Volume vapor fraction (void fraction).                       |
| voidgj   | 接管空泡份额                      | Junction vapor fraction (void fraction).                     |



## AptPlot快速绘图rstplt

1. 安装jdk运行环境：双击 jdk-7u51-windows-x64.exe
2. 安装aptplot：右键`AptPlotInstaller.jar`——打开方式选择java (TM) Plateform SE binary——install or update from a local folder——选择当前文件夹提取安装包（含有文件AptPlot_6.5.2）——选择安装目录——按步骤操作完成安装
3. 处理relap再启动文件rstplot：打开aptplot的file目录——read——relap5 data——选择rstplt再启动文件——在弹出的select relap channels窗口选中关心的数据channel（单击/ +ctrl/ +shift）——plot绘制——可以在data sets中查看已选数据集，继续选中channel增加数据，或者在data sets中右键显示show/隐藏hide/删除kill数据，或者clear sets清空数据集。



其他注意事项/使用技巧

1. 不懂就去查文档：Help目录——help contents
2. 导出到Origin：select relap channels窗口——data sets框——选择数据右键Edits——复制数据（如何大批量复制：选中第一行第二列单元格，拖动滚动条到最后，按住Shift时单击最后一行最后一列单元格，ctrl +c）
3. 调整坐标轴范围：Plot菜单进入或双击四周框线弹出Axes设置——general——EDit设置编辑对象——start、stop设置范围
4. 调整绘图框大小位置：双击四角的黑点不松手，拖动即可调整。也可以去Graph Appearance——Main——viewport调整XY范围。
5. 调整图例框位置Legends Box：双击图例框或点击Plot菜单弹出Graph Appearance——编辑Leg. Box Location中的X Y（分别是左、上边的位置）
6. 选择数据操作麻烦，如果关闭select relap channels窗口后还想继续选择数据，只能重新去file目录——read——relap5 data。

## UltraEdit 列编辑outdta

1. 双击 ue_chinese_64.exe 64位版官方安装程序。
2. 安装完成后关闭程序，复制 IDM_Universal_Patch_v6.0.exe 到安装目录，以管理员身份运行。
3. 选择 ultra edit v27.x -(x64) , 点击patch。
4. 若提示未找到安装路径，点击是，手动选择安装目录下的 ProtectionPlusDLL.dll 打开，再次patch。
5. 主菜单——编辑菜单——列模式，打开即可对某一列数据进行操作，对outdta很实用。

## Origin严肃画图outdta

AptPlot功能少、操作繁琐，只能绘制某个控制体的某个变量随时间变化的曲线。如果想绘制该变量随所有控制体的变化曲线，则需要使用Origin和Ultraedit来处理outdta。此外，Origin的图也被视为学术规范。

