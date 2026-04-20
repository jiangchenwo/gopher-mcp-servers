Analyzing database: ../data/gopherGrades.db
===========================================

Found 8 tables:

- libed
- professor
- libedAssociationTable
- distribution
- termdistribution
- alembic_version
- classdistribution
- departmentdistribution

Detailed Table Information:
===========================

Table: libed
------------

Rows: 17
Columns:
• id: INTEGER (PK, NOT NULL)
• name: VARCHAR(128) (NOT NULL)

Table: professor
----------------

Rows: 11518
Columns:
• id: INTEGER (PK, NOT NULL)
• name: VARCHAR(255) (NOT NULL)
• RMP_score: FLOAT
• RMP_diff: FLOAT
• RMP_link: VARCHAR(512)
• x500: VARCHAR(16)

Table: libedAssociationTable
----------------------------

Rows: 2657
Columns:
• left_id: INTEGER (PK, NOT NULL)
• right_id: INTEGER (PK, NOT NULL)
Foreign Key Relationships:
libedAssociationTable.right_id -> classdistribution.id
libedAssociationTable.left_id -> libed.id

Table: distribution
-------------------

Rows: 29859
Columns:
• id: INTEGER (PK, NOT NULL)
• class_id: INTEGER (NOT NULL)
• professor_id: INTEGER
Foreign Key Relationships:
distribution.professor_id -> professor.id
distribution.class_id -> classdistribution.id

Table: termdistribution
-----------------------

Rows: 72193
Columns:
• id: INTEGER (PK, NOT NULL)
• dist_id: INTEGER (NOT NULL)
• students: INTEGER (NOT NULL)
• term: SMALLINT (NOT NULL)
• grades: JSON (NOT NULL)
Foreign Key Relationships:
termdistribution.dist_id -> distribution.id

Table: alembic_version
----------------------

Rows: 1
Columns:
• version_num: VARCHAR(32) (PK, NOT NULL)

Table: classdistribution
------------------------

Rows: 10919
Columns:
• id: INTEGER (PK, NOT NULL)
• campus: VARCHAR(8)
• dept_abbr: VARCHAR(4)
• course_num: VARCHAR(8)
• class_desc: VARCHAR(255) (NOT NULL)
• total_students: INTEGER (NOT NULL)
• total_grades: JSON (NOT NULL)
• onestop: VARCHAR(512)
• onestop_desc: VARCHAR(2048)
• cred_min: SMALLINT
• cred_max: SMALLINT
• srt_vals: JSON
Foreign Key Relationships:
classdistribution.campus -> departmentdistribution.campus
classdistribution.dept_abbr -> departmentdistribution.dept_abbr

Table: departmentdistribution
-----------------------------

Rows: 557
Columns:
• campus: VARCHAR(8) (PK)
• dept_abbr: VARCHAR(4) (PK, NOT NULL)
• dept_name: VARCHAR(255) (NOT NULL)

All Database Relationships:
---------------------------

libedAssociationTable.right_id -> classdistribution.id
libedAssociationTable.left_id -> libed.id
distribution.professor_id -> professor.id
distribution.class_id -> classdistribution.id
termdistribution.dist_id -> distribution.id
classdistribution.campus -> departmentdistribution.campus
classdistribution.dept_abbr -> departmentdistribution.dept_abbr
