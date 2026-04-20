---
name: gopher-grades
description: Search and analyze University of Minnesota (UMN) courses, professors, and grade distributions. Use when helping students find courses, analyze course difficulty, research professors, or explore liberal education requirements.
license: MIT
---

# Gopher Grades Skill

## Overview

This skill enables you to access and analyze University of Minnesota (UMN) course and grade data through the Gopher Grades MCP server. You can search for courses, analyze grade distributions, research professors, and help students make informed decisions about their education.

---

# Core Capabilities

## Available Tools

### 1. search_courses
Search for UMN courses by department, course number, level, GPA, keywords, or semester.

**Common Use Cases:**
- Find courses by department (e.g., "CSCI courses")
- Search by keyword (e.g., "Machine Learning")
- Find easy courses (high average GPA: `min_gpa=3.5`)
- Find challenging courses (low average GPA: `max_gpa=2.5`)
- Filter by course level (undergraduate, master, doctoral)

**Key Parameters:**
- `search_term`: Keyword search against course names and descriptions (e.g., "Machine Learning")
- `dept_abbr`: Department abbreviation (e.g., "CSCI", "MATH")
- `course_num`: Specific course number (e.g., "5511")
- `level`: Course level as list (e.g., `["undergraduate", "master"]` or `[3, 4]` for 3000-4999 level)
- `min_gpa`, `max_gpa`: Filter by average GPA range
- `terms`: Filter by semesters offered (e.g., `["Fall 2023"]` or `["Spring 2024", "Fall 2024"]`); accepts term name strings or raw term integers
- `limit`: Maximum results (default: 20)
- `campus`: Campus code (default: "UMNTC" for Twin Cities)

**Returns:**
- Course count and list of matching courses
- Department abbreviation, course number, course name
- Total students, grade distribution, GPA statistics
- Course description, credit range
- Student ratings (understanding, interest, effectiveness, etc.)
- OneStop link for official information

**Example Usage:**
```python
# Find machine learning courses
search_courses(search_term="Machine Learning")

# Find easy CSCI upper-level courses
search_courses(dept_abbr="CSCI", level=[3, 4], min_gpa=3.5)

# Find challenging physics courses
search_courses(dept_abbr="PHYS", max_gpa=2.5)

# Find Philosophy courses offered in either Spring or Fall 2024
search_courses(dept_abbr="PHIL", terms=["Spring 2024", "Fall 2024"])
```

### 2. get_grades_of_a_course
Get detailed grade information for a specific course, including breakdowns by professor and term.

**Use Cases:**
- Analyze how a course's difficulty varies by professor
- See grade trends over different semesters
- Understand liberal education requirements fulfilled

**Key Parameters:**
- `dept_abbr`: Department abbreviation (required)
- `course_num`: Course number (required)
- `campus`: Campus code (default: "UMNTC")

**Returns:**
- Complete course details
- Grade distributions by professor and term
- Rate My Professor scores for each instructor
- Academic term names (e.g., "Fall 2023")
- Liberal education requirements fulfilled
- GPA statistics for each professor/term combination

**Example Usage:**
```python
# Get detailed info for CSCI 5511
get_grades_of_a_course(dept_abbr="CSCI", course_num="5511")
```

### 3. search_professors
Search for professors by name to find their courses and ratings.

**Use Cases:**
- Research a professor before taking their class
- Find all courses taught by a specific professor
- Compare professors by Rate My Professor scores

**Key Parameters:**
- `professor_name`: Full or partial professor name
- `professor_id`: Database ID if known
- `limit`: Maximum results (default: 20)

**Returns:**
- Professor database ID and name
- Rate My Professor score, difficulty rating, and link
- Complete list of courses taught
- Terms taught for each course

**Example Usage:**
```python
# Search for Professor Smith
search_professors(professor_name="Smith")

# Get specific professor by ID
search_professors(professor_id=123)
```

### 4. get_grades_of_a_professor
Get comprehensive grading statistics for a professor across all courses and terms.

**Use Cases:**
- Analyze a professor's grading patterns
- Compare difficulty across different courses they teach
- Understand overall teaching effectiveness

**Key Parameters:**
- `professor_id`: The professor's database ID (required)

**Returns:**
- Professor details and Rate My Professor info
- Overall statistics: total students, unique courses, overall grade distribution
- Per-course details with aggregated grades
- Per-term grade statistics for each course
- GPA statistics at all levels

**Example Usage:**
```python
# Get all grading data for professor ID 123
get_grades_of_a_professor(professor_id=123)
```

### 5. get_liberal_education_courses
Find courses that fulfill specific liberal education requirements.

**Use Cases:**
- Help students find lib ed courses
- Explore options for specific requirements
- Find high-quality lib ed courses

**Key Parameters:**
- `libed_name`: Name of the liberal education requirement
- `campus`: Campus code (default: "UMNTC")
- `limit`: Maximum results (default: 50)

**Returns:**
- Liberal education requirement name
- Count of matching courses
- Course details with student counts and grades
- Credit ranges

**Example Usage:**
```python
# Find courses for Social Sciences requirement
get_liberal_education_courses(libed_name="Social Sciences")
```

### 6. query_database
Execute a raw SELECT query directly against the GopherGrades SQLite database.

> **Use only as a last resort.** The purpose-built tools above cover all common use cases. Only reach for this tool when none of them can answer the question (e.g., highly custom aggregations or joins not exposed elsewhere).

**Before writing a query**, consult the full database schema in `references/gopherGradesDatabaseInfo.md`. Key tables: `classdistribution`, `distribution`, `termdistribution`, `professor`, `libedAssociationTable`.

**Key Parameters:**
- `sql`: A `SELECT` statement. Non-SELECT statements are rejected.

**Returns:**
- `rows`: List of result dicts
- `row_count`: Number of rows returned

**Example Usage:**
```python
# Count courses per department on Twin Cities campus
query_database(sql="SELECT dept_abbr, COUNT(*) as count FROM classdistribution WHERE campus='UMNTC' GROUP BY dept_abbr ORDER BY count DESC LIMIT 10")
```

---

# Best Practices

## When Helping Students

### 1. Course Selection
**Start broad, then narrow:**
1. Use `search_courses` with general terms or department
2. Apply GPA filters if difficulty is a concern
3. Use `get_grades_of_a_course` for detailed analysis of specific courses
4. Compare professors using `search_professors` and `get_grades_of_a_professor`

### 2. Professor Research
**Follow this workflow:**
1. Search for professor by name with `search_professors`
2. Note the professor_id from results
3. Use `get_grades_of_a_professor` with that ID for detailed analysis
4. Compare grade distributions across different courses they teach

### 3. Grade Analysis
**Interpret GPA statistics carefully:**
- Average GPA above 3.5: Generally considered easier
- Average GPA 3.0-3.5: Moderate difficulty
- Average GPA below 3.0: More challenging
- Consider sample size (total students) for reliability
- Look at grade distribution, not just GPA

### 4. Liberal Education Requirements
**Help students efficiently:**
1. Use `get_liberal_education_courses` to list options
2. Cross-reference with `search_courses` using department or keywords
3. Use GPA filters to find appropriate difficulty level

## Response Formatting

### Present Data Clearly
- Use tables for comparing multiple courses or professors
- Highlight GPA statistics prominently
- Include Rate My Professor scores when available
- Link to OneStop for official course information
- Mention total student counts for context

### Example Response Structure
When a student asks "What are some good machine learning courses?":

1. Search for relevant courses
2. Present top results in a table with:
   - Course code and name
   - Average GPA and difficulty indicator
   - Total students (sample size)
   - Key student ratings
3. Suggest getting detailed info on specific courses
4. Offer to look up specific professors if interested

---

# Common Workflows

## Workflow 1: Finding the Right Course

```
Student: "I need an easy upper-level CS course"

Steps:
1. search_courses(dept_abbr="CSCI", level=[3, 4], min_gpa=3.5, limit=10)
2. Present results sorted by GPA
3. For top 2-3 courses, call get_grades_of_a_course()
4. Compare professor ratings and grade distributions
5. Make recommendations with reasoning
```

## Workflow 2: Professor Research

```
Student: "Is Professor Smith a good teacher?"

Steps:
1. search_professors(professor_name="Smith")
2. Get professor_id from results
3. get_grades_of_a_professor(professor_id=X)
4. Analyze:
   - Overall grade distribution
   - Consistency across courses
   - Rate My Professor scores
   - Student ratings
5. Provide balanced assessment
```

## Workflow 3: Liberal Education Planning

```
Student: "I need to fulfill my Social Sciences requirement"

Steps:
1. get_liberal_education_courses(libed_name="Social Sciences")
2. Apply filters if student has preferences (easy, specific dept, etc.)
3. Use search_courses() with min_gpa filter if difficulty matters
4. Present diverse options with different departments
5. Suggest getting details on interesting courses
```

## Workflow 4: Course Comparison

```
Student: "Should I take CSCI 5511 or CSCI 5521?"

Steps:
1. get_grades_of_a_course(dept_abbr="CSCI", course_num="5511")
2. get_grades_of_a_course(dept_abbr="CSCI", course_num="5521")
3. Compare:
   - Average GPA and grade distributions
   - Professor quality (RMP scores)
   - Student ratings
   - Course descriptions
4. Provide side-by-side comparison
5. Make recommendation based on student goals
```

---

# Data Interpretation Guide

## Understanding Grade Distributions

**Letter Grade Mapping:**
- A, A-: Excellent (4.0, 3.67 GPA)
- B+, B, B-: Good (3.33, 3.0, 2.67 GPA)
- C+, C, C-: Average (2.33, 2.0, 1.67 GPA)
- D+, D: Below Average (1.33, 1.0 GPA)
- F: Failing (0.0 GPA)
- S/N: Satisfactory/No Credit (S-N grading option)
- W: Withdrew from course

**Key Metrics:**
- **Average GPA**: Overall course difficulty indicator
- **Pass Rate**: Percentage of non-W, non-F grades
- **Withdrawal Rate**: Percentage of W grades (high = warning sign)
- **A Rate**: Percentage of A/A- grades (generosity indicator)

## Student Ratings

When available, student ratings include:
- **Deep Understanding**: How well the course promoted learning
- **Stimulated Interest**: How engaging the course was
- **Technical Effectiveness**: Quality of instruction
- **Activities Supported Learning**: How helpful assignments were
- **Effort Reasonable**: Workload assessment
- **Grading Standards**: Fairness of grading
- **Recommend**: Would students recommend this course/professor
- **Number of Responses**: Sample size for ratings

## Rate My Professor Scores

- **Score**: Overall rating (out of 5.0)
  - 4.0+: Excellent
  - 3.0-4.0: Good
  - Below 3.0: Mixed reviews
- **Difficulty**: Difficulty rating (out of 5.0)
  - Higher = more difficult
  - Consider in context of student goals

---

# Important Notes

## Data Scope
- Data is specific to University of Minnesota Twin Cities campus (UMNTC) courses and professors
- The grade data is up to Fall 2025.

## Data Limitations

1. **Historical Data**: Grade distributions represent past performance and may not predict future difficulty
2. **Sample Size**: Consider total_students for statistical reliability
3. **Professor Changes**: Courses may be taught by different professors in different terms
4. **Course Updates**: Course content and requirements may change over time
5. **Rate My Professor**: Self-selected reviews may not represent all students

## Campus Codes

- **UMNTC**: Twin Cities (default)
- Other campuses available but Twin Cities most comprehensive

## Academic Terms and Department Abbreviations

Check `references/abbreviationsAndTerms.json`.


---

# Error Handling

## Common Issues

**Course Not Found:**
- Verify department abbreviation is correct
- Check course number format
- Try searching with search_term first

**Professor Not Found:**
- Try partial name search
- Check spelling variations
- Some professors may not have data if they're new

**Empty Results:**
- Loosen filters (GPA range, course level)
- Broaden search terms
- Check campus parameter

**Liberal Ed Not Found:**
- Use partial matching in libed_name
- Get full list with get_abbreviations_and_terms()

---

# Quick Reference

## Typical Question Patterns

| Student Question | Tools to Use | Approach |
|-----------------|--------------|----------|
| "What are good CS courses?" | search_courses | Use dept_abbr, possibly min_gpa filter |
| "Is this course hard?" | get_grades_of_a_course | Analyze GPA, grade distribution |
| "Tell me about Professor X" | search_professors → get_grades_of_a_professor | Get ID, then detailed stats |
| "I need an easy lib ed" | get_liberal_education_courses → search_courses | Get options, filter by GPA |
| "Compare these courses" | get_grades_of_a_course (multiple) | Side-by-side analysis |
| "What does Professor X teach?" | search_professors → get_grades_of_a_professor | Lists all courses taught |

## Remember

- Always provide context with numbers (sample sizes, term dates)
- Be balanced in recommendations (acknowledge tradeoffs)
- Encourage students to check official sources (OneStop)
- Consider individual student goals and preferences
- Grade distributions are historical data, not guarantees
