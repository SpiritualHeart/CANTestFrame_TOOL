# CAN_TestFrame_TOOL
![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![PyQt5](https://img.shields.io/badge/GUI-PyQt5-green) ![Platform](https://img.shields.io/badge/平台-Windows-lightgrey) ![License](https://img.shields.io/badge/授权-仅供个人免费使用-orange)

Windows 平台 **CAN 测试帧代码自动生成工具**，基于 PyQt5 开发。

导入 I/O 枚举文本或 Excel 信号定义表，一键生成 CAN 测试帧 C 代码，以及上位机（Vehicle Spy / CANdb 等）所需的 INI / DBC 报文数据库文件，省去手工编写测试帧与逐条配置报文的重复劳动。

## 功能特性

### IO枚举文本

- 导入 I/O 枚举定义 txt 文件，自动识别 `INPUTS_ENUM` / `OUTPUTS_ENUM` 及 `NUM_OF_INPUTS` / `NUM_OF_OUTPUTS` 标记
- 自动生成 `app_can_test_frame_01_input_output.c`：输入/输出测试帧函数，8 字节 CAN 帧数据按位打包
- 同步生成上位机 `.ini` / `.dbc` 文件（Motorola 起始位格式）

### Debug表格

- 导入 Excel 信号定义表（`.xlsx`），自动跳过第 1 个工作表，为其余每个工作表生成独立的 CAN 测试帧 C 函数
- 支持合并单元格解析：信号位宽完全由 C 列合并格数决定（合并 n 格 = n bit；单字节行合并 n 格 = n×8 bit 多字节信号）
- 覆盖 1bit 置位、2~7bit 移位或运算、整字节直接赋值、多字节大端拆分等全部组合场景，并内置完整的格式校验与告警
- 为每个工作表独立生成上位机 `.ini` 与 `.dbc` 文件

### 其他

- 绿色单文件软件，解压即用，无需安装
- 详尽的格式校验：Byte 分组完整性、Bit 编号连续性、信号跨字节合法性、CAN ID 提取（兼容 `7E0` / `0x7E0` / `ID7E0` 三种写法）
- 内嵌 HTML 使用说明书（帮助 → 使用教程，自动在浏览器打开）
- 生成文件均为 Windows CRLF 换行 + UTF-8 无 BOM 编码，可直接用于嵌入式工程

## 界面预览

| 启动界面 | 导入子菜单 |
| :---: | :---: |
| ![启动界面](docs/images/shot1_empty.png) | ![导入子菜单](docs/images/shot3_import_menu.png) |

| IO枚举文本 模块 | Debug表格 模块 |
| :---: | :---: |
| ![IO枚举文本](docs/images/shot4_io_enum.png) | ![Debug表格](docs/images/shot6_debug_excel.png) |

## 快速开始

### 方式一：直接运行 exe

1. 下载 `CAN_TestFrame_TOOL.exe`
2. 双击运行（若杀毒软件误报，添加信任即可）
3. 菜单栏 **文件 → 导入** 选择功能模块，选择输入文件
4. 点击 **生成** ，在弹出的目录选择输出位置

### 方式二：从源码运行

环境要求：Python 3.8+，依赖 `PyQt5`、`openpyxl`

```bash
pip install PyQt5 openpyxl
python CAN_TestFrame.py
```

## 输入文件格式

### IO 枚举文本（IOenum.txt）

标准 C 枚举定义，包含以下结构即可被识别：

```c
typedef enum {
    IN1,            /* 枚举成员，逐个解析 */
    IN2,
    ...
    NUM_OF_INPUTS   /* 输入数量标记 */
} INPUTS_ENUM;

typedef enum {
    OUT1,
    OUT2,
    ...
    NUM_OF_OUTPUTS  /* 输出数量标记 */
} OUTPUTS_ENUM;
```

### Excel 信号定义表（.xlsx）

从第 2 个工作表开始解析（第 1 个忽略），第 11 行为表头，第 12 行起为数据。列定义如下：

| 列 | 字段 | 说明 |
| --- | --- | --- |
| A | Byte | 字节序号 0~7；单格 = 整字节，8 格纵向合并 = 该字节按 bit 定义 |
| B | Bit | bit 编号 0~7，仅 8 格合并组内需要填写 |
| C | Signal | 信号名，支持合并单元格（合并格数 = 信号位宽） |
| D | Type | 可选，仅作人工备注，不参与解析 |
| E | TestFrameID | CAN 测试帧 ID |

信号位宽判定规则：

| A 列形态 | C 列形态 | 位宽 |
| --- | --- | --- |
| 8 格合并组 | 单格 | 1 bit |
| 8 格合并组 | 纵向 n 格合并 | n bit |
| 单个单元格 | 单格 | 8 bit |
| 单个单元格 | 纵向 n 格合并 | n×8 bit（多字节信号，大端序） |

完整规则详见 [生成规则.md](生成规则.md)（含校验项、异常处理与示例）。

## 生成产物

| 文件 | 说明 |
| --- | --- |
| `app_can_test_frame_*.c` | CAN 测试帧 C 函数文件，可直接加入嵌入式工程 |
| `*.ini` | 上位机报文配置文件（Motorola 格式，Cycle=10） |
| `*.dbc` | CAN 报文数据库（`RX_` 前缀报文名，兼容 CANdb++） |

生成示例（混合位宽字节，先清零再按位或）：

```c
data[1] = 0u;
if (diag_err_level > 0u) { data[1] |= BIT0; }
data[1] |= (UINT8)((diag_err_level & 0x03u) << 1u);
```

## 从源码打包

```bash
pip install pyinstaller
pyinstaller CAN_TestFrame_TOOL.spec
```

打包产物为单文件 `CAN_TestFrame_TOOL.exe`，已内嵌 HTML 使用说明书与图标资源。

## 目录结构

```
CanIOEnumTool/
├── CAN_TestFrame.py              # 主程序源码
├── CAN_TestFrame_TOOL.exe         # 可执行文件（绿色单文件）
├── CAN_TestFrame_TOOL.spec        # PyInstaller 打包配置
├── CAN_TestFrame_TOOL使用说明书.html  # 内嵌说明书（HTML 版）
├── CAN_TestFrame_TOOL使用说明书.docx # 说明书 Word 源文件
├── 生成规则.md                     # Excel 解析与代码生成规则文档
├── logo_cat.png / logo_cat.ico    # 软件图标
├── docs/images/                   # 界面截图
└── output_files/                  # 生成产物示例
```

## 版本历史

- **V1.0.0**（2026-09-02）首个正式版本
  - IO枚举文本 / Debug表格 两大功能模块
  - 信号位宽完全由 Excel C 列合并格数决定（Type 列废弃）
  - 中文界面，内嵌 HTML 使用说明书

## 许可证

本项目仅供学习与内部工程使用，转载请注明出处。

# CAN_TestFrame_TOOL

![Version](https://img.shields.io/badge/Version-V1.0.0-orange)
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![PyQt5](https://img.shields.io/badge/GUI-PyQt5-green)
![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey)

Windows 平台 **CAN 测试帧代码自动生成工具**，基于 PyQt5 开发。

导入 I/O 枚举文本或 Excel 信号定义表，一键生成 CAN 测试帧 C 代码，以及上位机（Vehicle Spy / CANdb 等）所需的 INI / DBC 报文数据库文件，省去手工编写测试帧与逐条配置报文的重复劳动。

## 功能特性

### IO枚举文本

- 导入 I/O 枚举定义 txt 文件，自动识别 `INPUTS_ENUM` / `OUTPUTS_ENUM` 及 `NUM_OF_INPUTS` / `NUM_OF_OUTPUTS` 标记
- 自动生成 `app_can_test_frame_01_input_output.c`：输入 / 输出测试帧函数，8 字节 CAN 帧数据按位打包
- 同步生成上位机 `.ini` / `.dbc` 文件（Motorola 起始位格式）

### Debug表格

- 导入 Excel 信号定义表（`.xlsx`），自动跳过第 1 个工作表，为其余每个工作表生成独立的 CAN 测试帧 C 函数
- 支持合并单元格解析：信号位宽完全由 C 列合并格数决定（合并 n 格 = n bit；单字节行合并 n 格 = n×8 bit 多字节信号）
- 覆盖 1bit 置位、2~7bit 移位或运算、整字节直接赋值、多字节大端拆分等全部组合场景，内置完整的格式校验与告警
- 为每个工作表独立生成上位机 `.ini` 与 `.dbc` 文件

## 快速开始

操作流程：启动软件 → **文件 → 导入** → 选择功能模块与输入文件 → 预览解析结果 → **生成** → 选择输出目录。

### 方式一：直接运行 exe

1. 获取 `CAN_TestFrame_TOOL.exe`（单文件绿色版，无需安装；exe 不入库，可通过 Releases 或其他渠道分发）
2. 双击运行（若杀毒软件误报，添加信任即可）
3. 菜单栏 **文件 → 导入** 选择功能模块，选择输入文件
4. 点击 **生成**，在弹出的目录选择框中确定输出位置

### 方式二：从源码运行

环境要求：Python 3.8+，依赖 `PyQt5`、`openpyxl`

```bash
pip install PyQt5 openpyxl
python CanIOEnumTool/CAN_TestFrame.py
```

## 输入文件格式

### IO 枚举文本（IOenum.txt）

标准 C 枚举定义，包含以下结构即可被识别：

```c
typedef enum {
    IN1,            /* 枚举成员，逐个解析 */
    IN2,
    ...
    NUM_OF_INPUTS   /* 输入数量标记 */
} INPUTS_ENUM;

typedef enum {
    OUT1,
    OUT2,
    ...
    NUM_OF_OUTPUTS  /* 输出数量标记 */
} OUTPUTS_ENUM;
```

### Excel 信号定义表（.xlsx）

从第 2 个工作表开始解析（第 1 个忽略），第 11 行为表头，第 12 行起为数据。列定义如下：

| 列 | 字段 | 说明 |
| --- | --- | --- |
| A | Byte | 字节序号 0~7；单格 = 整字节，8 格纵向合并 = 该字节按 bit 定义 |
| B | Bit | bit 编号 0~7，仅 8 格合并组内需要填写 |
| C | Signal | 信号名，支持合并单元格（合并格数 = 信号位宽） |
| D | Type | 可选，仅作人工备注，不参与解析 |
| E | TestFrameID | CAN 测试帧 ID |

信号位宽判定规则：

| A 列形态 | C 列形态 | 位宽 |
| --- | --- | --- |
| 8 格合并组 | 单格 | 1 bit |
| 8 格合并组 | 纵向 n 格合并 | n bit |
| 单个单元格 | 单格 | 8 bit |
| 单个单元格 | 纵向 n 格合并 | n×8 bit（多字节信号，大端序） |

完整规则、校验项与异常处理详见 [生成规则.md](生成规则.md)。

## 生成产物

| 文件 | 说明 |
| --- | --- |
| `app_can_test_frame_*.c` | CAN 测试帧 C 函数文件，可直接加入嵌入式工程 |
| `*.ini` | 上位机报文配置文件（Motorola 格式，Cycle=10） |
| `*.dbc` | CAN 报文数据库（`RX_` 前缀报文名，兼容 CANdb++） |

生成示例（混合位宽字节，先清零再按位或）：

```c
data[1] = 0u;
if (diag_err_level > 0u) { data[1] |= BIT0; }
data[1] |= (UINT8)((diag_err_level & 0x03u) << 1u);
```

## 从源码打包

```bash
cd CanIOEnumTool
pip install pyinstaller
pyinstaller CAN_TestFrame_TOOL.spec
```

打包产物为单文件 `CAN_TestFrame_TOOL.exe`，已内嵌 HTML 使用说明书与图标资源。

## 目录结构

```
.
├── README.md                                 # 本说明
├── 生成规则.md                                # Excel 解析与代码生成规则
├── IOenum.txt                                # I/O 枚举输入文件
├── BEBG can debug msg_*.xlsx                 # Excel 信号定义输入文件
└── CanIOEnumTool/                            # 工具目录
    ├── CAN_TestFrame.py                      # 主程序源码
    ├── CAN_TestFrame_TOOL.spec               # PyInstaller 打包配置
    ├── CAN_TestFrame_TOOL使用说明书.html      # 使用说明书（内嵌于 exe）
    ├── 生成规则.md                            # 生成规则（工具目录内副本）
    └── logo_cat.png / logo_cat.ico           # 软件图标
```

## 版本历史

- **V1.0.0**（2026-09-02）首个正式版本
  - IO枚举文本 / Debug表格 两大功能模块
  - 信号位宽完全由 Excel C 列合并格数决定（Type 列废弃）
  - 中文界面，内嵌 HTML 使用说明书

## 许可证

本项目仅供学习与内部工程使用，转载请注明出处。
