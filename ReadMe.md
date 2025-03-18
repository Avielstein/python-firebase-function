# Setting Up Firebase Cloud Functions with Python for a React Native (Expo) App

This guide provides step-by-step instructions to set up and deploy Python-based Firebase Cloud Functions and integrate them with a React Native app using Expo.

## Prerequisites

Before starting, ensure you have the following installed and set up:

- **Node.js** (LTS version recommended)
- **Firebase CLI**, install it globally via:
  ```bash
  npm install -g firebase-tools
  ```
- **Python** (>=3.11 recommended)
- **Virtual Environment (venv)** for Python
- **React Native (Expo)** development environment, installed via:
  ```bash
  npm install -g expo-cli
  ```
- **A Firebase project set up**

## Step 1: Initialize Firebase Project

1. Go to the [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. Enable Cloud Firestore under "Build > Firestore Database," choosing **Native mode**.
3. Install Firebase CLI (if not already installed):
   ```bash
   npm install -g firebase-tools
   ```
4. Authenticate Firebase CLI with your Google account:
   ```bash
   firebase login
   ```
5. Create a directory for your Firebase functions:
   ```bash
   mkdir firebase-backend && cd firebase-backend
   ```
6. Initialize Firebase services in your project:
   ```bash
   firebase init
   ```
   - Select **Functions** when prompted.
   - Choose an existing Firebase project.
   - Choose **Python** as the programming language.
   - Install dependencies when prompted.

## Step 2: Set Up Your Python Environment

Navigate to the `functions` directory inside your Firebase project and set up a Python virtual environment:

```bash
cd functions
python -m venv venv
source venv/bin/activate  # On Windows use: venv\Scripts\activate
```

Install necessary dependencies:

```bash
pip install firebase-functions firebase-admin flask
```

## Step 3: Writing Your First Firebase Function

Open `functions/main.py` and add the following sample function:

```python
from firebase_functions import https_fn
from flask import jsonify
from datetime import datetime

@https_fn.on_request()
def get_server_time(request):
    current_time = datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')
    return jsonify({
        "message": "Hello from Firebase Cloud Functions!",
        "server_time": current_time
    })
```

## Step 4: Deploying the Cloud Function

Deploy your function to Firebase with:

```bash
firebase deploy --only functions
```

Once deployment is complete, Firebase provides a function URL, for example:

```bash
https://us-central1-YOUR_PROJECT_ID.cloudfunctions.net/get_server_time
```

Once you've deployed it you have to go to google cloud and set the security rules to allow unauthorized hits if you want to test it without auth

https://console.cloud.google.com/run

You can test the function by visiting the provided URL in your browser.

## Step 5: Testing Functions Locally Using Firebase Emulator

Firebase provides an emulator suite to test functions locally before deploying:

Start the emulator:

```bash
firebase emulators:start --only functions
```

The function will be available at a local URL such as:

```bash
http://127.0.0.1:5001/YOUR_PROJECT_ID/us-central1/get_server_time
```

Use this local endpoint for testing in your Expo app during development.

## Step 6: Integrating with React Native (Expo)

Install Firebase in your React Native project:

```bash
expo install firebase
```

Initialize Firebase in your app (`App.js`):

```javascript
import { initializeApp } from 'firebase/app';
import { getFunctions, httpsCallable } from 'firebase/functions';

const firebaseConfig = {
  apiKey: 'YOUR_API_KEY',
  authDomain: 'YOUR_PROJECT_ID.firebaseapp.com',
  projectId: 'YOUR_PROJECT_ID',
};

const app = initializeApp(firebaseConfig);
const functions = getFunctions(app);

const fetchTime = async () => {
  const getTime = httpsCallable(functions, 'get_server_time');
  try {
    const response = await getTime();
    console.log('Current Time:', response.data.server_time);
  } catch (error) {
    console.error('Error fetching time:', error);
  }
};

fetchTime();
```

## Step 7: Securing Firebase Cloud Functions

Set Firestore security rules to prevent unauthorized access. Navigate to the Firebase Console under Firestore > Rules and update them as follows:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

## Step 8: Debugging and Monitoring

If you encounter issues during deployment, try the following:

- Ensure the correct Firebase project is selected:

  ```bash
  firebase use --add
  ```

- Check for syntax errors in your function code.
- View Firebase function logs:

  ```bash
  firebase functions:log
  ```

- Run the emulator locally to check for issues before deploying.

## Step 9: Deploying Firebase to Production

Once you have verified the functions locally, deploy them to production with:

```bash
firebase deploy
```

## Conclusion

Congratulations! You have successfully:

- Set up Firebase Cloud Functions with Python.
- Integrated Firebase into a React Native (Expo) app.
- Tested functions locally using Firebase Emulator.

You can now extend your functions to include database interactions, authentication, and other Firebase services.
