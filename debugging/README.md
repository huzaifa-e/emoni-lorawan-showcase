[← Overview](../README.md)

# <img src="../media/icons/debugging.png" width="40" alt="" align="top"> Debugging

Tracking says *that* something is wrong. These tools say *why*, from the desk, for a device that is in a basement somewhere.

## Failure log

<p align="center">
  <img src="images/failure-log.jpg" width="900" alt="Upload failure log: count of late uplinks, expected interval, total delay, and one entry per late uplink with expected and actual time, the cause and the event codes and battery of that uplink">
</p>

Every uplink that arrived later than the device's interval allows becomes an entry: expected and actual gap, the previous uplink, the event codes and the battery of that uplink, and a **cause**. The cause is worked out from the data:

* recognition failed, or the raw value contains digits the device could not read,
* a digit roll-over artefact (a wheel caught between 9 and 0 on a mechanical counter),
* the value went down, or rose faster than the meter can physically allow,
* a firmware event code decoded into its meaning,
* otherwise *cause unknown*. In the screenshot the readings and codes are fine, so the next place to look is the radio side: this unit was in the middle of a firmware update over the air at the time, which the link-health panel below shows.

## Link health

<p align="center">
  <img src="images/link-health.jpg" width="900" alt="Link health over 7 days: ACKs answered, link margin worst 10 %, rejoins per day, uplinks received, SNR per gateway hourly, and a gateway table">
</p>

Four numbers decide whether a radio problem is the network's or the device's:

| Tile | Meaning |
|---|---|
| **ACKs answered** | share of confirmed uplinks the network acknowledged. low ACKs with high delivery point at the downlink side (gateway duty cycle, FUOTA traffic) |
| **Link margin, worst 10 %** | how much SNR is left over what the data rate needs, on the bad days |
| **Rejoins per day** | how often the device gave up on its session ([wake cycle](../device/README.md#one-wake-up)) |
| **Uplinks received** | from the frame counter: what was sent against what arrived |

Below that, SNR per gateway per hour, and a table of every gateway that heard the device.

## Payload explorer and anomalies

<p align="center">
  <img src="images/anomalies-payload.jpg" width="900" alt="IMU acceleration, the anomaly log with re-baseline events, and the payload explorer plotting one decoded field over time">
</p>

* **Payload explorer**: any field the decoder produced, plotted over time. For a question nobody built a chart for yet.
* **Anomaly log**: every overshoot, manual and automatic re-baseline, with the jump, the threshold and what was done.
* **IMU**: tilt and movement, to tell a device that was knocked from one that misreads.

## Connectivity tests

The platform can only explain a late uplink if the firmware's network behaviour is known. Connectivity tests **change the network under a live device on purpose** and record how it copes and what it costs. A runner on the [bench host](../remote-lab/README.md) drives the self-hosted network server's API:

<p align="center">
  <img src="../diagrams/act-conn-test.svg" width="560" alt="Activity diagram: save settings, apply each step, observe, check recovery, restore, push the metrics, report">
</p>

| Step | What the server does | What it verifies |
|---|---|---|
| Observe | nothing | the baseline: readings, confirmed uplinks, data rate, SNR, airtime |
| Session deleted | forgets the device's session | the device notices and rejoins, and how long that takes |
| Downlinks blocked | sends no ACKs and no join accepts | probes, rejoin attempts and back-off, and recovery once downlinks return |
| Fixed data rate | forces a spreading factor and TX power | delivery and airtime at the slowest and fastest data rates |
| Test downlinks | sends harmless unknown commands | the firmware ignores them safely |

<p align="center">
  <img src="images/conn.gif" width="900" alt="A connectivity test run: the uplink timeline coloured by session and the metrics per step">
</p>

<p align="center">
  <img src="images/conn-tests.jpg" width="49%" alt="Connectivity test runs with their steps as chips">
  <img src="images/conn-run.jpg" width="49%" alt="One run: uplinks on a timeline coloured by session, and the metrics of each step">
</p>

Each uplink sits on a timeline coloured by session, with a dashed line at every new join. In the run shown, the device rejoined 2 minutes after its session was deleted and recovered 5 minutes after downlinks came back, as the published rules say it should. Settings are always restored, even when a run is stopped early.

## Power tests

<p align="center">
  <img src="images/power-tests.jpg" width="900" alt="Deep sleep by run: one bar per bench recording, log scale">
</p>

Every recording from the [remote lab](../remote-lab/README.md) lands on `/power` by itself. A run page shows:

* the whole recording on a log scale, so a 100 mA wake and a 28 µA sleep fit on one chart,
* the **typical wake** split into phases (start-up and recognition, the radio burst, the receive windows), each with its time and charge,
* the **deep sleep**: raw samples, one converter burst, the distribution of the samples, and the drift of each sleep over the run,
* **battery life** at 5 min to 6 h intervals,
* every wake in a table, and the device's console log next to the samples.

<p align="center">
  <img src="images/power-run.jpg" width="900" alt="A power run: sleep mean 28.3 µA, 22 wakes, the whole recording, a typical wake, the deep-sleep detail and the battery life table">
</p>

A supply sweep shows each voltage step separately, with the sleep current, power and wake time per step:

<p align="center">
  <img src="images/supply-sweep.jpg" width="900" alt="A supply sweep run, one chart per voltage step">
</p>

## The AI debugger

On any device page, a **Debugger** button opens a panel where you ask a question in plain language. The server runs an AI agent next to the data:

<p align="center">
  <img src="../diagrams/seq-ai-debugger.svg" width="100%" alt="Sequence diagram: question, context, agent loop with read-only SQL and source reads, streamed trace, optional question back to the developer, answer, report and notes saved">
</p>

From a saved report, shortened (a battery question on a field unit):

> **Verdict.** The selected window is the cleanest six hours this device has had in weeks: every uplink on time, no recognition errors. There is no elevated drain to explain, and the window is too short to measure drain anyway. The real findings are outside it: a cadence change that doubled the uplink load, and a recurring battery spike artefact.
>
> **Battery.** Single-sample jumps of +900 to +1700 mV occur 38 times in the history, each followed straight away by a return to the previous level. Only three jumps are real pack swaps, each preceded by a depleted reading and a multi-hour gap. Real drain, from daily medians: about 0.85 mV per uplink, the same as two packs ago.
>
> **Runtime.** Use the empirical floor, not the nominal empty voltage: earlier packs collapsed within a day once they left the plateau. Plan a pack swap within about two weeks and re-check the trend in 3–4 days.

The report separates artefacts from real pack swaps, works on daily medians and states what the selected window can show. More in [AI in the loop](../ai/README.md).
