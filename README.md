# ISO 2025 Performance Summary (OPAS-Derived)

## Overview

This repository documents the process used to generate year-end summary statistics for the Indianapolis Symphony Orchestra’s 2025 calendar year, created in support of an end-of-year “ISO Wrapped” marketing initiative.

The objectives were to quantify:

- How frequently each classical composer was performed  
- How many distinct works by each composer were programmed  
- The total number of concerts presented  
- The number of guest soloists appearing during the year (excluding Yuletide)

The reporting period covers **January 1–December 31, 2025**, corresponding to:
- The spring portion of the **2024–25 season**
- The fall portion of the **2025–26 season**

All data was sourced from **OPAS**, with additional spreadsheet-based processing where OPAS reporting alone was insufficient.

---

## Background & Request

Marketing asked whether it was possible to pull, from OPAS or related tools:

- Composer performance counts for the calendar year  
- Guest artist totals (excluding Yuletide)  
- A total concert count  

Because OPAS does not provide a single turnkey report that combines all of these metrics cleanly, the final results were produced using a combination of:

- SQL queries against OPAS tables  
- Manual data cleanup  
- Excel and Power Query transformations  

---

## Composer Performance Counts

### SQL Query Used

The following query was used to calculate the number of concerts featuring music by each composer and the number of distinct works performed by each composer.

    SELECT
        RTRIM(sComposers.LastName) + ', ' + RTRIM(sComposers.FirstName) AS Composer,
        COUNT(aDates.ID) AS Date_Count,
        COUNT(DISTINCT sWorks.ID) AS Distinct_Works_Performed
    FROM aDates
    INNER JOIN aDate_Works ON aDates.ID = aDate_Works.Date_ID
    INNER JOIN sWorks ON aDate_Works.Work_ID = sWorks.ID
    INNER JOIN sComposers ON sWorks.Composer_ID = sComposers.ID
    WHERE
        aDates.Date_ BETWEEN '1/1/2025' AND '12/31/2025'
        AND aDates.EventType_ID = 1
        AND aDates.Duties > 0
    GROUP BY sComposers.LastName, sComposers.FirstName
    ORDER BY sComposers.LastName;

---

## Spreadsheet Processing (Composer Data)

Query results were pasted into Excel (**Tab 2** of the attached spreadsheet). The composer list was manually cleaned to remove arrangers and “traditional” or non-specific composer listings. All counts reflect **concert appearances**, not rehearsals or services.

---

## John Williams Work Consolidation

Initial results showed a disproportionately high number of distinct works for **John Williams**. Further investigation revealed that OPAS treats multiple excerpts from the same film as separate works.

For marketing purposes, each **film** was treated as a single work.

### Additional Processing Steps

The query was expanded to include the work title field. All John Williams entries were isolated and copied into **Tab 3** of the spreadsheet, where film titles were manually consolidated so each film was counted only once.

This adjustment better aligns the data with audience-facing interpretations of the repertoire.

---

## Guest Soloist Count

OPAS does not provide a clean, single report of distinct guest soloists for a calendar year.

### Process Used

1. An existing OPAS report for 2025 was exported to **Microsoft Word**, containing soloist names, instruments, and additional unused metadata.  
2. The data was pasted into Excel and unnecessary columns were removed.  
3. **Power Query** was used to normalize name formatting and deduplicate entries.  

**Result:** 52 distinct guest soloists appeared during the 2025 calendar year.

This data appears in **Tab 1** of the spreadsheet.

---

## Total Concert Count

The following query was used to calculate the total number of qualifying concerts presented in 2025.

    SELECT
        COUNT(*) AS Total_Concerts_2025
    FROM aDates
    WHERE
        Date_ >= '2025-01-01'
        AND Date_ <  '2026-01-01'
        AND EventType_ID = 1
        AND Duties > 0;

This includes all qualifying classical and pops concerts.

---

## Files Included

- **2025 Performances.xlsx**
  - Tab 1: Guest soloist list (cleaned via Power Query)
  - Tab 2: Composer performance counts
  - Tab 3: John Williams work-level cleanup

---

## Notes & Caveats

- All counts depend on OPAS data integrity and classification.
- Some manual cleanup was required where OPAS granularity exceeded marketing needs.
- Counts represent **concert appearances**, not unique programs or total repertoire minutes.
- This workflow prioritizes clarity and communicability over strict musicological precision.

---

## Intended Use

This data is intended for end-of-year marketing content, public-facing “ISO Wrapped” summaries, and internal reference and benchmarking. It is **not** intended as an archival or scholarly catalog of repertoire.
