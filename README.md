# DataBlanketLauncher

DataBlanketLauncher is a small Windows launcher/updater for the Data Blanket Unity build. It checks a hosted `manifest.json`, downloads the listed `.7z` package, verifies SHA-256, installs into a local `App` folder, and launches `Data Blanket.exe`.

## Public HTTPS or SharePoint layout

Expected files:

```text
External Sharing/
  AppManifest/
    manifest.json
    PublicAppVersions/
      DataBlanket-<version>.7z
```

Use anonymous "Anyone with the link" direct download URLs for both files. The launcher cannot use a SharePoint folder view such as `Forms/AllItems.aspx`; `manifestUrl` must return raw JSON.

## Private GitHub layout

For a private GitHub build repository, create a fine-grained personal access token with access only to `DataBlanket/DataBlanketBuilds` and repository `Contents` set to `Read-only`.

The launcher uses the GitHub REST API when `githubToken` is set:

- `GET /repos/{owner}/{repo}/contents/{manifestPath}`
- `GET /repos/{owner}/{repo}/releases/tags/{version-or-package-url-tag}`
- `GET /repos/{owner}/{repo}/releases/assets/{asset_id}` with `Accept: application/octet-stream`

The token is stored on the client machine and should be treated as recoverable. Keep it read-only and scoped only to the builds repo.

## Launcher folder layout

```text
DataBlanketLauncher.exe
DataBlanketLauncher.json
App/
Downloads/
Staging/
Backups/
Logs/launcher.log
```

`App` contains the installed Unity build. `Backups` keeps only the most recent previous app folder during an update.

## Configure

Edit the `DataBlanketLauncher.json` beside the published launcher:

```json
{
  "manifestUrl": "https://datablanket.sharepoint.com/...",
  "installDirectory": "App",
  "executableName": "Data Blanket.exe",
  "githubOwner": "DataBlanket",
  "githubRepo": "DataBlanketBuilds",
  "githubManifestPath": "manifest.json",
  "githubRef": "main",
  "githubToken": ""
}
```

For private GitHub, leave `manifestUrl` empty and set `githubToken`:

```json
{
  "manifestUrl": "",
  "installDirectory": "App",
  "executableName": "Data Blanket.exe",
  "githubOwner": "DataBlanket",
  "githubRepo": "DataBlanketBuilds",
  "githubManifestPath": "manifest.json",
  "githubRef": "main",
  "githubToken": "github_pat_..."
}
```

## Manifest

```json
{
  "version": "1.00.00.78",
  "packageUrl": "https://github.com/DataBlanket/DataBlanketBuilds/releases/download/1.00.00.78/DataBlanket.7z",
  "packageSha256": "64-character-sha256",
  "fileName": "DataBlanket-1.00.00.78.7z",
  "releaseNotes": ""
}
```

The `.7z` package must contain the Unity build contents at archive root, including `Data Blanket.exe`, `Data Blanket_Data`, `UnityPlayer.dll`, and `GameAssembly.dll`.

## Publish launcher

```powershell
.\Tools\DataBlanketLauncher\Publish-DataBlanketLauncher.ps1
```

The default output is:

```text
Tools\DataBlanketLauncher\publish
```

## Create manifest

After creating `C:\Users\Charles\Documents\GitHub\Builds\WindowsIL2CPP\DataBlanket.7z` and uploading it to SharePoint:

```powershell
.\Tools\DataBlanketLauncher\New-AppManifest.ps1 `
  -Version "1.00.00.78" `
  -PackageUrl "https://datablanket.sharepoint.com/..."
```

Upload the generated `manifest.json` to `External Sharing/AppManifest`.

## Behavior

- If the installed version is current or newer, the launcher starts `App\Data Blanket.exe`.
- If an update is available, it waits for any running `Data Blanket.exe` process to close.
- If the download hash does not match `packageSha256`, install is blocked and the current `App` folder is left untouched.
- If folder replacement fails, the launcher attempts to restore the previous `App` folder from backup.
