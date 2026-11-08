# Keystore recovery — WHERE the backups live (no secrets in this file)

**Status (2026-11-08, Phase 0): the release keystore has NOT been created yet.**
It waits on one user step: provide the keystore password via the Secure Vault.
Everything else in Phase 0 is done. Do not cut a release build until the
keystore below exists and is backed up in both places.

## Spec (when created)

- File (working copy): `~/.keystores/shell-release.keystore`
- Type: PKCS12 · algorithm: RSA 2048 · validity: 30 years (10,950 days)
- Alias: `shell`
- Passwords: keystore password + key password, both stored in the Secure Vault
  (never in this repo, never in chat, never in any plan doc).

## Generation command (run once the password exists)

```sh
export JAVA_HOME="$HOME/.local/jdk17"
mkdir -p ~/.keystores
"$JAVA_HOME/bin/keytool" -genkeypair -v \
  -keystore ~/.keystores/shell-release.keystore \
  -storetype PKCS12 -keyalg RSA -keysize 2048 -validity 10950 \
  -alias shell
```

`keytool` prompts for the keystore password, the key password, and the
certificate details (name / organisation / city / state / country — the
user's own details; they go into the public certificate). Re-enter the
Secure Vault password at the prompt; do not paste it into any file.

Verify (shows alias + validity dates only — never the password):

```sh
"$JAVA_HOME/bin/keytool" -list -v -keystore ~/.keystores/shell-release.keystore -alias shell
```

## Backups — two places minimum (non-negotiable)

1. **Muse Secure Vault** — the keystore FILE plus both passwords, stored as
   secrets. Status: _pending — the keystore does not exist yet._
2. **An offline backup the user holds** — the user copies
   `~/.keystores/shell-release.keystore` to an encrypted USB drive or their
   password manager's file storage. Status: _pending._

## Signing certificate SHA-256 fingerprint

_To be recorded here after generation (public information; also what
developer-verification registration needs)._

```
SHA-256: (pending keystore creation)
```

## Why this matters

Android upgrades only when package name **and** signing key match. A lost
keystore means the installed app can never be updated — the only recovery
is uninstall + reinstall, which wipes the app's data. There is no reset,
no recovery, no support ticket. The release pipeline (Phase 1) verifies at
build time that the APK was signed with the fingerprint above, so a
wrong-keystore build can never ship.
