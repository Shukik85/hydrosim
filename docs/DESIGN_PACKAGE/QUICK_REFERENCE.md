# QUICK REFERENCE: CAT 320D + FORMULAS + PYTHON SNIPPETS

## CAT 320D Constants

```python
# Dimensions (meters)
BOOM_LENGTH = 6.45
ARM_LENGTH = 4.20
BUCKET_LENGTH = 1.20

# Masses (kg)
BOOM_MASS = 1800
ARM_MASS = 950
BUCKET_MASS = 450

# Center of mass distances (from pivot, meters)
BOOM_COM = 3.0
ARM_COM = 1.8
BUCKET_COM = 0.5

# Moments of inertia (kg*m²)
BOOM_MOI = 8000
ARM_MOI = 1200
BUCKET_MOI = 150

# Hydraulic
MAIN_PUMP_DISP = 350  # cc/rev
MAX_PRESSURE_BAR = 350
SAFETY_RELIEF_BAR = 385
PILOT_PRESSURE_BAR = 25

# Cylinders
BOOM_CYL_AREA = 254e-4  # m² (piston)
BOOM_CYL_STROKE = 2.45
ARM_CYL_AREA = 113e-4
ARM_CYL_STROKE = 1.80
BUCKET_CYL_AREA = 52e-4
BUCKET_CYL_STROKE = 1.20

# Sensor ranges
PRESSURE_MIN, PRESSURE_MAX = 0, 400  # bar
ANGLE_MIN, ANGLE_MAX = -np.pi/2, np.pi/2  # radians
TEMP_MIN, TEMP_MAX = -20, 100  # °C
RPM_MIN, RPM_MAX = 0, 2000
```

## Key Formulas (One-Liners)

```python
# Kinematics
boom_tip = boom_len * np.array([np.cos(α), np.sin(α)])

# Torque from weight
tau_gravity = mass * g * com_dist * np.cos(angle)

# Pressure from torque
pressure_bar = (torque_nm / lever_arm_m) / (piston_area_m2 * 1e5)

# Flow rate
q_l_min = (pump_cc_rev * rpm / 1000) * efficiency

# Angular acceleration (ODE)
alpha = (tau_cyl - tau_grav - tau_fric) / moment_of_inertia

# Feature: dP/dt (pressure spike)
dp_dt = np.diff(pressures) / (dt_sec * 1000)  # bar/sec

# Anomaly score (ensemble)
score = 0.4*lstm_norm + 0.4*isolation_norm + 0.2*threshold
is_anomaly = score > 0.5
```

## Import Statements (Paste at Top of Files)

```python
# Mechanics
import numpy as np
from scipy.integrate import odeint
from scipy.optimize import fsolve
from dataclasses import dataclass

# ML
import torch
import torch.nn as nn
from sklearn.ensemble import IsolationForest
from sklearn.preprocessing import StandardScaler

# API
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
import os
```

## Common Patterns

### Loading CAT 320D Config

```python
import yaml

with open("backend/config/cat320d.yaml") as f:
    CAT_CONFIG = yaml.safe_load(f)

BOOM_LEN = CAT_CONFIG["geometry"]["boom_length"]
```

### Unit Tests Template

```python
import pytest
from backend.mechanics.kinematics import forward_kinematics_boom

def test_forward_kinematics_boom_zero_angle():
    result = forward_kinematics_boom(0.0)
    assert np.allclose(result, [6.45, 0.0])

def test_forward_kinematics_boom_45_deg():
    result = forward_kinematics_boom(np.pi/4)
    expected = 6.45 * np.array([np.cos(np.pi/4), np.sin(np.pi/4)])
    assert np.allclose(result, expected, atol=0.01)
```

### API Endpoint Template

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter(prefix="/api/v1", tags=["mechanics"])

class SimulateRequest(BaseModel):
    boom_angle_deg: float = Field(..., ge=-90, le=90)
    arm_angle_deg: float = Field(..., ge=-180, le=0)
    bucket_angle_deg: float = Field(..., ge=-180, le=0)

@router.post("/simulate")
async def simulate_excavator(req: SimulateRequest):
    result = forward_kinematics_full(
        np.radians(req.boom_angle_deg),
        np.radians(req.arm_angle_deg),
        np.radians(req.bucket_angle_deg)
    )
    return {"bucket_tip": result["bucket_tip"].tolist()}
```

### Sensor Data Logging

```python
async def log_telemetry(sensor_data: dict):
    """Log sensor data to TimescaleDB."""
    query = """
    INSERT INTO telemetry_data (timestamp, pressure_boom, pressure_arm, angle_boom, temperature)
    VALUES (NOW(), %s, %s, %s, %s)
    """
    # Execute with parameterized query (SQL injection safe)
    db_conn.execute(query, (
        sensor_data["pressure_boom"],
        sensor_data["pressure_arm"],
        sensor_data["angle_boom"],
        sensor_data["temperature"]
    ))
```

---

**Last updated:** 2026-01-30
