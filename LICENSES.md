# Licences

The Paper C material sits under `paperC/` on the `paperc-*` branches. Four licences apply, by origin.

| What | Where (on the `paperc-*` branches) | Licence | Text |
|---|---|---|---|
| Code written for this study: harness, converters, verifier, analyses, figure scripts | `paperC/harness/`, `paperC/verify/`, `paperC/analysis/`, `paperC/figures/`, `paperC/__init__.py` and other scripts not listed below | MIT | `LICENSE` |
| Material derived from the Catala encodings of [catala-examples](https://github.com/CatalaLang/catala-examples): the French family and housing benefits and US Internal Revenue Code §121 | `paperC/catala_family_benefits/`, `paperC/catala_more/`, `paperC/catala_code2/`; and, under `paperC/specs/`, `paperC/specs2/` and `paperC/results/`, every folder or file for the samples `FR-*` and `US-*` | Apache-2.0 | `LICENSES/Apache-2.0.txt` |
| Material derived from [OpenFisca Aotearoa](https://github.com/digitalaotearoa/openfisca-aotearoa): the New Zealand Supported Living Payment, Accommodation Supplement and Sole Parent Support | `paperC/openfisca_nz/`, `paperC/openfisca_code2/`; and, under `paperC/specs/`, `paperC/specs2/` and `paperC/results/`, every folder or file for the samples `NZ-*` | AGPL-3.0 | `LICENSES/AGPL-3.0.txt` |
| The UCI Nursery control table | `paperC/specs/CTRL-NUR/`, `paperC/results/CTRL-NUR/` | CC BY 4.0, as distributed by the UCI Machine Learning Repository | https://creativecommons.org/licenses/by/4.0/ |

The `paperc-*` branches were cut from the `main` branch of [latc-check](https://github.com/carlosalbertorussell/latc-check), so their history also contains that package and its results. Those files keep the licences of latc-check: MIT for code and CC BY 4.0 for data.

Where a file mixes material of two origins, the more restrictive licence applies to it.
