# Quiz_app_java

---
- name: Deploy keystore files and set permissions for UK UAT
  hosts: all
  become: true
  tasks:

    - name: Ensure keystore directory exists
      file:
        path: /opt/juniper/keystore
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
