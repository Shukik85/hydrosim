# 01 FOUNDATIONAL ARCHITECTURE

## ISO/ГОСТ Стандарты

### ISO 13849-1: Safety of Machinery — Control Systems
- **SIL 2** требуется для гидравлических систем
- Надёжность: ≥ 99.9% (среднее время отказа ≥ 1000 часов)
- Наше решение: логирование + health checks

### ISO 6162: Hydraulic Fluid Power Systems
- Давление: рабочее 350 bar (CAT 320D)
- Коэффициент безопасности: 4:1 (предельное 1400 bar)
- Симуляция: ±5% от рабочих параметров

### ГОСТ 52138-2003: PLC и системы реального времени
- Время отклика: <100ms p90
- Retention: 5 лет данных
- Мониторинг: 24/7

---

## Excavator Kinematics (Forward & Inverse)

### Звенья экскаватора (4 основных):
1. **Platform** (неподвижная база)
2. **Boom** (стрела, угол α)
3. **Arm** (рукоять, угол β)
4. **Bucket** (ковш, угол γ)

### Forward Kinematics
```
Boom: p_b = [L_b·cos(α), L_b·sin(α)]
Arm:  p_a = p_b + [L_a·cos(α+β), L_a·sin(α+β)]
Bucket: p_bkt = p_a + [L_bkt·cos(α+β+γ), L_bkt·sin(α+β+γ)]
```

### Inverse Kinematics
Решаем систему:
```
x_target = x_bkt
y_target = y_bkt
→ найти α, β, γ (используя численный метод)
```

---

## Hydraulic Load Model

### Статика:
```
Torque = Weight · g · CoM_distance · cos(angle)
Pressure = Torque / (cylinder_area × rod_length)
```

### Динамика:
```
m·a = F_cylinder - F_gravity - F_friction - F_soil_resistance
```

### Параметры CAT 320D:
- Boom weight: 1800 kg
- Arm weight: 950 kg
- Bucket weight: 450 kg
- Max pressure: 350 bar
- Cylinder stroke: 1.5-2.5 м (в зависимости от узла)

---

## ML Diagnostic Model

### Data Pipeline:
```
Sensor Data (100 Hz)
    ↓
Feature Extraction (time window = 1 min)
    ↓
Ensemble Model (LSTM + Isolation Forest)
    ↓
Anomaly Score (0-1)
    ↓
Alert (if score > threshold)
```

### Обнаруживаемые аномалии:
1. **Leakage** (утечка масла) — давление падает
2. **Cavitation** (кавитация) — скачки давления
3. **Mechanical wear** (износ) — увеличение шума
4. **Thermal runaway** (тепловой разгон) — температура растёт

---

## Security & Safety

### Безопасность кода:
- ✅ Параметризованные SQL (Django ORM)
- ✅ httpOnly + secure cookies
- ✅ JWT refresh rotation (24h)
- ✅ CSRF token на все POST запросы
- ✅ Rate limiting 100 req/min

### Контроль доступа:
- Operator (read-only dashboard)
- Technician (export data)
- Admin (settings + alerts)

### Audit Trail:
- Все действия логируются
- Логи храняться 5 лет в TimescaleDB

**Last updated:** 2026-01-30
