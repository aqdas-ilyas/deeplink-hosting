# Deep-link hosting for link.getouttheretravelapp.com

These files make **Universal Links (iOS)** and **App Links (Android)** work for
trip sharing. The ROOT domain (`getouttheretravelapp.com`) is taken by the
GoDaddy website builder, which cannot serve custom files — so deep links live
on the **`link.` subdomain**, hosted free on **GitHub Pages**.

```
https://link.getouttheretravelapp.com/.well-known/apple-app-site-association
https://link.getouttheretravelapp.com/.well-known/assetlinks.json
https://link.getouttheretravelapp.com/trip/<id>            (index.html)
```

## Current status of the files

- `apple-app-site-association` — Team ID `ZC7LDM9583` + bundle ID filled in. ✅
- `assetlinks.json` — contains the **debug** keystore SHA-256. Before the Play
  Store release, ADD the Play App Signing fingerprint to the same array
  (Play Console → Setup → App integrity).
- `trip/index.html` — `REPLACE_APP_STORE_URL` still pending; the page hides the
  App Store button until it's filled in (do this once the app is live).
- `CNAME` — tells GitHub Pages the custom domain. Do not delete.

## How it's hosted (GitHub Pages)

1. A small **public** repo holds a copy of these files (public is fine — these
   files are meant to be publicly served).
2. Repo → Settings → Pages → deploy from branch `main` → custom domain
   `link.getouttheretravelapp.com` → **Enforce HTTPS**.
3. GoDaddy DNS (Domain → DNS → Add record):
   `CNAME  link  →  <account>.github.io`
4. `_headers` / `.htaccess` are only needed for Netlify/Cloudflare/Apache
   hosting — harmless to keep as a fallback option.

After any change here, copy the changed files into the hosting repo and push.

## Verify after deploying

- iOS (Apple CDN): `https://app-site-association.cdn-apple.com/a/v1/link.getouttheretravelapp.com`
- Android: https://developers.google.com/digital-asset-links/tools/generator
- Plain check: the three URLs at the top open in a browser.

## App side (already done in code)

- `link.getouttheretravelapp.com` set in `shareUtils`, navigation `prefixes`,
  the Android App Links intent-filter, and the iOS Associated Domains
  entitlement (`GetOutThere.entitlements`, wired via CODE_SIGN_ENTITLEMENTS).
- `AppDelegate.swift` forwards `open url` / `continue userActivity` to
  `RCTLinkingManager`.
- iOS caches the AASA file **at install time** — after deploying/changing it,
  delete and reinstall the app on the test device.

## What this gives you

- App **installed** → tapping a shared link opens the trip. ✅
- App **not installed** → the `/trip/<id>` page sends the user to the right store. ✅
- **Deferred** (open the exact trip *after* install) is best-effort via the
  clipboard trick in `index.html` + app-side clipboard read on first launch.
  (True guaranteed deferred needs a paid service like Branch — optional.)
