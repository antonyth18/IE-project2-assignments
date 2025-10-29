## Enhanced Proposal: File Versioning and Recovery Module

This module enhances the website's functionality by providing a vital safety net for user data, moving beyond simple storage to offering robust data integrity and disaster recovery.

### 1. Module Description and Core Features

The primary goal is to maintain an immutable history of every file's state, preventing permanent loss from user error or corruption.

| Feature | Description | Benefit |
| :--- | :--- | :--- |
| **Automatic Version Capture** | A new version is created automatically every time a user uploads a file with the same name/identifier as an existing file. The old version is archived, not overwritten. | Ensures no change is ever permanently lost. |
| **History View** | A new interface displays a chronological list of all versions of a selected file, including the date, time, and file size for each snapshot. | Provides a clear audit trail and ability to pinpoint a specific point in time for recovery. |
| **Instant Restore** | Allows the user to select any archived version and promote it to the **current active version** with a single click. The restore operation is non-destructive (it creates a new version based on the old one). | Fast recovery from mistakes without having to manually download and re-upload an old file. |
| **Metadata Comparison** | For advanced use, show the metadata difference between two versions (e.g., "Size changed from 5MB to 8MB," "Modified on X date"). | Helps users quickly identify the correct version to restore. |

---

### 2. Technical Integration and Requirements

Implementing this feature requires coordinated changes across both the backend storage logic and the database structure.

#### Backend (Storage and Encryption)

1.  **Storage Path Modification:** The current storage path must be extended to include a version identifier.
    * **Old Path Example:** cloud/user_id/file_id/data.enc
    * **New Path Example:** cloud/user_id/file_id/v_UUID/data.enc
    * The UUID ensures that every single file snapshot, regardless of the user's friendly version number (v1, v2), is uniquely addressable in the cloud storage.
2.  **Encryption:** Encryption and decryption will remain unchanged, but they will be applied consistently to *each* stored version snapshot.
3.  **File Download Logic:** The download route must be updated to accept an optional `version_id`. If no `version_id` is provided, it defaults to retrieving the file linked to the database's `latest_version_id`.

#### Database Schema Update

A new `FileVersion` collection (or table) would be required, linked to the main `File` record:

| Field | Type | Purpose |
| :--- | :--- | :--- |
| `_id` | ObjectId | Primary key for the specific version. |
| `parentFileId` | ObjectId | Reference to the main file entry in the Dashboard table. |
| `versionNumber` | Number | User-friendly version (e.g., 1, 2, 3). |
| `cloudPath` | String | The unique, full path to the encrypted snapshot in the cloud. |
| `isCurrent` | Boolean | Flag to identify the currently active version. |
| `uploadedAt` | Date | Timestamp for when this specific version was uploaded. |

#### Frontend (Dashboard and UI)

* **Dashboard Row:** Add a "Version History" icon (<V /> or clock icon) next to the Download button on the main dashboard file listing.
* **Modal View:** Implement a modal component that fetches the list of versions for a specific `parentFileId` and displays the version history table with the **Restore** and **Download** actions.

---

### 3. User Benefits and Security Enhancement

| Benefit Category | Impact |
| :--- | :--- |
| **Data Integrity** | Eliminates the risk of permanent data loss due to accidental overwrites or file corruption, boosting user confidence in the service as a reliable long-term storage solution. |
| **Compliance/Auditing**| Provides a historical record of file changes, which is valuable for professional users who may need to demonstrate the state of a document at a specific point in time. |
| **User Experience (UX)**| Simplifies recovery. Instead of a cumbersome process of locating a backup, users get instant, self-service recovery directly within the file dashboard. |

This module transforms the application from a simple storage locker into a robust, professional data management system.