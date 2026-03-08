# Анализ форматирования вывода нарушений в AI Mess Detector

## 1. TextFormatter (Компактный формат, по умолчанию)

### Структура класса
- **Файл**: `src/Reporting/Formatter/TextFormatter.php`
- **Формат**: Однострочный, совместим с GCC/Clang и парсируется grep/awk

### Формат вывода
```
file:line: severity[rule]: message (symbol)
```

**Пример:**
```
src/Service/UserService.php:42: error[cyclomatic-complexity]: Cyclomatic complexity of 25 exceeds threshold (UserService::calculateDiscount)
src/Service/UserService.php:120: warning[cyclomatic-complexity]: Cyclomatic complexity of 12 exceeds threshold (UserService::processOrder)

1 error(s), 1 warning(s) in 1 file(s)
```

### Информация в каждом нарушении:
1. **File path** - `violation->location->file`
2. **Line number** - `violation->location->line` (опционально, может быть null)
3. **Severity** - `violation->severity` → "error" или "warning"
4. **Rule name** - `violation->ruleName` (в квадратных скобках)
5. **Message** - `violation->message` (основной текст нарушения)
6. **Symbol name** - `violation->symbolPath->getSymbolName()` (в скобках, опционально)

### Обработка разных уровней:
- **Method**: `UserService::calculateDiscount` (namespace\class::method)
- **Class**: `UserService` (только имя класса)
- **Namespace**: (нет символа, пусто)
- **File**: (нет символа, пусто)
- **Function**: `myComplexFunction` (только имя функции)

---

## 2. JsonFormatter (PHPMD-совместимый JSON)

### Структура
```json
{
  "version": "1.0.0",
  "package": "aimd",
  "timestamp": "2026-03-04T...",
  "files": [
    {
      "file": "src/Service/UserService.php",
      "violations": [
        {
          "beginLine": 42,
          "endLine": 42,
          "rule": "cyclomatic-complexity",
          "symbol": "App\\Service\\UserService::calculateDiscount",
          "priority": 1,
          "severity": "error",
          "description": "Cyclomatic complexity of 25 exceeds threshold",
          "metricValue": 25
        }
      ]
    }
  ],
  "summary": {
    "filesAnalyzed": 42,
    "filesSkipped": 1,
    "violations": 2,
    "errors": 1,
    "warnings": 1,
    "duration": 0.23
  }
}
```

### Информация в каждом нарушении:
- `beginLine`, `endLine` - номер строки
- `rule` - имя правила
- `symbol` - полный путь символа (namespace\class::member)
- `priority` - 1 для Error, 3 для Warning
- `severity` - "error" или "warning"
- **description** - это основное сообщение (`violation->message`)
- `metricValue` - числовое значение метрики (если есть)

---

## 3. TextVerboseFormatter (Многострочный человекочитаемый формат)

### Структура вывода
```
AI Mess Detector Report
==================================================

Violations:

  [ERROR] src/Service/UserService.php:42
    App\Service\UserService::calculateDiscount
    Rule: cyclomatic-complexity
    Cyclomatic complexity of 25 exceeds threshold

  [WARNING] src/Service/UserService.php:120
    App\Service\UserService::processOrder
    Rule: cyclomatic-complexity
    Cyclomatic complexity of 12 exceeds threshold

--------------------------------------------------
Files: 42 analyzed, 1 skipped | Errors: 1 | Warnings: 1 | Time: 0.23s
```

### Информация в каждом нарушении:
1. **Severity** в квадратных скобках
2. **File path** и **line number**
3. **Full symbol path** - `violation->symbolPath->toString()`
4. **Rule name** - `Rule: {rule}`
5. **Message** - основной текст

### Сортировка:
- Errors перед Warnings
- По file path
- По line number

---

## 4. CheckstyleFormatter (XML формат для CI систем)

### Структура
```xml
<?xml version="1.0" encoding="UTF-8"?>
<checkstyle version="3.0">
  <file name="src/Service/UserService.php">
    <error line="42" severity="error" 
            message="Cyclomatic complexity of 25 exceeds threshold" 
            source="aimd.cyclomatic-complexity"/>
    <error line="120" severity="warning" 
            message="Cyclomatic complexity of 12 exceeds threshold" 
            source="aimd.cyclomatic-complexity"/>
  </file>
</checkstyle>
```

### Атрибуты error элемента:
- `line` - номер строки
- `severity` - "error" или "warning"
- **message** - основное сообщение (`violation->message`)
- `source` - "aimd." + rule name

---

## 5. SarifFormatter (SARIF 2.1.0 JSON для GitHub/VS Code/Azure)

### Структура
```json
{
  "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/master/...",
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "AI Mess Detector",
          "version": "0.1.0",
          "informationUri": "https://github.com/FractalizeR/php_ai_mess_detector",
          "rules": [
            {
              "id": "cyclomatic-complexity",
              "name": "Cyclomatic Complexity",
              "shortDescription": {
                "text": "Code complexity exceeds threshold"
              },
              "defaultConfiguration": {
                "level": "warning"
              }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "cyclomatic-complexity",
          "level": "error",
          "message": {
            "text": "Cyclomatic complexity of 25 exceeds threshold"
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "src/Service/UserService.php",
                  "uriBaseId": "%SRCROOT%"
                },
                "region": {
                  "startLine": 42,
                  "startColumn": 1
                }
              }
            }
          ]
        }
      ]
    }
  ]
}
```

### Информация в каждом результате:
- `ruleId` - имя правила
- `level` - "error" или "warning"
- **message.text** - основное сообщение (`violation->message`)
- `locations[0].physicalLocation.artifactLocation.uri` - file path
- `locations[0].physicalLocation.region.startLine` - line number

---

## 6. AnalysisContext (Контекст для правил)

### Структура (`src/Core/Rule/AnalysisContext.php`)
```php
final readonly class AnalysisContext {
    public function __construct(
        public MetricRepositoryInterface $metrics,
        public array $ruleOptions = [],
        public ?DependencyGraphInterface $dependencyGraph = null,
        public array $additionalData = [],
    ) {}

    public function getOptionsForRule(string $ruleName): array {
        return $this->ruleOptions[$ruleName] ?? [];
    }

    public function getAdditionalData(string $key): mixed {
        return $this->additionalData[$key] ?? null;
    }
}
```

### Информация доступная к правилам:
1. **$metrics** - репозиторий метрик, позволяет получить значения для символов
2. **$ruleOptions** - опции конкретного правила (пороги, включено/отключено)
3. **$dependencyGraph** - граф зависимостей (для анализа циклических зависимостей)
4. **$additionalData** - доп. данные (например, найденные циклы)

---

## 7. RuleOptionsInterface (Как хранятся пороги)

### Интерфейс (`src/Core/Rule/RuleOptionsInterface.php`)
```php
interface RuleOptionsInterface {
    public static function fromArray(array $config): self;
    public function isEnabled(): bool;
    public function getSeverity(int|float $value): ?Severity;
}
```

### Пример реализации - ComplexityOptions
```php
final readonly class ComplexityOptions implements HierarchicalRuleOptionsInterface {
    public function __construct(
        public MethodComplexityOptions $method = new MethodComplexityOptions(),
        public ClassComplexityOptions $class = new ClassComplexityOptions(),
    ) {}
}
```

### Пример уровня опции - MethodComplexityOptions
```php
final readonly class MethodComplexityOptions implements LevelOptionsInterface {
    public function __construct(
        public bool $enabled = true,
        public int $warning = 10,
        public int $error = 20,
    ) {}

    public function getSeverity(int|float $value): ?Severity {
        if ($value >= $this->error) {
            return Severity::Error;
        }
        if ($value >= $this->warning) {
            return Severity::Warning;
        }
        return null;
    }
}
```

### Как правило создает нарушение:
```php
// В методе analyze() правила:
$message = sprintf('Cyclomatic complexity of %d exceeds threshold', $ccn);
$severity = $this->options->getSeverity($ccn);

return [
    new Violation(
        location: new Location($file, $line),
        symbolPath: SymbolPath::forMethod($namespace, $class, $method),
        ruleName: 'cyclomatic-complexity',
        message: $message,
        severity: $severity,  // Определено опциями
        metricValue: $ccn,
    ),
];
```

---

## 8. Violation структура (`src/Core/Violation/Violation.php`)

```php
final readonly class Violation {
    public function __construct(
        public Location $location,        // file, line
        public SymbolPath $symbolPath,    // namespace, class, method
        public string $ruleName,          // rule identifier
        public string $message,           // violation message
        public Severity $severity,        // Error or Warning
        public int|float|null $metricValue = null,
        public ?RuleLevel $level = null,
    ) {}

    public function getFingerprint(): string {
        return sprintf('%s:%s', $this->ruleName, $this->symbolPath->toCanonical());
    }
}
```

---

## Заключение

**Все форматеры ВКЛЮЧАЮТ message field:**
- **TextFormatter**: вторая часть строки после `:` (перед символом в скобках)
- **JsonFormatter**: поле `description`
- **TextVerboseFormatter**: последняя строка в блоке нарушения
- **CheckstyleFormatter**: атрибут `message`
- **SarifFormatter**: `message.text`

**Контекст вокруг message:**
- File path и line number ВСЕГДА показываются
- Symbol name (класс, метод, функция) показывается, когда применимо
- Rule name ВСЕГДА показывается (кроме TextVerbose, где тоже есть)
- Severity (Error/Warning) ВСЕГДА показывается
- Метрика (метрическое значение) опционально
