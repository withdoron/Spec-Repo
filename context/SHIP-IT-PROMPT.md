GOAL: Update all project documentation to reflect what shipped this session. This is the session-end SOP — run after every work session.

WHERE:
- context/SESSION-LOG.md
- context/ACTIVE-CONTEXT.md
- DECISIONS.md
- STATUS-TRACKER.md
- checklists/LAUNCH-CHECKLIST.md
- PROJECT-BRAIN.md (only if architecture or philosophy shifted — flag if so)
- NODE-LAB-MODEL.md (only if a node changed phase — flag if so)
- SEEDLING-TRACKER.md (only if a new seed was planted or matured — flag if so)

INSTRUCTIONS:

1. Read the session summary provided below.

2. Append a new entry to context/SESSION-LOG.md using this format:
---
### Session Log — [TODAY'S DATE]
**Focus:** [one-line description]
**Shipped:**
[numbered list of what shipped]
**Decisions made:**
[any DEC-XXX decisions, or "None"]
**Next up:**
[what's queued]
---

3. Overwrite context/ACTIVE-CONTEXT.md with updated status. Preserve the existing structure — only update the fields that changed:
- Current phase
- What's in flight
- What just shipped
- What's next
- Any active blockers

4. If any new DEC-XXX decisions were made, append them to DECISIONS.md using the existing format (Date, Context, Decision, Rationale, Status).

5. In STATUS-TRACKER.md:
- Add a row to the Session Log table for today
- Check off any LAUNCH-CHECKLIST items that landed
- Update any phase or node status that changed

6. In checklists/LAUNCH-CHECKLIST.md:
- Mark [x] on any items that shipped this session
- Do not uncheck anything

7. Check PROJECT-BRAIN.md, NODE-LAB-MODEL.md, and SEEDLING-TRACKER.md:
- If nothing changed that affects these docs, leave them untouched and output: "PROJECT-BRAIN: no changes needed / NODE-LAB-MODEL: no changes needed / SEEDLING-TRACKER: no changes needed"
- If something did change, update the relevant section and flag what you changed in the commit message

CONSTRAINTS:
- Do not rewrite docs from scratch — append and update only
- Do not change terminology, naming conventions, or doc structure
- Preserve all existing entries in SESSION-LOG and DECISIONS — append only
- If uncertain whether something belongs in a doc, add it and note the uncertainty in a comment
- Git commit message: "docs: session-end update [DATE] — [one-line summary of what shipped]"

---
SESSION SUMMARY:
[Mycelia pastes the session summary here before handing to Doron]
