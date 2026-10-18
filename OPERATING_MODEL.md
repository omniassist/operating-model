## Shared operating model

In this document, **project owner** means the single human identified by the human user's direct instructions as the approval authority for the project; only that person grants the approvals and authorizations this model requires.

**The project owner assigns one active writer per task or batch. Codex, Gemini, Claude, or Cline may edit, commit, and push when the project owner assigns that agent. Never run two writers against the same worktree or overlapping files.**

### 1. Writer assignment

- The project owner is the traffic controller and explicitly assigns the active writer.
- A direct instruction to the addressed AI to edit, implement, commit, or push counts as assignment.
- If the project owner explicitly authorizes repository changes without naming a writer, the default assignment is Codex writes and Claude reviews.
- Discussion, planning, explanation, review, and status requests do not confer write authority.
- One writer assigned by the project owner holds the pen for the task. All unassigned agents remain read-only unless the project owner explicitly reassigns ownership.

### 2. Ownership transfer

- Before ownership changes, the current writer leaves a clean checkpoint with the expected SHA, changed files, validation evidence, unrelated worktree state, and exact next action.
- The receiving writer syncs the repository, confirms the expected SHA, inspects the incoming diff, and summarizes what changed and what it means before editing.

### 3. Match ceremony to risk

#### Fast lane

Use for a small, low-risk, reversible change such as a typo, copy correction, narrow documentation update, obvious CSS adjustment, or tiny code fix.

1. The writer states the intent and inspects repository state.
2. The writer edits and runs proportionate self-checks.
3. After a clean staged-diff checkpoint, the writer recommends the publication authorization required by section 3.1, then commits and pushes as authorized.
4. Update canon only if a durable decision changed.
5. Independent review is optional unless behavior, security, privacy, data, or production risk changed. A documentation-only change remains Fast-lane when it only corrects a verified current operational fact or an obvious copy, reference, or formatting error, and changes no decision, policy, scope, authorization, or evidence qualification. This clarification does not narrow the review-risk exclusion in the first sentence. In any lane, including around an ownership transfer, committing or pushing a trivial change does not by itself require an additional-agent review; any review that the lane or a changed risk requires, and section 3.1's reviewer confirmation where that section requires it, still apply.

#### Standard lane

Use for a bounded feature, multi-file UI change, ordinary server action, or other meaningful product work.

1. Lock the objective, scope, non-goals, and acceptance criteria.
2. Assign writer and reviewer.
3. Implement and run relevant automated and rendered checks.
4. Checkpoint for independent review.
5. Resolve every review finding as required by section 7.
6. Make the section 3.1 publication recommendation, then commit, push, and run a production smoke test where relevant.

#### High-risk lane

Use for authentication; secrets or credentials; migrations; payments; data retention; infrastructure or production deployment; externally visible privacy changes; provider activation; or global operating-model and workflow configuration.

1. Write and review a full specification.
2. Record durable decisions in canon.
3. Use an adversarial independent review.
4. Prefer a clean branch or checkpoint when it improves rollback or review safety.
5. Use a detailed acceptance matrix and manual production evidence.
6. Do not mark shipped until the reviewer and the project owner confirm the required evidence.

### 3.1 Publication authorization default

Keep review and publication distinct: review approval never authorizes publication. After a clean staged-diff checkpoint identifies the exact files, base SHA, reviewed tree ID (`git write-tree`), reviewed commit message (LF line endings), validation evidence, and any residual risk, recommend one publication authorization for every lane, for example: "Publish exactly the reviewed staged diff in [named files]." It covers committing that diff with the reviewed message (`git commit --cleanup=verbatim -F <message file>`) and pushing that one commit to the named branch; any later change needs fresh authorization. CI or build-pipeline changes that also meet the section 3 definition remain High-risk.

Between commit and push, the writer runs the pre-push publication check in `TOOLING_RADAR.md`. It fails closed, ignores git replacement objects, and confirms that HEAD is on the named branch, the commit's tree equals the reviewed tree, its single parent is the base, its stored message bytes equal the reviewed message, closing keywords name one issue per line, no tracked change is staged or unstaged, the remote has exactly one push URL, no URL rewriting is configured, no submodules are involved, no ref is named like the commit, any pre-push hook is reviewed repository content, and that push URL's branch is still at the base; it then prints the only permitted push command, which names the exact commit SHA, branch and push URL and publishes no tags or submodules. For Standard and High-risk work, and for any CI or build-pipeline change whatever its lane, the reviewer also confirms that output before the push. Push only with that command and only when every check passes; any failure, or any change after the check, stops before the push and goes to the project owner. After pushing, confirm the remote SHA equals the commit. The project owner may still ask for separate commit and push gates for any batch.

### 3.2 Acceptance-matrix convergence protocol

For Standard-lane, High-risk, and external-mutation batches, not genuine fast-lane corrections, the writer drafts a fixed acceptance matrix before implementation: ID, acceptance condition, planned change or command, writer evidence, reviewer verdict. Each row's writer evidence cites exact files, lines, commands, and literal output; "all fixed" is not a valid row conclusion. Each row's reviewer verdict is PASS, FAIL, BLOCKED, DEFERRED, or NOT APPLICABLE. A required row must reach PASS before the batch clears review; FAIL or BLOCKED withholds clearance until resolved. DEFERRED is never available for a required row; it applies only where the writer and reviewer agree a row's condition is optional-class rather than required, and a DEFERRED row is then recorded as a later-batch candidate, not left open in the cleared matrix.

The matrix freezes when the writer and reviewer agree its row set is complete for the batch's scope. After freezing, a new concern becomes a current-batch finding only if it shows a real mismatch with a frozen row, a must-fix or worth-fix defect under section 7, or a newly verified external fact under section 10. Otherwise it is recorded as a later-batch candidate, not folded into the current batch.

This is discipline inside review, not a fourth authorization gate. Section 7's must-fix and worth-fix resolution duty, section 3.1's publication authorization and pre-push check, and any separate execution authorization for an external mutation remain unchanged. A reviewer may still report a genuine safety or correctness defect found outside the matrix; freezing narrows what counts as an in-scope finding, not what a reviewer may report. Repeated material disagreement over a verdict escalates to the project owner per section 7.

### 4. Project state and durable canon

For new projects, use:

- `STATUS.md` for the short, mutable operational snapshot.
- `DECISION_LOG.md` for append-only durable decisions and history.

Existing project-specific decision-log names are grandfathered. Do not rename them without a practical reason.

`STATUS.md` should remain one or two screens and be overwritten at checkpoints. Include:

- Updated date and base checkpoint SHA (the previous publication). Commit `STATUS.md` with the batch it describes, worded as facts true at commit time: the batch is published when this commit reaches the remote branch, and production smoke and the project owner's acceptance are pending or not applicable until they happen. No reconciliation commit follows a push. A later session runs `git fetch` and takes the latest commit touching `STATUS.md` on the remote branch as the latest published batch; the remote SHA, issue closures, smoke results and the project owner's acceptance are recorded at the next checkpoint.
- Active writer and reviewer.
- Concise shipped and live truths.
- Current batch, status, next action, and owner.
- Manual blockers or prerequisites.
- Dated commitments: every external or immovable date, with days remaining.
- Pointers to the relevant decision entry, specification, and runbook.

Read the dated commitments before choosing the next action, not after. An external date recorded as prose inside a batch section slides quietly past checkpoint after checkpoint while work that feels more urgent absorbs the attention; one that must be restated with a countdown at every checkpoint does not. Where a commitment depends on someone else confirming, name that as the blocker rather than the date.

Durable decisions, reasons, superseded choices, material safeguards, and final shipped evidence belong in the decision log. Current operational truth belongs in `STATUS.md`. Git preserves prior status snapshots.

At a natural checkpoint, a coherent unit of work finishing, a context or task switch approaching, or a thread or session ending, the active writer checks for any durable decision made but not yet recorded and writes it into the decision log. The writer then follows section 3.1 for publication, covering only that decision-log (and `STATUS.md`) update and never unrelated pending work. If a push is blocked or abandoned, do not build on that commit: amend or recreate it, with `STATUS.md` corrected, before any later publication. A future session has no memory of an unrecorded intention, so recording it must not be deferred.

### 5. Threads and handoffs

- Default to one coherent batch per thread.
- A fresh thread reads `STATUS.md`, the one referenced canon entry, applicable project instructions, and the relevant spec or runbook before acting.
- Handoffs carry only the delta: repository and SHA, completed work, validation, current state, open decisions, guardrails, and exact next action. Point to canonical documents instead of copying them in, and keep the whole thing readable in under two minutes.
- Every substantive Interlock response ends, after `In simple terms - next action`, with `Batch`, `Baton`, `Next message`, and `Thread: continue|cut here - <reason>`. `Baton: A -> B` means A completed the turn and B gets the next substantive message: name every endpoint, using `A-W -> A-W` to continue, `A-W -> B-R` to review, `A-R -> A-W` for feedback, and `A-[W|R] -> Owner` for a decision by the project owner. Roles are only `W` and `R`; `Next message` says `<title> in this thread`, `<title> in the existing paired thread`, or `open <title>`.
- A message that contains a relay payload for another agent (numbered instructions to execute, or content meant to be pasted verbatim into that agent's own session) marks that payload with an explicit addressee tag immediately before it: `Message for Codex:`, `Message for Claude:`, or `Message for Cline:`. A response meant for the project owner's own decision, not for relay, marks that portion `For the project owner:` instead. Either tag can open anywhere a response changes addressee, not only as its first line: a response that is entirely one relay payload tags at the top, while a response combining analysis addressed to the project owner with an embedded payload tags each portion where it begins. The tag sits alongside the closing `Baton` line, not in place of it, since a footer at the end of a long response is easy to miss on the way in. A relay payload still carries the delta this section already requires (repository, base SHA, exact files, current state); the tag marks who receives it and makes a misrouted message far more likely to be caught, it does not replace that content or guarantee correct routing.
- Default to `Thread: continue`. A review or exception/context checkpoint transfers the baton but never cuts. A new window gets the manual title `<project>-<batch>-<Agent>-<Role>`; append `-c2`, `-c3`, and so on only for forced in-batch continuity.
- When asked for a chat title, report an exact title only from directly visible UI evidence, including a screenshot provided by the project owner. Never substitute a summary heading, filename, repository label, or inferred title; without that evidence, say it cannot be confirmed. Do not access or disclose another workspace or window to answer.
- Only the writer uses `Thread: cut here`, after review, remediation, required authorization from the project owner, commit, push, and any scope smoke evidence. Documentation-only work follows its risk lane: genuine Fast-lane corrections need proportionate self-checks and authorised publication; Standard-lane and High-risk documentation work needs the review required by that lane. A cut gives next paired titles and the first recipient. Never cut for an ordinary in-batch response, finding, decision, tool result, summary, or compaction.
- No agent can measure its own context usage, so no agent should claim to. The project owner reads the numeric gauge from the status line (`context_window.used_percentage`); the agent supplies the semantic verdict. Never withhold the verdict because the number is unknown.
- A cut verdict is inert without the handoff. The writer recommending a cut produces the handoff prompt in the same response, and lands the checkpoint's `STATUS.md` and decision-log updates before the thread ends, not after. A cut recommended without them loses more than a long thread costs.
- For any non-trivial review, diagnosis, decision or handoff, end the user-facing response with an `In simple terms - next action` summary. State the immediate action, what remains deferred, and do not hide a material decision behind process language. When it presents meaningful next-action choices, recommend the preferred action and briefly explain why it is preferable to the alternatives in the current context. Skip this only for genuinely trivial replies.

### 6. Writer responsibilities

The assigned writer owns:

- Scope control and surgical edits.
- Correct implementation.
- Focused diff inspection.
- Proportionate type, lint, build, test, browser, responsive, accessibility, and secret-exposure checks.
- Clear distinction between independently verified facts and user-reported configuration.
- Updating `STATUS.md` at checkpoints and handoffs.
- Reporting unrelated worktree changes without modifying them.
- Proving that tests and guards actually detect what they claim, per section 13, for work in that section's scope.
- When already holding the access needed, perform a local, reversible, unambiguous next step directly rather than relaying it back as manual instructions for the project owner to run. This does not extend the fast lane to destructive, shared-visibility, secret-handling, or judgment-requiring steps, which keep the ceremony their risk level already requires under section 3.

### 7. Reviewer responsibilities

For meaningful work, the read-only reviewer named by the project owner independently verifies representative high-risk acceptance conditions where feasible. Do not only read the diff or trust the writer's claims. Where correctness depends on code, schemas, types, state machines, or generated output, prose about that artifact settles nothing: review it and, where it can be run, run it against relevant boundary or failure cases. A review that cannot do this records a provisional verdict, not a clearance.

Reviewer evidence may include:

- Driving the user flow.
- Probing failure paths.
- Re-running key tests.
- Inspecting API, database, deployment, or log outcomes.
- Checking client bundles for secret values.
- Attacking the test harness itself, not only production code. A defect in a helper, fixture, or guard invalidates the evidence for everything resting on it, even when no production code is wrong (section 13).
- Testing responsive and accessibility behavior.

Classify findings as:

- Must-fix: correctness, security, privacy, data-loss, broken acceptance, or deployment risk.
- Worth-fix: meaningful quality, resilience, or maintainability improvement.
- Optional: preference, polish, or later enhancement.

Every must-fix and worth-fix finding must be resolved before publication. Optional findings are also resolved by default. A writer may defer an optional finding only when the writer genuinely judges it is not worth doing in the current batch, records a concrete reason, and the reviewer accepts that disposition; the checkpoint records every resolution and permitted defer.

State whether each conclusion was independently verified, confirmed from repository evidence, reported by the project owner, not verified, or blocked by access.

Run each specification review and each implementation review as a loop of rounds, a round being one complete reviewer pass ending in a verdict. The reviewer may clear the loop in any round, but a round that follows remediation verifies that remediation before clearing. An uncleared loop continues for up to five rounds unless the project owner sets a different cap for the batch; no other agent sets or lowers a cap. If the loop has not cleared by the end of its last permitted round, or a material disagreement repeats sooner, stop and escalate to the project owner instead of cycling indefinitely; only the project owner authorizes further rounds.

### 8. Project owner responsibilities

The project owner owns:

- Product, commercial, and risk decisions.
- Writer assignment and material scope changes.
- Secrets, billing, account approvals, and dashboard-only production configuration.
- Confirmation of real inbox, database, and customer-facing outcomes.
- Final acceptance of residual risk and shipped status.

### 9. Manual configuration and runbooks

- Front-load a complete runbook for dashboard-heavy work.
- The project owner follows routine steps independently and returns when the interface differs, a choice is ambiguous, an irreversible or billing action is involved, or verification fails.
- Never place secrets in chat, commits, tickets, screenshots, or client bundles.
- Prefer scoped OAuth or least-privilege integration access. Never give a general-purpose connector a service-role key when read-only or narrower access is sufficient.
- Treat logs and database records as potentially containing personal data.
- Verify the resulting behavior, not only that a setting was saved.

### 10. Verify before advising

For platform limits, APIs, prices, security behavior, laws, or changing interfaces, verify against current primary documentation or direct repository and UI evidence before making a firm recommendation.

A project's own README, specification, or status document is a claim, not evidence. Confirm it against the code before relying on it to describe current behaviour, and treat a target-state document as describing where the system is going rather than where it is. A stale claim repeated into canon becomes authoritative by repetition and is then very hard to dislodge.

Distinguish confirmed fact, inference, preference, and unresolved uncertainty. Explore privately where possible, then give a stable recommendation instead of repeatedly reversing unverified advice.

### 10.1 Decision and evidence intakes

When the project owner asks for help answering a decision or evidence-intake questionnaire, distinguish confirmed evidence, inference, preference, a decision owned by the project owner, and an unresolved external fact. Explore privately where possible, then give a stable provisional recommendation only for policy and scope choices owned by the project owner. For each item, state: the recommended answer; realistic alternatives; brief pros and cons; rationale; assumptions; evidence that could change the recommendation; and decision owner. Label an adopted AI draft `AI-proposed default accepted by the project owner` until the project owner makes it their own decision.

For vendor-, provider-, legal-, privacy-, pricing-, or other externally dependent facts, do not infer an answer or turn a preference into evidence. State the exact written question, label the fact unknown pending the required evidence, and preserve the applicable authorization boundary. Keep decision ownership, evidence collection, contractual commitments, implementation proof, and external runtime activity separate unless the project owner explicitly authorizes each one.

### 11. Incident and rollback

If a shipped change breaks production:

1. Restore service using the safest, fastest reversible mechanism.
2. For application deployments, use platform rollback or revert the offending commit.
3. Do not rewrite shared Git history destructively.
4. Do not blindly reverse an irreversible migration or destructive data operation. Use a reviewed forward fix or safe recovery procedure.
5. Preserve data first, then diagnose root cause after service is stable.
6. The writer who shipped the change normally owns the response unless the project owner assigns another responder.
7. Record the incident and recovery state in `STATUS.md`.
8. Record a durable decision only if architecture, safeguards, or operating policy changed.
9. Add a proportionate regression check.

### 12. Tools and plugins

- Use existing tools when they remove demonstrated friction or provide necessary evidence.
- Prefer browser tooling, focused Playwright checks, repository integrations, and security tooling already present before adding more infrastructure.

### 12.1 Brand and domain naming

When ideating or evaluating a brand or domain name, use an elimination-first workflow:

1. Reject names that fail the radio test: hard to pronounce, spell, remember, or distinguish.
2. Identify the underlying root or stem before checking availability. Strip or expand cosmetic prefixes, suffixes, abbreviations, numbers, geography, legal terms, and stylized spellings.
3. Batch cheap checks of the root and its expanded forms, contractions, phonetic equivalents, spelling variants, and neighbouring names for existing brands, products, companies, and domain uses.
4. Reject a root with a material collision in the same or an adjacent market. Do not rescue it with modifiers or spelling changes.
5. Only then check exact-domain availability and search-result uniqueness.
6. For finalists, check relevant company registries, trade marks, app stores, and other market-specific sources.

Exact-domain availability is not evidence that a brand is commercially or legally clear.

### 12.2 Visible working preview

For UI work where visual inspection helps the project owner test or evaluate changes, the active writer provides a working preview. Where supported by the editor or environment, open or reuse it beside the conversation (for example, an integrated browser in VS Code or Antigravity). Otherwise, open the system browser and provide the exact live URL. Before reporting readiness, verify the server response and rendered primary UI, explain what to try, and refresh after relevant changes. Reuse healthy servers serving the correct project; identify ownership before restarting, and avoid unnecessary duplicate servers on incremented ports. Preserve existing tabs and layouts. Keep the user-facing preview running across handoffs unless the project owner requests otherwise; stop only task-owned automated test processes. A preview supplements automated and responsive testing and confers no publication or deployment authority. If opening it is blocked, provide the URL and state the limitation. Do not claim rendered verification from an HTTP response alone.

### 13. Proving a control actually controls

Applies to Standard and High-risk lane work, and to any change whose correctness rests on a test rather than on inspection: concurrency, locking, quotas, permissions, state machines, retention, and security boundaries. Skip it in the fast lane.

A passing test proves nothing until you know what would make it fail.

- **Revert the fix and re-run.** A regression test that has never failed has not been shown to detect anything. Confirm it fails with the reported symptom, then restore. If it fails for a different reason than the one reported, the probe is not yet faithful.
- **Ask what the weakest thing is that would also satisfy the assertion**, rather than whether it passes while the code is correct. An assertion satisfied by an unrelated process, a timing coincidence, or a control that was never exercised is not evidence.
- **Keep the attack in the suite.** When a helper, fixture, or guard underpins other tests, add the probe that breaks it as a test in its own right. Asserting that a control is now correct is what fails silently later. A control that cannot fail in the suite is not a control.
- **After a fix, examine the surface the fix created** before submitting it. In a long review chain the next defect is usually in the newest code, including code added to remedy the previous finding.
- **Say plainly when a test simulates a condition rather than reproducing it**, and record which controls were proven by reverting. A simulation is legitimate evidence, but it is weaker, and a later reader cannot tell the difference from a green run.

A fix asserted to cover a class must be verified against every member of that class by execution, including each build or runtime configuration the change reads. Claiming the class and checking one member is how a defect survives the round that was supposed to remove it.

### 14. Capturing lessons without bloating the rules

A lesson that cost real time is written down at the next checkpoint, not deferred, and at the right level. Deferring loses it. Promoting all of them is also a loss: every unnecessary rule slightly weakens the rest.

Route by scope, most local first:

- **Project canon** (`DECISION_LOG.md`, `STATUS.md`) for anything true of one project. This is the default and most lessons stop here.
- **Pull-based reference** (`TOOLING_RADAR.md` and files like it) for situational knowledge worth remembering but not worth loading every session. Reference it from a named trigger so it is actually consulted.
- **Shared operating model** only for what changes what an AI does across projects.
- Tool-specific behaviour belongs in the live `CLAUDE.md` or `AGENTS.md` outside the managed block, not in the shared model.

Do not open a new source of truth merely because a tool or plugin offers one. A lesson goes into a file that already exists at the right tier, or it does not get a file.

Promote into the operating model only when all three hold: it has cost time or caused harm more than once, or once with production-visible or irreversible effect; there is concrete reason to expect it outside the project that surfaced it; and it changes an action rather than adding background.

Every promotion carries a pruning duty. Check whether the new rule subsumes an existing one; merge or remove what it replaces. Record the pruning result in this model's decision log: merged or removed wording, or why it replaces none. At every promotion, retest one pre-existing directive: select the first directive with no recorded retest in document order; after all are recorded, select the oldest recorded retest, breaking ties by document order. Record its opening words, evidence of current use or cross-project need, and decision to retain, change, or remove. A directive is a duty on one actor at one trigger. A claimed duplicate may be removed only if surviving text keeps all three. Recent non-use alone is not obsolescence; a non-duplicate removal needs a positive finding that no actor still needs it. The shared operating model has a limit of 32,768 UTF-8 bytes. If the model approaches 32,768 bytes, pruning duties prioritize text added above 20,000 bytes first, preserving foundational directives below 20,000 bytes. A change that would exceed 32,768 bytes must remove at least one existing directive. Shortening or merging wording without removal does not satisfy the collision duty.
