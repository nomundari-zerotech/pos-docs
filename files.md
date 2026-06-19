# Documentation Files

This page lists the documentation files that are published through GitBook.

## Root Pages

| File | Purpose |
| --- | --- |
| `README.md` | GitBook landing page. |
| `SUMMARY.md` | GitBook sidebar structure. |
| `.gitbook.yaml` | GitBook sync configuration. |
| `business-rules-and-processes.md` | Joplin-exported feature overview, business rules, and process diagrams. |
| `class-diagrams.md` | Joplin-exported Mermaid class diagrams for core domain objects. |
| `files.md` | Documentation file index. |

## Architecture Files

| File | Purpose |
| --- | --- |
| `architecture/system-components.md` | Joplin-exported PlantUML system component diagram. |

## Feature Files

| File | Purpose |
| --- | --- |
| `features/login-flow.md` | Desktop login sequence and signed header details. |
| `features/pos-login-and-license-activation.md` | POS login and license activation PlantUML flow. |
| `features/signed-pos-request-verification.md` | Signed POS request verification PlantUML sequence. |
| `features/rbac-permission-check.md` | RBAC permission check PlantUML flow. |
| `features/pos-checkout-activity.md` | POS checkout activity PlantUML flow. |
| `features/pos-checkout-sequence.md` | POS checkout sequence PlantUML flow. |
| `features/sales-process-diagram.md` | Sales process PlantUML flow. |
| `features/master-data-import-to-organization-products.md` | Master data import PlantUML flow. |
| `features/inventory-stock-movement.md` | Inventory stock movement PlantUML flow. |

## Diagram Files

| File | Purpose |
| --- | --- |
| `diagrams/README.md` | Rendered diagram index for GitBook. |
| `diagrams/*.puml` | Editable PlantUML source files. |
| `diagrams/*.svg` | Browser-renderable diagram exports used by GitBook pages. |
| `_resources/*` | Joplin-exported image resources used by synced pages. |

## GitBook Publishing Files

GitBook reads `.gitbook.yaml` from the repository root and uses `SUMMARY.md` to build the sidebar. The sidebar is grouped as Architecture, Features, and Files.
