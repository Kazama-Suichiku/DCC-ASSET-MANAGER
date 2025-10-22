# DCC Asset Manager

跨 DCC 的资产管理工具，现已同时支持 Houdini 与 Maya，提供统一的配置与缓存目录，统一的启动入口与一致的用户体验。

## 项目结构

```
DCC-ASSET-MANAGER/
├── launcher.py                  # 统一启动器（自动检测 DCC 并启动对应工具）
├── config/                      # 统一配置目录（运行后自动创建）
│   ├── houdini_ai.ini          # Houdini AI 助手配置（API Key 等）
│   ├── houdini_asset_checker.ini
│   └── maya_asset_check.ini    # Maya 资产检查配置
├── cache/                       # 统一缓存目录（运行后自动创建）
│   └── [Houdini 场景快照等]
├── shared/                      # 共享模块
│   ├── __init__.py
│   ├── common_utils.py         # 仓库根/配置/缓存路径工具
│   └── p4v_utils.py            # P4V 查找/启动（Maya/Houdini 共用）
├── HOUDINI_HIP_MANAGER/        # Houdini 工具
│   ├── main.py                 # Houdini 入口
│   ├── QUICK_SHELF_CODE.py     # 快速 Shelf 代码
│   ├── SHELF_TOOL_SETUP.md     # Shelf 设置说明
│   ├── core/                   # 核心逻辑
│   │   ├── __init__.py
│   │   ├── hip_manager.py      # HIP 文件管理器主窗口
│   │   └── asset_checker.py    # 资产检查器
│   ├── ui/                     # UI 组件
│   │   ├── __init__.py
│   │   ├── ai_tab.py          # AI 助手标签页（主界面）
│   │   ├── chat_window.py     # 全屏对话窗口
│   │   ├── widgets.py         # 自定义控件（浮动图标、加载动画等）
│   │   └── dialogs.py         # 对话框
│   └── utils/                  # 工具函数
│       ├── __init__.py
│       ├── ai_client.py       # AI API 客户端（OpenAI/DeepSeek/Ollama）
│       ├── common_utils.py    # 通用工具
│       ├── hip_utils.py       # HIP 文件操作
│       ├── p4v_utils.py       # P4V 集成
│       └── mcp/               # MCP（Model Context Protocol）系统
│           ├── __init__.py
│           ├── client.py      # MCP 客户端（节点操作）
│           ├── server.py      # MCP 服务器
│           ├── logger.py      # 日志
│           └── settings.py    # 设置
└── MAYA_MANAGER/               # Maya 工具（Suichiku 的 Maya 工具箱）
    ├── maya_main.py            # Maya 入口
    ├── core/                   # 核心逻辑
    │   ├── __init__.py
    │   └── asset_checker.py   # 资产检查器
    ├── ui/                     # UI 组件
    │   ├── __init__.py
    │   ├── main_window.py     # 主窗口
    │   ├── asset_check_tab.py # 资产检查标签页
    │   └── export_tab.py      # 导出标签页
    └── utils/                  # 工具函数
        ├── __init__.py
        ├── export_utils.py    # 导出工具
        └── p4v_utils.py       # P4V 集成
```

## 统一启动

两种方式二选一：

- 在 DCC 内调用统一启动器：
    ```python
    import sys
    sys.path.insert(0, r"C:\Users\KazamaSuichiku\Desktop\DCC-ASSET-MANAGER")
    import launcher
    launcher.show_tool()
    ```
- 或分别配置 Houdini/Maya 的 Shelf 工具/脚本按钮，脚本同上（详见下文）。

工具会自动检测当前运行环境（Houdini / Maya），并启动对应的工具窗口。

## 安装与配置（配置方法）

- 环境要求（推荐，主要是为了支持PySide6）
    - Windows（提供 P4V Windows 查找逻辑）
    - Python 3.9+
    - PySide6（UI）
    - **requests**（推荐，用于云端 AI API 调用，更好的 SSL 兼容性）
        ```powershell
        pip install requests
        ```
    - Houdini 21/20.5 / Maya 2025（按需）

- 将仓库根路径加入 sys.path（两种方式任选其一）
    - 在 DCC 的启动脚本或 Shelf 中加入：
        ```python
        import sys
        sys.path.insert(0, r"C:\\Users\\KazamaSuichiku\\Desktop\\DCC-ASSET-MANAGER")
        import launcher
        launcher.show_tool()
        ```
    - 或按需复制到公共插件路径并加入 sys.path。

- 统一配置与缓存
    - 配置目录：`<repo>/config/`（自动创建）
        - Maya：`maya_asset_check.ini`（由 UI 保存）
        - Houdini：历史/设置也统一写入此目录
    - 缓存目录：`<repo>/cache/`（自动创建）
        - Houdini：快照等临时文件

- P4V（可选）
    - 自动查找：常见安装路径、注册表、PATH、环境变量 `P4V_HOME`
    - 启动：工具栏或按钮一键启动（共用 `shared/p4v_utils.py`）

## 使用与功能（功能说明）

### 统一启动
- 在 Houdini 或 Maya 中调用 `launcher.show_tool()`，自动检测 DCC 并打开相应工具窗口。

## Houdini 工具（HIP Manager）

主要功能：
- HIP 文件管理：浏览、打开、保存、最近列表
- 快照预览：场景快照统一保存到仓库根目录的 `cache/`
- 资产检查：节点/参数检查（按规则）
- 导出支持：几何资产导出（bgeo/geo 等）
- P4V 集成：可一键打开 P4V 客户端

路径与配置：
- 所有配置与历史文件统一存放在仓库根目录 `config/` 下（已从旧路径迁移）。
- 生成的快照统一存放于仓库根目录 `cache/` 下。

在 Houdini 中快速配置 Shelf（推荐）：
```python
import sys, os
tool_path = r"C:\Users\KazamaSuichiku\Desktop\DCC-ASSET-MANAGER"
if tool_path not in sys.path:
        sys.path.insert(0, tool_path)

if 'launcher' in sys.modules:
        import importlib, launcher
        importlib.reload(launcher)
else:
        import launcher

launcher.show_tool()
```

### Houdini AI 助手

**核心功能:**
- **智能对话** - 与 OpenAI/DeepSeek/Ollama 模型对话，获取 Houdini 技术支持
- **完整对话历史** - 多轮对话，AI 能理解上下文并连续回答
- **快速响应模式** - 限制 512 tokens 和 30 秒超时，快速获取简短答案（默认开启）
- **标准模式** - 2048 tokens 和 120 秒超时，适合复杂问题和节点网络生成

**Houdini MCP 集成（Model Context Protocol）:**
- **读取选中节点** - 将当前选中节点的详细信息注入对话（类型、参数、错误等）
  - ☑️ **包含所有参数** - 可选读取节点的所有参数值（键值对形式）
- **搜索节点类型** - 搜索 Houdini 中可用的节点类型
- **查询文档** - 查询 Houdini 官方文档（支持 Labs 节点如 `sop/labs::splatter`）
- **创建节点** - 手动创建单个节点
- **自动建树** - AI 自动生成 ```mcp``` 代码块，工具解析后在 Houdini 中创建节点网络
  - 支持的 MCP 动作：`create_nodes`, `set_parameter`, `connect_nodes`, `delete_node`
  - 智能错误提示：使用错误动作名（如 `set_parm`）时提供友好建议

**对话管理:**
- **🔄 开启新话题** - 清空对话历史，重新开始（系统提示会自动重新添加）
- **🖵 全屏对话** - 独立窗口查看完整对话，支持 Context 消息显示
- **💾 导出对话** - 保存为 .txt 或 .md 文件，包含时间戳和元数据
- **⏰ 时间戳** - 每条消息显示发送时间

**用户体验优化:**
- **加载动画** - 旋转的圆点加载指示器，清晰的"正在请求..."提示
- **浮动图标** - 最小化到桌面浮动图标，右键菜单快速操作
- **窗口状态管理** - 系统最小化按钮自动转换为浮动图标，状态始终一致
- **无控制台输出** - 所有反馈通过 UI 显示，不污染 Houdini 控制台
- **超时重试** - DeepSeek 请求超时自动重试 2 次，提供详细错误提示
- **代理检测** - 使用 DeepSeek 时自动检测系统代理并提示可能的连接问题

**提供商与模型:**
- **OpenAI（云端）** - gpt-4o-mini / gpt-4o / gpt-4o-mini-translate（需 API Key）
- **DeepSeek（云端）**（默认） - deepseek-chat / deepseek-coder（性价比高，响应快）
- **Ollama（本地免费）** - llama3.1:8b-instruct / qwen2.5:7b-instruct / mistral:7b-instruct / phi3:mini（无需 API Key）

使用 Ollama（本地免费）
- Windows 安装：到 https://ollama.com 下载并安装，启动后默认服务端口 11434。
- 首次使用先拉取模型（示例）：
    ```powershell
    ollama pull llama3.1:8b-instruct
    # 或者
    ollama pull qwen2.5:7b-instruct
    ```
- 在工具里“提供商”选择 “Ollama（本地免费）”，选择对应模型即可对话。

**API Key 配置（二选一，用于 OpenAI/DeepSeek 云端）:**

1. **环境变量（推荐）** - 在 Windows PowerShell 中设置后重启 Houdini
   ```powershell
   # OpenAI
   [Environment]::SetEnvironmentVariable('OPENAI_API_KEY', '<你的Key>', 'User')
   # 或项目专用变量名
   [Environment]::SetEnvironmentVariable('DCC_AI_OPENAI_API_KEY', '<你的Key>', 'User')
   
   # DeepSeek
   [Environment]::SetEnvironmentVariable('DEEPSEEK_API_KEY', '<你的Key>', 'User')
   # 或
   [Environment]::SetEnvironmentVariable('DCC_AI_DEEPSEEK_API_KEY', '<你的Key>', 'User')
   ```

2. **工具内设置** - 点击"设置 API Key…"按钮
   - 输入对应提供商的 API Key
   - 勾选"保存到本机配置"可将密钥保存到 `config/houdini_ai.ini`（仅本机，已被 .gitignore 忽略）
   - 或仅在当前会话使用（不保存到文件）

3. **Ollama 本地服务**（可选）
   - 默认连接 `http://127.0.0.1:11434`
   - 可在工具内或 `config/houdini_ai.ini` 中自定义：`ollama_base_url:http://your-ip:11434`

**性能优化与使用建议:**
- ⚡ **快速模式（默认）** - 512 tokens/30s，适合快速问答和简单节点创建
- 🔓 **标准模式** - 2048 tokens/120s，适合复杂节点网络生成和详细解答
- 🚀 **最快速度** - 使用 Ollama 本地模型，响应时间 1-3 秒（需提前安装）
- 💡 **MCP 使用技巧** - 开启新话题后，明确告诉 AI"请使用 MCP 指令创建节点"效果更好

**常见错误与排查:**
- **401 认证失败** - 检查 API Key 是否正确、是否有权限访问对应模型
- **429/insufficient_quota** - 配额不足，前往平台控制台查看套餐与账单
- **代理连接失败** - 如果看到黄色警告横幅，说明检测到系统代理，建议清除或使用 Ollama
- **SSL 协议错误** (`EOF occurred in violation of protocol`)
  - **原因** - Houdini 内置 Python 的 OpenSSL 版本过旧
  - **解决方案** - 安装 `requests` 库（更好的 SSL 兼容性）
    ```powershell
    "C:\Program Files\Side Effects Software\Houdini 21.0.000\python39\python.exe" -m pip install requests
    ```
- **JSON 解析失败/生成中断**
  - 可能是响应超时或内容过长被截断
  - 建议简化请求或开启快速模式
  - 或分多次创建（先创建几个节点，再创建其他节点）
- **AI 不使用 MCP 指令**
  - 开启新话题后明确要求："请使用 MCP 指令创建节点"
  - 系统提示词会自动重新添加，但 AI 需要明确指示

**安全提示:**
- 不要将 API Key 明文提交到代码库
- 优先使用环境变量，或使用本机 INI 保存（已被 `.gitignore` 忽略）

## Maya 工具（Suichiku 的 Maya 工具箱）

窗口包含两个标签页：

1) 资产检查（带配置 UI）
- 不再使用 CSV，直接在工具中配置检查项并保存至 `config/maya_asset_check.ini`
- 可配置检查项示例：
    - Maya 单位检查（可开关）
    - UV 集数量/命名检查
    - 多边形面数上限检查
- 运行后显示问题明细（按选择或全场景）

2) 模型导出
- 支持格式：FBX、Alembic（.abc）、OBJ
- 命名规则：`SC_资产类型_资产名称_编号`（已从 ZGAME 前缀改为 SC）
- 枢轴/归零工具：
    - “一键归到世界中心”：将选择对象移动使其枢轴在世界 (0,0,0)
    - “枢轴位置”下拉与“应用枢轴”：按几何中心/八角等快速设置枢轴
    - “刷新枢轴状态”按钮：手动重新评估当前选择是否已归零
- 导出门槛：只有当“当前选择的对象枢轴已归零”时，导出按钮才会可用
- 额外提醒（非阻断）：如果当前选择已归零，但场景内仍存在其他网格对象未归零，会在工具界面顶部显示显眼的黄色告警横幅，同时不阻碍本次导出

Maya 中快速运行脚本（可做成 Shelf 按钮）：
```python
import sys
tool_path = r"C:\Users\KazamaSuichiku\Desktop\DCC-ASSET-MANAGER"
if tool_path not in sys.path:
        sys.path.insert(0, tool_path)
import launcher
launcher.show_tool()
```

插件依赖（按需）：
- FBX：`fbxmaya` 插件
- Alembic：`AbcExport` 插件
- OBJ：Maya 内置导出类型 `OBJexport`

选择兼容性：
- 工具对 transform/shape/组件选择做了统一处理，无论是选择物体、形状还是面/点/边组件，都会推导为对应的 transform 进行操作。

## 统一的配置与缓存

- 配置目录：`<repo>/config/`
    - Maya 检查配置：`maya_asset_check.ini`
    - Houdini 配置与历史：`houdini_*.ini` 等（已从旧路径迁移）
- 缓存目录：`<repo>/cache/`
    - 例如：Houdini 生成的快照

相关路径与创建逻辑由 `shared/common_utils.py` 统一管理。

## 依赖与环境

- Python 3.9+（推荐）
- PySide6（UI）
- Houdini 18.0+ / Maya 2020+（按需）

提示：工具内对 `maya.cmds` 和 `hou` 使用了“动态导入”，在非对应 DCC 环境下也能浏览代码，不会报错。

## 疑难排查

- 无法启动窗口：确保将仓库根路径加入 `sys.path` 后再 `import launcher`；也可用绝对路径。
- Maya 导出按钮不可用：请先确保当前选择的对象枢轴在世界原点（使用“一键归到世界中心”或设置枢轴后点击“刷新枢轴状态”）。
- 导出失败：检查插件是否加载成功（FBX: `fbxmaya`，Alembic: `AbcExport`），并查看 Maya Script Editor 输出。
- Houdini 快照未显示：确认仓库根目录下的 `cache/` 是否存在（工具会在需要时自动创建）。

## 版本历史

- v2.1 — 集成 Maya 工具箱（资产检查 UI、模型导出、显眼场景告警）、统一 config/cache 路径、Houdini 快照迁移至 cache/
- v2.0 — 重构为多 DCC 支持架构
- v1.0 — Houdini HIP Manager 初始版本

## 作者

KazamaSuichiku

## 许可证

内部工具，仅供项目使用

---

## Maya 功能详解（功能说明）

- 资产检查
    - 存储：`config/maya_asset_check.ini`
    - 配置项：
        - enable_unit_check：是否检查单位为米（m）
        - checks（JSON）：
            - uv_sets：限制 UV 集数量（仅匹配“目标后缀”的对象）
            - poly_faces：限制多边形面数（仅匹配“目标后缀”的对象）
    - 实用功能：一键改为米（m）、批量添加前缀/后缀、帮助弹窗

- 模型导出
    - 仅导出选中对象；未选择不允许导出
    - 枢轴需要在 (0,0,0)；提供“一键归到世界中心”“应用枢轴”“刷新枢轴状态”
    - 合并/分别导出开关：
        - 勾选：将所有选中对象导出为一个文件
        - 取消：分别为每个对象导出文件，文件名追加 `__对象名`
    - 支持格式：FBX / Alembic(.abc) / OBJ
    - 命名规则：`SC_资产类型_资产名称_编号.扩展名`
    - 警告：若仅所选归零但场景有其他未归零网格，会显示黄色横幅（不阻断）

---

## 导出规则与行为（功能说明）

- 命名与多文件
    - 统一命名：`SC_资产类型_资产名称_编号.扩展名`
    - 分别导出时：自动追加 `__对象名`

- 支持的导出插件（Maya）
    - FBX：`fbxmaya`
    - Alembic：`AbcExport`
    - OBJ：`OBJexport`

- 选择与归零
    - 仅导出选中对象（transform/shape/组件均会解析为 transform）
    - 选中对象的枢轴需在 (0,0,0)

---

## 共享模块（配置方法 + 功能说明）

- `shared/common_utils.py`
    - 获取仓库根路径
    - 统一创建/使用 `<repo>/config` 与 `<repo>/cache`
    - INI 配置的 load/save（支持在 INI 中存 JSON 字段）

- `shared/p4v_utils.py`
    - P4V 路径查找：常见安装路径、注册表、PATH、`P4V_HOME`
    - 运行状态检查：基于 `tasklist`
    - 一键启动：Maya/Houdini 共用

---

## 作者

KazamaSuichiku

## 许可证

内部工具，仅供项目使用
