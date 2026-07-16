A **service account** is a special type of account that represents an **application or automated process**, rather than a person.

Think of it like this:

- **User account** → Used by a human.
  - Example: `kavindu@gmail.com`
  - Used to sign in to Firebase Console or Google Cloud Console.

- **Service account** → Used by software.
  - Example: `firebase-distribution-ci@my-project.iam.gserviceaccount.com`
  - Used by Azure DevOps, GitHub Actions, Jenkins, or other automated systems.

### Why does it exist?

An automated pipeline cannot open a browser and log in as a person every time it runs. Instead, it uses a service account to authenticate securely.

For example:

```text
Azure DevOps Pipeline
        │
        │ Uses JSON key
        ▼
Service Account
        │
        │ Has permission to upload builds
        ▼
Firebase App Distribution
```

The **JSON file** contains credentials (a private key and information about the service account) that allow the pipeline to prove its identity to Google.

### Permissions

A service account has **only the permissions you grant it**.

For example, you might allow it to:

- ✅ Upload APKs to Firebase App Distribution.
- ✅ Access Cloud Storage.
- ❌ Delete Firebase projects (unless you explicitly grant that permission).

This follows the **principle of least privilege**, meaning you give it only the access it needs.

### Why is it better than a user account?

Suppose your company has a developer named Alice.

If the pipeline uses Alice's account:

- Alice leaves the company.
- Her Google account is disabled.
- The pipeline stops working.

If the pipeline uses a service account:

- Alice can leave.
- Bob can join.
- The pipeline continues to work because it uses the project's service account, not a person's account.

In short, a **service account is a non-human identity created for applications and automation**. It lets tools like Azure DevOps securely access Google Cloud and Firebase services without depending on an individual user's Google account.

---

### Why we use a Service Account JSON instead of a Firebase CLI Token

We use a **Firebase Service Account JSON key** because it is intended for CI/CD automation and is **not tied to an individual user's Google account**. The service account belongs to the Firebase/Google Cloud project, so the Azure DevOps pipeline continues to work even if team members leave or their accounts are removed.

In contrast, a **Firebase CLI token** is generated from a specific user's Google account (`firebase login:ci`) and uses that user's permissions. If the user's account is revoked, deleted, or loses access to the project, the pipeline may stop working.

For these reasons, the **Service Account JSON** provides better security, stability, and maintainability, making it the recommended authentication method for Azure DevOps pipelines.

---

A **service account is a Google Cloud Platform (GCP) feature**, not a Firebase-specific feature.

Firebase is built on top of Google Cloud, so when you create or manage a Firebase project, there is an underlying Google Cloud project.

The relationship looks like this:

```text
Google Cloud Project
│
├── Service Accounts   ← GCP feature
├── IAM (Permissions)
├── Cloud Storage
├── Cloud Functions
└── ...
     │
     └── Firebase Project
         ├── App Distribution
         ├── Authentication
         ├── Firestore
         ├── Hosting
         └── ...
```

### Why do you use a GCP service account for Firebase?

Firebase services trust Google Cloud's authentication system.

So when Azure DevOps uploads an APK to **Firebase App Distribution**, it authenticates using a **GCP service account**.

The flow is:

```text
Azure DevOps
      │
      │ JSON Key
      ▼
Google Cloud Service Account
      │
      │ Authenticated by Google
      ▼
Firebase App Distribution
```

### Key point

- **Service Account** → Created and managed in **Google Cloud IAM**.
- **Permissions** → Granted through **Google Cloud IAM**.
- **Used to access** → Firebase services (App Distribution, Firestore, Storage, etc.) and other GCP services.

So, although you use it to access Firebase, the service account itself belongs to **Google Cloud (GCP)**, because every Firebase project is backed by a Google Cloud project.

---

You're already in the correct place. The **Firebase Console** doesn't let you create service accounts directly—it links you to **Google Cloud IAM**, because service accounts are a GCP feature.

### Method 1 (Recommended): From the Firebase Console

In your second screenshot:

1. Go to **Project Settings** → **Service accounts**.

2. Click **Manage service account permissions** (top right).

3. This opens **Google Cloud IAM**.

4. In the left menu, click **Service Accounts** (not **IAM**).

5. Click **+ Create Service Account**.

6. Fill in:
   - **Service account name:** `azure-devops-firebase`
   - **Service account ID:** (auto-generated)
   - **Description:** `Used by Azure DevOps to distribute Android builds`

7. Click **Create and Continue**.

8. Grant the required role(s). For Firebase App Distribution, you can start with:
   - **Firebase App Distribution Admin**

   (If this role isn't available, you can temporarily use **Editor** for testing, but avoid it in production because it grants broad access.)

9. Click **Done**.

---

### Create the JSON key

After the service account is created:

1. Open **IAM & Admin → Service Accounts**.
2. Click your new service account.
3. Go to the **Keys** tab.
4. Click **Add Key** → **Create new key**.
5. Choose **JSON**.
6. Click **Create**.

The JSON file will be downloaded automatically.

Upload this JSON file to **Azure DevOps → Pipelines → Library → Secure Files**.

---

### Your flow will be

```text
Firebase Project
       │
       ▼
Google Cloud Project
       │
       ▼
Create Service Account
       │
       ▼
Generate JSON Key
       │
       ▼
Upload JSON to Azure DevOps Secure Files
       │
       ▼
Pipeline authenticates using the service account
       │
       ▼
Firebase App Distribution
```
