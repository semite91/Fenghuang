# Fenghuang

Fenghuang is a Claude Code plugin marketplace. It holds the team's coding standards and workflow as skills, so every Claude Code session follows the same rules.

The marketplace is named `semite` and contains one plugin, `semite-standards`.

## Installation

Add the marketplace and install the plugin from inside Claude Code:

```
/plugin marketplace add <path-or-git-url-of-this-repo>
/plugin install semite-standards@semite
```

Once installed, Claude loads a skill on its own when a task matches its description. You can also call a skill directly, for example `/semite-standards:coding-standards`.

## Repository layout

```
.claude-plugin/marketplace.json        Marketplace manifest
plugins/semite/.claude-plugin/plugin.json   Plugin manifest
plugins/semite/skills/<skill>/SKILL.md      One folder per skill
project_template.md                    Template for starting a new project brief
```

## Skills

| Skill | Use it when | Summary |
|---|---|---|
| `standards` | Starting any coding task | The overall process: agents, atomic tasks, planning, and when to ask first |
| `workflow` | Planning, testing, reviewing, and opening PRs | The step-by-step TDD workflow and session handoff prompts |
| `coding-standards` | Writing or reviewing code | Naming, clean code, performance, and security rules |
| `frontend-standards` | Building frontend components | Rules for where components live and what they contain |
| `design` | Styling and visual design work | LESS structure, design tokens, and the design source |
| `3rd-party-libraries` | Adding or using an external library | Security vetting, placement, and how to use it correctly |

### `standards`: coding process

The entry point that ties the other skills together.

- Sets up separate agents for backend, frontend, testing, and code review. The agents tell each other about changes that affect them, and their decisions are summarized and brought to the user.
- Splits requests, review results, and test results into atomic steps, where each step has one purpose.
- Plans with Matt Pocock's `grill-with-docs` skill before coding starts.
- Lets `coding-standards` win over Claude's own defaults when the two disagree.
- Sends library questions to `3rd-party-libraries`.
- Asks before changing more than one file.

### `workflow`: test-driven workflow

**Plan and code**
1. Turn the requirements into test scenarios, split into atomic steps.
2. Record the scenarios on the related GitHub issue, not in separate issues.
3. Save them as `infrastructure/test/test_scenarios_YYYY_MM_DD_hh_mm.txt`.
4. Wait for the user to approve them by hand.
5. Write the technical documentation.
6. Stop after three attempts at the same action.

**Review and test**
1. Check the code against the `test_scenarios` file and save the results as dated `.txt` files.
2. List the required changes in the terminal as an SRS, apply them, and check again until nothing fails.
3. Check code quality with Alibaba's OpenCodeReview tool.

**PR**
1. Put the last approved SRS in the PR description.
2. Commit every change when the work is finished.

The skill also includes:
- Two prompts for moving work to a new session when the token budget runs out: one for the old session to summarize itself, and one for the new session to pick the work back up without assuming anything.
- A fixed response format for reporting code changes (file, change, and cause, plus a map of the changes).

### `coding-standards`: clean code

- **Naming**
  - `camelCase` for variables, parameters, and frontend methods.
  - `PascalCase` for classes, backend methods, DTOs, events, DB objects, and enums.
  - `UPPERCASE` for constants.
  - `IPascalCase` for interfaces.
  - `snake_case` for frontend file and folder names.
- **Principles**
  - Follow SOLID, DRY, and YAGNI.
  - Keep methods small and names meaningful.
  - No magic numbers or hardcoded values.
  - Ask before introducing a design pattern.
- **Performance (.NET / EF Core)**
  - Analyze each method's Big O complexity.
  - Dispose resources and manage their lifetimes.
  - Use pagination, projection, or streaming instead of fetching everything.
  - Filter in the database with `IQueryable`.
  - Use `AsNoTracking` by default and `AsSplitQuery` where it is faster.
  - Give every cache bounds.
  - Use `StringBuilder` for large string work.
- **Security**
  - Keep no secrets in the repo.
  - Use HTTPS.
  - Sanitize input and prevent SQL injection.
  - Add CSP against XSS, an allowlist against SSRF, and antiforgery tokens against CSRF.
- **Tests:** keep tests separate, with their configuration in env files.

### `frontend-standards`: components

- Never create a component without asking. The user decides which components exist.
- Put components in `/wwwroots/src/components/<component_name>/`.
- Use a library already in the project before writing new code.
- Make components reusable, with no hardcoded values and no business logic.
- Put component-specific services in the component's folder.
- Never call an API from inside a component. Use a service file, and ask which API file to use.

### `design`: styling and design system

- Use LESS. Put the shared tokens in `wwwroot/src/css/`:
  - `typography.less`, with modes such as `big-font` and `small-font`
  - `color.less`, with modes such as `dark` and `light`
  - `border.less`, for borders, radii, and shadows
  - `spaces.less`, for spacing
- Put images in `wwwroot/src/image/`.
- Give each component its own style file in the component's folder, never inline in its HTML or TypeScript.
- Base architecture decisions on the Google Modern Web Guidance MCP together with the UI/UX Pro Max skills.
- Designs come from Claude Design. In Claude Code, pull them in with `/design-sync`.

### `3rd-party-libraries`: external dependencies

- **Vet before adding.** Check:
  - The source (an official registry only)
  - Maintenance activity and known vulnerabilities
  - Install scripts and native binaries
  - Network, filesystem, and environment access
  - Obfuscated code, input handling, and cryptography
  - Secure defaults, CI and tests, security tooling, and signed releases
- **Justify it.** A library must not add complexity, slow the code down, or open security holes.
- **Placement.** Put libraries in `outsource/`, and multi-file libraries in a folder named after the library.
- **Use it correctly.** Use documentation for the installed version, in this order:
  1. Context7 or a similar docs tool
  2. Official docs
  3. Changelogs
  4. Installed types and source
  5. Web search

  Avoid deprecated APIs, follow the migration guides, and prefer the library's own patterns.
- **Verify.** Run type checks, lint, and tests, watch for deprecation warnings, and commit the lockfile.

## Starting a new project

`project_template.md` is a brief to fill in at the start of a project. It covers construction (an AI-first approach and which agent harness to use), design, frontend, backend/API, security, AI-native features, and technical tools. Give it to Claude together with these skills to plan the project step by step.
