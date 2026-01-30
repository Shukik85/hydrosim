# 02 REFERENCE EXCAVATOR SPEC: CAT 320D

## Physical Specifications

| Parameter | Value | Notes |
|-----------|-------|-------|
| Operating weight | 20,000 kg | Без ковша |
| Boom length | 6.45 m | От оси вращения |
| Arm length | 4.20 m | От шарнира boom-arm |
| Bucket capacity | 0.76 m³ | Стандартный ковш |
| Depth of dig | 4.45 m | Максимальное углубление |
| Height of reach | 10.00 m | От земли до верха стрелы |

## Power System (Hydraulic)

| Component | Specification |
|-----------|--------------|
| Main pump | Displacement: 350 cc/rev |
| Max pressure | 350 bar (рабочее) |
| Safety relief | 385 bar |
| Pilot pressure | 25 bar |
| Flow rate | 180 L/min (при 1800 RPM) |

## Cylinders (Основные 4)

```
BOOM_CYL:
  Rod diameter: 90 mm
  Piston area: 254 cm²
  Stroke: 2.45 m
  Max force: ~890 kN

ARM_CYL:
  Rod diameter: 70 mm
  Piston area: 113 cm²
  Stroke: 1.80 m
  Max force: ~395 kN

BUCKET_CYL (левый):
  Rod diameter: 50 mm
  Piston area: 52 cm²
  Stroke: 1.20 m
  Max force: ~182 kN

BUCKET_CYL (правый):
  Rod diameter: 50 mm
  Piston area: 52 cm²
  Stroke: 1.20 m
  Max force: ~182 kN
```

## Sensor Data Points

CAT 320D оснащена следующими датчиками (или их эквивалентами):

```
Hydraulic System:
  - Main pump pressure (0-400 bar)
  - Boom cylinder pressure (A/B port)
  - Arm cylinder pressure (A/B port)
  - Bucket cylinders pressure (left/right, A/B port)
  - Tank return pressure
  - Fluid temperature

Machine Motion:
  - Boom angle (analog 0-10V)
  - Arm angle (analog 0-10V)
  - Bucket angle (analog 0-10V)
  - Swing angle (0-360°)

Diagnostics:
  - Engine load (0-100%)
  - Engine RPM
  - Fuel level
  - Ambient temperature
```

## Sensor Sampling Rates

```
High-frequency (1000 Hz):
  - Cylinder pressures (для обнаружения cavitation)

Standard (100 Hz):
  - Angles, RPM, temperature

Slow (1 Hz):
  - Fuel level, diagnostics
```

## Typical Duty Cycle

```
30% Digging (full pressure, max load)
40% Swing & positioning
20% Idle
10% Transport/moving
```

**Last updated:** 2026-01-30
