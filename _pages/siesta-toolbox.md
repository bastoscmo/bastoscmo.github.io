---
layout: archive
title: "STB-SUITE — Siesta Toolbox Suite"
permalink: /development/siesta-toolbox/
author_profile: true
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

**A unified command-line toolkit for SIESTA DFT workflows**

![Version](https://img.shields.io/badge/version-1.8.1-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Python](https://img.shields.io/badge/python-%E2%89%A53.9-blue.svg)
![Compatibility](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey.svg)

**Author:** Dr. Carlos M. O. Bastos — University of Brasília (UnB), 2025
[bastoscmo.github.io](https://bastoscmo.github.io)

---

##  Introduction

**STB-SUITE (Siesta Toolbox Suite)** is a comprehensive collection of command-line tools designed to assist users of the **SIESTA** DFT code through every step of their workflow — from input generation to post-processing and structural analysis.

It provides a unified, intuitive interface that simplifies and accelerates computational materials research.

Stable version: 1.8.1

Dev version: 1.9.8

Next stable version 2.0.0 (may 2026)

---

##  Features

###  Calculation Preparation

* **`stb-inputfile`** – Generate FDF input files from structure files with suggested settings.
* **`stb-kgrid`** – Automatically suggest Monkhorst-Pack k-point grids based on a target density.
* **`stb-kpath`** – Generate high-symmetry paths for band-structure calculations.
* **`stb-strain`** – Create supercells with uniaxial or biaxial strain (Cartesian coordinates).

###  Analysis & Post-Processing

* **`stb-bands`** – Analyze SIESTA band structures and calculate band gaps.
* **`stb-dos`** – Parse PDOS.xml files for total and projected density of states.
* **`stb-convdos`** – Apply Gaussian convolution to DOS data for smoothing.
* **`stb-structural`** – Compute lattice parameters, coordination numbers, and ECN values.
* **`stb-symmetry`** – Identify space group, point group, crystal system, and Wyckoff positions.

###  Utilities & Interfaces

* **`stb-translate`** – Convert between structure formats (CIF, POSCAR, FDF, XYZ, XSF, FHI, DFTB).
* **`stb-siesta2wtb`** – Export SIESTA Hamiltonians to the **Wantibexos** tight-binding format.
* **`stb-clean`** – Remove unnecessary calculation files and clean directories.
* **`stb-suite`** – Unified terminal interface providing interactive access to all tools.

---

## What is new in 1.8.1 version?


---

## Installation and Requirements

* **Python ≥ 3.9**
* **Conda** (recommended)

### Python dependencies

```
numpy
matplotlib
ase
pymatgen
spglib
sisl
argparse (builtin)
```

---

##  Installation

###  Conda (recommended)

```bash
conda install bastoscmo::stb_suite
```

###  Manual installation

```bash
git clone https://github.com/bastoscmo/stb-suite.git
cd stb-suite
pip install .
```

---

## ▶️ Usage

Launch the interactive interface:

```bash
stb_suite
```

Run individual tools directly:

```bash
stb-inputfile structure.fdf --type relax
stb-kgrid --file POSCAR --density 0.2
stb-symmetry --input struct.cif --filetype cif
```

## 🧾 License

Distributed under the **MIT License**.
© 2025 Dr. Carlos M. O. Bastos – University of Brasília (UnB)


