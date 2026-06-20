# Agent Instructions

Project: badminton-analysis

## Working Guidelines

- Read the repository structure, README, package metadata, and existing tests before making changes.
- Preserve user changes and avoid reverting work you did not create.
- Keep edits scoped to the requested task and follow the project's existing style.
- Prefer the repository's established commands for build, lint, test, and formatting.
- When behavior changes, add or update focused tests where practical.
- Before finishing, report what changed and which checks were run.

## Development Verification

- For any `web_app/` change, always start the web frontend locally and capture a screenshot before finishing to confirm the page still renders and the changed feature is not broken.
- Use the project web command unless it changes: `cd web_app && python -m http.server 5173`.
- If the web UI depends on backend behavior, start the backend as well and verify the connected flow rather than relying only on demo data.
- For any `backend/` or analysis-pipeline change, test the backend with a real `.mp4` file through the API or web upload flow before finishing.
- Prefer an existing small MP4 fixture or sample from the repo when available; otherwise ask the user which video to use.
- Report the exact web URL, screenshot method/location, MP4 filename, and backend/API checks that were run.

<!-- pr-workflow-hook:v1 -->
## PR Workflow

- For every GitHub PR, review both `AGENTS.md` and `CLAUDE.md` before opening or updating the PR.
- If the change affects setup, commands, architecture, workflows, conventions, or agent expectations, update both files in the same PR.
- Preserve existing project-specific guidance; add concise notes instead of replacing unrelated content.
<!-- /pr-workflow-hook:v1 -->
