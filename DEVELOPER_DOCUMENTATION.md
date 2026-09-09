# Light Weight Baby Developer Documentation

## 1. Scope And Current State

Light Weight Baby is currently a client-side React/Vite prototype. The application is implemented in two source files:

- `src/main.jsx`: application state, screens, forms, calculations, persistence, and export behavior.
- `src/styles.css`: all visual styling and responsive layout rules.

There is no server, API, database driver, authentication provider, router, test suite, or backend service in this repository. Data is stored in the browser with `localStorage`. The Markdown files describe the intended product and domain model; they are not implemented backend contracts.

The application can be run with:

```powershell
npm install
npm run dev
```

The production build is created with:

```powershell
npm run build
```

## 2. Project Structure

```text
Gym app/
|-- src/
|   |-- main.jsx
|   `-- styles.css
|-- index.html
|-- package.json
|-- package-lock.json
|-- 1. Vision and Features.md
|-- 2. Entities.md
|-- 3. UI.md
|-- 4. Work Flows.md
|-- Built in Exercises (No estimates).txt
|-- Built in Exercises and Estimated Levels in Body Weight Factors.xlsx
`-- DEVELOPER_DOCUMENTATION.md
```

### Entry points

- `index.html` provides the HTML document and the `#root` mount point.
- `src/main.jsx` imports React, Lucide icons, the stylesheet, and mounts `<App />` with `createRoot`.
- `package.json` defines the Vite development, build, and preview scripts.

### Planning and source data

- `1. Vision and Features.md` defines the product goals and requested capabilities.
- `2. Entities.md` defines the intended domain entities and relationships.
- `3. UI.md` defines the intended tabs, analytics sections, and profile sections.
- `4. Work Flows.md` defines the intended user workflows.
- `Built in Exercises (No estimates).txt` is represented partially by the built-in exercise seed list in `src/main.jsx`.
- The supplied estimated-level spreadsheet is not parsed or used by the application.

## 3. Application Architecture

`App` in `src/main.jsx` is the root component and owns nearly all application state. It selects one of three top-level tabs:

- `Sessions`
- `Analytics`
- `Profile`

The desktop sidebar and mobile bottom navigation both call the same `setTab` state setter. There is no URL routing; refreshing the browser always returns to the default Sessions tab.

The root component also owns:

- Built-in and custom exercises.
- Workout templates.
- Workout sessions.
- Profile information.
- The active modal or full-screen session flow.
- The selected historical session.
- Toast notifications.

The implementation is intentionally compact, but the result is a large single module. Component functions are defined below `App` in the same file rather than split into feature folders.

## 4. Main Features And Flows

### 4.1 Sessions

Relevant code:

- `App` renders `SessionsView` for the Sessions tab.
- `SessionsView` displays summary metrics, session history, and workout plans.
- `SessionRow` makes an old session clickable and keyboard accessible.
- `SessionDetail` displays and edits an existing session.
- `SessionStartFlow` implements the new-session choice screen and the exercise tracking screen.
- `ExercisePicker` provides searchable exercise selection.
- `readActiveSessionDetails` collects the current session form values before saving.
- `calculateSessionVolume` calculates volume as the sum of `weight * reps` for all sets.

Flow for creating a session:

1. The user clicks `Start New Session` in `SessionsView`.
2. `SessionStartFlow` first displays stored workout templates and `Custom workout`.
3. A template preloads its exercise IDs and default set counts. A custom workout starts with an unselected `Choose Exercise` row.
4. `ExercisePicker` lets the user search and select an exercise.
5. Each exercise has a set count plus Weight and Reps inputs for each set.
6. The user can add exercises and start the rest timer.
7. On save, `saveSession` generates an ID and timestamp, reads set details, calculates volume, and adds the session to React state.
8. A `useEffect` writes the session list to `localStorage`.
9. The session appears in the history list.

Historical session flow:

1. The user clicks a `SessionRow`.
2. `SessionDetail` loads stored `session.details`, or creates blank fallback rows for old records.
3. The user can edit the session name, exercise selections, set counts, Weight, Reps, save, or delete.
4. Saving recalculates volume and replaces the matching session in state.
5. Deleting requires browser confirmation and removes the record from state and storage.

Session storage key:

```text
light-weight-baby-sessions-v8
```

Current session shape is approximately:

```js
{
  id: number,
  createdAt: number,
  date: string,
  name: string,
  exercises: number,
  sets: number,
  volume: string,
  details: [
    {
      exerciseId: number,
      sets: [
        { weight: string, reps: string }
      ]
    }
  ]
}
```

### 4.2 Workout templates

Relevant code:

- `ProfileView` selects the `Workouts` subview.
- `Workouts` lists templates and exposes Create and Edit actions.
- `WorkoutModal` creates or edits a template.
- `saveTemplate` inserts a new template or replaces an existing template by ID.

The workflow in `4. Work Flows.md` is mostly implemented:

- Create a workout from Profile -> Workouts.
- Enter a name.
- Select exercises with `ExercisePicker`.
- Set default set counts.
- Save.
- Edit loads the selected template and updates it without creating a duplicate.

Template storage key:

```text
light-weight-baby-templates-v3
```

Template shape:

```js
{
  id: number,
  name: string,
  color: string,
  exercises: [
    { exerciseId: number, sets: number }
  ]
}
```

There is no template delete action and no ability to create an exercise directly inside the workout editor. The session flow can open the Create Exercise modal, but that modal currently replaces the session screen rather than returning its new exercise to the in-progress form.

### 4.3 Exercises

Relevant code:

- `exercisesSeed` in `src/main.jsx` contains the built-in catalog.
- `ExerciseModal` implements custom exercise creation.
- `Exercises` displays the Profile exercise library.
- `ExercisePicker` searches by exercise name and muscle group.
- `ExerciseAnalytics` displays exercise-level analytics.

The create-exercise workflow is implemented:

1. Profile -> Exercises.
2. Click Create an exercise.
3. Enter name.
4. Select primary muscle.
5. Enter optional secondary muscle text.
6. Save.

Custom exercise storage key:

```text
light-weight-baby-exercises-v3
```

Built-in exercise records have an ID, name, muscle, best value, and color field. Exercise colors are no longer displayed in the UI, but color properties remain in the data objects and are still used by some workout template visual styling.

The supplied exercise text file contains more exercises than the current seed array. The spreadsheet containing estimated level factors is not loaded.

### 4.4 Profile

Relevant code:

- `ProfileView` provides the Profile subnavigation.
- `PersonalInformation` saves name and gender.
- `Workouts` manages templates.
- `Exercises` manages custom exercises.
- `SettingsView` is currently display-oriented.

Profile storage key:

```text
light-weight-baby-profile-v3
```

Profile shape:

```js
{
  name: string,
  gender: string
}
```

Weight is intentionally not editable in Profile. `PersonalInformation` reads the latest entry from the Analytics weight storage and displays it as read-only.

### 4.5 Weight entries

Relevant code:

- `WeightView` in `src/main.jsx`.
- Analytics -> Weight opens the entry control.
- Profile only displays the latest value.

Weight storage key:

```text
light-weight-baby-weight-v2
```

Weight shape:

```js
{
  id: number,
  weight: string,
  date: string
}
```

Weight entries are persisted in `WeightView` and displayed as separated value/date rows. They are not connected to a graph yet; the current view is a list with an empty state when no entries exist.

### 4.6 Analytics

`AnalyticsView` selects these subviews from the list defined in `3. UI.md`:

- Overview
- Weight
- Exercises
- Distribution
- Performance
- Personal Records
- Export

#### Overview

`Overview` currently displays empty or limited summary states. The weekly streak is calculated by `workoutWeekStreak`, which groups session timestamps by Monday-based calendar week and counts consecutive weeks backward from the current week.

The calendar is currently an empty-state presentation. It does not yet highlight real workout dates or link a calendar day to a session.

#### Exercises

`ExerciseAnalytics` now aggregates stored session details. For a selected exercise it calculates:

- Number of recorded sets.
- Total volume.
- Best Weight/Reps record.
- Epley estimated 1 REP MAX using `weight * (1 + reps / 30)`.
- Date associated with the best estimated record.

The view does not yet draw historical line graphs or calculate a strength classification from the spreadsheet factors.

#### Distribution

`Distribution` aggregates valid recorded sets by the exercise's primary muscle group. It supports:

- Percentage of sets done.
- Power distribution, using Epley-style estimated strength contribution.

It displays an ordered list rather than the pie chart specified by `3. UI.md`. Bodyweight sets with a zero weight are currently excluded because the aggregation requires both Weight and Reps to be truthy.

#### Performance

`Performance` aggregates:

- Total volume across sessions.
- Average session volume.
- A list of session volume by date.

It does not yet render a graph. Volume strings are parsed after removing comma separators.

#### Personal Records

`PersonalRecords` calculates the best recorded Weight/Reps pair for every exercise with saved data. It displays exercise name, weight, reps, and date. It does not yet implement the seven configurable canonical PR slots described in `3. UI.md`.

#### Export

`ExportView` supports JSON, CSV, and Excel (`.xlsx`) file generation using the `xlsx` package. It exports stored session summary rows.

The time-period selector is currently visual only; the selected period does not filter exported data. Export is client-side and does not call an API.

## 5. Application Flow And Data Access

There are no database or API calls. The actual flow is:

```text
User action
  -> React component event handler
  -> App state setter
  -> derived calculation or local component state
  -> localStorage useEffect
  -> UI rerender
```

Examples:

### Save a session

```text
SessionStartFlow form submit
  -> saveSession
  -> readActiveSessionDetails
  -> calculateSessionVolume
  -> setSessions
  -> localStorage key light-weight-baby-sessions-v8
  -> SessionsView / AnalyticsView rerender
```

### Edit a session

```text
SessionDetail form submit
  -> calculateVolume
  -> App onSave callback
  -> replace matching session by ID
  -> localStorage effect
  -> return to Sessions history
```

### Create a workout

```text
WorkoutModal form submit
  -> saveTemplate
  -> add or replace template by ID
  -> localStorage key light-weight-baby-templates-v3
  -> Profile and session choice screens rerender
```

### Export

```text
ExportView button click
  -> map sessions to export rows
  -> JSON.stringify, CSV construction, or XLSX workbook creation
  -> Blob and temporary download link
  -> browser download
```

## 6. Intended Models And Actual Models

The intended entity model is documented in `2. Entities.md` and includes:

- User
- User Weight
- Exercise
- Workout Template
- Workout Template Exercise
- Workout Session
- Workout Exercises
- Workout Exercise Sets

The current implementation only has browser objects corresponding approximately to:

- User/Profile: `profile` state and local storage.
- User Weight: `WeightView` entries and local storage.
- Exercise: built-in and custom exercise arrays.
- Workout Template: `templates` array.
- Workout Session: `sessions` array.
- Workout Exercise and Sets: nested `session.details` records.

There are no primary keys enforced by a database, foreign keys, migrations, constraints, transactions, or server-side validation. IDs are generated with `Date.now()` for newly created records; built-in exercise IDs are array-based constants.

## 7. Authentication And Authorization

Authentication is not implemented.

There is:

- No login or registration screen.
- No password handling.
- No token or session-cookie handling.
- No user identity lookup.
- No authorization checks.
- No multi-user isolation.

All browser users of the same local browser profile can access the same local data. Anyone with access to the browser's developer tools can read or modify the stored values. This is acceptable for a local prototype but not for production or sensitive fitness data.

## 8. External Dependencies And Integrations

Declared in `package.json`:

- `react`: component rendering and state.
- `react-dom`: React DOM mounting.
- `lucide-react`: interface icons.
- `vite`: development server and production bundling.
- `@vitejs/plugin-react`: declared build tooling dependency.
- `xlsx`: client-side Excel workbook creation.

External runtime integration:

- Google Fonts are imported in `src/styles.css` for Manrope and DM Mono.
- No REST, GraphQL, Firebase, Supabase, payment, calendar, analytics, or authentication integration exists.
- Browser `localStorage`, `Blob`, `URL.createObjectURL`, and download links provide persistence/export capabilities.

## 9. Business Rules

### Implemented rules

- A session's volume is the sum of `weight * reps` for all saved sets.
- Session weekly streaks count consecutive Monday-based calendar weeks with at least one session.
- A custom session starts with an unselected exercise and displays `Choose Exercise`.
- Template sessions preload configured exercises and default set counts.
- Epley estimated 1 REP MAX is `weight * (1 + reps / 30)`.
- A session can be edited or deleted after creation.
- Weight is entered only in Analytics and displayed read-only in Profile.
- Exercise analytics are derived from stored session details.
- Export formats are JSON, CSV, and XLSX.

### Planned but not implemented

From `1. Vision and Features.md`, `3. UI.md`, and `4. Work Flows.md`:

- Previous workout comparison.
- Actual rest timer integration into completed-set flow beyond the visible countdown control.
- PR progression history.
- Strength classifications: Beginner, Intermediate, Advanced, Elite.
- Graphs for volume, sets, 1RM, weight, and consistency.
- Clickable calendar days linked to sessions.
- Seven canonical power-distribution PR slots.
- User-configurable workout colors with calendar linking.
- Full data export date filtering.
- Backend persistence and iPhone packaging.

### Memberships and payments

No membership, booking, payment, subscription, or class-booking concepts exist in the planning files or source code. They are not part of the current domain model.

## 10. Potential Bugs And Risks

### High priority

1. **No backend or authentication**
   - All data is local to one browser profile.
   - Data is not synced between devices and is not protected.
   - Clearing site data permanently deletes the records.

2. **Session save reads the DOM directly**
   - `readActiveSessionDetails` uses `document.querySelectorAll` instead of React state.
   - This is fragile if markup changes, if multiple session screens exist, or if a picker label does not match an exercise name exactly.
   - A React-controlled form model would be safer and easier to test.

3. **Custom exercise creation can interrupt an active session**
   - The session flow opens the exercise modal by replacing the current modal state.
   - There is no callback that adds the newly created exercise to the active session row and returns the user to the session with their unsaved inputs preserved.

4. **No validation for missing exercise selection**
   - A custom session can be saved while a row still has `Choose Exercise`.
   - The DOM reader filters out unselected rows, which can silently reduce the saved exercise count.

### Medium priority

5. **Local storage parsing is unguarded**
   - `JSON.parse` is used directly for profile, exercises, templates, sessions, and weight data.
   - Corrupt or manually edited storage causes the app to fail during initialization.

6. **Local storage writes are not transactional**
   - Multiple components write separate keys independently.
   - There is no schema migration or recovery strategy when keys change between versions.

7. **Analytics ignores bodyweight sets**
   - Distribution and exercise calculations require a nonzero Weight and Reps.
   - Bodyweight exercises can be valid with Weight `0`, but are currently excluded.

8. **Workout color state is partly vestigial**
   - Exercise color fields remain in the data even though exercise colors were removed from the UI.
   - Template colors are only visual and are not tied to calendar days.

9. **Export time period is not applied**
   - The user can select a time range, but `ExportView` exports all sessions.

10. **The app relies on browser `confirm`**
    - Delete confirmation uses `window.confirm`, which is functional but inconsistent with the custom UI and awkward on mobile.

### Low priority

11. **Date formatting depends on browser locale**
    - `toLocaleDateString()` can produce different formats by device and locale.

12. **`AnalyticsView` summary metrics are still partly static**
    - Overview contains hardcoded zero/empty values for some metrics instead of deriving all values from sessions.

13. **No routing or deep linking**
    - Analytics subviews, Profile subviews, and session details cannot be opened directly by URL.

14. **No automated tests**
    - Behavior has been validated manually in the browser, but there are no unit, integration, or end-to-end test files.

## 11. Dead Code And Unused Code

- `SessionStartModal` remains defined in `src/main.jsx`, but the app renders `SessionStartFlow` instead. `SessionStartModal` is dead code and should be removed.
- `Legend` remains defined but is no longer used after Distribution changed from a legend/donut presentation to a calculated list.
- `History`, `Activity`, and some other imported icons should be checked against current JSX usage; unused imports add noise and may trigger lint failures if linting is introduced.
- `BarChart3`, `CalendarDays`, and the other icon imports are used by the current UI; the unused list should be confirmed with an ESLint rule rather than manually maintained.
- `.bar-chart` and related chart styles remain even though `Chart` now renders an empty state rather than bars.
- Several older modal styles, including `.template-list`, `.start-modal`, and `.workout-modal`, may be unused after the SessionStartFlow redesign. They should be removed after a visual regression check.
- The old planning data in the Excel file is not consumed by code.

## 12. Technical Debt And Refactoring Opportunities

### Suggested near-term refactor

Split `src/main.jsx` into feature modules:

```text
src/
|-- app/
|   |-- App.jsx
|   |-- storage.js
|   `-- calculations.js
|-- features/
|   |-- sessions/
|   |   |-- SessionList.jsx
|   |   |-- SessionDetail.jsx
|   |   `-- SessionStartFlow.jsx
|   |-- analytics/
|   |   |-- AnalyticsView.jsx
|   |   |-- ExerciseAnalytics.jsx
|   |   `-- calculations.js
|   |-- profile/
|   |-- exercises/
|   `-- workouts/
|-- data/
|   `-- builtInExercises.js
`-- styles/
    `-- styles.css
```

### Storage layer

Create a small storage adapter with:

- Namespaced keys.
- Safe JSON parsing.
- Versioned migrations.
- Centralized read/write functions.
- A way to reset local demo data intentionally.

### Domain model

Use consistent numeric types for Weight, Reps, volume, and estimated 1RM. Avoid storing formatted strings such as `"1,140 kg"` as the source of truth. Store numeric `volumeKg` and format only in the UI.

### Forms

Replace DOM scraping in `readActiveSessionDetails` with controlled React state. This will fix the biggest source of silent data loss and make validation possible.

### Analytics

Centralize calculations into pure functions and test them with data fixtures:

- `calculateSessionVolume(details)`
- `calculateExerciseStats(sessions, exerciseId)`
- `calculateDistribution(sessions, exercises)`
- `calculatePersonalRecords(sessions, exercises)`
- `calculateWeeklyStreak(sessions)`

### Testing

Add:

- Unit tests for calculations and storage migrations.
- Component tests for each form.
- End-to-end tests for the workflows in `4. Work Flows.md`.
- A test fixture containing two sessions with different exercises and sets.

### Production architecture

For a real iPhone application, move data to an authenticated backend or a local database layer with synchronization. The current localStorage architecture is useful for prototyping but does not provide backup, multi-device access, conflict handling, or security.

## 13. Recommended Implementation Order

1. Remove dead components and unused CSS/imports.
2. Introduce controlled session form state.
3. Add safe, centralized storage with migrations.
4. Finish data-driven Overview, calendar, and exercise graphs.
5. Implement configurable PR slots and strength classifications using the supplied spreadsheet.
6. Apply export time filters and store numeric source values separately from display strings.
7. Add automated tests for all workflows.
8. Choose and implement authentication plus a real persistence backend.
9. Package the validated web experience for iPhone.
