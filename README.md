# 🎥 MediaMTX: RTMP Ingress to Local RTSP Egress Guide (Secured)

This guide details how to use **MediaMTX** (formerly `rtsp-simple-server`) to receive multiple incoming RTMP video streams and make them available as local RTSP streams on the same server. This setup is highly efficient as it avoids unnecessary transcoding.

**Public IP Address Used in this Example:** `STATIC_PUBLIC_IP`

---

## 🛠️ Step 1: Preparation and Installation

1. **Install MediaMTX:** Download and install the MediaMTX binary for your server's operating system.  
2. **Ensure Nginx is Removed:** Confirm that the Nginx RTMP module is fully uninstalled or disabled to avoid port conflicts with MediaMTX on the default RTMP port (`1935`).  
3. **Identify Ports:** MediaMTX will use the following default ports:
   * **RTMP Ingress:** `1935/TCP` (For pushing streams to the server)
   * **RTSP Egress:** `8554/TCP` (For reading streams from the server)

---

## ⚙️ Step 2: MediaMTX Configuration (`mediamtx.yml`)

The default configuration enables RTMP and RTSP, but you must add security credentials for publishing and may want to simplify RTSP transport.

### A. Required: Essential Security Configuration

Add the following block to the end of your `mediamtx.yml` file under the `paths:` section.  
This enforces authentication for anyone pushing a stream.

```yaml
paths:
  # The 'all_others' path applies these settings to all stream paths (e.g., /live/cam001)
  all_others:
    # --- REQUIRED SECURITY ---
    # Set a unique username and password for your cameras/encoders to use.
    publishUser: 'my_secure_rtmp_user'
    publishPass: 'A_very_strong_password_123!'
```

### B. Optional: Force TCP for RTSP (Recommended for Firewalls)

To simplify firewall rules and rely only on TCP (port 8554), change the `rtspTransports` setting under the **Global settings → RTSP server** section:

```yaml
# Find this section in your mediamtx.yml:
# Global settings -> RTSP server

# Change this line:
# rtspTransports: [udp, multicast, tcp]

# To this:
rtspTransports: [tcp]
```

---

## 🔒 Step 3: Firewall and Port Forwarding

Ensure inbound connections on the required ports can reach the MediaMTX service.

1. **OS/Server Firewall (Windows/Linux/Cloud Security Group):**  
   Ensure the following ports are explicitly allowed for **inbound** traffic:
   * **TCP Port 1935**
   * **TCP Port 8554**

2. **Router/External NAT (If applicable):**  
   You must configure **Port Forwarding** on your router to direct traffic from your public IP (`STATIC_PUBLIC_IP`) on ports 1935 and 8554 to the server's **local private IP** (e.g., `192.168.1.100`).

---

## 🚀 Step 4: Stream URLs and Access

Once MediaMTX is started and the firewall is configured, use these URLs.  
Each stream must use a **unique path** (e.g., `cam001`, `cam002`, etc.).

### 1. Ingress URL (Pushing TO the Server)

The camera/encoder configuration must use this URL and the credentials set in Step 2A.

```
rtmp://STATIC_PUBLIC_IP:1935/live/cam001
```

### 2. Egress URL (Reading FROM the Server)

Use a player like VLC Media Player to read the converted RTSP stream.  
This URL is only active **after** a stream is successfully pushing to the RTMP Ingress URL.

```
rtsp://STATIC_PUBLIC_IP:8554/live/cam001
```

---

## 🧩 Troubleshooting Common Errors

If you receive the error **"VLC is unable to open the MRL"** or the stream does not connect:

| Issue | How to Check | Solution |
| :--- | :--- | :--- |
| **Firewall/NAT Block** | Test port 8554 from an external network using a tool like `telnet` or an online port checker. | **Open/Forward** TCP ports 1935 and 8554 on ALL firewalls (OS, Cloud, Router). |
| **MediaMTX Not Running** | Check Task Manager (Windows) or use `sudo systemctl status mediamtx` (Linux). | **Restart** the MediaMTX service. Check logs for "address already in use" errors. |
| **No Active Stream** | Check the MediaMTX console output for a message confirming a publisher has connected to the RTMP port (1935). | **Confirm** the stream source (camera/OBS) is pushing, and is using the correct URL and the required `publishUser`/`publishPass`. |

---

✅ **Done!**  
Your MediaMTX server now securely receives RTMP streams and makes them available via local RTSP links.
