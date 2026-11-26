# Product Library Change Log

Version identifiers use the release date in `YYYY-MM-DD` format. New entries are added at the top and describe changes relative to the preceding recorded version.

Current contract: product power uses kW/kvar, energy and all period deltas use kWh, and temperature uses ℃. SQL units are converted in instance mappings. Periods are current calendar periods in the configured reporting time zone, with valid cumulative-source baselines; yearly calculation remains pending. PCS/Hybrid AC power is input-positive and battery power/current is charge-positive. Entries below record version-specific history; superseded entries and old ID tables are migration evidence, not current configuration. Current JSON and README define the active contract.

## 2026-11-26

### Remove unsupported PV start/stop status

- Removed PV_Group M1 Start Stop Status because the current Sol-Ark SQL and supplied read-only map expose no explicit PV start/stop state. Zero PV power must not be treated as a confirmed stop state.
- Preserved all remaining point IDs, units, period sources, display flags and topology. Updated README with the PV1/PV2 input addresses and the PV3 15K-only restriction from the supplied map; no independent MPPT product or runtime route was added.

### Add PV DC voltage and current measurements

- Added PV_Group M103 DC Voltage (V) and M104 DC Current (A), supplied by an explicitly mapped MPPT input or configured EMS rule. Documented input boundaries and limits on combining independent MPPT readings.
- Preserved existing point IDs, cumulative/period sources, display metadata, product identity and topology. New points have no default-display markers. Updated README; no runtime rules, routes or consumer code were changed.

### Use PV as the type and display name

- Changed PV_Group type and displayName to PV. Supersedes the PV_Group type introduced in the classification refinement below.
- Kept name PV_Group, filename PV_Group.json, topology references, and all point definitions unchanged. Updated README to distinguish the type/display label from the stable product identifier.

### Refine product types by device function

- Replaced broad classifications for ten products: Battery uses Battery, both PCS variants use PCS, both PV inverter variants use AC_Inverter, both hybrid variants use Hybrid_Inverter, both diesel variants use Diesel, and PV_Group uses PV_Group.
- Single-phase and three-phase products share the same functional type. Station, Environment, Meter and Load remain unchanged. Supersedes earlier ESS/Generator/Inverter grouping descriptions.
- Updated README with the ten concrete types, Chinese meanings and the Diesel example. Product names, point definitions, units, IDs, display flags and topology are unchanged. Consumer code and runtime configurations were not changed.
- Removed **Diesel_Source_Verification.md** and **SQL_Unit_Verification.md**; current unit mapping, period rules and diesel source notes live in README and the CHANGELOG contract header above.

## 2026-11-25

### Whole-library units and period-energy consistency audit

- Checked all 16 formal products and 115 period-energy points. Confirmed common business units, valid local cumulative-source references, complete five-period sets, matching single/three-phase shared fields, and hidden period monitor flags.
- Standardized period descriptions with reporting time zone, current calendar boundaries, inherited kWh units, configured rules/baselines, and unavailable results on missing baselines or counter resets. Preserved measurement boundaries, IDs, point names, and display attributes.
- Kept generic Diesel temperature descriptions independent of vendor register addresses; source-specific Fahrenheit conversion remains in the mapping documentation. Made PV inverter AC power direction explicit for both signs.
- Corrected README enum mapping wording and topology example text. Marked the abandoned source-unit decision and intermediate AC sign convention as historical, with the current contract stated at the top.
- This audit updates product descriptions and documentation only. SQL, consumer code, runtime routes and calculation services were not changed; annual aggregation still requires implementation/configuration.

### Restore consistent product business units

- Supersedes the Preserve confirmed SQL engineering units entry below. Restored 34 measurement units: kW for active power, kvar for reactive power, kWh for energy and period deltas, and ℃ for temperature, including Meter and Load business consistency.
- Retained source SQL engineering units and scaling. Documented source-to-product conversions, Fahrenheit offset, cumulative-source period units, and separate control-write conversions in README, SQL_Unit_Verification.md and Diesel_Source_Verification.md.
- IDs, names, enum definitions, display flags, SQL and runtime configurations remain unchanged. No frontend or service changes were made.

### Preserve confirmed SQL engineering units (superseded)

Historical intermediate decision, fully superseded by Restore consistent product business units above. No current product should adopt source W/Wh/degF as its business unit based on this entry.

- Matched Diesel coolant temperature to degF; Hybrid powers to W and heat-sink temperature to degC; Battery average temperature to degC; Meter active/reactive power to W/var and cumulative/period energy to Wh.
- Removed Meter descriptions requiring Wh-to-kWh conversion. Preserved SQL scaling, point IDs, names, display metadata, property specifications and unverified action units.
- Added SQL_Unit_Verification.md listing confirmed sources, changes and unresolved source-dependent units. No SQL or runtime routes were changed. Existing conversions must be reviewed before deployment.

### Diesel manual and SQL source verification

- Added Diesel_Source_Verification.md with MQ/Basler source pages, conditional single-phase voltage/current mappings, candidate control registers, and unresolved commissioning items.
- Confirmed the documented MQ DCA20SPXU4F is single-phase 120/240 V split-phase and uses Single_Phase_Diesel. Its operation manual lists ECU-754; actual installation of the optional Basler controller remains unverified.
- Confirmed fuel-level percentage semantics across Basler metering/sender documentation; the native 44806 row has no unit, so raw-value correspondence and sender calibration still require field verification.
- Distinguished documented remote start/stop/alarm-reset support from the SQL's absent control/adjustment routes. Documented why the percentage Base Load Level setting does not establish an off-grid kW Power Set capability.
- Updated README with the conclusion and verification link. Product JSON, point IDs, display metadata, SQL, and runtime services were not changed.


### Final AC variant consistency check

- Verified paired single-phase/three-phase templates for AC Inverter, Hybrid Inverter, PCS, Diesel, Meter, and Load. Shared points within each pair have matching names, IDs, units, types, enum definitions, and display attributes.
- Made generator total/per-phase active-power directions explicit: positive generation toward the AC system, negative reverse power into the generator. Matched the existing Diesel draft descriptions.
- PCS/Hybrid AC power and AC targets remain input-positive. Battery-side power/current/targets remain charge-positive. Generator and PV inverter AC output, meter orientation, and load consumption retain their product-specific boundaries.
- Added a current six-family P/M/A count and yearly-period range table to README, clarified default visibility and phase grouping, and made the Diesel topology example valid standalone JSON.
- No IDs, point names, enum values, display flags, or product connection rules changed in this check. Runtime interfaces, sources, signs, scaling, and annual computations still require the documented instance-level verification.



### Complete single-phase and three-phase AC product coverage

- Added Single_Phase_Diesel, displayed as Diesel Generator (Single Phase), with six properties, 14 measurements, and four actions.
- Retained common generator status, power, frequency, cumulative energy, yearly/period energy, and action IDs from Three_Phase_Diesel. Replaced phase-specific AC measurements with M107 AC Voltage and M108 AC Current.
- Documented voltage terminal reference and current leg selection for 120/240 V split-phase generators. Never map a leg current as a sum of leg currents.
- Added reciprocal links in both meter templates. Existing Three_Phase_Diesel definitions remain intact.
- Documented paired templates for AC Inverter, Hybrid Inverter, PCS, Diesel, Meter, and Load. The catalog now contains 16 products. This supersedes the earlier three-phase-only Diesel catalog decision.
- Actual wiring, device source validity, command support, and unit/sign conversions still require instance mapping. No runtime instance migration or service changes were performed.



### PCS and Hybrid AC-side convention: input-positive

- Updated AC Power and Power Set in Single_Phase_PCS, Three_Phase_PCS, Single_Phase_Hybrid_Inverter, and Three_Phase_Hybrid_Inverter: positive from the AC system into the device, negative from the device toward the AC system.
- Battery Power, Battery Current, and Battery Power Set retain positive charging and negative discharging. Charge/discharge energy counters and period calculations retain their definitions.
- PCS charging normally has positive AC and battery-side power; discharging normally has negative power on both sides. Hybrid PV generation can charge the battery while supplying AC output, so battery direction is determined by Battery Power.
- Supersedes the earlier output-positive AC-side convention in these four products, including the AC-side paragraph of the Battery-side sign convention entry below. Other product AC direction conventions retain their current definitions.
- Updated README and matching local drafts. Only direction descriptions changed in product JSON; point IDs, names, units, enums, display metadata, topology, and cumulative-source references are retained.
- Review existing sign-dependent routes, rules, charts, and control mappings before deployment. Sol-Ark conversion-side AC and battery raw signs require confirmation; this update does not modify the site SQL, frontend, running services, or instance routes.



### Battery-side sign convention: charge-positive

- Changed Battery Power, Battery Current, and Battery Power Set descriptions in Battery, both PCS variants, and both Hybrid variants to positive for charging and negative for discharging. Battery now explicitly describes the current direction.
- At this intermediate revision, AC power/targets were output-positive. That AC convention was subsequently superseded by PCS and Hybrid AC-side convention: input-positive above. The current contract has charging-positive battery power/current and input-positive AC power/targets; Hybrid AC output can also originate from PV.
- This entry supersedes earlier discharge-positive battery-side conventions recorded in the migration history. Existing routes, sign-dependent rules, control commands, charts, and name-based consumers need explicit sign review before upgrading.
- LYNK raw current already matches the new convention. Sol-Ark raw battery power/current signs still require confirmation. Cumulative and period charge/discharge energy definitions retain separate directions.
- Updated README and matching local draft descriptions. IDs, names, units, enum options, display metadata, and topology are retained. No runtime routes or services were changed.



### Product semantics and device mapping documentation

- Updated the current numbering table with yearly outputs and added Diesel M213 AC Yearly Energy to the README walkthrough.
- Documented Station as the start/stop default-display exception and clarified how monitorVisible hides period energy despite retained isDisplay markers.
- Separated platform battery-port direction conventions from manufacturer raw signs. Documented LYNK current sign conversion, independent action-route verification, optional device capabilities, and the information required for instance mapping.
- Recorded the remaining Battery Current description gap explicitly. This update changes README and CHANGELOG only; product JSON, the site SQL, frontend, and services retain their current contents.



### Current-calendar-year energy

- Appended 22 Yearly energy measurements across 13 products, preserving all existing point IDs and definitions. Each new point uses the cumulative source of the corresponding quarterly measurement.
- Yearly means year-to-date from January 1 in the configured reporting time zone. A valid year-start baseline and annual calculation support are required. The current Period Delta implementation supports daily/weekly/monthly/quarterly only; annual computation is pending.
- All yearly energy points use attr.monitorVisible = "false", alongside the existing hidden period energy measurements.
- This entry extends the earlier period ranges. Historical tables describe earlier template versions. No frontend or calculation service changes are included.

| Product | Added yearly measurements |
| --- | --- |
| Three_Phase_AC_Inverter | M213 PV Yearly Energy |
| Single_Phase_Meter | M205 AC Import Yearly Energy |
| Three_Phase_Load | M205 Load Yearly Energy |
| Single_Phase_PCS | M209 Yearly Charge Energy, M210 Yearly Discharge Energy |
| Three_Phase_PCS | M209 Yearly Charge Energy, M210 Yearly Discharge Energy |
| Three_Phase_Diesel | M213 AC Yearly Energy |
| Single_Phase_Load | M205 Load Yearly Energy |
| PV_Group | M205 PV Yearly Energy |
| Three_Phase_Hybrid_Inverter | M217 Yearly Charge Energy, M218 Yearly Discharge Energy, M219 PV Yearly Energy, M220 Load Yearly Energy |
| Single_Phase_Hybrid_Inverter | M217 Yearly Charge Energy, M218 Yearly Discharge Energy, M219 PV Yearly Energy, M220 Load Yearly Energy |
| Battery | M209 Yearly Charge Energy, M210 Yearly Discharge Energy |
| Single_Phase_AC_Inverter | M213 PV Yearly Energy |
| Three_Phase_Meter | M205 AC Import Yearly Energy |



### Period energy monitor visibility

- Added attr.monitorVisible = "false" to all 88 daily, weekly, monthly, and quarterly energy measurements across 13 products.
- Kept existing IDs, names, values, descriptions, enum definitions, default-display flags, display groups, and cumulative-source references.
- Monitor visibility defaults to visible when unspecified. An explicit string "false" takes precedence over isDisplay for monitor cards, featured cards, and display groups. Statistics, history access, and rule calculations continue to use these measurements.
- Frontend and backend consumers are unchanged in this update. Page filtering requires their support for the new metadata.



### Three-phase measurement display groups

- Added attr.displayGroup to 27 existing phase measurements in Three_Phase_Diesel, Three_Phase_Load, and Three_Phase_Meter.
- L1/L2/L3 voltage points use AC Voltage; current points use AC Current; active power points use AC Power.
- Preserved point IDs, names, values, descriptions, array order, and default display flags. No phase field was added.
- Documented the optional display metadata. Frontend grouping and data-path support are outside this change.



### Enum option descriptions

- Added English description fields to all 43 enum options across 11 product definitions. Existing point IDs, names, units, types, option keys, labels, values, and display metadata are retained.
- Status descriptions explain the reported business meaning. Station, environmental-monitor, and PV-group start/stop states use their own explicit source context.
- Action descriptions explain the requested operation and the feedback used to confirm its outcome.
- Documented the optional enumData[].description field and updated the README examples. AI consumers need this field passed through their product-definition data path.

### Current point numbering and PCS default display

- Single_Phase_PCS and Three_Phase_PCS retain exactly three isDisplay markers: M1 SOC, M102 Battery Power, and M202 Daily Discharge Energy. The later monitorVisible rule hides period energy, so SOC and Battery Power are the effective visible defaults. AC Power remains available in details.
- Matching point meanings use consistent names, units, and measurement boundaries. IDs are local to each product and point table; cross-product ID equality is not required.
- P and A tables number consecutively from 1. M1–M100 contains basic states/measurements, M101–M200 contains electrical measurements and cumulative counters, and M201–M300 contains period results. Keep documented fixed-period offsets and existing product-specific gaps.
- The current generic inverter/PCS layouts are summarized below. These current ranges supersede intermediate IDs recorded in the historical entries. This update changes only PCS display metadata and documentation.

| Product pair | Basic M IDs | Electrical/cumulative M IDs | Period M IDs |
| --- | --- | --- | --- |
| Single/Three Phase AC Inverter | M1–M2 | M101–M104 | M209–M213 |
| Single/Three Phase PCS | M1–M3 | M101–M107 | M201–M210 |
| Single/Three Phase Hybrid Inverter | M1–M3 | M101–M111 | M201–M220 |

The following dated entries retain earlier changes and migration history. Resolve historical mappings against the source template version and the current product JSON.

## 2026-11-24

### Start Stop Status and Grid Status

- Removed **Grid Status** from Single_Phase_AC_Inverter, Three_Phase_AC_Inverter, Single_Phase_PCS, and Three_Phase_PCS. On PCS, **Charge Discharge Status** renumbered from M4 to **M3**.
- Added **Start Stop Status** (enum Stopped/Running) to every product that lacked it: Hybrid (M3), PV_Group (M1), Env (M3), Battery (M11). AC Inverter, Diesel, and PCS already had Start Stop Status; IDs unchanged except PCS charge/discharge as above.
- **Single_Phase_Meter**, **Three_Phase_Meter**, **Single_Phase_Load**, and **Three_Phase_Load** keep empty **A** tables and no Start Stop measurement; meters and loads are monitor-only at the product template.
- Station **M1** renamed from **Status** (Stopped/Running/Fault) to **Start Stop Status** (Stopped/Running only).

### AC Inverter and PCS (generic, Hybrid-aligned)

- **Single_Phase_AC_Inverter** and **Three_Phase_AC_Inverter**: P aligned with Hybrid PV side (P1–P5 Max/Rated/Max PV); M trimmed to M1 Heat Sink Temperature, M2 Start Stop Status, M101 AC Power, M102 DC Input Power, M103 AC Frequency, M104 PV Energy Total, M209–M212 PV period energies. Removed Efficiency, AC/DC V/I, per-phase electricals, and M131 AC Energy Total. Actions match Hybrid PV/AC subset: Start, Stop, Power Set (removed Clear Error).
- **Single_Phase_PCS** and **Three_Phase_PCS**: P1–P6 contains the Hybrid battery/AC property subset in consecutive order; M contains M1 SOC, M2 Heat Sink Temperature, M3 Start Stop Status, M101 AC Power, M102–M104 battery power/voltage/current, M105 AC Frequency, M106–M107 charge/discharge totals, M201–M208 period energies. Removed Charge Discharge Status, per-phase AC measurements, AC import/export energy IDs (M132–M133, M213–M220), and Clear Error. Actions: Start, Stop, Power Set, Battery Power Set (same as Hybrid).
- **ID 顺延（保留分段特殊规则）**：各表内连续编号；P/M/A 仍按层段 **M1–M100 / M101–M200 / M201+** 与 AC 周期 **M209–M212**（Hybrid 约定）分段，段内不留空号。AC Inverter：M1–M2 状态，M101–M104 电气与累计，M209–M212 周期。PCS：P1–P6（无 PV 不留 P4–P5 空位），M1–M3 状态，M101–M107 电气与累计，M201–M208 充放周期（与 Hybrid/Battery 一致）。

### Documentation

- README adds a product walkthrough for `Three_Phase_Diesel` (topology, P1–P6, M layers, default display, actions, and mapping notes) aligned with the current JSON.

### Station

- Removed P parameters Rated Capacity and Station Type. Renumbered remaining P points to P1–P4: Longitude, Latitude, Altitude, Scenario Parameters. M and A are unchanged.
- M1 Status is now an enum with Stopped, Running, and Fault. Instance mapping supplies actual device values.
- M2 Saving Billing is now `type: number` with unit `$` for numeric savings in US dollars.

### Battery

- Kept separate charge and discharge power measurements (M5/M6) and their cumulative energy inputs (M9/M10).
- Renamed M1/M2 to Battery Voltage and Battery Current. M11 is now an enum with Idle, Charging, and Discharging. Actual device values are configured per instance.
- Added M31 Avg Battery Temperature. Removed M17/M18 cell arrays and M19 Battery System from the generic template.

| Period energy | Previous IDs | New IDs |
| --- | --- | --- |
| Daily charge / discharge | M23 / M24 | M201 / M202 |
| Weekly charge / discharge | M25 / M26 | M203 / M204 |
| Monthly charge / discharge | M27 / M28 | M205 / M206 |
| Quarterly charge / discharge | M29 / M30 | M207 / M208 |

Before upgrading an existing instance, migrate references in routes, rule outputs, alarms, and history queries by point meaning and source template version. Sites still on M101–M108 or other legacy IDs must verify the old mapping. History under previous IDs needs to be queried or migrated explicitly.

#### Battery properties and display order

- Defined five manufacturer-specified properties: usable energy (P1 Max Capacity), nominal DC voltage (P2), maximum continuous charge current (P3), maximum continuous discharge current (P4), and maximum continuous discharge power (P5). Values are configured per instance.
- Review saved properties and rule references by meaning before upgrading. Previous property IDs identify different specifications and must not be copied by ID. Verify the capacity basis before reusing a legacy value.
- Moved M3, M4, M11–M16, and M22 to the end of the measurement list with their IDs and definitions intact. The action definitions are unchanged.

### Diesel properties

- Removed P6 Min Power, P8 Startup Time, P9 Min Runtime, and P10 Response Time.
- Renumbered Fuel Consumption Rate from P7 to P6. Migrate existing property values and rule references by meaning before upgrading. The former P6 identified Min Power.

### Diesel and PCS actions

- Removed A4 Off On Grid and renumbered A5 Clear Error to A4.
- Before upgrading existing instances, remove old Off On Grid routes and references, then migrate Clear Error routes and references from A5 to A4. Reset must never inherit the previous A4 binding.

### Phase-specific AC inverter and diesel products

- Replaced AC_Inverter with Single_Phase_AC_Inverter and Three_Phase_AC_Inverter using the reviewed point definitions.
- Replaced Diesel with Three_Phase_Diesel. Battery and PCS retain the changes recorded above.
- Updated reciprocal PV_Group and Meter connection rules. Removed Meter references to the already deleted EV_Charging_Load and HVAC_Load products.
- Existing instances require an explicit product-name and point-ID migration. Update property values, routes, rules, alarms, history queries, and saved topology references by meaning. Point IDs in the replacement templates may have different meanings.
- Consumer product classification, name-based statistics, and cloud import catalogs must be adapted before deployment.

### Environment monitoring

- Kept Temperature and Humidity as M1 and M2, renumbered from M3 and M4. Both are marked for default display.
- Removed the assumed water leakage, lightning protection, fire protection, and emergency stop measurements and cleared the action table.
- Before upgrading existing Env instances, remove references to the deleted points and actions. Migrate temperature from M3 to M1 and humidity from M4 to M2, including routes, rules, alarms, and history queries. The previous M1/M2 identified safety signals.

### Three-phase PCS

- Kept product identity PCS and changed displayName to PCS (Three Phase).
- Defined positive AC/DC power and DC current as discharging. Negative values indicate charging. A3 Power Set is an AC active power target with the same discharge-positive convention.
- Split the ambiguous voltage property into Max AC Voltage (line-to-line) and Max DC Voltage; removed the configured conversion efficiency.
- Added DC Current, AC Import Energy Total, AC Export Energy Total, and eight period-energy outputs at M201–M208. Cumulative sources are M10 and M11. Configure period rules separately.
- Defined M7 start/stop, M8 grid state, and M9 charge/discharge state as enums with instance-configured actual values.
- Moved phase voltages to M101–M103, line voltages to M105–M107, phase currents to M111–M113, and phase active powers to M121–M123.

| Existing measurement | Previous ID | New ID |
| --- | --- | --- |
| AC total power / DC power | M1 / M2 | M1 / M2 |
| DC voltage | M6 | M3 |
| AC frequency / Temperature | M17 / M13 | M5 / M6 |
| Start Stop Status / Grid Status / Direction | M14 / M15 / M16 | M7 / M8 / M9 |
| Phase A/B/C voltage | M7–M9 | M101–M103 |
| Phase A/B/C current | M10–M12 | M111–M113 |
| Phase A/B/C power | M3–M5 | M121–M123 |

Property migration: P1 stays P1; old P3/P4/P5 become P4/P5/P6. Determine whether old P2 was AC or DC before moving its value to P2 or P3. Old P6 Conversion Efficiency is removed. Review power signs and enum mappings, then migrate routes, property values, rule/alarm references, and history queries by meaning before upgrading instances.

### Compact phase measurement IDs

- Updated Single_Phase_AC_Inverter, Three_Phase_AC_Inverter, Three_Phase_Diesel, and PCS to share compact phase IDs: voltages M101–M103, currents M104–M106, active powers M107–M109, and line-to-line voltages M110–M112 where defined.
- Migration from the intermediate layout: M111–M113 to M104–M106; M121–M123 to M107–M109; existing line voltages M105–M107 to M110–M112. Apply as a simultaneous mapping by product version and point meaning to avoid overlapping IDs.
- The earlier migration entries describe intermediate versions. Use this final layout for new instances.

### Remove line voltage measurements

- Removed M110–M112 line-to-line voltage measurements from Three_Phase_Diesel and PCS. The retained phase measurements remain consecutive at M101–M109.
- Updated the corresponding Diesel draft and removed line-to-line voltages from the deferred hybrid inverter draft.
- Remove references to these deleted measurements before upgrading existing instances.

### Unified measurement placement and names

- Basic measurements use M1–M100. AC/DC electrical measurements and cumulative energy counters use M101–M200. Period energy and period maximum-demand values use M201–M300.
- Common measurement names share fixed IDs across products. Different energy boundaries retain distinct names. Single_Phase_AC_Inverter uses one AC Output Power measurement instead of duplicate total and line power measurements.
- Updated cumulative-source references inside descriptions and synchronized existing drafts. Product IDs, properties, actions, topology, units, and device-sign conventions retain their current definitions.
- These mappings supersede intermediate measurement layouts described earlier in this entry. Apply mappings simultaneously using the source template version. Update routes, rules, alarms, name-based selectors, saved configuration, and history queries before upgrading existing instances.

#### Three_Phase_AC_Inverter

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| AC Output Power | M1 | AC Output Power | M101 |
| DC Input Power | M2 | DC Input Power | M120 |
| DC Input Voltage | M3 | DC Input Voltage | M121 |
| DC Input Current | M4 | DC Input Current | M122 |
| AC Frequency | M5 | AC Frequency | M106 |
| Temperature | M6 | Temperature | M1 |
| Efficiency | M7 | Efficiency | M2 |
| Start Stop Status | M8 | Start Stop Status | M3 |
| Grid Status | M9 | Grid Status | M4 |
| AC Energy Total | M10 | AC Energy Total | M131 |
| L1 Voltage | M101 | L1 Voltage | M111 |
| L2 Voltage | M102 | L2 Voltage | M112 |
| L3 Voltage | M103 | L3 Voltage | M113 |
| L1 Current | M104 | L1 Current | M114 |
| L2 Current | M105 | L2 Current | M115 |
| L3 Current | M106 | L3 Current | M116 |
| L1 Active Power | M107 | L1 Active Power | M117 |
| L2 Active Power | M108 | L2 Active Power | M118 |
| L3 Active Power | M109 | L3 Active Power | M119 |
| AC Daily Energy | M201 | AC Daily Energy | M209 |
| AC Weekly Energy | M202 | AC Weekly Energy | M210 |
| AC Monthly Energy | M203 | AC Monthly Energy | M211 |
| AC Quarterly Energy | M204 | AC Quarterly Energy | M212 |

#### Meter

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| Total Active Power | M1 | AC Power | M102 |
| Total Reactive Power | M2 | AC Reactive Power | M103 |
| Power Factor | M3 | AC Power Factor | M105 |
| Frequency | M4 | AC Frequency | M106 |
| A Phase Voltage | M5 | L1 Voltage | M111 |
| B Phase Voltage | M6 | L2 Voltage | M112 |
| C Phase Voltage | M7 | L3 Voltage | M113 |
| A Phase Current | M8 | L1 Current | M114 |
| B Phase Current | M9 | L2 Current | M115 |
| C Phase Current | M10 | L3 Current | M116 |
| A Phase Active Power | M11 | L1 Active Power | M117 |
| B Phase Active Power | M12 | L2 Active Power | M118 |
| C Phase Active Power | M13 | L3 Active Power | M119 |
| Total Energy | M14 | Meter Energy Total | M138 |
| Energy Today | M15 | Meter Daily Energy | M229 |
| Energy This Month | M16 | Meter Monthly Energy | M231 |
| Max Demand This Month | M17 | AC Max Demand This Month | M241 |
| Current Demand | M18 | AC Current Demand | M158 |

#### Three_Phase_Load

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| Total Active Power | M1 | AC Power | M102 |
| Total Reactive Power | M2 | AC Reactive Power | M103 |
| Power Factor | M3 | AC Power Factor | M105 |
| Frequency | M4 | AC Frequency | M106 |
| A Phase Voltage | M5 | L1 Voltage | M111 |
| B Phase Voltage | M6 | L2 Voltage | M112 |
| C Phase Voltage | M7 | L3 Voltage | M113 |
| A Phase Current | M8 | L1 Current | M114 |
| B Phase Current | M9 | L2 Current | M115 |
| C Phase Current | M10 | L3 Current | M116 |
| A Phase Active Power | M11 | L1 Active Power | M117 |
| B Phase Active Power | M12 | L2 Active Power | M118 |
| C Phase Active Power | M13 | L3 Active Power | M119 |
| Total Energy | M14 | Load Energy Total | M137 |
| Energy Today | M15 | Load Daily Energy | M225 |
| Energy This Month | M16 | Load Monthly Energy | M227 |
| Max Demand This Month | M17 | AC Max Demand This Month | M241 |
| Current Demand | M18 | AC Current Demand | M158 |

#### Three_Phase_Diesel

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| AC Output Power | M1 | AC Output Power | M101 |
| AC Energy Total | M2 | AC Energy Total | M131 |
| Fuel Level | M13 | Fuel Level | M1 |
| Coolant Temperature | M14 | Coolant Temperature | M2 |
| Start Stop Status | M15 | Start Stop Status | M3 |
| AC Frequency | M16 | AC Frequency | M106 |
| Engine Load | M17 | Engine Load | M4 |
| L1 Voltage | M101 | L1 Voltage | M111 |
| L2 Voltage | M102 | L2 Voltage | M112 |
| L3 Voltage | M103 | L3 Voltage | M113 |
| L1 Current | M104 | L1 Current | M114 |
| L2 Current | M105 | L2 Current | M115 |
| L3 Current | M106 | L3 Current | M116 |
| L1 Active Power | M107 | L1 Active Power | M117 |
| L2 Active Power | M108 | L2 Active Power | M118 |
| L3 Active Power | M109 | L3 Active Power | M119 |
| AC Daily Energy | M201 | AC Daily Energy | M209 |
| AC Weekly Energy | M202 | AC Weekly Energy | M210 |
| AC Monthly Energy | M203 | AC Monthly Energy | M211 |
| AC Quarterly Energy | M204 | AC Quarterly Energy | M212 |

#### Single_Phase_Load

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| Load Power | M1 | Load Power | M152 |
| Energy Used | M2 | Load Energy Total | M137 |
| Voltage | M3 | AC Voltage | M109 |
| Current | M4 | AC Current | M110 |
| Frequency | M5 | AC Frequency | M106 |
| Power Factor | M6 | AC Power Factor | M105 |
| Reactive Power | M7 | AC Reactive Power | M103 |
| Apparent Power | M8 | AC Apparent Power | M104 |
| Energy Used Today | M9 | Load Daily Energy | M225 |

#### PV_Group

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| PV Power (Array) | M1 | PV Power (Array) | M139 |
| PV Voltage (Array) | M2 | PV Voltage (Array) | M140 |
| PV Current (Array) | M3 | PV Current (Array) | M141 |
| Sub PVI | M4 | Sub PVI | M142 |
| Energy Today | M5 | PV Daily Energy | M221 |
| Start Stop Status | M6 | Start Stop Status | M1 |
| PV Power | M7 | PV Power | M143 |
| PV Voltage | M8 | PV Voltage | M144 |
| PV Current | M9 | PV Current | M145 |
| Peak Efficiency | M10 | Peak Efficiency | M2 |
| Average Efficiency | M11 | Average Efficiency | M3 |
| Minimum Efficiency | M12 | Minimum Efficiency | M4 |
| Solar Panels | M13 | Solar Panels | M5 |
| PV System | M14 | PV System | M6 |
| Energy Total | M15 | PV Energy Total | M136 |

#### Battery

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| Battery Voltage | M1 | Battery Voltage | M127 |
| Battery Current | M2 | Battery Current | M128 |
| Charge Power | M5 | Charge Power | M129 |
| Discharge Power | M6 | Discharge Power | M130 |
| SOC | M7 | SOC | M1 |
| SOH | M8 | SOH | M2 |
| Charge Energy | M9 | Charge Energy | M134 |
| Discharge Energy | M10 | Discharge Energy | M135 |
| BMS Charge Current Limit | M20 | BMS Charge Current Limit | M3 |
| BMS Discharge Current Limit | M21 | BMS Discharge Current Limit | M4 |
| Avg Battery Temperature | M31 | Avg Battery Temperature | M5 |
| Max Battery Pack Temperature | M3 | Max Battery Pack Temperature | M6 |
| Min Battery Pack Temperature | M4 | Min Battery Pack Temperature | M7 |
| Charge Discharge Status | M11 | Charge Discharge Status | M8 |
| Max Cell Voltage | M12 | Max Cell Voltage | M146 |
| Min Cell Voltage | M13 | Min Cell Voltage | M147 |
| Avg Cell Voltage | M14 | Avg Cell Voltage | M148 |
| Cell Voltage Difference | M15 | Cell Voltage Difference | M149 |
| Avg Cell Temperature | M16 | Avg Cell Temperature | M9 |
| Online Battery Pack Count | M22 | Online Battery Pack Count | M10 |

#### Hybrid_Inverter

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| Inverter AC Output Power | M1 | AC Output Power | M101 |
| PV Input Power | M2 | DC Input Power | M120 |
| Battery Power | M3 | Battery Power | M126 |
| Battery Voltage | M4 | Battery Voltage | M127 |
| Battery Current | M5 | Battery Current | M128 |
| Battery SOC | M6 | SOC | M1 |
| Load Power | M7 | Load Power | M152 |
| Inverter Output Frequency | M8 | AC Frequency | M106 |
| Heat Sink Temperature | M9 | Heat Sink Temperature | M2 |
| PV Energy Today | M10 | PV Daily Energy | M221 |
| PV Energy Total | M11 | PV Energy Total | M136 |
| Battery Charge Energy Total | M12 | Charge Energy | M134 |
| Battery Discharge Energy Total | M13 | Discharge Energy | M135 |
| Operating Status | M14 | Operating Status | M3 |
| Start Stop Status | M15 | Start Stop Status | M4 |

#### Single_Phase_AC_Inverter

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| AC Output Power | M1 | AC Output Power | M101 |
| DC Input Power | M2 | DC Input Power | M120 |
| DC Input Voltage | M3 | DC Input Voltage | M121 |
| DC Input Current | M4 | DC Input Current | M122 |
| AC Frequency | M5 | AC Frequency | M106 |
| Temperature | M6 | Temperature | M1 |
| Efficiency | M7 | Efficiency | M2 |
| Start Stop Status | M8 | Start Stop Status | M3 |
| Grid Status | M9 | Grid Status | M4 |
| AC Energy Total | M10 | AC Energy Total | M131 |
| AC Output Voltage | M101 | AC Output Voltage | M107 |
| AC Output Current | M104 | AC Output Current | M108 |
| AC Output Active Power | M107 | AC Output Power | M101 |
| AC Daily Energy | M201 | AC Daily Energy | M209 |
| AC Weekly Energy | M202 | AC Weekly Energy | M210 |
| AC Monthly Energy | M203 | AC Monthly Energy | M211 |
| AC Quarterly Energy | M204 | AC Quarterly Energy | M212 |

#### PCS

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| AC Power | M1 | AC Power | M102 |
| DC Power | M2 | DC Power | M123 |
| DC Voltage | M3 | DC Voltage | M124 |
| DC Current | M4 | DC Current | M125 |
| AC Frequency | M5 | AC Frequency | M106 |
| Temperature | M6 | Temperature | M1 |
| Start Stop Status | M7 | Start Stop Status | M2 |
| Grid Status | M8 | Grid Status | M3 |
| Charge Discharge Status | M9 | Charge Discharge Status | M4 |
| AC Import Energy Total | M10 | AC Import Energy Total | M132 |
| AC Export Energy Total | M11 | AC Export Energy Total | M133 |
| L1 Voltage | M101 | L1 Voltage | M111 |
| L2 Voltage | M102 | L2 Voltage | M112 |
| L3 Voltage | M103 | L3 Voltage | M113 |
| L1 Current | M104 | L1 Current | M114 |
| L2 Current | M105 | L2 Current | M115 |
| L3 Current | M106 | L3 Current | M116 |
| L1 Active Power | M107 | L1 Active Power | M117 |
| L2 Active Power | M108 | L2 Active Power | M118 |
| L3 Active Power | M109 | L3 Active Power | M119 |
| AC Import Daily Energy | M201 | AC Import Daily Energy | M213 |
| AC Export Daily Energy | M202 | AC Export Daily Energy | M217 |
| AC Import Weekly Energy | M203 | AC Import Weekly Energy | M214 |
| AC Export Weekly Energy | M204 | AC Export Weekly Energy | M218 |
| AC Import Monthly Energy | M205 | AC Import Monthly Energy | M215 |
| AC Export Monthly Energy | M206 | AC Export Monthly Energy | M219 |
| AC Import Quarterly Energy | M207 | AC Import Quarterly Energy | M216 |
| AC Export Quarterly Energy | M208 | AC Export Quarterly Energy | M220 |

### Single-phase PCS

- Added Single_Phase_PCS with displayName PCS (Single Phase). Existing PCS remains the three-phase product.
- Shared states, AC/DC power, DC voltage/current, cumulative energy, period outputs, and action IDs follow the three-phase template.
- Defined M109 AC Voltage (line-to-neutral) and M110 AC Current. Single-phase AC Power at M102 is the sole active power measurement.
- Added reciprocal Battery and Meter topology connections. The single-phase AC voltage/current properties describe the single-phase connection.
- This generic template requires model-specific measurement and control routes before use.

### Battery net power and consecutive electrical IDs

- Replaced Charge Power and Discharge Power with Battery Power, positive for discharge and negative for charging.
- Numbered battery electrical measurements consecutively from M101. Basic properties/states, period IDs M201–M208, and action definitions retain their current definitions.
- Period charge/discharge calculations now reference cumulative M104/M105. Synchronize the Battery draft.
- This Battery mapping supersedes the shared-ID layout above. Name-based consumers must read Battery Power and handle its sign. Configure one normalized net-power source instead of merging two existing routes onto the new point. Migrate other route/rule/alarm references and history queries by meaning.

| Previous name | Previous ID | New name | New ID |
| --- | --- | --- | --- |
| Battery Voltage | M127 | Battery Voltage | M101 |
| Battery Current | M128 | Battery Current | M102 |
| Charge Power | M129 | Battery Power | M103 |
| Discharge Power | M130 | Battery Power | M103 |
| Charge Energy | M134 | Charge Energy | M104 |
| Discharge Energy | M135 | Discharge Energy | M105 |
| Max Cell Voltage | M146 | Max Cell Voltage | M106 |
| Min Cell Voltage | M147 | Min Cell Voltage | M107 |
| Avg Cell Voltage | M148 | Avg Cell Voltage | M108 |
| Cell Voltage Difference | M149 | Cell Voltage Difference | M109 |

### Hybrid inverter core measurements

- Updated the existing Hybrid_Inverter product and synchronized its draft. Retained two basic measurements, fourteen electrical measurements/cumulative counters, and twenty-four period energy outputs.
- Kept PV input, battery, inverter AC output, load, and grid boundaries distinct. Added grid power and directional counters plus cumulative load energy. Omitted phase, MPPT, and GEN details from the core template.
- Removed Operating Status and Start Stop Status measurements because the available site point list has no confirmed source. Retained the actions with an explicit writable-interface requirement; the supplied Sol-Ark protocol map is read-only.
- Removed the MPPT Count property, renamed Rated AC Power to Max Power, and renumbered the remaining properties consecutively. Former P7/P8/P9 become P6/P7/P8.
- Renamed A3 Active Power Setpoint to Power Set. Its configured route must actually support an AC active power target.
- Period M201–M208 uses charge/discharge cumulative M110/M111. M209–M212 uses PV M109; M213–M216 uses load M112; M217–M220 uses grid import M113; M221–M224 uses grid export M114. Configure EMS rules separately.
- This mapping supersedes intermediate Hybrid_Inverter definitions. Migrate routes, rule inputs/outputs, alarms, name-based consumers, saved values, and history queries by point meaning and source version. Normalize battery power/current to discharge-positive and grid power to import-positive.

| Measurement | Previous ID | Final ID |
| --- | --- | --- |
| SOC | M1 | M1 |
| Heat Sink Temperature | M2 | M2 |
| Operating Status | M3 | Removed: no confirmed source |
| Start Stop Status | M4 | Removed: no confirmed source |
| AC Output Power | M101 | M101 |
| AC Frequency | M106 | M108 |
| DC Input Power | M120 | M102 |
| Battery Power | M126 | M103 |
| Battery Voltage | M127 | M106 |
| Battery Current | M128 | M107 |
| Charge Energy | M134 | M110 |
| Discharge Energy | M135 | M111 |
| PV Energy Total | M136 | M109 |
| Load Power | M152 | M104 |
| PV Daily Energy | M221 | M209 |

### Inverter power targets and AC inverter properties

- Clarified Hybrid_Inverter A3 Power Set as a signed target at the inverter conversion AC connection: positive toward the AC side, negative into the inverter.
- Clarified A4 Battery Power Setpoint as a signed target at the battery DC connection: positive discharge, negative charge. Both commands require a verified writable interface and conversion to the device sign convention.
- Updated the AC Output Power description to use the same signed measurement boundary.
- Removed P6 String Count and P7 MPPT Count from both Single_Phase_AC_Inverter and Three_Phase_AC_Inverter. Remaining properties keep P1–P5 in their existing order. Review references to the removed properties before upgrading.
- Synchronized the corresponding local drafts.

### Hybrid inverter grid period energy

- Removed M217–M224 grid import/export daily, weekly, monthly, and quarterly energy outputs.
- Retained grid power and cumulative grid import/export energy. The remaining period outputs stay consecutive at M201–M216.
- Remove rule outputs and other references to the deleted period points before upgrading existing instances. Synchronized the local draft.

### Off-grid hybrid inverter scope

- Scoped Hybrid_Inverter to the current off-grid project. Removed Grid Power and cumulative grid import/export measurements; grid period energy had already been removed.
- Renumbered retained electrical measurements consecutively at M101–M111 and updated the cumulative-source references for period energy. Period outputs remain M201–M216.
- Default display now includes DC Input Power, Battery Power, and SOC. Diagnostic measurements and counters remain available in details. Product topology and actions retain their definitions.
- Migrate existing references by meaning before upgrading and remove references to deleted grid measurements. Synchronized the local draft.

| Measurement | Previous ID | New ID |
| --- | --- | --- |
| Grid Power | M105 | Removed |
| Battery Voltage | M106 | M105 |
| Battery Current | M107 | M106 |
| AC Frequency | M108 | M107 |
| PV Energy Total | M109 | M108 |
| Charge Energy | M110 | M109 |
| Discharge Energy | M111 | M110 |
| Load Energy Total | M112 | M111 |
| Grid Import Energy Total | M113 | Removed |
| Grid Export Energy Total | M114 | Removed |

### Off-grid Meter core measurements

- Removed load-flexibility and load-scheduling properties P5–P10; retained P1–P4 metering-point configuration.
- Kept AC power, reactive power, power factor, frequency, and phase voltage/current/active power measurements. Electrical IDs are consecutive at M101–M115.
- Replaced ambiguous Meter Energy Total with AC Import Energy Total (M114) and AC Export Energy Total (M115). Import/export directions refer to meter installation orientation.
- Removed current demand and monthly maximum demand. Defined input-only daily/weekly/monthly/quarterly outputs at M201–M204, calculated from cumulative M114.
- Default display is AC Power, AC Frequency, and AC Import Energy Total. The action table remains empty.
- Existing instances need route, name-selector, rule/alarm, and history-query migration. Verify the direction of the old cumulative counter before assigning either new counter, and normalize W/var/Wh sources to kW/kvar/kWh.

| Measurement | Previous ID | Final mapping |
| --- | --- | --- |
| AC Power | M102 | M101 |
| AC Reactive Power | M103 | M102 |
| AC Power Factor | M105 | M103 |
| AC Frequency | M106 | M104 |
| L1 Voltage | M111 | M105 |
| L2 Voltage | M112 | M106 |
| L3 Voltage | M113 | M107 |
| L1 Current | M114 | M108 |
| L2 Current | M115 | M109 |
| L3 Current | M116 | M110 |
| L1 Active Power | M117 | M111 |
| L2 Active Power | M118 | M112 |
| L3 Active Power | M119 | M113 |
| Meter Energy Total | M138 | Split by direction: M114 / M115 after source verification |
| AC Current Demand | M158 | Removed |
| Meter Daily Energy | M229 | M201 only after verifying import direction |
| Meter Monthly Energy | M231 | M203 only after verifying import direction |
| AC Max Demand This Month | M241 | Removed |

### Phase-specific meter and PCS identities

- Added Single_Phase_Meter (Meter (Single Phase)) and renamed Meter to Three_Phase_Meter (Meter (Three Phase)).
- The meter variants share whole-device and cumulative/period energy IDs. The single-phase template uses M105 AC Voltage and M108 AC Current, omitting per-phase powers because M101 already represents single-phase active power.
- Renamed PCS.json and product name PCS to Three_Phase_PCS; its measurements, properties, actions, and displayName PCS (Three Phase) are retained.
- Updated reciprocal product connections and existing local draft topology references. A meter can be chosen by actual installation wiring; these product-level rules do not validate electrical compatibility.
- Migrate existing instance product names and saved topology references from Meter to Three_Phase_Meter when the installation is three-phase, and from PCS to Three_Phase_PCS. Name-based consumer classification and cloud import catalogs require adaptation before deployment.

### Phase-specific hybrid inverter products

- Replaced Hybrid_Inverter with Single_Phase_Hybrid_Inverter and Three_Phase_Hybrid_Inverter, displayed as Hybrid Inverter (Single Phase) and Hybrid Inverter (Three Phase).
- Both variants retain the core measurement layout and shared IDs. No per-phase details were added. Single-phase covers an installation with explicitly documented single-phase or split-phase voltage reference; the three-phase property specifies line-to-line rated voltage.
- Renamed M101 AC Output Power to AC Power to express bidirectional conversion-side active power consistently with PCS. Renamed M102 DC Input Power to PV Input Power to distinguish PV input from the battery DC connection.
- Renamed A4 Battery Power Setpoint to Battery Power Set. Battery power remains discharge-positive; charge/discharge cumulative energy remains on the battery DC side. PV, battery, conversion-side AC, and load boundaries retain separate measurements.
- Updated reciprocal connections and draft references. Replaced the old hybrid draft with two matching phase-specific files.
- Existing instances require explicit product-name migration based on the installed wiring and updates to name-based selectors, including the renamed measurements/action. The three-phase template is generic; actual device data/control capabilities require model-specific routes.

### Inverter measurement boundaries and common names

- Standardized PV Input Power across both AC inverter and both hybrid inverter variants. In AC inverter templates the former DC Input Power name changes at the same ID M120.
- Clarified hybrid conversion-side AC Power, battery-side Battery Power, PV Input Power, and load-port Load Power boundaries.
- Clarified hybrid PV cumulative/period energy as DC input, charge/discharge energy as the battery DC port, and load energy as the AC load port.
- Retained both phase-specific hybrid core templates and their existing IDs, properties, actions, and topology. Synchronized the corresponding drafts.
- Name-based AC inverter consumers must migrate DC Input Power selectors to PV Input Power. Device AC conversion power must not be interpreted as battery discharge alone because PV can also supply it.

### AC and DC power naming

- Standardized whole-device AC active power as AC Power in both AC inverter variants and Three_Phase_Diesel. Point IDs and direction descriptions are retained.
- Standardized PV DC input power as DC Input Power in both AC inverter and both hybrid inverter variants. Descriptions identify the PV input boundary.
- Hybrid Battery Power and Load Power retain their port-specific names because the same product has multiple independent power boundaries.
- Synchronized existing drafts. This naming decision supersedes the PV Input Power name recorded above. Update name-based selectors from AC Output Power to AC Power and from PV Input Power to DC Input Power as applicable.

### Off-grid PV group core measurements

- Kept P1 Max Power as the installed PV module nameplate DC power in kW and moved String Count from P5 to P2. Removed the other legacy properties.
- Replaced duplicate/ambiguous measurements with M101 DC Power and M102 PV Energy Total. Added daily/weekly/monthly/quarterly outputs at M201–M204, calculated from M102.
- Removed assumed operating/efficiency/percentage measurements and cleared all four actions.
- Default display is DC Power, PV Energy Total, and PV Daily Energy.
- Existing instances require migration by measurement boundary and meaning. The former PV Power (M143) may map to M101 only if it covers the same group. The former PV Energy Total (M136) maps to M102 under the same boundary. The former PV Daily Energy (M221) maps to M201 after checking the daily calculation basis.
- P2 previously meant Max Voltage; migrate the old String Count value from P5 before configuring the new P2. Remove references to deleted properties, measurements, and actions.
- Inverter-wide PV counters must not be copied to multiple PV groups. Station statistics must avoid double-counting the same PV energy at both PV group and inverter boundaries. Configure period rules separately.

### Off-grid generic load templates

- Updated Single_Phase_Load and Three_Phase_Load with shared AC Power, AC Reactive Power, AC Power Factor, AC Frequency, Load Energy Total, and daily/weekly/monthly/quarterly Load energy names.
- Kept four configured electrical limits/ratings as P1 Max Power, P2 Max Voltage, P3 Max Current, P4 Rated Frequency. Former P4/P1/P2/P3 map to new P1/P2/P3/P4. Removed load flexibility and scheduling properties P5–P10.
- Electrical measurements start at M101. Three-phase quantities follow the current Meter layout. The single-phase template retains matching IDs without duplicate active power. Period energy outputs M201–M204 use cumulative M114.
- Removed single-phase apparent power and three-phase demand measurements from the core templates. Actions remain empty. Display labels are Load (Single Phase) and Load (Three Phase).
- Default display includes AC Power, Load Energy Total, and Load Daily Energy. Route only measurements that cover the load boundary. Verify direction and unit conversion before mapping meter import energy.
- Migrate saved properties, routes, name-based selectors, rules/alarms, and history queries by point meaning and source version. Avoid double-counting the same branch energy through both Meter and Load instances. Configure period rules separately.

#### Single_Phase_Load

| Previous measurement | Previous ID | Final measurement |
| --- | --- | --- |
| AC Reactive Power | M103 | AC Reactive Power / M102 |
| AC Apparent Power | M104 | Removed |
| AC Power Factor | M105 | AC Power Factor / M103 |
| AC Frequency | M106 | AC Frequency / M104 |
| AC Voltage | M109 | AC Voltage / M105 |
| AC Current | M110 | AC Current / M108 |
| Load Energy Total | M137 | Load Energy Total / M114 |
| Load Power | M152 | AC Power / M101 |
| Load Daily Energy | M225 | Load Daily Energy / M201 |

#### Three_Phase_Load

| Previous measurement | Previous ID | Final measurement |
| --- | --- | --- |
| AC Power | M102 | AC Power / M101 |
| AC Reactive Power | M103 | AC Reactive Power / M102 |
| AC Power Factor | M105 | AC Power Factor / M103 |
| AC Frequency | M106 | AC Frequency / M104 |
| L1 Voltage | M111 | L1 Voltage / M105 |
| L2 Voltage | M112 | L2 Voltage / M106 |
| L3 Voltage | M113 | L3 Voltage / M107 |
| L1 Current | M114 | L1 Current / M108 |
| L2 Current | M115 | L2 Current / M109 |
| L3 Current | M116 | L3 Current / M110 |
| L1 Active Power | M117 | L1 Active Power / M111 |
| L2 Active Power | M118 | L2 Active Power / M112 |
| L3 Active Power | M119 | L3 Active Power / M113 |
| Load Energy Total | M137 | Load Energy Total / M114 |
| AC Current Demand | M158 | Removed |
| Load Daily Energy | M225 | Load Daily Energy / M201 |
| Load Monthly Energy | M227 | Load Monthly Energy / M203 |
| AC Max Demand This Month | M241 | Removed |

## 2026-11-23

### Product `displayName`

- Added root-level `displayName` on every product JSON for UI labels (`name` with underscores replaced by spaces).
- Renamed legacy `display_name` on `Three_Phase_Load` to `displayName`.

### Unified `enumData` (removed `options`)

- Removed all `options` fields. P parameters with predefined choices now use `type: "enum"` and `enumData` like M/A.
- Every `enumData` item includes `key`, `label`, and `value`; product templates keep `value` as an empty string.
- Migrated P enums: Load Flexibility (load products and Meter), HVAC Type, Charger Type, Station Type.

## 2026-11-19

### Point ID renumbering

P, M, and A each start at 1 and use consecutive IDs within each product. This renumbering retains all measurements listed below.

| Product | Measurement | Previous ID | New ID |
| --- | --- | --- | --- |
| `Battery` | BMS Charge Current Limit | M22 | M20 |
| `Battery` | BMS Discharge Current Limit | M23 | M21 |
| `Battery` | Online Battery Pack Count | M24 | M22 |
| `Battery` | Daily Charge Energy | M101 | M23 |
| `Battery` | Daily Discharge Energy | M102 | M24 |
| `Battery` | Weekly Charge Energy | M103 | M25 |
| `Battery` | Weekly Discharge Energy | M104 | M26 |
| `Battery` | Monthly Charge Energy | M105 | M27 |
| `Battery` | Monthly Discharge Energy | M106 | M28 |
| `Battery` | Quarterly Charge Energy | M107 | M29 |
| `Battery` | Quarterly Discharge Energy | M108 | M30 |
| `Hybrid_Inverter` | Load Power | M8 | M7 |
| `Hybrid_Inverter` | Inverter Output Frequency | M11 | M8 |
| `Hybrid_Inverter` | Heat Sink Temperature | M13 | M9 |
| `Hybrid_Inverter` | PV Energy Today | M15 | M10 |
| `Hybrid_Inverter` | PV Energy Total | M16 | M11 |
| `Hybrid_Inverter` | Battery Charge Energy Total | M18 | M12 |
| `Hybrid_Inverter` | Battery Discharge Energy Total | M20 | M13 |
| `Hybrid_Inverter` | Operating Status | M27 | M14 |
| `Hybrid_Inverter` | Start Stop Status | M28 | M15 |
| `Single_Phase_Load` | Energy Used Today | M10 | M9 |

Apply the mappings by point meaning and source template version when migrating instance routes, rules, alarms, or history queries. For Battery templates that still use legacy M20/M21 for today-energy values, those routes must be reviewed separately before switching to the new template. The new M20/M21 are BMS current limits.

### V1.3.0 enum templates

- Migrated three M status points from `type: "string"` with `options` to `type: "enum"` with `enumData`: EV charging status (M9), HVAC operating mode (M12), and hybrid inverter operating status (M14).
- P parameters with predefined choices use `type: "string"` with `options`.
- Preserved the existing option order and defined each option with a point-local string key and an English label. Actual values are supplied during device configuration.

### Discrete action templates

- Added one enum option to each existing start, stop, fault reset, emergency stop and fire protection action across eight products.
- Kept start and stop as separate actions with their existing point IDs. Each instance supplies the value to dispatch through its configured action route.

### Default measurements

- Reduced the default display to two or three key measurements per product using `M[].attr.isDisplay`.

### Off-grid measurements

- Added BMS charge and discharge current limits (M20/M21) and online battery pack count (M22) to Battery.
- Removed the legacy Battery M20/M21 today-energy measurements. The daily, weekly, monthly, and quarterly rule outputs are now M23–M30.
- Renamed Diesel M13 to Fuel Level and M14 to Coolant Temperature.

## 2026-10-29

### Battery connection count

- Removed the Battery-side connection maximum for the combined `Hybrid_Inverter` / `PCS` group (`max: null`).
- A Battery may connect to multiple PCS devices, multiple Hybrid Inverters, or a mixture of both, with no product-level maximum.
- Kept the minimum of one connection and all P/M/A point definitions unchanged.
- Updated the topology description and README examples to match the rule.

## 2026-09-14

### Display-point metadata

- Removed product-level `defaultDisplayMeasureIds` arrays from all 13 product definitions.
- Added `attr.isDisplay: "true"` directly to the Measure points that were previously selected by those arrays.
- Kept the selected Measure sets unchanged; only the representation moved from product-level references to point-local metadata.
- Updated the product contract and validation guidance in `README.md`.

## 2026-09-08

### Change statistics

| Item | Count |
| --- | ---: |
| Current product definitions | 13 |
| New product definitions | 1 (`Meter`) |
| Removed product definitions | 5 |
| Updated retained product definitions | 12 |
| Updated documentation files | 1 |

### Product catalog

- Added `Meter` as an independent product for topology metering and electrical aggregation.
- Removed obsolete placeholder or container definitions: `ESS`, `Load`, `Generator`, `Inverter`, and `Distribution_Board`.
- Removed product-level `pName` and `canCreateInstance` fields.
- Removed topology `enabled`, `type`, `components`, and `connectableProducts` fields.
- Standardized all retained topology products as independent device nodes; `Hybrid_Inverter` is represented as one device instead of a Composite with internal nodes.
- Added product `type`, product descriptions, and flat default `defaultDisplayMeasureIds` measurement lists.
- Kept the existing Battery P/M/A point IDs and definitions; this release only adds description metadata.

### Field changes

| Scope | Field | Change | Notes |
| --- | --- | --- | --- |
| Product | `type` | Added | Product classification used by the gateway and topology library. |
| Product | `description` | Added or standardized | English product description without unit text. |
| Product | `pName` | Removed | The stable product identifier is `name`. |
| Product | `canCreateInstance` | Removed | All retained device products use the same standalone-node model. |
| Product | `defaultDisplayMeasureIds` | Added | Flat list of 3–5 default Measure IDs; no primary/detail hierarchy. |
| Topology | `topology.image` | Retained | Product image resource reference. |
| Topology | `topology.connections` | Added | Node-wide connection-rule groups. |
| Topology | `topology.description` | Added | English business explanation displayed below the connection-rule tooltip. |
| Topology | `topology.enabled` | Removed | Presence of `topology` now determines whether a product participates in the topology library. |
| Topology | `topology.type` | Removed | Standalone/composite/container categorization is no longer stored in products. |
| Topology | `topology.components` | Removed | Hybrid Inverter no longer creates internal component nodes. |
| Topology | `topology.connectableProducts` | Removed | Replaced by reciprocal connection groups. |
| Connection rule | `products` | Added | Explicit list of permitted peer product names for the current node. |
| Connection rule | `min` | Added | Minimum number of connected peer instances across the whole product group. |
| Connection rule | `max` | Added | Maximum number of connected peer instances across the whole product group; `null` means unlimited. |
| Connection rule | rule-level `description` | Removed | Tooltip explanation belongs only to `topology.description`. |
| Point | `description` | Added or standardized | English point description without unit text; units remain in `unit`. |

### Connection rules

- Replaced legacy one-sided product connection lists with reciprocal `topology.connections` groups using `products`, `min`, and `max`.
- Defined connection constraints at the whole-node level. The four canvas handles remain visually selectable but are business-equivalent.
- Added one topology-level connection description per participating product for the connection-rule tooltip.
- Added Meter as the electrical aggregation node. Its rule allows the supported source and load products without imposing a Meter-side connection count.
- Removed `topology` from `Station` and `Env`; they are excluded from the product drag library and energy connection graph.

### Point metadata

- Added descriptions to product, topology, Property, Measure, and Action definitions.
- Standardized every JSON `description` as English text without unit text. Units remain exclusively in the `unit` field.
- Simplified Hybrid Inverter measurements and actions around PV conversion, PCS functions, battery status, and Start/Stop status.

### Documentation

- Reworked `README.md` to document the simplified product model, node-wide connection-group semantics, tooltip behavior, and display-point rules.

## Entry format

For each subsequent version, add a new `## YYYY-MM-DD` section above this entry with:

- a compact change-statistics table;
- added, removed, and changed product definitions;
- connection-rule or topology behavior changes;
- point-definition and metadata changes;
- affected documentation or validation changes.
