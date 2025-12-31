# Claude Code Guidelines

## Coding Philosophy

- **Don't jump into coding immediately.** Present options considered and your recommendation to the user first.
  - Do this when: multiple valid approaches exist, the choice affects architecture, or the task is ambiguous.
  - Skip this when: the task is straightforward, follows established patterns, or the user gave specific instructions.
  - When uncertain: bias toward action with a brief note explaining your choice.

- **Simplicity is the ultimate sophistication.** Do not over-engineer. Add complexity only to the minimal level required to achieve objectives.
  - Signs of over-engineering: abstractions with one implementation, configuration for things that never change, unused "flexibility".
  - When in tension: err toward simplicity now. It's easier to add complexity later than to remove it.

- **Balance short-term vs long-term.** The long-term is uncertain, so don't over-engineer for it. However, don't myopically focus on short-term only. Focus on preventing long-term goals from being blocked while delivering short-term.
  - Deliver working solutions for immediate needs.
  - Avoid choices that would make future changes significantly harder.
  - Don't build for hypothetical requirements, but leave room for them when it costs little.

- **Let the user validate runtime behavior** in most cases, but be ready to do this yourself if asked. Guide the user on how to verify behavior locally or remotely.

## Code Organization & Factoring

### Striving for a Single Source of Truth

- **Prefer sharing over duplication**, but recognize the tradeoff. Sharing is ideal when the concept is truly the same; duplication can sometimes be acceptable when sharing would introduce unnecessary coupling or complexity.
  - Accept duplication when: sharing would create awkward dependencies, the "shared" code would need constant special-casing, or the concepts only look similar but will evolve differently.

- **Before writing new code**, search for existing implementations. Reuse or extend when it makes sense.

- When modifying code, look for similar patterns elsewhere that could benefit from consolidation.

### Constants & Literal Values

Extract magic strings, numbers, and config values to named constants. Organize by scope:

| Scope | Location | Example |
|-------|----------|---------|
| File-private | Top of the file | `const MAX_RETRIES = 3` |
| Module-shared | Module's constants file | `constants.ts` in feature directory |
| Global | Root-level constants | `src/constants/` or `src/config/` |

Prefer explicit names over inline values. `DEBOUNCE_MS` is better than `300`.

### Type Organization

- Shared types belong in dedicated type files/directories, not scattered across implementations.
- Colocate types with their primary usage when not shared.
- Avoid defining the same fundamental type differently in multiple files.

### Functions & Utilities

- Extract repeated logic into shared utilities at the appropriate level.
  - When the same logic appears in 2+ places with identical intent, extract it.
  - When similar logic appears but with different contexts, duplication may be acceptable. Evaluate whether a shared abstraction would be clear or forced.
- Prefer composable, single-purpose functions over monolithic ones.

### Continuous Improvement

- **When touching code, look for opportunities to improve organization.**
  - Primary: Complete the task you were asked to do.
  - Secondary: Fix adjacent issues (duplication, poor naming) only if small and directly related.
  - Don't: Expand scope significantly or make "while I'm here" improvements that aren't requested.
  - When you notice larger issues, mention them to the user rather than fixing unasked.

- Consolidate duplicates discovered during implementation.
- Refactor toward better structure incrementally, not all at once.
- Leave code better than you found it, but don't scope-creep.

## Naming & Readability

- **Clarity beats brevity.** `getUserAccountBalance` is better than `getAcctBal`.
- Names should reveal intent. If a comment is needed to explain what something is, the name should probably be improved.
- Comments explain *why*, not *what*. The code shows what; comments explain non-obvious reasoning or constraints.
- Avoid abbreviations unless universally understood in the domain (e.g., `http`, `url`, `id`).

## API & Interface Design

- **Consistency within a codebase beats theoretical ideals.** Match existing patterns unless there's strong reason to deviate.
- Make invalid states unrepresentable when possible. Design types so wrong usage doesn't compile.
- Prefer explicit parameters over implicit context or global state.
- Consider the caller's perspective: is this interface easy to use correctly and hard to use incorrectly?

## Error Handling & Robustness

- **Fail fast for programmer errors**, handle gracefully for user/external errors.
  - Programmer errors (bugs, invalid states that shouldn't happen): fail with clear messages.
  - User/external errors (bad input, network failures): handle with recovery or actionable feedback.
- Error messages should be actionable: what happened, why, and ideally what to do about it.
- Validate at system boundaries (user input, external APIs). Trust internal code paths.
- Prefer specific error types over generic ones when the caller needs to distinguish failure modes.

## Design for Failure

- **Assume things will fail.** Networks, disks, external services, even your own code. Design with this in mind.
- **Timeouts are mandatory** for external operations. A missing timeout is a latent bug.
- **Retries need backoff.** Immediate retry storms make outages worse.
- **Idempotency enables recovery.** If an operation can be safely retried, it should be designed that way.
- **Graceful degradation over total failure.** When a non-critical component fails, continue with reduced functionality.
- **Circuit breakers prevent cascade.** When a dependency is failing, stop calling it temporarily.
- **Fail-safe defaults.** In error states, default to the safer/more restrictive behavior.

## Security Mindset

- **Never trust external input.** Validate and sanitize at system boundaries.
- Principle of least privilege: request only permissions needed, expose only what's necessary.
- Be aware of common vulnerabilities: injection (SQL, command, XSS), authentication bypasses, insecure defaults.
- When handling credentials, secrets, or PII: consider storage, transmission, logging, and access control.
- When uncertain about security implications, flag for review rather than guessing.

## Configuration

- **Externalize environment-specific values.** Never hardcode URLs, credentials, or env-specific settings.
- **Secrets never in code or version control.** Use environment variables, secret managers, or secure vaults.
- **Validate config early.** Fail fast on startup if misconfigured, not later at runtime.
- Distinguish between: static config (deploy-time), dynamic config (runtime-changeable), and secrets.

## Caching

- **Cache invalidation is hard.** Be explicit about your invalidation strategy before adding a cache.
- **Measure before caching.** Don't add complexity for theoretical performance gains.
- **Every cache needs a TTL or explicit invalidation.** "Cache forever" is rarely correct.
- Cache at the right layer. Closer to the edge is faster but harder to invalidate.

## Operability

- **Health check endpoints** for anything long-running.
- **Graceful shutdown.** Finish in-flight work, stop accepting new work.
- **Structured logging with context.** Include request IDs, user IDs, and other correlation data.

## Performance

- **Don't optimize without evidence.** Measure first, then optimize the actual bottleneck.
- Clarity trumps micro-optimization. Prefer readable code unless profiling proves it's a problem.
- Consider algorithmic complexity (O(n) vs O(n²)) during design, not just after problems appear.
- Be mindful of performance cliffs: operations that are fast at small scale but degrade badly.

## Testing Philosophy

- **Test behavior, not implementation.** Tests should survive refactoring.
- Focus testing on: complex logic, edge cases, integration points, and things that have broken before.
- Prefer real dependencies over mocks when practical. Mock at true boundaries (network, filesystem, time, external services).
- Test names should describe the scenario and expected outcome, not the method being tested.

## Dependencies

- **Prefer standard library and existing project dependencies** before adding new ones.
- When evaluating new dependencies: Is it actively maintained? Is the size/complexity justified? Can we remove it later if needed?
- Small, focused dependencies are preferable to large frameworks when you only need one feature.
- Understand what a dependency does before adding it. Audit for security-sensitive code.

## Debugging Approach

- **Reproduce reliably before attempting to fix.** A fix you can't verify isn't confirmed.
  - When the root cause is obvious from code analysis, fix directly.
  - When unclear, reproduce first, then narrow down systematically.
- Form a hypothesis, then test it. Don't just try things randomly.
- Add logging/instrumentation as you debug; it often helps future debugging too.

## Git Operations

- Do not perform git staging, un-staging, committing, pushing, or deleting unless you have verified permission from the user.

## Effective Tool Usage

### Hierarchical Exploration

- **Context is finite and must be managed deliberately.** A 200K token window can't hold a production codebase. A single large file might be 10K tokens; loading files directly fills context with raw code, leaving less room for reasoning. When context becomes saturated with file dumps and debug output, quality degrades.

- **Use hierarchical abstraction to compress information.** Don't load raw code into your main session. Build layers of abstraction, each compressing the layer below:
  ```
  Raw Codebase (millions of lines)     → 100% of information
           ↓
  File Summaries & Dependency Maps     → ~5%
           ↓
  Architectural Understanding          → ~0.5%
           ↓
  Task-Specific Decision               → ~0.05%
  ```
  Each layer filters ruthlessly. A million lines of code becomes a few thousand tokens of relevant context.

- **Your main session is an orchestrator, not a workhorse.** Reading 15 files directly costs ~80K tokens. Subagents reading those same files and returning summaries costs ~5K tokens in your main session. Delegate the heavy reading.

- **Use parallel subagents for exploration, then synthesize:**
  ```
  Level 1: Parallel exploration subagents
  ├── Subagent A: Explore auth module → returns summary
  ├── Subagent B: Explore API routes → returns summary
  ├── Subagent C: Explore data layer → returns summary
  └── Subagent D: Explore UI components → returns summary

  Level 2: Synthesis subagents (if needed)
  ├── Subagent E: Combine A + B findings → architectural summary
  └── Subagent F: Combine C + D findings → data flow summary

  Level 3: Final synthesis
  └── Main session: Integrate all summaries into coherent understanding
  ```

- **When to use hierarchical exploration:**
  - Codebase has many files across multiple modules
  - You need to understand patterns across different areas
  - Direct file reading would consume too much context

- **How to delegate effectively:**
  - Use Task or Explore agents for heavy reading
  - Give each agent a focused scope (one module, one concern)
  - Request summaries, not raw file dumps
  - Run independent explorations in parallel
  - If context matters, include relevant plan details in the prompt or instruct the agent to read plan.md

### Delegating Verbose Operations

- **Delegate anything that produces verbose output.** Running tests, analyzing logs, searching across many files, reviewing build output—these belong in subagents that return summaries, not raw dumps.

- **The principle:** If raw output would be thousands of tokens but actionable information fits in a structured summary, delegate it.

- **What a good summary includes:**

  **Test runs:**
  - Overall pass/fail counts
  - Each failing test: name, file location, assertion that failed
  - Error messages (deduplicated if repeated)
  - Patterns observed ("all failures involve database connections")
  - Enough detail to fix or know exactly where to look next

  **Log analysis:**
  - Error types and frequencies
  - Specific error messages (deduplicated)
  - Time patterns (when errors started, frequency, ongoing vs resolved)
  - Affected services/components
  - Correlations or likely root causes

  **Code search:**
  - Where the pattern appears (files, functions, line numbers)
  - How it's used in each context
  - Usage patterns ("mostly in controllers, some in tests")
  - Anything unusual worth noting
  - Enough to act without re-searching

  **Build errors:**
  - Each error with file:line and the actual message
  - What each error means / likely cause
  - Dependencies between errors ("fixing #1 may resolve #2-4")

- **What to tell the subagent:**
  - Run the operation
  - Analyze the results
  - Return: what happened, what failed, what patterns emerged
  - Include specific locations/names needed for follow-up
  - Omit the raw output unless explicitly asked
