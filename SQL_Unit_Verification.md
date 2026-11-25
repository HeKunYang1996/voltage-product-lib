# SQL 采集单位与产品业务单位核对（2026-11-25）

当前约定为产品库统一业务单位，SQL 保留设备采集单位。此约定取代此前直接以 SQL 单位修改产品点表的决定。接入映射可配置缩放和偏移，不能直接把采集单位当作产品单位。

| 产品/点位 | SQL 来源 | SQL 工程单位 | 产品单位 | 映射换算 |
| --- | --- | --- | --- | --- |
| 两版 Diesel / Coolant Temperature | channel 3 T26，4799 | degF | ℃ | (F − 32) × 5/9 |
| 两版 Diesel / AC Power、分相有功 | channel 3 T18、T15–17 | kW | kW | 数值不变 |
| 两版 Diesel / AC Energy Total | channel 3 T38 | kWh | kWh | 数值不变 |
| 两版 Hybrid / AC Power | channel 1 T56 | W | kW | ÷1000 |
| 两版 Hybrid / DC Input Power | channel 1 T66–68，有效 MPPT 输入功率合计 | W | kW | 合计后 ÷1000 |
| 两版 Hybrid / Battery Power、Load Power | channel 1 T69、T59 | W | kW | ÷1000 |
| 两版 Hybrid / Heat Sink Temperature | channel 1 T22 | degC | ℃ | 数值不变 |
| Battery / Avg Battery Temperature | channel 2 T11 | degC | ℃ | 数值不变 |
| 两版 Meter / 总、分相有功 | channel 4 T12、T16–18 | W | kW | ÷1000 |
| 两版 Meter / AC Reactive Power | channel 4 T13 | var | kvar | ÷1000 |
| 两版 Meter / AC Import/Export Energy Total | channel 4 T31、T34，方向按安装确认 | Wh | kWh | ÷1000 |

Hybrid 的累计电量来源为 kWh，产品保持 kWh。V、A、Hz、% 等已一致的单位保持不变。同一产品的单相/三相版本共用点采用一致业务单位。全部周期电量采用 kWh，从已转换的业务累计源计算差量，不再除以 1000。

## 缩放与偏移

上述转换作用于 SQL 已应用 scale/offset 后的工程值；设备原始整数倍率与业务单位换算不能重复应用。如果配置直接从原始寄存器值生成产品值，应将原始倍率与单位换算合并。

例如 SQL 已将原始温度解释为华氏度后，产品转换可用 scale=5/9、offset=−160/9（按 y=x×scale+offset），32°F 对应 0℃、212°F 对应 100℃。若配置端仅支持乘法倍率，温度还需支持偏移或通过计算规则换算，不能只乘 5/9。

控制下发按 A 点业务单位和设备可写单位反向换算，不能直接沿用测量倍率；P 铭牌规格继续按各自定义填写。本次没有更改任何控制值或路由配置。

## 待确认的来源及能力

- Diesel Fuel Level 的 SQL 单位为空，产品保持 %，现场读值、功能启用和标定仍需核实，详见 Diesel_Source_Verification.md。
- SQL 没有专用 AC Inverter、PCS、Env、PV_Group 或通用 Load 通道，实际绑定与测量边界仍待确认。它们继续使用产品业务单位。Load 若取 Phoenix 的 W/var/Wh，应转成 kW/kvar/kWh；若取 Sol-Ark 的 W/kWh，分别除以 1000 和保持原值。
- LYNK 通道没有 Battery Power 直接读点。若确认使用同一电池边界的 Sol-Ark T69，应将 W 转成 kW；累计充放电量也须先确认来源。
- 包温极值、逐电芯测量等缺来源点继续待确认，不能用平均电池温度直接替代。
- 来源单位换算不等于正负方向转换，原始功率方向、CT 方向、无效值和安装边界仍按既定核对文档处理。
- Basler 实际安装、A-B/A-C 接线和控制能力仍待现场确认。SQL 没有对应控制/调节点插入定义，不能认定 A 点已经可下发。

本次恢复 34 个测量点的统一业务单位，并同步产品库文档。ID、名称、枚举和展示标记保持原值，SQL、运行实例、前端及服务未改。部署时须核对实际映射，避免遗漏或重复换算。
