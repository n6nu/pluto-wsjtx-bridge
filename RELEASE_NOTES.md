# Pluto WSJT-X Bridge — Release Notes



## v1.1.5 -- Help-menu polish + LGPL compliance + bundled user guide (2026-05-08)

Quality release on top of v1.1.4. No functional changes to the
RF / decode / wire paths.

New Help menu (View menu joined by Help):
  - Help -> User Guide (F1): opens the bundled beta-tester guide
    PDF in the system PDF reader.
  - Help -> About Qt: standard Qt LGPLv3 attribution dialog.

LGPL compliance polish:
  - Beta-tester guide PDF now ships with each installer.
  - Full text of LGPL-3.0 (Qt 6) and LGPL-2.1 (SoXR / libusb)
    ship in the install dir at Licenses/.
  - THIRD_PARTY_LICENSES.md gains an explicit source-availability
    section pointing at the per-bridge repos and upstream URLs.

Drop-in upgrade from v1.1.4.

## v1.1.4 — DC blocker user-enableable on RF-direct receivers (2026-05-07)

The DC blocker checkbox is now editable on RF-direct receivers
(HackRF / RTL-SDR / SDRplay / Pluto / AirSpy) — previously locked
greyed-out as of v1.1.3. Default state is unchanged (OFF on
RF-direct, ON on sound-card sources), so the on-the-bench behaviour
of a fresh install is identical. Diagnostic scenarios that want
the software IIR HP back on can now toggle it from Settings without
an INI edit.

Sound-card-IQ sources (FunCube Pro+ V2, FlexRadio DAX-IQ, Malachite
via iq-rx-bridge): unchanged.

Drop-in upgrade from v1.1.3.

## v1.1.3 — DC blocker default-off for RF-direct receivers (2026-05-06)

DC blocker is now default-OFF for RF-direct receivers (HackRF /
RTL-SDR / SDRplay / Pluto / AirSpy) and grayed out in Settings.
Their hardware DC correction at the SDR API level (and SDRplay's
Low-IF NCO chain in particular) handles the chip's residual offset
upstream; the v1.1.2 software IIR HP was redundant for these radios
and produced a small spike at QMAP centre on the SDRplay Low-IF
path (G3WDG bench report 2026-05-06).

Sound-card-IQ sources (FunCube Pro+ V2, FlexRadio DAX-IQ, Malachite
via iq-rx-bridge) keep the DC blocker default-ON: they have no
hardware DC mitigation, the LO leakage is real, and the IIR HP
is the only thing notching it out.

INI key linrad/dc_block_enabled is unchanged; existing INIs keep
their stored value. Only the first-launch default flips per device.

Drop-in upgrade from v1.1.2.

## v1.1.2 — DC blocker for zero-IF receivers (2026-05-05)

DC blocker for zero-IF receivers, removes the LO-leakage spike that
FunCube Pro+ V2 / HackRF / RTL-SDR / Pluto / AirSpy leak at the centre
of the spectrum. Per-sample IIR high-pass at the front of both the
on-screen waterfall (FftEngine) and the QMAP wire path (LinradServer);
cutoff = 100 Hz, well below any audio offset Q65 / FT8 cares about.

Toggle in Settings → "DC blocker (zero-IF spike removal)", default ON.
Toggling on also resets the I/Q balance EMA so a stale DC accumulator
from earlier samples doesn't keep subtracting against now-DC-free
input for ~2 s.

Drop-in upgrade from v1.1.1. INI key linrad/dc_block_enabled added
(default true; honours the previous behaviour for anyone who never
opens Settings).

## v1.1.1 — capability-gated IQ rate combo (2026-05-05)

Tightens the IQ-rate combo. Internally adds a per-device capability
list; only sound-card-IQ devices currently restrict their offered
rates -- RF-side bridges (HackRF / RTL-SDR / SDRplay / Pluto / AirSpy)
are unchanged in behaviour and still offer 96 / 128 / 192 / 256 kHz.

Drop-in upgrade from v1.1.0. No INI changes.

## v1.1.0 — UI refresh: fixed window, Settings menu, Linrad rate readout (2026-05-05)

User-visible polish across the bridge UI; no behavioural changes on
the wire (96 kHz IQ format unchanged).

- **Fixed-size 400x640 main window**. Replaces the freely-resizable
  640x540 minimum. Window opens identically every session and
  doesn't drift; conditional banners (manual-freq override,
  transverter IF readout) word-wrap rather than clip.
- **Settings is a top-level menu** in the menu bar (shortcut
  `Ctrl+,`) -- was a button at the bottom of the State group. Frees
  ~40 px of vertical real estate for the waterfall.
- **Linrad rate readout** in the State grid, between the device row
  and RX status. Reads the active LinradServer output rate; matches
  what's persisted in the INI.
- **Settings dialog reflow**: radio gain panel and bridge-wide
  group sit horizontally side-by-side. Was vertical, ran past the
  bottom of 1080p laptops with the deeper panels.
- **New "Linrad IQ rate" combo** in the Settings dialog. Defaults
  to "96 kHz (QMAP Default)" -- same wire format as before; matches
  every shipped QMAP release.
- UDP data port (`50004`) and Linrad host (`127.0.0.1`) /
  TCP port (`49812`) editors gained tooltips explaining their use
  in multi-instance setups.

INI compatible with v1.0.x. Drop-in upgrade.

## v1.0.0 — stable (2026-05-04)

Promoted out of beta. The full TX+RX Pluto / Pluto+ transceiver
made successful **2-way Q65 contacts on an IC-705 (2026-05-04)**
and has soaked through the day's iterations on CAT, audio routing,
and TX silencing (v0.99.0 → v0.99.8). The 0.99.x line ends here.

Cumulative since v0.99.0 first build:

- **All RX features of `pluto-rx-bridge` v1.0.0** — libiio over IP,
  AD9361 4 gain modes, auto-reconnect, opt-in CAT, cross-INI seed,
  Discover button, etc.
- **Real-audio TX**: WSJT-X output captured from VB-Cable, run
  through `SsbModulator`'s Hilbert phaser (USB/LSB), resampled
  48 kHz I + Q to the Pluto's IQ rate, scaled to int16, and pushed
  to the AD9361 DAC.
- **WSJT-X PTT-driven half-duplex**. Both UDP
  `transmittingChanged` and CAT `T VFOA n` / `\set_ptt VFO n`
  drive the same RX→TX→RX swap. Hamlib PTT values `1`/`2`/`3`
  all correctly start TX (PKTUSB / PKTLSB data modes send `3`).
- **TX silenced on key-up and bridge exit**: AD9361 TX_LO powered
  down + max attenuation parked. No residual carrier between
  transmissions.
- **Settings GUI**: TX attenuation 0..−89.75 dB, TX RF bandwidth,
  TX audio input device. Hot-swap on Apply. RX-side: gain mode,
  manual gain, sample rate, RF BW, PPM, libiio Discover button,
  CAT server toggle + port.
- **`--test-tone` bench-verification path** still available for
  quick TX path testing without WSJT-X in the loop.

INI compatible with v0.99.8 — drop-in upgrade.

## v0.99.8 — fix: PTT for WSJT-X data modes (PTT value 3 = DATA) (2026-05-04)

**Critical follow-up to v0.99.7.** With v0.99.7 installed, Test PTT
fired CAT commands but the bridge logged `[CAT PTT] off — TX→RX swap`
instead of `ON`. From the live console:

```
CAT CMD: "T VFOA 3"
[CAT PTT] off — TX→RX swap
```

WSJT-X in PKTUSB / PKTLSB modes (FT8 / FT4 / Q65 / etc.) sends Hamlib
PTT value **3** (`PTT_DATA`), not 1. Our CAT handler was matching
only the literal string `"1"` for ON; `"3"` fell through to the else
branch and got parsed as OFF.

Hamlib PTT enum:

| Value | Meaning |
|-------|---------|
| 0 | OFF |
| 1 | ON (standard) |
| 2 | MIC PTT |
| 3 | DATA PTT (WSJT-X data modes) |

Fix: both the short-form `T` and long-form `\set_ptt` handlers now
treat any non-zero value as ON. Affects all CAT-enabled bridges
(bridge-core change), but only this one is being patched today —
the rest inherit on their next rebuild.

Drop-in upgrade from v0.99.7.

## v0.99.7 — fix: WSJT-X Test PTT now actually fires CAT \set_ptt (2026-05-03)

**Critical fix.** Symptom: Test CAT green, Test PTT silent.

The bridge's CAT server response to `\dump_state` was advertising
`has_set_freq=1` and `has_get_freq=1` but missing `has_set_ptt=1`
and `has_get_ptt=1`. Even though `ptt_type=0x1` (`RIG_PTT_RIG`)
told WSJT-X the rig is CAT-PTT-capable, WSJT-X 2.6+ checks the
explicit `has_set_ptt` flag before issuing `\set_ptt` over CAT —
so Test PTT clicked but no command went out.

Fix: dump_state now advertises:

- `has_set_ptt=1` (CAT-driven PTT supported)
- `has_get_ptt=1`
- `has_set_mode=1`
- `has_get_mode=1`

(The bridge already implements all four. The `\set_ptt VFO 0|1`
handler that emits `pttChanged` was working since v0.99.0;
WSJT-X just wasn't sending the command.)

Drop-in upgrade from v0.99.6.

## v0.99.6 — fix: silence TX on key-up / bridge exit (2026-05-03)

**Critical fix.** Previously, after `stopTx` (or bridge exit), the
AD9361's TX_LO stayed running with the user's configured attenuation
in place. Even with no IQ samples being pushed, the mixer's LO
leakage radiated a steady residual carrier at the dial frequency —
audible as a tone on any USB-mode receiver tuned 1 kHz below.
Bridge exited cleanly but the Pluto kept transmitting until
power-cycled.

Fix: TX is now actively silenced in three places:

- **`stopTx()`** — write `hardwaregain = -89.75 dB` (max attenuation,
  ~90 dB suppression) AND `altvoltage1 powerdown = 1` (TX_LO off).
- **`openLocked()`** — same parking on initial open, so a fresh
  bridge start doesn't briefly leak carrier in the window between
  `applyAllParamsLocked` and the first `startTx`.
- **`tearDownLocked()`** — defensive: if stopTx wasn't called for
  some reason (crash, abnormal exit), park TX before destroying the
  libiio context. Last chance to silence the chip.

`startTx()` undoes the parking — restores the user's
`tx_attenuation_db` and powers TX_LO back up — so the radiate path
is unchanged when the bridge is actually keyed.

If you've been seeing a tone on a nearby receiver after stopping
the bridge, that was this bug. Upgrade and the carrier dies on
key-up.

## v0.99.5 — TX power + TX RF bandwidth in Settings (2026-05-03)

The Settings dialog now exposes two TX-side controls that were
CLI-only in v0.99.x:

- **TX attenuation** (`pluto/tx_attenuation_db`): AD9361 hardware TX
  attenuation in dB. Range 0 (full power, ~+0 dBm out on AD9363) to
  −89.75 (minimum). Step 0.25 dB. **The bridge default is −30 dB**
  (safe for bench testing into nearby SDRs), but that's typically
  too quiet for a real rig (IC-705, FT-991A, etc.) across the
  bench — raise it toward 0 dB if you can't hear the bridge.
- **TX RF bandwidth** (`pluto/tx_rf_bandwidth_hz`): the AD9361's
  TX-side analog LPF. 200 kHz to 40 MHz.

Hot-swap on Apply via `PlutoDevice::setTxAttenuationDb` /
`setTxRfBandwidthHz` (both mutex-protected, no bridge restart needed).

Drop-in upgrade from v0.99.4.

## v0.99.4 — TX audio input device picker in Settings (2026-05-03)

The Settings dialog now has a **TX audio input** combobox alongside
the existing **RX audio output** picker. Was CLI-only (`--tx-device`)
in v0.99.x — frustrating for users who couldn't find where to point
the bridge at WSJT-X's VB-Cable output.

- Combobox enumerates Windows audio capture devices via
  `QtAudioBridge::availableInputDevices()`.
- Pick the device WSJT-X is configured to send TX audio to (typically
  **CABLE Output (VB-Audio Virtual Cable)** if you've routed WSJT-X
  output to VB-Cable Line 1).
- Persists to `[audio] tx_device` in the INI.
- Hot-swap on Apply — `stopTxCapture` + `startTxCapture` on the new
  device, no bridge restart needed.

The change is in bridge-core's `RxSettingsDialog` (gated on a
`show_tx_audio` ctor flag) so it's a no-op for the RX-only siblings;
the ctor flag is set to `true` from `pluto-wsjtx-bridge`'s main.cpp.
When `hackrf-wsjtx-bridge` eventually retrofits onto `RxMainWindow`,
it'll inherit this for free.

Drop-in upgrade from v0.99.3.

## v0.99.3 — fix WSJT-X PTT-method-CAT (2026-05-03)

The bridge's CAT server (in bridge-core, shared with the other CAT-
enabled bridges) reported `ptt_type=0x8` in its `dump_state` response.
That's `RIG_PTT_GPION` in Hamlib's enum (inverted GPIO pin) — wrong
for a CAT-controlled rig.

Symptom: WSJT-X **Settings → Radio → PTT method = CAT** showed
**"PTT device: GPIO"** and refused to actually send `\set_ptt` over
CAT. The bridge's PTT-toggle path never fired. Test PTT failed.

Fix: changed `ptt_type=0x1` (`RIG_PTT_RIG`, "the rig handles PTT over
CAT") — which is what we actually implement
(`\set_ptt VFO 0|1` → `emit pttChanged(bool)`).

After upgrading, in WSJT-X:

- Settings → Radio → **PTT method = CAT**
- Click **Test PTT** — should turn red, no errors. Click again to
  release.
- The bridge log now shows `[CAT PTT] ON — RX→TX swap`.

Drop-in upgrade from v0.99.2.

> **Cross-bridge note:** the same `ptt_type` fix is in
> `bridge-core/CatServer.cpp`, so the other CAT-enabled bridges
> (hackrf-wsjtx-bridge, pluto-rx-bridge, rtlsdr-rx-bridge) inherit
> the fix on their next rebuild. If you have any of those installed
> and were hitting the same WSJT-X "PTT device: GPIO" symptom, watch
> for their next patch release.

## v0.99.2 — zero-config first launch (cross-INI seed + Discover button) (2026-05-03)

Two fixes for the most common first-launch friction: the bridge's
default URI is `ip:192.168.2.1` (stock USB-RNDIS), but a Pluto+ on the
LAN gets a DHCP address — so a fresh install showed "Pluto not
reachable" until the user edited the URI by hand.

### (a) Cross-INI seed from `pluto-rx-bridge`

If you already have **`pluto-rx-bridge`** installed and configured for
your Pluto, the WSJT-X bridge now copies the `[pluto]` section
(`uri`, `sample_rate`, `rf_bandwidth_hz`, `ppm_correction`,
`manual_gain_db`, `gain_mode`) from the rx-bridge's INI on first
launch. **Zero config when both bridges are installed**: open the
WSJT-X bridge, it works against the same Pluto.

The seed runs only once — first launch. Subsequent edits to either
bridge's INI stay independent.

### (b) "Discover" button in Settings

Settings dialog → Pluto settings → **Discover** button (next to the
Host URI field) runs libiio's scan over the network (mDNS / DNS-SD
via libxml2) AND USB backends, populates the URI line edit with the
first device found.

Useful for:

- Pluto+ on a DHCP address that just changed.
- Fresh install where `pluto-rx-bridge` isn't around to seed from.
- Switching between USB-RNDIS and Ethernet without remembering the IP.

If nothing is discovered (typical on a Windows box without Bonjour /
mDNS), the status line under the button suggests the manual-fallback
URIs to try (`ip:192.168.2.1`, `ip:<your-LAN-ip>`, `usb:`).

Drop-in upgrade from v0.99.1.

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
