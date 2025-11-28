# Knowledge Base - Cleaning Checklists Project

**Last Updated:** 2025-11-28
**Maintained By:** AI Sessions (see handoffs/)

---

## Project Overview

Interactive HTML-based cleaning checklists for household deep cleaning. Features progress tracking, localStorage persistence, and team-based task assignment.

---

## Architecture

### Tech Stack
- **Frontend:** Pure HTML5, CSS3, JavaScript (no frameworks)
- **Persistence:** Browser localStorage
- **Styling:** CSS gradients, flexbox, responsive design
- **Integration:** Google Sheets (for session logging)

### Key Patterns

#### Checkbox State Persistence
```javascript
// Save pattern
localStorage.setItem('team-member-1-' + checkbox.id, checkbox.checked);

// Load pattern
const saved = localStorage.getItem('team-member-1-' + checkbox.id);
if (saved === 'true') { checkbox.checked = true; }
```

#### Progress Bar Calculation
```javascript
const progress = (checkedBoxes.length / checkboxes.length) * 100;
document.getElementById('progressBar').style.width = progress + '%';
```

---

## File Naming Convention

### Current Format (as of 2025-11-28)
```
[Cleaning_Type]_[YYYY-MM-DD]_[Location]_[Role]_[Frequency].html
```

### Examples
- `Deep_Clean_2025-10-06_Master_Bedroom_and_Kitchen_Person_1_One_Time.html`
- `Deep_Clean_2025-11-26_Full_House_Single_Cleaner_Prioritized.html`

### Note
The naming convention guide may show a different format (date at end). Files have been updated to place date after cleaning type.

---

## Team Assignment

### Person 1 (Purple Theme)
- Master Bedroom
- Master Bathroom
- Kitchen
- Shared: Patio, entryway, washer/dryer

### Person 2 (Pink/Cyan Theme)
- Second Bedroom / Office
- Second Bathroom
- Living Room
- Dining Room
- Shared: Hallway, front door, light bulbs

---

## Cleaning Phases

All deep clean checklists follow this structure:

1. **Phase 1: Declutter & Organize**
   - Remove items from surfaces
   - Organize closets/cabinets
   - Collect laundry
   - Clear countertops

2. **Phase 2: Deep Clean (Top to Bottom)**
   - High areas (ceiling fans, vents, corners)
   - Walls & windows
   - Mid-level surfaces
   - Lower areas & floors

---

## Gotchas & Lessons Learned

| Issue | Solution |
|-------|----------|
| Windows bash doesn't have `ren` | Use `mv` command instead |
| Glob may miss files | Use `ls -la` for complete listing |
| localStorage keys are ID-based | Don't change checkbox IDs without migration |

---

## External Integrations

### Google Sheets
- Session logging capability exists (commit 2d0a1c1)
- Credentials stored in `.credentials/` (gitignored)
- Exact functionality: TBD (needs investigation)

---

## Session History

| Session ID | Date | Model | Summary |
|------------|------|-------|---------|
| ses001 | 2025-11-28 | opus-4.5 | Initial handoff infrastructure, file renames |

---

*This knowledge base is progressively updated by each AI session.*
