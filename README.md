# DataBlanketLauncher

DataBlanketLauncher is a small Windows launcher/updater for the Data Blanket Unity build. It checks a hosted `manifest.json`, downloads the listed `.7z` package, verifies SHA-256, installs into a local `App` folder, and launches `Data Blanket.exe`.

## Public GitHub layout

This public repository contains the update manifest and release packages:

```text
DataBlanketBuilds
  manifest.json
  Releases
    <version>
      DataBlanket.7z
```

The launcher reads the manifest without authentication from:

```text
https://raw.githubusercontent.com/DataBlanket/DataBlanketBuilds/main/manifest.json
```

Release packages use their public GitHub release download URLs. Do not place a GitHub token in a distributed launcher configuration.

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
  "manifestUrl": "https://raw.githubusercontent.com/DataBlanket/DataBlanketBuilds/main/manifest.json",
  "installDirectory": "App",
  "executableName": "Data Blanket.exe",
  "githubOwner": "DataBlanket",
  "githubRepo": "DataBlanketBuilds",
  "githubManifestPath": "manifest.json",
  "githubRef": "main",
  "githubToken": ""
}
```

`githubToken` must remain empty in every distributed launcher configuration.

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

After creating `C:\Users\Charles\Documents\GitHub\Builds\WindowsIL2CPP\DataBlanket.7z` and uploading it to a public GitHub release:

```powershell
.\Tools\DataBlanketLauncher\New-AppManifest.ps1 `
  -Version "1.00.00.78" `
  -PackageUrl "https://github.com/DataBlanket/DataBlanketBuilds/releases/download/1.00.00.78/DataBlanket.7z"
```

Commit the generated `manifest.json` to the `main` branch.

## Behavior

- If the installed version is current or newer, the launcher starts `App\Data Blanket.exe`.
- If an update is available, it waits for any running `Data Blanket.exe` process to close.
- If the download hash does not match `packageSha256`, install is blocked and the current `App` folder is left untouched.
- If folder replacement fails, the launcher attempts to restore the previous `App` folder from backup.
