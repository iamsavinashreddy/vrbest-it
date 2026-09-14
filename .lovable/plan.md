# Fix GitHub security alerts (browserslist, brace-expansion, js-yaml x2, PostCSS, Vite)

## Goal
Resolve all open GitHub Dependabot alerts in one pass by upgrading the vulnerable packages and regenerating the lockfiles.

## Vulnerabilities found in the lockfile

| Package | Current | Latest | Issue |
|---|---|---|---|
| browserslist | 4.24.2 | 4.28.9 | CVE-2026-73088 — crash/prototype write via custom stats |
| brace-expansion | 1.1.11 | 5.0.9 | DoS via exponential-time expansion |
| js-yaml | 4.1.0 | 5.4.2 | Merge-key chains cause quadratic CPU use |
| postcss | 8.4.47 | 8.5.28 | Arbitrary file read via sourceMappingURL in CSS comments |
| vite | 5.4.10 | 8.3.0 | server.fs.deny bypass on Windows alternate paths |

`browserslist`, `brace-expansion`, and `js-yaml` are pulled in transitively (via `autoprefixer`, `eslint`, etc.). `postcss`, `vite`, and `autoprefixer` are direct devDependencies.

## Plan

1. **Convert binary Bun lockfile to text**
   - Run `bun install --save-text-lockfile` to produce a readable `bun.lock`, and remove the old binary `bun.lockb` so GitHub and scanners can inspect it.

2. **Upgrade direct dependencies in package.json**
   - Bump `postcss`, `autoprefixer`, and `vite` (plus related tooling such as `@vitejs/plugin-react-swc` if needed) to their latest compatible versions.

3. **Force-patched versions for transitive packages**
   - Add an `overrides` block in `package.json` pinning `browserslist`, `brace-expansion`, and `js-yaml` to patched versions so nested dependencies cannot reintroduce the vulnerable ones.

4. **Regenerate package-lock.json**
   - Run a fresh `npm install` so `package-lock.json` is in sync and free of the flagged versions (this also keeps your GitHub Actions `npm ci` step working).

5. **Verify**
   - Confirm none of the vulnerable versions remain in either lockfile.
   - Run `vite build` to ensure the upgrades (especially the Vite major bump) do not break the site.
   - If the Vite major upgrade breaks the build, fall back to the latest patched 5.x/6.x line instead and note it.

## Outcome
All five GitHub security alerts will be resolved, both lockfiles will be text-based and scanner-friendly, and the site will still build and deploy to GitHub Pages/AWS S3 as before.
