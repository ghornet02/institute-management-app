# Institute Management App

Android app for running day-to-day institute work: logging in by role, browsing students, and building the weekly timetable. Written in Kotlin with Jetpack Compose and Material 3.

## Features

- **Role-based login** for Academic Office, Faculty and Student accounts, with each screen gated by role
- **Dashboard** with a navigation drawer, quick actions and a search overlay
- **Student directory** with keyword search and filters for batch, scholarship, course, stream and CGPA
- **Student profiles** with permission checks (students can only open their own)
- **Timetable builder**:
  - create, rename and delete timetables
  - add and edit course templates
  - drag courses from the unallocated list onto the weekly grid
  - blocks a drop that double-books a room or a faculty member, with a message naming the clash
  - warns when a subject or a faculty member goes over their weekly limit
- **Local persistence**: users, students and timetables are saved as JSON in app storage and seeded with demo data on first run

## Tech stack

- Kotlin 2.2, Android Gradle Plugin 9.1
- Jetpack Compose (BOM 2026.02) with Material 3 and Navigation Compose
- minSdk 24, targetSdk 35, JDK 11

## Project structure

```
app/src/main/java/com/example/ims/
├── MainActivity.kt          entry point, sets the Compose content
├── core/
│   ├── AppDataStore.kt      data models, JSON load/save, seed data
│   └── MockAuth.kt          user profile and role models
└── ui/
    ├── IMSApp.kt            root composable: session, routing, role checks
    ├── navigation/          destination enum
    ├── screens/
    │   ├── login/
    │   ├── dashboard/
    │   ├── studentsearch/   directory, filters, profile
    │   └── timetable/       grid, drag and drop, clash and workload checks
    └── theme/
```

`IMSApp` loads the saved data (or seeds it), keeps track of the signed-in user and the current screen, and refuses to open screens the user's role can't see. Every change goes back through `AppDataStore`.

## Access rules

| Role | Dashboard | Timetable | Student directory | Profiles |
|---|---|---|---|---|
| Academic Office | yes | create and edit | yes | any student |
| Faculty | yes | view | no | no |
| Student | yes | view the main timetable | no | own profile |

## Running it

Open the project in Android Studio and run the `app` configuration on an emulator or a device. From the command line:

```sh
./gradlew assembleDebug        # build
./gradlew installDebug         # install on a connected device
./gradlew test                 # unit tests
```

Demo accounts (username / password):

| Role | Accounts |
|---|---|
| Academic Office | `admin` / `admin` |
| Faculty | `faculty1` / `faculty1` ... `faculty3` / `faculty3` |
| Student | `student1` / `student1` ... `student3` / `student3` |

The login screen also has quick-fill buttons for each role.

## Design

The UI mockups are in [`docs/ui`](docs/ui). The class responsibilities and sequence diagrams are in [`docs/design`](docs/design).

A few design decisions:

- **Consistency.** The top bar and the side menus are shared composables (`NavigationMenuPanel`, `RightMenuPanel`), so the dashboard, timetable and search screens all have the same layout, colours and type.
- **Feedback.** While you drag a course, a ghost card follows your finger and the cells you can drop on turn blue. Opening search blurs and dims the screen behind it.
- **Error handling.** `hasConflict` stops a drop that would double-book a room or a faculty member and explains why, for example "Room conflict: B-201 is already occupied at this time". `computeWorkloadAlerts` shows a badge when a limit is exceeded rather than blocking the change.

## Authors

Built jointly by [ghornet02](https://github.com/ghornet02) and [Shardul-Kholam](https://github.com/Shardul-Kholam), who contributed equally to the design and implementation.

## Limitations

Authentication is local and uses plain demo credentials, and all data lives in a JSON file on the device. Moving to a real backend, Room for storage, and ViewModels for state would be the next steps.
