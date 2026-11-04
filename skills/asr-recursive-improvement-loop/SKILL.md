---
name: asr-recursive-improvement-loop
description: Run one cycle of a recursive improvement loop for a personal or fine-tuned speech-to-text model. Reads the transcript history since the last review, flags clear mistranscriptions, confirms with the user what they meant, adds confirmed words to an error list and prepares targeted sentences for the next incremental fine-tune. Use for "review my transcripts", "what has my ASR model been getting wrong", "mine my dictation history", "run the improvement loop", or setting this loop up for an ASR project.
---

# ASR recursive improvement loop

One cycle: **read history → flag → confirm with the user → update the error list → prepare sentences → hand off to training.** Run it about daily. Background: this repo's `README.md`.

## Setup (first run)

Ask for anything you cannot find, then save it to `asr-loop.json` in the project root:

```json
{
  "history_source": "path to the transcript history (SQLite, CSV, JSONL or a folder)",
  "text_field": "the column holding the model's transcript",
  "model_field": "the column holding the model version, or null",
  "error_list": "data/error-list.jsonl",
  "sentences_out": "data/targeted-sentences.jsonl",
  "cursor": null,
  "train_command": "how the user runs an incremental fine-tune, or null",
  "sentences_per_word": 5
}
```

If the history has no model version, recommend adding one, because without it nobody can tell which version fixed what. Open databases read-only, and work on a copy if the app holds a lock.

## 1. Read

Read every row after `cursor`. If `cursor` is null, read the last 24 hours. Read the rows yourself: word lists and spellcheckers miss real-word errors (a brand name heard as an ordinary word). For a large history, split it across subagents.

## 2. Flag

Flag anything that reads as a mishearing of something the speaker plausibly said:

- tech, product and brand names, and acronyms
- domain terms
- names of people, companies and projects
- foreign-language words, including a word turned into a sound-alike in the main language
- ordinary words heard as something else

Do not flag casing or punctuation alone, disfluencies the speaker really said, or spellings plausible for what was said. If context does not settle what was meant, mark the row "unsure". Never guess it away or drop it.

Group by correct form. Check the error list: a known word with a known wrong form is one more occurrence, a new wrong form is an update, and anything else is new.

## 3. Confirm

Show one table: heard as, should be, count, new or update, kind. Let the user strike or correct rows. **The user's confirmation is the label: never add an unconfirmed word.**

## 4. Update the error list

Append or update one JSON line per word. Keep it to words: no sentence or context from the history, since transcripts are private and this list may be shared.

```json
{"word": "Qdrant", "heard_as": ["Qdrint", "quadrant"], "kinds": ["tech"], "meaning": "vector database", "occurrences": 2, "models": ["my-model v3"], "first_seen": "2026-11-01", "last_seen": "2026-11-04", "takes": 0, "target": 10, "status": "training", "source": "history"}
```

## 5. Prepare sentences

For each word below its target, write `sentences_per_word` new natural sentences in the user's style, to `sentences_out`. Write fresh sentences and never copy from the history. Vary the word's position and the surrounding words.

For a format or pattern error rather than a word, write about 150 sentences for that one pattern, about a quarter of them contrast sentences containing the neighbouring form that must stay as it is. Only choose patterns the audio can decide. If only the speaker's intent separates two forms, say so and skip it.

## 6. Hand off

Tell the user how many sentences are waiting to be read. Training is theirs to start: it costs money and changes the model they use, so run `train_command` only when asked. Recommend continuing from the latest weights, with replay data mixed in and a held-out test set kept apart.

## 7. Close

Set `cursor` to the newest row reviewed. Report words added and updated, words still unsure, and any word that keeps failing after reaching its target: that one needs different sentences, not more of the same.

## Optional: challenge sentences

If the user's recording app can transcribe a read sentence, log each attempt with the audio path, ground truth, model output, model version, a pass or fail, and failure types. Treat real failures as training data and skip inconsequential ones.
