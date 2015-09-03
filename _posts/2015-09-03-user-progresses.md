---
layout: post
title: Code Talk Notes: User Progresses Update
---

# Notes from Flatiron Code Talk

Pain points:
- Submission retention
- previously: deleting User Progress if Content deleted (but not if Lesson deleted)
  - irrecoverable
  - if Lesson deleted, ups were orphaned
- UP associated with Lesson, Content, User, Assignment
- UP had loose connection to Github Repository through repo name

Solution:
- User Progress no longer tied to Content / Lesson
- Now tied to Github Repository
- Assignment published_at now irrelevant
- Now any work done on a lesson is done on all versions of that lesson (same progress across batches)
- ^^ aka you get credit everywhere
