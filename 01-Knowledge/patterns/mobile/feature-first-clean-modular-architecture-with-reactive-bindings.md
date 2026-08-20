---
title: "Feature-First Clean Modular Architecture with Reactive Bindings"
type: pattern
tags: [pattern, mobile, flutter, getx, clean-architecture, modularity, reactive, state-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787110568549-448b547b
sources: ["Harvest 1787110568549 448b547b.json"]
---

# Feature-First Clean Modular Architecture with Reactive Bindings

Feature-First Clean Modular Architecture with Reactive State Binding using GetX and Drift in Flutter.

## 1. Overview & Architecture

Pola modularitas codebase Flutter berbasis fitur (Feature-First) yang memisahkan tanggung jawab ke dalam layer Data (Datasource & Repository), Presentation (Controller & UI), dan Dependency Injection (Bindings).

```text
[Page / Widget View] ◄───(GetX Obx / Watch Stream)──── [Feature Controller]
        │                                                      │
        ▼ (UI Event)                                           ▼
[Controller Method] ─────────(Calls)───────────► [Feature Repository]
                                                               │
                                       ┌───────────────────────┴───────────────────────┐
                                       ▼                                               ▼
                         [Local Datasource (Drift)]                       [Remote Datasource (Dio)]
```

## 2. Implementation & Code Structure

lib/app/app_binding.dart
lib/app/routes/app_pages.dart
lib/app/routes/app_routes.dart
lib/features/<feature>/bindings/<feature>_binding.dart
lib/features/<feature>/data/datasource/<feature>_local_datasource.dart
lib/features/<feature>/data/datasource/<feature>_api_datasource.dart
lib/features/<feature>/data/repository/<feature>_repository.dart
lib/features/<feature>/presentation/controller/<feature>_controller.dart
lib/features/<feature>/presentation/pages/<feature>_page.dart
lib/features/<feature>/presentation/widgets/<feature>_widget.dart

## 3. Key Implementation Points

- Feature-first folder structure isolating data, presentation, and bindings per domain feature
- GetX Bindings for lazy dependency injection on route navigation
- Repository pattern acting as single source of truth mediating between local and remote datasources
- Reactive streaming using Drift watch streams piped into GetX observables

## 4. Code Examples

### Repository layer providing clean domain interface combining local drift stream and remote operations

```dart
class AccountRepository {
  final AccountLocalDatasource _local;

  AccountRepository(this._local);

  Future<List<Account>> getAccounts() => _local.getAccounts();
  Stream<List<Account>> watchAccounts() => _local.watchAccounts();
  Future<Account?> getAccount(String serverId) => _local.getAccount(serverId);
  Future<void> updateBalance(String serverId, double newBalance) =>
      _local.updateBalance(serverId, newBalance);
}
```

### Feature binding using GetX lazyPut to instantiate dependencies on demand

```dart
class AuthBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut<AuthLocalDatasource>(() => AuthLocalDatasource(Get.find()));
    Get.lazyPut<AuthController>(() => AuthController());
  }
}
```

## 5. Considerations & Best Practices

- Keep presentation layer decoupled from direct database or HTTP queries by using Repository as gateway
- Use lazy bindings to avoid unnecessary memory overhead on app boot
- Maintain unidirectional data flow from repository stream to controller to UI widget

## 6. Related Knowledge

- Clean Architecture
- Getx State Management

## 7. Source

- Harvest 1787110568549 448b547b.json
