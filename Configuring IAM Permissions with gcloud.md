### What is IAM?

Google Cloud offers Cloud Identity and Access Management (IAM), which lets you manage access control by defining who has what access for which resource.

- Google Account
- Service account
- Google group
- Google Workspace account
- Cloud Identity domain
- All authenticated users
- All users - [Google](https://cloud.google.com/iam/docs/overview#concepts_related_identity)

**Roles**
>  [!NOTE]
>  A role is a collection of permissions. You cannot assign a permission to the user directly; instead you grant them a role. When you grant a role to a user, you grant them all the permissions that the role contains. - [Google](https://cloud.google.com/iam/docs/overview#roles)
     
**Task 1. Configure the gcloud environment**
1. Open the list of compute instances by going to Navigation Menu > Compute Engine > VM instances.
  
2. On the line with the compute instance named centos-clean, click SSH.
  
3. First test, confirm that gcloud is successfully installed by checking the version. Inside the SSH session run.
   ```
   gcloud --version
   ```
 <ins>***Create a new instance and updating the default zone***</ins>
 
 After verification that gcloud command-line tool is installed , make some changes by creating a compute instance.
  1. First, authenticate in gcloud. Inside the SSH session, run.
     ```
     gcloud auth login
     Press ENTER when you see the prompt Do you want to continue (Y/n)?
     ```
  2. Navigate to the link displayed in a new tab.
    
  3. Click on your active username (), and click Allow.
    
  4. When you see the prompt Enter the following verification code in gcloud CLI on the machine you want to log into, click on the copy button then go back to the SSH session, and paste the code into the prompt Enter authorization code.
    
  5. In the SSH session, set the region and zone.
     ```
     gcloud config set compute/region "Region1"
     gcloud config set compute/zone "Zone1"
     ```
  6. Inside the SSH session run.
     ```
     gcloud compute instances create lab-1 --zone "Zone1" --machine-type=e2-standard-2
     ```
  7. Check your current gcloud configuration. Inside the SSH session run.
     ```
     gcloud config list
     ```
  8. Now list all the zones available to use by running the following inside the SSH session run.
     ```
     gcloud compute zones list
     ```
  9. Identify one of the other zones in the same region as you. For example, if your current zone is us-west2-a, you could select us-west2-b. 
    
 10. Change your current zone for another zone in the same region. Inside the SSH session run the following, replacing ZONE with the zone you selected.
      ```
      gcloud config set compute/zone ZONE
      ```
  11. Verify the zone change was made. Inside the SSH session run.
      ```
      gcloud config list
      You see the stated zone reflects the change you made.
      ```
> [!NOTE] 
> 1. You can change other settings using the gcloud config set command. Those changes are permanent; they are written to your home directory<br/>
> 2. The default configuration is stored in ~/.config/gcloud/configurations/config_default<br/>
> 3. If you want to use a zone other than the default zone when creating an instance, you can use --zone switch. For example, gcloud compute instances create lab-1 --zone us-central1-f<br/>
  
  **Task 2. Create and switch between multiple IAM configurations**
  
  - You have now set up one account. In situations when you need to work on different teams or access different accounts, you can also manage that with gcloud config.
  - In your next task you learn how to create a second configuration and switch between both of them.
    
  <ins>***Create a new IAM configuration***</ins>
  
  - This account has read-only (viewer) access to the first project. You create a new configuration for that user.
    
  1. Start a new gcloud configuration for the second user account. Inside the SSH session run.
       ```
       gcloud init --no-launch-browser
       ```
  2. Select option 2, Create a new configuration. 
    
  4. configuration name: Type user2. 
    
  5. Log in with a new account: select option 3 - you're logging in with the other provided user name.
    
  6. Press ENTER when you see the prompt Do you want to continue (Y/n)?
     
  7. Navigate to the link displayed in a new tab.
     
  8. Click Use another account --> Copy the second user account (), and paste into the email or phone prompt.
  
  9. Copy the same password that you started the lab with, and paste into the enter your password prompt.

 10. Click I understand--> Allow to
      
  - You are accepting that the Cloud SDK has the same access as your Google account.
    
 11. When you see the prompt Enter the following verification code in gcloud CLI on the machine you want to log into, click on the copy button then go back to the SSH session and paste the code into the prompt Enter authorization code.
 
 12.  For Pick cloud project to use: locate your current project () and then type in the number that corresponds to the project.
     
<ins>***Test the new account***</ins>
- This new account has viewer only access to the project, so you can test that you are indeed using this account by trying to view and then create some resources.

1. Check that you can view details in the first project. Inside the SSH session run.
   ```
   gcloud compute instances list
   ```
2. Check that you cannot create an instance in the first project, as your assigned role is basic viewer. Inside the SSH session run.
   ```
   gcloud compute instances create lab-2 --zone "Zone2" --machine-type=e2-standard-2
   Because the second user account has only viewer access, they are not allowed to create an instance, so this command fails. It takes a little time to fail.
   ```
3. Change back to your first user's configuration (default). Inside the SSH session run.
   ```
   gcloud config configurations activate default
   ```
**Task 3. Identify and assign correct IAM permissions**
- You have been provided two user accounts for this project. The first user has complete control of both projects and can be thought of as the admin account. The second user has viewer only access to the two projects. Call the second user a devops user and that user identity represents a typical devops level user.
  
<ins>***Examine roles and permissions***</ins>

1. To view all the roles, run the following inside the SSH session run.
   ```
   gcloud iam roles list | grep "name:"
   ```
2. Examine the compute.instanceAdmin predefined role. Inside the SSH session run - [Google](https://cloud.google.com/iam/docs/permissions-reference)
   ```
   gcloud iam roles describe roles/compute.instanceAdmin 
   ```
