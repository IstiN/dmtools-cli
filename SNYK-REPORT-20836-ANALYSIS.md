# Snyk Scan Report 20836 — Analysis & Resolution

**Product**: EPM TEST | **Scan**: One-Time-Scan (Snyk) | **Date**: 2026-04-01  
**Scope**: dmtools-core module only. Findings in other packages are noted but not actioned.

---

## Summary

| Severity | Total | Fixed | False Positive | Other Package |
|----------|-------|-------|----------------|---------------|
| High     | 2     | 0     | 2              | 0             |
| Medium   | 16    | 1     | 15             | 0             |
| Low      | 14    | 0     | 14             | 0             |
| **Total**| **32**| **1** | **31**         | **0**         |

---

## Findings Detail

| # | ID | Severity | Type | File | Line | Status | Comments |
|---|-----|----------|------|------|------|--------|---------|
| 1 | 13975374 | High | JS/HardcodedNonCryptoSecret | `integrationTest/resources/js/dialChatViaJs.js:4` | `${secrets.DIAL_API_KEY!''}` | **False Positive** | This is a FreeMarker template expression `${secrets.KEY!''}` resolved at runtime by `JSAIClient` before JS evaluation. No secret is hardcoded. The JS itself guards against unresolved values: `if (DIAL_API_KEY.startsWith("$")) { return "Error: not replaced" }` |
| 2 | 13975375 | High | JS/HardcodedNonCryptoSecret | `main/resources/js/geminiChatViaJs.js:6` | `${secrets.GEMINI_API_KEY!''}` | **False Positive** | Same pattern as #1 — FreeMarker placeholder injected at runtime by `GeminiJSAIClient`. Not a hardcoded secret. |
| 3 | 13975376 | Medium | Java/Pt (Path Traversal) | `PropertyReader.java:210` → `AbstractRestClient.java:370` | `System.getenv` → `FileUtils.writeStringToFile` | **False Positive** | Snyk traces env var `RESPONSES_CACHE_PATH` through PropertyReader into a cache-file write. This path is admin/developer-configured infrastructure, not user-supplied tainted input. Not exploitable by an untrusted attacker. |
| 4 | 13975377 | Medium | Java/Pt | `PropertyReader.java:210` → `AbstractRestClient.java:370` | Same flow | **False Positive** | Duplicate of #3 (different Snyk data-flow path for the same code). |
| 5 | 13975378 | Medium | Java/Pt | `PropertyReader.java:210` → `AbstractRestClient.java:370` | Same flow | **False Positive** | Duplicate of #3. |
| 6 | 13975379 | Medium | Java/Pt | `PropertyReader.java:210` → `AbstractRestClient.java:370` | Same flow | **False Positive** | Duplicate of #3. |
| 7 | 13975380 | Medium | Java/Pt | `PropertyReader.java:210` → `CliExecutionHelper.java:196` | `System.getenv` → `Files.copy` | **False Positive** | Target folder path comes from admin-configured env var (e.g. `WORKSPACE_PATH`). The `targetFolder.resolve(fileName)` would be the concern, but the taint source tracked by Snyk is the folder config, not the filename from external input. Admin-controlled path. |
| 8 | 13975381 | Medium | Java/Pt | `PropertyReader.java:210` → `CliExecutionHelper.java:196` | Same flow | **False Positive** | Duplicate of #7. |
| 9 | 13975382 | Medium | Java/Pt | `JEstimator.java:76` → `JEstimator.java:98` | `args[0]` → loop var `i` | **False Positive** | `args[0]` is a JQL query string. The file path at line ~98 is `"reports/JAI_ESTIMATION_" + ticket.getKey() + ".html"` — constructed from a Jira ticket key (format `PROJECT-123`, server-validated, cannot contain `../`). Not exploitable. |
| 10 | 13975383 | Medium | Java/Pt | `JEstimator.java:76` → `JEstimator.java:141` | `args[0]` → `FileUtils.readFileToString` | **False Positive** | Same as #9 — the file read is on a hardcoded-prefix path derived from validated Jira ticket keys, not from raw `args[0]`. |
| 11 | 13975384 | Medium | Java/Pt | `JEstimator.java:76` → `JEstimator.java:141` | Same | **False Positive** | Duplicate of #10. |
| 12 | 13975385 | Medium | Java/Pt | `PropertyReader.java:210` → `MermaidIndex.java:377` | `System.getenv` → `Files.readAllBytes` | **False Positive** | Path from admin env var configuration. Same reasoning as #3. |
| 13 | 13975386 | Medium | Java/Pt | `UrlEncodedJobTrigger.java:24` → `RunCommandProcessor.java:164` | `args[0]` → `Files.readString` | **False Positive** | `UrlEncodedJobTrigger` is a CLI entry point run by admins/operators. `RunCommandProcessor` reads a config file path passed as a CLI argument. This is a CLI tool, not a web service — path traversal via CLI arguments is operator-controlled, not attacker-controlled. |
| 14 | 13975387 | Medium | Java/SSRF | `PropertyReader.java:210` → `BedrockAIClient.java:1005` | `System.getenv` | **False Positive** | AWS endpoint URL comes from admin env var (`BEDROCK_ENDPOINT`). SSRF requires attacker control of the URL; env var configuration is admin-controlled infrastructure setup. |
| 15 | 13975388 | Medium | Java/SSRF | `PropertyReader.java:210` → `BedrockAIClient.java:1005` | Same | **False Positive** | Duplicate of #14. |
| 16 | 13975389 | Medium | Java/SSRF | `PropertyReader.java:210` → `BedrockAIClient.java:1005` | Same | **False Positive** | Duplicate of #14. |
| 17 | 13975390 | Medium | Java/SSRF | `PropertyReader.java:210` → `BedrockAIClient.java:1102` | Same flow, different sink line | **False Positive** | Same reasoning as #14 — admin-configured AWS regional endpoint. |
| 18 | 13975391 | Medium | Java/SSRF | `GitHubWorkflowUtils.java:259` → `GitHubWorkflowUtils.java:270` | `conn.getHeaderField("Location")` → `new URL(url).openConnection` | **Fixed** (commit `8ff45c1`) | Real SSRF risk: HTTP 301/302 redirect Location header was used directly in `new URL(url).openConnection()` without scheme validation. A server-side redirect to `http://169.254.169.254/` (cloud metadata) was theoretically possible. **Fix**: added `https://` scheme check before following redirect in `resolveLogsRedirect()` and added pre-validation in `downloadBytes()`. |
| 19 | 13975360 | Low | Java/InsecureHash | `JiraClient.java:955` | `DigestUtils.md5Hex` | **False Positive** | MD5 used for **cache key generation** (constructing unique cache file names from URLs). Not used for password hashing or any cryptographic security purpose. MD5 is acceptable as a non-cryptographic hash/fingerprint for cache keys. |
| 20 | 13975361 | Low | Java/InsecureHash | `JiraClient.java:2570` | `DigestUtils.md5Hex` | **False Positive** | Same as #19 — cache key generation for downloaded artifacts. |
| 21 | 13975362 | Low | Java/InsecureHash | `JiraClient.java:2599` | `DigestUtils.md5Hex` | **False Positive** | Same as #19. |
| 22 | 13975363 | Low | Java/InsecureHash | `RallyClient.java:468` | `DigestUtils.md5Hex` | **False Positive** | Cache key generation. |
| 23 | 13975364 | Low | Java/InsecureHash | `FigmaClient.java:281` | `DigestUtils.md5Hex` | **False Positive** | Cache key for image/file downloads. |
| 24 | 13975365 | Low | Java/InsecureHash | `AzureDevOpsClient.java:1201` | `DigestUtils.md5Hex` | **False Positive** | Cache key generation. |
| 25 | 13975366 | Low | Java/InsecureHash | `AbstractRestClient.java:464` | `DigestUtils.md5Hex` | **False Positive** | Base class cache key from request URL + body hash. Non-cryptographic use. |
| 26 | 13975367 | Low | Java/NoHardcodedCredentials | `GitLabTest.java:153` | `"user1"` | **False Positive** | Fictional username in unit test mock data for GitLab comment author. Not a real credential. Test file only, never deployed. |
| 27 | 13975368 | Low | Java/NoHardcodedCredentials | `GitLabTest.java:160` | `"user1"` | **False Positive** | Same test, different assertion. |
| 28 | 13975369 | Low | Java/NoHardcodedCredentials | `GitLabTest.java:167` | `"system"` | **False Positive** | System-type user label in test mock. Not a credential. |
| 29 | 13975370 | Low | Java/NoHardcodedCredentials | `GitLabTest.java:187` | `"user1"` | **False Positive** | Test mock data. |
| 30 | 13975371 | Low | Java/NoHardcodedCredentials | `GitLabTest.java:194` | `"user1"` | **False Positive** | Test mock data. |
| 31 | 13975372 | Low | Java/NoHardcodedCredentials | `GitLabTest.java:201` | `"system"` | **False Positive** | Test mock data. |
| 32 | 13975373 | Low | Java/NoHardcodedCredentials | `TestRailClientTest.java:21` | `"test@example.com"` | **False Positive** | Standard test placeholder email address in unit test. Not a real credential or PII. |

---

## Fix Applied

### Finding 18 — SSRF in GitHubWorkflowUtils (commit `8ff45c1`)

**File**: `dmtools-core/src/main/java/com/github/istin/dmtools/github/GitHubWorkflowUtils.java`

**Change**:
```java
// resolveLogsRedirect() — before following redirect:
if (!location.startsWith("https://")) {
    throw new IOException("Redirect location is not HTTPS — aborting to prevent SSRF: " + location);
}

// downloadBytes() — before opening connection:
if (!url.startsWith("https://")) {
    throw new IOException("Only HTTPS URLs are permitted for artifact download: " + url);
}
```

---

## Notes

- All path traversal findings (Java/Pt, findings 3–13) are **Snyk taint-flow false positives**. The taint source in all cases is `System.getenv` in `PropertyReader.java` — meaning the paths are read from operator-controlled environment variables, not from untrusted user input.
- All SSRF findings from `PropertyReader → BedrockAIClient` are false positives for the same reason: AWS endpoint URLs are admin-configured.
- All MD5 (`InsecureHash`) findings are false positives: MD5 is used solely for **non-security cache key hashing**, not for password storage or cryptographic operations.
- All `NoHardcodedCredentials` findings in test files are false positives: `"user1"`, `"system"`, `"test@example.com"` are clearly fictional test fixtures with zero security value.
