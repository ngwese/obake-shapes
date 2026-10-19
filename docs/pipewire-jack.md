# JACK shapes on host PipeWire

How to build and run a Debian Trixie Singularity shape whose JACK application
talks to the host PipeWire graph. The host from `host-pipewire` runs PipeWire
**1.6.9**; Trixie ships a **1.4.2** client, which is the supported direction
(older client, newer server), including JACK MIDI.

`chuck` and `rnbo-runner` follow this guide. `mod-host` and `siren` still use
Bookworm; convert them the same way when ready.

## Build

Use Trixie for both stages:

- build: `debian:trixie`
- final: `debian:trixie-slim` (or `debian:trixie` if the final stage needs a
  fuller toolchain)

At **build** time, install the JACK **headers** only:

- `libjack-jackd2-dev` (and the rest of the application's build deps)

Do **not** install the `jackd2` server package. Link the application as a
normal JACK client (`-ljack` / `pkg-config jack`).

At **runtime** (final stage), install:

- `pipewire-jack` (pulls `libpipewire-0.3-0` and `pw-jack`)

On Trixie, the ALSA runtime library is `libasound2t64` (not `libasound2`).

The runscript should be the application under `pw-jack`, for example:

```
%runscript
	pw-jack /usr/local/bin/my-app "$@"
```

`pw-jack` prepends PipeWire's private `libjack.so.0` to `LD_LIBRARY_PATH`.
The process must not start `jackd`, `jackdbus`, or a PipeWire daemon inside
the container; the host already owns the graph. Packages that happen to ship
unit files for those daemons stay inert under Singularity.

## Run

As `obake`, bind the user runtime directory (where the host socket lives) and
set `XDG_RUNTIME_DIR`:

```sh
runtime="/run/user/$(id -u)"
singularity run \
  --bind "$runtime" \
  --env "XDG_RUNTIME_DIR=$runtime" \
  my-shape.sif
```

The client opens `$XDG_RUNTIME_DIR/pipewire-0` on the host. Without that
bind, the container cannot see the socket.

## ALSA

- **Sequencer MIDI** (`/dev/snd/seq`): Singularity already binds `/dev`.
  Clients that only use the ALSA sequencer do not need `pipewire-jack`.
- **ALSA PCM**: install `pipewire-alsa` in the final stage and use the
  PipeWire default PCM. Those clients need the same runtime-directory bind
  as JACK clients. The ChucK and RNBO images in this repository are JACK
  clients and do not install `pipewire-alsa`.
