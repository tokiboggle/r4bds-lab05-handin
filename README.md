# Project Contributors

- Name (student id) — GitHub username
- Add the rest of the group here

# Lab 5 hand-in

This is the shared working repository. The code is intentionally empty.

The report has to answer two questions:

1. What characterises the peptides binding to the HLAs?
2. What characterises the T-cell receptors binding to the peptide–HLA complexes?

Write the analysis in `R/`. The hand-in that gets rendered and zipped for DTU Learn is `doc/lab05_handin.qmd`.

# How to get the data

Do not commit data. `data/` is listed in `.gitignore`.

Download `ImmuneCODE-MIRA-Release002.1.zip` from the immuneACCESS project for Nolan et al. 2020 (DOI 10.21417/ADPT2020COVID). Use `peptide-detail-ci.csv` and `subject-metadata.csv` from that zip. Do not use the superseded `ImmuneCODE-MIRA-Release002.zip`.

Put the two csv files in `data/_raw/` and do not edit them. `R/01_load.qmd` should read those files and write the loaded tables.

# How the folders fit together

- `data/_raw/` holds the original files.
- `data/01_dat_load.tsv` and `data/01_dat_load_meta.tsv` are written by `R/01_load.qmd`.
- `data/02_dat_clean.tsv` and `data/02_dat_clean_meta.tsv` are written by `R/02_clean.qmd`.
- `data/03_dat_aug.tsv` is written by `R/03_augment.qmd`.
- `R/00_all.qmd` runs the whole project.
- `R/99_proj_func.R` is where repeated code goes.
- `results/` holds the rendered html files and the plots that go in the report.
- `doc/lab05_handin.qmd` is the micro-report.

Open `lab05_handin.Rproj` before rendering. Each `.qmd` should run on its own, reading the table written by the previous script.

# Using it from RStudio

This is the workflow from [Lab 7](https://r4bds.github.io/lab07.html).

1. RStudio → New Project → Version Control → Git. Repository URL: `https://github.com/tokiboggle/r4bds-lab05-handin`.
2. Open every `.qmd` in the Visual editor. Mixing Visual and Source on the same file is what produces the merge conflicts in that lab.
3. In the Git tab: stage, commit, **Pull**, then Push.
4. Do new work on a branch named with your student id. Push the branch and open a pull request into `main`.
5. `data/` and `data/_raw/` are in `.gitignore`. GitHub is for the code, not the csv files. `.RData` and `.Rproj.user` are ignored for the same reason: they belong to one person's session.
