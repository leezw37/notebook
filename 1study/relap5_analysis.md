# 问题



# 目录

[TOC]

# RELAP5-mod 3.3简介







# 其他



## 子程序简介



| 序号 | 子程序    | 功能                                                         |      |
| ---- | --------- | ------------------------------------------------------------ | ---- |
| 1    | accum     | 模拟安注箱水力、壁面和水表面的换热、蒸汽圆顶冷凝及水面蒸发的过程。 |      |
| 2    | alg63     | alg63子程序使用高斯消去法求解一个5×5矩阵方程组               |      |
| 3    | alg63n    | 子程序主要使用了高斯消去法求解两个5×5矩阵方程组，然后依据方程的解计算中间变量 |      |
| 4    | amux      | 将矩阵A以点积的形式乘以一个向量，矩阵A以压缩稀疏行的形式存储 |      |
| 5    | bishop    | 计算超临界条件下的bishop换热系数和次临界状态下的D-B换热系数，当换热包选项htopta=161时，被dittus调用 |      |
| 6    | borbnd    | 利用BPLU算法计算压力系数矩阵，包括LU分解和回代求解压力系数矩阵 |      |
| 7    | bparam    | 被tsetsl子程序调用，用于瞬态计算前BPLU结构初始化，将系统矩阵重排为border-profile的形式，是BPLU法矩阵重排的主要子程序。 |      |
| 8    | bpcnst    | bpcnst子程序被bpord子程序调用，主要用于BPLU求解器的重排技术之一——constant removal。 |      |
| 9    | bpform    | 被bpord子程序调用，主要用于BPLU求解器的重排技术之一——constant removal |      |
| 10   | bplu      | 被borbnd子程序调用，对系数矩阵进行LU分解。bplu子程序是主要的系数矩阵分解进程 |      |
| 11   | bpmap22   | 被tsetsl子程序调用，用于瞬态计算开始前BPLU结构初始化，将系统矩阵重排为border-profile的形式，是BPLU法矩阵重排的主要子程序。Bpmap22与bpmap的区别是，bpmap22只用作控制体变量个数不超过8或a22子矩阵的分解。 |      |
| 12   | bpmul     | 被setrhs子程序调用，setrhs只有在出现错误时被调用。bpmul子程序辅助setrhs，对解子矩阵行求和 |      |
| 13   | bpord     | 被bparam子程序调用，主要用于BPLU求解器的重排运算             |      |
| 14   | bppart    | 是BPLU矩阵求解器中重排处理的重要部分，主要作用是在重排后建立分区索引数组indblk，实现矩阵数值分解时的并行效果 |      |
| 15   | bprcm     | 是BPLU矩阵求解器中重排处理的重要部分，主要作用是利用Reverse Cuthill-Mckee算法对系数矩阵进行重排，使得系数矩阵形成border-banded的形式 |      |
| 16   | bpsqlu    | 被borbnd和bplu子程序调用，利用部分选主元的高斯消去法处理a（borbnd）或a22（bplu）子矩阵 |      |
| 17   | bpsqsl    | 被borbnd子程序调用，在bpsqlu和bplu子程序之后。主要作用是在bpsqlu子程或bplu的LU分解之后进行回代，得到Ax=b方程中的解x |      |
| 18   | bpsub     | 被borbnd子程序调用，在bplu子程序之后。主要作用是在LU分解后进行回代，得到Ax=b方程中的解x |      |
| 19   | brntrn    | 使用二阶 Godunov方法计算硼的输运                             |      |
| 20   | bwlim     | BPLU矩阵求解器中重排处理的重要部分，主要作用是利用带宽限制算法对系数矩阵进行重排，使得系数矩阵形成border-banded的形式。 |      |
| 21   | ccfl      | 先判断接管是否发生逆流流动限制(Countercurrent Flow Limitation,CCFL)，若在接管处存在ccfl，则利用Wallis-Kutateladze的淹没限制方程对ccfl进行计算。子程序最后根据半隐和近隐处理格式不同，分别计算其所需的中间变量；若为半隐格式，计算接管汽液两相速度velfj、velgj，计算半隐格式所需的中间变量vfdpk、vgdpk、vfdpl、vgdpl；若为近隐格式，程序计算近隐格式计算所需的中间变量coefv、sourcv、difdpk、difdpl。 |      |
| 22   | celmdr    | 计算包壳的杨氏模量和泊松比                                   |      |
| 23   | chfcal    | 根据选项采用不同的方法进行CHF的预测                          |      |
| 24   | chfitr    | 使用INTER CHF 模型计算临界热流密度                           |      |
| 25   | chfkut    | 使用Kutateladze CHF关系式和Folkin-Goldberg空泡因子以及Ivey Morris冷却因子计算临界热流密度 |      |
| 26   | chforn    | 针对窄缝板状燃料组件进行CHF的预测                            |      |
| 27   | chfosm    | 使用Osmachkin关系式计算石墨堆功率通道的临界热流密度CHF       |      |
| 28   | chfpgf    | 使用PGF关系式计算临界热流密度比通量形式                      |      |
| 29   | chfpgg    | 使用PGG关系式计算临界热流密度比几何形式                      |      |
| 30   | chfpgp    | 使用PGP关系式计算临界热流密度比的动力形式                    |      |
| 31   | chfpg     | 计算临界热流密度比                                           |      |
| 32   | chfsrl    | 根据SRL关系式计算临界热流密度                                |      |
| 33   | chftab    | 根据1986 AECL-UO CHF查询表进行CHF的预测                      |      |
| 34   | chklev    | 控制控制体之间两相液位的移动                                 |      |
| 35   | chngva    | 计算由于包壳变形引起的控制体面积变化                         |      |
| 36   | coev3d    | 计算速度矩阵系数coefv项，并计算相关动量通量项convf、convg    |      |
| 37   | conden    | 计算冷凝换热的各相换热系数和热流密度                         |      |
| 38   | condn2    | 采用了新的模型计算冷凝换热，只有当输入卡1的第45个选项开启时，才会调用这个子程序进行计算 |      |
| 39   | convar    | 用来模拟水力系统中的控制系统的子程序                         |      |
| 40   | courn1    | 计算最小基于接管速度的courant限值（时间步长限值）            |      |
| 41   | cournt    | 计算最小基于体平均速度的courant限值（时间步长限值）          |      |
| 42   | cov3dl    | 计算三维部件的动量通量项的速度矩阵方程的边界条件             |      |
| 43   | cov3dy    | 计算速度矩阵系数coefv项，并计算相关动量通量项convf、convg，针对三维控制体y方向速度。 |      |
| 44   | cov3dz    | 计算速度矩阵系数coefv项，并计算相关动量通量项convf、convg，针对三维控制体y方向速度。 |      |
| 45   | cplexp    | 使用NUREG-0630的数据计算塑性应变。该子程序假设应用了间隙导热模型，当环向应力为负的时候跳过该子程序的计算过程 |      |
| 46   | cprssr    | 计算压缩机的压差、转矩和转速，以及异步电动机的转矩           |      |
| 47   | cramer2   | 通过克莱默法则解一系列方程                                   |      |
| 48   | cthxpr    | 计算锆合金包壳的径向热膨胀                                   |      |
| 49   | daxpy     | 一个向量乘以一个常数再加上另一个向量                         |      |
| 50   | ddot      | 计算两个向量的点积(数量积)，采用开式循环使增量等于一         |      |
| 51   | detector  |                                                              |      |
| 52   | detmnt    | 采用高斯消去法用于计算行列式的值的子程序                     |      |
| 53   | dittus    | 计算Dittus-Boelter强迫对流换热关系式。                       |      |
| 54   | dmpcom1   | 指向seldmp1或selpdmp2的单元1或2写入重启记录                  |      |
| 55   | dmplst    | 计算common /ftb/ 中的参数,如数据标识、数组大小、文件数量、上次描述的文件位置等，对所有链接表循环，对每个链接表内的参数设立相关的指针参数，并通过指针参数ix、iy输出各个参数相关的指针位置 |      |
| 56   | dnrm2     | 求n维向量dx的范数                                            |      |
| 57   | dyer      | 通过GE干燥器模型计算干燥器出口的汽液空间份额                 |      |
| 58   | dtstep    | 瞬态计算过程中选择时间步长以及控制输出和画图的频率           |      |
| 59   | eccmxj    |                                                              |      |
| 60   | eccmxv    |                                                              |      |
| 61   | eprij     |                                                              |      |
| 62   | eqfinal   |                                                              |      |
| 63   | errorn    |                                                              |      |
| 64   | fidis2    |                                                              |      |
| 65   | fidisj    |                                                              |      |
| 66   | fidisv    |                                                              |      |
| 67   | fildmp    |                                                              |      |
| 68   | flux3d    |                                                              |      |
| 69   | fmvlwr    |                                                              |      |
| 70   | fricid    |                                                              |      |
| 71   | fwdrag    |                                                              |      |
| 72   | gapcon    |                                                              |      |
| 73   | gasthc    |                                                              |      |
| 74   | gcsub     |                                                              |      |
| 75   | gctp,     |                                                              |      |
| 76   | genchn    |                                                              |      |
| 77   | gesep     |                                                              |      |
| 78   | gesub     |                                                              |      |
| 79   | gniel     |                                                              |      |
| 80   | grdnrj    |                                                              |      |
| 81   | griftj    |                                                              |      |
| 82   | helphd    |                                                              |      |
| 83   | hifbub    |                                                              |      |
| 84   | hloss     |                                                              |      |
| 85   | hseflw    |                                                              |      |
| 86   | ht1tdp    |                                                              |      |
| 87   | ht2tdp    |                                                              |      |
| 88   | htadv     |                                                              |      |
| 89   | htcond    |                                                              |      |
| 90   | htfilm    |                                                              |      |
| 91   | htfinl    |                                                              |      |
| 92   | htheta    |                                                              |      |
| 93   | htlev     |                                                              |      |
| 94   | htrc1     |                                                              |      |
| 95   | htrc2     |                                                              |      |
| 96   | hydro     | 控制水力学计算过程                                           |      |
| 97   | hzflow    |                                                              |      |
| 98   | idfind    |                                                              |      |
| 99   | jacksn    |                                                              |      |
| 100  | jchoke    |                                                              |      |
| 101  | jprop     |                                                              |      |
| 102  | katokj    |                                                              |      |
| 103  | helm      |                                                              |      |
| 104  | khoo      |                                                              |      |
| 105  | kloss     |                                                              |      |
| 106  | koshi     |                                                              |      |
| 107  | level     |                                                              |      |
| 108  | lint1     |                                                              |      |
| 109  | lusol0    |                                                              |      |
| 110  | madata1   |                                                              |      |
| 111  | majout    |                                                              |      |
| 112  | mapmat    |                                                              |      |
| 113  | mcheck    |                                                              |      |
| 114  | mdata2    |                                                              |      |
| 115  | mhdfwf    |                                                              |      |
| 116  | mirec1    |                                                              |      |
| 117  | mixcon    |                                                              |      |
| 118  | modmem    |                                                              |      |
| 119  | mover     |                                                              |      |
| 120  | mprop     |                                                              |      |
| 121  | ncfilm    |                                                              |      |
| 122  | ncprop    |                                                              |      |
| 123  | ncwall    |                                                              |      |
| 124  | newtseg1  |                                                              |      |
| 125  | nnkmout   |                                                              |      |
| 126  | noncnd    |                                                              |      |
| 127  | outpoint  |                                                              |      |
| 128  | outputtr  |                                                              |      |
| 129  | packer    |                                                              |      |
| 130  | petukv    |                                                              |      |
| 131  | pgmres    |                                                              |      |
| 132  | phantj    |                                                              |      |
| 133  | phantv    |                                                              |      |
| 134  | pimplt    |                                                              |      |
| 135  | pintfc    |                                                              |      |
| 136  | plstrn    |                                                              |      |
| 137  | pltwda    |                                                              |      |
| 138  | pltwrt    |                                                              |      |
| 139  | pminv1    |                                                              |      |
| 140  | pminv4    |                                                              |      |
| 141  | pminvd    |                                                              |      |
| 142  | pminvf    |                                                              |      |
| 143  | pminvr    |                                                              |      |
| 144  | pminvx    |                                                              |      |
| 145  | pol2s     |                                                              |      |
| 146  | pol8      |                                                              |      |
| 147  | polat     |                                                              |      |
| 148  | polatr    |                                                              |      |
| 149  | prebun    |                                                              |      |
| 150  | precr     |                                                              |      |
| 151  | prednb    |                                                              |      |
| 152  | presej    |                                                              |      |
| 153  | preseq    |                                                              |      |
| 154  | psatpd    |                                                              |      |
| 155  | pset      |                                                              |      |
| 156  | pstdnb    |                                                              |      |
| 157  | pstpd2    |                                                              |      |
| 158  | pump      |                                                              |      |
| 159  | pump2     |                                                              |      |
| 160  | qfhtrc    |                                                              |      |
| 161  | qfmove    |                                                              |      |
| 162  | qfsrch    |                                                              |      |
| 163  | qmwr      |                                                              |      |
| 164  | qsplit    |                                                              |      |
| 165  | radht     |                                                              |      |
| 166  | rkin      |                                                              |      |
| 167  | rmblnk1   |                                                              |      |
| 168  | rstimg    |                                                              |      |
| 169  | rstrda    |                                                              |      |
| 170  | rstrec    |                                                              |      |
| 171  | ruplas    |                                                              |      |
| 172  | seta      |                                                              |      |
| 173  | setrhs    |                                                              |      |
| 174  | sieder    |                                                              |      |
| 175  | simplt    |                                                              |      |
| 176  | solr      |                                                              |      |
| 177  | sol       |                                                              |      |
| 178  | sortup    |                                                              |      |
| 179  | splin3    |                                                              |      |
| 180  | sstchk    |                                                              |      |
| 181  | stacc     |                                                              |      |
| 182  | statep    |                                                              |      |
| 183  | state     |                                                              |      |
| 184  | stcset    |                                                              |      |
| 185  | stdsp     |                                                              |      |
| 186  | stgodu    |                                                              |      |
| 187  | sth2x1    |                                                              |      |
| 188  | sth2x2    |                                                              |      |
| 189  | sth2x3    |                                                              |      |
| 190  | sth2x5    |                                                              |      |
| 191  | sth2x6    |                                                              |      |
| 192  | stnpu     |                                                              |      |
| 193  | stnsat    |                                                              |      |
| 194  | stntp     |                                                              |      |
| 195  | stntx     |                                                              |      |
| 196  | stnx      |                                                              |      |
| 197  | stpu00    |                                                              |      |
| 198  | stpu0p    |                                                              |      |
| 199  | stpu2p    |                                                              |      |
| 200  | stpu2t    |                                                              |      |
| 201  | stpupu    |                                                              |      |
| 202  | stputp    |                                                              |      |
| 203  | strip     |                                                              |      |
| 204  | strpu     |                                                              |      |
| 205  | strpx     |                                                              |      |
| 206  | strsat    |                                                              |      |
| 207  | strtp1    |                                                              |      |
| 208  | strtx     |                                                              |      |
| 209  | strx      |                                                              |      |
| 210  | stvnpx    |                                                              |      |
| 211  | stvrpx    |                                                              |      |
| 212  | suboil    |                                                              |      |
| 213  | surftn    |                                                              |      |
| 214  | svpu2p    |                                                              |      |
| 215  | svpupu    |                                                              |      |
| 216  | syssol    |                                                              |      |
| 217  | tcnvsl    |                                                              |      |
| 218  | tempi     |                                                              |      |
| 219  | tempifc   |                                                              |      |
| 220  | thront    |                                                              |      |
| 221  | tideal    |                                                              |      |
| 222  | timset    |                                                              |      |
| 223  | tran      | 控制瞬态计算过程                                             |      |
| 224  | trilu     |                                                              |      |
| 225  | trip      |                                                              |      |
| 226  | trisub    |                                                              |      |
| 227  | trnctl    |                                                              |      |
| 228  | trnset    |                                                              |      |
| 229  | tsetsl    |                                                              |      |
| 230  | tstate    |                                                              |      |
| 231  | turbst    |                                                              |      |
| 232  | valve     |                                                              |      |
| 233  | varvol    |                                                              |      |
| 234  | vexplt    |                                                              |      |
| 235  | vfinl     |                                                              |      |
| 236  | vimdbt    |                                                              |      |
| 237  | vimplt    |                                                              |      |
| 238  | vlvela    |                                                              |      |
| 239  | voidangle |                                                              |      |
| 240  | volvel    |                                                              |      |
| 241  | volvol    |                                                              |      |
| 242  | vtfrnt    |                                                              |      |
| 243  | zbrent    |                                                              |      |
| 244  | zfslgj    |                                                              |      |





## 1



