# iSimulate Bridge

AMM/MoHSES module for connecting to an iSimulate monitor on the local network.

---

## Fork context — why this repository is kept (added 2026-09-12)

This is an **unmodified fork of third-party upstream code**: the MoHSES iSimulate
Bridge from the University of Washington CREST lab (Rainer Leuschke,
`rainer@uw.edu`), last changed upstream on 2023-12-14. Nothing here has been
altered, and nothing here should be — see *Modification policy* below.

It is retained because it is, as far as we can establish, **the only written
record anywhere of the iSimulate REALITi control protocol**:

| What it establishes | Where |
|---|---|
| mDNS/DNS-SD discovery via `_realiti_v1._tcp` | `src/service_discovery.c` |
| WebSocket transport, target `/`, JSON text frames | `src/websocket_session.cpp` |
| The full connect handshake and 14 packet types | `src/iSimulateBridge.cpp` |
| The REALITi monitor model IDs (25 named, in the range 1–28) | `src/cl_arguments.c` |

Those 1,135 lines are the sole reference behind
**[`simlink/docs/REALITI-PROTOCOL.md`](https://github.com/drseanwing/simlink/blob/main/docs/REALITI-PROTOCOL.md)**,
a language-neutral extraction of the protocol with a citation for every claim,
and behind the REALITi adapter and mock in
[`simlink/adapters/`](https://github.com/drseanwing/simlink/tree/main/adapters).

### Where this sits in the wider system

```
MoHSES / AMM  ──DDS──▶  this bridge (C++)  ──WS──▶  iSimulate REALITi monitor
                                                          ▲
Laerdal qCPR  ─┐                                          │
SimMan/LLEAP  ─┴─▶  SimLink canonical parameter bus  ──────┘
                    (TypeScript, drseanwing/simlink)
```

This bridge serves MoHSES. The SimLink bus serves the Laerdal side. They reach
the same REALITi hardware by the same protocol, which is why extracting that
protocol out of C++/DDS mattered. See
[`simlink/docs/INTEGRATION-ARCHITECTURE.md`](https://github.com/drseanwing/simlink/blob/main/docs/INTEGRATION-ARCHITECTURE.md).

### What this code does NOT tell us

Recorded in full in the *"Not evidenced by the reference"* section of
`REALITI-PROTOCOL.md`. The two that matter:

- **No CPR ingest.** `SettingsPacket` carries `seeThruCPR` and
  `cprDepthMeasureInch`, and `ChangeActionPacket` carries `ventilated` and
  `perfusion`, so REALITi clearly has CPR-aware display modes — but this bridge
  contains **no packet that feeds CPR telemetry in**, because MoHSES has no CPR
  source. qCPR → REALITi CPR display therefore cannot be built from this
  evidence. Closing it needs a WebSocket capture of iSimulate's own CPR-capable
  controller.
- **Waveform IDs are almost entirely unknown.** `ecgWaveform` evidences only
  two values — `9` = sinus (`src/iSimulateBridge.cpp:53`, corroborated at
  `:304`) and `14` = ventricular tachycardia (`:388`) — against 33 members of
  Laerdal's `BasicRhythmEnum`. `bpWaveform`/`spo2Waveform` `0` = normal and
  etCO2 `0`/`1`/`2` = normal/obstructive-1/obstructive-2 are also evidenced.
  The two known ECG IDs are not ordered by clinical family (9 sinus, 14 VT,
  10–13 unknown), so the remaining 31 morphologies cannot be interpolated and
  are not recoverable from this source.

### Modification policy

Treat this repository as **read-only reference**. Interoperability work belongs
in `simlink`, not here — a fork that diverges from upstream loses its value as an
independent protocol witness. If upstream publishes changes, rebase rather than
patch.

### Provenance

`LICENSE.md` carries a bare copyright line (`Copyright (c) 2023 University of
Washington, CREST`) with no explicit grant of rights attached. Confirm terms with
the upstream authors before redistributing this code or anything derived from it.
The protocol extraction in `simlink` is a clean-room-style interoperability
description — facts about a wire format, not copied source — but the underlying
permissions question is worth settling explicitly rather than assuming.

## Dependencies

The iSimulate Bridge requires the [AMM Standard Library](https://github.com/AdvancedModularManikin/amm-library) be built and available (see AMM lib dependencies).
The iSimulate Bridge module also requires:

- avahi-client
- avahi-common

`$ sudo apt install libavahi-client-dev`

## Installation

```bash
    $ git clone https://github.com/DivisionofHealthcareSimulationSciences/isimulate-bridge.git
    $ cd isimulate-bridge
    $ mkdir build && cd build
    $ cmake ..
    $ cmake --build . --target install
```

## Contact
Contact Rainer Leuschke (rainer@uw.edu) with any questions.
