# obake-shapes

Portable, containerized workloads ("shapes") for **obake**, a headless Linux
server for running user-defined audio effects and synthesis processes, plus
sequencers, MIDI processors, and generative control systems for MIDI and OSC.

| Path           | Purpose                                                     |
|----------------|-------------------------------------------------------------|
| `chuck/`       | ChucK audio programming language runtime                    |
| `mod-host/`    | mod-host LV2 plugin host with example plugins               |
| `rnbo-runner/` | RNBO OSCQuery runner (`rnbooscquery`)                       |
| `serialosc/`   | serialosc bridge for Monome grid controllers                |
| `siren/`       | SBCL Common Lisp environment for musical control and synths |

Each shape directory contains the Singularity definition file(s) that build
its workload.

This repository is consumed as the `shapes/` submodule of the top-level
`obake` repository.
