# User Role Provisioning and Permission Management System
 - ### Google Cloud Identity and Access Management (IAM) service lets you create and manage permissions for Google Cloud resources. Cloud IAM unifies access control for Google Cloud services into a single system and provides a consistent set of operations.
## *What we are ganna learn about today*
 - How permissions work in Google Cloud
 - How to give access to another user
 - How to remove access
 - How IAM roles control what users can do

### We will use 2 Users
| User       | Role          |
| ---------- | ------------- |
| Username 1 | Project Owner (Admin)|
| Username 2 | Viewer (Read Only)       |

### What we will Build
1.Login with 2 accounts

2.Create a Cloud Storage bucket

3.Upload a file

4.Check whether User 2 can view it

5.Remove access

6.Add limited storage access only

## Step 1 - Login with Two Users
- Two separate users are used to demonstrate IAM role behavior.
   - Username 1
     - Project Owner
     - Full administrative access
     - Can manage IAM policies
     - Can create resources
   - Username 2
     - Initially assigned Viewer role
     - Read-only access
     - Cannot modify IAM policies
## Step 2 - Create a Cloud Storage bucket
