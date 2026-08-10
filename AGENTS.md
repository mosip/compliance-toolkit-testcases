# AGENTS.md

## Repository Overview

This repository holds the MOSIP **Compliance Toolkit (CTK)** test case definitions. It is a flat data repository — three hand-maintained JSON files, no source code, no build tooling. Each file is a JSON array of test case objects for one CTK project type:

- `compliance_test_definitions_abis.json` — 32 test cases for ABIS (Automated Biometric Identification System)
- `compliance_test_definitions_sbi.json` — 200 test cases for SBI (Sensor Biometric Interface devices)
- `compliance_test_definitions_sdk.json` — 97 test cases for SDK (biometric algorithm SDKs)

These definitions describe, for a given spec method (e.g. `device`, `insert`, `check-quality`), what request/response schema to expect and which validators to run against a partner's response. They are **not** consumed directly at runtime by any application code in this repo — there is none. Per MOSIP's own documentation (`docs/ctk-test-cases.md` in the `mosip/documentation` repo), these test cases get loaded into the CTK backend's database, and `docs/ctk-setup-steps.md` describes the actual load mechanism: an operator copies the JSON array from a file here into a `saveTestCases` API request body sent to the CTK backend service ([mosip-compliance-toolkit](https://github.com/mosip/mosip-compliance-toolkit)). The files are explicitly **not** uploaded to MinIO object storage the way schema/test-data assets are.

## Technology Stack

- **Format:** JSON only. No programming language, no package manager, no build system.
- **Consumer:** the [mosip-compliance-toolkit](https://github.com/mosip/mosip-compliance-toolkit) Spring Boot service, which persists these definitions in its `mosip_toolkit` database after they're submitted via its `saveTestCases` API.

## Build & Test Commands

There is nothing to build in this repo. The only useful local check is confirming a JSON file you edited still parses. Two options that work without installing anything beyond what ships with Python or Node:

```shell
python -m json.tool compliance_test_definitions_sdk.json > /dev/null
```

```shell
node -e "JSON.parse(require('fs').readFileSync('compliance_test_definitions_sdk.json', 'utf8'))"
```

There is no `.github/workflows` directory in this repository (verified via `git ls-tree`) — no CI pipeline runs against pushes or pull requests here. JSON validity is not automatically checked by any pipeline; verify it yourself before opening a PR.

## Configuration

There are no configuration files, secrets, or environment-specific values in this repo. The JSON files contain only test case metadata (IDs, descriptions, schema names, validator names, spec attributes) — no credentials, URLs, or environment settings.

## Project Structure Notes

The repo root is the entire repo:

```text
compliance-toolkit-testcases/
├── README.md                                # single line, repo name only
├── compliance_test_definitions_abis.json    # ABIS test cases
├── compliance_test_definitions_sbi.json     # SBI test cases
└── compliance_test_definitions_sdk.json     # SDK test cases
```

A tree of per-directory `AGENTS.md` files is not warranted here — there is only one directory (root) and one kind of content (JSON test case arrays), so a single root file covers the whole repo.

Each entry in these arrays follows a common shape: `testCaseType`, `testName`, `testId` (e.g. `SBI1000`, `SDK2001`, `ABIS3000` — the numeric prefix range distinguishes the project type), `specVersion`, `testDescription`, `isNegativeTestcase`, `methodName`, `requestSchema`, `responseSchema`, `validatorDefs` (an array of validator name/description pairs), and `otherAttributes` (project-specific extra metadata such as `modalities`, `biometricTypes`, or `sdkPurpose`). Some entries also carry an `inactive` flag (and, in the SBI file only, `inactiveForAndroid` / `androidTestDescription`) to disable a test case without deleting it.

## Development Workflow

- The default integration branch is `develop`. Release branches follow `release-<version>` naming (e.g. `release-1.4.0`), and there is also a `rel14x` branch used for that release line.
- There is no local dev environment to run — changes are direct edits to the JSON arrays.
- After editing, validate the file still parses as JSON (see Build & Test Commands) and check that `testId` values you add don't collide with existing ones in the same file.

## Pull Request Guidelines

- Target the `develop` branch unless the change is a backport or fix specifically for a release branch.
- Keep changes scoped to the file(s) for the project type you're changing (ABIS, SBI, or SDK) — avoid unrelated edits across files in the same PR.
- Sign off commits (`git commit -s`) per MOSIP contribution conventions.
- Because there is no CI here to catch a malformed file, double-check JSON validity locally before pushing (see Build & Test Commands).

## Repository-Specific Considerations

- Editing one of these JSON files does not, by itself, change CTK's behavior for anyone — a maintainer has to submit the updated array to the CTK backend's `saveTestCases` API for it to take effect (see `docs/ctk-setup-steps.md` in `mosip/documentation`). Mention this in the PR description if the change is meant to reach a running CTK instance.
- `testId` values are meaningful: the numeric prefix ranges are already partitioned by project type (`SBI1xxx`+, `SDK2xxx`+, `ABIS3xxx`+ based on current content) — follow the existing convention rather than picking an arbitrary new ID.
- `requestSchema` / `responseSchema` values are names, not paths — the actual JSON Schema files they refer to live in the `mosip-compliance-toolkit` repo's `resources/` folder, not here. Renaming a schema reference here without a matching schema on that side will break the test case.
- The three files are independent — there's no shared/common section between them, so a validator or schema name used in one file has no effect on the others.

## Agent rules

### Do

1. Validate any JSON file you edit actually parses before proposing or committing a change.
2. Keep edits to the file that matches the project type (ABIS, SBI, or SDK) you're actually changing.
3. Follow the existing `testId` numeric-range convention for the project type when adding a new test case.
4. Note in the PR description if a request/response schema name is being introduced or renamed, since the schema itself lives in a different repo.
5. Sign off commits (`git commit -s`) and target the `develop` branch.

### Do not

1. Do not invent or assume a build/CI process for this repo — there is none; do not claim CI validates JSON here.
2. Do not add source code, build files, or application logic — this repo is data-only by design.
3. Do not remove a test case to "clean up"; if a test case should stop running, prefer setting its `inactive` flag, consistent with existing entries.
4. Do not assume changes here take effect in a running CTK instance automatically — they don't, without a separate `saveTestCases` submission.
5. Do not touch files outside this repo's test-definition JSON files (e.g. this task never requires touching `.idea/` or unrelated local state).
