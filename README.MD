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

## Run Locally


    ```bash
    npm start
    ```

OPEN

    ```bash
    http://localhost:3000
    ```
