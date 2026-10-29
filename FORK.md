# Fork notes: Clash Party + mihomo-oix

This fork builds with [mihomo-oix](https://github.com/vernesong/mihomo-oix) in the
default core slot (`mihomo` internally, shown as `OIX`). The `Alpha`, `Smart`, and
`Specific Version` slots are unchanged.

## What changed

- `scripts/prepare.mjs`: the default `mihomo` core is downloaded from the
  `vernesong/mihomo-oix` rolling `Pre-Alpha` release instead of
  `MetaCubeX/mihomo` stable.
- Core labels: the `Mihomo` option is shown as `OIX` (all locales).
- `src/main/resolve/autoUpdater.ts`: app updates follow this fork, not upstream;
  GitHub Actions dev builds (`-d...`) skip update checks entirely so an upstream
  release cannot silently replace the custom core.
- `.github/workflows/build.yml`: macOS signing/notarization is optional. Without
  Apple secrets the workflow builds unsigned packages; with secrets it signs and
  notarizes as before. Telegram notifications are skipped when no token is set.

`mihomo-oix` keeps the mihomo CLI (`-d`, `-f`, `-t`, `-ext-ctl-unix`,
`-ext-ctl-pipe`, `-post-up`, `convert-ruleset`) and REST API, so the rest of the
app works unchanged. OIX-specific managed-subscription features are opt-in via
`OIX_TOKEN` / `OIX_PROFILE`; they are not exposed by this GUI. Do not set
`OIX_PROFILE=1` if you want Clash Party to keep managing the profile, and unset
`OIX_TOKEN` if you see an OIX-managed provider appearing unexpectedly.

## Build with GitHub Actions

1. Push the branch to `cospotato/clash-party`.
2. Open **Actions → Build → Run workflow**, select the branch, run it.
3. Each platform job uploads the usual artifacts to the `dev` prerelease:
   - Windows: `*setup.exe`, `*portable.7z`, checksums
   - Linux: `.deb`, `.rpm`, `.pkg.tar.zst`, checksums
   - macOS: `.pkg`, checksums (unsigned unless Apple secrets are configured)
4. You can also download the per-job artifacts from the workflow run page.

A tag push matching `v*` runs the release path instead of the `dev` path.
Auto-update checks in this fork point at `cospotato/clash-party`; dev builds do
not auto-update. If you publish releases, upload `latest.yml` (the `updater` job
does this) to the latest release.

### Optional Apple signing

Without these repository secrets the macOS jobs still build, but the `.pkg` is
unsigned and Gatekeeper will require right-click → Open (or
`xattr -dr com.apple.quarantine`). To sign/notarize, add:

- `CSC_LINK`, `CSC_KEY_PASSWORD` — Developer ID Application `.p12`
- `CSC_INSTALLER_LINK`, `CSC_INSTALLER_KEY_PASSWORD` — Developer ID Installer `.p12`
- `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID` — notarization
- optional `CSC_INSTALLER_NAME` if the installer identity is not auto-detected

Windows binaries are unsigned (the workflow has no Windows code-signing step);
macOS packages are unsigned unless the Apple secrets above are configured. Expect
SmartScreen/Gatekeeper warnings.

## Updating the oix core

The default core is pinned to the rolling `Pre-Alpha` release:

```text
https://github.com/vernesong/mihomo-oix/releases/download/Pre-Alpha/version.txt
```

Every `pnpm prepare` / CI build downloads the current oix build. To pin a fixed
version, point the `MIHOMO_VERSION_URL` / `MIHOMO_URL_PREFIX` constants in
`scripts/prepare.mjs` at your own release and keep the asset names compatible
with `MIHOMO_MAP` (the Windows zip must contain
`mihomo-windows-<arch>.exe`; the Unix asset is a gzip stream).

The in-app “upgrade core” button calls the core's `/upgrade`; mihomo-oix points
its alpha update channel at its own `Pre-Alpha` release, so it stays on oix.
On packaged installs the core directory may be root-owned; in that case the
self-update fails and keeps the existing binary (same limitation as the upstream
“Specific Version” installer).

## OIX licensing / proprietary notice

The mihomo-oix repository's README states GPL-3.0, but its `component/oix`
package is marked proprietary/confidential and reverse engineering is
prohibited. Personal builds are one thing; **do not redistribute a bundled oix
binary without confirming permission with the oix author**. This fork only
consumes the published release binary and does not modify or analyze the oix
component.

## Useful commands

```bash
pnpm install
pnpm prepare
pnpm build:win    # or build:mac / build:linux
pnpm test
```
