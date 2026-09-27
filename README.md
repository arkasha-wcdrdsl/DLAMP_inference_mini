# DLAMP RWRF / FCNv2 72 hours outputs

## Description

本資料夾包含有 `era5_*_IC.npy` 的個案。每個個案各有兩種 DLAMP boundary strategy。
每個 strategy 包含一個 NetCDF4 檔案，一個日期會有兩個檔案：

```text
forecast_current_72h_YYYYMMDDHH_oneway.nc
forecast_current_72h_YYYYMMDDHH_nudging.nc
```

## NetCDF dimensions

| Dimension | Size | Meaning |
|---|---:|---|
| `time` | 73 | F000–F072，每小時一筆，UTC valid time |
| `level` | 13 | pressure levels |
| `y`, `x` | 224, 224 | DLAMP grid |
| `upper_var` | 6 | upper-air variable labels |
| `surface_var` | 7 | surface variable labels |

## NetCDF variables

| Variable | Dimensions | dtype | Meaning |
|---|---|---|---|
| `time` | `(time)` | int64 | UTC valid time，Unix epoch seconds |
| `level` | `(level)` | int32 | `[50,100,150,200,250,300,400,500,600,700,850,925,1000]` hPa |
| `upper_var` | `(upper_var)` | string | `Z,T,U,V,W,Qv` |
| `surface_var` | `(surface_var)` | string | surface channel labels |
| `upper` | `(time,level,y,x,upper_var)` | float32 | physical upper-air values |
| `surface` | `(time,y,x,surface_var)` | float32 | physical surface values |
| `lat` | `(y,x)` | float32 | latitude grid |
| `lon` | `(y,x)` | float32 | longitude grid |
| `valid` | `(time)` | uint8 | `1` means the forecast sample exists |
| `source_init_time` | `(time)` | int64 | source RWRF initial time |
| `input_valid_time` | `(time)` | int64 | source RWRF input valid time |
| `source_path` | `(time)` | string | source RWRF NetCDF path |

`upper` 的 variable 順序：

```text
Z, T, U, V, W, Qv
```

`surface` 的 variable 順序：

```text
T@Meter2,
U@Meter10,
V@Meter10,
Qv@Meter2,
SST@SeaSurface,
PSFC@Surface,
RAINNC@Surface
```

## Units

| Variable | Unit |
|---|---|
| `Z` | m |
| `T`, `T@Meter2`, `SST@SeaSurface` | K |
| `U`, `V`, `W`, `U@Meter10`, `V@Meter10` | m s-1 |
| `Qv`, `Qv@Meter2` | kg kg-1 |
| `PSFC@Surface` | Pa |
| `RAINNC@Surface` | mm |

## Initial field and boundary

DLAMP initial field uses direct RWRF:

```text
RWRF folder: t-3h
RWRF file:   valid at t
```

For example, `20241031 00UTC` uses:

```text
/wk3/rwrf_data/2024-10-30_21/wrfout_d01_2024-10-31_00_interp.nc
```

Boundary exchange remains FCNv2 at DLAMP steps `6,12,18,...,72`.

## RAINNC at F000

The direct RWRF initial field has no DLAMP output `RAINNC` value at F000. Therefore the F000 `RAINNC@Surface` values use the NetCDF fill value. F001–F072 contain the DLAMP output rainfall channel.

## Reading example

```python
from netCDF4 import Dataset

path = "20241031_0000/forecast_current_72h_2024103100_oneway.nc"

with Dataset(path) as ds:
    times = ds.variables["time"][:]
    levels_hpa = ds.variables["level"][:]
    upper_names = ds.variables["upper_var"][:].tolist()
    surface_names = ds.variables["surface_var"][:].tolist()
    upper = ds.variables["upper"][:]
    surface = ds.variables["surface"][:]
```

The stored arrays are physical values. They are not direct ONNX standardized inputs.
