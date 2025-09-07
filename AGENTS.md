# Repository Guidelines

## Synopsis
This repository is an Obsidian-based “second brain” vault. Contributions focus on clear Markdown notes, consistent folder prefixes (000–600), and maintainable dashboards. Keep edits small and well-labeled, preserve internal links, and prefer improving structure over adding redundancy.

## Project Structure & Module Organization
- 000_inbox: Quick capture. Rough notes that will later be refactored or moved.
- 100_start: Onboarding context (e.g., `how_i_work.md`, `readme.md`). Ground rules and operating methods.
- 200_life: Personal values and life docs (e.g., `values.md`). Stable, rarely moved.
- 300_disciplines: Skill/area tracks. Subfolders such as `310_holistic_dev`, `320_mastery_over_math_and_ai`, `330_financial_health` contain `milestones.md`, `pillars.md`, `tasks.md`, plus `refs/`, `research_ideas/`, and reading lists when relevant.
- 400_projects: Active work by domain (`410_personal_projects`, `420_professional_projects`, `430_class_projects`, `archived/`). Each project keeps `milestones.md`, `tasks.md`, and `refs.md` near the work.
- 500_dashboards_and_lists: High-level views (`520_project_dashboard.md`, `530_task_dashboard.md`, lists like reading/watch). Update these when moving or renaming notes.
- 600_resources: Reusable material (routines, exercise, recipes, Obsidian/Dataview tips, quotes, writings). Treat as reference, not project-specific.
- Markwhen: Timeline assets. Keep filenames descriptive and cross-link back to notes.
- Root files: `readme.md` (status), `directory_structure.txt` (overview; update after structural changes), images/assets. `.obsidian/` exists locally but is gitignored.

## Build, Test, and Development Commands
- Open vault: `obsidian .` (or open this folder in Obsidian).
- Search notes: `rg "keyword" 300_disciplines 400_projects`
- Verify links (quick): `rg "\[\["` to scan internal links; fix broken ones.
- Optional lint: If installed, `markdownlint **/*.md` to catch formatting issues.

## Coding Style & Naming Conventions
- Markdown: Use `#`–`###` headings, fenced code blocks, and short sections.
- Naming: Keep human-readable names; maintain numeric directory prefixes. Prefer consistent casing within a folder.
- Linking: Use Obsidian internal links (`[[Note Name]]`) and relative paths for files.
- Dataview: Keep fields stable if used; avoid renaming field keys casually.

## Commit & Pull Request Guidelines
- Commits: `type: summary` (e.g., `add: reading list for salmonella project`). Group related file moves and dashboard updates.
- PRs: Describe scope, list touched directories, include screenshots for dashboard changes, and note any moved/renamed notes.

## Security & Configuration Tips
- Do not commit secrets or private tokens in notes.
- Large binaries: prefer linking or storing in a dedicated assets area.
- Obsidian config is local; avoid repo-wide plugin assumptions.
