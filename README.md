# Open POP Piano

A browser replacement for the **POP Piano** app, for pianos with light-up keys.
Open a MIDI or MusicXML file and play it on the piano (sound and lights) while following the score, or learn it one segment at a time.

**Use it online:** https://bondimachine.github.io/open-pop-piano/

It's a single HTML file: open `index.html` in desktop Chrome or Edge and click **Connect piano**. The only external piece is the score renderer, [OpenSheetMusicDisplay](https://opensheetmusicdisplay.org/), which is loaded from a CDN the first time a file is opened.

## Features

- **Files**: MIDI (`.mid`) and MusicXML (`.musicxml`, `.xml`, compressed `.mxl`). Open them with the button or drop them anywhere on the page.
- **Score**: shows the sheet music with a cursor, and the notes being played (or to play, in Learn mode) highlighted in blue. MusicXML files show their own notation, repeats included. For MIDI files a score is generated from the notes (quantized to 16ths, split at middle C into treble and bass staves), so it's approximate.
- **Play**: plays the song on the piano with the keys lighting up as they sound, with adjustable speed and a seek bar. Tracks/channels/staves can be turned on or off (e.g. to practice one hand).
- **Learn**: plays a segment of about 5 seconds (always whole bars), then lights up the next note or chord and waits until you play it, step by step, until the segment is done. Then it moves on to the next one.
- **Keyboard**: shows the keys you press and lets you turn lights on and off by clicking.

## Compatible pianos

- Tested with a **Vangoa 88-key lighted piano**.
- It should also work with other light-up keyboards built on the same Bluetooth module (the ones that work with POP Piano). They show up as **`GZUT-MIDI`** when connecting.

## Requirements

- Desktop **Chrome or Edge** (the page uses Web Bluetooth, which Safari and Firefox don't support).
- The piano accepts one connection at a time: close POP Piano on your phone and don't pair the piano in your computer's MIDI settings (Audio MIDI Setup on a Mac) before connecting.

## Protocol

The piano uses standard BLE MIDI, but its firmware only accepts packets with the fixed timestamp bytes `80 80`. The OS MIDI stack (including Web MIDI) always sends real timestamps, so regular MIDI apps can't drive the lights; this page writes to the Bluetooth characteristic directly.

- Service `03b80e5a-ede8-4b33-a751-6ce34ec4c700`, characteristic `7772e5db-3868-4112-a1a9-f2669d106bf3` (write without response; notifications carry the keys you press).
- Light a key: `80 80 F0 4D 4C 4E 45 <note> 02 80 F7` (`4D 4C 4E 45` is "MLNE" in ASCII). Use `00` instead of `02` to turn it off.
- All lights off: `80 80 F0 50 00 00 00 00 00 80 F7`
- Play a sound: `80 80 90 <note> <velocity>` and `80 80 80 <note> 00` to release. Several notes can share one packet, each preceded by `80`. Notes sent this way don't light their keys.
- No initialization or handshake is needed.

The protocol was worked out by capturing the Bluetooth traffic between the POP Piano app and the piano.

Not affiliated with Vangoa or POP Piano.
