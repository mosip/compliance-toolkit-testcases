# AGENTS.md

## Repository Overview

MOSIP **Compliance Toolkit (CTK)** test case definitions — a flat data repo, no source/build tooling. Three JSON arrays, one per CTK project type:

- `compliance_test_definitions_abis.json` — 32 cases, ABIS
- `compliance_test_definitions_sbi.json` — 200 cases, [SBI](https://docs.mosip.io/develop/biometrics/secure-biometric-interface) (biometric capture device protocol)
- `compliance_test_definitions_sdk.json` — 97 cases, biometric algorithm SDKs

Not consumed at runtime by anything in this repo. An operator copies a file's array into the CTK backend's (`mosip-compliance-toolkit`) `saveTestCases` API to load it — see [CTK setup guides](https://docs.mosip.io/compliance-tool-kit/how-to-guides/how-to-set-up-ctk). Not uploaded to MinIO like schema/test-data assets.

## Technology Stack

JSON only — no language, package manager, or build system. Consumer: `mosip-compliance-toolkit` (Spring Boot), persists to its `mosip_toolkit` DB via `saveTestCases`.

## Build & Test Commands

No build. Only check: the edited file still parses as JSON (syntax only, no schema check) — **no CI runs here** (no `.github/workflows`), so do this yourself before every PR:

```shell
python -m json.tool compliance_test_definitions_sdk.json > /dev/null  # swap in the file you edited
```

## Configuration

None — no secrets, no env-specific values. Files hold only test-case metadata.

## Project Structure Notes

```text
compliance-toolkit-testcases/
├── compliance_test_definitions_abis.json
├── compliance_test_definitions_sbi.json
└── compliance_test_definitions_sdk.json
```

Entry shape: `testCaseType`, `testName`, `testId` (e.g. `SBI1000`, `SDK2001`, `ABIS3000` — prefix = project type), `specVersion`, `testDescription`, `isNegativeTestcase`, `methodName`, `requestSchema`, `responseSchema`, `validatorDefs` (name/description pairs), `otherAttributes` (project-specific: `modalities`, `biometricTypes`, `sdkPurpose`). `inactive` flag disables a case without deleting it (SBI also has `inactiveForAndroid`/`androidTestDescription`).

## Development Workflow

Default branch `develop`; release branches `release-<version>` (e.g. `release-1.4.0`, `release-1.4.x`). No dev environment — direct JSON edits. Validate JSON, and check the new `testId` is unique across all three files (they share one backend collection).

## Pull Request Guidelines

Target `develop` (unless a release-branch backport). Scope a PR to one project type's file. Sign off commits (`git commit -s`).

## Repository-Specific Considerations

- A JSON edit alone changes nothing live — a maintainer must submit it via CTK's `saveTestCases` API (`CTK_ADMIN` role required; never commit tokens). Mention in the PR if it should reach a running instance.
- `testId` prefix ranges are partitioned by type (`SBI1xxx`+, `SDK2xxx`+, `ABIS3xxx`+) — follow them, don't invent new ranges.
- `requestSchema`/`responseSchema` are names, not paths — actual schemas live in `mosip-compliance-toolkit`'s `resources/`. Renaming here without a matching change there breaks the case. Each project type's names are its own (no cross-file reuse).
- Validator names (e.g. `SchemaValidator`) are shared across all three files, mapping to one CTK backend implementation — not file-local.

## Agent rules

### Do

1. Validate JSON before proposing/committing a change.
2. Scope edits to one project type's file; follow its `testId` range for new cases.
3. Note schema renames/additions in the PR (schema lives in the other repo).

### Do not

1. Assume any CI/build process exists here, or add source/build files.
2. Delete a test case to disable it — set `inactive` instead.
3. Assume a change is live without a separate `saveTestCases` submission.
4. Touch unrelated files (e.g. `.idea/`, local state).
