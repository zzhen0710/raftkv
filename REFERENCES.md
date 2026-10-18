# Reference Survey

> 调研日期：2026-10-13。目的：为初学者选择教学材料与工程参照，不是给开源项目做完整质量排名。
>
> 返回 [README.md](README.md)。

## 1. 我们实际查了什么

使用 GitHub 连接器读取公开仓库元数据、README、目录与部分源码，并查阅官方课程及协议资料。下表固定本次观察的版本，便于以后复查；日期指所查分支 HEAD 提交时间，不是整个项目的最新活动时间。

**没有 clone、构建、运行这些项目，没有验证其全部算法或性能。** “值得参考”表示在所查材料中找到了明确的教学或工程价值，不表示已经认证实现正确。

| 仓库 | 定位 | 固定版本 | HEAD 提交日期（UTC） | 检查范围 |
| --- | --- | --- | --- | --- |
| [talent-plan/tinykv](https://github.com/talent-plan/tinykv) | 课程骨架；Go | [cf7ed8da34fe](https://github.com/talent-plan/tinykv/tree/cf7ed8da34fea520c7e2f909e23bd5c27414961a) | 2025-05-03 | README、项目 1/2 文档、目录、Raft 测试片段 |
| [etcd-io/raft](https://github.com/etcd-io/raft) | 共识库；Go | [638f42739b77](https://github.com/etcd-io/raft/tree/638f42739b77572b5708eb0d8a776a272ade71ed) | 2026-10-09 | README、目录、测试网络源码片段 |
| [eBay/NuRaft](https://github.com/eBay/NuRaft) | 共识库；C++ | [46c73ad37cf5](https://github.com/eBay/NuRaft/tree/46c73ad37cf5750342dd7b3b222422993f638251) | 2026-10-12 | README、集成文档、calculator 文档、故障测试片段 |
| [etcd-io/etcd](https://github.com/etcd-io/etcd) | 完整服务；Go | [a2b3d9fe8643](https://github.com/etcd-io/etcd/tree/a2b3d9fe864358c87a91176f388bc0cbe0da18dd) | 2026-10-12 | README、目录、robustness 测试说明 |
| [skyzh/mini-lsm](https://github.com/skyzh/mini-lsm) | 单机存储课程；Rust | [2f7ce31598e8](https://github.com/skyzh/mini-lsm/tree/2f7ce31598e86282ae7d372a55c99381c58e42e0) | 2026-09-23 | README |
| [youngyangyang04/KVstorageBaseRaft-cpp](https://github.com/youngyangyang04/KVstorageBaseRaft-cpp) | 中文学习实现；C++ | [f38c752cccd9](https://github.com/youngyangyang04/KVstorageBaseRaft-cpp/tree/f38c752cccd9561a5461fb7225514ed897c2ac08) | 2025-11-13 | README、目录、CMake、测试样例、持久化及调用源码 |
| [Charlie-XIAO/raft-kv](https://github.com/Charlie-XIAO/raft-kv) | 个人学习实现；Go 等语言的说明 | [786c54a6737f](https://github.com/Charlie-XIAO/raft-kv/tree/786c54a6737ffc9e0892763a1615eb6ba0de7c84) | 2025-08-04 | README、完整目录树 |

选择依据：任务是否清楚、模块边界是否可解释、能否找到测试与故障语义、是否适合当前学习阶段。不以 Star 数、项目名或求职宣传作质量结论。

## 2. 最值得借鉴的材料

### TinyKV：借课程拆分，不照搬全部目标

- 事实：Project 1 从单机 KV 开始，包含 Put/Delete/Get/Scan；所谓实现存储层，在该阶段主要是包装现成 Badger 的接口，不要求先从头写一套存储引擎。
- 事实：Project 2 将 Raft 逻辑和 KV 集成分层；核心通过 tick、消息和状态变化驱动。后续课程还有多 Raft 组与事务。
- 抽查：Raft 测试设置初始状态、输入事件，再检查状态和输出消息；这比只能启动三台服务的演示更有利于定位错误。
- 我的判断：最适合借鉴“如何缩小每次的学习问题”。对当前项目，只先取单机和单组 Raft 的思路；不把 Go、分片和事务一并变成前置要求。

证据：[Project 1](https://github.com/talent-plan/tinykv/blob/cf7ed8da34fea520c7e2f909e23bd5c27414961a/doc/project1-StandaloneKV.md)、[Project 2](https://github.com/talent-plan/tinykv/blob/cf7ed8da34fea520c7e2f909e23bd5c27414961a/doc/project2-RaftKV.md)、[Raft 测试](https://github.com/talent-plan/tinykv/blob/cf7ed8da34fea520c7e2f909e23bd5c27414961a/raft/raft_paper_test.go)。

### NuRaft：观察 C++ 集成，特别看清示例边界

- 事实：库提供 Raft 和网络层；使用者仍需提供日志存储、应用状态机、状态管理等实现。接库不等于不需要理解持久化。
- 事实：calculator 示例复制的是整数上的计算操作，文档明确说明数据全部易失、不写磁盘。节点重新加入时可从仍存活的集群追赶，不等于本地磁盘恢复，更不等于全体退出后数据仍在。
- 抽查：故障测试使用可控的网络和计时机制，覆盖隔离旧 Leader、重新选举等情形。
- 我的判断：适合观察三节点行为和理解 C++ 的集成责任；也可以成为后续服务实现路线的候选。暂时不替用户锁定依赖。

证据：[如何使用](https://github.com/eBay/NuRaft/blob/46c73ad37cf5750342dd7b3b222422993f638251/docs/how_to_use.md)、[calculator](https://github.com/eBay/NuRaft/blob/46c73ad37cf5750342dd7b3b222422993f638251/examples/calculator/README.md)、[故障测试](https://github.com/eBay/NuRaft/blob/46c73ad37cf5750342dd7b3b222422993f638251/tests/unit/failure_test.cxx)。

### etcd-io/raft：先让协议逻辑可测试

- 事实：仓库提供共识核心，把磁盘、网络等留给调用方；README 描述以消息和时钟事件为输入的确定性状态机设计。
- 抽查：测试网络支持断开连接、丢包与延迟，并允许用随机源控制故障选择。所查网络文件仍有实际等待，不应将它直接说成“全虚拟时间模拟器”。
- 我的判断：最值得学的是边界。先用事件驱动的小模型理解协议，再让真实网络和磁盘成为外层适配，通常更容易解释错误。

证据：[README](https://github.com/etcd-io/raft/blob/638f42739b77572b5708eb0d8a776a272ade71ed/README.md)、[测试网络](https://github.com/etcd-io/raft/blob/638f42739b77572b5708eb0d8a776a272ade71ed/rafttest/network.go)。

### etcd：学习怎样质疑自己的正确性

- 事实：etcd 是实际使用的分布式 KV 服务；其 robustness 测试文档把 API 保证、故障注入和历史检查联系起来，还保留了已发现问题及复现记录。
- 我的判断：成熟项目也需要持续发现边界问题。我们先借鉴“制造故障 → 保存操作历史 → 与契约对照”，不先阅读整套实现。
- 限制：默认分支的固定 SHA 用于本次源码考察，不是建议安装的稳定版本。真正运行时另选合适发布版本并核对文档。

证据：[项目定位](https://github.com/etcd-io/etcd/blob/a2b3d9fe864358c87a91176f388bc0cbe0da18dd/README.md)、[故障测试说明](https://github.com/etcd-io/etcd/blob/a2b3d9fe864358c87a91176f388bc0cbe0da18dd/tests/robustness/README.md)、[API 保证](https://etcd.io/docs/v3.6/learning/api_guarantees/)。

### mini-lsm：用来区分“存储”与“共识”

- 事实：课程围绕 Rust 单机 LSM 存储，涉及内存表、磁盘表、压缩、恢复及后续事务主题，不是 Raft 复制课程。
- 我的判断：它说明“把一台机器上的数据存好”本身就很有深度。当前只认清边界；若后续发现自己更喜欢存储，可把它作为转向依据，不必同时开做。
- 限制：本次只读 README，未检查该仓库实现与测试。代码许可与课程文字版权应分别核对。

证据：[README](https://github.com/skyzh/mini-lsm/blob/2f7ce31598e86282ae7d372a55c99381c58e42e0/README.md)。

## 3. 中文 C++ 项目：可以读，但要带着证据判断

### KVstorageBaseRaft-cpp

它把 Raft、RPC、存储等放在一个 C++ 项目中，中文说明降低了阅读门槛。这是它对当前学习者的吸引力，也容易让人把“模块很多”当作能力很强。

**本次发现一个具体的恢复风险：**

1. 所查版本的 `Persister` 构造函数，用 `std::ios::out | std::ios::trunc` 打开 Raft 状态和快照文件。
2. `trunc` 会截断已有文件；这不是保留旧状态后恢复的打开方式。
3. 在 `kvServer.cpp` 的初始化流程中，先构造 `Persister`，随后初始化 Raft，之后才读取快照。

这是源码静态证据。**据此不能把该版本直接当成进程重启恢复的正确范本。** 本次没有构建或运行复现，也不据一个问题断言所有模块都无价值。

其他观察：

| 观察 | 对我们的意义 |
| --- | --- |
| 构建涉及 C++20、muduo 等，另有 RPC、协程等模块 | 全盘照搬会同时引入多个学习负担 |
| 抽查的 `test/run.cpp` 主要是序列化样例 | 一个名为 test 的目录不等于已有系统性的故障测试 |
| 本次未取得充分的端到端故障验证证据 | 不能把“README 声称实现”当作“已验证保证” |
| 部分深入资料在仓库外，本次未读取付费内容 | 评价仅基于公开取得的材料 |
| 本次未确认明确代码许可 | 后续需要复制时先核对，当前不导入代码 |

我的判断：适合对照“模块如何连起来”，也适合练习源码质疑；不选为唯一教材，不先照抄持久化实现。

证据：[持久化实现](https://github.com/youngyangyang04/KVstorageBaseRaft-cpp/blob/f38c752cccd9561a5461fb7225514ed897c2ac08/src/raftCore/Persister.cpp)、[调用顺序](https://github.com/youngyangyang04/KVstorageBaseRaft-cpp/blob/f38c752cccd9561a5461fb7225514ed897c2ac08/src/raftCore/kvServer.cpp)、[构建文件](https://github.com/youngyangyang04/KVstorageBaseRaft-cpp/blob/f38c752cccd9561a5461fb7225514ed897c2ac08/CMakeLists.txt)、[测试样例](https://github.com/youngyangyang04/KVstorageBaseRaft-cpp/blob/f38c752cccd9561a5461fb7225514ed897c2ac08/test/run.cpp)。

### raft-kv：暂不作为 C++ 起点

`Charlie-XIAO/raft-kv` 的 README 描述了 Go 实现及 C++、Rust 重实现，但本次检查该固定版本的完整目录树，根目录只有 `gosrc`、`images` 等，没有所述 `cppsrc`、`rustsrc`。

这不证明其他分支或未来版本没有代码；它说明本次证据不足以把这个版本推荐成可直接使用的 C++ 示例。暂缓选用，避免你跟着不存在于当前目录的说明找文件。

证据：[README](https://github.com/Charlie-XIAO/raft-kv/blob/786c54a6737ffc9e0892763a1615eb6ba0de7c84/README.md)、[固定版本目录](https://github.com/Charlie-XIAO/raft-kv/tree/786c54a6737ffc9e0892763a1615eb6ba0de7c84)。

## 4. GitHub 之外的主要教材

| 资料 | 何时看 | 要带着的问题 |
| --- | --- | --- |
| [Raft 官网](https://raft.github.io/)与[原论文](https://raft.github.io/raft.pdf) | 理解基本复制后，写协议前 | 安全性依靠哪些规则，而不是哪些直觉？ |
| [MIT Raft 实验](https://pdos.csail.mit.edu/6.824/labs/lab-raft1.html) | 准备做选主、复制、恢复时 | 每一阶段怎样被测试？ |
| [MIT KV 实验](https://pdos.csail.mit.edu/6.824/labs/lab-kvraft1.html) | 共识层连接 KV 时 | 提交、应用、返回与重试怎样衔接？ |
| [Raft 动画](https://thesecretlivesofdata.com/raft/) | 读完导引后 | 断开节点后，谁还能继续组织操作？ |

注意 MIT 的持久化实验使用课程提供的 `Persister` 抽象，并非在该作业里直接实现真实磁盘写入。实验通过与磁盘同步、掉电恢复是不同层次的保证。[实验说明](https://pdos.csail.mit.edu/6.824/labs/lab-raft1.html)

动画页面依赖交互渲染，本次没有实际操作动画。论文和课程用于核对概念，不应把任意第三方课程答案仓库当作规范。

## 5. 怎样自己判断下一个项目值不值得读

只需先问六个问题：

1. **它解决什么问题？** 是课程骨架、共识库、存储引擎，还是完整 KV？
2. **声明了哪些保证？** 进程重启、断网、全体退出后分别怎样？
3. **在哪段代码实现？** 沿着请求、日志、提交、应用、响应追一遍。
4. **怎么知道它没错？** 有无能触发错误的测试、故障记录、历史检查？
5. **运行条件是什么？** 是否缺代码、依赖或说明，示例是否只在内存中？
6. **哪些内容可以复用？** 先核对许可，再区分使用库与自己实现。

无需读懂全部源码才开始。带着一个具体问题找到证据，比收藏十个仓库更有帮助。

## 6. 本目录怎样使用这些资料

- 当前只保存原创导引、链接与摘要，不复制第三方实现。
- 调研中的固定版本用于复查，不自动成为未来的依赖版本。
- 真正运行或复用前再核对依赖、发布版本与许可；上游变化后可更新考察记录。
- 若后续实验推翻当前判断，在这里记录反例和修正依据。
