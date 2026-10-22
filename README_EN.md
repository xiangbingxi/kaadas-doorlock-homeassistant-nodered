# Kaadas Smart Door Lock Integration for Home Assistant & Apple HomeKit (Node-RED)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node-RED](https://img.shields.io/badge/Node--RED-Supported-red.svg)](https://nodered.org/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-blue.svg)](https://www.home-assistant.io/)
[![fnOS](https://img.shields.io/badge/fnOS-Tested-brightgreen.svg)](https://www.fnnas.com/)

[**简体中文文档**](./README.md)

---

Integrate your Kaadas smart door lock with **Home Assistant** using Node-RED to poll the Kaadas cloud API. The integration can optionally be bridged to **Apple HomeKit**, enabling lock event monitoring, unlock credential identification, battery monitoring, visitor doorbell snapshots, security alarm snapshots, and Apple Home notifications.

Node-RED retrieves lock events and device information from the Kaadas cloud API, processes the returned data, and updates Home Assistant entities. Combined with local image caching, a camera entity, and HomeKit Bridge, this provides a way to connect your smart lock to your home automation system.

> 💡 **Important Notes**
>
> 1. The project was initially configured for Docker-based deployment on fnOS (Feiniu OS).
> 2. The lock model actually tested is the **Kaadas K20 Pro Max**. Other models may use different API fields or response structures, and compatibility with other models has not been verified.
> 3. Kaadas cloud API parameters, authentication methods, and response structures may change depending on the app version, lock model, or server implementation. Always refer to your own packet capture results.
> 4. The local snapshot path, such as `/config/www/lock_latest.jpg`, must match the actual volume mounts between Node-RED and Home Assistant.
> 5. **HomeKit integration is optional.** If you only need lock events, battery information, and snapshots in Home Assistant, you do not need to configure HomeKit Bridge.
> 6. This project relies on the Kaadas cloud API; it does not provide direct local-network control of the lock. Cloud API changes, connectivity problems, or expired credentials may affect functionality.

---

## 🧪 Tested Environment

The following software and hardware configuration has been tested:

| Component | Tested Configuration |
| :--- | :--- |
| Smart Lock | Kaadas K20 Pro Max |
| Home Assistant Installation | Home Assistant Container |
| Home Assistant Core | 2026.8.1 |
| Home Assistant Frontend | 20260729.6 |
| Node-RED | 5.0.4 |
| Node-RED Deployment | Docker container |

These are the configurations verified so far. This does not imply that other lock models or software versions have been tested.

### Verified Features

- **Lock event notifications:** Each lock or unlock event can trigger the corresponding event-processing and notification workflow.
- **Unlock credential identification:** The integration retrieves user IDs associated with fingerprint, face recognition, and other configured unlocking credentials, where provided by the API.
- **Consistent user IDs:** Repeated unlock events using the same credential return the same user ID, while different credentials can return different IDs.
- **Battery updates:** The lock's remaining battery level can be updated in Home Assistant.
- **Doorbell snapshot updates:** When a visitor presses the doorbell, the snapshot can be updated in Home Assistant.
- **HomeKit snapshot synchronization:** After the relevant HomeKit configuration is completed, the doorbell snapshot can be synchronized for display in Apple Home.

> **About user IDs:** The integration retrieves an identifier returned by the lock API. This identifier corresponds to an unlocking credential configured on the lock; it does not directly reveal a person's real name. To identify household members, map the returned IDs to the fingerprints, face profiles, or other credentials configured on your own lock.

---

## 📸 Showcase

### 1. Apple HomeKit Integration & Snapshots (Optional)

| Apple Home App Overview | Doorbell Snapshot View |
| :---: | :---: |
| ![HomeKit Overview](images/homekit_overview.png) | ![HomeKit Camera](images/homekit_camera.png) |
| *Fig. 1: Lock status and a doorbell snapshot thumbnail in Apple Home* | *Fig. 2: Full-screen snapshot view with image orientation correction* |

---

### 2. Home Assistant Dashboard & Activity Log

| Home Assistant Dashboard | Lock Activity Log |
| :---: | :---: |
| ![HA Overview](images/ha_overview.png) | ![HA Log](images/ha_log.png) |
| *Fig. 3: Home Assistant dashboard showing lock status and the latest snapshot* | *Fig. 4: Lock events and related status information* |

---

### 3. Node-RED Flow Topology

![Node-RED Architecture](images/flow.png)

*Fig. 5: Node-RED flows covering lock event logs, device and battery information, doorbell snapshots, and alarm snapshot processing. Actual functionality depends on the imported flow and your configuration.*

---

## 🌟 Acknowledgements & References

The foundational approach to lock event polling and device information retrieval was inspired by the following open-source project and community discussion:

- GitHub Repository: [MF-142/kaadas](https://github.com/MF-142/kaadas)
- Hassbian Forum Thread: [Kaadas Smart Lock Integration (Node-RED)](https://bbs.hassbian.com/thread-24121-1-1.html)

Building on these references, this project integrates lock event processing, snapshot downloading and local caching, and Home Assistant and Apple HomeKit support.

Key extensions include:

1. **Visitor Doorbell Snapshots:** Retrieves snapshots captured when a visitor presses the doorbell and saves them to a local cache.
2. **Security Alarm Snapshots:** Processes snapshot information returned by the cloud API for supported alarm events, such as repeated authentication failures or tampering alerts. Actual support depends on the API response.
3. **Aliyun OSS Image Rotation:** Uses image-processing parameters to correct snapshots that appear sideways or upside down due to the camera's physical orientation.
4. **Apple HomeKit Integration:** Uses Home Assistant camera and doorbell-related entities with HomeKit Bridge to expose relevant information to Apple Home.

---

## ✨ Features

- 🚪 **Lock Events & Activity Logs:** Retrieves lock events and synchronizes them with Home Assistant. The workflow can identify the unlocking method and associated user ID when these details are returned by the API.
- 👤 **Unlock Credential Identification:** Distinguishes unlocking credentials using the IDs returned by the lock API. Repeated events from the same credential retain a consistent ID.
- 🔋 **Battery & Device Monitoring:** Retrieves the remaining battery percentage and other device information exposed by the API.
- 🔔 **Automatic Doorbell Snapshot Updates:** Fetches cloud snapshots after a visitor presses the doorbell and updates the local image cache.
- 🖼️ **Snapshot Orientation Correction:** Supports image rotation using Aliyun OSS image-processing parameters.
- 🚨 **Security Alarm Processing:** Processes supported alarm information returned by the API and can trigger configured snapshot or notification actions.
- 🏠 **Home Assistant Integration:** Uses Node-RED and Home Assistant nodes to expose lock events, device information, and snapshots to the home automation system.
- 🍎 **Optional Apple HomeKit Integration:** Uses HomeKit Bridge to synchronize supported entities for display in Apple Home.

> The lock event processing, credential ID consistency, battery updates, doorbell snapshot updates, and HomeKit snapshot synchronization have been tested in the environment listed above. The exact alarm types, notification behavior, and compatibility with other lock models depend on the API response and individual configuration.

---

## 🛠️ Prerequisites

Before getting started, prepare the following:

- A working Home Assistant instance.
- A running Node-RED instance.
- Network access to the Kaadas cloud service.
- A packet capture tool for inspecting requests from the Kaadas mobile app.
- HomeKit Bridge, if you want to integrate the lock with Apple Home.
- A directory accessible to both Node-RED and Home Assistant if you want to display locally cached snapshots.

### 1. Home Assistant Setup

#### 1.1 Install Node-RED Companion

1. Search for and install the `Node-RED Companion` integration through HACS.
2. Open **Settings → Devices & Services**.
3. Add and enable the integration.
4. Confirm that the Home Assistant nodes in Node-RED can connect to your Home Assistant instance.

> The interface and configuration steps may vary depending on the versions of Node-RED, Home Assistant, and the related node packages.

#### 1.2 Configure Generic Camera

To display the locally cached snapshot as a Home Assistant camera, configure a Generic Camera in `configuration.yaml`, or use a supported integration setup interface.

Example:

```yaml
camera:
  - platform: generic
    name: "Kaadas Doorbell Camera"
    still_image_url: "http://127.0.0.1:8123/local/lock_latest.jpg"
    verify_ssl: false
```

**Adjust the configuration to match your environment:**

- `/local/lock_latest.jpg` corresponds to `www/lock_latest.jpg` in the Home Assistant configuration directory.
- Home Assistant must be able to access the image file.
- `127.0.0.1` refers to the current network namespace. If Node-RED and Home Assistant run in separate Docker containers, their respective loopback addresses do not refer to the same container.
- If another container or device needs to access Home Assistant, use the appropriate host or service address for your network configuration.
- Generic Camera configuration options may differ across Home Assistant versions. Use the configuration method supported by your installed version.

After configuration, verify that Home Assistant can load the snapshot.

#### 1.3 Configure HomeKit Bridge (Optional)

> Skip this section if you do not want to integrate the lock with Apple Home.

1. Open **Settings → Devices & Services** in Home Assistant.
2. Add or open `HomeKit Bridge`.
3. Configure the bridge to include the relevant doorbell and camera entities.
4. Follow the pairing process provided by Home Assistant to add the bridge to Apple Home.

Entity IDs used in the original flow may include:

- Doorbell entity: `binary_sensor.kaadas_doorbell`
- Snapshot camera entity: `camera.men_suo_mao_yan_zhua_pai`

These IDs are examples only. **Use the entity IDs that actually exist in your Home Assistant instance**, and verify their entity types, states, and HomeKit Bridge compatibility.

If you want to use doorbell or camera features in HomeKit, verify that the corresponding Home Assistant entities update correctly and that the snapshot URL is accessible.

---

### 2. Directory Mapping (Docker / fnOS Example)

Snapshots downloaded by Node-RED must be saved to a directory that Home Assistant can access, typically the `www` directory under the Home Assistant configuration directory.

Example directory mapping:

| Item | Example Path |
| :--- | :--- |
| Home Assistant host configuration directory | `/vol1/1000/docker/homeassistant/config` |
| Home Assistant image directory | `/vol1/1000/docker/homeassistant/config/www` |
| Target path inside the Node-RED container | `/config/www/lock_latest.jpg` |
| Home Assistant local image URL | `/local/lock_latest.jpg` |

These paths are examples. Adjust them to match your own Docker volume configuration.

#### Directory Mapping Requirements

- The Home Assistant configuration directory should contain a `www` folder.
- Node-RED must have write access to the snapshot directory.
- If Node-RED and Home Assistant run in separate Docker containers, ensure that Node-RED can access the directory through a shared volume or another appropriate mechanism.
- If the containers do not share the directory, writing to `/config/www` inside the Node-RED container does not automatically make the file available to Home Assistant.

The default output path in the Node-RED `覆盖保存为本地图片` (save/overwrite local image) node is:

```text
/config/www/lock_latest.jpg
```

Adjust this path according to your actual volume mappings.

> **Important:** If the snapshot does not appear, first verify that the image file is actually written to Home Assistant's `www` directory. Then check the camera URL and Docker networking configuration.

---

### 3. Retrieve Kaadas API Credentials via Packet Capture

This project uses parameters from requests made by the Kaadas mobile app to access the cloud API. Tools that support HTTPS packet capture may be used, such as Stream or Thor on iOS, or Charles and mitmproxy on a computer.

The exact procedure depends on the app's networking implementation, certificate validation, and current API behavior. Only inspect requests from devices and accounts you are authorized to access.

The three key parameters to identify are:

| Parameter | Purpose | Description |
| :--- | :--- | :--- |
| `token` | Request authentication | Usually an authentication credential in a request header or parameter |
| `wifiSN` | Lock identification | Identifies the relevant lock device |
| `uid` | Account identification | Used by certain device-information requests |

The parameter names and locations must be verified against your own packet capture results.

**Security precautions:**

- `token`, `uid`, and device serial numbers may contain sensitive account or device information.
- Never publish real credentials in a public GitHub repository.
- Do not expose complete authentication values in screenshots, logs, or issue reports.
- Replace personal credentials with placeholders before sharing a flow template.
- If a credential may have been exposed, refresh or replace it using the methods supported by the service.

---

## 🚀 Quick Start

### 1. Import the Flow

1. Download [`flows-template.json`](./flows-template.json) from this repository.
2. Open the Node-RED editor.
3. Click the menu in the upper-right corner and select **Import**.
4. Select and upload the downloaded JSON file.
5. Check the imported nodes, connections, and configuration.

It is recommended to back up your existing Node-RED flows before importing the template.

### 2. Configure Your Credentials

Find the corresponding nodes in the imported flow and replace the template placeholders with the values obtained from your own packet capture.

| Node Name | Parameters to Configure |
| :--- | :--- |
| `获取最近门锁事件` (Get Recent Lock Events) | `YOUR_KAADAS_TOKEN`, `YOUR_LOCK_WIFI_SN` |
| `获取门锁信息请求数据设置` (Lock Info Request Settings) | `YOUR_KAADAS_TOKEN`, `YOUR_KAADAS_UID` |
| `访客门铃请求设置` (Visitor Doorbell Request Settings) | `YOUR_KAADAS_TOKEN`, `YOUR_LOCK_WIFI_SN` |
| `预警请求设置` (Alarm Request Settings) | `YOUR_KAADAS_TOKEN`, `YOUR_LOCK_WIFI_SN` |
| `推送抓拍图到手机` (Push Snapshot to Mobile, optional) | `YOUR_HA_NOTIFY_SERVICE` |

A mobile notification service might look similar to:

```text
notify.mobile_app_iphone
```

Use the actual service name configured in your Home Assistant instance. Depending on the node implementation, the field may expect the service name alone or a complete service path. Follow the field instructions in your imported flow.

**Note:** Node names and fields may differ between flow-template versions. If the nodes or parameters do not match this documentation, refer to the actual `flows-template.json` in the repository rather than copying incompatible fields.

### 3. Connect Home Assistant

1. Double-click a Home Assistant entity node, such as `凯迪仕门铃` (Kaadas Doorbell).
2. Select or create a Home Assistant server configuration.
3. Enter the correct Home Assistant address.
4. Provide a Long-Lived Access Token if required by the node.
5. Save the configuration and check the connection status.

If Node-RED and Home Assistant run in separate Docker containers, use an address reachable from the Node-RED container. Do not automatically use `127.0.0.1`.

A Long-Lived Access Token grants access to Home Assistant. Keep it secure and never commit it to a public repository or expose it in logs.

### 4. Verify Local Snapshot Storage

If doorbell snapshots are enabled, confirm that:

1. The Home Assistant configuration directory contains a `www` folder.
2. Node-RED can write to the target image directory.
3. The snapshot workflow creates or updates `lock_latest.jpg`.
4. Home Assistant can access the image through `/local/lock_latest.jpg`.
5. If HomeKit is enabled, the camera entity used by the bridge can load the latest snapshot.

### 5. Deploy

After completing the configuration:

1. Click **Deploy** in the upper-right corner of the Node-RED editor.
2. Check the connection status of the Home Assistant nodes.
3. Review the Node-RED debug panel and runtime logs.
4. Trigger a lock event and verify that it is retrieved and processed.
5. Test doorbell snapshots, battery updates, and any other enabled features.

A green `connected` status generally indicates that the relevant connection has been established. It does not guarantee that every cloud request, image-processing step, or entity update has succeeded. Verify each feature by triggering actual events.

---

## 🏗️ Execution Architecture

![Node-RED Architecture](images/flow.png)

The following diagram illustrates the main processing logic. The actual nodes, connections, and execution order are defined by `flows-template.json`.

```text
┌──────────────────────────────────────────────────────────────┐
│                  Lock Events & Device Information             │
│                                                              │
│  [Get Recent Lock Events] ──► [Parse Events] ──► [Update HA]   │
│                                                              │
│  [Get Lock Info] ──────────► [Parse Device Info] ──► [Battery]│
└──────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────┐
│                  Doorbell & Alarm Snapshots                  │
│                                                              │
│  [Visitor Doorbell /visit] ──► [Extract Snapshot] ──┐        │
│                                                     ├─► [Has Image?]
│  [Security Alarm /alarm] ────► [Extract Snapshot] ──┘        │
│                                                              │
│                            ▼                                 │
│                    [Download Snapshot]                        │
│                            ▼                                 │
│                  [OSS Image Rotation]                         │
│                            ▼                                 │
│       [Save to /config/www/lock_latest.jpg]                   │
│                            ▼                                 │
│                  [Update HA Camera]                           │
│                            ▼                                 │
│                 [HomeKit Bridge, Optional]                    │
└──────────────────────────────────────────────────────────────┘
```

### Doorbell and Snapshot Processing

1. The flow receives a visitor doorbell or alarm event from the cloud API.
2. It extracts the available snapshot information from the response.
3. The image is downloaded and processed as configured.
4. The image is saved to a local directory accessible by Home Assistant.
5. The Home Assistant camera entity loads the updated image.
6. If HomeKit Bridge is configured, the relevant camera or doorbell entity can be synchronized with Apple Home.

Successful image processing, notifications, and HomeKit presentation depend on the API response, file permissions, entity configuration, and the behavior of the relevant platforms.

---

## 🔍 Verification Checklist

After deployment, use this checklist to verify the integration:

- [ ] Node-RED can connect to Home Assistant.
- [ ] Requests to the Kaadas cloud API return data successfully.
- [ ] The flow retrieves and processes lock and unlock events.
- [ ] The corresponding Home Assistant entities or logs update correctly.
- [ ] Different unlocking credentials can return their corresponding user IDs.
- [ ] Repeated events from the same credential return a consistent user ID.
- [ ] The lock's battery level updates correctly.
- [ ] The flow retrieves a snapshot after a visitor presses the doorbell.
- [ ] The snapshot is saved to Home Assistant's `www` directory.
- [ ] The Home Assistant camera entity can load the latest image.
- [ ] If HomeKit is enabled, Apple Home displays the bridged entities correctly.
- [ ] If mobile notifications are enabled, the notification service triggers correctly.

Not every item is mandatory. HomeKit, mobile notifications, and alarm snapshots only need to be tested if you have enabled those features.

---

## 🧩 Troubleshooting

### 1. Node-RED Cannot Connect to Home Assistant

Check the following:

- Home Assistant is running.
- Node-RED can reach the Home Assistant network address.
- The server URL, port, and Long-Lived Access Token are correct.
- The relevant Home Assistant nodes are installed and configured correctly.

In Docker, `127.0.0.1` normally refers to the current container, not another container. Configure the address according to your actual network topology.

### 2. Lock Events Are Not Updating

Check the following:

- The `token` is valid.
- The `wifiSN` belongs to the correct lock.
- The request parameters match your packet capture results.
- The cloud API returns a valid response.
- The Node-RED debug panel does not show request or parsing errors.

The cloud API may change with app or server updates. Do not assume that all lock models use exactly the same request format.

### 3. User IDs Are Not Identified Correctly

Check the following:

- The response contains a user ID or credential identifier.
- The flow extracts the correct field.
- The response structure is consistent across different unlocking events.

A user ID is an identifier returned by the lock API and does not necessarily reveal the real name of the person using the lock. Its meaning depends on the device's API and credential configuration.

### 4. Battery Level Is Not Updating

Check the following:

- The device-information request succeeds.
- The response contains a battery field.
- Node-RED parses the field correctly.
- The corresponding Home Assistant entity updates as expected.

If the request succeeds but the entity does not change, inspect the field mapping and entity-update nodes.

### 5. Doorbell Snapshots Are Not Displayed

Check the following:

- The doorbell event is triggered.
- The cloud response contains a valid snapshot URL.
- The image-download node executes successfully.
- Node-RED has write permission for the target directory.
- The image is actually saved to Home Assistant's `www` directory.
- The camera entity's image URL is correct.
- The Docker containers have the required shared directory mapping.

### 6. Snapshot Orientation Is Incorrect

The flow can use an Aliyun OSS image-processing parameter to rotate the image, for example:

```text
?x-oss-process=image/rotate,90
```

Adjust the rotation angle according to the actual image orientation. Confirm that the image URL supports the relevant processing parameter. If the service does not support it, the image may not rotate as expected.

### 7. Doorbell or Snapshot Does Not Appear in HomeKit

Check the following:

- The corresponding entity exists in Home Assistant.
- The camera entity can display the image correctly.
- HomeKit Bridge is configured and paired.
- The required entities are included in the bridge.
- The entity types, states, and features are supported by your installed Home Assistant and Apple Home versions.

A camera image working in Home Assistant does not guarantee that every HomeKit device or operating system version will display doorbell notifications in exactly the same way.

### 8. Can Other Kaadas Models Be Used?

The model currently verified is the **Kaadas K20 Pro Max**.

Other models may differ in API endpoints, field names, response structures, credentials, or snapshot handling. The processing approach may be adaptable, but packet capture and testing are required. Direct compatibility with other models is not guaranteed.

---

## 🔐 Security & Privacy

This project involves cloud API access, device identifiers, authentication credentials, and home automation data.

Please observe the following precautions:

- Never publish real `token`, `uid`, `wifiSN`, or Long-Lived Access Token values.
- Do not upload Node-RED flows containing personal credentials to a public repository.
- Remove sensitive information from logs and screenshots before sharing them.
- Configure access permissions for Home Assistant, Node-RED, and Docker appropriately.
- Understand the privacy and security implications of using the Kaadas cloud API before deployment.

This is a community-developed integration and is not an official Home Assistant integration provided by Kaadas. Users are responsible for verifying that the configuration is appropriate for their own devices and network environments.

---

## 📄 License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).
