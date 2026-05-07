# User Role Provisioning and Permission Management System
 - ### Google Cloud Identity and Access Management (IAM) service lets you create and manage permissions for Google Cloud resources. Cloud IAM unifies access control for Google Cloud services into a single system and provides a consistent set of operations.
## *What we are ganna learn about today*
 - Assign a role to a second user
 - Remove assigned roles associated with Cloud IAM

### We will use 2 Users
| User       | Role          |
| ---------- | ------------- |
| Username 1 | Project Owner (Admin)|
| Username 2 | Viewer (Read Only)       |

## Task 1. Explore the IAM console and project level roles
- Return to the **Username 1** Cloud Console page.
- Select **Navigation menu > IAM & Admin > IAM**. You are now in the "IAM & Admin" console.
- Click **+GRANT ACCESS** button at the top of the page.
   - Editor
   - Owner
   - Viewer
 - Now switch to the **Username 2** console.
 - Navigate to the IAM & Admin console, select **Navigation menu > IAM & Admin > IAM**.
    
## Task 2. Prepare a Cloud Storage bucket for access testing.
**Create a bucket**
- Create a Cloud Storage bucket with a unique name. From the Cloud Console, select **Navigation menu > Cloud Storage > Buckets**.
- Click **+CREATE-->CREATE-->CONFIRM**.
**Upload a sample file**
- On the Bucket Details page click **UPLOAD FILES**.
- Browse your computer to find a file to use. Any text or html file will do.
- Click on the three dots at the end of the line containing the file and click **Rename**.
- Rename the file **‘sample.txt'**.
- Click **RENAME**.

## Task 3. Remove project access.
