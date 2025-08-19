# Using the TP-Link TL-WN722N for Wi-Fi Monitoring in Kali Linux

This guide provides step-by-step instructions for setting up the popular TP-Link TL-WN722N wireless USB adapter for monitor mode and packet injection in Kali Linux.

The TL-WN722N has come in several hardware versions over the years, each using a different chipset. It's crucial to identify your version to ensure you use the correct drivers and procedures.

*   **Version 1 (v1):** Uses the **Atheros AR9271** chipset. It is highly sought after because it supports monitor mode and packet injection out-of-the-box with standard Kali Linux drivers.
*   **Version 2/3 (v2/v3):** Uses the **Realtek RTL8188EUS** chipset. This version requires custom drivers to enable monitor mode and packet injection.

> **⚠️ Legal Disclaimer:** This guide is for educational and authorized testing purposes only. Unauthorized packet injection or monitoring of networks you do not own or have permission to test is illegal. Use this information responsibly.

## Prerequisites

*   A working installation of Kali Linux (or another Debian-based pentesting distro).
*   A TP-Link TL-WN722N USB adapter.
*   An internet connection (for downloading drivers if needed).
*   `sudo` or root privileges.

---

### Step 1: Identify Your Adapter Version

First, plug in your TL-WN722N adapter. Open a terminal and run the following command to list your USB devices and their chipsets.

```bash
lsusb
```

Look for a line corresponding to your wireless adapter.
*   If you see **`Atheros Communications, Inc. AR9271 802.11n`**, you have **Version 1**. It will work out-of-the-box. You can skip to **Step 3**.

*   If you see **`Realtek Semiconductor Corp. RTL8188EUS 802.11n Wireless Network Adapter`**, you have **Version 2 or 3**. You must install custom drivers. Proceed to **Step 2**.

### Step 2: Install Drivers (for v2/v3 Only)

If you have the Realtek `RTL8188EUS` chipset, you need to install drivers that support monitor mode and packet injection. The `aircrack-ng` team recommends a specific driver set available via DKMS (Dynamic Kernel Module Support).

1.  **Update your package lists:**
    ```bash
    sudo apt update
    ```

2.  **Install the necessary drivers and headers:**
    ```bash
    sudo apt install -y realtek-rtl8188eus-dkms
    ```

3.  **Reboot your system** to ensure the new kernel module is loaded correctly.
    ```bash
    sudo reboot
    ```

After rebooting, your v2/v3 adapter should be ready.

### Step 3: Enable Monitor Mode

Now, we will put the wireless adapter into monitor mode using the `aircrack-ng` suite.

1.  First, identify the name of your wireless interface (it's usually `wlan0`).
    ```bash
    iwconfig
    ```

2.  It's a good practice to check for and kill processes that can interfere with monitor mode.
    ```bash
    sudo airmon-ng check kill
    ```

3.  Start monitor mode on your interface. Replace `wlan0` if your interface has a different name.
    ```bash
    sudo airmon-ng start wlan0
    ```

This command will create a new virtual interface for monitoring. **Note the name of the new interface** in the output. It will typically be `wlan0mon` or `mon0`. We will use `wlan0mon` in the following examples.

### Step 4: Test Packet Injection

Before performing any complex attacks, it's best to confirm that packet injection is working correctly.

Run the following command, replacing `wlan0mon` with your monitor mode interface name.

```bash
# Replace wlan0mon with your actual monitor interface name
sudo aireplay-ng --test wlan0mon
```

You should see output indicating that injection is working. Look for a message like `Injection is working!` or a high percentage of packets being acknowledged.

### Step 5: Example Usage (Deauthentication Attack)

Packet injection can be used for many purposes. A common example is sending deauthentication packets to disconnect a specific client from a network.

> **Note:** Only perform this on a network you own and have explicit permission to test.

To send a single deauthentication packet to a client, you need the BSSID (MAC address) of the Access Point and the MAC address of the client you want to disconnect.

```bash
# Usage: sudo aireplay-ng -0 <count> -a <AP_BSSID> -c <Client_MAC> <interface>
# Example:
sudo aireplay-ng -0 1 -a 00:11:22:AA:BB:CC -c 33:44:55:DD:EE:FF wlan0mon
```

*   `-0 1`: Perform one deauthentication attack.
*   `-a 00:11:22:AA:BB:CC`: The MAC address of the target Access Point.
*   `-c 33:44:55:DD:EE:FF`: The MAC address of the target client.
*   `wlan0mon`: Your monitor mode interface.

### Step 6: Stop Monitor Mode

When you are finished, you can stop monitor mode and return your adapter to its normal "managed" state.

```bash
# Replace wlan0mon with your monitor interface name
sudo airmon-ng stop wlan0mon
```

This will remove the monitor interface and should restore the original interface (`wlan0`) so you can connect to Wi-Fi networks again. You may need to restart your network manager if you have trouble reconnecting:

```bash
sudo service network-manager restart
```

---

## Troubleshooting

*   **Adapter Not Detected:** Ensure the USB adapter is securely plugged in. Run `lsusb` to see if the system recognizes it. Try a different USB port.
*   **`airmon-ng start` Fails:** Make sure you ran `sudo airmon-ng check kill` first. If it still fails, the drivers may not be loaded correctly (especially for v2/v3). A reboot often helps.
*   **Injection Test Fails:** This is almost always a driver issue. For v1, this is rare but could indicate a faulty adapter. For v2/v3, ensure you installed the `realtek-rtl8188eus-dkms` package and rebooted. Some USB ports may not provide enough power, so try a powered USB hub or a different port.
