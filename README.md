# wallhaven-wallpaper

A small Bash script that sets a fresh GNOME wallpaper from
[Wallhaven](https://wallhaven.cc) once a day.

- Picks from all of Wallhaven, or only from tags you choose (games, anime,
  artists, ...), samples a few random pages of results, and prefers the
  better-favorited half of what it finds.
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

Out of the box the script picks from **all of Wallhaven** (all categories,
SFW only, landscape images of at least 1920x1080). Everything is configurable in an optional
config file:

```sh
cp config.example ~/.config/wallhaven/config
```

Uncomment and change only what you need. The file is plain Bash.

### Only use certain tags

To limit wallpapers to games, anime, artists or themes you like, set `TAGS`
to a list of `"Name|tag_id"` entries:

```sh
TAGS=(
    "The Legend of Zelda|1777"
    "Final Fantasy|997"
)
```

Each run tries the tags in a random order until it finds a wallpaper you
haven't had yet. To find a tag id, open the tag on Wallhaven: the number in
the URL (`wallhaven.cc/tag/1777`) is the id.

### All settings

| Setting | Default | Meaning |
|---|---|---|
| `TAGS` | *(empty)* | Tags to pick from; empty = all of Wallhaven |
| `PURITY` | `100` | `100` = SFW only, `110` = SFW + sketchy |
| `CATEGORIES` | `111` | General / anime / people flags, e.g. `110` = no people |
| `RATIOS` | `landscape` | `landscape`, `portrait`, or ratios like `16x9,16x10`; `""` = any |
| `MIN_WIDTH` / `MIN_HEIGHT` | `1920` / `1080` | Minimum resolution |
| `DIR` | `~/Pictures/wallhaven` | Where wallpapers are downloaded |
| `BLOCK_DAYS` | `365` | Days before a wallpaper may repeat |
| `FALLBACK_RECENT` | `40` | Recent wallpapers avoided when all else is exhausted |
| `KEEP_FILES` | `30` | Downloaded wallpapers to keep |
| `PAGE_SAMPLES` | `5` | Random result pages sampled per search |
| `QUALITY_PERCENT` | `50` | Keep the top N% of samples by favorites |

State (used-wallpaper history, last-changed date) lives in `~/.config/wallhaven/`.

## License

MIT
