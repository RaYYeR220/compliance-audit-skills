# Where License Metadata Lives, per Ecosystem

Loaded by `SKILL.md` Phase 2. For each package manager: which files declare dependencies, where the resolved transitive set lives, where to read each package's license, and how to tell dev/build dependencies from runtime ones.

Prefer the **lock file** for the complete transitive set. Read the **installed package metadata** for the most reliable per-package license. The manifest's own `license` field describes *your* project, not its dependencies.

---

## npm / Node.js

- **Manifest**: `package.json` — `dependencies`, `devDependencies`, `peerDependencies`, `optionalDependencies`. The top-level `license` field is *your* project's license.
- **Lock files**: `package-lock.json` (npm), `yarn.lock` (Yarn), `pnpm-lock.yaml` (pnpm). These hold the full resolved tree with versions.
- **Per-package license**: `node_modules/<pkg>/package.json` → `license` (SPDX) or legacy `licenses` array. Also a `LICENSE` file in the package dir.
- **Dev vs runtime**: `devDependencies` are not shipped to consumers of a published package, but **are** bundled into a web app build. For a published library, dev deps don't propagate; for a deployed web app, treat bundled deps as shipped.
- **Notes**: `"license": "UNLICENSED"` signals proprietary. `"private": true` prevents publishing. Browser bundles ship dependency code to users — their licenses apply even though it "feels" like SaaS.

---

## Python / PyPI

- **Manifest**: `pyproject.toml` (`[project] dependencies`, `[project.optional-dependencies]`, or tool-specific like `[tool.poetry.dependencies]`), `requirements*.txt`, `setup.py`/`setup.cfg`, `Pipfile`.
- **Lock files**: `poetry.lock`, `Pipfile.lock`, `pdm.lock`, or pinned `requirements.txt`.
- **Per-package license**: installed `*.dist-info/METADATA` → `License:` field and `Classifier: License :: OSI Approved :: ...` trove classifiers. Also a `LICENSE` in the dist-info.
- **Dev vs runtime**: dev/test groups (`[tool.poetry.group.dev]`, `optional-dependencies` like `test`/`dev`, separate `requirements-dev.txt`).
- **Notes**: License info in Python metadata is historically inconsistent; cross-check the classifier and the `License:` field. PEP 639 introduces SPDX `license`/`license-files` in newer metadata — prefer it when present.

---

## Rust / Cargo

- **Manifest**: `Cargo.toml` — `[dependencies]`, `[dev-dependencies]`, `[build-dependencies]`. Your crate's `license`/`license-file` is under `[package]`.
- **Lock file**: `Cargo.lock` — full resolved set.
- **Per-package license**: each crate's `Cargo.toml` `license` is a SPDX expression (e.g. `MIT OR Apache-2.0`, very common in Rust). Cached under `~/.cargo/registry/src/.../<crate>/Cargo.toml`.
- **Dev vs runtime**: `[dev-dependencies]` (tests/benches) and `[build-dependencies]` (build scripts) are not part of the shipped binary's runtime deps in the same way; build deps run at build time.
- **Notes**: Rust crates overwhelmingly use `MIT OR Apache-2.0` dual licensing — record that you may choose either.

---

## Go modules

- **Manifest**: `go.mod` — `require` directives (direct + indirect, the latter marked `// indirect`).
- **Lock-equivalent**: `go.sum` (hashes, not licenses) + the full module graph via the build list.
- **Per-package license**: a `LICENSE`/`COPYING` file in each module, cached under `$GOPATH/pkg/mod/<module>@<version>/`. Go modules don't carry a structured license field — you must read the file.
- **Dev vs runtime**: Go doesn't distinguish dev dependencies in `go.mod`; test-only imports still appear. Tooling like `go-licenses` can scope to what a binary actually links.
- **Notes**: Vendored deps may live in `vendor/`. Go binaries are statically linked by default — copyleft in a linked module ships inside the binary.

---

## Java / Kotlin (Maven, Gradle)

- **Manifest**: `pom.xml` (`<dependencies>`, `<scope>`), `build.gradle`/`build.gradle.kts` (`implementation`, `api`, `testImplementation`, `compileOnly`).
- **Resolved set**: Maven `dependency:tree`; Gradle `dependencies` task (cannot run here — infer from manifests + any lockfile like `gradle.lockfile`).
- **Per-package license**: declared in each artifact's POM `<licenses>` block (in the local `~/.m2/repository` or the published POM).
- **Dev vs runtime**: scopes matter — `test` and `provided`/`compileOnly` are not shipped at runtime; `compile`/`implementation`/`runtime` are.
- **Notes**: Some POMs omit `<licenses>`; fall back to the artifact's bundled license or registry data. Android apps bundle runtime deps into the APK/AAB — they ship.

---

## Ruby (Bundler)

- **Manifest**: `Gemfile` — `gem` lines, `group :development, :test` blocks.
- **Lock file**: `Gemfile.lock` — full resolved set with versions.
- **Per-package license**: each gem's `.gemspec` → `license`/`licenses`. Installed under the gems dir.
- **Dev vs runtime**: `group :development`/`:test`.
- **Notes**: Some gems leave `license` blank in the gemspec — read the bundled `LICENSE`.

---

## PHP (Composer)

- **Manifest**: `composer.json` — `require`, `require-dev`. Your project's `license` is a top-level field.
- **Lock file**: `composer.lock` — includes a `license` array per package (convenient: licenses are right in the lock file).
- **Dev vs runtime**: `require-dev`.
- **Notes**: `composer.lock` is one of the better sources — license is recorded per package directly.

---

## .NET (NuGet)

- **Manifest**: `*.csproj` (`<PackageReference>`), `packages.config` (older), `Directory.Packages.props` (central versions).
- **Lock file**: `packages.lock.json` when enabled.
- **Per-package license**: modern packages embed `<license>` (SPDX expression or file) in the `.nuspec`; older ones use `licenseUrl` (must follow the URL). Cached under the global packages folder.
- **Dev vs runtime**: `<PackageReference ... PrivateAssets="all">` and analyzer/build packages are not shipped at runtime.
- **Notes**: `licenseUrl`-only packages need the URL resolved to identify the license; flag as needs-review if unresolved.

---

## Dart / Flutter (pub)

- **Manifest**: `pubspec.yaml` — `dependencies`, `dev_dependencies`.
- **Lock file**: `pubspec.lock` — resolved set.
- **Per-package license**: a `LICENSE` file in each package; pub.dev shows the license. Cached under the pub cache.
- **Dev vs runtime**: `dev_dependencies` (tests, build, lints).
- **Notes**: Flutter apps bundle runtime packages into the app — they ship. Flutter's generated `Licenses` (the `LicenseRegistry`) can surface attributions in-app.

---

## Cross-ecosystem rules

1. **Lock file first**: it gives the complete transitive set and exact versions, which determines the actual license (critical for projects that relicensed across versions).
2. **Installed metadata beats registry guesses**: the `package.json`/`METADATA`/`.gemspec`/POM inside the installed package is the most reliable license source.
3. **Distinguish shipped from build-time**: copyleft/source-available risk hinges on whether the dependency ends up in the artifact you distribute or the service you host. Use each ecosystem's dev/test/build markers.
4. **Web and mobile bundles ship dependencies to users** even for "SaaS" — browser JS and mobile app code carry their dependencies' licenses.
5. **Multiple ecosystems in one repo**: audit each; a monorepo may have npm + Go + Python simultaneously. Attribute findings to the package path.
6. **No lock file**: audit direct dependencies only and explicitly warn that transitive dependencies were not analyzed.
