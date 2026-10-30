# Data and code for evaluating learned emulators of operando microscopy

This repository accompanies the paper **Evaluating Next-Frame Predictions,
Long-Term Predictions, and Protocol Timing in Operando Microscopy** by Nidhish
Sagar, Jiho Han, Miles Cranmer, and Akshay Rao (NeurIPS 2026 Workshop:
Sim2Science: ML with Imperfect Scientific Models;
[OpenReview](https://openreview.net/forum?id=2HYHoXLdIi)).

The study evaluates learned video emulators on operando interferometric
scattering microscopy. Image intensity is treated as an uncalibrated optical
measurement. It is not interpreted as concentration, stoichiometry, phase,
diffusivity, transport, or mechanism.

## Quick start

Run these commands from the directory that contains this README. The source
download is 1.49 GB. The eight decompressed HDF5 files occupy 2.26 GB; allow at
least 4 GB of free space, or 5–6 GB if you also generate the model-ready
arrays.

Requirements are Python 3.10 or newer and the `zstd` command-line program. On
Ubuntu or Debian, install Zstandard with `sudo apt install zstd`; on macOS, use
`brew install zstd`.

```bash
python3 scripts/prepare_source_data.py --download --decompress
```

This command works after downloading the repository ZIP. It recognizes the
small Git LFS pointer files in the ZIP, replaces them with the actual archives,
checks every SHA-256 hash, and decompresses all eight files. It is safe to rerun
after an interrupted download. A local Git LFS installation is not required.
Successful completion ends with eight verified archives and creates:

```text
data/source_uncompressed/*.h5
```

To confirm that the HDF5 files are readable and contain the expected datasets:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements-data.txt
python3 scripts/inspect_source_data.py
```

The source archives can also be browsed and downloaded individually from the
[data directory](https://github.com/sagarnidhish/sim2science-2026-operando/tree/main/data/source).

## What is included

- Eight single-particle HDF5 movies: four particles recorded at 25 °C and the
  same four particles recorded at 45 °C. Each file also contains camera timing,
  voltage, and current.
- Deterministic preprocessing code that creates the model-ready arrays.
- Training and evaluation code for the matched model comparison, identity
  holdouts, rollout tests, protocol-input tests, and the Walrus diagnostic.
- Fixed split logic, model settings, seeds, environment files, tests, and
  compact numerical summaries used in the paper.
- Code and data licenses plus file checksums.

## What is regenerated

Model-ready arrays, trained checkpoints, prediction tensors, tables, figures,
and the paper PDF are generated from the source movies by the supplied scripts,
which keeps the download small.

## 1. Download and unpack the source data

The source files are compressed losslessly with Zstandard and stored through
Git LFS. The repository ZIP contains small pointer files rather than duplicate
copies of the data. The quick-start command above replaces those pointers with
the actual files from GitHub, checks their SHA-256 hashes, and
decompresses them.

```bash
python3 scripts/prepare_source_data.py --download --decompress
```

To verify archives that have already been downloaded, without downloading them
again:

```bash
python3 scripts/prepare_source_data.py --verify --decompress
```

See `data/README.md` for the file structure and measurement units.

## 2. Create the model-ready arrays (optional)

```bash
python3 -m pip install -r requirements-analysis.txt
python3 data_prep/make_well_data.py \
  --source-dir data/source_uncompressed \
  --output-dir data_prep/arrays
```

This step is needed for model training, but not for viewing or analysing the
source HDF5 data. The preprocessing is applied independently to each movie. It
trims a one-frame timing mismatch when present, standardizes the complete
movie, resizes frames to 128 × 128, normalizes voltage and current, and
interpolates these protocol channels onto the camera times.

## 3. Run validation tests

```bash
python3 -m unittest discover -s tests -v
```

The tests check persistence alignment, common optical thresholds, split
integrity, result metadata, aggregation units, the data downloader, and the
evaluation scripts. They validate the implementation but do not retrain the
models.

## 4. Train and evaluate

Install the CUDA environment on a compatible GPU system:

```bash
python3 -m pip install -r requirements-training-cuda.txt
```

One matched-grid configuration can be run with a compatible CUDA GPU:

```bash
python3 train/train_matched_grid.py \
  --payload-id 0 \
  --arrays-dir data_prep/arrays \
  --out-root results/reproduced
```

Payload identifiers 0 through 23 cover the six model families, image-only or
protocol-conditioned input, and direct or residual targets. A complete matched
comparison runs all 24 payloads. The
scripts under `analysis/` validate and aggregate completed runs. The compact
files under `results/frozen/` and `artifacts/` record the numerical summaries
reported in the paper.

## Troubleshooting

- If `zstd` is missing, install it using the command for your operating system
  above and rerun the same preparation command.
- If a download is interrupted, rerun the preparation command. Partial files
  are never accepted as completed archives.
- If `--verify` reports a checksum mismatch for a ZIP pointer file, rerun with
  `--download`; the downloader will replace it with the real archive.
- The `.h5.zst` archives are stored with Git LFS. A repository ZIP contains
  small pointer files, which the preparation command replaces with the
  archives.

## Dataset scope

The eight movies are paired measurements of four physical graphite particles.
The four particles share one cell-level protocol trace, so they are not four
independent cell experiments. Temporal windows are optimization samples, not
experimental replicates. The data support a controlled comparison of models
and evaluation choices, not population-level claims about cells or graphite.

## Licenses and citation

Code is released under the MIT License (`LICENSE-CODE`). Source data are
released under CC BY 4.0 (`LICENSE-DATA.md`). If you use the data or code,
please cite:

```bibtex
@inproceedings{sagar2026evaluating,
  title     = {Evaluating Next-Frame Predictions, Long-Term Predictions, and
               Protocol Timing in Operando Microscopy},
  author    = {Sagar, Nidhish and Han, Jiho and Cranmer, Miles and Rao, Akshay},
  booktitle = {NeurIPS 2026 Workshop: Sim2Science: ML with Imperfect
               Scientific Models},
  year      = {2026},
  url       = {https://openreview.net/forum?id=2HYHoXLdIi}
}
```
