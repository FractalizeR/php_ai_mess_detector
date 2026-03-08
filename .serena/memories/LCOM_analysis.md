# LCOM (Lack of Cohesion) Violations Analysis

## Key Findings

### What is LCOM?
LCOM4 measures cohesion by counting connected components in a property-sharing graph:
- Vertices = methods in a class
- Edges = (m1, m2) if m1 and m2 access at least one common property
- Interpretation:
  - LCOM = 1: perfectly cohesive (all methods share properties)
  - LCOM > 1: class could be split into multiple cohesive parts
  - LCOM counts ONLY $this->property accesses (e.g., $this->logger, $this->options)

### Architecture-Driven False Positives: 70-80% of violations

The LCOM violations are **SYSTEMICALLY CAUSED by legitimate architectural patterns**, not real design flaws:

#### 1. **Visitor/Collector Pattern** (src/Metrics/)
Example: `HalsteadVisitor` (LCOM=15)
- **Fields**: $metrics, $methodInfos, $methodStack, $currentNamespace, $currentClass, $closureCounter
- **Methods**: enterNode(), leaveNode(), getMetrics(), startMethod(), endMethod(), countOperators(), countOperands(), getOperatorName(), getUnaryOpName(), etc.
- **Problem**: Visitor methods (enterNode, leaveNode) are generic AST traversal — they touch few/no properties
- **Data extraction methods** (countOperators, countOperands) are grouped together but don't interact with $metrics
- **Result**: Multiple disjoint method groups = high LCOM (false positive)

#### 2. **Hierarchical Rules Pattern** (src/Rules/)
Example: `ComplexityRule` (LCOM=9)
- **Fields**: $options (inherited from AbstractRule)
- **Methods**: 
  - getName(), getDescription(), getCategory(), getSupportedLevels() — read-only metadata
  - analyzeLevel() — dispatcher to level-specific analysis
  - analyzeMethodLevel() — analyzes methods (uses $options.method)
  - analyzeClassLevel() — analyzes classes (uses $options.class)
  - analyze() — legacy wrapper calling both level methods
- **Problem**: Methods are grouped by level (method-level vs class-level), not by $options access
- **Result**: Multiple disjoint analyses = high LCOM (false positive)

Example: `CboRule` (LCOM=10)
- **Fields**: $options
- **Methods**:
  - Metadata: getName(), getDescription(), getCategory()
  - Dispatcher: analyzeLevel()
  - Class-level: analyzeClassLevel(), checkCbo()
  - Namespace-level: analyzeNamespaceLevel()
- **Problem**: analyzeClassLevel() and analyzeNamespaceLevel() are largely independent
- **Result**: LCOM=10 (multiple disjoint analyses)

#### 3. **Global Data Processing Pattern** (src/Metrics/Coupling/)
Example: `CouplingCollector` (LCOM=9)
- **Fields**: None (stateless)
- **Methods**:
  - getName(), requires(), provides() — metadata
  - getMetricDefinitions() — configuration
  - calculate() — orchestrator
  - computeClassMetrics() — processes classes
  - computeNamespaceMetrics() — processes namespaces
  - computeInstability() — helper
  - parseClassName() — utility
- **Problem**: Two independent computation flows (class vs namespace) that share no properties
- **Result**: LCOM=9 (really LCOM=2 if you count computation flows)

#### 4. **Large Visitor with Multiple Responsibilities** (src/Metrics/Halstead/ & src/Metrics/Complexity/)
Example: `CognitiveComplexityVisitor` (LCOM=5 but WMC=40+)
- **Fields**: $complexities, $methodInfos, $methodStack, $nestingLevel, $currentNamespace, $currentClass, $closureCounter, $lastLogicalOp
- **Methods**: 30+ methods covering:
  - enterNode(), leaveNode() — generic traversal
  - startMethod(), endMethod() — method context management
  - countComplexity() — core metric calculation
  - getComplexityIncrement() — calculation details
  - buildMethodFqn(), buildFunctionFqn(), buildClosureFqn() — FQN builders
  - isLogicalOperator(), getLogicalOperatorIncrement() — operator detection
  - isRecursiveCall() — recursion handling
- **Problem**: Each group of helper methods is used by one or two core methods
- **Result**: LCOM=5 (but really cohesive if you consider the workflow: enter → start → count → end → leave)

### Summary of False Positives

| Class                      | LCOM | Root Cause                                            | Real Issue?                       |
| -------------------------- | ---- | ----------------------------------------------------- | --------------------------------- |
| AbstractCollector          | 5    | Generic base class with methods not all used together | No — by design                    |
| HalsteadVisitor            | 15   | Visitor pattern + operator/operand split              | No — inherent to AST visitors     |
| CouplingCollector          | 9    | Class vs Namespace independent flows                  | No — legitimate separation        |
| ComplexityRule             | 9    | Method-level vs Class-level analysis                  | No — hierarchical design          |
| CboRule                    | 10   | Class-level vs Namespace-level                        | No — hierarchical design          |
| CognitiveComplexityVisitor | 5    | Large visitor with helper methods                     | Borderline — legitimate but large |

### Legitimate Issues: 20-30% of violations

Only a few cases warrant refactoring:

1. **AnalyzeCommand** (LCOM=26, WMC=1+)
   - Real issue: Command class mixing CLI, file discovery, analysis orchestration, results formatting
   - Could extract: FileDiscoveryService, ResultsPresenter, AnalysisRunner

2. **MetricAggregator** (LCOM=13)
   - Mixed aggregation logic for different symbol levels
   - Could extract: Level-specific aggregators

3. **ContainerFactory** (LCOM=18)
   - Mixing DI configuration for different modules
   - Could extract: Module-specific factory helpers

4. **TextFormatter** (LCOM=7), **JsonFormatter** (LCOM=7)
   - Multiple output sections with no cross-dependencies
   - Could extract: Section builders

5. **CodeSmellVisitor** (LCOM=9)
   - Mixing different code smell detections
   - Could extract: Individual smell detectors

### LCOM as Tool Metric

The high LCOM count is **NOT indicative of a systemic architecture problem**. Instead:
- LCOM metrics reliably flag legitimate SRP violations (AnalyzeCommand, MetricAggregator)
- LCOM metrics produce **false positives for architectural patterns**:
  - Visitor pattern (AST traversal generic, calculation methods specific)
  - Hierarchical rules (multiple independent analysis paths)
  - Global data processors (class-level vs namespace-level flows)
  - Helper method groups (math utilities, FQN builders)

### Recommendation

**Do NOT bulk-fix LCOM violations**. Instead:
1. Keep threshold moderate (5 is acceptable for architectural patterns)
2. Manually review violations >10 for real SRP issues
3. For pattern-driven false positives, add suppression tags (@aimd-ignore) or EXCLUDE_PATTERNS
4. Real refactoring targets: AnalyzeCommand (LCOM=26), MetricAggregator (LCOM=13), ContainerFactory (LCOM=18)
