# Fix browserslist CVE-2026-73088 security alert

## Goal
Resolve the GitHub Dependabot/security alert for `browserslist` (CVE-2026-73088) in `package-lock.json` and ensure the project no longer ships the vulnerable version.

## Current state
- `package-lock.json` pins `browserslist@4.24.2`.
- `bun.lockb` (binary) is also present, so the repo has two lockfiles.
- `browserslist` is a transitive dependency of `autoprefixer` → `postcss`.
- Latest patched `browserslist` is `4.28.9`.

## Plan

1. **Convert binary Bun lockfile to text**
   - Run `bun install --save-text-lockfile` to produce a readable `bun.lock`.
   - Remove the old binary `bun.lockb` so scanners and GitHub can inspect the lockfile.

2. **Upgrade browserslist and its parent dependencies**
   - Update `autoprefixer` and `postcss` to their latest compatible versions, which will pull a patched `browserslist`.
   - If the transitive update alone does not clear the alert, add an `overrides` entry in `package.json` to force `browserslist` to the patched version.

3. **Regenerate `package-lock.json`**
   - Run `npm install` (or `npm update browserslist`) so `package-lock.json` is in sync with `package.json` and contains the fixed version.

4. **Verify the vulnerable version is gone**
   - Search both lockfiles to confirm no `browserslist` version older than the patched one remains.
   - Run the project build (`vite build`) to ensure the dependency update does not break compilation.

5. **Close the security finding**
   - If a persisted finding exists in the Lovable security dashboard, mark it as fixed after the lockfiles are updated and the build succeeds.

## Outcome
The repository will contain only text lockfiles, `browserslist` will be on a CVE-free version, and the GitHub security alert will be resolved.
