# ELK Docker container for AEMCS CDN and AEM Log Analysis

This project provides a pre-configured ELK (Elasticsearch, Logstash, and Kibana) stack to help you aggregate, analyze, and visualize logs from Adobe Experience Manager as a Cloud Service (AEMCS).

It supports both **CDN logs** and **AEM application logs** (Author, Publish, and Dispatcher).

## 🚀 Overview for Developers

If you are new to the project, this tool helps us troubleshoot issues, monitor traffic, and analyze performance locally without relying solely on cloud-based logging tools. 

You simply download logs from AEM Cloud Manager, drop them into specific folders, and spin up the Docker container. Logstash will automatically parse them, Elasticsearch will index them, and Kibana will let you visualize them using pre-built dashboards.

### Supported Dashboards

We have built a suite of dashboards to provide insights out-of-the-box:

**AEM Dashboards (New!)**
- **AEM Ops Overview Dashboard**: High-level overview of AEM operations, tier health, and general status.
- **AEM Error Triage Dashboard**: Quickly identify and triage common AEM errors and exceptions.
- **AEM Error Deep Dive Dashboard**: Detailed view for investigating specific AEM errors and their stack traces.
- **AEM Warning Analysis Dashboard**: Focuses on analyzing AEM warning logs to proactively address potential issues.
- **AEM Request Performance Dashboard**: Analysis of AEM request response times and performance bottlenecks.
- **AEM Access Traffic Dashboard**: Insights into traffic directly hitting AEM instances (author and publish tiers).

**CDN Dashboards**
- **CDN Cache Hit Ratio**: Insights into cache hit/miss/pass ratios and top requested URLs.
- **Traffic Filter Rules (WAF) Analysis**: Analyzes flagged and blocked requests, top attacks by WAF Rule ID, and attacker IPs.
- **Content Request Dashboard**: Tracks requests counting toward content requests for license measurement.

---

## 🛠 Prerequisites

- Install [Docker Desktop](https://docs.docker.com/engine/install/).
- Allocate sufficient memory to Docker. The containers are configured to use a combined **~3 GB of RAM** (`Preferences -> Resources -> Advanced`). Ensure your system has at least 4 GB available for Docker. *(Note: We have optimized the memory footprint so it runs smoothly on standard developer machines!)*
- Download the logs you want to analyze from [Adobe Cloud Manager](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-logs.html?lang=en).

---

## 📂 Where to Place Your Logs

Before starting the container, place your logs in the correct directory. You must unzip any `.log.gz` files so they end in `.log`.

### 1. AEM & Dispatcher Logs
AEM application logs (like `aemerror`, `aemaccess`, `aemrequest`) and Dispatcher logs (`httpdaccess`, `httpderror`, `aemdispatcher`) go into the `aem-logs` folder. 

Group them by environment. For example:
```shell
AEMCS-CDN-Log-Analysis-Tooling/ELK/aem-logs/dev/
AEMCS-CDN-Log-Analysis-Tooling/ELK/aem-logs/stage/
AEMCS-CDN-Log-Analysis-Tooling/ELK/aem-logs/prod/
```
Drop all files for a specific environment into its respective folder. The filename must contain the log type token (e.g., `publish-aemerror-2023.log`). Logstash relies on the filename to pick the correct parser!

### 2. CDN Logs
CDN logs go into the `logs` folder. Similar to AEM logs, group them by environment:
```shell
AEMCS-CDN-Log-Analysis-Tooling/ELK/logs/dev/
AEMCS-CDN-Log-Analysis-Tooling/ELK/logs/stage/
AEMCS-CDN-Log-Analysis-Tooling/ELK/logs/prod/
```

---

## 🏃‍♂️ How to Run the Project

1. **Clone the repository** and navigate to the ELK directory:
    ```shell
    git clone git@github.com:adobe/AEMCS-CDN-Log-Analysis-Tooling.git
    cd AEMCS-CDN-Log-Analysis-Tooling/ELK
    ```

2. **Add your logs** into the `aem-logs/<env>/` or `logs/<env>/` directories as explained above.

3. **Start the ELK stack**:
    ```shell
    docker compose up
    ```
    *Tip: You can add more logs while the container is running. Logstash will automatically detect and parse new `.log` files.*

4. **Open Kibana**:
    Once the console output settles, navigate to [http://localhost:5601](http://localhost:5601) in your browser.

5. **Import Dashboards**:
    - Go to the **Menu** (top-left) -> **Stack Management** -> **Saved Objects**.
    - Click **Import** and select the `.ndjson` files from the `ELK/dashboards/` directory in this repository.

6. **View the Dashboards**:
    - Go to **Menu** -> **Analytics** -> **Dashboards**.
    - Open any imported dashboard to start exploring your data!
    - Use the time filter (top-right) to ensure your log timestamps are covered.
    - Use the `aem_env_name` filter to toggle between `dev`, `stage`, or `prod`.

---

## 🔧 Troubleshooting & Tips

**I added logs but don't see them in Kibana?**
- Ensure the files end in `.log` (unzip `.gz` files).
- Expand your time range in Kibana (top right corner). Logs are based on UTC, which might differ from your local timezone.
- If you need Logstash to re-read files from scratch, stop the container, delete the hidden tracking files, and restart:
  ```shell
  docker compose stop
  rm -f aem-logs/.sincedb*
  rm -f logs/.sincedb*
  docker compose up
  ```

**How do I completely reset the environment?**
To wipe all data in Elasticsearch and start fresh:
```shell
docker compose down -v
rm -f aem-logs/.sincedb*
rm -f logs/.sincedb*
```

**How do I stop the containers?**
Press `Ctrl+C` in the terminal where it's running, or run:
```shell
docker compose down
```

**Note on Content Requests Dashboard:**
For the Content Request Dashboard to be fully accurate, ensure custom fields `req_ref` and `bot_name` are configured in your CDN via Cloud Manager. Otherwise, counts may be inaccurate.

