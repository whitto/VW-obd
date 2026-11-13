# VW OBD Console

A self-contained HTML diagnostic page for Chrome on macOS, developed with a 2006 manual Volkswagen Golf Mk5 in mind.

All JavaScript and CSS are included in `index.html`. There is no build step, package installation or application server dependency. Optional fault-reference links open external websites.

**Vehicle testing is pending.** Standard OBD functions and the experimental VW KWP2000 implementation have been checked with simulated responses. Compatibility with a physical Golf and a generic Bluetooth adapter has not been established.

## Requirements

- Chrome on a Mac with Bluetooth enabled.
- An ELM-compatible Bluetooth Low Energy (BLE) OBD adapter exposing the supported UART service.
- An HTTPS page or a local server on `localhost`.
- The vehicle parked, with the adapter connected and ignition switched on.

The current Bluetooth implementation uses these UUIDs, documented for the OBDLink CX.

| Purpose | UUID |
| --- | --- |
| Service | `0000fff0-0000-1000-8000-00805f9b34fb` |
| Notifications | `0000fff1-0000-1000-8000-00805f9b34fb` |
| Writes | `0000fff2-0000-1000-8000-00805f9b34fb` |

Generic adapters vary. The Bluetooth name or an “ELM327” label alone does not establish compatibility. Bluetooth Classic serial adapters and other BLE profiles need additional connection support.

[OBDLink CX adapter documentation](https://support.obdlink.com/support/solutions/articles/43000746707-obdlink-cx-adapter-notes)

## Open the page

### Run locally on your Mac

1. Download this repository using **Code → Download ZIP**, then extract it.
2. Open Terminal and change into the extracted directory containing `index.html`.
3. If Python 3 is installed, start a server bound to your Mac's loopback interface.

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

4. Open [http://localhost:8000](http://localhost:8000) in Chrome.
5. Keep Terminal open while using the page. Press **Control-C** in Terminal to stop the server.

Use localhost or HTTPS for Bluetooth access. If Chrome reports Bluetooth unavailable, check the browser, page origin and macOS Bluetooth permissions.

### Host with GitHub Pages

1. In this repository, open **Settings → Pages**.
2. Under **Build and deployment**, select **Deploy from a branch**.
3. Choose **main** and **/ (root)**, then save.
4. Wait for GitHub to publish the page and open the HTTPS address shown in Pages settings.

These are setup instructions. GitHub Pages activation has not been verified for this repository.

### Try without a car

Click **Try demo**. All displayed readings and faults in this mode are simulated. You can scan codes, start live readings, select VW modules and try clearing simulated faults.

Disconnect to leave demo mode before connecting a real adapter.

## Connect to the vehicle

1. Park the car and plug the adapter into its diagnostic socket.
2. Switch on the ignition.
3. Close any other diagnostic app connected to the adapter.
4. Open the page directly in Chrome.
5. Click **Connect adapter**, choose your adapter and grant the requested Bluetooth permission.
6. Wait for the connection status to confirm that standard OBD communication has succeeded.

The connection currently requires a successful standard OBD probe before enabling VW module operations.

## Read standard OBD faults

Click **Scan fault codes**. The page requests stored, pending and permanent engine/emissions codes.

A missing or unusable response is reported separately. “No faults reported” applies to responding services and does not establish that every vehicle module is fault-free.

Descriptions are included for a small selection of common codes. Other returned code numbers remain visible.

## Clear standard OBD faults

1. Record the fault codes and any diagnostic information you need.
2. Park with the engine off and ignition on.
3. Click **Clear codes** and review the confirmation.
4. Confirm to send the OBD clear request.
5. Review the automatic rescan.

Clearing can erase freeze-frame information and reset emissions readiness monitors. Permanent codes normally remain until the vehicle verifies that the fault has been resolved. Active faults may return. A successful clear acknowledgement does not establish that a repair has succeeded.

## Live readings

Click **Start live readings** to poll engine speed, coolant temperature and vehicle speed. Click **Stop live readings** to stop polling.

Unsupported or missing values appear as **Unavailable**. Scanning and clearing are disabled while live polling is active.

## Experimental VW KWP2000

The VW section implements a limited KWP2000 diagnostic session over Volkswagen TP 2.0 using ELM raw CAN commands.

1. Connect the adapter.
2. Select a **Control module**.
3. Click **Read module faults**.
4. Review the fault numbers, descriptions where available, status interpretation and raw values.
5. For an eligible module that was read successfully, use **Clear module faults** and confirm to clear and rescan it.

The module selector includes engine, ABS, climate control, central electronics, airbags, steering wheel, instruments, CAN gateway, immobiliser, door modules, steering assistance, convenience and radio. These are selectable targets. Their presence and successful access depend on the vehicle and adapter.

After an operation, the page closes the VW channel and attempts to restore standard OBD communication. If restoration fails, reconnect the adapter through the page.

### Current limitations

- This is an experimental implementation with no physical vehicle validation.
- Raw CAN support and sufficiently fast adapter/Bluetooth responses are required.
- This implementation targets KWP2000 over TP 2.0. It has no dedicated KWP2000 K-line or UDS module transport.
- It reads one selected module at a time. Automatic module discovery and a full-vehicle scan are not implemented.
- Full VW fault-code coverage and complete fault descriptions are not implemented.
- Supported fault responses contain a count followed by two-byte fault numbers and one status byte per fault. Other formats produce an error.
- Response-pending handling, transport retransmission and all controller-specific variations are not implemented.
- Module connection failures leave that module's fault state unknown.
- Fault codes identify reported conditions. They do not by themselves identify which part needs replacement.

### Readable VW fault descriptions

The current built-in dictionary includes these six codes.

| VW code | Description |
| --- | --- |
| 00532 | Battery supply voltage (B+) |
| 00778 | Steering angle sensor (G85) |
| 00928 | Driver door locking unit (F220) |
| 01217 | Driver front side-airbag igniter (N199) |
| 01314 | Engine control module |
| 01316 | ABS control module |

Each mapped code includes a link to its Ross-Tech reference. Unknown codes retain their raw number and show that a description is unavailable.

### Status interpretation

The page labels status using an ISO 14230 interpretation. This interpretation has not been confirmed for every VW module.

| Status field | Displayed meaning |
| --- | --- |
| Bits 6–5 = 00 | No fault detected at request |
| Bits 6–5 = 01 | Stored fault, currently absent |
| Bits 6–5 = 10 | Pending fault |
| Bits 6–5 = 11 | Active and stored fault |
| Bit 4 set | Test incomplete |
| Bit 4 clear | Test complete |
| Bit 7 set | Warning indicator requested |
| Bit 7 clear | No warning indicator requested |

If any of bits 0–3 are set, the page reports that the module-specific interpretation is unavailable. It always retains the original status byte and raw DTC.

### Airbag clearing

Airbag-module clearing is disabled. Ross-Tech specifically warns that clearing certain airbag igniter short-circuit faults can cause deployment. Follow the applicable repair procedure before attempting airbag fault clearing with appropriate equipment.

[Airbag fault 01217 reference](https://wiki.ross-tech.com/wiki/index.php/01217)

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| Connect adapter is disabled | Open directly in Chrome using HTTPS or localhost. Check Bluetooth availability and macOS permissions. |
| Adapter is absent from the chooser | Check power, BLE support and whether another application is using it. |
| Service or characteristic cannot be found | The adapter may use a different BLE profile or Bluetooth Classic. |
| No OBD-II response | Check ignition, adapter seating and vehicle communication. |
| Adapter rejects a raw CAN command | That firmware may not support the experimental VW transport. |
| TP channel or timing failure | The module may be absent, use another protocol, or exceed the adapter's timing capabilities. |
| Clear result unconfirmed | Read faults again before assuming they were cleared. |
| Reconnect required | Disconnect and reconnect through the page to restore standard OBD mode. |

Expand **Adapter responses** to inspect the most recent commands and replies when diagnosing a connection problem.

## Data handling

The page processes readings in the browser and keeps its response log in memory. The application code does not upload diagnostic results or save them to browser storage. Reloading clears the current session. Opening a fault-reference link contacts the external reference site.

## Files and validation

`index.html` is the complete application. No external script or stylesheet is required.

Before publication, the bundled scripts passed syntax checks. Earlier protocol simulations exercised channel setup, response reassembly, fault reading, clearing acknowledgement and malformed responses. The demo UI was also checked. Physical Bluetooth, vehicle timing and module-specific diagnostic results remain unverified.

## References

- [VW TP 2.0 protocol overview](https://icanhack.nl/knowledge-base/diagnostics/vw-tp20/)
- [KWP2000 service and status reference](https://andrewrevill.co.uk/ReferenceLibrary/OBDII%20Specifications%20-%20KWP2000%20DaimlerChrysler%202002.pdf)
- [Ross-Tech fault-code reference](https://wiki.ross-tech.com/wiki/index.php/Category:Fault_Codes)
