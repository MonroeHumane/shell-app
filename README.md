# Shell — a self-updating Android app shell

One Android app, installed once on the phone. On open it checks a small
`version.json` on GitHub for a newer APK and prompts (never forces) to
download and install it. Inside, a home screen lists mini-apps — most are
content that updates live with no APK at all; a few are native and arrive
through the APK update loop.

**Status: Phase 0 (toolchain, repo, keystore, host).** Toolchain installed,
repo + host structure done. The release keystore is **not yet created** —
it waits on the user to provide its password via the Secure Vault (see
`docs/keystore-recovery.md`). No APK exists yet; the shell itself is Phase 1.

## Decisions (confirmed 2026-11-08)

- App name: **Shell** · package: **`org.monroehumane.shell`**
- Host: **GitHub Releases** on this public repo. The APK is publicly
  downloadable by anyone with the link — accepted trade-off (see the plan).
- First mini-apps (Phase 2): dashboard (content) + token usage (native
  screen, content data served behind a Worker + token — never on a public URL).
- Check on every open, result cached 6 h; prompt only when a newer
  `versionCode` exists; "Skip this version"; download only on user tap;
  never forced, never silent.
- Full plan: `research_notes/self-updating-app-plan-20261107.md` in the
  agent workspace (not in this repo).

## Update contract (`version.json`)

Served with `Cache-Control: no-cache` from:

```
https://raw.githubusercontent.com/MonroeHumane/shell-app/main/version.json
```

```json
{
  "versionCode": 3,
  "versionName": "1.2",
  "apkUrl": "https://github.com/MonroeHumane/shell-app/releases/download/v1.2/app-release.apk",
  "sha256": "<lowercase hex SHA-256 of the APK, computed by the publish script>",
  "notes": "Shown verbatim in the update prompt; keep to 1-3 short lines.",
  "publishedAt": "2026-11-14T18:00:00Z"
}
```

`versionCode` (integer) is the only field that decides "newer".
The current `version.json` in this repo is a placeholder (`versionCode: 0`,
never newer than any installed build) until Phase 1's publish script
writes the real one as part of a release.

## Toolchain (this VM)

- JDK 17: Temurin 17.0.21 at `~/.local/jdk17` (`JAVA_HOME`).
  `apt` was unusable during setup (`apt-get update` hung with no output),
  so the Temurin tarball was used instead (the plan's documented fallback).
- Android SDK at `~/Android` (`ANDROID_HOME`): cmdline-tools 12.0,
  platform-tools 37.0.2, platforms;android-35 (rev 2), build-tools 35.0.0.
- **Known limitation:** the JVM's `HttpURLConnection` cannot fetch through
  this VM's egress proxy (it fails parsing the proxy's CONNECT response),
  so `sdkmanager` could not download packages. The SDK packages were
  installed manually with `curl` (which works through the proxy) from the
  official repository XML, SHA-1-verified, with `package.xml` files written
  so `sdkmanager --list_installed` recognises them. If `sdkmanager` ever
  needs the network again here, expect the same failure — use `curl`.
  Gradle dependency downloads in Phase 1 use a different HTTP stack and
  still need to be proven on this VM.
- Environment is persisted in `~/.bashrc` (`JAVA_HOME`, `ANDROID_HOME`,
  `PATH`).

## Signing

Release keystore: **not yet created** (waiting on the user — see
`docs/keystore-recovery.md`). Spec when created: PKCS12, RSA 2048,
30-year validity, alias `shell`, working copy at `~/.keystores/` (never
committed). Gradle reads the git-ignored `keystore.properties`
(see `keystore.properties.example`); the signing certificate's SHA-256
fingerprint will be recorded in `docs/keystore-recovery.md` (public info).

## Layout (as built in Phase 1)

```
shell-app/
  version.json               # update contract (placeholder until first release)
  keystore.properties.example
  docs/keystore-recovery.md   # WHERE the keystore backups live (no secrets)
  app/                       # Phase 1: Kotlin + Jetpack Compose, minSdk 26, targetSdk 35
  scripts/publish-release.sh # Phase 1: build -> sha256 -> version.json -> gh release create
```

**Never in this repo:** the keystore, any password, any token.
