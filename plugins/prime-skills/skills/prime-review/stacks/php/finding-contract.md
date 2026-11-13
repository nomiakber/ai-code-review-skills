# Finding contract

## What not to flag

- Style or formatting that PHPCS governs, or a preference between equivalent patterns. Breaks of the project's own rules are not style; flag them.
- Speculative performance ("could be slow at scale") without a concrete cost in this code. Admin code runs for one user at a time; frontend code runs on every visitor page view.
- Anything in the excluded paths.
- Changes that would break project rules.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the class and method or function name.
- **Evidence**: what the code does, and a concrete scenario where it fails or costs something (inputs or state, then the outcome). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers and the hooks that reach it.
- **Severity**:
  - Critical: exploitable security issue or data loss.
  - High: functional bug, broken flow, or a severe performance or resource problem users will hit.
  - Medium: reliability gap, or a maintenance problem with real cost.
  - Low: minor improvement, test gap on low-risk code, optional optimization.
- **Confidence**: High (the failure follows directly from the code) or Medium (the code is verified, but the impact depends on runtime conditions you can't confirm). Drop anything lower.
- **Fix**: the smallest change that resolves it, chosen with the ponytail ladder: does it need to exist → already in the codebase → PHP or WordPress core → installed dependency → one line → minimum code. Keep behavior and public APIs, including fired hooks, unless the behavior itself is the bug; if a breaking change is unavoidable, list the affected consumers. State impact in words unless you measured it.
- **Fix check**: the fix works on the minimum WordPress and PHP versions, for every caller, and for every consumer of a changed response (JS reducers, add-ons). A fix that changes a response shape names the client change it needs.
- **Severity in context**: if core or another open path already allows the same thing (for example, core's `profile.php` changes the email without a password), lower the severity and say why. If nothing reaches the path today, call it hardening.
- **Category**: one of Security, Correctness, WordPress, PHP, Performance, Resources, Maintenance, Dependencies, Testing.
