
# Kalman Filter Photovoltaic Power Prediction Model 

## ✅ STEP 1: Hourly Forecast of Solar Irradiance

**उद्देश्य:** हर घंटे में Solar Panel पर कितनी धूप (irradiance) पड़ती है, यह पता करना।

### 🔹 1.1 Astronomical Irradiance (बिना बादलों के)
Formula:
```
ION = (24/π) * ISC * (1 + 0.033 * cos(360N / 365)) * (cosφ * cosδ * sin(ws) + sinφ * sinδ)
```
- ISC = Solar Constant (1367 W/m²)
- N = दिन का क्रम (1 Jan = 1)
- φ = Latitude
- δ = Solar Declination
- ws = Solar Hour Angle

### 🔹 1.2 Sunshine Time Correction (Tilt का प्रभाव)
Tilted panels के कारण sunrise/sunset समय correction करना पड़ता है।
```
tup = 12 + wup / 15
tdown = 12 + wdown / 15
```

### 🔹 1.3 Irradiance Attenuation (Atmosphere और Clouds से)
```
ID = ION * Pm * cos(i)
Id = 0.5 * ION * sin(α) * [1 - (Pm / (1 - 1.4 * ln(P) * cos²θ))]
IS = ID + Id
```

### 🔹 1.4 Cloud Cover Adjustment
Clouds से धूप की attenuation को adjust किया जाता है:
```
d = c + b * CC + a * CC²
Iscs = IS * d
```
जहाँ CC = Cloud Cover



## ✅ STEP 2: Photoelectric Conversion Model

**उद्देश्य:** Step 1 से मिले irradiance को electrical power में convert करना।

### 🔹 2.1 Current Equation
```
I = Isc * [1 - k2 * (exp(V / (k1 * Voc)) - 1)]
```

### 🔹 2.2 Empirical Constants
```
k1 = ((Vmp / Voc) - 1) / ln(1 - Imp / Isc)
k2 = (1 - Imp / Isc) * exp(-Vmp / (k1 * Voc))
```

### 🔹 2.3 Power Output
```
P(V) = V * Isc * [1 - k2 * (exp(V / (k1 * Voc)) - 1)]
```

### 🔹 2.4 Temperature & Radiation Effects
```
Isc = Iscref * (G / Gref) * (1 + a * ΔT)
Voc = Vocref * ln(e + b * ΔG) * (1 - c * ΔT)
Imp = Impref * (G / Gref) * (1 + a * ΔT)
Vmp = Vmpref * ln(e + b * ΔG) * (1 - c * ΔT)
```
जहाँ ΔG = G/Gref - 1 और ΔT = T - Tref



## ✅ STEP 3: Kalman Filter Prediction Model Based on Forecasting Experience

**उद्देश्य:** Real-time correction से prediction को ज़्यादा accurate बनाना।

### 🔹 3.1 State and Observation Equations
```
xt = A * xt-1 + Qt
yt = H * xt + Rt
```

### 🔹 3.2 Observation Polynomial Equation
```
yt = x0 + x1 * mt + x2 * mt² + x3 * mt³ + Rt
```

### 🔹 3.3 Noise Estimates
```
Qt = (1/6) * Σ[(xi+1 - xi - avg)²]
Rt = (1/6) * Σ[(yi - Hi*xi - avg)²]
```

### 🔹 3.4 Kalman Prediction-Correction Process
```
xt/t-1 = A * xt-1
Pt/t-1 = A * Pt-1 * A^T + Qt

Kt = Pt/t-1 * H^T / (H * Pt/t-1 * H^T + Rt)
xt = xt/t-1 + Kt * (yt - H * xt/t-1)
Pt = (I - Kt * H) * Pt/t-1
```



## ✅ STEP 4: Simulation and Result Analysis

### 🔸 4.1 डेटा का स्रोत
- DKASC (Desert Knowledge Australia Solar Centre)
- April 20–22, 2019

### 🔸 4.2 मॉडल्स की तुलना
1. Kalman Filter with Forecasting Experience
2. Kalman Filter with Historical Experience
3. DBN (Deep Belief Network)

### 🔸 4.3 Error Metrics (Absolute Percentage Error)
```
Eape = |ypred - yactual| / yactual * 100%
```

### 🔸 4.4 Result Summary
- Kalman (Forecasting): Error < 3% (normal), < 8% (low light)
- Kalman (Historical): Error up to 15%
- DBN: Max error up to 18%, needs 30 days data
- Execution Time: Kalman = 7.8s, DBN = 108s

### 🔸 4.5 निष्कर्ष
Kalman Filter based on forecasting experience is:
- Fast, lightweight
- Accurate even during sudden weather changes
- Better than DBN and historical-based models


 
