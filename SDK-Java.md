# SDK: Java

> **Audience:** Developers using the Java SDK (JVM-based languages: Java, Kotlin, Scala)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Java SDK reference: Maven Central coordinates, package layout, JDK target.

## What this page should contain

- Maven Central coordinates: `com.labacacia:nps-sdk` (verify) + current pin
- Minimum JDK version (currently Java 21 toolchain per build.gradle.kts)
- Package layout: `com.labacacia.nps.{ncp,nwp,nip,ndp,nop}`
- AssuranceLevel `fromWire("")` returning ANONYMOUS (uses `wire == null || wire.isEmpty()`)
- Build tool examples: Gradle Kotlin DSL, Gradle Groovy, Maven
- Kotlin interop notes
- Test framework convention (JUnit 5 / etc)

## Source material to draw from

- `labacacia/NPS-sdk-java/README.md` + `README.cn.md`
- `NPS-sdk-java/src/main/java/com/labacacia/nps/` for package layout
- `NPS-Dev/impl/java/` for upstream source
- `NPS-sdk-java/build.gradle.kts`

## Cross-links

- [SDK Quickstart](SDK-Quickstart)
- [SDK Identity and Authentication](SDK-Identity-and-Authentication)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
