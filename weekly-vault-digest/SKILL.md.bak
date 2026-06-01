---
name: weekly-vault-digest
description: Weekly digest of active projects, recent sessions, and decisions into the Obsidian AI vault.
---

Generate Diego's weekly memory digest from his Obsidian "AI" vault.

VAULT ACCESS
- Vault root: C:\Users\dzvio\Obsidian\AI\AI (a standard Obsidian vault of markdown files).
- Preferred: use your file tools (Read/Write/Edit/Glob) directly on that path if the folder is accessible.
- Fallback: if you don't have file access, use the mcp-obsidian MCP tools (list_files_in_vault, list_files_in_dir, get_file_contents, search, append_content). This requires Obsidian to be running with the AI vault open.
- If you can reach the vault by neither method, stop and report "weekly digest skipped — vault not accessible (open Obsidian with the AI vault, or mount the folder)." Do not fabricate content.

WHAT TO GATHER
1. Active projects: read every note in Projects/. Select those with frontmatter `status: active`. For each capture: title, health (🟢/🟡/🔴 from its Status block), stage, and the top 1–2 unchecked items under "## Next actions".
2. Recent sessions: read notes in Memory/Sessions/ with `created` within the last 7 days; capture a one-line takeaway each.
3. Recent decisions: read Memory/Decisions.md; capture entries dated within the last 7 days.
4. Open actions: roll up ALL unchecked `- [ ]` items across active projects into one consolidated checklist, grouped by project.
5. Watchlist: flag any active project that is health 🟡/🔴, or whose `updated` date is more than 14 days ago (stale).

OUTPUT
- Create a new note at  Memory/Digests/<TODAY> — Weekly Digest.md  where <TODAY> is today's date as YYYY-MM-DD. Follow the format of Templates/Weekly-Digest.md. Fill the real date into the frontmatter and title. Keep it tight and scannable — a Monday status snapshot for a Procurement Director.
- Then update _Home.md: under the "## Recent digests" section, prepend a bullet linking the new digest (e.g. "- [[<TODAY> — Weekly Digest]]"); keep only the 5 most recent bullets.

RULES
- Follow the conventions in _Claude-Memory-Protocol.md. Append/create only — never delete anything.
- Never write any API key or anything from the Security folder into a note.
- If there are no active projects or recent activity, still create the digest and note "No active projects / no recent activity this week."