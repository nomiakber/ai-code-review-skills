# Finding contract

## What not to flag

- Style or formatting that lint or Prettier governs, or a preference between equivalent patterns. Breaks of the project's own rules (for example Tailwind v3 class names in JSX) are not style; flag them.
- Speculative performance ("could be slow at scale") without a concrete cost in this code. Admin screens run for one user at a time; frontend scripts run on every visitor page view.
- Anything in the excluded paths.
- Changes that would break project rules.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the function or component name.
- **Evidence**: what the code does, and a concrete scenario where it fails or costs something (inputs or state, then the outcome). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers and the server side where relevant.
- **Severity**:
  - Critical: exploitable security issue or data loss.
  - High: functional bug, broken flow, or a severe performance or resource problem users will hit.
  - Medium: accessibility barrier, reliability gap, or a maintenance problem with real cost.
  - Low: minor improvement, test gap on low-risk code, optional optimization.
- **Confidence**: High (the failure follows directly from the code) or Medium (the code is verified, but the impact depends on runtime conditions you can't confirm). Drop anything lower.
- **Fix**: the smallest change that resolves it, chosen with the ponytail ladder: does it need to exist → already in the codebase → platform or `@wordpress/*` → installed dependency → one line → minimum code. Keep behavior and public APIs unless the behavior itself is the bug; if a breaking change is unavoidable, list the affected consumers. State impact in words unless you measured it.
- **Fix check**: the fix works in the project's supported browsers and WordPress version, for every caller and consumer (other screens, add-ons using a public global or filter), and against the server's real response shape. A fix that needs a server change names it.
- **Severity in context**: if another open path already allows the same thing, lower the severity and say why. If nothing reaches the path today, call it hardening.
- **Category**: one of Security, Correctness, React, WordPress, Performance, Resources, Accessibility, Maintenance, Dependencies, Testing.
