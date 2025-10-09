# Quiz_app_java

---
- name: Deploy keystore files and set permissions for UK UAT
  hosts: all
  become: true
  tasks:

    - name: Ensure keystore directory exists
      file:
        path: /opt/juniperuu/keystore
        state: directory
        mode: '0775'

    - name: Copy truststore.jks
      copy:
        src: truststore.jks
        dest: /opt/juniper/keystore/truststore.jks
        mode: '0775'

    - name: Copy master_key.txt
      copy:
        src: master_key.txt
        dest: /opt/juniper/keystore/master_key.txt
        mode: '0775'

    - name: Create telemetry store directory with drwxr-x--- permission
      file:
        path: /opt/juniper/applicationcode/devops/build/juniper-stream-kafka-telemetry/store
        state: directory
        owner: appuser        # 🔁 Replace with actual owner
        group: appgroup       # 🔁 Replace with group needing access
        mode: '0750'

    - name: Ensure log directory is readable by other users
      file:
        path: /var/log/juniper/
        state: directory
        mode: '0755'
      tags: log-permission



Here's the flow of API calls along with UI actions and error handling in the format you requested:

---

**START**  
↓  
**User Action: Creates Feed**  
- The user initiates an action to create a feed.  
↓  
**Core Info Page Loads**  
- The Core Info Page UI starts loading.  
↓  
**APIs Triggered for Core Info Page**  
- `getOverallCountInfo`  
- `getFeedConfigFeedSequence`  
- `getFeedConnectorImages`  
↓  
**API Responses Processed**  
- The API responses are received and processed.  
- The UI updates based on the received data.  
↓  
**Error Handling**  
- If an API request fails, check and debug in:  
  - **`getCounts` function in `index.js`** (for `getOverallCountInfo`)  
  - **`apiUtils.js` or `sideNavigation.js`** (for API request configurations)  
  - **Check Console Logs (`console.log(err)`)** for debugging issues.  
- Possible fixes:  
  - Ensure correct API URLs in `API_URLS` object.  
  - Verify API request headers and parameters.  
  - Handle empty or undefined responses in UI logic (`menuCountInfo[0]`).  
↓  
**Final UI Updates**  
- If data is valid, `setCounts` and `updateCount` update the UI.  
- If there are errors, an appropriate fallback or error message is shown.  

---

This structured flow should help you understand API calls and UI updates, along with where to debug errors if they occur. Let me know if you need modifications!




---
- name: Download and deploy artifact from Nexus
  hosts: all
  vars:
    artifact_build_loc: "/opt/build"
    nexus_repo_url: "https://nexus304.systems.uk.hsbc:8081/repository/maven-releases"
    group_id: "com.juniperx.services"
    artifact_id: "juniper-bqload"
    artifact_version: "1.0.0"
    nexus_user: "{{ vault_nexus_user }}"
    nexus_password: "{{ vault_nexus_password }}"
    dest_folder: "/opt/juniper/customscheduler"

  tasks:

    - name: Create build directory if not exists
      file:
        path: "{{ artifact_build_loc }}"
        state: directory
        mode: '0755'

    - name: Remove old files from build location
      file:
        path: "{{ artifact_build_loc }}/{{ artifact_id }}"
        state: absent

    - name: Download artifact (jar/zip) from Nexus
      community.general.maven_artifact:
        group_id: "{{ group_id }}"
        artifact_id: "{{ artifact_id }}"
        version: "{{ artifact_version }}"
        repository_url: "{{ nexus_repo_url }}"
        username: "{{ nexus_user }}"
        password: "{{ nexus_password }}"
        extension: "zip"
        dest: "{{ artifact_build_loc }}"
        validate_certs: no
      register: download_result

    - name: Debug to verify download success
      debug:
        msg: "Artifact downloaded successfully to {{ download_result.dest }}"

    - name: Unzip downloaded artifact
      unarchive:
        src: "{{ artifact_build_loc }}/{{ artifact_id }}-{{ artifact_version }}.zip"
        dest: "{{ dest_folder }}"
        remote_src: yes
      when: download_result is defined

    - name: Set permissions for destination folder
      file:
        path: "{{ dest_folder }}"
        mode: '0755'
        recurse: yes

    - name: Verify files exist in destination
      command: ls -l "{{ dest_folder }}"
      register: result

    - debug:
        var: result.stdout_lines


