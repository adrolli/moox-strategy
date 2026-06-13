# Moox Press ↔ Moox Native Sync Roadmap

## Zielbild

Moox Press bleibt WordPress-kompatibel und nutzt WordPress weiterhin als Gutenberg-Runtime.

Moox Native bleibt eigenständig und funktioniert ohne WordPress.

Wenn Moox Press installiert ist, synchronisiert eine Bridge WordPress-Objekte und Moox-native Objekte zuverlässig miteinander.

Die gleiche Infrastruktur soll langfristig auch für Headless APIs, externe Systeme sowie Moox ↔ Moox Synchronisation verwendet werden.

---

## Definitionen

Moox Native Entity ist die kanonische Form.
Sie besteht aus Model, Filament Resource, Config, Policies und Migrations.

Headless, Press und Sync sind optionale Capabilities dieser Entity.

moox Press ist ein optionales Package. Wird es installiert, muss ein Command existieren, der den ersten (Massen)-Sync übernimmt. Wird es deinstalliert, passiert gar nichts, der Sync bleibt auf dem letzten Stand stehen. Wird es erneut installiert, muss die Sync-Gap geschlossen werden.

Moox Entities sind nativ ohne Press. Sofern eine Entity als Press Projection registriert wird, wird sie mit WordPress synchronisiert.

Media ist eine spezielle Press Projection und wird in WordPress als Attachment dargestellt.

### Begriffe

#### Driver

Technischer Zugriff auf ein Backend oder externes System.

#### Projection

Darstellung einer Native Entity in einem externen System.

#### Mapping

Zuordnung von Identitäten zwischen Native Entities und externen Projektionen.

#### Sync

Transport von Änderungen zwischen Native Entities und Projektionen.

#### Origin

Historischer Entstehungskontext eines Datensatzes.

#### Field Mode

Verhalten eines Feldes innerhalb einer Projection.

Mögliche Field Modes:

- sync
- native_only
- projection_only
- readonly
- derived
- ignored

WordPress Attachments sind niemals kanonisch.
Sie sind Projektionen einer kanonischen Moox Media Entity.

---

## Grundsatzentscheidungen

- Kein WordPress-Core-Fork.
- Keine WPDB-Simulation in der ersten Ausbaustufe.
- WordPress Multisite wird Standard für mehrsprachige Press-Installationen.
- Moox Native bleibt unabhängig von WordPress.
- Press aktiviert eine eigene Sync-Strategy, statt das Verhalten von Moox global zu verändern.
- Mapping/Sync gilt für Media, Post Types, Taxonomies, Users, Options und Relations.
- Live-Sync zunächst synchron, schwere Nebenjobs per Queue.
- Repair/Backfill Commands sind Pflicht.
- APIs, Sync und externe Integrationen sollen auf einer gemeinsamen Entity-Definition basieren.
- Keine Speziallösungen für Press, wenn dieselbe Infrastruktur auch für Headless oder Moox ↔ Moox nutzbar ist.

---

## Iteration 0: Entity API Foundation

### Ziel

Eine zentrale Infrastruktur schaffen, auf der künftig aufbauen:

- Moox Press Sync
- Headless APIs
- Moox ↔ Moox Synchronisation
- Externe Integrationen
- Mobile Apps
- Business-Systeme
- Frontends

Die Entity wird dabei zur zentralen Definition.

### Architektur evaluieren

- Bestehende API-Ansätze im Projekt erfassen
- Bestehende Sync-Ansätze (`moox/sync`) analysieren
- Bestehende Connect-Ansätze (`moox/connect`) analysieren
- Doppelungen und Überschneidungen identifizieren
- Naming evaluieren:
  - `moox/headless`
  - `moox/sync`
  - Erweiterung bestehender Pakete
  - neues Infrastruktur-Paket

### Entity API Definition

Definition auf Entity-Ebene ermöglichen.

Beispielsweise:

- CRUD
- Filter
- Relations
- Media
- Taxonomies
- Translations
- Events
- Sync
- Permissions

Konfiguration bevorzugt über bestehende Moox-Config-Strukturen.

### API Standard definieren

- Route-Konvention
- Versionierung
- Authentifizierung
- Authorization
- Error-Handling
- Pagination
- Filtering
- Sorting
- Includes
- Translations
- Media Responses
- Taxonomy Responses

Entscheidung treffen:

- REST + JSON
- JSON:API
- Hybrid
- Eigener Standard

### Sync Standard definieren

Einheitliche Event-Struktur definieren.

Beispiele:

- `media.created`
- `media.updated`
- `media.deleted`
- `post.updated`
- `term.attached`
- `user.updated`

Festlegen:

- Payload-Struktur
- Event-Namen
- Retry-Verhalten
- Locking
- Conflict Handling

### Identity & Mapping Model

- UUID Strategie definieren
- Canonical IDs definieren
- External IDs definieren
- Mapping Strategie definieren
- Projection Identity definieren
- Locale Mapping definieren
- Origin Tracking definieren

### Driver-Konzept

Unterschiedliche Backends unterstützen.

Beispiele:

- Native Driver
- Press Driver
- Remote Driver
- Future Drivers

### SDK evaluieren

Prüfen ob benötigt:

- PHP SDK
- JavaScript SDK
- Laravel Client
- externe Integrationen

Entscheidung vertagen bis API Foundation stabil ist.

### Moox Build Integration

Prüfen wie Build künftig automatisch generiert:

- API Definition
- API Endpoints
- Policies
- Resources
- Tests
- Sync Definitionen

Neue Packages sollen standardmäßig API-ready sein.

### Deliverables

- Config-Erweiterung für Headless/API
- Config-Erweiterung für Press Projection
- Entity Registry / Entity Resolver
- Driver Interface
- Headless Route Generator
- Press Projection Interface
- Sync Event Definition
- Identity Model
- UUID Strategie
- Mapping Contract
- Beispiel-Entity für Tests
- Tests für Config, Registry, Routes und Driver Resolution

---

## Iteration 1: Multisite Installer stabilisieren

- `moox:wp-install` mit Multisite sauber testen
- Multisite als Standard für Press evaluieren
- `wp-config.php`-Generierung stabilisieren
- Blog/Site ↔ Locale Mapping finalisieren
- Filament Resources für Sites/Blogs prüfen
- Smoke Tests für Single Site und Multisite
- Upgrade-Pfad für bestehende Press-Installationen definieren

---

## Iteration 2: Sync Foundation

- Mapping-Tabelle definieren
- Sync-Service-Interface definieren
- Object Types definieren:
  - post
  - page
  - media
  - term
  - user
  - option
- Locale/Blog Mapping integrieren
- Atomic Lock Konzept für WP ↔ Moox einbauen
- Idempotente `updateOrCreate`-Logik pro Object Type
- Sync-Richtung und Driver-Konzept definieren:
  - Native Driver
  - Press Driver
- Repair/Backfill Command anlegen:
  - `moox:press-sync --all`
  - `moox:press-sync --type=media`
  - `moox:press-sync --repair`

### Mapping enthält mindestens

- native_type
- native_id
- native_uuid
- external_system
- external_type
- external_id
- blog_id
- locale
- status
- last_synced_at
- last_native_hash
- last_external_hash
- error_code
- error_message

### Deliverables

- Mapping Model + Migration
- Sync Log Model + Migration
- Lock Service
- Sync Manager
- Press Sync Driver
- Native Sync Driver
- Backfill/Repair Command
- Tests für Mapping, Locking, Idempotenz und Repair

---

## Iteration 3: Media Sync

- Gemeinsame Dateiablage klären
- WP Attachment ↔ Moox Media Mapping
- Alt, Caption, Title, Description mappen
- Responsive Image Sizes mappen
- Media Metadata mappen
- Spatie Media Library Integration prüfen
- Astrotomic Translation Mapping prüfen
- Delete/Trash/Restore Verhalten definieren
- Moox Media UI mit Press-Datenfluss verbinden

### Media Localization Strategy

- File ist global/shared
- Media Identity ist global
- Alt, Caption, Title und Description sind translatable
- WP Attachments sind locale-spezifische Projections
- Unterschiedliche Dateien pro Locale sind möglich, aber nicht Standard
- Responsive Sizes werden geteilt
- Shared File Strategy definieren
- Upload Origin Strategy definieren

### Deliverables

- Media Projection Model
- Attachment Mapping
- Shared File Strategy
- Locale Metadata Strategy

---

## Iteration 4: Post Types und Taxonomies

- WP Posts ↔ Moox Entities mappen
- WP Pages ↔ Moox Entities mappen
- Custom Post Types berücksichtigen
- Categories/Tags/Custom Taxonomies mappen
- Term Relationships synchronisieren
- Slug, Status, Author, Dates, Parent, Menu Order mappen
- Meta Keys Strategie definieren
- Gutenberg Content als Press-managed Projection Field behandeln

---

## Iteration 5: Users, Roles, Permissions

- WP Users ↔ Moox Users mappen
- Roles und Capabilities mappen
- Auth-Strategie klären
- Permission-Konflikte definieren
- Filament Permissions mit WP Roles abgleichen

---

## Iteration 6: Audit, Revisions, Activity

- WP Revisions erfassen
- Moox Audit / Spatie ActivityLog anbinden
- Sync Events loggen
- Diff-/History-Ansicht evaluieren
- Rollback-Verhalten klären

---

## Iteration 7: Hardening

- Konfliktfälle definieren
- Fehlerstatus im Mapping speichern
- Retry-Mechanismus
- Queue für schwere Tasks
- Tests für:
  - Multisite
  - Locale Mapping
  - Delete/Trash
  - Media
  - Terms
- Monitoring/Debug UI für Sync-Status
- Dokumentation für Press vs Native Driver

---

## Offene Detailthemen

- Atomic Lock WP ↔ Moox
- Responsive Image Sizes
- Meta Keys
- WP Roles / Capabilities
- Trash / Restore
- Revisions / Autosaves
- Gutenberg Save Hooks
- Bulk Actions
- Repair / Backfill
- Bestehende Installationen migrieren
- Was passiert bei Plugin-Deaktivierung?
- Was passiert bei fehlgeschlagenem Sync?
- API Versionierung
- Entity API Definition
- Moox ↔ Moox Synchronisation
- Connect / Sync / Headless Konsolidierung
- Canonical ID / UUID Strategie
- Field Mapping
- Field Modes
- Meta Mapping
- Capability Mapping
- API Auth
- Loop Prevention
- Sync Status UI
- Media Localization Strategy
- Shared Files vs Locale Files
- Projection Identity

---

## Beispiel Config für eine Entity

```php
return [

    'headless' => true,

    'press' => [
        'enabled' => true,
        'post_type' => 'project',

        'supports' => [
            'title',
            'editor',
            'thumbnail',
            'excerpt',
            'revisions',
        ],

        'taxonomies' => [
            'project_category',
            'project_tag',
        ],

        'fields' => [

            'title' => [
                'map' => 'post_title',
                'mode' => 'sync',
            ],

            'slug' => [
                'map' => 'post_name',
                'mode' => 'sync',
            ],

            'status' => [
                'map' => 'post_status',
                'mode' => 'sync',
            ],

            'gutenberg_blocks' => [
                'map' => 'post_content',
                'mode' => 'projection_only',
                'managed_by' => 'press',
            ],

        ],

    ],

];
```
