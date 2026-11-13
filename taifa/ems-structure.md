# Taifa EMS Repository Structure

This guide describes the `taifa-ems/` repository in this workspace. The codebase is a modular OpenEMS/FEMS energy-management system: Java/OSGi bundles provide Edge and Backend functionality, `ui/` contains the Angular/Ionic application, and `doc/` contains the Antora documentation site.

## Repository map

```text
taifa-ems/
├── io.openems.backend.*       Backend services and the Backend application
├── io.openems.common*         Shared APIs, utilities, bridges, and tariff modules
├── io.openems.core.*          Cross-cutting runtime support
├── io.openems.edge.*          Edge runtime, device integrations, and control logic
├── io.openems.oem.*           OEM-specific modules
├── io.openems.shared.*        Shared infrastructure modules
├── ui/                        Angular/Ionic web and mobile UI
├── doc/                       Antora documentation sources and build
├── cnf/                       Bnd workspace and OSGi dependency configuration
├── gradle/                    Gradle wrapper and convention/build logic
├── tools/                     Packaging, deployment, release, and development scripts
├── .github/                   CI workflows, contribution material, and agent skills
├── build.gradle.kts           Root Gradle build and shared build tasks
├── settings.gradle.kts        Gradle plugin setup and included application projects
├── gradle.properties          Java, plugin, and build settings
├── gradle.lockfile            Locked Gradle dependency versions
├── gradlew / gradlew.bat      Gradle wrapper launchers
├── AGENTS.md                  Repository-specific development guidance
└── README.md                  Project overview and links to documentation
```

The Java workspace contains roughly 240 `io.openems.*` bundle directories. Most bundles are independently organized OSGi modules rather than conventional subpackages of one Java application.

## Main code areas

### Backend: `io.openems.backend.*`

Backend bundles run server-side services that connect and manage Edge systems. Examples include:

- `io.openems.backend.application/` — Backend application assembly and its `.bndrun` runtime definition.
- `io.openems.backend.edge.manager/`, `io.openems.backend.edge.application/` — Edge management and combined Backend/Edge app assembly.
- `io.openems.backend.authentication.*`, `io.openems.backend.b2b*`, `io.openems.backend.uiwebsocket/` — Authentication and client-facing communication.
- `io.openems.backend.metadata.*`, `io.openems.backend.timedata.*`, `io.openems.backend.tariff*` — Metadata, time-series data, and tariff services.
- `io.openems.backend.alerting/`, `io.openems.backend.metrics.prometheus/`, `io.openems.backend.simulator/` — Alerting, metrics, and simulation support.

### Edge: `io.openems.edge.*`

Edge bundles run on the site controller. The `io.openems.edge.*` family includes shared Edge APIs and runtime bundles as well as many focused integrations and applications. Common module families are:

- `battery.*`, `batteryinverter.*`, `meter.*`, `evse.*`, `heatpump.*`, and other device bundles for hardware integrations and device APIs.
- `bridge.*` for communication protocols such as HTTP, Modbus, MQTT, and M-Bus.
- `controller.*` for energy-management and control algorithms.
- `application/` for the Edge application assembly.
- `common/` and `api` bundles for shared Edge runtime concepts and interfaces.

### Shared Java modules

- `io.openems.common/` and the adjacent `io.openems.common.*` bundles hold APIs and utilities reused across Edge and Backend.
- `io.openems.core.*` provides cross-cutting runtime support.
- `io.openems.oem.*` and `io.openems.shared.*` contain OEM-specific and shared infrastructure code.

### Typical Java bundle layout

Most bundles follow this pattern:

```text
io.openems.<area>.<module>/
├── bnd.bnd
├── src/io/openems/.../       Production Java source
├── test/io/openems/.../      Java tests
├── resources/                Bundle resources (when needed)
└── doc/                      Module-specific documentation (when needed)
```

OSGi components are assembled from these bundles. Look at a nearby bundle in the same family before adding a new one; its `bnd.bnd`, package layout, component configuration, and tests are usually the best local examples.

## UI: `ui/`

The UI is an Angular/Ionic application with Capacitor Android and iOS projects.

```text
ui/
├── src/
│   ├── app/
│   │   ├── edge/             Edge live view, history, installation, schedule, settings
│   │   ├── index/            Index and overview flows
│   │   ├── shared/           Reusable components, services, JSON-RPC, types, utilities
│   │   ├── user/             User-related UI
│   │   └── changelog/        Changelog views
│   ├── assets/               Shared assets, translations, images, and JSON
│   ├── environments/         Environment configuration
│   └── themes/               Theme-specific assets, styles, and environment overrides
├── android/                  Capacitor Android project
├── ios/                      Capacitor iOS project
├── resources/                Mobile app icons and splash assets
├── eslint-rules/             Custom lint rules
├── scripts/                  UI helper scripts
├── angular.json              Angular project and build/serve configuration
└── package.json              UI dependencies and npm scripts
```

Generic UI features belong under `src/app/`; reusable frontend code belongs under `src/app/shared/`. Branding and theme-specific behavior belong under `src/themes/`. `ui/AGENTS.md` documents the UI-specific conventions and validation commands.

## Build, runtime, and documentation support

- **Gradle:** `settings.gradle.kts` configures plugins and explicitly includes the Backend app, combined Backend/Edge app, and documentation projects. The Bnd workspace plugin manages the modular bundle projects. `build.gradle.kts` defines common compilation, testing, quality, and application build tasks.
- **Bnd/OSGi:** `cnf/` contains workspace configuration, Checkstyle and formatter settings, and the Maven dependency declaration used for the OSGi runtime. App `.bndrun` files define runtime assemblies.
- **Gradle build logic:** `gradle/build-logic/` contains repository-specific Gradle conventions and plugins.
- **Documentation:** `doc/` is an Antora module. Content is under `doc/modules/ROOT/pages/`; images are under `doc/modules/ROOT/assets/images/`.
- **Tools:** `tools/` contains scripts and support files for packaging, deployment, Docker, release preparation, and integration tests.
- **Automation:** `.github/` contains repository workflows and development guidance.

## Where to start

| If you need to… | Start here |
|---|---|
| Change a device integration | Find its `io.openems.edge.*` bundle; inspect `src/` and `test/` |
| Change an Edge control algorithm | Find the related `io.openems.edge.controller.*` bundle |
| Change Backend behavior | Find the relevant `io.openems.backend.*` service bundle |
| Change a shared API or utility | Check `io.openems.common/` and the relevant `io.openems.common.*` module |
| Change a UI screen or reusable feature | `ui/src/app/`; shared UI code is in `ui/src/app/shared/` |
| Change branding or a themed environment | `ui/src/themes/` |
| Change docs | `doc/modules/ROOT/pages/` |
| Change app assembly or runtime dependencies | The app `.bndrun` file and `cnf/` configuration |

## Development notes

- Repository guidance is in `taifa-ems/AGENTS.md`; frontend-specific guidance is in `taifa-ems/ui/AGENTS.md`.
- The configured Java source and target versions are 25 (`gradle.properties`).
- Use the Gradle wrapper (`./gradlew` on Unix-like systems, `gradlew.bat` on Windows) and run the narrowest relevant module task.
- UI lint and tests are managed from `ui/` through the npm scripts in `ui/package.json`.
