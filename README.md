# SQL Vulnerable Cloud Cybersecurity Lab

![GitHub repo size](https://img.shields.io/github/repo-size/JQCVSC/bank-of-anthos-sql-vulnerable-cloud-cybersecurity)
![GitHub contributors](https://img.shields.io/github/contributors/JQCVS/bank-of-anthos-sql-vulnerable-cloud-cybersecurity)
![GitHub stars](https://img.shields.io/github/stars/JQCVS/bank-of-anthos-sql-vulnerable-cloud-cybersecurity?style=social)
![GitHub forks](https://img.shields.io/github/forks/JQCVS/bank-of-anthos-sql-vulnerable-cloud-cybersecurity?style=social)
![GitHub issues](https://img.shields.io/github/issues/JQCVS/bank-of-anthos-sql-vulnerable-cloud-cybersecurity)

## 📖 Overview
This lab is a modified version of the original Bank of Anthos web application, intentionally configured to include SQL vulnerabilities. It serves as an educational platform for ethical hacking, cloud security testing, and demonstrating SQL vulnerabilities in a cloud-native environment using **Google Cloud Platform (GCP)** and **Kubernetes**.

## 🎯 Objectives
- Deploy a cloud-native web application with intentional security flaws.
- Learn how to identify and exploit SQL vulnerabilities.
- Demonstrate common security testing methodologies for cloud applications.
- Gain hands-on experience with tools like **Docker**, **Kubernetes**, **OWASP ZAP**, and **SQLMap**.

## 📝 Lab Requirements
- **Operating System**: Parrot Security OS (or a similar penetration testing Linux distro)
- **Tools Installed**: Docker, kubectl, Google Cloud CLI (gcloud), Nmap, OWASP ZAP, SQLMap
- **Cloud Account**: A Google Cloud account to set up a Google Kubernetes Engine (GKE) cluster.

---

## 🚀 Step 1: Set Up the Environment

### 1.1 Install Dependencies
Make sure the following tools are available on Parrot Security OS:
- **Docker**
- **kubectl**
- **gcloud** (Google Cloud CLI)

### 1.2 Clone the Repository
Fork and clone the modified Bank of Anthos repository:

```bash
git clone https://github.com/YOUR_USERNAME/sql-vulnerable-cloud-cybersecurity
cd sql-vulnerable-cloud-cybersecurity/
```

1.3 Configure GCP Environment
Set up your Google Cloud project:

Copy code
```bash
export PROJECT_ID=<YOUR_PROJECT_ID>
export REGION=us-central1
```
# Enable necessary APIs
gcloud services enable container.googleapis.com --project=${PROJECT_ID}
1.4 Create a GKE Cluster

Copy code
```bash
gcloud container clusters create-auto sql-vulnerable-cloud \
  --project=${PROJECT_ID} --region=${REGION}
```
📦 Step 2: Deploy the Application
2.1 Deploy the App on GKE
Deploy the application using Kubernetes manifests:

Copy code
```bash
kubectl apply -f ./extras/jwt/jwt-secret.yaml
kubectl apply -f ./kubernetes-manifests
```

2.2 Access the App
Get the external IP address of the frontend service:

Copy code
```bash
kubectl get service frontend
```

Visit the external IP in your web browser to access the app.

🔍 Step 3: Vulnerability Scanning
3.1 Install Masscan
If Masscan is not already installed, install it:

Copy code
```bash
sudo apt update
sudo apt install masscan
```
3.2 Perform Reconnaissance
Scan the external IP for open ports:

Copy code
```bash
sudo masscan <EXTERNAL_IP> -p0-65535 --rate=1000
```
3.3 Static Analysis
Search for Hardcoded Credentials:

Copy code
```bash
grep -r "password" ./src/
```
Check for SQL Injection Points: Look for user input fields in Python or Java files that interact with the database.
3.4 Dynamic Testing
OWASP ZAP:

Copy code
```bash
zap-cli start
zap-cli spider http://<EXTERNAL_IP>
```
Burp Suite: Test for SQL injection using the intruder tool.
🛠️ Step 4: Exploiting Vulnerabilities
4.1 SQL Injection
Use SQLmap to exploit SQL injection vulnerabilities:


Copy code
```bash
sqlmap -u "http://<EXTERNAL_IP>/login" --data "username=admin&password=admin" --dbs
```
4.2 JWT Token Manipulation
Decode JWT Tokens:

Copy code
```bash
jwt_tool <TOKEN> -pc payload.txt
```
Attempt to Bypass Authentication: Modify the token payload to escalate privileges.
🧹 Step 5: Clean Up
When done, delete the GKE cluster:


Copy code
```bash
gcloud container clusters delete sql-vulnerable-cloud \
  --project=${PROJECT_ID} --region=${REGION}
```
⚖️ Ethical Considerations
Always perform testing in a controlled environment and do not misuse these techniques. This project is for educational purposes only.

📚 Additional Resources
Parrot Security OS Documentation
SQLmap User Manual
Google Cloud Kubernetes Engine
📜 License
This project is licensed under the MIT License.

🙌 Acknowledgements
Original Bank of Anthos app by Google.
Parrot Security OS for providing powerful security tools.
💬 Contact
For any questions or feedback, please open an issue on this repository.

Make sure to replace `YOUR_USERNAME`, `<YOUR_PROJECT_ID>`, and `<EXTERNAL_IP>` with the appropriate values for your GitHub username, Google Cloud project ID, and the actual external IP address. This formatted README.md should be easy to copy and paste into your repository.

## Screenshots

| Sign in                                                                                                        | Home                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Login](/docs/img/login.png)](/docs/img/login.png) | [![User Transactions](/docs/img/transactions.png)](/docs/img/transactions.png) |


## Service architecture

![Architecture Diagram](/docs/img/architecture.png)

| Service                                                 | Language      | Description                                                                                                                                  |
| ------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                              | Python        | Exposes an HTTP server to serve the website. Contains login page, signup page, and home page.                                                |
| [ledger-writer](/src/ledger/ledgerwriter)              | Java          | Accepts and validates incoming transactions before writing them to the ledger.                                                               |
| [balance-reader](/src/ledger/balancereader)            | Java          | Provides efficient readable cache of user balances, as read from `ledger-db`.                                                                |
| [transaction-history](/src/ledger/transactionhistory)  | Java          | Provides efficient readable cache of past transactions, as read from `ledger-db`.                                                            |
| [ledger-db](/src/ledger/ledger-db)                     | PostgreSQL    | Ledger of all transactions. Option to pre-populate with transactions for demo users.                                                         |
| [user-service](/src/accounts/userservice)              | Python        | Manages user accounts and authentication. Signs JWTs used for authentication by other services.                                              |
| [contacts](/src/accounts/contacts)                     | Python        | Stores list of other accounts associated with a user. Used for drop down in "Send Payment" and "Deposit" forms.                              |
| [accounts-db](/src/accounts/accounts-db)               | PostgreSQL    | Database for user accounts and associated data. Option to pre-populate with demo users.                                                      |
| [loadgenerator](/src/loadgenerator)                    | Python/Locust | Continuously sends requests imitating users to the frontend. Periodically creates new accounts and simulates transactions between them.      |

## Interactive quickstart (GKE)

The following button opens up an interactive tutorial showing how to deploy Bank of Anthos in GKE:

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?show=ide&cloudshell_git_repo=https://github.com/GoogleCloudPlatform/bank-of-anthos&cloudshell_workspace=.&cloudshell_tutorial=extras/cloudshell/tutorial.md)

## Quickstart (GKE)

1. Ensure you have the following requirements:
   - [Google Cloud project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project).
   - Shell environment with `gcloud`, `git`, and `kubectl`.

2. Clone the repository.

   ```sh
   git clone https://github.com/GoogleCloudPlatform/bank-of-anthos
   cd bank-of-anthos/
   

3. Set the Google Cloud project and region and ensure the Google Kubernetes Engine API is enabled.

   ```sh
   export PROJECT_ID=<PROJECT_ID>
   export REGION=us-central1
   gcloud services enable container.googleapis.com \
     --project=${PROJECT_ID}


   Substitute `<PROJECT_ID>` with the ID of your Google Cloud project.

4. Create a GKE cluster and get the credentials for it.

   ```sh
   gcloud container clusters create-auto bank-of-anthos \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Creating the cluster may take a few minutes.

5. Deploy Bank of Anthos to the cluster.

   ```sh
   kubectl apply -f ./extras/jwt/jwt-secret.yaml
   kubectl apply -f ./kubernetes-manifests
   ```

6. Wait for the pods to be ready.

   ```sh
   kubectl get pods
   ```

   After a few minutes, you should see the Pods in a `Running` state:

   ```
   NAME                                  READY   STATUS    RESTARTS   AGE
   accounts-db-6f589464bc-6r7b7          1/1     Running   0          99s
   balancereader-797bf6d7c5-8xvp6        1/1     Running   0          99s
   contacts-769c4fb556-25pg2             1/1     Running   0          98s
   frontend-7c96b54f6b-zkdbz             1/1     Running   0          98s
   ledger-db-5b78474d4f-p6xcb            1/1     Running   0          98s
   ledgerwriter-84bf44b95d-65mqf         1/1     Running   0          97s
   loadgenerator-559667b6ff-4zsvb        1/1     Running   0          97s
   transactionhistory-5569754896-z94cn   1/1     Running   0          97s
   userservice-78dc876bff-pdhtl          1/1     Running   0          96s
   ```

7. Access the web frontend in a browser using the frontend's external IP.

   ```sh
   kubectl get service frontend | awk '{print $4}'
   ```

   Visit `http://EXTERNAL_IP` in a web browser to access your instance of Bank of Anthos.

8. Once you are done with it, delete the GKE cluster.

   ```sh
   gcloud container clusters delete bank-of-anthos \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Deleting the cluster may take a few minutes.

