# Weighted EDP Statistics for Seismic Loss Assessment

## 1. Overview

This repository contains two MATLAB codes that improve HAZUS assembly-based loss assessment by using **weighted average EDP statistics** instead of single values (like maximum EDP response). The approach combines two EDP percentiles with an optimized weighting factor (α) to better match detailed component-based results.

**Key Innovation:** Instead of using just maximum EDP, we use:
```
Weighted_EDP = α × 75_Percentile + (1-α) × 25_Percentile
```

## 2. The Two-Step Process

### Step 1: Find Optimal Alpha Values
**File:** `Alpha_Optimization_EDP_Statistics_25_75_Percentiles.m`

- **What it does:** Finds the best α value for each building by matching assembly-based results with FEMA P-58 component-based targets
- **How:** Uses iterative optimization with adaptive step sizes
- **Output:** `Alpha_Values_25_75Percentiles.csv` with α for each building

### Step 2: Apply Alpha Using Regression
**File:** `EDP_Statistics_Sensitivity_analysis.m`

- **What it does:** Uses a regression model `log(α) = f(log(Period))` to calculate α for any building, then computes EAL
- **How:** Applies period-dependent α values to weighted EDP statistics
- **Output:** Complete EAL breakdown in Excel format

## 3. Required Files

### 3.1. Input Data
```
Data/
├── GuanDataBase-IMs.csv              # Ground motion intensities
├── Building_Info_URGENT.xlsx         # Building periods (column 19)
├── FEMAP58_EAL_NormEA.xlsx  # Target EAL (column 11)
├── USGSHazard_data.csv              # Seismic hazard curves
├── Guan_database/                   # Building response files
│   ├── StructuralResponses/EDPsUnder240GMs/
│   │   ├── PeakStoryDrift/Building_X_PSDR.csv
│   │   ├── PeakFloorAcceleration/Building_X_PFA.csv
│   │   └── ResidualStoryDrift/Building_X_RSDR.csv
│   └── BuildingDesigns/Building_X/Geometry.csv
└── HAZUS/                           # Fragility and cost data
    ├── Fragility_Data_Structural.csv
    ├── Fragility_Data_NonstructuralDrift.csv
    ├── Fragility_Data_NonstructuralAcc.csv
    └── Cost_Data.csv
```

## 4. Key Parameters

### Code 1 (Optimization)
```matlab
tolerance = 1e-6;           % Convergence criteria (tighter = more accurate)
initial_step_size = 0.01;   % Starting step size for α
max_iterations = 1000;      % Prevent infinite loops
```

### Code 2 (Regression Application)
```matlab
% Current regression model (35th-65th percentiles)
log_alpha = -0.20187*(log(TimePeriod)) + 1.4919;

% Current weighting approach
Weighted_PSDR = alpha * Percentile75_PSDR + (1-alpha) * Percentile25_PSDR;
```

## 5. Output Files

### From Code 1
- `Alpha_Values_25_75Percentiles.csv` - One α value per building

### From Code 2
Excel file with 11 columns:
| Column | Content | Units |
|--------|---------|-------|
| 1 | Building ID | - |
| 2-6 | EAL by component (Struct, NS-Drift, NS-Acc, Demo, Total) | $ |
| 7-11 | Normalized EAL by component | - |

## 6. Customization

### Change EDP Statistics
```matlab
% Use different percentiles
Weighted_PSDR = alpha * Percentile90_PSDR + (1-alpha) * Percentile10_PSDR;

% Use max-mean combination
Weighted_PSDR = alpha * Max_PSDR + (1-alpha) * Mean_PSDR;
```

### Update Regression Model
```matlab
% After running Code 1, develop your own regression
periods = csvFilePath1(:,19);
alphas = readmatrix('Alpha_Values_25_75Percentiles.csv');
mdl = fitlm(log(periods), log(alphas));

% Update this line in Code 2 with your coefficients
log_alpha = slope*log(TimePeriod) + intercept;
```

## 7. Contact
**Primary Contact:**
- Shiva Baddipalli (PhD Candidate): shivalinga.baddipalli@usu.edu

**Corresponding Author:**  
- Dr. Mohsen Zaker Esteghamati (Assistant Professor): mohsen.zaker@usu.edu
- Department of Civil and Environmental Engineering
- Utah State University, Logan, UT, United States
