---
title: "End-to-End Testing and Validation"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Overview and objectives

Validate each boundary independently, then correlate the complete telemetry and command paths. Use FastAPI `/docs` or `/openapi.json` to confirm the schema of the deployed version before testing. The sample room is identified by `device_id=room_01`.

This section combines a complete test matrix with browser, API, PostgreSQL, and physical-hardware evidence. Each item states what its evidence proves so that a single screenshot is not used to support conclusions beyond what is visible.

## Step 1 - Establish the test protocol and validation strategy

1. Record the date, tester, application commit ID, firmware build, AWS Region, and `device_id`.
2. Redact credentials, private addresses, and account identifiers from published evidence.
3. Capture the relevant request/response, logs, SQL state, device output, dashboard state, and physical response.
4. Enter the observed result in **Actual/evidence** and assign **Pass**, **Fail**, or **Not Run** only after checking it.
5. Restore the hardware and services to a safe state after failure tests.

Correlate the command path across the following layers:

```text
React + Vite UI
    -> FastAPI command endpoint
    -> PostgreSQL commands table
    -> YOLO UNO command polling
    -> Physical actuator response
    -> ACK request
    -> Command state: Executed
```

No single screenshot proves this complete path. Frontend/API evidence confirms browser requests, hardware evidence confirms physical response, and PostgreSQL evidence confirms persistence and command state. An end-to-end result is accepted only after the related evidence layers have been correlated.

## Step 2 - Execute and record the test matrix

| ID | Objective | Preconditions | Steps | Expected result | Actual/evidence | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Backend health | Backend service is active | Send `GET /api/health` | HTTP 200 and the documented health response | Figure 8 in section 5.5 and the health response | **Pass** |
| T02 | Submit telemetry | OpenAPI schema is known and RDS is reachable | POST one valid payload for `device_id=room_01` | Successful response and a stored telemetry record | Telemetry response and corresponding `latest`/`history` data | **Pass** |
| T03 | Retrieve latest telemetry | T02 is complete | Send `GET /api/devices/room_01/latest` | The newest record for `room_01` is returned | Figure 15 and the dashboard telemetry cards | **Pass** |
| T04 | Retrieve telemetry history | Multiple records exist | Send `GET /api/devices/room_01/history` | Ordered history for `room_01` is returned | Figures 14 and 15, including the history charts | **Pass** |
| T05 | Create a command | Backend and database are available | POST one supported command | A command is created with an ID and initial `Pending` state | Figure 15 `commands` request and Figure 9 command records | **Pass** |
| T06 | Poll for a command | YOLO UNO is online | Observe polling after T05 | The device receives the correct command and ID | Hardware demonstration video | **Pass** |
| T07 | Control the fan | Fan is wired and powered safely | Send `FAN_ON` and `FAN_OFF`; then return to automatic mode | The fan responds, the command is acknowledged, and automatic rule-based control resumes | Figure 16 and the hardware demonstration video | **Pass** |
| T08 | Control the light | LED/light is wired and powered safely | Send `LIGHT_ON`, then `LIGHT_OFF` | The physical LED/light matches both commands | Figure 17 and the hardware demonstration video | **Pass** |
| T09 | Control the curtain | Servo is wired and powered safely | Send `CURTAIN_OPEN`, then `CURTAIN_CLOSE` | The servo moves to the open and closed positions configured in firmware | Figure 18 and the hardware demonstration video | **Pass** |
| T10 | Verify the ACK lifecycle | A command from T05–T09 exists | Observe the ACK request and query the same command ID | The command changes from `Pending` to `Executed` | Figure 9 and the acknowledged command record | **Pass** |
| T11 | Verify PostgreSQL persistence | A database session is available | Query telemetry and commands after API refresh | Records remain available and can be queried again | Figure 9 and the repeated SQL query | **Pass** |
| T12 | Verify CloudWatch logs | CloudWatch Agent and log collection are configured | Generate a health or telemetry request | A corresponding backend event appears in the expected log stream | Backend log evidence in section 5.9 | **Pass** |

## Step 3 - Validate frontend API requests

1. Start the React + Vite frontend and open the control panel.
2. Open **Chrome DevTools > Network** and select **Fetch/XHR**.
3. Observe the periodic `latest` and `history` requests.
4. Operate a supported control and confirm that a `commands` request is sent.
5. Verify that the displayed requests receive HTTP 200 responses.

![Frontend API requests in Chrome DevTools](/images/5-Workshop/5.8-validation/control-panel-api-request.png)

*Figure 15. Chrome DevTools confirms that the frontend requests to `latest`, `history`, and `commands` receive HTTP 200 responses from the FastAPI backend.*

The screenshot shows telemetry on the dashboard and repeated XHR requests for `latest`, `history`, and `commands`. It confirms frontend-to-backend communication during periodic REST polling; it does not establish a fixed response-time guarantee.

## Step 4 - Test fan control

1. Place the device in manual mode when required by the firmware behavior.
2. Send `FAN_ON` and `FAN_OFF` from the dashboard.
3. Compare the dashboard state with the physical fan response.
4. Return to automatic mode and confirm that deterministic rule-based control resumes.
5. Check that the related command is acknowledged.

![Dashboard and physical fan control validation](/images/5-Workshop/5.8-validation/dashboard-hardware-control-fan.png)

*Figure 16. End-to-end fan-control validation by comparing the dashboard state with the physical fan response.*

The captured frame correlates the dashboard control with an observable physical response. By itself, it does not prove database persistence or the ACK transition.

## Step 5 - Test light control

1. Send `LIGHT_ON` from the dashboard and observe the physical LED/light.
2. Send `LIGHT_OFF` and verify that it turns off.
3. Inspect the related command record and confirm its final `Executed` state.

![Dashboard and physical LED control validation](/images/5-Workshop/5.8-validation/dashboard-hardware-control-led.png)

*Figure 17. End-to-end light-control validation, with the dashboard command confirmed by the physical LED state.*

## Step 6 - Test curtain control

1. Send `CURTAIN_OPEN` and observe the servo movement.
2. Send `CURTAIN_CLOSE` and observe the reverse movement.
3. Compare both movements with the open and closed positions configured in the firmware.
4. Inspect the related command record and confirm its final `Executed` state.

![Dashboard and physical curtain servo control validation](/images/5-Workshop/5.8-validation/dashboard-hardware-control-curtain.png)

*Figure 18. End-to-end curtain-control validation by comparing the OPEN/CLOSE dashboard command with the physical servo movement.*

The acceptance criterion uses the positions configured in the firmware; this report does not assume a specific servo angle.

## Step 7 - Run API and database checks and verify command state

From EC2 Linux Bash, verify backend health and telemetry retrieval:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Create telemetry using the camelCase fields documented in section 5.6. Create a command with `{ "command": "FAN_ON" }`. The Pydantic field also has the alias `Command`; because `populate_by_name=True`, the lowercase field `command` is accepted. A device record normally exists after the first valid telemetry request.

In PostgreSQL `psql`, inspect recent commands:

```sql
SELECT
    id,
    device_id,
    command,
    state,
    timestamp
FROM commands
ORDER BY id DESC
LIMIT 6;
```

A polling device may acknowledge a command quickly enough that a later query no longer shows `Pending`. Preserve the command POST response when it contains the initial state, then query the same command ID after ACK to confirm `Executed`.

The dashboard sends a command to FastAPI, and the backend stores it in PostgreSQL. YOLO UNO polls for the latest pending command, executes the corresponding actuator action, and calls the ACK endpoint. The backend then changes that command from `Pending` to `Executed`.

[Figure 9 in section 5.5](../5.5-Backend-and-Database/) provides the PostgreSQL evidence for this layer. It shows recent commands for `device_id=room_01`, including `CURTAIN_OPEN`, `CURTAIN_CLOSE`, `MODE_AUTO`, and `LIGHT_OFF`, in the `Executed` state after acknowledgement.

The dashboard-to-hardware behavior is also available in the [Google Drive demonstration video](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing). The screenshots were extracted from the video and may therefore appear slightly blurred; use the video to inspect the complete control sequence.

## Expected Result

Every T01–T12 row contains an observed **Actual/evidence** value and a **Pass**, **Fail**, or **Not Run** status. A passing end-to-end result correlates the appropriate device or command ID across the API, PostgreSQL, firmware, dashboard, physical actuator, and relevant logs.

The accepted results include HTTP 200 responses for `latest`, `history`, and `commands`; telemetry for `device_id=room_01`; physical fan, LED/light, and servo responses; ACK submission; and final command state `Executed`.

## Troubleshooting

Do not use the test plan to invent latency, throughput, availability, or reliability measurements. A failed test should record the failing layer, request or log evidence, owner, correction, and rerun result.

| Symptom | Check |
| :--- | :--- |
| `latest`, `history`, or `commands` does not return HTTP 200 | Check the Vite proxy/base URL, FastAPI route, backend service, and browser console |
| `Pending` is not visible | Preserve the command POST response, then query the same command ID after ACK |
| The dashboard changes but the actuator does not respond | Confirm manual/automatic mode, device Wi-Fi, polling, command spelling, wiring, and power |
| UI and PostgreSQL show different states | Compare the same command ID and refresh the newest database row after ACK |
| A command is repeated | Compare command IDs and separate an ACK retry from repeating the actuator action |
| The servo moves incorrectly | Check the configured open/closed positions and the servo power connection |
| The test cannot be reproduced | Record commit IDs, Region, device ID, timestamp/time zone, and exact preconditions |
| Evidence contains sensitive information | Redact and recapture the image; rotate any exposed secret before continuing |

Next: [configure and validate CloudWatch](../5.9-CloudWatch-Monitoring/).