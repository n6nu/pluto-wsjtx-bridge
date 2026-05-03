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

## Latest release — v0.99.3 (fix WSJT-X PTT method=CAT)

| Variant | Download |
|---|---|
| **Windows 10 / 11** (installer) | **[pluto-wsjtx-bridge-0.99.3-setup.exe](pluto-wsjtx-bridge-0.99.3-setup.exe)** |

Two fixes for the most common first-launch friction (the default
`ip:192.168.2.1` URI doesn't match a Pluto+ on a DHCP LAN address):

1. **Cross-INI seed from `pluto-rx-bridge`.** If you have the RX-only
   bridge installed and configured, the WSJT-X bridge copies its
   `[pluto]` section (URI, sample rate, RF BW, PPM, gain) on first
   launch. Zero config when both bridges are installed.
2. **"Discover" button in Settings** runs libiio's scan over the
   network (mDNS) + USB backends and populates the Host URI line edit
   with the first found Pluto.

Real-audio TX (v0.99.1 feature, still here) — captures WSJT-X output
from VB-Cable, runs through the SsbModulator Hilbert phaser, resamples
48 kHz I + Q to the Pluto's complex-IQ rate, scales to int16, and
pushes to the AD9361. WSJT-X PTT (UDP `transmittingChanged` or CAT
`\set_ptt`) drives half-duplex `startTx` / `stopTx`.

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
