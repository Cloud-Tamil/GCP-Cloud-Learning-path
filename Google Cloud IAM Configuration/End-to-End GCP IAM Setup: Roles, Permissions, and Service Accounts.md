### 🔐 What is IAM GCP?

Identity and Access Management in Google Cloud Platform is a service that helps you control who identity can do what role/permission on which resource - [Google](https://cloud.google.com/iam/docs/overview#concepts_related_identity)

### 🔐 What is Roles in GCP?

A role is a collection of permissions. You cannot assign a permission to the user directly; instead you grant them a role. When you grant a role to a user, you grant them all the permissions that the role contains. - [Google](https://cloud.google.com/iam/docs/overview#roles)

### 🔐 What is a Permission in GCP?

A permission in GCP is a single action that can be performed on a resource.
     
### Task 1. Configure the gcloud environment.
1. Open the list of compute instances by going to Navigation Menu **> Compute Engine > VM instances**.
  
2. On the line with the compute instance named **centos-clean, click SSH**.
  
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
    
  3. Click on your active **username (), and click Allow**.
    
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
  
### Task 2. Create and switch between multiple IAM configurations.
  
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
### Task 3. Identify and assign correct IAM permissions.
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
<ins>***Grant access to the second user to the second project***</ins>
- Next you attach the basic role of "viewer" to the second user onto the second project.
   - To the user and an organization
   - To a user and a project

1. Switch gcloud configuration back to the second user (user2). Inside the SSH session run.
   ```
   gcloud config configurations activate user2
   ```
2. Now you're back to user2 --> Set PROJECTID2 to the second project. Inside the SSH session, run the following.
   ```
   echo "export PROJECTID2="PROJECT_ID"" >> ~/.bashrc

   . ~/.bashrc
   gcloud config set project $PROJECTID2
   ```
3. When prompted, Do you want to continue (Y/n)?, type N and press ENTER.
    - This means that user 2 doesn't have access to the PROJECTID2 project, which you fix in the next section.
      
 <ins>***Assign the viewer role to the second user in the second project***</ins>
 1. Switch back to the default gcloud configuration, which has the permission to grant access to the second user. Inside the SSH session run.
    ```
    gcloud config configurations activate default
    ```
2. Install jq.
   ```
   sudo yum -y install epel-release
   sudo yum -y install jq
   ```
3. Inside the SSH session, run the following.
   ```
   echo "export USERID2="Username2"" >> ~/.bashrc
   . ~/.bashrc
   gcloud projects add-iam-policy-binding $PROJECTID2 --member user:$USERID2 --role=roles/viewer
   ```
### Task 4. Test that user2 has access.
1. Switch your gcloud configuration to user2. Inside the SSH session run.
   ```
   gcloud config configurations activate user2
   ```
2. Change the configuration for user2 to the second project. Inside the SSH session run.
   ```
   gcloud config set project $PROJECTID2
   ```
3. Verify you have viewer access. Inside the SSH session run.
   ```
   gcloud compute instances list
   ```
4. Try to create an instance in the second project as the second user. Inside the SSH session run.
   ```
   gcloud compute instances create lab-2 --zone "Zone2" --machine-type=e2-standard-2
   ```
5. Switch your gcloud configuration to default. Inside the SSH session run.
   ```
   gcloud config configurations activate default
   ```
<ins>***Create a new role with permissions***</ins>
- Next, create the new role with the set of permissions needed for the devops team.
1. Create a custom role called devops that has the permissions to create an instance. Inside the SSH session run.
   ```
   gcloud iam roles create devops --project $PROJECTID2 --permissions        compute.instances.create,compute.instances.delete,compute.instances.start,compute.instances.stop,compute.instances.update,compute.disks.create,compute.subnetworks.use,compute.subnetworks.useExternalIp,compute.instances.setMetadata,compute.instances.setServiceAccount"
   ```
   - The full name of the role is listed, note the role is in the project so the path is in the pattern of projects/PROJECT/roles/ROLENAME.
     
<ins>Bind the role to the second account to both projects</ins>
- You now have the role created and need to bind the user and the role to the project. Use gcloud projects add-iam-policy-binding to perform the binding. To make this command easier to execute, set a couple of environment variables first; the project id and the user account.
1. Bind the role of iam.serviceAccountUser to the second user onto the second project. Inside the SSH session run.
   ```
   gcloud projects add-iam-policy-binding $PROJECTID2 --member user:$USERID2 --role=roles/iam.serviceAccountUser
   ```
2. Bind the custom role devops to the second user onto the second project. You can find the second user account on the left of this page. Make sure you set USERID to the second user account.
   ```
   Inside the SSH session run
   gcloud projects add-iam-policy-binding $PROJECTID2 --member user:$USERID2 --role=projects/$PROJECTID2/roles/devops
   ```
<ins>Test the newly assigned permissions.</ins>
1. Switch your gcloud configuration to user2. Inside the SSH session run.
   ```
   gcloud config configurations activate user2
   ```
2. Try to create an instance called lab-2. Inside the SSH session run.
   ```
   gcloud compute instances create lab-2 --zone "Zone2" --machine-type=e2-standard-2
   ```
3. Verify the instance exists. Inside the SSH session run.
   ```
   gcloud compute instances list
   ```
   <img width="806" height="536" alt="image" src="https://github.com/user-attachments/assets/257b360c-2232-4060-92b7-782b9e8e2f99" />

### Task 5. Using a service account
- A Service Account in Google Cloud Platform (GCP) is a special type of identity that is used by applications, virtual machines, or services to interact with Google Cloud resources securely. Unlike normal Google accounts that represent human users, a service account represents a non-human entity. It allows automated systems to authenticate and perform actions without requiring a person to log in each time<br/>
In simple terms, if a user account is for a person, a service account is for a machine.
[google](https://cloud.google.com/iam/docs/service-accounts)

<ins>***Create a service account***</ins>
1. Switch your gcloud configuration to default, user2 doesn't have the rights to set up and configure service accounts. Inside the SSH session run.
   ```
   gcloud config configurations activate default
   ```
2. Set the project to PROJECTID2 in your configuration. Inside the SSH session run.
   ```
   gcloud config set project $PROJECTID2
   ```
3. Create the service account. Inside the SSH session run.
   ```
   gcloud iam service-accounts create devops --display-name devops
   ```
4. Get the service account email address. Inside the SSH session run.
   ```
   gcloud iam service-accounts list  --filter "displayName=devops"
   ```
5. Put the email address into a local variable called SA. Inside the SSH session run.
   ```
   SA=$(gcloud iam service-accounts list --format="value(email)" --filter "displayName=devops")
   ```
6. Give the service account the role of iam.serviceAccountUser. Inside the SSH session run.
   ```
   gcloud projects add-iam-policy-binding $PROJECTID2 --member serviceAccount:$SA --role=roles/iam.serviceAccountUser
   ```
### Task 6. Using the service account with a compute instance.
1. Give the service account the role of compute.instanceAdmin. Inside the SSH session run.
   ```
   gcloud projects add-iam-policy-binding $PROJECTID2 --member serviceAccount:$SA --role=roles/compute.instanceAdmin
   ```
2. Create an instance with the devops service account attached. You also have to specify an access scope that defines the API calls that the instance can make. Inside the SSH session run.
   ```
   gcloud compute instances create lab-3 --zone "Zone2" --machine-type=e2-standard-2 --service-account $SA --scopes "https://www.googleapis.com/auth/compute"
   ```
### Task 7. Test the service account.
1. Connect to the newly created instance using gcloud compute ssh. Inside the SSH session run.
   ```
   gcloud compute ssh lab-3 --zone "Zone2"
   ```
2. Press ENTER when asked if you want to continue --> Press ENTER twice to skip making a password.
3. The default image used already contains gcloud configuration. Inside the SSH session run.
   ```
   gcloud config list
   ```
4. Create an instance. This tests that you have the necessary permissions via the service account.
   ```
   gcloud compute instances create lab-4 --zone "Zone2" --machine-type=e2-standard-2
   ```
5. Check roles attached are working. Inside the SSH session run.
   ```
   gcloud compute instances list
   ```
   <img width="801" height="534" alt="image" src="https://github.com/user-attachments/assets/4ee59b7d-8202-482f-b925-2f314b292300" />
