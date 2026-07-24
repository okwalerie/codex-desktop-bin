# codex-desktop-bin (private)

automated rpm builds of [ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux)
for fedora silverblue/bluefin.

kept **private** deliberately: the rpm payload is OpenAI's proprietary Codex
Desktop app (converted from the official macOS `Codex.dmg`), which upstream
declines to redistribute publicly. these builds are for personal use.

## how it works

- `build-and-release.yml` runs every 6h: a cheap job fingerprints the upstream
  DMG (etag/last-modified/length); if it differs from the fingerprint recorded
  in the latest release body, native Fedora 44 container jobs build RPMs
  (`PACKAGE_WITH_UPDATER=0` — rpm-ostree owns updates) and a single release job
  publishes their architecture-qualified assets under a build-timestamp tag.
  x86_64 is always built. aarch64 is added automatically when an online
  self-hosted runner has the `codex-desktop` and `arm64` labels; native builds
  are required because the package payload is architecture-specific. Old
  releases are pruned (keep 3).
- `workflow_dispatch` forces a rebuild (use after upstream fixes patches for a
  new DMG that broke the build).
- a keepalive job pushes an empty commit if the branch is >25 days quiet, so
  github doesn't disable the cron after 60 days of inactivity.

## consuming on silverblue

private release assets can't be served by reprox.dev or fetched by librepo, so
the laptop runs an hourly systemd user timer (`codex-desktop-sync`) that uses a
fine-grained PAT (contents:read on this repo only) to `gh release download` the
latest rpm into a local `createrepo` dir at `/var/lib/codex-desktop-repo`;
rpm-ostree layers `codex-desktop` from that `file://` repo and picks up new
versions during normal (automatic) upgrades.

client-side tooling lives on the laptop in `~/dev/codex-desktop-silverblue/`.
