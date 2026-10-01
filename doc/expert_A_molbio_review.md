# Expert A review — molecular biology (antigen presentation, HLA class I, TCR/CDR3)

Reviewer role: molecular biologist. This is a pre-hand-in science review of the Group 21 draft in the uploaded `her_group21_lab05.qmd` (title “Lab 5 Assignment: Group 21”). It is not a DTU Learn submission, and it does not reanalyze the ImmuneCODE tables. No sequence counts, bit scores, or logo letters are invented below. Where a residue is named, it is either (i) a letter she should copy from her own logo only if that letter is actually tall, or (ii) a published, allele-typical anchor offered as a check against the figure.

Course questions this report is allowed to answer:

1. What characterises peptides binding to HLAs?
2. What characterises TCRs binding to peptide–HLA complexes?

Method in the draft: ImmuneCODE MIRA class I (`peptide-detail-ci`) joined to subject HLA metadata, then `ggseqlogo`.

## Verdict

The analysis skeleton is what this lab asks for: tidy the two tables, join on `Experiment`, and show sequence logos for common peptide and CDR3β lengths. That part is course-typical and should stay.

The prose is not safe to hand in as written. Several sentences state a general biological rule that two logos cannot support, and a few of them are wrong even as textbook biology. The risky ones are:

- One shared motif sentence for HLA-A\*02:01 and HLA-B\*07:02 (“position 2 is leucine; position 9 is leucine, valine, or tyrosine”).
- Treating every allele in a subject’s type as the allele that presented the peptide.
- Reading conserved CDR3β ends as the part that recognizes peptide–HLA, and the variable middle as a demonstrated specificity motif.
- The comment that `relationship = "manu-to-many"` fixed the join warning.
- A background paragraph she would struggle to defend if asked (haplotype wording, “3 classes”, TAP/chaperone sentence, “the CTL” killing any cell that shows a viral peptide).

The background is copied from the lab page, which explicitly says it may be pasted. Copying it is course-typical. The lab page also contains the typos and the haplotype/pathway problems. Pasting does not make those sentences true. For a hand-in, replace that paragraph with the short version in [Replacement prose](#replacement-prose). The lab’s own disclaimer (end of the CDR3 section) already says the allele that presented the peptide is missing from the table. The report should say that in one or two sentences and then describe the logos more narrowly.

## Typos and small language errors

Fix these even if the surrounding sentence is rewritten. Several are in the lab-page paragraph; they still should not reach a reader.

| Location | As written | Correction |
| --- | --- | --- |
| Background | Major Histocompatilibty Complex | Major Histocompatibility Complex |
| Background | pathology of vira | replication of viruses, if a virus sentence is kept at all |
| Background | viral proteins, is broken down | viral proteins are broken down |
| Background | Each of us have | Each of us has |
| Background | the virus first confirmed case | the first confirmed case |
| Data description | peptide_dara | peptide_data |
| Join comment | manu-to-many | many-to-many |
| TCR paragraph | pMHC.complex | pMHC complex (or peptide–HLA complex) |
| Peptide paragraph | highest bits . | highest information content. |

Also in that peptide paragraph: “predominated” is understandable, but the sentence merges two alleles and should be replaced, not just polished. “early 20s” is vague; the year 2020 is enough if the pandemic is mentioned at all.

`relationship = "many-to-many"` does not repair anything. In current dplyr it tells the join that a many-to-many match is intentional, so the warning stops. The row multiplication remains. The comment should not say “fixed”.

## What is fine and course-typical

Keep these. They match the lab tasks and they are reasonable biology if the wording stays modest.

- Two aims, ImmuneCODE MIRA class I plus subject metadata, logos rather than a long exercise dump. The assignment asked for a short micro-report.
- Splitting `TCR BioIdentity` on `+` into CDR3β, V gene, and J gene. The sequences in the lab examples start with C and end with F, which is the usual immunoSEQ CDR3β string (anchors included).
- Expanding the comma-separated peptide list to one peptide per row, then dropping empty peptide cells.
- Restricting the HLA table to A, B, and C. The peptide file is the class I MIRA panel (CD8). Class II genes in this metadata begin with D (DP, DQ, DR). Say that in one clause so the `select(-starts_with("D"))` step has a reason.
- Cutting allele names to the first two fields (`A*02:01:01` to `A*02:01`). That is what the lab asks, and two-field resolution is the protein type used for classical motifs. In the prose, write HLA-A\*02:01, not `A02:01`. Dropping the asterisk is a coding convenience from the lab (T20), not a change of meaning.
- Bar plots of length, then logos at the modal length. Nine-residue peptides are the usual class I length, so a mode at 9 would be expected. State it as the mode of these histograms. Other lengths in the same table also come from the class I panel.
- `distinct(peptide)` before the peptide logo, and `distinct(CDR3b)` before the TCR logo. The logo then shows sequence variety, unweighted by how many rows carried the same string.
- Building separate logos for HLA-A\*02:01 and HLA-B\*07:02, and separate CDR3β logos for two peptide contexts (lab tasks T22–T25).
- The positional peptide claim, once it is limited to these plots: class I 9-mer logos often have their highest conservation at position 2 and position 9. That is the right kind of answer to question 1. The residue identities have to be read per allele (next section).
- Citing Nolan et al. 2020 for the database, and keeping the course figure if a pathway picture is wanted. McCarthy and Weinberg 2015, which the lab cites, is about the immunoproteasome and viral infection. It can stay as the figure credit the lab uses. It should not be asked to support the TAP, Golgi, or CTL sentences.

## COVID background: accuracy versus need

The first confirmed SARS-CoV-2 case reported in Denmark was 27 February 2020. That date is fine. SARS-CoV-2 as the virus and COVID-19 as the disease are fine.

None of that answers either course question. The Danish date, the sentence about packing components into a viral envelope, and the general “pathology of viruses” opening are extra surface area. Someone with a light biology background will be examined on whatever they leave in the Background.

What the aims need is one sentence on the data: these tables are TCRβ sequences linked to SARS-CoV-2 peptides from MIRA experiments (Nolan et al. 2020), plus the HLA types of the subjects in those experiments. A correct four-sentence sketch of class I presentation is optional, because the lab figure is already about that pathway. If it stays, use the replacement below. If time is short, cut the pathway and keep the data sentence plus the haplotype caveat in the Analysis.

## Haplotype versus allele — do not skip this

This is the main interpretation limit of the whole report. The lab states it as a disclaimer and leaves the fix out of scope. The hand-in still needs the limit in plain language so the logos are not described as proven binding motifs.

**Allele.** One version of one HLA gene. HLA-A\*02:01 is an allele. HLA-B\*07:02 is a different allele of a different gene.

**Locus.** HLA-A, HLA-B, and HLA-C are three classical class I genes (loci). They are not three classes. Class I and class II are the classes. Class II in this metadata is the D-named genes she removed.

**Haplotype.** The alleles inherited together on one chromosome from one parent. For classical class I that is one HLA-A, one HLA-B, and one HLA-C allele (class II genes sit on the same haplotype, but they are not used here).

**A person’s type.** Two haplotypes, one maternal and one paternal. That is a diploid genotype: up to two alleles at HLA-A, two at HLA-B, and two at HLA-C. “Six HLA alleles” is only true for classical class I, and only when all three loci are heterozygous. A homozygous locus contributes one distinct protein, not two. This count leaves out non-classical class I (E, F, G) and all of class II. HLA genes are among the most polymorphic in the human population. One person carries only their own few alleles; they are not themselves “the most diverse genes.”

**What the join actually is.** Each MIRA row says that a CDR3β was associated with a peptide in an experiment, and that the subject in that experiment has a list of alleles. It does not say which allele held the peptide. A peptide on the cell surface sits in one allele’s groove. The same cell also presents other peptides on its other class I alleles. Joining on `Experiment` pastes every peptide onto every allele of that subject. Filtering `Allele == "A02:01"` keeps peptides from people who carry HLA-A\*02:01, including peptides those people presented on HLA-B or HLA-C.

So the logo is: unique 9-mers from experiments in which the subject carries this allele. It is not, by itself, the ligand set of that allele. Write that next to both peptide logos. It also applies to the CDR3 logos, which use the same join.

Published motifs can be used only as a comparison. If the tall letters match the known anchors of that allele, the logo is *consistent with* presentation by that allele. If they do not, report the letters that are actually tall and keep the caveat. Do not edit the sentence until it matches the textbook.

## TAP / proteasome paragraph

The current sentence mixes the order of the pathway and the grammar (“aided by chaperones bound to the MHC-I and then across the Golgi”). A defender of this paragraph would be asked what the chaperones do and which cell kills which target. Safer to replace the whole pathway with the short version, or delete it.

Accurate sketch, at the depth this course needs:

- Class I presentation is mainly of proteins made in the cell (viral proteins, and defective products of translation). The proteasome, including the immunoproteasome in inflamed cells, cuts them into peptides.
- TAP (TAP1/TAP2) moves peptides from the cytosol into the endoplasmic reticulum. Length and the C-terminal residue affect transport. The proteasome product is not always the final ligand: ERAP1 can trim the N-terminus in the ER.
- The class I molecule is the heavy chain plus β2-microglobulin. Loading is helped by the peptide-loading complex (TAP, tapasin, calreticulin, ERp57). Tapasin favors peptides that sit stably in the groove. “Chaperones bound to MHC” is the wrong picture of that step.
- A stable peptide–MHC complex leaves the ER and travels through the Golgi to the cell surface.
- A CD8+ T cell recognizes one specific peptide–MHC complex through its TCR. An effector cytotoxic T cell can then kill that cell, typically by perforin and granzymes or by Fas ligand. Display of a viral peptide does not activate “the CTL” in general. Most circulating CD8+ T cells have other specificities. Priming of a naive T cell is a separate step and usually involves a professional antigen-presenting cell and co-stimulation, not any infected cell.

None of those extra names (ERAP1, tapasin, Fas) are required in the report. They are here so the old sentence is not replaced by a new overclaim. The replacement prose uses the shorter form.

## Peptide logos: position 2 and position 9

### What a logo can support

`ggseqlogo` on equal-length strings stacks amino acids at each position. With the default bits scale, a tall stack means that position is conserved in the set that was passed in, and the tall letters are the enriched residues. A short stack means the position is variable in that set.

For these 9-mers the logo can support statements of the form: “In the unique 9-mers from A\*02:01-positive experiments, position 2 is enriched for … and position 9 is enriched for …, relative to the other positions in the same logo.” It cannot support affinity, a universal HLA rule, or a claim that those peptides were shown to bind that allele.

Class I peptides are usually 8–11 residues (sometimes longer, with the middle bulging out of the groove). The primary anchors are position 2 (B pocket) and the C-terminal residue (F pocket). For a 9-mer the C-terminus is position 9. For the 11-mer used later (`HTTDPSFLGRY`), the C-terminal anchor would be position 11, not position 9. The middle of the peptide is more available to the TCR and is usually more variable in a ligand logo. Secondary anchors exist for some alleles; mention one only if that position is clearly taller than its neighbors in that specific logo.

Position 2 and position 9 being the most conserved columns is a fair reading if that is what both figures show. Name the amino acids separately for each allele, and only name a residue whose letter is tall in that figure.

### The sentence that should not be handed in

> “The peptides binding to HLA class I molecules are characterized by two main positions visible in the two plots above. It is clear that position 2 and position 9 show the highest bits. Sequence logos built for individual alleles show that position 2 is dominated by leucine, while position 9 is predominated by leucine, valine, or tyrosine; the remaining positions, however, are way more variable.”

Problems:

- “HLA class I molecules” and “individual alleles” are written as one motif. A\*02:01 and B\*07:02 do not share their position-2 anchor.
- Published ligand motifs, as a check against the figures and not as a result to paste if the figure disagrees: HLA-A\*02:01 9-mers are typically enriched for a hydrophobic residue at position 2 (often leucine or methionine) and a hydrophobic C-terminus (often valine or leucine). HLA-B\*07:02 9-mers are typically enriched for proline at position 2, and a hydrophobic C-terminus (often leucine or phenylalanine). Proline, not leucine, is the B\*07:02 position-2 anchor. Tyrosine is not the usual C-terminal anchor of either allele. A C-terminal tyrosine preference is typical of other alleles (for example HLA-A\*01:01). If a Y appears in one of these two logos, it may be real enrichment in this filtered set, including peptides from other alleles in the same subjects. Describe the letter if it is tall, and do not promote it to “the” position-9 motif of both alleles.
- “It is clear” and “determines binding” overclaim a T-cell-association table. The set is also shaped by which SARS-CoV-2 peptides were in the MIRA pools, by the amino-acid composition of those viral proteins, and by which responses were large enough to be called. Leucine is a common residue in proteins, so a leucine-rich column is not automatically an anchor. The anchor interpretation is the comparison of positions inside one logo (position 2 and the C-terminus versus the middle), plus agreement with the known motif of that allele.
- “The remaining positions are way more variable” is all right only as a description of these two plots. If a third position is visibly conserved, say so for that allele only.

Suggested replacement for this paragraph is in [Replacement prose](#replacement-prose). After looking at the two figures, fill the blanks with the tall letters. If B\*07:02 position 2 is proline in the figure, say proline. If it is not, do not write proline just because the textbook expects it.

## CDR3β logos: V and J ends versus the junction

### What the two TCR logos can support

She plots CDR3β sequences of one length, linked to one peptide, from subjects who carry one named allele (`LLFLVLIML` with length-15 CDR3β and A\*02:01; `HTTDPSFLGRY` with length-13 CDR3β and C\*04:01). Each logo describes that filtered set only. The second peptide is 11 residues long, so it is outside the “we use length 9 below” sentence. Either drop that sentence’s implication that everything downstream is a 9-mer, or add a clause that the second TCR example is an 11-mer.

A logo can support: which CDR3β positions look conserved, and which letters are tall, among the distinct sequences that passed the filter. It cannot support a general rule for “T-cell receptors,” a paired αβ motif, or a measured binding affinity. MIRA records antigen-driven enrichment of TCRβ clonotypes. The course question says “binding”; in the results, “associated with” or “reported for” is the accurate verb. One clause is enough.

### Ends versus junction

The sentence to replace:

> “T-cell Receptors that bind a specific pMHC.complex show high conservation around the first 3 positions, as well as in the last position, which tend to be structurally fixed zones of the TCR shared across most CDR3b sequences. However, as we can see, the middle regions seem to be far more variable, which determines which particular peptide-HLA combination a T-cell Receptor can recognize.”

The observation (ends more conserved, middle more variable) is a common look for this kind of logo. The mechanism in that sentence is not.

CDR3β, as stored here, runs from the conserved V-gene cysteine to the conserved J-gene phenylalanine (the IMGT CDR3 bounds). Those two residues are conserved because they define the CDR3, not because they were selected on this peptide.

- The N-terminal residues after the cysteine are largely germline V-gene sequence. Across many TRBV genes those positions are often similar (a logo commonly shows C, then A/S). That conservation remains if the logo mixes V genes that all happen to encode those residues.
- The C-terminal residues are largely germline J-gene sequence. The final F is shared by essentially all TRBJ genes, which is why “the last position” looks invariant. The two or three residues before it depend on which J gene was used (`YEQYF`, `NEQFF`, `TEAFF`, and so on). A mix of J genes conserves the final F and only partly conserves the positions just before it.
- The junction is the middle: the somatically diversified V–D–J join, including nucleotides added by terminal deoxynucleotidyl transferase and bases chewed back from the gene ends. That is where CDR3β sequences differ between clones.

“Structurally fixed zones of the TCR” mixes this up with framework regions. The ends are still part of CDR3. They look fixed in a logo because they are germline-encoded and, at the very ends, definitionally conserved.

A variable middle does not show that those positions “determine” the peptide–HLA specificity. It shows that many different junctions were linked to the same peptide in this table. That is the usual private CD8 response: different CDR3β sequences can be associated with one peptide, and this logo will not collapse to one central motif. Specificity of a T cell is the whole receptor (TCRα and TCRβ, including which V and J genes were chosen). This file has β only. V and J are parsed in the code and then unused; the logo therefore mixes different V and J genes, which is exactly why the germline ends look like a shared motif.

If, for one peptide, the center of the logo is obviously conserved as well as the ends, then say that this particular set shares junction residues, and name those letters from the figure. Do not write that sentence unless the figure shows it. From the wording “far more variable,” the current draft is describing diversity, so the conclusion has to be diversity, not a discovered recognition motif.

Same haplotype limit as for the peptides: these CDR3β sequences were linked to the peptide in subjects who carry the named allele. The presenting allele is not in the row.

## Other claims that are easy to over-read

- The standard-amino-acid `filter()` calls are printed and not assigned back to `peptide_data`. The report therefore does not show that the logos used only the 20 standard residues. Either assign the filter, or say that the check was inspected and what it showed. Do not write that non-standard characters were removed.
- `slice_sample()` in a rendered report changes every knit. It is fine during lab exploration. For the hand-in, a deterministic `slice_head()` or a seeded sample is easier to defend. That is reproducibility, not biology.
- Empty HLA fields: the lab text says to drop `NA` and `""` before using alleles. The draft’s allele cleanup does not show that drop. It does not change the A\*02:01 logo if the filter is exact, but it does belong in the caveat that the HLA table is incomplete for some experiments.
- “Most frequent, therefore used for the logo” is a design choice, not a biological law. Say that longer and shorter peptides were set aside so that one column is one position.

## Replacement prose

The following is written so it can be pasted over the corresponding sections. Blanks marked `[tall letters in that logo]` must be filled from the figures. Leave a residue out if it is not clearly enriched. Do not copy the textbook anchors into the blanks unless the logo shows them.

### Background (short; replaces the pasted lab paragraph)

Class I HLA molecules display short peptides on the cell surface for CD8+ T cells. In this report the peptides and TCRβ sequences come from the ImmuneCODE MIRA class I experiments (Nolan et al. 2020): a CDR3β sequence was linked to a SARS-CoV-2 peptide in a given subject, and that subject’s HLA type was recorded separately.

A classical class I haplotype is the HLA-A, HLA-B, and HLA-C allele inherited on one chromosome. Each person has two haplotypes. The MIRA tables do not record which of those alleles presented a given peptide. A logo made by joining peptides to every allele of the subject is therefore the set of sequences from people who carry that allele, not a proven ligand set of that allele.

### Peptide result (replaces the paragraph under the two peptide logos)

The logos below use unique 9-mers from experiments in which the subject carries the named allele. Nine residues is the most common peptide length in this dataset, so each column is the same peptide position. Other lengths were left out. These are MIRA associations, not a direct binding measurement.

In the HLA-A\*02:01 logo, the most conserved positions are 2 and 9. Position 2 is enriched for [tall letters in that logo]. Position 9 is enriched for [tall letters in that logo]. The positions in between are more variable in this logo. That pattern is what would be expected if many of these 9-mers used the usual A\*02:01 anchors (a hydrophobic residue at position 2 and at the C-terminus), but peptides presented by other alleles in the same subjects are still in the set.

In the HLA-B\*07:02 logo, describe positions 2 and 9 separately. Position 2 is enriched for [tall letters in that logo]. Position 9 is enriched for [tall letters in that logo]. The published B\*07:02 anchor at position 2 is proline, not leucine. Use that only as a comparison with this figure.

Conservation at position 2 and at the C-terminus fits the closed class I groove: those side chains point into the B and F pockets. The more variable middle is the part of a 9-mer that is more available to the TCR. The logos show positional conservation in these filtered sets. They do not measure which pocket a given peptide used.

### TCR result (replaces the paragraph under the two CDR3 logos)

The first logo is distinct CDR3β sequences of length 15 linked to the peptide LLFLVLIML in subjects carrying HLA-A\*02:01. The second is distinct CDR3β sequences of length 13 linked to the 11-mer HTTDPSFLGRY in subjects carrying HLA-C\*04:01. The same allele caveat applies: the subject carries the allele, and the table does not prove that this allele presented the peptide. Only the β chain is present.

In these strings the CDR3β runs from the V-gene cysteine to the J-gene phenylalanine. Conservation at the start of the logo is the cysteine and the following germline V-gene residues. Conservation at the last position is the phenylalanine shared by J genes. Those ends are expected to look similar across many CDR3β sequences even when the receptors do not share a peptide-specific motif. The middle is the V–D–J junction, which is different in different clones. Where that middle is variable, these TCRs do not share one CDR3β motif for the peptide. The logo does not show the TCRα chain, and it does not show which CDR3 residue contacts the peptide.

## Discussion points for Expert B (bioinformatics)

- Confirm the many-to-many join: how many distinct alleles are attached to each `Experiment`, and by what factor peptide and CDR3 rows are multiplied. Silencing the warning did not change that.
- Confirm empty HLA strings and `NA` alleles are still in `meta_data` after the `str_sub` / `str_remove` steps, and whether any experiment has no usable class I type.
- Check that `str_sub(Allele, 1, 7)` yields a real two-field allele for every non-empty value in this file (two-field names such as `A*02:01` and three-field names such as `A*02:01:01`). Flag any string it corrupts.
- The standard-amino-acid filters are not assigned. Count CDR3β and peptide strings that contain a character outside `ARNDCQEGHILKMFPSTWYV`, and whether `ggseqlogo` ever saw them.
- For each logo, report `n`: unique 9-mers for A\*02:01, unique 9-mers for B\*07:02, unique length-15 CDR3β for `LLFLVLIML` + A\*02:01, unique length-13 CDR3β for `HTTDPSFLGRY` + C\*04:01. A motif sentence needs a large enough `n` to be worth making; the biology review did not count these.
- Verify the length histograms actually peak at 9 (peptides) and 15 (CDR3β) before that sentence stays. The second TCR logo is length 13 and the peptide is length 11, so the “used below” sentence does not cover every figure.
- Say whether logos are in bits or probability (`ggseqlogo` default is bits) and that every string in a logo has the same length.
- `distinct(peptide)` and `distinct(CDR3b)` mean the logos are unweighted. Note that a peptide recognized by many TCRs counts once. That is a fair choice for “what do the sequences look like,” and it is different from a frequency-weighted logo.
- The V and J columns are parsed and never used. A useful check, still within the course tools: among CDR3β sequences in one logo, how many distinct V genes and J genes are there? A mixture explains conserved ends without any extra model.
- Optional, only if the logos look leucine-heavy everywhere: a background of amino-acid frequencies from the MIRA peptides themselves (or a column-shuffled logo) would show whether position 2 stands out above the viral peptide composition. Do not add this unless the group has time. The course does not require it, and it is not a substitute for the allele-assignment caveat.
