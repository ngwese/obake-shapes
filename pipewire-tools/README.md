# pipewire-tools

Minimal Alpine shape with the PipeWire CLI toolset (`pw-cat` / `pw-play` /
`pw-record`, `pw-top`, `pw-profiler`, `pw-cli`, and the rest of Alpine's
`pipewire-tools` package). The container is a client of the host PipeWire
socket; it does not start a daemon.

The app version pin tracks Alpine's PipeWire package (currently **1.6.9**,
matching the host from `host-pipewire`).

## Build

From the repository root (Singularity with Docker bootstrap):

```sh
singularity build --fakeroot \
  --build-arg-file shapes/pipewire-tools/build-args.txt \
  pipewire-tools.sif \
  shapes/pipewire-tools/pipewire-tools.def
```

## Run

As `obake` on a host with PipeWire already running for that user, use the
wrapper (binds `/run/user/<uid>` and sets `XDG_RUNTIME_DIR`):

```sh
# Image next to the wrapper, at the repo root, or via PIPEWIRE_TOOLS_SIF.
shapes/pipewire-tools/pipewire-tools pw-top
shapes/pipewire-tools/pipewire-tools pw-cli info
shapes/pipewire-tools/pipewire-tools pw-cat --help
shapes/pipewire-tools/pipewire-tools pw-profiler -h
```

Override the image path with `PIPEWIRE_TOOLS_SIF=/path/to/pipewire-tools.sif`
when the SIF is stored elsewhere.
