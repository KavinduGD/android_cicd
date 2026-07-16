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
