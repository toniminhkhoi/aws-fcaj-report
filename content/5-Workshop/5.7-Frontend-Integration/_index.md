---
title: "Frontend Integration"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Overview and objectives

Run the React + Vite frontend locally, connect it to FastAPI on EC2 through HTTP REST, and display periodically refreshed telemetry, actuator controls, rule-based recommendations, and history for the sample room identified by `device_id = room_01`.

## Step 1 - Configure the React Frontend

The project uses React, Vite, TypeScript, Tailwind CSS, Axios, and Recharts. From Windows PowerShell:

```powershell
git clone <REPOSITORY_URL>
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Use the Node version required by `package.json` and keep the repository lockfile. The frontend runs locally outside AWS.

## Step 2 - Connect the Frontend to the FastAPI Backend

Use relative `/api` paths with the Vite development proxy:

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": {
        target: "http://<EC2_PUBLIC_IP>:8000",
        changeOrigin: true,
      },
    },
  },
});
```

Restart Vite after changing its configuration. If the project uses `VITE_API_BASE_URL`, store it in an ignored `.env.local` rather than duplicating the EC2 URL across components.

The main requests are:

```text
GET  /api/health
GET  /api/devices/room_01/latest
GET  /api/devices/room_01/history
POST /api/devices/room_01/commands
```

The **LIVE AWS** badge must depend on a successful backend/API response, not only on the React page loading.

## Step 3 - Display Live Telemetry

Render three cards for temperature, humidity, and light intensity. The frontend refreshes data periodically through REST polling, so the dashboard is described as **near-real-time**, not as a fixed-latency real-time system. Provide loading, retryable error, and last-updated states where appropriate.

## Step 4 - Render the Remote Control Panel

The control panel supports:

- `FAN_ON` / `FAN_OFF`;
- `LIGHT_ON` / `LIGHT_OFF`;
- `CURTAIN_OPEN` / `CURTAIN_CLOSE`; and
- switching between `MODE_MANUAL` and `MODE_AUTO`.

Disable a selected control while its request is in flight, prevent duplicate pending commands, and distinguish command acceptance from physical execution. The backend-provided command ID/state should be used for tracking instead of relying only on local UI state.

<p align="center">
  <img src="/images/5-Workshop/5.7-frontend/dashboard-overview-control-panel.png"
       alt="React Vite IoT dashboard with live telemetry and actuator controls"
       width="100%" />
</p>

*Figure 13. The React + Vite dashboard displaying near-real-time telemetry and controls for the fan, light, and curtain for the sample room identified by `device_id = room_01`.*

Figure 13 shows the locally running React + Vite interface, the EC2 FastAPI/RDS PostgreSQL/React Vite stack label, three **LIVE AWS** telemetry cards, and controls for the fan, light, curtain, and mode. The UI can display Manual Override or Auto according to its current state.

## Step 5 - Display Rule-Based Analysis and History

### Rule-Based Analysis and Recommendations

The recommendation panel evaluates fixed conditions based on temperature, humidity, light, and time. Examples in the interface include recommending that the fan be turned off outside working hours, reporting an acceptable humidity range, and recommending curtain movement when light is high. This is deterministic rule-based analysis; it is not machine learning, predictive analytics, or a trained model.

The history charts display temperature, humidity, and light data retrieved from Amazon RDS through:

```text
GET /api/devices/room_01/history
```

<p align="center">
  <img src="/images/5-Workshop/5.7-frontend/dashboard-analysis-history.png"
       alt="Rule-based recommendations and telemetry history charts"
       width="100%" />
</p>

*Figure 14. Rule-based recommendations and historical temperature, humidity, and light charts retrieved from Amazon RDS.*

## Step 6 - Expected Results

- The local React + Vite frontend loads successfully.
- The **LIVE AWS** badge reflects a successful backend/API request.
- Telemetry for `device_id = room_01` appears on the three cards.
- Fan, light, curtain, and mode controls are rendered.
- The history charts receive records from the history endpoint.
- Recommendations are described as rule/threshold based, not as machine learning.
- Telemetry refresh is described as near-real-time REST polling without an unsupported latency claim.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| Vite proxy returns 404 | Verify the proxy key, target, plural API path, and restart Vite |
| Browser reports CORS | Confirm the request uses the proxy or review the backend CORS policy |
| Telemetry cards are empty | Check `/api/health`, latest response shape, `device_id`, and loading/error handling |
| History chart is blank | Check the history response, timestamps, and empty-array handling |
| Status is always online | Bind the badge to a real health/API response instead of component mount |
| Commands are duplicated | Disable in-flight controls and check whether a matching command is pending |
| UI reports success too early | Display request acceptance separately from ACK/`Executed` and physical action |

Next: [run end-to-end validation](../5.8-End-to-End-Testing/).
