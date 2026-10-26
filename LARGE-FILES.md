# Large files

Anonymous GitHub serves files of up to 8 MB. Every file larger than 7.5 MB (7,500,000 bytes) on the `paperc-*` branches is stored here gzipped with `gzip -9 -n`, so the compressed file is reproducible and carries no original name or timestamp. There are 17 such files, all `ground_truth.csv` tables. Each `.gz` is under 8 MB, so none needed splitting.

## Restoring

Restore one file in place, keeping the `.gz`:

```sh
gunzip -k paperc-data-fr/paperC/specs/FR-AF-1/ground_truth.csv.gz
```

Restore all of them from the top of this tree:

```sh
find . -name 'ground_truth.csv.gz' -exec gunzip -k {} +
```

If a later version of this tree ever holds a file split into parts (`<name>.gz.part-aa`, `<name>.gz.part-ab`, …, made with `split -b 7M`), join and decompress the parts in order with `cat <name>.gz.part-* | gunzip > <name>`. No file is split at present.

## Checking

Each sample folder's `SHA256SUMS` lists the sha256 of the uncompressed `ground_truth.csv`. Restore the file first, then check it from inside its folder:

```sh
cd paperc-data-fr/paperC/specs/FR-AF-1
gunzip -k ground_truth.csv.gz
sha256sum -c SHA256SUMS
```

The table below gives both checksums, so the `.gz` can also be checked as stored.

## Files

| Original path | Size (bytes) | sha256 of the original | Compressed file | Size (bytes) | sha256 of the compressed file |
|---|---:|---|---|---:|---|
| `paperc-code2-fr/paperC/specs2/FR-AF-2/ground_truth.csv` | 24,773,697 | `57e297da59c311399a8595f811f32a81673e6981e8930bdb3b03ccb1e6be86e7` | `ground_truth.csv.gz` | 664,725 | `51370f93ecfc53bef09dceda9f18454c07be7e04c1ef746dbaa894f0e4f1eef9` |
| `paperc-code2-fr/paperC/specs2/FR-AF-3/ground_truth.csv` | 15,905,309 | `49f2bbe44121f7ec5c26484fdd8290351a6ba4daf858ba532f3a69b2bd62e9c2` | `ground_truth.csv.gz` | 578,195 | `21f5ad3163749ce1449eff351fe15d9dfe40d2017237bd5604e84401e0926f5e` |
| `paperc-code2-fr2/paperC/specs2/FR-APL-3/ground_truth.csv` | 8,458,381 | `0d185863777c929c39f0b54fc8ab9c0f51d9cd914245d0c6a069ad6a3e9e13d4` | `ground_truth.csv.gz` | 176,972 | `2745c73cbbda03e46c21ee6953da9c8f9bc4fa35eb3d1084f4495d87ea4ab880` |
| `paperc-code2-fr2/paperC/specs2/FR-APL-4/ground_truth.csv` | 11,283,643 | `9fb6eb5f16a79bd89102c5545cc0317c4503742059e49e381650929b43a2fafe` | `ground_truth.csv.gz` | 279,626 | `9711e1455bec658e2a00f81a4436c8bd8cdb4867a8e9e576d8fabd64af0abddc` |
| `paperc-data-fr/paperC/specs/FR-AF-1/ground_truth.csv` | 13,592,667 | `48f4260fb816029ac9d273b3bd637aba5b0e3082e1d733e59d326a060ef06a31` | `ground_truth.csv.gz` | 559,214 | `57580dd10ae7627b9e3c1809063c199f9e9170591e5c8e167f83c593fdc4e418` |
| `paperc-data-fr/paperC/specs/FR-AF-2/ground_truth.csv` | 18,904,558 | `a1fe3171d7f6c3874c75b112e294f897a6e776fe69f648d36765c76c432a9105` | `ground_truth.csv.gz` | 485,869 | `dbd4a2b1170d49944e70414fb3feb3eeb43a1e892b8712e487b87b2610e6e8ff` |
| `paperc-data-fr/paperC/specs/FR-AF-3/ground_truth.csv` | 16,707,714 | `fd2ce541599c50025ce6c82c5a2887f142475790749037ccfbc50105ecbe981c` | `ground_truth.csv.gz` | 531,428 | `f9395a52001b00c2bc50183d17916072c0b98ee9a61c9e302e77d44174eee282` |
| `paperc-data-nz-fix/paperC/specs/NZ-SPS/ground_truth.csv` | 10,765,656 | `50892dd0f6e3c99e37a3ade75b10a644506c9ff82cc08d538fa6a7d392402962` | `ground_truth.csv.gz` | 343,590 | `f819fe2734d41b98e5084e56f5faade56c88a42d75ac228608f371f7a5e62afa` |
| `paperc-data-nz/paperC/specs/NZ-SPS/ground_truth.csv` | 10,765,656 | `50892dd0f6e3c99e37a3ade75b10a644506c9ff82cc08d538fa6a7d392402962` | `ground_truth.csv.gz` | 343,590 | `f819fe2734d41b98e5084e56f5faade56c88a42d75ac228608f371f7a5e62afa` |
| `paperc-run-fr/paperC/specs/FR-AF-1/ground_truth.csv` | 13,592,667 | `48f4260fb816029ac9d273b3bd637aba5b0e3082e1d733e59d326a060ef06a31` | `ground_truth.csv.gz` | 559,214 | `57580dd10ae7627b9e3c1809063c199f9e9170591e5c8e167f83c593fdc4e418` |
| `paperc-run-fr/paperC/specs/FR-AF-2/ground_truth.csv` | 18,904,558 | `a1fe3171d7f6c3874c75b112e294f897a6e776fe69f648d36765c76c432a9105` | `ground_truth.csv.gz` | 485,869 | `dbd4a2b1170d49944e70414fb3feb3eeb43a1e892b8712e487b87b2610e6e8ff` |
| `paperc-run-fr/paperC/specs/FR-AF-3/ground_truth.csv` | 16,707,714 | `fd2ce541599c50025ce6c82c5a2887f142475790749037ccfbc50105ecbe981c` | `ground_truth.csv.gz` | 531,428 | `f9395a52001b00c2bc50183d17916072c0b98ee9a61c9e302e77d44174eee282` |
| `paperc-run-nz/paperC/specs/NZ-SPS/ground_truth.csv` | 10,765,656 | `50892dd0f6e3c99e37a3ade75b10a644506c9ff82cc08d538fa6a7d392402962` | `ground_truth.csv.gz` | 343,590 | `f819fe2734d41b98e5084e56f5faade56c88a42d75ac228608f371f7a5e62afa` |
| `paperc-verify-fr/paperC/specs/FR-AF-1/ground_truth.csv` | 13,592,667 | `48f4260fb816029ac9d273b3bd637aba5b0e3082e1d733e59d326a060ef06a31` | `ground_truth.csv.gz` | 559,214 | `57580dd10ae7627b9e3c1809063c199f9e9170591e5c8e167f83c593fdc4e418` |
| `paperc-verify-fr/paperC/specs/FR-AF-2/ground_truth.csv` | 18,904,558 | `a1fe3171d7f6c3874c75b112e294f897a6e776fe69f648d36765c76c432a9105` | `ground_truth.csv.gz` | 485,869 | `dbd4a2b1170d49944e70414fb3feb3eeb43a1e892b8712e487b87b2610e6e8ff` |
| `paperc-verify-fr/paperC/specs/FR-AF-3/ground_truth.csv` | 16,707,714 | `fd2ce541599c50025ce6c82c5a2887f142475790749037ccfbc50105ecbe981c` | `ground_truth.csv.gz` | 531,428 | `f9395a52001b00c2bc50183d17916072c0b98ee9a61c9e302e77d44174eee282` |
| `paperc-verify-nz/paperC/specs/NZ-SPS/ground_truth.csv` | 10,765,656 | `50892dd0f6e3c99e37a3ade75b10a644506c9ff82cc08d538fa6a7d392402962` | `ground_truth.csv.gz` | 343,590 | `f819fe2734d41b98e5084e56f5faade56c88a42d75ac228608f371f7a5e62afa` |
