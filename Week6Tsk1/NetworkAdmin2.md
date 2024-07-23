
# Deploying 3 VMs Using VMware Workstation

Using VMware Workstation, I took the steps below to deploy 3 VMs and achieve the network configuration:

## Create Three VMs

Create three VMs using VMware Workstation and a predefined VMDK disk (located at [OSBoxes](https://www.osboxes.org/ubuntu/#ubuntu-24-04-vmware)), follow these steps:

1. **Open VMware Workstation**

   Start VMware Workstation on your machine.

2. **Create a New Virtual Machine**

   For each of the three VMs (Main Office Server, Branch Office Server, and Client VM), follow these steps:

   - **Select File > New Virtual Machine...**
   
   - **Select Disk**

     Choose "Use an existing virtual disk" and browse to the location of your predefined VMDK file.

   - **Specify Disk File**

     Browse and select the predefined VMDK file.

   - **Name the Virtual Machine**

     Give a name to the virtual machine, e.g., "Main Office Server," "Branch Office Server," or "Client VM." Specify the location where you want to store the VM files.

   - **Finish**

     Click "Finish" to complete the VM creation process.

   Repeat these steps for each of the three VMs.

## Network Configuration

### Create Virtual Networks

1. **Create Virtual Networks in VMware Workstation:**

   - **MainOfficeNet:**
     1. Open VMware Workstation.
     2. Go to `Edit` > `Virtual Network Editor`.
     3. Click `Add Network` and select a network (e.g., `VMnet2`).
     4. Set the network to `Host-only`.
     5. Set the subnet IP to `192.168.1.0` and the subnet mask to `255.255.255.0`.
     6. Disable the DHCP for this network.
     7. Rename VMnet3 to MainOfficeNet.

   - **BranchOfficeNet:**
     1. Follow the same steps to create another network (e.g., `VMnet3`).
     2. Set the network to `Host-only`.
     3. Set the subnet IP to `192.168.2.0` and the subnet mask to `255.255.255.0`.
     4. Enable the DHCP for this network.
     5. Rename VMnet3 to BranchOfficeNet.

   - **VPN:**
     1. Create VMnet4 in VMware Workstation.
     2. Go to `Edit` > `Virtual Network Editor`.
     3. Click `Add Network` and select VMnet4.
     4. Set the network type to `Host-only`.
     5. Set the subnet IP to `10.0.0.0` and the subnet mask to `255.255.255.0`.
     6. Ensure that the DHCP is disabled for this network.
     7. Rename VMnet4 to VPN.

2. **Assign Networks to VMs:**

   - **Main Office Server:**
     1. Go to the settings of the Main Office Server VM.
     2. Under `Network Adapter`, add two network adapters.
     3. Attach the first adapter to `VMnet2 (MainOfficeNet)` and the second adapter to `VMnet4` for the VPN.

   - **Branch Office Server:**
     1. Go to the settings of the Branch Office Server VM.
     2. Under `Network Adapter`, add two network adapters.
     3. Attach the first adapter to `VMnet3 (BranchOfficeNet)` and the second adapter to `VMnet4` for the VPN.

   - **Client VM:**
     1. Go to the settings of the Client VM.
     2. Under `Network Adapter`, attach the adapter to `VMnet3 (BranchOfficeNet)`.

3. **Power On the VMs**

   Power on each VM by right-clicking each VM and selecting "Power On."

### Configure Network Interfaces

#### Updating or Replacing Existing Connections

1. **Identify and Modify Existing Connections**

   - **Delete Existing Connections:**

     ```bash
     sudo nmcli con delete netplan-ens33
     sudo nmcli con delete 'Wired connection 1'
     sudo nmcli con delete MainOfficeNet
     sudo nmcli con delete VPNNet
     ```

   - **Add New Connections:**

     ```bash
     # Add MainOfficeNet to ens33
     sudo nmcli con add type ethernet ifname ens33 con-name MainOfficeNet ip4 192.168.1.10/24
     sudo nmcli con modify MainOfficeNet connection.autoconnect yes

     # Add VPN Network to ens34
     sudo nmcli con add type ethernet ifname ens34 con-name VPNNet ip4 10.0.0.10/24
     sudo nmcli con modify VPNNet connection.autoconnect yes
     ```

2. **Apply and Bring Up Connections**

   - **Bring Up the Connections:**

     ```bash
     sudo nmcli con up MainOfficeNet
     sudo nmcli con up VPNNet
     ```

3. **Verify Configuration**

   Verify the configuration to ensure they are set to autoconnect across reboots:

   ```bash
   nmcli con show
   nmcli con show --active
   ip addr show
   ```

   You should see the connections with the `autoconnect` flag set to `yes`.

4. **Restart NetworkManager**

   Restart NetworkManager to apply the changes:

   ```bash
   sudo systemctl restart NetworkManager
   ```

### Configuration for Branch Office Server

1. **Configure Network Interfaces:**

   - **Delete Existing Connections:**

     ```bash
     sudo nmcli con delete netplan-ens33
     sudo nmcli con delete 'Wired connection 1'
     sudo nmcli con delete BranchOfficeNet
     sudo nmcli con delete VPNNet
     ```

   - **Add New Connections:**

     ```bash
     # Add BranchOfficeNet to ens33
     sudo nmcli con add type ethernet ifname ens33 con-name BranchOfficeNet ip4 192.168.5.10/24
     sudo nmcli con modify BranchOfficeNet connection.autoconnect yes

     # Add VPN Network to ens34
     sudo nmcli con add type ethernet ifname ens34 con-name VPNNet ip4 10.0.0.20/24
     sudo nmcli con modify VPNNet connection.autoconnect yes
     ```

2. **Bring Up the Connections:**

   ```bash
   sudo nmcli con up BranchOfficeNet
   sudo nmcli con up VPNNet
   ```

### Configuration for Client VM

1. **Delete Existing Connections:**

   ```bash
   sudo nmcli con delete 'Wired connection 1'
   sudo nmcli con delete BranchOfficeNet
   ```

2. **Add DHCP Configuration:**

   ```bash
   nmcli con add type ethernet ifname ens33 con-name BranchOfficeNet ipv4.method auto
   ```

3. **Bring Up the Connection:**

   ```bash
   sudo nmcli con up BranchOfficeNet
   ```

### Verify Configuration

- **Check IP Address:**

  ```bash
  nmcli device show ens33
  nmcli device show ens34
  ```


# Provide Internet Access from Branch Office Server to Client System

To provide internet access from your branch office server (which has internet access) to your client system (which does not), you can set up Network Address Translation (NAT) and configure IP forwarding on the branch office server. Here’s a step-by-step guide to achieve this on your Ubuntu server:

## 1. Enable IP Forwarding

You need to enable IP forwarding on the branch office server to allow it to forward packets between the client system and the internet.

Open the `/etc/sysctl.conf` file with a text editor:

```bash
sudo nano /etc/sysctl.conf
```

Find the line:

```bash
#net.ipv4.ip_forward=1
```

Uncomment it (remove the `#`), so it reads:

```bash
net.ipv4.ip_forward=1
```

Apply the changes:

```bash
sudo sysctl -p
```

## 2. Configure NAT (Network Address Translation)

Use `iptables` to configure NAT on the branch office server. This will allow the client system to use the branch office server’s internet connection.

Run the following commands:

```bash
sudo iptables -t nat -A POSTROUTING -o ens38 -j MASQUERADE
sudo iptables -A FORWARD -i ens33 -o ens38 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i ens38 -o ens33 -j ACCEPT
```

Replace `<internet_interface>` with the name of the network interface connected to the internet (e.g., `eth0`, `ens33`) and `<client_interface>` with the name of the network interface connected to the client system (e.g., `eth1`, `ens34`).

## 3. Save the `iptables` Configuration

To ensure that your `iptables` rules persist after a reboot, you need to save them. On Ubuntu, you can use the `iptables-save` command and store the rules in a file.

Save the rules:

```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

Install `iptables-persistent` to load the rules at boot:

```bash
sudo apt-get install iptables-persistent
```

Check that the NAT rule is in place:

```bash
sudo iptables -t nat -L -v
```

## 4. Configure the Client System

Ensure that the client system has its default gateway set to the branch office server’s IP address on the local network.

On the client system, you can set the default gateway by editing the `/etc/netplan/01-netcfg.yaml` file or using the `ip route` command. For example:

```bash
sudo ip route add default via 192.168.5.10
```

## 5. Test the Configuration

Test the internet connectivity from the client system by pinging an external website or using a web browser.

```bash
ping google.com
```

---

# Create a Systemd Service to Apply Network Routes on Boot

To create a systemd service that applies network routes on boot, follow these steps:

## 1. Create a Script to Apply the Routes

First, create a script that applies the routes. Let’s place this script in `/usr/local/bin` and make it executable.

### Create the Script

```bash
sudo nano /usr/local/bin/setup-routes.sh
```

### Add the Route Configuration

Add the following content to the script:

```bash
#!/bin/bash

# Remove any existing default route via the specified gateway
ip route del default via 192.168.5.10 2>/dev/null

# Add the default route
ip route add default via 192.168.5.10
```

Replace `192.168.5.10` with your actual gateway if needed.

### Make the Script Executable

```bash
sudo chmod +x /usr/local/bin/setup-routes.sh
```

## 2. Create a Systemd Service File

Next, create a systemd service file that will run this script on boot.

### Create the Service File

```bash
sudo nano /etc/systemd/system/setup-routes.service
```

### Add the Service Configuration

Add the following content to the service file:

```ini
[Unit]
Description=Apply Network Routes
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/setup-routes.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

This configuration sets up a one-time service that runs after the network is up and remains active after execution.

### Reload systemd and Enable the Service

Reload systemd to recognize the new service and enable it to run at boot:

```bash
sudo systemctl daemon-reload
sudo systemctl enable setup-routes.service
```

### Start the Service Manually (Optional)

If you want to start the service immediately without rebooting, you can use:

```bash
sudo systemctl start setup-routes.service
```

## 3. Verify the Service

Check the status of the service to ensure it is working correctly:

```bash
sudo systemctl status setup-routes.service
```

You should see that the service is active and running without errors. If there are issues, the logs can help diagnose the problem:

```bash
journalctl -u setup-routes.service
```

- **Script Location:** `/usr/local/bin/setup-routes.sh`
- **Service File Location:** `/etc/systemd/system/setup-routes.service`

---

# Setting Up Time Synchronization Using Chrony

To set up time synchronization using Chrony instead of NTP, follow these steps for both the server and client machines:

## Setting Up Chrony on the NTP Server

### Install Chrony

On the server machine:

```bash
sudo apt update
sudo apt install chrony
```

### Configure Chrony

Edit the Chrony configuration file `/etc/chrony/chrony.conf` to set up the server:

```bash
sudo nano /etc/chrony/chrony.conf
```

Update or add the following lines:

```conf
pool ntp.ubuntu.com iburst maxsources 4
allow 192.168.5.0/24

# Allow clients from the branch office network to access this server
allow 192.168.5.0/24
```

### Restart Chrony Service

```bash
sudo systemctl restart chronyd
```

### Verify Chrony Status

```bash
sudo systemctl status chronyd
```

Verify synchronization:

```bash
chronyc sources
```

## Setting Up Chrony on the Clients

### Install Chrony

On each client machine:

```bash
sudo apt update
sudo apt install chrony
```

### Configure Chrony

Edit the Chrony configuration file `/etc/chrony/chrony.conf` to point to the NTP server:

```bash
sudo nano /etc/chrony/chrony.conf
```

Update or add the following line to specify the NTP server:

```conf
server 192.168.5.10 iburst
```

### Restart Chrony Service

```bash
sudo systemctl restart chronyd
```

### Verify Chrony Status

```bash
sudo systemctl status chronyd
```

Verify synchronization:

```bash
chronyc sources
```

