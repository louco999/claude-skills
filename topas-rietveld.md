# TOPAS Rietveld Refinement Skill

## Trigger
Use this skill whenever the user provides XRD data files (.xy, .xdd, .raw, .dat, .xye) and/or CIF structure files and asks for Rietveld refinement, Pawley refinement, phase analysis, quantitative analysis, or any crystallographic fitting using TOPAS. Also trigger when the user mentions "精修", "Rietveld", "TOPAS", "XRD fitting", or asks to refine lattice parameters / atomic positions.

---

## TOPAS Software Location
- GUI: `C:\topas\ta.exe`
- CLI engine: `C:\topas\tc.exe` — runs `.inp` files: `C:/topas/tc.exe myfile.inp`
- Examples: `C:\topas\test_examples\`
- Output fit data: use `Out_Yobs_Ycalc_and_Difference(filename.txt)` keyword in INP
- Plot with Python: `C:/Users/hanyi/gsas2main/python.exe`

---

## INP File Structure (complete template)

```
/* Comment block */
r_wp  0          ' will be filled after refinement
do_errors        ' compute ESDs

xdd "datafile.xy"           ' or RAW(), DAT(), XDD() — matches file extension
  lam                        ' wavelength block (use instead of CuKa2 for custom)
    ymin_on_ymax 0.001
    la 0.5791 lo 1.540596 lh 0.437    ' Cu Ka1
    la 0.3209 lo 1.544490 lh 0.52     ' Cu Ka2
  ' OR use macros:
  ' CuKa2(0.001)    — Cu Ka1+Ka2 simple
  ' CuKa5(0.0001)   — Cu Ka with 5 lines (most accurate)

  Radius(173)              ' goniometer radius in mm (173 or 200.5 typical)
  LP_Factor(17)            ' Lorentz-polarization: 17=no mono, 26.4=graphite mono, 27=Ge mono
  Full_Axial_Model(12, 20, 12, 5.1, @ 8)  ' axial divergence: L_S, L_D, Rs, 2xh_D, 2xh_S
  ' OR: Simple_Axial_Model(@, 12)
  Divergence(1)            ' divergence slit in degrees (0.5, 1, 2 typical)
  Slit_Width(0.1)          ' receiving slit width in mm

  bkg @ 0 0 0 0 0 0        ' Chebyshev polynomial background (add more terms if needed)
  One_on_X(@, 1000)        ' 1/X background for low-angle air scatter
  Zero_Error(@, 0)         ' zero-point error in 2theta (degrees); also: ZE(@, 0)
  ' Absorption(@, 50)       ' flat plate absorption correction

  str                      ' structure block (one per phase)
    space_group  Ia-3      ' use TOPAS notation: F_M_3_M, R_-3_c, P_21/C etc.
    ' OR: STR(space_group_name, "phase_name")

    phase_name "My Phase"
    scale @ 0.001          ' scale factor — always refine (@)

    ' Lattice — use appropriate macro:
    Cubic(@ 5.41)
    ' Tetragonal(@ 3.80, @ 9.51)
    ' Hexagonal(@ 4.91, @ 5.41)
    ' Orthorhombic(@ 5.00, @ 7.00, @ 9.00)
    ' Monoclinic(@ 6.00, @ 8.00, @ 5.00, @ 110.0)   — b-unique, beta
    ' Trigonal(@ 5.00, @ 13.00)
    ' Triclinic: a @ b @ c @ al @ be @ ga @
    ' Manual: a @ 5.00  b @ 7.00  c @ 9.00

    ' Atomic sites
    site  Ce1  x 0   y 0   z 0       occ Ce+4  1  beq @ 0.5
    site  O1   x 0.25 y 0.25 z 0.25  occ O-2   1  beq @ 0.5
    ' beq = isotropic B factor; beq @ 1 is a good starting value
    ' For fixed coordinates use value without @
    ' For symmetry constraints use: x = 1/3;  y = 2/3;  z @ 0.25

    ' Microstructure (peak broadening)
    CS_L(@, 100)         ' Lorentzian crystallite size (nm) — most common
    ' CS_G(@, 100)       ' Gaussian crystallite size
    ' CS(@, 100)         ' combined = CS_L + CS_G in quadrature
    Strain_L(@, 0.01)    ' Lorentzian microstrain
    ' Strain_G(@, 0.01)  ' Gaussian microstrain

    ' Optional
    MVW(0, 0, 0)                        ' molecular volume weight (for density calc)
    r_bragg 0                           ' phase R-Bragg factor
    PO(@, 1, , 0 0 1)                   ' March-Dollase preferred orientation [hkl]
    ' PO_Spherical_Harmonics(sh, 8)     ' spherical harmonics PO (more flexible)
    append_bond_lengths                 ' output bond lengths
    Out_CIF_STR(output.cif)            ' output refined CIF

  ' Output
  Out_Yobs_Ycalc_and_Difference(fit.txt)
```

---

## Space Group Name Syntax (TOPAS)
Spaces replaced by underscores, special chars:
- `-` → `-` (keep minus for inversion)
- `/` → `/` or `_`
- Examples:
  - `Fm-3m` → `F_M_-3_M` or `F_M_3_M`
  - `R-3c` → `R_-3_c` or `R_-3_C`
  - `P21/c` → `P_21/C` or `P_1_21/C_1`
  - `C2/c` → `C_2/C` or `C_1_2/C_1`
  - `Pnma` → `P_N_M_A`
  - `P63/mmc` → `P_63/M_M_C`
  - `I41/amd` → `I_41/A_M_D` or `I_41/A_M_D:2`

---

## Wavelength Macros
```
CuKa1(0.001)    ' Cu Ka1 only, lh=linewidth
CuKa2(0.001)    ' Cu Ka1 + Ka2 (2 lines)
CuKa5(0.0001)   ' Cu Ka with 5 components (most accurate for lab data)
CoKa3(0.0001)   ' Co radiation
MoKa2(0.001)    ' Mo radiation
```

LP_Factor values:
- `0` = synchrotron / no polarization
- `17` = no monochromator (typical Bragg-Brentano)
- `26.4` = graphite monochromator
- `27` = Ge monochromator

---

## Rietveld Refinement Strategy (Multi-Round)

### Round 1 — Scale only
```
scale @ 0.001
' All other structure params fixed (no @)
' No atomic positions refined
CS_L(!csl, 100)   ' ! = fixed but named
```
**Goal:** Rwp drops significantly. If not, check space group or data range.

### Round 2 — Lattice parameters
```
Cubic(@ 5.41)    ' add @ to refine
scale @ 0.001
CS_L(!csl, 100)
```
**Goal:** Peak positions align. Rwp improves.

### Round 3 — Background + Zero error
```
bkg @ 0 0 0 0 0
Zero_Error(@, 0)
One_on_X(@, 1000)   ' if low-angle scatter present
```

### Round 4 — Peak shape (microstructure)
```
CS_L(@, 100)       ' refine crystallite size
Strain_L(@, 0.01)  ' refine microstrain (if peaks are asymmetric)
```
**Note:** Only refine CS_L OR Strain_L first, then both. Don't refine CS_G and CS_L simultaneously at first.

### Round 5 — Atomic positions (x, y, z)
Add `@` to fractional coordinates. Fix special-position coordinates (those constrained by symmetry).
```
site Fe  x @ 0.14  y 0.25  z @ -0.04   occ Fe 1 beq @ 0.5
```

### Round 6 — Thermal parameters (Beq)
```
site Fe  x @ 0.14  y 0.25  z @ -0.04   occ Fe 1 beq @ 0.5
```
Start all beq at 0.5–1.0. Constrain to reasonable range: `beq @ 0.5 min 0.01 max 5`

### Round 7 — Occupancy (if needed)
Only refine if physical reason (vacancies, mixed occupancy):
```
occ Fe+3 @ 0.9
```

### Round 8 — Preferred orientation (if needed)
```
PO(@, 1, , 0 0 1)              ' March-Dollase along [001]
' OR
PO_Spherical_Harmonics(sh, 6)  ' more flexible, use order 4–8
```

### Round 9 — Full refinement
All parameters on simultaneously. Use `do_errors` to get ESDs.

---

## Reading CIF Files
Extract from CIF manually:
1. `_symmetry_space_group_name_H-M` or `_space_group_name_H-M_alt` → space group
2. `_cell_length_a/b/c`, `_cell_angle_alpha/beta/gamma` → lattice
3. `_atom_site_label`, `_atom_site_fract_x/y/z`, `_atom_site_occupancy`, `_atom_site_B_iso_or_equiv` → sites

---

## Quality Indicators
| Indicator | Excellent | Good | Acceptable | Poor |
|-----------|-----------|------|------------|------|
| Rwp | <5% | 5–8% | 8–12% | >15% |
| GOF (χ²) | ~1.0 | 1–2 | 2–4 | >5 |
| R-Bragg | <2% | 2–4% | 4–8% | >10% |
| Rwp/Rexp | ~1.0 | 1–2 | 2–3 | >3 |

---

## Common Problems and Fixes

| Problem | Symptom | Fix |
|---------|---------|-----|
| Wrong space group | Systematic missing peaks | Check CIF, try related SG |
| Wrong lattice | All peaks shifted | Refine cell params first |
| Background too high | Broad humps not fit | Add more bkg terms, or One_on_X |
| Peak too broad/narrow | Systematic width mismatch | Adjust CS_L, Strain_L |
| Preferred orientation | Intensity mismatch for specific planes | Add PO or PO_Spherical_Harmonics |
| Rwp stuck > 15% | No improvement | Check data range, exclude bad regions |
| beq goes negative | Unstable refinement | Fix beq, check atom position |
| Scale factor → 0 | Phase absent | Check if phase exists in data |
| Diverging parameters | Over-parameterized | Fix some params, reduce correlation |

---

## Useful Keywords
```
Exclude { 20 25 }          ' exclude 2theta range (e.g., sample holder peak)
start_X 10                 ' start data at 10 degrees
finish_X 80               ' end data at 80 degrees
rebin_with_dx_of 0.02      ' rebin noisy data
convolution_step 2         ' faster calculation for broad peaks
num_runs 5                 ' run refinement N times
continue_after_convergence ' keep running after convergence
iters 100                  ' max iterations
process_times              ' print timing info
verbose -1                 ' suppress output (for batch)
only_penalties             ' minimize penalties only (no diffraction data)
bootstrap_errors 100       ' bootstrap error estimation
```

---

## Multi-Phase Refinement
```
xdd "data.xy"
  CuKa5(0.0001)
  ...instrument params...
  bkg @ 0 0 0 0 0

  str
    phase_name "Phase 1"
    space_group Fm-3m
    Cubic(@ 5.41)
    ...
    scale @ 0.001
    MVW(0, 0, 0)
    weight_percent 0    ' auto-calculated

  str
    phase_name "Phase 2"
    space_group Pnma
    Orthorhombic(@ 5.0, @ 7.0, @ 9.0)
    ...
    scale @ 0.0005
    MVW(0, 0, 0)
    weight_percent 0
```
Weight % is automatically calculated when `MVW` and `scale` are present.

---

## Pawley Refinement (for indexing / unit cell check)
```
xdd "data.xy"
  ...instrument params...
  bkg @ 0 0 0 0 0

  hkl_Is
    space_group P21/c
    a @ 10.0  b @ 8.0  c @ 6.0  be @ 100.0
    CS_L(@, 200)
    default_I_attributes 1 val_on_continue = Rand(0,5);
```

---

## Outputting Results
```
Out_Yobs_Ycalc_and_Difference(fit.txt)   ' 3-column: Yobs Ycalc Diff
Out_CIF_STR(refined.cif)                  ' refined structure as CIF
Out_CIF_Bonds_Angles(refined.cif)         ' bond lengths/angles
Create_hklm_d_Th2_Ip_file(peaks.hkl)     ' peak list
append_bond_lengths                        ' to console output

' Custom output:
out results.txt
  Out(a, "a = %.6f Angstrom\n")
  Out(r_wp, "Rwp = %.4f %%\n")
```

---

## Running TOPAS from Command Line
```bash
# Single file
C:/topas/tc.exe myfile.inp

# Check output: myfile.out (log), fit.txt (Yobs/Ycalc/Diff)

# Multi-run for better convergence
# Add to inp: num_runs 3
```

---

## Plotting Fit Results (Python)
```python
import numpy as np
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

# fit.txt has 3 columns: Yobs Ycalc Diff
data = np.loadtxt('fit.txt')
yobs, ycalc, diff = data[:,0], data[:,1], data[:,2]

# Generate 2theta axis from start_X, step, n_points
# e.g., start=10, step=0.01, n=7001 → linspace(10, 80, 7001)
tth = np.linspace(start_X, finish_X, len(yobs))

fig, axes = plt.subplots(2, 1, figsize=(14, 8), dpi=150,
                         gridspec_kw={'height_ratios': [3, 1], 'hspace': 0.06})
ax = axes[0]
ax.plot(tth, yobs,  color='#1565C0', lw=0.7, alpha=0.85, label='Observed')
ax.plot(tth, ycalc, color='#C62828', lw=1.0, alpha=0.9,  label='Calculated')
ax.legend(); ax.set_ylabel('Intensity'); ax.tick_params(labelbottom=False)

ax2 = axes[1]
ax2.plot(tth, diff, color='#2E7D32', lw=0.7, label='Difference')
ax2.axhline(0, color='gray', lw=0.5, ls='--')
ax2.set_xlabel('2θ (degree)'); ax2.set_ylabel('Difference')

plt.savefig('fit.png', dpi=180, bbox_inches='tight')
```
Use Python at: `C:/Users/hanyi/gsas2main/python.exe`

---

## Workflow for User-Provided CIF + XY Data

1. **Parse CIF** → extract space group, a/b/c/angles, atom sites (x,y,z,occ,beq)
2. **Read XY** → determine 2theta range, step size, radiation source
3. **Build INP** → use template above, set start_X/finish_X from data
4. **Round 1** → scale only (tc.exe run)
5. **Round 2** → + lattice
6. **Round 3** → + background, zero error
7. **Round 4** → + peak shape
8. **Round 5** → + atomic positions
9. **Round 6** → + beq, full do_errors
10. **Parse output** → read r_wp, GOF, refined params from .out file
11. **Plot** → generate fit.png with Python
12. **Report** → summarize Rwp, GOF, lattice params, notable features
