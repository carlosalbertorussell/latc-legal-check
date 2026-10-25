# latc-legal-check

This repository holds the code, tables, results and verification records for the paper:

> C. A. Russell, *Do the Examples Cover the Law? Testing Example-Driven Completion on Encoded Legislation*.

The study tests whether LATC, an algorithm that completes a decision table from example cases, can serve as the completion step of a Rules as Code model. It converts fourteen rules into finite decision tables and evaluates each table in full through its encoding. The rules come from two public encodings of legislation:

- [Catala](https://github.com/CatalaLang/catala-examples): French family benefits, French housing benefits, and section 121 of the US Internal Revenue Code;
- [OpenFisca Aotearoa](https://github.com/digitalaotearoa/openfisca-aotearoa): New Zealand social-security eligibility.

LATC then completes each table from the encodings' official test cases and from random cases. A protocol was registered before any result was computed.

## Layout: one branch per stage

The work is kept on branches, so every stage can be cited by commit.

| Branch | Content |
|---|---|
| `paperc-harness` | Protocol (`paperC/PROTOCOL.md`, committed at `ee7c527`) and the experiment harness |
| `paperc-catala-family-benefits` | Catala build of the French family-benefit encoding |
| `paperc-data-fr` | Tables FR-PF and FR-AF-1 to FR-AF-3 |
| `paperc-data-fr2` | Tables FR-APL-1 to FR-APL-5, US-121-1 and US-121-2 |
| `paperc-data-fr2-fix` | Corrected US-121-2 |
| `paperc-data-nz` | Tables NZ-SLP, NZ-AS and NZ-SPS |
| `paperc-data-nz-fix` | Corrected NZ-SLP |
| `paperc-run-fr`, `paperc-run-fr2`, `paperc-run-nz` | Harness runs and reports |
| `paperc-verify-fr`, `paperc-verify-fr2`, `paperc-verify-nz` | Independent verification |
| `paperc-code2-fr`, `paperc-code2-fr2`, `paperc-code2-nz` | Blinded second conversions |
| `paperc-analysis` | The analyses of Sections 4.6 and 4.7 |
| `paperc-figures` | Figures 1–3, the script that draws them, and their data |

Each branch documents how to reproduce its stage in the README files under `paperC/`. The harness uses the [`latc_check`](https://github.com/carlosalbertorussell/latc-check) package at commit `c6d2802`, unmodified.

## Licences

The code is MIT. Material derived from Catala is Apache-2.0, material derived from OpenFisca Aotearoa is AGPL-3.0, and the UCI Nursery control table is CC BY 4.0. `LICENSES.md` gives the full map by folder and sample.

## History

These branches were first published in [latc-check](https://github.com/carlosalbertorussell/latc-check) and were moved here unchanged, with every commit hash preserved.
