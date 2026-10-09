---
name: wattpad-automation
description: "Use when automating Wattpad login, story, or publishing."
version: 1.0.0
metadata:
  hermes:
    tags: [wattpad, automation, playwright, cookies, publishing, web-novel]
    category: social-media
---

# Wattpad Automation

Use when automating Wattpad actions including cookie-based session authentication, story creation, chapter drafting, and publishing via Playwright browser automation.

## Authentication & Session Management

- Parse raw header cookies or `cookie.txt` key-value pairs (specifically `token`, `wp_id`, `te_session_id`, `_ga`).
- Add cookies to the browser context for domain `.wattpad.com` and path `/`.
- Verify authenticated state by loading `https://www.wattpad.com/home` or checking for username tokens / user avatar dropdown in the DOM, rather than relying solely on HTTP response codes.

## Story Creation Flow & Pitfalls

### Critical Pitfalls
- **Avoid `/myworks/new`**: This legacy URL renders an old Backbone/React hybrid form where title input and tag sync fail, and clicking save does not submit.
- **Avoid direct POST `/api/v3/stories`**: Direct API requests to `/api/v3/stories` or `/api/v3/users/<username>/stories` return `405 Method Not Allowed` or `404 Not Found` without internal session tokens and AWS WAF telemetry.
- **Correct Entry Point**: Use modern creator route `https://www.wattpad.com/write/story/new` (accessible from `/myworks` via "+ Cerita baru" / "+ Buat cerita").

### Step-by-Step Automation Pipeline

1. **Setup Story Metadata (`/write/story/new`)**:
   - **Title**: `input#title`
   - **Description**: `textarea`
   - **Story Type (*Diperlukan)**: Click category button (`button:has-text("Fiksi")`, `Fiksi Penggemar`, `Nonfiksi`, `Puisi`).
   - **Genre Selection**: Selecting story type dynamically reveals `Genre #1` (*Diperlukan). Click `button:has-text("Pilih genre #1")` and select a primary genre (e.g. `Romansa`).
   - **Tags**: `input[placeholder*="Pisahkan tag"]`, submit with Enter.
   - **Submit**: Scroll to top (`window.scrollTo(0, 0)`) and click `button:has-text("Simpan & Lanjutkan")`.

2. **Chapter Drafting (`/myworks/<story_id>/write/<part_id>`)**:
   - **Chapter Title**: Focus and type into chapter title element.
   - **Chapter Body**: Click `.story-editor` / `[contenteditable="true"]`, type text by paragraphs.
   - **Autosave Verification**: Check draft status indicator (`Draf (N kata) Tersimpan`).
   - **Publish Navigation**: Click `page.get_by_role("button", name="Publikasikan")` to transition to review.

3. **Pre-Publish Metadata (`/write/story/<story_id>/part/<part_id>/publish`)**:
   - **Genre #1 (*Diperlukan)**: Must be selected via the custom accessible combobox trigger `#fiction-type-combobox-trigger` (role `combobox`, expands `[role="listbox"]`). Selecting a primary genre (e.g., `Romansa`) expands subgenre triggers.
   - **HTML5 Validation Trap**: The text input `input#tags` may have `required` set while tag pills are already active in hidden inputs. Remove the attribute via `removeAttribute('required')` to avoid silent form submission blocks.
   - **Submit**: Scroll to top and click `button:has-text("Simpan & Publikasikan")`.

4. **Final Confirmation Modal ("Publikasikan bab") — CRITICAL**:
   - Submitting the `/publish` form redirects back to the chapter editor (`/myworks/<story_id>/write/<part_id>`).
   - At this point, the story is **STILL A DRAFT**. A modal dialog titled **"Publikasikan bab"** with overlay `#publish-modal-overlay` appears (or is triggered by clicking "Publikasikan" in the header).
   - Radio option "Publikasikan sekarang" is preselected.
   - Click the wide black button `Publikasikan` inside the modal (`locator('[role="dialog"] button:has-text("Publikasikan"), .modal button:has-text("Publikasikan")')`).
   - Only after this modal confirmation does the chapter transition from `Draf` to live reader view (`100% Lengkap`, reading time, engagement metrics).
   - Verify live story at canonical URL `https://www.wattpad.com/story/<story_id>-<slug>`: ensure the grey `Draf` badge above the title is absent.

5. **Adding Subsequent Chapters (Chapter 2, 3, etc.)**:
   - Navigate to the story overview page: `https://www.wattpad.com/myworks/<story_id>`.
   - Click the "+ Bab Baru" button (`button.on-newpart` or `button:has-text("Bab Baru")`).
   - Browser navigates to the new part editor: `https://www.wattpad.com/myworks/<story_id>/write/<new_part_id>`.
   - Set chapter title on `h2#story-title` (contenteditable), and body on `div.story-editor`.
   - Click top-right publish button (label may be `Publikasikan` or English `Publish`).
   - Confirm publication in the "Publikasikan bab" modal (`role="dialog"` -> `button:has-text("Publikasikan")`).
   - Verify on the public story page: both chapters appear under the `Bab` tab with public timestamps (e.g., *"1 menit yang lalu"*).

## References

- `references/story-creation-and-publishing.md` — Playwright boilerplate, selector specifications, and verification snippets.
