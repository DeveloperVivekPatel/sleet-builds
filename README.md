# sleet-builds

Builds Sleet's Windows apps and the offline edition image with GitHub Actions. This repo holds only the build steps: every build takes the live code from `airwix_erp_v2` (private), so nothing here has to be updated when that code changes.

## Build

Actions › **Build** › **Run workflow**, then choose:

- **What:** `tally-connector`, `server-setup`, `offline-image` or `all`
- **Version:** e.g. `1.0.0`
- **ERP branch:** the `airwix_erp_v2` branch or commit to build from (default `main`, the code on production)

| What | Comes from (in airwix_erp_v2) | Goes to |
|---|---|---|
| Sleet Tally Connector | `connector/` | Releases: `tally-connector-v<version>` and `tally-connector-latest` |
| Sleet Server Setup | `installer/` + `deploy/offline/docker-compose.yml`, `Caddyfile` | Releases: `server-setup-v<version>` and `server-setup-latest` |
| Offline image | `deploy/offline/offline.Dockerfile` | `ghcr.io/developervivekpatel/sleet-offline:<version>` and `:latest` |

The exes are built on a Windows machine with Go 1.24 and Wails 2.11 (Wails 2.11 cannot read Go 1.25+), with the connector's Go tests run first. They are not code-signed, so Windows shows "Windows protected your PC" (More info › Run anyway).

## Download links (stay the same for every build)

- Tally Connector: https://github.com/DeveloperVivekPatel/sleet-builds/releases/download/tally-connector-latest/SleetTallyConnector.exe
- Server Setup: https://github.com/DeveloperVivekPatel/sleet-builds/releases/download/server-setup-latest/SleetServerSetup.exe

The Tally Connector link is the ERP website's `VITE_TALLY_CONNECTOR_URL` (Settings › Import download button).

## Settings (Settings › Secrets and variables › Actions)

| Name | Kind | Value |
|---|---|---|
| `ERP_REPO_TOKEN` | Secret | A classic personal access token with the `repo` scope from an account that can read `airwix_erp_v2` (a fine-grained token cannot reach a repo owned by another personal account) |
| `SLEET_CLOUD_URL` | Variable (optional) | Sleet API the apps talk to; default `https://app.getsleet.cloud/api` |
| `LICENSE_PUBLIC_KEY` | Variable | Public licence key for the offline image (`node scripts/offline-license.js keys`); needed only for `offline-image` |
| `SLEET_MANIFEST_URL`, `SLEET_MANIFEST_KEY`, `SLEET_REPORT_URL` | Variables (optional) | Server Setup's download list address, its Ed25519 public key and the problem-report address (see `installer/build.sh`) |

Private keys (the licence private key, the download-list signing key) never go here.

## Known gaps

- `installer/frontend/dist` (Server Setup's screen) is not in `airwix_erp_v2` because the root `.gitignore` drops `dist`; the Server Setup build stops with that message until it is added with `git add -f installer/frontend/dist`.
- The offline image on ghcr.io starts private; make the package public (or give client PCs a pull login) before Setup pulls it.
