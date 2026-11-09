# ali-games

Tiny browser games with saved player names and per-game leaderboards.

## Enable the global leaderboards

The games work without setup, but scores are then saved only in the current browser. To share scores globally:

1. Create a Firebase project at [Firebase Console](https://console.firebase.google.com/) and register a **Web app**.
2. In **Authentication → Sign-in method**, enable **Anonymous**. Create a Firestore database.
3. Paste the web app's Firebase config into `FIREBASE_CONFIG` in `index.html`.
4. In **Firestore → Rules**, publish these rules:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function owns(uid) {
      return request.auth != null && request.auth.uid == uid;
    }

    match /players/{uid} {
      allow read: if owns(uid);
      allow create, update: if owns(uid)
        && request.resource.data.keys().hasAll(['displayName', 'updatedAt'])
        && request.resource.data.keys().hasOnly(['displayName', 'updatedAt'])
        && request.resource.data.displayName is string
        && request.resource.data.displayName.size() > 0
        && request.resource.data.displayName.size() <= 20;
    }

    match /leaderboards/{game}/scores/{uid} {
      allow read: if true;
      allow create, update: if owns(uid)
        && game in ['snake', 'memory', 'dodge', 'ttt', 'breakout', 'click']
        && request.resource.data.keys().hasAll(['uid', 'displayName', 'score', 'updatedAt'])
        && request.resource.data.keys().hasOnly(['uid', 'displayName', 'score', 'updatedAt'])
        && request.resource.data.uid == request.auth.uid
        && request.resource.data.displayName is string
        && request.resource.data.displayName.size() > 0
        && request.resource.data.displayName.size() <= 20
        && request.resource.data.score is number
        && request.resource.data.score >= 0
        && request.resource.data.score <= 1000000;
    }
  }
}
```

Each browser gets an anonymous Firebase account; only that account can write its own name and score documents. Scores are submitted by the browser, so this is suitable for a casual leaderboard, not a cheat-proof competition.
