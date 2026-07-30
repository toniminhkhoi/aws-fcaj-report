---
title: "End-to-End Testing and Validation"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Step 1 - Define the validation strategy

Validate the command path by correlating evidence from several layers:

```text
React + Vite UI
    -> FastAPI command endpoint
    -> PostgreSQL commands table
    -> YOLO UNO command polling
    -> Physical actuator response
    -> ACK request
    -> Command state: Executed
```

The sample room is identified by `device_id=room_01`. Record the test time and command ID where available, and redact credentials, private addresses, and account identifiers before publishing evidence.

No single screenshot proves the complete path. Frontend/API evidence confirms browser requests, hardware evidence confirms the physical response, and PostgreSQL evidence confirms command persistence and state. An end-to-end result is accepted only after these evidence layers have been correlated.

## Step 2 - Validate frontend API requests

1. Start the React + Vite frontend and open the control panel.
2. Open **Chrome DevTools > Network** and select the **Fetch/XHR** filter.
3. Observe the periodic `latest` and `history` requests.
4. Operate a supported control and confirm that a `commands` request is sent.
5. Verify that the displayed requests receive HTTP 200 responses.

![Frontend API requests in Chrome DevTools](/images/5-Workshop/5.8-validation/control-panel-api-request.png)

*Figure 15. Chrome DevTools confirms that the frontend requests to `latest`, `history`, and `commands` receive HTTP 200 responses from the FastAPI backend.*

The screenshot shows telemetry on the dashboard and repeated XHR requests for `latest`, `history`, and `commands`. It confirms successful frontend-to-backend communication during periodic REST polling; it does not establish a fixed response-time guarantee.

## Step 3 - Test fan control

1. Place the device in manual mode when required by the firmware behavior.
2. Send `FAN_ON` or `FAN_OFF` from the dashboard.
3. Compare the selected dashboard control with the physical fan response.
4. Return to automatic mode and confirm that rule-based control resumes.
5. Check that the related command is acknowledged.

![Dashboard and physical fan control validation](/images/5-Workshop/5.8-validation/dashboard-hardware-control-fan.png)

*Figure 16. End-to-end fan-control validation by comparing the dashboard state with the physical fan response.*

The captured frame is used to correlate the dashboard control state with an observable physical fan response. It does not, by itself, prove database persistence or the ACK transition.

| ID | Test | Expected result | Evidence | Status |
| :--- | :--- | :--- | :--- | :---: |
| T01 | Send `FAN_ON` or `FAN_OFF` | The fan changes state and the command is acknowledged | Figure 16, demonstration video, and command record | **Pass** |
| T02 | Return to automatic mode | The device resumes deterministic rule-based automatic control | Dashboard state and hardware demonstration | **Pass** |

## Step 4 - Test light control

1. Send `LIGHT_ON` from the dashboard and observe the physical LED.
2. Send `LIGHT_OFF` and verify that the LED turns off.
3. Inspect the related command record and confirm its final `Executed` state.

![Dashboard and physical LED control validation](/images/5-Workshop/5.8-validation/dashboard-hardware-control-led.png)

*Figure 17. End-to-end light-control validation, with the dashboard command confirmed by the physical LED state.*

| ID | Test | Expected result | Evidence | Status |
| :--- | :--- | :--- | :--- | :---: |
| T03 | Send `LIGHT_ON` | The physical LED turns on | Figure 17 and demonstration video | **Pass** |
| T04 | Send `LIGHT_OFF` | The physical LED turns off | Hardware demonstration video | **Pass** |
| T05 | Inspect the command state | The acknowledged light command has state `Executed` | PostgreSQL command record | **Pass** |

## Step 5 - Test curtain control

1. Send `CURTAIN_OPEN` and observe the servo movement.
2. Send `CURTAIN_CLOSE` and observe the reverse movement.
3. Compare both movements with the open and closed positions configured in the firmware.
4. Inspect the related command record and confirm its final `Executed` state.

![Dashboard and physical curtain servo control validation](/images/5-Workshop/5.8-validation/dashboard-hardware-control-curtain.png)

*Figure 18. End-to-end curtain-control validation by comparing the OPEN/CLOSE dashboard command with the physical servo movement.*

The acceptance criterion uses the positions configured in the firmware; it does not assume a specific angle in this report.

| ID | Test | Expected result | Evidence | Status |
| :--- | :--- | :--- | :--- | :---: |
| T06 | Send `CURTAIN_OPEN` | The servo moves to the configured open position | Figure 18 and demonstration video | **Pass** |
| T07 | Send `CURTAIN_CLOSE` | The servo moves to the configured closed position | Figure 18 and demonstration video | **Pass** |
| T08 | Inspect the command state | The acknowledged curtain command has state `Executed` | PostgreSQL command record | **Pass** |

## Step 6 - Verify command state

For each supported operation, the dashboard sends a command to FastAPI and the backend stores it in PostgreSQL. YOLO UNO polls for the latest pending command, executes the corresponding actuator action, and calls the ACK endpoint. The backend then changes that command from `Pending` to `Executed`.

Use the API and database checks below to correlate the same `device_id`, command, ID, and state:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

```sql
SELECT id, device_id, command, state, timestamp
FROM commands
ORDER BY id DESC
LIMIT 6;
```

[Figure 9 in section 5.5](../5.5-Backend-and-Database/) provides the PostgreSQL evidence used for this layer. It shows recent commands for `device_id=room_01`, including `CURTAIN_OPEN`, `CURTAIN_CLOSE`, `MODE_AUTO`, and `LIGHT_OFF`, with state `Executed` after acknowledgement.

The dashboard-to-hardware behavior is also available in the [Google Drive demonstration video](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing). The screenshots were extracted from the video and may therefore appear slightly blurred; use the video to inspect the control sequence in more detail.

## Step 7 - Review expected results

The validation is successful when all of the following are observed:

- Frontend requests to `latest`, `history`, and `commands` receive HTTP 200 responses.
- The dashboard displays telemetry associated with `device_id=room_01`.
- The physical fan, LED, and curtain servo respond to their supported commands.
- The device acknowledges completed commands and their database state becomes `Executed`.
- Published evidence contains no credentials, private addresses, or account identifiers.
- Conclusions distinguish browser/API evidence, physical hardware evidence, and database-state evidence.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| `latest`, `history`, or `commands` does not return HTTP 200 | Check the Vite proxy/base URL, FastAPI route, backend service, and browser console |
| The dashboard changes but the actuator does not respond | Confirm manual/automatic mode, device Wi-Fi, polling, command spelling, wiring, and power |
| A command remains `Pending` | Check device polling, the command ID used by the ACK endpoint, and backend logs |
| UI and PostgreSQL show different states | Compare the same command ID and refresh the newest database row after ACK |
| The servo moves incorrectly | Check the configured open/closed positions and the servo power connection |
| Evidence contains sensitive information | Redact and recapture the image; rotate any exposed secret before continuing |

Next: [configure and validate CloudWatch](../5.9-CloudWatch-Monitoring/).