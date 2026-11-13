# Finding contract

Owned by this skill. The skill's `review-criteria.md` supplies Review areas and the Category list. Sub-agents read this file together with that criteria file.

## What not to flag

- Style or formatting that the project's lint or formatter governs, or a preference between equivalent patterns. Breaks of the project's own rules are not style; flag them.
- Speculative performance ("could be slow at scale") without a concrete cost in this code. Judge cost against how the code actually runs (for example one-user admin vs every visitor request vs background workers).
- Anything in the excluded paths.
- Changes that would break project rules.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the function, method, component, or class name that owns the behavior.
- **Evidence**: what the code does, and a concrete scenario where it fails or costs something (inputs or state, then the outcome). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers and consumers where relevant (including server or API handlers when the skill's Extra scope says to follow them).
- **Severity**:
  - Critical: exploitable security issue or data loss.
  - High: functional bug, broken flow, or a severe performance or resource problem users will hit.
  - Medium: accessibility barrier, reliability gap, or a maintenance problem with real cost.
  - Low: minor improvement, test gap on low-risk code, optional optimization.
- **Confidence**: High (the failure follows directly from the code) or Medium (the code is verified, but the impact depends on runtime conditions you can't confirm). Drop anything lower.
- **Fix**: the smallest change that resolves it, chosen with the ponytail ladder: does it need to exist → already in the codebase → platform or framework API → installed dependency → one line → minimum code. Keep behavior and public APIs unless the behavior itself is the bug; if a breaking change is unavoidable, list the affected consumers. State impact in words unless you measured it.
- **Fix check**: the fix works on the project's declared runtime targets (language, framework, and browsers or platforms the project supports), for every caller and consumer, and against real response or data shapes. A fix that needs a change outside the audited folder names that change.
- **Severity in context**: if another open path already allows the same thing, lower the severity and say why. If nothing reaches the path today, call it hardening.
- **Category**: one category from the calling skill's Category list in `review-criteria.md`, using those names verbatim.
