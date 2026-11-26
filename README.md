# ub-mode

`ub-mode` is an agent skill that enforces disciplined software engineering principles when writing or changing code. It is loosely based on portions of [@poteto](https://github.com/poteto)'s rules (`poteto-mode` / `pstack`), reduced and paired with deterministic quality-gate tools:

1. **First Make it Work** — Laziness Protocol, Foundational Thinking, TDD, Redesign From First Principles, Attack the Premise, Subtract Before You Add, Outcome-Oriented Execution, Build the Lever, and Architecture (Model the Domain, Boundary Discipline, Type System Discipline, Make Operations Idempotent, Separate Before Serializing Shared State).
2. **Then Make it Right** — Fix Root Causes, Sequence Work into Verifiable Units, Test Behavior (Not Implementation), [Crapper](https://github.com/cbarrymore/crapper) (`CRAP < 10`), [Dryer](https://github.com/cbarrymore/dryer) (duplication `< 0.8`), [Mutator](https://github.com/cbarrymore/mutator) (cover the uncovered, kill the survivors, delete dead code), and Refactoring.
3. **Practice** — Keep a Log, Guard the Context Window, Worktree Cleanup, and Commit and Push in small ordered commits.

This fork targets C# projects on Windows. The skill calls the tools from PowerShell through their venv executables.

## Companion Tool Repositories

`ub-mode` relies on deterministic quality-gate tools to enforce structure, complexity, duplication, and test thoroughness. This fork uses the C#-capable forks:

- [cbarrymore/crapper](https://github.com/cbarrymore/crapper) — Computes cyclomatic complexity, test coverage, and CRAP score per function so every function stays under the CRAP threshold (`< 10`).
- [cbarrymore/dryer](https://github.com/cbarrymore/dryer) — Detects structural code duplication so duplicate blocks (`>= 0.8` similarity) are eliminated.
- [cbarrymore/mutator](https://github.com/cbarrymore/mutator) — Runs mutation testing to expose uncovered code and surviving mutants that tests fail to catch.
- [unclebob/uml-viewer](https://github.com/unclebob/uml-viewer) — Visualizes components, coverage, CRAP, mutation scores, and dependency directions.

## Requirements

- Windows with PowerShell
- Python 3.11 or newer on `PATH` as `python`
- .NET 8 SDK or newer
- Test projects that reference `coverlet.collector` (for coverage)
- Developer Mode or an elevated shell for mutator (its workers use symlinks)

## Installation

Clone the three tools side by side (mutator imports `..\crapper`) and point `UB_TOOLS` at their parent directory:

```powershell
$tools = "$HOME\ub-tools"
New-Item -ItemType Directory -Force $tools | Out-Null
foreach ($t in 'crapper', 'dryer', 'mutator') {
  git clone "https://github.com/cbarrymore/$t.git" "$tools\$t"
  Push-Location "$tools\$t"; python -m venv .venv; .venv\Scripts\pip install -e .; Pop-Location
}
[Environment]::SetEnvironmentVariable('UB_TOOLS', $tools, 'User')
```

Any directory works; only `UB_TOOLS` has to point at it. Open a new shell (and restart the agent) so it sees the variable. If a tool's venv is missing, the skill installs it on first use.

Copy `SKILL.md` into your agent's skills directory as `ub-mode\SKILL.md` (for example `$HOME\.claude\skills\ub-mode\SKILL.md` or `$HOME\.cursor\skills\ub-mode\SKILL.md`), then invoke `/ub-mode`.
