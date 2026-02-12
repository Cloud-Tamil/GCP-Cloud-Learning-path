### What is IAM?

Google Cloud offers Cloud Identity and Access Management (IAM), which lets you manage access control by defining who has what access for which resource.

- Google Account
- Service account
- Google group
- Google Workspace account
- Cloud Identity domain
- All authenticated users
- All users
  - [Google](https://cloud.google.com/iam/docs/overview#concepts_related_identity)

**Roles**
>  [!NOTE]
>  A role is a collection of permissions. You cannot assign a permission to the user directly; instead you grant them a role. When you grant a role to a user, you grant them all the permissions that the role contains.
  - [Google](https://cloud.google.com/iam/docs/overview#roles)
     
**Task 1. Configure the gcloud environment**
1.Open the list of compute instances by going to Navigation Menu > Compute Engine > VM instances.
2.On the line with the compute instance named centos-clean, click SSH.
3.First test, confirm that gcloud is successfully installed by checking the version. Inside the SSH session run.
   ```
   gcloud --version
   ```
> **Create a new instance and updating the default zone**
- After verification that gcloud command-line tool is installed , make some changes by creating a compute instance.
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
     ```
     > [!NOTE]
     > You see the stated zone reflects the change you made.

     > You can change other settings using the gcloud config set command. Those changes are permanent; they are written to your home directory.

     > The default configuration is stored in ~/.config/gcloud/configurations/config_default.

     > If you want to use a zone other than the default zone when creating an instance, you can use --zone switch. For example, gcloud compute instances create lab-1 --zone us-central1-f
