# DMTools Core — License Compliance Notes

This document records the compliance rationale for license obligations flagged by
Black Duck/Protex for the `dmtools-core` published artifact.

---

## Project License

**Apache License 2.0** — see `LICENSE` file and Maven POM `publishing {}` block in
`dmtools-cli/dmtools-core/build.gradle`.

Copyright 2023-2026 EPAM Systems, Inc.

---

## 1. CPL 1.0 — Common Public License 1.0 (JUnit 4)

**Flagged component:** `junit:junit:4.13.2`

**Why Black Duck flags CPL 1.0:** JUnit 4 originated as a CPL 1.0 project. During the
transition from CPL to EPL, some class files retained CPL 1.0 headers. Black Duck detects
both CPL 1.0 and EPL 1.0 on the `junit:junit:4.13.2` artifact.

**Scope in build.gradle:**
```groovy
testImplementation 'junit:junit:4.13.2'
```

**Is it shipped?** **NO.**
- Gradle's `runtimeClasspath` (which feeds the shadow JAR) excludes `testImplementation`.
- The published Maven POM contains no `compile` or `runtime` dependency on `junit:junit`.
- Verified by shadow JAR config: `configurations = [project.configurations.runtimeClasspath]`.

**Obligations triggered:** None. CPL 1.0 obligations (attribution, include license, disclose
source of modifications) only apply when the component is distributed. This component is not
distributed in the dmtools-core artifact.

**Protex action:** Mark CPL 1.0 obligations as **Not Applicable / Fulfilled** — component is
test-only and not included in the shipped artifact.

---

## 2. EPL 1.0 — Eclipse Public License 1.0 (JUnit 4)

**Flagged component:** `junit:junit:4.13.2`

Same component and same reasoning as CPL 1.0 above. JUnit 4.13.2 is published under EPL 1.0
as the primary license. It is `testImplementation` only and not shipped.

**Obligations triggered:** None for the same reasons.

**Protex action:** Mark EPL 1.0 obligations as **Not Applicable / Fulfilled**.

---

## 3. EPL 2.0 — Eclipse Public License 2.0 (JUnit 5 / JUnit Vintage)

**Flagged components:**
- `org.junit.jupiter:junit-jupiter:5.11.4`
- `org.junit.vintage:junit-vintage-engine:5.11.4`

**Scope in build.gradle:**
```groovy
testImplementation platform("org.junit:junit-bom:5.11.4")
testImplementation 'org.junit.jupiter:junit-jupiter'
testRuntimeOnly    "org.junit.vintage:junit-vintage-engine:5.11.4"
```

**Is it shipped?** **NO.**
- Both are in `testImplementation` / `testRuntimeOnly` scope.
- Excluded from `runtimeClasspath` and therefore from the shadow JAR and published POM.

**Usage pattern:** Dynamic linking (JVM classpath) only. No source code inclusion.

**Obligations triggered:** None. EPL 2.0 obligations (attribution, include license, state
changes, disclose source of modifications) only apply when the component is distributed.

**Protex action:** Mark EPL 2.0 obligations as **Not Applicable / Fulfilled**.

---

## 4. UPL 1.0 — Universal Permissive License 1.0 (GraalVM)

**Flagged components (shipped):**
- `org.graalvm.polyglot:polyglot:24.1.1`
- `org.graalvm.js:js-community:24.1.1`
- `org.graalvm.js:js-scriptengine:24.1.1`

**What these are:** GraalVM Community Edition Maven artifacts for the Polyglot API and
JavaScript engine. These are standalone libraries that work on any JDK 17+ — they are NOT
the GraalVM JDK distribution, which has different licensing.

**What they are used for:** JavaScript execution engine enabling AI workflow automation
(JobJavaScriptBridge, FormulaEvaluator, JSAIClient, JSPresentationMakerBridge, DMToolsBridge).

**License analysis:**
UPL 1.0 is a permissive, OSI-approved license created by Oracle.

| Aspect | UPL 1.0 Answer |
|--------|---------------|
| Commercial use | ✅ Explicitly permitted |
| Closed-source distribution to customers | ✅ Permitted — no source disclosure required |
| Modification without disclosure | ✅ Permitted — no copyleft |
| Patent rights | ✅ Explicit patent grant included |
| Copyleft | ❌ None |
| Obligation | Copyright notice only — covered by NOTICE file |

**Is it compatible with Apache 2.0?** Yes. UPL 1.0 is Apache-compatible.
See: https://oss.oracle.com/licenses/upl/

**Obligations triggered:** Attribution only — fulfilled by the NOTICE file.

**Comparison with Rhino (MPL 2.0):** UPL 1.0 is MORE permissive than Mozilla Rhino's
MPL 2.0, which carries file-level copyleft. GraalVM CE is the correct choice.

**Protex action:** Mark UPL 1.0 attribution obligation as **Fulfilled** — covered by NOTICE.

---

## 5. Oracle Java SE — False Positive

**Black Duck detection:** Scanner flags `java.*` and `javax.*` stdlib imports as "Java SE",
which may be associated with Oracle's commercial Java SE license.

**Actual runtime:** The project builds and runs exclusively on **OpenJDK** (Eclipse Temurin
distribution). This is confirmed in `.github/actions/setup-java-only/action.yml`:

```yaml
- name: Set up JDK ${{ inputs.java-version }}
  uses: actions/setup-java@v4
  with:
    distribution: 'temurin'   # Eclipse Temurin — OpenJDK distribution by Adoptium
    java-version: '23'        # default; also supports 17
```

**Temurin license:** OpenJDK is licensed under **GPL 2.0 with Classpath Exception**.
The Classpath Exception explicitly allows the JDK to be used as a runtime for proprietary
applications without triggering GPL obligations. This is fully compatible with Apache 2.0.

**Oracle Java SE license:** NOT applicable. The Oracle Java SE commercial license applies
only when using Oracle's own JDK binary. This project does not use Oracle's JDK binary.

**Protex action:** Mark Oracle Java SE as **Not Applicable** — project uses OpenJDK/Temurin.

---

## 6. BSD 3-Clause — google-auth-library

**Flagged component (shipped):** `com.google.auth:google-auth-library-oauth2-http:1.23.0`

**License:** BSD 3-Clause — permissive, no copyleft.

**Obligations:** Attribution (copyright notice) in documentation or NOTICE file.
Fulfilled by the NOTICE file.

**Protex action:** Mark BSD 3-Clause attribution as **Fulfilled** — covered by NOTICE.

---

## Summary

| Component | License | Shipped? | Action |
|-----------|---------|----------|--------|
| junit:junit:4.13.2 | CPL 1.0 / EPL 1.0 | ❌ test-only | Mark Not Applicable |
| junit-jupiter 5.11.4 | EPL 2.0 | ❌ test-only | Mark Not Applicable |
| junit-vintage 5.11.4 | EPL 2.0 | ❌ test-only | Mark Not Applicable |
| GraalVM (3 artifacts) | UPL 1.0 | ✅ shipped | Mark Fulfilled (NOTICE) |
| google-auth-library | BSD 3-Clause | ✅ shipped | Mark Fulfilled (NOTICE) |
| Oracle Java SE | — | ❌ false positive | Mark Not Applicable (OpenJDK/Temurin) |
| All other shipped deps | Apache 2.0 / MIT / Public Domain | ✅ shipped | Mark Fulfilled (NOTICE) |

All obligations for shipped components are fulfilled by the `NOTICE` file and the
`licenses/` directory in this module.

---

## Compliance Summary (for review board)

DMTools Core (com.github.istin:dmtools-core) is an Apache License 2.0 open-source library
developed by EPAM Systems, Inc. The shipped artifact includes 31 third-party components
covered by five permissive licenses — Apache 2.0 (majority), MIT, UPL 1.0, BSD 3-Clause,
and Public Domain — with zero copyleft licenses in the production artifact. The three
GraalVM Community Edition components (UPL 1.0) are Oracle's permissive open-source license,
which explicitly permits commercial use, closed-source distribution to customers, and
modification without source disclosure, identical in effect to Apache 2.0; the only
obligation is a copyright notice, satisfied by the NOTICE file. JUnit 4 (CPL 1.0 / EPL 1.0)
and JUnit 5 / JUnit Vintage (EPL 2.0) are confined to testImplementation / testRuntimeOnly
Gradle configurations and are provably absent from the shadow JAR and published POM, making
all EPL and CPL obligations inapplicable to downstream consumers. Black Duck's detection of
Oracle Java SE is a false positive: the project builds and runs exclusively on Eclipse
Temurin (OpenJDK), as confirmed by .github/actions/setup-java-only/action.yml
(distribution: 'temurin'), which is licensed under GPL 2.0 with Classpath Exception and is
fully compatible with Apache 2.0 distribution. Attribution obligations for all shipped
components are fulfilled by dmtools-cli/dmtools-core/NOTICE; license texts are provided in
dmtools-cli/dmtools-core/licenses/.
