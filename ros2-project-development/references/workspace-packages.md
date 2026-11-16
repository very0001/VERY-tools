# 工作空间与功能包

## 从一开始设计

在写节点之前，画出功能流和依赖方向：基础功能互相通过契约连接；组合包依赖基础包，基础包不反向依赖组合包。按变化原因、独立运行和复用需求划分功能，不按每个函数、每一种设备硬拆一包。功能包名称应符合所用 ROS 2 工具链的命名要求、体现职责。

以下为工作空间层级，`src/` 下直接放 ROS 2 功能包；目录仅在确有相应功能时创建：

```text
robot_ws/
├── src/
│   ├── sensor_reader/         # Python 基础功能包（ament_python）
│   ├── motion_driver/         # C++ 基础功能包（ament_cmake）
│   ├── robot_interfaces/     # 按需建立：msg/、srv/、action/
│   └── robot_bringup/         # 按需建立：系统 launch、配置和协调节点
├── build/                     # colcon 构建生成
├── install/                   # colcon 安装生成
├── log/                       # 构建日志
├── README.md
└── requirements.txt           # 存在 pip 依赖时
```

### Python 功能包：ament_python

默认参考 ROS 2 官方教程及常见 Python 包结构，同名目录是可导入的 Python 模块包：

```text
sensor_reader/                       # ROS 2 功能包根目录
├── package.xml                     # 包信息和 ROS / rosdep 依赖
├── setup.py                        # Python 包、资源和可执行入口安装
├── setup.cfg                       # ROS 2 可执行入口安装位置
├── resource/
│   └── sensor_reader               # ament 资源索引标记文件
├── sensor_reader/                  # 与功能包同名的 Python 模块包
│   ├── __init__.py
│   ├── reader_node.py              # 节点和 main 入口
│   └── helpers.py                  # 此包内部共用 Python 代码（按需）
├── launch/                         # 本包启动文件（按需）
├── config/                         # 参数和配置（按需）
├── test/                           # 测试（按需）
└── README.md
```

`setup.py` 使用 `find_packages(exclude=['test'])` 等方式发现模块，`console_scripts` 中注册例如 `reader = sensor_reader.reader_node:main`。`data_files` 要安装 `resource/<包名>` 和 `package.xml`，并按需将 `launch/`、`config/` 等资源安装到 `share/<包名>/`；`setup.cfg` 的 `script_dir` 和 `install_scripts` 指向 `$base/lib/<包名>`。`package.xml` 声明实际构建工具、ROS 运行及测试依赖。`__init__.py` 保持在 Python 模块包中，可复用模块也放在 Python 包内，而不是为模仿 C++ 布局塞入 `include/`。资源读取使用包索引，详见 [资源、路径与依赖](paths-dependencies.md)。

### C++ 功能包：ament_cmake

```text
motion_driver/
├── package.xml
├── CMakeLists.txt
├── src/                            # C++ 源文件与节点入口
├── include/motion_driver/          # 对外公开的头文件（按需）
├── launch/                         # 按需
├── config/                         # 按需
├── test/                           # 按需
└── README.md
```

通过 `CMakeLists.txt` 声明依赖、构建并安装节点到 `lib/<包名>`；按需安装并导出公共库及头文件，安装 launch、配置等到 `share/<包名>`。消息接口通常用独立的 `ament_cmake` 接口包和 rosidl 生成。涉及 Python/C++ 混合包时根据实际构建类型选用 `ament_cmake_python` 或拆分功能包，不把纯 Python 模板机械套用。

上述同名 Python 模块目录是新建 `ament_python` 包的默认方案，不代表其他能正确安装的 Python 布局不合法。已有项目尊重已验证的结构；用户明确指定其他布局时，配置包发现和安装，不仅调整目录外观。用 `colcon list` 检查工作空间的包发现和重名，并通过构建或依赖图工具检查依赖关系与依赖环。

## 基础功能包

- 每包说明输入输出、职责和不负责的内容；节点的参数、设备和资源位置可配置，不隐式依赖其他功能包的某个启动脚本。
- 按语言和构建类型组织代码：新建 `ament_python` 包默认使用包内同名 Python 模块目录；C++ 实现放 `src/`、公共头文件按需放 `include/<package>/`。已有包或用户明确要求其他合法结构时，保留或按要求设计，并验证 Python 包发现、导入和资源安装。
- Python 需遵循解释器导入和 setuptools 包发现机制；`__init__.py` 是 Python 包的一部分，不能为了视觉目录整洁直接移动或删除。`__pycache__` 是运行缓存，不纳入源代码版本管理。检查 `setup.py` 的包发现、`console_scripts`、资源索引标记与 `package.xml` 安装、launch/config 安装，以及 `setup.cfg` 的执行入口位置；普通安装后在新 shell 验证 `ros2 pkg prefix`、`ros2 pkg executables`、`ros2 run`、Python 模块导入和已安装资源的发现。
- 提供适合的独立节点运行方式和可重复的单模块验证输入。例如传感器包可用录制数据，处理包可用话题注入，纯算法可脱离 ROS 做单元测试。与功能相关的依赖只声明给实际需要的包。
- 目录树只是设计意图；README 要以真实文件与真实命令为准。不要为满足模板复制空文件夹，不把第三方实现或多个版本任意散落在包根目录。

## 系统组合包

用有意义的包名，不固定叫 AGENT。提供一个或多个 `launch/`，选择已启用的基础节点并传入参数及资源配置；动态状态、超时、重试和业务流程由组合包中的协调节点负责，不把 launch 当作运行期状态机。组合层可有节点实现流程控制，但不得拷贝基础算法或依赖基础包的私有源码路径。

参数不同、提示词不同或工作模式不同的组合可用 launch 参数及配置选择；说明选择方式和默认值。确有新组合需求时优先扩展组合层；仅当基础能力本身不足时才修改或扩展基础包。

## 自定义消息与接口

优先选择语义、单位和时间约定合适的标准消息；确实有稳定的跨包契约无法表达时，在独立接口包中按需创建 `msg/`、`srv/`、`action/`，为字段、单位、坐标系、状态码、版本和兼容性写说明。相关 `package.xml`、`CMakeLists.txt`、rosidl 生成依赖必须齐全；生成之后在新 shell 验证能否发现、导入及跨包使用。

相关接口可集中到同一个接口包，但按项目规模和复用边界判断，不为了统一而引入不必要的全局耦合。接口包不依赖消费它的业务包；保留兼容性，已有类型更名或删除前确认影响。

参考：[ROS 2 Humble Python 发布与订阅教程（源码）](https://github.com/ros2/ros2_documentation/blob/humble/source/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.rst)、[ROS 2 功能包开发指南（源码）](https://github.com/ros2/ros2_documentation/blob/humble/source/How-To-Guides/Developing-a-ROS-2-Package.rst)。
