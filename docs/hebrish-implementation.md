# Hebrish implementation notes

How the playbook maps onto the real tooling. Checked 2026-11-04 against the repos and datasets named here. `ASR-Training` and `Voice-Training-Recorder` are private repos.

## Components

| Playbook step | Real thing | Where |
| --- | --- | --- |
| Base model | NVIDIA Parakeet TDT 0.6B v2 | Fine-tune line `Hebrish/Sabich-Rolling`, v2.8 at HF revision `bc6b34e` |
| Daily app | Hebrish desktop and Android | `Hebrish-ASR/Hebrish-App` |
| History, desktop | SQLite `~/.local/share/hebrish/history.db` | App 0.7.0+; `model_rev` and `decode` per row from 0.7.3 |
| History, Android | SQLite `history.db`, exported as CSV | Android 0.8.1+; no History screen yet (backlog B-19) |
| Agent review, desktop | Skill `asr-dev/dictation-history-dataset` | `Claude-ASR-Dev-Plugin`; cursor `last_id` in the Recorder's `data/priority-review.json` |
| Agent review, Android | Skill `asr-dev/android-history-review` | Same plugin; cursor `last_reviewed_at` in the private dataset's `state.json` |
| Private staging | `danielrosehill/Hebrish-Dictation-History`, `danielrosehill/Daniel-Android-Transcript-Exports` | Private HF datasets, text only for desktop, CSV exports for Android |
| Error list | `Hebrish/ASR-Error-List` | Public, gated `auto`, revision `e8e0e6d`: 1,844 words, 436 occurrences, 1,527 active |
| Promotion | Agent `error-list-updater`, `scripts/apply_findings.py` | `Voice-Training-Recorder` |
| Targeted recording | Priority words in the capture app | https://train.hebrish.app; sentences are generated, never copied from dictation |
| Pattern fixes | Skill `voice-model-training/focus-cycle` | About 150 sentences for one pattern, about a day per cycle; started 2026-11-04 |
| Incremental training | Skill `voice-model-training/rolling-update` | Modal; continues from the last rolling weights on new takes plus replay |
| Challenge sentences | Test page of the capture app | Recorder releases 174 and 175; `app/public/shared/check.js`; `ASR-Training/docs/test-mode.md` |
| Sentence queue | Skill `queue-test-sentences` | Sentences on current web topics, served first on the Test page |
| Graded takes | `Hebrish/Model-Checks` | Public, gated `auto`; subset of `danielrosehill/Read-Aloud-Voice-Sentences` |
| Machine second opinion | Skill `judge-model-checks` | Fills the `ai_*` columns, reports human and Claude agreement |

Last measured rolling run: `r26-roll` (v2.8, 2026-11-03), 341 new takes, 2,436 training rows, 5.9 minutes training, 18.5 minutes end to end.

## Naming

- The playbook's "challenge sentences" are the **Test page**. The name "correction mode" has been used for it in conversation, but **Correct mode** is a separate feature: free speech, with the transcript fixed by hand (`ASR-Training/docs/corrections.md`). Both feed `Hebrish/Model-Checks`.
- "Cognate fallback" is the failure type key `hebrew-to-english`. All eight keys are in `ERROR_TYPES` in `check.js`: `wrong-word`, `hebrew-to-english`, `hebrew-spelling`, `name-brand`, `number-format`, `dropped-word`, `added-word`, `split-merge`.

## Known gaps

1. **`Hebrish/Model-Checks` has no rows yet.** At revision `3dd4b1b` it holds only its card and schema. The parent corpus at revision `9fd2829` (2026-11-04) has 0 of 19,990 rows with a `check_verdict` or `mode: correction`, so graded takes had not been published by then. Whether any are waiting on the phone or in R2 was not checked. The sample in `samples/` is therefore illustrative.
2. **The human gate does not yet control training.** The playbook describes the intent: inconsequential failures are not trained on. Per `test-mode.md` and the Recorder's `docs/dataset.md`, every Test page take trains like an ordinary bank take whatever its verdict, and only Correct mode's Acceptable takes are excluded. This comes from the docs; the training code was not checked.
3. **Failing terms are not wired back automatically.** `judge-model-checks` writes `data/model-checks/latest.json`, but `lexicon-saturation` and `vocab-gaps` do not read it yet, so vocabulary is added by hand.
4. **Rolling runs have no eval.** `rolling-update` trains and publishes only. "Improving" is judged from daily use, and benchmarks are run separately on request.
5. **The review window is a cursor.** The skills read everything since the last review rather than a strict 24 hours. Run daily, they come to the same thing.
