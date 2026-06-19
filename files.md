# Documentation Files

This page lists the documentation files that are published through GitBook.

## Root Pages

| File | Purpose |
| --- | --- |
| `README.md` | GitBook landing page. |
| `SUMMARY.md` | GitBook sidebar structure. |
| `.gitbook.yaml` | GitBook sync configuration. |
| `business-rules-and-processes.md` | Feature behavior, business rules, and process diagrams. |
| `class-diagrams.md` | Mermaid class diagrams for core domain objects. |
| `files.md` | Documentation file index. |

## Diagram Files

| File | Purpose |
| --- | --- |
| `diagrams/README.md` | Rendered diagram index for GitBook. |
| `diagrams/*.puml` | Editable PlantUML source files. |
| `diagrams/*.svg` | Browser-renderable diagram exports used by GitBook pages. |

## GitBook Publishing Files

GitBook reads `.gitbook.yaml` from the repository root and uses `SUMMARY.md` to build the sidebar. The sidebar is grouped as Architecture, Features, and Files.
