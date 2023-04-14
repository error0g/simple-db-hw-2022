# 6.5830/6.5831 实验室 1:SimpleDB

**分配：**2022 年 9 月 14 日星期三

**到期日期：**美国东部时间 2022 年 9 月 28 日星期三晚上 11:59

<!--
**Bug Update:** We have a [page](bugs.html) to keep track
of SimpleDB bugs that you or we find. Fixes for bugs/annoyances will also be
posted there. Some bugs may have already been found, so do take a look at the
page to get the latest version/ patches for the lab code.
-->

在 6.5830/6.5831 的实验室作业中，你将编写一个名为 SimpleDB 的基本数据库管理系统。对于这个实验室，你将专注于实现访问磁盘上存储的数据所需的核心模块；在未来的实验室中，你将添加对各种查询处理运算符以及事务、锁定和并发查询的支持。

SimpleDB 是用 Java 编写的。我们已经为你提供了一组大部分未实现的类和接口。你需要为这些类编写代码。我们将通过运行一组使用[JUnit](http://junit.sourceforge.net/)编写的系统测试来对你的代码进行评分。我们还提供了一些单元测试，我们不会使用这些测试进行评分，但你可能会发现这些测试有助于验证你的代码是否有效。除了我们的测试之外，我们还鼓励你开发自己的测试套件。

本文档的其余部分描述了 SimpleDB 的基本架构，给出了一些关于如何开始编码的建议，并讨论了如何在实验室中进行操作。

我们建议你尽早开始这个实验室。这需要你编写大量的代码！

<!--

##  0.  Find bugs, be patient, earn candy bars 

SimpleDB is a relatively complex piece of code. It is very possible you are
going to find bugs, inconsistencies, and bad, outdated, or incorrect
documentation, etc.

We ask you, therefore, to do this lab with an adventurous mindset.  Don't get
mad if something is not clear, or even wrong; rather, try to figure it out
yourself or send us a friendly email.  We promise to help out by posting bug
fixes, new commits to the HW repo, etc., as bugs and issues are reported.

<p>...and if you find a bug in our code, we`ll give you a candy bar (see
[Section 3.3](#bugs))!

-->
<!--你可以找到[here](bugs.html)</p>-->

## 0. 环境设置

**按照说明从课程 GitHub 存储库下载实验室 1 的代码开始**

------

**更新（2022年9月26日）：**如果你在 9 月 26 日或之后启动此实验室，你将从此存储库中获得的代码将包括启动代码和以后实验室的测试。若要仅获取用于实验室 1 的代码和测试的子集，请切换到 `lab1` 分支（ `git checkout lab1`）。

------

这些说明是为 Athena 或任何其他基于 Unix 的平台（如 Linux、macOS 等）编写的。因为代码是用 Java 编写的，所以它也应该在 Windows 下工作，尽管本文档中的说明可能不适用。

除非你在 Athena 上运行，否则你还需要确保安装了 Java 开发工具包（JDK）。SimpleDB 至少需要 Java 8（请注意，“Java 8”和“Java 1.8”指的是 Java 的同一版本，前者用于品牌宣传）。默认情况下，此项目已配置为使用 Java 11. 如果你运行的是旧版本的 Java，则需要更新 `build.xml` 中的以下行：


```xml
<!-- Update the value below to match your version (e.g., "1.8"). -->
<property name="sourceversion" value="11"/>
```

**我们的autorader使用Java 11，所以请不要使用任何比Java 11更新的功能（实验室不需要它们）。**

你可以按如下方式安装 JDK：
- 在 macOS 上： `brew install openjdk@11`
- 在 Ubuntu（或 WSL）上： `sudo apt install openjdk-11-jdk`
- 在 Windows 上：请参阅此[guide](https://docs.microsoft.com/en-us/java/openjdk/install)。

我们还包含了关于将项目与 IntelliJ、VSCode 或 Eclipse 一起使用的[第1.2节](#ides)。

## 1. 开始

SimpleDB 使用[Ant构建工具](http://ant.apache.org/)编译代码并运行测试。Ant 类似于[make](http://www.gnu.org/software/make/manual/)，但构建文件是用 XML 编写的，在某种程度上更适合 Java 代码。大多数现代 Linux 发行版都包括 Ant。在 Athena 下，它包含在 `sipb` 储物柜中，你可以通过在 Athena 提示符下键入 `add sipb` 来访问该储物柜。请注意，在某些版本的 Athena 上，你还必须运行 `add -f java` 才能为 Java 程序正确设置环境。有关更多详细信息，请参阅[关于使用Java的Athena文档](http://web.mit.edu/java/www/)。

为了在开发过程中帮助你，除了用于评分的端到端测试外，我们还提供了一组单元测试。这些绝不是全面的，你不应该仅仅依靠它们来验证你的项目的正确性（使用那些 6.1040（以前的 6.170）的技能！）。

要运行单元测试，请使用 `test` 构建目标：


```
$ cd [project-directory]
$ # run all unit tests
$ ant test
$ # run a specific unit test
$ ant runtest -Dtest=TupleTest
```

你应该看到类似于以下内容的输出：


```
 build output...

test:
    [junit] Running simpledb.CatalogTest
    [junit] Testsuite: simpledb.CatalogTest
    [junit] Tests run: 2, Failures: 0, Errors: 2, Time elapsed: 0.037 sec
    [junit] Tests run: 2, Failures: 0, Errors: 2, Time elapsed: 0.037 sec

 ... stack traces and error reports ...
```

上面的输出表明在编译过程中出现了两个错误；这是因为我们给你的代码还不能工作。当你完成实验室的部分工作时，你将努力通过额外的单元测试。

如果你希望在编写代码时编写新的单元测试，则应将它们添加到 `test/simpledb` 目录中。

有关如何使用 Ant 的更多详细信息，请参阅[manual](http://ant.apache.org/manual/)。[正在运行的Ant](http://ant.apache.org/manual/running.html)部分提供了有关使用命令的详细信息。然而，下面的快速参考表应该足以用于实验室工作。

命令 | 描述
--- | ---
ant | 构建默认目标（对于 simpledb，这是 dist）。
ant-projecthelp| 列出 `build.xml` 中的所有目标并进行说明。
ant dist | 在 src 中编译代码，并将其打包到 `dist/simpledb.jar` 中。
ant test | 编译并运行所有单元测试。
ant runtest-Dtest=testname | 运行名为 `testname` 的单元测试。
ant systemtest | 编译并运行所有系统测试。
ant runsystest-Dtest=testname| 编译并运行名为 `testname` 的系统测试。

如果你在 windows 系统下，不想从命令行运行 ant 测试，也可以从 eclipse 运行它们。右键单击 build.xml，在 targets 选项卡中，可以看到“runtest”、“runsystest”等。例如，从命令行中选择 runtest 相当于“ant runtest”。可以在“Main”选项卡的“Arguments”文本框中指定“-Dtest=testname”等参数。请注意，你还可以通过从 build.xml 复制、修改目标和参数并将其重命名为 runtest_build.xml 来创建 runtest 的快捷方式。

### 1.1. 运行端到端测试

我们还提供了一套端到端测试，最终将用于评分。这些测试被构造为 JUnit 测试，它们位于 `test/simpledb/systemtest` 目录中。要运行所有系统测试，请使用 `systemtest` 构建目标：


```
$ ant systemtest

 ... build output ...

    [junit] Testcase: testSmall took 0.017 sec
    [junit]     Caused an ERROR
    [junit] expected to find the following tuples:
    [junit]     19128
    [junit] 
    [junit] java.lang.AssertionError: expected to find the following tuples:
    [junit]     19128
    [junit] 
    [junit]     at simpledb.systemtest.SystemTestUtil.matchTuples(SystemTestUtil.java:122)
    [junit]     at simpledb.systemtest.SystemTestUtil.matchTuples(SystemTestUtil.java:83)
    [junit]     at simpledb.systemtest.SystemTestUtil.matchTuples(SystemTestUtil.java:75)
    [junit]     at simpledb.systemtest.ScanTest.validateScan(ScanTest.java:30)
    [junit]     at simpledb.systemtest.ScanTest.testSmall(ScanTest.java:40)

 ... more error messages ...
```

这表示此测试失败，显示检测到错误的堆栈跟踪。要进行调试，请先读取发生错误的源代码。测试通过后，你将看到以下内容：


```
$ ant systemtest

 ... build output ...

    [junit] Testsuite: simpledb.systemtest.ScanTest
    [junit] Tests run: 3, Failures: 0, Errors: 0, Time elapsed: 7.278 sec
    [junit] Tests run: 3, Failures: 0, Errors: 0, Time elapsed: 7.278 sec
    [junit] 
    [junit] Testcase: testSmall took 0.937 sec
    [junit] Testcase: testLarge took 5.276 sec
    [junit] Testcase: testRandom took 1.049 sec

BUILD SUCCESSFUL
Total time: 52 seconds
```

#### 1.1.1 创建伪表

你可能想要创建自己的测试和数据表来测试自己的 SimpleDB 实现。你可以创建任何 `.txt` 文件，并使用以下命令将其转换为 SimpleDB 的 `HeapFile` 格式的 `.dat` 文件：


```
$ java -jar dist/simpledb.jar convert file.txt N
```

其中， `file.txt` 是文件名， `N` 是该文件中的列数。请注意， `file.txt` 必须采用以下格式：


```
int1,int2,...,intN
int1,int2,...,intN
int1,int2,...,intN
int1,int2,...,intN
```

…其中每个 `intN` 都是非负整数。

要查看表格的内容，请使用 `print` 命令：


```
$ java -jar dist/simpledb.jar print file.dat N
```

其中， `file.dat` 是使用 `convert` 命令创建的表的名称，是文件中的列数。

<a name="ides"></a>

### 1.2. 使用 IDE

IDE（集成开发环境）是图形化的软件开发环境，可以帮助你管理更大的项目。我们建议使用[IntelliJ](https://www.jetbrains.com/idea/)或[VSCode](https://code.visualstudio.com/)。如果你喜欢冒险，你也可以尝试[Eclipse](https://www.eclipse.org/ide/)。

我们提供了有关设置[IntelliJ](https://www.jetbrains.com/idea/)、[VSCode](https://code.visualstudio.com)和的说明。对于 IntelliJ，我们使用的是终极版，你可以通过你的 mit.edu 帐户获得教育许可证[here](https://www.jetbrains.com/community/education/#students)。我们为 Eclipse 提供的指令是通过使用 Java 1.7 的 Eclipse for Java Developers（而不是企业版）生成的。我们强烈鼓励你为这个项目设置并学习其中一个 IDE。

#### 1.2.1 设置 IntelliJ

IntelliJ 是一个现代的 Java IDE，它很受欢迎，而且一些帐户使用起来更直观。要使用 IntelliJ，请首先安装它并打开应用程序。在“项目”下，选择“打开”并导航到项目根目录。双击 `.project` 文件（你可能需要将操作系统配置为显示隐藏文件才能查看），然后单击“以项目形式打开”。IntelliJ 支持 Ant 的工具窗口，你可能希望根据[here](https://www.jetbrains.com/help/idea/ant.html)的说明进行设置，但这对开发来说并不重要。你可以找到 IntelliJ 功能的详细演练[here](https://www.jetbrains.com/help/idea/discover-intellij-idea.html)。


#### 1.2.2.设置 VSCode

VSCode 是一个流行的免费可扩展代码编辑器，可与包括 Java 在内的多种语言配合使用。你可以找到安装说明[here](https://code.visualstudio.com/docs/setup/setup-overview)。

一旦安装了 VSCode，如果你想访问 Java 语言功能（例如调试器、自动完成等），还需要安装[Java扩展包](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)。要执行此操作，请首先单击左侧边栏上的“块”图标（Windows/Linux 上的 Ctrl-Shift-X 或 macOS 上的 \-Shift-X）。然后，搜索“java 扩展包”，当你看到扩展包时点击 install（这应该是第一个结果）。

最后，要使用 SimpleDB，请单击“文件”>“打开文件夹”，然后导航到克隆此存储库的文件夹。项目应该加载，并且你应该能够看到代码。当你第一次打开它时，VSCode 可能会问你是否[信任项目](https://code.visualstudio.com/docs/editor/workspace-trust)。你需要单击“是”才能访问该项目的所有 VSCode 功能。

#### 1.2.3.设置 Eclipse

**准备代码库**

运行以下命令为 IDE 生成项目文件：


```
ant eclipse
```

**在Eclipse中设置实验室**

* 安装 Eclipse 后，启动它，并注意第一个屏幕要求你为工作区选择一个位置（我们将此目录称为 $W）。选择包含你的简单 db-hw 存储库的目录。
* 在 Eclipse 中，选择 File->New->Project->Java->JavaProject，然后按下 Next。
* 输入“simple-db-hw”作为项目名称。
* 在输入项目名称的同一屏幕上，选择“从现有源创建项目”，然后浏览到 $W/simple-db-hw。
* 单击 finish，你应该能够在屏幕左侧的 project Explorer 选项卡中看到“simple db hw”作为一个新项目。打开这个项目可以看到上面讨论的目录结构——实现代码可以在“src”中找到，单元测试和系统测试可以在“test”中找到

**注：**此类假设你使用的是 Oracle 官方版本的 Java。这是 macOS 上的默认设置，对于大多数 Windows Eclipse 安装来说也是如此；但许多 Linux 发行版默认为备用 Java 运行时（如 OpenJDK）。请从[Oracle网站](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载最新的 Java 8 更新，并使用该 Java 版本。如果你不切换，你可能会在以后的实验室中的一些性能测试中看到虚假的测试失败。

**运行单个单元和系统测试**

要运行单元测试或系统测试（两者都是 JUnit 测试，可以用相同的方式初始化），请转到屏幕左侧的 PackageExplorer 选项卡。在“simpledb-hw”项目下，打开“test”目录。单元测试位于“simpledb”包中，系统测试位于“simpledb.systemtests”包中。要运行其中一个测试，请选择测试（它们都被称为*测试.java*-不要选择 TestUtil.java 或 SystemTestUtil.jar），右键单击它，选择“运行方式”，然后选择“JUnit 测试”。这将显示一个 JUnit 选项卡，它将告诉你 JUnit 测试套件中各个测试的状态，并向你显示异常和其他错误，这些错误将帮助你调试问题。

**运行Ant构建目标**

如果你想运行诸如“ant test”或“ant systemtest”之类的命令，请在 Package Explorer 中右键单击 build.xml。选择“运行方式”，然后选择“Ant Build…”（注意：选择带有省略号（…）的选项，否则将不会显示一组要运行的构建目标）。然后，在下一个屏幕的“目标”选项卡中，勾选要运行的目标（可能是“dist”和“test”或“systemtest”之一）。这应该会运行构建目标，并在 Eclipse 的控制台窗口中向你显示结果。

### 1.3. 实施提示

在开始编写代码之前，我们**强烈鼓励**让你通读整个文档，以了解 SimpleDB 的高级设计。

你将需要填写任何未实现的代码。很明显，我们认为你应该在哪里编写代码。你可能需要添加私有方法和/或辅助类。你可以更改 API，但请确保我们的[grading](#grading)测试仍在运行，并确保在撰写时提及、解释和捍卫你的决定。

除了你需要为此实验室填写的方法外，类接口还包含许多方法，你需要在后续实验室中才能实现这些方法。这些将按类别显示：


```java
// Not necessary for lab1.
public class Insert implements DbIterator {
}
```

或按方法：


```Java
public boolean deleteTuple(Tuple t)throws DbException{
    // TODO: some code goes here
    // not necessary for lab1
    return false;
}
```

你提交的代码应该在不必修改这些方法的情况下进行编译。

我们建议在本文档中进行练习，以指导你的实施，但你可能会发现不同的顺序对你更有意义。

**以下是您可以继续进行SimpleDB实现的一种方法的大致概述：**

---

* 实现用于管理元组的类，即 `Tuple`、 `TupleDesc`。我们已经为你实现了 `Field`、、 `StringField` 和 `Type`。由于你只需要支持整数和（固定长度）字符串字段以及固定长度元组，因此这些操作非常简单。
* 实现 `Catalog`（这应该非常简单）。
* 实现 `BufferPool` 构造函数和方法。
* 实现访问方法、 `HeapPage` 和以及相关的 ID 类。这些文件中有很大一部分已经为你编写好了。
* 实现运算符 `SeqScan`。
* 此时，你应该能够通过 `ScanTest` 系统测试，这是本实验室的目标。

---

下面的第 2 节将更详细地介绍这些实现步骤以及与每个步骤相对应的单元测试。

### 1.4. 交易、锁定和恢复

当你浏览我们为你提供的接口时，你会看到许多对锁定、事务和恢复的引用。你不需要在这个实验室中支持这些功能，但你应该在代码的接口中保留这些参数，因为你将在未来的实验室中实现事务和锁定。我们为你提供的测试代码会生成一个伪造的事务 ID，并将其传递给它运行的查询的运算符；你应该将此事务 ID 传递给其他操作符和缓冲池。

## 2. SimpleDB 体系结构和实施指南

SimpleDB 包括：

* 表示字段、元组和元组模式的类；
* 将谓词和条件应用于元组的类；
* 一种或多种访问方法（例如，堆文件），将关系存储在磁盘上，并提供迭代这些关系的元组的方法；
* 处理元组的运算符类的集合（例如，select、join、insert、delete 等）；
* 一个缓冲池，用于在内存中缓存活动元组和页面，并处理并发控制和事务（对于这个实验室来说，这两者都不需要担心）；和
* 一个目录，用于存储有关可用表及其模式的信息。

SimpleDB 没有包含许多你可能认为是“数据库”一部分的内容。特别是，SimpleDB 没有：

* （在这个实验室中），一个 SQL 前端或解析器，允许你将查询直接键入 SimpleDB。相反，查询是通过将一组运算符链接到一个手工构建的查询计划中来构建的（请参见[第2.7节](#query_walkthrough)）。我们将提供一个简单的解析器供以后的实验室使用。
* 意见。
* 除整数和固定长度字符串之外的数据类型。
* （在这个实验室里）查询优化器。
* （在这个实验室）指数。

在本节的其余部分中，我们将描述你需要在本实验室中实现的 SimpleDB 的每个主要组件。你应该使用本讨论中的练习来指导你的实现。本文档绝不是 SimpleDB 的完整规范；你需要决定如何设计和实现系统的各个部分。请注意，对于 Lab 1，除了顺序扫描之外，你不需要实现任何运算符（例如，select、join、project）。你将在未来的实验室中添加对其他操作员的支持。

### 2.1. 数据库类

Database 类提供对作为数据库全局状态的静态对象集合的访问。特别是，这包括访问目录（数据库中所有表的列表）、缓冲池（当前驻留在内存中的数据库文件页的集合）和日志文件的方法。你不需要担心这个实验室中的日志文件。我们已经为你实现了数据库类。你应该查看这个文件，因为你需要访问这些对象。

### 2.2. 字段和元组

SimpleDB 中的元组是非常基本的。它们由 `Field` 对象的集合组成， `Tuple` 中的每个字段一个 `Field` 是不同数据类型（例如，整数、字符串）实现的接口 `Tuple` 对象由底层访问方法（例如堆文件或 B 树）创建，如下一节所述。元组也有一个类型（或模式），称为_元组描述符_，由 `TupleDesc` 对象表示。此对象由 `Type` 对象的集合组成，元组中的每个字段一个，每个对象描述相应字段的类型。

### 练习 1

**在以下位置实现骨架方法：**

---
* src/java/simpledb/storage/TupleDesc.java
* src/java/simpledb/storage/Tuple.java
---

在这一点上，你的代码应该通过单元测试 `TupleTest` 和 `TupleDescTest`。此时， `modifyRecordId()` 应该会失败，因为你还没有实现它。

### 2.3. 目录

目录（SimpleDB 中的 class ＜ gt r ＝“118”/>）由表列表和当前数据库中表的模式组成。你需要支持添加新表的功能，以及获取有关特定表的信息。与每个表关联的是一个 `TupleDesc` 对象，该对象允许操作员确定表中字段的类型和数量。

全局目录是为整个 SimpleDB 进程分配的 `Catalog` 的单个实例。全局编录可以通过 `Database.getCatalog()` 方法检索，全局缓冲池也是如此（使用 `Database.getBufferPool()`）。

### 练习 2

**在以下位置实现骨架方法：**

---
* src/java/simpledb/common/Catalog.java
--- 

此时，你的代码应该通过 `CatalogTest` 中的单元测试。

### 2.4. 缓冲池

缓冲池（SimpleDB 中的 class ＜ gt r ＝“125”/>）负责缓存内存中最近从磁盘读取的页面。所有操作员都通过缓冲池从磁盘上的各种文件读取和写入页面。它由固定数量的页面组成，由 `BufferPool` 构造函数的 `numPages` 参数定义。在以后的实验室中，你将实施驱逐政策。对于这个实验室，你只需要实现构造函数和 SeqScan 运算符使用的 `BufferPool.getPage()` 方法。缓冲池最多应存储 `numPages` 页。对于这个实验室，如果对不同的页面发出了超过 `numPages` 的请求，那么你可以抛出 DbException，而不是实现驱逐策略。在未来的实验室中，你将被要求执行驱逐政策。

 `Database` 类提供了一个静态方法， `Database.getBufferPool()`，该方法为整个 SimpleDB 进程返回对单个 BufferPool 实例的引用。

### 练习 3

**在中执行 `getPage()` 方法：

---
* src/java/simpledb/storage/BufferPool.java
---

我们尚未提供 `BufferPool` 的单元测试。你实现的功能将在下面的 `HeapFile` 的实现中进行测试。你应该使用 `DbFile.readPage` 方法访问 `DbFile` 的页面。


<!-- When more than this many pages are in the buffer pool, one page should be evicted from the pool before the next is loaded.  The choice of eviction policy is up to you; it is not necessary to do something sophisticated. -->

<!--
<p>

Notice that `BufferPool` asks you to implement
a `flush_all_pages()` method.  This is not something you would ever
need in a real implementation of a buffer pool.  However, we need this method
for testing purposes.  You really should never call this method from anywhere
in your code.
-->

### 2.5. `HeapFile` 访问方法

访问方法提供了一种从以特定方式排列的磁盘读取或写入数据的方法。常见的访问方法包括堆文件（元组的未排序文件）和 B 树；对于这项任务，你将只实现一个堆文件访问方法，我们已经为你编写了一些代码。

 `HeapFile` 对象被排列成一组页面，每个页面都由固定数量的字节组成，用于存储元组（由常量 `BufferPool.DEFAULT_PAGE_SIZE` 定义），包括一个标头。在 SimpleDB 中，数据库中的每个表都有一个 `HeapFile` 对象。 `HeapFile` 中的每个页面都被安排为一组槽，每个槽可以容纳一个元组（SimpleDB 中给定表的元组大小都相同）。除了这些槽之外，每个页面都有一个标头，该标头由位图组成，每个元组槽有一个位。如果与特定元组相对应的比特是 1，则表明该元组是有效的；如果为 0，则元组无效（例如，已被删除或从未初始化。） `HeapFile` 对象的页面属于实现 `Page` 接口的 `HeapPage` 类型。页面存储在缓冲池中，但由 `HeapFile` 类进行读取和写入。

SimpleDB 将堆文件存储在磁盘上，其格式与存储在内存中的格式大致相同。每个文件由磁盘上连续排列的页面数据组成。每个页面由一个或多个字节组成，表示标题，后面是实际页面内容的_页面大小_字节。每个元组的内容需要 *8 位，标头需要 1 位。因此，可以容纳在单个页面中的元组的数量为：


```
tuples per page = floor((page size * 8) / (tuple size * 8 + 1))
```

其中，_元组大小_是页面中元组的大小（以字节为单位）。这里的想法是，每个元组在头中需要一个额外的存储位。我们计算一个页面中的位数（通过将页面大小乘以 8），并将这个数量除以元组中的位数，得到每页的元组数。floor 操作四舍五入到最接近的整数元组（我们不希望在页面上存储部分元组！）

一旦我们知道每页元组的数量，存储标头所需的字节数就简单地为：


```
header bytes = ceiling(tuples per page / 8)
```

上限操作四舍五入到最接近的整数字节数（我们存储的头信息永远不会少于一个完整字节）

每个字节的低（最低有效）位表示文件中较早的插槽的状态。因此，第一字节的最低比特表示页面中的第一个槽是否正在使用。第一个字节的倒数第二位表示页面中的第二个插槽是否正在使用，依此类推。此外，请注意，最后一个字节的高位可能与文件中实际存在的插槽不对应，因为插槽的数量可能不是 8 的倍数。还要注意，所有 Java 虚拟机都是[big-endian](http://en.wikipedia.org/wiki/Endianness)。

### 练习 4

**在以下位置实现骨架方法：**

---
* src/java/simpledb/storage/HeapPageId.java
* src/java/simpledb/storage/RecordId.java
* src/java/simpledb/storage/HeapPage.java
---

尽管你不会在实验室 1 中直接使用它们，但我们要求你在＜ gt r ＝“157”/＞中实现＜ gt r=“155”/＞和＜ gt r=“156”/＞。这需要在页头中推送位。你可能会发现，查看 `HeapPage` 或中提供的其他方法有助于理解页面布局。

你还需要在页面中的元组上实现 `Iterator`，这可能涉及辅助类或数据结构。

此时，你的代码应该通过 `HeapPageIdTest`、和 `HeapPageReadTest` 中的单元测试。

在实现 `HeapPage` 之后，你将在此实验室中为 `HeapFile` 编写方法，以计算文件中的页数并从文件中读取页面。然后，你将能够从磁盘上存储的文件中提取元组。

### 练习 5

**在以下位置实现骨架方法：**

---
* src/java/simpledb/storage/HeapFile.java
--- 

要从磁盘读取页面，首先需要计算文件中的正确偏移量。提示：你需要对文件进行随机访问，以便以任意偏移量读取和写入页面。从磁盘读取页面时，不应调用 `BufferPool` 实例方法。

你还需要实现 `HeapFile.iterator()` 方法，该方法应遍历 HeapFile 中每个页面的元组。迭代器必须使用 `BufferPool.getPage()` 方法来访问 `HeapFile` 中的页面。此方法将页面加载到缓冲池中，并最终用于（在以后的实验室中）实现基于锁定的并发控制和恢复。不要在 `open()` 调用中将整个表加载到内存中——这将导致非常大的表出现内存不足错误。

此时，你的代码应该通过 `HeapFileReadTest` 中的单元测试。

### 2.6. 操作员

操作员负责查询计划的实际执行。它们实现了关系代数的运算。在 SimpleDB 中，运算符是基于迭代器的；每个运算符都实现 `DbIterator` 接口。

通过将较低级别的运算符传递到较高级别的运算符的构造函数中，即通过“将它们链接在一起”，将运算符连接到一个计划中。计划左侧的特殊访问方法运算符负责从磁盘读取数据（因此其下没有任何运算符）。

在计划的顶部，与 SimpleDB 交互的程序只需调用根运算符上的 `getNext()`；然后，此运算符对其子级调用 `getNext()`，依此类推，直到调用这些叶运算符为止。它们从磁盘中获取元组并将其向上传递到树（作为 `getNext()` 的返回参数）；元组以这种方式在计划中向上传播，直到它们在根处输出，或者被计划中的另一个运算符组合或拒绝。

<!--
For plans that implement `INSERT` and `DELETE` queries,
the top-most operator is a special `Insert` or `Delete`
operator that modifies the pages on disk.  These operators return a tuple
containing the count of the number of affected tuples to the user-level
program.

<p>
-->

对于这个实验室，你只需要实现一个 SimpleDB 操作符。

### 练习 6。

**在以下位置实现骨架方法：**

---
* src/java/simpledb/execution/SeqScan.java
---

此运算符按顺序扫描构造函数中 `tableid` 指定的表页中的所有元组。此运算符应通过 `DbFile.iterator()` 方法访问元组。

此时，你应该能够完成 ScanTest 系统测试。干得好！

你将在随后的实验室中填写其他操作员。

<a name="query_walkthrough"></a>

### 2.7. 一个简单的查询

本节的目的是说明如何将这些不同的组件连接在一起以处理一个简单的查询。

假设你有一个数据文件“some_data_file.txt”，其中包含以下内容：


```
1,1,1
2,2,2 
3,4,4
```

你可以将其转换为 SimpleDB 可以查询的二进制文件，如下所示：


```bash
java -jar dist/simpledb.jar convert some_data_file.txt 3
```

这里，参数“3”告诉 conver 输入有 3 列。

以下代码在此文件上实现了一个简单的选择查询。此代码等效于 SQL 语句 `SELECT * FROM some_data_file`。


```java
package simpledb;
import java.io.*;

public class test {

    public static void main(String[] argv) {

        // construct a 3-column table schema
        Type types[] = new Type[]{ Type.INT_TYPE, Type.INT_TYPE, Type.INT_TYPE };
        String names[] = new String[]{ "field0", "field1", "field2" };
        TupleDesc descriptor = new TupleDesc(types, names);

        // create the table, associate it with some_data_file.dat
        // and tell the catalog about the schema of this table.
        HeapFile table1 = new HeapFile(new File("some_data_file.dat"), descriptor);
        Database.getCatalog().addTable(table1, "test");

        // construct the query: we use a simple SeqScan, which spoonfeeds
        // tuples via its iterator.
        TransactionId tid = new TransactionId();
        SeqScan f = new SeqScan(tid, table1.getId());

        try {
            // and run it
            f.open();
            while (f.hasNext()) {
                Tuple tup = f.next();
                System.out.println(tup);
            }
            f.close();
            Database.getBufferPool().transactionComplete(tid);
        } catch (Exception e) {
            System.out.println ("Exception : " + e);
        }
    }

}
```

我们创建的表有三个整数字段。为了表达这一点，我们创建了一个 `TupleDesc` 对象，并向其传递一个<gtr gtr="182">对象的数组，以及可选的<gtr gtr="183">字段名的数组。一旦我们创建了这个 `TupleDesc`，我们就会初始化一个 `HeapFile` 对象，该对象表示存储在 `some_data_file.dat` 中的表。一旦我们创建了表，我们就将其添加到目录中。如果这是一个已经在运行的数据库服务器，我们会加载这个目录信息。我们需要显式地加载它，以使此代码自包含。

一旦我们完成了数据库系统的初始化，我们就创建了一个查询计划。我们的计划只包括从磁盘扫描元组的 `SeqScan` 运算符。通常，这些运算符是通过引用适当的表（在 `SeqScan` 的情况下）或子运算符（在 Filter 等情况下）来实例化的。然后，测试程序在 `SeqScan` 运算符上重复调用 `hasNext` 和。当元组从 `SeqScan` 输出时，它们会在命令行上打印出来。

我们**强烈建议**你可以将此作为一个有趣的端到端测试来尝试，它将帮助你获得为 simpledb 编写自己的测试程序的经验。你应该使用上面的代码在＜ gt r ＝“193”/＞目录中创建文件“test.java”，并在代码上方添加一些“import”语句，然后将＜ gt r=“194”/＞文件放在顶级目录中。然后运行：


```bash
ant
java -classpath dist/simpledb.jar simpledb.test
```

请注意， `ant` 编译 `test.java` 并生成一个包含该文件的新 jarfile。

## 3. 后勤

你必须提交你的代码（见下文）以及描述你的方法的简短文章（最多 2 页）。该书面报告应：

* 描述你做出的任何设计决定。这些对于实验室 1 来说可能是最小的。
* 讨论并证明你对 API 所做的任何更改。
* 描述代码中任何缺失或不完整的元素。
* 描述一下你在实验室里花了多长时间，是否有什么特别困难或令人困惑的地方。

### 3.1. 协作

这个实验室应该是一个人可以管理的，但如果你更喜欢和伴侣一起工作，这也是可以的。不允许更大的团队。请在你的个人写作中清楚地表明你与谁共事过，如果有人的话。

### 3.2. 提交作业

<!--
To submit your code, please create a <tt>6.830-lab1.tar.gz</tt> tarball (such
that, untarred, it creates a <tt>6.830-lab1/src/simpledb</tt> directory with
your code) and submit it on the [6.830 Stellar Site](https://stellar.mit.edu/S/course/6/sp13/6.830/index.html). You can use the `ant handin` target to generate the tarball.
-->

我们将使用 Gradescope 自动升级所有编程作业。你们都应该被邀请到类实例；如果没有，请查看 Piazza 的邀请码。如果你仍然有问题，请告诉我们，我们可以帮助你设置。你可以在截止日期前多次提交代码；我们将使用 Gradescope 确定的最新版本。将撰写的内容与你提交的内容一起放入一个名为 `lab1-writeup.txt` 的文件中。

如果你与合作伙伴一起工作，则只需要一个人提交到 Gradescope。但是，请确保将另一个人添加到你的群中。还要注意，每个成员都必须有自己的书面记录。请将你的 Kerberos 用户名添加到文件名和写入本身中（例如， `lab1-writeup-username1.txt` 和 `lab1-writeup-username2.txt`）。

提交到 Gradescope 的最简单方法是使用包含代码的 `.zip` 文件。在 Linux/macOS 上，你可以通过运行以下命令来执行此操作：


```bash
$ zip -r submission.zip src/ lab1-writeup.txt

# If you are working with a partner:
$ zip -r submission.zip src/ lab1-writeup-username1.txt lab1-writeup-username2.txt
```

### 3.3. 提交错误

请向[6.5830-staff@mit.edu](mailto:6.5830-staff@mit.edu)提交（友好！）错误报告。当你这样做时，请尝试包括：

* 错误的描述。
* 一个 `.java` 文件，我们可以将其放在 `test/simpledb` 目录中，编译并运行。
* 一个 `.txt` 文件，其中包含再现错误的数据。我们应该能够使用 HeapFileEncoder 将其转换为 `.dat` 文件。

如果你是第一个报告代码中特定错误的人，我们会给你一块糖果！

<!--最新的错误报告/修复可以在[here](bugs.html)中找到。-->

<a name="grading"></a>

### 3.4 分级

75% 的分数将取决于你的代码是否通过系统测试套件，我们将对其进行测试。这些测试将是我们提供的测试的超集。在提交代码之前，你应该确保它不会从 `ant test` 和 `ant systemtest` 中产生错误（通过所有测试）。

**重要信息：**在测试之前，Gradescope 将用我们的这些文件版本替换你的 `build.xml` 以及目录的全部内容。这意味着你无法更改 `.dat` 文件的格式！你还应该小心更改我们的 API。你应该测试你的代码是否编译了未修改的测试。

提交后，你应该立即从 gradescope 获得失败测试的反馈和错误输出（如果有的话）。给出的分数将是你在作业中亲笔签名部分的分数。额外 25% 的分数将基于你的写作质量和我们对你代码的主观评估。这一部分也将在我们为你的作业打分后发布在 gradescope 上。

我们在设计这项任务时玩得很开心，希望你喜欢黑客！
