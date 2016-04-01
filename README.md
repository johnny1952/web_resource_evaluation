## 利用Jmeter自动化评估WEB应用的资源监控模型设计

> Jmeter介绍和使用请看这里：http://www.i3geek.com/archives/1147
> 
> 本项目开源，项目地址：https://github.com/yangengzhe/web_resource_evaluation

### 背景

在云计算发展飞速的今天，随着SaaS应用的大量产生，关注点从编写应用逐渐转移到维护应用的稳定运行。如何高效的保证SaaS应用稳定的运行显得格外重要。

在目前的大多数云监控系统中，大部分是对应用的性能指标进行实时的控制，一旦超出预定阈值就会进行警报。但是这样的警报已经为时过晚，系统已经处于紧张状态，所以，为了保证系统稳定的运行，在突发之前就及时向管理人员发出警告，并给予管理员充足的维护时间是十分必要的。

因此，本系统主要是考虑，如何预测系统的状态，在系统崩溃前就预测到并及时的处理，以减少对用户产生的影响。而如何预测，本文则提出一种基于压测建立应用资源评估模型的方法，根据模型掌握应用运行的各个状态，以保证应用的文档。
### 目标

监控Tomcat下web应用的运行状态，建立资源评估模型，并根据模型对运行中的应用进行评估和预警

### 前期准备

#### 1、自动化压测工具JMeter

JMeter是个很好的测试工具，使用帮助：http://www.i3geek.com/archives/1147。但是本系统中需要自动化的测试，不能人工手动的操作，所以自动化工具是必要条件之一。

原理：通过JMeter录制脚本，生成jmx文件。在命令行下调用jmx脚本，同时传入参数，根据参数决定线程数以及网站，实现自动化测试。同时记录结果。

项目地址：https://github.com/yangengzhe/JMeter

#### 2、Tomcat的status应用

使用帮助：http://www.i3geek.com/archives/1126

利用应用采集的数据，轻松的获取运行中WEB的处理时间、请求数等信息。通过这些信息计算出平均响应时间，从而评估出应用的运行性能，和优化意见。

原理：定时通过访问应用，抓取xml。解析xml，获取关键数据，进行分析并记录。

项目地址：https://github.com/yangengzhe/tomcat_status

#### 3、系统状态的监控

使用帮助：http://www.i3geek.com/archives/1184

利用sigar.jar对系统的运行状态进行监控，实时记录不同并发时系统的CPU、内存、网络传输速度，从而评估出完整的应用性能模型。

项目地址：https://github.com/yangengzhe/sigar-system_runtime

### 技术基础

#### 1、压力测试（JMeter）

运行前，对应用进行压力测试，测试应用的稳定性，和性能评估。

1. 录制脚本
2. 分别测试并发为1、2、3、4、5、10、50、100、200、300、400、500...
3. 得到性能曲线（并发-吞吐率）

注:吞吐率=请求数/处理时间

#### 2、status应用监测

运行中，或压测时，从服务器内部角度分析应用的响应时间和处理效率。

1. 访问status.xml获得相关数据
2. 返回的数据中记录总的处理时间1，和总请求数1
3. 等待3秒
4. 访问status.xml获得相关数据
5. 返回的数据中记录总的处理时间2，和总请求数2
6. 得到平均处理时间：处理时间2-处理时间1 / 总请求数2-总请求数1

#### 3、远程处理时间检测

运行中，模拟发出请求，从用户体验角度分析应用的响应时间和处理效率。

1. 记录开始时间（本地时间）
2. 发送请求
3. 记录结束时间（本地时间）
4. 得到平均处理时间：开始时间-结束时间
5. 可以多次测量取平均值

#### 4、系统状态的检测

运行中，检测服务器运行时的资源消耗，从能源性能角度分析和优化。

1. 记录开始时间
2. 远程服务器开启服务端
3. 客户端定期请求返回CPU、内存以及网络传输速度
4. 结束计时
5. 多次测量取平均值以及峰值

### 实现步骤
#### 1、性能评估模型

通过运行前对应用的压力测试（Jmeter）以及status的应用检测，完成并发数和平均响应时间，并发数和吞吐率的关系模型。并进行存储。

一般情况下，并发数与吞吐率的关系是随着并发数的增加，吞吐率快速增加，当并发数达到‘最佳吞吐率’时，吞吐率达到最大值；当并发数再增加时，吞吐率几乎不变，但是当并发数增加到‘最大吞吐率’时，达到系统极限；当并发数再增加时，反而会使吞吐率下降并且会产生错误；当并发数继续增大时，最终使系统宕机。

#### 2、实时监控

在应用运行时，对应用的平均处理时间进行监控分析。通过status应用检测以及远程处理时间检测，两个方面进行判断，一旦应用超过模型中的预警值，要及时向管理员进行预警。

### 分析过程
#### 1、都需要测量什么？
WEB应用中对性能的评估主要是吞吐率、并发数、响应时间、传输时间等。

#### 2、如何对应用的性能进行评估？
应用的性能评估，简单来说就是对用户的直观感受，所以需要对并发和响应时间等角度进行分析。

#### 3、怎么测试性能？压力测试！
通过翻阅大量资料，发现个很方便、很适合的软件——JMeter。通过它可以简单、方便的测试出软件的并发、吞吐率。

#### 4、怎么自动化处理？
编写插件，利用java程序进行自动化处理，并采集和分析数据。

#### 5、仅仅用JMeter就可以了吗？为了体验还要保证运行时无误！
所以不仅仅要在运行前进行压测，还要在运行中对应用进行监控，实时发现问题，保证应用的正常使用。

#### 6、一切都用服务器的工具，简单测试就够了吗？还需要模拟用户，以用户角度考虑！
通过模拟程序，发出http请求，记录时间，完成一次模拟请求的操作，从而断定用户角度的感受，对有问题的页面及时反馈。

#### 7、这么多功能，怎么用？集成！
功能模块有了，最重要的就是集成，集成到一起实现自动化，就可以简单方便的变成通用工具了！

### 问题解决
#### 为什么运行时通过平均响应时间分析而不通过吞吐率分析呢？

吞吐率= 用户数*每个用户的请求数 / 总时间

1、运行时，无法保证所有请求的同时开始或结束，也就是说无法判断请求所用的总时间。

2、若根据一次请求的响应来判断，则负载会影响吞吐率，从而无法与模型比较，失去测试的意义。