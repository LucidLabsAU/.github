# Releasing macOS apps

How Lucid Labs builds, signs, notarises and ships its native macOS apps
(Sitrep, TimeIT, and any future SwiftPM menu bar app) outside the Mac App
Store: GitHub Releases for direct download, and Intune for managed Macs.

## How it works

Each app repo has a small `.github/workflows/release.yml` that calls the
reusable workflow in this repo,
[`macos-app-release.yml`](../.github/workflows/macos-app-release.yml):

1. `swift test`
2. arm64 build (`swift build -c release --arch arm64`)
3. assemble `<App>.app`, stamping `CFBundleShortVersionString` from the tag
   (`v1.2.3` → `1.2.3`) and `CFBundleVersion` from the run number, with any
   app extensions (see below) in `Contents/PlugIns/`
4. sign with hardened runtime, notarise and staple, if the Developer ID secrets
   are present; ad-hoc sign and warn if they aren't
5. `<App>-<version>.zip`, plus `<App>-<version>.pkg` installing to
   `/Applications`, signed with the Installer identity and notarised when
   present
6. `SHA256SUMS.txt`
7. a GitHub Release on `v*` tags, or an Actions artefact for dry runs

Triggers in each app repo:

| Trigger | Result |
| --- | --- |
| push a `v*` tag | GitHub Release with zip, pkg and checksums |
| `gh workflow run release.yml -R LucidLabsAU/<repo> -f dry-run=true` | Actions artefact, no release |

Every run deploys to the repo's **`release` environment**. That environment
holds the signing secrets, needs approval from a required reviewer, and only
accepts `main` and `v*` tags. Approve waiting runs from the run page, under
**Review deployments**.

Signing turns on by itself: once the secrets below are on an app repo's
`release` environment, the next run signs and notarises without any workflow
change. A half-configured environment (a certificate without its password or
API key) fails the run instead of shipping something half-signed.

### App extensions (widgets)

SwiftPM has no app-extension product type, so an extension such as a WidgetKit
widget is built as an ordinary executable target, linked the way Xcode links
extensions (`-e _NSExtensionMain`, `-application_extension`). The workflow
wraps it into a bundle when the caller lists it in `app-extensions`, one per
line as `<executable> <info-plist> <entitlements>`:

```yaml
    with:
      app-extensions: |
        SitrepWidget Support/Widget-Info.plist Support/SitrepWidget.entitlements
```

Each one becomes `Contents/PlugIns/<executable>.appex`, takes the app's
version and build number (the system won't load an extension whose version
differs from its host app's), and is signed with its own entitlements before
the app is signed over it. WidgetKit only loads sandboxed extensions, so the
entitlements need at least `com.apple.security.app-sandbox`. The Info.plist
must name the executable and carry `NSExtension` → `NSExtensionPointIdentifier`;
the run checks both before building.

## Secrets

| GitHub `release` environment secret | Key Vault secret | Contents |
| --- | --- | --- |
| `DEVELOPER_ID_APP_P12_BASE64` | `apple-developer-id-app-p12` | Developer ID Application identity (.p12), base64 |
| `DEVELOPER_ID_INSTALLER_P12_BASE64` | `apple-developer-id-installer-p12` | Developer ID Installer identity (.p12), base64 |
| `P12_PASSWORD` | `apple-developer-id-p12-password` | Export password shared by both .p12 files |
| `ASC_KEY_ID` | `apple-asc-key-id` | App Store Connect API key ID |
| `ASC_ISSUER_ID` | `apple-asc-issuer-id` | App Store Connect API issuer ID |
| `ASC_KEY_P8_BASE64` | `apple-asc-key-p8` | App Store Connect API private key (.p8), base64 |

Key Vault holds the masters. GitHub gets copies, because Lucid vaults are
firewalled (`defaultAction: Deny`) and hosted runners have no fixed IP. Never
add a standing IP rule to a vault to let CI in.

## One-time setup: Apple Account Holder

Only the Account Holder of team **N3DE89M9C7** (Lucid Labs Pty Ltd) can create
Developer ID certificates. Do this on a Mac, signed in to
[developer.apple.com](https://developer.apple.com/account) as the Account
Holder.

1. **Create a signing request.** Keychain Access → Certificate Assistant →
   *Request a Certificate From a Certificate Authority*. Enter the Account
   Holder email and the common name `Lucid Labs Pty Ltd`, choose *Saved to
   disk*. One CSR can be used for both certificates.
2. **Developer ID Application.** Certificates → **+** → *Developer ID
   Application* → G2 Sub-CA → upload the CSR → download → double-click to
   install in the login keychain.
3. **Developer ID Installer.** Same again with *Developer ID Installer*.
4. **Check both identities are present:**

   ```bash
   security find-identity -v -p codesigning | grep "Developer ID Application: Lucid Labs Pty Ltd (N3DE89M9C7)"
   security find-identity -v -p basic       | grep "Developer ID Installer: Lucid Labs Pty Ltd (N3DE89M9C7)"
   ```

5. **Export the `.p12` files.** Keychain Access → *My Certificates* → expand
   each identity so the private key shows → right-click → *Export* → `.p12`.
   Use one strong password for both: that's `P12_PASSWORD`. Name the exports
   `DeveloperIDApplication.p12` and `DeveloperIDInstaller.p12`.
6. **App Store Connect API key.** [App Store Connect](https://appstoreconnect.apple.com)
   → Users and Access → Integrations → App Store Connect API → *Team Keys*
   (request API access first if prompted) → **+** → name it
   `GitHub notarisation`, access **Developer** → Generate. Download
   `AuthKey_<KEYID>.p8`; Apple only lets you download it once. Note the
   **Key ID** and the **Issuer ID** shown above the key list.

Developer ID certificates last five years. Stapled releases keep working after
expiry because the signature is timestamped. Don't revoke a Developer ID
certificate unless its key is compromised: revocation stops everything signed
with it from launching.

## Store the masters in Key Vault and copy them to GitHub

Run in zsh from the Mac that holds the exports. You need the **Key Vault
Secrets Officer** role on the vault, plus rights to change its network rules.
Sign in to the Lucid Labs tenant first and confirm the context with
`az account show`. This opens the vault to your IP, writes the masters, copies
them to both repos' `release` environments, and closes the vault again, even
if a step fails.

```zsh
VAULT="<vault-name>"               # replace with the Lucid platform Key Vault name
REPOS=(LucidLabsAU/sitrep LucidLabsAU/timer)
MYIP=$(curl -fsS https://api.ipify.org)
read -rs 'P12PW?.p12 export password: '; echo
read -r 'KEYID?ASC key ID: '
read -r 'ISSUER?ASC issuer ID: '

az keyvault network-rule add --name "$VAULT" --ip-address "$MYIP/32" -o none
{ (
  setopt err_exit pipe_fail   # stop at the first failure; the always block still closes the vault
  az keyvault secret set --vault-name "$VAULT" -o none --name apple-developer-id-app-p12       --file DeveloperIDApplication.p12 --encoding base64
  az keyvault secret set --vault-name "$VAULT" -o none --name apple-developer-id-installer-p12 --file DeveloperIDInstaller.p12  --encoding base64
  az keyvault secret set --vault-name "$VAULT" -o none --name apple-asc-key-p8                --file AuthKey_"$KEYID".p8      --encoding base64
  az keyvault secret set --vault-name "$VAULT" -o none --name apple-developer-id-p12-password --value "$P12PW"
  az keyvault secret set --vault-name "$VAULT" -o none --name apple-asc-key-id                --value "$KEYID"
  az keyvault secret set --vault-name "$VAULT" -o none --name apple-asc-issuer-id             --value "$ISSUER"

  for repo in $REPOS; do
    for pair in \
      DEVELOPER_ID_APP_P12_BASE64:apple-developer-id-app-p12 \
      DEVELOPER_ID_INSTALLER_P12_BASE64:apple-developer-id-installer-p12 \
      P12_PASSWORD:apple-developer-id-p12-password \
      ASC_KEY_ID:apple-asc-key-id \
      ASC_ISSUER_ID:apple-asc-issuer-id \
      ASC_KEY_P8_BASE64:apple-asc-key-p8
    do
      az keyvault secret show --vault-name "$VAULT" --name "${pair#*:}" --query value -o tsv \
        | tr -d '\n' | gh secret set "${pair%%:*}" --env release --repo "$repo"
    done
  done
) } always {
  az keyvault network-rule remove --name "$VAULT" --ip-address "$MYIP/32" -o none \
    || print -u2 "✗ could not remove $MYIP/32 from $VAULT: remove it by hand now"
  unset P12PW
}
rules=$(az keyvault network-rule list --name "$VAULT" --query "ipRules[].value" -o tsv) \
  && [[ $rules != *"$MYIP"* ]] \
  && print "✓ vault closed to $MYIP" \
  || print -u2 "✗ can't confirm $MYIP was removed from $VAULT: check its network rules now"
```

If a step fails, the block stops there, the vault still closes, and zsh
reports the failing command. Every command can safely run twice, so fix the
cause and run the block again.

Then keep the `.p12` files and `.p8` out of Downloads and cloud-synced folders,
or delete them; Key Vault is the copy of record. To onboard another app repo
later, create its `release` environment (required reviewer, `main` and `v*`
only) and rerun the copy loop with that repo in `REPOS`.

## Local signing and notarisation

Importing the `.p12` files (double-click) gives a Mac the signing identities.
Store notarisation credentials once, in the login keychain, under the profile
name `notary`:

```bash
xcrun notarytool store-credentials notary \
  --key AuthKey_<KEYID>.p8 --key-id <KEYID> --issuer <ISSUER_ID>
```

Each app's `Makefile` mirrors the pipeline:

```bash
make release                       # arm64, hardened runtime; ad-hoc (SIGN_ID=-) by default
make pkg INSTALLER_ID="Developer ID Installer: Lucid Labs Pty Ltd (N3DE89M9C7)"
make notarize NOTARY_PROFILE=notary \
  SIGN_ID="Developer ID Application: Lucid Labs Pty Ltd (N3DE89M9C7)" \
  INSTALLER_ID="Developer ID Installer: Lucid Labs Pty Ltd (N3DE89M9C7)"
```

`make notarize` stops straight away if `NOTARY_PROFILE` is unset or `SIGN_ID`
is still ad-hoc.

## Cutting a release

```bash
git tag -a v1.0.0 -m "Sitrep 1.0.0" && git push origin v1.0.0
```

Approve the run on the `release` environment. The release gets
`<App>-1.0.0.zip`, `<App>-1.0.0.pkg`, `SHA256SUMS.txt` and generated notes.
Tags must be numeric (`v1`, `v1.2`, `v1.2.3`); Apple and Intune reject
suffixes such as `-beta`.

Verify a download:

```bash
shasum -a 256 -c SHA256SUMS.txt
spctl --assess --type execute --verbose=2 Sitrep.app    # source=Notarized Developer ID
xcrun stapler validate Sitrep.app
pkgutil --check-signature Sitrep-1.0.0.pkg              # Developer ID Installer signature
xcrun stapler validate Sitrep-1.0.0.pkg                 # notarisation ticket stapled
spctl --assess --type install --verbose=2 Sitrep-1.0.0.pkg   # source=Notarized Developer ID
lipo -archs Sitrep.app/Contents/MacOS/Sitrep            # arm64
```

## Intune delivery (macOS line-of-business)

Use the **signed `.pkg`** from the release. Intune's macOS line-of-business
type only accepts a `.pkg` signed with a Developer ID Installer certificate,
so unsigned builds can't be delivered this way. The pipeline's pkg meets the
other requirements too: it holds one app, installs to `/Applications`, isn't
relocatable, and carries its version and `CFBundleVersion` in the package
metadata.

**Add the app** (Intune admin center → Apps → All Apps → Create → macOS →
*Line-of-business app*):

1. Upload `<App>-<version>.pkg`.
2. Name, description, publisher `Lucid Labs Pty Ltd`, minimum OS macOS 14
   Sonoma. Add a logo, because Company Portal hides LOB apps without one. The
   repo's `Support/AppIcon.icns` exported to PNG works.
3. **Ignore app version: No.** Intune then reinstalls when the deployed version
   differs, which is how updates reach Macs.
4. **Install as managed: Yes.** This works because the pkg holds a single app
   in `/Applications`. It enables the *Uninstall* assignment, and removing the
   MDM profile removes the app.

**Assign it to a group.** Assignments → *Required* for an Entra group of users
or devices (installs silently at the next check-in), or *Available for enrolled
devices* for self-service through Company Portal. *Uninstall* is only offered
because the app is managed.

**Ship a new version.** macOS LOB apps don't use supersedence. You replace the
package on the same app: the app → Properties → App information → Edit →
*Select file to update* → upload the new `.pkg` → check **App version** shows
the new number. Intune only treats it as an update if
`CFBundleShortVersionString` goes up, which a new `v*` tag guarantees. Macs
with a *Required* assignment install it at their next check-in and retry every
24 hours on failure.

Intune also has an unmanaged *macOS app (PKG)* type that accepts unsigned
packages. It isn't the path for these apps: it skips Gatekeeper's notarisation
guarantee and gives up managed uninstall.

## Beyond these apps

The same Developer ID Application and Developer ID Installer certificates, in
team N3DE89M9C7, also unblock signing and notarising the CHARLi HL7 bridge
installer. It's waiting on this same pair.
