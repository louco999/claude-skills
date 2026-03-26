# TOPAS Rietveld Refinement Skill
# TOPAS 里特沃尔德精修技能

## Trigger / 触发条件

**Use this skill when the user:**
- Provides XRD data files (`.xy`, `.xdd`, `.raw`, `.dat`, `.xye`) and/or CIF structure files
- Asks for Rietveld refinement, Pawley refinement, phase analysis, or quantitative phase analysis
- Mentions keywords: **"精修"**, **"Rietveld"**, **"TOPAS"**, **"XRD fitting"**, **"拟合"**
- Wants to refine lattice parameters, atomic positions, thermal parameters, or microstructure

**当用户：**
- 提供 XRD 数据文件和/或 CIF 结构文件
- 要求 Rietveld 精修、Pawley 精修、物相分析、定量分析
- 提到"精修"、"拟合"、"Rietveld"、"TOPAS"等关键词

---

## Software Location / 软件位置

| Item | Path |
|------|------|
| TOPAS GUI | `C:\topas\ta.exe` |
| TOPAS CLI engine | `C:\topas\tc.exe` |
| Run refinement | `C:/topas/tc.exe filename.inp` (no extension needed) |
| Examples | `C:\topas\test_examples\` |
| Python (with numpy/matplotlib) | `C:\Users\hanyi\miniforge3\python.exe` |

> **Output files:** `filename.out` (log + refined params), fit data via `Out_Yobs_Ycalc_and_Difference("fit.txt")`

---

## Complete INP Template / 完整 INP 模板

```
/* Comment / 注释 */
r_wp  0          ' will be updated after refinement
do_errors        ' calculate ESDs (误差标准偏差)

xdd "datafile.xy"                    ' or: RAW() DAT() XDD()
  x_calculation_step = Yobs_dx_at(Xo);  convolution_step 4
  start_X  10                        ' starting 2theta
  finish_X 80                        ' ending 2theta

  ' --- Wavelength / 波长 ---
  lam ymin_on_ymax 0.0001
    la 1.0  lo 1.540596  lh 0.437   ' custom: weight, lambda, linewidth
  ' Shortcuts: CuKa2(0.001)  CuKa5(0.0001)  CoKa3(0.0001)  MoKa2(0.001)
  ' Synchrotron: la 1.0  lo 0.69  lh 0.01

  ' --- Instrument / 仪器参数 ---
  LP_Factor(17)             ' 0=synchrotron, 17=no mono, 26.4=graphite, 27=Ge
  Full_Axial_Model(12, 20, 12, 5.1, @ 8)  ' axial divergence (lab diffractometer)
  ' Simple_Axial_Model(@, 10)              ' simpler alternative
  Divergence(1)             ' divergence slit in degrees
  Slit_Width(0.1)           ' receiving slit in mm
  Radius(173)               ' goniometer radius in mm (173 or 200.5)
  ' For synchrotron: LP_Factor(90), Simple_Axial_Model with small value

  ' --- Background / 背景 ---
  bkg @ 0 0 0 0 0 0 0 0 0  ' Chebyshev polynomial (add more terms if needed)
  One_on_X(@, 1000)         ' 1/X term for low-angle air scatter
  Zero_Error(@, 0)          ' zero-point error in 2theta; also: ZE(@, 0)
  ' Absorption(@, 50)       ' flat-plate absorption correction

  ' --- Structure Block / 结构块 ---
  str
    space_group  "F M -3 M"   ' TOPAS accepts standard HM notation with spaces
    ' Common: "R 3"  "P 21/c"  "C 2/c"  "I 41/a m d"  "R -3 c"

    ' Lattice / 晶格参数
    Cubic(@ 5.41)
    ' Tetragonal(@ 3.80, @ 9.51)
    ' Hexagonal(@ 4.91, @ 5.41)   ' also for trigonal hexagonal setting
    ' Orthorhombic(@ 5.00, @ 7.00, @ 9.00)
    ' Monoclinic(@ 6.00, @ 8.00, @ 5.00, @ 110.0)
    ' Triclinic:  a @ 5  b @ 7  c @ 9  al @ 90  be @ 100  ga @ 90
    ' Manual a=b constraint (e.g. R3):  a hex_a 5.19  b hex_a 5.19  c @ 13.8

    ' Atomic sites / 原子坐标
    site  Ce1  x 0     y 0     z 0      occ Ce+4  1  beq @ 0.5
    site  O1   x 0.25  y 0.25  z 0.25   occ O-2   1  beq @ 0.5
    ' Fixed coord:        x 0  (no @)
    ' Symmetry constraint: x = 1/3;  y = 2/3;  z @ 0.25
    ' Shared Beq:         prm bCo 1.0  min 0.01 max 5  then  beq = bCo;

    ' Peak shape / 峰形
    PV_Peak_Type(pku,0.02, pkv,0.02, pkw,0.02, !pkx,0.00, pky,0.18, !pkz,0.00)
    ' CS_L(@, 100)      ' Lorentzian crystallite size (nm)
    ' Strain_L(@, 0.01) ' Lorentzian microstrain

    scale @ 0.001         ' always refine
    r_bragg 0
    MVW(0, 0, 0)          ' mass/volume/weight% output
    append_bond_lengths
    Out_CIF_STR("refined.cif")

Out_Yobs_Ycalc_and_Difference("fit.txt")
```

---

## LP_Factor Guide / LP 因子说明

| Source / 来源 | Value | Notes |
|--------|-------|-------|
| Synchrotron 同步辐射 | `90` | Horizontally polarized beam 水平极化 |
| No monochromator 无单色器 | `17` | Typical Bragg-Brentano lab 实验室最常见 |
| Graphite monochromator 石墨单色器 | `26.4` | Common lab setup |
| Ge monochromator Ge 单色器 | `27` | High-resolution lab |

---

## Space Group Notation / 空间群写法

TOPAS 推荐使用标准 Hermann-Mauguin 符号（带空格）：

| Standard | TOPAS Input | Example phase |
|----------|-------------|---------------|
| Fm-3m | `"F M -3 M"` | CeO2, NaCl |
| R-3c | `"R -3 c"` | Al2O3 (corundum) |
| P21/c | `"P 21/c"` | monoclinic, most common |
| C2/c | `"C 2/c"` | monoclinic |
| Pnma | `"P n m a"` | orthorhombic |
| P63/mmc | `"P 63/m m c"` | hexagonal |
| R3 | `"R 3"` | Co3TeO6 (HP) |
| I41/amd | `"I 41/a m d"` | TiO2 anatase |

Also accepted (underscore style 下划线写法): `F_M_-3_M`, `R_-3_c`, `P_21/C`

---

## Multi-Round Refinement Strategy / 多轮精修策略

> 这是从 TOPAS 官方 examples 学习总结的**推荐精修流程**。

### Round 1 — Scale Only / 第一轮：仅比例因子
```
scale @ 0.001
' All other params fixed: use ! prefix
```
✅ **Goal:** Rwp drops significantly.
❌ If Rwp barely changes → wrong space group or completely wrong structure.

---

### Round 2 — Lattice + Background + Zero Error / 第二轮：格子+背景+零点
```
Cubic(@ 5.41)       ' open lattice with @
bkg @ 0 0 0 0 0
Zero_Error(ze, 0)
scale @ 0.001
```
✅ **Goal:** Peak positions align. Rwp improves 10–30%.

---

### Round 3 — Peak Shape / 第三轮：峰形（最关键一步）
```
PV_Peak_Type(pku,0.02, pkv,0.02, pkw,0.02, !pkx,0.00, pky,0.18, !pkz,0.00)
Simple_Axial_Model(axial, 10.0)   ' for lab diffractometer
' LP_Factor(90) + Simple_Axial_Model for synchrotron
```
✅ **Goal:** Peak widths match. Often the **largest Rwp improvement**.
⚠️ If Rwp still > 30% after lattice + peak shape → **wrong starting atomic coordinates!**

---

### Round 4 — Atomic Positions / 第四轮：原子坐标
```
site Fe  x @ 0.14  y 0.25  z @ -0.04   occ Fe 1  beq !b 1.0
```
- Fix special-position coordinates (constrained by symmetry)
- **Keep beq fixed** at this stage to avoid correlation
- Open x, y, z one site at a time if unstable

---

### Round 5 — Thermal Parameters / 第五轮：热参数
```
prm bCo 1.0  min 0.01 max 5
prm bO  1.0  min 0.01 max 5
site Co1  x 0  y 0  z @ 0.37  occ Co 1  beq = bCo;
site O1   x @ 0.25  y @ 0.01  z @ 0.25  occ O 1  beq = bO;
```
- Constrain same-element beq (减少参数相关性)
- If beq hits min limit → correlation with position; fix beq or widen range

---

### Round 6 — Full Refinement + ESDs / 第六轮：全精修+误差
```
do_errors
chi2_convergence_criteria 0.0001
append_bond_lengths
Out_CIF_STR("final.cif")
```

---

## Quality Indicators / 精修质量指标

| Indicator 指标 | Excellent 优秀 | Good 好 | Acceptable 可接受 | Poor 差 |
|-----------|-----------|------|------------|------|
| **R_wp** | < 5% | 5–8% | 8–12% | > 15% |
| **GOF (χ²)** | ~1.0 | 1–2 | 2–4 | > 5 |
| **R-Bragg** | < 2% | 2–4% | 4–8% | > 10% |

> 💡 **GOF < 1** → over-parameterized (过拟合).
> 💡 **GOF > 3** → model needs improvement or data has systematic errors.
> 💡 **R-Bragg** reflects structure quality independent of background.

---

## Common Problems & Fixes / 常见问题与解决方法

| Problem 问题 | Symptom 症状 | Fix 解决方案 |
|-------------|-------------|------------|
| Wrong space group 空间群错误 | Systematic absent/extra peaks | Check CIF, try related SG |
| Wrong starting positions 坐标起点错 | Rwp stuck >30% even peaks aligned | Use published coords at similar P/T |
| Background too high 背景过高 | Broad humps not fit | Add more bkg terms; `One_on_X` |
| Peak widths wrong 峰宽不符 | Systematic shape mismatch | Refine PV_Peak_Type or CS_L/Strain |
| Preferred orientation 择优取向 | Specific reflection intensities wrong | `PO(@,1,,0 0 1)` or `PO_Spherical_Harmonics(sh,6)` |
| beq hits minimum 热参数到极限 | beq → 0 or negative | Fix beq; check site symmetry |
| Scale → 0 比例因子为零 | Phase absent from data | Verify phase; check 2theta range |
| Atomic position unstable 坐标不稳 | Large jumps in x/y/z | Add `min`/`max`; check multiplicity |

---

## Useful Keywords / 常用关键词参考

```
' Data range / 数据范围
start_X 10        finish_X 80
Exclude { 30 40 }                   ' exclude region 排除区间
rebin_with_dx_of 0.02               ' rebin noisy data 合并数据点

' Convergence / 收敛控制
iters 100000
chi2_convergence_criteria 0.001
continue_after_convergence
num_runs 3                          ' repeat N times 重复N次

' Output / 输出
MVW(0, 0, 0)
append_bond_lengths
Out_CIF_STR("out.cif")
Create_hklm_d_Th2_Ip_file("peaks.hkl")
Out_Yobs_Ycalc_and_Difference("fit.txt")

' Advanced / 高级
do_errors                           ' ESD calculation 计算误差
bootstrap_errors 100                ' bootstrap error 自举误差
convolution_step 4                  ' faster for broad peaks
x_calculation_step = Yobs_dx_at(Xo);  ' adaptive step 自适应步长
process_times                       ' print timing 输出计算时间
```

---

## Synchrotron Data Tips / 同步辐射数据要点

```
lam ymin_on_ymax 0.0001 la 1.0 lo 0.69 lh 0.01  ' monochromatic
LP_Factor(90)          ' fully polarized beam 全极化束
Simple_Axial_Model(axial, 2.0)   ' small value; or omit
x_calculation_step = Yobs_dx_at(Xo);  convolution_step 4
```

| Feature | Lab XRD | Synchrotron |
|---------|---------|------------|
| Wavelength | Cu Kα (1.54 Å) | Tunable (e.g. 0.69 Å) |
| LP_Factor | 17 / 26.4 / 27 | **90** |
| Peak width | PV + axial model | PV only (narrow) |
| Background | Often significant | Near zero (normalized data) |
| Scale factor | ~0.001 | ~1e-9 (if normalized) |

---

## Quantitative Phase Analysis / 定量物相分析

```
str
  phase_name "Quartz"
  Hexagonal(@ 4.913, @ 5.405)
  space_group "P 31 2 1"
  ...
  scale @ 0.001
  MVW(0, 0, 0)
  weight_percent 0    ' auto-calculated 自动计算

str
  phase_name "Corundum"
  Hexagonal(@ 4.758, @ 12.99)
  space_group "R -3 c"
  ...
  scale @ 0.001
  MVW(0, 0, 0)
  weight_percent 0
```
> Weight% = automatically calculated when `MVW` and `scale @` are both present.

---

## Plotting with Python / Python 出图

```python
# Python: C:\Users\hanyi\miniforge3\python.exe
import numpy as np, matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

data = np.loadtxt(r'path\to\fit.txt')   # 3 cols: Yobs  Ycalc  Diff
yobs, ycalc, diff = data[:,0], data[:,1], data[:,2]
tth = np.linspace(start_X, finish_X, len(yobs))

fig, axes = plt.subplots(2, 1, figsize=(14, 8), dpi=150,
                         gridspec_kw={'height_ratios': [3, 1], 'hspace': 0.05})
ax = axes[0]
ax.plot(tth, yobs,  color='#1565C0', lw=0.8, label='Observed')
ax.plot(tth, ycalc, color='#C62828', lw=1.0, label='Calculated')
ax.set_ylabel('Intensity'); ax.set_xlim(start_X, finish_X)
ax.legend(); ax.tick_params(labelbottom=False)
ax.set_title(f'Rietveld Fit  Rwp = {rwp:.2f}%')

ax2 = axes[1]
ax2.plot(tth, diff, color='#2E7D32', lw=0.7)
ax2.axhline(0, color='gray', lw=0.5, ls='--')
ax2.set_xlabel('2θ (degree)'); ax2.set_ylabel('Difference')
ax2.set_xlim(start_X, finish_X)

plt.tight_layout()
plt.savefig(r'path\to\fit.png', dpi=180, bbox_inches='tight')
```

---

## Full Workflow Summary / 完整精修流程总结

| Step 步骤 | Action 操作 | Expected Rwp 预期 Rwp |
|-----------|------------|----------------------|
| 0. Prepare | Parse CIF + read XY range | — |
| 1. Scale | `scale @` only | 50–90% |
| 2. Lattice | + `a/b/c @` + `bkg @` + `ZE` | 30–60% |
| 3. Peak shape | + `PV_Peak_Type` or `CS_L` | **10–25%** ← biggest drop |
| 4. Positions | + `x/y/z @`, beq fixed | 8–20% |
| 5. Beq | + thermal params | 5–15% |
| 6. Final | `do_errors` + bond lengths + CIF out | **< 10% target** |

> ⚠️ **Critical insight / 关键经验：**
> If Rwp is stuck **above 30%** even after lattice + peak shape → starting atomic coordinates are wrong.
> 如果精修到格子和峰形之后 Rwp 仍然 >30%，说明原子坐标起点有问题。
> **Solution:** Use coordinates from a published structure at similar conditions (P, T).
> **解决：** 使用相近压力/温度条件下已发表的结构坐标作为起点。
