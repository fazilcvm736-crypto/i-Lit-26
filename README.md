rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    match /users/{uid} {
      allow read: if request.auth != null && request.auth.uid == uid;
      allow create: if request.auth != null && request.auth.uid == uid
                    && request.resource.data.role == 'leader';
      allow update: if isAdmin();
    }
    match /fest/{docId} {
      allow read: if request.auth != null;
      allow write: if isAdmin();
    }
  }
}
