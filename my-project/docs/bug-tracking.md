# Olympiad Portal — Bug Tracking

This log records the bugs found in the Olympiad Portal and how each one was fixed. Bugs were found through automated testing (unit, integration and API tests), during deployment, and through use of the portal.

Every bug fixed in the application code has an automated test that reproduces it, so it cannot return unnoticed. See the [test policy](testing.md#3-test-policy).

## How bugs are recorded

| Field | Meaning |
|---|---|
| **ID** | Unique reference (BUG-001, BUG-002, …) |
| **Area** | The part of the system affected |
| **Priority** | Critical, High, Medium or Low (see [priority levels](testing.md#42-recording-and-triage)) |
| **Found by** | Unit tests, integration tests, API tests, deployment, or user feedback |
| **Bug** | What went wrong |
| **Fix** | What was changed to fix it |
| **Status** | Fixed, or open with the reason |

## Summary

| Priority | Count |
|---|---|
| Critical | 5 |
| High | 17 |
| Medium | 15 |
| Low | 6 |
| **Total** | **43** (all fixed) |

---

## Security and data protection

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-001 | Marking | Critical | API tests | Any organiser, or an educator from any school, could change the marks on any answer in any round. The endpoint checked the user's role but not whether they owned the round or the learner. | Marking checks that the user is the organiser who owns the round, or an educator marking a learner from their own school. An answer from a different round is reported as not found. | Fixed |
| BUG-002 | Invitations | Critical | API tests | An educator who had been removed from their school could still create student invitation codes. | Creating student codes needs an active school membership, the same as every other invitation endpoint. | Fixed |
| BUG-003 | Invitations | High | API tests | The public endpoints for checking invitation codes and registering had no limit on repeated requests, so short invitation codes could be guessed by trying many in a row. | Rate limiting added to the public authentication endpoints: 100 requests per 15 minutes per IP address, adjustable with `AUTH_RATE_LIMIT_MAX`. The app trusts the hosting proxy, so each user is counted by their own address. | Fixed |
| BUG-004 | Error handling | High | API tests | An ID that wasn't a valid UUID (for example `/olympiads/abc`) caused a server error (500) whose message exposed internal database details and server file paths. | Invalid IDs return 400 "Invalid ID format" with no internal details. | Fixed |
| BUG-005 | Student papers | High | Use of the portal | The student "Download paper" button opened the paper's raw storage path, which doesn't work for private files and exposed where files are stored. | A student endpoint returns a short-lived signed download link for rounds the student is registered for, once the round has opened. Storage paths are no longer sent to students. | Fixed |
| BUG-006 | Invitations | Low | API tests | Invitation responses included the stored hash of the invitation link's secret token. | The token hash is removed from responses. | Fixed |
| BUG-007 | Logout | High | Use of the portal | "Log out" only returned to the login page. The login token, cached student profile and cached data stayed in the browser, so the next person on a shared computer could see the previous user's name and data. | Logging out clears the login token, the cached profile, cached page data and saved exam progress, and ends the Supabase session. | Fixed |

## Authentication and registration

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-008 | Sign-up | Critical | API tests | Sign-up rewrote some email addresses. For example, `first.last+olympiad@gmail.com` was saved as `firstlast@gmail.com`, so the user could not log in with the address they typed. The same rewriting stopped invitations matching the invited address. | The email check only trims spaces and converts to lowercase; the address is otherwise kept exactly as typed. | Fixed |

## Exams and exam sitting

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-009 | Exam sitting | Critical | Integration tests | Answers could be lost. Autosave used one shared timer, so answering the next question within 0.8 seconds cancelled the save of the previous answer. That answer was also not marked as unsaved, so refreshing the page lost it. | Each question has its own save timer. Answers are marked as unsaved until the server confirms them, so they are restored from the device after a refresh. | Fixed |
| BUG-010 | Exam sitting | Critical | Integration tests | A student who had started but not submitted an online exam was shown as "submitted" on their dashboard and round page, with no way to return to the exam. | An online exam that has been started but not submitted no longer counts as submitted, so "Sit exam online" stays available to resume it. | Fixed |
| BUG-011 | Exam sitting | Medium | Integration tests | The exam timer showed "NaN:NaN:NaN" for a moment while the exam loaded. | The timer stays blank until the exam's closing time is known. | Fixed |
| BUG-012 | Exam sitting | High | API tests | A learner could end up with two sets of results for one round: one captured on paper by their school and one from sitting the exam online. | A learner with an online sitting can't also be captured offline, and a learner whose results were captured on paper can't start the online exam (409 in both cases). | Fixed |
| BUG-013 | Marking | Medium | Unit tests | Selecting the same multiple-choice option more than once could count it more than once when scoring. | Duplicate selections are ignored when scoring. | Fixed |
| BUG-014 | Question builder | High | Unit tests | When a multiple-choice option was left blank, the options after it shifted up on save, but the answer key did not. The saved correct answer then pointed at the wrong option. | The correct-answer positions are renumbered to skip blank options before saving. | Fixed |
| BUG-015 | Exam totals | High | Integration tests | Exam point totals on the exam builder and exam review pages were joined as text instead of added, for example "2" + "3" showed as "23". The database sends points as text. | Points are converted to numbers before they are added. | Fixed |
| BUG-016 | Exam display | Low | Integration tests | A question worth one point showed "1 pts". | Points are converted to numbers before choosing "pt" or "pts". | Fixed |
| BUG-017 | Question builder | Medium | Integration tests | Editing a question and saving without changing its points sent the points to the server as text. | Points are sent as a number. | Fixed |
| BUG-018 | Question builder | Medium | Integration tests | The points and negative marking fields couldn't be cleared: clearing them put the value straight back, so typing "4" produced "14". | The fields can be emptied while typing. An empty field returns to its default when the user leaves it, and invalid values show a message and disable saving. The same fix was applied to the marking queue's points field. | Fixed |
| BUG-019 | Questions | Low | API tests | Questions could be edited or deleted through the address of a different round than the one they belong to. | A question that doesn't belong to the round in the address is reported as not found. | Fixed |
| BUG-020 | Paper viewer | High | Unit tests | The side-by-side paper and memo viewer crashed with "is not a function", because a shared access-check function it used was never exported. | The function is exported, with a test confirming it stays available. | Fixed |

## Submissions, results and registration

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-021 | Offline results | High | API tests | After submitting results for one learner, the round page hid the submission form, so an educator could only submit results for one learner per round. | The round page lists every submission for the school's learners in that round. The form stays open, only offers learners who don't have results yet, and shows a message once everyone is done. | Fixed |
| BUG-022 | Offline results | High | API tests | The same learner's results could be submitted twice by leaving out the `route` field on one request and including it on another. | The default route is applied before checking for duplicates, so both requests are recognised as the same submission. | Fixed |
| BUG-023 | Offline results | High | API tests | Results could be submitted for a learner who wasn't registered for the round. | Results can only be submitted for learners registered for the round (403 otherwise). | Fixed |
| BUG-024 | Offline results | Medium | API tests | A submission with no answers was reported as "auto-marked", so the educator's receipt showed "(auto-marked)" even though nothing was marked. | A submission with no answers is not reported as auto-marked. | Fixed |
| BUG-025 | Submissions | Low | API tests | A submission could be viewed through the address of a different round than the one it belongs to. | A submission that doesn't belong to the round in the address is reported as not found. | Fixed |
| BUG-026 | Learner registration | High | API tests | Learners could be registered for rounds that had already closed, been marked or released their results. | Registration is only accepted while a round is scheduled or open. Registering for a whole olympiad skips closed rounds, and returns an error if every round has closed. The Entrants page shows which rounds the learner was registered for. | Fixed |
| BUG-027 | Learner registration | Medium | API tests | The list of registered learners for a round returned an empty list for a round that didn't exist, or for a school not taking part, instead of an error. | The list returns 404 for an unknown round and 403 for a school not registered for the olympiad, like the other educator endpoints. | Fixed |
| BUG-028 | Results | Medium | Integration and API tests | The educator round page showed results as available, and requested them, while the round was still being marked; the server rejects that request. | Results are shown as available, and requested, only after they are released. | Fixed |

## Rounds, olympiads and schools

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-029 | Rounds | High | Integration tests | Deleting a scheduled round that had a question paper or registered learners failed with a database error. | The round's paper, questions and registrations are deleted together with the round in a single transaction. | Fixed |
| BUG-030 | Rounds | Medium | API tests | Editing a round with a non-numeric qualifying threshold caused a server error (500). | The threshold is validated when a round is edited, the same as when it is created (422 with a clear message). | Fixed |
| BUG-031 | Schools page | High | Integration tests | The organiser's Schools page read fields the server doesn't send. It showed "Invalid Date" and zero educators and learners for every school. | The page uses the server's actual data: educator count and names, learner count, contact email, and invitation status (Pending, Accepted or Expired). | Fixed |
| BUG-032 | Olympiads page | Medium | Integration tests | The Olympiads page passed the wrong settings to the page layout, so the sidebar didn't show the signed-in role correctly. Its Rounds column always showed 0. | The page passes the correct layout settings and reads the round count the server sends. | Fixed |

## Notifications

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-033 | Announcement rules | Medium | Integration tests | Saving a new announcement rule showed an error message even though the rule had been saved. | The form is cleared using a saved reference to the form, which remains valid after the save finishes. | Fixed |
| BUG-034 | Announcement rules | Medium | API tests | Creating a rule without a trigger or recipient caused a server error (500) that exposed internal database details. | Trigger and recipient are required when creating a rule (400 with a clear message). | Fixed |
| BUG-035 | Announcement rules | Low | API tests | A rule could be updated or deleted through the address of a different olympiad owned by the same organiser. | A rule that doesn't belong to the olympiad in the address is reported as not found. | Fixed |

## Error handling and validation

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-036 | Error handling | Medium | API tests | A request with badly formed JSON caused a server error (500) instead of telling the client what was wrong. | Returns 400 "Malformed JSON in request body". | Fixed |
| BUG-037 | Error handling | Medium | API tests | A request from a website not allowed to use the API (CORS) caused a server error (500). | Returns 403 "Origin is not allowed by CORS". | Fixed |
| BUG-038 | Error handling | Medium | API tests | Deleting a record still used by other records (for example, a question that already has answers) caused a server error (500). | Returns 409 "This record is still referenced by other records", and nothing is deleted. | Fixed |
| BUG-039 | Paper upload | Medium | API tests | Uploading a file over 20 MB, or a file under an unexpected field name, caused a server error (500). | Returns 413 for a file that is too large and 400 for an unexpected file field. | Fixed |
| BUG-040 | Validation | Low | API tests | Several validation errors only said "Invalid value" (question points and options, exam status, learner details, notification rules, IDs in the address), or repeated the same message twice. | Each field has a specific message, for example "max_points must be a number greater than 0". | Fixed |

## Deployment

| ID | Area | Priority | Found by | Bug | Fix | Status |
|---|---|---|---|---|---|---|
| BUG-041 | Frontend deployment (Vercel) | High | Deployment | The deployed frontend was built without its settings (API address and Supabase keys), so login and sign-up did not work. The variables were marked as sensitive, so the build step could not read them. | The frontend variables were re-added as non-sensitive, since they are public by design, and the frontend was redeployed. | Fixed |
| BUG-042 | Backend deployment (Render) | High | Deployment | The pipeline step that tells Render to deploy the new backend image failed, because the Render service ID stored as a pipeline secret was wrong. | The `RENDER_SERVICE_ID` secret was corrected to the service's `srv-…` ID. | Fixed |
| BUG-043 | Backend deployment (Docker) | High | Deployment | The backend Docker image failed to build on the Gitea Actions runner because the build had no network access to download packages. | The image is built with `docker build --network=host`, giving the build step the runner's network. | Fixed |

---

## Reporting a new bug

New bugs are logged as issues in the project's Gitea repository, using the [user feedback process](testing.md#4-user-feedback-process). Include:

1. **Role** (organiser, educator or student) and the **page or endpoint**
2. **Steps** to reproduce the bug
3. **Expected** result and **actual** result
4. A **screenshot** or error message, if available

Once a bug is fixed, with a test that reproduces it, it is added to this log with the next ID.
