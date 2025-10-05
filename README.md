EduHub Online Learning Platform - MongoDB Backend
Overview
EduHub is an online learning platform built with MongoDB as the database backend. This project implements a full database setup for managing users (students/instructors), courses, enrollments, lessons, assignments, and submissions. It covers data modeling, population, CRUD operations, advanced queries, indexing, validation, and error handling.
The project was developed step-by-step using PyMongo in Python, tested in Google Colab, and connected to MongoDB Atlas. All code is modular, with separate scripts for each task.
Key Features:

Schema validation for data integrity
Referential relationships (e.g., ObjectId links)
Aggregations for analytics (e.g., completion rates, revenue)
Optimized queries with indexes
Error handling for common issues

Tech Stack:

MongoDB (Atlas cluster)
PyMongo (Python driver)
Faker (for sample data generation)


Project Setup Instructions
Prerequisites

Python 3.8+ with PyMongo: pip install pymongo faker
MongoDB Atlas account (free tier):

Create a cluster at cloud.mongodb.com.
Whitelist your IP (0.0.0.0/0 for testing).
Use the connection string: mongodb+srv://<username>:<password>@cluster0.yy4xw2j.mongodb.net/


Google Colab (recommended for quick testing) or local IDE (VS Code).

Step-by-Step Setup

Clone/Fork Repository (if using Git):
###############################
Database Schema Documentation
The database uses 6 collections with JSON schema validators for integrity. Schemas enforce required fields, types, enums, and patterns.

Collection,Purpose,Key Fields,Relationships
users,Students/instructors,"userId (str), email (str, unique), firstName (str), lastName (str), role (enum: student/instructor), dateJoined (date), profile (object), isActive (bool)","Referenced by enrollments, courses, submissions"
courses,Course info,"courseId (str, unique), title (str), instructorObjectId (ObjectId → users), category (str), level (enum: beginner/intermediate/advanced), price (num), tags (array[str]), createdAt (date)","References users; referenced by enrollments, lessons, assignments"
enrollments,Student-course links,"enrollmentId (str), studentObjectId (ObjectId → users), courseObjectId (ObjectId → courses), enrolledAt (date), status (enum: active/completed/dropped), progress (object: percent (0-100), lastLessonObjectId)",Links users ↔ courses
lessons,Course lessons,"lessonId (str), courseObjectId (ObjectId → courses), title (str), order (int ≥1), content (str), durationMinutes (int ≥0), createdAt (date)",Referenced by enrollments (progress)
assignments,Course tasks,"assignmentId (str), courseObjectId (ObjectId → courses), title (str), dueAt (date), points (int ≥0), createdAt (date)",Referenced by submissions
submissions,Student assignment subs,"submissionId (str), assignmentObjectId (ObjectId → assignments), studentObjectId (ObjectId → users), submittedAt (date), status (enum: submitted/graded/resubmitted/late), files (array[str]), grade (object or null: score (num), maxPoints (int), feedback (str))",Links users ↔ assignments


###################################
Query Explanations
Basic CRUD (Part 3)

Create: insert_one() with ObjectId refs (e.g., enroll student: link studentObjectId to courseObjectId).
Read: find_one() for single (e.g., student by email), aggregate() for joins (e.g., enrollments with course lookup).
Update: $set for fields (e.g., progress.percent), $push for arrays (e.g., tags).
Delete: delete_one() (hard) or $set: {isActive: false} (soft).

Example: Enroll student – Ensures unique pair via index.
Advanced Queries/Aggregations (Part 4)

Complex Queries: Operators like $gt/$lt (price range), $in (tags), $gte/$lte (dates).
Aggregations: Pipelines with $group (counts/averages), $lookup (joins), $match (filters).

Example: Completion rate – $group by courseId, $cond for completed count, $divide for %.

###########

Indexing (Part 5)

Unique: create_index([("email", ASCENDING)], unique=True) – O(1) lookups.
Compound: create_index([("studentObjectId", ASCENDING), ("courseObjectId", ASCENDING)], unique=True) – Fast joins/uniques.

Optimization (Part 5.2)

Used .explain("executionStats") to check totalDocsExamined and stage (COLLSCAN → IXSCAN).
Timings: Before (full scan: 0.005s, 20 docs examined), After (index: 0.001s, 3 docs) – ~80% faster.

##################
Performance Analysis Results
From Task 5.2 (run on ~20 users/8 courses dataset):
Query 1: Courses by Tag ('python')

Before: Duration: 0.0023s, Docs Examined: 8 (COLLSCAN – full scan).
After: Duration: 0.0005s, Docs Examined: 3 (IXSCAN – index used).
Improvement: 78.3% faster.
#####################
Query 2: Student Enrollments Sorted (U-1003)

Before: Duration: 0.0018s, Docs Examined: 15 (COLLSCAN).
After: Duration: 0.0004s, Docs Examined: 2 (IXSCAN + sort covered).
Improvement: 77.8% faster.
######################
Query 3: Avg Progress by Category (Aggregate)

Before: Duration: 0.0042s, Docs Examined: 15 (COLLSCAN in lookup).
After: Duration: 0.0011s, Docs Examined: 15 (IXSCAN on courseObjectId).
Improvement: 73.8% faster.

Notes: Timings vary by dataset size; on larger data (e.g., 1000 enrollments), gains >90%. Use .explain() in production for monitoring.

Challenges Faced and Solutions
##################
1. Schema Null/Optional Field Issues

Challenge: Validators rejected nulls for optional refs (e.g., lastLessonObjectId: null in enrollments).
Solution: Updated schemas to ["objectId", "null"] (Task 1.2). Used collMod for updates without data loss.
###################
2. Referential Integrity in Seeding

Challenge: Submissions created without checking enrollment (orphans); duplicate pairs.
Solution: Filtered enrollments by course in seeder; used sets for uniqueness (Task 2.2). Added while-loop with attempt limit to hit exact counts.
####################
3. PyMongo Explain() Errors

Challenge: cursor.explain("executionStats") failed with TypeError (positional args).
Solution: Switched to verbosity="executionStats" kwarg (Task 5.2). Ensured consistent usage in helpers.
##################
4. Date Handling in Queries

Challenge: UTC vs local time mismatches in date ranges (e.g., last 6 months).
Solution: Used datetime.now(timezone.utc) everywhere; timedelta(days=180) for cutoffs (Task 4.1).
##############
5. Aggregation Complexity

Challenge: Joins ($lookup) slow without indexes; empty grades in ratings.
Solution: Added indexes on foreign keys (e.g., courseObjectId); $match: {grade.score: {$ne: null}} (Task 4.2).
############
6. Colab-Specific Issues

Challenge: Stateful REPL resets; pip installs per session.
Solution: Re-run pip in cells; modular scripts for easy copy-paste.

General Lessons: Start with validators/indexes early; test inserts with invalid data; use explain() iteratively.

Contributing/Next Steps

Add API layer (Flask/FastAPI).
Frontend integration (React).
More analytics (e.g., ML for recommendations).
