# Pluto WSJT-X Bridge — Beta Releases

Windows installer downloads for the **Pluto WSJT-X Bridge** — a Qt6
C++ application that turns an **ADALM-Pluto** or **Pluto+** into a
WSJT-X-driven full TX+RX transceiver. Sibling to the
[HackRF WSJT-X Bridge](https://github.com/n6nu/HackRF-WSJTX-Bridge),
with the AD9361/AD9363 in place of HackRF One.

For the **RX-only** companion (Pluto observation while a real radio
handles TX) see the separate [pluto-rx-bridge](https://github.com/n6nu/pluto-rx-bridge)
repo.

The bridge talks to the Pluto over **libiio** (network backend by
default — `ip:HOST` over USB-RNDIS or Ethernet) and shares the entire
RX wiring with `pluto-rx-bridge` (auto-reconnect, opt-in rigctld CAT
server, configurable sample rate / RF BW / gain / ppm). On top, this
build adds the AD9361 TX path: `cf-ad9361-dds-core-lpc` device,
voltage0/1 DAC outputs, libiio push worker, TX attenuation control,
half-duplex shared LO with RX, and CLI `--test-tone` for bench
verification.

Author: **Andreas Junge, N6NU** &lt;<andreas@n6nu.org>&gt;.

---

## Latest release — v0.99.6 (fix: silence TX on key-up / exit)

| Variant | Download |
|---|---|
| **Windows 10 / 11** (installer) | **[pluto-wsjtx-bridge-0.99.6-setup.exe](pluto-wsjtx-bridge-0.99.6-setup.exe)** |

**Critical fix.** Earlier v0.99.x builds left the AD9361's TX_LO
running with the user's configured attenuation in place after
`stopTx` (or bridge exit). Even with no IQ being pushed, the mixer's
LO leakage radiated a steady residual carrier at the dial frequency
— audible as a tone on any USB-mode receiver tuned 1 kHz below.
Bridge exited cleanly but the Pluto kept transmitting until
power-cycled.

v0.99.6 actively silences TX in three places: `stopTx` (sets
`hardwaregain = -89.75 dB` AND `TX_LO powerdown = 1`), `openLocked`
(same parking on initial open so a fresh start can't briefly leak
carrier), and `tearDownLocked` (defensive — last-chance silence
before destroying the libiio context). `startTx` undoes the parking
when the bridge is actually keyed.

If you've been seeing a tone on a nearby receiver after stopping
the bridge, that was this bug. Upgrade and the carrier dies on
key-up.

Cumulative since v0.99.0:

- **v0.99.6** — silence TX on key-up / exit (this release).
- **v0.99.5** — TX attenuation + TX RF BW controls in Settings.
- **v0.99.4** — TX audio input device picker in Settings.
- **v0.99.3** — fix `ptt_type=0x1` so WSJT-X PTT method=CAT
  actually fires (was reporting `0x8` = `RIG_PTT_GPION`, making
  WSJT-X show "PTT device: GPIO" and decline to send `\set_ptt`).
- **v0.99.2** — cross-INI seed from `pluto-rx-bridge` + "Discover"
  button (libiio scan) so a fresh install on a Pluto+ with a LAN
  DHCP address comes up with zero config.
- **v0.99.1** — real-audio TX wired up. WSJT-X output captured
  from VB-Cable → `SsbModulator` Hilbert phaser → resampled to
  the Pluto's IQ rate → AD9361 DAC. WSJT-X PTT (UDP
  `transmittingChanged` or CAT `\set_ptt`) drives half-duplex
  `startTx` / `stopTx`.
- **v0.99.0** — TX foundation: `PlutoDevice` extended with TX
  path, `--test-tone` CLI for bench verification.

### WSJT-X setup

1. WSJT-X Settings → **Audio** → output to **VB-Cable Line 1** (or
   whichever Windows audio device you'd like the bridge to capture
   from).
2. WSJT-X Settings → **Radio** → **Hamlib NET rigctl**, Network
   Server `127.0.0.1:4536`. (Or rely on UDP-only PTT — the bridge
   listens to `transmittingChanged` over WSJT-X UDP either way.)
3. Launch the bridge:
   ```
   pluto-wsjtx-bridge.exe --host ip:<your-pluto-ip> --cat
   ```
4. Tune dial, click WSJT-X **Tune** or pick a CQ. The Pluto should
   radiate the modulated audio for as long as PTT is asserted.

### Still NOT in v0.99.x (deferred)

- Custom MainWindow with TX-side gain panel + manual PTT button.
  The bridge currently uses bridge-core's `RxMainWindow` (no PTT
  button) — TX is driven entirely by WSJT-X UDP / CAT.
- TX-side auto-reconnect (RX has it; TX bails on push error).
- TX `IqBalancer` / DC-offset calibration.

### Bench verification (--test-tone, no WSJT-X needed)

If you want to verify the TX path without WSJT-X in the loop:

```
cd "C:\Program Files\Pluto WSJT-X Bridge"
pluto-wsjtx-bridge.exe --no-gui --console --host ip:192.168.2.1 --test-tone --tx-freq 432.0 --tx-attenuation -10
```

Replace `192.168.2.1` with your Pluto's IP if it's on the LAN. Tune
any nearby receiver (HackRF, RTL-SDR, real radio, SDRplay) to the
same freq and confirm the carrier. **An antenna on the Pluto's TX1
SMA jack is required** — without one, internal trace coupling is
below noise floor and the bridge looks like it isn't working.

---

## Bundled third-party libraries

The installer ships with everything the bridge needs at runtime —
no separate libiio install or Visual C++ runtime install required:

- **libiio v0.26** (Analog Devices, LGPLv2-or-later) — talks to
  the Pluto's IIO endpoints over the network/USB backends.
- **libusb-1.0**, **libxml2**, **libserialport** — libiio's
  transitive deps; supplied by the official ADI Windows binary
  snapshot.
- **Qt 6.8.3** (LGPLv3) — GUI and core runtime.
- **FFTW3 / SoXr / pthreads4w** — DSP support libraries used by
  the bridge-core shared code.

Full per-library licence text in
[THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md). Bridge code
itself is **GPLv3** — see [LICENSE](LICENSE).

---

## Source

Source lives in the family monorepo at
**[n6nu/sdr-bridges](https://github.com/n6nu/sdr-bridges)** under
`apps/pluto-wsjtx-bridge/` (TX-side glue: `--test-tone` etc.) +
`radios/pluto/` (`PlutoDevice` extended with TX methods, shared
with `pluto-rx-bridge`) + `bridge-core/` (RX wiring, GUI, CAT,
Linrad, FFT, demod, etc.).

---

## See the full release-by-release changelog

[RELEASE_NOTES.md](RELEASE_NOTES.md)
