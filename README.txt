# HardDA Course: Оптимизация таблицы user_dm_events

**Автор:** [Alex1Di]  
**Курс:** Karpov Course - HardDA  
**Репозиторий:** [Clickhouse_optimize_user](https://github.com/Alex1Di/Clickhouse_optimize_user)

---

## 📈 Общие результаты

| Показатель | Исходная таблица | Оптимизированная таблица | Изменение |
|------------|------------------|--------------------------|-----------|
| **Размер** | 4.72 GiB | 1006.59 MiB (~0.98 GiB) | **▼ 79%** |
| **Строки** | 55,916,675 | 55,916,675 | без изменений |

> **Экономия места: почти в 5 раз!** (с 4.72 GiB до 0.98 GiB)

---

## 🔧 Выполненные оптимизации

### 1. Типы данных
- `cnt_session_initiation` → UInt8 (вместо UInt32)
- `cnt_order_via_phone` → UInt8
- Остальные `cnt_*` → UInt16
- ID → UInt256

### 2. Кодеки сжатия
- `CODEC(Delta, ZSTD)` — для монотонных данных
- `CODEC(T64, ZSTD)` — для целых чисел
- `CODEC(ZSTD)` — для строк

### 3. LowCardinality
- `platform LowCardinality(String)` — для поля с малым числом уникальных значений

### 4. ALIAS поля
- `week_start_date ALIAS toMonday(event_date)` — не хранится, вычисляется на лету

### 5. Ключ сортировки
- `ORDER BY (platform, user_pseudo_id, user_x_phone_id, event_date)`

### 6. Настройки
- `ratio_of_defaults_for_sparse_serialization = 0.89`
- `index_granularity = 8192`

---

## 📊 Сравнение размера колонок

| Колонка | Мой размер | Исходный размер | Экономия |
|---------|-----------|-----------------|----------|
| **user_x_phone_id** | 327.14 MiB | 1.82 GiB | **▼ 82%** |
| **user_pseudo_id** | 276.24 MiB | 1.70 GiB | **▼ 84%** |
| **platform** | 256.81 KiB | 51.38 MiB | **▼ 99.5%** |
| **cnt_events** | 82.53 MiB | 134.06 MiB | **▼ 38%** |
| **cnt_session_initiation** | 35.26 MiB | 92.33 MiB | **▼ 62%** |
| **cnt_view_advertisement** | 76.02 MiB | 124.09 MiB | **▼ 39%** |
| **cnt_add_to_favorites** | 23.29 MiB | 29.06 MiB | **▼ 20%** |
| **event_date** | 80.50 MiB | 437.58 KiB | ⚠️ увеличился* |

> *`event_date` увеличился, т.к. перестал быть первым в ORDER BY, но это компенсируется общей экономией

---

## 🔥 Ключевые достижения

- **platform**: сжался в ~200 раз (51 MiB → 256 KiB)
- **user_pseudo_id**: сжался в ~6 раз (1.70 GiB → 276 MiB)
- **user_x_phone_id**: сжался в ~5.7 раз (1.82 GiB → 327 MiB)
- **Общий размер таблицы уменьшился на 79%**

---

## 📋 Доля нулевых значений в метриках

| Метрика | Доля нулей |
|---------|-----------|
| cnt_order_via_phone | высокая |
| cnt_send_message | высокая |
| cnt_display_phone | высокая |
| cnt_new_advertisement_open | высокая |
| cnt_events | низкая |

