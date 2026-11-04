# Sample data

| File | Source | Real or illustrative |
| --- | --- | --- |
| `asr-error-list.sample.jsonl` | [`Hebrish/ASR-Error-List`](https://huggingface.co/datasets/Hebrish/ASR-Error-List) `words.jsonl`, revision `e8e0e6d` | **Real rows**, copied unchanged |
| `asr-error-occurrences.sample.jsonl` | Same dataset, `occurrences.jsonl` | **Real rows**, copied unchanged: one word across model versions |
| `model-checks.sample.jsonl` | Columns from [`Hebrish/Model-Checks`](https://huggingface.co/datasets/Hebrish/Model-Checks) `meta/schema.json`, revision `3dd4b1b` | **Illustrative**: every row has `_illustrative: true`, and the sentences are written for this sample |

`Hebrish/Model-Checks` has no published rows yet, so its sample shows the shape of the data, not real grades. Replace it with real rows once they are published. The real schema has 87 columns; this shows the ones that matter here, and the train and test splits are separate files rather than a column.

## Error list fields

`word` is the correct form and `heard_as` the wrong forms as the model wrote them. `occurrences` counts confirmed errors. `takes` and `target` show recordings made against recordings wanted, and `status` is `training` until the target is reached. `source` is `history` when the word came from a reviewed dictation and `manual` when added by hand. No sentence or dictation context is ever stored.

## Model check fields

`text` is the ground truth, `model_hypothesis` is what the model wrote and `asr_model` pins the exact weights. `check_verdict` is the human grade, `check_errors` the words it got wrong and `check_error_types` the failure types. Row `example-0004` shows a difference judged inconsequential: it is saved as a pass with no errors.
