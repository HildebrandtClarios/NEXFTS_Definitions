# NEXFTS Test Definition — Syntax Reference

> **NEXFTS** – Neutral Exchange Format for Battery Test Sequences
> Schema version: **0.3.0**
> Schema: [`nexfts.schema.json`](../nexfts_schema/nexfts.schema.json)

---

## Table of Contents

- [NEXFTS Test Definition — Syntax Reference](#nexfts-test-definition--syntax-reference)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [1. `nexfts_schema_version`](#1-nexfts_schema_version)
    - [Versioning scheme](#versioning-scheme)
    - [Version history](#version-history)
  - [2. `metadata`](#2-metadata)
  - [3. `global_values`](#3-global_values)
  - [4. `global_limits`](#4-global_limits)
  - [5. `steps`](#5-steps)
    - [Required fields per step](#required-fields-per-step)
    - [Step types](#step-types)
    - [5.1 `rest` – Rest phase](#51-rest--rest-phase)
    - [5.2 `discharge` – Discharge](#52-discharge--discharge)
    - [5.3 `charge` – Charge](#53-charge--charge)
    - [5.4 `begin` – Loop start](#54-begin--loop-start)
    - [5.5 `cycle` – Loop end](#55-cycle--loop-end)
    - [5.6 `sub` – Subroutine call](#56-sub--subroutine-call)
    - [5.7 `goto` – Unconditional jump](#57-goto--unconditional-jump)
    - [5.8 `set` – Set temperature](#58-set--set-temperature)
    - [5.9 `pause` – Planned test interruption (since 0.3.0)](#59-pause--planned-test-interruption-since-030)
  - [6. Step Attributes](#6-step-attributes)
    - [6.1 `temperature`](#61-temperature)
    - [6.2 `duration`](#62-duration)
    - [6.3 `current` / `voltage`](#63-current--voltage)
    - [6.4 `limit`](#64-limit)
      - [Available limit conditions](#available-limit-conditions)
      - [6.4.1 Simple conditions: `voltage` / `current` / `time` / `charge`](#641-simple-conditions-voltage--current--time--charge)
      - [6.4.2 `cumulative_charge` — Charge sum across explicit steps](#642-cumulative_charge--charge-sum-across-explicit-steps)
      - [Allowed operators](#allowed-operators)
    - [6.5 `result`](#65-result)
  - [7. Expressions and Global Value References (since 0.2.0)](#7-expressions-and-global-value-references-since-020)
    - [7.1 Where expressions are used](#71-where-expressions-are-used)
    - [7.2 Supported expression syntax](#72-supported-expression-syntax)
    - [7.3 Example](#73-example)
  - [Example files](#example-files)
    - [Cold-crank / EN standard](#cold-crank--en-standard)
    - [EN 50342-6](#en-50342-6)
    - [IEC 60095-8 – Power Performance Characteristics (PPC)](#iec-60095-8--power-performance-characteristics-ppc)
    - [IEC 60095-8 – Charge Recovery (CR) Test](#iec-60095-8--charge-recovery-cr-test)
    - [IEC 60095-8 – Endurance Tests](#iec-60095-8--endurance-tests)

---

## Overview

A NEXFTS test definition is a YAML file that completely describes a battery test.
It consists of five mandatory sections:

| Section | Required | Description |
|---|---|---|
| `nexfts_schema_version` | ✅ | Schema version against which the file is validated |
| `metadata` | ✅ | Name, version and description of the test |
| `global_values` | ✅ | Global variables (e.g. rated current, rated capacity) |
| `global_limits` | ✅ | Global safety limits |
| `steps` | ✅ | Sequence of test steps (at least 1) |

---

## 1. `nexfts_schema_version`

Specifies which schema version the file was written against.

```yaml
nexfts_schema_version: "0.3.0"
nexfts_schema_version: "0.3.0"
```

| Field | Type | Required | Format |
|---|---|---|---|
| `nexfts_schema_version` | `string` | ✅ | `"X.Y"` or `"X.Y.Z"` |

### Versioning scheme

The NEXFTS schema follows **Semantic Versioning** (SemVer) with three numeric components:

```
MAJOR.MINOR.PATCH
  │     │     └── Bug fixes / documentation corrections (no schema changes)
  │     └──────── New features, backward-compatible extensions
  └────────────── Breaking changes (existing files may become invalid)
```

| Increment | When | Backward-compatible? | Example |
|---|---|---|---|
| **PATCH** (0.1.0 → 0.1.1) | Typo fixes in descriptions, clarified documentation, no structural change | ✅ Yes | Corrected `description` text in schema |
| **MINOR** (0.1.1 → 0.2.0) | New optional fields, extended `enum` lists, relaxed type constraints | ✅ Yes | `duration.value` accepts expressions; `global_values.unit` gains time units |
| **MAJOR** (0.x → 1.0.0) | Removed fields, renamed keys, tightened constraints that invalidate existing files | ❌ No | Hypothetical: renaming `global_values` to `variables` |

> Every NEXFTS YAML file declares the schema version it was written against. The declaration identifies the intended schema version; it does not by itself guarantee compatibility with a different schema version.

> **Current version:** `0.3.0` — The leading `0` indicates that the schema is still in its initial development phase. Breaking changes may occur before `1.0.0` without a major version bump.

### Version history

| Version | Date | Key changes |
|---|---|---|
| `0.1.0` | 2025 | Initial schema: `rest`, `discharge`, `charge`, `begin`, `cycle`, `sub`, `goto`, `set` step types |
| `0.1.1` | 2025 | Added `cumulative_charge` limit; `medium` field in `temperature` |
| `0.2.0` | 2026-06-22 | `global_values.value` and `duration.value` accept expressions; `global_values.unit` extended with time units |
| `0.3.0` | 2026-06-22 | New step type `pause` (planned test interruption); `rest` supports optional `limit` with voltage monitoring and `goto` |

---

## 2. `metadata`

Describes the test and optionally contains a standards reference.

```yaml
metadata:
  name: "cranking_performance_EN"
  version: "1.0.0"
  description: "Cranking performance test according to EN 50342-1"
  standard:                         # optional
    name: "EN 50342-1"              # required if standard is specified
    title: "Lead-acid starter batteries - General requirements and methods of test"
    date: "2015-04"
    clause: "Clause 9.1"
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | ✅ | Unique name of the test |
| `version` | `string` | ✅ | Version of the test definition |
| `description` | `string` | ✅ | Short description of the test |
| `standard` | `object` | ⬜ | Standards reference |
| `standard.name` | `string` | ✅ (if `standard`) | Standard identifier, e.g. `"EN 50342-1"` |
| `standard.title` | `string` | ⬜ | Full title of the standard |
| `standard.date` | `string` | ⬜ | Publication date, e.g. `"2015-04"` |
| `standard.clause` | `string` | ⬜ | Referenced clause, e.g. `"Clause 9.1"` |

---

## 3. `global_values`

Defines named variables that can be referenced in step expressions (e.g. `"Icc"`, `"0.6 * Icc"`).

```yaml
global_values:
  Icc:
    value: 680
    unit: "A"
    description: "Cold cranking current (EN 50342-1)"   # optional
  Uc:
    value: 14.4
    unit: "V"
    description: "Charge voltage"
  tDCH:
    value: "(0.02 * Cn - 0.083) / 48 * 3600"
    unit: "s"
    description: "Computed discharge duration referencing other global_values"
  tDCH:
    value: "(0.02 * Cn - 0.083) / 48 * 3600"
    unit: "s"
    description: "Computed discharge duration referencing other global_values"
```

| Field | Type | Required | Allowed values |
|---|---|---|---|
| `value` | `number` or `string` | ✅ | Numeric value **or** expression referencing other `global_values` keys, e.g. `"(0.02 * Cn - 0.083) / 48 * 3600"` |
| `unit` | `string` | ✅ | `"A"`, `"V"`, `"Ah"`, `"s"`, `"seconds"`, `"min"`, `"minutes"`, `"h"`, `"hours"` |
| `value` | `number` or `string` | ✅ | Numeric value **or** expression referencing other `global_values` keys, e.g. `"(0.02 * Cn - 0.083) / 48 * 3600"` |
| `unit` | `string` | ✅ | `"A"`, `"V"`, `"Ah"`, `"s"`, `"seconds"`, `"min"`, `"minutes"`, `"h"`, `"hours"` |
| `description` | `string` | ⬜ | Free text |

> **Expression support (since 0.2.0):** The `value` field accepts arithmetic expressions as strings. These expressions may reference other `global_values` keys (e.g. `Cn`); see [Expressions and Global Value References](#7-expressions-and-global-value-references-since-020).
>
> **Time units (since 0.2.0):** The `unit` field now also accepts time units (`"s"`, `"seconds"`, `"min"`, `"minutes"`, `"h"`, `"hours"`), enabling duration-type global values such as computed discharge times.

> **Unit consistency:** When a `global_values` key is referenced in a step field's `value` expression (e.g. `value: "Icc"`), use the same unit for the step field as for the referenced key. NEXFTS expressions do not perform unit conversion.

---

## 4. `global_limits`

Defines global safety limits for the entire test.

```yaml
global_limits:
  max_voltage:
    value: 16.5
    unit: "V"
  max_current:
    value: 300
    unit: "A"
  max_temperature:
    value: 60
    unit: "degC"
```

| Field | Type | Allowed units |
|---|---|---|
| `max_voltage` | `object` | `"V"`, `"mV"` |
| `max_current` | `object` | `"A"`, `"mA"` |
| `max_temperature` | `object` | `"degC"` |

Each object has the required fields `value` (`number`) and `unit` (`string`).

---

## 5. `steps`

Sequence of test steps. At least one step is required.
Every step has at minimum the required fields `id` and `type`.
Sequence of test steps. At least one step is required.
Every step has at minimum the required fields `id` and `type`.
Further required fields depend on the step type (see table below).

### Required fields per step

| Field | Type | Description |
|---|---|---|
| `id` | `integer` ≥ 1 | Unique step ID |
| `type` | `string` | Step type (see below) |

### Step types

| Type | Description | Required fields |
|---|---|---|
| `rest` | Rest phase | `id`, `type`, `temperature`, `duration` |
| `discharge` | Discharge | `id`, `type`, `temperature`, `current` **or** `voltage`, `duration` and/or `limit` |
| `charge` | Charge | `id`, `type`, `temperature`, `voltage`, `current`, `duration` and/or `limit` |
| `begin` | Start of a loop | `id`, `type`, `level`, `comment` |
| `cycle` | End of a loop | `id`, `type`, `level`, `count` |
| `sub` | Subroutine call | `id`, `type`, `name` |
| `goto` | Unconditional jump to a step | `id`, `type`, `target_id` |
| `set` | Set temperature | `id`, `type`, `temperature` (optional: `wait`) |
| `pause` | Planned test interruption for manual operations | `id`, `type`, `comment` |
| `pause` | Planned test interruption for manual operations | `id`, `type`, `comment` |

---

### 5.1 `rest` – Rest phase

Pauses the test for a defined duration. No current flows; the temperature is maintained.

```yaml
# Basic rest – fixed duration only
# Basic rest – fixed duration only
- id: 1
  type: rest
  temperature:
    value: 25
    unit: "degC"
    medium: "air"
  duration:
    value: 24
    unit: "hours"
  result:
    - name: "U_OCV"
      description: "Open-circuit voltage at end of rest"
  comment: "Initial rest at 25°C"
```

```yaml
# Rest with voltage monitoring and conditional goto
- id: 10
  type: rest
  temperature:
    value: 40
    unit: "degC"
    medium: "water"
  duration:
    value: 3
    unit: "h"
  limit:
    voltage:
      operator: less_than
      value: 11.0
      unit: "V"
    goto:
      target_id: 20
      comment: "Jump to check-up if voltage drops below 11.0 V during rest"
  result:
    - name: "U(3h)"
      description: "Voltage after 3 h rest"
  comment: "Relaxation 3 h; abort and jump to step 20 if U < 11.0 V"
```

```yaml
# Rest with voltage monitoring and conditional goto
- id: 10
  type: rest
  temperature:
    value: 40
    unit: "degC"
    medium: "water"
  duration:
    value: 3
    unit: "h"
  limit:
    voltage:
      operator: less_than
      value: 11.0
      unit: "V"
    goto:
      target_id: 20
      comment: "Jump to check-up if voltage drops below 11.0 V during rest"
  result:
    - name: "U(3h)"
      description: "Voltage after 3 h rest"
  comment: "Relaxation 3 h; abort and jump to step 20 if U < 11.0 V"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"rest"` |
| `temperature` | ✅ | Test temperature (see [6.1 `temperature`](#61-temperature)) |
| `duration` | ✅ | Rest duration (see [6.2 `duration`](#62-duration)) |
| `limit` | ⬜ | Optional abort criterion during rest (see [6.4 `limit`](#64-limit)) |
| `limit` | ⬜ | Optional abort criterion during rest (see [6.4 `limit`](#64-limit)) |
| `result` | ⬜ | Measurement results to be recorded (see [6.5 `result`](#65-result)) |
| `comment` | ⬜ | Free-text description |

> **`limit` on rest steps (since 0.3.0):** While `duration` defines the planned rest time, an optional `limit` can trigger an early abort — typically a voltage threshold to detect cell failure during long rest phases. Combined with `limit.goto`, this enables conditional branching (e.g. jump to a check-up sequence if the battery self-discharges abnormally).
>
> | Variant | `duration` | `limit` | Behaviour |
> |---|---|---|---|
> | Fixed duration only | ✅ | — | Rest ends after timer expires |
> | Duration + voltage monitoring | ✅ | ✅ | Rest ends after timer **or** early if voltage condition is met |

> **`limit` on rest steps (since 0.3.0):** While `duration` defines the planned rest time, an optional `limit` can trigger an early abort — typically a voltage threshold to detect cell failure during long rest phases. Combined with `limit.goto`, this enables conditional branching (e.g. jump to a check-up sequence if the battery self-discharges abnormally).
>
> | Variant | `duration` | `limit` | Behaviour |
> |---|---|---|---|
> | Fixed duration only | ✅ | — | Rest ends after timer expires |
> | Duration + voltage monitoring | ✅ | ✅ | Rest ends after timer **or** early if voltage condition is met |

---

### 5.2 `discharge` – Discharge

Discharges the battery either at a **constant current (CC)** or at a **constant voltage (CV)**. Exactly one of `current` or `voltage` must be specified. The step ends after a fixed duration, when an abort criterion is reached, or both.

```yaml
# Variant A: CC discharge, time-controlled (fixed duration)
- id: 5
  type: discharge
  temperature:
    value: -18
    unit: "degC"
    medium: "air"
  current:
    value: "0.33 * In"
    unit: "A"
  duration:
    value: 6
    unit: "s"
  comment: "Pulsed CC discharge 6 s"
```

```yaml
# Variant B: CC discharge, condition-controlled (abort on voltage drop)
- id: 3
  type: discharge
  temperature:
    value: -18
    unit: "degC"
    medium: "air"
  current:
    value: "Icc"
    unit: "A"
  limit:
    voltage:
      operator: less_than
      value: 9.0
      unit: "V"
  comment: "CC discharge until voltage drops below 9.0 V"
```

```yaml
# Variant C: CC discharge, combined (target duration + early abort on voltage drop)
- id: 3
  type: discharge
  temperature:
    value: -18
    unit: "degC"
    medium: "air"
  current:
    value: "Icc"
    unit: "A"
  duration:
    value: 10
    unit: "seconds"
  limit:
    voltage:
      operator: less_than
      value: 9.5
      unit: "V"
  comment: "CC discharge 10 s, abort early if voltage drops below 9.5 V"
```

```yaml
# Variant D: CV discharge (constant voltage, no current setpoint)
- id: 4
  type: discharge
  temperature:
    value: 25
    unit: "degC"
    medium: "air"
  voltage:
    value: 10.5
    unit: "V"
  limit:
    current:
      operator: less_than
      value: 0.5
      unit: "A"
  comment: "CV discharge at 10.5 V until current drops below 0.5 A"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"discharge"` |
| `temperature` | ✅ | Test temperature (see [6.1 `temperature`](#61-temperature)) |
| `current` | ✅² | Discharge current for CC mode (see [6.3 `current` / `voltage`](#63-current--voltage)) |
| `voltage` | ✅² | Discharge voltage setpoint for CV mode (see [6.3 `current` / `voltage`](#63-current--voltage)) |
| `duration` | ✅¹ | Target discharge duration (see [6.2 `duration`](#62-duration)) |
| `limit` | ✅¹ | Abort criterion (see [6.4 `limit`](#64-limit)) |
| `result` | ⬜ | Measurement results to be recorded (see [6.5 `result`](#65-result)) |
| `comment` | ⬜ | Free-text description |

> ¹ At least one of `duration` or `limit` is required. Both together are recommended for standardised discharge steps: `duration` = planned test duration (normal case), `limit` = early abort on battery failure (exception case).
>
> ² Exactly one of `current` or `voltage` is required:
> - **CC discharge** (`current`): Battery is discharged at a constant current. Typical use: cold-crank, capacity test.
> - **CV discharge** (`voltage`): Battery is discharged at a constant voltage setpoint. Typical use: voltage-controlled drain, deep-discharge to a defined endpoint voltage.

---

### 5.3 `charge` – Charge

Charges the battery with a defined voltage and a maximum current. As with `discharge`, the step can be time-controlled, condition-controlled, or combined.

```yaml
# Variant A: time-controlled (fixed charge duration)
- id: 6
  type: charge
  temperature:
    value: 25
    unit: "degC"
    medium: "water"
  voltage:
    value: "Uc"
    unit: "V"
  current:
    value: "5 * In"
    unit: "A"
  duration:
    value: 24
    unit: "hours"
  comment: "Final charge at 25°C, 24 h"
```

```yaml
# Variant B: condition-controlled (IU charge until tail current)
- id: 6
  type: charge
  temperature:
    value: 25
    unit: "degC"
    medium: "water"
  voltage:
    value: 14.4
    unit: "V"
  current:
    value: "5 * In"
    unit: "A"
  limit:
    logic: AND
    current:
      operator: less_than
      value: 2.0
      unit: "A"
  comment: "IU charge – stop when tail current drops below 2 A"
```

```yaml
# Variant C: combined (maximum charge time + abort at tail current)
- id: 6
  type: charge
  temperature:
    value: 25
    unit: "degC"
    medium: "water"
  voltage:
    value: 14.4
    unit: "V"
  current:
    value: "5 * In"
    unit: "A"
  duration:
    value: 24
    unit: "hours"
  limit:
    current:
      operator: less_than
      value: 2.0
      unit: "A"
  comment: "IU charge – max 24 h or until tail current drops below 2 A"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"charge"` |
| `temperature` | ✅ | Test temperature (see [6.1 `temperature`](#61-temperature)) |
| `voltage` | ✅ | Charge voltage (see [6.3 `current` / `voltage`](#63-current--voltage)) |
| `current` | ✅ | Maximum charge current (see [6.3 `current` / `voltage`](#63-current--voltage)) |
| `duration` | ✅¹ | Maximum charge duration (see [6.2 `duration`](#62-duration)) |
| `limit` | ✅¹ | Abort criterion, e.g. tail current (see [6.4 `limit`](#64-limit)) |
| `result` | ⬜ | Measurement results to be recorded (see [6.5 `result`](#65-result)) |
| `comment` | ⬜ | Free-text description |

> ¹ At least one of `duration` or `limit` is required.

---

### 5.4 `begin` – Loop start

Marks the start of a loop block. The associated `cycle` element must carry the same `level` value. Loops can be nested (using different `level` values).

```yaml
- id: 4
  type: begin
  level: 1
  comment: "Begin loop – repeat steps 05–06 for 4320 times"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"begin"` |
| `level` | ✅ | Nesting depth (`integer` ≥ 1); `1` = outermost loop |
| `comment` | ✅ | Description of the loop |

> `begin` must **not** contain a `temperature` field.
> `begin` must **not** contain a `temperature` field.
> The number of iterations is specified in the associated [`cycle` step](#55-cycle--loop-end) via `count`.

---

### 5.5 `cycle` – Loop end

Marks the end of a loop block and specifies the number of repetitions. After `count` iterations, execution continues with the next step after `begin`.

```yaml
- id: 7
  type: cycle
  level: 1
  count: 4320
  comment: "End loop – return to begin (step 04) until 4320 repetitions completed"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"cycle"` |
| `level` | ✅ | Must match the `level` of the corresponding `begin` |
| `count` | ✅ | Number of loop iterations (`integer` ≥ 1) |
| `comment` | ⬜ | Free-text description |

> `cycle` must **not** contain a `temperature` field.

---

### 5.6 `sub` – Subroutine call

Calls a named sub-sequence. The subroutine is defined as a standalone YAML file or a named block and is executed in full before the main sequence continues.

```yaml
- id: 3
  type: sub
  name: "PP1"
  comment: "Call sub-sequence PP1 – Pulse Profile 1"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"sub"` |
| `name` | ✅ | Name of the subroutine to call, e.g. `"PP1"` |
| `comment` | ⬜ | Free-text description |

> `sub` must **not** contain a `temperature` field.

---

### 5.7 `goto` – Unconditional jump

Jumps unconditionally to another step in the sequence. For **conditional** jumps (triggered when a limit is reached) see [`limit.goto`](#64-limit) in section 6.4.

```yaml
- id: 8
  type: goto
  target_id: 3
  comment: "Jump back to step 3"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"goto"` |
| `target_id` | ✅ | ID of the target step (`integer` ≥ 1) |
| `comment` | ⬜ | Free-text description |

> `goto` must **not** contain a `temperature` field.

---

### 5.8 `set` – Set temperature

Changes the test temperature at an arbitrary point in the sequence without performing any electrical operation. Use this step whenever the target temperature must be changed between two other steps.

The `wait` attribute controls whether the sequence is held at this step until the climate system has reached the setpoint, or whether it continues immediately after the setpoint command is issued.

```yaml
# Variant A: continue immediately after sending the setpoint command (default)
- id: 9
  type: set
  temperature:
    value: -18
    unit: "degC"
    medium: "air"
  wait: false
  comment: "Start cooling to -18°C; next step begins immediately (e.g. a long rest covers the soak time)"
```

```yaml
# Variant B: wait for chamber to reach setpoint before continuing
- id: 9
  type: set
  temperature:
    value: 25
    unit: "degC"
    medium: "air"
  wait: true
  comment: "Switch chamber to 25°C and wait until setpoint is reached"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"set"` |
| `temperature` | ✅ | Target temperature (see [6.1 `temperature`](#61-temperature)) |
| `wait` | ⬜ | `false` *(default)*: continue immediately · `true`: hold until setpoint reached |
| `comment` | ⬜ | Free-text description |

> **`wait: false` (default):** The setpoint command is issued and the sequence continues immediately to the next step. This is useful when the temperature change takes a long time and the next step (e.g. a `rest`) can cover the soak time, or when the test procedure explicitly requires that the sequence not wait.
>
> **`wait: true`:** The test system pauses at this step and waits for the climate chamber or water basin to confirm that the temperature setpoint has been reached before advancing to the next step. Use this when the subsequent step is sensitive to temperature (e.g. a discharge at a defined cold temperature) and no separate soak `rest` follows.
>
> `set` does **not** carry `duration`, `current`, `voltage`, or `limit`.

---

### 5.9 `pause` – Planned test interruption (since 0.3.0)

Interrupts the test program and waits for operator confirmation before continuing. Unlike `rest`, the duration is **unknown** — the test system holds until the operator confirms continuation.

Typical use cases: manual measurements, visual inspection, electrolyte level adjustment, sensor recalibration, or any manual intervention that must be completed before the test continues.

```yaml
- id: 5
  type: pause
  comment: "Weigh battery and record electrolyte level before continuing"
```

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique step ID (`integer` ≥ 1) |
| `type` | ✅ | `"pause"` |
| `comment` | ✅ | Description of the manual operation to be performed |

> `pause` does **not** carry `temperature`, `duration`, `current`, `voltage`, or `limit`.
>
> **Difference from `rest`:**
>
> | | `rest` | `pause` |
> |---|---|---|
> | Duration | Defined (e.g. 3 h) | Unknown — operator-controlled |
> | Continuation | Automatic after timer expires | Manual (operator presses start) |
> | Temperature | Maintained at setpoint | Not controlled (no `temperature` field) |
> | Electrical | No current, OCV monitored | No current, no monitoring |
> | Use case | Relaxation, thermal soak | Manual inspection, measurements, adjustments |
>
---

## 6. Step Attributes

This section describes all attributes that can be used within step definitions.
Each attribute is shared across multiple step types; refer to the per-step tables in [section 5](#5-steps) to see which attributes apply to which step type.

---

### 6.1 `temperature`

```yaml
temperature:
  value: -18
  unit: "degC"
  medium: "air"     # optional
```

| Field | Type | Required | Allowed values |
|---|---|---|---|
| `value` | `number` | ✅ | Temperature in degrees Celsius |
| `unit` | `string` | ✅ | `"degC"` |
| `medium` | `string` | ⬜ | `"air"`, `"water"` |

---

### 6.2 `duration`

```yaml
duration:
  value: 24
  unit: "hours"
```

```yaml
# Expression referencing a global_values key (since 0.2.0)
duration:
  value: "1 + tDCH"
  unit: "s"
```

```yaml
# Expression referencing a global_values key (since 0.2.0)
duration:
  value: "1 + tDCH"
  unit: "s"
```

| Field | Type | Required | Allowed values |
|---|---|---|---|
| `value` | `number` or `string` | ✅ | Numeric value **or** expression referencing `global_values` keys, e.g. `"tDCH"`, `"1 + tDCH"` |
| `value` | `number` or `string` | ✅ | Numeric value **or** expression referencing `global_values` keys, e.g. `"tDCH"`, `"1 + tDCH"` |
| `unit` | `string` | ✅ | `"s"`, `"seconds"`, `"min"`, `"minutes"`, `"h"`, `"hours"` |

> **Expression support (since 0.2.0):** Like `current.value` and `voltage.value`, `duration.value` now also accepts string expressions that reference `global_values` keys. This allows computed durations that automatically adapt when a referenced global value changes.

> **Expression support (since 0.2.0):** Like `current.value` and `voltage.value`, `duration.value` now also accepts string expressions that reference `global_values` keys. This allows computed durations that automatically adapt when a referenced global value changes.

---

### 6.3 `current` / `voltage`

```yaml
current:
  value: "Icc"        # number or expression using global_values
  unit: "A"

voltage:
  value: "Uc"         # number or expression using global_values
  unit: "V"
```

| Field | Type | Required | Allowed values |
|---|---|---|---|
| `current.value` | `number` or `string` | ✅ | Number or expression, e.g. `"0.6 * Icc"` |
| `current.unit` | `string` | ✅ | `"A"`, `"mA"` |
| `voltage.value` | `number` or `string` | ✅ | Number or expression, e.g. `"Uc"` |
| `voltage.unit` | `string` | ✅ | `"V"`, `"mV"` |

---

### 6.4 `limit`

Defines abort conditions for a step. Multiple conditions are combined using `logic`.
If no condition is met, execution continues with the next step.
Defines abort conditions for a step. Multiple conditions are combined using `logic`.
If no condition is met, execution continues with the next step.
The optional `goto` attribute allows jumping to any step when the condition is satisfied.

> **Note:** `limit` is used exclusively as a **cut-off criterion** (e.g. voltage drops below a threshold, current falls below the tail-current level).
> **Note:** `limit` is used exclusively as a **cut-off criterion** (e.g. voltage drops below a threshold, current falls below the tail-current level).
> The **duration** of a step is specified via `duration`. For `discharge` and `charge` steps the same three variants apply:
>
> | Variant | `duration` | `limit` | Semantics |
> |---|---|---|---|
> | Time-controlled | ✅ | — | Fixed run time, no abort criterion |
> | Condition-controlled | — | ✅ | Abort at threshold (e.g. voltage drop, tail current reached) |
> | Combined | ✅ | ✅ | Fixed target duration **and** early abort at threshold |
>
> **Recommendations:**
> - `discharge`: The combination `duration` + `limit.voltage` is the semantically clearest form for standardised discharge steps – `duration` describes the planned test duration (normal case), `limit` defines the early abort on battery failure (exception case).
> - `charge`: For an IU charge (constant voltage until tail current) only `limit.current` is needed. An additional `duration` can serve as a maximum charge time.
> - A `limit.time` as the **only** criterion should be replaced by `duration`.

#### Available limit conditions

| Condition | Tracks | Typical use case |
|---|---|---|
| `voltage` | Terminal voltage | Discharge cutoff, charge threshold |
| `current` | Charge/discharge current | Tail-current detection for IU charge |
| `time` | Elapsed time in step | Use `duration` instead where possible |
| `charge` | Accumulated Ah **within this step** | Charge ratio CR per individual step |
| `cumulative_charge` | Sum of Ah **across explicit steps** | Charge ratio CR spanning multiple steps (e.g. main charge + finish charge) |
| `goto` | — | Jump to another step when any condition is met |

---

#### 6.4.1 Simple conditions: `voltage` / `current` / `time` / `charge`

All four conditions share the same structure: `operator`, `value`, `unit`.
They can be combined freely within a `limit` block using `logic`.

```yaml
limit:
  logic: OR   # OR = first satisfied condition triggers abort (default), AND = all must be satisfied
  voltage:
    operator: less_than
    value: 9.5
    unit: "V"
  time:
    operator: equal_to
    value: 10
    unit: "seconds"
  charge:
    operator: greater_than_or_equal
    value: "Q_CR_min"   # reference to a global_values key
    unit: "Ah"
  goto:                 # optional: jump when condition is satisfied
    target_id: 5
    comment: "Jump to step 5 if limit is reached"
```

| Field | Type | Required | Allowed values |
|---|---|---|---|
| `logic` | `string` | ⬜ | `"OR"`, `"AND"` (default: `"OR"`) |
| `voltage.operator` | `string` | ⬜ | see [Operators](#allowed-operators) |
| `voltage.value` | `number` | ⬜ | Any number |
| `voltage.unit` | `string` | ⬜ | `"V"`, `"mV"` |
| `current.operator` | `string` | ⬜ | see [Operators](#allowed-operators) |
| `current.value` | `number` | ⬜ | Any number |
| `current.unit` | `string` | ⬜ | `"A"`, `"mA"` |
| `time.operator` | `string` | ⬜ | see [Operators](#allowed-operators) |
| `time.value` | `number` | ⬜ | Any number |
| `time.unit` | `string` | ⬜ | `"s"`, `"seconds"`, `"min"`, `"minutes"`, `"h"`, `"hours"` |
| `charge.operator` | `string` | ⬜ | see [Operators](#allowed-operators) |
| `charge.value` | `number` or `string` | ⬜ | Number or `global_values` key expression |
| `charge.unit` | `string` | ⬜ | `"Ah"`, `"mAh"` |
| `goto.target_id` | `integer` ≥ 1 | ✅ (if `goto`) | ID of the target step when condition is met |
| `goto.comment` | `string` | ⬜ | Optional description |

> **`charge`** tracks the charge accumulated **within the current step only**.
> To aggregate charge across multiple steps use `cumulative_charge` (see [6.4.2](#642-cumulative_charge--charge-sum-across-explicit-steps)).

> If `limit.goto` is **not** specified, execution continues with the next step after the condition is met.
> If `limit.goto` is **not** specified, execution continues with the next step after the condition is met.
> If `limit.goto` is **specified**, execution jumps to `target_id` when the condition is met.

---

#### 6.4.2 `cumulative_charge` — Charge sum across explicit steps

Stops the step when the **sum of charged capacity across a specified list of steps** reaches
the threshold. Earlier steps in the list must already be completed; their charge totals are
read from the measurement log. The **current step must be included** in the list.

Typical use case: a two-phase charge sequence (main charge + constant-current finish charge)
where the charge ratio CR ≥ 1.08 must be satisfied over both steps combined.

```yaml
global_values:
  Q_CR_min:
    value: 32.4
    unit: "Ah"
    description: "Minimum recharged capacity for CR ≥ 1.08 (= 1.08 × Cn / 2)"

steps:
  - id: 3
    type: charge          # main charge – per-step limit
    ...
    limit:
      charge:
        operator: greater_than_or_equal
        value: "Q_CR_min"
        unit: "Ah"

  - id: 4
    type: charge          # finish charge – cumulative limit across steps 3+4
    ...
    limit:
      cumulative_charge:
        steps: [3, 4]     # explicit list; current step (4) must be included
        operator: greater_than_or_equal
        value: "Q_CR_min"
        unit: "Ah"
```

| Field | Type | Required | Allowed values / Notes |
|---|---|---|---|
| `cumulative_charge.steps` | `integer[]` | ✅ | List of step IDs (≥ 2 entries, unique, must include current step ID) |
| `cumulative_charge.operator` | `string` | ✅ | see [Operators](#allowed-operators) |
| `cumulative_charge.value` | `number` or `string` | ✅ | Number or `global_values` key expression |
| `cumulative_charge.unit` | `string` | ✅ | `"Ah"`, `"mAh"` |

> **Usage constraints:**
> - The `steps` list must contain the ID of the **current step** (otherwise the step would not contribute to its own limit).
> - All step IDs in `steps` must **exist** in the sequence.
> - Non-current steps must appear **earlier** in the sequence (future steps cannot be referenced).
>
> **Runtime behaviour:** if the cumulative threshold is already satisfied by earlier steps alone
> (e.g. step 3 alone charged ≥ Q_CR_min), the condition in step 4 is met immediately and
> step 4 ends after minimal charge.

---

#### Allowed operators

| Operator | Meaning |
|---|---|
| `less_than` | `<` |
| `greater_than` | `>` |
| `equal_to` | `=` |
| `less_than_or_equal` | `≤` |
| `greater_than_or_equal` | `≥` |

---

### 6.5 `result`

Optional list of measurement results to be recorded at this step.

```yaml
result:
  - name: "U_2s_1"
    description: "2s voltage measurement 1"
  - name: "U11"
    description: "SSOF2 step1 voltage at 30s"
    at:                   # optional: point in time of measurement
      value: 30
      unit: "s"
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | ✅ | Name of the result variable, e.g. `"U_0"`, `"U_2s_1"` |
| `description` | `string` | ⬜ | Description of the measured value |
| `at.value` | `number` | ✅ (if `at`) | Point in time of measurement |
| `at.unit` | `string` | ✅ (if `at`) | `"s"`, `"seconds"`, `"min"`, `"minutes"`, `"h"`, `"hours"` |

---

## 7. Expressions and Global Value References (since 0.2.0)

String values in selected fields can contain arithmetic expressions that refer to names defined in `global_values`.

### 7.1 Where expressions are used

| Field | Accepted value |
|---|---|---|
| `global_values.<name>.value` | Number or expression string |
| `duration.value` | Number or expression string |
| `current.value` / `voltage.value` | Number or expression string |
| `limit.charge.value` / `limit.cumulative_charge.value` | Number or expression string |

Other numeric fields, including `temperature.value` and the simple voltage, current, and time limit values, use numbers.

### 7.2 Supported expression syntax

| Element | Example | Description |
|---|---|---|
| Numeric literal | `42`, `3.14` | Integer or float |
| Variable reference | `Cn`, `tDCH` | Must match a `global_values` key exactly (case-sensitive) |
| Arithmetic operators | `+`, `-`, `*`, `/` | Standard precedence; `/` is float division |
| Parentheses | `(0.02 * Cn - 0.083)` | Override precedence |

> **Not supported:** Functions (e.g. `sqrt()`), exponentiation (`**`), comparison operators, or conditional logic. Keep expressions simple — complex calculations should be documented in `description` and pre-computed where possible.

The schema allows these values to be numbers or strings; interpreting an expression and resolving its references is the responsibility of the NEXFTS-consuming application.

### 7.3 Example

```yaml
global_values:
  Cn:
    value: 60
    unit: "Ah"
    description: "Nominal capacity C20"
  In:
    value: 3.0
    unit: "A"
    description: "Nominal current In = Cn / 20 h"
  tDCH:
    value: "(0.02 * Cn - 0.083) / 48 * 3600"
    unit: "s"
    description: "Low rate discharge duration per micro-cycle"

steps:
  - id: 1
    type: charge
    temperature:
      value: 25
      unit: "degC"
    duration:
      value: "1 + tDCH"    # → 1 + 83.775 = 84.775 s
      unit: "s"
    voltage:
      value: 14.0
      unit: "V"
    current:
      value: 100
      unit: "A"
```

---

## Example files

The following complete NEXFTS test definitions are available under [`test_definitions`](../test_definitions/).

### Cold-crank / EN standard

| File | Description |
|---|---|
| [`coldcrank_EN.yaml`](../test_definitions/coldcrank_EN.yaml) | Cold-crank performance test according to EN 50342-1 |

### EN 50342-6

| File | Description |
|---|---|
| [`DCA_PreCycling_Table8.yaml`](../test_definitions/EN50342-6/DCA_PreCycling_Table8.yaml) | DCA pre-cycling procedure |
| [`DCA_qDCA_Table9.yaml`](../test_definitions/EN50342-6/DCA_qDCA_Table9.yaml) | DCA charge-acceptance procedure |
| [`DCApp_Table10.yaml`](../test_definitions/EN50342-6/DCApp_Table10.yaml) | DCApp procedure |
| [`MHT_V2_Table7.yaml`](../test_definitions/EN50342-6/MHT_V2_Table7.yaml) | Micro-hybrid test procedure |

### IEC 60095-8 – Power Performance Characteristics (PPC)

Sub-procedures (called via `sub` step from the sequences below):

| File | Norm | Description |
|---|---|---|
| [`PP1_procedure_IEC60095-8_Table9.yaml`](../test_definitions/IEC60095-8/PP1_procedure_IEC60095-8_Table9.yaml) | Table 9 | Pulse Profile PP1 – currents relative to Icc (0.2·Icc, 0.4·Icc) |
| [`PP2_procedure_IEC60095-8_Table10.yaml`](../test_definitions/IEC60095-8/PP2_procedure_IEC60095-8_Table10.yaml) | Table 10 | Pulse Profile PP2 – agreed fixed amplitudes I1, I2 |

PPC test sequences at **Beginning of Life (BOL)**:

| File | Norm | Categories | Discharge |
|---|---|---|---|
| [`PPC_BOL_cat1_IEC60095-8_Table4.yaml`](../test_definitions/IEC60095-8/PPC_BOL_cat1_IEC60095-8_Table4.yaml) | Table 4 | Cat. 1 | 10 h / 50 % Cn |
| [`PPC_BOL_cat23_IEC60095-8_Table5.yaml`](../test_definitions/IEC60095-8/PPC_BOL_cat23_IEC60095-8_Table5.yaml) | Table 5 | Cat. 2+3 | 4 h / 20 % Cn |
| [`PPC_BOL_cat123_IEC60095-8_Table3.yaml`](../test_definitions/IEC60095-8/PPC_BOL_cat123_IEC60095-8_Table3.yaml) | Table 3 | Cat. 1+2+3 | 4 h + 6 h / 50 % Cn, 2 × PP1 |

PPC test sequences at **End of Life (EOL)**:

| File | Norm | Categories | Discharge |
|---|---|---|---|
| [`PPC_EOL_cat1_IEC60095-8_Table7.yaml`](../test_definitions/IEC60095-8/PPC_EOL_cat1_IEC60095-8_Table7.yaml) | Table 7 | Cat. 1 | 5 h / 25 % Cn |
| [`PPC_EOL_cat23_IEC60095-8_Table8.yaml`](../test_definitions/IEC60095-8/PPC_EOL_cat23_IEC60095-8_Table8.yaml) | Table 8 | Cat. 2+3 | 2 h / 10 % Cn |
| [`PPC_EOL_cat123_IEC60095-8_Table6.yaml`](../test_definitions/IEC60095-8/PPC_EOL_cat123_IEC60095-8_Table6.yaml) | Table 6 | Cat. 1+2+3 | 2 h + 3 h / 25 % Cn, 2 × PP1 |

### IEC 60095-8 – Charge Recovery (CR) Test

| File | Norm | Description |
|---|---|---|
| [`CR_continuous_IEC60095-8_Table11.yaml`](../test_definitions/IEC60095-8/CR_continuous_IEC60095-8_Table11.yaml) | Table 11 | CR Test – continuous 72 h quiescent drain (0.033·In) |
| [`CR_pulsed_IEC60095-8_Table12.yaml`](../test_definitions/IEC60095-8/CR_pulsed_IEC60095-8_Table12.yaml) | Table 12 | CR Test – pulsed 72 h discharge (4320 × 6 s / 54 s) |

### IEC 60095-8 – Endurance Tests

| File | Norm | Description |
|---|---|---|
| [`Endurance_50DoD_IEC60095-8_Table13.yaml`](../test_definitions/IEC60095-8/Endurance_50DoD_IEC60095-8_Table13.yaml) | Table 13 | Endurance 50 % DoD cycling – 360 cycles at 40 °C; uses `limit.charge` and `limit.cumulative_charge` for CR ≥ 1.08 |
| [`Endurance_opt_cycle_IEC60095-8_Table14.yaml`](../test_definitions/IEC60095-8/Endurance_opt_cycle_IEC60095-8_Table14.yaml) | Table 14 | Optional endurance cycle test – 100 h inner loop + `goto` for indefinite repetition |
| [`Endurance_17p5DoD_IEC60095-8_Table15.yaml`](../test_definitions/IEC60095-8/Endurance_17p5DoD_IEC60095-8_Table15.yaml) | Table 15 | Endurance 17.5 % DoD – pre-discharge + 85-cycle loop + Ce capacity measurement |
