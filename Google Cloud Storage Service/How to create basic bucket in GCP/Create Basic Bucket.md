# Google Cloud Storage — Implementation
```
User
  │
  ▼
Google Cloud Console
  │
  ▼
Bucket Name (task-12345)
  │
  ├── Iamge.png
  ├── folder1/
  │      └── folder2/
  │             └── Iamge1.png
  │
  └── Public URL Access
 ```
## Task 1. Create a bucket
- In the Cloud console, go to **Navigation menu () > Cloud Storage > Buckets.**
- Click **-->Create**
- Name your bucket: Enter a **unique name** for your bucket.
- Choose Region for Location type and us-central1 for Location.
- Choose Standard for Set a default class.
- Choose Uniform for Access control and uncheck Enforce public access prevention on this bucket to turn it off.
  
<img width="1899" height="800" alt="image" src="https://github.com/user-attachments/assets/a5292359-d481-4899-88be-d5583e8bf7ee" />

## Task 2. Upload an object into the bucket
- In the Cloud Storage browser page, click the name of the bucket that you created.
- In the Objects tab, click Upload > Upload files.
- In the file dialog, go to the file that you downloaded and select it.
- Ensure the file is named kitten.png. If it is not, click the three dot icon for your file, select Rename from the dropdown, and rename the file to kitten.png.

<img width="1906" height="774" alt="image" src="https://github.com/user-attachments/assets/bce71f21-ffbd-460c-a650-ea20aee46d7f" />

## Task 3. Share a bucket publicly
- Click the Permissions tab above the list of files.
- Ensure the view is set to View by principals tab. Click Grant access to view the Add principals pane.
- In the New principals box, enter allUsers.
- In the Select a role drop-down, select Cloud Storage > Storage Object Viewer.
- Click Save.
- In the Are you sure you want to make this resource public? window, click Allow public access.
- To verify, click the Objects tab to return to the list of objects. Your object's Public access column should read Access granted to public principals.
- Press the Copy URL button for your object and paste it into a separate tab to view your image.
```
https://storage.googleapis.com/YOUR_BUCKET_NAME/Iamge.png
```
<img width="1902" height="781" alt="image" src="https://github.com/user-attachments/assets/8ba4a8c4-b709-4a74-b5cc-189f9d650d94" />

## Task 4. Create folders
- In the Objects tab, click Create folder.
- Enter folder1 for Name and click Create.
- Click folder1-->Click Create folder.
- Enter folder2 for Name and click Create.
- Click folder2.
- Click Upload > Upload files.

<img width="1913" height="821" alt="image" src="https://github.com/user-attachments/assets/349dd469-3e5a-431f-9e6c-7c8199e7d8c9" />

## Task 5. Delete a folder
- Click the arrow next to Bucket details to return to the buckets level.
- Select the bucket.
- Click on the Delete button.
- In the window that opens, type DELETE to confirm the deletion of the folder.
- Click Delete to permanently delete the folder and all objects and subfolders in it.
