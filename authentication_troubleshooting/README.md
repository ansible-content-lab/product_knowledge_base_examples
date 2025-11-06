### **How to Run the AAP User Authentication Troubleshooting Data Collection Playbook**

This playbook collects comprehensive API data from Ansible Automation Platform (AAP) to troubleshoot user authentication issues, particularly duplicate user login problems after certificate updates or upgrades.

Based on troubleshooting requirements from past issues, this playbook gathers all necessary API information requested by Red Hat Support and Engineering teams to diagnose user authentication conflicts including:
- Users unable to login after SSL certificate updates
- Duplicate key constraint violations for users
- User conflicts during resource sync operations
- LDAP authentication issues in AAP 2.5+ containerized environments

---

### **Prerequisites**

Before you begin, please ensure you have the following information:

1. **Repository URL:** `https://github.com/ansible-content-lab/product_knowledge_base_examples/`
2. **Playbook Name:** `authentication_troubleshooting/aap_user_troubleshooting_data_collection.yml`
3. **Permissions:** Administrative access to your AAP Automation Execution UI
4. **Credential:** A credential using the AAP Platform credential type

---

### **Step 1: Create a New Project**

First, you need to create a Project in AAP to pull the diagnostic playbook from the Git repository.

1. Log in to your Ansible Automation Platform Controller.
2. In the left-hand navigation menu, go to **Resources** > **Projects**.
3. Click the **Add** button.
4. Fill in the form:
   * **Name:** `AAP User Authentication Troubleshooting`
   * **Organization:** Select your default organization.
   * **Source Control Type:** Select `Git`.
   * **Source Control URL:** `https://github.com/ansible-content-lab/product_knowledge_base_examples/`
   * **Source Control Branch/Tag/Ref:** Leave as-is (it will default to `main`).
   * **Source Control Credential:** Leave blank; this repository is public.
5. Click **Save**.
6. After saving, find your new project in the list and click the **Sync** icon (circular arrows) to download the project files. Wait for the sync to complete successfully.

---

### **Step 2: Create a Job Template**

Next, create a Job Template to define *how* to run the playbook. This job will run against `localhost` (the AAP instance itself) to collect authentication data.

1. In the left-hand navigation menu, go to **Resources** > **Templates**.
2. Click the **Add** button and select **Add Job Template**.
3. Fill in the form:
   * **Name:** `AAP User Authentication Troubleshooting Data Collection`
   * **Job Type:** `Run`
   * **Inventory:** Select `Demo Inventory` (this inventory includes `localhost`).
   * **Project:** Select the project you created in Step 1 (`AAP User Authentication Troubleshooting`).
   * **Playbook:** Select `authentication_troubleshooting/aap_user_troubleshooting_data_collection.yml` from the dropdown.
   * **Limit:** Type `localhost` into this field. This is **critical** as it ensures the job runs on the AAP instance itself.
   * **Credentials:** Select your existing AAP Platform credential
   * **Extra Variables:**
     - Add the extra variable `save_to_file` and set the default to `false` (select **Prompt on Launch** if you would like to control file saving per run)
     - Optionally add `target_users` as a list to investigate specific users only (select **Prompt on Launch** for flexibility)
4. Click **Save**.

---

### **Step 3: Launch the Job**

Now you are ready to run the diagnostic collection.

1. Go back to the **Templates** page.
2. Find the template you just created.
3. Click the **Launch** icon (rocket ship) to run the job.

#### **Optional: Specify Target Users**

If you only want to collect data for specific users experiencing issues:
1. Click the **Launch** icon
2. In the **Extra Variables** section, add:
   ```yaml
   target_users:
     - gazzellic
     - akbarm
     - inyaj
   ```
3. Click **Next** and then **Launch**

#### **Optional: Save Data to File**

If you want to save the collected data to a JSON file on the AAP instance:
1. Click the **Launch** icon
2. In the **Extra Variables** section, set:
   ```yaml
   save_to_file: true
   ```
3. Click **Next** and then **Launch**

The data will be saved to `/tmp/aap_user_troubleshooting_<hostname>_<timestamp>.json`

---

### **Step 4: Review the Output**

* **Next Steps:**
  1. Review the job output log. The playbook displays all collected API data in JSON format.
  2. The playbook collects the following information:
     - **Authenticators Configuration** - All configured authenticators (LDAP, SAML, etc.) and their settings
     - **User Information** - All users or specific targeted users
     - **User Authenticators** - Which authenticators each user is associated with
     - **Service Index Resources** - User resources across all AAP services (gateway, controller, galaxy, eda)
     - **Service Metadata** - Service IDs and version information
     - **Authenticator Maps** - Role, team, and organization mapping rules
  3. Share the collected output with Red Hat Support for analysis
  4. If `save_to_file` was set to `true`, retrieve the JSON file from `/tmp/` directory on the AAP instance

---

### **What Data is Collected**

This playbook addresses the specific API endpoints requested in **AAP-56517**:

| API Endpoint | Purpose |
|--------------|---------|
| `/api/gateway/v1/authenticators/` | Configuration of all authenticators including auto-migration settings |
| `/api/gateway/v1/users/` | All users in the system or targeted users |
| `/api/gateway/v1/users/<id>/authenticators/` | User-specific authenticator associations |
| `/api/gateway/v1/authenticator_maps/` | Role and team mapping rules |
| `/api/<service>/service-index/resources/` | User resources in each service (gateway, controller, galaxy, eda) |
| `/api/<service>/service-index/metadata/` | Service IDs for all AAP services |

---

### **Common Use Cases**

#### **Case 1: User Cannot Login After Certificate Update**

1. Run the job template with the affected user specified:
   ```yaml
   target_users:
     - affected_username
   ```
2. Review the output for:
   - Multiple authenticator associations
   - Missing auto-migration flags
   - Duplicate entries in service index

#### **Case 2: Multiple Users Experiencing Login Issues**

1. Run the job template without specifying `target_users` to collect all users
2. Set `save_to_file: true` to save the complete output
3. Share the JSON file with Red Hat Support

#### **Case 3: Quick Check for Specific Users**

1. Specify the problematic usernames in `target_users`
2. Keep `save_to_file: false` to view output in job log only

---

### **Troubleshooting**

#### **Issue: "Authentication Failed" Error**

**Check:**
- Verify the AAP Platform credential has API access permissions
- Ensure the credential user is not locked out
- Verify the credential is properly configured

#### **Issue: Timeout Errors**

**Solution:**
- Reduce the number of users by specifying `target_users` with a smaller list
- Check network connectivity to AAP services

#### **Issue: Missing Data in Output**

**Solution:**
- Verify all AAP services (gateway, controller, galaxy, eda) are running
- Check that the credential has permissions to access all services

---

### **Security Considerations**

- The playbook uses AAP Platform credentials which are securely stored
- Collected data may contain user information (usernames, emails)
- Review collected data before sharing externally
- If saving to file, delete JSON output files after analysis
- Files are saved to `/tmp/` which may be cleared on reboot
