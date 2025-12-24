# Tutorial: Creating Functionalized Graphene with GOPY

This tutorial explains how to generate graphene sheets functionalized with **hydroxyl (-OH)** and **carboxyl (-COOH)** groups using GOPY for computational chemistry calculations.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Creating Pristine Graphene](#creating-pristine-graphene)
5. [Adding Functional Groups](#adding-functional-groups)
6. [Size Recommendations](#size-recommendations)
7. [Converting to Other Formats](#converting-to-other-formats)
8. [Examples](#examples)
9. [Controlling Functional Group Placement](#controlling-functional-group-placement)
10. [Creating Pores and Holes](#creating-pores-and-holes)
11. [Adding Heavy Metal Ions](#adding-heavy-metal-ions)

---

## Prerequisites

### Required Python Packages

```bash
pip install numpy scipy
```

| Package | Version | Purpose |
|---------|---------|---------|
| Python | ≥ 3.8 (3.9+ recommended) | Runtime |
| NumPy | Any recent version | Array operations |
| SciPy | Any recent version | Spatial calculations |

### Verify Installation

```bash
python -c "import numpy; import scipy; print('All packages installed!')"
```

---

## Installation

1. Clone or download the GOPY repository
2. Navigate to the GOPY directory
3. Verify GOPY works:

```bash
cd /path/to/GOPY
python GOPY.py help
```

---

## Quick Start

Generate a small functionalized graphene in two commands:

```bash
# Step 1: Create pristine graphene (10 Å × 10 Å)
python GOPY.py generate_PG 10 10 pristine.pdb

# Step 2: Add 3 OH groups
python GOPY.py generate_GO pristine.pdb 0 0 3 graphene_OH.pdb
```

---

## Creating Pristine Graphene

### Command Syntax

```bash
python GOPY.py generate_PG <X_dimension> <Y_dimension> <output_file>
```

- **X_dimension**: Width in Ångströms (Å)
- **Y_dimension**: Height in Ångströms (Å)
- **output_file**: Name of the output PDB file

### Size Reference Table

| Dimensions (Å) | Approximate Atoms | Use Case |
|----------------|-------------------|----------|
| 10 × 10 | ~50 | Quick tests, ORCA DFT |
| 15 × 15 | ~105 | Small DFT calculations |
| 20 × 20 | ~180 | Medium DFT calculations |
| 30 × 30 | ~400 | Large DFT (demanding) |
| 50 × 50 | ~1000 | Classical MD only |
| 100 × 100 | ~4000 | Classical MD simulations |

### Examples

```bash
# Small (for quantum chemistry)
python GOPY.py generate_PG 10 10 PG_small.pdb

# Medium
python GOPY.py generate_PG 20 20 PG_medium.pdb

# Large (for MD simulations)
python GOPY.py generate_PG 50 50 PG_large.pdb
```

---

## Adding Functional Groups

### Command Syntax

```bash
python GOPY.py generate_GO <input_PG.pdb> <N_COOH> <N_epoxy> <N_OH> <output_file>
```

| Parameter | Description |
|-----------|-------------|
| `input_PG.pdb` | Path to pristine graphene PDB file |
| `N_COOH` | Number of carboxyl (-COOH) groups to add |
| `N_epoxy` | Number of epoxy (-O-) groups to add (set to 0 if not needed) |
| `N_OH` | Number of hydroxyl (-OH) groups to add |
| `output_file` | Name of the output PDB file |

### Functional Group Details

#### Hydroxyl (-OH)
- **Residue name**: `H1A`
- **Atoms**: CY (carbon), OL (oxygen), HK (hydrogen)
- **Total atoms per group**: 3

```
    H (HK)
    |
    O (OL)
    |
    C (CY) ← attached to graphene plane
```

#### Carboxyl (-COOH)
- **Residue name**: `C1A`
- **Atoms**: CY (carbon), C4 (carboxyl carbon), OJ (=O), OK (-O-), HK (hydrogen)
- **Total atoms per group**: 5

```
        O (OJ)
        ‖
    C (C4)
   / \
  O   O-H
(OK) (HK)
  |
  C (CY) ← attached to graphene plane
```

---

## Size Recommendations

### For Quantum Chemistry (ORCA, Gaussian, etc.)

| System Size | Graphene | Functional Groups | Total Atoms | Calculation Time |
|-------------|----------|-------------------|-------------|------------------|
| Minimal | 10×10 Å | 2-3 | ~60 | Minutes |
| Small | 15×15 Å | 5-10 | ~130 | Hours |
| Medium | 20×20 Å | 10-15 | ~220 | Many hours |
| Maximum practical | 25×25 Å | 15-20 | ~300 | Days |

### For Classical MD (GROMACS, AMBER, LAMMPS)

| System Size | Graphene | Functional Groups | Total Atoms |
|-------------|----------|-------------------|-------------|
| Small | 30×30 Å | 20-50 | ~500 |
| Medium | 50×50 Å | 50-100 | ~1200 |
| Large | 100×100 Å | 100-400 | ~4500 |

### Coverage Calculation

To calculate functional group coverage:

```
Coverage (%) = (Number of functional groups / Number of C atoms) × 100
```

Example: 50 OH groups on 1000 C atoms = 5% coverage

---

## Converting to Other Formats

### PDB to XYZ (for ORCA)

```bash
# Create XYZ file from PDB
awk 'BEGIN{n=0} /^ATOM/{n++} END{print n; print "Functionalized graphene"}' input.pdb > output.xyz
awk '/^ATOM/{
  atom=$3
  if(atom ~ /^C/) elem="C"
  else if(atom ~ /^O/) elem="O"
  else if(atom ~ /^H/) elem="H"
  else elem=substr(atom,1,1)
  printf "%s  %12.6f  %12.6f  %12.6f\n", elem, $6, $7, $8
}' input.pdb >> output.xyz
```

### Using Open Babel

```bash
# Install Open Babel
conda install -c conda-forge openbabel

# Convert PDB to various formats
obabel input.pdb -O output.xyz      # XYZ format
obabel input.pdb -O output.mol2     # MOL2 format
obabel input.pdb -O output.sdf      # SDF format
```

---

## Examples

### Example 1: OH-Only Graphene (Small, for ORCA)

```bash
# Create 10×10 Å pristine graphene
python GOPY.py generate_PG 10 10 PG_10x10.pdb

# Add 5 OH groups (no COOH, no epoxy)
python GOPY.py generate_GO PG_10x10.pdb 0 0 5 graphene_5OH.pdb
```

**Result**: ~50 C + 15 atoms (5 OH × 3) = ~65 total atoms

---

### Example 2: COOH-Only Graphene (Small, for ORCA)

```bash
# Create 15×15 Å pristine graphene
python GOPY.py generate_PG 15 15 PG_15x15.pdb

# Add 5 COOH groups (no epoxy, no OH)
python GOPY.py generate_GO PG_15x15.pdb 5 0 0 graphene_5COOH.pdb
```

**Result**: ~105 C + 25 atoms (5 COOH × 5) = ~130 total atoms

---

### Example 3: Mixed COOH + OH (Medium, for DFT)

```bash
# Create 20×20 Å pristine graphene
python GOPY.py generate_PG 20 20 PG_20x20.pdb

# Add 3 COOH and 6 OH groups
python GOPY.py generate_GO PG_20x20.pdb 3 0 6 graphene_mixed.pdb
```

**Result**: ~180 C + 15 (COOH) + 18 (OH) = ~213 total atoms

---

### Example 4: High Coverage for MD

```bash
# Create 50×50 Å pristine graphene
python GOPY.py generate_PG 50 50 PG_50x50.pdb

# Add 50 COOH and 100 OH groups (~15% coverage)
python GOPY.py generate_GO PG_50x50.pdb 50 0 100 GO_high_coverage.pdb
```

---

### Example 5: Following Lerf-Klinowski Model

The Lerf-Klinowski model suggests GO has approximately:
- C:O ratio of ~2:1
- Formula approximation: C₂₀(OH)₂(O)₂(COOH)₁

```bash
# Create graphene
python GOPY.py generate_PG 50 50 PG.pdb  # ~1000 atoms

# Calculate groups needed:
# 1000 C atoms / 20 = 50 formula units
# COOH: 50 × 1 = 50
# Epoxy: 50 × 2 = 100
# OH: 50 × 2 = 100

python GOPY.py generate_GO PG.pdb 50 100 100 GO_lerf_klinowski.pdb
```

---

## Controlling Functional Group Placement

### Top-Side Only Placement

By default, GOPY places functional groups randomly on **both sides** of the graphene sheet. To place them on the **top side only**, modify the `top_or_down()` function in `GOPY.py`:

```python
def top_or_down():
    """Random top or below draw."""
    # For TOP ONLY:
    return 1
    
    # For BOTTOM ONLY:
    # return -1
    
    # For RANDOM (both sides):
    # ct = random.randint(1,2)
    # return 1 if ct == 1 else -1
```

### Random Seed for Reproducibility

To get reproducible random placement, add at the beginning of GOPY.py:

```python
import random
random.seed(42)  # Use any integer for reproducible results
```

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Encoding error on line 2377 | Add `#!/usr/bin/python` and `# -*- coding: latin-1 -*-` at file start |
| "Not all groups placed" | Reduce number of requested groups or increase graphene size |
| Python version errors | Use Python 3.9+ |

### Verifying Output

Check your output file:

```bash
# Count total atoms
wc -l output.pdb

# Count each residue type
grep -c "GGG" output.pdb  # Pristine carbon atoms
grep -c "H1A" output.pdb  # Hydroxyl atoms (divide by 3 for groups)
grep -c "C1A" output.pdb  # Carboxyl atoms (divide by 5 for groups)
```

---

## Sample ORCA Input File

For quantum chemistry calculations, use this template:

```
# ORCA input for functionalized graphene
! B3LYP def2-SVP D3BJ
! Opt
! PAL4

%maxcore 4000

* xyzfile 0 1 graphene.xyz
```

---

## References

- GOPY Paper: https://doi.org/10.1016/j.softx.2020.100586
- Tutorial on GO topology: See `Tutorial_GO_topology.md` in this repository

---

## Creating Pores and Holes

GOPY can create **nanopores** in graphene sheets, useful for membrane simulations and transport studies.

### Command Syntax

```bash
python GOPY.py generate_hole <input.pdb> <N_holes> <min_size> <max_size> <direction> <location> <cleanup> <output.pdb>
```

### Parameters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `N_holes` | Integer | Number of holes to create |
| `min_size` | Integer | Minimum hole size (in atoms removed) |
| `max_size` | Integer | Maximum hole size (in atoms removed) |
| `direction` | `u` or `m` | `u` = unidirectional (linear), `m` = multidirectional (irregular) |
| `location` | `i` or `e` | `i` = interior only (not touching edges), `e` = can touch edges |
| `cleanup` | `a` or `n` | `a` = cleanup small fragments, `n` = no cleanup |

### Hole Types

```
Unidirectional (u):          Multidirectional (m):
                              
  ○ ○ ○ ○ ○ ○                   ○ ○ ○ ○ ○ ○
  ○ ○ · · · ○                   ○ · · · ○ ○
  ○ ○ · · · ○     vs            ○ · · · · ○
  ○ ○ · · · ○                   ○ ○ · · ○ ○
  ○ ○ ○ ○ ○ ○                   ○ ○ ○ ○ ○ ○
                              
  (elongated)                   (irregular/round)
```

### Examples

#### Single Small Pore
```bash
# Create pristine graphene
python GOPY.py generate_PG 30 30 PG_30x30.pdb

# Create 1 hole, size 5-10 atoms, multidirectional, interior only, with cleanup
python GOPY.py generate_hole PG_30x30.pdb 1 5 10 m i a graphene_1pore.pdb
```

#### Multiple Pores
```bash
# Create 5 holes of size 10-20 atoms each
python GOPY.py generate_hole PG_30x30.pdb 5 10 20 m i a graphene_5pores.pdb
```

#### Large Irregular Pores (for membrane studies)
```bash
python GOPY.py generate_PG 50 50 PG_50x50.pdb
python GOPY.py generate_hole PG_50x50.pdb 3 20 40 m i a membrane_pores.pdb
```

#### Elongated Slit Pores
```bash
# Unidirectional creates more slit-like holes
python GOPY.py generate_hole PG_30x30.pdb 2 15 25 u i a graphene_slits.pdb
```

### Pore Size Estimation

| Atoms Removed | Approximate Pore Diameter |
|---------------|---------------------------|
| 5-10 | ~3-5 Å |
| 10-20 | ~5-8 Å |
| 20-40 | ~8-12 Å |
| 40-80 | ~12-18 Å |

### Combining Holes with Functional Groups

You can first create holes, then add functional groups to the edges:

```bash
# Step 1: Create pristine graphene
python GOPY.py generate_PG 30 30 PG.pdb

# Step 2: Create holes
python GOPY.py generate_hole PG.pdb 2 10 15 m i a PG_holes.pdb

# Step 3: Add functional groups (they may attach to pore edges)
python GOPY.py generate_GO PG_holes.pdb 5 0 10 GO_porous.pdb
```

### Important Notes

1. **Cleanup recommended**: Always use `a` for cleanup to remove dangling atoms
2. **Interior pores**: Use `i` to keep pores away from sheet edges (better for periodic simulations)
3. **Size limits**: Don't request holes larger than ~30% of sheet area
4. **Multiple attempts**: GOPY makes up to 50 attempts per hole; if placement fails, reduce hole count or size

---

## Quick Command Reference

```bash
# Generate pristine graphene
python GOPY.py generate_PG <X> <Y> <output.pdb>

# Add functional groups (COOH, epoxy, OH)
python GOPY.py generate_GO <input.pdb> <N_COOH> <N_epoxy> <N_OH> <output.pdb>

# Create holes/pores
python GOPY.py generate_hole <input.pdb> <N_holes> <min_size> <max_size> <u|m> <i|e> <a|n> <output.pdb>

# Add COO⁻ instead of COOH (use alternate script)
python GOPY_COO.py generate_GO <input.pdb> <N_COO> <N_epoxy> <N_OH> <output.pdb>

# Get help
python GOPY.py help
```

