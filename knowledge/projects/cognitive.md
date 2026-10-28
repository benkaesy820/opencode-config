## cognitive — from-scratch Rust "cognitive" model trainer

- 2026-10-25 — **parked by Ben's decision; the project directory is deleted and nothing is running.** Built and measured: a ternary-weight recurrent byte-level English model that beat a bigram on both ranking and likelihood (accuracy 0.4035 = 1.57x bigram, cross-entropy 2.1043 nats), crossing the trigram and approaching the 4-gram bar at 1.7521 nats. The design followed from the hardware — 2-bit weights are 16x less traffic than fp16, which is what lets a model fit in this box's RAM at all; compute, not method, was the wall on scale. The idea, the architecture, every measurement and the resume list live in `cognitive.md` in this directory; the code is archived at `~/cognitive-2026-10-25.tar.gz`. Do not treat this as live work.

