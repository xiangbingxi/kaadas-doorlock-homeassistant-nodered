# 凯迪仕智能门锁接入 Home Assistant & Apple HomeKit (Node-RED)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node-RED](https://img.shields.io/badge/Node--RED-Supported-red.svg)](https://nodered.org/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-blue.svg)](https://www.home-assistant.io/)

通过 Node-RED 轮询凯迪仕云端 API，将凯迪仕智能门锁无缝接入 Home Assistant，并桥接至 Apple HomeKit 实现**门锁状态监控、电量读取、访客门铃即时抓拍、预警抓拍以及 Apple 设备原生门铃卡片弹窗**。

---

## 🌟 致谢与参考

本项目的基础门锁状态轮询与电量读取思路参考了以下开源项目与社区讨论：
* GitHub 仓库：[MF-142/kaadas](https://github.com/MF-142/kaadas)
* 瀚思彼岸讨论帖：[接入凯迪仕智能锁（node-red） - 瀚思彼岸论坛](https://bbs.hassbian.com/thread-24121-1-1.html)

在此基础上，本项目进行了大幅重构与深度扩展，新增了**双轨抓拍流水线与 HomeKit 原生联动**：
1. **猫眼访客抓拍支持**：捕获按门铃瞬间的抓拍照并下载至本地缓存。
2. **异常报警抓拍支持**：支持连续输错密码锁定、防撬等异常抓拍。
3. **阿里云 OSS 图像自动旋转校正**：解决门锁猫眼感光元件物理横向导致的原图倒置/横向问题，自动纠正为竖屏。
4. **Apple HomeKit 原生门铃弹窗联动**：门铃触发时，iPhone、Apple Watch、Mac 及 Apple TV 屏幕顶部自动弹出带实时猫眼画面的原生大横幅。

---

## ✨ 核心特性

- 🚪 **人员开锁与日志记录**：实时同步指纹/密码/钥匙等开锁事件，自动映射用户编号（`User1`~`User4`）状态，关门上锁后自动复位。
- 🔋 **设备健康度监测**：定时同步门锁电池剩余百分比（`power`）、Wi-Fi 状态、锁具固件型号等至 Home Assistant 实体。
- 🔔 **门铃抓拍自动纠正**：访客按门铃后秒级拉取云端抓拍照，并追加阿里云 OSS `?x-oss-process=image/rotate,90` 参数纠正为正常竖屏视角。
- 🚨 **安全锁定预警**：开锁验证连续失败 5 次或防撬报警时，即刻拉取违规人员现场抓拍。
- 🍎 **HomeKit 原生体验**：结合 Home Assistant 的通用摄像头（Generic Camera）与门铃传感器，实现全屋苹果设备的原生门铃实况通知。

---

## 🛠️ 前置准备

### 1. Home Assistant 配置
1. **安装 Node-RED Companion**：
   * 在 HACS 中搜索并安装 `Node-RED Companion` 集成，安装完成后在 **设置 ➔ 设备与服务** 中添加并启用该集成。
2. **配置通用摄像头 (Generic Camera)**：
   * 在 Home Assistant 的 `configuration.yaml` 中添加以下配置（或在 **设备与服务 ➔ 通用摄像头** UI 界面配置）：
     ```yaml
     camera:
       - platform: generic
         name: "门锁猫眼抓拍"
         still_image_url: "[http://127.0.0.1:8123/local/lock_latest.jpg](http://127.0.0.1:8123/local/lock_latest.jpg)"
         verify_ssl: false
     ```
3. **配置 HomeKit 桥接**：
   * 在 **设置 ➔ 设备与服务 ➔ HomeKit Bridge** 中，将门铃实体 `binary_sensor.kaadas_doorbell`（类型为 `motion` 或 `doorbell`）与摄像头 `camera.men_suo_mao_yan_zhua_pai` 关联同步到 Apple 家庭。

---

### 2. Node-RED 目录映射 (Docker 部署必看)
为了使 Node-RED 下载的图片能被 Home Assistant 读取，请确保在部署 Node-RED 容器时，已将 Home Assistant 的 `www` 目录挂载进 Node-RED：
* **宿主机路径**：`/path/to/homeassistant/config/www`
* **Node-RED 容器内路径**：`/config/www`

---

### 3. 抓包获取凯迪仕凭据
使用抓包工具（如 iOS 上的 Stream / Thor，或电脑端 Charles / mitmproxy）抓取凯迪仕 App 请求，记录以下 3 个核心凭据：
* **`token`**：请求头中的认证凭据（形如 `GR6qN8sz...`）。
* **`wifiSN`**：门锁的唯一序列号（形如 `K2A12523...`）。
* **`uid`**：用户账号 ID（形如 `6062767...`）。

---

## 🚀 部署步骤

1. **导入流程**：
   * 下载本仓库的 [`flows-template.json`](./flows-template.json)。
   * 打开 Node-RED ➔ 点击右上角菜单 ➔ **导入 (Import)** ➔ 选择并上传该 JSON 文件。

2. **填入个人凭据**：
   在导入的流程中，找到带有注释标记的节点，替换为你的真实凭据：
   * **`获取最近门锁事件`**：填入 `YOUR_KAADAS_TOKEN` 与 `YOUR_LOCK_WIFI_SN`
   * **`获取门锁信息请求数据设置`**：填入 `YOUR_KAADAS_TOKEN` 与 `YOUR_KAADAS_UID`
   * **`访客门铃请求设置`**：填入 `YOUR_KAADAS_TOKEN` 与 `YOUR_LOCK_WIFI_SN`
   * **`预警请求设置`**：填入 `YOUR_KAADAS_TOKEN` 与 `YOUR_LOCK_WIFI_SN`
   * **`推送抓拍图到手机`**（可选）：填入你的 HA 手机通知服务名 `YOUR_HA_NOTIFY_SERVICE`（如 `mobile_app_iphone`）

3. **连接 Home Assistant**：
   * 双击流程中的任意 HA 实体节点（如 `凯迪仕门铃`），在 Server 配置中填入你的 Home Assistant 地址和长期访问令牌 (Long-Lived Access Token)。

4. **部署上线**：
   * 点击右上角 **“部署” (Deploy)**，观察节点下方显示绿色 `connected` 即表示配置成功！

---

## 🏗️ 运行流程架构

```text
┌─────────────────────────────────────────────────────────────┐
│  [访客门铃 /visit] ──► [提取访客抓拍] ──┐                     │
│                                       ├──► [判断是否有图片]  │
│  [异常报警 /alarm] ──► [提取预警抓拍] ──┘         │         │
└───────────────────────────────────────────────────┼─────────┘
                                                    ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. [下载抓拍图片] ➔ OSS自动旋转90° ➔ 覆盖保存为 /config/www/lock_latest.jpg
│ 2. [设置 ON] ➔ Kaadas_Doorbell 门铃触发 ➔ 延时2秒 ➔ [设置 OFF]
│ 3. [Apple HomeKit 联动] ➔ iPhone/Apple TV 弹出带实况抓拍的原生大横幅！
└─────────────────────────────────────────────────────────────┘
