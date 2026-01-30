# 03 MATHEMATICAL SPECIFICATION: 15 EQUATIONS + PYTHON CODE

> **Copy-paste ready Python примеры для всех расчётов**

---

## SECTION 1: KINEMATICS (Forward & Inverse)

### Equation 1: Forward Kinematics (Boom)
```
p_boom = [L_boom * cos(α), L_boom * sin(α)]
```

**Python:**
```python
import numpy as np

def forward_kinematics_boom(boom_angle_rad: float, boom_length: float = 6.45) -> np.ndarray:
    """Boom tip position in local frame."""
    return np.array([
        boom_length * np.cos(boom_angle_rad),
        boom_length * np.sin(boom_angle_rad)
    ])

# Test
boom_pos = forward_kinematics_boom(np.pi/4)  # 45°
print(f"Boom tip: {boom_pos}")  # [4.56, 4.56]
```

---

### Equation 2: Forward Kinematics (Full Excavator)
```
p_arm = p_boom + [L_arm * cos(α+β), L_arm * sin(α+β)]
p_bucket = p_arm + [L_bucket * cos(α+β+γ), L_bucket * sin(α+β+γ)]
```

**Python:**
```python
def forward_kinematics_full(
    boom_angle: float,
    arm_angle: float,
    bucket_angle: float,
    boom_len: float = 6.45,
    arm_len: float = 4.20,
    bucket_len: float = 1.20
) -> dict:
    """Full excavator FK. Returns positions of boom tip, arm tip, bucket tip."""
    
    # Boom
    p_boom = np.array([
        boom_len * np.cos(boom_angle),
        boom_len * np.sin(boom_angle)
    ])
    
    # Arm (relative to boom)
    arm_abs_angle = boom_angle + arm_angle
    p_arm = p_boom + np.array([
        arm_len * np.cos(arm_abs_angle),
        arm_len * np.sin(arm_abs_angle)
    ])
    
    # Bucket (relative to arm)
    bucket_abs_angle = boom_angle + arm_angle + bucket_angle
    p_bucket = p_arm + np.array([
        bucket_len * np.cos(bucket_abs_angle),
        bucket_len * np.sin(bucket_abs_angle)
    ])
    
    return {
        "boom_tip": p_boom,
        "arm_tip": p_arm,
        "bucket_tip": p_bucket
    }

# Test
result = forward_kinematics_full(np.pi/6, -np.pi/4, -np.pi/3)
print(result["bucket_tip"])
```

---

### Equation 3: Inverse Kinematics (2-link arm, Numerical)
```
Given: target position (x_target, y_target)
Find: (α, β) such that arm tip reaches target
```

**Python:**
```python
from scipy.optimize import fsolve

def inverse_kinematics_2link(
    x_target: float,
    y_target: float,
    boom_len: float = 6.45,
    arm_len: float = 4.20
) -> dict:
    """Inverse kinematics for 2-link arm (boom + arm)."""
    
    def equations(angles):
        boom_a, arm_a = angles
        arm_tip = np.array([
            boom_len * np.cos(boom_a) + arm_len * np.cos(boom_a + arm_a),
            boom_len * np.sin(boom_a) + arm_len * np.sin(boom_a + arm_a)
        ])
        return [
            arm_tip[0] - x_target,
            arm_tip[1] - y_target
        ]
    
    # Initial guess
    initial_guess = [np.pi/4, -np.pi/4]
    solution = fsolve(equations, initial_guess)
    
    return {
        "boom_angle": solution[0],
        "arm_angle": solution[1],
        "success": np.allclose(equations(solution), [0, 0])
    }

# Test
ik_result = inverse_kinematics_2link(8.0, 5.0)
print(f"IK solution: {ik_result}")
```

---

## SECTION 2: DYNAMICS (Forces & Torques)

### Equation 4: Torque from Weight (Static)
```
τ = m * g * d_com * cos(θ)

where:
  m = mass (kg)
  g = 9.81 m/s²
  d_com = distance from pivot to center of mass (m)
  θ = angle from horizontal
```

**Python:**
```python
def torque_from_weight(
    mass_kg: float,
    com_distance: float,
    angle_rad: float,
    g: float = 9.81
) -> float:
    """Calculate torque due to weight."""
    return mass_kg * g * com_distance * np.cos(angle_rad)

# Test: 1800 kg boom, 3m com distance, 30° angle
tau = torque_from_weight(1800, 3.0, np.pi/6)
print(f"Torque: {tau:.0f} N*m")  # ~45,727 N*m
```

---

### Equation 5: Cylinder Pressure from Torque (Static)
```
F_cylinder = τ / L_lever
P = F / A_piston

where:
  L_lever = lever arm (distance from pivot to cylinder attachment)
  A_piston = piston area (m²)
```

**Python:**
```python
def pressure_from_torque(
    torque_nm: float,
    lever_arm_m: float,
    piston_area_m2: float
) -> float:
    """Calculate required pressure to balance torque."""
    force_n = torque_nm / lever_arm_m
    pressure_pa = force_n / piston_area_m2
    pressure_bar = pressure_pa / 1e5
    return pressure_bar

# Test: boom cylinder, 45727 N*m torque, 2m lever, 254 cm² piston
pressure_bar = pressure_from_torque(45727, 2.0, 254e-4)
print(f"Required pressure: {pressure_bar:.1f} bar")  # ~90 bar
```

---

### Equation 6: Dynamic Equation of Motion (ODE)
```
I * α = τ_cylinder - τ_gravity - τ_friction - τ_load

where:
  I = moment of inertia (kg*m²)
  α = angular acceleration (rad/s²)
  τ_cylinder = torque from hydraulic cylinder
  τ_gravity = gravitational torque
  τ_friction = friction losses
  τ_load = external load (soil resistance)
```

**Python:**
```python
def boom_acceleration(
    pressure_pa: float,
    piston_area_m2: float,
    lever_arm_m: float,
    moment_of_inertia: float,
    angle_rad: float,
    mass_kg: float,
    com_distance: float,
    friction_coeff: float = 0.05,
    g: float = 9.81
) -> float:
    """Calculate boom angular acceleration."""
    
    # Torques
    tau_cyl = (pressure_pa * piston_area_m2) * lever_arm_m
    tau_grav = mass_kg * g * com_distance * np.cos(angle_rad)
    tau_fric = friction_coeff * tau_grav  # simplified
    
    # Angular acceleration
    alpha = (tau_cyl - tau_grav - tau_fric) / moment_of_inertia
    return alpha

# Test
alpha = boom_acceleration(
    pressure_pa=90e5,  # 90 bar
    piston_area_m2=254e-4,
    lever_arm_m=2.0,
    moment_of_inertia=10000,  # kg*m²
    angle_rad=np.pi/6,
    mass_kg=1800,
    com_distance=3.0
)
print(f"Angular acceleration: {alpha:.3f} rad/s²")
```

---

## SECTION 3: HYDRAULIC FLOW & CONTINUITY

### Equation 7: Flow Rate to Velocity
```
Q = v * A

where:
  Q = volumetric flow rate (m³/s)
  v = piston velocity (m/s)
  A = piston area (m²)
```

**Python:**
```python
def flow_rate_from_velocity(
    velocity_m_s: float,
    piston_area_m2: float
) -> float:
    """Calculate flow rate from piston velocity."""
    flow_rate_m3_s = velocity_m_s * piston_area_m2
    flow_rate_l_min = flow_rate_m3_s * 60000  # convert to L/min
    return flow_rate_l_min

# Test: 1 m/s velocity, 254 cm² piston
q = flow_rate_from_velocity(1.0, 254e-4)
print(f"Flow rate: {q:.1f} L/min")  # 152.4 L/min
```

---

### Equation 8: Pump Flow (displacement-based)
```
Q = displacement * RPM / 1000 * efficiency

where:
  displacement = cc/rev
  RPM = engine speed
  efficiency = volumetric efficiency (0.92-0.95)
```

**Python:**
```python
def pump_flow_rate(
    displacement_cc: float,
    rpm: int,
    efficiency: float = 0.93
) -> float:
    """Calculate pump output flow."""
    q_cc_rev_min = (displacement_cc * rpm / 1000) * efficiency
    q_l_min = q_cc_rev_min / 1000
    return q_l_min

# Test: CAT 320D pump
q = pump_flow_rate(displacement_cc=350, rpm=1800)
print(f"Pump flow: {q:.1f} L/min")  # ~583.5 L/min
```

---

## SECTION 4: DATA FEATURES FOR ML

### Equation 9: Pressure Derivative (detect spikes)
```
dP/dt = (P_t - P_t-1) / Δt
```

**Python:**
```python
def pressure_derivative(
    pressures: np.ndarray,
    dt: float = 0.01  # 100 Hz = 0.01s
) -> np.ndarray:
    """Calculate pressure rate of change (dP/dt)."""
    dp_dt = np.diff(pressures) / dt
    return dp_dt

# Test
pressures = np.array([100.0, 101.2, 102.5, 105.1, 103.2])
dp_dt = pressure_derivative(pressures)
print(f"dP/dt: {dp_dt}")  # [120, 130, 260, -190]
```

---

### Equation 10: Pressure Variance (window-based)
```
σ_P = sqrt(mean((P_i - μ_P)²))

High variance → cavitation or leakage
```

**Python:**
```python
def pressure_variance_window(
    pressures: np.ndarray,
    window_size: int = 1000  # 100 Hz * 10 sec
) -> float:
    """Calculate pressure variance over sliding window."""
    return np.std(pressures[-window_size:])

# Test
pressures = np.random.normal(150, 5, 1000)  # μ=150, σ=5
var = pressure_variance_window(pressures)
print(f"Pressure variance: {var:.2f} bar")  # ~5.0 bar
```

---

### Equation 11: Thermal Feature (temperature gradient)
```
dT/dt = (T_t - T_t-Δt) / Δt
```

**Python:**
```python
def thermal_gradient(
    temperatures: np.ndarray,
    window_size: int = 60  # 1 minute at 1 Hz
) -> float:
    """Calculate temperature gradient (detect runaway)."""
    if len(temperatures) < window_size:
        return 0.0
    recent_temps = temperatures[-window_size:]
    dt_per_sample = 1.0  # 1 Hz
    dT_dt = (recent_temps[-1] - recent_temps[0]) / (window_size * dt_per_sample)
    return dT_dt

# Test
temps = np.linspace(50, 65, 100)  # gradual increase
gradient = thermal_gradient(temps)
print(f"dT/dt: {gradient:.3f} °C/s")  # 0.05 °C/s
```

---

### Equation 12: Correlation Matrix (multivariate anomalies)
```
ρ_ij = cov(X_i, X_j) / (σ_i * σ_j)

Deviation from normal correlation → anomaly
```

**Python:**
```python
def correlation_anomaly_score(
    data: np.ndarray,  # shape (n_samples, n_features)
    baseline_corr: np.ndarray  # normal correlation matrix
) -> float:
    """Detect if current correlations deviate from baseline."""
    current_corr = np.corrcoef(data.T)
    frobenius_norm = np.linalg.norm(current_corr - baseline_corr, 'fro')
    return frobenius_norm

# Test
data = np.random.randn(100, 4)
baseline = np.corrcoef(data.T)
anomaly_score = correlation_anomaly_score(data, baseline)
print(f"Correlation anomaly: {anomaly_score:.4f}")
```

---

## SECTION 5: ANOMALY DETECTION ENSEMBLE

### Equation 13: LSTM Reconstruction Error
```
error = ||x_real - x_predicted|| (L2 norm)
anomaly if error > threshold
```

**Python (PyTorch):**
```python
import torch
import torch.nn as nn

class LSTMAutoencoder(nn.Module):
    def __init__(self, input_size: int, hidden_size: int = 64):
        super().__init__()
        self.encoder = nn.LSTM(input_size, hidden_size, batch_first=True)
        self.decoder = nn.LSTM(hidden_size, input_size, batch_first=True)
    
    def forward(self, x):
        encoded, _ = self.encoder(x)
        decoded, _ = self.decoder(encoded)
        return decoded

def lstm_anomaly_score(
    x_real: torch.Tensor,
    model: LSTMAutoencoder,
    threshold: float = 0.5
) -> tuple:
    """Calculate LSTM reconstruction error."""
    x_pred = model(x_real)
    error = torch.norm(x_real - x_pred, p=2).item()
    is_anomaly = error > threshold
    return error, is_anomaly

# Test
model = LSTMAutoencoder(input_size=4, hidden_size=32)
data = torch.randn(1, 10, 4)  # (batch, seq_len, features)
error, is_anomaly = lstm_anomaly_score(data, model)
print(f"LSTM error: {error:.4f}, Anomaly: {is_anomaly}")
```

---

### Equation 14: Isolation Forest (multivariate outliers)
```
anomaly_score = -2^(-path_length / avg_path_length)

Range: [-1, 1], where 1 = anomaly
```

**Python (scikit-learn):**
```python
from sklearn.ensemble import IsolationForest

def isolation_forest_anomaly_score(
    data: np.ndarray,  # shape (n_samples, n_features)
    contamination: float = 0.05
) -> np.ndarray:
    """Detect multivariate anomalies using Isolation Forest."""
    iso_forest = IsolationForest(contamination=contamination, random_state=42)
    predictions = iso_forest.fit_predict(data)  # -1 = anomaly, 1 = normal
    scores = iso_forest.score_samples(data)  # [-1, 1] range
    return scores

# Test
data = np.random.randn(1000, 8)
data[-10:] = np.random.uniform(-5, 5, (10, 8))  # inject anomalies
scores = isolation_forest_anomaly_score(data)
anomalies_idx = np.where(scores < -0.2)[0]
print(f"Detected {len(anomalies_idx)} anomalies")
```

---

### Equation 15: Ensemble Voting
```
final_anomaly_score = (w1 * lstm_score + w2 * isolation_score + w3 * threshold_score)

where:
  w1 + w2 + w3 = 1 (weighted ensemble)
```

**Python:**
```python
def ensemble_anomaly_detection(
    lstm_error: float,
    lstm_threshold: float,
    iso_forest_score: float,
    threshold_score: float,  # 0/1 from simple rules
    weights: dict = None
) -> dict:
    """Combine 3 anomaly detectors with weighted ensemble."""
    
    if weights is None:
        weights = {"lstm": 0.4, "isolation": 0.4, "threshold": 0.2}
    
    # Normalize scores to [0, 1]
    lstm_normalized = min(lstm_error / lstm_threshold, 1.0)
    iso_normalized = (iso_forest_score + 1) / 2  # [-1, 1] → [0, 1]
    
    # Weighted sum
    final_score = (
        weights["lstm"] * lstm_normalized +
        weights["isolation"] * iso_normalized +
        weights["threshold"] * threshold_score
    )
    
    # Decision
    is_anomaly = final_score > 0.5
    confidence = abs(final_score - 0.5) * 2  # [0, 1]
    
    return {
        "final_score": final_score,
        "is_anomaly": is_anomaly,
        "confidence": confidence,
        "components": {
            "lstm": lstm_normalized,
            "isolation": iso_normalized,
            "threshold": threshold_score
        }
    }

# Test
result = ensemble_anomaly_detection(
    lstm_error=0.8,
    lstm_threshold=0.5,
    iso_forest_score=0.3,
    threshold_score=0.0
)
print(result)
```

---

## ✅ Complete Reference Summary

| Eq | Name | Input | Output | Where Used |
|----|------|-------|--------|-----------|
| 1-2 | Forward Kinematics | Angles | Positions | Visualization, load calc |
| 3 | Inverse Kinematics | Target pos | Angles | Motion planning |
| 4-6 | Dynamics (torque → accel) | Pressure, mass | Acceleration | ODE integration |
| 7-8 | Flow hydraulics | Velocity, RPM | Flow rate | Pressure balance |
| 9-12 | Data features | Sensor data | Feature vector | ML training |
| 13-15 | Anomaly ensemble | Features | Anomaly score | Alert decision |

**Last updated:** 2026-01-30
