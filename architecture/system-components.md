# System Components

```plantuml

@startuml
title Kass POS System Components

skinparam componentStyle rectangle

package "Frontend Apps" {
  [Admin Next.js] as AdminUi
  [Super Admin Next.js] as SuperAdminUi
  [Internal Master Data Next.js] as MasterDataUi
  [Desktop Electron Shell] as DesktopShell
  [Desktop Next.js Renderer] as DesktopRenderer
}

package "Backend Monolith: apps/backend" {
  [Elysia App Entrypoint] as Backend
  [Admin Surface /admin] as AdminSurface
  [POS Surface /pos] as PosSurface
  [Super Admin Surface /super-admin] as SuperAdminSurface
  [Master Data Surface /master-data] as MasterDataSurface
  [Permission Sync Bootstrap] as PermissionSync
}

package "Shared Packages" {
  [@repo/api-client] as ApiClient
  [@repo/db-schema] as DbSchema
  [@repo/db-core] as DbCore
  [@repo/db-lite] as DbLite
  [@repo/permission-middleware] as PermissionMiddleware
  [@repo/ui] as SharedUi
}

database "PostgreSQL" as Postgres
database "PGlite Local DB" as Pglite
storage "Desktop Local Settings" as LocalSettings

AdminUi --> AdminSurface
SuperAdminUi --> SuperAdminSurface
MasterDataUi --> MasterDataSurface
DesktopRenderer --> DesktopShell
DesktopShell --> PosSurface

Backend --> AdminSurface
Backend --> PosSurface
Backend --> SuperAdminSurface
Backend --> MasterDataSurface

AdminSurface --> PermissionMiddleware
PosSurface --> PermissionMiddleware
SuperAdminSurface --> PermissionMiddleware
MasterDataSurface --> PermissionMiddleware
PermissionSync --> Postgres

AdminSurface --> DbCore
PosSurface --> DbCore
SuperAdminSurface --> DbCore
MasterDataSurface --> DbCore
DbCore --> Postgres
DbCore --> DbSchema

DesktopShell --> DbLite
DbLite --> Pglite
DesktopShell --> LocalSettings
DesktopShell --> ApiClient
ApiClient --> PosSurface

AdminUi --> SharedUi
SuperAdminUi --> SharedUi
MasterDataUi --> SharedUi
DesktopRenderer --> SharedUi
@enduml
