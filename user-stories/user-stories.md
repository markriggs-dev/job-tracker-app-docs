# Job Tracker — User Stories

Version 1.0 | April 2026  
Author: Mark

## Overview

This document captures the user stories for the Job Tracker application. Stories are organized by domain and labeled with a priority and development phase. Each story follows the standard format: As a [role], I want to [action], so that [benefit], followed by acceptance criteria that define when the story is complete.

Stories are assigned to one of three phases. MVP covers the core functionality required to replace the current OneNote and spreadsheet workflow. Phase 2 adds notifications and AI-assisted features. Phase 3 covers reporting and analytics.

## Priority and phase legend

| Label | Meaning | Phase |
|-------|---------|-------|
| Must have | Core functionality. App is incomplete without this. | MVP |
| Should have | Important but not launch-blocking. | Phase 2 |
| Nice to have | Valuable addition for future iterations. | Phase 3 |

## Story summary

| ID | Title | Phase | Priority |
|----|-------|-------|----------|
| US-001 | Register a new account | MVP | Must have |
| US-002 | Log in with existing credentials | MVP | Must have |
| US-003 | Log out of the application | MVP | Must have |
| US-004 | Access demo account from login page | MVP | Must have |
| US-005 | Reset forgotten password | MVP | Must have |
| US-010 | Create a new job requisition record | MVP | Must have |
| US-011 | View a list of all my job requisitions | MVP | Must have |
| US-012 | View the detail of a single job requisition | MVP | Must have |
| US-013 | Edit an existing job requisition | MVP | Must have |
| US-014 | Update the status of a job requisition | MVP | Must have |
| US-015 | Store the job description text with a requisition | MVP | Must have |
| US-016 | Delete a job requisition | MVP | Must have |
| US-017 | Filter and search job requisitions by status or keyword | MVP | Should have |
| US-020 | Add a contact to a job requisition | MVP | Must have |
| US-021 | Capture contact role type for a job requisition | MVP | Must have |
| US-022 | Edit an existing contact record | MVP | Must have |
| US-023 | Associate a contact with multiple job requisitions | MVP | Should have |
| US-024 | View all contacts associated with a job requisition | MVP | Must have |
| US-030 | Log an activity entry against a job requisition | MVP | Must have |
| US-031 | Record raw message content in a journal entry | MVP | Must have |
| US-032 | Tag contacts involved in a journal entry | MVP | Must have |
| US-033 | View the full activity history for a job requisition | MVP | Must have |
| US-034 | Edit a journal entry | MVP | Should have |
| US-040 | Upload a resume version for a job requisition | MVP | Must have |
| US-041 | Add a change description when uploading a new resume version | MVP | Must have |
| US-042 | View the version history of resumes for a job requisition | MVP | Must have |
| US-043 | Download a specific resume version | MVP | Must have |
| US-050 | Receive an email reminder when no activity has been logged | Phase 2 | Should have |
| US-051 | Configure reminder types and day thresholds | Phase 2 | Should have |
| US-052 | Enable or disable individual reminder types | Phase 2 | Should have |
| US-053 | Receive a reminder when an application expiry date is approaching | Phase 2 | Should have |
| US-060 | Extract key requirements from a job description | Phase 2 | Should have |
| US-061 | Draft a recruiter outreach or follow-up message using AI | Phase 2 | Should have |
| US-064 | Generate a tailored resume using AI | MVP | Must have |
| US-062 | Score a job requisition against my target role criteria | Phase 2 | Nice to have |
| US-063 | Generate interview prep questions from a job description | Phase 2 | Nice to have |
| US-070 | View a pipeline dashboard showing requisitions by status | Phase 3 | Nice to have |
| US-071 | View activity trends over time | Phase 3 | Nice to have |
| US-072 | View response rate by source or role type | Phase 3 | Nice to have |

---

## Authentication and account management

These stories cover how users register, log in, and access the application. Authentication is handled by Auth0 — the application does not manage passwords or credentials directly.

### US-001

As a **new user**, I want to **register for an account**, so that **I can access the application with my own private data**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Registration page is accessible from the login screen
- User can register with an email address and password
- Email verification is sent upon successful registration
- Duplicate email addresses are rejected with a clear message
- After verification, user is redirected to the login page

### US-002

As a **registered user**, I want to **log in with my email and password**, so that **I can access my job tracking data securely**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Login page is the default landing page for unauthenticated users
- Successful login redirects to the job requisition dashboard
- Invalid credentials display a clear error message
- A JWT token is issued upon successful login and stored client-side
- All subsequent API requests include the JWT token in the Authorization header

### US-003

As a **logged-in user**, I want to **log out of the application**, so that **my session is ended and my data is protected**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A logout option is accessible from the main navigation
- Logging out clears the JWT token from the client
- User is redirected to the login page after logout
- Attempting to access a protected route after logout redirects to login

### US-004

As a **visitor**, I want to **log in using the demo account**, so that **I can explore the application without creating an account**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- The login page displays a visible note indicating demo credentials are available
- Demo account username and password are shown on the login page
- Demo account has pre-populated sample data across all features
- Demo account data is read-only or resets on a schedule to prevent abuse

### US-005

As a **registered user**, I want to **reset my forgotten password**, so that **I can regain access to my account without contacting support**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A forgot password link is visible on the login page
- Clicking the link prompts the user for their registered email address
- A password reset email is sent via Auth0
- The reset link expires after a configurable time period
- After resetting, the user can log in with the new password

---

## Job requisition management

These stories cover the core job tracking workflow — creating, viewing, updating, and managing job requisitions. This is the central domain of the application.

### US-010

As a **job seeker**, I want to **create a new job requisition record**, so that **I can start tracking a new opportunity from the moment I discover it**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A form allows entry of: found on URL, apply at URL, company name, role title, date discovered, application expiry date
- Job description text can be pasted directly into a rich text editor with formatting preserved
- All required fields are validated before saving
- On save, the new requisition appears in the dashboard list
- Status defaults to Discovered on creation

### US-011

As a **job seeker**, I want to **view a list of all my job requisitions**, so that **I can see all opportunities I am tracking at a glance**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Dashboard displays all requisitions belonging to the logged-in user
- Each row shows: company name, role title, status, date discovered, and date submitted
- List is sorted by date discovered descending by default
- No data from other users is ever visible

### US-012

As a **job seeker**, I want to **view the full detail of a single job requisition**, so that **I can review all information I have captured about an opportunity**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Clicking a requisition from the dashboard opens a detail view
- Detail view shows all fields, the job description, associated contacts, activity journal, and linked resume
- All sections are accessible from a single page

### US-013

As a **job seeker**, I want to **edit an existing job requisition**, so that **I can keep the information current as the opportunity evolves**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- An edit action is available from the detail view
- All fields captured during creation are editable
- Changes are saved and reflected immediately in both the detail view and dashboard list
- Editing does not reset the status or clear the activity journal

### US-014

As a **job seeker**, I want to **update the status of a job requisition**, so that **I can track where each opportunity is in my pipeline**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Status can be updated from the detail view or inline from the dashboard
- Available statuses are: Discovered, Applied, In Progress, Waiting on Response, Interview Scheduled, Offer Received, Closed, Withdrawn
- Status is visually distinct in the dashboard list

### US-015

As a **job seeker**, I want to **store the full job description text with a requisition**, so that **I have a reference copy if the original listing is removed**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Job description can be pasted into a rich text editor on the requisition form with formatting preserved
- Stored description is viewable from the detail page
- Description persists even if the source URL becomes unavailable

### US-016

As a **job seeker**, I want to **delete a job requisition**, so that **I can remove records that are no longer relevant**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A delete action is available from the detail view
- A confirmation prompt is shown before deletion
- Deletion removes the requisition and all associated journal entries, contact links, and resume versions
- The action cannot be undone

### US-017

As a **job seeker**, I want to **filter and search my job requisitions**, so that **I can quickly find specific opportunities without scrolling through the full list**.

**Priority:** Should have  
**Phase:** MVP

**Acceptance criteria:**
- Dashboard includes a search field that filters by company name or role title
- A status filter allows displaying only requisitions in a selected status
- Filters can be combined
- Clearing filters restores the full list

---

## Contact management

These stories cover capturing and managing the people associated with each job requisition, including agency recruiters, account managers, company recruiters, and hiring managers.

### US-020

As a **job seeker**, I want to **add a contact to a job requisition**, so that **I can track who I have been communicating with for each opportunity**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A contact can be added from within the job requisition detail view
- Contact fields include: name, email, LinkedIn URL, role type, and agency name if applicable
- Multiple contacts can be associated with a single requisition
- A requisition can have contacts of different role types simultaneously

### US-021

As a **job seeker**, I want to **assign a role type to each contact**, so that **I know at a glance who each person is and their relationship to the opportunity**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Role type is a required field when adding a contact
- Available role types are: Agency Recruiter, Agency Account Manager, Company Recruiter, Hiring Manager, Network Contact
- Role type is displayed alongside the contact name in all views

### US-022

As a **job seeker**, I want to **edit an existing contact record**, so that **I can correct or update contact details as I learn more**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- An edit action is available from the contact entry within a requisition
- All contact fields are editable
- Changes are reflected immediately

### US-023

As a **job seeker**, I want to **associate an existing contact with a new requisition**, so that **I do not have to re-enter contact details if the same person reaches out about a different role**.

**Priority:** Should have  
**Phase:** MVP

**Acceptance criteria:**
- When adding a contact, the user can search existing contacts by name or email
- Selecting an existing contact links them to the requisition without duplicating the record
- Each association can have its own role type independent of other requisitions

### US-024

As a **job seeker**, I want to **view all contacts associated with a job requisition**, so that **I have a clear picture of my network around each opportunity**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Contacts are displayed within the requisition detail view
- Each contact shows name, role type, email, LinkedIn URL, and agency if applicable
- LinkedIn URL is displayed as a clickable link

---

## Activity journal

These stories cover the journal system that tracks all activity against a job requisition over time — calls, emails, interviews, follow-ups, and any other touchpoints.

### US-030

As a **job seeker**, I want to **log an activity entry against a job requisition**, so that **I have a dated record of everything that has happened with each opportunity**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- An add entry action is available from within the requisition detail view
- Entry fields include: date, activity type, description of what transpired
- Activity types include: Application Submitted, Recruiter Call, Interview, Follow Up Sent, Offer Received, Rejection Received, Other
- Entry date defaults to today but can be changed
- Entries are displayed in reverse chronological order

### US-031

As a **job seeker**, I want to **record raw message content in a journal entry**, so that **I have a verbatim record of recruiter or hiring manager communications**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Each journal entry includes an optional notes field for pasting email, DM, or message content with formatting preserved
- Notes content is stored as HTML and displayed in a read-only block
- There is no character limit on the notes field

### US-032

As a **job seeker**, I want to **tag contacts involved in a journal entry**, so that **I can see which people were part of each interaction without searching separately**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- When creating a journal entry, the user can select one or more contacts from those associated with the requisition
- Selected contacts are displayed within the journal entry
- A contact must be associated with the requisition before they can be tagged in a journal entry

### US-033

As a **job seeker**, I want to **view the full activity history for a job requisition**, so that **I can review the complete timeline of an opportunity before any call or interview**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- Activity journal is displayed within the requisition detail view
- Entries are sorted in reverse chronological order by default
- Each entry shows: date, activity type, notes, and tagged contacts

### US-034

As a **job seeker**, I want to **edit a journal entry**, so that **I can correct errors or add detail to an entry after it was saved**.

**Priority:** Should have  
**Phase:** MVP

**Acceptance criteria:**
- An edit action is available on each journal entry
- All fields are editable after creation
- Changes are reflected immediately

---

## Resume management

These stories cover uploading, linking, and downloading resume files associated with specific job requisitions.

### US-040

As a **job seeker**, I want to **upload a resume**, so that **I have a library of resume versions to draw from**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A file upload control is available on the Resumes page
- Accepted file types are PDF and DOCX
- Upload date and file name are recorded automatically

### US-041

As a **job seeker**, I want to **link a resume to a job requisition**, so that **I have a record of exactly which resume was submitted for each opportunity**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A link resume action is available within the requisition detail view
- The user selects from their uploaded resumes
- Only one resume can be linked per requisition at a time
- The linked resume is displayed on the requisition detail page

### US-042

As a **job seeker**, I want to **view all resumes I have uploaded**, so that **I can manage my resume library**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A Resumes page displays all uploaded resumes
- Each entry shows file name and upload date
- Resumes can be deleted from this page

### US-043

As a **job seeker**, I want to **download a resume**, so that **I can retrieve any version for reference or reuse**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- A download action is available on each resume entry
- Clicking download initiates a file download
- The downloaded file name matches the original uploaded file name

---

## Notifications and reminders

These stories cover configurable email reminders that help the user stay on top of their job search activity. This is a Phase 2 feature enabled by the Kafka event stream and notification service.

### US-050

As a **job seeker**, I want to **receive an email reminder when I have not logged activity on a requisition**, so that **I do not let opportunities go cold without realizing it**.

**Priority:** Should have  
**Phase:** Phase 2

**Acceptance criteria:**
- The notification service monitors job requisitions for inactivity
- If no journal entry has been logged within the configured number of days, a reminder email is sent
- The email includes the company name, role title, and a link to the requisition
- Reminders are only sent for requisitions in active statuses such as Applied, In Progress, or Waiting on Response
- Closed or Withdrawn requisitions do not trigger reminders

### US-051

As a **job seeker**, I want to **configure reminder types and day thresholds in settings**, so that **I can control how frequently I am reminded based on my own job search cadence**.

**Priority:** Should have  
**Phase:** Phase 2

**Acceptance criteria:**
- A settings section allows the user to view all available reminder types
- Each reminder type has a configurable day threshold
- Changes to settings take effect for future reminders immediately
- Settings are stored per user and do not affect other users

### US-052

As a **job seeker**, I want to **enable or disable individual reminder types**, so that **I only receive the reminders that are relevant to how I manage my search**.

**Priority:** Should have  
**Phase:** Phase 2

**Acceptance criteria:**
- Each reminder type has an enable/disable toggle in settings
- Disabling a reminder type stops all future reminders of that type
- Re-enabling a reminder type resumes reminders from that point forward

### US-053

As a **job seeker**, I want to **receive a reminder when an application expiry date is approaching**, so that **I do not miss the deadline to submit an application**.

**Priority:** Should have  
**Phase:** Phase 2

**Acceptance criteria:**
- A reminder is sent when the application expiry date is within the configured threshold
- The reminder email includes the company name, role title, expiry date, and a link to the requisition
- The reminder is only sent once per requisition per threshold crossing
- If the requisition status is already Closed or Withdrawn, no reminder is sent

---

## AI-assisted features

These stories cover AI and LLM-powered features that help the user work more efficiently. All AI features are user-triggered via an explicit action — analysis is never fired automatically on save. This gives the user control over when content is ready for analysis and avoids processing incomplete drafts.

### US-064

As a **job seeker**, I want to **generate a tailored resume using AI**, so that **I can quickly produce a version of my resume optimized for a specific job description without starting from scratch**.

**Priority:** Must have  
**Phase:** MVP

**Acceptance criteria:**
- An AI tab is available on the job requisition detail view
- The user selects a work experience profile and an AI instruction profile before generating
- The system assembles a complete prompt server-side using the stored job description, experience document content, and AI profile instructions
- The assembled prompt is displayed in a copyable text area
- Step-by-step instructions guide the user through pasting the prompt into Claude.ai
- If the experience document is a plain text file, its content is embedded directly in the prompt
- If the experience document is a binary format (PDF/DOCX), the prompt instructs the user to attach it manually in Claude.ai
- The generate action is user-triggered — it never runs automatically

---

### US-060

As a **job seeker**, I want to **extract key requirements from a job description on demand**, so that **I can quickly understand what a role needs without reading the full description manually**.

**Priority:** Should have  
**Phase:** Phase 2

**Acceptance criteria:**
- An extract requirements button is available on any requisition that has a stored job description
- The user explicitly triggers the action — extraction does not run automatically on save
- The AI service parses the job description and returns a structured list of key requirements
- Results are displayed within the requisition detail view
- The user can dismiss or re-trigger the extraction

### US-061

As a **job seeker**, I want to **draft a recruiter outreach or follow-up message using AI**, so that **I can compose professional communications faster and with better quality**.

**Priority:** Should have  
**Phase:** Phase 2

**Acceptance criteria:**
- A draft message action is available within the activity journal section of a requisition
- The user selects a message type such as initial outreach, follow-up, or thank you
- The AI generates a draft based on the job description, company name, and role title
- The draft is displayed in an editable text area before sending or saving
- The user can regenerate the draft with a single action if the first result is not satisfactory

### US-062

As a **job seeker**, I want to **score a job requisition against my target role criteria**, so that **I can prioritize which opportunities to invest time in**.

**Priority:** Nice to have  
**Phase:** Phase 2

**Acceptance criteria:**
- User can define target role criteria in settings such as preferred title, industry, or tech stack
- A score action on a requisition sends the job description and user criteria to the AI service
- A match score and brief rationale are returned and displayed on the requisition detail
- Score is informational only and does not affect status or filtering

### US-063

As a **job seeker**, I want to **generate interview prep questions from a job description**, so that **I can prepare for interviews more effectively using the stored job description**.

**Priority:** Nice to have  
**Phase:** Phase 2

**Acceptance criteria:**
- A generate prep questions button is available on requisitions with a stored job description
- The AI returns a list of likely interview questions based on the role and requirements
- Questions are displayed within the requisition detail view
- The user can save the questions as a journal entry for reference

---

## Reporting and analytics

These stories cover dashboard and reporting features that give the user insight into their overall job search performance. This is a Phase 3 effort and lowest priority.

### US-070

As a **job seeker**, I want to **view a pipeline dashboard showing all requisitions by status**, so that **I can see the shape of my job search at a glance**.

**Priority:** Nice to have  
**Phase:** Phase 3

**Acceptance criteria:**
- A dashboard view shows a count of requisitions grouped by status
- A visual representation such as a kanban board or status chart is displayed
- Clicking a status group navigates to a filtered list of those requisitions

### US-071

As a **job seeker**, I want to **view my activity trends over time**, so that **I can see whether I am staying consistent with my job search efforts**.

**Priority:** Nice to have  
**Phase:** Phase 3

**Acceptance criteria:**
- A chart shows the number of journal entries logged per week or month
- The user can select a date range for the report
- Periods of inactivity are visually highlighted

### US-072

As a **job seeker**, I want to **view my response rate by source or role type**, so that **I can identify which sources and role types are generating the most traction**.

**Priority:** Nice to have  
**Phase:** Phase 3

**Acceptance criteria:**
- A report shows the percentage of applications that received a response grouped by source
- A second view groups the same data by target role type
- Data is based on journal entry activity relative to application submissions
