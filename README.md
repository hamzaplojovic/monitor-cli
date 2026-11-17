# mon

Control an external monitor's brightness, volume and resolution from the macOS terminal.

`mon` is a small bash script. It is written for a Dell S2722QC connected over USB-C to an Apple Silicon Mac. It talks to the monitor over DDC/CI (the standard that lets a computer change monitor settings) through [m1ddc](https://github.com/waydabber/m1ddc), and it changes the display mode through [displayplacer](https://github.com/jakehilborn/displayplacer).

```console
$ mon
brightness: 40
volume:     20
resolution: 2560x1440

$ mon bright +10
luminance: 40 -> 50
```

## Requirements

- macOS on Apple Silicon
- A monitor that supports DDC/CI, connected over USB-C or DisplayPort
- `m1ddc` and `displayplacer`:

```bash
brew install m1ddc displayplacer
```

## Install

```bash
git clone https://github.com/hamzaplojovic/monitor-cli.git ~/mon-cli
ln -s ~/mon-cli/mon /opt/homebrew/bin/mon
```

The script keeps its mute state in `~/mon-cli`, so clone it to that path or change `MUTE_FILE` at the top of the script.

### Point it at your monitor

The script has the monitor's ID hardcoded near the top. Replace both values with your monitor's ID:

```bash
m1ddc display list        # the UUID for DDC_UUID
displayplacer list        # the "Persistent screen id" for DP_ID
```

## Usage

| Command | What it does |
|---|---|
| `mon` or `mon status` | Show brightness, volume and resolution |
| `mon bright` | Show brightness (short form: `b`) |
| `mon bright 50` | Set brightness to 50 |
| `mon bright +10` / `-10` | Raise or lower brightness by 10 |
| `mon vol` | Show speaker volume (short form: `v`) |
| `mon vol 30` | Set volume to 30 |
| `mon vol +5` / `-5` | Raise or lower volume by 5 |
| `mon mute` | Set volume to 0 and remember the old level |
| `mon unmute` | Restore the volume from before `mon mute` |
| `mon res` | Show the current resolution (short form: `r`) |
| `mon res list` | List the scaled 60 Hz resolutions you can pick |
| `mon res 1080` | 1920x1080: large text |
| `mon res 1440` | 2560x1440: medium |
| `mon res 1692` | 3008x1692: small text, most space |
| `mon res 2304x1296` | Any resolution from `mon res list` |
| `mon help` | Show help |

Brightness and volume run from 0 to 100. A value outside that range is capped at 0 or 100.

## Notes

- **Mute is emulated.** The S2722QC ignores the DDC mute command. `mon mute` sets the volume to 0 instead and saves the old level in `.volume_before_mute`. If there is no saved level, `mon unmute` restores 50.
- **Reads are retried.** Right after a write, the monitor sometimes returns a nonsense value such as 28288. `mon` retries a read up to five times and accepts only an answer between 0 and 100.
- **Other displays stay where they are.** A resolution change reuses the current `displayplacer` layout and changes only this monitor, so the laptop screen and the arrangement do not move.
