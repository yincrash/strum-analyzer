# Belt Strum Tuner

A single-page, no-install web tool that listens through your microphone and tells you the frequency of a plucked 3D-printer belt. Built for tensioning CoreXY belts by ear-plus-numbers: pick your printer, pluck the belt, read the verdict.

**Use it here: https://yincrash.github.io/strum-analyzer/**

Everything runs in the browser. No audio leaves your device.

## How to use

1. Open the page in Chrome, Safari, Firefox, or a phone browser and allow microphone access.
2. Pick your printer from the preset list, or choose Custom and type a target range.
3. Position the gantry as your printer's docs describe (for example, 150 mm from the tensioner) and pluck the belt like a guitar string with the mic a few centimeters away.
4. Read the **Last strum** number. It is the median of the readings taken while the belt was ringing. The verdict tells you whether to tighten or loosen.
5. Adjust, pluck again, and compare against the history list.

You can also analyze an existing recording, such as a phone voice memo, with **Analyze a recording**. Every pluck found in the file is added to the history and the loudest one's spectrum is drawn. **Record 10 s & analyze** captures the page's own microphone input, runs it through the same file analysis, and offers the recording to save, which is handy for comparing against a recording app or for sharing.

## Presets

| Printer | Target | Note |
|---|---|---|
| ZeroG Mercury One.1 (XY) | 110–120 Hz | X gantry 150 mm from the tensioner |
| Voron 2.4 / Trident (A/B belts) | ~110 Hz | 150 mm span between idler centers |
| Voron 2.4 (Z belts) | ~140 Hz | starting point |
| Creality K1 / K1 Max / K1C (XY) | 110 Hz | |
| RatRig V-Core 4 (CoreXY belts) | 87 ±1 Hz | quarter turn of tensioner ≈ 10 Hz |
| RatRig V-Core 4 Hybrid (hybrid belts) | 84 ±1 Hz | |
| Prusa CORE One | 90–98 Hz | lower belt ≥ 90, upper belt ≤ 98 |
| Prusa CORE One L | 85–95 Hz | lower belt ≥ 85, upper belt ≤ 95 |
| Prusa XL | 82–85 Hz | community value, not an official spec |

Presets are starting points taken from public documentation. Belt frequency depends on the free span length and where you pluck, so always follow your own printer's manual.

Sources:
- Voron: [Secondary Printer Tuning](https://docs.vorondesign.com/tuning/secondary_printer_tuning.html)
- RatRig: [V-Core 4 calibration](https://docs.ratrig.com/v-core-4-0/commissioning-guide-hub/corexy/calibration) and [V-Core 4 Hybrid calibration](https://wiki.ratrig.com/products/v-core-4-0/commissioning-guide/hybrid/03-calibration)
- Creality: [K1 series XY belt tension](https://wiki.creality.com/en/k1-flagship-series/k1-series-general-documents/xy-axis-belt-tension)
- Prusa: [CORE One belt tension](https://help.prusa3d.com/article/adjusting-belt-tension-core-one-core-one-indx-core-one-l_845048), [XL belt tension](https://help.prusa3d.com/article/adjusting-belt-tension-xl_401793)
- ZeroG: [Mercury One.1 manual](https://docs.zerog.one/)

## Controls

- **Target**: the acceptable band. Filled in by the preset; editing it switches the preset to Custom.
- **Search range**: pitches outside this range are ignored, and the spectrum plot spans it. Narrow it if the detector locks onto a harmonic or room noise.
- **Gate**: input level below which nothing is measured. Lower it if the level bar never turns green; raise it if room noise keeps it green.
- **Min clarity**: how clean a tone the pitch detector needs before its value is used to refine the spectral peak. Belt plucks ring for only 100–200 ms, so 0.6 is a reasonable default.
- **Test tone**: plays 115 Hz through the speakers so you can confirm the mic and detector agree.

## Reading the history

Each strum shows its frequency, the verdict, which estimator produced it, and the prominent peaks in that pluck with harmonics of the reading labeled (2×, 3×). Two cautions can appear:

- **also N Hz (not a harmonic)**: a second tone above the reading that rang with the pluck and is not one of its harmonics. That is another ringing part, such as the other belt, a longer span, or the frame. If that tone is actually the belt, the belt is tighter than the reading says, so check the spectrum before tightening further.
- **stronger tone at N Hz outside search range**: the loudest tone in 20–1200 Hz lies outside the search range you set. A search range capped below the belt's frequency makes the tool report half the true value, so widen the range.

Steady tones that were already present before the pluck, such as mains hum or a fan, are learned as background while the input is quiet and ignored. If the input never goes quiet, the page tells you to raise the gate.

The page also measures the noise floor and never lets the gate sit within 8 dB of it. A strum that peaks less than 12 dB above the floor is marked **weak**: its reading is low confidence. A good pluck stands 20 dB or more above the room, which usually means the mic within a few centimeters of the belt and fans or air conditioning off.

Settings are remembered in the browser.

## How it works

Two estimators run on every frame while the input is above the gate:

- **Spectral peak**: the strongest peak in the search range of a 32768-point FFT, parabolic-interpolated, accepted only when it stands at least 12 dB above the median level in the range and at least 10 dB above the background spectrum learned while the input was quiet. This is the primary reading and matches how the printer communities measure belts with spectrum-analyzer apps.
- **Pitch detector**: the McLeod Pitch Method (normalized square difference, key-maximum picking, parabolic interpolation) on an 85 ms window. When it agrees with the spectral peak within 3 percent it supplies the final number, since it is more precise. When it disagrees it is ignored, because on short noisy belt plucks it tends to pick a subharmonic. It is used alone only for a very clean tone (clarity above 0.9).

Each strum's reported value is the median of its frame readings over the decay. The history shows which estimator produced each value.

## Tips

- 60 Hz mains hum has a harmonic at 120 Hz. A steady 120 Hz reading with nothing plucked is hum from a power supply or fan, not the belt.
- The page asks the browser to disable noise suppression and automatic gain so the pluck is not filtered out.
- Microphone access requires https or localhost. The GitHub Pages link above is https. To run locally: `python3 -m http.server` in this folder and open http://localhost:8000.

## License

MIT
