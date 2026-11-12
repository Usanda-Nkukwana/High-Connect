# High Connect LMS - setup

## 1. Create the Flutter project shell
```bash
flutter create high_connect          # only to get android/ios/web folders
# copy this folder's lib/, pubspec.yaml, analysis_options.yaml, firestore.rules, firebase.json over it
# DELETE any old lib files not listed below (teacher.dart, old person.dart)
flutter pub get
```

## 2. Connect Firebase
```bash
dart pub global activate flutterfire_cli
flutterfire configure           # creates lib/firebase_options.dart
```
In the Firebase console:
1. **Authentication > Sign-in method**: enable **Email/Password**.
2. **Firestore Database**: create a database.
3. Deploy the rules: `firebase deploy --only firestore:rules` (or paste `firestore.rules` into the Rules tab).

## 3. Create the first administrator (one-time)
Admins cannot self-register.
1. Authentication > Users > **Add user** (email + password). Copy the **User UID**.
2. Firestore > collection `users` > document id = that UID, fields:
   `email` (string, lowercase), `role` = `admin`, `status` = `active`,
   `firstName`, `surname` (strings), `gender` = `` , `phone` = `` , `staffNumber` = `` ,
   `grade` (null), `courseIds` (array, empty).
3. Run the app, choose **Admin**, sign in, then on Overview press **Load standard subjects**.

## 4. Run
`flutter run` (Android, iOS or `-d chrome`).

## Files
| File | Purpose |
|---|---|
| main.dart | Firebase init, theme, `AuthGate` (routes by role/status) |
| auth_service.dart | Sign-in, registration, admin-created users, profile stream |
| lms_data.dart | `LmsStore`: live Firestore data + all admin writes |
| models.dart | AppUser, Course, Schedule, Announcement, Resource |
| subject_catalog.dart | South African subject list + grade ranges |
| sign_in.dart / register.dart / password.dart | Auth screens |
| admin.dart, schedules.dart, lecturer.dart, student.dart, subject.dart, person.dart | Admin area |
| portal.dart, common_pages.dart | Student + lecturer area, timetable, news, profile, subject page |
| shell.dart, theme.dart | Navigation shell, shared theme/widgets/helpers |
| firestore.rules | Security rules (role-based, 7-subject limit, unique ID number) |

## Firestore data model (replaces schema.sql)
- `users/{uid}` role, status, names, grade, staffNumber, `courseIds[]` (student_courses + lecturer_courses)
- `users/{uid}/private/profile` national ID + date of birth (owner/admin only)
- `nationalIds/{id}` one doc per ID number (enforces UNIQUE)
- `courses/{id}` code, name, category, minGrade, maxGrade; `courses/{id}/resources/*` learning content
- `schedules/{id}` course, lecturer, weekday, start/end minutes, venue, academicYear
- `announcements/{id}` title, body, courseId (null = everyone), author

## Known limits
- Deleting a person removes their profile/data, but the Firebase **Auth login** must be deleted in the console (or a Cloud Function).
- ID validation uses the Luhn check; set `kStrictIdCheck = false` in theme.dart to accept any 13 digits while testing.
- Announcements are filtered per user in the app, not in the rules.
- Enrollment is one list per student (current year); `academicYear` is kept on timetable entries.
- Not compiled here (no Flutter SDK in my sandbox): run `flutter analyze` first and send me any errors.
