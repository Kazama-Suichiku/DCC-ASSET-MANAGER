# DCC Asset Manager

跨 DCC 的资产管理工具，现已同时支持 Houdini 与 Maya，提供统一的配置与缓存目录，统一的启动入口与一致的用户体验。

## 项目结构

```
DCC-ASSET-MANAGER/
├── launcher.py                  # 统一启动器（自动检测 DCC 并启动对应工具）
├── shared/                      # 共享模块
│   ├── common_utils.py          # 仓库根/配置/缓存 路径工具
│   └── p4v_utils.py             # P4V 查找/启动（Maya/Houdini 共用）
├── HOUDINI_HIP_MANAGER/         # Houdini 工具
│   ├── main.py
│   ├── core/
│   ├── ui/
│   └── utils/
└── MAYA_MANAGER/                # Maya 工具（Suichiku 的 Maya 工具箱）
    ├── maya_main.py             # Maya 入口
    ├── core/
    ├── ui/
    └── utils/

# 运行后会在仓库下生成/使用：
config/   # 统一配置目录（各 DCC 的配置文件都在这里）
cache/    # 统一缓存目录（如 Houdini 快照等）
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

- 环境要求
    - Windows（提供 P4V Windows 查找逻辑）
    - Python 3.9+
    - PySide6（UI）
    - Houdini 21/20.5 / Maya 2025+（按需）

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

## 版本历史

- v2.2
    - 导出：仅导出所选对象；新增“合并导出为一个文件”开关与“导出帮助”说明
    - 资产检查：UV 集与面数检查支持“目标后缀”过滤；新增批量前后缀重命名与“一键米（m）”
    - 共享：合并 Maya/Houdini 的 P4V 工具到 `shared/p4v_utils.py`
- v2.1
    - 集成 Maya 工具箱（资产检查 UI、模型导出、显眼场景告警）
    - 统一 config/cache 路径，Houdini 快照迁移至 cache/
- v2.0
    - 重构为多 DCC 支持架构
- v1.0
    - Houdini HIP Manager 初始版本

