# NHSRC PHC Supply Chain - Data Quality Report

## Dataset Overview
- **Total Records**: 6,480
- **Date Range**: 2024-01-01 to 2024-06-28
- **Unique Facilities**: 3
- **Unique SKUs**: 12

## Data Quality Metrics
- **Missing Values**: 0
- **Outliers Flagged**: 145

## Category Distributions

### VED Category Distribution
Vital        2700
Essential    2160
Desirable    1620
dtype: int64

### FSN Category Distribution  
Fast    3240
Slow    3240
dtype: int64

### Expiry Risk Distribution
LOW (>180d)         2636
MEDIUM (90–180d)    1874
HIGH (30–90d)       1633
CRITICAL (<30d)      337
dtype: int64

## Cleaning Operations Applied
1. Column name standardization
2. Date format conversion
3. Missing value imputation
4. Outlier detection (flagged, not removed)
5. Derived field computation
6. Master data validation

## Notes
- Outliers are flagged but not removed to preserve outbreak signals
- All SKUs have corresponding master data
- Data is NHSRC-compliant and ready for analysis

**Generated**: 2025-12-03 17:27:44
