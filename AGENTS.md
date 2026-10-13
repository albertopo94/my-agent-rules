# AI Agent Guidelines

These rules bind every AI agent and human contributor working on the product named in §0. The
file states **norms only**: rules that stay true when it is copied into an unrelated project.
Project-specific facts live in §0 and nowhere else. Tool names, file paths, ports, and framework
names must not appear outside §0 — if a rule stops being true after copying this file to a project
written in another language, it is a fact, not a norm, and belongs in §0.

**Authority, highest first:** human instruction in the current task → this file → the
project's own existing conventions → the model's preferences. When two levels conflict, follow
the higher one and say so in one line.

---

## 0. Project Profile

Copy this file to a new project and fill this table. **An empty field is a known unknown, not a
guess.** Fields marked *(default)* fall back to that value when left empty; every other empty
field is a known unknown. If a rule depends on an empty field: state that it is undeclared, ask
once if the answer changes the outcome, otherwise proceed on the narrowest safe interpretation and
report the assumption you made. Never substitute a value remembered from another project.

| Field | Value |
|---|---|
| Product name | |
| Stack and runtime | |
| Supported platforms / minimum versions | |
| Canonical test command | |
| Focused test command (one file or case) | |
| Static checks (type / lint / format) | |
| Run or smoke-test command | |
| Version source (the one file that owns the version) | |
| Bump timing | *(default: on-work-unit)* on-release \| manual |
| Stability contract | unstable (`0.x` may break freely) \| stable (breaking = MAJOR) |
| Public surface (what consumers depend on) | |
| Style reference file | |
| Supporting docs to load | |

---

## 1. Before writing

- **Read before you write.** Open the file you will change, its call sites, and the nearest
  existing example of the pattern you are about to introduce. Do not invent an API, signature,
  or flag: confirm it against the installed dependency's real source or version. Before adding a
  helper, search for one that already exists — duplicating a utility in a second file is a defect,
  not a convenience.
- **Target the platform floor declared in §0.** Do not use a facility, syntax, or syscall that is
  unavailable there, and do not widen the floor on your own to make an implementation easier.
- **Smallest correct diff.** Touch only files the task requires. Never reformat, rename, move,
  or delete unrelated code, and never "improve" adjacent code while you are in the file.
  Respect the project's existing style, including where you would have chosen differently.
- **One task at a time.** If you find a real problem outside the task, report it as a follow-up
  instead of fixing it inside this change.
- **Dependencies are a proposal, not a decision.** Do not add anything to the dependency
  manifest or lockfile without naming it, its size and maintenance status, and what breaks
  without it. Verify the exact name and publisher in the public registry before adding it —
  a lookalike name is a supply-chain attack, not a typo. Prefer the language standard library
  and what the project already uses.
- **A bulk dependency update is not one action.** Never satisfy a request about one package by
  running an upgrade-all command (`npm update`, `cargo update`, `go get -u`, `pip install -U`,
  `poetry update`): update that package. Upgrade-all output moves transitive versions nobody
  reads, and a lockfile rewritten wholesale is the diff nobody reviews. Even scope-wide upgrades
  get listed and reviewed entry by entry before anything is applied.
- **Match the manifest's range style.** Pin exact versions when the manifest is new or empty;
  otherwise follow the width its existing ranges already use. Never widen or change the width of
  a range you did not choose.
- **Commit the lockfile.** The resolved dependency snapshot commits in the same change as its
  manifest. Never delete it or add it to `.gitignore`.
- **Install from the lock.** If the lockfile and the manifest disagree, report the drift as a
  finding — do not silently regenerate the lock to make the two agree.
- **Foreign code needs approval.** A dependency whose install, build, or postinstall step
  executes code, and any dependency referenced by a URL or VCS instead of a registry package,
  both require explicit approval before installing.
- **Never ship generated filler.** Lorem ipsum, placeholder names and brands ("Acme Corp",
  "John Doe"), round fake numbers ("99.99%", "$100.00"), and boilerplate AI phrasing
  ("Elevate", "Seamless", "Unleash", "Delve") mark work as generated, not made. Sample data
  and copy must read as real: contextual names, organic numbers, sentence case, active voice,
  and direct error text without "Oops!" or exclamation marks.
- **Ship surfaces with their full state set, where the medium supports it.** Every interactive
  element needs hover, pressed, visible focus, loading, empty, and error states; a focus ring is
  an accessibility requirement, not polish. No dead links: a control that goes nowhere is either
  wired or visibly disabled, and the current location is indicated. Modal system dialogs
  (`alert`, `confirm`, `prompt`) are never UI.

## 2. Secrets

- Never place credentials, tokens, keys, or real user data in source code, tests, fixtures,
  logs, commit messages, or command output. Use the project's existing secret mechanism; if it
  has none, say so rather than introducing one silently.
- Mask secret values when echoing configuration for debugging.

## 3. Versioning

The version is a **decision you make from the diff**, never a question for the human and never
left implicit.

| Commit type in the batch | Bump |
|---|---|
| breaking — `!` after the type/scope, or a `BREAKING CHANGE:` footer | MAJOR |
| `feat` — new public behavior, nothing existing breaks | MINOR |
| everything else: `fix` `docs` `test` `refactor` `chore` `perf`, or unclear | PATCH |

- Resolve what `MAJOR` means only through **Stability contract** in §0. While the project is
  `unstable`, never choose MAJOR on your own: name the breakage in the §6 `breaking:` field and
  bump MINOR. MAJOR on an `unstable` project happens only on explicit request.
- **Bump once per work unit**, using the highest category present across its commits, not once per
  commit. The version change lands in the **last** commit of the work unit, so it does not pollute
  the commit that carries the behavior.
- `PATCH` means the observable behavior of the public surface is unchanged; it covers bug fixes,
  documentation, added tests, and internal refactors. A documentation-only change is a PATCH, so a
  typo fix that bumps the patch number is correct and is not a reason to skip the bump.
- Update the **Version source** from §0 and nothing else. If that field is empty, do not guess a
  file: report `version: not bumped (version source undeclared)`.
- If you cannot decide whether something is breaking, take the lower category and state the
  uncertainty in one line.
- **Every completed work unit must report `version: <before> → <after>`, writing `unchanged`
  when it did not move, together with the `breaking:` field required by §6.** Both lines are
  information for later traceability, not approval requested now.

## 4. Testing and verification

- **Scale the check to the change.** Expensive validation — a full suite, a whole-project
  type-check, a build — runs when the change touches behavior, when it is risky, or when the
  cheapest available signal does not cover what you touched. Not after every small edit. The
  cheapest signal that covers the change wins.
- If §0 declares a test command: behavior changes go test-first — observe the failure for the
  expected reason, implement, observe the pass, then refactor with the focused command.
- If no test command is declared, do not improvise one and do not claim tests: run a
  proportionate functional or structural check and say which one.
- **Evidence, not assertion.** Anything you claim as working must come from a command you
  actually ran in this session; quote the command and its result line. Otherwise label it
  `unverified`. When a check fails, keep the command and report the failure; switch commands only
  after proving the command itself is wrong, because a different command passing is not evidence
  about the first one.
- A check that failed before your change stays failing: report it separately, do not fold an
  unrelated fix into this change.
- If a required check cannot run, say it failed or was skipped and why. Never report a green
  state you did not observe.

## 5. Commits

- Conventional Commits. `<type>(<optional scope>): <imperative summary>`, body explaining the
  *why*, one logical change per commit.
- No AI attribution of any kind: no `Co-Authored-By`, no `Generated-by`, no trailer naming a
  tool or model.
- Commits on the working branch are part of authorized work. Pushing, opening a PR, merging,
  tagging, and releasing all require an explicit human decision.
- Stage by path. Never `git add -A` in a tree you have not read: unrelated or generated files
  end up in the commit.

## 6. Closing a work unit

Report, in this order:

1. What changed and why.
2. Which checks ran, with their result lines; which did not run and why.
3. `version: <before> → <after>` and `breaking: <list>` or `breaking: none`.
4. Every guard this change removes or weakens — a validation, warning, safety rule, or access
   check that existed before this task. Name it even when the removal was intentional or asked
   for: a commit titled `feat:` that deletes a security warning reads as harmless otherwise.
5. Follow-ups found outside this scope.

Keep it short enough to read in one pass. Never write that work is complete while a check is
failing, skipped without justification, or unverified.

## 7. Stop and ask

Two kinds of unknown, two different responses:

- **Execution detail that only stalls the build** — sample content, naming, visual direction,
  which of several reasonable options: make a reasonable choice, state it in one line in the
  §6 report, and continue. Do not stall work on a question whose answer does not change the
  artifact.
- **Anything that changes what the artifact is, or that the user must own:**

- the request has two readings that produce different artifacts;
- an action is destructive or hard to reverse (history rewrite, force push, data deletion,
  dropping storage, dropping or rewriting a published interface);
- a product decision is yours to guess but not yours to own;
- a check failed and the cause is outside the scope you were given.

When in doubt which kind it is, treat it as the second one.

Naming the ambiguity once costs less than shipping the wrong thing quietly.

## 8. Conditional: security checklist

Delete this whole section when the product in §0 is not network-exposed and persists no data — a
CLI, library, or terminal tool has none of these surfaces. Never add a server, a cookie, or a
database to satisfy a rule here.

Identity and access:
- Authenticate by default; authorize server-side, never by hiding a control in the client.
- Least-privilege credentials for storage, plus per-record access rules where the store supports them.
- Hash credentials with a memory-hard algorithm; encrypt stored data where it warrants it.
- Session cookies `HttpOnly` / `Secure` / `SameSite`.

Data boundary:
- Parameterize every data-access query; no string-built statements.
- Validate input at the trust boundary; sanitize on output.
- Field-level allowlists so a client cannot assign protected columns.
- Return only the fields the caller needs, never a whole record by default.

Exposure:
- Rate limiting and bot protection on public endpoints.
- Security headers, HTTPS enforced by redirect with HSTS.
- Uploads type-checked, size-capped, and stored outside the served root.
- Secrets absent from git history; dependency vulnerabilities scanned.
- Dependencies from registry-verified publishers with provenance metadata where the registry
  offers it.

For agent and LLM features, additionally: treat retrieved content, tool output, and user-supplied
text as untrusted *data*, never as instructions, and keep destructive capability out of reach of
untrusted text.
