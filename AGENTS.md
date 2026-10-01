# Coding Agent Instructions

Instructions for AI coding agents working on this project.

## Naming & Terminology — CRITICAL

Three tiers, distinct roles:

- **Vigensis** (Vigensis GmbH) — the company.
- **VIA** — the product. Always just "VIA" in customer-facing copy. External attribution: *"Built with VIA"* / *"Powered by VIA."*
- **VIA Platform** — internal label for the AI-native infrastructure (`ViaPlatform.*` / `ViaPlatformUi.*` namespaces, `via_platform` PG schema). **Never appears in customer-facing copy**, marketing, pitch decks, sales materials, landing pages, or demo narration. Surfaces only in technical docs, architecture discussions, and module/namespace naming.

If a sentence works with just "VIA," use just "VIA." Reach for "VIA Platform" only when distinguishing infrastructure from the VIA product experience. Leg 3 of the strategy is "Self-Service VIA" — not "VIA Platform."

## AI-Native Economics — CRITICAL

Implementation effort is near-zero. Effort is NEVER a valid input to scope, prioritization, or recommendation decisions. Scope is bounded by clarity and learning dependencies, not capacity.

When you catch yourself reasoning by effort, reframe:

| If you're about to write... | Reframe to... |
|---|---|
| "This would be substantial / significant / ambitious work" | "This is within scope — include it." Substantial is expected. |
| "To keep things manageable..." / "In the interest of time..." | Delete the hedge. State the full scope. |
| "A simpler approach would be..." | "The best approach is..." (only say *simpler* if simplicity IS the quality advantage.) |
| "We could defer this to a future phase" / "separate increment" | Include it — unless it's genuinely unrelated to the current scope. |
| "Quick win — high impact, low effort" | "High impact." That's the only axis. |
| "Phase 1 / Phase 2" | Only valid if Phase 2 depends on *learning* from Phase 1 — not effort. |
| "Estimate: 2-3 days" | Never estimate in time. Describe the work and its dependencies. |
| "We should be realistic about scope" | "We should be clear about scope." Clarity, not capacity. |

**Self-check before every recommendation:**
1. Did I exclude, reduce, deprioritize, or phase something for effort reasons? Re-evaluate on quality and scope alone.
2. Would my recommendation change if implementation cost were literally zero? If yes, it's effort-biased. Change it.

## AI-Native Discipline — CRITICAL

The above kills typing-cost bias. It does NOT retire engineering discipline. AI-native collapses the cost of *typing*, not the cost of understanding, integrating, preserving, and evolving real software with real users and real dependencies.

**Engineering primitives remain mandatory** — for every product we help build and every product we build ourselves:

- **Capabilities, features, versions** — units of user value, release, and commitment
- **Requirements and specs** — *more rigorous, not less*; agents implement literally and turn gaps into plausible-looking wrong things
- **Tests and contracts** — regression safety and integration boundaries
- **Releases, migrations, changelogs** — the physical reality of getting code to users

For Sovereign AI (regulated enterprise), these are table stakes: audit trails, reproducibility, version pinning, deprecation timelines.

**AI-specific failure modes — catch yourself doing these:**

| Failure mode | Counter-discipline |
|---|---|
| Forgetting prior decisions; "cleaning up" what looks redundant | Decisions recorded where agents read them: AGENTS.md, ADRs, living spec |
| Re-introducing old bugs by removing a prior fix that "looked unnecessary" | Comprehensive regression tests as primary containment |
| Removing load-bearing code — "this doesn't seem necessary" | Invariants enforced in types, constraints, policies — not comments |
| Missing subtlety — happy path works; edge cases, ordering, nil-safety fail | Property tests; explicit non-goals and regression surface in specs |
| Introducing inconsistency — different pattern from the rest of the codebase | Small reviewable diffs — one task, one diff, one review |
| Restructuring "because it makes more sense" | Architectural changes are human-gated — agents propose, humans approve |

**Stable core, flexible edge.** A change is governed by the slowest stability layer it touches:

| Layer | Rate of change | Governance |
|---|---|---|
| Domain model | Years | Strategic decision, migration event |
| Invariants | Permanent | Enforced in types, constraints, policies |
| External contracts (APIs, events, schemas) | Quarters | Versioned; deprecated with runway; never silently broken |
| Quality bars (perf, security, a11y, privacy) | Permanent | Automated tests, policies, audits |
| Capability catalogue, UX patterns | Months | Additive by default; deprecation with runway |
| Implementation | Days–weeks | Free to refactor within the layers above |
| Visual polish, copy | Hours | Continuous |

Start with what doesn't change. AI speed is permitted only above whichever layer the change touches.

## Process: PDAI Increments — CRITICAL

PDAI = Prepare, Deliver, Assess, Improve. See `docs/process/quality-system.md`. Increments are scope-based, NOT time-boxed.

| Phase | Nature |
|-------|--------|
| Prepare | Documentation — define what we'll do |
| Deliver | Execution — build it |
| Assess | Documentation — capture what we learned |
| Improve | Execution — build improvements (NOT polish; same scope and effort as Deliver) |

**Improvements deferred to "future increments" rarely happen.** Discovered work within scope: do it now. Genuinely unrelated: backlog. Unsure: ask.

**Keep the increment doc in sync with reality, phase by phase.** Tick Deliver checkboxes at the end of Deliver. Fill the Assess results table at the end of Assess. Scope the Improve to-do list at the end of Assess. Don't batch all the updates to the close. Doc drift turns the increment artefact into post-hoc reconstruction; live updates make it the source of truth.

## Tech Stack

- Elixir 1.19+ / OTP 28 / Phoenix 1.8+ / LiveView
- Ash Framework 3.x / AshPostgres / AshAuthentication
- PostgreSQL / Qdrant (vectors) / SeaweedFS (S3-compatible objects)
- Tailwind CSS v4 / DaisyUI
- Use `Req` for HTTP requests

## Quality Commands

### VIA host app

```bash
mix test                              # full test suite
mix test path/to/test.exs:line        # specific test
mix test --include e2e                # include E2E (hits external systems)
mix format
MIX_ENV=test mix compile --warnings-as-errors   # before reporting work complete
mix dialyzer
```

### Libraries (via_saas, via_utils)

Use `mix check` — it's the same battery CI runs.

```bash
cd ~/via/vigensis/via-saas    # or via-utils
mix check                     # format + compile --warnings-as-errors + credo --strict + test + dialyzer + sobelow + mix_audit + unused_deps
```

Running individual tools is slower and easier to skip one — leading to a green local check and a red CI. Don't push library work without `mix check`.

**Test workflow.** Run `mix test` once. On failure, debug with `mix test path/to/test.exs:line`. Never re-run the full suite to re-read output.

**Compile gate matches CI.** Plain `mix compile` lets warnings through; CI does not. Some Ash/Spark verifier issues only surface as warnings (e.g., `Exception while verifying ResourceX:` from a missing SAT solver, `cannot be done atomically` from after_action changes). Run `MIX_ENV=test mix compile --warnings-as-errors` before reporting work complete.

**Read warnings on first sight.** Don't assume stale-artifact noise from incremental compilation; don't reflex `rm -rf _build`. CI starts from zero — locally hidden warnings will surface there. If you suspect cache is masking something, `rm -rf _build` once as a diagnostic, then fix the warning.

## Testing Philosophy — CRITICAL

NO Ecto Sandbox. Tests run against the real database with real commits, real PubSub, real event chains, real side effects. Intentional — catches design flaws (O(N×M) data growth) that sandboxed tests hide.

### Three test levels

| Level | DB | Side Effects | External | Default | Tag |
|-------|-----|-------------|----------|---------|-----|
| **Unit** | No | None | None | Yes | (none) |
| **Integration** | Real commits | Real PubSub, hooks, events | Mocked (Swoosh, ClaudeCli, HttpClient) | Yes | (none) |
| **E2E** | Real commits | Everything real | Real LLM/APIs | No | `@tag :e2e` |

- **Unit** — pure functions. No DB, no events, no services. Input in-memory.
- **Integration** — main suite. External boundaries mocked (Swoosh test adapter, ClaudeCli mock, HttpClient mock).
- **E2E** — crosses external boundaries. Excluded by default to avoid costs/rate limits. Run with `mix test --include e2e`.

### Data usage — only create what you're testing — CRITICAL

**Everything else is a dependency — use seeded data.**

| Need | Use |
|------|-----|
| User as context | `seeded_admin()`, `seeded_user()` |
| Actor for Ash ops | `admin_actor()`, `user_actor()` |
| Domain as context | `test_domain()` |
| Product as context | `test_product()` |
| Org as context | `system_org()` |
| Test is testing user registration | Create a user, verify the chain |
| Test is testing domain CRUD | Create/update/destroy, verify the chain |
| Test creates entities it's not testing | **Design smell — refactor to seeded data** |

Every entity creation triggers real side effects. Domain creation fires PubSub events and may start workflows. User registration creates actors, orgs, auto-joins. Creating "for convenience" multiplies side effects across the entire suite and accumulates data in the shared database.

#### Never author canonical names on `via()` — CRITICAL

Under slug-keyed idempotent upserts, `[:product_id, :slug]` is a unique identity. A **primitive-specific test** (one that exercises a single Via framework primitive — the `*_terms_test.exs` files, the per-primitive `self/*_test.exs` files) that creates a *fixed-canonical-name* primitive on the shared `via()` product will upsert onto — or, via `on_exit` destroy, **cascade-delete** — a row in VIA's **seeded self-model**. No sandbox → real commits → the collision is real (it cost us a whole flake class: `self/architecture_test` clobbering the seeded `ProductCreated`).

| Test kind | Product | Names | Pattern |
|---|---|---|---|
| Primitive mechanics (`*_terms_test.exs`) | `test_product()` | `#{unique_id()}` (disposable) | author + `on_exit` cleanup |
| Perspective self-description (`self/*_test.exs`) | `via()` | canonical | **READ** the `Via.Self.*` seed, assert (membership, not exclusive `==`); never author |
| Asserting VIA's own facts (spark, existence) | `via()` | — | read-only |

`via()` is reserved for VIA's seeded canonical self-model (`Via.Self.*` via `Via.Self.Seeds.ensure_via!`). **As of self-model seed v2, all eight perspectives — Strategy, Architecture, Products, Engineering, Governance, Operations, Payer, User — have graduated to `Via.Self.*` seed modules.** The `self/*_test.exs` perspective tests now READ the seed and assert; mechanics coverage lives in the `*_terms_test.exs` cases. The only self tests that still author on `via()` are the two sketch-walk **trace** tests, and only trace-specific *placeholder* primitives whose names are deliberately distinct from canonical slugs — each carries an explicit **authoring-allowlist** paragraph in its moduledoc (the authoring gate greps for it). A `self/*_test` that authors a fixed *canonical* name on `via()` is a regression: that content belongs in the seed, and the test reads it.

Append-only / no-upsert-identity primitives are seeded idempotently via `Via.Self.Record.ensure!/5` (product-scoped) or `ensure_world!/4` (world-scoped) — read-then-create keyed on a natural field — since many primitive resources still lack a working upsert identity (see the seed-model modules and the v2 increment's Assess).

### Async by default — CRITICAL

All tests MUST be `async: true` unless documented otherwise. Tests using unique identifiers (`unique_id`, `unique_slug`, `unique_email`) and reading seeded data are inherently safe.

Legitimate `async: false` reasons:

| Disqualifier | Example |
|---|---|
| Modifies Application env | `Application.put_env(:via, :via_infra_path, ...)` |
| Mutates shared ETS tables | `:ets.delete(:platform_role_permissions, key)` |
| PubSub broadcasts to topics other tests subscribe to | Cross-test interference |
| Modifies shared seeded data | Updating `test_product().local_path` |
| Named GenServer conflicts | Starting a globally-registered process |
| Unscoped bulk deletes | `Ash.read!() |> Enum.each(&Ash.destroy!)` without filter |

If a test needs `async: false`, comment why on the `use` line:

```elixir
# async: false — modifies Application env (:via_infra_path) and seeded product local_path
use Via.DataCase, async: false
```

Design for async first. If you reach for `async: false`, consider whether the test can be redesigned (scope deletes, unique data, avoid global state).

### Coherent testing — CRITICAL

Prefer fewer tests with more assertions over many small tests that each re-trigger the same side effects.

```elixir
# WRONG — 4 tests, 4 domain creations, 4× full side-effect chain
test "creates domain" do ... end
test "auto-assigns readers" do ... end
test "fires DomainCreated event" do ... end
test "sends notification" do ... end

# RIGHT — 1 test, 1 creation, coherent verification
test "domain creation triggers reader assignment and event" do
  {:ok, domain} = create_domain(...)

  assert domain.name == "Test"

  {:ok, memberships} = list_memberships(domain.id)
  assert length(memberships) > 0

  assert domain.status == :setting_up
end
```

- Group assertions verifying different aspects of the same operation
- Each test = one coherent scenario
- A test that creates an entity verifies the full expected side-effect chain
- Don't split side-effect verification across tests that re-trigger the same operation

### Side-effect awareness

Know the chain before triggering an operation:

- **Domain create** → DomainCreated event → may start initialize-domain workflow
- **User register** → creates Actor + Org → auto-joins orgs → accepts pending invitations
- **Membership create** → after_transaction hooks → email via MembershipSender

If the test doesn't need these, use seeded data instead.

### Cleanup

No sandbox rollback. Tests creating data they're testing must `on_exit` cleanup. But cleanup is secondary — if it gets complex, the test probably shouldn't be creating that data.

### What NOT to do

- Never create entities "for convenience" when seeded data exists
- Never tag a test `:slow` to hide poor design — fix the test
- Never add sandbox mode — the realistic model is intentional and has caught real design flaws

## Architecture: Domain Functions, Not Changesets in UI — CRITICAL

LiveViews, controllers, scripts, seeds, tests, and background jobs **MUST NOT** build Ash changesets directly to mutate entities. All entity creation, mutation, and orchestration goes through **domain functions** — Ash `code_interface` entries or plain functions on the domain module.

A LiveView, an Oban worker, a CLI command, and a JSON/GraphQL endpoint should all reach the same behaviour through the same function. Behaviour scattered into UI handlers can't be reused, can't be tested coherently, and can't be exposed as an API surface without duplication.

```elixir
# In the domain
resources do
  resource MyResource do
    define :create_my_resource, action: :create
    define :do_thing, action: :do_thing
  end
end

# Or — when orchestration spans resources or needs pre-work — a plain function:
def send_invitation(scope_type, host_id, role_id, email, opts \\ []) do
  with {:ok, scope} <- find_scope(scope_type, host_id),
       {:ok, invitation} <-
         Invitation
         |> Ash.Changeset.for_create(:create, %{...})
         |> Ash.create(actor: opts[:actor]) do
    {:ok, invitation}
  end
end
```

Then everywhere — LiveView, test, GraphQL resolver, Oban job, seed:

```elixir
MyDomain.create_my_resource(attrs, actor: actor)
MyDomain.send_invitation(:product, product.id, role.id, email, actor: admin)
```

**Rules:**

- LiveView event handlers, controller actions, scripts, seeds: never call `Ash.Changeset.for_create/for_update/for_destroy` followed by `Ash.create/update/destroy`. Always go through a domain function.
- If a domain function doesn't exist for a use case, **add one** — don't reach into the resource directly. Authorize via the actor passed in; never `authorize?: false` outside the resource.
- Resource actions are self-contained: every after_action / change / validation belongs declared on the action, not orchestrated by the caller. If a Product needs a Scope row registered on create, that's the action's job, not the LiveView's.
- The only legitimate raw-changeset zone is **inside the resource module** (changes, validations) and **bootstrap operations** (creating the global scope, the system actor, the user-registrar service actor — where there's literally no actor yet).

**Smell test:** LiveView event handler reaches for `Ash.Changeset.for_create(...)` — stop, find the missing domain function, add it. The handler should compose three lines, not thirty.

## Completion Standards

**Before reporting work complete:**

1. Old code deleted — replacing X with Y means deleting X
2. All call sites updated — grep for the old thing, update every reference
3. Wiring complete — components that should talk actually do
4. Tests pass (`mix test`)
5. No dangling references — imports, aliases, docs to deleted code
6. Documentation matches reality — don't document behaviour that isn't implemented

**Decontamination mindset.** Systematic cleanup doesn't stop halfway. Discovered debt blocking your work is resistance, not a separate concern. Fix it now.

## Common Pitfalls

### IS-A relationships use shared primary key — CRITICAL

When an Ash resource is a *subtype* of another (X IS-A Y — destroying parent destroys child, child has no identity of its own), use the shared-primary-key form. Don't add a redundant FK column.

```elixir
# WRONG — has-a 1:1 with separate FK column. Adds a redundant column,
# the invariant "user.actor_id == actor.id" lives in convention only,
# and silently breaking it produces hours of debugging.
attributes do
  uuid_primary_key :id
  attribute :actor_id, :uuid, allow_nil?: true
end

relationships do
  belongs_to :actor, Actor do
    define_attribute? false
    source_attribute :actor_id
  end
end

# RIGHT — IS-A. Same column is both PK and FK. Removing the relationship
# fails to compile. Invariant is structurally enforced.
relationships do
  belongs_to :actor, Actor do
    source_attribute :id
    primary_key? true
    allow_nil? false
    attribute_writable? true   # only if create flow forces the id
  end
end

postgres do
  references do
    reference :actor, on_delete: :delete   # cascade matches IS-A
  end
end
```

| Relationship | Pattern | Example |
|---|---|---|
| **IS-A** (subtype) | shared-PK belongs_to | `User` IS-A `Actor`, `AgentProfile` IS-A `Actor` |
| **HAS-A 1:1** (optional extension) | regular belongs_to + unique constraint on FK | `UserProfile` HAS-A `User` |
| **HAS-A 1:N or N:1** | regular belongs_to | `Document` belongs_to `Product` |
| **HAS-A N:M** | many_to_many through join | `Domain` ↔ `Work` |

Canonical examples: `Via.Saas.Identity.User` and `AgentProfile`, both subtypes of `Actor`. See `via_saas/AGENTS.md` for full rationale.

### Ash policy blocks — CRITICAL

```elixir
# WRONG — multiple blocks must ALL pass independently
policy action_type(:destroy) do
  forbid_if expr(system_flow == true)
end
policy action_type(:destroy) do
  authorize_if expr(^actor(:role) == :admin)
end

# RIGHT — single block with sequential checks
policy action_type(:destroy) do
  forbid_if expr(system_flow == true)
  authorize_if expr(^actor(:role) == :admin)
  authorize_if expr(owner_id == ^actor(:id))
end
```

### HasPermission vs HasScopePermission — CRITICAL

`HasPermission` is a `SimpleCheck` that resolves to `:global` scope on read queries (subject is `Ash.Query`, not changeset) — it only checks platform-level roles. Product/org membership permissions are invisible to it. Members will be silently denied.

```elixir
# WRONG — read fails for regular product members
policy action_type(:read) do
  authorize_if {Platform.Authorization.Checks.HasPermission,
                permission: :contribute_to_product}
end

# RIGHT — generates SQL filter on accessible product_ids
policy action_type(:read) do
  authorize_if {Platform.Authorization.Checks.HasScopePermission,
                permission: :contribute_to_product}
end
```

| Action | Use | Why |
|---|---|---|
| `:read` | `HasScopePermission` | FilterCheck — generates SQL WHERE; works with memberships |
| `:create`, `:update`, `:destroy` | `HasPermission` | SimpleCheck — infers scope from changeset data |
| Global resources (no `product_id`) | `HasPermission` | No scoping; permission is platform-level |

### Ash actor timing — CRITICAL

```elixir
# RIGHT — actor available during changeset construction
Post |> Ash.Changeset.for_create(:create, params, actor: user) |> Ash.create()

# WRONG — too late
Post |> Ash.Changeset.for_create(:create, params) |> Ash.create(actor: user)
```

### Never bypass authorization — CRITICAL

`authorize?: false` undermines the security model. There is **always** a correct actor. Use the right one for the context.

| Function | When |
|----------|-------------|
| `Platform.Identity.system_actor()` | Cached system actor with full permissions. System-context operations, no user. |
| `Platform.Identity.load_service_actor!("slug")` | Specific service actor by slug. Scoped service-level permissions. |
| `Platform.Authorization.with_permissions(actor, :global)` | Decorate an existing actor with extra permissions. |

**Inside policy checks** (need to read data to make authorization decisions):

```elixir
# WRONG — bypasses authorization to "just read"
def match?(actor, _ctx, _opts) do
  memberships = Ash.read!(Membership, authorize?: false)
  Enum.any?(memberships, &(&1.user_id == actor.id))
end

# RIGHT — system actor for policy-internal reads
def match?(actor, _ctx, _opts) do
  memberships = Ash.read!(Membership, actor: Platform.Identity.system_actor())
  Enum.any?(memberships, &(&1.user_id == actor.id))
end
```

**Background jobs / hooks with no user:**

```elixir
# RIGHT — system actor for system-initiated work
def perform(%Oban.Job{args: %{"id" => id}}) do
  system = Platform.Identity.system_actor()
  record = Ash.get!(MyResource, id, actor: system)
  Ash.update!(record, %{status: :processed}, actor: system)
end
```

**Loading relationships after an authorized read** — pass the same actor through, don't skip:

```elixir
# RIGHT
record = Ash.get!(MyResource, id, actor: user)
record = Ash.load!(record, [:organization, :memberships], actor: user)
```

**Inside after_action / after_transaction hooks** — use the changeset's actor or system actor:

```elixir
change after_action(fn changeset, record, _ctx ->
  actor = changeset.context[:private][:actor] || Platform.Identity.system_actor()
  Ash.update!(related, %{status: :active}, actor: actor)
  {:ok, record}
end)
```

**The ONLY legitimate `authorize?: false`** is inside `Platform.Identity.SystemActorCache.load_system_actor/0` — the bootstrap that loads the system actor that authorizes everything else. Single chicken-and-egg exception. Nowhere else.

If a query returns empty or an action is forbidden, the fix is the correct actor or the policy — never skipping authorization.

### `Ash.destroy()` return types

```elixir
case Ash.destroy(changeset) do
  :ok -> # direct success
  {:ok, _} -> # with return_destroyed?: true
  {:error, error} -> # error
end
```

### Elixir gotchas

- **Block expressions bind result:** `socket = if connected?(socket), do: assign(socket, :val, val), else: socket`
- **Structs don't support Access syntax:** `changeset.field`, not `changeset[:field]`
- **Lists don't support index access:** `Enum.at(list, 0)`, not `list[0]`
- **Never `String.to_atom/1` on user input** — memory leak risk
- **Never nest multiple modules in one file** — causes cyclic dependencies
- **`after_batch`** is bulk-ops only — use `after_transaction` for single creates

### Error handling: pipelines over nesting — CRITICAL

Prefer pipeline-style with `{:ok, value}` / `{:error, reason}` and function heads matching tuple shape over nested `case` or `with`. When a function has more than one level of case/with nesting, refactor to a pipeline.

```elixir
# WRONG — nested case
def process(input, ctx) do
  case validate(input) do
    :ok ->
      case execute(input, ctx) do
        {:ok, result} ->
          case post_validate(result) do
            :ok -> {:ok, result}
            {:error, reason} -> {:error, reason}
          end
        {:error, _} = error -> error
      end
    {:error, _} = error -> error
  end
end

# RIGHT — pipeline with function heads
def process(input, ctx) do
  input
  |> validate_input()
  |> execute_step(ctx)
  |> post_validate_result()
end

defp execute_step({:ok, input}, ctx), do: do_execute(input, ctx)
defp execute_step({:error, _} = error, _ctx), do: error
```

- Each stage takes `{:ok, value}` or `{:error, reason}` and passes errors through
- Function heads match on tuple shape — no nested case
- Nil values get their own head: `defp do_thing(nil), do: {:error, "thing not found"}`
- **Single-level `case`** with 2-3 branches: fine.
- **`with`**: acceptable for genuinely independent operations with simple error passthrough; prefer pipes if logic is sequential.
- **`try/rescue`**: external boundaries only (Port, System.cmd, Task.async, parsing external input). Never internal flow.

### GenServer init — CRITICAL

`init/1` must be fast, pure, infallible:

- No DB calls — use `handle_continue` for loading external data
- No bang functions — `load_service_actor!`, `File.mkdir_p!`, `Ash.get!` all crash init
- No network/filesystem I/O — defer to `handle_continue`
- Always `Process.flag(:trap_exit, true)` — ensures `terminate/2` runs
- Always implement `terminate/2` — at minimum log; ideally cleanup (mark records failed, close ports, cancel timers)

```elixir
def init(opts) do
  Process.flag(:trap_exit, true)
  state = %{id: opts[:id], resource: nil}
  {:ok, state, {:continue, :load_dependencies}}
end

def handle_continue(:load_dependencies, state) do
  case load_resource(state.id) do
    {:ok, resource} -> {:noreply, %{state | resource: resource}}
    {:error, reason} -> {:stop, reason, state}
  end
end

def terminate(reason, state) do
  Logger.info("[MyServer] Terminating: #{inspect(reason)}")
  cleanup(state)
end
```

## Web Layer

Web-layer rules (LiveView file structure, AshAuthentication discovery, typography) live in `lib/via_web/AGENTS.md` and `lib/via_web/live/AGENTS.md`. They auto-load when working in those subtrees.

## Knowledge Tools — Context7 library IDs

When fetching docs for the Ash family, use these IDs:

| Library | ID |
|---------|----|
| Ash Framework | `/websites/hexdocs_pm_ash` |
| AshPostgres | `/ash-project/ash_postgres` |
| AshAuthentication | `/websites/hexdocs_pm_ash_authentication` |
| AshPhoenix | `/websites/hexdocs_pm_ash_phoenix` |
| AshOban | `/websites/hexdocs_pm_ash_oban` |
| AshGraphql | `/ash-project/ash_graphql` |
| AshJsonApi | `/ash-project/ash_json_api` |
| Phoenix / LiveView / Elixir | resolve with Context7 |

For Apple framework docs, use Apple Docs MCP (`mcp__apple-docs__*`), not Context7.

## Apple Platform Work

When working on Apple (iOS/macOS/iPadOS) products in this repo:

- **Project tooling:** `tuist generate`, `tuist build`, `tuist test`. Headless via `xcodebuild` / `xcrun simctl`. XcodeBuildMCP if available — `.mcp.json` args need the `mcp` subcommand: `["-y", "xcodebuildmcp@latest", "mcp"]`.
- **Data layer:** Apple products choose Core Data or SwiftData based on the CloudKit sharing requirement. See "Choosing Core Data vs SwiftData" in `docs/guides/apple-platform-engineering/guide.md`.
- **Specialist agent:** activate `apple-genius` for interactive consultation.

## Project-specific environment

- **NEVER run `mix run priv/repo/seeds_test.exs` directly** — pollutes dev database.
- After creating Ash resources: `mix ash.codegen --name description` then `mix ash.migrate`.

## Utilities — use `via_utils`, don't reinvent — CRITICAL

VIA depends on `via_utils` (private library at `vigensis/via-utils`, registered as a system product). Single source of truth for generic, domain-neutral utilities.

**Before writing a utility helper, check `via_utils`:**

| Need | Module / function |
|------|-------------------|
| slugify, humanize, truncate, initials, humanize_filename | `Via.Utils.Strings` |
| relative timestamp formatting | `Via.Utils.Time.relative/1` |
| markdown rendering (GFM via MDEx, lightweight regex variant) | `Via.Utils.Markdown` |
| filesystem tree, cross-device move, sanitize_filename | `Via.Utils.Files` |
| git CLI (clone, pull, push, branch, status, log, URL parse) | `Via.Utils.Git` |
| URL type detection, HTTP reachability validation | `Via.Utils.Url` |

If `via_utils` doesn't have what you need:

1. **Generic?** Would the function still make sense in a habit-tracker SaaS or a standalone CLI? If yes, it belongs in `via_utils`. Open `~/via/vigensis/via-utils/`, add to the right module (`Strings`, `Files`, `Time`, etc.), write doctest + tests, run `mix check`, commit, push. Then `mix deps.update via_utils` in VIA.
2. **VIA-specific?** (knows about products, expertise domains, knowledge graphs, workflows) — keep in VIA.
3. **Naming.** Group by what the function operates on (`Strings`, `Files`, `Time`). `Helpers` is forbidden as a module name in `via_utils`.

Never duplicate a utility. If you find yourself writing a `slugify`, `humanize`, `format_relative_time`, or filesystem helper that already exists in `via_utils`, replace your callsite with the library call.

See `~/via/vigensis/via-utils/AGENTS.md` for contributing.

## Context & Reference

- `.claude/context/product-context.md` — product-specific context
- `.claude/context/organization-context.md` — organization-wide context
- `lib/via_web/AGENTS.md` — web-layer rules (auto-loads in subtree)
- `lib/via_web/live/AGENTS.md` — LiveView file structure (auto-loads in subtree)

For comprehensive patterns beyond this file:

- `docs/reference/idiomatic-elixir.md` — core Elixir idioms
- `docs/reference/phoenix-expert.md` — advanced LiveView/PubSub patterns
- `docs/reference/ash-expert.md` — advanced Ash patterns
- `docs/reference/exposing-domains-to-agents.md` — exposing a VIA primitive domain over the agent GraphQL surface (enum rule, action selection, auth, enforcement tests)
- `docs/reference/style-guide.md` — writing style (active voice, concise, no filler)

Read `docs/README.md` before creating or modifying documentation files.
