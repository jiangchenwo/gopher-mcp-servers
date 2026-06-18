# Interpreting Gopher Grades Data

Read this when you're about to analyze grades, ratings, or professor scores for a
student — it explains what the raw numbers mean so your recommendations are
grounded rather than guessed. The tools return structured data; this file is the
key for reading it well.

## Grade distributions

Every course, professor, and term carries a `grades` dict mapping letter grades to
student counts, plus a `grades_stats` object computed from it.

**Letter grade → GPA value:**

| Grade | GPA  | Meaning |
|-------|------|---------|
| A, A- | 4.00, 3.67 | Excellent |
| B+, B, B- | 3.33, 3.00, 2.67 | Good |
| C+, C, C- | 2.33, 2.00, 1.67 | Average |
| D+, D | 1.33, 1.00 | Below average |
| F | 0.00 | Failing |
| S / N | — | Satisfactory / No-credit (S-N option; excluded from GPA) |
| W | — | Withdrew (excluded from GPA) |

**Metrics in `grades_stats`:**
- **average_gpa** — the headline difficulty proxy (see thresholds below).
- **Pass rate** — share of non-W, non-F grades.
- **Withdrawal rate** — share of W grades. A high W rate is a louder warning sign
  than a middling GPA, because it reflects students who bailed.
- **A rate** — share of A/A- grades; a grading-generosity signal.

## Reading average GPA as difficulty

A rough field guide, not a verdict:
- **≥ 3.5** — generally easier / generously graded.
- **3.0 – 3.5** — moderate.
- **< 3.0** — more challenging.

Two caveats that matter more than the threshold itself:
- **Sample size.** A 3.9 GPA over 12 students is noise; over 1,200 students it's a
  pattern. Always weight by `total_students`, and say so when a number is thin.
- **GPA is not the whole story.** A course can have a high average GPA and a high
  withdrawal rate (the strugglers left). Look at the full distribution and the W
  rate, not just the mean.

## Student ratings (SRT)

When present (`student_ratings` / `srt_vals`), these are official end-of-term survey
scores. Match the field to what the student actually cares about:

| Field | What it tells you |
|-------|-------------------|
| Deep Understanding | How much real learning the course promoted |
| Stimulated Interest | How engaging it was |
| Technical Effectiveness | Quality of instruction |
| Activities Supported Learning | Whether assignments helped |
| Effort Reasonable | Workload — high score means manageable |
| Grading Standards | Perceived fairness of grading |
| Recommend | Would students recommend it |
| Number of Responses | Sample size for the ratings above |

A student optimizing for *learning* cares about Deep Understanding and Technical
Effectiveness; one worried about *workload* cares about Effort Reasonable.

## Rate My Professor (RMP)

Self-selected, off-the-record reviews — useful but biased toward strong opinions.
- **RMP_score** (out of 5): ≥4.0 excellent, 3.0–4.0 good, <3.0 mixed.
- **RMP_diff** (out of 5): higher = harder, interpret relative to the student's goals.

When RMP and the official SRT scores disagree, say so rather than averaging them —
they measure different populations.

## Data scope and limitations

- **Campus:** data covers UMN Twin Cities (`UMNTC`, the default). Other campus codes
  exist but Twin Cities is by far the most complete.
- **Coverage:** grade data runs through Fall 2025.
- **Historical, not predictive.** Distributions describe the past. The professor,
  syllabus, and difficulty can all change.
- **Professor turnover.** The same course is often taught by different instructors
  across terms — pull per-professor data rather than assuming the course is uniform.
- **RMP is self-selected.** Treat it as one signal among several, not ground truth.

Lead recommendations with the numbers and their sample sizes, point students to
OneStop for official details, and frame everything as historical evidence rather
than a guarantee.
