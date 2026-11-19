# Product Library Change Log

Version identifiers use the release date in `YYYY-MM-DD` format. New entries are added at the top and describe changes relative to the preceding recorded version.

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
