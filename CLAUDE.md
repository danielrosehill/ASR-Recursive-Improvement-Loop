# ASR Recursive Improvement Loop

Public idea documentation and a one-skill Claude Code plugin.

Release tier: small

- `README.md` and `pdf/playbook.typ` hold the same playbook: change both, then rebuild the PDF.
- Build: `typst compile --root . --font-path ~/repos/github/audio-dictation/Hebrish-Private-Plugin/assets/fonts pdf/playbook.typ pdf/ASR-Recursive-Improvement-Loop.pdf` (any IBM Plex Sans install works as the font path).
- The PDF asset name carries no version, so the README link stays valid across releases.
- The blog write-up (`blog.danielrosehill.com`, Contentful entry `5EZTfj7Dc8z3wqOBm66LRh`) serves its own copy of the PDF from `Personal-Site-Blog/website/public/assets/files/asr-recursive-improvement-loop.pdf`. `/assets/` is cached immutable for a year, so after a PDF rebuild copy it there under a new file name and update the post's link.
