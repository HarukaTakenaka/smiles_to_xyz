# SMILES → XYZ

Convert SMILES strings into 3D Cartesian coordinates (`.xyz`) for small organic molecules. Two ways to use it:

- **Web page:** runs entirely in your browser; nothing to install. https://harukatakenaka.github.io/smiles_to_xyz/
- **Jupyter notebook:** RDKit-based, for batch work and scripting.

The geometries are force-field quality. They are intended as starting structures for xtb or DFT optimization, not as final geometries.

---

## Web page

### How to use

1. Open the page.
2. Enter one molecule per line: the SMILES, then an optional name separated by a space.

   ```
   CC(=O)Oc1ccccc1C(=O)O aspirin
   Cn1cnc2c1c(=O)n(C)c(=O)n2C caffeine
   C[C@H](N)C(=O)O L-alanine
   ```

   Lines starting with `#` are ignored. If no name is given, the SMILES is used as the file name.
3. Click **Convert**.
4. Get the results:
   - Click a molecule in the list to view it in 3D. Drag the view to rotate it.
   - **Download** saves that molecule's `.xyz` file.
   - **Copy** puts the XYZ text on the clipboard.
   - **Download all (.zip)** gives one `.xyz` file per molecule.
   - **Download combined .xyz** gives a single multi-frame file.

### Options

| Option | Default | Meaning |
|---|---|---|
| Conformers to try | 10 | Number of conformers generated per molecule; the lowest-energy one is kept. Increase for flexible molecules. |
| MMFF94s+ minimization | on | Minimizes each conformer with the force field before comparing energies. |

### Privacy

The site is public, but your molecules are never uploaded. All conversion happens in your own browser. The page needs an internet connection to load the chemistry library and fonts from public CDNs (jsDelivr, Google Fonts).

---

## Jupyter notebook

`smiles_to_xyz.ipynb` does the same job with RDKit.

```bash
pip install rdkit          # or: conda install -c conda-forge rdkit
jupyter notebook smiles_to_xyz.ipynb
```

- `smiles_to_xyz(smiles, name, n_confs=10)` returns the XYZ text and a summary dictionary (formula, charge, force-field energy).
- The batch cell converts a `{name: SMILES}` dictionary. It writes the files to `xyz_out/`, plus a combined `all_molecules.xyz` and `xyz_out.zip`.
- `read_smiles_file()` loads molecules from a `.smi`/`.txt` file (`SMILES name` per line) or from a CSV with `smiles` and `name` columns.

It also works in Google Colab; uncomment the `%pip install rdkit` line first.

---

## Output format

Standard XYZ, in Ångström, with all hydrogens included:

```
21

C      3.460880    -0.875803    -4.592096
C      3.627505    -0.294232    -3.220159
O      ...
```

Line 1 is the number of atoms. Line 2 is the comment line, **left blank on purpose** so it cannot be mistaken for a DFT energy. The XYZ format requires this line to exist, so don't delete it. The remaining lines are element and x, y, z coordinates.

The MMFF energy is shown on the web page and in the notebook summary for comparing conformers only. It is not written to the files and has no meaning for QM calculations.

---

## Using the files for xtb and DFT

The XYZ file does not store charge or spin, so set them yourself.

**xtb**

```bash
xtb molecule.xyz --opt --chrg 0 --uhf 0    # --uhf = number of unpaired electrons
```

**ORCA**

```
! B3LYP D3BJ def2-SVP Opt
* xyzfile 0 1 molecule.xyz
```

**Gaussian:** copy the coordinate lines (everything after line 2) into a `.com` file:

```
%chk=molecule.chk
#p opt freq B3LYP/6-31G(d) EmpiricalDispersion=GD3BJ

molecule

0 1
C      3.460880    -0.875803    -4.592096
...

```

Gaussian requires a blank line at the end of the file.

A typical workflow is this geometry → xtb optimization → DFT optimization. For flexible molecules, run a proper conformer search (for example with CREST) before DFT, because the lowest of a few force-field conformers is not guaranteed to be the global minimum.

---

## Method

| | Web page | Notebook |
|---|---|---|
| Library | [OpenChemLib](https://github.com/cheminfo/openchemlib-js) 9.25.1 (JavaScript) | [RDKit](https://www.rdkit.org) |
| 3D embedding | OpenChemLib conformer generator (torsion library) | ETKDGv3 |
| Force field | MMFF94s+ | MMFF94 (UFF fallback) |
| Selection | lowest-energy conformer | lowest-energy conformer |

Stereocenters (`@`, `@@`) and double-bond geometry (`/`, `\`) written in the SMILES are preserved. If stereochemistry is not specified, one stereoisomer is chosen arbitrarily.

## Limitations

- These are force-field geometries; always optimize them at your QM level of choice.
- Elements without MMFF parameters, such as boron and most metals, are not minimized. The web page flags these molecules, and their geometries should be checked visually.
- Spin multiplicity is not determined. Ordinary closed-shell organics are singlets; set radicals and triplets by hand.
- The tools are meant for small organic molecules. Large or highly flexible systems will work, but the single structure kept may be far from the global minimum.

## Credits

The web page uses OpenChemLib (BSD-3-Clause), loaded from jsDelivr. The notebook uses RDKit (BSD-3-Clause). 
