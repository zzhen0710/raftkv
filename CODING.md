# RaftKV C++ 工程规范

更新：2026-10-19。适用于后续实现；当前尚无源码。采用 C++20，按实际问题使用语言特性，不要求覆盖语法清单。协作规则见 [AGENTS](AGENTS.md)，版本范围见 [ROADMAP](ROADMAP.md)，逐节实施与审查见 [workflow](docs/workflow.md)，测试要求见 [testing](docs/testing.md)。

## 最小风格基线

命名延续已有工程笔记习惯；排版采用 Google clang-format 基础样式。这是项目规范，不声称完整遵循 Google Style。

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| 类、结构体、类型别名 | PascalCase | RaftNode、LogIndex |
| 函数与方法 | camelCase | appendEntries |
| 变量与参数 | snake_case | next_index |
| 私有成员 | snake_case_ | current_term_ |
| 枚举类型 / 枚举值 | PascalCase | Role::Follower |
| 常量 | kPascalCase | kMaxMessageSize |
| 命名空间 | 小写 | raftkv |
| 文件 | snake_case，.h / .cpp | raft_node.h |

- 缩进、空格、换行交给统一版本的 clang-format；初版使用 Google 基础样式的两空格缩进。配置与工具版本随首次工程建立核验。
- 头文件自包含，直接包含所需声明；统一使用项目路径对应的 include guard。实现文件先包含自身头文件，再按系统/标准库、第三方、项目头分组。
- 头文件不写全局 using namespace；内部辅助实现限制在适当作用域。注释说明不变量、所有权、失败语义与设计理由。
- 大规模纯格式变更与行为变更分开；不顺手改动无关文件。

## 对象、资源与接口

- 优先按值持有和组合；需要动态独占所有权时使用 unique_ptr。shared_ptr 只用于真实共享所有权，不用它掩盖生命周期问题。
- RAII 管理 fd、socket、线程等资源；业务代码不手动配对 new/delete。一般类型遵循 Rule of Zero，资源包装类按需要显式禁止复制、提供移动。
- 裸指针、引用、string_view、span 默认不拥有资源。视图不得越过所有者生命周期；异步队列通常保存拥有的数据，除非借用期限有明确保证。
- 封装用于保护不变量，不为每个字段机械生成 getter/setter。输入在边界验证，包括长度、溢出、无效 ID 与协议状态。
- enum class、explicit 构造和必要的 nodiscard 表达约束。Term/LogIndex 若需防混用，采用不同包装类型；普通 using 别名没有这种类型隔离能力。
- 预期操作失败以明确结果传播；v0.1 可用 optional/枚举等简单结果，随 I/O 需求统一状态类型，不预建通用 Result 框架。不盲目照搬全局禁用异常；第三方异常及线程入口边界在接入时明确处理。
- 析构中的失败不能作为关键数据已持久化的依据；需要报告结果的 flush/sync 使用显式接口，不忽略错误。

## 面向对象与 C++20 的使用边界

- 封装与 RAII 全程实践。继承主要用于确有替换需求的接口，例如真实文件操作与故障注入实现；抽象基类按用途提供虚析构，覆盖函数写 override，避免对象切片。
- Leader/Follower/Candidate 初版作为节点角色状态，不拆成继承树。不为“以后也许替换”给每个模块加基类。
- 友元可用于强值类型的非成员比较；不普遍授予测试或其他模块访问内部状态的权限。
- span 用于有范围的字节视图；默认比较或 <=> 用于语义明确的值类型；指定成员初始化适合测试/配置聚合。
- Concepts 只约束真实泛型需求，ranges 保持简短可读；std::expected 不属于 C++20，不直接作为基线依赖。
- jthread/stop_token 仅在实际需要线程时引入。停止是协作式的，必须设计唤醒、退出和成员销毁顺序，自动 join 不能解决永久阻塞。
- modules、coroutines、复杂无锁结构不进入当前主线；不为了学习关键词增加生产路径复杂度。

## 协议、存储与并发

- Raft 核心只处理事件与状态转换，不直接开 socket、读写文件或 sleep；外层执行动作并反馈结果。相同状态和事件（含受控随机选择）产生可重放结果。
- 从选主阶段定义持久化依赖；磁盘版本初期采用简单同步顺序，不照抄上游并行优化。不能把复制、持久化、提交、应用混为一个状态。
- 每个节点的核心状态由单一执行流修改。增加线程时说明其所有者、共享字段、同步方式与关闭过程；默认不 detach，不持锁调用未知回调。
- 磁盘/网络格式明确字段宽度、字节序、版本、长度边界与校验；不得直接写出结构体内存布局，bit_cast 也不能替代序列化设计。
- 窄化/有符号性转换先验证适用范围，不能只加 cast 消除警告；输入长度、加乘运算、资源限制与 I/O 结果在相应边界检查。
- 测试从契约与反例推导；工具通过不能替代协议安全性和持久化分析。Python 测试代码的可读性与注释、失败清理同样接受审查，细则见 testing。

## 工具引入与检查

| 时机 | 工具或检查 | 完成要求 |
| --- | --- | --- |
| v0.1 工程建立 | CMake、Ninja、CTest、clang-format | C++20、关闭语言扩展，构建/测试入口可复现，统一格式版本 |
| v0.1 首批实现 | 编译警告、ASan + UBSan | 检查实际代码路径，记录命令与结果 |
| v0.2–v0.3 | 保守的 clang-tidy 检查、基础 CI | 先启用高价值规则，不一次打开所有 family |
| 核心稳定后 | Clang 第二编译器 | 实际验证所用 C++20 特性与构建 |
| 引入多线程时 | 独立 TSan 配置 | 覆盖共享状态与关闭路径，不与 ASan 混成一套 binary |
| 正确性稳定后 | 基准程序、必要时 perf | 先测量再决定优化 |

CMake 使用所需标准并设为 REQUIRED，关闭扩展；生成 compile_commands.json 供工具使用。初始警告至少包含 Wall/Wextra/Wpedantic，转换等检查在相关模块评估后加入。依赖警告不靠全项目压制解决。

初期只建实际需要的配置；共享参数进入项目 preset，机器专用参数留本地且不提交。工具安装、CI 接入和 sanitizer 执行必须有实际结果才记为完成。

## 规范依据

- [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)：可读性、一致性与自动化；不是全部政策照搬。
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)：资源、所有权、类型安全与类层次。
- [Clang-Format](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)：格式配置按实际安装版本核对。
- [CMake Ninja](https://cmake.org/cmake/help/latest/generator/Ninja.html)：构建后端。

本规范是在 2026-10-19 研究评审后选取的项目基线。后续只在真实问题需要时调整，规范更新不等于已经完成相关实践。
