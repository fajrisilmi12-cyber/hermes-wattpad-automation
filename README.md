# Hermes Wattpad Automation Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/fajrisilmi12-cyber/hermes-wattpad-automation/blob/main/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/fajrisilmi12-cyber/hermes-wattpad-automation)](https://github.com/fajrisilmi12-cyber/hermes-wattpad-automation/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/fajrisilmi12-cyber/hermes-wattpad-automation)](https://github.com/fajrisilmi12-cyber/hermes-wattpad-automation/network/members)
[![GitHub last commit](https://img.shields.io/github/last-commit/fajrisilmi12-cyber/hermes-wattpad-automation)](https://github.com/fajrisilmi12-cyber/hermes-wattpad-automation/commits/main)
[![GitHub repo size](https://img.shields.io/github/repo-size/fajrisilmi12-cyber/hermes-wattpad-automation)](https://github.com/fajrisilmi12-cyber/hermes-wattpad-automation)

Hermes Agent skill to automate Wattpad story creation, chapter writing, and dual-stage publishing using Playwright and cookie authentication.

---

## Overview

This skill enables AI agents and automated scripts running on [Hermes Agent](https://github.com/NousResearch/Hermes-Agent) to authenticate, draft, and publish stories and multiple chapters directly to Wattpad without relying on deprecated or broken internal endpoints.

### Key Capabilities

- **Cookie-Based Authentication**: Seamless session restore using standard `cookie.txt` without interactive login forms or CAPTCHA triggers.
- **Modern Story Initialization**: Automates the `/write/story/new` creator route rather than the legacy `/myworks/new` modal view.
- **Custom Accessible Combobox Handling**: Handles Wattpad's custom `role="combobox"` genre and subgenre selectors dynamically.
- **Dual-Stage Publishing Confirmation**: Resolves the subtle modal overlay (`#publish-modal-overlay` / `#react-modal`) required to transition chapters from *Draft* to *100% Live*.
- **Multi-Chapter Management**: Automates creating and publishing subsequent chapters (Chapter 2, 3, etc.) via `/myworks/<story_id>`.

---

## Directory Structure

```text
hermes-wattpad-automation/
├── LICENSE
├── README.md
├── SKILL.md
├── .gitignore
└── skills/
    └── wattpad-automation/
        ├── SKILL.md
        └── references/
            └── story-creation-and-publishing.md
```

---

## Installation into Hermes Agent

### Method 1: Direct Skill Copy

Clone or copy the skill directory into your Hermes skills directory:

```bash
mkdir -p ~/.hermes/skills/social-media/wattpad-automation/
cp -r skills/wattpad-automation/* ~/.hermes/skills/social-media/wattpad-automation/
```

### Method 2: Git Submodule or Clone

```bash
git clone https://github.com/fajrisilmi12-cyber/hermes-wattpad-automation.git ~/.hermes/skills/wattpad-automation
```

---

## Prerequisites

- **Python 3.10+**
- **Playwright**:
  ```bash
  pip install playwright
  python -m playwright install chromium
  ```
- **Active Session Cookies**: Export your logged-in Wattpad cookies (specifically `token`, `wp_id`, `locale`) to a text file.

---

## Workflow & Pitfalls Solved

| Pitfall / Challenge | Root Cause | Solution Implemented |
|---|---|---|
| **Form Silent Submission Failure** | Legacy `/myworks/new` Backbone modal | Navigates directly to `https://www.wattpad.com/write/story/new` |
| **API 405 Method Not Allowed** | Direct POST to `/api/v3/stories` | Uses headless browser navigation with AWS WAF token telemetry |
| **Combobox Selection Timeout** | Genre dropdown is not native `<select>` | Targets `#fiction-type-combobox-trigger` and clicks within `[role="listbox"]` |
| **HTML5 Validation Trap** | `input#tags` marked `required` | Strips `required` attribute via DOM manipulation before form submission |
| **Stuck in "Draft" Status** | Pre-publish form redirects to editor without auto-publishing | Detects and triggers the final confirmation modal button inside `#react-modal` |

---

## License

This project is licensed under the [MIT License](LICENSE).
