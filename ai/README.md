[← Overview](../README.md)

# <img src="../media/icons/ai.png" width="40" alt="" align="top"> AI in the loop

The AI agent has the same access I do: the dashboard's data through MCP, the bench through SSH, the firmware and the documents. It starts recordings, waits for the device to fall asleep, analyses the samples, reads the console, writes the next test build and asks me for the one thing it cannot do itself. I keep the soldering iron, the flash button and the decision about what ships.

<p align="center">
  <img src="images/session.gif" width="900" alt="Replay of a bench session: PPK2 runs over SSH, a bare sleep build matching the datasheet, a pin register dump that shows the leak">
</p>

<p align="center"><sub>A bench session of 12 Nov 2026, condensed from its transcript. Commands, numbers and times are real; hardware specifics are generalised.</sub></p>

## Two ways in

| | MCP endpoint | Built-in debugger |
|---|---|---|
| Where | any MCP client (a desktop assistant, a coding agent) | the **Debugger** panel on every device page |
| Auth | OAuth or a personal token | dashboard sign-in |
| Runs | on the client | an AI agent harness inside the platform's container |
| Sees | 26 tools over the fleet | the same tools plus the dashboard and firmware source with git history |
| Leaves | `[mcp]` notes on what it changed | a saved report and distilled notes on the device |

### MCP tools

| Tool | What it returns |
|---|---|
| `emoni_overview` | every device: status, last uplink, battery, latest value. The starting point |
| `emoni_analysis` | what stands out: offline or late devices and why, firmware errors, recognition misses, battery drain, anomalies |
| `emoni_device`, `emoni_uplinks`, `emoni_events` | one device in full; raw uplinks with decoded events |
| `emoni_battery`, `emoni_readings`, `emoni_consumption` | discharge cycles and days to empty; readings; consumption per hour, day, month, year |
| `emoni_conn_tests`, `emoni_conn_test` | connectivity-test runs, step by step |
| `emoni_notes`, `emoni_reports`, `emoni_anomalies` | field knowledge, saved analyses, the anomaly log |
| `emoni_query` | read-only SQL for anything else; secret columns refused |
| `emoni_source_read`, `_search`, `_history`, `_read_at` | source, with git history |
| `emoni_note_add`, `emoni_device_set`, … | a few small writes, each leaving a note |

**Not offered, on purpose:** downlinks, deletes, counter resets, account links. Anything that changes a device in the field waits for a person in the dashboard.

## Who did what on the bench

| The AI agent | Me |
|---|---|
| read the documentation, the firmware and the vendor SDK source | moved probes and leads, re-soldered a part for a branch measurement |
| wrote the test builds: phase rotations, a register dump, a bare sleep build | built and flashed them |
| started and stopped recordings on the bench host, analysed the raw samples | calibrated the device on its meter, pressed the button when asked |
| waited for "30 s of deep sleep" with a background monitor | read test points with a multimeter when asked |
| pushed each run and its notes to the dashboard | decided what a result meant for the product, and what ships |

From the agent's transcripts of the firmware work, June to November 2026:

| | |
|---|---|
| SSH commands on the bench host | **662** |
| power recordings, analyses and exports | **462** |
| MCP calls to the dashboard | **409** |
| console reads | **294** |
| connectivity-test operations | **97** |
| builds or flashes by the agent | **6**: I did the rest |

## One exchange, shortened

> **me:** the test setup is ready, run the tests
>
> **agent:** *(starts a recording on the bench host)* Still about 0 µA. No current is flowing through the profiler at all.
>
> **me:** tell me in terms of wires
>
> **agent:** With those colours the leads are most likely swapped. On the profiler cable red is the output and brown the input.
>
> **agent:** The wake current comes through now, but in deep sleep it reads exactly 0 µA, and this chip cannot sleep at zero. In series, the shunt puts its rail below the rest of the board, so it feeds itself through its pins from the pull-ups. Let's power the branch from the profiler instead.
>
> **agent:** *(waits for 30 s of deep sleep, analyses)* The microcontroller alone sleeps at 64.4 µA. Its datasheet says about 8.

Then: a bare sleep build at 8.4 µA (the chip is fine), a dump of every pin register before sleep, the wake line's internal pull-down found fighting the board's pull-up, a phase test, and the question that decides the cost:

> **me:** do we need a revision of the schematic or can this be handled in firmware?
>
> **agent:** Firmware is enough: switch the wake line's internal pulls off before every deep sleep. The external pull-up still holds the line and the button still wakes the device. An earlier board had the button on a pin without this default; the redesign moved it.

1 h 24 min passed between "run the tests" and the first full-board sleep at 30 µA. That included rewiring the profiler twice, a recalibration, re-soldering a part, and four test builds.

## What makes it work

* Every bench action is a command on the host, so the agent can measure and analyse in the same turn.
* The dashboard keeps each run with its setup, firmware, notes and waveforms, so a later session starts from recorded numbers.
* Every number is checked for physical plausibility before the work moves on ("a chip cannot sleep at 0 µA", "the same power at two input voltages means the load is behind the converter"). That is how a false 4× improvement and a 0 µA reading were caught.
* The agent asks for a wire to be moved or a build to be flashed. It has no access to field devices, and I decide what ships.
