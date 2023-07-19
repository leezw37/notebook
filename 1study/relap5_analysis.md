# 问题



# 目录

[TOC]



# 程序主要流程



1. 输入: inputd
   1. 输入卡: rnewp
   2. 初始化: mewp
2. 瞬态计算: trnctl调用tran
   1. 时间步长控制体及输出: dtstep
   2. TDV和TDJ计算: tstate
   3. 壁面导热计算: htadv	(Qwg;Qwf、壁面蒸汽蒸发量)
      1. 再淹没导热计算: qfmove	
      2. 非再淹没一维导热计算: htatdp
         1. 对流换热计算: htrcl
            1. 壁面蒸汽产生率: suboil
            2. 单相对流换热: dittus
            3. 核态沸腾: prednb
            4. CHF:  chfcal	
            5. 过渡及膜态沸腾: pstdnb	
            6. 冷凝: conden	
   4. 水力学计算: hydro	
      1. 相间传热传质: phantv ( 得到hig`*`A，hif`*`A)
         1. 流型判断：horizhifreg，verthifreg，得到flomap(1,ix)赋值给floreg。
         2. 判断postdry：无干涸后则湿壁面相间换热wethif，干涸则dryhif。对应流型的换热。
            1. wethif：
               1. bubhif，floreg=4
               2. slughif, floreg=5
               3. amisthif, floreg=6
               4. dispwethif, floreg=7
               5. horizhif, floreg=12
      2. 相间摩擦: phantj ( 得到 fi)	
         1. horizdragreg，vertdragreg
         2.  wetdrag，drydrag
            1. wetdrag
               1. bubdrag
               2. slugdrag
               3. amistdrag
               4. dispdrag
               5. hstratdrag
         3. 为垂直分层流调整相间传热adjusthif
         4. 修正: filterdrag，finaldrag
      3. 壁面摩擦: fwdrag ( 得到 fwalf，fwalg)	
      4. 局部阻力: hloss ( 得到 hlossg、Hlossf)	
      5. 显式速度: vexplt(计算质量、动量，能量方程源项，并求解显式速度)
      6. 临界流: jchoke(计算临界流量)
      7. ccfl
      8. 最终速度和压力: vfinl
         1. 5*5系数矩阵和压力矩阵计算: preseq
         2. 求解压力矩阵: syssol
      9. 内能，空泡份额: eqfinl
      10. 物性: state
          1. stacc，计算安注箱状态方程，存储变量
          2. statep
             1.  classifyvols，控制体分类：一般控制体（非安注箱、TDV）、空气控制体和无空气的一般控制体（不凝性气体质量份额quala<1e-9，且，此时间步不会出现空气）。
             2. supertosub (numnormvol)
             3. withair (numnormvol, numairvol, matrl) 和 nwithair (numnormvol, numairvol, matrl)
             4. withoutair (numnoairvol)
             5. 最终物性：statepfinal(numnormvol,check)
             6. 质量误差检查：masserror (check,syserr,sysesq,sysmsq,sysvol,sysvsq)
      11. 水位追踪模型计算: level
      12. jprop (0): Get donered properties (include stratified void fraction)
      13. 控制体平均速度: vlvela
   5. 中子动力学: rkin
   6. 控制系统：convar
   7. 输出到outdta
3. 其他	

# 子程序速查表

| 子程序    | 功能                                                         |
| --------- | ------------------------------------------------------------ |
| accum     | 模拟安注箱水力、壁面和水表面的换热、蒸汽圆顶冷凝及水面蒸发的过程。 |
| alg63     | alg63子程序使用高斯消去法求解一个5×5矩阵方程组               |
| alg63n    | 子程序主要使用了高斯消去法求解两个5×5矩阵方程组，然后依据方程的解计算中间变量 |
| amux      | 将矩阵A以点积的形式乘以一个向量，矩阵A以压缩稀疏行的形式存储 |
| bishop    | 计算超临界条件下的bishop换热系数和次临界状态下的D-B换热系数，当换热包选项htopta=161时，被dittus调用 |
| blkdta    | 定义一些不应改变的数据                                       |
| borbnd    | 利用BPLU算法计算压力系数矩阵，包括LU分解和回代求解压力系数矩阵 |
| bparam    | 被tsetsl子程序调用，用于瞬态计算前BPLU结构初始化，将系统矩阵重排为border-profile的形式，是BPLU法矩阵重排的主要子程序。 |
| bpcnst    | bpcnst子程序被bpord子程序调用，主要用于BPLU求解器的重排技术之一——constant removal。 |
| bpform    | 被bpord子程序调用，主要用于BPLU求解器的重排技术之一——constant removal |
| bplu      | 被borbnd子程序调用，对系数矩阵进行LU分解。bplu子程序是主要的系数矩阵分解进程 |
| bpmap22   | 被tsetsl子程序调用，用于瞬态计算开始前BPLU结构初始化，将系统矩阵重排为border-profile的形式，是BPLU法矩阵重排的主要子程序。Bpmap22与bpmap的区别是，bpmap22只用作控制体变量个数不超过8或a22子矩阵的分解。 |
| bpmul     | 被setrhs子程序调用，setrhs只有在出现错误时被调用。bpmul子程序辅助setrhs，对解子矩阵行求和 |
| bpord     | 被bparam子程序调用，主要用于BPLU求解器的重排运算             |
| bppart    | 是BPLU矩阵求解器中重排处理的重要部分，主要作用是在重排后建立分区索引数组indblk，实现矩阵数值分解时的并行效果 |
| bprcm     | 是BPLU矩阵求解器中重排处理的重要部分，主要作用是利用Reverse Cuthill-Mckee算法对系数矩阵进行重排，使得系数矩阵形成border-banded的形式 |
| bpsqlu    | 被borbnd和bplu子程序调用，利用部分选主元的高斯消去法处理a（borbnd）或a22（bplu）子矩阵 |
| bpsqsl    | 被borbnd子程序调用，在bpsqlu和bplu子程序之后。主要作用是在bpsqlu子程或bplu的LU分解之后进行回代，得到Ax=b方程中的解x |
| bpsub     | 被borbnd子程序调用，在bplu子程序之后。主要作用是在LU分解后进行回代，得到Ax=b方程中的解x |
| brntrn    | 使用二阶 Godunov方法计算硼的输运                             |
| bwlim     | BPLU矩阵求解器中重排处理的重要部分，主要作用是利用带宽限制算法对系数矩阵进行重排，使得系数矩阵形成border-banded的形式。 |
| ccfl      | 先判断接管是否发生逆流流动限制(Countercurrent Flow Limitation,CCFL)，若在接管处存在ccfl，则利用Wallis-Kutateladze的淹没限制方程对ccfl进行计算。子程序最后根据半隐和近隐处理格式不同，分别计算其所需的中间变量；若为半隐格式，计算接管汽液两相速度velfj、velgj，计算半隐格式所需的中间变量vfdpk、vgdpk、vfdpl、vgdpl；若为近隐格式，程序计算近隐格式计算所需的中间变量coefv、sourcv、difdpk、difdpl。 |
| celmdr    | 计算包壳的杨氏模量和泊松比                                   |
| chfcal    | 根据选项采用不同的方法进行CHF的预测                          |
| chfitr    | 使用INTER CHF 模型计算临界热流密度                           |
| chfkut    | 使用Kutateladze CHF关系式和Folkin-Goldberg空泡因子以及Ivey Morris冷却因子计算临界热流密度 |
| chforn    | 针对窄缝板状燃料组件进行CHF的预测                            |
| chfosm    | 使用Osmachkin关系式计算石墨堆功率通道的临界热流密度CHF       |
| chfpgf    | 使用PGF关系式计算临界热流密度比通量形式                      |
| chfpgg    | 使用PGG关系式计算临界热流密度比几何形式                      |
| chfpgp    | 使用PGP关系式计算临界热流密度比的动力形式                    |
| chfpg     | 计算临界热流密度比                                           |
| chfsrl    | 根据SRL关系式计算临界热流密度                                |
| chftab    | 根据1986 AECL-UO CHF查询表进行CHF的预测                      |
| chklev    | 控制控制体之间两相液位的移动                                 |
| chngva    | 计算由于包壳变形引起的控制体面积变化                         |
| coev3d    | 计算速度矩阵系数coefv项，并计算相关动量通量项convf、convg    |
| conden    | 计算冷凝换热的各相换热系数和热流密度                         |
| condn2    | 采用了新的模型计算冷凝换热，只有当输入卡1的第45个选项开启时，才会调用这个子程序进行计算 |
| convar    | 用来模拟水力系统中的控制系统的子程序                         |
| courn1    | 计算最小基于接管速度的courant限值（时间步长限值）            |
| cournt    | 计算最小基于体平均速度的courant限值（时间步长限值）          |
| cov3dl    | 计算三维部件的动量通量项的速度矩阵方程的边界条件             |
| cov3dy    | 计算速度矩阵系数coefv项，并计算相关动量通量项convf、convg，针对三维控制体y方向速度。 |
| cov3dz    | 计算速度矩阵系数coefv项，并计算相关动量通量项convf、convg，针对三维控制体y方向速度。 |
| cplexp    | 使用NUREG-0630的数据计算塑性应变。该子程序假设应用了间隙导热模型，当环向应力为负的时候跳过该子程序的计算过程 |
| cprssr    | 计算压缩机的压差、转矩和转速，以及异步电动机的转矩           |
| cramer2   | 通过克莱默法则解一系列方程                                   |
| cthxpr    | 计算锆合金包壳的径向热膨胀                                   |
| daxpy     | 一个向量乘以一个常数再加上另一个向量                         |
| ddot      | 计算两个向量的点积(数量积)，采用开式循环使增量等于一         |
| detector  |                                                              |
| detmnt    | 采用高斯消去法用于计算行列式的值的子程序                     |
| dittus    | 计算Dittus-Boelter强迫对流换热关系式。                       |
| dmpcom1   | 指向seldmp1或selpdmp2的单元1或2写入重启记录                  |
| dmplst    | 计算common /ftb/ 中的参数,如数据标识、数组大小、文件数量、上次描述的文件位置等，对所有链接表循环，对每个链接表内的参数设立相关的指针参数，并通过指针参数ix、iy输出各个参数相关的指针位置 |
| dnrm2     | 求n维向量dx的范数                                            |
| dyer      | 通过GE干燥器模型计算干燥器出口的汽液空间份额                 |
| dtstep    | 瞬态计算过程中选择时间步长以及控制输出和画图的频率           |
| eccmxj    |                                                              |
| eccmxv    |                                                              |
| eprij     |                                                              |
| eqfinal   |                                                              |
| errorn    |                                                              |
| fidis2    |                                                              |
| fidisj    |                                                              |
| fidisv    |                                                              |
| fildmp    |                                                              |
| flux3d    |                                                              |
| fmvlwr    |                                                              |
| fricid    |                                                              |
| fwdrag    |                                                              |
| gapcon    |                                                              |
| gasthc    |                                                              |
| gcsub     |                                                              |
| gctp,     |                                                              |
| genchn    |                                                              |
| gesep     |                                                              |
| gesub     |                                                              |
| gniel     |                                                              |
| gninit    | gninit performs once only calculations and general initializations  including setting common block lengths and clearing them. |
| grdnrj    |                                                              |
| griftj    |                                                              |
| helphd    |                                                              |
| hifbub    |                                                              |
| hloss     |                                                              |
| hseflw    |                                                              |
| ht1tdp    |                                                              |
| ht2tdp    |                                                              |
| htadv     |                                                              |
| htcond    |                                                              |
| htfilm    |                                                              |
| htfinl    |                                                              |
| htheta    |                                                              |
| htlev     |                                                              |
| htrc1     |                                                              |
| htrc2     |                                                              |
| hydro     | 控制水力学计算过程                                           |
| hzflow    |                                                              |
| idfind    |                                                              |
| inputd    | Controls all input data processing.  If an error is found, editing for the case is completed, but all following cases only have their input listed |
| jacksn    |                                                              |
| jchoke    |                                                              |
| jprop     | Donors junction properties from adjacent volume quantities   |
| katokj    |                                                              |
| helm      |                                                              |
| htadv     | Controls advancement of heat structures and computes heat added to   hydrodynamic volumes. |
| khoo      |                                                              |
| kloss     |                                                              |
| koshi     |                                                              |
| level     | c  level tracks two-phase level in 1-D vertical components.<br/>c<br/>c  Cognizant engineer: kuo,wlw<br/>c<br/>c  Purpose --<br/>c  Perform calculations to detect a two-phase level within a<br/>c  hydrodynamic cell.<br/>c  Compute level propagation velocity, above-level void fraction,<br/>c  below-level void fraction, and level position related to<br/>c  the bottom of the cell.<br/>c  Set level flags.<br/>c  This subroutine consists of three main blocks.<br/>c    1) the first block determines the level position, either by<br/>c       examination on the level criteria (icheck = 0), or by<br/>c       extrapolation of the level position (icheck = 1).<br/>c    2) the second block detemines the pressure gradient correction<br/>c       term for each volume containing a level after propagating<br/>c       the level to the adjacent volume if the level position<br/>c       determined in the first block lies outside of the volume. The<br/>c       junctions effected by either level movement or by level<br/>c       appearance are marked for processing by the next block.<br/>c    3) the velocities in the junctions marked by the second block<br/>c       are recomputed to be consistent with the level positions.<br/>c  The first two blocks are contained in a loop over all volumes<br/>c  while the second loop is over a list containing the indices<br/>c  of the junctions marked by the loop over volumes. |
| lint1     |                                                              |
| lusol0    |                                                              |
| madata1   |                                                              |
| majout    |                                                              |
| mapmat    |                                                              |
| mcheck    |                                                              |
| mdata2    |                                                              |
| mhdfwf    |                                                              |
| mirec1    |                                                              |
| mixcon    |                                                              |
| modmem    |                                                              |
| mover     |                                                              |
| mprop     |                                                              |
| ncfilm    |                                                              |
| ncprop    |                                                              |
| ncwall    |                                                              |
| newtseg1  |                                                              |
| nnkmout   |                                                              |
| noncnd    |                                                              |
| nwithair  | nwithair (numnormvol, numairvol, matrl)<br />Compute equation of state and derivatives for volumes with air<br/>c  Specifically for fluid 15 = new steam tables 9-16-00 |
| outpoint  |                                                              |
| outputtr  |                                                              |
| packer    |                                                              |
| petukv    |                                                              |
| pgmres    |                                                              |
| phantj    |                                                              |
| phantv    | 相间传热<br />Subroutine computes interphase heat transfer and also calculates   some information for vexplt.  This replaces the volume loop in   earlier subroutine phaint, which replaced earlier subroutines   fidrag and mdot. |
| pimplt    |                                                              |
| pintfc    |                                                              |
| plstrn    |                                                              |
| pltwda    |                                                              |
| pltwrt    |                                                              |
| pminv1    |                                                              |
| pminv4    |                                                              |
| pminvd    |                                                              |
| pminvf    |                                                              |
| pminvr    |                                                              |
| pminvx    |                                                              |
| pol2s     |                                                              |
| pol8      |                                                              |
| polat     |                                                              |
| polatr    |                                                              |
| prebun    |                                                              |
| precr     |                                                              |
| prednb    |                                                              |
| presej    |                                                              |
| preseq    |                                                              |
| psatpd    |                                                              |
| pset      |                                                              |
| pstdnb    |                                                              |
| pstpd2    |                                                              |
| pump      |                                                              |
| pump2     |                                                              |
| pzrlev    | Added subroutine for calculate water level in PRIZER component        Much similar to icompn.f file. Since pzr volume component index,        is reset in 'trnset' before tansient calculation, use kk = przvn  This routine is called every time advancement during transient |
| qfhtrc    |                                                              |
| qfmove    |                                                              |
| qfsrch    |                                                              |
| qmwr      |                                                              |
| qsplit    |                                                              |
| radht     |                                                              |
| rcards    | Subroutine reads input data for the next case, writes data on disk if not last case. |
| rkin      |                                                              |
| rmblnk1   |                                                              |
| rstimg    |                                                              |
| rstrda    |                                                              |
| rstrec    |                                                              |
| ruplas    |                                                              |
| seta      |                                                              |
| setreason | Routine that sets all the reasons for the setting of the success flag |
| setrhs    |                                                              |
| sieder    |                                                              |
| simplt    |                                                              |
| solr      |                                                              |
| sol       |                                                              |
| sortup    |                                                              |
| splin3    |                                                              |
| sstchk    |                                                              |
| stacc     | stacc advances the energy equation, evaluates the equation of state<br/>c  for an accumulator, and performs a backup of the accumulator<br/>c  variables on a time step failure. |
| statep    | Compute equation of state and derivitives for time advanced volumes. |
| state     |                                                              |
| stcset    |                                                              |
| stdsp     |                                                              |
| stgodu    |                                                              |
| sth2x1    |                                                              |
| sth2x2    |                                                              |
| sth2x3    |                                                              |
| sth2x5    |                                                              |
| sth2x6    |                                                              |
| stnpu     |                                                              |
| stnsat    |                                                              |
| stntp     |                                                              |
| stntx     |                                                              |
| stnx      |                                                              |
| stpu00    |                                                              |
| stpu0p    |                                                              |
| stpu2p    |                                                              |
| stpu2t    |                                                              |
| stpupu    |                                                              |
| stputp    |                                                              |
| strip     |                                                              |
| strpu     |                                                              |
| strpx     |                                                              |
| strsat    |                                                              |
| strtp1    |                                                              |
| strtx     |                                                              |
| strx      |                                                              |
| stvnpx    |                                                              |
| stvrpx    |                                                              |
| suboil    |                                                              |
| surftn    |                                                              |
| svpu2p    |                                                              |
| svpupu    |                                                              |
| syssol    |                                                              |
| tcnvsl    |                                                              |
| tempi     |                                                              |
| tempifc   |                                                              |
| thront    |                                                              |
| tideal    |                                                              |
| timinit   | Initializes the timing routines                              |
| timset    | Timing subroutine.  Maintains two nested timing measures.    |
| tran      | 控制瞬态计算过程                                             |
| trilu     |                                                              |
| trip      |                                                              |
| trisub    |                                                              |
| trnctl    | calls set up, advancement, and finish subroutines for  transient problem. |
| trnset    |                                                              |
| tsetsl    |                                                              |
| tstate    | processes time dependent volumes and junctions.              |
| turbst    |                                                              |
| valve     |                                                              |
| varvol    |                                                              |
| vexplt    | Subroutine computes the explicit liquid and vapor velocities and<br/>c  the pressure gradient coefficients needed for the implicit pressure<br/>c  solution.  It also computes the old time source terms for the mass<br/>c  and energy equations. |
| vfinl     |                                                              |
| vimdbt    |                                                              |
| vimplt    |                                                              |
| vlvela    |                                                              |
| voidangle |                                                              |
| volvel    |                                                              |
| volvol    |                                                              |
| vtfrnt    |                                                              |
| wethif    | Subroutine computes wetted wall interphase heat transfer.    |
| zbrent    |                                                              |
| zfslgj    |                                                              |





# CVF源代码调试运行

### 安装

- 安装包文件夹为``Compaq.Visual.Fortran.v6.6.win10-已测试`，找到`X86文件夹`内的`SETUPX86.EXE`，右键，属性，设置以兼容模式运行这个程序（如win7,winxp），并勾选以管理员身份运行。选择右上角第一项install visual fortran。
- 名字公司随便填，序列号从`crack文件夹`内生成复制（无法全部复制，其他手敲）
- 安装方式typical，设置第一个文件夹路径，如`f: \pro\cvf\common`，则其他路径也随之变为cvf文件夹下。
- 继续安装，96%时弹出环境变量更新，选择yes。安装完成，不注册，确定，不安装array visualizer。
- 安装完毕，在安装目录下找到`DFDEV.EXE`（`F: \pro\cvf\common\MSDEV98\BIN\DFDEV.EXE`），右键属性，设置兼容模式为win7+管理员身份运行。
- 从此可以正常打开`Relap33\Relap33.dsw`项目文件，测试build菜单rebuild all，无报错则成功（中文路径会报错）。



### 使用调试

- `Relap33\Relap\`文件夹内存放有indta、outdta、rstplt和`Debug`文件夹，内含Relap.exe，可以单独拿到indta同级目录下运行。
- 设定debug模式：
  点击菜单Build/Set Active Project Configuration，选 *- Win32 Debug，OK，即设定为debug模式。
- 以debug模式执行到报错处：
  点击“Go (F5)”按钮，或直接按F5键，则执行程序，并在第一个出错语句处停止，在该语句前有一个小黄色箭头。
  若程序没错，则一直执行完毕，自动关闭dos窗口。此时，宜用“！”按钮或“Ctrl+F5”键，执行完成后，dos窗口等待用户关闭。
- `Ctrl + F10` debug 执行完光标行前一行
- `F9`设置断点：
  若希望执行时在某一语句处暂停，可将光标置于该语句，点击“手”形状的按钮，或按F9键，则程序执行到该语句时停在该语句处。
- `F10`下一行，`F11`下一步，`shift + F11`跳出子程序：区别是F10下一行（不进入子过程程序段）或者F11（遇到子过程进入子过程程序段继续单步执行）
- 查看变量值：
  小黄箭头停在某语句时，按下Variable按钮，显示当前程序段的变量值；对于简单变量，将光标放在该变量上，则即时显示该变量值。
- 运行调试时改动了源代码，如果直接链接生成Relap.exe会报错`cannot open file "Debug/Relap.exe"  Error executing link.exe.`（relap.exe还在运行没有解除占用）。退出compaq，删除outdta和rstplt，重新打开项目，ctrl+F7仅编译当前修改文件，再 F7 链接主程序relap.exe（或者rebuild all）发现无报错成功编译。
- 

```fortran

！调试插入类似语句制造某个时刻和某个控制体的断点

if ((timehy .gt. 2.70e0) .and. (volno(1,i) .eq. 110010000)) then 
	write(*,*) timehy,volno(1,i)
end if
```

## 问题

### 只能build relaplib不能relap.exe

有时报错已存在outdta，但是实际已删除，或者只能build relaplib不能build relap.exe。

在Build菜单`set active configuration`选择`Relap-Win32 Debug`。



## 调试思路

从五变量出发，XnUfUgP, 形如Ax=B,发现Uf的方程异常？问题源项或者与变量相乘的系数，源项是液相能量方程的本构之和，经查壁面无问题，相间本构有问题。

因为以前的本构中液相都是除以的液膜项，现在是三流畅，在弥散流无液膜项算不下去，替换为液膜+液滴项-12,-8

调试发现可能是不能从Mist pre-CHF MPR 7过渡到Annular mist ANM 6，两者界限在0.9999左右?直接换回relap自己的本构。（更改判断准则？考虑液滴项？）


一系列操作，可以突破0.999，到达0.86，但是却不是环雾流Annular mist ANM 6，仍然是弥散流，只有液滴没有液膜。因而发现水平工况流型判断的bug，仅判断液相，三流场时无法转换。再查看它的源项：夹带和沉降本构，发现只会沉降，不会夹带，导致液滴项份额从1e-4到1e-13。但是应该影响不大？


eqfinl.f(492): 	write(484,*)timehy,uf(i),hif(i),hig(i),voidg(i),voide(i),volno(1,i),velf(i),velg(i)

生成fort.484文件，调试查看变量值



## 测试

phantv

verthifreg

phantv

wethif

hydro

phantj





```
0******** Error return from pminvd because the matrix is singular.  row     21 is dependent on the rest.
0####################################################################################################################################
     syssol Diagnostic printout, timehy =  1.8149195E-02, dt =  8.1266095E-04, ncount =       130, help =  1, lsuces =  0, fail = F
```

0Solution array 
 ==========================================================================================================================
        sourcp(i)    sourcp(i+1)  sourcp(i+2)  sourcp(i+3)  sourcp(i+4)  sourcp(i+5)  sourcp(i+6)  sourcp(i+7)  sourcp(i+8)
 ==========================================================================================================================
         0.0000       34.193       110.15       70.189       84.382       93.208       94.192       94.759       95.032    
         95.096       94.946       94.590       94.043       93.290       92.322       91.157       89.807       88.249    
         86.475       84.509       82.365       80.048       77.548       74.867       72.003       68.957       65.715    
         62.281       58.639       54.793       50.726       46.405       41.858       37.060       31.963       26.577    
         20.899       15.045       8.8569      -1052.8     NaN            0.0000    
0Singularity parameter (if gerr .lt. 0.0, the matrix solution is singular)
 gerr =  -1.0000    
0####################################################################################################################################
      jprop Diagnostic printout, timehy =  1.8149195E-02, dt =  8.1266095E-04, ncount =       130, help =  1, lsuces =  0, fail = F
0Junctiondonoredproperties,icheck=  1







# 其他





## 传统的系统分析程序的不足

1. 两流体六方程模型的两流体和单压力假设还不够精细。
   - 两流体六方程模型只对汽相和液相建模，缺少对**液滴行为**的描述，从而在模拟一些液滴存在的工况时有较大的误差，例如：环雾流区域的壁面传热以及反应堆再淹没阶段的壁面传热。
     - 环状流区域，液膜附着在管道内壁，中心高速流动的气体会剪切液膜，产生大量的液滴，当液膜的厚度小于某个临界值之后，流体不再能导出足够的热量，导致发生沸腾危机现象中的干涸现象，在干涸后区域，液滴与壁面之间的传热扮演了重要的角色。
     - 再如在核反应堆破口事故的再淹没现象中，骤冷前沿附近产生的液滴速度较大，会进入比水位更高的位置，提前冷却壁面。
     - 总结：对于环雾流，液膜附着在管壁上，管道中心高速流动的蒸汽会剪切管壁的液膜，产生大量弥散的小液滴，同时液滴会接触液膜而沉降变为液膜，夹带沉降模型决定了环雾流区域液滴的份额，对于相间换热和相间摩擦的计算至关重要。 液滴夹带是一个较为复杂的过程，与两种现象有关[24]：1）由于近壁面液相核态沸腾产生的气泡从液膜中溢出到达主流区域，然后破裂变为小的液滴，在高壁面热流密度条件下，这部分液滴量比较多；2）在环雾流区域，附着在管壁内侧的液膜是凹凸不平的，当主流区的高速蒸汽流过液膜表面时，部分凸起的液膜会被剪切下来变为小液滴。
   - 此外两流体六方程模型中假设汽液两相处于压力平衡状态，即**假设汽相和液相压力相等**，对于泡状流、水平分层流等汽液相压力存在明显差异的工况，传统两流体模型的模拟结果不够真实。
2. 半隐数值算法的稳定性受 Courant 值的影响，对慢瞬态过程计算效率较低。当前主流的系统分析程序大都采用半隐数值算法，该算法比较成熟，且实现简单，但其会在时间上引入一阶数值误差，在模拟多物理场耦合的问题时数值误差较大，并且由于半隐数值算法稳定性受限于 **Courant 限值**，计算中**时间步长不能取太大**，这使得程序模拟长时间的慢瞬态过程时计算效率较低。
3. 核反应堆系统是一个中子物理场、流场、结构导热场等多物理场耦合的系统，然而当前主流的系统分析程序在模拟该多物理耦合系统时大都采用“脱耦”求解的方式，耦合方式分为两种：一种是每个物理场单独求解，在每一个时间步开始时刻传递参数，但这种耦合方式导致了耦合参数在时间步内存在差异；另外一种方式是 Pircard 迭代，即在一个时间步内求解完各个物理场之后，传递参数进行迭代，直至所有的物理场全部收敛在进行下一个时间步的计算，但这种耦合方式收敛速度慢，计算效率低。

## 大破口

事故初期，一回路发生破口，堆芯冷却剂从底部和顶部同时流失，堆芯水位迅速降低。破口处临界喷放导致RCS压力迅速降低，液体闪蒸，堆芯内形成的蒸汽引入负反应性，导致堆芯次临界，功率急剧下降。

由于假设事故发生同时丧失厂外电，主泵处于惰转状态，破口喷放过程中堆芯存在短暂的正向流动过程，随着泵转速的进一步下降，冷却剂通过蒸汽发生器(SG)传热管和主泵流向破口的流动阻力逐渐大于流经堆芯和下降腔的阻力，冷却剂以向下的流动为主，堆芯流量出现反转，该流动反转有利于堆芯冷却。

随着RCS压力下降，RCS压力达到ACC背压并触发其动作，ACC水经冷管段流入下降段环腔，与经堆芯、下降段环腔向破口反向流动的蒸汽相互作用，形成两相逆向流动及流动滞止(CCFL)，大量的安注冷水经下降段环腔而未经过堆芯流向破口，安注冷水被旁通。随着反向流动蒸汽流量的进一步降低，安注水经下降段进入堆芯下腔室，达到堆芯活性区底部。

随着安注冷水注入堆芯，燃料棒开始**再淹没**，伴随着堆芯储能导致的多次冷却剂闪蒸和堆芯-下降段间的流动剧烈波动，堆芯得到有效"冷却，骤冷前沿快速推进，堆芯衰变热和储能持续稳定释放，堆芯水位缓慢上升，事故后约90 s，ACC排空，约5 min后，堆芯实现淹没。

## 两相流数值算法

当前两相流数值算法大都采用**半隐数值算法**，相较于显式算法，半隐数值算法能够更加精确地模拟两相流过程，数值实现过程简单，相关研究非常成熟，且数值稳定性相对较好。

目前半隐数值算法主要可以分为两种思路，一种是将质量方程和动量方程联立，形成压力矩阵求解，迭代求解压力和速度，然后对能量方程迭代求解焓值等，应用这种思路的典型算法有 SIMPLE[48]、IMPLEC、SIMPLER、CACE[8]、ICE[49]算法以及 RELAP5 程序使用的类似的算法，这类算法在算法实现过程中只需要形成压力矩阵，矩阵的阶数与控制体的个数相同，计算效率较高，但是对于不同的问题，压力矩阵的形
成过程是不相同的，且对于多物理场全耦合系统，耦合参数不只有压力、速度、内能等流体参数，该数值算法的求解思路是不适用的。

另外一种半隐数值算法的实现思路是将所有的守恒方程联立形成一个大的方程组，然后使用牛顿迭代法求解。由于牛顿迭代法通用性很强，基于牛顿迭代法的半隐数值解法可以用于求解多物理场耦合系统。但是由于该方法将所有的守恒方程联立形成了一个阶数较大的方程组，求解方程组的效率较低，而且半隐数值算法与显式算法一样，数值稳定性受 Courant 值的限制，时间步长不能取较大的值，因此在计算长时间的慢瞬态问题时计算效率较低。



**全隐数值算法**的数值稳定性不受 Courant 值的限制，且随着计算机的发展，占据内存等缺点不再是制约全隐数值算法发展的因素，因此近年来国内外研究人员对全隐数值算法开展了一系列的研究。使用全隐数值算法求解两相流模型时，除时间项中旧时刻的值以外，守恒方程中所有的量都是未知的，且两相流数学模型是非线性方程，早期全隐数值算法的研究中通常使用经典的牛顿迭代法求解非线性方程组。已有的两相流模型都是基于没有相间本构模型的均相流模型，或是没有考虑本构模型的两流体模型，构造雅克比矩阵比较容易。

而完整的两流体模型中包含了大量的相间换热、相间摩擦、壁面换热、壁面摩擦等本构模型，且不同两相流型或者不同的换热模式下同一种本构模型也是不同的，本构模型大多都是求解变量的函数，在形成雅克比矩阵时需要每一个模型对每一个变量进行求导，该过程非常复杂，因此近年来一种不写雅克比矩阵的牛顿-克拉洛夫子空间迭代法（Jacobian-Free NewtonKrylov，JFNK）被用来求解全隐数值算法中的非线性方程组。JFNK 方法是一种高效的求解非线性方程组的迭代法，自从被提出之后在许多领域都有广泛的应用。

半隐数值算法研究比较成熟，其中使用压力-速度修正形成压力矩阵的半隐数值算法效率较高，而全隐数值算法的研究还相对较少，基于牛顿法的全隐数值算法不容易形成雅克比矩阵，且效率较低。虽然当前国内外基于 JFNK 方法的全隐数值算法在两相流方面的研究较多，对均相流、漂移流以及两流体模型等都有涉及，但是研究整体还处于起步阶段，算法的适用性和精度验证只针对数值算法本身的精度，对两相流传热的模拟也仅限于核态沸腾单管换热，一些 CHF 后传热以及反应堆瞬态事故后的重要瞬态现象中数值算法的适用性研究尚未涉及。

## JFNK 方法中预处理技术

JFNK 方法中使用 Krylov 子空间迭代法求解牛顿修正方程，Krylov 子空间迭代法基于伽辽金原理，在较小维度的子空间中寻找大型方程组的近似解，降低计算成本，提高计算效率。但是如果方程组系数矩阵性质较差，特征值分布会比较分散，量级较小的特征值很容易会被较大的特征值影响，则在有限维度的子空间中很难寻找到满足要求的方程组的近似解，因此需要通过矩阵预处理技术改善系数矩阵的性质，聚拢系数矩阵的特征值，降低在子空间中寻找近似解的难度，提高 Krylov 子空间迭代收敛速率。当在两流体模型中引入能量方程后，系数矩阵中会多出一些密度对压力、密度对内能的偏导项，这些偏导项的量级一般在 10
^-4^~10^-6^，使得雅克比系数矩阵的性质进一步病态化，系数矩阵特征值的分布很分散，Krylov 子空间迭代往往都找不到方程组的解，因此研究人员尝试对 JFNK 方法中的系数矩阵进行预处理，改善矩阵性质，加快收敛速度。

矩阵预处理技术的主要思想是寻找系数矩阵 A 的近似矩阵 P，使得方程组 A∙ x = b的求解变为求解(A∙ P^-1^)∙(P∙ x) = b，理论上预处理矩阵 P 越接近系数矩阵 A，新的系数矩阵 A∙ P^-1^越接近单位矩阵，特征值越集中分布在 1 周围，Krylov 子空间迭代法收敛速度越快。常用的预处理矩阵构造方法分为两大类，一类是通用的代数预处理技术，另一类是基于物理的预处理技术。

通用的代数预处理技术是普适的，目前主要的方法有不完全 LU 分解（ILU）预处理技术、系数矩阵逆预处理技术、多项式预处理技术以及多重网格预处理技术[65]，这些技术虽然不能对某一类问题求得最优解，但是适用性很强，对大多数问题都能起到合理的加速效果，且容易开发和使用。

第二类预处理矩阵构造方法是基于物理的预处理技术，这种方法是针对某一类特殊问题构造预处理矩阵，可以避免使用纯代数预处理时产生一些非物理意义的解，因此常用于求解一些物理问题，也是目前 JFNK 方法主流的预处理技术，该预处理技术也可以分为算子分裂法、半隐式离散法和低阶离散等三种方法。

算子分裂预处理矩阵的构造方法是利用 Picard 线性化对偏微分方程的非线性离散方程组进行线性化，然后对方程组中的方程进行排序，使得他们具有一定的耦合性。该方法常用于不同模块耦合计算的物理问题，且要求每一个物理模块的雅克比系数矩阵是容易获得的。

基于低阶离散的预处理技术使用一阶雅克比矩阵作为预处理矩阵，然后用完全 LU和不完全 LU 分解(ILU)预处理线性方程组的系数矩阵。但是该方法同样需要先构造一个低阶的雅克比矩阵，然后使用 LU 分解和 ILU 分解对其进行预处理，LU 和 ILU 分解同样会浪费计算时间，降低计算效率，因此研究人员提出了使用基于半隐式离散的预处理技术。

相比于全隐雅克比矩阵，半隐雅克比矩阵中少了部分新时刻的耦合求解项，矩阵的形式以及系数矩阵的量级与全隐雅克比矩阵是相似的，且半隐数值算法中本构模型一般是显式的，雅克比矩阵也容易获得，因此可以将半隐雅克比矩阵当作全隐数值算法的预处理矩阵，这类基于物理的预处理方法常用于全隐式离散方程组的矩阵预处理。



纯代数的预处理技术需要提前写出雅克比系数矩阵，不适用于当前两流体三流场模型全隐数值算法的预处理。基于物理的预处理技术中，算子分裂的预处理技术适用于不同物理场之间的耦合求解问题，基于低阶离散的预处理技术效率较低，而基于半隐数值算法的预处理技术计算效率更高，因此在后续研究中拟使用基于半隐数值算法的预处理技术，针对两流体三流场模型的特点构造预处理矩阵。




