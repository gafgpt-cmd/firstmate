# Home layout reference

AXI-steno. Detailed operational-home file inventory offloaded from `AGENTS.md` §2.
Schema owner: `docs/configuration.md`. Child fields + mutation: each producing script header/help.
A `state/<id>.status` line = wake event, not current-state truth; `bin/fm-crew-state.sh` owns reconciliation.

## Tracked (shared template)

```
AGENTS.md            job description (CLAUDE.md = real @AGENTS.md pointer)
CONTRIBUTING.md      contributor workflow + repo conventions
README.md            public overview + dev notes
.github/workflows/   shared CI + PR enforcement, committed
.tasks.toml          tasks-axi markdown backend config, default backlog backend (§10)
.agents/skills/      firstmate-loaded internal skills, committed; metadata.internal=true
.claude/skills       symlink -> .agents/skills (claude compat)
skills/              standalone public installer-facing skills, committed; NOT loaded by firstmate
bin/                 helper scripts, committed; read each header before first use
```

## Captain-private, gitignored

```
.env                 optional Relay pairing token; presence-gates §14
projects/            cloned repos; read-only except hard-rule-1 concrete captain-approved op
.no-mistakes/        local validation state + evidence
```

## config/ (LOCAL, gitignored operating choices)

```
crew-harness          crewmate harness override; absent/"default" = same as firstmate. Inherited literal: concrete primary adapter also controls secondmate home crewmates (§4)
crew-dispatch.json    optional per-task harness/model/effort profiles; human-editable NL rules (§4). Inherited by secondmates
secondmate-harness    harness PRIMARY uses to launch SECONDMATEs: "<harness> [<model>] [<effort>]" (§4); absent/"default" -> crew-harness -> firstmate own. NOT inherited (secondmates don't spawn secondmates)
backlog-backend       absent/"tasks-axi" = default; "manual" = force hand-editing (§10). Inherited
backend               runtime session-provider backend for new tasks; absent = auto-detect -> tmux. tmux = verified reference (docs/tmux-backend.md); herdr/zellij/orca/cmux experimental (respective docs); herdr+cmux auto-detectable, zellij+orca always explicit, codex-app NOT accepted (docs/codex-app-backend.md). Inherited per secondmate-provisioning
calm                  Pi Calm presentation preference; NOT inherited; docs/configuration.md "Pi Calm preference"
supervision-branch-model supervision-branch-effort  Pi supervision-branch model + reasoning-effort pins written by /supervision-model; independently settable; NOT inherited; docs/configuration.md "Pi supervision branch model and effort"
stow-pass-horizon     optional presence flag opting home into /stow default-off pass-count decay horizon; NOT inherited; docs/configuration.md "Stow pass horizon"
watched-tools.json    optional list of tools this home depends on, read by update check armed with bin/fm-tool-update-check.sh; firstmate-maintained but human-editable; NOT inherited; docs/configuration.md "Watched tool updates"
startup-memory-budget primary-authoritative per-home; materialized 7,500 est tokens by locked primary bootstrap; inherited; docs/configuration.md "Startup memory budget"
herdr-presentation-spaces  "off"/"on" for Herdr default-on disposable single-task visual projection (default-on only >= Herdr version floor); inherited; docs/herdr-backend.md
trace-context         presence flag = default-off native W3C trace-context propagation to spawned agents; inherited; docs/configuration.md + docs/trace-context.md
turnend-churn-absorb  optional presence flag opting home into default-off absorb of bare turn-end wakes on pane churn; NOT inherited; docs/configuration.md "Turn-end pane-churn absorb"
cmux-socket-password  optional cmux control-socket password; read fresh per cmux CLI call; never overrides ambient CMUX_SOCKET_PASSWORD when absent (docs/cmux-backend.md)
wedge-alarm           optional away-mode active-alert directives; absent = auto (macOS Notification Center when available); docs/wedge-alarm.md
x-mode.env            generated Relay watcher cadence; source before arming watcher when present
```

## data/ (LOCAL personal fleet records, gitignored whole)

```
backlog.md         task queue, dependencies, history
captain.md         domain-local captain preferences + working style; canonical even if harness memory mirrors; inspect-then-update
captain-shared.md  main-authoritative shared captain preferences, read-only to secondmates; owned by secondmate-provisioning
learnings.md       fleet-local operational facts/gotchas; dated, evidence-backed, curated; rewrite+prune not append; created lazily
projects.md        thin fleet nav registry, per-project standing delivery posture; firstmate-private; parsed by fm-project-mode.sh (§6)
secondmates.md     local+remote secondmate routing table; firstmate-private; maintained by secondmate seed helpers (§6)
<id>/brief.md      per-task crewmate brief, or per-secondmate charter brief when kind=secondmate
<id>/report.md     scout deliverable, written by crewmate; survives teardown
```

## state/ (runtime records + signals, gitignored)

```
<id>.status        crewmate-appended "<state>: <note>" wake-event lines, NOT current-state truth
<id>.turn-ended    touched by turn-end hooks
<id>.grok-turnend-token   firstmate-owned grok hook registry token; removed by teardown
<id>.kimi-turnend-token   firstmate-owned Kimi hook registry token; removed by teardown
<id>.gemini-settings.json  firstmate-owned per-task Gemini settings carrying busy-state + turn-end hooks, reached via GEMINI_CLI_SYSTEM_SETTINGS_PATH so nothing is written into the project's own .gemini/; removed by teardown
<id>.muse-session  muse busy-source binding (sessions root + task worktree); fm-spawn writes, teardown removes
<id>.cursor-session  cursor busy-source binding (projects root, task worktree, prior conversations); fm-spawn writes, teardown removes
<id>.reconcile-nudged  epoch second of last inventory-reconcile nudge sent to this secondmate; bin/fm-secondmate-reconcile.sh owns its per-home cooldown window
<id>.backlog-close  exact backlog transition a teardown recorded before removing the task's record, so an interrupted cleanup can still finish at next session start; bin/fm-backlog-transition-lib.sh owns format+replay; a landed transition removes it
<id>.inbox/        durable steering inbox: sequenced firstmate instruction records worker acks by moving to its handled/ subdir; written by fm-send; ordinary records re-rung+escalated by watcher, explicit fire-and-forget records excluded from that ladder; removed by teardown (bin/fm-task-inbox-lib.sh)
<id>.meta          task metadata; producer script header owns fields+contract; docs/configuration.md routes backend+trace-context
<id>.herdr-presentation  quarantinable attempt+restart-binding journal for Herdr visual projection; never task/endpoint authority; docs/herdr-backend.md
<id>.check.sh      authenticated slow poll; watcher dispatches validated PR data + byte-identified Relay shim via trusted repo scripts, runs registered custom checks from hash-validated private snapshots, rejects every other state check without execution
<id>.check-trust   private content binding by fm-check-register.sh for intentional custom check
<id>.pr-poll       private validated data sidecar for byte-static PR merge poll
<id>.pr-poll-registration  private transactional provenance: task + canonical metadata identity + sidecar + static poll publication
<id>.pr-poll-retirement  private identity-bound crash-recovery receipt for one exact validated merged result; removed after poll artifacts retire
<id>.pr-poll-merge-notified  canonical PR identity of last merge outcome delivered for this task; bin/fm-pr-lib.sh owns marker format + identity mechanics; bin/fm-merge-outcome-lib.sh owns locked publication, duplicate suppression, replacement
branch-outcomes.jsonl .branch-outcomes-cursor .branch-outcomes-processed .<task>.branch-outcome-index .branch-outcome-index-ready  Pi supervision-branch durable outcome store, its read cursor, main's processed marker, bounded latest per-task status-coverage caches, their recovery marker; bin/fm-branch-outcome.sh owns formats
branch-session/ .branch-session .branch-mirror-cursor  branch's per-main-session conversations, pointer to the current one, dialog-mirror cursor; extension-owned (docs/pi-supervision-branch.md)
.branch-eligible-rows .branch-eligible-owner .main-eligible-rows  per-actor wake-row claims + branch-owner evidence; docs/watcher-continuity.md owns the ack contract
.lease-<task>      per-task supervision lease naming which actor (main/branch) may change that task; bin/fm-lease-lib.sh owns contract guarded scripts enforce
x-watch.check.sh   generated Relay poll shim; present only when opted in (§14)
tool-updates.check.sh  generated watched-tool update poll shim + its .check-trust binding; present only after bin/fm-tool-update-check.sh arm; its report record .tool-updates keeps one pending update from being reported on every poll
pending-replies/   parent-owned secondmate pending-reply records (correlation id, delivery vs reply, recovery, escalation); fm-pending-reply-lib.sh
procevent/         registered process-to-event sources, one private record per canonical source id; written only by bin/fm-procevent.sh; presence alone keeps supervision required (§13)
procevent-inbox/   private captured results + durable handled-ack markers; source output lives here, never in event line
decision-bindings/ private records marking captured-answer source feeding keyed-answer intake, legacy origin on pre-collapse records; written only by bin/fm-captain-hold.sh bind; dropped by unbind + source retirement (§13; docs/captain-hold-lifecycle.md)
when/              private condition->action watch specs, trust bindings, single-fire markers; written only by bin/fm-procevent-when.sh (§13 trigger)
inbox/             captain notes captured out of band by bin/fm-inbox.sh, incl voice handover's queued requests; each note appends one `check` wake, stays pending until acked with `bin/fm-inbox.sh drain --ack <id>` (moves to inbox/handled/); docs/voice-relay.md
x-inbox/           generated Relay pending mention payloads; fmx-respond drains (§14)
x-context/         generated Relay durable per-request reply context + one-wake offer markers, keyed by request_id; survives inbox cleanup, expires <=7 days (§14; bin/fm-x-lib.sh)
x-outbox/          generated Relay dry-run reply+dismiss previews; inspect when FMX_DRY_RUN set
public-followup/   generated private transport for promised public replies: retained open-loop registrations, typed terminal-result inbox, results staged for an owning home on another machine, accepted/rejected ledgers, retirement receipts (§14; bin/fm-public-followup.sh)
x-poll.error x-poll.claim-error  generated Relay + offer-claim diagnostic dedupe markers
.startup-network.*  status/report/per-step timings/inline-print claim/lock for the deferred startup stage running network checks + the inactive-outcome scan off the digest's blocking path; bin/fm-startup-network.sh
.wake-queue        durable queued wakes until post-handling ack: epoch<TAB>seq<TAB>kind<TAB>key<TAB>payload
.watcher-down      private generation-bound recovery state; couples watcher downtime + durable wake presentation + post-handling ack; never touch
.<id>.open-decisions-cursor  per-task byte cursor + folded open-decision set bounding OPEN DECISIONS scan cost to new status appends; written only by fm-classify-lib.sh status_open_decisions_incremental; removed by teardown; safe to delete (forces one full re-fold)
.status-presentation-cursor .status-presentation-lock  fleet-wide per-task status identity + independent annotation and outcome-backstop byte offsets, with serialization lock preventing already-presented lines replaying while preserving delayed signal annotations; owned by fm-classify-lib.sh; row retired by teardown
.afk               durable away-mode flag; present = sub-supervisor may inject escalations (set by /afk, cleared on user return)
.watch.lock .wake-queue.lock  watcher singleton + queue serialization locks
.claude-autoarm.lock .claude-autoarm-epoch .claude-autoarm-failure-notified .claude-autoarm-failure-alarmed .turnend-claude-blocks .turnend-claude-blocks.lock   Claude Stop auto-arm single-flight/epoch/failure-episode/attended-alarm/guard-budget/budget-lock; never touch
.cursor-park-owner .cursor-park-owner.lock .turnend-cursor-blocks   Cursor stop-hook owner + publication/commit lock + bounded repair-nag budget; never touch
.hash-* .count-* .stale-* .stale-since-* .churn-since-* .paused-* .wedge-escalations-* .writing-* .seen-* .hb-surfaced-* .last-* .heartbeat-streak   watcher internals; never touch
.watch-triage.log  watcher absorbed-wake debug log (size-capped); never relied on, safe to delete
.last-watcher-beat  watcher liveness beacon, touched every poll (incl absorbing benign wakes); guard scripts read
.subsuper-* .supervise-daemon.*   sub-supervisor internals; never touch
```
