[comment]: <> (please keep all comment items at the top of the markdown file)
[comment]: <> (please do not change the ***, as well as <div> placeholders for Note and Tip layout)
[comment]: <> (please keep the ### 1. and 2. titles as is for consistency across all demoguides)
[comment]: <> (section 1 provides a bullet list of resources + clarifying screenshots of the key resources details)
[comment]: <> (section 2 provides summarized step-by-step instructions on what to demo)


[comment]: <> (this is the section for the Note: item; please do not make any changes here)
***
### Azure Container Apps with Auto-Scaling, Traffic Splitting & Jobs - demo scenario

<div style="background: lightgreen; 
            font-size: 14px; 
            color: black;
            padding: 5px; 
            border: 1px solid lightgray; 
            margin: 5px;">

**Note:** Below demo steps should be used **as a guideline** for doing your own demos. Please consider contributing to add additional demo steps.
</div>

[comment]: <> (this is the section for the Tip: item; consider adding a Tip, or remove the section between <div> and </div> if there is no tip)

<div style="background: lightyellow; 
            font-size: 14px; 
            color: black;
            padding: 5px; 
            border: 1px solid lightgray; 
            margin: 5px;">

**Tip:** This scenario showcases 3 independent demos (Auto-Scaling, Traffic Splitting, Container Apps Jobs) that can be run individually or together. Total demo time: ~25 minutes.
</div>

***
### 1. What Resources are getting deployed
This scenario deploys an Azure Container Apps environment with 3 containerized applications and 3 Container Apps Jobs, backed by Azure Container Registry, Service Bus, and a managed identity — all orchestrated via Azure Developer CLI (azd).

* **rg-contappdemo** - Azure Resource Group containing all 12 resources
* **Azure Container Registry** - Stores container images for all apps and jobs (Basic tier)
* **Container Apps Environment** - Managed environment hosting all Container Apps and Jobs (KEDA 2.17, Dapr 1.13)
* **dashboard** - Container App: Vue.js real-time dashboard for triggering load tests
* **ingestion-service** - Container App: .NET 8 API with HTTP auto-scaling (1–100 replicas)
* **hello-api** - Container App: .NET 8 API demonstrating revision-based traffic splitting
* **data-processor-scheduled** - Container Apps Job: Cron-triggered (every 2 minutes)
* **data-processor-manual** - Container Apps Job: On-demand manually triggered
* **data-processor-parallel** - Container Apps Job: Parallel execution (3 workers simultaneously)
* **Azure Service Bus Namespace** - Messaging backbone for event processing
* **User-Assigned Managed Identity** - Secure authentication to ACR and Service Bus
* **Log Analytics Workspace** - Centralized logging and monitoring

<img src="screenshots/01-ResourceGroup_Overview.png" alt="Resource Group Overview showing all 12 deployed resources" style="width:70%;">
<br></br>

<img src="screenshots/02-Container_Registry_Overview.png" alt="Azure Container Registry overview with login server and Basic pricing tier" style="width:70%;">
<br></br>

<img src="screenshots/03-Container_Registry_Repositories.png" alt="Container Registry repositories showing 4 container images" style="width:70%;">
<br></br>

<img src="screenshots/04-Container_Apps_Environment.png" alt="Container Apps Environment overview with all 3 apps and 3 jobs listed" style="width:70%;">
<br></br>

### 2. What can I demo from this scenario after deployment

After the deployment completes, you can run **3 independent demos** showcasing core Azure Container Apps capabilities:

---

#### Demo 1: HTTP Auto-Scaling (≈10 min)

> **Key Message:** *"Container Apps can automatically scale from 1 to 100+ replicas based on HTTP traffic — no manual configuration of KEDA or HPA required."*

1. Navigate to **ingestion-service** in the Azure Portal. Show the Container App overview, confirming it is in **Running** status.

<img src="screenshots/05-Ingestion_Service_Overview.png" alt="Step 1 - ingestion-service Container App overview" style="width:70%;">
<br></br>

2. Click **Revisions and replicas** to show the current state: **1 active revision** with **1 replica** running and 100% traffic.

<img src="screenshots/06-Ingestion_Revisions_Initial.png" alt="Step 2 - Initial state showing 1 revision, 1 replica" style="width:70%;">
<br></br>

3. Click **Scale** to show the scaling configuration: Min replicas = 1, Max replicas = 100, with an **HTTP scaler** rule (Cooldown: 300s, Polling interval: 30s).

<img src="screenshots/07-Ingestion_Scale_Rules.png" alt="Step 3 - Scale settings with HTTP scaler, max 100 replicas" style="width:70%;">
<br></br>

4. Open the **Dashboard** application URL in a new browser tab. Point out the **Contoso Analytics** interface with the "Send 100 Events" button and the live architecture diagram.

<img src="screenshots/08-Dashboard_App.png" alt="Step 4 - Dashboard application with Send 100 Events button" style="width:70%;">
<br></br>

5. Click **"Send 100 Events"** and wait for the confirmation message: *"✅ Complete! 100 events sent."*

<img src="screenshots/09-Dashboard_Load_Sent.png" alt="Step 5 - Dashboard showing 100 events sent successfully" style="width:70%;">
<br></br>

6. Navigate back to **ingestion-service → Revisions and replicas** in the Azure Portal. Observe the replica count has automatically scaled up (in this demo, it scaled to **15 replicas**) in response to the HTTP load.

<img src="screenshots/10-Ingestion_Scaled_15_Replicas.png" alt="Step 6 - Revisions page showing auto-scaled to 15 replicas" style="width:70%;">
<br></br>

7. **Talking Point:** *"That's it — no KEDA installation, no HPA manifests, no Kubernetes config. Container Apps handles all the scaling infrastructure for you. After the 5-minute cooldown period, replicas will automatically scale back down to 1."*

---

#### Demo 2: Traffic Splitting with Revisions (≈8 min)

> **Key Message:** *"Container Apps supports blue-green and canary deployments out of the box using revision-based traffic splitting — no ingress controller required."*

1. Open the **hello-api** URL in a browser. Show the current version displaying a blue **"v1"** badge with the .NET 8 runtime info and hostname.

<img src="screenshots/11-HelloAPI_v1.png" alt="Step 1 - Hello API showing v1 blue badge" style="width:70%;">
<br></br>

2. In the Azure Portal, navigate to **hello-api → Revisions and replicas**. Show the single active revision running 100% of traffic.

<img src="screenshots/12-HelloAPI_Revisions.png" alt="Step 2 - hello-api revisions showing single revision at 100% traffic" style="width:70%;">
<br></br>

3. To demonstrate traffic splitting, run the following CLI commands:

```powershell
# Set variables
$rg = "<your-resource-group>"
$acr = "<your-acr>.azurecr.io"

# Enable Multiple revision mode
az containerapp revision set-mode -n hello-api -g $rg --mode Multiple

# Build and push v2 image (update v2 styling in code first)
cd src/hello-api
docker build -t "$acr/hello-api:v2" .
docker push "$acr/hello-api:v2"

# Deploy v2 as a new revision
az containerapp update -n hello-api -g $rg --image "$acr/hello-api:v2"

# List revisions to get exact names
az containerapp revision list -n hello-api -g $rg --query "[].name" -o tsv

# Split traffic 50/50 between v1 and v2
az containerapp ingress traffic set -n hello-api -g $rg `
  --revision-weight "<v1-revision>=50" "<v2-revision>=50"
```

4. Refresh the hello-api URL multiple times — you will see the response alternate between **v1** and **v2** badges, demonstrating the 50/50 traffic split.

5. Once validated, shift 100% traffic to v2:

```powershell
az containerapp ingress traffic set -n hello-api -g $rg `
  --revision-weight "<v2-revision>=100"
```

6. **Talking Point:** *"With Container Apps, canary and blue-green deployments are a single CLI command. No service mesh, no ingress controller annotations — just set the percentages."*

---

#### Demo 3: Container Apps Jobs (≈7 min)

> **Key Message:** *"Container Apps supports three types of jobs — scheduled, manual, and event-driven — all managed without Kubernetes CronJob YAML or kubectl."*

1. Navigate to **data-processor-scheduled** in the Azure Portal. Show the job overview: Trigger Type = **Schedule**, Cron expression = `*/2 * * * *` (every 2 minutes), Parallelism = 1, Completion count = 1.

<img src="screenshots/13-Job_Scheduled_Overview.png" alt="Step 1 - Scheduled job overview with cron expression" style="width:70%;">
<br></br>

2. Click **Execution history** to show the automatic executions. Note the regular 2-minute intervals with **Succeeded** status and execution durations of ~27–50 seconds.

<img src="screenshots/14-Job_Scheduled_Execution_History.png" alt="Step 2 - Scheduled job execution history showing many succeeded runs" style="width:70%;">
<br></br>

3. **Talking Point:** *"Scheduled jobs run automatically based on the cron expression. No CronJob YAML, no kubectl — just define the schedule in your Bicep template."*

4. Navigate to **data-processor-manual**. Show the overview: Trigger Type = **Manual**, Parallelism = 1, Completion count = 1. Trigger it via CLI:

```powershell
az containerapp job start -n data-processor-manual -g $rg
```

<img src="screenshots/15-Job_Manual_Overview.png" alt="Step 4 - Manual job overview" style="width:70%;">
<br></br>

5. **Talking Point:** *"Manual jobs are perfect for on-demand tasks — data migrations, report generation, cleanup tasks."*

6. Navigate to **data-processor-parallel**. Show the overview: Trigger Type = **Manual**, Parallelism = **3**, Completion count = **3** — meaning 3 workers run simultaneously. Trigger it via CLI:

```powershell
az containerapp job start -n data-processor-parallel -g $rg
```

<img src="screenshots/16-Job_Parallel_Overview.png" alt="Step 6 - Parallel job overview with parallelism of 3" style="width:70%;">
<br></br>

7. **Talking Point:** *"Parallel jobs split work across multiple instances. Great for processing large datasets quickly — each worker handles a portion."*

---

#### Demo Summary

| Feature | Container Apps | Kubernetes |
|---------|---------------|------------|
| **HTTP Scaling** | Built-in, just set threshold | Install KEDA + HPA |
| **Traffic Splitting** | Portal or CLI toggle | Ingress controller + annotations |
| **Batch Jobs** | Bicep/ARM definition | CronJob YAML + kubectl |
| **Infrastructure** | Fully managed | Cluster management required |

[comment]: <> (this is the closing section of the demo steps. Please do not change anything here to keep the layout consistant with the other demoguides.)
<br></br>
***
<div style="background: lightgray; 
            font-size: 14px; 
            color: black;
            padding: 5px; 
            border: 1px solid lightgray; 
            margin: 5px;">

**Note:** This is the end of the current demo guide instructions. After the demo, clean up all resources by running `azd down --force --purge`.
</div>



