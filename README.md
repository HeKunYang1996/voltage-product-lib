# Voltage Product Library

产品库定义设备分类、P/M/A 点位、默认展示数据及产品级连线限制。
当前保留 16 个产品。全部设备使用同一种节点形态，不再定义内部组件或容器。

## 产品分类

| type | 设备含义 | 产品 |
| --- | --- | --- |
| Station | 站点 | Station |
| Environment | 环境监测 | Env |
| Battery | 电池 | Battery |
| PCS | 储能变流器 | Single_Phase_PCS、Three_Phase_PCS |
| AC_Inverter | 光伏逆变器 | Single_Phase_AC_Inverter、Three_Phase_AC_Inverter |
| Hybrid_Inverter | 混合逆变器 | Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter |
| Diesel | 柴油发电机 | Single_Phase_Diesel、Three_Phase_Diesel |
| PV | 光伏组 | PV_Group |
| Meter | 电表 | Single_Phase_Meter、Three_Phase_Meter |
| Load | 负载 | Single_Phase_Load、Three_Phase_Load |

`type` 表示具体设备类型，用于产品分组和类型识别。单相、三相版本共用类型，相制由具体产品标识及定义区分。类型标识沿用英文，中文含义见上表。产品顶层的 `type` 与 P/M/A 点位的值类型 `type` 是不同层级的字段。
`PV_Group` 的 `type` 和 `displayName` 均为 `PV`；稳定产品标识 `name`、文件名 `PV_Group.json` 和拓扑引用继续使用 `PV_Group`。
分类值不要求存在同名产品文件。此前 Battery/PCS 共用的 ESS、Diesel/PV_Group 共用的 Generator、两类逆变器共用的 Inverter 已细化为上表类型。分类占位产品及 Distribution_Board 仍保持移除。
产品级连线继续按具体产品 name 校验，类型不定义连接端口或功率方向。接入或消费端若按旧分类值分支，应按此分类表同步核对。
Single_Phase_Meter 和 Three_Phase_Meter 分别用于单相、三相 AC 计量，共用整机功率、频率、功率因数及双向累计电量名称；按现场实际接线选择产品。两者均无控制动作。
Single_Phase_AC_Inverter、Three_Phase_AC_Inverter 和 Three_Phase_PCS、Single_Phase_PCS 仍用于独立设备；Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter 以一个实例表达两类功能。

**通用模板对齐（2026-11-25）**：AC Inverter、PCS 与 Hybrid 按测量名称和分层对齐。AC Inverter 覆盖 PV DC 输入和 AC 转换侧，PCS 覆盖 AC 与电池侧。P、A 各自从 1 连续编号；M 按基础、电气和周期区段编号。AC Inverter 的基础段为 M1–M2、电气段为 M101–M104、PV 周期段为 M209–M213；PCS 的基础段为 M1–M3、电气段为 M101–M107、充放电周期段为 M201–M210；Hybrid 的基础段为 M1–M3、电气段为 M101–M111、周期段为 M201–M220。各产品的同含义点允许使用不同 ID，具体编号以产品 JSON 为准。这三类通用模板暂不包含分相测量，Meter、Load、Diesel 保留各自已有分相点表。

单相、三相负载为通用用电监测模板，仅保留电气配置和重点测量，不定义负载调度参数或控制动作；用电量可映射覆盖该负载的电表输入计数器。
发电机提供 Single_Phase_Diesel 和 Three_Phase_Diesel 两种模板。两版共用机组状态、总有功、累计发电量、周期电量和动作名称及 ID；单相版使用 AC Voltage、AC Current，三相版使用 L1/L2/L3 分相测量。120/240 V 裂相机型选单相模板，并在实例映射中明确电压参考端和电流对应的相线。三相模板示例见下文「产品说明示例」。

## 交流产品相制覆盖

交流类产品均提供单相和三相模板，按实际设备及接线选择。相制影响 AC 端口及分相测量，同一类产品的共用测点按名称和计量口径对齐。

| 类别 | 单相 | 三相 |
| --- | --- | --- |
| 光伏逆变器 | Single_Phase_AC_Inverter | Three_Phase_AC_Inverter |
| 混合逆变器 | Single_Phase_Hybrid_Inverter | Three_Phase_Hybrid_Inverter |
| PCS | Single_Phase_PCS | Three_Phase_PCS |
| 柴油发电机 | Single_Phase_Diesel | Three_Phase_Diesel |
| 电表 | Single_Phase_Meter | Three_Phase_Meter |
| 负载 | Single_Phase_Load | Three_Phase_Load |

Battery、PV_Group 定义 DC 侧测量；Env、Station 定义环境和站点信息，这四类保留各自通用产品。

PV_Group 新增 M103 DC Voltage（V）、M104 DC Current（A），可由对应 MPPT 输入映射或规则链生成。实例应记录输入来源、测量边界和计算方法；不同 MPPT 的电压不可相加，电流仅在确认同电压边界并联关系后才可求和，不直接合并独立 MPPT 电流。未配置有效来源或规则时保持缺失值。M101 DC Power、M102 PV Energy Total 及 M201–M205 周期电量的 ID 和累计源保持不变；新增两点不增加默认重点展示标记。PV 不保留 Start Stop Status（原 M1）：当前 Sol-Ark SQL/只读协议没有独立 PV 启停状态，零功率不能直接解释为停机。剩余点位 ID 保持不变。

### 当前交流模板核对表（2026-11-25）

点数按 P/M/A 顺序列出。每对产品的共用测点使用相同名称、ID、单位、枚举与展示标记；不同产品类别按各自计量边界独立编号。相制差异保留在额定电压、电流说明及适用的分相测量中。

| 产品类别 | 单相 P/M/A | 三相 P/M/A | 周期电量 ID（含本年） |
| --- | --- | --- | --- |
| AC_Inverter | 5/11/3 | 5/11/3 | M209–M213 |
| Hybrid_Inverter | 8/34/4 | 8/34/4 | M201–M220 |
| PCS | 6/20/4 | 6/20/4 | M201–M210 |
| Diesel | 6/14/4 | 6/21/4 | M209–M213 |
| Meter | 4/13/0 | 4/20/0 | M201–M205 |
| Load | 4/12/0 | 4/19/0 | M201–M205 |

AC Inverter、PCS、Hybrid 当前采用整机重点测量，单相与三相模板可有相同的 M 点表。Meter、Load、Diesel 的三相模板额外提供 L1/L2/L3 测量。单相模板的整机有功仅用一个 AC Power 点表示。

默认重点展示须同时满足 isDisplay 为字符串 "true" 且 monitorVisible 不为字符串 "false"。全部周期电量（含日和年）在监控中隐藏；三相电压、电流、有功分别使用 AC Voltage、AC Current、AC Power 展示分组。

## 字段约定

| 字段 | 用途 |
| --- | --- |
| name | 稳定产品标识，实例及连接规则按此字段关联 |
| displayName | 界面展示名称；一般由 `name` 去掉下划线得到，不改动 `name` 本身 |
| type | 具体设备类型，单相/三相版本共用 |
| description | 产品说明 |
| topology | 可选；仅参与拓扑绘制的产品定义此对象 |
| topology.image | 可选图片资源标识 |
| topology.connections | 产品级连线规则数组 |
| topology.description | 整体连接说明，用于 tooltip 底部 |
| M[].attr.isDisplay | 可选的默认重点展示标记；字符串值为 `"true"` 时选入，监控中同时受 monitorVisible 约束 |
| M[].attr.monitorVisible | 可选的监控可见性标记；字符串值为 `"false"` 时在监控数据中隐藏，未配置时默认可见 |
| M[].attr.displayGroup | 可选的展示分组；三相电压、电流、有功分别使用 AC Voltage、AC Current、AC Power |
| P / M / A | 参数、测量及动作定义 |

点位使用 `id/name/unit/type`，并可附带 `description` 等元数据。点位类型支持 `number`、`string` 和 `enum`。
每个产品的 P、M、A 点表独立编号，表内 ID 唯一，通常按数值升序排列；Battery 将当前现场未映射的测点按原 ID 放到 M 数组末尾。基础点从 1 开始，当前整理的通用 AC Inverter、PCS、Hybrid 各段内顺序编号；周期固定段及其他产品现有空档按各自 JSON 保留，不为跨产品对齐而新增占位点。M1–M100 放置状态、温度等基础点；M101–M200 放置 AC/DC 电气测量和累计电量；M201–M300 放置周期电量与周期最大需量。同含义测点统一名称、业务单位和计量口径，各产品按自身点表独立编号，不要求跨产品 ID 相同；同一产品的点位名称须唯一。Battery 的日、周、月、季度充放电量使用 M201–M208。空点表保持为空。已有实例升级到重新编号的点表时，按变更记录迁移点位引用。协议地址在接入层维护。
默认展示信息直接放在测点上：需要展示的 M 点位增加 `"attr": { "isDisplay": "true" }`，未配置或值不为 `"true"` 的点位不默认展示。展示顺序遵循 M 数组顺序；如果以后需要独立排序，可在 `attr` 中增加排序元数据。每个产品标记 2–3 个关键 M 点位。离网设备主要展示功率、电量或设备特有的关键测量，Station 展示现有两个测点。可启停类设备在 M1–M100 段提供 **Start Stop Status**（Stopped/Running）；**PV_Group**、**Meter** 与 **Load** 无 **A** 控制、也无启停 **M** 点。设备类模板默认不突出展示启停状态（`Start Stop Status`）；Station 是业务站点状态展示的例外，保留 M1 的默认重点标记，含义以显式业务状态源为准。已移除 **Grid Status**。
展示时过滤未映射或不可用点位，不以零值代替缺失值。
监控可见性优先于默认重点展示：`attr.monitorVisible === "false"` 的测点不进入监控卡片、默认重点卡片或监控展示分组；`isDisplay` 保留其原有默认重点展示含义。日、周、月、季度、本年度电量均标记为监控隐藏，点位定义、规则计算、历史查询和统计用途保留。未配置 `monitorVisible` 的测点默认可见。消费端需读取并应用此字段后才会改变页面效果。
三相测点保留 L1/L2/L3 名称，仅通过 `attr.displayGroup` 标记共同展示关系；组内顺序遵循 M 数组中的 L1、L2、L3 顺序。该字段只描述展示分组，不计算合计、不改变点位值，也不替代默认展示标记。
单相和三相 PCS 的 `isDisplay` 仍标记 SOC（M1）、Battery Power（M102）、Daily Discharge Energy（M202）；其中日电量被 `monitorVisible: "false"` 隐藏，监控页实际可见的默认重点点位为 SOC 和 Battery Power。

### 周期电量统一口径

全库 115 个周期电量点均为当前周期起点至当前时刻的累计差量，业务单位统一为 kWh，从同一产品中说明所指的累计源 M 点计算。各累计源均对应日、周、月、季度、本年度五种周期。周期点不表示上一完整周期，不按瞬时功率直接积分，也不把设备启动后的会话电量直接当成当日电量。

| 周期 | 起点（按配置的统计时区） |
| --- | --- |
| Daily | 当日 00:00 |
| Weekly | 当前日历周的配置起点；实例与规则须使用同一周起始设置 |
| Monthly | 当月 1 日 00:00 |
| Quarterly | 当前日历季度起点，1/4/7/10 月 1 日 00:00 |
| Yearly | 当年 1 月 1 日 00:00 |

累计源先完成单位和方向映射，周期差量直接继承 kWh，不再次缩放。充电、放电、PV、负载、发电和电表输入电量分别使用各自累计源，不混用 AC/DC 边界，不叠加同一电池在 Battery 与 PCS/Hybrid 中重复测量的电量。电表 Import/Export 以安装方向为准，并不自动表示购电/售电。

所有周期点均需配置计算规则和有效周期基线。缺少基线、源计数器复位或更换计数源时，结果保持无有效值，重新确认有效基线后才恢复计算，不将负差量、缺失值或会话值当成周期电量。全部日、周、月、季度和本年度点均为监控隐藏。以上为产品契约，当前年度计算仍待适配，并不代表运行服务已实现所有规则。

### 本年度电量

Yearly 表示本日历年从 1 月 1 日起到当前的累计电量，跨年重新建立年度基线。各产品在现有周期段末尾追加年度点，原有点位 ID 保留。年度点统一使用 monitorVisible 为字符串 "false"，用于统计和历史查询。

年度电量沿用同边界累计计数器；计算服务需支持年度周期、明确统计时区并具有有效年初基线。当前 Period Delta 仅支持日、周、月、季度，年度计算尚待适配。配置时区应与年度计算逻辑一致，基线或数据缺失时保持无有效值。

### 枚举点位

P、M、A 中有固定选项的点位统一使用 `type: "enum"` 和 `enumData`。数组顺序即展示顺序。不再使用 `options`。

```json
{
  "id": 9,
  "name": "Charging Status",
  "unit": "",
  "type": "enum",
  "enumData": [
    { "key": "1", "label": "Charging", "value": "", "description": "Energy is flowing into the battery during charging." },
    { "key": "2", "label": "Idle", "value": "", "description": "The source reports an idle charging state." },
    { "key": "3", "label": "Completed", "value": "", "description": "The source reports that charging has completed." }
  ]
}
```

- `key` 是点位内唯一的选项编号，用字符串存储。新选项从 `"1"` 起编号，调整展示顺序时保留已有编号。
- `label` 是选项的默认展示名称。
- `value` 在产品模板中留空字符串；实例配置或点位路由阶段填写实际取值。
- `description` 是可选的英文业务说明，用于帮助 AI 客服理解选项含义。当前模板的枚举选项均填写该字段；动作选项描述请求意图和确认反馈，状态选项描述明确的上报状态。结合点位说明、实例映射、当前读数和时间戳解释实际设备状态。
- M 枚举用配置的实际值匹配设备上报。A 枚举用配置的实际值向当前实例的动作路由下发，操作界面只提供已配置有效下发值和路由的选项。

启动和停止分别保留原有 A 点位和独立路由，每个点位提供一个枚举选项。下面是启动动作的定义节选：

```json
{
  "id": 1,
  "name": "Start",
  "type": "enum",
  "enumData": [
    { "key": "1", "label": "Start", "value": "", "description": "Request the device to start. Confirm the outcome using its start/stop status." }
  ]
}
```

停止动作也可以用 `"1"` 作为选项编号，因为编号在各自的点位内有效。实际下发值和寄存器地址按实例配置。

柴发启停状态与动作的完整写法见 `Three_Phase_Diesel.json` 中的 **M3 Start Stop Status**、**A1 Start**、**A2 Stop**（与上表相同的 `enumData` 结构）。

## 四个连接点共享产品规则

上、下、左、右四个画布连接点等效，不在产品中绑定业务角色或固定方位。
拓扑边可继续保存 sourceHandle/targetHandle 以恢复绘图位置；切换连接方位不改变业务关系和计数。

示例：Battery 的规则。

```json
{
  "connections": [
    {
      "products": ["Single_Phase_Hybrid_Inverter", "Three_Phase_Hybrid_Inverter", "Three_Phase_PCS", "Single_Phase_PCS"],
      "min": 1,
      "max": null
    }
  ],
  "description": "Connect to one or more single-phase or three-phase hybrid inverters and/or PCS devices across this group, with no product-level maximum."
}
```

| 规则字段 | 含义 |
| --- | --- |
| products | 允许连接的具体产品 name；不按 type 分类匹配 |
| min | 发布时，该规则分组至少连接的对端设备数量 |
| max | 该规则分组最多连接的对端设备数量；null 表示没有产品级上限 |

单条 `connections` 规则只描述允许的对端产品和数量限制；不要添加 `description`。`topology.description` 是整组连接规则的业务说明，前端将其显示在该产品连接规则 tooltip 的底部。

未定义 `topology` 的产品不参与拓扑编辑器和拓扑画布；Station 与 Env 属于此类产品，仍可在设备管理和监控中使用。

### 校验语义

1. 双方产品都必须有允许对方的规则；未声明即不允许。未定义 topology 的产品不参与拓扑。
2. 四个连接点共同计数。同一条规则中不同 products 的对端数量合计，不是分别计算限额。
3. 一个对端产品只能出现在本产品的一条规则中，避免重复计数或不明确的优先级。
4. 禁止自环和同一对节点的重复边，包括从不同方位重复连线；当前库不声明同产品之间连线。
5. 新增连线时分别检查两端规则及 max。删除、换向后重新计算受影响节点的连接数。
6. min 允许在编辑和草稿阶段暂时不满足，发布时统一校验。max 按不同对端节点统计。
7. 连线无业务方向，不以 source/target 推断充放电或供电方向；运行态功率另外读取点位。
8. null 不表示实际接线、并联数量或电气容量无限制；这些仍由现场设备和配置决定。

### 当前默认规则

| 产品 | 对端产品分组 | min | max |
| --- | --- | --- | --- |
| Battery | Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter、Three_Phase_PCS、Single_Phase_PCS 合计 | 1 | null |
| PV_Group | Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter、Single_Phase_AC_Inverter、Three_Phase_AC_Inverter 合计 | 1 | 1 |
| Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter | PV_Group | 0 | null |
| Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter | Battery | 0 | null |
| Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter | 两类 Meter | 1 | 1 |
| Single_Phase_AC_Inverter、Three_Phase_AC_Inverter | PV_Group | 0 | null |
| Single_Phase_AC_Inverter、Three_Phase_AC_Inverter | 两类 Meter | 1 | 1 |
| Three_Phase_PCS | Battery | 0 | null |
| Three_Phase_PCS | 两类 Meter | 1 | 1 |
| Single_Phase_PCS | Battery | 0 | null |
| Single_Phase_PCS | 两类 Meter | 1 | 1 |
| Single_Phase_Diesel、Three_Phase_Diesel | 两类 Meter | 1 | 1 |
| 两类负载 | 两类 Meter | 1 | 1 |
| 两类 Meter | Single_Phase_Hybrid_Inverter、Three_Phase_Hybrid_Inverter、Single_Phase_AC_Inverter、Three_Phase_AC_Inverter、Three_Phase_PCS、Single_Phase_PCS、Single_Phase_Diesel、Three_Phase_Diesel、两类负载合计 | 0 | null |
| Station、Env | 不参与能量连线，保留固定信息卡 | — | — |

这些是本项目的初始配置，不是厂家通用电气限制。混合逆变器的光伏与电池连接暂不设必连组合条件。
Meter 关联表示业务拓扑，不表示把计量设备当作配电箱的实际接线端子。
每站两类 Meter 合计数量、全图连通性和环路策略属于站点级校验，不在每个产品里重复定义。

### Tooltip

先由规则自动生成“允许连接的产品、最少/最多数量、当前已连接数量”，底部附加 `topology.description`。
产品文件不保存当前数量、占用方位或实例 ID；这些由当前拓扑计算。
提示文案仅用于解释，程序校验以 products/min/max 为准。

## 产品说明示例：Three_Phase_Diesel（三相柴发）

以下以当前仓库中的 `Three_Phase_Diesel.json` 为准，说明 P/M/A 分层、拓扑、默认展示与接入习惯。实例文件名为稳定标识 **Three_Phase_Diesel**；界面展示名为 **Diesel Generator (Three Phase)**；分类 **type** 为 `Diesel`。旧产品名 **Diesel** 升级时需按 `CHANGELOG.md` 中的 ID 映射整体迁移，不能只改 `product_name`。

### 拓扑与计量

- 参与能量拓扑；画布图片资源为 `device-Diesel.png`。
- **连线规则**：必须与 **Single_Phase_Meter** 或 **Three_Phase_Meter** 之一相连，且本规则分组内合计 **min=1、max=1**（发布时须满足「恰好一台电表」）。
- **对端互惠**：两类 Meter 的 `connections` 均声明允许 `Three_Phase_Diesel`；连线校验按双侧规则与共同计数执行。
- 连线仅表达业务计量关联，不表示物理配电端子；发电功率与电量以柴发实例 **M** 点或映射到电表测点为准，不从边的 source/target 推断方向。

```json
{
  "topology": {
    "image": "device-Diesel.png",
    "connections": [
      {
        "products": ["Single_Phase_Meter", "Three_Phase_Meter"],
        "min": 1,
        "max": 1
      }
    ],
    "description": "One Meter connection is required."
  }
}
```

### 配置参数 P（P1–P6）

均为 `number`，填写厂家铭牌或项目设计值；协议寄存器在接入层映射，产品库不保存地址。

| ID | 名称 | 单位 | 含义 |
| --- | --- | --- | --- |
| P1 | Max Power | kW | 主用（prime）额定有功输出 |
| P2 | Rated AC Voltage | V | 三相输出线电压额定值 |
| P3 | Rated AC Current | A | 额定工况下各相输出电流 |
| P4 | Fuel Tank Capacity | L | 油箱容积 |
| P5 | Rated Frequency | Hz | 额定输出频率 |
| P6 | Fuel Consumption Rate | L/kWh | 参考负载下的单位发电油耗 |

2026-11-24 起已删除旧版中的 Min Power、Startup Time、Min Runtime、Response Time 等 P 点；**P6** 现为 **Fuel Consumption Rate**（由原 P7 顺号），升级时勿按 ID 硬拷贝旧属性值。

### 测量点 M（分层）

点表遵循 **M1–M100 机组状态**、**M101–M200 电气量与累计电量**、**M201–M300 周期电量**；本产品的周期发电量在 **M209–M213**，依次表示日、周、月、季度和本年度电量，累计电量源点为 **M131**。

| 区段 | ID | 名称 | 单位 | 默认展示 | 说明 |
| --- | --- | --- | --- | --- | --- |
| 基础 | M1 | Fuel Level | % | 是 | 油箱油量百分比；仅映射百分比或经标定的换算 |
| 基础 | M2 | Coolant Temperature | ℃ | 否 | 冷却液温度 |
| 基础 | M3 | Start Stop Status | —（enum） | 否 | 枚举 **Stopped / Running**；有真实状态源再映射，不推断物理启停 |
| 基础 | M4 | Engine Load | % | 否 | 控制器上报负载率 |
| 电气 | M101 | AC Power | kW | 是 | 三相总有功输出 |
| 电气 | M106 | AC Frequency | Hz | 否 | 输出频率 |
| 分相 | M111–M113 | L1–L3 Voltage | V | 否 | 各相对中性点电压 |
| 分相 | M114–M116 | L1–L3 Current | A | 否 | 各相电流 |
| 分相 | M117–M119 | L1–L3 Active Power | kW | 否 | 各相有功 |
| 累计 | M131 | AC Energy Total | kWh | 否 | 机端累计发电量；周期电量计算的输入计数器 |
| 周期 | M209 | AC Daily Energy | kWh | 重点标记保留，监控隐藏 | 当日发电量，EMS 按 M131 相对日基线差分 |
| 周期 | M210–M212 | AC Weekly/Monthly/Quarterly Energy | kWh | 否 | 周/月/季发电量，算法同源 |
| 周期 | M213 | AC Yearly Energy | kWh | 否 | 本日历年电量，以 M131 为累计源；年度计算能力和年初基线仍需配置 |

**默认重点标记**（`M[].attr.isDisplay === "true"`）为 **Fuel Level、AC Power、AC Daily Energy**，共 3 项；其中 AC Daily Energy 因 `monitorVisible: "false"` 在监控中隐藏，实际显示 Fuel Level 和 AC Power；监控优先展示实时功率和油位，周期电量留给统计与历史查询。**Start Stop Status** 故意未标默认展示。

本产品的整机有功命名为 **AC Power**（M101），统计与设备详情应优先使用该名称及 **AC Energy Total / AC Daily Energy**，与旧名 **AC Output Power**、**Energy Today** 等区分时按含义迁移。

### 控制动作 A（A1–A4）

| ID | 名称 | 类型 | 说明 |
| --- | --- | --- | --- |
| A1 | Start | enum（单选项 Start） | 启动柴发；须配置可写路由 |
| A2 | Stop | enum（单选项 Stop） | 停机 |
| A3 | Power Set | number（kW） | 有功设定值（依控制器能力映射） |
| A4 | Clear Error | enum（单选项 Reset Fault） | 故障复位 |

已移除旧版 **A4 Off On Grid**；原有 Clear Error 路由需从 **A5 迁至 A4**。Start 与 Stop 仍为两个独立 A 点、两条路由，不合并为单一「启停控制」枚举点。

### 接入与映射要点

1. **分相 ID**：本产品的相电压、电流、有功依次使用 M111–M113、M114–M116、M117–M119。其他产品按各自 JSON 编号；本产品未定义线电压点。
2. **枚举**：M3、A1、A2、A4 的 `enumData[].value` 在模板中为空，实例/路由阶段填写设备实际值；模板 `key` 仅为选项编号，`enumData[].value` 才是实例实际值；导入云端枚举时将实例实际值作为云端 key，label 作为云端展示 value。
3. **周期电量**：M209–M212 可由 EMS 周期规则按 **M131** 计算日、周、月、季度差量；M213 为本年度电量，现有 Period Delta 尚无年度选项。映射 M131 后还需配置相应计算规则及有效周期基线，才能生成周期结果；所有周期电量在监控中隐藏。
4. **与电表关系**：拓扑上必须挂一台 Meter 做分支计量；机端 M101/M131 与电表 **AC Power / Import Energy** 可能数值接近但测量边界不同，统计配置应写明取柴发机端还是表计侧，避免双计叠加。
5. **展示缺失**：未映射或无效测点不展示，不以 0 代替缺失值（全库约定）。

契约全文以 `Three_Phase_Diesel.json` 中各点 `description` 为准；升级与 ID 对照见 `CHANGELOG.md` 中 **Three_Phase_Diesel** 与 **Diesel properties / Diesel and PCS actions** 小节。

## 柴油机资料核对（2026-11-25）

MQ Power DCA20SPXU4F 为 120/240 V 单相三线（裂相）机型，选择 **Single_Phase_Diesel**。Basler 支持多种相制，SQL 同时列出三相字段不能证明机组为三相。MQ 操作手册列出 ECU-754 控制器，现场是否选配 Basler 尚待核实。

Fuel Level 的产品语义为油位百分比。Basler 功能说明明确采用 %，但 SQL 对应的原生寄存器 44806 行单位为 N/A，需对照现场显示和标定确认。单相电压、电流按实际 A-B/A-C 接法选择参考端和相线；当前一个 AC Current 点不能同时表达裂相两腿的不平衡。

Basler 手册提供远程启停和报警复位接口，当前 SQL 尚无控制/调节路由。Base Load Level 为特定运行模式下的额定功率百分比设定，离网 Power Set 能力仍待确认。寄存器、条件映射、手册页码与当前待确认清单见 [柴油机与 Basler 接入核对](Diesel_Source_Verification.md)。

## 通用产品与设备接入边界

产品库约定展示、统计和 AI 解释所用的业务口径。厂家型号及固件的协议字段、单位、正负方向、枚举编码和可写能力在接入配置中核实。同类产品允许只映射实际支持的测点；模板列出的测点与动作应分别核实来源和能力。无法确认或无法支持的项保持未配置，界面按缺失值处理。

### 功率、电流与动作方向

- Battery Power、Battery Current 以及 Battery Power Set 的平台目标口径统一为充电正、放电负，适用于 Battery、PCS 和 Hybrid 的电池 DC 端口。Battery Current 的点位描述也已写明此约定。累计 Charge Energy 和 Discharge Energy 仍分别记录两个方向的电量。
- LYNK Serial CAN 的电流是充电正、放电负，与电池侧目标口径一致，无需为方向额外取反。其他厂家原始值可能采用相反方向；Sol-Ark 当前提供的电池功率、电流寄存器未明确说明正负方向，须按对应版本协议或实测确认后映射。
- 取反只适用于同一测量对象、同一计量边界且已确认相反符号的值。单位换算、倍率、CT 安装方向、无效值需要分别核实。多端口或多个设备的值不能仅因名称相近就相互替代。
- PCS 和 Hybrid 的 AC Power、Power Set 均以流入设备为正、流出设备为负。AC Inverter、Diesel 的发电输出，以及 Meter、Load 的安装方向或消费口径，继续以各产品点位说明为准；不能把一个产品的符号约定直接套到其他产品。AI 客服应同时读取产品点位说明及实例映射。
- PCS 从 AC 侧取电为电池充电时，AC Power 和 Battery Power 通常均为正；电池经 PCS 向 AC 侧放电时，两者通常均为负。Hybrid 可以同时由 PV 给电池充电并向 AC 负载供电，因此 AC Power 可以为负而 Battery Power 为正；以 Battery Power 判断电池充放电，不从 AC Power 单独推断。
- A 点的下发方向独立核实。平台目标值必须转换成厂家命令约定的符号、单位和编码；读取接口的方向不能作为写入接口方向的依据。当前提供的 Sol-Ark Modbus 协议只读，模板中的 A 点仍需有经确认的其他可写接口才可配置使用。

| PCS / Hybrid 点位 | 正值 | 负值 |
| --- | --- | --- |
| AC Power / Power Set | AC 系统向设备输入功率 | 设备向 AC 系统输出功率 |
| Battery Power / Battery Current / Battery Power Set | 电池充电 | 电池放电 |

以上为平台口径。当前 Sol-Ark 材料明确了 Grid、GEN 各自的符号，但寄存器 175 的逆变转换 AC 功率及 190/191 的电池功率、电流仍需核实原始正负方向；这些端口不能互相替代。SQL 的采集读数和控制路由应分别确认转换。

### 实例映射需记录的信息

记录实际设备型号与固件、原始通道及点位、测量端口、原始单位和倍率、正负号或枚举含义、平台目标含义，以及读写能力。周期统计还需记录累计源、时区、基线及数据完整性。来源不足时保留待确认状态，不能填零或假定设备已经执行动作。

## 消费端适配

当前文件是改造后的产品契约；本次仅修改产品库，没有修改后端、编辑器或运行环境。

- 删除 pName、canCreateInstance、topology.enabled、topology.type、components 和 connectableProducts。
- 旧 Rust 解析器仍要求部分已删除字段，新文件不能直接替换部署，需先更新解析类型和产品转换逻辑。
- 旧连接代码采用单侧声明即可连接；新规则要求双方允许并按分组共同计数，需同步实现。
- PointDef 与云端导入模型需要接收 description；边端转换时应透传，不能继续固定返回 None。
- 默认展示点位改由 `M[].attr.isDisplay` 标记；`defaultDisplayMeasureIds` 与旧主指标/详情结构均不再使用。
- 枚举消费端需读取并透传 `enumData`。转换到云端物模型时，使用人工配置的实际值作为 `valueType.extra.enumData[].key`，使用产品选项的 `label` 作为对应的 `value`。
- 旧拓扑迁移须明确处理 Distribution_Board 和其 Meter 子组件；不要仅替换产品名称便直接发布。
- 内嵌产品需要更新依赖并重新构建；外部产品目录需按后端实际加载机制刷新。


## 产品业务单位与 SQL 采集单位（2026-11-25）

产品库统一采用业务单位：有功功率 kW、无功功率 kvar、电量 kWh、温度 ℃，电表与负载沿用同一口径。SQL 保留设备采集工程单位及已有 scale/offset；接入映射负责将其转换为产品单位。例如 W/var/Wh 除以 1000，华氏温度按 (F − 32) × 5/9 换算。摄氏温度 degC 与 ℃ 物理单位相同，无需数值缩放。周期差量继承业务累计源的单位，不重复换算。P、M、A 各自按其定义确认输入输出单位，控制下发需要按可写协议反向换算。具体来源、转换和待确认项见 [SQL 采集单位核对](SQL_Unit_Verification.md)。

### Sol-Ark PV 输入采集范围

当前 init_site_devices.sql 的 channel 1 已配置 PV1/PV2 电压、电流和输入功率读点：地址分别为 109/110/186、111/112/187。电压、电流由 SQL 按 0.1 转为 V/A，功率源为 W，映射到 PV 的 DC Power 时转换为 kW。PV3 的 113/114/188 在当前 SQL 和对应 V1.5 协议中注明仅适用于 15K，不能据这些定义认定 18K 支持第三路读取。

以上是逆变器协议的 PV 输入采集定义，没有在产品库中新增独立 MPPT 产品，也不证明运行实例已完成映射或现场通信。具体 PV 输入与物理 MPPT、组串的对应关系须按型号和接线确认；可将对应输入读数映射到 PV 的 DC Voltage/DC Current/DC Power 或配置规则计算。
