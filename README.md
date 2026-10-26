# Paper C: material for review

This tree holds the code, tables, results and verification records for the paper *Do the Examples Cover the Law? Testing Example-Driven Completion on Encoded Legislation*, arranged for anonymous review.

The study tests whether LATC, an algorithm that completes a decision table from example cases, can serve as the completion step of a Rules as Code model. It converts fourteen rules into finite decision tables and evaluates each table in full through its encoding. The rules come from two public encodings of legislation:

- [Catala](https://github.com/CatalaLang/catala-examples): French family benefits, French housing benefits, and section 121 of the US Internal Revenue Code;
- [OpenFisca Aotearoa](https://github.com/digitalaotearoa/openfisca-aotearoa): New Zealand social-security eligibility.

LATC then completes each table from the encodings' official test cases and from random cases. A protocol was registered before any result was computed.

## Layout: one folder per stage

The work was done on 18 git branches, one per stage. Here each branch is a top-level folder of the same name, and each folder holds that branch's `paperC/` folder, copied with `git archive` from the head commit listed below. Each branch also carries, outside `paperC/`, the unchanged files of the `latc_check` package it was cut from (commit `c6d2802`); those files are left out here, because they hold no Paper C material.

| Folder | Source branch head | Content |
|---|---|---|
| `paperc-harness/` | `89795da` | Protocol (`paperC/PROTOCOL.md`, committed at `ee7c527`), the experiment harness and the CTRL-NUR control |
| `paperc-catala-family-benefits/` | `2046eb8` | Catala build of the French family-benefit encoding |
| `paperc-data-fr/` | `0aee62d` | Tables FR-PF and FR-AF-1 to FR-AF-3 |
| `paperc-data-fr2/` | `6566904` | Tables FR-APL-1 to FR-APL-5, US-121-1 and US-121-2 |
| `paperc-data-fr2-fix/` | `1b9debe` | Corrected US-121-2 and its checks |
| `paperc-data-nz/` | `7efbb1e` | Tables NZ-SLP, NZ-AS and NZ-SPS (and the supplementary NZ-AS-1964) |
| `paperc-data-nz-fix/` | `acd42a0` | Corrected NZ-SLP and its checks |
| `paperc-run-fr/` | `7099c6b` | Harness runs and `REPORT-FR.md` |
| `paperc-run-fr2/` | `eb508e7` | Harness runs and `REPORT-FR2.md` |
| `paperc-run-nz/` | `b835740` | Harness runs and `REPORT-NZ.md` |
| `paperc-verify-fr/` | `83a0d78` | Independent verification, `VERIFY-FR.md` |
| `paperc-verify-fr2/` | `9050c33` | Independent verification, `VERIFY-FR2.md` |
| `paperc-verify-nz/` | `d93c3c0` | Independent verification, `VERIFY-NZ.md` |
| `paperc-code2-fr/` | `c957417` | Blinded second conversion, FR-PF and FR-AF |
| `paperc-code2-fr2/` | `9e448d5` | Blinded second conversion, FR-APL and US-121 |
| `paperc-code2-nz/` | `723d8e9` | Blinded second conversion, NZ |
| `paperc-analysis/` | `b54095b` | The analyses of Sections 4.6 and 4.7 (M2 and M6), `REPORT-M2-M6.md` |
| `paperc-figures/` | `36f687e` | Figures 1–3, the script that draws them, their data and cross-checks |

Each folder documents how to reproduce its stage in the README files under its `paperC/`.

## Large tables

Seventeen `ground_truth.csv` tables are larger than the 8 MB that the review mirror serves, so they are stored gzipped as `ground_truth.csv.gz`. [LARGE-FILES.md](LARGE-FILES.md) lists each one with its size and checksums and shows how to restore it. The `SHA256SUMS` file in each sample folder refers to the uncompressed `ground_truth.csv`, so restore a table before checking it.

## Reproducing

[REPRODUCE.md](REPRODUCE.md) gathers in one guide the software, the commands in order, the expected outputs and the source of every table and figure. It is written for git checkouts of the branches. Where it checks out a branch, `<branch>`, use the folder `<branch>/` of this tree, and run the commands from that folder, so that `paperC` sits directly under the working directory.

Two differences from a branch checkout:

- **`latc_check`.** A branch carries the `latc_check` package at its root, and the harness imports it from there. The folders here hold `paperC/` only, so scripts that import `latc_check` need the package installed separately, at commit `c6d2802abe667a9a49cbcb3848f12407fdf00ef5`, as REPRODUCE.md §1 describes. It needs only the standard library (Python 3.10 or later). Clone it outside this tree and put it on the import path, for example `PYTHONPATH=/path/to/latc-check python3.12 -m paperC.harness.run …` from inside a folder.
- **Git history.** The folders carry no git metadata, in the same way as the per-branch `.tar.gz` archives of REPRODUCE.md §2. They suffice to read the results and to run the oracles, `sha256sum -c`, the harness `run` command, the verifier's `recompute.py` and the figure script. The steps that read git (CTRL-DET, the analysis scripts and the comparisons that read `origin/<branch>`) need the branches themselves.

## Licences

The code is MIT. Material derived from Catala is Apache-2.0, material derived from OpenFisca Aotearoa is AGPL-3.0, and the UCI Nursery control table is CC BY 4.0. [LICENSES.md](LICENSES.md) gives the full map by folder and sample; its paths under `paperC/` apply inside every folder here. The licence texts are in `LICENSE` and `LICENSES/`.
