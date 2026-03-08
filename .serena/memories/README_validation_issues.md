# README.md Validation Report

## Summary
Found inconsistencies between README documentation and actual codebase in Infrastructure, Reporting, and Configuration directories.

### Issues Found

#### 1. **src/Infrastructure/README.md**
**Critical**: Missing entire Parallel/ directory and several utility classes

- **Missing directory**: `Parallel/` (10 PHP files total)
  - FileProcessingTask.php
  - WorkerBootstrap.php
  - Strategy/ (4 files: AmphpParallelStrategy, SequentialStrategy, StrategySelector, WorkerCountDetector)
  - Serializer/ (3 files: IgbinarySerializer, PhpSerializer, SerializerInterface, SerializerSelector)

- **Missing files**:
  - `Console/OutputHelper.php` - Helper for writing large output to console
  - `Cache/CacheWriteException.php` - Exception for cache write failures
  - `DependencyInjection/CompilerPass/ParallelCollectorClassesCompilerPass.php`

#### 2. **src/Infrastructure/Console/README.md**
**Major**: Missing Command/ subdirectory and Progress/ implementations

- **Missing Command/ subdirectory structure**:
  - CheckCommand.php (main command, "thin orchestrator")
  - BaselineCleanupCommand.php
  - GraphExportCommand.php
  - HookInstallCommand.php
  - HookStatusCommand.php
  - HookUninstallCommand.php

- **Missing Progress/ implementations**:
  - ConsoleProgressBar.php
  - DelegatingProgressReporter.php

Note: These files ARE listed in parent Infrastructure/README.md structure, but not in Console/README.md

- **Missing utility class**:
  - OutputHelper.php

#### 3. **src/Infrastructure/Profiler/README.md**
**Minor**: Incorrect class locations

- ProfilerHolder.php - documented as "Infrastructure/Profiler/" but actually in **Core/Profiler/**
- NullProfiler.php - documented as "Infrastructure/Profiler/" but actually in **Core/Profiler/**
- Span.php - documented as "Infrastructure/Profiler/" but actually in **Core/Profiler/**
- ProfilerInterface.php - documented as "Infrastructure/Profiler/" but actually in **Core/Profiler/**

All these should reference "Core/Profiler/" not "Infrastructure/Profiler/"

#### 4. **src/Configuration/README.md**
**Minor**: Missing 2 implementation files

- `ConfigurationProviderInterface.php` - Runtime configuration provider interface
- `RuleOptionsParserFactory.php` - Factory for creating RuleOptionsParser with CLI aliases

#### 5. **src/Reporting/README.md**
**Info**: False positives (examples, not actual files)

- "Formatter.php" - mentioned in text examples but not a real file
- "UserService.php" - mentioned in code examples but not a real file
(These are intentional examples in the documentation, not errors)

### Files on Disk vs Documentation

#### Infrastructure/Console/
- **In Infrastructure/README.md structure**: Mentioned correctly
- **In Console/README.md**: NOT listed under Command/ subdirectory section
- Commands exist on disk but not documented in Console/README.md

#### Infrastructure/Parallel/
- **On disk**: 10 PHP files across multiple subdirectories
- **In Infrastructure/README.md**: Completely absent from structure diagram
- **Nature**: Parallel execution strategies and serialization for amphp/parallel

#### Configuration/
- **On disk**: 20 PHP files
- **In README.md**: 18 mentioned
- **Missing**: ConfigurationProviderInterface.php, RuleOptionsParserFactory.php
