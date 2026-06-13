# **Moox Press ↔ Moox Native Sync Roadmap**

## **Zielbild**

Moox Press bleibt WordPress-kompatibel und nutzt WordPress weiterhin als Gutenberg-Runtime.

Moox Native bleibt eigenständig und funktioniert ohne WordPress.

Wenn Moox Press installiert ist, synchronisiert eine Bridge WordPress-Objekte und Moox-native Objekte zuverlässig miteinander.

Die gleiche Infrastruktur soll langfristig auch für Headless APIs, externe Systeme sowie Moox ↔ Moox Synchronisation verwendet werden.

## Definitionen

Moox Native Entity ist die kanonische Form.
Sie besteht aus Model, Filament Resource, Config, Policies und Migrations.

Headless, Press und Sync sind optionale Capabilities dieser Entity.

moox Press ist ein optionales Package, wird es installiert, muss ein Command existieren, der den ersten (Massen)-Sync übernimmt, wird es deinstalliert, passiert gar nichts, Sync bleibt auf dem Stand stehen. Wird es erneut installiert, muss die Sync-Gap geschlossen werden.

moox Entities sind also Nativ ohne Press und - sofern als CPT registriert - werden sie mit Press gesynct. Media ist automatisch ein "CPT".

## **Grundsatzentscheidungen**

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

## **Iteration 0: Entity API Foundation**

### **Ziel**

Eine zentrale Infrastruktur schaffen, auf der künftig aufbauen:

- Moox Press Sync
- Headless APIs
- Moox ↔ Moox Synchronisation
- Externe Integrationen
- Mobile Apps
- Business-Systeme
- Frontends

Die Entity wird dabei zur zentralen Definition.

### **Architektur evaluieren**

- Bestehende API-Ansätze im Projekt erfassen
- Bestehende Sync-Ansätze (`moox/sync`) analysieren
- Bestehende Connect-Ansätze (`moox/connect`) analysieren
- Doppelungen und Überschneidungen identifizieren
- Naming evaluieren:
  - `moox/headless`
  - `moox/sync`
  - Erweiterung bestehender Pakete
  - neues Infrastruktur-Paket

### **Entity API Definition**

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

### **API Standard definieren**

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

### **Sync Standard definieren**

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

### **Driver-Konzept**

Unterschiedliche Backends unterstützen.

Beispiele:

- Native Driver
- Press Driver
- Remote Driver
- Future Drivers

### **SDK evaluieren**

Prüfen ob benötigt:

- PHP SDK
- JavaScript SDK
- Laravel Client
- externe Integrationen

Entscheidung vertagen bis API Foundation stabil ist.

### **Moox Build Integration**

Prüfen wie Build künftig automatisch generiert:

- API Definition
- API Endpoints
- Policies
- Resources
- Tests
- Sync Definitionen

Neue Packages sollen standardmäßig API-ready sein.

### **Deliverables**

- Config-Erweiterung für Headless/API
- Config-Erweiterung für Press Projection
- Entity Registry / Entity Resolver
- Driver Interface
- Headless Route Generator
- Press Projection Interface
- Sync Event Definition
- Beispiel-Entity für Tests
- Tests für Config, Registry, Routes und Driver Resolution

## **Iteration 1: Multisite Installer stabilisieren**

- `moox:wp-install` mit Multisite sauber testen
- Multisite als Standard für Press evaluieren
- `wp-config.php`-Generierung stabilisieren
- Blog/Site ↔ Locale Mapping finalisieren
- Filament Resources für Sites/Blogs prüfen
- Smoke Tests für Single Site und Multisite
- Upgrade-Pfad für bestehende Press-Installationen definieren

## **Iteration 2: Sync Foundation**

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

### **Deliverables**

- Mapping Model + Migration
- Sync Log Model + Migration
- Lock Service
- Sync Manager
- Press Sync Driver
- Native Sync Driver
- Backfill/Repair Command
- Tests für Mapping, Locking, Idempotenz und Repair

## **Iteration 3: Media Sync**

- Gemeinsame Dateiablage klären
- WP Attachment ↔ Moox Media Mapping
- Alt, Caption, Title, Description mappen
- Responsive Image Sizes mappen
- Media Metadata mappen
- Spatie Media Library Integration prüfen
- Astrotomic Translation Mapping prüfen
- Delete/Trash/Restore Verhalten definieren
- Moox Media UI mit Press-Datenfluss verbinden

## **Iteration 4: Post Types und Taxonomies**

- WP Posts ↔ Moox Entities mappen
- WP Pages ↔ Moox Entities mappen
- Custom Post Types berücksichtigen
- Categories/Tags/Custom Taxonomies mappen
- Term Relationships synchronisieren
- Slug, Status, Author, Dates, Parent, Menu Order mappen
- Meta Keys Strategie definieren
- Gutenberg Content / Blocks als WP-owned Content behandeln

## **Iteration 5: Users, Roles, Permissions**

- WP Users ↔ Moox Users mappen
- Roles und Capabilities mappen
- Auth-Strategie klären
- Permission-Konflikte definieren
- Filament Permissions mit WP Roles abgleichen

## **Iteration 6: Audit, Revisions, Activity**

- WP Revisions erfassen
- Moox Audit / Spatie ActivityLog anbinden
- Sync Events loggen
- Diff-/History-Ansicht evaluieren
- Rollback-Verhalten klären

## **Iteration 7: Hardening**

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

## **Offene Detailthemen**

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
- Field Mapping: WP fields ↔ Moox attributes
- Meta Mapping
- Capability Mapping
- API Auth
- Loop Prevention
- Sync Status UI

## **Beispiel Config für eine Entity**

```php
return [

    // exposes standardized Moox API
    'headless' => true,

    // optional WordPress/Gutenberg projection
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
    ],

];
```
