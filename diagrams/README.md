# PlantUML Diagrams

This directory contains editable PlantUML source diagrams and exported SVG diagrams for core Kass POS flows.

## POS Login And License Activation

Source: `pos-login-license-activity.puml`

![POS login and license activity](pos-login-license-activity.svg)

## POS Checkout Activity

Source: `pos-checkout-activity.puml`

![POS checkout activity](pos-checkout-activity.svg)

## POS Checkout Sequence

Source: `pos-checkout-sequence.puml`

![POS checkout sequence](pos-checkout-sequence.svg)

## Signed POS Request Sequence

Source: `signed-pos-request-sequence.puml`

![Signed POS request sequence](signed-pos-request-sequence.svg)

## Master Data Import

Source: `master-data-import-activity.puml`

![Master data import activity](master-data-import-activity.svg)

## Inventory Stock Movement

Source: `inventory-stock-movement-activity.puml`

![Inventory stock movement activity](inventory-stock-movement-activity.svg)

## RBAC Permission Check

Source: `rbac-permission-check-activity.puml`

![RBAC permission check activity](rbac-permission-check-activity.svg)

## System Components

Source: `system-components.puml`

![System components](system-components.svg)

## Updating Diagrams

Render with a PlantUML extension or CLI, for example:

```sh
plantuml docs/diagrams/*.puml
```

Keep the `.puml` files as the source of truth and commit updated SVG exports for GitBook.
