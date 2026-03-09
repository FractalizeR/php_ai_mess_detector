# Task B: Metric Name Constants — Hardcoded Strings in Rules

## Summary

Rules reference metrics via hardcoded strings in `requires()` methods and `metrics->get()` calls. These strings should be constants for safety and maintainability. Below is the complete dependency map: Collector → Metric Name → Rules that use it.

---

## Metric Name Constants Already Used in Rules

These rules already use `private const` for metric names:

### Complexity Metrics

#### CCN (Cyclomatic Complexity)
- **Collector**: `CyclomaticComplexityCollector::METRIC_CCN = 'ccn'`
- **Rules**:
  - `ComplexityRule` → `private const METRIC_CCN = 'ccn'` (line 26)
    - requires(): `[self::METRIC_CCN]` (line 48)
    - get calls: `'ccn'` (line 131), `'ccn.max'` (line 171)

#### Cognitive Complexity
- **Collector**: `CognitiveComplexityCollector::METRIC_COGNITIVE = 'cognitive'`
- **Rules**:
  - `CognitiveComplexityRule` → `private const METRIC_COGNITIVE = 'cognitive'` (line 25)
    - requires(): `[self::METRIC_COGNITIVE]` (line 48)

#### NPath Complexity
- **Collector**: `NpathComplexityCollector::METRIC_NPATH = 'npath'`
- **Rules**:
  - `NpathComplexityRule` → `private const METRIC_NPATH = 'npath'` (line 25)
    - requires(): `[self::METRIC_NPATH]` (line 50)

#### WMC (Weighted Methods per Class)
- **Collector**: `WmcCollector::METRIC_WMC = 'wmc'`
- **Rules**:
  - `WmcRule` → `private const METRIC_WMC = 'wmc'`, `METRIC_IS_DATA_CLASS = 'isDataClass'` (lines 27-28)
    - requires(): `[self::METRIC_WMC, self::METRIC_IS_DATA_CLASS]` (line 50)
    - get calls: `'isDataClass'` (line 68), `'wmc'` (line 72)

### Coupling Metrics

#### CBO (Coupling Between Objects)
- **Collector**: `CouplingCollector::METRIC_CBO = 'cbo'`, `METRIC_CA = 'ca'`, `METRIC_CE = 'ce'`
- **Rules**:
  - `CboRule` → `private const METRIC_CBO = 'cbo'`, `METRIC_CA = 'ca'`, `METRIC_CE = 'ce'` (lines 30-32)
    - requires(): `[self::METRIC_CBO, self::METRIC_CA, self::METRIC_CE]` (line 54)
    - get calls: `'cbo'` (line 140), `'ca'` (line 150), `'ce'` (line 151)

#### Instability
- **Collector**: `InstabilityCollector` (derived from CBO)
- **Rules**:
  - `InstabilityRule` → `private const METRIC_INSTABILITY = 'instability'`, `METRIC_CA = 'ca'`, `METRIC_CE = 'ce'` (lines 30-32)
    - requires(): `[self::METRIC_INSTABILITY, self::METRIC_CA, self::METRIC_CE]` (line 54)

#### Distance
- **Collector**: `DistanceCollector`
- **Rules**:
  - `DistanceRule` → `private const METRIC_DISTANCE = 'distance'`, `METRIC_ABSTRACTNESS = 'abstractness'`, `METRIC_INSTABILITY = 'instability'` (lines 39-41)
    - requires(): `[self::METRIC_DISTANCE, self::METRIC_ABSTRACTNESS, self::METRIC_INSTABILITY]` (line 70)
    - get calls: `'distance'` (line 105), `'abstractness'` (line 115), `'instability'` (line 116)

#### ClassRank
- **Collector**: `ClassRankCollector::METRIC_CLASS_RANK = 'classRank'`
- **Rules**:
  - `ClassRankRule` → `private const METRIC_CLASS_RANK = 'classRank'` (line 25)
    - requires(): `[self::METRIC_CLASS_RANK]` (line 47)
    - get calls: `'classRank'` (line 63)

### Structure Metrics

#### DIT (Depth of Inheritance Tree)
- **Collector**: `InheritanceDepthCollector::METRIC_DIT = 'dit'`
- **Rules**:
  - `InheritanceRule` → `private const METRIC_DIT = 'dit'` (line 25)
    - requires(): `[self::METRIC_DIT]` (line 47)
    - get calls: `'dit'` (line 63)

#### NOC (Number of Children)
- **Collector**: `InheritanceDepthCollector::METRIC_NOC = 'noc'`
- **Rules**:
  - `NocRule` → `private const METRIC_NOC = 'noc'` (line 25)
    - requires(): `[self::METRIC_NOC]` (line 48)
    - get calls: `'noc'` (line 60)

#### LCOM (Lack of Cohesion)
- **Collector**: `LcomCollector::METRIC_LCOM = 'lcom'`
- **Rules**:
  - `LcomRule` → `private const METRIC_LCOM = 'lcom'` (line 27)
    - requires(): `[self::METRIC_LCOM, 'methodCount', 'isReadonly']` (line 46)
    - get calls: `'isReadonly'` (line 65), `'methodCount'` (line 70), `'lcom'` (line 72)

### Maintainability Metrics

#### MI (Maintainability Index)
- **Collector**: `MaintainabilityIndexCollector::METRIC_MI = 'mi'`
- **Rules**:
  - `MaintainabilityRule` → `private const METRIC_MI = 'mi'` (line 25)
    - requires(): `[self::METRIC_MI, 'methodLoc']` (line 46)
    - get calls: `'mi'` (line 60), `'methodLoc'` (line 61)

---

## Hardcoded Strings (NOT Using Constants)

These rules use hardcoded strings in `requires()` and should be refactored:

### Size Metrics

#### MethodCount
- **Collector**: `MethodCountCollector::METRIC_METHOD_COUNT = 'methodCount'`
- **Rules**:
  - `MethodCountRule` → **HARDCODED** `'methodCount'` (line 43)
    - get calls: `'methodCount'` (line 80)
  - `LcomRule` → **HARDCODED** `'methodCount'` (line 46)
    - get calls: `'methodCount'` (line 70)

#### ClassCount
- **Collector**: `ClassCountCollector::METRIC_CLASS_COUNT = 'classCount'`
- **Rules**:
  - `ClassCountRule` → **HARDCODED** `'classCount'` (line 43)
    - get calls: `'classCount'` (line 80)
  - `ClassCountRule` → **HARDCODED** `'classCount.sum'` in aggregation (line 85)
  - `InstabilityRule` → **HARDCODED** `'classCount.sum'` (line 197)
  - `CboRule` → **HARDCODED** `'classCount.sum'` (line 177)

#### PropertyCount
- **Collector**: `PropertyCountCollector` provides `'propertyCount'`, `'isReadonly'`, `'isPromotedPropertiesOnly'`
- **Rules**:
  - `PropertyCountRule` → **HARDCODED** `['propertyCount', 'isReadonly', 'isPromotedPropertiesOnly']` (line 42)
    - get calls: `'isReadonly'` (line 59), `'propertyCount'` (line 60), `'isPromotedPropertiesOnly'` (line 61)
  - `LcomRule` → **HARDCODED** `'isReadonly'` (line 46, 65)

### Code Smell Metrics

#### ParameterCount
- **Collector**: `ParameterCountCollector::METRIC_PARAMETER_COUNT = 'parameterCount'`
- **Rules**:
  - `LongParameterListRule` → **HARDCODED** `'parameterCount'` (line 46)
    - get calls: `'parameterCount'` (line 98)

#### UnreachableCode
- **Collector**: `UnreachableCodeCollector::METRIC_UNREACHABLE_CODE = 'unreachableCode'`
- **Rules**:
  - `UnreachableCodeRule` → **HARDCODED** `'unreachableCode'` (line 43)
    - get calls: `'unreachableCode'` (line 79), `'unreachableCode.firstLine'` (line 82)

#### UnusedPrivate
- **Collector**: `UnusedPrivateCollector` provides `'unusedPrivate.total'`, dynamic `'unusedPrivate.{type}.line.{i}'`
- **Rules**:
  - `UnusedPrivateRule` → **HARDCODED** `['unusedPrivate.total', ...]` (lines 56-58)
    - get calls: `'unusedPrivate.total'` (line 73), `'unusedPrivate.{$type}.line.{$i}'` (line 85)

#### TypeCoverage
- **Collector**: `TypeCoverageCollector` provides `'typeCoverage.param'`, `'typeCoverage.return'`, `'typeCoverage.property'` (with `.total` and `.missing` variants)
- **Rules**:
  - `TypeCoverageRule` → **HARDCODED** `'typeCoverage'` (line 45)
    - get calls: `'typeCoverage.param'`, `'typeCoverage.paramTotal'`, `'typeCoverage.return'`, `'typeCoverage.returnTotal'`, `'typeCoverage.property'`, `'typeCoverage.propertyTotal'` (lines 75-92)

#### Security Metrics
- **Collectors**: `HardcodedCredentialsCollector`, `SensitiveParameterCollector`, `SecurityPatternCollector`
- **Rules**:
  - `HardcodedCredentialsRule` → **HARDCODED** `'security.hardcodedCredentials.count'` (line 42)
  - `SensitiveParameterRule` → **HARDCODED** `'security.sensitiveParameter.count'` (line 42)
  - `SqlInjectionRule`, `XssRule`, `CommandInjectionRule` inherit from `AbstractSecurityPatternRule` which uses dynamic `"security.{$type}.count"` (line 58)

#### IdenticalSubExpression
- **Collector**: `IdenticalSubExpressionCollector`
- **Rules**:
  - `IdenticalSubExpressionRule` → dynamically builds requires from FINDING_TYPES (lines 64-67):
    - `"identicalSubExpression.{$type}.count"` for each type (identical_operands, duplicate_condition, etc.)

#### Code Smell (Abstract)
- **Collector**: `CodeSmellCollector`
- **Rules**:
  - `AbstractCodeSmellRule` → dynamically builds from `getSmellType()` (lines 50-52):
    - `"codeSmell.{$type}.count"` for each smell type (goto, eval, exit, etc.)
  - Concrete rules: `GotoRule`, `EvalRule`, `ExitRule`, `EmptyCatchRule`, `DebugCodeRule`, `ErrorSuppressionRule`, `CountInLoopRule`, `SuperglobalsRule`, `BooleanArgumentRule`

---

## Aggregated Metrics (Not Direct Collector Output)

Rules sometimes access aggregated metrics that are computed during the Aggregation phase:

- `'ccn.max'` — max CCN per class (aggregated from method-level CCN)
- `'cognitive.max'` — max cognitive complexity per class (aggregated)
- `'npath.max'` — max NPATH per class (aggregated)
- `'classCount.sum'` — total class count in namespace (aggregated)
- `'methodLoc'` — method-level lines of code (from LocCollector)

These are not direct collector outputs but computed during aggregation and should be documented in the Metrics README.

---

## Refactoring Targets

### High Priority (Direct string literals in requires())

1. **Size metrics**:
   - `MethodCountRule` → add `private const METRIC_METHOD_COUNT = 'methodCount'`
   - `ClassCountRule` → add `private const METRIC_CLASS_COUNT = 'classCount'`
   - `PropertyCountRule` → add constants for all three metrics

2. **Code Smell metrics**:
   - `LongParameterListRule` → add `private const METRIC_PARAMETER_COUNT = 'parameterCount'`
   - `UnreachableCodeRule` → add `private const METRIC_UNREACHABLE_CODE = 'unreachableCode'`
   - `UnusedPrivateRule` → add `private const METRIC_UNUSED_PRIVATE_TOTAL = 'unusedPrivate.total'`

3. **Design metrics**:
   - `TypeCoverageRule` → add constants for all type coverage metrics

### Medium Priority (Hardcoded strings in get() calls)

These rules use constants for requires() but hardcode strings in get():
- Check all get() calls and use constant values instead

### Low Priority (Dynamic metric names)

These use string interpolation and are acceptable:
- `"codeSmell.{$type}.count"` in AbstractCodeSmellRule
- `"security.{$type}.count"` in AbstractSecurityPatternRule
- `"identicalSubExpression.{$type}.count"` in IdenticalSubExpressionRule

---

## Collector Metric Names Reference

| Collector                       | Metric Name(s)                                                     | Symbol Level        |
| ------------------------------- | ------------------------------------------------------------------ | ------------------- |
| CyclomaticComplexityCollector   | `ccn`                                                              | method, class       |
| CognitiveComplexityCollector    | `cognitive`                                                        | method, class       |
| NpathComplexityCollector        | `npath`                                                            | method, class       |
| WmcCollector                    | `wmc`, `isDataClass`                                               | class               |
| CouplingCollector               | `cbo`, `ca`, `ce`                                                  | class, namespace    |
| InstabilityCollector            | `instability`                                                      | class, namespace    |
| AbstractnessCollector           | `abstractness`                                                     | namespace           |
| DistanceCollector               | `distance`                                                         | namespace           |
| ClassRankCollector              | `classRank`                                                        | class               |
| InheritanceDepthCollector       | `dit`, `noc`                                                       | class               |
| LcomCollector                   | `lcom`                                                             | class               |
| MaintainabilityIndexCollector   | `mi`                                                               | class, file         |
| MethodCountCollector            | `methodCount`                                                      | class               |
| ClassCountCollector             | `classCount`                                                       | namespace           |
| PropertyCountCollector          | `propertyCount`, `isReadonly`, `isPromotedPropertiesOnly`          | class               |
| LocCollector                    | `loc`, `methodLoc`                                                 | class, method, file |
| ParameterCountCollector         | `parameterCount`                                                   | method, function    |
| UnreachableCodeCollector        | `unreachableCode`                                                  | file                |
| UnusedPrivateCollector          | `unusedPrivate.total`, `unusedPrivate.{type}.line.{i}`             | file                |
| TypeCoverageCollector           | `typeCoverage.{section}`                                           | class, method       |
| CodeSmellCollector              | `codeSmell.{type}.count`, `codeSmell.{type}.line.{i}`              | file                |
| HardcodedCredentialsCollector   | `security.hardcodedCredentials.count`, `.line.{i}`, `.pattern.{i}` | file                |
| SensitiveParameterCollector     | `security.sensitiveParameter.count`, `.line.{i}`                   | file                |
| SecurityPatternCollector        | `security.{type}.count`, `.line.{i}`, `.superglobal.{i}`           | file                |
| IdenticalSubExpressionCollector | `identicalSubExpression.{type}.count`, `.line.{i}`                 | file                |
