# Adding a Subject / Class to CUBE — Runbook

The goal of the registry work is that subject metadata lives in **one file**
(`subjects.json`) instead of being scattered. This runbook lists every step to
bring a new subject or class online, and flags which steps are now automatic.

Legend:  ✅ = now driven by the registry (do it once)   ✋ = still manual (for now)

---

## 1. Google Sheet ✋
Create the two tabs for the class, named exactly as the `code` you will use:

- `<Code>`      — short-answer log (e.g. `12Mus-1`)
- `<Code>_MC`   — multiple-choice log (e.g. `12Mus-1_MC`)

Paste the correct **header row** into Row 1:
- Standard science-style layout → copy Row 1 from `12VA-1` (the current
  new-order header). The webhook is header-aware, so the tab may use its own
  header order — but every column you want populated must have a header.
- A genuinely different subject (e.g. Music with composition columns) may have
  its own header row; that's fine — just make sure each field name matches what
  the marking app sends (see `LogWebhook.gs` field list).

## 2. `subjects.json` ✅ (single source of truth)
Add one row:

```json
{ "code":"12Mus-1", "label":"12Mus-1", "full":"Music — 12Mus-1",
  "emoji":"🎵", "kla":"CAPA", "group":"Music",
  "color":"#b39ddb", "cls":"c-music", "hasMC":true }
```

- `code`   = the Sheet tab name and the dashboard's switch key
- `kla`    = one of `Science | HSIE | PDHPE | TAS | CAPA`
- `group`  = sub-heading within that KLA (e.g. `"Music"`), or `null` for a
             standalone subject
- `color`  = accent hex   ·   `cls` = a panel colour class (add matching
             `.c-music` CSS in `Analysis.html` if it's a brand-new colour)
- `hasMC`  = does a student MC bank exist yet? (`false` for VA/ANCH today)

Once this is hosted, the **dashboard rebuilds itself** from it:
- ✅ tabs loaded (`SHEET_NAMES`, `MC_SHEET_NAMES`)
- ✅ MC tab mapping (`MC_TAB_FOR`)
- ✅ KLA colours/labels (`KLA`)
- ✅ the subject sidebar button, sub-group heading, and count
- ✅ selection/active-state behaviour

There are **no other lists to edit in `Analysis.html`.** (If the subject is a new
KLA column that doesn't exist yet, add one row to `KLA_LAYOUT` in the dashboard —
that's the only presentation piece not in the registry.)

## 3. `config-<Subject>.json` ✋
Create the marking config (copy the closest existing subject and edit:
`subjectCode`, `subjectName`, `classCodes`, `sheetTabFull`, `sheetTabCube`,
`syllabus`, `directiveTiers`, `bandRules`, …). For band-marked / multi-page
practical subjects also set `markingMode` and `responsePagesMax` (as in
`config-VisArt.json`). Host it next to the other configs.

## 4. Cloudflare KV — MC questions ✋ (only if `hasMC:true`)
Create the namespace `CUBE_QUESTIONS_<CODE>` and load the vetted question set
(key: `questions`). Add the code to the worker's KV binding map.

## 5. MC coach `SUBJECT_CONFIG` ✋ *(until wired to the registry — see note)*
Add the subject to `SUBJECT_CONFIG` in `mc-sa-coach.html` (year pills, module
pills, `syllabusFrom`, `mcYears`, `hasMC`). Apply to the DEPLOY copy too.

## 6. Webhook routing ✋ *(until wired to the registry — see note)*
In `LogWebhook.gs`, add the class to `TAB_MAP` and `MC_TAB_MAP`. Multi-class
subjects also need their code recognised in the `FULL_ROW` class-resolution
(currently a per-subject `if` ladder — candidate for the next consolidation).

## 7. Deploy ✋
Drag-and-drop the changed files to GitHub Pages; redeploy the Apps Script and
(if the KV map changed) the Cloudflare Worker.

## 8. Verify ✋
- Sidebar shows the new button under the right KLA, with a live count.
- Log one short-answer mark → row lands under the correct headers in the tab.
- Dashboard reads it (Submission Log, cards, band/mark as appropriate).
- If `hasMC`, run one MC question end-to-end.

---

### Note on steps 5 & 6
`subjects.json` is now the shared source of truth, but only the **dashboard**
reads it so far. The next consolidation step is to have the **MC coach** and the
**webhook** read `subjects.json` too (the webhook via a cached `UrlFetchApp`
call with a hardcoded fallback), which would fold steps 5 and 6 into step 2. Do
that before the subject count grows much further.
