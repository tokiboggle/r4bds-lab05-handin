# Expert B — bioinformatics review of Group 21 Lab 5

Reviewer: Expert B (immune repertoire / MHC motifs, tidy R / Quarto).  
Source reviewed: teammate draft `her_group21_lab05.qmd` (uploaded copy; title “Lab 5 Assignment: Group 21”).  
Compared with the course lab (https://r4bds.github.io/lab05.html, tasks T14–T25 and the haplotype disclaimer) and this repo’s hand-in layout (`doc/lab05_handin.qmd`, `README.md`).

This review did **not** re-run ImmuneCODE, did **not** render logos, and does **not** report sequence counts, bit scores, or which residue is tallest. Residue-level sentences below are claims in her prose, not new results. **Do not upload anything to Learn from this file.**

Her draft has no Conclusions heading. The two Analysis paragraphs are treated as the conclusion sentences.

---

## Flags (these undermine the biological claims)

### 1. Allele string cleanup is a 7-character cut, and blank alleles are kept

```r
mutate(Allele_F_1_2 = str_sub(Allele, start = 1, end = 7))
mutate(Allele = str_remove(Allele_F_1_2, "\\*"))
```

The lab metadata preview mixes `A*02:01`, `A*02:01:01`, `A*02:10`, `B*07:02:01`, `C*04:01:01`, and blank `""` HLA cells. For the common 2-field and 3-field strings (`X*DD:DD` and `X*DD:DD:DD`), characters 1–7 then dropping `*` do give `A02:01`, `B07:02`, `C04:01`, and they keep `A*02:10` distinct from `A*02:01`. That part matches lab T19–T20.

Gaps:

- `""` and `NA` are never dropped (T19 asks for that). Blank haplotype cells stay in `meta_data` and enter the join.
- A protein field with 3 digits is silently renamed. `A*02:101` cut to 7 characters becomes `A02:10`, which is already a real allele in the lab preview. Those rows would be analysed as HLA-A*02:10.
- No check that the cut still matches `^[ABC]\\*[0-9]{2}:[0-9]{2}$`.

**Fix**

```r
meta_data <- meta_data |>
  mutate(Allele = str_squish(Allele)) |>
  filter(!is.na(Allele), Allele != "") |>
  mutate(Allele_F_1_2 = str_extract(Allele, "^[ABC]\\*\\d{2,3}:\\d{2,3}")) |>
  filter(!is.na(Allele_F_1_2)) |>
  mutate(Allele = str_remove(Allele_F_1_2, "\\*")) |>
  distinct(Experiment, Allele)
```

Before trusting it, print `count(Allele, sort = TRUE)` and confirm `A02:01` and `A02:10` are separate rows. `str_extract` is safer than `str_sub(..., 1, 7)` because a 3-digit protein field is not aliased onto another allele.

**Methods sentence:** “HLA class I alleles were parsed to two-field resolution (`A*02:01:01` to `A02:01`). Blank allele cells were removed. `A02:01` and `A02:10` were kept as different alleles.”

### 2. `relationship = "many-to-many"` does not assign a restricting allele

```r
left_join(meta_data, by = "Experiment", relationship = "many-to-many") |>
  distinct()
```

Comment in the draft: the warning was “fixed” by `relationship = "many-to-many"` (comment also misspells it `manu-to-many`). The argument only tells dplyr the cartesian product is intentional. Each peptide–TCR row is copied onto every class I allele of that experiment (up to six, fewer if homozygous or blank). `distinct()` removes exact duplicate rows. It does not collapse different alleles.

`filter(Allele == "A02:01")` keeps peptides from experiments where the subject **carries** A*02:01. The peptide may have been presented by another allele on the same haplotype. The lab’s own joined preview shows this: `HTTDPSFLGRY` appears next to `C07:01` in a sampled row, and `LLFLVLIML` appears next to more than one allele label. That is the join, not a restriction result.

**Fix:** keep `relationship = "many-to-many"` so the pipeline runs, delete the “we fixed it” comment, and use an `inner_join` after blank alleles are removed so unmatched experiments do not remain as `Allele == ""` / `NA`.

```r
peptide_meta_data <- peptide_data |>
  inner_join(meta_data, by = "Experiment", relationship = "many-to-many") |>
  distinct()
```

**Methods sentence:** “Peptide rows were joined to every class I allele recorded for the same `Experiment`. The allele label means the subject carries that allele. It is not a measured restricting element.”

### 3. Amino-acid filters are not written back

```r
peptide_data |>
  filter(str_detect(CDR3b, "[^ARNDCQEGHILKMFPSTWYV]", negate = TRUE))
peptide_data |>
  filter(str_detect(peptide, "[^ARNDCQEGHILKMFPSTWYV]", negate = TRUE))
```

Neither pipe assigns `peptide_data`. Lab T14 is only a console exercise. The hand-in still needs the filter applied. The lab preview contains `unproductive+TCRBV28-01+...`, so `CDR3b == "unproductive"` survives into the length bars. The specific logos may miss that string (it is 12 letters), but any non-proteogenic character that does match the later filters stays in the logo.

**Fix**

```r
aa_ok <- "[^ARNDCQEGHILKMFPSTWYV]"
peptide_data <- peptide_data |>
  filter(
    str_detect(CDR3b, aa_ok, negate = TRUE),
    str_detect(peptide, aa_ok, negate = TRUE)
  )
```

**Methods sentence:** “CDR3β and peptide strings were kept only when every character was one of the 20 standard amino acids. Non-productive rearrangements labelled `unproductive` were removed.”

### 4. Haplotype caveat is missing where the claims are made

Background says each person has six HLA alleles (A, B, and C from each parent). The Analysis then says the logos show peptides **binding to** HLA-A*02:01 and HLA-B*07:02, and TCRs **binding** a specific pMHC. The lab disclaimer is the missing sentence: MIRA records that a CDR3β recognised a peptide in a subject, and that subject had a haplotype. Which of the alleles presented the peptide is not in the table. The course says the workaround is out of scope, so the report has to say so.

Also drop the over-count: classical class I is **up to** six alleles; homozygotes have fewer. Class II was correctly removed with `select(-starts_with("D"))` (those columns are `DPA1...`, `DRB1...`, and so on). Do not describe the logos as class II, and do not describe them as “all of HLA class I”.

**Analysis sentence to add under both logo blocks:** “These are peptides (or CDR3β sequences) from experiments whose subject carries the named allele, not biochemical binders of that allele alone.”

### 5. Second CDR3 logo (`C04:01` / `HTTDPSFLGRY`, length 13) should not carry the conclusion

Lab T24 is the required CDR3 logo: `k_CDR3b == 15`, `Allele == "A02:01"`, `peptide == "LLFLVLIML"`. T25 is “play with other combinations”. A condensed micro-report needs T24 only, unless a second logo is a stated contrast.

`HTTDPSFLGRY` is 11 amino acids, so it never enters the 9-mer peptide logos. The CDR3 length is 13, after the text has just committed to length 15. Published MIRA analysis (Mayer-Blackwell et al., tcrdist3 / meta-clonotypes) links this ORF1ab peptide (MIRA1) to **HLA-A*01**, with A*01 enrichment and NetMHCpan support, not to HLA-C*04:01. A `C04:01` filter still returns rows whenever a subject who responded to that peptide also carries C*04:01, which the many-to-many join will do. The logo is then a mixture, not “TCRs that bind C*04:01–HTTDPSFLGRY”.

**Fix:** remove this logo from the hand-in. If the group keeps a second example, pick it only after `count(k_CDR3b)` inside that allele–peptide filter, report that n, and do not pool it with the A*02:01 logo. Expert A should confirm the A*01 restriction before any C*04:01 sentence survives.

### 6. Length “modes” are asserted, not computed, and then not followed

The draft says length 9 and CDR3β length 15 “are the most frequent” and “will be used below”, from untitled bar charts. There is no `count(k_peptide)` or `count(k_CDR3b)`. The second logo uses CDR3 length 13. `ggseqlogo` needs equal-length strings, so a length filter is required. The global mode of all tidy rows is a reasonable default for the peptide logos (T22/T23 use 9-mers; T24 uses CDR3 length 15). It is not a licence to switch to 13 without a count, and it is not proof that every allele prefers 9-mers.

Bars are counts of tidy rows after `distinct()` (one CDR3–V–J–experiment–peptide), not counts of subjects and not unique peptides. A 9-mer with many TCRs outweighs a rare 11-mer.

**Fix**

```r
peptide_data |> count(k_peptide, sort = TRUE)
peptide_data |> count(k_CDR3b, sort = TRUE)
```

Put that table in the report and cite it. For a peptide logo, also check the mode inside the allele filter before locking every allele to 9. For the one CDR3 logo, state the T24 rule explicitly: length 15, A*02:01, `LLFLVLIML`.

**Analysis sentence:** “Sequence logos require one length. Peptide logos use 9-mers, the modal peptide length in the tidy table (see count). The CDR3β logo uses length 15, the length specified for the A*02:01 / LLFLVLIML example. Other lengths are not shown.”

### 7. `distinct()` changes what the logo means

| Call | What it does to the claim |
| --- | --- |
| `distinct()` after unnesting peptides | One row per CDR3β, V, J, experiment, and peptide. Duplicate peptide strings in one `Amino Acids` cell collapse. Length bars count these rows. |
| `distinct()` after the join | Drops exact copies (homozygous allele stored twice). Different alleles remain. |
| `distinct(peptide)` before `ggseqlogo` | One vote per peptide string. Matches T22 (“unique” 9-mers). A peptide from one subject counts the same as a peptide from fifty. Bits describe that set of strings. |
| `distinct(CDR3b)` before the TCR logo | Drops V gene, J gene, and experiment. The same CDR3 amino acids with two TRBV genes become one sequence. Public clones shared by many subjects count once. V-gene bias is invisible. |

`n_peptides` is computed and then dropped, and `separate()` is hardcoded to 13 columns. Lab Q4 is “what is the maximum?”, and the worked example uses 13 because that maximum is 13. Compute it and set the width from the result so extra peptides are not dropped by `separate()`.

```r
max_n <- max(peptide_data$n_peptides, na.rm = TRUE)
peptide_names <- str_c("peptide_", seq_len(max_n))
```

**Methods sentence:** “Peptide logos are unweighted unique 9-mer strings. The CDR3β logo is unweighted unique CDR3β amino-acid strings of length 15. V and J genes are parsed and then not used in the logo.”

### 8. Data paths will not render from this repo

```r
read_csv(here("data", "peptide-detail-ci.csv.gz"))
read_csv(here("data", "subject-metadata.csv"), na = "N/A")
```

`README.md` says raw files live in `data/_raw/` as `peptide-detail-ci.csv` and `subject-metadata.csv`, from ImmuneCODE-MIRA-Release002.1 only, and `data/` is gitignored. The `.csv.gz` path matches the Cloud exercise (gzip the peptide file, then read it). It does not match a fresh clone of this project. Metadata is a different folder and a different extension from the peptide file.

`na = "N/A"` replaces readr’s default `c("", "NA")`. Age/Gender `"N/A"` becomes missing, which is what the lab hint asks. Empty HLA cells stay as `""` only because `""` is no longer an NA string. Set the NA vector explicitly.

**Fix**

```r
peptide_data <- read_csv(here("data", "_raw", "peptide-detail-ci.csv"))
meta_data <- read_csv(
  here("data", "_raw", "subject-metadata.csv"),
  na = c("", "NA", "N/A")
)
```

`read_csv` also reads `.gz` if that is the only copy. Open `lab05_handin.Rproj` before render so `here()` finds the project root.

### 9. Hand-in file and embed-resources

Her YAML already has `embed-resources: true` and the title group number is 21. That part is right for a zipped HTML.

Still not Learn-ready:

- `doc/lab05_handin.qmd` is the empty template. Her analysis is outside the file this repo says to render and zip.
- The MHC figure is a remote URL (`r4bds.github.io/...png`). A self-contained HTML still breaks offline if that URL is not downloaded into the project. Use a file under `doc/` or drop the image and keep the McCarthy and Weinberg 2015 citation.
- No chunk labels, no figure captions, no n in the caption, plots not given axis titles (`k_peptide` is not a caption).
- `slice_sample()` with no `set.seed()`, so the data-description tables change every render.
- `library(ggseqlogo)` is loaded twice. Harmless; load it once at the top.
- Background is pasted from the lab, including “Histocompatilibty”, “pathology of vira”, and “viral proteins, is”. Fine for a draft; the haplotype sentence is the one that conflicts with the results (flag 4).

### 10. Authors and student ids are missing

Her YAML has a title and no `author`. `doc/lab05_handin.qmd` still says `Group XX` and `Add names and student ids`. `README.md` still has the placeholder contributor line. Learn needs names and student ids in the header of the rendered HTML. This review does not invent them.

```yaml
---
title: "Lab 5 Assignment: Group 21"
author:
  - "Full name (sXXXXXX)"
  - "Full name (sXXXXXX)"
format:
  html:
    embed-resources: true
editor: visual
---
```

---

## Do the logos support her conclusion sentences?

No counts or bit scores were computed here. “Supports” means the sentence is something that plot is able to show. Several sentences claim more than an allele-filtered `ggseqlogo` can show.

### Peptides

> “The peptides binding to HLA class I molecules are characterized by two main positions visible in the two plots above. It is clear that position 2 and position 9 show the highest bits.”

- “Binding to HLA class I” is not what the join measures (flags 2 and 4).
- Two plots (A*02:01 and B*07:02, 9-mers, unique peptides) cannot characterise HLA class I as a class. HLA-C is absent from the peptide logos.
- “Position 2 and position 9 show the highest bits” can stay only as a per-plot description, after the group looks at the rendered figure, with n in the caption. This review does not confirm it.

> “Sequence logos built for individual alleles show that position 2 is dominated by leucine, while position 9 is predominated by leucine, valine, or tyrosine; the remaining positions, however, are way more variable.”

This sentence pools both alleles and names residues. T23 asks for a second allele so the motif is seen to be allele-specific. One shared residue list erases that. Write one sentence per logo, and name a residue only when that stack is the one you see in that plot. Do not add tyrosine at P9 unless that logo shows it. Textbook anchors differ (A*02:01 is typically P2 Leu/Met and PΩ Val/Leu; B*07:02 is typically P2 Pro and a hydrophobic PΩ). Use that only as a sanity check when reading the figure, not as a result.

### TCRs

> “T-cell Receptors that bind a specific pMHC.complex show high conservation around the first 3 positions, as well as in the last position, which tend to be structurally fixed zones of the TCR shared across most CDR3b sequences.”

- “Bind a specific pMHC” overstates haplotype membership (flags 2, 4, 5).
- Adaptive CDR3β strings include the conserved Cys at the start and Phe (sometimes Trp) at the end. Positions 2–3 are often germline-encoded from the V gene (`CASS...`). High bits at the ends are expected for almost any length-matched CDR3β logo, including TCRs filtered to a different peptide. The figure can support “the ends of these length-15 CDR3β strings are conserved”. It cannot support “this conservation is why they recognise LLFLVLIML–A*02:01”. A same-length control logo was not made.
- The structural-anchor wording is a general CDR3 fact. Expert A should own that sentence. “As we can see” should not present it as a finding from these two filters.

> “However, as we can see, the middle regions seem to be far more variable, which determines which particular peptide-HLA combination a T-cell Receptor can recognize.”

A variable middle in one logo supports a diverse CDR3β set for that filter. It does not show that those positions determine which pMHC is recognised. That would need a comparison across peptides, or a structural argument. The second logo is a different length (13 vs 15), so “the middle” is not the same coordinates. Drop logo 2, and replace this sentence.

**Replacement conclusion (only if the rendered T22/T23/T24 figures actually look like this; otherwise describe what is on the figure):**

Peptide logos are unique 9-mers from subjects who carry the named allele. Describe A*02:01 and B*07:02 separately. State the haplotype limit in the same paragraph.

The CDR3β logo is unique length-15 CDR3β strings found with peptide `LLFLVLIML` in A*02:01-carrying experiments. Conserved ends match the CDR3β anchors shared by most CDR3β sequences. The middle is described only as more variable in this set. Specificity is not assigned to those positions from this single logo.

---

## Paste-ready code skeleton

Wire this into `doc/lab05_handin.qmd` after names and ids are real. Do not treat this as a result.

```r
library(tidyverse)
library(here)
library(ggseqlogo)

peptide_data <- read_csv(here("data", "_raw", "peptide-detail-ci.csv"))
meta_data <- read_csv(
  here("data", "_raw", "subject-metadata.csv"),
  na = c("", "NA", "N/A")
)

meta_data <- meta_data |>
  select(-starts_with("D")) |>
  rename(
    A1 = `HLA-A...9`, A2 = `HLA-A...10`,
    B1 = `HLA-B...11`, B2 = `HLA-B...12`,
    C1 = `HLA-C...13`, C2 = `HLA-C...14`
  ) |>
  pivot_longer(c(A1, A2, B1, B2, C1, C2),
               names_to = "Gene", values_to = "Allele") |>
  mutate(Allele = str_squish(Allele)) |>
  filter(!is.na(Allele), Allele != "") |>
  mutate(Allele_F_1_2 = str_extract(Allele, "^[ABC]\\*\\d{2,3}:\\d{2,3}")) |>
  filter(!is.na(Allele_F_1_2)) |>
  mutate(Allele = str_remove(Allele_F_1_2, "\\*")) |>
  distinct(Experiment, Allele)

peptide_data <- peptide_data |>
  select(`TCR BioIdentity`, Experiment, `Amino Acids`) |>
  separate(
    `TCR BioIdentity`,
    into = c("CDR3b", "V_gene", "J_gene"),
    sep = "\\+"
  ) |>
  mutate(n_peptides = str_count(`Amino Acids`, ",") + 1)

# Set peptide_names from max(n_peptides) before separate().
peptide_data <- peptide_data |>
  separate(`Amino Acids`, into = peptide_names, sep = ",", fill = "right") |>
  pivot_longer(starts_with("peptide_"), names_to = "peptide_n", values_to = "peptide") |>
  select(-n_peptides, -peptide_n) |>
  drop_na(peptide) |>
  distinct() |>
  filter(
    str_detect(CDR3b, "[^ARNDCQEGHILKMFPSTWYV]", negate = TRUE),
    str_detect(peptide, "[^ARNDCQEGHILKMFPSTWYV]", negate = TRUE)
  ) |>
  mutate(k_CDR3b = str_length(CDR3b), k_peptide = str_length(peptide))

peptide_meta_data <- peptide_data |>
  inner_join(meta_data, by = "Experiment", relationship = "many-to-many") |>
  distinct()

# Peptide logos: unique 9-mers among carriers of that allele.
# CDR3 logo: unique length-15 CDR3s for LLFLVLIML in A*02:01 carriers.
# Record n_distinct() in each figure caption.
# Do not add the C*04:01 / HTTDPSFLGRY logo to the conclusion.
```

---

## Discussion points for Expert A (mol bio)

1. MIRA (Nolan et al. 2020, ImmuneCODE release 002.1) links a CDR3β to a peptide in a subject. The subject’s classical class I haplotype is metadata. The restricting allele is not a column. Confirm that “peptides binding to HLA-X” must be rewritten as “peptides observed in people who carry HLA-X”.
2. Up to six classical class I alleles (A, B, C on two haplotypes). Homozygosity means fewer. The background’s “each of us have a total of 6” and “3 classes” wording needs a correction if it stays.
3. Sanity-check the two peptide logos separately after they are rendered: A*02:01 anchors are expected at P2 (Leu/Met) and the C-terminus (Val/Leu); B*07:02 is expected at P2 (Pro) and a hydrophobic C-terminus. If both plots are described as P2 leucine and P9 L/V/Y, the sentence is pooling alleles. Tyrosine at P9 should be named only if that plot shows it.
4. CDR3β logos: position 1 Cys and the final Phe/Trp are the conserved CDR3 boundaries (germline V/J), not evidence of pMHC specificity. A variable middle means diverse CDR3s in that filter, not a demonstration that the middle “determines” the peptide–HLA pair. Please supply the sentence you want in the report.
5. `HTTDPSFLGRY` (11-mer) is treated in published MIRA work as an HLA-A*01-linked ORF1ab peptide, not as a C*04:01 ligand. Please confirm whether any C*04:01 sentence can remain. Expert B’s recommendation is to drop that logo.
6. `LLFLVLIML` + A*02:01 + CDR3 length 15 is the course’s T24 example. Please say whether Expert A is willing to call it an A*02:01 ligand, or only “the peptide the lab asked us to plot”. This review did not establish a published restriction for that peptide.
7. Copied background errors that are yours if the paragraph stays: proteasome → peptide → TAP → ER peptide-loading complex → Golgi → surface; “chaperones bound to MHC-I and then across the Golgi” is muddled. CD8+ recognition of a viral pMHC can lead to killing of the infected cell; keep that short.

---

## Joint revision checklist (Learn-ready micro-report)

Do not submit the current draft. Port the revised report into `doc/lab05_handin.qmd`, render, zip the HTML, one course only.

- [ ] Real names and student ids in the YAML `author` field and in `README.md`. Title group number 21.
- [ ] `embed-resources: true` kept. Remote MHC image replaced with a local file or removed. Citation kept.
- [ ] Data read from `data/_raw/` (Release 002.1). `na = c("", "NA", "N/A")`. Project rendered with `lab05_handin.Rproj` open.
- [ ] Class II columns dropped. Alleles parsed to two-field form. Blanks removed. `A02:01` ≠ `A02:10` checked by a count, not assumed.
- [ ] Amino-acid filter assigned back to `peptide_data`.
- [ ] `max(n_peptides)` computed and used as the `separate()` width.
- [ ] Join on `Experiment` kept as many-to-many, with a methods sentence that this is haplotype membership. “We fixed the warning” deleted.
- [ ] `count(k_peptide)` and `count(k_CDR3b)` printed and cited. Peptide logos at the modal length (lab uses 9). One CDR3 logo: length 15, `A02:01`, `LLFLVLIML`.
- [ ] `C04:01` / `HTTDPSFLGRY` / length 13 logo removed from the hand-in unless Expert A signs a separate, caveated paragraph. It does not support the shared conclusion.
- [ ] Captions state filters and `n` unique sequences. Axes say “peptide length” and “CDR3β length”, not only `k_peptide`.
- [ ] `distinct(peptide)` and `distinct(CDR3b)` explained as unweighted unique strings. V/J not interpreted.
- [ ] Conclusions rewritten per logo. No class-wide HLA motif. No “middle determines specificity” from one logo. Haplotype caveat in the same section.
- [ ] `set.seed()` if any `slice_sample()` stays; otherwise drop random previews.
- [ ] Expert A signs the biology sentences in the Discussion list. Artist signs figures (captions, labels, readable logo panels, no remote image).
- [ ] Render `doc/lab05_handin.qmd` to HTML and zip that file. Nobody uploads this review, and nobody uploads an unrendered `.qmd`, to Learn.
