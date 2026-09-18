# argentum-privacy

Privacy policy for the **Argentum News** app (iOS and Android), hosted on
GitHub Pages.

- Live: https://luisbanasco-hub.github.io/argentum-privacy/
- **Single source: [`index.html`](index.html)** (served at the root) and
  [`account-deletion.html`](account-deletion.html). Edit those.
- [`privacy-policy.md`](privacy-policy.md) is **not** a source. It is a pointer
  left behind when the markdown copy was retired, so that an old link still
  leads somewhere. Editing it changes nothing that anyone reads.

Both apps link the live URL literally — iOS in `AboutView.swift`,
`SettingsView.swift` and `PaywallView.swift`; Android through
`BuildConfig.PRIVACY_URL` in `app/build.gradle.kts`. **The URL is part of two
published apps: do not move or rename these files.**

Update the effective date on any material change, and record what backs each
claim in [`VERIFICACION.md`](VERIFICACION.md).
