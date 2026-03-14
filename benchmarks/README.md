# Benchmark Projects

PHP projects used for metric calibration and validation.

## Purpose

These projects serve as a reference corpus for:
- **Calibrating** health score formulas (target percentile distribution)
- **Validating** metric correctness across diverse codebases
- **Regression testing** formula changes against real-world data

## Project List

### Open-source (from `benchmarks/composer.json`)

| ID                      | Package                      | Description                      |
| ----------------------- | ---------------------------- | -------------------------------- |
| symfony-console         | symfony/console              | Symfony Console component        |
| symfony-di              | symfony/dependency-injection | Symfony DI component             |
| symfony-http-foundation | symfony/http-foundation      | Symfony HttpFoundation component |
| symfony-http-kernel     | symfony/http-kernel          | Symfony HttpKernel component     |
| symfony-routing         | symfony/routing              | Symfony Routing component        |
| phpunit                 | phpunit/phpunit              | PHPUnit testing framework        |
| php-parser              | nikic/php-parser             | PHP Parser by nikic              |
| doctrine-orm            | doctrine/orm                 | Doctrine ORM                     |
| doctrine-dbal           | doctrine/dbal                | Doctrine DBAL                    |
| flysystem               | league/flysystem             | Flysystem filesystem abstraction |
| composer                | composer/composer            | Composer package manager         |
| monolog                 | monolog/monolog              | Monolog logging library          |
| guzzle                  | guzzlehttp/guzzle            | Guzzle HTTP client               |
| laravel-framework       | laravel/framework            | Laravel Framework                |

### Self-analysis

| ID   | Path   | Description             |
| ---- | ------ | ----------------------- |
| aimd | `src/` | AI Mess Detector itself |

### Proprietary (local only, not in repo)

| ID              | Size        | Description            |
| --------------- | ----------- | ---------------------- |
| newlk           | ~3100 files | Large proprietary app  |
| bank-lkz-atb    | ~2200 files | Banking app            |
| comm-platform   | ~1800 files | Communication platform |
| async-bank-bus  | ~1000 files | Async banking bus      |
| booking         | ~900 files  | Booking service        |
| event           | ~600 files  | Event service          |
| bank-lkz        | ~600 files  | Banking LKZ            |
| event-streaming | ~250 files  | Event streaming        |
| documenter      | ~120 files  | Documentation tool     |
| batch           | ~115 files  | Batch processing       |
| s3-file-proxy   | ~40 files   | S3 file proxy          |

## Usage

```bash
# Install benchmark dependencies
cd benchmarks && composer install

# Collect benchmark data
php scripts/collect-benchmark-data.php [output-file.json]
```

Output is written to `docs/internal/benchmark-data.json` by default.

## Known Issues

- DuplicationDetector causes OOM on large projects (php-parser, doctrine-dbal, newlk, bank-lkz-atb, async-bank-bus)
- Workaround: increase `memory_limit` or skip duplication analysis
