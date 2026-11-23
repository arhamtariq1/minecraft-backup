# minecraft-backup

Back up a Minecraft server on Linux without copying world files while autosave is active. This Bash script handles Minecraft server backups through screen, tmux, or RCON, with tar archives or a restic repository as the destination.

## Features

- Back up one or more world directories; repeat `-i` for each world.
- Disable world autosaving during the backup, request a save and flush, then re-enable autosaving afterward.
- Send server commands through screen, tmux, RCON, or an `itzg/docker-minecraft-server` container's RCON CLI.
- Write compressed tar archives or use restic for backups.
- Remove older backups with the default thinning policy or oldest-first deletion, or disable deletion entirely.
- Print backup status in Minecraft chat with `-c`.

## Installation

From a checkout of this project, make the script executable and check its options:

```bash
chmod +x backup.sh
./backup.sh -h
```

Run it on Linux with Bash. For tar backups, install `tar` and your chosen compression command; the default is `gzip`. RCON also requires `xxd`. Install `restic` if you plan to use that backend, or screen/tmux if that is how you manage the server.

## Usage

For a server with RCON enabled:

```bash
./backup.sh -c \
  -i /home/user/server/world \
  -o /mnt/storage/backups \
  -s localhost:25575:secret \
  -w rcon
```

Replace `secret` with your RCON password. The script backs up the world directory to `/mnt/storage/backups` and reports status in Minecraft chat. Restrict network access to the RCON port.

For a server in a screen session named `minecraft`:

```bash
./backup.sh -c -i /home/user/server/world -o /mnt/storage/backups -s minecraft
```

For a tmux session named `0`, add `-w tmux` and use `-s 0`.

To use a restic repository instead of tar archives:

```bash
export RESTIC_PASSWORD="restic-pass-secret"
./backup.sh -c -i /home/user/server/world -r /mnt/storage/backups-restic -s localhost:25575:secret -w rcon
```

Keep the restic password available for restores.

### Retention and scheduling

The default `thin` policy keeps the last 24 hourly and last 30 daily backups, then uses remaining capacity for weekly backups. Use `-d sequential` to delete the oldest backup first, or `-d none` to disable deletion. The default maximum backup count is 128; change it with `-m`.

For hourly tar backups, add an entry with `crontab -e`:

```cron
00 * * * * /path/to/backup.sh -c -i /home/user/server/world -o /mnt/storage/backups -s minecraft
```

Use `-t` to enable the optional lock file. Run `./backup.sh -h` for the full option list.

### Restore and consistency

**Test your backups before you need them.** Extract a gzip-compressed tar backup into a separate directory:

```bash
mkdir restored-world
cd restored-world
tar -xzvf /path/to/backups/2019-04-09_02-15-01.tar.gz
```

Check the extracted directory layout before moving the world into place. Stop the server before replacing its world files. For restic backups, use `restic restore`.

**Live backups can contain inconsistent player data.** Minecraft can still save player data after `save-off`, so inventories and world contents may differ in a backup. The script requests saves and a disk flush, but cannot eliminate this risk while players remain active. Screen and tmux connections use a five-second wait by default; adjust it with `-z`.

## License

MIT. This project is based on [nicolaschan/minecraft-backup](https://github.com/nicolaschan/minecraft-backup). Retain the upstream copyright notice and MIT license text when redistributing the code.

## Related projects

The supplied related-project list includes these repositories. Their purposes were not provided, so no functionality is claimed here.

- [Rahul14103/portspider](https://github.com/Rahul14103/portspider): Consult its README for its purpose and usage.
- [RedZinXL/warc-gpt](https://github.com/RedZinXL/warc-gpt): Consult its README for its purpose and usage.
- [sakthivelj10-ai/kstone](https://github.com/sakthivelj10-ai/kstone): Consult its README for its purpose and usage.

