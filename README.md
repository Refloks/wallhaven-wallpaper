# wallhaven-wallpaper

A small Bash script that sets a fresh GNOME wallpaper from
[Wallhaven](https://wallhaven.cc) once a day.

- Picks a random franchise/tag from a configurable list, samples a few random
  pages of results, and prefers the better-favorited half of what it finds.
- Remembers which wallpapers it has used and won't repeat one for a year
  (falls back to "not one of the last 40" if everything is exhausted).
- Only changes the wallpaper once per day, even if run several times.
- Waits for the network at login, keeps only the newest 30 downloads, and
  validates downloads (HTTPS only, Wallhaven domains only, real image types).

## Requirements

- GNOME (the wallpaper is set via `gsettings` / `org.gnome.desktop.background`)
- `bash` 4.3+, `curl`, `jq`, `file`, coreutils (`shuf`, `sort`, …)
- A Wallhaven account and API key

## Install

1. Get an API key from <https://wallhaven.cc/settings/account> and save it:

   ```sh
   mkdir -p ~/.config/wallhaven
   printf '%s' 'YOUR_API_KEY' > ~/.config/wallhaven/api-key
   chmod 600 ~/.config/wallhaven/api-key
   ```

2. Install the script:

   ```sh
   install -Dm755 wallhaven-wallpaper ~/.local/bin/wallhaven-wallpaper
   ```

3. Run it at login with the systemd user unit:

   ```sh
   install -Dm644 systemd/wallhaven-wallpaper.service \
       ~/.config/systemd/user/wallhaven-wallpaper.service
   systemctl --user daemon-reload
   systemctl --user enable wallhaven-wallpaper.service
   ```

## Usage

```sh
wallhaven-wallpaper          # change wallpaper (at most once per day)
wallhaven-wallpaper --force  # change it now regardless
```

Logs from the login run: `journalctl --user -u wallhaven-wallpaper`.

## Configuration

Edit the variables at the top of the script:

| Variable | Default | Meaning |
|---|---|---|
| `DIR` | `~/Pictures/wallhaven` | Where wallpapers are downloaded |
| `PURITY` | `100` | `100` = SFW only, `110` = SFW + sketchy |
| `MIN_WIDTH` / `MIN_HEIGHT` | `2560` / `1440` | Minimum resolution (16:9 only) |
| `BLOCK_DAYS` | `365` | Days before a wallpaper may repeat |
| `FALLBACK_RECENT` | `40` | Recent wallpapers avoided when all else is exhausted |
| `KEEP_FILES` | `30` | Downloaded wallpapers to keep |
| `PAGE_SAMPLES` | `5` | Random result pages sampled per franchise |
| `QUALITY_PERCENT` | `50` | Keep the top N% of samples by favorites |

The `FRANCHISES` array lists `"Name|tag_id"` pairs. To find a tag id, open a
tag on Wallhaven — the number in the URL (`wallhaven.cc/tag/1777`) is the id.

State (used-wallpaper history, last-changed date) lives in `~/.config/wallhaven/`.

## License

MIT
