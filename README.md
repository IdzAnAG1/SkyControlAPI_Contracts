# SkyControl API Contracts

Централизованный репозиторий gRPC proto-контрактов для микросервисной платформы **SkyControl** — системы управления и мониторинга БПЛА (дронов).

---

## Содержание

- [О проекте](#о-проекте)
- [Структура репозитория](#структура-репозитория)
- [Сервисы и API](#сервисы-и-api)
  - [Auth — Аутентификация](#auth--аутентификация)
  - [Platform — Управление устройствами](#platform--управление-устройствами)
  - [Telemetry — Телеметрия](#telemetry--телеметрия)
  - [Analysis Service — Аналитика](#analysis-service--аналитика)
  - [Route Planner — Планировщик маршрутов](#route-planner--планировщик-маршрутов)
- [Быстрый старт](#быстрый-старт)
- [Инструменты](#инструменты)

---

## О проекте

Данный репозиторий хранит единый источник истины для всех gRPC-контрактов платформы SkyControl. Каждый микросервис подключает этот репозиторий как зависимость и генерирует код на своей стороне.

**Технологический стек:**
- [Protocol Buffers v3](https://protobuf.dev/)
- [Buf](https://buf.build/) — линтинг, управление зависимостями, кодогенерация
- [Google API Annotations](https://github.com/googleapis/googleapis) — HTTP/REST-транскодинг поверх gRPC

---

## Структура репозитория

```
proto/
├── auth/
│   └── v1/
│       └── auth.proto              # Аутентификация пользователей
├── platform/
│   └── v1/
│       └── platform.proto          # Регистрация и управление устройствами
├── telemetry/
│   └── v1/
│       └── telemetry.proto         # Телеметрия БПЛА в реальном времени
├── analysis_service/
│   └── v1/
│       └── anal_service.proto      # Аналитический сервис (в разработке)
└── route_planner/
    └── v1/
        └── route_planner.proto     # Планировщик маршрутов (в разработке)
buf.yaml                            # Конфигурация Buf
buf.lock                            # Lockfile зависимостей
Makefile                            # Команды для работы с зависимостями
```

---

## Сервисы и API

### Auth — Аутентификация

**Package:** `skycontrol.auth.v1`

Управление учётными записями пользователей.

| RPC | HTTP | Описание |
|-----|------|----------|
| `Register` | `POST /api/v1/auth/register` | Регистрация нового пользователя |
| `Login` | `POST /api/v1/auth/login` | Вход в систему |

<details>
<summary>Схемы сообщений</summary>

```protobuf
message RegisterRequest {
  string email    = 1;
  string password = 2;
  string nickname = 3;
  string username = 4;
}

message RegisterResponse {
  string user_id     = 1;
  string token       = 2;
  string err_message = 3;
}

message LoginRequest {
  string email    = 1;
  string password = 2;
}

message LoginResponse {
  string user_id     = 1;
  string token       = 2;
  string err_message = 3;
}
```
</details>

---

### Platform — Управление устройствами

**Package:** `skycontrol.platform.v1`

Регистрация и получение информации об устройствах (БПЛА) пользователя.

| RPC | HTTP | Описание |
|-----|------|----------|
| `RegisterDevice` | `POST /api/v1/platform/register` | Регистрация нового устройства |
| `GetUserDevices` | `GET /api/v1/platform/get_user_device` | Список устройств пользователя (с пагинацией) |
| `GetDevice` | `GET /api/v1/platform/get_device` | Получение информации об устройстве |

<details>
<summary>Схемы сообщений</summary>

```protobuf
message Device {
  string device_id             = 1;
  string device_name           = 2;
  string device_description    = 3;
  string device_specifications = 4;
}

message RegisterDeviceRequest {
  string user_id               = 1;
  string device_name           = 2;
  string device_description    = 3;
  string device_specifications = 4;
}
```
</details>

---

### Telemetry — Телеметрия

**Package:** `skycontrol.telemetry.v1`

Получение телеметрических данных с БПЛА в реальном времени: координаты, скорость, заряд батареи, состояние камеры.

| RPC | HTTP | Описание |
|-----|------|----------|
| `GetDevicesInfo` | `GET /api/v1/telemetry/get_device_info` | Краткая телеметрия по всем устройствам |
| `GetDeviceInfo` | `GET /api/v1/telemetry/get_device_info` | Расширенная телеметрия одного устройства |

<details>
<summary>Схемы сообщений</summary>

```protobuf
// Расширенная информация об устройстве
message ExtendedDeviceInfo {
  string    device_id       = 1;
  string    device_name     = 2;
  Timestamp device_up_time  = 3;
  string    device_status   = 4;
  DevicePower    device_power    = 5;
  DevicePosition device_position = 6;
  DeviceVelocity device_velocity = 7;
  DeviceCamera   device_camera   = 8;
}

// GPS-координаты
message DevicePosition {
  double latitude      = 1;
  double longitude     = 2;
  double altitude      = 3;
  double home_distance = 4;
}

// Вектор скорости
message DeviceVelocity {
  double ground_speed   = 1;
  double vertical_speed = 2;
  double north_speed    = 3;
  double east_speed     = 4;
  double down_speed     = 5;
}

// Питание
message DevicePower {
  double voltage = 1;
  double current = 2;
  double power   = 3;
}

// Камера
message DeviceCamera {
  bool   has_camera    = 1;
  string camera_type   = 2;
  string camera_model  = 3;
  string camera_status = 4;
}
```
</details>

---

### Analysis Service — Аналитика

**Package:** `skycontrol.analysis.service.v1`

> Сервис находится в разработке. Контракты будут добавлены в следующих версиях.

---

### Route Planner — Планировщик маршрутов

**Package:** `skycontrol.route.planner.v1`

> Сервис находится в разработке. Контракты будут добавлены в следующих версиях.

---

## Быстрый старт

### Требования

- [Buf CLI](https://buf.build/docs/installation) `>= 1.x`

### Установка / обновление зависимостей

```bash
make depup
# или напрямую:
buf dep update
```

### Линтинг

```bash
buf lint
```

### Кодогенерация (в целевом сервисе)

Добавьте этот репозиторий как зависимость в `buf.yaml` вашего сервиса:

```yaml
deps:
  - buf.build/<org>/skycontrol-api-contracts
```

Затем запустите:

```bash
buf generate
```

---

## Инструменты

| Инструмент | Назначение |
|------------|------------|
| [Buf](https://buf.build/) | Линтинг, форматирование, управление зависимостями proto |
| [Google API Annotations](https://buf.build/googleapis/googleapis) | HTTP-транскодинг gRPC (REST gateway) |
| [grpcurl](https://github.com/fullstorydev/grpcurl) | Ручное тестирование gRPC-эндпоинтов |
