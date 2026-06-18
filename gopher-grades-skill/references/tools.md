# Gopher Grades Tool Reference

Full parameter and return detail for each tool. The MCP schemas carry the canonical
signatures; this file adds the semantics and edge cases worth knowing before a call —
read the entry for a tool when you need more than the one-line intro in SKILL.md.

## Table of contents
- [search_courses](#search_courses)
- [get_grades_of_a_course](#get_grades_of_a_course)
- [search_professors](#search_professors)
- [get_grades_of_a_professor](#get_grades_of_a_professor)
- [get_liberal_education_courses](#get_liberal_education_courses)
- [get_abbreviations_and_terms](#get_abbreviations_and_terms)
- [query_database](#query_database)

---

## search_courses

Find courses by department, number, level, GPA, keyword, or term. The entry point for
most "what should I take" questions.

**Parameters** (all optional):
- `search_term` — keyword matched against course name and description (e.g.
  `"Machine Learning"`). Spaces are ignored in matching.
- `dept_abbr` — department abbreviation, e.g. `"CSCI"`, `"MATH"`. Case-insensitive.
- `course_num` — exact course number, e.g. `"5511"`.
- `level` — list of levels. Accepts integers keyed on the first digit
  (`[3, 4]` = 3000–4999) **or** strings (`["undergraduate", "master", "doctoral"]`).
- `min_gpa` / `max_gpa` — average-GPA bounds. `min_gpa=3.5` finds easier courses;
  `max_gpa=2.5` finds harder ones.
- `terms` — list of semesters offered; each entry is a name string (`"Fall 2023"`,
  `"Spring 2024"`, `"Summer 2024"`) or a raw term integer. A course matches if it was
  offered in any listed term.
- `campus` — default `"UMNTC"`.
- `limit` — max results, default `20`.

**Returns:** `count` and a `courses` list. Each course has `dept_abbr`, `course_num`,
`course_name`, `total_students`, aggregated `total_grades`, `grades_stats` (incl.
`average_gpa`), `onestop_link`, `course_description`, `cred_min`/`cred_max`, and
`student_ratings`.

**Note:** results are ordered by enrollment (`total_students`), **not** by GPA. To
rank by difficulty, sort the returned list by `grades_stats.average_gpa` yourself, or
constrain with `min_gpa`/`max_gpa`.

**Examples:**
```python
search_courses(search_term="Machine Learning")
search_courses(dept_abbr="CSCI", level=[3, 4], min_gpa=3.5)   # easy upper-level CS
search_courses(dept_abbr="PHYS", max_gpa=2.5)                  # challenging physics
search_courses(dept_abbr="PHIL", terms=["Spring 2024", "Fall 2024"])
```

---

## get_grades_of_a_course

Detailed view of a single course, broken down by professor and term.

**Parameters:**
- `dept_abbr` (required) — e.g. `"CSCI"`.
- `course_num` (required) — e.g. `"5511"`.
- `campus` — default `"UMNTC"`.

**Returns:** full course details plus a `distributions` list — one entry per
professor × term — each with `professor_name`, `professor_id`, RMP score / difficulty
/ link, the academic `term` name (e.g. `"Fall 2023"`), `students`, `grades`, and
`grades_stats`. Also returns `libeds`: the liberal-education requirements satisfied.
Returns `{"error": "Course not found"}` if the course doesn't exist.

**Use for:** "is this course hard?", and course-vs-course comparisons (call once per
course). Reading the per-professor spread shows how much difficulty depends on
instructor.

```python
get_grades_of_a_course(dept_abbr="CSCI", course_num="5511")
```

---

## search_professors

Resolve a professor name (or known ID) to their record and course list. **Step 1** of
professor research — you need the `professor_id` it returns before you can pull full
stats.

**Parameters** (provide at least one of the first two):
- `professor_name` — full or partial name; spaces ignored in matching.
- `professor_id` — database ID, if already known.
- `limit` — default `20`.

**Returns:** a list of professors, each with `professor_id`, `professor_name`, RMP
score / difficulty / link, and `courses_taught` (each: `dept_abbr`, `course_num`,
`course_name`, and `terms_taught` as academic-term names). Ordered by RMP score
(highest first, nulls last). Returns an error entry if neither name nor ID is given.

```python
search_professors(professor_name="Smith")
search_professors(professor_id=123)
```

---

## get_grades_of_a_professor

Complete grading record for one professor. **Step 2** of professor research — feed it
the `professor_id` from `search_professors`.

**Parameters:**
- `professor_id` (required).

**Returns:** a dict with three keys:
- `professor` — details + RMP info.
- `overall_statistics` — total students, `unique_courses`, overall grade distribution,
  and aggregate GPA stats.
- `details_per_course` — keyed by `"DEPT NUM"`; each course has its aggregated grades,
  course-level stats, and `grades_per_term` with per-term students/grades/stats.

Returns `{"error": "Professor not found"}` for an unknown ID.

```python
get_grades_of_a_professor(professor_id=123)
```

---

## get_liberal_education_courses

Courses fulfilling a specific liberal-education requirement.

**Parameters:**
- `libed_name` (required) — partial matches work (e.g. `"Social Sciences"`).
- `campus` — default `"UMNTC"`.
- `limit` — default `50`.

**Returns:** `libed`, `count`, and a `courses` list (`dept_abbr`, `course_num`,
`class_desc`, `total_students`, `total_grades`, credit range), ordered by enrollment.
Returns an error if the requirement name doesn't match. Pair with `search_courses`
(using `min_gpa`) to filter the options by difficulty. For the exact requirement
names, see `get_abbreviations_and_terms`.

```python
get_liberal_education_courses(libed_name="Social Sciences")
```

---

## get_abbreviations_and_terms

Returns the reference data for valid department abbreviations and term encodings. Call
it when a student's department or term is ambiguous, or to validate an abbreviation
before another query. The same content is bundled at
[abbreviationsAndTerms.json](abbreviationsAndTerms.json).

Takes no arguments.

---

## query_database

Raw read-only `SELECT` against the SQLite database. **Last resort only** — the
purpose-built tools cover all common cases. Reach for this only when a question needs
an aggregation or join the other tools can't express (e.g. a university-wide ranking,
since `search_professors` requires a name or ID and can't return a global top-N).

**Parameters:**
- `sql` (required) — a single `SELECT` statement. Non-`SELECT` statements are rejected
  with a `ValueError`.

**Returns:** `rows` (list of dicts) and `row_count`.

Read the schema in [gopherGradesDatabaseInfo.md](gopherGradesDatabaseInfo.md) before
writing SQL. Note that `grades` / `total_grades` columns are JSON blobs — you can't
compute GPA in SQL directly; pull the rows and compute in your analysis, or use a
purpose-built tool that already returns `grades_stats`.

```python
# Highest-rated professors university-wide (not expressible via search_professors)
query_database(sql="SELECT name, RMP_score FROM professor WHERE RMP_score IS NOT NULL ORDER BY RMP_score DESC LIMIT 15")

# Courses per department on the Twin Cities campus
query_database(sql="SELECT dept_abbr, COUNT(*) AS count FROM classdistribution WHERE campus='UMNTC' GROUP BY dept_abbr ORDER BY count DESC LIMIT 10")
```
