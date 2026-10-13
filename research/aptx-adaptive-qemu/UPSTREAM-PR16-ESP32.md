Following the ESP32-as-SDR suggestion in this thread, we ran the measurement with
an ESP32-C5 (ESP-SDR / ESPARGOS raw-IQ firmware, receive only, no protocol
decoding) and added the control that our earlier sniffer-based capture could not
provide: the same phone, at the same distance, playing the same track, with only
the Bluetooth codec changed.

Method: 200 burst captures per state, 819 us each, at 2441 MHz (20 MSa/s, 11 MHz
analog filter, fixed manual gain). Detector: per capture, the largest rise of a
1 MHz band above its own 8 MHz neighbourhood, after DC removal; states compared
by the fraction of captures with a burst >6 dB.

| state | bursts >6 dB | z vs idle | p |
| --- | --- | --- | --- |
| phone, SBC, 30-50 cm (standard EDR) | 43/200 = 21.5 % | 5.85 | 5e-09 |
| phone, AAC, 30-50 cm (standard EDR) | 23/200 = 11.5 % | 3.53 | 4e-04 |
| host, aptX HD, ~0.5 m (standard EDR) | 32/200 = 16.0 % | 4.66 | 3e-06 |
| phone, aptX Adaptive, 30-50 cm | 6/200 = 3.0 % | 0.31 | 0.76 |
| nothing playing | 5/200 = 2.5 % | - | - |

So every standard-EDR link in the room is plainly visible, including the same
phone at the same distance, while the same phone on aptX Adaptive is
statistically indistinguishable from silence (3.0 % vs 2.5 %, p=0.76; SBC vs
Adaptive z=5.64, p=2e-08). This reproduces our earlier null result without
relying on address recognition: the receiver decodes nothing, and the only
variable between the 21.5 % and the 3.0 % row is the codec.

Two limits we want to be explicit about. At centimetre range the front end
compresses (SBC reads 5.0 % there), so only the 30-50 cm rows are valid. And the
detector is deliberately narrowband: it shows Adaptive media is not standard-EDR
bursts, but it does not by itself exclude a wideband emission in the same band --
that needs a multi-scale detector, which we have not run yet. The 5 GHz band is
also unchecked.

On the transmitting side, to save anyone else the time: ESP-SDR is receive-only
by design ("we are not providing a transmitter implementation at this time"), and
the ESP32 transmit work that exists is narrowband polar modulation (FM/SSB), not
an arbitrary wideband waveform. So an ESP32 cannot stand in for the vendor
silicon that appears to be required at the source end.

Raw data and scripts: https://github.com/baizhu945/openaptx (branch
research/aptx-adaptive-final), research/aptx-adaptive-qemu/STATUS.md section 7.5.
