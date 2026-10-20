# ptp-tools

Minimal Alpine shape with PTP and hardware-timestamping tools for
AES67/RAVENNA bring-up: `ethtool`, the linuxptp suite (`ptp4l`, `phc2sys`,
`pmc`, `phc_ctl`, `hwstamp_ctl`, …), `tcpdump`, and `ip`. Sample `ptp4l`
configs live under `configs/` in the tree and at `/opt/ptp-tools/configs`
inside the image.

The shape is an operator toolkit. It does not start a host PTP service and
does not install an AES67/RAVENNA audio stack.

The app version pin tracks Alpine's `linuxptp` package (currently **4.4**).

## Build

From the repository root (Singularity with Docker bootstrap):

```sh
singularity build --fakeroot \
  --build-arg-file shapes/ptp-tools/build-args.txt \
  ptp-tools.sif \
  shapes/ptp-tools/ptp-tools.def
```

## Run

The wrapper uses the host network namespace (Singularity default), binds
`/dev` so `/dev/ptp*` is visible, and adds `CAP_NET_ADMIN` and
`CAP_SYS_TIME` for hardware timestamping and clock adjustment. Your
Singularity/Apptainer install must allow those capabilities (often requires
a setuid or rootful install).

```sh
# Image next to the wrapper, at the repo root, or via PTP_TOOLS_SIF.
shapes/ptp-tools/ptp-tools ethtool -T eth0
shapes/ptp-tools/ptp-tools phc_ctl /dev/ptp0 caps
shapes/ptp-tools/ptp-tools ptp4l -f /opt/ptp-tools/configs/aes67-client.cfg \
  -i eth0 -m
```

Override the image path with `PTP_TOOLS_SIF=/path/to/ptp-tools.sif` when the
SIF is stored elsewhere.

## AES67 / RAVENNA bring-up ladder

Replace `eth0` with the studio NIC. These steps assume a host that already
has PTP/PHY timestamping in the kernel.

1. Probe timestamping and the PHC:

   ```sh
   shapes/ptp-tools/ptp-tools ethtool -T eth0
   shapes/ptp-tools/ptp-tools hwstamp_ctl -i eth0 -t 1 -r 1
   shapes/ptp-tools/ptp-tools phc_ctl /dev/ptp0 caps
   shapes/ptp-tools/ptp-tools phc_ctl /dev/ptp0 get
   ```

2. Lock an AES67 client to the studio GM (or use `hwts-client.cfg` /
   `swts-client.cfg` while debugging):

   ```sh
   shapes/ptp-tools/ptp-tools ptp4l \
     -f /opt/ptp-tools/configs/aes67-client.cfg -i eth0 -m
   ```

3. Query PTP datasets (another terminal while `ptp4l` runs):

   ```sh
   shapes/ptp-tools/ptp-tools pmc -u -b 0 'GET CURRENT_DATA_SET'
   shapes/ptp-tools/ptp-tools pmc -u -b 0 'GET TIME_STATUS_NP'
   ```

4. Optionally sync the system clock to the PHC once locked:

   ```sh
   shapes/ptp-tools/ptp-tools phc2sys -s eth0 -c CLOCK_REALTIME -w -m
   ```

5. Optionally capture PTP on the wire (UDP 319/320), with adapter timestamps
   when the NIC supports them:

   ```sh
   shapes/ptp-tools/ptp-tools tcpdump -i eth0 -j adapter \
     'udp port 319 or udp port 320'
   ```

## Sample configs

| File | Path in image | Role |
|------|---------------|------|
| `hwts-client.cfg` | `/opt/ptp-tools/configs/hwts-client.cfg` | Minimal HWTS client |
| `swts-client.cfg` | `/opt/ptp-tools/configs/swts-client.cfg` | Software-TS client |
| `aes67-client.cfg` | `/opt/ptp-tools/configs/aes67-client.cfg` | AES67/RAVENNA client |
| `lab-gm.cfg` | `/opt/ptp-tools/configs/lab-gm.cfg` | Scratch-LAN lab GM |

**Warning:** `lab-gm.cfg` is for a scratch LAN only. Do not run it on a
studio AES67/RAVENNA PTP network.
