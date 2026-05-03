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

## Latest release — v0.99.0 (TX foundation)

| Variant | Download |
|---|---|
| **Windows 10 / 11** (installer) | **[pluto-wsjtx-bridge-0.99.0-setup.exe](pluto-wsjtx-bridge-0.99.0-setup.exe)** |

**This release is the TX FOUNDATION.** The libiio push path is wired
up and bench-verified — emits a 1 kHz tone via `--test-tone` that a
nearby receiver with antenna can clearly see (+30 dB above noise
floor at the bridge's TX freq). What's NOT yet in v0.99.0:

- Audio capture from VB-Cable
- `SsbModulator` integration (real-audio TX)
- Custom MainWindow with PTT button
- WSJT-X `transmittingChanged` → real-audio TX
- TX-side auto-reconnect
- TX `IqBalancer` / DC-offset tuning

Use this build to confirm your Pluto's TX hardware is alive on your
bench with a known-good antenna. Wait for v0.99.x for actual on-air
WSJT-X QSOs.

### Bench verification

Once installed, from a Command Prompt:

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
