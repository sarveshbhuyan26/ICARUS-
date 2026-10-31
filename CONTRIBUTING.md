# Contributing

This is an early hackathon prototype and no project-wide license has been selected.
Discuss nontrivial contributions with the maintainers before submitting work.

## Development

Follow the root README to create an environment and install
`preppilot/requirements-dev.txt`. Run the engine, bank and regression checks from
`preppilot/` before proposing changes. Keep the server single-worker.

- Keep learner policy pure in `engine.py`; add a relevant self-check for policy changes.
- Add bank checks for original questions and preserve source attribution for imports.
- Use textContent for question, AI and learner text; do not inject it through innerHTML.
- Preserve responsive layouts and both colour schemes.
- Do not commit accounts, API keys, private transcripts or installed dependencies.
- Explain the observed problem, resulting behavior and validation in your PR.

The current eight regression tests are in `preppilot/test_regressions.py`.
Changes to the browser flow also require a browser check; Python tests alone do not
validate appearance or frontend interactions. Do not add unsupported learning claims.
