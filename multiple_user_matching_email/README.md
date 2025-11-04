### **How to Run an AAP Multiple Users with Matching Email Diagnostic Playbook**

New user association logic that exists within AAP 2.6 and may affect customers that had a previous implementation of AAP across 2.4 and 2.5 that allowed multiple user accounts that shared the same email address. As described within our [documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/access_management_and_authentication/gw-configure-authentication#user-association-and-attr-sync), we are using verified email addresses from an IdP source during SSO to match to any existing user accounts in order to consolidate user access to matching users. The motivation of that change was to match the behavior seen with Controller 2.4 giving platform administrators less ambiguity in how users are created and matched.
For customers on 2.6, you may now encounter an error when multiple users are found with a matching email addresses and this playbook will help identify those users.

---

### **Prerequisites**

Before you begin, please ensure you have the following information:

1.  **Repository URL:** `https://github.com/ansible-content-lab/product_knowledge_base_examples/`
2.  **Playbook Name:** `multiple_user_matching_email/duplicate_users_csv.yml`
3.  **Permissions:** Administrative access to your AAP Automation Execution UI.
4.  **Credential:** A credential using the AAP Platform credential type

---

### **Step 1: Create a New Project**

First, you need to create a Project in AAP to pull the diagnostic playbook from the Git repository.

1.  Log in to your Ansible Automation Platform Controller.
2.  In the left-hand navigation menu, go to **Resources** > **Projects**.
3.  Click the **Add** button.
4.  Fill in the form:
    * **Name:** `AAP Multiple Matching Users by Email`
    * **Organization:** Select your default organization.
    * **Source Control Type:** Select `Git`.
    * **Source Control URL:** `https://github.com/ansible-content-lab/product_knowledge_base_examples/`
    * **Source Control Branch/Tag/Ref:** Leave as-is (it will default to `main`).
    * **Source Control Credential:** Leave blank; this repository is public.
5.  Click **Save**.
6.  After saving, find your new project in the list and click the **Sync** icon (circular arrows) to download the project files. Wait for the sync to complete successfully.

---

### **Step 2: Create a Job Template**

Next, create a Job Template to define *how* to run the playbook. This job will run against `localhost` (the AAP instance itself) to test its own outbound connections.

1.  In the left-hand navigation menu, go to **Resources** > **Templates**.
2.  Click the **Add** button and select **Add Job Template**.
3.  Fill in the form:
    * **Name:** `Run Duplicate User Check`
    * **Job Type:** `Run`
    * **Inventory:** Select `Demo Inventory` (this inventory includes `localhost`).
    * **Project:** Select the project you created in Step 1.
    * **Playbook:** Select `duplicate_users_csv.yml` from the dropdown.
    * **Limit:** Type `localhost` into this field. This is **critical** as it ensures the job runs on the AAP instance itself.
    * **Credentials:** Select your existing AAP Platform credential
    * **Extra Variable:** Add the extra variable output_format and set the default to either 'csv' or 'json' (select Prompt on Launch if you would like to control the output per run)
4.  Click **Save**.

---

### **Step 3: Launch the Job**

Now you are ready to run the diagnostic test.

1.  Go back to the **Templates** page.
2.  Find the template you just created.
3.  Click the **Launch** icon (rocket ship) to run the job.

* **Next Steps:**
    1.  Review the job output log. The playbook is designed to show *which* users in the system have a record with at least 2 matching email.
        * **Note:** Depending on how you retrieve the csv output from the job logs, you may need to remove extraneous characters for the output to be readable in a text file
    3.  Use the output to decide on which steps to take when consolidating the user records as per the article
    4.  Note: Blank email address will show as duplicates as well and can be ignored
