# agent-skills-collection

A private collection of custom Agent Skills (SKILL.md files) for AI coding agents — currently covering security auditing and version-control workflows.

## Skills

| Skill | Category | Description | SKILL.md |
|---|---|---|---|
| check-overflow | security | Read-only C/C++ memory vulnerability scanner — traces variable lifecycles, buffer bounds, pointer arithmetic, and integer overflow paths without modifying code. Produces a structured findings report. | [SKILL.md](skills/security/check-overflow/SKILL.md) |
| fix-overflow | security | Patches C/C++ memory-safety vulnerabilities identified by a paired detection skill (e.g. check-overflow). Applies fixes in severity order, re-verifies each one, and updates the original report. | [SKILL.md](skills/security/fix-overflow/SKILL.md) |
| git-auto | version-control | Analyzes staged and unstaged changes, writes conventional commit messages, and stages files safely. Enforces forbidden-action guards and a never-stage list for secrets and binaries. | [SKILL.md](skills/version-control/git-auto/SKILL.md) |
| github-design | version-control | Generates and organizes a project's documentation — README, CHANGELOG, docs/, and .github/ community health files. Read-only with respect to source code; outputs only standalone doc files. | [SKILL.md](skills/version-control/github-design/SKILL.md) |

## How these are used

Each skill is a standalone SKILL.md file with YAML frontmatter (name, description, category, risk level) and detailed markdown instructions. They're designed for AI coding agents that support custom skills — currently used with Antigravity CLI (Google Gemini) and compatible with any agent that reads SKILL.md files from a skills directory.

To use a skill, place or symlink its folder into the agent's configured skills path. The agent reads the SKILL.md at invocation time and follows its instructions for the task at hand.

## Repository structure

```
skills/
├── security/
│   ├── check-overflow/    # Vulnerability detection (read-only audit)
│   │   ├── SKILL.md
│   │   └── README.md      # Ashift tool download instructions
│   └── fix-overflow/      # Vulnerability patching
│       └── SKILL.md
└── version-control/
    ├── git-auto/           # Commit automation
    │   └── SKILL.md
    └── github-design/     # Documentation generation
        └── SKILL.md
```
