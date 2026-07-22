---
title: "Visualizing Analytics with Shiny Dashboards"
weight: 55
chapter: false
pre: " <b> 5.5. </b> "
---

## 5.5.1 Environment information

- OS: **Ubuntu 22.04 (Jammy)** – Running on an EC2 instance in a private subnet  
- PostgreSQL: **v18** (installed via the `apt.postgresql.org` repository)  
- Shiny Server: `.deb` binary distribution from RStudio (Posit)  
- Service User: `shiny`  
- Application Path: `/srv/shiny-server/sbw_dashboard/app.R`

---

## 5.5.2 Install system packages (system libs)

Note: Ensure the NAT Gateway is enabled prior to downloading system packages.  
Connect to the EC2 instance using **SSM Session Manager** (or SSH temporarily, if configured), then execute:

```bash
# 1) Refresh the package lists
sudo apt-get update

# 2) Install R (if it is not already present)
sudo apt-get install -y r-base

# 3) Install PostgreSQL client and development headers (needed for RPostgres)
#    If using PG 18, use postgresql-server-dev-18
#    (adjust the version number 18 -> 14, 15, etc., as appropriate)
sudo apt-get install -y postgresql-client-18 postgresql-server-dev-18

# 4) Install libpq and libssl (required for building RPostgres)
sudo apt-get install -y libpq-dev libssl-dev

# 5) (If Shiny Server is not yet installed)
#    Regardless of the installation method, remember these key paths:
#    - shiny-server service: /etc/systemd/system/shiny-server.service
#    - application directory: /srv/shiny-server/
#    - run user: shiny
```

Verify that `libpq` and the necessary development headers are installed:

```bash
dpkg -l | grep -E 'libpq-dev|postgresql-server-dev' || echo "MISSING_LIBS"
ls -l /usr/include/postgresql/libpq-fe.h || echo "NO_LIBPQ_HEADER"
```

If you **do not see any error messages**, the installation was successful.

---

## 5.5.3 Configure R library folder for user `shiny`

To ensure Shiny Server can load the necessary R packages, install them under the `shiny` user within this specific directory:

- `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`

Run the following commands:

```bash
sudo -u shiny R --vanilla <<'EOF'
# Create the library directory for the shiny user if it's missing
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)

# Prepend R_LIBS_USER to the top of .libPaths()
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

q("no")
EOF
```

The output `LIBPATHS` should list `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1` as the first entry.

---

## 5.5.4 Install required R packages

The dashboard requires the following R packages:

- `shiny`
- `DBI`
- `RPostgres`
- `dplyr`
- `ggplot2`
- `lubridate`
- `pool`

Install all of them executing as the `shiny` user:

```bash
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)

q("no")
EOF
```

💡 **Troubleshooting `libpq-fe.h` or `libpq` errors:**

1. Double-check that `libpq-dev`, `postgresql-server-dev-XX`, and `libssl-dev` are properly installed.  
2. Once the libraries are confirmed, re-run `install.packages("RPostgres", ...)` to build the package.  

Test that the packages can be successfully loaded:

```bash
sudo -u shiny R --vanilla <<'EOF'
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

library(shiny)
library(DBI)
library(RPostgres)
library(dplyr)
library(ggplot2)
library(lubridate)
library(pool)

cat("All packages loaded OK
")
q("no")
EOF
```

If **no errors** occur, the R environment is ready.

---

## 5.5.5 Deploying the Shiny app

### 5.5.5.1 Create the app folder and copy code

```bash
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
```

Create (or overwrite) the main application file:

```bash
sudo nano /srv/shiny-server/sbw_dashboard/app.R
# PASTE THE FULL app.R CODE (the complete version you intend to deploy)
# Press Ctrl+O, Enter, and then Ctrl+X to save and exit
```

Verify the file permissions are set correctly:

```bash
sudo chown shiny:shiny /srv/shiny-server/sbw_dashboard/app.R
sudo chmod 644 /srv/shiny-server/sbw_dashboard/app.R
```

### 5.5.5.2 Restart Shiny Server

```bash
sudo systemctl restart shiny-server
sudo systemctl status shiny-server
```

---

## 5.5.6 Check the app from EC2 (local)

From within your active SSM session (terminal) on the EC2 instance:

```bash
# Verify the Shiny Server welcome page
curl -m 5  -sS -o /dev/null -w "WELCOME HTTP %{http_code}
"   http://127.0.0.1:3838/

# Verify the SBW dashboard app
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```

Receiving `DASHBOARD HTTP 200` confirms the application is running successfully.

If you encounter a `500` status code:

```bash
LATEST=$(ls -1t /var/log/shiny-server/sbw_dashboard-shiny-*.log | head -n 1)
echo "LATEST=$LATEST"
sudo tail -n 100 "$LATEST"
```

Review the error log output to diagnose the issue.

---

## 5.5.7 Access the dashboard from your local machine

Since the EC2 instance resides in a **private subnet**, access is routed through **SSM port forwarding**:

```bash
# Example command using AWS CLI v2 on your local workstation:
aws ssm start-session   --target <INSTANCE_ID_PRIVATE>   --document-name AWS-StartPortForwardingSessionToRemoteHost   --parameters '{"host":["127.0.0.1"],"portNumber":["3838"],"localPortNumber":["3838"]}'
```

Once the session is established, open a web browser on your local machine and navigate to:

```text
http://127.0.0.1:3838/sbw_dashboard/
```

The dashboard should load and display elements such as:

- **KPI cards** (aggregating total events, users, sessions, etc.)  
- Trend charts detailing **events over time**, **event mix**, and **events by login state**  
- A **Products & Raw sample** tab (featuring pagination, newest records first, and automatic refreshing—depending on the specific app implementation)

---

## 5.5.8 Quick summary of important commands

```bash
# Install required system libraries
sudo apt-get update
sudo apt-get install -y r-base postgresql-client-18 postgresql-server-dev-18 libpq-dev libssl-dev

# Install R packages as the shiny user
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)
q("no")
EOF

# Deploy the application code
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo nano /srv/shiny-server/sbw_dashboard/app.R   # Paste your code here
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
sudo systemctl restart shiny-server

# Verify the dashboard is accessible locally
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```
