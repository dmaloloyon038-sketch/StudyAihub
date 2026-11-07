# StudyHub AI

## Architecture
Static HTML/CSS/vanilla JS (ES modules) -> Firebase Auth + Firestore + Storage (client SDK, modular v10)
-> Cloud Functions (Node 20) as the ONLY path to the AI provider. Secrets live in Functions secrets, never in the frontend.

## Firestore schema (all under users/{uid}; every doc has ownerId, createdAt, updatedAt)
- users/{uid}: displayName, email, settings{theme,notifications,pomodoro}
- subjects: name, color, description
- notes: subjectId, title, content, topics[], tags[], pinned, attachments[{path,name,type,size}]
- activities: subjectId, title, description, deadline, priority(low|medium|high), status(not_started|in_progress|completed)
- events: subjectId?, title, type(deadline|exam|study|other), start, end
- flashcards: subjectId, front, back, status(new|review|mastered)
- quizzes: subjectId, title, difficulty, questions[], results[{score,total,takenAt}]
- studySessions: subjectId?, plannedMinutes, actualMinutes, completed, startedAt
- conversations: subjectId?, messages written by backend only
Relations use IDs (subjectId), never duplicated content.

## Security strategy
1. Firestore/Storage rules: default deny, request.auth.uid == userId, field/type validation, immutable ownerId (Phase 2).
2. AI via Cloud Functions: verify ID token, validate and size-limit input, per-user quota, no logging of note content (Phase 6).
3. App Check (reCAPTCHA v3) on Firestore, Storage, Functions.
4. Safe DOM: textContent / escapeHTML only for user content.
5. Auth guard is UX; rules are the real enforcement.

## Roadmap
1 Design system + auth + nav (done) | 2 Rules + config | 3 Dashboard + subjects | 4 Notes + uploads
5 Activities + calendar | 6 Secure AI | 7 Flashcards, quizzes, planner | 8 Analytics, settings, polish

## Test in Acode
1. Create the Firebase project (below), paste config into js/firebase-config.js.
2. Open the folder in Acode, open login.html, tap the Run (play) button: Acode serves it at http://localhost:8080.
   ES modules do not work from file://, so always use the Run button.
3. Firebase Console > Authentication > Sign-in method: enable Email/Password. Authentication > Settings > Authorized domains: localhost is there by default.
4. Firestore Database > Create database (production mode). Until Phase 2 rules are deployed, registration's profile write will be denied: that is the safe default.
