Moox Entity Projection Foundation — Iteration 0 Implementation Spec

Goal

Build the neutral foundation for entity projections, drivers, mappings and sync events.

This iteration must not implement any concrete external system.

The foundation must be testable with an in-memory driver and a dummy Project entity.

⸻

Non-Goals

Do not implement WordPress support.

Do not implement Press support.

Do not create HTTP routes.

Do not create database migrations.

Do not create queues or jobs.

Do not implement real sync execution.

Do not add new Composer dependencies.

Do not refactor unrelated packages.

Do not invent new package names unless explicitly required by the existing Moox structure.

Do not add CMS-specific terminology.

⸻

Core Terms

Entity

A Moox native entity.

The entity is the canonical Moox representation.

Projection

A system-specific representation of an entity.

Examples in tests may use neutral names like JsonProjection or ArrayProjection.

Driver

Technical access layer for a projection backend.

In Iteration 0 only in-memory test drivers are allowed.

Mapping

Identity relation between a native entity and a projection entity.

Sync Event

A DTO describing a change that can be transported between entity and projection layers.

Sync Context

A DTO carrying metadata for one sync operation, especially origin and correlation information.

⸻

Hard Rule

The complete implementation of Iteration 0 must work without knowing that WordPress exists.

Forbidden words in class names, namespaces, config keys, tests, comments and docs:

* WordPress
* WP
* Press
* Gutenberg
* Post Type
* CPT
* Attachment

⸻

Standard Exceptions

Use only these exception types unless existing project conventions require otherwise:

InvalidFieldModeException
InvalidProjectionDefinitionException
DriverNotFoundException
EntityNotFoundException
ProjectionNotFoundException

Do not invent additional exception hierarchies in Iteration 0.

⸻

DTO Rules

DTOs must be plain PHP objects.

Allowed:

* readonly properties
* value objects
* immutable update methods
* array serialization

Forbidden:

* container access
* service resolution
* Laravel facades
* Eloquent models
* database access
* HTTP access
* queue access

No helpers except existing project conventions.

⸻

Ticket 0.1 — Entity Capability Config

Goal

Allow existing Moox entity configs to declare optional projection/headless/sync capabilities without implementing them yet.

Do

Add support for neutral capability config keys.

Minimum supported structure:

return [
    'headless' => true,
    'projections' => [
        'json' => [
            'enabled' => true,
            'driver' => 'array',
            'fields' => [
                'title' => [
                    'map' => 'title',
                    'mode' => 'sync',
                ],
            ],
        ],
    ],
    'sync' => [
        'enabled' => true,
    ],
];

Support field modes:

sync
native_only
projection_only
readonly
derived
ignored

Do Not

Do not create routes.

Do not create controllers.

Do not create database tables.

Do not add concrete external system config.

Do not hardcode package-specific entity names.

Acceptance Criteria

* Config can be parsed.
* Missing optional keys do not break parsing.
* Disabled projections are ignored.
* Unknown field modes fail validation.
* Tests cover enabled and disabled projections.

⸻

Ticket 0.2 — Entity Registry

Goal

Create a registry that can resolve configured entities and their capabilities.

Interfaces / Classes

Create or adapt according to existing Moox conventions:

EntityRegistry
EntityDefinition
EntityCapabilityResolver

Required Behavior

The registry must provide:

all(): array
get(string $name): ?EntityDefinition
has(string $name): bool
capabilities(string $name): array
projections(string $name): array

EntityDefinition should expose:

name()
config()
hasCapability(string $capability): bool
projection(string $name): ?ProjectionDefinition
projections(): array

Do Not

Do not scan arbitrary files if Moox already has config loading conventions.

Do not introduce reflection-heavy magic.

Do not require Filament bootstrapping in unit tests.

Acceptance Criteria

* Dummy Project entity can be registered.
* Registry can list projections.
* Registry can resolve field mappings.
* Registry handles missing config safely.
* Unit tests use only dummy config.

⸻

Ticket 0.3 — Projection Contract

Goal

Define a neutral contract for projections.

Interface

interface ProjectionInterface
{
    public function name(): string;
    public function driver(): string;
    public function fields(): array;
    public function field(string $name): ?array;
    public function isEnabled(): bool;
}

Create value object:

ProjectionDefinition

Field Definition

Each field may contain:

map
mode
scope
transform

scope is allowed only for projection_only.

Allowed value:

projection

Do Not

Do not implement concrete external projections.

Do not add system-specific fields.

Do not introduce ownership concepts.

Do not use managed_by.

Do not use stored_in.

Acceptance Criteria

* ProjectionDefinition validates field modes.
* scope is rejected unless mode is projection_only.
* Projection can return mapped field names.
* Tests cover invalid field mode and invalid scope.

⸻

Ticket 0.4 — Driver Contract

Goal

Define a neutral driver interface.

Interface

interface DriverInterface
{
    public function name(): string;
    public function supports(string $operation): bool;
    public function read(MappingData $mapping): ?array;
    public function write(MappingData $mapping, array $payload): DriverResult;
    public function delete(MappingData $mapping): DriverResult;
}

Create:

DriverRegistry
DriverResult
ArrayDriver

ArrayDriver is only for tests and local foundation validation.

Do Not

Do not connect to external systems.

Do not use HTTP clients.

Do not use queues.

Do not write database records.

Acceptance Criteria

* DriverRegistry can register and resolve drivers.
* ArrayDriver can read/write/delete from memory.
* DriverResult contains success status and optional error data.
* Tests verify unsupported operations fail predictably.

⸻

Ticket 0.5 — Mapping Contract

Goal

Define the neutral data structure for mapping native entities to projections.

DTO

final class MappingData
{
    public function __construct(
        public readonly string $nativeType,
        public readonly string|int|null $nativeId,
        public readonly ?string $nativeUuid,
        public readonly string $externalSystem,
        public readonly string $externalType,
        public readonly string|int|null $externalId,
        public readonly ?string $externalUuid,
        public readonly ?string $locale,
        public readonly ?string $origin,
        public readonly ?string $originSystem,
        public readonly ?DateTimeInterface $originCreatedAt,
        public readonly string $status,
        public readonly ?DateTimeInterface $lastSyncedAt,
        public readonly ?string $lastNativeHash,
        public readonly ?string $lastExternalHash,
        public readonly ?string $errorCode,
        public readonly ?string $errorMessage,
    ) {}
}

Do

Support creation from array.

Support conversion to array.

Support immutable update methods if project conventions allow.

Do Not

Do not create database migrations.

Do not create Eloquent models.

Do not persist mappings.

Acceptance Criteria

* MappingData can be created from array.
* MappingData can be serialized to array.
* Required fields are validated.
* externalUuid is optional.
* Origin fields are optional but supported.

⸻

Ticket 0.6 — Sync Event Contract

Goal

Define neutral sync event DTOs.

DTO

final class SyncEvent
{
    public function __construct(
        public readonly string $eventId,
        public readonly string $eventName,
        public readonly string $entityType,
        public readonly string|int|null $entityId,
        public readonly ?string $entityUuid,
        public readonly string $projection,
        public readonly string $driver,
        public readonly string $operation,
        public readonly array $payload,
        public readonly SyncContext $context,
        public readonly DateTimeInterface $occurredAt,
    ) {}
}

Supported operations:

created
updated
deleted
attached
detached
synced
failed

Do Not

Do not dispatch Laravel events.

Do not implement listeners.

Do not implement queues.

Do not implement retry logic.

Acceptance Criteria

* SyncEvent can be created.
* SyncEvent validates operation.
* SyncEvent contains context.
* SyncEvent can be serialized to array.
* Tests cover invalid operation.

⸻

Ticket 0.7 — Sync Context Contract

Goal

Define context used for loop prevention and tracing.

DTO

final class SyncContext
{
    public function __construct(
        public readonly string $correlationId,
        public readonly ?string $causationId,
        public readonly string $origin,
        public readonly ?string $originSystem,
        public readonly bool $suppressReemit,
        public readonly array $metadata = [],
    ) {}
}

Required Behavior

Provide factory:

SyncContext::new(
    string $origin,
    ?string $originSystem = null,
    bool $suppressReemit = false,
    array $metadata = [],
): self

Provide helper:

shouldReemit(): bool

Do Not

Do not implement lock handling.

Do not implement event dispatching.

Do not implement queue logic.

Acceptance Criteria

* Context generates correlation ID.
* Context supports causation ID.
* suppressReemit prevents re-emission via shouldReemit().
* Tests verify loop-prevention metadata exists.

⸻

Ticket 0.8 — Projection Resolver

Goal

Resolve projections for an entity and select the correct driver.

Class

ProjectionResolver

Required Methods

forEntity(string $entity): array
projection(string $entity, string $projection): ?ProjectionDefinition
driverFor(string $entity, string $projection): ?DriverInterface

Do Not

Do not instantiate concrete drivers manually if DriverRegistry exists.

Do not hardcode projection names.

Do not throw fatal errors for disabled projections.

Acceptance Criteria

* Resolver returns enabled projections.
* Resolver ignores disabled projections.
* Resolver resolves ArrayDriver for test projection.
* Resolver returns null for missing projection.

⸻

Ticket 0.9 — Example Project Entity Fixture

Goal

Create a neutral test fixture entity config.

Example Config

return [
    'headless' => true,
    'projections' => [
        'json' => [
            'enabled' => true,
            'driver' => 'array',
            'fields' => [
                'title' => [
                    'map' => 'title',
                    'mode' => 'sync',
                ],
                'slug' => [
                    'map' => 'slug',
                    'mode' => 'sync',
                ],
                'content' => [
                    'map' => 'body',
                    'mode' => 'projection_only',
                    'scope' => 'projection',
                ],
                'computed_label' => [
                    'map' => 'label',
                    'mode' => 'derived',
                ],
            ],
        ],
    ],
    'sync' => [
        'enabled' => true,
    ],
];

Do Not

Do not use real project package code.

Do not use external systems.

Do not use database models unless existing test conventions require fixtures.

Acceptance Criteria

* Fixture loads in tests.
* Registry resolves fixture.
* Projection resolves fixture fields.
* Driver resolves fixture driver.

⸻

Ticket 0.10 — Tests

Goal

Prove the foundation works without external systems.

Required Test Areas

* Config parsing
* Entity registry
* Projection definition
* Driver registry
* Array driver
* Mapping DTO
* Sync event DTO
* Sync context DTO
* Projection resolver
* Field mode validation
* Scope validation
* Disabled projection handling

Required Test Scenario

Use dummy Project entity:

Project
  -> json projection
  -> array driver

Test flow:

1. Register Project entity config.
2. Resolve Project entity.
3. Resolve json projection.
4. Resolve array driver.
5. Create MappingData.
6. Create SyncContext.
7. Create SyncEvent.
8. Write payload to ArrayDriver.
9. Read payload from ArrayDriver.
10. Assert no external system assumptions exist.

Do Not

Do not test external systems.

Do not test HTTP routes.

Do not test queues.

Do not test database migrations.

Do not test real sync execution.

Acceptance Criteria

* Tests pass without external services.
* Tests pass without database setup unless existing package test bootstrap requires it.
* No forbidden terminology appears in implementation code, tests, comments or docs.
* Foundation can be used by future concrete drivers.

⸻

Final Acceptance Criteria for Iteration 0

Iteration 0 is complete when:

* Entity configs can declare neutral capabilities.
* Entity registry can resolve configured entities.
* Projection definitions can be parsed and validated.
* Driver interface exists.
* In-memory ArrayDriver exists for testing.
* MappingData DTO exists.
* SyncEvent DTO exists.
* SyncContext DTO exists.
* ProjectionResolver exists.
* Dummy Project entity fixture passes tests.
* No external system implementation exists.
* No database migration exists.
* No queue/job exists.
* No HTTP route/controller exists.
* No forbidden terminology exists in implementation code, tests, comments or docs.

⸻

Cursor Agent Instructions

Implement only Iteration 0.

Do not make architectural decisions beyond this spec.

Do not rename concepts.

Do not introduce concrete external systems.

Do not implement future iterations.

Do not install packages.

Do not refactor unrelated code.

When unsure, add a TODO comment and stop instead of inventing behavior.
