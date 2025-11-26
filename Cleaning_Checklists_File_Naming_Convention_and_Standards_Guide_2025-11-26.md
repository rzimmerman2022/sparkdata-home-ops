# Cleaning Checklists File Naming Convention and Standards Guide

**Document Version:** 1.0
**Created Date:** 2025-11-26
**Last Modified:** 2025-11-26
**Author:** Ryan
**Classification:** Internal Documentation

---

## 1. Purpose and Scope

This document establishes the official naming conventions and organizational standards for all cleaning checklist files within the `Cleaning Checklists` directory. These standards ensure consistency, readability, and ease of identification for both human users and AI-assisted systems.

---

## 2. Applicable Standards

This naming convention adheres to the following standards and best practices:

- **ISO 8601** - Date and time format representation (YYYY-MM-DD)
- **ISO 9001:2015** - Quality management principles for document control
- **General Records Management Best Practices** - Clear, descriptive, human-readable naming

---

## 3. File Naming Convention Structure

### 3.1 Standard Format

All cleaning checklist files shall follow this naming pattern:

```
[Cleaning_Type]_[Location_or_Zone]_[Assigned_Role]_[Frequency]_[YYYY-MM-DD].[extension]
```

**Important:** File names use underscores (`_`) for word separation. Folder names use spaces.

### 3.2 Component Definitions

| Component | Description | Examples |
|-----------|-------------|----------|
| **Cleaning Type** | The category or depth of cleaning | `Deep_Clean`, `Weekly_Clean`, `Monthly_Clean`, `Move_Out_Clean`, `Seasonal_Clean` |
| **Location or Zone** | The area(s) covered by the checklist | `Master_Bedroom`, `Kitchen`, `Full_Home`, `Office`, `Living_Areas` |
| **Assigned Role** | The person or team assignment | `Person_1`, `Person_2`, `Team_A`, `All_Members` |
| **Frequency** | How often this checklist is used | `One_Time`, `Weekly`, `Biweekly`, `Monthly`, `Quarterly`, `Annual` |
| **Date** | ISO 8601 formatted date of creation or scheduled clean | `2025-11-26` |
| **Extension** | File type | `.html`, `.md`, `.pdf` |

### 3.3 Naming Examples

| File Name | Description |
|-----------|-------------|
| `Deep_Clean_Master_Bedroom_and_Kitchen_Person_1_One_Time_2025-10-06.html` | Deep cleaning checklist for master bedroom and kitchen, assigned to person 1, one-time clean scheduled for October 6, 2025 |
| `Deep_Clean_Office_and_Living_Areas_Person_2_One_Time_2025-10-06.html` | Deep cleaning checklist for office and living areas, assigned to person 2, one-time clean scheduled for October 6, 2025 |
| `Weekly_Clean_Full_Home_All_Members_Recurring_2025-11-26.html` | Weekly cleaning checklist covering full home for all household members |
| `Monthly_Clean_Kitchen_Deep_Scrub_Person_1_Monthly_2025-12-01.html` | Monthly deep kitchen scrub checklist |
| `Move_Out_Clean_Full_Home_All_Members_One_Time_2025-12-15.html` | Complete move-out cleaning checklist |

---

## 4. Title Case Capitalization Rules

All file names shall use **Title Case** capitalization with the following rules:

### 4.1 Capitalize

- First and last words
- All nouns, verbs, adjectives, adverbs, and pronouns
- Words with four or more letters

### 4.2 Do Not Capitalize

- Articles (a, an, the) unless first or last word
- Short prepositions (in, on, at, for, to) unless first or last word
- Coordinating conjunctions (and, but, or, nor) unless first or last word

### 4.3 Spacing and Special Characters

#### File Names

| Rule | Correct | Incorrect |
|------|---------|-----------|
| Use underscores between words | `Deep_Clean_Kitchen` | `Deep-Clean-Kitchen`, `Deep Clean Kitchen` |
| Underscores for multi-word components | `Person_1` | `Person 1` |
| No hyphens for word separation | `Move_Out_Clean` | `Move-Out-Clean` |
| Numbers without leading zeros for roles | `Person_1` | `Person_01` |
| ISO date format with hyphens (only exception) | `2025-11-26` | `2025_11_26`, `11-26-2025` |

#### Folder Names

| Rule | Correct | Incorrect |
|------|---------|-----------|
| Use spaces between words | `Cleaning Checklists` | `Cleaning_Checklists`, `Cleaning-Checklists` |
| Title Case with spaces | `Deep Clean Archives` | `Deep_Clean_Archives` |

---

## 5. Directory Organization Structure

```
c:\projects\Cleaning Checklists\
│
├── [Naming Convention Documentation]
│   └── Cleaning_Checklists_File_Naming_Convention_and_Standards_Guide_2025-11-26.md
│
├── [Deep Clean Checklists]
│   ├── Deep_Clean_Master_Bedroom_and_Kitchen_Person_1_One_Time_2025-10-06.html
│   └── Deep_Clean_Office_and_Living_Areas_Person_2_One_Time_2025-10-06.html
│
├── [Weekly Checklists]
│   └── (future files)
│
├── [Monthly Checklists]
│   └── (future files)
│
└── [Special Event Checklists]
    └── (future files)
```

**Note:** Subdirectories are optional. Files may remain in the root directory if the collection is small.

---

## 6. Version Control

### 6.1 Document Revisions

When a checklist is significantly revised, create a new file with the updated date rather than overwriting the original. This preserves historical records.

### 6.2 Version Notation (Optional)

For documents requiring multiple versions on the same date, append a version indicator:

```
Deep_Clean_Full_Home_All_Members_One_Time_2025-11-26_v2.html
```

---

## 7. File Format Standards

### 7.1 Preferred Formats

| Format | Use Case |
|--------|----------|
| `.html` | Interactive checklists with progress tracking, checkboxes, and visual styling |
| `.md` | Documentation, guides, and reference materials |
| `.pdf` | Printable versions for offline use |

### 7.2 Encoding

- All text files shall use **UTF-8** encoding
- HTML files shall include `<meta charset="UTF-8">` declaration

---

## 8. Metadata Requirements

Each checklist file should contain the following metadata (in file header or comments):

1. **Title** - Full descriptive title
2. **Date Created** - ISO 8601 format
3. **Date of Scheduled Clean** - When applicable
4. **Assigned To** - Person or team designation
5. **Zones Covered** - List of areas included
6. **Estimated Duration** - Approximate time to complete

---

## 9. Quick Reference Card

```
NAMING FORMULA (FILES):
[Type]_[Location]_[Role]_[Frequency]_[YYYY-MM-DD].[ext]

EXAMPLES:
Deep_Clean_Kitchen_Person_1_One_Time_2025-10-06.html
Weekly_Clean_Full_Home_All_Members_Recurring_2025-11-26.html
Monthly_Clean_Bathrooms_Person_2_Monthly_2025-12-01.html

RULES:
✓ Use Title Case
✓ Use underscores (_) for file names
✓ Use spaces for folder names
✓ Use ISO 8601 dates (YYYY-MM-DD) - hyphens only in dates
✓ Be descriptive and verbose
✓ Include assigned person/role
✓ Include cleaning frequency
```

---

## 10. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-26 | Ryan | Initial document creation |

---

**End of Document**
