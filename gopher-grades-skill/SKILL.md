---
name: gopher-grades
description: Search and analyze University of Minnesota (UMN) courses, professors, and grade distributions. Use when helping students find courses, gauge course difficulty, research or compare professors, plan a semester, or explore liberal education requirements at UMN.
license: MIT
---

# Gopher Grades

Help UMN students make informed course and professor decisions using historical
grade distributions, Rate My Professor scores, and official student ratings, served
by the Gopher Grades MCP server.

The MCP tools below already document their own parameters and return shapes in their
schemas — you'll see those when you call them. This skill covers the part the
schemas don't: which tool to reach for, the non-obvious usage patterns, and how to
turn the raw numbers into a sound recommendation.

## Mental model

A few things to internalize before you start:

- **Average GPA is the difficulty proxy.** Higher GPA ≈ easier/more generously
  graded. Rough guide: ≥3.5 easier, 3.0–3.5 moderate, <3.0 harder. Always weight by
  sample size and look at the full distribution — see
  [references/data-interpretation.md](references/data-interpretation.md).
- **Professors take two calls.** There's no single tool from name → stats. Use
  `search_professors` to resolve a name to a `professor_id`, then
  `get_grades_of_a_professor` with that ID. Skipping the first step is the most
  common mistake.
- **`search_courses` sorts by enrollment, not GPA.** If the student wants "easy,"
  pass `min_gpa`/`max_gpa` to filter — don't just eyeball the first results.
- **Defaults:** campus is `UMNTC` (Twin Cities) and data runs through Fall 2025.
- **Everything is historical.** Past terms don't guarantee future difficulty;
  instructors and syllabi change.

## Tools at a glance

Brief intros below; for full parameters, return shapes, and examples see
[references/tools.md](references/tools.md).

- **`search_courses`** — find courses by department, number, keyword, level, GPA, or
  term. The entry point for most "what should I take" questions.
- **`get_grades_of_a_course`** — deep dive on one course: per-professor and per-term
  grade breakdowns, RMP scores, and the lib-ed requirements it satisfies.
- **`search_professors`** — resolve a name to a `professor_id`, RMP scores, and the
  courses they teach. **Step 1** of professor research.
- **`get_grades_of_a_professor`** — full grading record for one `professor_id`.
  **Step 2** of professor research.
- **`get_liberal_education_courses`** — courses fulfilling a lib-ed requirement.
- **`get_abbreviations_and_terms`** — valid department abbreviations and term codes;
  call it when a department or term is ambiguous.
- **`query_database`** — raw read-only `SELECT`. Last resort only, for custom
  aggregations the other tools can't express (e.g. university-wide rankings).

## Routing: question → tools

| Student asks… | Tools | Approach |
|---|---|---|
| "Good / easy courses in dept X?" | `search_courses` | Filter by `dept_abbr`, add `min_gpa`; sort results by `average_gpa` |
| "Is this specific course hard?" | `get_grades_of_a_course` | Read GPA, distribution, withdrawal rate; note variation across professors |
| "Tell me about Professor X" | `search_professors` → `get_grades_of_a_professor` | Resolve ID, then pull full record |
| "Which section/prof should I pick?" | `get_grades_of_a_course` (or two `get_grades_of_a_professor`) | Compare per-prof GPA + SRT + RMP for the same course |
| "I need an easy lib-ed for requirement Y" | `get_liberal_education_courses` → `search_courses` | List options, then filter/sort by difficulty |
| "Compare course A vs course B" | `get_grades_of_a_course` ×2 | Side-by-side GPA, ratings, descriptions |

## Core workflows

**Finding a course** — start broad, then narrow. `search_courses` with a department
or keyword and any difficulty filter → sort by `average_gpa` → `get_grades_of_a_course`
on the top 2–3 → compare professors and distributions → recommend with reasoning.

**Researching a professor** — `search_professors(name)` → take the `professor_id`
→ `get_grades_of_a_professor(id)` → assess overall distribution, consistency across
courses, RMP, and SRT → give a balanced read.

**Planning lib-eds** — `get_liberal_education_courses(requirement)` → apply the
student's constraints (difficulty, department, workload) via `search_courses` with
`min_gpa` → present a few varied options.

**Comparing options** — pull the same data for each candidate and lay it out
side-by-side (a table works well) so the tradeoff is visible, then recommend against
the student's stated goal (easy A vs. real learning vs. light workload).

## Presenting results

- Use a table whenever you're comparing more than one course or professor.
- Always attach sample size (`total_students`) — a GPA without an N is misleading.
- Surface RMP and student ratings when available, and match the rating to the
  student's actual concern (workload → Effort Reasonable; learning → Deep
  Understanding). The rating glossary is in
  [references/data-interpretation.md](references/data-interpretation.md).
- Link to OneStop for official course info, and frame everything as historical
  evidence, not a guarantee.
- Be balanced — name the tradeoffs rather than overselling a single "best" answer.

## When results come back empty

Loosen before you give up: widen the GPA range, drop the level filter, broaden the
search term, or confirm the `dept_abbr` and campus via `get_abbreviations_and_terms`.
For professors, try a partial-name search and check spelling variants — newer
instructors may simply have no data yet.

## Reference files

- [references/tools.md](references/tools.md) — full parameters, return shapes, and
  examples for every tool. Read the relevant entry when the one-line intro above
  isn't enough.
- [references/data-interpretation.md](references/data-interpretation.md) — grade/GPA
  scale, ratings glossary, RMP interpretation, and data caveats. **Read it before
  analyzing numbers for a student.**
- [references/gopherGradesDatabaseInfo.md](references/gopherGradesDatabaseInfo.md) —
  full DB schema. Read before writing any `query_database` SQL.
- [references/abbreviationsAndTerms.json](references/abbreviationsAndTerms.json) —
  department abbreviations and term-code reference.
