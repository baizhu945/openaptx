# Reply draft: the ESP32-SDR route, and the measurement this thread still owes

Thanks for the pointers, and for the Snapdragon-only conclusion -- it matches
what we found from the other direction. We unpacked both Windows 11 25H2 ISOs
(x86_64 and ARM64): neither contains any aptX component, and the inbox A2DP
driver `BthA2dp.sys` implements aptX Adaptive only as an offloaded "sideband
codec" for which "the profile driver" does "not support codec operations". The
audio path Microsoft documents for it is a *sideband* link -- "often an I2S or
PCM interface ... or SLIMbus ... or SoundWire" -- driven by Microsoft-defined
`HCI_VS_MSFT_Avdtp_*` vendor commands. So "only on Snapdragon PCs" is right,
and it is a hardware-integration requirement rather than a licence flag.

## What the ESP32 route can and cannot do today

We read both repositories and the ESPARGOS documentation before replying,
because the difference matters:

- **Raw IQ receive, any supported ESP32**: 2.2-2.7 GHz, ~13-54 MHz analog
  bandwidth, up to 80 MSa/s into SRAM, 10-bit I/Q; burst capture at low duty
  cycle (browser WebSDR, on-chip FFT).
- **ESP32-S3 continuous receive**: decimated USB stream, 250 down to
  15.625 kSa/s (+/-125 kHz), usable from SDR++/GNU Radio via esp-sdr-bridge.
- **ESP32-S31 board / ESPARGOS One**: continuous 20 MSa/s (USB) or 40 MSa/s
  (Ethernet), receive only; ESPARGOS One is an 8-antenna array at 350 EUR,
  EU professional customers only.
- **Arbitrary transmit**: the ESPARGOS FAQ says "Yes, this is also possible",
  but "we are not providing a transmitter implementation at this time due to
  the potential for misuse".
- **ESP32-S3 transmit that exists**: polar modulation, 459 Hz frequency steps
  and 0.28 dB amplitude steps -- measured carrier, FSK up to 5 kHz, narrowband
  FM and SSB voice. Narrowband; the author notes transmitting needs an amateur
  licence.
- **Raw I/Q transmit engine (SRAM, 80 Msps)**: found by h0m3us3r, reported
  reproduced, but not used by that transmitter yet: research stage.

So the receive half of your idea is not just viable, it is *cheap*: a ~30 EUR
ESP32 dev board turns into a 2.4 GHz SDR with more bandwidth than the
Ubertooth One we have been using, and with no need to recognise any protocol.
The transmit half is the hard part: nobody publishes a wideband transmitter,
and even with one we would still have to reproduce the link (access code,
hopping, framing) *and* have the sink accept media from a transmitter that did
not negotiate the stream. That is why we are treating PHY emulation as a
research curiosity rather than a plan, and would rather spend the effort on
measurement first.

## What we know, and what we do not

Our evidence so far, from a sniffer (Ubertooth One) plus in-situ controls:

- With a phone held against the antenna and *audibly* streaming aptX Adaptive
  to a Sennheiser MOMENTUM 5, a 79-channel survey found **zero** packets for
  the phone's address across **307 s**; an independent single-address mode also
  saw zero and could not derive a UAP.
- The control rules out a blind instrument: the same host streaming aptX HD to
  the same headset (standard EDR) is seen at 0.7-1.6 hits/s, the host's own
  aptX Adaptive stream at 0.52 hits/s, and the same link idle at 0.02 hits/s.
  Had the phone's link emitted like the control, 215-460 hits were expected.
- What the sniffer cannot tell us is *what* the phone transmits. It cannot
  demodulate EDR payloads at all, and it only reports access codes it
  recognises. That is exactly the gap an ESP32 SDR closes.

## The measurement we plan to run

Four conditions, same board, same placement, ~1-2 minutes each:

1. phone idle (baseline);
2. phone playing aptX Adaptive to the headset (the case in question);
3. this host playing aptX HD to the same headset (known-standard reference);
4. this host playing aptX Adaptive (our own standard-EDR stream, second
   reference).

Criteria: does an emission appear that correlates with playback; how wide is
it (about 1 MHz and GFSK-like, or wider and not GFSK); does it hop like
BR/EDR? A standard-BR/EDR result would falsify our own "not on the standard
PHY" reading, which we would be glad to publish; a non-standard result would
pin the problem to the vendor link for good.

## Questions

1. Has anyone used ESP-SDR to look at Bluetooth (BR/EDR) traffic rather than
   Wi-Fi? Any board/settings advice for the 2.4 GHz ISM band, given the burst
   duty cycle?
2. For a continuous stream, is the ESP32-S31 streaming profile the only
   practical option, or is repeated burst capture with on-chip FFT enough to
   characterise an emission's bandwidth and cadence?
3. Is there any published, lawful way to *transmit* an arbitrary waveform on
   these chips that we should know about, given that ESPARGOS withholds one?
4. If someone has an ESPARGOS One or an S31 board and an aptX Adaptive
   headset, the capture above would settle a question that has been open in
   this PR for a while -- we are happy to share scripts and the raw captures.

We will post the measurements here either way, including if they contradict
what we have written above.
