[README.md](https://github.com/user-attachments/files/32100645/README.md)
# Tariff Street (20288) Block 02 – window remedials sign-off pipeline — for Claude

Job: 77 scratched-window defects raised by Enabl Consulting's Quality Manager report
(dated 4 Sep 2026) for Block 02 Apartments, Levels 2–9, being made good by WEJB.

## The pieces
- `tariff-street-windows.html` — the site form the lad uses on his phone. Hosted on the WEJB
  GitHub Pages site (https://adambrook-wejb.github.io/) next to the timesheet form. Level →
  apartment → items; photo required before an item can be ticked Done; "Can't complete" with a
  reason; finger signature; sends one Web3Forms email per apartment sign-off. Works offline and
  retries. State lives on the phone (localStorage + IndexedDB) until the office regenerates the
  page with completed items baked in.
- `form_template.html` + `build_form.py` — regenerate the form:
  `python3 build_form.py done_baseline.json sent_baseline.json` (both optional).
- `records.json` — the 77 items parsed from the report (ref, level, apt, type, room, defect,
  inspector photo count, raised date, status). Source of truth for the schedule.
- `windows_office.py` — parser for the `WEJB-WINDOWS-V1` blocks, Excel updater, client
  completion report (HTML → PDF), form regeneration. `python3 windows_office.py --help`.
- `extract_before.py` — pulls the inspector's photos out of the original `windows.pdf` into
  `before/` and `before_photos.json` so the client report can show as-reported vs completed.
  Needs the original PDF attached to the chat (230 photos, all mapped, verified 11.09.26).
- `plans/plan_config.json` + `plans/build_plans.py` — floor plans. One entry per level: the
  architect's PDF (Leach Rhodes Walker 8198-LRW-B2-0X-DR-A-20-15X "Block 2 – Clements"), the crop
  box in PDF points, the apartment label positions and one box per room that has items
  (`[apt, room name exactly as in records.json, centre_x_pt, centre_y_pt, width_pt, height_pt]`).
  Get the label coordinates from the PDF text layer with pdfplumber (`extract_words`), assign
  rooms to apartments by eye (nearest-label is NOT reliable), then `python3 build_plans.py --check`
  and look at `check_L0X.png` before building the form. Output `plans/plans.json` is what
  `build_form.py` embeds (WebP, ~200 KB a level). All eight levels (02–09) traced 11.09.26:
  Levels 2–4 share one layout (6 flats), Levels 5–9 another (5 flats).
- The form also embeds the inspector's photos for every item (from `before/`) so the lad can see
  which window is meant, and a per-level plan view (tap a room → its items; "Show on plan" on an
  item → the room highlighted).
- `Tariff_Street_Block02_Window_Remedials_Schedule.xlsx` — Adam's pricing schedule (Schedule /
  Rates / Summary tabs). The office run adds columns S–W (Site Status, Completed On, Signed By,
  Work Carried Out / Reason, After Photos) and a Progress tab, leaving rates/qty/notes untouched.

## Where the sign-offs land
Web3Forms email from `notify@web3forms.com` to the access key's mailbox
(timesheets@westendjoiners.co.uk if the fire door key is reused) with copies to
leigh@westendjoiners.co.uk and adam@westendjoiners.co.uk. Subject:
`Window sign-off — Tariff Street B2 — APT-B2-02-01 — 2 done, 1 not done`.
The email's **Message** field holds the machine block; photos and the signature PNG are
Web3Forms attachments (download links `https://api.web3forms.com/download?file=...`, named
`<ref>-<n>-<timestamp>.jpg` and `signature-<APT>-<timestamp>.png`, matching the `P|` / `S|` lines).

```
WEJB-WINDOWS-V1
SITE: 20288 Tariff Street - Block 02 Apartments
APT: APT-B2-02-01
LEVEL: 02
TYPE: 1B1P
FITTER: Leigh
SIGNED: 2026-09-11T09:37:20.974Z        (UTC)
SIGNED_UK: 11/09/2026 10:37 BST
FORM: 2026-09-11
---
D|2338|Kitchen, Dining & Living Room|Scratches polished out|2      done: ref|room|what was done|photos
X|2323|Bedroom 01|No access - tenant out|0                         not completed: ref|room|reason|photos
P|2338|2338-1-1789119440.jpg                                       one line per photo
S|signature-APT-B2-02-01-1789119440.png
N|Tenant was in, all fine                                          optional note
END
```
A second sign-off for the same apartment only contains the items that were new since the last
send (e.g. the one that couldn't be done first time). Latest line per ref wins.

## Office procedure
1. **Scan** `timesheets@westendjoiners.co.uk` (outlook_email_search, mailboxOwnerEmail set,
   sender `notify@web3forms.com`) for subjects starting `Window sign-off —` since the last run.
   Ignore Timesheet / Vehicle check / Fire door emails.
2. **Transcribe** each email's `WEJB-WINDOWS-V1 … END` block verbatim into
   `submissions/<nn> <APT>.txt`. Keep every block (they accumulate — the run merges them).
3. **Download** the photos and signature from the email's attachment links into `submissions/`
   with exactly the filenames in the `P|`/`S|` lines. `api.web3forms.com` must be on the
   organisation's Claude network allowlist for this to work from the cloud workspace; if a
   download is blocked, say so — the run still works, the report just shows "No photographs".
4. **Check** with `python3 windows_office.py parse --subs submissions` — every block prints its
   apt, fitter, time and counts; anything under `!!` needs looking at before going further.
5. **Run** `python3 windows_office.py run --subs submissions --xlsx <latest schedule xlsx>
   --out out --client "<client name>"` (add `--attended-only` for an interim report). Produces:
   updated Excel (recalculate with the xlsx skill's recalc.py), completion report PDF, and a
   regenerated `tariff-street-windows.html` with the completed refs baked in.
6. **Deliver** in chat to whoever asked. The regenerated form goes back onto GitHub Pages
   (same filename) so every phone shows the office's view; the Excel/PDF are dragged into
   SharePoint by the requester.

## Hard rules
- Nothing is guessed. A block that doesn't parse is reported, not patched.
- Never email the lad. Summaries only to Adam/Leigh, and only when asked to send.
- Never change rates/qty/notes in the schedule — only columns S–W and the Progress tab.
- The `--- ` separator, `|` field order and `END` are load-bearing: keep the form and
  `windows_office.py` in step if either changes (bump `WEJB-WINDOWS-V1` if the format changes).
