# PHP Mailbox 3.0.0 native preparation

Prepared 2026-09-16 10:48 Australia/Melbourne. Status: candidate verified and committed; independent review and ready PR pending. ROOT owns live review settlement, merge, tag, Packagist ingestion, and final public-consumer acceptance.

## Immutable inputs and exact scope

- Source: `Sendmux/sendmux-sdk@fb6085ce643c117dc3f2beb29c4fc57c303272b7`, `packages/php/mailbox/`, tree `dda0527f9a4ceb27bf404d5d6606ba5d54bdd4a4`.
- Fresh split base: `ae656e8e08bd5dfa131203cdf286fa664605f607` from `origin/main`.
- Package commit: `4bf4126aaf13dbda795b0ccbd19704a340efcc02`, branch `agent/mcp-2-release`, intended target `main`.
- Package delta: five added files, ten modified files, no deletions; 2,285 insertions and 45 deletions across exactly 15 files. This evidence is the only additional tracked file.
- All 151 package files match the immutable source except `CHANGELOG.md:3`: `Unreleased` becomes `3.0.0 (2026-09-16)`.
- Added files: `MailboxIdentityListMeta`, `MailboxQueryMeta`, `MailboxSyncMeta`, `MailboxThreadMessagesMeta`, and `MailboxThreadMessageSummaryCursorListResponse`, under `src/Model/`.
- Existing MAIN remains at `0be60731bbb9b3e06c0f6c0090fe33a578949ef3`; its user-owned `.gitignore` and `.claude/` are preserved. The unrelated auth-surface worktree was held for unshipped work; reaper dry run removed nothing.

Canonical preparation follows immutable `scripts/check-php-splits.mjs:8-87`: version-free package manifest, explicit path fixture for the candidate, real Composer resolution/install, validation, and platform checks. Public Core replaces the local Core fixture. Package source and generators were not redesigned.

## Correctness trace

`README.md:19-29` preserves the public-availability gate, 3.0/2.0 migration links, and complete persistent-cookie warning, including boolean `HostOnly`. `CHANGELOG.md:5-12` preserves the breaking changes-response migration and security notes. `composer.json:11-13` retains the approved floors: Guzzle `^7.15.5`, PSR7 `^2.13.1`, Core `^2.1.1`; no manifest `version` or repository override is introduced.

`src/Configuration.php:103,432` updates the public User-Agent/debug versions to 3.0.0. `src/Api/MailboxAPIApi.php:3587,3677,3818` selects `MailboxChangesResponse|MailboxTypedChangesResponse` for ordinary/HTTP-info/async calls; `:19080-19093` tries the two declared models and rejects a body that matches neither. The candidate check exercises both variants and invalid-body rejection through all four public call forms.

`src/Api/MailboxAPIApi.php:13149,13219,13349` selects the distinct thread-message response. Its `meta` maps to `MailboxThreadMessagesMeta`; ordinary message metadata maps to `MailboxSyncMeta`. Identity metadata maps to `MailboxIdentityListMeta`; quota/submission/thread-list metadata maps to `MailboxQueryMeta`. These mappings are declared at `src/Model/<response>.php:61-66` and exercised after installed deserialization.

`src/Model/MailboxThreadMessagesMeta.php:265-279` marks absent `request_id` or `thread_id` invalid while allowing absent optional `sync_state`. Acceptance calls the nested metadata model's public validation explicitly; this is not a claim that every response envelope recursively validates metadata automatically. The existing `MailboxThreadContentMeta` is the fifth metadata class covered by the native round-trip probe.

Budget trace: the new changes-response loop has exactly two variants, with no network or database calls inside it. Metadata adds a fixed set of fields and no persisted/per-process coordination state. Request/retry counts and payload limits are unchanged. No concurrency or latency benchmark is claimed for this package-only preparation.

## Installed verification

The isolated consumer uses a clean mirrored Mailbox candidate at fixture version 3.0.0 and exact public Core 2.1.1. Its only path repository is Mailbox; it starts without a lockfile, installs normal public dependencies, and keeps auditing enabled. PHP 8.4.10 and PHPUnit 11.5.56 were used.

| Gate | Result |
| --- | --- |
| Candidate and installed package strict validation | Both pass. |
| Consumer validation | Passes with the expected exact-version-pin warnings. |
| Normal Composer install | Passes; 38 installed packages; only Core and Mailbox are Sendmux packages. |
| Public Core provenance | `v2.1.1`, Git/ZIP reference `42769909a6c2fb34fa11657c8dce7086c00e9c25`; matches fresh Packagist metadata and all 12 public Core Git-file hashes. |
| Resolved dependency floors | Guzzle 7.15.5, PSR7 2.13.1. |
| Platform and audit | PHP plus nine required extensions pass; zero advisories and abandoned packages. |
| Unchanged PHPUnit selection | 37 tests, 217 assertions, zero errors/failures/skips. |
| Native metadata acceptance | Five metadata classes, six response mappings, empty ordinary/thread lists with absent optional state, and explicit required-thread-ID validation pass. |
| Native API acceptance | 16 checks pass: both changes variants, invalid success-body rejection, and thread response selection across sync/HTTP-info/async forms. |
| Runtime version | Installed getter and debug report both identify 3.0.0. |
| Source and installed parity | All 151 Mailbox files match source/candidate/snapshot with the dated-heading exception. |
| Autoload provenance | Four Core and nine Mailbox classes reflect to this consumer's installed vendor paths. |
| Syntax and diff | All 12 added/modified PHP files lint; diff whitespace checks pass. |

The unchanged test files are `CoreTest`, `OAuthRetryTest`, `MailboxAttachmentTest`, and `MailboxStreamArgumentsTest`, with six support classes and the source PHPUnit configuration. All 11 test/config/helper inputs match immutable source bytes. Tests: +0 / -0 tracked cases; no assertion was weakened or skipped.

## Native acceptance sensitivity

ROOT explicitly authorised small package-specific acceptance probes because no pre-existing PHP metadata test was found. These are additional native acceptance, not prior CI coverage, and are retained under `verification-inputs/`. They use public serializer/model methods and the existing Guzzle `MockHandler` transport seam; no Sendmux API or provider call occurs.

The same probes execute against the old split namespace, then against installed Mailbox. Three observed RED results each exit 255 with the expected assertion; all corresponding installed runs exit successfully:

- Version: `AssertionError: Native Mailbox version must be 3.0.0`; baseline reports `OpenAPI-Generator/2.1.0/PHP` and `2.1.0`, candidate reports 3.0.0.
- Metadata: `AssertionError: MailboxIdentityCursorListResponse must preserve metadata and empty data`; baseline strips `identity_state`, candidate preserves it. The full green run also verifies the other listed metadata boundaries.
- API: `AssertionError: Changes API must return its declared data variant`; baseline returns `MailboxGetChanges200Response`, candidate selects `MailboxChangesResponse` or `MailboxTypedChangesResponse` and preserves payloads.

These are sensitivity checks for the release's declared behavior, not claims that the old package's version was defective. The baseline run stops at the first failing assertion in each probe; it is not separate RED evidence for every later case.

## Durable receipts and resources

MAIN `.claude/artifacts/mcp-2-release/` retains all outputs. The worktree's artifact link points to that same directory.

| Receipt | SHA-256 |
| --- | --- |
| `source-installed-parity.json` | `011b58a8190fe134a9fdaa1e8b3278eb01f8f0f3134179da7681a5c45eed90aa` |
| `phpunit.log` | `1996c9f4860fff308943a1e64d4311ecd00b8f6e762469c64d8a274cff3a7540` |
| `verification-run.log` | `acb53a792dfa6327653533dd11f9660e13e81b3ada95c4b64067373b42f91a07` |
| `api-verification-run.log` | `a8690280ebb649f2bf92a2a70cbb891565d46e29dc0a88074f65f934adc3f837` |
| `consumer-composer.lock` | `4c9e25a01011c48cf005f0a6279b78c40a12ecac33578f24aa0ee4bb05038559` |

Additional receipts: `phpunit.xml`, `test-input-parity.json`, `autoload-provenance.json`, `installed-{core,mailbox}.log`, `installed-packages.json`, `consumer-composer.json`, `packagist-{core,mailbox}-before.json`, package-validation/platform/audit logs, `version-green.log`, `metadata-green.log`, and `api-green.log`. `verification-inputs/` preserves the exact acceptance probes, provenance check, execution scripts, and immutable source process helper.

`source-tree.tar.gz`, `candidate-package.tar.gz`, `baseline-package.tar.gz`, and `consumer-inputs.tar.gz` preserve reconstruction inputs; their hashes are in `cleanup.json`. Task-owned `source.KRQtq1`, `baseline.yy9A9o`, and `consumer.jlYJTY` were removed and verified absent. All 28 recorded child PIDs and process groups are absent; process helper closures and exact handles are in `process-cleanup.json`. Command session 48030 completed with exit 0. No server, browser, container, tunnel, or seeded data was created. The release worktree remains for the PR.

## Review and remaining gates

Independent pre-push review: pending. The actual diff requires `major`: public response signatures/types and generated schema classes change, including bounded model-selection logic.

Skills informed source-faithful release notes, public-seam acceptance, fixed-variant/IO/state review, generated-code restraint, provenance checks, and the scoped independent-review brief. The approved package source was not refactored or broadened. No rules-evolution candidate was found.

Pending: independent review, ready major PR, live review/check settlement, ROOT merge/tag/publication, exact public Mailbox 3.0.0 version/reference, and public-consumer acceptance. This PHP 8.4.10 candidate check does not claim a full PHP matrix, browser journey, real OAuth/provider integration, production latency result, or coordinated app/manual acceptance. Parked: none within this package delta.
