# 04 SOFTWARE ARCHITECTURE

## Project Structure

```
hydrosim/
├─ backend/
│  ├─ mechanics/
│  │  ├─ __init__.py
│  │  ├─ kinematics.py          # FK/IK: уравнения 1-3
│  │  ├─ dynamics.py            # ODE solver: уравнение 6
│  │  ├─ hydraulics.py          # Pressure/flow: уравнения 4-5, 7-8
│  │  └─ tests/
│  │     ├─ test_kinematics.py
│  │     ├─ test_dynamics.py
│  │     └─ test_hydraulics.py
│  │
│  ├─ ml/
│  │  ├─ __init__.py
│  │  ├─ features.py            # Feature extraction: уравнения 9-12
│  │  ├─ models.py              # LSTM + Isolation Forest: уравнения 13-14
│  │  ├─ ensemble.py            # Voting: уравнение 15
│  │  └─ tests/
│  │     ├─ test_features.py
│  │     ├─ test_models.py
│  │     └─ test_ensemble.py
│  │
│  ├─ api/
│  │  ├─ __init__.py
│  │  ├─ main.py                # FastAPI app
│  │  ├─ routes/
│  │  │  ├─ simulate.py         # POST /simulate
│  │  │  ├─ predict.py          # POST /predict (ML inference)
│  │  │  └─ telemetry.py        # GET /telemetry (sensor data)
│  │  └─ models.py              # Pydantic models
│  │
│  ├─ config/
│  │  ├─ __init__.py
│  │  ├─ settings.py            # Django settings
│  │  ├─ cat320d.yaml           # CAT 320D parameters
│  │  └─ model_weights/         # Pre-trained weights
│  │
│  └─ tests/
│     ├─ integration/
│     │  └─ test_full_pipeline.py
│     └─ fixtures/
│        └─ synthetic_data.py
│
├─ frontend/
│  ├─ components/
│  │  ├─ SimulationDashboard.vue
│  │  ├─ AnomalyAlert.vue
│  │  └─ TelemetryChart.vue
│  └─ pages/
│     ├─ index.vue
│     └─ diagnostics.vue
│
├─ docs/
│  └─ DESIGN_PACKAGE/
│     ├─ 00_START_HERE.md
│     ├─ 03_MATHEMATICAL_SPECIFICATION.md
│     └─ ... (15 files)
│
├─ tests/
│  └─ pytest.ini
│
├─ .github/
│  └─ workflows/
│     ├─ backend-ci.yml
│     └─ frontend-ci.yml
│
├─ docker-compose.yml
├─ pyproject.toml               # Poetry + Ruff config
└─ README.md
```

## Core Models (TypeScript + Python types)

### Python Side

```python
# backend/mechanics/kinematics.py
from dataclasses import dataclass
from typing import Tuple
import numpy as np

@dataclass
class LinkConfiguration:
    """Неменяемая конфигурация звена (boom, arm, bucket)."""
    name: str
    length: float  # meters
    mass: float  # kg
    com_distance: float  # distance from pivot to center of mass
    moment_of_inertia: float  # kg*m²

@dataclass
class CylinderData:
    """Data single hydraulic cylinder."""
    name: str
    piston_area: float  # m²
    rod_area: float  # m²
    max_stroke: float  # meters
    lever_arm: float  # meters

class ExcavatorKinematics:
    \"\"\"Full 3-link excavator kinematics.\"\"\"
    
    def __init__(self, cfg: dict):
        self.boom = LinkConfiguration(**cfg["boom"])
        self.arm = LinkConfiguration(**cfg["arm"])
        self.bucket = LinkConfiguration(**cfg["bucket"])
    
    def forward_kinematics(
        self,
        boom_angle: float,
        arm_angle: float,
        bucket_angle: float
    ) -> Tuple[np.ndarray, np.ndarray, np.ndarray]:
        \"\"\"Returns (boom_tip, arm_tip, bucket_tip) positions.\"\"\"
        # Implementation (see 03_MATHEMATICAL_SPECIFICATION.md)
        pass
    
    def inverse_kinematics(
        self,
        target_x: float,
        target_y: float
    ) -> Tuple[float, float, float]:
        \"\"\"Returns (boom_angle, arm_angle, bucket_angle).\"\"\"
        # Implementation (see eq. 3)
        pass

# backend/ml/features.py
from dataclasses import dataclass
from typing import Dict, List
import numpy as np

@dataclass
class SensorWindow:
    \"\"\"Sliding window of sensor data.\"\"\"
    timestamp: int  # unix timestamp
    pressure_boom: np.ndarray  # [n_samples]
    pressure_arm: np.ndarray
    pressure_bucket_l: np.ndarray
    pressure_bucket_r: np.ndarray
    temperature: np.ndarray
    boom_angle: np.ndarray
    arm_angle: np.ndarray
    bucket_angle: np.ndarray

class FeatureExtractor:
    \"\"\"Extract 50+ features from sensor window.\"\"\"
    
    def __init__(self, window_size: int = 1000):
        self.window_size = window_size
    
    def extract_features(self, window: SensorWindow) -> Dict[str, float]:
        \"\"\"Returns dict of feature_name: value.\"\"\"
        features = {}
        
        # Pressure statistics (eq. 9-10)
        features["pressure_boom_mean"] = np.mean(window.pressure_boom)
        features["pressure_boom_std"] = np.std(window.pressure_boom)
        features["pressure_boom_dpdt_max"] = np.max(np.abs(np.diff(window.pressure_boom)))
        
        # Temperature (eq. 11)
        features["temp_gradient"] = (window.temperature[-1] - window.temperature[0]) / len(window.temperature)
        
        # Angle velocities
        features["boom_velocity"] = np.mean(np.diff(window.boom_angle))
        
        return features
```

## API Contract (OpenAPI)

```yaml
openapi: 3.0.0
info:
  title: Hydraulic Diagnostic Platform
  version: 1.0.0

paths:
  /api/v1/simulate:
    post:
      summary: Симуляция гидравлики
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                boom_angle_deg: number
                arm_angle_deg: number
                bucket_angle_deg: number
                pump_pressure_bar: number
      responses:
        '200':
          description: Результат симуляции
          content:
            application/json:
              schema:
                type: object
                properties:
                  boom_tip: [number, number]
                  bucket_tip: [number, number]
                  pressures:
                    type: object
                    properties:
                      boom: number
                      arm: number
  
  /api/v1/predict:
    post:
      summary: ML предикция аномалии
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                sensor_window: object  # SensorWindow
      responses:
        '200':
          description: Anomaly score + alert
          content:
            application/json:
              schema:
                type: object
                properties:
                  anomaly_score: number  # 0-1
                  is_anomaly: boolean
                  confidence: number
                  alert_type: string  # leakage | cavitation | wear | thermal
```

## Security Implementation

- ✅ **SQL Injection Prevention**: Django ORM only
- ✅ **XSS Protection**: format_html() + escape()
- ✅ **CSRF**: Token on all POST
- ✅ **Auth**: JWT (access token 15min, refresh 7 days)
- ✅ **Rate Limiting**: 100 req/min per IP
- ✅ **Audit Trail**: All actions logged to TimescaleDB
- ✅ **Secret Management**: Vault integration (no hardcoded keys)

**Last updated:** 2026-01-30
