# Wattpad Story Creation & Publishing Reference

## Playwright Boilerplate for Cookie Auth & Publishing

```python
import os
from urllib.parse import unquote
from playwright.sync_api import sync_playwright

def parse_cookie_header(cookie_raw: str, domain: str = ".wattpad.com"):
    cookies = []
    for part in cookie_raw.split(";"):
        part = part.strip()
        if not part or "=" not in part:
            continue
        k, _, v = part.partition("=")
        cookies.append({
            "name": k.strip(),
            "value": unquote(v.strip()),
            "domain": domain,
            "path": "/"
        })
    return cookies

def create_and_publish_story(cookies, title, description, chapter_title, chapter_text, genre="Romansa"):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True, args=["--no-sandbox", "--disable-dev-shm-usage"])
        ctx = browser.new_context(
            user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120 Safari/537.36",
            locale="id-ID",
            viewport={"width": 1366, "height": 900}
        )
        ctx.add_cookies(cookies)
        page = ctx.new_page()

        # Step 1: Metadata setup
        page.goto("https://www.wattpad.com/write/story/new", wait_until="networkidle")
        page.fill("input#title", title)
        page.fill("textarea", description)
        page.locator('button:has-text("Fiksi")').first.click()
        page.wait_for_timeout(500)

        # Select Genre #1
        genre_btn = page.locator('button:has-text("Pilih genre #1")')
        if genre_btn.is_visible():
            genre_btn.click()
            page.locator(f'text={genre}, [role="option"]:has-text("{genre}")').first.click()

        # Save and proceed to editor
        page.evaluate("window.scrollTo(0, 0)")
        page.locator('button:has-text("Simpan & Lanjutkan")').click()
        page.wait_for_timeout(8000)

        # Step 2: Chapter editor
        # Focus title
        page.locator('text=Bab 1 tanpa judul').last.click()
        page.keyboard.press("ControlOrMeta+A")
        page.keyboard.type(chapter_title)

        # Fill body
        body = page.locator('.story-editor, [contenteditable="true"]').last
        body.click()
        for para in chapter_text.split("\n\n"):
            if para.strip():
                page.keyboard.type(para.strip())
                page.keyboard.press("Enter")
                page.keyboard.press("Enter")

        page.wait_for_timeout(3000)
        page.get_by_role("button", name="Publikasikan").click()
        page.wait_for_timeout(4000)

        # Step 3: Publish metadata page
        page.wait_for_timeout(3000)
        # Select Genre #1 via combobox
        try:
            page.click("#fiction-type-combobox-trigger", timeout=3000)
            page.wait_for_timeout(800)
            page.locator(f'[role="listbox"] >> text={genre}').last.click()
            page.wait_for_timeout(800)
        except Exception:
            pass

        # Select target audience if available
        try:
            page.select_option("#target-audience", index=1)
        except Exception:
            pass

        # Remove redundant required attribute from tags input to avoid silent HTML5 block
        page.evaluate('''() => {
            const t = document.getElementById("tags");
            if (t && t.hasAttribute("required")) t.removeAttribute("required");
        }''')

        page.evaluate("window.scrollTo(0, 0)")
        page.locator('button:has-text("Simpan & Publikasikan")').click()
        page.wait_for_timeout(6000)

        # Step 4: Final Modal Confirmation ("Publikasikan bab")
        # Submitting redirects back to /write/<part_id> where the confirmation modal appears.
        # If not visible, click the header "Publikasikan" once to open it.
        modal_btn = page.locator('[role="dialog"] button:has-text("Publikasikan"), .modal button:has-text("Publikasikan")').last
        if not modal_btn.is_visible():
            page.locator('button:has-text("Publikasikan")').first.click()
            page.wait_for_timeout(2000)
            modal_btn = page.locator('[role="dialog"] button:has-text("Publikasikan"), .modal button:has-text("Publikasikan")').last

        if modal_btn.is_visible():
            modal_btn.click()
            page.wait_for_timeout(6000)

        story_id = page.url.split("/myworks/")[1].split("/")[0] if "/myworks/" in page.url else page.url.split("/story/")[1].split("/")[0]
        public_url = f"https://www.wattpad.com/story/{story_id}"
        browser.close()
        return public_url
```

## Key Selectors & Element Contracts

| Step | Purpose | Selector / Strategy |
|---|---|---|
| Step 1 | Story Title | `input#title` |
| Step 1 | Description / Blurb | `textarea` |
| Step 1 | Category Pill | `button:has-text("Fiksi")` |
| Step 1 | Dynamic Genre #1 | `button:has-text("Pilih genre #1")` |
| Step 1 | Save & Continue | `button:has-text("Simpan & Lanjutkan")` (requires top scroll) |
| Step 2 | Part Title | `.part-title`, `h1`, or text replace on `Bab 1 tanpa judul` |
| Step 2 | Part Body Canvas | `.story-editor` or `[contenteditable="true"]` |
| Step 2 | Publish Trigger | `page.get_by_role("button", name="Publikasikan")` |
| Step 3 | Genre Combobox | `#fiction-type-combobox-trigger` -> `[role="listbox"] >> text=<Genre>` |
| Step 3 | Save & Publish Metadata | `button:has-text("Simpan & Publikasikan")` |
| Step 4 | Final Modal Dialog Confirm | `[role="dialog"] button:has-text("Publikasikan"), .modal button:has-text("Publikasikan")` |
