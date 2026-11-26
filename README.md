# sleet-builds

Builds Sleet's two Windows apps with GitHub Actions. This repo holds only the build steps: every build takes the live code from `airwix_erp_v2` (private), so nothing here changes when that code changes. Code changes go to `airwix_erp_v2` as usual; nothing is built until a version is released.

## Releasing a version

The normal way is Sleet's master admin › **App Version Policies** › Add: choose **Sleet Tally Connector** or **Sleet Server, offline edition**, enter the version, keep **Build on GitHub** and **Publish as Current Release** on, and save. Sleet starts the build here (`appReleaseBuildService.js` in airwix_erp_v2) and the list shows its state (Queued, Building, Built, Failed) with a link to the run; **Build again** reruns it. By hand: Actions › the app's workflow › Run workflow.

| Workflow | Builds (from airwix_erp_v2) | Release |
|---|---|---|
| `tally-connector.yml` | `connector/` → `SleetTallyConnector.exe` (Go tests first) | `tally-connector-v<version>`; with publish also `tally-connector-latest` |
| `offline-server.yml` | the five parts of `installer/README.md` (app image from `deploy/offline/offline.Dockerfile`, Linux system from `installer/linux`, `postgres:16`, `caddy:2`, Microsoft's WSL package), `hashes.json`, and `installer/` → `SleetServerSetup.exe` | `offline-server-v<version>`; with publish also Setup in `offline-server-latest`. Sleet signs the download list itself (`GET /api/offline-server/manifest`) for the version marked current in App Version Policies |

Inputs: `version`, `erp_ref` (airwix_erp_v2 branch or commit, default `main`, the code on production), `publish`, `release_id` (the App Version row, filled in by Sleet; the run is named `... #<release_id>` so Sleet can find it).

The exes are built on Windows machines with Go 1.24 and Wails 2.11 (Wails 2.11 cannot read Go 1.25+). They are not code-signed, so Windows shows "Windows protected your PC" (More info › Run anyway).

## Download links

- Website download (Settings › Import): `GET /api/tally-connector/download` on the Sleet API, which sends to the current release's exe, or to `tally-connector-latest` while that release is still building.
- Fixed links: `https://github.com/DeveloperVivekPatel/sleet-builds/releases/download/tally-connector-latest/SleetTallyConnector.exe` and `.../offline-server-latest/SleetServerSetup.exe`.

## Settings

This repo (Settings › Secrets and variables › Actions):

| Name | Kind | Value |
|---|---|---|
| `ERP_REPO_TOKEN` | Secret | Classic personal access token, `repo` scope, from an account that can read `airwix_erp_v2` (a fine-grained token cannot reach a repo owned by another personal account) |
| `SLEET_CLOUD_URL` | Variable (optional) | Sleet API the apps talk to and the keys come from; default `https://app.getsleet.cloud/api` |
| `SLEET_REPORT_URL` | Variable (optional) | Where Setup's "Report to Sleet" posts; default `<SLEET_CLOUD_URL>/offline-server/report` (saved in master admin › System Logs) |

No signing key is kept here. The offline build reads the trusted public keys from Sleet (`GET /api/offline-server/trusted-keys?purpose=licence|updates`, managed in master admin › Signing Keys) and compiles them in; the private halves never leave the Sleet server.

The Sleet API (`apps/backend/.env`): `GITHUB_BUILD_TOKEN` (fine-grained token of the account that owns this repo, this repo only, Actions: read and write), optional `GITHUB_BUILD_REPO` (default `DeveloperVivekPatel/sleet-builds`) and `GITHUB_BUILD_ERP_REF` (default `main`).
