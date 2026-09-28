# SteamCloudInspector v1.1.0

Windows x64 console utility for inspecting, downloading, backing up, deleting, restoring, and fully resetting Steam Cloud data through the native `ISteamRemoteStorage` interface.

The default AppID is **730** for Counter-Strike 2.

This is an independent utility and is not affiliated with Valve.

## Main commands

```bat
SteamCloudInspector.exe status
SteamCloudInspector.exe list
SteamCloudInspector.exe download [output-directory]
SteamCloudInspector.exe inspect "cloud/file/name"
SteamCloudInspector.exe delete "cloud/file/name" --confirm DELETE-730
SteamCloudInspector.exe delete-all --confirm DELETE-ALL-730
SteamCloudInspector.exe full-reset --confirm FULL-RESET-730
SteamCloudInspector.exe restore <backup-directory> --confirm RESTORE-730
SteamCloudInspector.exe local-list
SteamCloudInspector.exe local-copy [output-directory]
SteamCloudInspector.exe archive-local --confirm ARCHIVE-730
```

Global options:

```text
--appid <id>            Default: 730
--accountid <steam3>    Override the active Steam3 account ID
--steam-api <dll-path>  Override steam_api64.dll location
```

## Full reset

The final release adds a complete reset workflow:

```bat
SteamCloudInspector.exe full-reset --confirm FULL-RESET-730
```

It performs the following sequence:

1. Verifies that CS2 is not running.
2. Creates a timestamped backup package under `ResetBackups`.
3. Downloads every Steamworks-visible Cloud file.
4. Copies the entire local `userdata\<Steam3ID>\730` directory.
5. Deletes all enumerated Cloud files through `ISteamRemoteStorage::FileDelete`.
6. Verifies that the API now enumerates zero files.
7. Waits for `cloud_log.txt` to confirm every remote deletion and the completed upload batch.
8. Shuts down the Steamworks context.
9. Waits for the user to exit Steam normally.
10. Renames the active local folder to:

```text
730_PRE_FULL_RESET_YYYYMMDD_HHMMSS
```

The tool does not forcibly terminate Steam and does not permanently delete the local AppID folder.

A successful reset ends with:

```text
[OK]   FULL RESET COMPLETED SUCCESSFULLY.
```

## Backup layout

A full reset creates:

```text
ResetBackups\App730_full_reset_YYYYMMDD_HHMMSS\
    Cloud\
        manifest.json
        files\
    Local\
        local\
        remote\
        remotecache.vdf
    RESET_INFO.txt
```

Cloud manifests include the tool name, version, and author.

## Manual local archive

Use this only when the Cloud reset succeeded but the automatic local rename could not complete:

```bat
SteamCloudInspector.exe archive-local --confirm ARCHIVE-730
```

When Steam is already closed and the active account cannot be read from the registry, specify the Steam3 account ID:

```bat
SteamCloudInspector.exe archive-local --accountid 107###### --confirm ARCHIVE-730
```

## Download and inspect

```bat
SteamCloudInspector.exe download
SteamCloudInspector.exe download "D:\CS2-Cloud-Backup"
SteamCloudInspector.exe inspect "cs2_user_convars.vcfg"
SteamCloudInspector.exe inspect "socache.dt"
```

Binary files are shown as a hexadecimal preview. Text files are printed directly.

## Delete and restore

Delete one file:

```bat
SteamCloudInspector.exe delete "cs2_user_convars.vcfg" --confirm DELETE-730
```

Delete every enumerated Cloud file:

```bat
SteamCloudInspector.exe delete-all --confirm DELETE-ALL-730
```

Restore a structured backup:

```bat
SteamCloudInspector.exe restore "CloudBackups\App730_before_delete_all_20260622_120000" --confirm RESTORE-730
```

Every destructive Cloud operation creates a mandatory backup first.

## CS2 Cloud files observed

Typical AppID 730 entries include:

```text
cfg/cs2_loadout_favorites.txt
cfg/cs2_preferred_items.txt
cs2_user_convars.vcfg
cs2_user_keys.vcfg
socache.dt
voice_ban.dt
```

CS2 uses Auto-Cloud. The API-visible list and the local `userdata` directory are intentionally displayed separately because local-only files can also exist, including `cs2_machine_convars.vcfg`, `cs2_video.txt`, and `*_lastclouded` files.

## Safety behavior

- Mutating Cloud commands refuse to run while `cs2.exe` is active.
- `full-reset` refuses to rename the local folder until the remote delete batch is confirmed.
- Cloud deletion is refused when enumeration is unexpectedly empty, except inside `full-reset`, where an already empty Cloud is treated as a valid state.
- Paths from Cloud filenames are normalized and traversal paths are rejected.
- Steam must run under the same Windows user and privilege level as the tool.
- Backups are created before destructive operations.

## Runtime implementation

No Steamworks SDK installation is required. The tool dynamically loads the game's `steam_api64.dll` and resolves the Flat API exports at runtime.

Initialization order:

```text
SteamAPI_InitFlat
SteamAPI_Init
```

Remote Storage accessor order:

```text
SteamAPI_SteamRemoteStorage_v016
SteamAPI_SteamRemoteStorage_v015
SteamAPI_SteamRemoteStorage_v014
```

**by St1cky**

<img width="979" height="247" alt="image" src="https://github.com/user-attachments/assets/d2de45e0-6960-4504-8453-5ec7cfa14433" />
<img width="623" height="180" alt="image" src="https://github.com/user-attachments/assets/4e8f3817-ce34-4ae7-adfc-d13cf3602cbd" />
