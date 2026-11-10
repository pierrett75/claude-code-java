# claude-code-java

Claude Code plugin with my conventions for Java 25 / Spring Boot 4 projects.

## Skills
| Skill | Use for |
|-------|---------|
| `spring-boot-patterns` | Controllers, services (interface + Impl), repositories, DTOs, exception handling, config, tests |
| `clean-code` | Naming, method size, nesting, duplication, comments, magic values |
| `solid-principles` | Class design: SRP, Open/Closed, Liskov, Interface Segregation, Dependency Inversion |
| `security-audit` | Security rules and audits mapped to OWASP Top 10:2025 (Spring Security 7) |

## Install

### Option A: local install from this folder (no GitHub needed)
In Claude Code (any project):
```
/plugin marketplace add C:\Users\pierr\project\claude-code-java
/plugin install java-spring-conventions@pierrett75-java
```
`pierrett75-java` is the marketplace name from `.claude-plugin/marketplace.json`. Restart Claude Code (or run `/reload-plugins`) afterwards.

### Option B: install from GitHub (any computer)
```
/plugin marketplace add pierrett75/claude-code-java
/plugin install java-spring-conventions@pierrett75-java
```

### Option C: try it for one session only (while editing skills)
From a terminal:
```
claude --plugin-dir C:\Users\pierr\project\claude-code-java
```
Nothing is installed; the plugin is loaded just for this session. A `--plugin-dir` copy takes precedence over an installed plugin with the same name, so you can test changes without uninstalling.

### Check it's loaded
- `/plugin` opens the plugin manager (installed plugins, enable/disable, uninstall).
- Type `/java-spring-conventions:` and the four skills should be suggested.
- `claude plugin validate C:\Users\pierr\project\claude-code-java` checks the manifest and structure.

## Using the skills

### Automatically
You don't have to call them. Claude sees each skill's `description` and loads the full skill when the task matches. For example:
- "Create a REST controller and service for orders" -> `spring-boot-patterns`
- "Refactor this method, it's hard to read" -> `clean-code`
- "This class does too much" -> `solid-principles`
- "Check this endpoint for vulnerabilities" -> `security-audit`

### Explicitly
Plugin skills are namespaced with the plugin name: `/java-spring-conventions:<skill-name>`, optionally followed by your request.
```
/java-spring-conventions:spring-boot-patterns create CRUD for Product (name, price, stock)
/java-spring-conventions:clean-code review src/main/java/com/example/order/OrderServiceImpl.java
/java-spring-conventions:solid-principles check the payment package
/java-spring-conventions:security-audit audit the whole project before release
```
Calling a skill explicitly is useful when Claude didn't pick it up automatically, or when you want a review against exactly these rules.

### Combining skills
Ask for several at once, e.g. "Review OrderController against spring-boot-patterns, clean-code and security-audit". Skills reference each other where rules overlap; `spring-boot-patterns` is the source of truth for project conventions.

## Updating

After editing a skill:
1. Bump `version` in `.claude-plugin/plugin.json`.
2. Commit and push (for GitHub installs).
3. In Claude Code:
   ```
   /plugin marketplace update pierrett75-java
   /reload-plugins
   ```
   For a local install (Option A), `/reload-plugins` is enough after the marketplace update; for Option C just run `/reload-plugins`.

## Using the skills in the Claude app (claude.ai / desktop)
The plugin format is for Claude Code. For the Claude app, zip a single skill folder (e.g. `skills/clean-code/` containing `SKILL.md`) and upload it under Settings -> Capabilities -> Skills. Upload each skill separately.

## Adding a skill
1. Copy `templates/skill/SKILL.md` to `skills/<skill-name>/SKILL.md`.
2. Set `name` (same as the folder) and a `description` that says when to use it - this is what triggers it automatically.
3. Keep it short: list your own rules, not textbook material Claude already knows.
4. Add it to the table above, bump the version, commit, update.
