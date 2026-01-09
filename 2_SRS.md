# Software Requirements Specification (SRS)
## Functionality Overview 
- User authentication & recovery.
- User stats & graphs (total backlog items, % of done, etc).
- Creation of backlogs with theme.
- Update backlog's theme.
- Delete backlog.
- Export backlog as JSON.
- Create backlog log (title, free textarea, rating out of 10).
- Update single backlog log.
- Export single backlog log as JSON.
- Delete single backlog log.
- Sort backlog logs (by creation date, rating, done/not done, etc).
- Filter backlog logs (search, rating, etc).
- Make backlog PUBLIC (opt in, requires confirmation).
- Fetch & view a random public backlog.
- View a public backlog via URL.
- View backlog's stats/graphs.

## Constraints
- API speed (DRF & lack of async support).
- Accesibility (NES css is a minimal framework).
- Exposed API & possible vulnerabilities.
- Self hosting ease (setup, security, etc.)

## Specific Requirements
1. SR-01: Account system
The default User model provided by Django will be used. The user API must provide the basic functionality to:
  - Create an account via email, username & password.
  - Log in via email & password.
  - Account recovery via email code.
  - Log out.
  - Account deletion with email confirmation.

2. SR-02: Backlog creation
There will be a 'Backlog' model, which must provide the following:
  - Creation: an authenticated user can create a Backlog instance, being directly related to it.
  - Toggle visibility: the Backlog's owner must be able to toggle a switch, requiring confirmation, to make it public/private. It should be private by default during Backlog creation.
  - Fetching: related to the visibility, a private backlog should be only accesible by its owner.
  - Deletion: after a method of confirmation (to be determined), the user should be able to delete a backlog from listings. However, it should be a soft delete at a database level by defect, a hard delete being 'opt-in'.

3. SR-03: Backlog log instances
A Backlog will contain a relation with many 'BacklogLog' model instances. These will:
  - Be able to be created, read, updated & deleted (soft delete, hard delete being 'opt-in') only if the creating user is the backlog's owner.
  - Contain the data: title, free text area, rating, status (pending, in progress, done) & estimated time.

4. SR-04: Navigation
All main screens should be accesible from the main navbar, including "Landing", "User Backlogs", "View Public Backlogs" & "Profile".
The screen to access public backlogs should be separated from the rest POIs (points of interest). 

5. SR-05: Stats & Graphs
Profiles & backlogs must provide a screen, component or view to display stats & graphs generated from said stats, using client-side rendering (via CSS, JS, etc.)

## Functional Requirements
1. FRQ-USR: User registry (functional).
   1. FRQ-USR-01: User registry.
    An unauthenticated user must be able to create an account providing an email, (optional) username & password. If no username is provided, it shall be automatically generated via Faker, or the provided email (to be determined).
    An email should be sent to the registered account's email, providing a link to confirm the entered address that expires in 10 minutes. It will be possible to confirm the user's email later on as well.
  2. FRQ-USR-02: User login.
    An unauthenticated user must be able to log in to the system by providing their account's email & password. Account's username is not involved in the process.
  3. FRQ-USR-03: Account recovery.
    1. FRQ-USR-03-A: User forgot their password.
      A user must be able to request an email to be send to their registered account's email, containing a one-time-use code that expires in the next 10 minutes after its generation for security purposes.
    2. FRQ-USR-03-B: User-email validation.
      A user that has completed FRQ-USR-03-A successfully shall be able to send the one-time-use to the system's server.
    3. FRQ-USR-03-C: Restore account password.
      A user that has successfully completed FRQ-USR-03-A and FRQ-USR-03-B will be able to send a new password to the system's server, updating it. This shall not log in the user.

2. FRQ-BKLG: Backlog functional requirements.
  1. FRQ-BKLG-01: Backlog creation.
    An authenticated & active (not banned) user will be able to create Backlog instances, providing a "Theme" and, optionally, toggling on an option to make it public. Said toggle should require confirmation such as a pop up, double-click confirmation or text input confirmation.
  2. FRQ-BKLG-02: Backlog theme edit.
    An active (not banned), Backlog's owner shall be able to edit their Backlogs themes. This should require confirmation via a pop up, double-click confirmation or text input confirmation
  3. FRQ-BKLG-03: Backlog visibility toggle.
    A Backlog's owner must be able to toggle its visibility, requiring confirmation via a pop up, double-click confirmation or text input confirmation.
    It is suggested to add some kind of functionality to fetch public/private backlogs for this.
  4. FRQ-BKLG-04: Backlog fetch, visibility validation.
    Shall a Backlog be requested, the system must validate that the requesting user is either: 
    a. The Backlog's owner.
    b. The Backlog is public.
    In the event of both failed verifications, the server should not send the Backlog's data.
  5. FRQ-BKLG-05: Safe Backlog deletion.
    An active user (not banned) shall be able to delete any their Backlogs. The deletion should be a soft delete, not deleting the Backlog from the database but rather populating a field that excludes it from being fetched.
    Optionally, the user shall be able to toggle an option to make a hard delete instead, requiring confirmation via a pop up, double click or text input confirmation.
  6. FRQ-BKLG-06: Backlog export.
    An accessible Backlog (meaning the user can fetch & view it) can be exported/downloaded as a JSON file. Other formats such as CSV, XML, and others might be considered.

3. FRQ-LOG: Backlog log (from now on refered as 'Log') functional requirements
  1. FRQ-LOG-01: Log creation.
    A backlog's owner shall be able to create Log instances for their Backlogs.
    Fields include "Title", "Free text area", "rating", "status" & "estimated time".
    Only the "Title" field is required to create a Log, as the rest of fields can be populated later on & the status will be set as "Pending" by default.
  2. FRQ-LOG-02: Log updating.
    A Log's owner will be able to update its fields. Shall any of the fields try to be updated with an unvalid value, the whole operation should fail.
  3. FRQ-LOG-03: Log deletion.
    A log's owner can delete it, this being a hard delete for simplicity.
  4. FRQ-LOG-04: Log exports.
    An accessible Log (meaning the user can fetch & view it) can be exported/downloaded as a JSON file. Other formats such as CSV, XML, and others might be considered.

4. FRQ-BKLGLOG: Functional requirements for Backlog & Backlog Logs
  1. FRQ-BKLGLOG-01: Log listing.
    A Backlog will list its related Logs.
    The default ordering shall be "Most recent first", using the Logs's "updated_at" field.
  2. FRQ-BKLGLOG-02: Log filtering & order.
    A Backlog Log listing can be filtered by "Status" and "Title" search.
    Ordering can be changed, using criteria such as "Alphabetically/Alphabetically (reversed)", "Recently Updated/Recently Updated (reversed)", "Estimated hours/Estimated hours(reversed)" & possibly further more.
  3. FRQ-BKLGLOG-03: Backlog deletion.
    Upon the event of a Backlog's hard delete, its Logs will cascade, meaning all Logs will be hard deleted.

## Non-functional Requirements
1. NFR-USR: Users (non functional).
  1. NFR-USR-01: User access to a Backlog.
    An authenticated & active user should be able to open one of their existing Backlogs from any screen, in less than 5 (directed to the task) clicks.
  2. NFR-USR-02: Point of Interest's Reachability.
    The frontend's main Point of Interests should have clear and visible access components, such as buttons with sufficient contrast & colored text. 
  3. NFR-USR-03: Errors feedback.
    All errors, no matter their origin (frontend or backend) should be notified to the user in a clear way, such as notifications.
  4. NFR-USR-04: i18n compliance.
    Not required for the backend/API. The frontend will comply with i18n using "react-i18next". English and spanish translations will be initially served. Future voluntary translation might be considered.
  5. NFR-USR-05: Loading feedback.
    All actions that require loading time, such as API calls & others, will have "Loading" indicators, them being Loading messages, Custom loading animations, Loading spinners and more.
  6. NFR-USR-06: WCAG 2.1 AA Compliance.
    The frontend should comply with WCAG 2.1 at an AA level. This includes being able to be navigated via tab & enter key presses, and screen readers following the flow of the screen correctly.

2. NFR-BKLG: Backlogs (non functional).
  1. NFR-BKLG-01: Backlog creation timing.
    A Backlog creation should not take a (no use of scren readers, no visibility problems, etc.) User more than 1 minute. This includes being already authenticated, sending the creation request & recieving feedback for the action.

3. NFR-FRNT: Frontend theme.
  The platorm's frontend shall have a color theme option (light/dark) that can be toggled at user's preference. Default choice should be fetched from the user system's configuration. 


## Appendices
### Frontend CSS Framework choice change
For the record: originally, the CSS framework "NES.css" was planned to be used. However, it's too minimal and it's got a "game-ish" look and feel. Material UI was chosen as a "profesional" alternative.

### i18n Translations future plan
Once the platform's "done", and it's gone open-source, I'd like to make a repository for translations. Have I forgotten to do so? Feel free to open an issue, thanks.
