# Joplin Desktop Pre-Release Regression Checklist

* **Target Application:** Joplin Desktop (Open-Source Note-Taking & To-Do App)
* **Changelog Source:** https://joplinapp.org/help/about/changelog/desktop/
* **Releases Reviewed (Last 5 Official Releases):**
  * `v3.7.22` (2026-10-28) — https://joplinapp.org/help/about/changelog/desktop/#v3722-2026-10-28t223259z
  * `v3.7.21` (2026-09-25) — https://joplinapp.org/help/about/changelog/desktop/#v3721-2026-09-25t213112z
  * `v3.7.18` (2026-09-11) — https://joplinapp.org/help/about/changelog/desktop/#v3718-2026-09-11t174756z
  * `v3.7.16` (2026-09-06) — https://joplinapp.org/help/about/changelog/desktop/#v3716-2026-09-06t091756z
  * `v3.6.15` (2026-06-20) — https://joplinapp.org/help/about/changelog/desktop/#v3615-2026-06-20t110411z

---

## Top 5 Checks to Automate First (Priority Automation Candidates)

1. **`REG-01` (Sync Deadlock on HTTP 422):** Automatable via a mock HTTP sync server returning `422` so every build verifies the sync engine never deadlocks or hangs indefinitely.
2. **`REG-02` (WebDAV Sync HTTP 409 Conflict Recovery):** High-frequency user workflow where a simulated `409` WebDAV response cleanly catches post-upgrade sync regressions in CI.
3. **`REG-05` (Table Editor Inline Markdown Preservation):** Deterministic editor input/output behavior (`**bold**`, `*italic*`, `` `code` ``) makes Playwright/Spectron UI automation fast and reliable.
4. **`REG-11` (ENEX Import with Unescaped Ampersands):** Fixture-based `.enex` file import runs headlessly in seconds and prevents silent data-loss regressions during Evernote migrations.
5. **`REG-14` (AI Chat Reply Reasoning-Block Leakage):** Mocked reasoning-model API responses deterministically verify that internal `<think>` / reasoning blocks never leak into user-facing replies.

---

## 1. Synchronization & External Storage

| ID | Automate First? | What to Do (Test Steps) | What Correct Looks Like (Expected Result) | Past Fix Protected (Release & Issue Link) |
| :--- | :--- | :--- | :--- | :--- |
| **REG-01** | **Yes (#1)** | Configure sync against a test Joplin Server endpoint and trigger synchronization when the server rejects an item with an `HTTP 422` response. | Sync logs the item-level rejection error, skips or flags the rejected item, and completes the remaining sync queue without freezing or entering a permanent deadlock. | **v3.7.21** — Fix sync deadlock when Joplin Server rejects an item with HTTP 422 ([PR #16544](https://github.com/laurent22/joplin/pull/16544) / [Issue #16504](https://github.com/laurent22/joplin/issues/16504)) |
| **REG-02** | **Yes (#2)** | Upgrade an existing profile to the candidate build and run full synchronization against a WebDAV sync target (e.g., Nextcloud or Apache/Tomcat WebDAV). | Sync completes without throwing `HTTP 409 Conflict` errors on existing remote resources or lock files. | **v3.7.21** — Fix WebDAV sync failing with 409 errors after 3.7 upgrade ([PR #16471](https://github.com/laurent22/joplin/pull/16471) / [Issue #16422](https://github.com/laurent22/joplin/issues/16422)) |
| **REG-03** | No | Open a note in an external editor, save changes in Joplin first so Joplin holds newer content, and then trigger an external-editor file reload or stale save event. | Joplin detects the timestamp/revision difference and refuses to silently overwrite newer note content with older external file content. | **v3.7.16** — External editor: Add additional logic to avoid overwriting a note with older content ([PR #16352](https://github.com/laurent22/joplin/pull/16352) / [Issue #16350](https://github.com/laurent22/joplin/issues/16350)) |
| **REG-04** | No | Trigger an `x-callback-url` deep link into Joplin Desktop from an external launcher/script expecting a return callback URL response. | Joplin executes the requested action and dispatches the `x-success` / callback response back to the calling application instead of dropping it. | **v3.7.21** — Fix x-callback-url responses never reaching the calling app ([PR #16497](https://github.com/laurent22/joplin/pull/16497)) |

---

## 2. Note Editor, Rendering & Rich Text

| ID | Automate First? | What to Do (Test Steps) | What Correct Looks Like (Expected Result) | Past Fix Protected (Release & Issue Link) |
| :--- | :--- | :--- | :--- | :--- |
| **REG-05** | **Yes (#3)** | Create a Markdown table containing `**bold**`, `*italic*`, `` `inline code` ``, and `[links](https://example.com)` inside cells, open it in the Rich Text / Table Editor, edit a cell, and switch back to Markdown. | All inline Markdown formatting inside every cell is preserved and rendered accurately without stripping syntax or corrupting cell alignment. | **v3.7.21** — Fix table editor losing and mis-rendering inline markdown ([PR #16632](https://github.com/laurent22/joplin/pull/16632)) |
| **REG-06** | No | Create a note using Fountain screenplay syntax (scene headings, character cues, dialogue, parentheticals) and inspect the rendered preview pane. | Fountain elements render with proper screenplay formatting and indentation without raw syntax artifacts or broken layout blocks. | **v3.6.15** — Fix Fountain rendering issue ([PR #15659](https://github.com/laurent22/joplin/pull/15659)) |
| **REG-07** | No | Create and apply tags containing special Unicode characters (accented characters, non-Latin scripts, emoji, or composite diacritics) across multiple notes. | Joplin reuses the existing matching tag rather than creating duplicate tags with identical visible names. | **v3.6.15** — Fix duplicate tag creation for special unicode characters ([PR #15613](https://github.com/laurent22/joplin/pull/15613) / [Issue #15608](https://github.com/laurent22/joplin/issues/15608)) |
| **REG-08** | No | Switch the application UI language to Japanese (`日本語`), open a note, and inspect the tag bar below the note title. | The tag area displays cleanly with properly aligned labels, input field, and tag pills without overlapping or clipped UI controls. | **v3.7.21** — Tag area broken with Japanese language ([PR #16483](https://github.com/laurent22/joplin/pull/16483) / [Issue #16427](https://github.com/laurent22/joplin/issues/16427)) |

---

## 3. Whiteboard & Canvas Interactions

| ID | Automate First? | What to Do (Test Steps) | What Correct Looks Like (Expected Result) | Past Fix Protected (Release & Issue Link) |
| :--- | :--- | :--- | :--- | :--- |
| **REG-09** | No | Open a Whiteboard canvas, right-click on the canvas or open the Whiteboard menu, and select **Add note to whiteboard**. | The **Add note to whiteboard** menu item is visible, enabled, and opens the note picker to place an existing note onto the canvas. | **v3.7.21** — Whiteboard: Add missing "Add note to whiteboard" menu item ([PR #16468](https://github.com/laurent22/joplin/pull/16468) / [Issue #16457](https://github.com/laurent22/joplin/issues/16457)) |
| **REG-10** | No | Open a Whiteboard canvas and double-click on an empty region of the canvas background. | A new note card is created at the double-clicked coordinates and enters edit mode immediately. | **v3.7.21** — Whiteboard: Double-click empty canvas to add a note ([PR #16470](https://github.com/laurent22/joplin/pull/16470)) |

---

## 4. Data Import & Migration

| ID | Automate First? | What to Do (Test Steps) | What Correct Looks Like (Expected Result) | Past Fix Protected (Release & Issue Link) |
| :--- | :--- | :--- | :--- | :--- |
| **REG-11** | **Yes (#4)** | Import an Evernote `.enex` export file containing raw unescaped `&` characters inside note bodies, titles, or source URLs. | The `.enex` importer parses the XML cleanly without throwing an entity parsing failure and imports all notes intact. | **v3.7.21** — Fix ENEX import failing on Evernote's unescaped ampersands ([PR #16580](https://github.com/laurent22/joplin/pull/16580)) |
| **REG-12** | No | Import a notebook/section exported from Microsoft OneNote and open the resulting imported notes in Joplin's viewer and editor. | Imported text renders at normal, readable body font sizes rather than microscopic/shrunken inline font styles. | **v3.7.18** — Fix notes imported from OneNote are displayed with a very small font size ([PR #16440](https://github.com/laurent22/joplin/pull/16440) / [Issue #16424](https://github.com/laurent22/joplin/issues/16424)) |

---

## 5. AI Assistant, MCP Server & Local Models

| ID | Automate First? | What to Do (Test Steps) | What Correct Looks Like (Expected Result) | Past Fix Protected (Release & Issue Link) |
| :--- | :--- | :--- | :--- | :--- |
| **REG-13** | No | Configure an AI provider using a reasoning-capable model (e.g., `o1`/`o3` or a local reasoning model) and click **Check AI configuration**. | The configuration test succeeds without failing on unsupported temperature/parameter errors specific to reasoning models. | **v3.7.21** — Fix AI configuration test failing on reasoning models ([PR #16594](https://github.com/laurent22/joplin/pull/16594)) |
| **REG-14** | **Yes (#5)** | Send a prompt in the AI Chat panel backed by a reasoning model that emits internal thought/reasoning tokens before its final answer. | Only the final assistant response is displayed in the chat bubble; raw reasoning blocks do not leak into the visible reply. | **v3.7.21** — Fix reasoning blocks leaking into AI chat replies ([PR #16595](https://github.com/laurent22/joplin/pull/16595)) |
| **REG-15** | No | Configure a local **LM Studio** or private-network LAN OpenAI-compatible endpoint with a blank API key and send a chat message. | Joplin recognizes the private network / LM Studio endpoint as local, allows an empty API key, and returns a valid completion. | **v3.7.21** — Fix LMStudio compatibility ([PR #16552](https://github.com/laurent22/joplin/pull/16552)) & Treat private network AI providers as local ([PR #16472](https://github.com/laurent22/joplin/pull/16472)) |
| **REG-16** | No | Press the configured **Toggle AI Chat** keyboard shortcut from the main editor window and send an MCP JSON-RPC notification request to the built-in MCP server. | The shortcut toggles the AI Chat panel open/closed, and the MCP server returns HTTP `202` for notification requests. | **v3.7.21** — Toggle AI Chat keystroke shortcut does nothing ([PR #16433](https://github.com/laurent22/joplin/pull/16433)) & Return 202 for MCP notification requests ([PR #16473](https://github.com/laurent22/joplin/pull/16473)) |

---

## 6. Packaging, Security & Desktop Shell

| ID | Automate First? | What to Do (Test Steps) | What Correct Looks Like (Expected Result) | Past Fix Protected (Release & Issue Link) |
| :--- | :--- | :--- | :--- | :--- |
| **REG-17** | No | Launch the Linux AppImage build on a desktop environment (GNOME/KDE) and inspect the window class and taskbar/dock grouping. | `StartupWMClass` matches the desktop entry (`@joplin/app-desktop`), grouping windows under the proper Joplin icon rather than a generic cog icon. | **v3.7.21** — Fix AppImage StartupWMClass ([PR #16547](https://github.com/laurent22/joplin/pull/16547) / [Issue #16434](https://github.com/laurent22/joplin/issues/16434)) |
| **REG-18** | No | Open **Help > About Joplin** (or inspect `process.versions.electron` in DevTools) and verify the bundled runtime version and upgrade banner behavior. | The Electron runtime is at `42.11.18` (or newer patched security release) and any version upgrade banner displays the accurate minimum required version. | **v3.7.22** — Security: Upgrade to Electron 42.11.18 ([PR #16964](https://github.com/laurent22/joplin/pull/16964)) & Upgrade app banner version fix ([PR #16819](https://github.com/laurent22/joplin/pull/16819)) |