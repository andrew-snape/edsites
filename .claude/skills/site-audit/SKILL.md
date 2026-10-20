---
name: site-audit
description: Leader workflow that audits the Edu Links site - dispatches url-checker, ui-checker and resource-finder subagents in parallel, then drafts the resulting link.yml edits for review. Use when the user asks to audit the site, check for dead links, check the UI, or find new resources.
---

You (running as the leader, Sonnet) orchestrate three Haiku subagents —
`url-checker`, `ui-checker`, `resource-finder` — to audit this repo's
`_data/links.yml` and site, then draft the resulting edits for the user to
review. Never commit; always leave changes staged only in the working tree
for the user to look over with `git diff`.

## 1. Load the data

Read `_data/links.yml` and `_config.yml`'s `subjects` list. Work subject by
subject (maths, literacy, class_utilities, research, pyp,
christian_studies, or whatever subjects are currently defined — don't
hardcode this list, re-derive it from `_config.yml`).

## 2. Phase 1 — URL validity (parallel, foreground)

For each subject, write its flattened entries to a scratchpad file, one
line per link: `<sub_area>|<name>|<url>`. Then spawn one `url-checker`
subagent per subject, all in the same message (parallel, `run_in_background:
false` — you need the results before drafting edits), prompting it with the
scratchpad file path.

Collect each subagent's `DEAD` / `SUSPECT` lists.

## 3. Phase 2 — UI check (foreground)

Spawn one `ui-checker` subagent to exercise the built site (nav, cards,
favicon fallback, search, console errors). Collect its findings.

## 4. Phase 3 — New resources (parallel, foreground)

For each subject, spawn one `resource-finder` subagent in parallel, giving
it that subject's sub-areas and current entries (names + URLs) so it can
avoid duplicates. Collect each subagent's proposed additions.

## 5. Report

Give the user a concise summary: counts of dead/suspect links per subject
(with the reason for each dead one), UI issues found (or "UI clean" if
none), and how many new resources are proposed per subject.

## 6. Draft the edits

In `_data/links.yml`:
- Remove entries confirmed **dead**.
- Leave **suspect** entries alone — list them in your report instead, since
  bot-blocking produces false positives; let the user decide.
- Add the proposed new resources under their matching `sub_area`, matching
  existing YAML formatting exactly (key order `name`, `url`, `description`;
  same indentation).

Do not touch `search.json` — mention in your report that it's a generated
index and should be rebuilt (`bundle exec jekyll build`) if the user wants
it to reflect the new entries before deploying.

Finish by pointing the user at `git diff _data/links.yml` to review before
committing. Do not run git add/commit/push yourself.
