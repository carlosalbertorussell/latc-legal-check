# Reproducing Paper C

This guide reproduces the results of C. A. Russell, *Do the Examples Cover the Law? Testing
Example-Driven Completion on Encoded Legislation*, from this repository. It lists the
software, how to get the material, the commands in order, what each command writes and
must give, where every table and figure of the paper comes from, and the limits of the
study.

Every command below is copied from the README or report of the branch named beside it.
Where a step has no script, the guide says so. All commands run from the repository root
of a checkout of the branch named in the step, unless a `cd` is shown.

## Contents

1. [Software versions](#1-software-versions)
2. [Getting the material](#2-getting-the-material)
3. [Commands, in order](#3-commands-in-order)
4. [Expected outputs](#4-expected-outputs)
5. [Map of the paper's results](#5-map-of-the-papers-results)
6. [Known limits](#6-known-limits)
7. [Dry-run record](#7-dry-run-record)

## 1. Software versions

| Component | Version | Used by | Recorded in |
|---|---|---|---|
| Python for the harness, the verifiers and the analysis | CPython 3.12.3, standard library only | `paperc-harness`, `paperc-run-*`, `paperc-verify-*`, `paperc-*-fix`, `paperc-analysis` | each `manifest.json` (`python.version`), `paperC/harness/README.md` |
| Python for the Catala oracles | 3.12 or later (tables built with 3.13.16 and rebuilt byte for byte with 3.12.3); 3.11 and older cannot import the Catala 1.3.0 runtime | `paperc-catala-family-benefits`, `paperc-data-fr`, `paperc-data-fr2`, `paperc-code2-fr`, `paperc-code2-fr2` | `paperC/specs/*/ENVIRONMENT.md` |
| Python for the OpenFisca oracles | CPython 3.11.17 in a virtualenv; OpenFisca-Core 41.5.7, numpy 1.24.4, PyYAML 6.0.3; full list in `paperC/openfisca_nz/requirements.lock` | `paperc-data-nz`, `paperc-data-nz-fix`, `paperc-code2-nz` (its own `pip_freeze.txt`), `paperc-verify-nz` (spot check) | `paperC/openfisca_nz/ENVIRONMENT.md` |
| Python for the figures | Python 3.13 with matplotlib 3.9.2 and numpy 2.1.3 (`paperC/figures/requirements.lock`) | `paperc-figures` | `paperC/figures/FIGURES.md` |
| Catala | compiler and `clerk` 1.3.0 (opam package `catala.1.3.0`, git tag `1.3.0` = `9534f81`), OCaml 4.14.2, opam 2.1.5 | building the Python modules and the `clerk test` runs | `paperC/catala_family_benefits/README.md` |
| catala-examples | commit `2694b17ddd7c2246e207e8f077d586349ee5abea` | every FR and US table | same |
| OpenFisca Aotearoa | `openfisca-aotearoa` 22.0.0 at commit `9d77ea7ce3c45b6292d4b4eaf3566b2fb00e7ceb` (AGPL-3.0, cloned at run time and never copied here) | every NZ table | `paperC/openfisca_nz/ENVIRONMENT.md` |
| `latc_check` | commit `c6d2802abe667a9a49cbcb3848f12407fdf00ef5` of https://github.com/carlosalbertorussell/latc-check | the harness (`latc.latc_odometer`, `latc.missing_count`, `reference.exact_complement`, `model.DecisionTable`) | `paperC/harness/README.md` |

**Installing `latc_check`.** Nothing needs installing. Every `paperc-*` branch descends
from `latc-check` commit `c6d2802` and carries the package at the repository root as
`latc_check/`, unchanged (`git diff c6d2802 origin/<branch> -- latc_check` is empty on all
18 branches). The harness imports it from the repository root, so run `python3.12 -m
paperC.harness.run` from there. The package needs only the standard library (Python 3.10
or later). To get it separately:

```sh
git clone https://github.com/carlosalbertorussell/latc-check
git -C latc-check checkout c6d2802abe667a9a49cbcb3848f12407fdf00ef5
cd latc-check && python3.12 -m unittest discover -s tests -t .   # 157 tests pass
```

**Installing Catala 1.3.0** (needed only to rebuild the generated Python or to rerun
`clerk test`; the oracles and the harness run without it). From
`paperc-catala-family-benefits:paperC/catala_family_benefits/README.md`, Ubuntu 24.04, as
root:

```sh
apt-get update && apt-get install -y opam ninja-build libgmp-dev pkg-config
export OPAMYES=1
opam init --disable-sandboxing --bare -n
opam switch create catala 4.14.2
eval $(opam env --switch=catala)
opam install catala.1.3.0          # about 10 minutes on 4 cores
git clone https://github.com/CatalaLang/catala-examples
cd catala-examples
git checkout 2694b17ddd7c2246e207e8f077d586349ee5abea
```

**Installing OpenFisca Aotearoa 22.0.0** (needed for the NZ oracles, the official NZ
tests and the NZ spot check). From `paperc-data-nz:paperC/openfisca_nz/setup_env.sh`,
which clones the encoding at `9d77ea7`, creates a Python 3.11 virtualenv and installs with
`requirements.lock` as constraints:

```sh
paperC/openfisca_nz/setup_env.sh /path/to/work
export OPENFISCA_AOTEAROA_DIR=/path/to/work/openfisca-aotearoa
# the oracles then run with /path/to/work/venv311/bin/python
```

## 2. Getting the material

### From git

```sh
git clone https://github.com/carlosalbertorussell/latc-legal-check
cd latc-legal-check
git fetch origin 'refs/heads/paperc-*:refs/remotes/origin/paperc-*'
git branch -r                      # origin/main and the 18 origin/paperc-* branches
```

Check out one stage per folder with `git worktree`, so that each stage keeps its own
tree:

```sh
git worktree add --detach ../paperc-harness origin/paperc-harness
cd ../paperc-harness
```

Keep the clone with its `origin/*` refs. Four steps read other branches through
`git show origin/<branch>:...` (the analysis scripts, `paperc-verify-nz` `compare.py`,
`paperc-data-nz-fix` `m6_slp_fix.py`), and CTRL-DET (`paperC.harness.determinism`) clones
the current commit.

Branch heads at release `paperc-data-v1`:

| Branch | Head | Stage |
|---|---|---|
| `paperc-harness` | `89795da` | protocol (registered at `ee7c527`), harness, CTRL-NUR control |
| `paperc-catala-family-benefits` | `2046eb8` | Catala build of the family-benefit encoding |
| `paperc-data-fr` | `0aee62d` | tables FR-PF, FR-AF-1, FR-AF-2, FR-AF-3 |
| `paperc-data-fr2` | `6566904` | tables FR-APL-1 to FR-APL-5, US-121-1, US-121-2 |
| `paperc-data-fr2-fix` | `1b9debe` | corrected US-121-2 and its checks |
| `paperc-data-nz` | `7efbb1e` | tables NZ-SLP, NZ-AS, NZ-SPS (and the supplementary NZ-AS-1964) |
| `paperc-data-nz-fix` | `acd42a0` | corrected NZ-SLP and its checks |
| `paperc-run-fr` | `7099c6b` | harness runs and `REPORT-FR.md` |
| `paperc-run-fr2` | `eb508e7` | harness runs and `REPORT-FR2.md` |
| `paperc-run-nz` | `b835740` | harness runs and `REPORT-NZ.md` |
| `paperc-verify-fr` | `83a0d78` | independent verification, `VERIFY-FR.md` |
| `paperc-verify-fr2` | `9050c33` | independent verification, `VERIFY-FR2.md` |
| `paperc-verify-nz` | `d93c3c0` | independent verification, `VERIFY-NZ.md` |
| `paperc-code2-fr` | `c957417` | blinded second conversion, FR-PF and FR-AF |
| `paperc-code2-fr2` | `9e448d5` | blinded second conversion, FR-APL and US-121 |
| `paperc-code2-nz` | `723d8e9` | blinded second conversion, NZ |
| `paperc-analysis` | `b54095b` | analyses M2 and M6, `REPORT-M2-M6.md` |
| `paperc-figures` | `36f687e` | Figures 1–3, their data and cross-checks |

### From the release archive

The release tagged `paperc-data-v1` holds the same material as files:

| File | Content |
|---|---|
| `paperc-data-v1.bundle` | `git bundle` of `main` and the 18 `paperc-*` branches, full history |
| `paperc-data-v1-<branch>.tar.gz` | `git archive` of the head of one branch, unpacking into `latc-legal-check-<branch>/` (19 files: `main` and the 18 `paperc-*` branches) |
| `MANIFEST.md` | every file of the release with its branch, head commit and sha256 |

Check the downloads, then clone the bundle. The clone names the bundle's branches
`origin/<branch>`, exactly as a clone from GitHub does, so every command of section 3
works unchanged:

```sh
sha256sum paperc-data-v1.bundle paperc-data-v1-*.tar.gz   # compare with MANIFEST.md
git bundle verify paperc-data-v1.bundle
git clone paperc-data-v1.bundle latc-legal-check
cd latc-legal-check
git branch -r
```

A `.tar.gz` holds one branch's files without git metadata. It is enough to read the
results and to run the oracles, `sha256sum -c`, the harness `run` command, the verifier's
`recompute.py` and the figure script. Outside a git checkout the harness still writes
`runs.csv`, `controls.csv` and `summary.md` byte for byte, while `manifest.json` records
`"harness_commit": "unknown"` and `"tracked_inputs_unmodified": null`. The steps that need git (CTRL-DET, the analysis, the comparisons that
read `origin/<branch>`) need the bundle or a clone.

## 3. Commands, in order

Time figures are wall-clock seconds from the branch reports (4-core container, often with
several runs in parallel) unless marked as measured in the dry run of section 7.

### Step 1. Rebuild the encodings and confirm that their official tests pass

**1a. Catala family benefits** (`paperc-catala-family-benefits`, `paperC/catala_family_benefits/README.md`).
With Catala installed and catala-examples at `2694b17` (section 1):

```sh
# in the catala-examples checkout
clerk test allocations_familiales/tests/                    # 31/31 interpreted checks pass
clerk test allocations_familiales/tests/ --backend=python   # 15/15 test scopes pass
clerk build allocations-familiales
# copy _targets/python/... into paperC/catala_family_benefits/generated/ as the README lists
```

Without Catala, rerun the official tests through the committed Python build:

```sh
cd paperC/catala_family_benefits
python3.12 run_official_tests.py     # 15 passed, 0 failed, 15 test scopes, 18 assertions checked
```

**1b. Catala housing benefits (APL) and US §121** (`paperc-data-fr2`, `paperC/catala_more/README.md`):

```sh
# in the catala-examples checkout
clerk test aides_logement/tests/tests_eligibilite_apl.catala_fr --verbose                    # 7/7
clerk test aides_logement/tests/tests_eligibilite_apl.catala_fr --backend=python --verbose   # 3/3
clerk test aides_logement/tests/ --verbose                                                   # 67/67
clerk test aides_logement/tests/ --backend=python --verbose                                  # 33/33
clerk test us_tax_code/tests/ --verbose                                                      # 18/18
clerk build aides-logement -b python
clerk build us_tax_code/python/Us_tax_code.py
# copy into paperC/catala_more/generated/ as the README lists
```

Through the committed Python build:

```sh
cd paperC/catala_more
python3.12 run_official_tests_apl.py     # 3 passed, 0 failed, 3 test scopes, 3 assertions checked
python3.12 run_official_tests_us121.py   # 6 passed, 0 failed, 6 test scopes, 6 assertions checked
```

`clerk test us_tax_code/tests/ --backend=python` runs no test, because the clerk target
`us-tax-code` declares only the OCaml backend (README, "How to rebuild").

**1c. OpenFisca Aotearoa** (`paperc-data-nz`, `paperC/openfisca_nz/ENVIRONMENT.md`):

```sh
paperC/openfisca_nz/setup_env.sh /path/to/work
paperC/openfisca_nz/run_official_tests.sh /path/to/work/openfisca-aotearoa /path/to/work/venv311 /tmp/logs
# supported_living_payment 50 passed, accommodation_supplement 47 passed, sole_parent_support 25 passed
```

**1d. Second-coder builds.** `paperc-code2-fr2:paperC/catala_code2/README.md` gives its
own `clerk` commands (the same Catala and catala-examples commits; `Us_tax_code.py`
compiled with `catala python us_tax_code/us_tax_code.catala_en -o ...`). `paperc-code2-fr`
reuses `paperC/catala_family_benefits/`. `paperc-code2-nz` installs OpenFisca with
`pip install -e openfisca-aotearoa` in a Python 3.11 virtualenv
(`paperC/specs2/NZ-*/ENVIRONMENT.md`; its `pip freeze` is in
`paperC/openfisca_code2/pip_freeze.txt`).

### Step 2. Regenerate or verify each table

Each sample folder holds `spec.json`, `ground_truth.csv`, `official_examples.csv`,
`oracle.py`, `ENVIRONMENT.md`, `CONVERSION.md` and `SHA256SUMS` (format:
`paperc-harness:paperC/specs/FORMAT.md`).

**2a. Verify the checksums (no software beyond coreutils).** In every sample folder:

```sh
sha256sum -c SHA256SUMS
```

| Branch | Sample folders |
|---|---|
| `paperc-harness` | `paperC/specs/CTRL-NUR` |
| `paperc-data-fr` | `paperC/specs/{FR-PF,FR-AF-1,FR-AF-2,FR-AF-3}` |
| `paperc-data-fr2` | `paperC/specs/FR-APL/FR-APL-{1..5}`, `paperC/specs/US-121/US-121-{1,2}` (US-121-2 uncorrected) |
| `paperc-data-fr2-fix` | as `paperc-data-fr2`, with the corrected `paperC/specs/US-121/US-121-2`; also `CTRL-NUR` |
| `paperc-data-nz` | `paperC/specs/{NZ-SLP,NZ-AS,NZ-SPS}` (NZ-SLP uncorrected), `paperC/openfisca_nz/supplementary/NZ-AS-1964` |
| `paperc-data-nz-fix` | as `paperc-data-nz`, with the corrected `paperC/specs/NZ-SLP`; also `CTRL-NUR` |
| `paperc-code2-fr`, `paperc-code2-fr2`, `paperc-code2-nz` | `paperC/specs2/<ID>` (second conversions) |

The run and verify branches carry copies of the same folders. All 76 `SHA256SUMS` files
on the 18 branches verify (section 7). To check all of them from a clone:

```sh
for b in $(git branch -r | grep -o 'paperc-.*'); do
  git worktree add --detach -q ../chk-$b origin/$b
  (cd ../chk-$b && for f in $(git ls-files | grep 'SHA256SUMS$'); do
     (cd "$(dirname "$f")" && sha256sum -c --quiet SHA256SUMS && echo "OK $b $(dirname "$f")"); done)
done
```

(This loop is a convenience written for this guide; the branches define only the
per-folder `sha256sum -c SHA256SUMS`.)

**2b. Rebuild the control table CTRL-NUR** (`paperc-harness`, `paperC/specs/CTRL-NUR/ENVIRONMENT.md`):

```sh
cd paperC/specs/CTRL-NUR
python3.12 oracle.py --enumerate
python3.12 oracle.py --check-official
```

**2c. Rebuild the French family-benefit tables** (`paperc-data-fr`, `paperC/specs/README-FR.md`):

```sh
cd paperC/specs/FR-AF-1      # or FR-PF, FR-AF-2, FR-AF-3
python3 oracle.py --check-official     # 15 official scopes directly + mapped examples; writes official_examples.csv
python3 oracle.py --enumerate          # writes ground_truth.csv; about 1-3 minutes on 4 cores for an FR-AF table
python3 oracle.py --self-check         # FR-AF tables: --self-check --sample 25
python3 oracle.py --sha256             # writes SHA256SUMS
sha256sum -c SHA256SUMS
```

**2d. Rebuild FR-APL and US-121** (`paperc-data-fr2`, `paperC/specs/FR-APL/FR-APL-1/ENVIRONMENT.md`; the same commands in each of the seven folders):

```sh
cd paperC/specs/FR-APL/FR-APL-1      # or FR-APL-2..5, ../../US-121/US-121-1, US-121-2
python3 oracle.py --check-official   # official tests, directly and through the oracle
python3 oracle.py --enumerate        # rewrites ground_truth.csv (same bytes)
python3 oracle.py --write-official   # rewrites official_examples.csv (same bytes)
python3 oracle.py --check-bands      # extra points inside every interval label
```

Interpreter cross-check of the ground truth, with Catala on `PATH`
(`paperC/catala_more/interpreter_crosscheck.py`, docstring):

```sh
python3 paperC/catala_more/interpreter_crosscheck.py write <catala-examples dir> 600
(cd <catala-examples dir> && clerk test $(cat xcheck_files.txt) --verbose --color=never > report.txt 2>&1)
python3 paperC/catala_more/interpreter_crosscheck.py read <catala-examples dir>/report.txt
```

The recorded run sampled at most 600 rows per table (README-FR2.md, "How the ground truth
was checked"); its output is `paperC/catala_more/interpreter_crosscheck_report.txt`.

**2e. Corrected US-121-2** (`paperc-data-fr2-fix`, `paperC/results/REPORT-US-121-2-FIX.md` section 6):

```sh
cd paperC/specs/US-121/US-121-2
python3.12 oracle.py --check-official && python3.12 oracle.py --enumerate && python3.12 oracle.py --check-bands
```

Interpreter check of all 900 non-excluded rows, with Catala on `PATH`
(`paperC/results/US-121-2-interpreter/xcheck_us121_2.py`, docstring):

```sh
python3.12 paperC/results/US-121-2-interpreter/xcheck_us121_2.py write <catala-examples dir>
(cd <catala-examples dir> && clerk test $(cat xcheck_files.txt) --verbose --color=never > report.txt 2>&1)
python3.12 paperC/results/US-121-2-interpreter/xcheck_us121_2.py read <catala-examples dir>/report.txt
```

**2f. Rebuild the NZ tables** (`paperc-data-nz`, `paperC/specs/NZ-SLP/ENVIRONMENT.md` and `paperC/openfisca_nz/rebuild_all.sh`):

```sh
export OPENFISCA_AOTEAROA_DIR=/path/to/work/openfisca-aotearoa
cd paperC/specs/NZ-SLP               # or NZ-AS, NZ-SPS
/path/to/work/venv311/bin/python oracle.py --enumerate        # NZ-SLP: 256 rows, one simulation per row
/path/to/work/venv311/bin/python oracle.py --check-official   # official YAML cases, directly and via the oracle
python3 ../../openfisca_nz/crosscheck.py                      # OpenFisca-free re-implementation of every NZ table
```

Or everything at once (it also rewrites the logs, docs and checksums):

```sh
OPENFISCA_AOTEAROA_DIR=/path/to/clone PY=/path/to/venv311/bin/python paperC/openfisca_nz/rebuild_all.sh
```

On `paperc-data-nz-fix`, `rebuild_all.sh` would rewrite `NZ-SLP/CONVERSION.md` without its
Correction section (`REPORT-NZ-SLP-FIX.md` section 8); use the per-folder commands there.
The corrected NZ-SLP (`REPORT-NZ-SLP-FIX.md` section 9):

```sh
git checkout paperc-data-nz-fix
paperC/openfisca_nz/setup_env.sh /path/to/work
export OPENFISCA_AOTEAROA_DIR=/path/to/work/openfisca-aotearoa
cd paperC/specs/NZ-SLP
/path/to/work/venv311/bin/python oracle.py --check-official
/path/to/work/venv311/bin/python oracle.py --enumerate
python3 ../../openfisca_nz/crosscheck.py
```

**2g. Rebuild the second conversions.**

* `paperc-code2-fr` (`paperC/specs2/README-FR-CODE2.md`), from `paperC/specs2/`:

  ```sh
  python3 build_fr_code2.py spec
  for s in FR-PF FR-AF-1 FR-AF-2 FR-AF-3; do
    python3 $s/oracle.py --check-official
    python3 $s/oracle.py --write-official
    python3 $s/oracle.py --enumerate        # about 6 and 10 minutes for FR-AF-2 and FR-AF-3 on 4 cores
  done
  python3 build_fr_code2.py outputs
  python3 build_fr_code2.py stats
  python3 build_fr_code2.py sums
  ```

* `paperc-code2-fr2` (`paperC/catala_code2/README.md`): `python3 paperC/catala_code2/make_oracles.py`
  assembles each `paperC/specs2/<ID>/oracle.py`; then in each folder:

  ```sh
  python3 oracle.py --check-official
  python3 oracle.py --write-official
  python3 oracle.py --enumerate
  python3 oracle.py --check-groups N   # grouped enum labels, every N-th row
  ```

* `paperc-code2-nz` (`paperC/specs2/README-NZ-CODE2.md`), with the Python 3.11 virtualenv active:

  ```sh
  cd paperC/specs2/NZ-SLP   # likewise NZ-AS, NZ-SPS
  python oracle.py --check-official --write-official
  python oracle.py --enumerate
  sha256sum -c SHA256SUMS
  ```

### Step 3. Run the harness on each sample

Harness unit tests (any branch that carries `paperC/harness/`):

```sh
python3.12 -m unittest discover -s paperC/harness/tests -t .   # 23 tests, OK
```

Run and CTRL-DET, per sample (`paperc-harness:paperC/harness/README.md`; the spec paths
are those of the run branches):

```sh
python3.12 -m paperC.harness.run --spec <spec folder> --out paperC/results/<ID>
python3.12 -m paperC.harness.determinism --spec <spec folder> --report paperC/results/<ID>/determinism.json
```

`determinism` clones the current commit twice, runs `run` in each and compares the four
outputs byte for byte; commit any change before running it.

To reproduce the committed outputs byte for byte, `manifest.json` included, run at the
harness commit recorded in that sample's `manifest.json` (`harness_commit`) and write into
`paperC/results/<ID>` of that checkout. At a later commit only `harness_commit` in the
manifest changes; writing outside `paperC/results/<ID>` also changes the output paths
recorded in the manifest.

| Sample | Spec folder | Committed results (branch) | Harness commit in `manifest.json` | `run` time |
|---|---|---|---|---|
| CTRL-NUR | `paperC/specs/CTRL-NUR` | `paperc-harness` | `0269952` | 23 s (dry run) |
| FR-PF | `paperC/specs/FR-PF` | `paperc-run-fr` | `b8feeca` | 4.2 s |
| FR-AF-1 | `paperC/specs/FR-AF-1` | `paperc-run-fr` | `b8feeca` | 475 s |
| FR-AF-2 | `paperC/specs/FR-AF-2` | `paperc-run-fr` | `b8feeca` | 375 s |
| FR-AF-3 | `paperC/specs/FR-AF-3` | `paperc-run-fr` | `b8feeca` | 591 s |
| FR-APL-1 | `paperC/specs/FR-APL/FR-APL-1` | `paperc-run-fr2` | `ac30804` | 1.1 s |
| FR-APL-2 | `paperC/specs/FR-APL/FR-APL-2` | `paperc-run-fr2` | `ac30804` | 231 s |
| FR-APL-3 | `paperC/specs/FR-APL/FR-APL-3` | `paperc-run-fr2` | `ac30804` | 10 s |
| FR-APL-4 | `paperC/specs/FR-APL/FR-APL-4` | `paperc-run-fr2` | `ac30804` | 3.7 s |
| FR-APL-5 | `paperC/specs/FR-APL/FR-APL-5` | `paperc-run-fr2` | `ac30804` | 8.5 s |
| US-121-1 | `paperC/specs/US-121/US-121-1` | `paperc-run-fr2` | `ac30804` | 0.8 s |
| US-121-2, corrected (used in the paper) | `paperC/specs/US-121/US-121-2` | `paperc-data-fr2-fix` | `cfacf7f` | 2.2 s (dry run) |
| US-121-2, before correction | `paperC/specs/US-121/US-121-2` | `paperc-run-fr2` | `ac30804` | 2.5 s |
| NZ-SLP, corrected (used in the paper) | `paperC/specs/NZ-SLP` | `paperc-data-nz-fix` | `d399359` | 1.2 s (dry run) |
| NZ-SLP, before correction | `paperC/specs/NZ-SLP` | `paperc-run-nz` | `859b826` | 1.3 s |
| NZ-AS | `paperC/specs/NZ-AS` | `paperc-run-nz` | `859b826` | 282 s |
| NZ-SPS | `paperC/specs/NZ-SPS` | `paperc-run-nz` | `859b826` | 435 s |
| NZ-AS-1964 (supplementary) | `paperC/openfisca_nz/supplementary/NZ-AS-1964` | `paperc-run-nz` (`--out paperC/results/supplementary/NZ-AS-1964`) | `859b826` | 1.8 s |

CTRL-DET takes about twice the `run` time (for example FR-AF-3 1,176 s, NZ-SPS 851 s).

Example, the control sample at its recorded commit:

```sh
git worktree add --detach ../rerun-CTRL-NUR 0269952
cd ../rerun-CTRL-NUR
python3.12 -m paperC.harness.run --spec paperC/specs/CTRL-NUR --out paperC/results/CTRL-NUR
for f in runs.csv controls.csv summary.md manifest.json; do
  git show origin/paperc-harness:paperC/results/CTRL-NUR/$f | cmp - paperC/results/CTRL-NUR/$f && echo "$f identical"
done
```

### Step 4. Run the independent verifier

The verifiers reimplement PROTOCOL.md without reading the harness or `latc_check`. Each
branch has four steps: recompute, ground-truth spot check through the encoding,
conversion audit, comparison with the runner. The conversion audit is a reading of the
conversion notes against the encoding and has no script beyond the probes named in the
audit files (`paperC/verify/audit/`).

**`paperc-verify-fr`** (`paperC/verify/VERIFY-FR.md`; script usage lines):

```sh
python3.12 paperC/verify/recompute.py paperC/specs/<ID> paperC/verify/<ID>       # ID = FR-PF, FR-AF-1, FR-AF-2, FR-AF-3
python3.12 paperC/verify/selftest.py paperC/specs/<ID> <max cells>               # fast path = literal set algebra
python3.12 paperC/verify/spotcheck.py . paperC/verify/spotcheck                  # 50 rows per table through the Catala build
python3.12 paperC/verify/spotcheck.py . paperC/verify/spotcheck --full-pf        # all 3,240 FR-PF rows
python3.12 paperC/verify/spotcheck.py . paperC/verify/spotcheck --extended FR-AF-1 5000 20261021   # likewise FR-AF-2, FR-AF-3
python3.12 paperC/verify/compare.py paperC/verify/<ID> <runner runs.csv> <runner controls.csv> paperC/verify/<ID>/compare.txt
```

The runner files come from `paperc-run-fr`, for example
`git show origin/paperc-run-fr:paperC/results/FR-PF/runs.csv > /tmp/runs.csv`. The
`--extended` arguments are the recorded supplementary draws (5,000 rows,
`random.Random(20261021)`), as printed in `paperC/verify/spotcheck/extended_<ID>.txt`.

**`paperc-verify-fr2`** (`paperC/verify/VERIFY-FR2.md`, `paperC/verify/spotcheck/SPOTCHECK.md`):

```sh
python3.12 paperC/verify/recompute.py paperC/specs/FR-APL/FR-APL-1 paperC/verify/FR-APL-1   # likewise the other six
python3.12 -I paperC/verify/spotcheck/spotcheck.py --full     # writes spotcheck/*.csv and summary.json
python3.12 paperC/verify/compare.py paperC/verify/<ID>/runs.csv <runner runs.csv from paperc-run-fr2>
```

**`paperc-verify-nz`** (`paperC/verify/VERIFY-NZ.md`, "Reproduce"):

```sh
python3.12 paperC/verify/recompute.py paperC/specs/NZ-SLP paperC/specs/NZ-AS paperC/specs/NZ-SPS
python3.11 -m venv V && V/bin/pip install -r <requirements.lock without the -e line>
git clone https://github.com/digitalaotearoa/openfisca-aotearoa C && git -C C checkout 9d77ea7ce3c4
V/bin/pip install --no-deps -e C
V/bin/python -W ignore paperC/verify/spotcheck.py paperC/specs/NZ-SLP paperC/specs/NZ-AS paperC/specs/NZ-SPS
V/bin/python -W ignore paperC/verify/spotcheck.py --full paperC/specs/NZ-SLP paperC/specs/NZ-AS paperC/specs/NZ-SPS
python3.12 paperC/verify/citations.py C paperC/specs/NZ-SLP paperC/specs/NZ-AS paperC/specs/NZ-SPS
python3.12 paperC/verify/compare.py origin/paperc-run-nz NZ-SLP NZ-AS NZ-SPS
```

**Corrected tables.** `paperc-data-fr2-fix:paperC/verify-fix/README.md`:

```sh
python3.12 paperC/verify-fix/recompute.py paperC/specs/US-121/US-121-2 paperC/verify-fix/US-121-2
python3.12 paperC/verify-fix/spotcheck_us121_2_all.py > paperC/verify-fix/US-121-2/spotcheck_all.log
python3.12 paperC/verify-fix/compare.py paperC/verify-fix/US-121-2/runs.csv paperC/results/US-121-2/runs.csv
python3.12 paperC/verify-fix/compare_renamed.py paperC/verify-fix/US-121-2/runs.csv paperC/results/US-121-2/runs.csv
```

`paperc-data-nz-fix:paperC/results/REPORT-NZ-SLP-FIX.md` section 9:

```sh
python3.12 paperC/verify-fix/recompute.py paperC/specs/NZ-SLP
/path/to/work/venv311/bin/python -W ignore paperC/verify-fix/spotcheck.py --full paperC/specs/NZ-SLP
python3.12 paperC/verify-fix/compare_renamed.py paperC/verify-fix/NZ-SLP/runs.csv paperC/results/NZ-SLP/runs.csv
python3.12 -I paperC/verify-fix/m6_slp_fix.py
```

### Step 5. Rerun the second-conversion analysis (`paperc-analysis`)

From `paperC/analysis/REPORT-M2-M6.md`, "Reproduce". The scripts read the data and
second-conversion branches with `git show origin/<branch>:...`, so run them in a clone
that has the `origin/paperc-*` refs:

```sh
paperC/analysis/fetch_sources.sh                 # catala-examples 2694b17, openfisca-aotearoa 9d77ea7ce3c4 into _ext/
git fetch origin                                 # the scripts read origin/<branch> with git show
python3.12 -I paperC/analysis/check_yaml_reader.py   # optional; needs PyYAML, checks the stdlib YAML reader
python3.12 -I paperC/analysis/m2_fr_af.py        # FR-PF, FR-AF   (about 10 s)
python3.12 -I paperC/analysis/m6_fr_af.py        # FR-PF, FR-AF   (about 4 min; runs the Catala 1.3.0 Python build)
python3.12 -I paperC/analysis/m2_fr2.py          # FR-APL, US-121
python3.12 -I paperC/analysis/m6_fr2.py
python3.12 -I paperC/analysis/m2_nz.py           # NZ-SLP, NZ-AS, NZ-SPS
python3.12 -I paperC/analysis/m6_nz.py
python3.12 -I paperC/analysis/build_report.py    # writes REPORT-M2-M6.md from fragments/
```

The analysis compares the second conversion with the NZ-SLP table before its correction
(`paperc-data-nz`). The comparison with the corrected NZ-SLP is
`python3.12 -I paperC/verify-fix/m6_slp_fix.py` on `paperc-data-nz-fix` (step 4).

### Step 6. Redraw Figures 1–3 (`paperc-figures`)

From `paperC/figures/FIGURES.md`, with Python 3.13:

```sh
python3 -m venv .venv && .venv/bin/pip install -r paperC/figures/requirements.lock
.venv/bin/python paperC/figures/make_figures.py
```

The script reads only `paperC/results/<ID>/{runs.csv,summary.md}` on that branch (copies
of the run and fix branches, each checked against its git blob in `SOURCES.csv`).

## 4. Expected outputs

| Step | Writes | Must give |
|---|---|---|
| 1a `run_official_tests.py` | stdout | `15 passed, 0 failed, 15 test scopes, 18 assertions checked`, exit 0 |
| 1a `clerk test` | stdout | 31/31 interpreted checks, 15/15 Python-backend scopes (`clerk_test_results.txt`) |
| 1b `run_official_tests_apl.py`, `_us121.py` | stdout | 3/3 and 6/6 test scopes passed |
| 1b `clerk test` | stdout | 7/7, 3/3, 67/67, 33/33, 18/18 (`paperC/catala_more/clerk_test_results.txt`) |
| 1c `run_official_tests.sh` | `<out>/<rule>.log` | 50, 47 and 25 passed, `exit status: 0` (compare `paperC/openfisca_nz/test_logs/`) |
| 2a `sha256sum -c SHA256SUMS` | stdout | every line `OK` in all 76 folders |
| 2b–2g `oracle.py --enumerate`, `--write-official` | `ground_truth.csv`, `official_examples.csv` | byte-identical to the committed files (`git status` shows no change; `sha256sum -c SHA256SUMS` passes) |
| 2b–2g `oracle.py --check-official` | stdout (FR-AF/FR-PF also rewrite `official_examples.csv`) | every official test passes directly and every mapped example agrees with the oracle; the oracles refuse to enumerate otherwise |
| 2d `interpreter_crosscheck.py read` | stdout | 3,186 rows run, 0 disagreements (`interpreter_crosscheck_report.txt`) |
| 2e `xcheck_us121_2.py read` | stdout | 900 rows, 204 label rows pass, 696 REJECTED rows fail at `section_121.catala_en:376`, 0 disagreements; the negative control is reported (`result.txt`) |
| 2f `crosscheck.py` | stdout | every row of NZ-SLP (256), NZ-AS (86,016), NZ-SPS (122,880), NZ-AS-1964 (512) agrees |
| 3 unit tests | stdout | 23 tests, OK |
| 3 `paperC.harness.run` | `runs.csv`, `controls.csv`, `summary.md`, `manifest.json` in `--out` | exit 0; byte-identical to the committed files at the recorded harness commit; 1,266 runs (1,263 where OFFICIAL is skipped: CTRL-NUR, FR-APL-4, FR-APL-5); `tracked_inputs_unmodified: true` in a git checkout |
| 3 controls (`controls.csv`) | | CTRL-HASH, CTRL-GT, CTRL-EXCL and CTRL-PLANT pass on every sample. CTRL-H3 passes where there are no REJECTED rows and fails, as PROTOCOL.md section 7(c) expects, on FR-AF-1, FR-APL-1, FR-APL-2, FR-APL-4, US-121-1 and US-121-2. CTRL-FULL is recorded without a pass criterion on legal samples and passes on CTRL-NUR |
| 3 `paperC.harness.determinism` | `determinism.json` | `"identical": true` (CTRL-DET), with `run_a` and `run_b` hashes equal to the committed outputs |
| 4 `recompute.py` | `runs.csv`, `controls.csv`, `summary.md` (and `plant.csv` on verify-fr) | the committed verifier outputs, byte for byte |
| 4 `compare.py` | `compare.txt` or stdout | 0 differing cells against the runner (on `paperc-verify-nz` the committed NZ-AS and NZ-SPS files say "comparison pending"; see section 7). On verify-fr2 and verify-fix the verdict line reads DIFFERENT only because the columns `sample`/`sample_id` and `n`/`size` are named differently; `compare_renamed.py` maps them and prints IDENTICAL |
| 4 spot checks | CSV and text files under `paperC/verify/spotcheck/` or `paperC/verify/<ID>/` | every drawn row agrees with the ground truth (`VERIFY-*.md`); on verify-fr2 only the `seconds` fields of `summary.json` change |
| 4 `m6_slp_fix.py` | stdout | `rows first 256, second 256, compared 256`, `tally: {'agree': 256}` (`paperC/verify-fix/NZ-SLP/m6_second_conversion.txt`) |
| 5 analysis scripts | `paperC/analysis/out/*.json`, `REPORT-M2-M6.md` | the committed files, unchanged |
| 6 `make_figures.py` | `fig{1,2,3}_*.{pdf,png}`, `fig{1,2,3}_data.csv`, `fig_checks.csv` | `checks passed: 345/345`, exit 0; with the pinned environment every output file is byte-identical to the committed one |

## 5. Map of the paper's results

All result folders used by the paper are collected, byte for byte, on `paperc-figures`
under `paperC/results/<ID>/` (`SOURCES.csv` names the source branch, commit and blob of
each). The paths below are on the branch in the second column.

| Item | Branch | File(s) holding the numbers | Command that produces them |
|---|---|---|---|
| **Table 1** (samples: \|𝒟\|, \|𝒱\|, EXCLUDED, REJECTED, official rows) | `paperc-figures` (copies), sources `paperc-run-fr`, `paperc-run-fr2`, `paperc-run-nz`, `paperc-data-fr2-fix` (US-121-2), `paperc-data-nz-fix` (NZ-SLP) | `paperC/results/<ID>/summary.md`, section "Sample" (\|D\|, \|V\|, \|E\|, \|R\|, official rows); the same counts in `paperC/specs/README-FR.md`, `README-FR2.md`, `README-NZ.md` and in each `CONVERSION.md` | `oracle.py --enumerate` (step 2) writes `ground_truth.csv`; `python3.12 -m paperC.harness.run` (step 3) counts it into `summary.md` |
| **Table 2** (official examples as LATC input: C1 valid coverage, C2 spurious share) | `paperc-figures` | `paperC/results/<ID>/summary.md`, section "OFFICIAL and FULL" (`valid_coverage`, `spurious_share`, rows `OFFICIAL C1` and `OFFICIAL C2`); `runs.csv` rows with `source = OFFICIAL`; `paperC/figures/fig1_data.csv`, `fig3_data.csv`; printed values in `make_figures.py` (`TABLE2`) | `python3.12 -m paperC.harness.run` (step 3); checked by `make_figures.py` (step 6) |
| **Table 3** (mean C1 valid coverage with 10 random examples) | `paperc-figures` | `paperC/results/<ID>/summary.md`, section "valid_coverage", rows `C1 UNIFORM 10` and `C1 ZIPF 10` (column `mean`); `paperC/figures/fig2_data.csv`; printed values in `make_figures.py` (`TABLE3`) | `python3.12 -m paperC.harness.run` (step 3); checked by `make_figures.py` (step 6) |
| **Table 4** (two conversions compared) | `paperc-analysis`; corrected NZ-SLP on `paperc-data-nz-fix` | `paperC/analysis/REPORT-M2-M6.md` ("M6: agreement of the two conversions" and the M6 sections per rule family); `paperC/analysis/out/m6_fr_af.json`, `m6_fr2.json`, `m6_nz.json`; `paperc-data-nz-fix:paperC/verify-fix/NZ-SLP/m6_second_conversion.txt` and `REPORT-NZ-SLP-FIX.md` section 7 | `m6_fr_af.py`, `m6_fr2.py`, `m6_nz.py`, `build_report.py` (step 5); `m6_slp_fix.py` (step 4) |
| **Table 5** (corrected NZ-SLP and US-121-2, before and after) | `paperc-data-fr2-fix`, `paperc-data-nz-fix`; "before" on `paperc-run-fr2` and `paperc-run-nz` | `paperC/results/REPORT-US-121-2-FIX.md` sections 1–3; `paperC/results/REPORT-NZ-SLP-FIX.md` sections 3–5; after: `paperC/results/US-121-2/summary.md`, `paperC/results/NZ-SLP/summary.md` on the fix branches; before: the same paths on `paperc-run-fr2` and `paperc-run-nz` | steps 2e and 2f (corrected tables), then `python3.12 -m paperC.harness.run` on each fix branch and on the run branch (step 3) |
| **Figure 1** (official-example coverage per table) | `paperc-figures` | `paperC/figures/fig1_official_coverage.{pdf,png}`, data `fig1_data.csv` | `make_figures.py` (step 6) |
| **Figure 2** (coverage against the number of random examples) | `paperc-figures` | `paperC/figures/fig2_coverage_vs_n.{pdf,png}`, data `fig2_data.csv` | `make_figures.py` (step 6) |
| **Figure 3** (what C2 generates with the official examples) | `paperc-figures` | `paperC/figures/fig3_c2_generated.{pdf,png}`, data `fig3_data.csv` | `make_figures.py` (step 6) |
| **Hypothesis verdicts, §4.7** (H1–H4) | `paperc-figures` (per-sample indicators); `paperc-run-fr`, `paperc-run-fr2`, `paperc-run-nz` (per-branch counts); fix branches; `paperc-analysis` | per sample: `paperC/results/<ID>/summary.md`, section "Hypothesis indicators for this sample (PROTOCOL.md section 8)"; per branch: "Cross-sample summary" of `REPORT-FR.md`, "Hypothesis indicators" of `REPORT-FR2.md`, "Summary across the three legal samples" of `REPORT-NZ.md`; after correction: section 3 of `REPORT-US-121-2-FIX.md`, section 5 of `REPORT-NZ-SLP-FIX.md`; H4: `paperc-harness:paperC/results/CTRL-NUR/summary.md` and `paperC/harness/README.md`; H1 with the whole test suite: `REPORT-M2-M6.md`, "M2 verdict on H1". The decision rules are in `PROTOCOL.md` section 8 | `python3.12 -m paperC.harness.run` writes each sample's indicators. **No script applies the decision rules of PROTOCOL.md section 8 over all 14 legal samples together**; the paper's verdicts come from applying them by hand to the 14 `summary.md` files on `paperc-figures` |

To list the per-sample indicators in one place (a `grep` written for this guide, on
`paperc-figures`):

```sh
grep -A4 '^## Hypothesis indicators' paperC/results/*/summary.md
```

## 6. Known limits

* **The second conversions were not run through the full protocol.** The blinded second
  conversions (`paperc-code2-fr`, `paperc-code2-fr2`, `paperc-code2-nz`) were compared with
  the first conversions row by row (M6, Table 4). The harness, the controls and the
  verifier were run on the first conversions only; no `paperC/results/` folder exists for
  any `paperC/specs2/` table.
* **The family-benefit tables were not run through the Catala interpreter.** FR-PF and
  FR-AF-1 to FR-AF-3 were evaluated through the Python code that Catala 1.3.0 generates,
  and their official tests pass under `clerk` and through that code. The row-by-row
  interpreter cross-check (`interpreter_crosscheck.py`, `xcheck_us121_2.py`) covers only
  FR-APL and US-121.
* **The study has not been replicated independently.** The verifier branches reimplement
  the protocol from its text and agree with the harness cell by cell, and a second coder
  converted every rule blind. All of these checks were made within this project, and no
  outside group has yet reproduced the study.
* Two tables were corrected after the independent checks (US-121-2, NZ-SLP). The paper uses
  the corrected tables; the earlier outputs stay on `paperc-run-fr2` and `paperc-run-nz`.
* CTRL-DET and the byte-identical reruns were run on Linux x86_64 with CPython 3.12.3.
  Other platforms or Python versions may change the float `repr` of some metrics; the
  protocol fixes the RNG as CPython 3.12 `random`.

## 7. Dry-run record

Commands run on 2026-10-26 in a Claude Code cloud container (Ubuntu, Linux x86_64, 4
vCPUs, CPython 3.12.3, 3.11 and 3.13.17), each in a clean worktree of the branch named.
Several commands ran in parallel, so times include contention. The dry run committed
nothing to any `paperc-*` branch; the comparison of row 24 was committed afterwards (see
the note below).

| # | Command (section) | Branch | Result |
|---|---|---|---|
| 1 | `sha256sum -c SHA256SUMS` in every sample folder (2a) | all 18 | 76 of 76 folders OK |
| 2 | harness unit tests (3) | `paperc-harness` | 23 tests, OK |
| 3 | `paperC.harness.run` on CTRL-NUR at `0269952`, into `paperC/results/CTRL-NUR` (3) | `paperc-harness` | exit 0, 1,263 runs, no control fails, 23 s; all four outputs byte-identical to the committed ones, `manifest.json` included |
| 4 | the same at the branch head `89795da`, written outside the repository | `paperc-harness` | `runs.csv`, `controls.csv`, `summary.md` identical; `manifest.json` differs only in `harness_commit` and the output paths (this is why section 3 says to run at the recorded commit) |
| 5 | `paperC.harness.determinism` on CTRL-NUR (3) | `paperc-harness` | `CTRL-DET PASS: outputs are byte-identical`, 45 s |
| 6 | `paperC.harness.run` at the recorded harness commit (3) | run and fix branches | all four outputs byte-identical, `manifest.json` included, for all 16 legal and supplementary samples the paper uses: FR-PF (5 s), FR-AF-1 (875 s), FR-AF-2 (630 s), FR-AF-3 (1,021 s), FR-APL-1 (1 s), FR-APL-2 (149 s), FR-APL-3 (9 s), FR-APL-4 (2 s), FR-APL-5 (16 s), US-121-1 (0.6 s), corrected US-121-2 (2.2 s), corrected NZ-SLP (1.2 s), NZ-AS (369 s), NZ-SPS (691 s), NZ-AS-1964 (6 s); four ran at a time |
| 7 | `run_official_tests.py`; `run_official_tests_apl.py`; `run_official_tests_us121.py` (1a, 1b) | `paperc-catala-family-benefits`, `paperc-data-fr2` | 15/15 scopes (18 assertions); 3/3; 6/6 |
| 8 | `oracle.py --check-official` and `--enumerate` (2c, 2d, 2e) | `paperc-data-fr` (FR-PF), `paperc-data-fr2` (FR-APL-1, US-121-1), `paperc-data-fr2-fix` (US-121-2) | every official test passes; rewritten files byte-identical (`git status` clean, `sha256sum -c` OK) |
| 9 | `setup_env.sh` (1c) | `paperc-data-nz-fix` | exit 0; `pip freeze` equals `requirements.lock` line for line |
| 10 | `run_official_tests.sh` (1c) | `paperc-data-nz` | 50, 47 and 25 passed, exit 0, 13 s |
| 11 | NZ-SLP `oracle.py --check-official`, `--enumerate`; `crosscheck.py` (2f) | `paperc-data-nz-fix` | 13/13 asserted values pass, 13/13 mapped; 256 rows, byte-identical; crosscheck agrees on all rows of NZ-SLP, NZ-AS, NZ-SPS, NZ-AS-1964 |
| 12 | `recompute.py` FR-PF; `compare.py`; `spotcheck.py . paperC/verify/spotcheck` (4) | `paperc-verify-fr` | outputs byte-identical; 0 of 31,650 cells differ; 50/50 rows agree in each of the four tables |
| 13 | `recompute.py` US-121-1; `compare.py`; `spotcheck.py --full` (4) | `paperc-verify-fr2` | outputs byte-identical; 0 of 29,118 shared cells differ (verdict line DIFFERENT for the two column names); `summary.json` differs only in its `seconds` fields (70 s) |
| 14 | `verify-fix/recompute.py` NZ-SLP; `compare_renamed.py`; `m6_slp_fix.py` (4) | `paperc-data-nz-fix` | outputs byte-identical; 31,650 cells, 0 differ, IDENTICAL; 256/256 rows agree |
| 15 | `xcheck_us121_2.py read` on the committed `clerk_report.txt` (2e) | `paperc-data-fr2-fix` | the text of `result.txt`: 204 pass, 696 REJECTED, 0 disagreements, negative control reported |
| 16 | `fetch_sources.sh`; `m2_fr_af.py` (10 s), `m2_fr2.py` (1 s), `m2_nz.py` (3 s), `m6_nz.py` (1 s), `m6_fr2.py` (3 s), `m6_fr_af.py` (693 s under load), `build_report.py` (5) | `paperc-analysis` | exit 0 for each; `out/*.json` and `REPORT-M2-M6.md` unchanged |
| 17 | figures venv from `requirements.lock`; `make_figures.py` (6) | `paperc-figures` | `pip freeze` equals the lock; `checks passed: 345/345`, exit 0, 5 s; every PDF, PNG and CSV byte-identical |
| 18 | harness `run` and verifier `recompute.py` in an unpacked `git archive` of `paperc-data-fr2-fix` (2) | `paperc-data-fr2-fix` | US-121-2 `runs.csv`, `controls.csv`, `summary.md` byte-identical; `manifest.json` records `harness_commit` `unknown`; verifier outputs byte-identical |
| 19 | Catala install of section 1 (`opam install catala.1.3.0`) | | `catala --version` and `clerk --version` print `1.3.0`; 654 s |
| 20 | `clerk test` of steps 1a and 1b in catala-examples `2694b17` | | 31/31 and 15/15 (family benefits); 7/7, 3/3, 67/67, 33/33, 18/18 (APL, US); `us_tax_code/tests/ --backend=python` prints NO TESTS WERE RUN, as documented |
| 21 | `clerk build allocations-familiales`, `clerk build aides-logement -b python`, `clerk build us_tax_code/python/Us_tax_code.py` (1a, 1b) | `paperc-catala-family-benefits`, `paperc-data-fr2` | the 19 generated modules of `catala_family_benefits/generated/` and the 30 of `catala_more/generated/` (all but `__init__.py`) byte-identical to the built ones |
| 22 | `interpreter_crosscheck.py write … 600`, `clerk test`, `read` (2d) | `paperc-data-fr2` | 3,186 rows, 0 disagreements, the per-table counts of `interpreter_crosscheck_report.txt`; 17 s |
| 23 | `xcheck_us121_2.py write`, `clerk test`, `read` (2e) | `paperc-data-fr2-fix` | 900 rows: 204 pass, 696 REJECTED at line 376, 0 disagreements, negative control reported |
| 24 | `spotcheck.py` (50 rows) and `compare.py origin/paperc-run-nz NZ-SLP NZ-AS NZ-SPS` (4) | `paperc-verify-nz` | 50/50 rows agree in each table, 3 s; 0 of 30,384 cells differ for each of the three samples (see the note below) |

Note on `paperc-verify-nz`: until 2026-10-26 its `NZ-AS/compare.txt` and
`NZ-SPS/compare.txt` read "comparison pending", and `VERIFY-NZ.md` listed the comparison
as pending, because the runner's NZ-AS and NZ-SPS results were pushed after the verifier
ran. Rerunning `python3.12 paperC/verify/compare.py origin/paperc-run-nz NZ-SLP NZ-AS NZ-SPS`
on that branch completes it: 0 of 30,384 cells in common columns differ for NZ-AS and for
NZ-SPS, CTRL-PLANT 0 of 422 sets differ, 0 of 462 summary rows differ. The two
`compare.txt` files and `VERIFY-NZ.md` with this result are committed on
`paperc-verify-nz` at `d93c3c0`. The copies on `paperc-data-nz-fix:paperC/verify-fix/`
still read "comparison pending"; `paperc-verify-nz` holds the completed record.

Not run here: the second-conversion rebuilds (step 2g), CTRL-DET on samples other than
CTRL-NUR, the harness runs of the uncorrected US-121-2 and NZ-SLP, the NZ verifier's
`--full` spot check and `citations.py`, `check_yaml_reader.py`, and `oracle.py
--enumerate` on the large FR-AF, FR-APL-2, NZ-AS and NZ-SPS tables.
