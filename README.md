# MineOS-Simulator-of-scheduling-and-memory-in-Minecraft
A Minecraft (Java) server functions like a mini operating system: a 50 ms clock (the "tick"), thousands of "processes" (players, mobs, redstone), a "RAM" consisting of chunks, and a disk. MineOS applies concepts from the course—scheduling, virtual memory, and I/O—to help us understand why a server lags and which design decisions can resolve it.


The project has two halves that are connected via a CSV file:

Real Paper server ──(plugin: /mineos record)──▶ trace.csv ──▶ MineOS simulator ──▶ policy comparison 
▲ │ 
└────────────(plugin: /mineos policy tiered, measure with /mineos status)◀──────────────┘


1. Model
Tick loop. Tick *k* begins at `max(end of previous tick, k * 50 ms)`. If the workload exceeds 50 ms, the tick is extended (MSPT > 50) and the TPS drops, just as in a real server.

Tasks (`TaskType`), with associated static priorities: PLAYER(0), REDSTONE(1), ENTITY(2), CHUNK_TICK(3). Each task incurs a CPU cost and requires a chunk to be present in RAM.

Memory. Each chunk acts as a page, while RAM consists of page frames. Accessing a missing chunk triggers a page fault: a "victim" is selected (using FIFO, LRU, or the Clock algorithm) and the data is read from the disk (taking 1.5 ms, handled by 1–2 I/O threads). With asynchronous I/O, the task is blocked while the CPU switches to another task; with synchronous I/O (as in the vanilla version), the main thread waits.

Comparison of Policies

Axis	Options
Scheduler	FCFS (vanilla) · PRIORITY (4 levels + Round Robin + aging) · MFQ (3 levels with degradation and boosting)
Chunk replacement	FIFO · LRU · CLOCK
Architecture	vanilla (synchronous I/O) · async · budget (caps the tick at 50 ms) · prefetch-naive · prefetch-throttled
Tick budget. With a budget, the tick is capped at 50 ms, and any pending work is deferred to the next tick. If an entity generates a new task while a previous one is still pending, the new task is discarded (resulting in fewer updates for that entity). This maintains a TPS of 20, at the cost of reduced update frequency for less critical elements. Metrics: mean/p95/p99 MSPT, TPS, % of ticks > 50 ms, CPU and disk utilization, page faults, hit ratio, latency (creation → completion) by task type, % of discarded updates.

2. How to run it
Without Maven (JDK 21 only):

javac -d out/main $(find sim-core/src/main -name '*.java')
java -cp out/main edu.urosario.mineos.cli.Experiments                        # full matrix -> results/results.csv
java -cp out/main edu.urosario.mineos.cli.Experiments --scenario heavy       # a single scenario
java -cp out/main edu.urosario.mineos.cli.Experiments --scenario medium --sweep-frames   # thrashing curve
java -cp out/main edu.urosario.mineos.cli.Experiments --trace mi-traza.csv --frames 2000 # real traces
With Maven: `mvn test` and `mvn package` (tests use JUnit 4.13.2).

Scenarios: light · medium (≈95% CPU) · heavy (overloaded) · explorers (players using elytra; stresses memory). The same seed always produces the same trace.

3. Findings (see `docs/` and `results/results.csv`)
One seed per scenario; 1100 ticks measured after a 100-tick warm-up period. Large differences are robust; small ones (e.g., LRU vs. Clock, ~1%) fall within the range of noise and should not be overinterpreted.

Priority protects the player; FCFS does not. In the "medium" scenario, the player-latency p99 is 5.1 ms (Priority) vs. 48 ms (FCFS). In the "heavy" scenario, it is 7.3 ms vs. 110–123 ms. The cost is borne by low-priority tasks: `CHUNK_TICK` reaches a p99 of 271 ms and loses 47% of its updates.
MFQ does not prioritize what is important—only what is short. Most tick-related work consists of small tasks, so these operations wait in the same queue as everything else (p50 of 19 ms in the "medium" scenario, compared to 2.6 ms with "Priority" scheduling). Meanwhile, "heavy" workloads are effectively starved: with a budget applied to the "heavy" category, Redstone processing reaches a p99 of approximately 7 seconds, even with a performance boost.
Budgeting transforms a total system collapse into a state of controlled degradation. In a "heavy" scenario without a budget, the server drops to 9 TPS; with a budget, it maintains 20 TPS, though it discards 56–66% of entity updates.
Synchronous I/O is the most costly operation. In the "medium" scenario (1,500 frames), vanilla Minecraft hits 70 ms/tick and 14.3 TPS, whereas using asynchronous I/O results in 46.8 ms/tick and 20 TPS.
LRU ≈ Clock ≪ FIFO. FIFO causes between 42% and 100% more page faults, depending on the scenario ("light" +42%; "medium" +62%—27,501 vs. 16,977; "explorers" +69%—10,986 vs. 6,493; "heavy" +100%). The Clock algorithm achieves results comparable to LRU while requiring only a single bit per frame.
Thrashing acts as a cliff rather than a gradual slope. In the "medium" scenario (using LRU), at 100% of available RAM, the server maintains 20 TPS with 23% disk utilization; at 80% RAM, it stays at 19.9 TPS but disk utilization jumps to 93%; at 60% RAM, it falls to 11.7 TPS; and at 40% RAM, it drops to 4.7 TPS.
Naive prefetching actually degrades server performance. It saturates the disk, forcing necessary read operations to wait in a queue (in the "explorers" scenario, the p99 tick time reaches 33 ms, compared to 18 ms without prefetching; 5,077 useful reads vs. 4,704 wasted reads). With throttling (preloading only when the disk is idle), page faults decrease by 36% (6,493 → 4,181) without penalizing the tick duration (18.4 ms), and nearly all preloaded data is utilized (2,325 / 65).
An effective CPU scheduler can negatively impact disk performance. Under memory pressure, the "Priority" scheduler defers tasks with the highest fault rates; consequently, their read requests are issued only when the CPU has no other pending tasks. While the number of page faults remains the same, the tick duration increases to 31.5 ms, compared to 19.4 ms with the MFQ (Multi-Level Feedback Queue) scheduler under a light load. This is why real-world systems prioritize I/O-bound processes.
4. Limitations (relevant to the report)
CPU costs are synthetic (based on fixed distributions) rather than empirically measured. The qualitative conclusions—such as which approach performs best, why, and where performance sharply degrades—are robust; however, the absolute figures do not represent any specific server environment. A real-world trace would be required to replace these synthetic costs.
Only one random seed is used per scenario; therefore, no confidence intervals are provided.
The simulation uses a single CPU thread; parallel execution across regions (as seen in Folia) is not modeled.
Entities and Redstone components are static; only players move. The writing of "dirty" chunks to disk during eviction is not modeled.
I/O latency is assumed to be constant (i.e., it does not account for variations between SSDs/HDDs or operating system caching).
In a real-world server, the tick duration also encompasses network processing, plugins, and garbage collection (GC)—none of which are present here.
5. Verification
The `sim-core` component compiles using `javac` (JDK 21) with no warnings.
All 31 JUnit tests pass. These include textbook-style reference strings with known outcomes (Silberschatz: FIFO 15 / LRU 12 / Clock 14 page faults; Belady’s anomaly: FIFO 9→10 with more frames, LRU 10→8), manually calculated timing scenarios (synchronous vs. asynchronous I/O), budget-based termination, update discarding, and reproducibility.
Exporting a trace to CSV and replaying it using the `--trace` flag yields results identical to a direct execution.
Not verified: a full `mvn package` (the development environment could not download dependencies; validation was limited to `mvn -o validate`) and the entire `paper
-plugin` module.



6. Paper Plugin (experimental)
Compile it (requires access to repo.papermc.io) using `mvn -Pplugin package`, then copy the .jar file from `paper-plugin/target` into the `plugins/` folder of a Paper 1.21.x server.

Command	Function
/mineos record [ticks]	Records players and mobs near a player into `plugins/MineOS/trace-*.csv`
/mineos status	Displays average/p95/p99 MSPT and the percentage of ticks exceeding 50 ms (over the last 20 seconds)
/mineos policy tiered / off	Enables/disables the tiered policy (adjusts mob AI frequency to 1, 1/2, or 1/4 of the normal rate based on distance from the player)
Validation procedure: Using the same server and load, run `policy off` → wait 20 seconds → check `status`; then run `policy tiered` → wait 20 seconds → check `status`. Compare the MSPT values.

Important limitations: Bukkit does not expose the CPU cost per entity; therefore, the recorded cost is an *a priori* model, scaled so that the modeled average tick time matches the measured one (this calibrates the total, not the individual distribution). Note that `REDSTONE` and `CHUNK_TICK` events are not recorded. Additionally, the policy relies on `Mob#setAware`; you should verify its availability and actual effect for the specific version of Paper you are using.
