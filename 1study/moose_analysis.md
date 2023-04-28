# 目录
[TOC]

# 概述
本文以MOOSE的第一个例子为研究对象，分析整个编译运行过程。其文件夹路径在 `moose/examples/ex01_inputfile`。
本文以亓欣波博士2017的两篇博客为基础拓展而来，在此表示感谢。

1. [多物理场面向对象模拟环境MOOSE运行过程分析](https://www.qixinbo.info/2017/02/14/moose-run/)

2. [MOOSE运行过程再分析](https://www.qixinbo.info/2017/07/28/moose-run-details/)

# 编译Makefile
整个编译过程都写在了Makefile文件中，修改以打开需要的模块。

## 概述
编译一个moose程序只需执行`make -j8`或者`make -j8 METHOD=dbg`，分别生成ex01-opt和ex01-dbg文件，后者可以调试，执行后输出如下：

```bash

Rebuilding symlinks in /home/lee/projects/moose/examples/ex01_inputfile/build/header_symlinks
Compiling C++ (in dbg mode) /home/lee/projects/moose/examples/ex01_inputfile/src/base/ExampleApp.C...
Compiling C++ (in dbg mode) /home/lee/projects/moose/examples/ex01_inputfile/src/main.C...
Linking Library /home/lee/projects/moose/examples/ex01_inputfile/test/lib/libex01_test-dbg.la...
Linking Library /home/lee/projects/moose/examples/ex01_inputfile/lib/libex01-dbg.la...
Linking Executable /home/lee/projects/moose/examples/ex01_inputfile/ex01-dbg...

```

从中可以看出大概流程：

1. 创建`build/header_symlinks`链接文件夹，包含程序自身include和test/include 文件夹内的所有头文件，如`ExampleApp.h`，`ExampleTestApp.h`，`xxxkernel.h`。
2. 创建`build/unity_src`文件夹，把每个src文件夹下的子文件夹分别合并成一个文件（`#include"/xxx/xxx/xxx.C"`），如`kernels_Unity.C`包含所有kernel文件。
3. 编译程序文件`src/base/ExampleApp.C`，用于注册程序、object、action等。
4. 编译真正的主程序`src/main.C`，包含初始化、注册所有相关app、运行程序等。
5. 链接库文件`lib/libex01-dbg.la`和`test/lib/libex01_test-dbg.la`，包含framework和module的文件
6. 链接生成可执行程序`ex01-dbg`。

## 定义环境变量

```makefile

EXAMPLE_DIR        ?= $(shell dirname `pwd`)
MOOSE_DIR          ?= $(shell dirname $(EXAMPLE_DIR))
FRAMEWORK_DIR      ?= $(MOOSE_DIR)/framework

```

利用?=操作符来赋值，通过调用shell的dirname命令来提取指定路径**的目录**。

对于fork出来的独立程序，路径一般与moose并列。如果是放在别的地方，就需要人为修改环境变量。

## 编译MOOSE框架和模块

```makefile

include $(FRAMEWORK_DIR)/build.mk
include $(FRAMEWORK_DIR)/moose.mk

include $(MOOSE_DIR)/modules/modules.mk

```

使用include包含基本框架framework和衍生模块modules各自的makefile文件（包含模块之间的依赖关系，自动开启相应的依赖模块。）

## 编译程序本体

```makefile

# dep apps
APPLICATION_DIR    := $(shell pwd)
APPLICATION_NAME   := ex01
BUILD_EXEC         := yes
GEN_REVISION       := no
include            $(FRAMEWORK_DIR)/app.mk
include            ../test.mk

```

这里使用framework下的app.mk这个makefile来编译本程序，涉及的路径、名字等参数会传入该文件中。

# 运行，main()
/home/lee/projects/moose/examples/ex01_inputfile/src/main.C:28

## 概述
所有程序的入口都是`main`函数，所以从这里入手能比较清晰地把握整个程序的脉络。`main`函数位于`src/main.C`文件中。

`main(int argc, char * argv[])`是C++ 的习惯写法，arg即argument参数，c即 counter计数 和v即value值。因此argc整数是统计运行主程序时给的参数个数，argv[]数组存放着每个参数。如运行`./ex01-dbg -i ex01.i`，则`argc = 3，argv = ["./ex01-dbg", "-i", "ex01.i"]`。3个参数分别是执行的程序的名字，程序的选项，选项的参数。

程序主要分4步——初始化、注册、创建和运行。

## 包含头文件

```cpp

// 声明具体问题类。
#include "ExampleApp.h"
// 该头文件包含了libMesh的libmesh头文件，从LibMeshInit中Public派生出MooseInit类。
#include "MooseInit.h"
// 用于创建和存储各种对象。
#include "MooseApp.h"
// 声明AppFactory类，用于创建各种对象。
#include "AppFactory.h"
```

包含程序本体的头文件，和MOOSE最基本的头文件。
其中，AppFactory类包含了一个宏定义：`#define registerApp(name) AppFactory::instance().reg<name>(#name)`，用于在ExampleApp中注册app对象。

程序私有的kernel、material等被包含在创建的unity_src文件中，以目标文件的形式被链接到最终的程序。所需的framework文件和module文件包含在lib/libex01-dbg.la链接文件中。

## 创建性能日志
[MOOSE性能日志PerfGraph](https://mooseframework.inl.gov/source/utils/PerfGraph.html)

```cpp

// PerfLog是libMesh的一个类，用来创建性能日志。
PerfLog Moose::perf_log("Example");

```

创建性能日志通过`TIME_SECTION`宏定义来实现，也经常会在程序中被看到，`TIME_SECTION(section_name, level, live_message="", print_dots=true)`。Section Timing是一种用于创建性能日志的技术，它可以帮助开发者快速地定位应用程序中的瓶颈和性能问题。Section Timing通过在应用程序中添加计时器并记录各个部分（或“节”）的执行时间来实现这一目标。每个“节”表示应用程序执行过程中的一个特定功能或任务，例如数据库查询、网络请求或UI渲染。通过记录每个“节”的执行时间，开发者可以确定哪些“节”需要优化以提高应用程序的性能。

## 初始化app
/home/lee/projects/moose/framework/src/base/MooseInit.C:25

`MooseInit init(argc, argv);` ，初始化MPI, solvers和 MOOSE。调用MooseInit类来进行初始化，实际上也调用了其父类LibMeshInit，从而对libMesh进行初始化。

## 注册App
/home/lee/projects/moose/examples/ex01_inputfile/src/main.C

`ExampleApp::registerApps();` ，注册这个程序的MooseApp和它依赖的东西。
每个基于MOOSE的程序都是继承自MooseApp这个类，该类提供创建各种对象的工厂factories以及存放它们的仓库warehouses。创建了私有程序后，必须实现几个重要的静态函数：`registerApps()`,`registerObjects()`, `associateSyntax() `

### 注册App本体
/home/lee/projects/moose/examples/ex01_inputfile/src/base/ExampleApp.C:41

`registerApp(ExampleApp);`其中registerApp不是一个函数或类，所以在doxygen中查不到，它是AppFactory中的一个宏定义：
`#define registerApp(name) AppFactory::instance().reg<name>(#name)`，位于`framework/include/base/AppFactory.h`

### 注册Objects

第二个是注册Objects，就是kernel、material、Executioners、UserObjects等等。本例没有用到自定义的Objects，所以该函数是空白的。但如果自定义了某个object，比如在example2中新增了一个Kernel，那么就需要注册该object：

```cpp

void
ExampleApp::registerObjects(Factory & factory)
{
  registerKernel(ExampleConvection);  // <- registration
}
```

其他类型的object也类似，比如：`registerAux()`, `registerBoundaryCondition()`, `registerMaterial()`, `registerInitialCondition()`等等。

### 关联语法
`associateSyntax()` 包括`associateSyntaxInner(syntax, action_factory);` 和 `  registerActions(syntax, action_factory);`，注册各种Action及其语法。[Action System](https://mooseframework.inl.gov/source/actions/Action.html) 完成创建Objects等任务，syntax就是解析输入文件后生成对应action的规则。

## 创建app实例
`std::shared_ptr<MooseApp> app = AppFactory::createAppShared("ExampleApp", argc, argv);` 创建一个程序实例，存储在便于清理的智能指针中。

## 运行app，MooseApp::run()
/home/lee/projects/moose/framework/src/base/MooseApp.C

`app->run();`, 这个`run`函数继承了`MooseApp::run()`，是程序运行的总调度，其除了撰写性能日志，主要执行的动作有三个：`setupOptions();`, `runInputFile();`, `executeExecutioner();`，即读取输入并配置、运行输入文件，求解。

### 选项配置及读入输入文件
`setupOptions`函数主要功能：1.`getParam`函数读取并执行程序时在命令行中输入的选项参数，如输入文件名。2.读取输入文件中的各个对象的信息。

1. `getParam`函数读取输入命令，如取得输入文件名，`_input_filename = "ex01.i"`。

    （通过调用InputParameters类的`getParamHelper`函数 ——> libMesh的parameters类的`get`函数。）

    如打印命令行中的选项参数。如运行`./ex01-dbg -h`，会读取`-h`选项参数执行`_command_line->printUsage();`，会输出所有选项参数及其用法

    ```bash

    Usage: ./ex01-dbg [<options>]
    Options:
      -h --help           Displays CLI usage statement.
      -i <input_files>    Specify one or multiple input files. Multiple files get merged into a single simulation input.
      // ...more...
    ```

2. 使用`_parser.parse(_input_filename);`解析输入文件。调用[Parser::`parse`函数](/home/lee/projects/moose/framework/src/parser/Parser.C)，通过`find(sec)`函数遍历每一个块section，执行`walkRaw(n->parent()->fullpath(), n->path(), n);`解析所有内容。

    解析过程中，会根据语法将各个块的信息读入，比如：
    `section_names = {"Mesh/", "Variables/", "Variables/diffused/", "Kernels/", "Kernels/diff/","Executioner/"}`

    然后对所有的块进行遍历，查找每一个块下对应的属性，比如对于Mesh块，其包含的属性有：
    `block_id block_name boundary_id boundary_name` 等等。

3. 通过`_action_warehouse.build();` 来构建一系列动作action，这些动作(通过`build`函数里的getSortedTask来获得)包括：`"setup_time_stepper", "add_variable",  "add_ic", "add_output", "add_kernel"` 等等。

### 执行输入文件
读入输入文件后，runInputFile()来执行里面的动作，即上一步build得到的动作列表（如读取网格文件），遍历函数：`executeActionsWithAction(task);`。并且检查输入文件中是否有未辨别出的变量。输入文件中的所有变量通过以下函数获得：`all_vars = _parser.getPotHandle()->get_variable_names();`

得到的结果是：`all_vars = {"Mesh/type", "Mesh/file", "Variables/diffused/order", "Variables/diffused/family", "Kernels/diff/type", "Kernels/diff/variable"...and more...}`

### 运行求解器，MooseApp::executeExecutioner()
主要包括初始化`_executioner->init();`和求解`_executioner->execute();`。调用`MooseApp::executeExecutioner()`函数。它们都是纯虚函数，需要根据具体的问题来求解。稳态问题调用`Steady::init()`和`Steady::execute()`，瞬态问题调用`Transient::init()`和`Transient::execute()`。
([executor](/home/lee/projects/moose/framework/src/executioners/Steady.C)似乎可以取代executioner system，输入文件写法可以不一样，但也兼容原来的executioner。)

#### 执行构造函数？
其实在执行该初始化函数之前，还隐性地执行了Executioner类的构造函数，显然在此之前先执行该构造函数的成员初始化器列表。其中：2.FEProblemBase类包含整个有限元问题的数学信息。3.Executioner构造函数本身获得求解器的信息。

```cpp
Executioner::Executioner(const InputParameters & parameters) :
 MooseObject(parameters),
 UserObjectInterface(this),
 PostprocessorInterface(this),
 Restartable(parameters, "Executioners"),
 _fe_problem(*parameters.getCheckedPointerParam<FEProblemBase *>("_fe_problem_base", "This might happen if you don't have a mesh")),
 _initial_residual_norm(std::numeric_limits<Real>::max()),
 _old_initial_residual_norm(std::numeric_limits<Real>::max()),
 _restart_file_base(getParam<FileNameNoExtension>("restart_file_base")),
 _splitting(getParam<std::vector<std::string> >("splitting"))
{}
```

##### FEProblemBase类
FEProblemBase类会构建整个有限元问题的系统，比如将问题中的变量读入到libMesh中，设定所使用的单元类型、插值次数等：`FEProblemBase::hasVariable (this=0x1028cc0, var_name="diffused")`

##### Executioner构造函数本身
Executioner本身的构造函数中主要就是获得求解器的信息，比如线性迭代和非线性迭代的精度、迭代步数等：

```cpp

// solver params
EquationSystems & es = _fe_problem.es();
es.parameters.set<Real> ("linear solver tolerance")
= getParam<Real>("l_tol");

es.parameters.set<Real> ("linear solver absolute step tolerance")
= getParam<Real>("l_abs_step_tol");

es.parameters.set<unsigned int> ("nonlinear solver maximum iterations")
= getParam<unsigned int>("nl_max_its");
// ...more...

```

#### 初始化
/home/lee/projects/moose/framework/src/executioners/Steady.C

/home/lee/projects/moose/framework/src/executioners/Transient.C

稳态初始化如下，瞬态较复杂。主要是初始化（待扩充？）和打印Framework、Mesh、Nonlinear等信息。

```cpp
void
Steady::init()
{
  checkIntegrity(); // 检查没有瞬态kernel
  _problem.execute(EXEC_PRE_MULTIAPP_SETUP); // 输出Framework、Mesh等信息
  _problem.initialSetup();
}
```
输出，调用Console的`outputSystemInformation`函数，输出求解问题的各种信息。还有`FEProblemBase::initialSetup`函数。`_app.getOutputWarehouse().mooseConsole();`函数把构造函数和对象初始化中的所有输出flush到终端。

这样就会产生如下的信息：

```
Framework Information:
MOOSE Version:           git commit c39379d4fe on 2023-04-10
LibMesh Version:         3f05f40f4a45060c1fd07281f8869a4667086291
PETSc Version:           3.16.6
SLEPc Version:           3.16.2
Current Time:            Tue Apr 18 11:30:24 2023
Executable Timestamp:    Mon Apr 17 19:24:02 2023

Parallelism:
  Num Processors:          1
  Num Threads:             1

Mesh:
  Parallel Type:           replicated
  Mesh Dimension:          3
  Spatial Dimension:       3
  Nodes:                   3774
  Elems:                   2476
  Num Subdomains:          1

Nonlinear System:
  Num DOFs:                3774
  Num Local DOFs:          3774
  Variables:               "diffused"
  Finite Element Types:    "LAGRANGE"
  Approximation Orders:    "FIRST"

Execution Information:
  Executioner:             Steady
  Solver Mode:             Preconditioned JFNK
```

#### 求解，Steady::execute()，Transient::execute()
稳态的`execute`函数包含以下几个重要函数。

```cpp

preExecute(); // 在真正计算之前要做一些提前计算，对于Steady问题没有要做的
_problem.advanceState(); // 获得上一时间步的状态，从而为进入下一时间步做好准备
preSolve(); // 在Solve之前要做的事，对于Steady问题没有要做的
_nl->solve(); // 开始求解!

```

[Steady](https://mooseframework.inl.gov/docs/doxygen/moose/Steady_8C_source.html)

[Transient](https://mooseframework.inl.gov/docs/doxygen/moose/Transient_8C_source.html)


# 详解稳态求解过程Steady::execute()
## 概述
/home/lee/projects/moose/framework/src/executioners/Steady.C

主要函数有预处理、更新状态、设置时间步、有限元求解、多程序耦合、后处理。

```cpp

preExecute(); // 预处理。对于Steady问题或者这个简单例子，无事可做
_problem.advanceState(); // 更新状态。获得上一时间步的状态，从而为进入下一时间步做好准备
_problem.timestepSetup(); // 设置时间步？还是不知道啥意思？
_last_solve_converged = _fixed_point_solve->solve(); // 有限元求解!FixedPointSolve::solve()
_problem.execMultiApps(EXEC_FINAL); // 多程序耦合的一系列函数
postExecute(); // 后处理。对于Steady问题或者这个简单例子，无事可做

```

## 更新状态advanceState
/home/lee/projects/moose/framework/src/problems/FEProblemBase.C

FEProblemBase::`advanceState`函数

1. `nl->copyOldSolutions();`遍历`_nl`复制非线性系统的旧值，_nl是一个存放所有非线性系统的verctor。实则调用`SystemBase::copyOldSolutions`函数，将`NonlinearSystemBase.h`文件中的`solutionUDot`函数、`solutionUDotDot`函数、`currentSolution`函数返回的_u_dot、_u_dotdot、Moose::PREVIOUS_NL_SOLUTION_TAG变量设置为旧值。(@param udotdot transient term)

2. `_aux->copyOldSolutions();`同理，及时向后Shift转移求解结果。

3. 对`_displaced_problem`问题也进行特殊的上述修改。

4. `_reporter_data.copyValuesBack();`调用`ReporterData::copyValuesBack()`函数，这个类帮助管理存储的已声明的Reporter object values。This design of the system is a generalization of the VectorPostprocessor system that the Reporter objects replaced.

5. 材料属性也要更新，`_material_props.shift();`， `_bnd_material_props.shift();`， `_neighbor_material_props.shift();`

## Adaptivity系统动态网格判断
遍历通过`_problem.adaptivity().getSteps()`获得的需要开始动态网格优化的步数steps。可参考[Adaptivity系统](https://mooseframework.inl.gov/moose/syntax/Adaptivity/)。

gdb 调试可以使用类似`p _problem.adaptivity()`和`p _problem`的命令输出很多东西。例如，`adaptivity`函数可以返回该系统的所有参数。

（`Steady.C`文件中可以看到有很多`#ifdef LIBMESH_ENABLE_AMR`和`#endif`，其中的代码都是会运行到的。因为moose默认开启了libmesh 库的动态网格优化功能adaptive mesh refinement (AMR)。）

## 时间步设置timestepSetup
通过调用`FEProblemBase::timestepSetup()`函数，时间步设置处理包括很多内容：瞬态、位移网格、线性搜索、材料、函数、aux辅助系统、用户对象、输出对象、耦合等，都调用函数形如`SubProblem::timestepSetup()`。( SubProblem是求解瞬态非线性问题的通用类。)

## 定点迭代真正求解，FixedPointSolve::solve()
/home/lee/projects/moose/framework/src/executioners/FixedPointSolve.C

### 概述

真正的求解，即[定点迭代法](https://en.wikipedia.org/wiki/Fixed-point_iteration)（不动点迭代法、固定点迭代法）。调用 `FixedPointSolve::solve()` 函数的流程如下：
1. 重置残差历史值。
2. 备份多程序以便求解失败后恢复。
3. 作为主程序和子程序做好松弛变量前的准备。
4. 真正的求解，定点迭代循环，输出残差。(`_has_fixed_point_its`一直是`false`)
5. 保存求解和时间步结束后的后处理器的值，以免在时间步开始时被覆盖。[postprocessors系统](https://mooseframework.inl.gov/syntax/Postprocessors/index.html)
6. 如果收敛，使用定点迭代法传输变量来更新子程序。
7. 打印`fixed point convergence`收敛原因。（跳过了，我们理解的迭代及输出搜在第四步定点迭代循环中的`solveStep`函数里）
8. 返回`converged`布尔值给`Steady.C` 的`_last_solve_converged`。

可以输出残差计算、是否收敛等内容。

```bash
0 Nonlinear |R| = 6.105359e+00
      0 Linear |R| = 6.105359e+00
...more
2 Nonlinear |R| = 3.802911e-10
 Solve Converged!
  Finished Solving                                                                       [ 16.32 s] [   37 MB]
```

### 定点迭代循环
`for (_fixed_point_it = 0; _fixed_point_it < _max_fixed_point_its; ++_fixed_point_it)`

1. 保存上次的后处理值作为求解前的值。
2. `solveStep`函数，每个时间步求解一个程序。
    * `autoAdvance`函数
    * _executioner.`preSolve`函数
    *
3. 计算并打印user-specified postprocessor assessing convergence
4. 更新时间步`_problem.dt() = current_dt; `

### FixedPointSolve::solveStep() --> FEProblemSolve::solve()

```cpp
bool FixedPointSolve::solveStep(Real & begin_norm, Real & end_norm,
                                const std::set<dof_id_type> & transformed_dofs)
{
  bool auto_advance = autoAdvance();
  // 是否自动步进。`autoAdvance`函数根据是否瞬态和是否有定点迭代。
  // 此例皆为否，因此结果为是。!(_has_fixed_point_its && _problem.isTransient());

  Real begin_norm_old = (...code...); // 计算之前的范数来给输出的范数着色。还有end_norm_old。
  _executioner.preSolve(); //
  _problem.execTransfers(EXEC_TIMESTEP_BEGIN); //
  _problem.updateMeshXFEM(); //
  _problem.execute(EXEC_TIMESTEP_BEGIN); //

  transformPostprocessors(true); // 传递定点后处理值ixed point postprocessors，要在(求解前 && 时间步初始的传输被接受后)

  begin_norm = _problem.computeResidualL2Norm(); // 计算L2范数，着色

  _problem.outputStep(EXEC_TIMESTEP_BEGIN);

  _problem.updateActiveObjects(); // 更新仓库已经激活的对象objects

  saveAllValues(true); // 在求解前保存当前变量和后处理器的值

  if (!_inner_solve->solve()){...code...}
  // 是调用FEProblemSolve::`solve`函数。
  // 1._problem.`solve`函数是真正真正真正的求解,调用FEProblemBase::`solve`函数。
  // 2.打印是否收敛和跳过求解，返回是否收敛。
}
```

#### FEProblemBase::solve()
/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:5350

```cpp
void
FEProblemBase::solve(const unsigned int nl_sys_num)
{
  setCurrentNonlinearSystem(nl_sys_num); // 获取并设置当前的非线性系统，_current_nl_sys = _nl[nl_sys_num].get();

  clearAllDofIndices(); // 清理过时的Dof索引，网格自适应会引起这个问题。

  Moose::PetscSupport::petscSetOptions(*this); // 设置PETSc选项

  Moose::setSolverDefaults(*this);

  initPetscOutput(); // 设置输出系统打印迭代信息

  possiblyRebuildGeomSearchPatches();

  _current_nl_sys->solve(); // NonlinearSystem::solve函数，位置/home/lee/projects/moose/framework/src/systems/NonlinearSystem.C

  _current_nl_sys->update();

  // sync solutions in displaced problem
  if (_displaced_problem)
    _displaced_problem->syncSolutions();

#if !PETSC_RELEASE_LESS_THAN(3, 12, 0)
  if (!_app.isUltimateMaster())
    PetscOptionsPop();
#endif
}
```

##### NonlinearSystem::solve()

FEProblemBase::computeResidualSys()

FEProblemBase::computeResidual()

/home/lee/projects/moose/framework/src/systems/NonlinearSystem.C:131

```cpp

void NonlinearSystem::solve()
{
  _nl_implicit_sys.nonlinear_solver->postcheck = Moose::compute_postcheck; // 连接postcheck函数与求解器（如果有damper或者需要更新解时）

  if (_fe_problem.solverParams()._type != Moose::ST_LINEAR) // 确定不是线性问题后，计算初始残差用于判断收敛
  {
    _fe_problem.computeResidualSys(_nl_implicit_sys, *_current_solution, *_nl_implicit_sys.rhs);
    // FEProblemBase::computeResidualSys函数 --> FEProblemBase::computeResidual函数 --> FEProblemBase::computeResidualInternal函数 --> NonlinearSystemBase::setSolution函数 --> _current_solution = &soln;

    _nl_implicit_sys.rhs->close();

    _initial_residual_before_preset_bcs = _nl_implicit_sys.rhs->l2_norm();

    if (_compute_initial_residual_before_preset_bcs)
      _console << "Initial residual before setting preset BCs: "
               << _initial_residual_before_preset_bcs << std::endl;
  }

  // Initialize the solution vector using a predictor and known values from nodal bcs
  setInitialSolution();

  // evaluate 求残差值来决定变量缩放因子
  computeScaling();

  assembleScalingVector();

  PetscNonlinearSolver<Real> & solver =
      static_cast<PetscNonlinearSolver<Real> &>(*_nl_implicit_sys.nonlinear_solver);

  if (_time_integrator) { } // (时间积分方法求解](https://mooseframework.inl.gov/syntax/Executioner/TimeIntegrator/)
  else
  {
    system().solve();
    // 1.设置——NonlinearImplicitSystem::solve() --> NonlinearImplicitSystem::set_solver_parameters()，设置最大迭代步数、残差容差tolerance等
    // 2.初始化——NonlinearImplicitSystem::solve() --> PetscNonlinearSolver<T>::init
    // 3.求解——NonlinearImplicitSystem::solve() --> nonlinear_solver->solve() --> PetscNonlinearSolver<T>::solve()——传参预处理矩阵、解的vector、残差vector、Krylov子空间

    _n_iters = _nl_implicit_sys.n_nonlinear_iterations();
    _n_linear_iters = solver.get_total_linear_iterations();
  }

  // store info about the solve
  _final_residual = _nl_implicit_sys.final_nonlinear_residual();

  if (_use_coloring_finite_difference)
    MatFDColoringDestroy(&_fdcoloring);
}
```



# 详解瞬态求解过程Transient::execute()

[Transient](https://mooseframework.inl.gov/docs/doxygen/moose/Transient_8C_source.html)

#### 初始化 init()

```cpp
void
Transient::init()
{
  if (!_time_stepper.get())
  {
    InputParameters pars = _app.getFactory().getValidParams("ConstantDT");
    pars.set<SubProblem *>("_subproblem") = &_problem;
    pars.set<Transient *>("_executioner") = this;

    // calculate "dt" properly.
    if (!_pars.isParamSetByAddParam("end_time") && !_pars.isParamSetByAddParam("num_steps") &&
        _pars.isParamSetByAddParam("dt"))
      pars.set<Real>("dt") = (getParam<Real>("end_time") - getParam<Real>("start_time")) /
                             static_cast<Real>(getParam<unsigned int>("num_steps"));
    else
      pars.set<Real>("dt") = getParam<Real>("dt");

    pars.set<bool>("reset_dt") = getParam<bool>("reset_dt");
    _time_stepper = _app.getFactory().create<TimeStepper>("ConstantDT", "TimeStepper", pars);
  }

  _problem.execute(EXEC_PRE_MULTIAPP_SETUP);
  _problem.initialSetup();

  // for restart run to override the start time,
  if (_app.isRestarting())
  {
    if (_app.hasStartTime())
      _time = _time_old = _app.getStartTime();
    else
      _time_old = _time;
  }

  _time_stepper->init();

  if (_app.isRecovering()) // Recover case
  {
    if (_t_step == 0)
      mooseError("Internal error in Transient executioner: _t_step is equal to 0 while recovering "
                 "in init().");

    _dt_old = _dt;
  }
}
```

#### 运行execute()
##### 概述
1. `preExecute`函数
2. `keepGoing`函数作为继续时间循环迭代的判断条件
3. 时间循环
    1. `incrementStepOrReject`函数求解步是否更新
    2. `preStep`函数，`preStep`函数
    3. `computeDT`函数，计算时间步长
    4. `takeStep`函数，一堆函数，计算dt及当前时间，`preSolve`函数，`step`函数，`post`函数
    5. `endStep`函数，计算误差Indicators 和 Markers，输出当前时间步的输出
    6. `postStep`函数，`postStep`函数

```cpp

void
Transient::execute()
{
  preExecute();

  // Start time loop...
  while (keepGoing())
  {
    incrementStepOrReject();
    preStep();
    computeDT();
    takeStep();
    endStep();
    postStep();
  }

  if (lastSolveConverged())
  {
    _t_step++;
    if (!_fixed_point_solve->autoAdvance())
    {
      _problem.finishMultiAppStep(EXEC_TIMESTEP_BEGIN, /*recurse_through_multiapp_levels=*/true);
      _problem.finishMultiAppStep(EXEC_TIMESTEP_END, /*recurse_through_multiapp_levels=*/true);
    }
  }

  if (!_app.halfTransient()){...}

  // This method is to finalize anything else we want to do on the problem side.
  _problem.postExecute();

  // This method can be overridden for user defined activities in the Executioner.
  postExecute();
}

```
#####


# 从kernel的computeQpResidual函数开始

main(int argc, char * argv[])
```cpp
// Begin the main program.
int
main(int argc, char * argv[])
{
  // Initialize MPI, solvers and MOOSE
  MooseInit init(argc, argv);

  // Register this application's MooseApp and any it depends on
  ExampleApp::registerApps();

  // Create an instance of the application and store it in a smart pointer for easy cleanup
  std::shared_ptr<MooseApp> app = AppFactory::createAppShared("ExampleApp", argc, argv);

  // Execute the application
  app->run();

  return 0;
}
```


MooseApp::run()
```cpp
void
MooseApp::run()
{
  TIME_SECTION("run", 3);
  if (isParamValid("show_docs") && getParam<bool>("show_docs"))
  {
    auto binname = appBinaryName();
    if (binname == "")
      mooseError("could not locate installed tests to run (unresolved binary/app name)");
    auto docspath = MooseUtils::docsDir(binname);
    if (docspath == "")
      mooseError("no installed documentation found");

    auto docmsgfile = MooseUtils::pathjoin(docspath, "docmsg.txt");
    std::string docmsg = "file://" + MooseUtils::realpath(docspath) + "/index.html";
    if (MooseUtils::pathExists(docmsgfile) && MooseUtils::checkFileReadable(docmsgfile))
    {
      std::ifstream ifs(docmsgfile);
      std::string content((std::istreambuf_iterator<char>(ifs)),
                          (std::istreambuf_iterator<char>()));
      content.replace(content.find("$LOCAL_SITE_HOME"), content.length(), docmsg);
      docmsg = content;
    }

    Moose::out << docmsg << "\n";
    _ready_to_exit = true;
    return;
  }

  if (showInputs() || copyInputs() || runInputs())
  {
    _ready_to_exit = true;
    return;
  }

  try
  {
    TIME_SECTION("setup", 2, "Setting Up");
    setupOptions();
    runInputFile();
  }
  catch (std::exception & err)
  {
    mooseError(err.what());
  }

  if (!_check_input)
  {
    TIME_SECTION("execute", 2, "Executing");
    executeExecutioner();
  }
  else
  {
    errorCheck();
    // Output to stderr, so it is easier for peacock to get the result
    Moose::err << "Syntax OK" << std::endl;
  }
}
```


MooseApp::executeExecutioner()
```cpp
void
MooseApp::executeExecutioner()
{
  TIME_SECTION("executeExecutioner", 3);

  // If ready to exit has been set, then just return
  if (_ready_to_exit)
    return;

  // run the simulation
  if (_use_executor && _executor)
  {
    Moose::PetscSupport::petscSetupOutput(_command_line.get());

    _executor->init();
    errorCheck();
    auto result = _executor->exec();
    if (!result.convergedAll())
      mooseError(result.str());
  }
  else if (_executioner)
  {
    Moose::PetscSupport::petscSetupOutput(_command_line.get());
    _executioner->init();
    errorCheck();
    _executioner->execute();
  }
  else
    mooseError("No executioner was specified (go fix your input file)");
}
```


Steady::execute()
```cpp
void
Steady::execute()
{
  if (_app.isRecovering())
  {
    _console << "\nCannot recover steady solves!\nExiting...\n" << std::endl;
    return;
  }

  _time_step = 0;
  _time = _time_step;
  _problem.outputStep(EXEC_INITIAL);
  _time = _system_time;

  preExecute();

  _problem.advanceState();

  // first step in any steady state solve is always 1 (preserving backwards compatibility)
  _time_step = 1;

#ifdef LIBMESH_ENABLE_AMR

  // Define the refinement loop
  unsigned int steps = _problem.adaptivity().getSteps();
  for (unsigned int r_step = 0; r_step <= steps; r_step++)
  {
#endif // LIBMESH_ENABLE_AMR
    _problem.timestepSetup();

    _last_solve_converged = _fixed_point_solve->solve(); // FixedPointSolve::solve()

    if (!lastSolveConverged())
    {
      _console << "Aborting as solve did not converge" << std::endl;
      break;
    }

    _problem.computeIndicators();
    _problem.computeMarkers();

    // need to keep _time in sync with _time_step to get correct output
    _time = _time_step;
    _problem.outputStep(EXEC_TIMESTEP_END);
    _time = _system_time;

#ifdef LIBMESH_ENABLE_AMR
    if (r_step != steps)
    {
      _problem.adaptMesh();
    }

    _time_step++;
  }
#endif

  {
    TIME_SECTION("final", 1, "Executing Final Objects")
    _problem.execMultiApps(EXEC_FINAL);
    _problem.finalizeMultiApps();
    _problem.postExecute();
    _problem.execute(EXEC_FINAL);
    _time = _time_step;
    _problem.outputStep(EXEC_FINAL);
    _time = _system_time;
  }

  postExecute();
}

```


FixedPointSolve::solve()
```cpp
bool
FixedPointSolve::solve()
{
  TIME_SECTION("PicardSolve", 1);

  Real current_dt = _problem.dt();

  _fixed_point_timestep_begin_norm.clear();
  _fixed_point_timestep_end_norm.clear();
  _fixed_point_timestep_begin_norm.resize(_max_fixed_point_its);
  _fixed_point_timestep_end_norm.resize(_max_fixed_point_its);

  bool converged = true;

  // need to back up multi-apps even when not doing fixed point iteration for recovering from failed
  // multiapp solve
  _problem.backupMultiApps(EXEC_TIMESTEP_BEGIN);
  _problem.backupMultiApps(EXEC_TIMESTEP_END);

  // Prepare to relax variables as a main app
  std::set<dof_id_type> transformed_dofs;
  if ((_relax_factor != 1.0 || !dynamic_cast<PicardSolve *>(this)) && _transformed_vars.size() > 0)
  {
    // Snag all of the local dof indices for all of these variables
    AllLocalDofIndicesThread aldit(_problem, _transformed_vars);
    ConstElemRange & elem_range = *_problem.mesh().getActiveLocalElementRange();
    Threads::parallel_reduce(elem_range, aldit);

    transformed_dofs = aldit.getDofIndices();
  }

  // Prepare to relax variables as a subapp
  std::set<dof_id_type> secondary_transformed_dofs;
  if (_secondary_relaxation_factor != 1.0 || !dynamic_cast<PicardSolve *>(this))
  {
    if (_secondary_transformed_variables.size() > 0)
    {
      // Snag all of the local dof indices for all of these variables
      AllLocalDofIndicesThread aldit(_problem, _secondary_transformed_variables);
      ConstElemRange & elem_range = *_problem.mesh().getActiveLocalElementRange();
      Threads::parallel_reduce(elem_range, aldit);

      secondary_transformed_dofs = aldit.getDofIndices();
    }

    // To detect a new time step
    if (_old_entering_time == _problem.time())
    {
      // Keep track of the iteration number of the main app
      _main_fixed_point_it++;

      // Save variable values before the solve. Solving will provide new values
      saveVariableValues(false);
    }
    else
      _main_fixed_point_it = 0;
  }

  for (_fixed_point_it = 0; _fixed_point_it < _max_fixed_point_its; ++_fixed_point_it)
  {
    if (_has_fixed_point_its)
    {
      if (_fixed_point_it == 0)
      {
        if (_has_fixed_point_norm)
        {
          // First fixed point iteration - need to save off the initial nonlinear residual
          _fixed_point_initial_norm = _problem.computeResidualL2Norm();
          _console << COLOR_MAGENTA << "Initial fixed point residual norm: " << COLOR_DEFAULT;
          if (_fixed_point_initial_norm == std::numeric_limits<Real>::max())
            _console << " MAX ";
          else
            _console << std::scientific << _fixed_point_initial_norm;
          _console << COLOR_DEFAULT << "\n" << std::endl;
        }
      }
      else
      {
        // For every iteration other than the first, we need to restore the state of the MultiApps
        _problem.restoreMultiApps(EXEC_TIMESTEP_BEGIN);
        _problem.restoreMultiApps(EXEC_TIMESTEP_END);
      }

      _console << COLOR_MAGENTA << "Beginning fixed point iteration " << _fixed_point_it
               << COLOR_DEFAULT << std::endl
               << std::endl;
    }

    // Save last postprocessor value as value before solve
    if (_fixed_point_custom_pp && _fixed_point_it > 0 && !getParam<bool>("direct_pp_value"))
      _pp_old = *_fixed_point_custom_pp;

    // Solve a single application for one time step
    bool solve_converged = solveStep(_fixed_point_timestep_begin_norm[_fixed_point_it],
                                     _fixed_point_timestep_end_norm[_fixed_point_it],
                                     transformed_dofs); // FixedPointSolve::solveStep()

    // Get new value and print history for the custom postprocessor convergence criterion
    if (_fixed_point_custom_pp)
      computeCustomConvergencePostprocessor();

    if (solve_converged)
    {
      if (_has_fixed_point_its)
      {
        if (_has_fixed_point_norm)
          // Print the evolution of the main app residual over the fixed point iterations
          printFixedPointConvergenceHistory();

        // Examine convergence metrics & properties and set the convergence reason
        bool break_out = examineFixedPointConvergence(converged);
        if (break_out)
          break;
      }
    }
    else
    {
      // If the last solve didn't converge then we need to exit this step completely (even in the
      // case of coupling). So we can retry...
      converged = false;
      break;
    }

    _problem.dt() =
        current_dt; // _dt might be smaller than this at this point for multistep methods
  }

  // Save postprocessors after the solve and their potential timestep_end execution
  // The postprocessors could be overwritten at timestep_begin, which is why they are saved
  // after the solve. They could also be saved right after the transfers.
  if (_old_entering_time == _problem.time())
    savePostprocessorValues(false);

  if (converged)
  {
    // Update the subapp using the fixed point algorithm
    if (_secondary_transformed_variables.size() > 0 &&
        useFixedPointAlgorithmUpdateInsteadOfPicard(false) && _old_entering_time == _problem.time())
      transformVariables(secondary_transformed_dofs, false);

    // Update the entering time, used to detect failed solves
    _old_entering_time = _problem.time();
  }

  if (_has_fixed_point_its)
    printFixedPointConvergenceReason();

  // clear history to avoid displaying it again on next solve that can happen for example during
  // transient
  _pp_history.str("");

  return converged;
}
```


FixedPointSolve::solveStep()
```cpp
bool
FixedPointSolve::solveStep(Real & begin_norm,
                           Real & end_norm,
                           const std::set<dof_id_type> & transformed_dofs)
{
  bool auto_advance = autoAdvance();

  // Compute previous norms for coloring the norm output
  Real begin_norm_old = (_fixed_point_it > 0 ? _fixed_point_timestep_begin_norm[_fixed_point_it - 1]
                                             : std::numeric_limits<Real>::max());
  Real end_norm_old = (_fixed_point_it > 0 ? _fixed_point_timestep_end_norm[_fixed_point_it - 1]
                                           : std::numeric_limits<Real>::max());

  _executioner.preSolve();

  _problem.execTransfers(EXEC_TIMESTEP_BEGIN);
  if (!_problem.execMultiApps(EXEC_TIMESTEP_BEGIN, auto_advance))
  {
    _fixed_point_status = MooseFixedPointConvergenceReason::DIVERGED_FAILED_MULTIAPP;
    return false;
  }

  if (_problem.haveXFEM() && _update_xfem_at_timestep_begin)
    _problem.updateMeshXFEM();

  _problem.execute(EXEC_TIMESTEP_BEGIN);

  // Transform the fixed point postprocessors before solving, but after the timestep_begin transfers
  // have been received
  if (_transformed_pps.size() > 0 && useFixedPointAlgorithmUpdateInsteadOfPicard(true))
    transformPostprocessors(true);
  if (_secondary_transformed_pps.size() > 0 && useFixedPointAlgorithmUpdateInsteadOfPicard(false) &&
      _problem.time() == _old_entering_time)
    transformPostprocessors(false);

  if (_has_fixed_point_its && _has_fixed_point_norm)
    if (_problem.hasMultiApps(EXEC_TIMESTEP_BEGIN) || _fixed_point_force_norms)
    {
      begin_norm = _problem.computeResidualL2Norm();

      _console << COLOR_MAGENTA << "Fixed point residual norm after TIMESTEP_BEGIN MultiApps: "
               << Console::outputNorm(begin_norm_old, begin_norm) << std::endl;
    }

  // Perform output for timestep begin
  _problem.outputStep(EXEC_TIMESTEP_BEGIN);

  // Update warehouse active objects
  _problem.updateActiveObjects();

  // Save the current values of variables and postprocessors, before the solve
  saveAllValues(true);

  if (_has_fixed_point_its)
    _console << COLOR_MAGENTA << "\nMain app solve:" << COLOR_DEFAULT << std::endl;
  if (!_inner_solve->solve()) // FEProblemSolve::solve()
  {
    _fixed_point_status = MooseFixedPointConvergenceReason::DIVERGED_NONLINEAR;

    // Perform the output of the current, failed time step (this only occurs if desired)
    _problem.outputStep(EXEC_FAILED);
    return false;
  }
  else
    _fixed_point_status = MooseFixedPointConvergenceReason::CONVERGED_NONLINEAR;

  // Use the fixed point algorithm if the conditions (availability of values, etc) are met
  if (_transformed_vars.size() > 0 && useFixedPointAlgorithmUpdateInsteadOfPicard(true))
    transformVariables(transformed_dofs, true);

  if (_problem.haveXFEM() && (_xfem_update_count < _max_xfem_update) && _problem.updateMeshXFEM())
  {
    _console << "\nXFEM modified mesh, repeating step" << std::endl;
    _xfem_repeat_step = true;
    ++_xfem_update_count;
  }
  else
  {
    if (_problem.haveXFEM())
    {
      _xfem_repeat_step = false;
      _xfem_update_count = 0;
      _console << "\nXFEM did not modify mesh, continuing" << std::endl;
    }

    _problem.onTimestepEnd();
    _problem.execute(EXEC_TIMESTEP_END);

    _problem.execTransfers(EXEC_TIMESTEP_END);
    if (!_problem.execMultiApps(EXEC_TIMESTEP_END, auto_advance))
    {
      _fixed_point_status = MooseFixedPointConvergenceReason::DIVERGED_FAILED_MULTIAPP;
      return false;
    }
  }

  if (_fail_step)
  {
    _fail_step = false;
    return false;
  }

  _executioner.postSolve();

  if (_has_fixed_point_its && _has_fixed_point_norm)
    if (_problem.hasMultiApps(EXEC_TIMESTEP_END) || _fixed_point_force_norms)
    {
      end_norm = _problem.computeResidualL2Norm();

      _console << COLOR_MAGENTA << "Fixed point residual norm after TIMESTEP_END MultiApps: "
               << Console::outputNorm(end_norm_old, end_norm) << std::endl;
    }

  return true;
}
```


FEProblemSolve::solve()
```cpp

bool
FEProblemSolve::solve()
{
  // This loop is for nonlinear multigrids (developed by Alex)
  for (MooseIndex(_num_grid_steps) grid_step = 0; grid_step <= _num_grid_steps; ++grid_step)
  {
    _problem.solve(); // FEProblemBase::solve(const unsigned int nl_sys_num)

    if (_problem.shouldSolve())
    {
      if (_problem.converged())
        _console << COLOR_GREEN << " Solve Converged!" << COLOR_DEFAULT << std::endl;
      else
      {
        _console << COLOR_RED << " Solve Did NOT Converge!" << COLOR_DEFAULT << std::endl;
        return false;
      }
    }
    else
      _console << COLOR_GREEN << " Solve Skipped!" << COLOR_DEFAULT << std::endl;

    if (grid_step != _num_grid_steps)
      _problem.uniformRefine();
  }
  return _problem.converged();
}
```


FEProblemBase::solve(const unsigned int nl_sys_num)
```cpp
void
FEProblemBase::solve(const unsigned int nl_sys_num)
{
  TIME_SECTION("solve", 1, "Solving", false);

  setCurrentNonlinearSystem(nl_sys_num);

  // This prevents stale dof indices from lingering around and possibly leading to invalid reads and
  // writes. Dof indices may be made stale through operations like mesh adaptivity
  clearAllDofIndices();
  if (_displaced_problem)
    _displaced_problem->clearAllDofIndices();

#if PETSC_RELEASE_LESS_THAN(3, 12, 0)
  Moose::PetscSupport::petscSetOptions(*this); // Make sure the PETSc options are setup for this app
#else
  // Now this database will be the default
  // Each app should have only one database
  if (!_app.isUltimateMaster())
    PetscOptionsPush(_petsc_option_data_base);
  // We did not add PETSc options to database yet
  if (!_is_petsc_options_inserted)
  {
    Moose::PetscSupport::petscSetOptions(*this);
    _is_petsc_options_inserted = true;
  }
#endif
  // set up DM which is required if use a field split preconditioner
  // We need to setup DM every "solve()" because libMesh destroy SNES after solve()
  // Do not worry, DM setup is very cheap
  if (_current_nl_sys->haveFieldSplitPreconditioner())
    Moose::PetscSupport::petscSetupDM(*_current_nl_sys);

  Moose::setSolverDefaults(*this);

  // Setup the output system for printing linear/nonlinear iteration information
  initPetscOutput();

  possiblyRebuildGeomSearchPatches();

  // reset flag so that linear solver does not use
  // the old converged reason "DIVERGED_NANORINF", when
  // we throw  an exception and stop solve
  _fail_next_linear_convergence_check = false;

  if (_solve)
    _current_nl_sys->solve(); // NonlinearSystem::solve()

  if (_solve)
    _current_nl_sys->update();

  // sync solutions in displaced problem
  if (_displaced_problem)
    _displaced_problem->syncSolutions();

#if !PETSC_RELEASE_LESS_THAN(3, 12, 0)
  if (!_app.isUltimateMaster())
    PetscOptionsPop();
#endif
}

```


NonlinearSystem::solve()
```cpp
void
NonlinearSystem::solve()
{
  // Only attach the postcheck function to the solver if we actually
  // have dampers or if the FEProblemBase needs to update the solution,
  // which is also done during the linesearch postcheck.  It doesn't
  // hurt to do this multiple times, it is just setting a pointer.
  if (_fe_problem.hasDampers() || _fe_problem.shouldUpdateSolution() ||
      _fe_problem.needsPreviousNewtonIteration())
    _nl_implicit_sys.nonlinear_solver->postcheck = Moose::compute_postcheck;

  if (_fe_problem.solverParams()._type != Moose::ST_LINEAR)
  {
    TIME_SECTION("nlInitialResidual", 3, "Computing Initial Residual");
    // Calculate the initial residual for use in the convergence criterion.
    _computing_initial_residual = true;
    _fe_problem.computeResidualSys(_nl_implicit_sys, *_current_solution, *_nl_implicit_sys.rhs); // FEProblemBase::computeResidualSys()
    _computing_initial_residual = false;
    _nl_implicit_sys.rhs->close();
    _initial_residual_before_preset_bcs = _nl_implicit_sys.rhs->l2_norm();
    if (_compute_initial_residual_before_preset_bcs)
      _console << "Initial residual before setting preset BCs: "
               << _initial_residual_before_preset_bcs << std::endl;
  }

  // Clear the iteration counters
  _current_l_its.clear();
  _current_nl_its = 0;

  // Initialize the solution vector using a predictor and known values from nodal bcs
  setInitialSolution();

  // Now that the initial solution has ben set, potentially perform a residual/Jacobian evaluation
  // to determine variable scaling factors
  if (_automatic_scaling)
  {
    if (_compute_scaling_once)
    {
      if (!_computed_scaling)
      {
        computeScaling();
        _computed_scaling = true;
      }
    }
    else
      computeScaling();
  }
#ifdef MOOSE_GLOBAL_AD_INDEXING
  // We do not know a priori what variable a global degree of freedom corresponds to, so we need a
  // map from global dof to scaling factor. We just use a ghosted NumericVector for that mapping
  assembleScalingVector();
#endif

  if (_use_finite_differenced_preconditioner)
  {
    _nl_implicit_sys.nonlinear_solver->fd_residual_object = &_fd_residual_functor;
    setupFiniteDifferencedPreconditioner();
  }

  PetscNonlinearSolver<Real> & solver =
      static_cast<PetscNonlinearSolver<Real> &>(*_nl_implicit_sys.nonlinear_solver);
  solver.mffd_residual_object = &_fd_residual_functor;

  solver.set_snesmf_reuse_base(_fe_problem.useSNESMFReuseBase());

  if (_time_integrator)
  {
    _time_integrator->solve();
    _time_integrator->postSolve();
    _n_iters = _time_integrator->getNumNonlinearIterations();
    _n_linear_iters = _time_integrator->getNumLinearIterations();
  }
  else
  {
    system().solve();
    _n_iters = _nl_implicit_sys.n_nonlinear_iterations();
    _n_linear_iters = solver.get_total_linear_iterations();
  }

  // store info about the solve
  _final_residual = _nl_implicit_sys.final_nonlinear_residual();

  if (_use_coloring_finite_difference)
    MatFDColoringDestroy(&_fdcoloring);
}

```


FEProblemBase::computeResidualSys()
```cpp
void
FEProblemBase::computeResidualSys(NonlinearImplicitSystem & sys,
                                  const NumericVector<Number> & soln,
                                  NumericVector<Number> & residual)
{
  TIME_SECTION("computeResidualSys", 5);

  ADReal::do_derivatives = false;

  computeResidual(soln, residual, sys.number()); // FEProblemBase::computeResidual()

  ADReal::do_derivatives = true;
}
```


FEProblemBase::computeResidual()
```cpp
void
FEProblemBase::computeResidual(const NumericVector<Number> & soln,
                               NumericVector<Number> & residual,
                               const unsigned int nl_sys_num)
{
  setCurrentNonlinearSystem(nl_sys_num);
  const auto & residual_vector_tags = getVectorTags(Moose::VECTOR_TAG_RESIDUAL);

  _fe_vector_tags.clear();

  for (const auto & residual_vector_tag : residual_vector_tags)
    _fe_vector_tags.insert(residual_vector_tag._id);

  computeResidualInternal(soln, residual, _fe_vector_tags); // FEProblemBase::computeResidualInternal()
}

```

FEProblemBase::computeResidualInternal()
```cpp
void
FEProblemBase::computeResidualInternal(const NumericVector<Number> & soln,
                                       NumericVector<Number> & residual,
                                       const std::set<TagID> & tags)
{
  TIME_SECTION("computeResidualInternal", 1);

  try
  {
    _current_nl_sys->setSolution(soln);

    _current_nl_sys->associateVectorToTag(residual, _current_nl_sys->residualVectorTag());

    computeResidualTags(tags); // FEProblemBase::computeResidualTags(const std::set<TagID> & tags)

    _current_nl_sys->disassociateVectorFromTag(residual, _current_nl_sys->residualVectorTag());
  }
  catch (MooseException & e)
  {
    // If a MooseException propagates all the way to here, it means
    // that it was thrown from a MOOSE system where we do not
    // (currently) properly support the throwing of exceptions, and
    // therefore we have no choice but to error out.  It may be
    // *possible* to handle exceptions from other systems, but in the
    // meantime, we don't want to silently swallow any unhandled
    // exceptions here.
    mooseError("An unhandled MooseException was raised during residual computation.  Please "
               "contact the MOOSE team for assistance.");
  }
}
```


FEProblemBase::computeResidualTags(const std::set<TagID> & tags)
```cpp
void
FEProblemBase::computeResidualTags(const std::set<TagID> & tags)
{
  TIME_SECTION("computeResidualTags", 5, "Computing Residual");

  _aux->zeroVariablesForResidual();

  unsigned int n_threads = libMesh::n_threads();

  _current_execute_on_flag = EXEC_LINEAR;

  // Random interface objects
  for (const auto & it : _random_data_objects)
    it.second->updateSeeds(EXEC_LINEAR);

  execTransfers(EXEC_LINEAR);

  execMultiApps(EXEC_LINEAR);

  for (unsigned int tid = 0; tid < n_threads; tid++)
    reinitScalars(tid);

  computeUserObjects(EXEC_LINEAR, Moose::PRE_AUX);

  _aux->residualSetup();

  if (_displaced_problem)
  {
    _aux->compute(EXEC_PRE_DISPLACE);
    try
    {
      try
      {
        _displaced_problem->updateMesh();
        if (_mortar_data.hasDisplacedObjects())
          updateMortarMesh();
      }
      catch (libMesh::LogicError & e)
      {
        throw MooseException("We caught a libMesh error in FEProblemBase");
      }
    }
    catch (MooseException & e)
    {
      setException(e.what());
    }
    try
    {
      // Propagate the exception to all processes if we had one
      checkExceptionAndStopSolve();
    }
    catch (MooseException &)
    {
      // Just end now. We've inserted our NaN into the residual vector
      return;
    }
  }

  for (THREAD_ID tid = 0; tid < n_threads; tid++)
  {
    _all_materials.residualSetup(tid);
    _functions.residualSetup(tid);
  }

  // Where is the aux system done? Could the non-current nonlinear systems also be done there?
  for (auto & nl : _nl)
    nl->computeTimeDerivatives();

  try
  {
    _aux->compute(EXEC_LINEAR);
  }
  catch (MooseException & e)
  {
    _console << "\nA MooseException was raised during Auxiliary variable computation.\n"
             << "The next solve will fail, the timestep will be reduced, and we will try again.\n"
             << std::endl;

    // We know the next solve is going to fail, so there's no point in
    // computing anything else after this.  Plus, using incompletely
    // computed AuxVariables in subsequent calculations could lead to
    // other errors or unhandled exceptions being thrown.
    return;
  }

  computeUserObjects(EXEC_LINEAR, Moose::POST_AUX);

  executeControls(EXEC_LINEAR);

  _current_execute_on_flag = EXEC_NONE;

  _app.getOutputWarehouse().residualSetup();

  _safe_access_tagged_vectors = false;

  _current_nl_sys->computeResidualTags(tags); // NonlinearSystemBase::computeResidualTags(const std::set<TagID> & tags)

  _safe_access_tagged_vectors = true;
}
```


NonlinearSystemBase::computeResidualTags(const std::set<TagID> & tags)
```cpp
void
NonlinearSystemBase::computeResidualTags(const std::set<TagID> & tags)
{
  TIME_SECTION("nl::computeResidualTags", 5);

  _fe_problem.setCurrentNonlinearSystem(number());
  _fe_problem.setCurrentlyComputingResidual(true);

  bool required_residual = tags.find(residualVectorTag()) == tags.end() ? false : true;

  _n_residual_evaluations++;

  // not suppose to do anythin on matrix
  deactiveAllMatrixTags();

  FloatingPointExceptionGuard fpe_guard(_app);

  for (const auto & numeric_vec : _vecs_to_zero_for_residual)
    if (hasVector(numeric_vec))
    {
      NumericVector<Number> & vec = getVector(numeric_vec);
      vec.close();
      vec.zero();
    }

  try
  {
    zeroTaggedVectors(tags);
    computeResidualInternal(tags); // NonlinearSystemBase::computeResidualInternal(const std::set<TagID> & tags)
    closeTaggedVectors(tags);

    if (required_residual)
    {
      auto & residual = getVector(residualVectorTag());
      if (_time_integrator)
        _time_integrator->postResidual(residual);
      else
        residual += *_Re_non_time;
      residual.close();
    }
    if (_fe_problem.computingScalingResidual())
      // We don't want to do nodal bcs or anything else
      return;

    computeNodalBCs(tags);
    closeTaggedVectors(tags);

    // If we are debugging residuals we need one more assignment to have the ghosted copy up to
    // date
    if (_need_residual_ghosted && _debugging_residuals && required_residual)
    {
      auto & residual = getVector(residualVectorTag());

      *_residual_ghosted = residual;
      _residual_ghosted->close();
    }
    // Need to close and update the aux system in case residuals were saved to it.
    if (_has_nodalbc_save_in)
      _fe_problem.getAuxiliarySystem().solution().close();
    if (hasSaveIn())
      _fe_problem.getAuxiliarySystem().update();
  }
  catch (MooseException & e)
  {
    // The buck stops here, we have already handled the exception by
    // calling stopSolve(), it is now up to PETSc to return a
    // "diverged" reason during the next solve.
  }

  // not supposed to do anything on matrix
  activeAllMatrixTags();

  _fe_problem.setCurrentlyComputingResidual(false);
}
```


NonlinearSystemBase::computeResidualInternal(const std::set<TagID> & tags)
```cpp
void
NonlinearSystemBase::computeResidualInternal(const std::set<TagID> & tags)
{
  TIME_SECTION("computeResidualInternal", 3);

  residualSetup();

  // Residual contributions from UOs - for now this is used for ray tracing
  // and ray kernels that contribute to the residual (think line sources)
  std::vector<UserObject *> uos;
  _fe_problem.theWarehouse()
      .query()
      .condition<AttribSystem>("UserObject")
      .condition<AttribExecOns>(EXEC_PRE_KERNELS)
      .queryInto(uos);
  for (auto & uo : uos)
    uo->residualSetup();
  for (auto & uo : uos)
  {
    uo->initialize();
    uo->execute();
    uo->finalize();
  }

  // reinit scalar variables
  for (unsigned int tid = 0; tid < libMesh::n_threads(); tid++)
    _fe_problem.reinitScalars(tid);

  // residual contributions from the domain
  PARALLEL_TRY
  {
    TIME_SECTION("Kernels", 3 /*, "Computing Kernels"*/);

    ConstElemRange & elem_range = *_mesh.getActiveLocalElementRange();

    ComputeResidualThread cr(_fe_problem, tags);
    Threads::parallel_reduce(elem_range, cr);

    if (_fe_problem.haveFV())
    {
      using FVRange = StoredRange<std::vector<const FaceInfo *>::const_iterator, const FaceInfo *>;
      ComputeFVFluxResidualThread<FVRange> fvr(_fe_problem, tags);
      FVRange faces(_fe_problem.mesh().faceInfo().begin(), _fe_problem.mesh().faceInfo().end());
      Threads::parallel_reduce(faces, fvr);
    }

    unsigned int n_threads = libMesh::n_threads();
    for (unsigned int i = 0; i < n_threads;
         i++) // Add any cached residuals that might be hanging around
      _fe_problem.addCachedResidual(i);
  }
  PARALLEL_CATCH;

  // residual contributions from the scalar kernels
  PARALLEL_TRY
  {
    // do scalar kernels (not sure how to thread this)
    if (_scalar_kernels.hasActiveObjects())
    {
      TIME_SECTION("ScalarKernels", 3 /*, "Computing ScalarKernels"*/);

      MooseObjectWarehouse<ScalarKernelBase> * scalar_kernel_warehouse;
      // This code should be refactored once we can do tags for scalar
      // kernels
      // Should redo this based on Warehouse
      if (!tags.size() || tags.size() == _fe_problem.numVectorTags(Moose::VECTOR_TAG_RESIDUAL))
        scalar_kernel_warehouse = &_scalar_kernels;
      else if (tags.size() == 1)
        scalar_kernel_warehouse =
            &(_scalar_kernels.getVectorTagObjectWarehouse(*(tags.begin()), 0));
      else
        // scalar_kernels is not threading
        scalar_kernel_warehouse = &(_scalar_kernels.getVectorTagsObjectWarehouse(tags, 0));

      bool have_scalar_contributions = false;
      const auto & scalars = scalar_kernel_warehouse->getActiveObjects();
      for (const auto & scalar_kernel : scalars)
      {
        scalar_kernel->reinit();
        const std::vector<dof_id_type> & dof_indices = scalar_kernel->variable().dofIndices();
        const DofMap & dof_map = scalar_kernel->variable().dofMap();
        const dof_id_type first_dof = dof_map.first_dof();
        const dof_id_type end_dof = dof_map.end_dof();
        for (dof_id_type dof : dof_indices)
        {
          if (dof >= first_dof && dof < end_dof)
          {
            scalar_kernel->computeResidual();
            have_scalar_contributions = true;
            break;
          }
        }
      }
      if (have_scalar_contributions)
        _fe_problem.addResidualScalar();
    }
  }
  PARALLEL_CATCH;

  // residual contributions from Block NodalKernels
  PARALLEL_TRY
  {
    if (_nodal_kernels.hasActiveBlockObjects())
    {
      TIME_SECTION("NodalKernels", 3 /*, "Computing NodalKernels"*/);

      ComputeNodalKernelsThread cnk(_fe_problem, _nodal_kernels, tags);

      ConstNodeRange & range = *_mesh.getLocalNodeRange();

      if (range.begin() != range.end())
      {
        _fe_problem.reinitNode(*range.begin(), 0);

        Threads::parallel_reduce(range, cnk);

        unsigned int n_threads = libMesh::n_threads();
        for (unsigned int i = 0; i < n_threads;
             i++) // Add any cached residuals that might be hanging around
          _fe_problem.addCachedResidual(i);
      }
    }
  }
  PARALLEL_CATCH;

  if (_fe_problem.computingScalingResidual())
    // We computed the volumetric objects. We can return now before we get into
    // any strongly enforced constraint conditions or penalty-type objects
    // (DGKernels, IntegratedBCs, InterfaceKernels, Constraints)
    return;

  // residual contributions from boundary NodalKernels
  PARALLEL_TRY
  {
    if (_nodal_kernels.hasActiveBoundaryObjects())
    {
      TIME_SECTION("NodalKernelBCs", 3 /*, "Computing NodalKernelBCs"*/);

      ComputeNodalKernelBcsThread cnk(_fe_problem, _nodal_kernels, tags);

      ConstBndNodeRange & bnd_node_range = *_mesh.getBoundaryNodeRange();

      Threads::parallel_reduce(bnd_node_range, cnk); // void parallel_reduce (const Range & range, Body & body)

      unsigned int n_threads = libMesh::n_threads();
      for (unsigned int i = 0; i < n_threads;
           i++) // Add any cached residuals that might be hanging around
        _fe_problem.addCachedResidual(i);
    }
  }
  PARALLEL_CATCH;

  mortarConstraints(Moose::ComputeType::Residual);

  if (_need_residual_copy)
  {
    _Re_non_time->close();
    _Re_non_time->localize(_residual_copy);
  }

  if (_need_residual_ghosted)
  {
    _Re_non_time->close();
    *_residual_ghosted = *_Re_non_time;
    _residual_ghosted->close();
  }

  PARALLEL_TRY { computeDiracContributions(tags, false); }
  PARALLEL_CATCH;

  if (_fe_problem._has_constraints)
  {
    PARALLEL_TRY { enforceNodalConstraintsResidual(*_Re_non_time); }
    PARALLEL_CATCH;
    _Re_non_time->close();
  }

  // Add in Residual contributions from other Constraints
  if (_fe_problem._has_constraints)
  {
    PARALLEL_TRY
    {
      // Undisplaced Constraints
      constraintResiduals(*_Re_non_time, false);

      // Displaced Constraints
      if (_fe_problem.getDisplacedProblem())
        constraintResiduals(*_Re_non_time, true);

      if (_fe_problem.computingNonlinearResid())
        _constraints.residualEnd();
    }
    PARALLEL_CATCH;
    _Re_non_time->close();
  }
}
```

判断是否并行
void parallel_reduce (const Range & range, Body & body)
```cpp
/**
 * Execute the provided reduction operation in parallel on the specified
 * range.
 */
template <typename Range, typename Body>
inline
void parallel_reduce (const Range & range, Body & body)
{
  Threads::BoolAcquire b(Threads::in_threads);

  // If we're running in serial - just run!
  if (libMesh::n_threads() == 1)
  {
    body(range); // ThreadedElementLoopBase<RangeType>::operator()，遍历所有单元的残差。
    return;
  }

#ifdef LIBMESH_ENABLE_PERFORMANCE_LOGGING
  const bool logging_was_enabled = libMesh::perflog.logging_enabled();

  if (libMesh::n_threads() > 1)
    libMesh::perflog.disable_logging();
#endif

  unsigned int n_threads = num_pthreads(range);

  std::vector<Range *> ranges(n_threads);
  std::vector<Body *> bodies(n_threads);
  std::vector<RangeBody<Range, Body>> range_bodies(n_threads);

  // Create copies of the body for each thread
  bodies[0] = &body; // Use the original body for the first one
  for (unsigned int i=1; i<n_threads; i++)
    bodies[i] = new Body(body, Threads::split());

  // Create the ranges for each thread
  std::size_t range_size = range.size() / n_threads;

  typename Range::const_iterator current_beginning = range.begin();

  for (unsigned int i=0; i<n_threads; i++)
    {
      std::size_t this_range_size = range_size;

      if (i+1 == n_threads)
        this_range_size += range.size() % n_threads; // Give the last one the remaining work to do

      ranges[i] = new Range(range, current_beginning, current_beginning + this_range_size);

      current_beginning = current_beginning + this_range_size;
    }

  // Create the RangeBody arguments
  for (unsigned int i=0; i<n_threads; i++)
    {
      range_bodies[i].range = ranges[i];
      range_bodies[i].body = bodies[i];
    }

  // Create the threads
  std::vector<pthread_t> threads(n_threads);

  // It may seem redundant to wrap a pragma in #ifdefs... but GCC
  // warns about an "unknown pragma" if it encounters this line of
  // code when -fopenmp is not passed to the compiler.
#ifdef LIBMESH_HAVE_OPENMP
#pragma omp parallel for schedule (static)
#endif
  // The use of 'int' instead of unsigned for the iteration variable
  // is deliberate here.  This is an OpenMP loop, and some older
  // compilers warn when you don't use int for the loop index.  The
  // reason has to do with signed vs. unsigned integer overflow
  // behavior and optimization.
  // http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html
  for (int i=0; i<static_cast<int>(n_threads); i++)
    {
#if !LIBMESH_HAVE_OPENMP
      pthread_create(&threads[i], nullptr, &run_body<Range, Body>, (void *)&range_bodies[i]);
#else
      run_body<Range, Body>((void *)&range_bodies[i]);
#endif
    }

#if !LIBMESH_HAVE_OPENMP
  // Wait for them to finish
  for (unsigned int i=0; i<n_threads; i++)
    pthread_join(threads[i], nullptr);
#endif

  // Join them all down to the original Body
  for (unsigned int i=n_threads-1; i != 0; i--)
    bodies[i-1]->join(*bodies[i]);

  // Clean up
  for (unsigned int i=1; i<n_threads; i++)
    delete bodies[i];
  for (unsigned int i=0; i<n_threads; i++)
    delete ranges[i];

#ifdef LIBMESH_ENABLE_PERFORMANCE_LOGGING
  if (libMesh::n_threads() > 1 && logging_was_enabled)
    libMesh::perflog.enable_logging();
#endif
}
```

ThreadedElementLoopBase<RangeType>::operator()(const RangeType & range, bool bypass_threading)
```cpp
template <typename RangeType>
void
ThreadedElementLoopBase<RangeType>::operator()(const RangeType & range, bool bypass_threading)
{
  try
  {
    try
    {
      ParallelUniqueId puid;
      _tid = bypass_threading ? 0 : puid.id;

      pre();

      _subdomain = Moose::INVALID_BLOCK_ID;
      _neighbor_subdomain = Moose::INVALID_BLOCK_ID;
      typename RangeType::const_iterator el = range.begin();
      for (el = range.begin(); el != range.end(); ++el)
      {
        if (!keepGoing())
          break;

        const Elem * elem = *el;

        preElement(elem);

        _old_subdomain = _subdomain;
        _subdomain = elem->subdomain_id();
        if (_subdomain != _old_subdomain)
          subdomainChanged();

        onElement(elem); // NonlinearThread::onElement(const Elem * elem)

        if (elem->subdomain_id() == Moose::INTERNAL_SIDE_LOWERD_ID ||
            elem->subdomain_id() == Moose::BOUNDARY_SIDE_LOWERD_ID)
        {
          postElement(elem);
          continue;
        }

        for (unsigned int side = 0; side < elem->n_sides(); side++)
        {
          std::vector<BoundaryID> boundary_ids = _mesh.getBoundaryIDs(elem, side);
          const Elem * lower_d_elem = _mesh.getLowerDElem(elem, side);

          if (boundary_ids.size() > 0)
            for (std::vector<BoundaryID>::iterator it = boundary_ids.begin();
                 it != boundary_ids.end();
                 ++it)
            {
              preBoundary(elem, side, *it, lower_d_elem);
              onBoundary(elem, side, *it, lower_d_elem);
            }

          const Elem * neighbor = elem->neighbor_ptr(side);
          if (neighbor)
          {
            preInternalSide(elem, side);

            _old_neighbor_subdomain = _neighbor_subdomain;
            _neighbor_subdomain = neighbor->subdomain_id();
            if (_neighbor_subdomain != _old_neighbor_subdomain)
              neighborSubdomainChanged();

            if (shouldComputeInternalSide(*elem, *neighbor))
              onInternalSide(elem, side);

            if (boundary_ids.size() > 0)
              for (std::vector<BoundaryID>::iterator it = boundary_ids.begin();
                   it != boundary_ids.end();
                   ++it)
                onInterface(elem, side, *it);

            postInternalSide(elem, side);
          }
        } // sides

        postElement(elem);
      } // range

      post();
    }
    catch (libMesh::LogicError & e)
    {
      throw MooseException("We caught a libMesh error in ThreadedElementLoopBase");
    }
    catch (MetaPhysicL::LogicError & e)
    {
      moose::translateMetaPhysicLError(e);
    }
  }
  catch (MooseException & e)
  {
    caughtMooseException(e);
  }
}
```


NonlinearThread::onElement(const Elem * elem)
```cpp
void
NonlinearThread::onElement(const Elem * elem)
{
  _fe_problem.prepare(elem, _tid);
  _fe_problem.reinitElem(elem, _tid);

  // Set up Sentinel class so that, even if reinitMaterials() throws, we
  // still remember to swap back during stack unwinding.
  SwapBackSentinel sentinel(_fe_problem, &FEProblem::swapBackMaterials, _tid);

  _fe_problem.reinitMaterials(_subdomain, _tid);

  if (_tag_kernels->hasActiveBlockObjects(_subdomain, _tid))
  {
    const auto & kernels = _tag_kernels->getActiveBlockObjects(_subdomain, _tid);
    for (const auto & kernel : kernels)
      compute(*kernel); // ComputeResidualThread::compute(ResidualObject & ro)
  }

  for (auto kernel : _fv_kernels)
    compute(*kernel);
}

```


ComputeResidualThread::compute(ResidualObject & ro)
```cpp
void
ComputeResidualThread::compute(ResidualObject & ro)
{
  ro.computeResidual(); // Kernel::computeResidual()，ro即Residual Object
}
```


Kernel::computeResidual()
```cpp
void
Kernel::computeResidual()
{
  prepareVectorTag(_assembly, _var.number());

  precalculateResidual();
  for (_i = 0; _i < _test.size(); _i++)
    for (_qp = 0; _qp < _qrule->n_points(); _qp++)
      _local_re(_i) += _JxW[_qp] * _coord[_qp] * computeQpResidual();
      // 遍历所有的变量试函数i和高斯正交点qp。
      // 调用Diffusion::computeQpResidual()
      //  Jacobian weight (_JxW[_qp])
      // coordinate (_coord[_qp])——平方的三倍=1？
      // gdb可以用p *_qrule看看, active quadrature rule, const QBase* const& KernelBase::_qrule


  accumulateTaggedLocalResidual();

  if (_has_save_in)
  {
    Threads::spin_mutex::scoped_lock lock(Threads::spin_mtx);
    for (const auto & var : _save_in)
      var->sys().solution().add_vector(_local_re, var->dofIndices());
  }
}
```


Diffusion::computeQpResidual()
```cpp

Real
Diffusion::computeQpResidual()
{
  return _grad_u[_qp] * _grad_test[_i][_qp];
}

```


# 其他
### kernel调用物性
====================  材料物性 →  系统残差  ====================


ADFluidPropertiesMaterialPh::computeQpProperties (this=0x555555b4a910)
    at /home/lee/projects/moose/modules/fluid_properties/src/materials/ADFluidPropertiesMaterialPh.C:76

Material::computeProperties (this=0x555555b4a910)
at /home/lee/projects/moose/framework/src/materials/Material.C:131
131	    for (_qp = 0; _qp < _qrule->n_points(); ++_qp)

MaterialData::reinit<std::vector<std::shared_ptr<MaterialBase>, std::allocator<std::shared_ptr<MaterialBase> > > > (this=0x555555a9f720,
    mats=std::vector of length 1, capacity 1 = {...})
at /home/lee/projects/moose/framework/build/header_symlinks/MaterialData.h:457
457	  for (const auto & mat : mats)

FEProblemBase::reinitMaterials (this=0x555555a84660, blk_id=0, tid=0, swap_stateful=true) at /home/lee/projects/moose/framework/src/problems/FEProblemBase.C:3322
3322	}

NonlinearThread::onElement (this=0x7fffffffbb10, elem=0x555555a4fff0) at /home/lee/projects/moose/framework/src/loops/NonlinearThread.C:109
109	  if (_tag_kernels->hasActiveBlockObjects(_subdomain, _tid))


====================  材料物性 →  系统残差  ====================


### auxkernel调用物性，densityaux

====================  材料物性 →  系统残差  ====================

/home/lee/projects/moose/modules/fluid_properties/src/materials/ADFluidPropertiesMaterialPh.C:61
 ADFluidPropertiesMaterialPh::computeQpProperties()

/home/lee/projects/moose/framework/src/materials/Material.C:131
Material::subdomainSetup()
	      computeQpProperties();

/home/lee/projects/moose/framework/build/header_symlinks/MaterialData.h:457
458	    mat->computeProperties();

/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:3322
FEProblemBase::reinitMaterials(SubdomainID blk_id, THREAD_ID tid, bool swap_stateful)


/home/lee/projects/moose/framework/src/loops/ComputeElemAuxVarsThread.C:114
ComputeElemAuxVarsThread<AuxKernelType>::onElement(const Elem * elem)


/home/lee/projects/moose/framework/src/systems/AuxiliarySystem.C:842
AuxiliarySystem::computeElementalVarsHelper
842	      ComputeElemAuxVarsThread<AuxKernelType> eavt(_fe_problem, warehouse, vars, true);

/home/lee/projects/moose/framework/src/systems/AuxiliarySystem.C:768
AuxiliarySystem::computeElementalVars(ExecFlagType type)
768	  TIME_SECTION("computeElementalVars", 3);

/home/lee/projects/moose/framework/src/systems/AuxiliarySystem.C:445
AuxiliarySystem::compute(ExecFlagType type)
445	    computeElementalVars(type);

/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:5834
FEProblemBase::computeResidualTags(const std::set<TagID> & tags)
5834	    _aux->compute(EXEC_LINEAR);

/home/lee/projects/moose/framework/src/loops/NonlinearThread.C:109
NonlinearThread::onElement(const Elem * elem)
109	  if (_tag_kernels->hasActiveBlockObjects(_subdomain, _tid))


/home/lee/projects/moose/framework/src/systems/NonlinearSystemBase.C:1522
NonlinearSystemBase::computeResidualInternal(const std::set<TagID> & tags)
1522	    if (_fe_problem.haveFV())


/home/lee/projects/moose/framework/src/systems/NonlinearSystemBase.C:744
NonlinearSystemBase::computeResidualTags(const std::set<TagID> & tags)
744	    closeTaggedVectors(tags);

/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:5861
FEProblemBase::computeResidualTags
5861	  _safe_access_tagged_vectors = true;


/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:5699
FEProblemBase::computeResidualInternal
5699	    _nl->disassociateVectorFromTag(residual, _nl->residualVectorTag());


/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:5487
FEProblemBase::computeResidual

/home/lee/projects/moose/framework/src/problems/FEProblemBase.C:5463
5463	  ADReal::do_derivatives = true;
FEProblemBase::computeResidualSys

/home/lee/projects/moose/framework/src/systems/NonlinearSystem.C:146
146	    _computing_initial_residual = false;
NonlinearSystem::solve

FEProblemSolve::solve (this=0x555555be3548) at /home/lee/projects/moose/framework/src/executioners/FEProblemSolve.C:262
262	    if (_problem.shouldSolve())

FixedPointSolve::solveStep (this=0x5555555e43d0, begin_norm=@0x555556131f20: 0, end_norm=@0x555555edd2f0: 0, transformed_dofs=std::set with 0 elements)
    at /home/lee/projects/moose/framework/src/executioners/FixedPointSolve.C:393
393	  if (!_inner_solve->solve())

FixedPointSolve::solve (this=0x5555555e43d0) at /home/lee/projects/moose/framework/src/executioners/FixedPointSolve.C:274
274	    bool solve_converged = solveStep(_fixed_point_timestep_begin_norm[_fixed_point_it],

Steady::execute (this=0x555555be3250) at /home/lee/projects/moose/framework/src/executioners/Steady.C:85
85	    _last_solve_converged = _fixed_point_solve->solve();

MooseApp::executeExecutioner (this=0x555555795930) at /home/lee/projects/moose/framework/src/base/MooseApp.C:1101
1101	    _executioner->execute();

MooseApp::run (this=0x555555795930) at /home/lee/projects/moose/framework/src/base/MooseApp.C:1414
1414	    TIME_SECTION("execute", 2, "Executing");

main (argc=6, argv=0x7fffffffd8b8) at /home/lee/projects/momentum/src/main.C:35
35	  return 0;
