# Fregata Selective Backup for macOS

This project addresses a simple need: keep an off-Mac copy of the surveillance footage that matters. It selects events Fregata detects as a **person** (`person`) or **car** (`car`), then puts their clips and available snapshots in iCloud Drive. If the Mac is stolen or damaged, footage that has finished syncing remains accessible from another device signed in to the same iCloud account.

For each event, the Mac script asks Fregata to create one clip from the short video segments recorded locally. The clip starts **30 seconds before the event begins** and ends **60 seconds after it finishes**. Fregata combines the segments covering that window into one easier-to-review video. The script then copies the clip and available snapshots to the iCloud Drive folder.

This package is made for **the Fregata macOS app**. It is not intended for a standard Frigate installation on a VM, NAS, Docker host, or Linux server. It never deletes source recordings or destination files.

The iCloud copy protects against loss of the Mac only after synchronization has finished. The script writes to the local iCloud Drive folder and cannot verify that Apple has completed the upload, so a very recent event may not yet be available in the cloud.

The clip is requested from Frigate's recording-clip API for the configured time window. The API base URL defaults to the local Frigate service. Keep the API on a trusted local network; this script does not implement API authentication.

## Requirements

- An Apple Silicon Mac with Fregata installed, configured, and running.
- Python 3 on the Mac.
- iCloud Drive enabled if you want copies to sync to iCloud.
- Fregata retention long enough for the script to fetch footage. The database and media must remain accessible on this Mac.

## Configure

1. Make sure Fregata is open and shows **Running** in its menu-bar icon. Finish its setup wizard and confirm the cameras are recording. In the Fregata menu, also enable **Settings → Launch at Login** so it returns after a Mac restart.
2. Download this package and double-click the ZIP to unpack it.
3. Open the extracted folder and double-click `Install.command`. If macOS blocks it, right-click the file and choose **Open**.
4. With Fregata's default locations, setup automatically finds `~/Fregata/config/frigate.db` and `~/Fregata/media`. If you moved these folders, a picker asks you to select them. Their locations are available from the Fregata menu-bar icon → Settings → Folders.
5. Keep the proposed Frigate-Record folder in iCloud Drive, or choose another existing folder. Make sure iCloud Drive is signed in; Frigate-Record is created automatically.
6. Keep the suggested API address `http://127.0.0.1:5000` unless you changed Fregata's internal port.
7. After checking access, setup enables scheduled backups. It does not backfill older events. New `person` and `car` events are processed every two minutes.
8. To verify, trigger a test event and look for its folder under iCloud Drive → Frigate-Record. Keep the Mac awake, your user session open, and Fregata running for clip assembly.

If Python 3 is missing, install Python 3 for macOS and run `Install.command` again. If Fregata is stopped or the Mac is asleep, clip assembly waits until both are available.

The installer creates a cutover marker at installation time. Only events starting after that time are handled; there is no historical backfill. It validates the database, folders, destination, and API before loading the LaunchAgent.

## Output and behavior

Files are stored under:

    destination/YYYY-MM-DD/camera/event-id/recordings/event.mp4

Available clean snapshots and thumbnails are stored in the event's snapshots directory. In-progress events may be refreshed as new stable footage becomes available. Completed clips are skipped once the recorded time range is unchanged.

The script writes to the local destination. It cannot confirm that a cloud provider has finished syncing files remotely.

## Check and run

Configuration check:

    python3 ~/Library/Application\ Support/FrigateSelectiveBackup/frigate_selective_backup.py --config ~/Library/Application\ Support/FrigateSelectiveBackup/config.json --check

Dry-run one event:

    python3 ~/Library/Application\ Support/FrigateSelectiveBackup/frigate_selective_backup.py --config ~/Library/Application\ Support/FrigateSelectiveBackup/config.json --dry-run --event-id EVENT_ID

Read the log:

    tail -n 20 ~/Library/Logs/FrigateSelectiveBackup.log

## Stop the scheduled job

Run:

    launchctl bootout gui/$(id -u)/org.frigate.selective-icloud-backup

This stops scheduled runs. It does not remove recordings or copied files.

## Privacy

The project contains no camera addresses, account names, passwords, or private network details. Configuration and logs are stored locally on the Mac. Do not commit a real config.json or logs to a public repository.

## License

Released under the MIT License. See LICENSE.
