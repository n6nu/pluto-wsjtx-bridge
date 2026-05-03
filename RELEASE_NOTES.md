# Pluto WSJT-X Bridge — Release Notes

## v0.99.1 — real-audio TX (WSJT-X PTT-driven) (2026-05-03)

The bridge can now transmit **WSJT-X audio**, not just a bench tone.
Capture WSJT-X output from VB-Cable, run it through the SsbModulator's
audio-rate Hilbert phaser (USB/LSB), resample 48 kHz I + Q to the
Pluto's complex-IQ rate, and push int16 samples to the AD9361.

### Setup

- Configure WSJT-X audio output to **VB-Cable Line 1** (or whatever
  Windows audio device you're routing through).
- Configure WSJT-X to point at the bridge's CAT server (Settings →
  Radio → Hamlib NET rigctl → `127.0.0.1:4536`) **OR** rely on
  WSJT-X UDP Status `transmittingChanged` to trigger PTT.
- Launch the bridge:

  ```
  pluto-wsjtx-bridge.exe --host ip:<your-pluto-ip> --cat
  ```

  (omit `--cat` if you want UDP-only PTT path.)

- Tune any nearby receiver to your dial freq, key WSJT-X (Tune button
  or click a CQ caller) — the AD9361 should radiate the modulated
  audio for as long as the PTT is asserted.

### What changed since v0.99.0

- `audio.startTxCapture()` runs at startup (unless `--test-tone`).
  Audio buffers continuously, idle until PTT is asserted.
- TX callback: `audio_tx_cb` pulls 48 kHz mono float audio from
  `audio.tx_ring`, runs `modulator.hilbertAudio()` for sideband
  selection, resamples I + Q to `sample_rate`, scales to int16
  (`scale=16384`, ~6 dB headroom against full-scale), fills the
  AD9361 push buffer.
- WSJT-X UDP `transmittingChanged` and CAT `\set_ptt` BOTH drive
  the same half-duplex sequence:
  - **PTT on**: `pluto.stopRx()` → `modulator.reset()` →
    `pluto.setPtt(true)` → `pluto.startTx()`
  - **PTT off**: `pluto.setPtt(false)` → `pluto.stopTx()` →
    `pluto.startRx()`
- Sideband-selection flag (`is_lsb`) shared between freq-source
  mode handlers and the TX callback so the right sideband is
  emitted on every burst.
- New `--tx-device <name>` CLI flag picks the audio capture
  source (default: system default audio input).

### Pluto vs HackRF TX path

Pluto is **direct-conversion** — TX_LO is the dial freq and the
AD9361 takes baseband I/Q directly. So we **skip** SsbModulator's
upconvert step (which HackRF needs because its TX is at IF, not
LO). The Hilbert-phasing audio I/Q is fed straight to the resampler
and then scaled to int16 for the DAC.

### Verified

- Build clean. Bridge launches, registers TX callback, RX streams,
  audio capture device opens.
- `\set_ptt VFO 1` over TCP triggers `[Pluto] RX stopped` →
  `[Pluto] TX worker started` → `[CAT PTT] ON — RX→TX swap`.
- `\set_ptt VFO 0` reverses cleanly.
- WSJT-X UDP path uses the same handler and is symmetric.

### Still NOT in v0.99.1 (deferred to v0.99.x)

- Custom MainWindow with a TX-side gain panel + manual PTT button
  (currently uses bridge-core's `RxMainWindow`, no PTT button —
  WSJT-X drives PTT entirely).
- TX-side auto-reconnect (RX has it; TX bails on libiio push error
  rather than retrying).
- TX `IqBalancer` / DC-offset calibration (we lean on AD9361's
  built-in TX calibration for now).

Drop-in upgrade from v0.99.0.

## v0.99.0 — TX foundation (2026-05-03)

First build of the **TX+RX transceiver** sibling to `pluto-rx-bridge`.
This release is the **TX foundation** — the libiio push path is wired
up and bench-verifiable, but full WSJT-X-driven audio TX is not yet
in. Use this release to confirm the AD9361 TX hardware is alive and
your antenna / setup is sane; wait for v0.99.x for actual on-air QSOs.

### What works

- **All RX features from `pluto-rx-bridge` v0.99.5** — same shared
  RX wiring (libiio context, gain modes, sample rate, RF bandwidth,
  PPM trim, auto-reconnect, opt-in CAT server, live UDP/CAT
  indicator). Drop-in replacement for the RX bridge in passive mode.
- **TX path scaffolding in `PlutoDevice`**:
  - `cf-ad9361-dds-core-lpc` device + `voltage0/1` outputs detected
    on open.
  - `iio_device_create_buffer(..., non-cyclic)` + `iio_buffer_push`
    loop on a dedicated worker thread.
  - `setTxAttenuationDb()` / `setTxRfBandwidthHz()` / `setPtt()` /
    `startTx()` / `stopTx()` on `PlutoDevice`.
  - Shared LO between TX and RX (TX_LO tracks RX_LO on every
    `setFrequency`); split-VFO behaviour can override later.
- **`--test-tone` CLI flag** generates a continuous 1 kHz tone via a
  phase-accumulator callback. Use it to verify radiation:

  ```
  pluto-wsjtx-bridge.exe --no-gui --test-tone --tx-freq 144.5 --tx-attenuation -30
  ```

  Tune any nearby receiver (HackRF, RTL-SDR, real radio) to the same
  freq and confirm the carrier is there.
- **`--tx-attenuation <dB>`** controls AD9361 TX0 hardware-gain in dB
  (range 0 to -89.75; default -30 dB for safe bench testing).
- **CAT server defaults to TCP 4536** (continuing the family
  sequence: 4532 / 4533 / 4534 / 4535 / **4536**), opt-in via
  Settings or `--cat`. Same auto-detect UDP-mute behaviour.

### What's NOT in this build (planned for v0.99.x)

- **Audio capture from VB-Cable** — no `SsbModulator` integration.
  WSJT-X audio output won't reach the Pluto yet.
- **WSJT-X PTT → real-audio TX**. The CAT `\set_ptt` and UDP
  `transmittingChanged` handlers don't gate audio transmission yet —
  only the `--test-tone` path is exercised.
- **Custom MainWindow with TX gain panel + PTT button**. v0.99.0
  reuses the bridge-core `RxMainWindow` for the GUI; TX is invisible
  in the GUI and only flips on via the `--test-tone` CLI.
- **TX auto-reconnect.** RX has it (mid-stream Pluto reboots); TX
  bails on error rather than retrying. Will be added once the basic
  TX path has bench hours.
- **TX-side IqBalancer / DC offset tuning.** AD9361 has good built-in
  TX calibration; we lean on it. May revisit.

### Known limitations

- The bridge **transmits continuously** while `--test-tone` is
  active. Use `-Ctrl+C` (or close the process) to stop. Don't leave
  it running into a real antenna unattended.
- TX and RX share the AD9361 LO. If you set a different `--tx-freq`
  the RX retunes too (half-duplex same-band model). Split-VFO needs
  v0.99.x.

### Hardware tested

Bench-built against an N6NU Pluto+ over Ethernet (firmware Tezuka
0.5.16.7, AD9363 reporting). TX path push loop iterates without
errors at 2.5 Msps. On-air receive verification with HackRF deferred
to user's bench (HackRF on the dev machine had a Zadig driver issue
during build).

### Source

Source lives in the family monorepo at
[n6nu/sdr-bridges](https://github.com/n6nu/sdr-bridges) under
`apps/pluto-wsjtx-bridge/` (app glue) + `radios/pluto/` (shared with
`pluto-rx-bridge` — `PlutoDevice` is a single class with both RX and
TX methods).
