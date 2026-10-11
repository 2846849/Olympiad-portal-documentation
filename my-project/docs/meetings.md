# Meeting Minutes

Records of the Olympiad Portal team's internal meetings and stakeholder (tutor/client) reviews, grouped by sprint. They show how the team planned its work, what feedback it received, and how it responded.

Minutes were exported from the [Trello board](https://trello.com/b/5cHqfLX9/olympiadportal) on 10 October 2026. Each record is kept as Markdown with a PDF version. Screenshots are attached where their on-screen date makes the association reasonable.

## Index

| Date | Meeting | Type | Sprint | Evidence | PDF |
|---|---|---|---|---|---|
| 2026-08-07 | First team meeting | Internal | 1 | None | `Sprint_1/2026-08-07_7_August_First_team_meeting.pdf` |
| Undated | First stakeholder meeting | Stakeholder | 1 | None | `Stakeholder_Reviews/Undated_First_Stakeholder_meeting.pdf` |
| 2026-08-14 | Team sync: task allocation | Internal | 1 | None | `Sprint_1/2026-08-14_Minutes_14_Aug.pdf` |
| 2026-08-24 | Team sync: Sprint 1 review | Internal | 1 | None | `Sprint_1/2026-08-24_Minutes_24_Aug.pdf` |
| 2026-08-25 | Sprint 1 stakeholder review | Stakeholder | 1 | None | `Stakeholder_Reviews/2026-08-25_Sprint_1_Stakeholder_Review.pdf` |
| 2026-09-14 | Team sync | Internal | 2 | Screenshot | `Sprint_2/2026-09-14_Minutes_for_14_September_meeting_Team_sync.pdf` |
| 2026-09-15 | Sprint 2 stakeholder review | Stakeholder | 2 | None | `Stakeholder_Reviews/2026-09-15_Sprint_2_Stakeholder_Review.pdf` |
| 2026-09-17 | Team meeting | Internal | 2 | Screenshot | `Sprint_2/2026-09-17_Minutes_17_September_2026.pdf` |
| 2026-09-23 | Sprint 3 stakeholder meeting | Stakeholder | 3 | Screenshot | `Stakeholder_Reviews/2026-09-23_Sprint_3_Stakeholder_Meeting.pdf` |
| 2026-09-26 | Team meeting | Internal | 3 | Screenshot | `Sprint_3/2026-09-26_Meeting_Minutes_26_September_2026.pdf` |
| 2026-10-05 | Sprint 4 planning | Internal | 4 | Rubric screenshot | `Sprint_4/Meeting_Minutes_05_October_2026.pdf` |
| 2026-10-07 | Final submission review (with tutor) | Internal | 4 | Rubric screenshot | `Sprint_4/Meeting_Minutes_07_October_2026.pdf` |

## Sprint 1 (to 25 August 2026): setup and basic tier

### 7–8 August: First team meeting
*In person, MSL labs. Full team.*

- Agreed **Agile** as the overall methodology (Scrum sits within it), releasing an MVP first and iterating.
- Confirmed the repository lives on **Gitea**. Engedzani pushed the initial MVP scaffold as a starting point.
- Agreed a branch-based Git workflow: branch from `main` per feature, commit and push often, open a pull request, have another member review and merge.
- Next step: allocate tasks and write user stories.

### First stakeholder meeting (undated)
*In person. Full team and client (Jaishil).*

- Walked through the project brief and demoed the MVP.
- **API:** must be the team's own, not a third party's.
- **Supabase:** may be used to host the database only.
- **Task tracking:** whichever tool is used must be publicly viewable without an account, which rules out Notion.

### 14 August: Team sync (task allocation)
*Online, Teams. Mashudu, Andile, Engedzani, Phumudzo, Nhlakanipho, Kedibone.*

- Agreed a single unified `users` table with roles (organiser, educator, learner) and a full schema drafted up front for documentation.
- Walked through the end-to-end flow: landing page, sign-up with role selection, then organiser, educator and student responsibilities.
- Practice papers can be downloaded early. Round papers are locked until the round opens and inaccessible after it closes.
- Planned three build sprints plus a final polish and documentation sprint, tackling basic, then intermediate, then advanced tiers.
- Agreed the Git methodology and project methodology both need documenting (rubric items).

### 24 August: Team sync (Sprint 1 review)
*Online. Full team.*

- Documentation to cover project structure, what is implemented, methodology, and a record of meetings, split into separate README files.
- Organiser backend walked through (router, service, controller). A **temporary email-only login bypass** was added for dashboard testing and flagged to be replaced with proper authentication.
- Educator/student side mostly working: school and educator registration, registering students, round-based access rules. Paper upload/download was still hardcoded on the frontend.
- API was local-only. Backend deployment and CI/CD to be set up. UI confirmed to render on mobile.
- Public docs site deployed on GitHub Pages.

### 25 August: Sprint 1 stakeholder review
*In person. Full team and client.*

- **Assessed as strong:** version control and Git methodology, documentation and work tracker, tech stack (React/Vite, Node/Express with Prisma, Supabase-hosted Postgres), stakeholder interaction, and Sprint 1 implementation.
- **Flagged for improvement: project methodology.** It must be documented, referenced and evidenced (sprint structure, workflow, ceremonies, how decisions trace back), not just named.
- **Response:** write a formal methodology document and a build-out plan mapping Sprints 2 and 3 to the Intermediate and Advanced tiers.

## Sprint 2 (to 15 September 2026): intermediate tier

### 14 September: Team sync
*Gitea / team sync. Full team.*

- Task owners: Kedi (API documentation, feedback form), Mashudu (deployment, automated testing), Andile (project documentation), Engedzani (database documentation), Phumudzo (bug tracker), Nhlakanipho (third-party code documentation).
- Aligned on sign-up flows for organisers and for educators/students.
- Agreed a designated **bug tracker manager** role.
- Reviewed how Gitea Actions runners are configured and monitored.

### 15 September: Sprint 2 stakeholder review
*Discussion during the Sprint 2 assessment.*

| Feedback | Team response |
|---|---|
| Track bugs in Gitea Issues | Use Gitea Issues for all bugs |
| Too much depends on email invitations (only organisers could sign up; educators invited students by email) | Review sign-up so users are not blocked when an invitation fails |
| Invitation links pointed to a local address and failed on other devices | Fix links to use a publicly reachable address |
| Keep meeting minutes in Gitea | Add minutes to the project records |
| Must be able to test the APIs | Prepare API tests and demonstrate them |

**Outcome:** the invitation flow was identified as a major blocker and carried into the next sprint.

### 17 September: Team meeting
*Microsoft Teams, 31 minutes. Kedibone, Mashudu, Engedzani, Phumudzo, Nhlakanipho.*

- Review the rubric at the start of each sprint. Unfinished earlier work still counts.
- Finish outstanding basic and intermediate features by Monday 21 September, then move to advanced.
- Let students and educators **sign up without an email invitation**. Email remains for notifications. Google sign-up suggested for later.
- Track bugs in repository issues and rewrite unclear user stories in plain language.
- Prepare for user testing, API testing and response-time checks. Arrange a tutor discussion and give the team access to deployment services.

## Sprint 3 (to 29 September 2026): advanced tier, testing and feedback

### 23 September: Sprint 3 stakeholder meeting
*Online. Tutor: Jaishil Patel. The current app was not demonstrated.*

| Feedback | Team response |
|---|---|
| The email invitation problem blocked parts of the student portal. Get all basic features working first | Fix student portal access and update the deployed frontend |
| Then remove errors from intermediate features before moving on | Verify basics, then fix intermediate errors before advanced work |
| Get real user feedback: observe people testing specific tasks (5 testers minimum, about 10 encouraged) | Run focused user testing and document findings and resulting changes |
| Use a code coverage tool; coverage should exceed 60% | Investigate and add coverage reporting |
| Make selected useful endpoints public and document them (Swagger suggested) | Improve API documentation |
| The deployed frontend lacked the latest features | Bring the deployment up to date |
| Educators may create online questions while organisers provide the paper | Continue the current educator/organiser workflow |

### 26 September: Team meeting
*Online. Kedibone, Engedzani, Mashudu, Madimabe, Nhlakanipho.*

- Reviewed the backlog of about 21 user stories. Goal: main features, including some advanced ones, ready for the 29 September review.
- Backlog to be tidied (E4 and E5 to be separated). Bugs logged as Gitea issues.
- Users should be asked to test **specific tasks** rather than explore freely.
- Mashudu to add code coverage and investigate performance. API requests to be demoed in Postman. An externally accessible endpoint (for example standings) was proposed, with scope and access rules still to be confirmed.

## Sprint 4 (to 11 October 2026): final submission

### 5 October: Sprint 4 planning
*Online, 1 hour. Engedzani, Kedi, Andile, Madimabe, Nhlakanipho (joined later).*

- Move unfinished stories to Sprint 4 on Trello, claim ownership, and mark complete only after code is pushed and the deployed app is checked.
- Walked through the database and API rubric, accessibility (a previous score of 80 was mentioned, with screen-reader support as a possible improvement), and behaviour to verify such as resuming after closing a tab.
- Plan to get role-based feedback, with each set of testers assigned one role.
- Engedzani volunteered to organise the documentation site and add minutes, sprint evidence and screenshots.

### 7 October: Final submission review
*Online, about 22 minutes. Kedi, Engedzani, Jaishil Patel, Andile, Madimabe.*

- Document existing user feedback clearly on the docs site rather than running another full round.
- Contact the assessor for specifics on earlier marking deductions.
- Performance figures recalled from PageSpeed: about 92 for educator and student views and about 74–75 for the organiser view. These were recollections, not newly verified, and the organiser side is to be investigated.
- The final database should contain realistic sample data, not throwaway test accounts or poorly labelled quizzes, and should avoid unnecessary duplication.
- API audit: meaningful endpoint hierarchy, no redundant endpoints, correct HTTP methods, accessible to intended consumers.
- Accessibility audit: contrast, semantic HTML, assistive technology. Add a code formatter such as Prettier.
- Add signup terms and a plain-language data-use explanation (Engedzani). Confirm the report format (LaTeX recommended).

## Key outcomes across sprints

| Theme | What happened |
|---|---|
| Methodology | Agile adopted from the first meeting. The Sprint 1 review required it to be formally documented and evidenced. |
| Sign-up and access | Email invitations were a repeated blocker (Sprints 2 and 3). Decision: sign-up must not depend on an invitation. |
| Testing | API tests, then code coverage (target above 60%), then Postman demos. |
| API | Hand-written by the team. Selected endpoints to be documented and made public. |
| User feedback | Role-based, task-specific testing. Results to be documented with the changes they led to. |
| Quality | Bug tracking in Gitea Issues, accessibility audit, formatter, and clean production data. |

