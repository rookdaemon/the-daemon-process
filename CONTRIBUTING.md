# CONTRIBUTING.md

## Development Workflow

This is a literary project managed through granules (multi-agent orchestrator).

### Branch Convention
- `main` — canonical text
- Worker branches: `worker-W-{n}-granule-G-{n}` (managed by granules)

### Commit Convention
- `feat: [chapter/artifact]` — new content
- `fix: [chapter]` — corrections (technical accuracy, voice breaks)
- `docs: [artifact]` — planning artifact updates
- `refactor: [chapter]` — prose restructuring without content changes

### Quality Gates
All chapter content must pass the CONSTRAINTS.md verification checklist before merge.
See CLAUDE.md for the full quality gate list.

### File Format
- All prose in Markdown
- One file per chapter: `chapters/chapter_NN_title.md`
- Planning artifacts in root
- Domain research in `/domain/`
- Specs in `/spec/`
- Reference passages in `/reference/`
