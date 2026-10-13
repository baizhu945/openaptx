Following up on the recipe in the comment above (source type 2, constant RTP
timestamp, non-zero target latency): I tested it end to end on an R2.2 sink and it
does not transfer, so the numbers may save someone else the experiment.

Setup: Sennheiser MOMENTUM 5 (advertises the R2 extension, feature word
0x0f000092), Intel AX210, BlueZ 5.87, PipeWire 1.7.0 with our aptX Adaptive codec
plugin. I implemented the constant timestamp as an option: with it enabled the
plugin sends `timestamp = samples per channel in this packet` (1200 = 0x4B0 for
our 25 ms / 676-byte frames at 48 kHz, upper 16 bits zero) and I set the AVDTP
configuration byte you describe as byte 6 to 0x12 (48 kHz + source type 2). Our
target-latency byte was already 0x64, i.e. 200 ms in 2 ms units, and our SSRC is
0; I did not try SSRC 0xA3.

Result, A/B in one sitting with everything else held identical (same headset
connection, same 440 Hz source, re-paired immediately before each leg, PipeWire
reporting `api.bluez5.codec = "aptx_adaptive"` in both legs):

| configuration | encoder output | air traffic (controller ACL counters) |
| --- | --- | --- |
| source type 2 + constant timestamp | 26.9 kB/s, 40 packets/s | **~0.9 kB/s, ~1.4 packets/s** |
| source type 0 + running timestamp | 27 kB/s, 40 packets/s | **27.4 kB/s, 40 packets/s** |

The encoder keeps producing frames at the right rate and period in both legs; what
changes is that with source type 2 the sink stops accepting them. This is not
silence-with-flow, it is a stall: the ACL counter collapses to about one 680-byte
packet per second and PipeWire logs a transport failure on that node. Neither leg
produced audio on this sink.

It is also not the timestamp mode. We hit the same stall earlier with source type
2 and a *running* timestamp, which is why the plugin had been sending source type
0 (0x10 at 48 kHz) since then. With source type 0 the stream runs cleanly at 40
packets/s, 676 bytes every 25 ms, and the sink still plays nothing -- the
behaviour this PR is about.

Two questions, since your sink is R1 and ours is R2.2 (the caveat you raised):

1. Did the Px7 ever stop accepting data during your bring-up, or did source type 2
   work first time?
2. Do you know whether the R2/R2.2 wrapper changes the meaning of the source-type
   bits? If source type 0 is what the offload path sets, it seems plausible that an
   R2.2 sink expects the wrapper's own timing and that flipping the bit cannot
   substitute for it.

The option and the A/B harness are in our tree if useful: commit 455da1b in
github.com/baizhu945/pipewire (`APTX_ADAPTIVE_TS_CONSTANT`, off by default), notes
in research/aptx-adaptive-qemu/HANDOFF.md section 21.34.
