# Wetland connectivity graphs predict nutrient exchange at floodplain margins

![build](https://img.shields.io/badge/build-passing-brightgreen)
![python](https://img.shields.io/badge/python-3.11-blue)
![status](https://img.shields.io/badge/paper-accepted-informational)

We represent a floodplain mosaic as a weighted connectivity graph and show that graph centrality predicts nutrient exchange measured at 22 margin plots. The paper introduces the graph construction and a cross-validation protocol for withheld wetland patches.

## What is in this repository

This repository hosts the working implementation of the connectivity-graph
analysis described in the manuscript. It is organised so that a reader can
inspect the exact pipeline used for the paper's figures:

```
src/            graph construction, centrality, cross-validation
data/           schema definitions and a miniature example mosaic
experiments/    configuration for the 22-plot evaluation
results/        placeholder for generated output
tests/          smoke tests
```

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Quick start

```bash
python -m src.pipeline --config experiments/paper.yaml --out results/run01
```

## Reproducing the paper tables

Table 2 compares graph centrality with the observed nutrient exchange at the
22 margin plots. Run the configuration in `experiments/paper.yaml` and inspect
`results/run01/summary.json`; the columns `centrality`, `exchange_observed`,
and `exchange_predicted` map one-to-one onto the published table.

Table 3 reports the withheld-patch cross-validation. The fold definitions live
in `experiments/folds.yaml`; no random seed is used because folds are defined
by catchment.

## Hardware

Any machine with 16 GB of memory is sufficient; the full evaluation ran in
about eleven minutes on a four-core laptop.

## Data availability

The plot-level measurements are archived by the regional agency. The miniature
mosaic in `data/` is a synthetic, redistributable example that exercises the
same schema.

### Input schema

| Column | Type | Description |
|--------|------|-------------|
| `plot_id` | string | Stable identifier for a margin plot; unique within a catchment. |
| `patch_id` | string | Identifier of the wetland patch adjacent to the plot. |
| `catchment` | string | Catchment code used to define cross-validation folds. |
| `exchange` | float | Measured nutrient exchange at the plot, in milligrams per square metre per day. |
| `water_table_depth` | float | Mean water-table depth over the measurement window, in centimetres. |
| `vegetation_cover` | float | Fractional vegetation cover recorded at the plot, between 0 and 1. |

### Derived tables

The pipeline derives a node table (one row per wetland patch) and an edge table
(one row per shared hydrological boundary). Centrality is computed on the edge
table with the weights defined in `experiments/paper.yaml`. The withheld-patch
evaluation groups plots by catchment; this deliberately leaks no information
across folds and explains why the fold assignment is fixed by geography rather
than randomised.

### Known limitations

The example mosaic contains six patches, which is too small to reproduce the
published centrality ordering. Use it only to check that the pipeline runs. The
full plot table is subject to the agency's redistribution policy and is not
included in this repository.

## Changelog

- 2026-09-28: initial import of the graph construction code.
- 2026-09-28: add cross-validation module and placeholder results folder.
- 2026-09-29: tidy documentation and add the miniature mosaic.
- 2026-10-02: add smoke tests for the public entry points.
- 2026-10-05: expand the pipeline docstrings and the configuration comments.
- 2026-10-08: rename `graph_tools` to `src/connectivity.py`.
- 2026-10-11: add a schema check for the plot-level input table.
- 2026-10-14: make the output folder creation idempotent.
- 2026-10-17: add a short troubleshooting section to the README.
- 2026-10-20: freeze the configuration used for the manuscript's figures.
- 2026-10-23: record the exact library versions used for the evaluation.

## Troubleshooting

**`KeyError: 'plot_id'`** — the input table is missing the plot identifier
column. Regenerate the example input from `data/schema.json`.

**`FileNotFoundError: experiments/folds.yaml`** — the fold definitions are
distributed with the paper's artifact bundle, not with the miniature example.

**Different numbers on a GPU machine** — the centrality computation is
deterministic; differences usually mean the input table was reordered. The
pipeline sorts by `plot_id` before building the graph.

## Development notes

The repository follows a simple branch model: `main` carries the working tree,
feature branches are used for experimental refactors and are not published.
Issues are disabled while the artifact is being prepared.

## Acknowledgements

This work was supported by the Willowbank internal research fund and by the
regional monitoring agency's data-sharing programme. Fieldwork was carried out
under permits WB-2024-118 and WB-2025-034.

## License

MIT. See `LICENSE`.

## Contact

Willowbank Watershed Lab <lab@willowbank-labs.org>

## Release status

**This repository is not released yet.**

The manuscript listed above has been accepted, but the artifact is still being
prepared for publication. The code, the processed data bundle, and the
reproduction scripts will be published here **after the camera-ready deadline**
and after the artifact-availability statement is finalized with the publisher.
Until then this repository is a private working tree: the layout is in place,
but the entry points may change without notice and should not be cited as the
paper's implementation.

If you need early access for review, contact the lab directly.
