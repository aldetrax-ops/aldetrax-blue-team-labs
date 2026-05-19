# Network Interface Troubleshooting Guide (Kali Linux)

### 1. Verify GUI Status
1. Click the **Kali Application Menu** icon.
2. Navigate to **Advanced Network Configuration**.
3. Check if the configuration window appears completely **blank**.

### 2. Check Interface Link State
1. Open the **Kali Terminal Shell**.
2. Execute the following command to list active IPv4 configurations:
   ```bash
   ip -4 addr

   Verify the output list:

1: lo: (Loopback adapter) should be present.

2: eth0: (Ethernet adapter) should be present, but without an assigned IP address and displaying state DOWN.

###3.Force the network interface to turn on by running
sudo ip link set eth0 up

 Wait a brief moment while the terminal pauses to process the command. The prompt will return to a blank line when finished.

### 4. Confirm IP Address Allocation
Verify the network interface status again:
   ```bash
   ip -4 addr
