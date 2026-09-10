# Offline PDF Reader

Planning package for a Flutter Android/iOS app that prepares English and Turkish PDFs locally, then reads them aloud offline, including while the phone is locked.

No application has been implemented or validated yet. The documents below define the intended behavior and release gates; performance numbers are targets, not measured results.

## Start here

1. [Product and technical design](docs/DESIGN.md): agreed scope, screens, document processing, storage, playback, dependencies, and risks.
2. [Acceptance criteria and test cases](docs/ACCEPTANCE_AND_TESTS.md): traceable release conditions, fixtures, automated checks, and device validation.
3. [Implementation and coding-agent handoff](docs/IMPLEMENTATION.md): milestone order, bounded work packages, ownership, and review rules.

Use the iOS Simulator as the primary development/debugging environment and Android emulators for Android coverage. Complete development and available automated/integration checks without waiting for connected phones. The user will connect an iPhone 13 and Galaxy S24+ for final hardware qualification; until then, record hardware-only checks as deferred, not passed. Do not build a runtime agent swarm or introduce a cloud processing dependency.
