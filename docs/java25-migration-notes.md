# Java 25 / Jakarta Migration Notes for `blue-ws`

## Current Status
- `pom.xml` targets Java 25 (`maven.compiler.release=25`) and the build succeeds on JDK 25.
- Core dependencies now point to the Jakarta artifacts (`jakarta.xml.bind-api`, `jakarta.validation-api`, `jakarta.activation-api`).
- `org.codehaus.mojo:jaxb2-maven-plugin` 3.1.0 is defined in `<pluginManagement>` and reused by each module execution.
- `jaxb2-basics` / `jaxb2-basics-annotate` rely on the `1.11.1-PUBLISHED-BY-MISTAKE` build; `jaxb-runtime` 2.3.5 is layered in to provide the legacy APIs the extension requires.
- All `.xjb` binding files use the Jakarta namespace (`https://jakarta.ee/xml/ns/jaxb`), Jakarta validation annotations, and drop the deprecated `extensionBindingPrefixes` attribute so they validate cleanly.
- The existing `exec-maven-plugin` steps still compile the generated sources with `find`/`javac`; these run successfully on JDK 25.

## Outstanding Issues
- Maven still prints warnings from its bundled Jansi/Guava libraries (`System::load` / `sun.misc.Unsafe`). These originate upstream and do not affect the build outcome.
- We remain dependent on the `jaxb2-basics` extension for `-Xannotate`; monitor for a maintained Jakarta-native alternative in the medium term.

## Next Steps
1. Track the `jaxb2-basics` project for an official Jakarta-compatible release or evaluate alternative annotation tooling.
2. Investigate replacing the `exec-maven-plugin` compilation step with the standard `maven-compiler-plugin` once the generation pipeline stabilises.
3. Coordinate with consuming services (e.g., `blue-backend`) to adopt the regenerated Jakarta DTOs and verify runtime behaviour.

## Testing Matrix
- Use Scoop to swap JDKs when validating the migration:
  - `scoop reset openjdk11` (baseline)
  - `scoop reset openjdk17` (intermediate tests)
  - `scoop reset openjdk25` (target runtime)
- Confirm Maven picks up the selected runtime via `mvn -version` before running builds.

## Open Questions
- Do we still need the `-Xannotate` plugin? Audit generated sources to confirm the annotations remain necessary for downstream validation.
- Should the DTO assemblies move to a `2.0.0` line to signal the Jakarta namespace shift to clients?
- Can we simplify packaging by compiling generated sources with Maven instead of the custom `exec` step?
