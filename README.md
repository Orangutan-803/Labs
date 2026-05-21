**Objective:** Deploy Azure Sentinel to collect, detect, and alert on SSH login attempts (both failed and successful) from a Linux Ubuntu VM exposed to the internet.

### Implementation Steps

1. **Infrastructure Setup**

-    Created an Ubuntu VM (`non-vulnerable-01`) with OpenSSH port 22 open to the internet, placed in resource group `orangutan-labs`.

   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/5a63918d-f3b6-4a8f-823b-1de21000c12d" />

-    Verified local SSH connectivity.

<img width="1003" height="535" alt="Image" src="https://github.com/user-attachments/assets/a0f23146-5725-4346-a8c2-77f456c582b1" />



2. **Sentinel Deployment**

- Created a Log Analytics workspace (`monkey-watch`) and enabled Azure Sentinel.

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/6c5509b5-b2f7-4439-b980-a545c670f942" />


<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/4eda89e3-7031-4ddf-a49b-a901ba45d393" />

3. **Log Collection Configuration**

- Used Syslog with Azure Monitor Agent (AMA) (recommended by DeepSeek AI for its general-purpose, low-setup design).

<img width="3334" height="1860" alt="Image" src="https://github.com/user-attachments/assets/8be0a259-d046-4d81-8aad-8acedd0311df" />

<img width="1916" height="996" alt="Image" src="https://github.com/user-attachments/assets/73a49bca-05b1-4ccf-8162-ba9b82064679" />

<img width="1919" height="993" alt="Image" src="https://github.com/user-attachments/assets/e28eebfa-20b8-4d3e-955f-6adda45be2f4" />

- Created a **Data Collection Rule (DCR)** associated with the VM, defining which log facilities and severity levels to forward.

<img width="1920" height="995" alt="Image" src="https://github.com/user-attachments/assets/1c110950-dfa8-4c4f-aa43-9ae0e3c9c875" />

<img width="1915" height="995" alt="Image" src="https://github.com/user-attachments/assets/93f8ca25-b9be-45d2-936c-b92fcaca76af" />


4. **Troubleshooting & Refinement using **

- Initial issue: No syslog data appeared in Sentinel after 20+ minutes (only heartbeat logs).

<img width="1918" height="998" alt="Image" src="https://github.com/user-attachments/assets/fa9b8f88-5ccd-48a0-928e-a8c87d893cc7" />

- Used the Azure observability agent (Logs tab) to query VM logs – confirmed heartbeats present, but no Syslog table.

<img width="1920" height="995" alt="Image" src="https://github.com/user-attachments/assets/c1b057ae-d597-4f7e-a4c9-ccc033547c83" />

<img width="924" height="202" alt="Image" src="https://github.com/user-attachments/assets/cdb99ae3-700c-4b27-8b14-b90b3c5c4c5b" />

<img width="649" height="372" alt="Image" src="https://github.com/user-attachments/assets/a68c153f-8df7-43b7-b331-0cb318e5e67e" />

- Leveraged Observability agent to guide further diagnosis. The AI provided four possible root causes:

<img width="918" height="341" alt="Image" src="https://github.com/user-attachments/assets/69cd37c4-c8ca-4518-9529-bbcf5f69cfc6" />

<img width="903" height="316" alt="Image" src="https://github.com/user-attachments/assets/64623962-ca67-415c-9928-42e2a093a57c" />

-     VM not associated with DCR → verified association, ruled out
-     AMA not installed correctly → verified installed and running
-     Syslog not generated on VM → SSH into VM, checked /var/log/syslog – logs were present

<img width="981" height="507" alt="Image" src="https://github.com/user-attachments/assets/e58d42cd-cded-4a69-811c-2dd7d9fcae4f" />

-     DCR facility/severity filters too narrow → **identified as the real issue**

- DeepSeek clarified how syslog filters work: I originally set filters to alert level only, expecting only auth alerts. That was too narrow.

<img width="1411" height="316" alt="Image" src="https://github.com/user-attachments/assets/bf5675fe-0716-4dca-9fdc-d1807f212d77" />

<img width="1512" height="746" alt="Image" src="https://github.com/user-attachments/assets/1ab52909-4b03-4bed-997b-2e990b0d98b9" />

Fix: Broadened DCR filters to capture a wider range of syslog events.

<img width="897" height="859" alt="Image" src="https://github.com/user-attachments/assets/2d09283b-dcbd-4ffc-9941-a07042b35ffc" />

Result: Within 30 minutes, over 1,000 syslog events ingested, and the Syslog table appeared.

<img width="1325" height="823" alt="Image" src="https://github.com/user-attachments/assets/d9cbd087-5539-4e91-a69e-2a65692b57c0" />

5. **Alerting Rules**

- Observed multiple failed SSH login attempts from various IPs (e.g., Romania) hitting the exposed VM.

<img width="1316" height="825" alt="Image" src="https://github.com/user-attachments/assets/0ce8995b-7cf9-437c-a8e4-9952063a1b17" />

<img width="1914" height="983" alt="Image" src="https://github.com/user-attachments/assets/52c37c86-bb7e-4208-8395-99cc894ffc73" />

- Created Sentinel analytics rules to generate alerts for:
- Failed SSH login attempts

<img width="1630" height="838" alt="Image" src="https://github.com/user-attachments/assets/c55f68a1-6aa4-47b6-b4dd-c98b830df582" />

- Successful SSH login attempts

<img width="1672" height="855" alt="Image" src="https://github.com/user-attachments/assets/f4ab9eb4-4e91-4a68-ad24-7bd895ba46f7" />

- Sentinel Alerts triggered

<img width="1914" height="989" alt="Image" src="https://github.com/user-attachments/assets/d4ef7dac-0d74-4ce7-bb41-7e243258b855" />

### Key Learnings

- Hands-on familiarity with Azure resource hierarchy (Resource Groups → VMs → Log Analytics → Sentinel).
- Practical experience integrating a Linux VM with Sentinel via Syslog + AMA + DCR.
- Importance of correctly tuning DCR filters – narrow rules miss data, overly broad rules generate noise.
- Used DeepSeek AI effectively as a troubleshooting partner – its unlimited prompts and "thinking model" helped vet responses and clarify syslog filter logic.
- Used Azure observability agent to query live logs and verify table creation, speeding up root-cause analysis.
- Real-world observation: public SSH ports attract almost immediate intrusion attempts, underscoring the value of proactive monitoring.

### Future Refinements

- Fine-tune DCR filters to exclude low‑severity or irrelevant syslog messages while retaining security‑relevant events.
- Automate response actions (e.g., IP blocking) based on Sentinel alerts.
