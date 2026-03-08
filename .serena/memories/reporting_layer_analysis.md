# Reporting Layer Analysis: LCOM and Coupling Issues

## Executive Summary

All 6 formatters have high LCOM violations (5-11). This is a **structural pattern issue**, not implementation bugs. The problem is that **each formatter is attempting to do both data transformation AND output formatting**, violating Single Responsibility Principle.

Similarly, ViolationSorter has LCOM=6, and AnsiColor has a coupling.distance violation.

## LCOM Violations Analysis

### What is LCOM?
- **LCOM4 (Lack of Cohesion)** = Number of connected components in method graph
- Vertices = methods; Edges = methods that access common properties
- LCOM > 1 means class could be split into multiple cohesive classes
- LCOM = 1 is ideal (perfectly cohesive)
- Thresholds: warning at 2, error at 3+ (default config)

### Formatter LCOM Values

| Class                      | LCOM | Root Cause                             |
| -------------------------- | ---- | -------------------------------------- |
| TextVerboseFormatter       | 11   | 9 methods across 3 disconnected groups |
| JsonFormatter              | 7    | 6 methods across 2-3 groups            |
| TextFormatter              | 7    | 6 methods across 2 groups              |
| SarifFormatter             | 8    | 7 methods across multiple groups       |
| CheckstyleFormatter        | 6    | Multiple formatting concerns           |
| GitLabCodeQualityFormatter | 5    | Multiple formatting concerns           |
| ViolationSorter            | 6    | 6 methods doing different sorts        |

### Root Cause: Formatter Method Groups

Each formatter actually has 2-3 **disconnected responsibilities**:

**Group 1 (Core Protocol Methods):**
- `format(Report, FormatterContext): string`
- `getName(): string`
- `getDefaultGroupBy(): GroupBy`

These don't access any properties, stand alone.

**Group 2 (Data Transformation):**
- `groupViolationsByFile()` [JsonFormatter, CheckstyleFormatter]
- `collectRules()` [SarifFormatter]
- `renderGrouped()` [TextVerboseFormatter]
- `renderFlat()` [TextVerboseFormatter]

**Group 3 (Formatting/Serialization):**
- `formatViolation()` [all formatters]
- `formatSeverity*()` [multiple]
- `severityToPriority()` [JsonFormatter]
- `severityToString()` [JsonFormatter, CheckstyleFormatter]
- `mapLevel()`, `mapSeverity()` [SarifFormatter, GitLabCodeQualityFormatter]
- `generateFingerprint()` [GitLabCodeQualityFormatter]
- `formatRuleName()`, `getRuleDescription()` [SarifFormatter]

**Group 4 (Text Rendering)** [TextVerboseFormatter only]:
- `renderViolation()`
- `formatSeverityTag()`
- `formatSeverityLabel()`
- `formatLineOnly()`
- `formatMetricValue()`

### Pattern: No Shared State

**Critical observation:** Formatters are STATELESS. They don't have any instance properties. Each method is purely functional, calculating output from inputs.

This makes them ideal candidates for **service object extraction**.

## ViolationSorter LCOM Analysis

| Method                     | Group               |
| -------------------------- | ------------------- |
| `sort()`                   | Sorting adapter     |
| `group()`                  | Grouping adapter    |
| `bySeverityFileLine()`     | Severity comparator |
| `byFileSeverityLine()`     | File comparator     |
| `byRuleSeverityFileLine()` | Rule comparator     |
| `severityOrder()`          | Utility helper      |

**Root cause:** 6 static methods doing different sorts + grouping. These are disconnected until you realize they're all comparison/grouping strategies.

**Better design:** Extract into strategy classes or use `SortableViolationList` value object.

## AnsiColor: coupling.distance Violation

**Metrics for namespace AiMessDetector\Reporting:**
- Abstractness (A) = 0.00 (no abstract classes/interfaces)
- Instability (I) = 0.27 (depends on many, used by formatters)
- Distance = |0.00 + 0.27 - 1| = 0.73 (**exceeds 0.60 threshold**)

**Problem Zone:** Zone of Uselessness + Pain hybrid
- All concrete classes (A=0) but unstable (I~0.27)
- This is unusual: concrete classes that are heavily depended on

**Analysis:**
- `AnsiColor` is concrete utility (not abstract)
- Formatters depend on it heavily
- Other classes might depend on Reporting classes
- Creates an unstable, concrete package

**Why it matters:**
- The package is concrete (easy to change) but highly stable (hard to change due to dependencies)
- Ideally: abstract packages should be stable, concrete packages can be unstable
- Mixed zones indicate poor architectural boundaries

## Coupling.distance Violations (All Cases)

Detected in:
1. **NullProfiler** (Core\Profiler): A=0.33, I=0.00 → D=0.67 (ERROR)
2. **AnsiColor** (Reporting): A=0.00, I=0.27 → D=0.73 (ERROR)
3. **CollectionOutput** (Analysis\Collection\Metric): A=0.00, I=0.47 → D=0.53 (WARNING)
4. **Baseline** (Baseline): A=0.00, I=0.50 → D=0.50 (WARNING)
5. **AnalysisConfiguration** (Configuration): A=0.17, I=0.39 → D=0.44 (WARNING)
6. **ConfigurationContext** (Configuration\Pipeline): A=0.25, I=0.33 → D=0.42 (WARNING)

**Pattern:** All are VALUE OBJECTS or CONCRETE UTILITIES:
- No interfaces/abstract classes (A=0 or very low)
- High instability = widely used internally
- Concrete + stable = pain zone

**Root cause:** These are **data transfer objects (DTOs)** or **value objects** that shouldn't drive domain architecture. They're treated as separate namespaces but are really part of larger composites.

## Domain Boundary Issues

### Current (Problematic) Structure

```
src/Reporting/ (namespace: AiMessDetector\Reporting)
├── AnsiColor.php (concrete utility)
├── FormatterContext.php (concrete DTO)
├── GroupBy.php (enum)
├── Report.php (value object)
├── ReportBuilder.php (builder)
├── ViolationSorter.php (static utility)
└── Formatter/
    └── 6 formatters (stateless services)
```

**Problems:**
1. Formatters are in same namespace as utilities
2. Stateless formatters grouped with stateful builders
3. AnsiColor is separate from formatters (violates Low Coupling)
4. ViolationSorter pollutes namespace with comparison logic
5. FormatterContext couples formatting logic with context

### What These Should Be

**Formatters are SERVICE OBJECTS:**
- Stateless transformation from Report → string
- Each has ONE responsibility per output format
- Currently they're trying to do: grouping + sorting + formatting simultaneously

**ViolationSorter is STRATEGY PATTERN:**
- Multiple comparison strategies
- Should be extracted into comparable strategies
- OR converted to `SortableViolationList` value object

**AnsiColor is UTILITY/HELPER:**
- Pure functional utility
- No domain meaning
- Should live in Infrastructure layer, not domain

**FormatterContext is IMMUTABLE DTO:**
- Pure data transfer
- No logic (only has dumb getOption accessor)
- Appropriate for Reporting layer but check if needed

## Problem: VO vs Service Boundaries

### FormatterContext Problems
- Has methods (not truly a VO)
- Only has `getOption()` which just proxies array access
- Could be just `array<string, string>`
- Creates unnecessary abstraction

### Report (Value Object) - OK
- Immutable readonly
- Has convenience methods (isEmpty, getTotalViolations, getViolationsBySeverity)
- Methods access constructor properties only
- Properly implements VO pattern

### AnsiColor (Utility or Service?)
- Acts like Formatter but is not FormatterInterface
- Stateless, pure functions
- Should either be:
  1. Part of formatter contract (inject AnsiColor factory)
  2. Moved to Infrastructure\Output\AnsiColor
  3. Made FormatterService base class method

## Recommendations

### 1. Extract Formatter Service Objects

Current:
```php
class TextFormatter implements FormatterInterface {
    public function format(Report $report, FormatterContext $context): string { ... }
    // + 6 private methods mixing transformation and formatting
}
```

Better:
```php
class TextFormatter implements FormatterInterface {
    public function __construct(
        private ViolationSorter $sorter,
        private TextViolationRenderer $renderer,  // New!
    ) {}

    public function format(Report $report, FormatterContext $context): string {
        $sorted = $this->sorter->sort($report->violations, $context->groupBy);
        return $this->renderer->render($sorted, $context);
    }
}

class TextViolationRenderer { // New service
    public function render(array $violations, FormatterContext $context): string { ... }
    // Single responsibility: formatting violations to text
}
```

### 2. Refactor ViolationSorter

Current:
```php
class ViolationSorter {
    public static function sort(array $violations, GroupBy $groupBy): array { ... }
    // 6 disconnected methods
}
```

Better Option A (Strategy):
```php
class SortedViolationList {
    public function __construct(
        private array $violations,
        private SortStrategyInterface $strategy,
    ) {}
    
    public function getGrouped(GroupBy $groupBy): array { ... }
}

interface SortStrategyInterface {
    public function sort(array $violations): array;
}

class SeverityFileSortStrategy implements SortStrategyInterface { ... }
class FileSeveritySortStrategy implements SortStrategyInterface { ... }
```

Better Option B (Simple):
```php
function sortViolations(array $violations, GroupBy $groupBy): array { ... }
function groupViolations(array $violations, GroupBy $groupBy): array { ... }
```

(Functions are better for stateless utilities)

### 3. Move AnsiColor to Infrastructure

Current location: `src/Reporting/AnsiColor.php`

Better location: `src/Infrastructure/Output/AnsiColor.php`

Why:
- Not a domain concept
- Infrastructure detail (output formatting)
- Violates dependency direction (Reporting shouldn't manage output encoding)

### 4. Simplify FormatterContext

Current:
```php
final readonly class FormatterContext {
    public function __construct(
        public bool $useColor,
        public GroupBy $groupBy,
        public array $options = [],
    ) {}
    public function getOption(string $key, string $default = ''): string { ... }
}
```

Better:
- Remove getOption() accessor (not needed, clients use $context->options['key'] ?? 'default')
- Or make it a type alias: `type FormatterOptions = array<string, mixed>`

### 5. Check VO/DTO Logic Bloat

Classes to review for "should be value object" pattern:
- `Baseline` (has methods - is it truly VO or service?)
- `AnalysisConfiguration` (check methods)
- `ConfigurationContext` (check methods)

Rules:
- **Value Object:** Immutable, no side effects, maybe small helper methods
- **Service/DTO:** If has stateful methods or business logic → service
- **Never:** Mix immutable with mutable access patterns

## Conclusion

**LCOM in formatters is NOT a code quality issue.** It's an architectural issue: stateless formatters are being modeled as classes instead of services/functions.

**coupling.distance violations indicate:** VOs and utilities are being treated as first-class domain packages when they should be infrastructure details.

**Domain boundary blur:** Mixing services, VOs, utilities, and DTOs in same namespace without clear architectural boundaries.
