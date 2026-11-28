# Shayan Web

English | [简体中文](README.zh-CN.md)

**Run coding agents on your computer. Stay in control wherever you work.**

Shayan (/ˈʃɑːjɛn/ SHAH-yen; 沙燕, *shāyàn*) is named after the Beijing swallow kite. Its Chinese name is 纸鸢.

Shayan Web is the browser interface for Shayan. It is a set of static files that Caddy on a server serves under the Web's own domain; the page reaches `shayand` on the Computer through a Relay. The Web and the Relay are installed separately ([Relay guide](../relay/README.md)) and can be on the same server or on different ones. The Web runs from its own package, `shayan-web-<version>-any.tar.gz`, signed and published in the releases of [`kotaoching/shayan-releases`](https://github.com/kotaoching/shayan-releases).

## Requirements

- A domain whose A record (and AAAA record, if you have IPv6) points to this server, with inbound TCP 80 and 443 open.
- Debian 12 or 13, or Ubuntu 24.04 LTS or 26.04 LTS, on x86_64 or arm64, and root or a user who can `sudo`.
- `sudo apt install caddy curl`. With `minisign` installed as well, every install and upgrade verifies the package's signature against the release key; without it, only HTTPS is relied on. Caddy runs as a system service, configured by `/etc/caddy/Caddyfile`.
- `/etc/caddy/Caddyfile` contains this line (add it once before installing; it does no harm while there are no site files yet):

  ```
  import sites/*.caddy
  ```

  The installer never edits `/etc/caddy/Caddyfile`. It only writes `/etc/caddy/sites/shayan-web.caddy`. If the line is missing, it stops and says so.

## Install

```sh
curl -fsSL https://github.com/kotaoching/shayan-releases/releases/download/web/install.sh | sudo sh -s -- --domain <web domain> [--dir <dir>]
```

`<dir>` defaults to `/srv/shayan-web`. To install a specific version, including one not yet marked as the latest:

```sh
curl -fsSL https://github.com/kotaoching/shayan-releases/releases/download/web-v<version>/install.sh | sudo sh -s -- --version <version> --domain <web domain>
```

The script downloads that version's package, verifies its signature with the release key built into the script (when `minisign` is installed), unpacks it and runs the package's own `install.sh`. When it finishes, it prints the installed version, the configuration file, whether Caddy is serving the site, the command to read the logs and the command to roll back, and whether `https://<domain>/` can be reached from this server (it may not be yet while DNS or the certificate is pending; that does not count as a failure).

## Upgrade

```sh
sudo /srv/shayan-web/current/fetch.sh                      # the latest version, if it is newer than the installed one
sudo /srv/shayan-web/current/fetch.sh --version <version>  # that version, older ones too
```

A normal upgrade switches `current` and leaves the site as it is. The installer still hands the configuration to Caddy (`caddy reload`, without `--force`); since it has not changed, Caddy does not reload it, and a Relay behind the same Caddy keeps its WebSocket connections. The next request gets the new version. Pages that are already open keep working with what they loaded; they show "A new version of Shayan is ready.", and tapping "Reload" switches to it.

## Moving from Patchbay 0.6.1

Shayan was called Patchbay up to 0.6.1. Patchbay does not update itself into the new name, and Shayan does not read Patchbay's data, so the move is made once by hand:

1. Remove Patchbay's Web as the Uninstall section below says, with `patchbay` in place of `shayan` (`/etc/caddy/sites/patchbay-web.caddy`, `/srv/patchbay-web`, `/etc/patchbay-web`).
2. If the Web moves to a new domain, point the domain at this server first.
3. Install Shayan's Web as the Install section above says. Phones open the new page and pair again; it does not read what the old page kept.

## Configuration

`/etc/shayan-web/web.env` holds one `KEY=value` per line. The first install writes it from its options; after that you edit it yourself. After editing, run `sudo <DIR>/current/install.sh` again. Once the file exists, options given to the installer have no effect, and it says so. If `DIR` or `DOMAIN` is missing, it stops and says what to add.

| Key | Meaning |
| --- | --- |
| `DIR` | The install directory, an absolute path; `/srv/shayan-web` by default |
| `DOMAIN` | The Web's domain; phones open `https://<DOMAIN>` |
| `UPDATE_SOURCE` | Where upgrades come from: `https://github.com/kotaoching/shayan-releases/releases/download/web`, or a local directory with the same layout |

- Changing the domain: edit `DOMAIN` and run the installer again. The site is rewritten for the new domain, checked on a copy of `/etc/caddy` first, then put in place and reloaded. Everything a phone has stored belongs to the old address, so on each phone open the new address, add it to the Home Screen again and pair again.
- Changing the directory: `sudo cp -a <old> <new>`, edit `DIR`, run `sudo <new>/current/install.sh`, then `sudo rm -r <old>`.

## Roll back

The server keeps the version that was replaced, so you can go back to it without a download:

```sh
sudo /srv/shayan-web/versions/<previous version>/install.sh
```

For an older version, install it by its version number (the second command under "Upgrade"). Every version stays available in the releases.

## Uninstall

```sh
sudo rm /etc/caddy/sites/shayan-web.caddy && sudo systemctl reload caddy
sudo rm -r /srv/shayan-web /etc/shayan-web
```

## Troubleshooting

- Logs: `journalctl -u caddy`. Whether Caddy is running: `systemctl status caddy`.
- The installer stops with "needs the line 'import sites/*.caddy'": add that line to `/etc/caddy/Caddyfile` and run it again.
- The install fails (Caddy is not running, or the reload fails): the installer exits with status 1 and prints which step failed, the log command and the rollback command. It does not switch back on its own.
- HTTPS does not answer: check the domain's A record, ports 80 and 443, and Caddy's certificate (in its logs).
- The phone reports `protocol`: the Web and the Relay use different versions of the Relay protocol; upgrade both to the latest.

## On the phone

Open `https://<web domain>`, add it to the Home Screen, and in it scan the pairing code that `shayan devices pair` shows on the Computer.
