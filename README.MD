# Rideau Canal Dashboard

This repo contains the web dashboard for the **Rideau Canal Skateway Real-Time Monitoring System**.  
It reads processed sensor data from Azure Cosmos DB and shows the status of three locations:

- Dow’s Lake  
- Fifth Avenue  
- NAC  

---

## Overview

### Dashboard Features

- Live cards for each location
  - Average ice thickness
  - Average surface temperature
  - Max snow accumulation
  - Min / max ice range
  - Min / max surface temperature range
  - Average external temperature
  - Reading count per 5-minute window
  - Safety status badge (Safe / Caution / Unsafe)
- Overall canal status badge
- Last hour charts for:
  - Ice thickness (per location)
  - Surface temperature (per location)
- Auto-refresh every 30 seconds

### Technologies Used

- **Backend:** Node.js, Express
- **Database:** Azure Cosmos DB (NoSQL, Core/SQL API)
- **Frontend:** HTML, CSS, vanilla JavaScript
- **Charts:** Chart.js
- **Azure Hosting:** Azure App Service

---

## Prerequisites

- Node.js (v18+ recommended)
- An Azure Cosmos DB account with:
  - Database: `RideauCanalDB`
  - Container: `SensorAggregations`
- Stream Analytics job already writing aggregated data into that container

---

## Installation

1. Clone the repo:

   ```bash
   git clone https://github.com/ObaidaKandakji/rideau-canal-dashboard.git
   cd rideau-canal-dashboard
   ```

2. Install dependencies:

    ```bash
    npm install
    ```
3. Copy the example environment file:

    ```bash
    cp .env.example .env
    ```

## Configuration

- Edit .env and set:

    ```bash
    COSMOS_ENDPOINT=https://<your-cosmos-account>.documents.azure.com:443/
    COSMOS_KEY=<your-primary-key>
    COSMOS_DATABASE=RideauCanalDB
    COSMOS_CONTAINER=SensorAggregations
    PORT=3000
    ```

- Run Locally

    ```bash
    npm start
    ```
## API Endpoints

All endpoints are JSON.

### GET /api/latest

Returns the latest 5-minute window for each location.

- Example request
```bash
GET /api/latest
```
- Example response

```bash

{
  "success": true,
  "timestamp": "2025-12-08T21:45:30.123Z",
  "data": [
    {
      "id": "DowsLake-2025-12-08T21:40:00.0000000Z",
      "deviceId": "dows-lake-device",
      "location": "DowsLake",
      "windowEnd": "2025-12-08T21:40:00.0000000Z",
      "avgIceThickness": 33.0,
      "minIceThickness": 16.9,
      "maxIceThickness": 44.8,
      "avgSurfaceTemperature": -5.0,
      "minSurfaceTemperature": -9.2,
      "maxSurfaceTemperature": 1.1,
      "maxSnowAccumulation": 18.9,
      "avgExternalTemperature": -13.5,
      "readingCount": 29,
      "safetyStatus": "Safe"
    }
    // ... one item per location
  ]
}
```

### GET /api/history/:location

Returns historical windows for a location

- Example request

    ```bash
    GET /api/history/DowsLake?limit=12
    ```

- Example response

    ```bash
    {
    "success": true,
    "location": "DowsLake",
    "data": [
        {
        "windowEnd": "2025-12-08T21:00:00.0000000Z",
        "avgIceThickness": 30.5,
        "avgSurfaceTemperature": -4.8,
        "maxSnowAccumulation": 12.3
        }
        // ... up to 12 windows, oldest to newest
    ]
    }
    ```

### GET /api/status

Returns the latest safety status for each location and an overall status.

- Example request

    ```bash
    GET /api/status
    ```
- Example response

    ```bash
    
    {
        "success": true,
        "overallStatus": "Safe",
        "locations": [
            {
            "location": "DowsLake",
            "safetyStatus": "Safe",
            "windowEnd": "2025-12-08T21:40:00.0000000Z"
            },
            {
            "location": "FifthAvenue",
            "safetyStatus": "Caution",
            "windowEnd": "2025-12-08T21:40:00.0000000Z"
            },
            {
            "location": "NAC",
            "safetyStatus": "Safe",
            "windowEnd": "2025-12-08T21:40:00.0000000Z"
            }
        ]
    }
    ```

### GET /api/all (debug)

Returns all documents from the Cosmos container, sorted by windowEnd descending.

- Example

    ```bash
    GET /api/all
    ```

- response

    ```bash
    {
    "success": true,
    "count": 123,
    "data": [ /* all docs */ ]
    }
    ```

### GET /health

Simple health check for the web app and config.

- Example 

    ```bash
    GET /health
    ```
- Example response
    ```bash
    {
    "status": "healthy",
    "timestamp": "2025-12-08T21:46:10.000Z",
    "cosmosdb": {
        "endpoint": "configured",
        "database": "RideauCanalDB",
        "container": "SensorAggregations"
        }
    }
    ```

## Deployment to Azure App Service

1. Create the Web App  
- In the Azure Portal, go to **App Services**  
- Click **Create** → **Web App**  
- Fill in the **Basics**:
  - Resource Group: `CST8916Final`
  - Name: `CST8916Final` 
  - Publish: `Code`
  - Runtime stack: `Node 24 LTS` 
  - Region: `Canada Central`
  - Pricing plan: `Free (F1)`  
  - Zone redundancy: `Disabled`
- Click **Review + create**, then **Create**

2. Connect Deployment from GitHub  
- Open your Web App in the portal  
- Go to **Deployment Center**  
- Source: `GitHub`  
- Sign in with your GitHub account  
- Organization: `<your GitHub username>`  
- Repository: `rideau-canal-dashboard`  (or any available name) 
- Branch: `main`  
- Click **Save** (or **Finish**) to enable continuous deployment

3. Configure Environment Variables  
- In the Web App, go to **Configuration** → **Application settings**  
- Click **New application setting** and add:

  - Cosmos container  
    - Name: `COSMOS_CONTAINER`  
    - Value: `SensorAggregations`  

  - Cosmos database  
    - Name: `COSMOS_DATABASE`  
    - Value: `RideauCanalDB`  

  - Cosmos DB endpoint (URI)  
    - Name: `COSMOS_ENDPOINT`  
    - Value: `https://rideau-canal-cosmos-db.documents.azure.com:443/`  

  - Cosmos DB primary key  
    - Name: `COSMOS_KEY`  
    - Value: `<CosmosDBPrimaryKey>` 


- Click **Save**, then let the app restart

After this, the dashboard should be live at:

```text
https://<your-app-name>.azurewebsites.net/
```

You can check:

```bash
https://<your-app-name>.azurewebsites.net/health
```

to make sure the app and Cosmos DB config are correct

## Dashboard Features (Details)

### Real-time updates

- Frontend calls `/api/latest` and `/api/status` every 30 seconds (configurable in `app.js`).
- The “Last updated” time on the page shows the time of the most recent fetch.

### Charts and visualizations

- Two Chart.js line charts:
  - Ice Thickness vs time (per location).
  - Surface Temperature vs time (per location).
- Uses `/api/history/:location` to plot about 1 hour of 5-minute windows.

### Safety status indicators

- Each location card has a safety badge that shows: **Safe**, **Caution**, or **Unsafe**.
- There is an overall canal status at the top:
  - **Safe** if all locations are Safe.
  - **Unsafe** if any location is Unsafe.
  - **Caution** otherwise.



## Troubleshooting

### 1. Blank dashboard / no data

- Check:

  ```text
  https://<app-name>.azurewebsites.net/health
  ```

If `cosmosdb.endpoint` is `"missing"`, your env vars are not set correctly in Azure.

Make sure:

- `COSMOS_ENDPOINT` and `COSMOS_KEY` are correct.
- Database and container names match exactly.
- Also check that your Stream Analytics job is **Running** and writing data into Cosmos.

### 2. API errors (500) or “Failed to fetch latest data”

- Open browser dev tools → **Network** tab.
- Call `/api/latest` and `/api/status` directly.
- If you see an error about Cosmos:
  - Recheck endpoint and keys.
  - Make sure the Cosmos firewall allows your App Service (or is open to “All networks” for this assignment).

### 3. Charts not showing lines

- Call `/api/history/DowsLake`, `/api/history/FifthAvenue`, `/api/history/NAC` directly.
- If `success` is `false` or `data` is empty:
  - Cosmos might not have enough historical documents yet.
  - Wait for more 5-minute windows or make sure Stream Analytics is configured correctly.

### 4. Deployed site still shows old code

- In App Service → **Deployment Center**, check that the latest deployment succeeded.
- Make sure you pushed your changes to the branch Azure is using (for example, `main`).
- You can also restart the app from the **Overview** blade to force a reload.
