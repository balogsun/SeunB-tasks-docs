# The Value of Network Administration Skills in Cloud Computing

In the cloud computing era, network administration skills are invaluable. Hybrid and multi-cloud environments require traditional networking knowledge to integrate on-premises infrastructure with cloud services. Cloud networking concepts like VLANs, subnets, routing, and firewalls are essential.

Network security skills apply directly to cloud security measures. Understanding containerization, microservices, and Infrastructure as Code (IaC) is crucial. Network troubleshooting, edge computing, compliance, cost optimization, and cloud-native networking all benefit from a strong foundation in traditional networking principles.

Lets picture a scenario where an organization is opening a new branch office and needs to set up a secure network infrastructure that integrates with the main office. 
As a DevOps engineer, you're tasked with designing, implementing, and automating this setup. 
I will be running through a step by step process of how to achieve this using a vmware worksation. 
The concept remains the same whether you use on-prem or cloud resources. 

# Deploying 3 VMs Using VMware Workstation

## Network Architecture Diagram:
<img width="399" alt="image" src="https://github.com/user-attachments/assets/e3d8a08f-10fd-4364-b3e1-4407b0015987">

### Diagram Details:

- **Main Office VPN Gateway**: Connects to the Branch Router via a VPN tunnel.
- **Branch Router**:  Provides internet access and routes traffic between the branch office and the main office via the VPN tunnel.
- **Branch Switch**:  Central hub connecting all devices within the branch office.
- **Peripheral Devices**: Connected directly to the Branch Switch.
- **Branch Server**: Manages DNS, VPN, DHCP, and NTP services.
- **Client Desktops**: Connect to the Branch Switch to access network resources and services.

## Using VMware Workstation, I took the steps below to deploy 3 VMs and achieve the network configuration:

### Create Three VMs

Create three VMs using VMware Workstation and a predefined VMDK disk (located at [OSBoxes](https://www.osboxes.org/ubuntu/#ubuntu-24-04-vmware)), follow these steps:

1. **Open VMware Workstation**

   Start VMware Workstation on your machine.

2. **Create a New Virtual Machine**

   For each of the three VMs (Main Office Server, Branch Office Server, and Client VM), follow these steps:

   - **Select File > New Virtual Machine...**

   - **Specify Disk File**

     Select `I will install the operating system later` and click `next`
     <img width="319" alt="image" src="https://github.com/user-attachments/assets/f8bad207-fa0f-45dd-9031-44b6120cfb8d">

     Select a guest operating system `Linux` and version `ubuntu`

     Choose the location where you have kept the custom .vmdk file
     Complete the setup and click `finish`
     <img width="346" alt="image" src="https://github.com/user-attachments/assets/6f3a2aa0-789b-4c4a-b2b4-7c364e395220">

     
   - **Name the Virtual Machine**

     Give a name to the virtual machine, e.g., "Main Office Server," "Branch Office Server," or "Client VM." Specify the location where you want to store the VM files.

   - **Finish**

     Click "Finish" to complete the VM creation process.

   Repeat these steps for each of the three VMs.

   Now select a virtual machine, click on `VM` then `settings`, this opens the VM properties, click `Add`, select `hard disk`, `scsi` `use an exisitng virtual disk`, browe for location of disk and click `finish`.
   
  <img width="549" alt="image" src="https://github.com/user-attachments/assets/eec72e38-88c8-4b79-b434-23fe8f18fe95">
  
Now you can start the VM with the imported disk [.vmdk file] you have selected. 

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
     3. Set the subnet IP to `192.168.5.0` and the subnet mask to `255.255.255.0`.
     4. Disable the DHCP for this network.
     5. Rename VMnet3 to BranchOfficeNet.

 	<img width="444" alt="image" src="https://github.com/user-attachments/assets/f0ccf75b-1b0d-49af-a095-2588c9e35b3a">


2. **Assign Networks to VMs:**

   - **Main Office Server:**
     1. Go to the settings of the Main Office Server VM.
     2. Under `Network Adapter`, add the network adapter.
     3. Attach the adapter to `VMnet2 (MainOfficeNet)`.

   - **Branch Office Server:**
     1. Go to the settings of the Branch Office Server VM.
     2. Under `Network Adapter`, add the network adapters.
     3. Attach the adapter to `VMnet3 (BranchOfficeNet)`.
     <img width="554" alt="image" src="https://github.com/user-attachments/assets/f89a6608-3895-4a9f-b12f-c7d8f84e7371">

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
     ```

   - **Add New Connections:**

     ```bash
     # Add MainOfficeNet to ens33
     sudo nmcli con add type ethernet ifname ens33 con-name MainOfficeNet ip4 192.168.1.10/24
     ```

2. **Apply and Bring Up Connections**

   - **Bring Up the Connection:**

     ```bash
     sudo nmcli con up MainOfficeNet
     ```

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
     ```

   - **Add New Connections:**
     ```bash
     # Add BranchOfficeNet to ens33
     sudo nmcli con add type ethernet ifname ens33 con-name BranchOfficeNet ip4 192.168.5.10/24
     ```

2. **Bring Up the Connections:**

   ```bash
   sudo nmcli con up BranchOfficeNet
   ```

   Verify the configuration:

   ```bash
   nmcli con show
   nmcli con show --active
   ip addr show
   ifconfig -a
   ```
   <img width="453" alt="image" src="https://github.com/user-attachments/assets/a63e56b4-0eee-40f7-a219-3b9f61cfb721">

   <img width="481" alt="image" src="https://github.com/user-attachments/assets/5f56f068-6dee-4151-9365-a1aa30fe2d1c">
   
### Configuration for Client VM

1. **Delete Existing Connections:**

   ```bash
   sudo nmcli con delete 'Wired connection 1'
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

  - From the Client VM, ensure it gets an IP address via DHCP and check connectivity:
    ```bash
    nmcli device show ens33
	ip route
    	ip addr
    ```
    <img width="483" alt="image" src="https://github.com/user-attachments/assets/3a64eff6-1ebd-4d0b-93c7-f4386fd5929a">
    <img width="490" alt="image" src="https://github.com/user-attachments/assets/8542e82f-ebff-408d-9342-b4ea07b8597a">
    

## Setting Up DNS:
#### Setting up DNS with BIND9 involves configuring both the main and branch servers to handle local domain resolution and forward external queries appropriately. Below are the detailed steps for setting this up.

### Main Server (main.abc.local)

#### 1. Install BIND9

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```

#### 2. Configure BIND9

Edit the BIND9 configuration files to set up the master server:

- **Edit named.conf.local**

  ```bash
  sudo nano /etc/bind/named.conf.local
  ```

  Add the following:

  ```plaintext
  zone "abc.local" {
      type master;
      file "/etc/bind/db.abc.local";
  };

  zone "5.168.192.in-addr.arpa" {
      type master;
      file "/etc/bind/db.192.168.5";
  };
  ```
<img width="395" alt="image" src="https://github.com/user-attachments/assets/4ce59ef1-60ef-478b-a839-c4c0b1a31164">

- **Create Zone Files**

  Create the forward zone file for `abc.local`:

  ```bash
  sudo nano /etc/bind/db.abc.local
  ```

  Add the following content:

  ```plaintext
  $TTL    604800
  @       IN      SOA     main.abc.local. admin.abc.local. (
                               2         ; Serial
                          604800         ; Refresh
                           86400         ; Retry
                         2419200         ; Expire
                          604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      main.abc.local.
  main    IN      A       192.168.1.10
  branch  IN      A       192.168.5.10
  client  IN      A       192.168.5.15
  ```

  Create the reverse zone file for `192.168.5.x`:

  ```bash
  sudo nano /etc/bind/db.192.168.5
  ```

  Add the following content:

  ```plaintext
  $TTL    604800
  @       IN      SOA     main.abc.local. admin.abc.local. (
                               2         ; Serial
                          604800         ; Refresh
                           86400         ; Retry
                         2419200         ; Expire
                          604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      main.abc.local.
  10      IN      PTR     branch.abc.local.
  15      IN      PTR     client.abc.local.
  ```

#### 3. Configure Forwarders

Edit named.conf.options to include forwarders:

```bash
sudo nano /etc/bind/named.conf.options
```

Add the following within the `options` block:

```plaintext
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;  // Google's DNS
        8.8.4.4;  // Google's DNS
    };

    dnssec-validation auto;

    listen-on-v6 { any; };
};
```
<img width="469" alt="image" src="https://github.com/user-attachments/assets/f1023bd8-4b4a-49d9-bf28-5dfbd1a387b5">

#### 4. Restart BIND9

```bash
sudo systemctl restart bind9
systemctl status bind9
systemctl restart named.service
```

### Branch Server (branch.abc.local)

#### 1. Install BIND9

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```

### Configure iptables to permit incoming dns queries from client systems. 
#### Allow incoming DNS traffic on port 53 (UDP)
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

#### Allow incoming DNS traffic on port 53 (TCP)
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Save the iptables rules
sudo sh -c "iptables-save > /etc/iptables/rules.v4"

#### 2. Configure BIND9 as Slave

Edit the BIND9 configuration files to set up the branch server:

- **Edit named.conf.local**

  ```bash
  sudo nano /etc/bind/named.conf.local
  ```

  Add the following:

  ```plaintext
  zone "abc.local" {
      type slave;
      file "/var/cache/bind/db.abc.local";
      masters { 192.168.1.10; };
  };

  zone "5.168.192.in-addr.arpa" {
      type slave;
      file "/var/cache/bind/db.192.168.5";
      masters { 192.168.1.10; };
  };
  ```

- **Configure Forwarding**

  Edit named.conf.options to forward unknown/external queries to the main server:

  ```bash
  sudo nano /etc/bind/named.conf.options
  ```

  Add the following within the `options` block:

  ```plaintext
  options {
      directory "/var/cache/bind";

      forwarders {
          192.168.1.10;  // Main server's IP
      };

      dnssec-validation auto;

      listen-on-v6 { any; };
  };
  ```
    <img width="341" alt="image" src="https://github.com/user-attachments/assets/bab15fa9-90f9-41c8-89f5-83052274b843">

#### 3. Restart and enable BIND9 service at every system reboot

```bash
sudo systemctl restart bind9
sudo systemctl enable bind9
```

#### Check Zone Transfer Logs on Branch Server

```bash
sudo systemctl status bind9
```
![Screenshot 2024-07-31 204515](https://github.com/user-attachments/assets/bb070436-a68c-47ea-b558-baa93d6ac287)

#### Test DNS Resolution

On the branch server, use `dig` to test:

```bash
dig main.abc.local @localhost
dig branch.abc.local @localhost
dig client.abc.local @localhost
dig google.com @localhost
```
![Screenshot 2024-07-31 204435](https://github.com/user-attachments/assets/6fe71dbf-04d3-4010-92a6-b07965f70fcd)

![Screenshot 2024-07-31 204554](https://github.com/user-attachments/assets/981aa138-1226-4a34-8333-0bd8e5557281)

![Screenshot 2024-07-31 204843](https://github.com/user-attachments/assets/b4220d0d-9155-4c75-aeec-63d9132b360c)

![Screenshot 2024-07-31 205042](https://github.com/user-attachments/assets/5236fd5c-c9bf-4e11-b62b-1f90c6cdeb44)

The dig output indicates that the DNS server running on localhost successfully resolved the domain main.abc.local to the IP address 192.168.1.10, but it issued a warning because .local is reserved for mDNS. The query was handled correctly with an authoritative answer and the response took 4 milliseconds.

The dig output indicates that the DNS server running on localhost successfully resolved google.com to the IP address 142.251.32.78. The query was handled correctly with an authoritative answer, and the response took 1 millisecond. The result shows that your local DNS server is capable of resolving external domain names as well, as it is properly configured to forward queries or perform DNS resolution.

By following these steps, your branch server will be configured to resolve local domain queries from its own zone files and forward any unknown or external domain queries to the main server. The main server is configured to forward external queries to public DNS servers (like Google's DNS).

### Client Configuration (192.168.5.15)

Ensure that the client machine is configured to use the branch office DNS server for its DNS queries.

Setup # operation for /etc/resolv.conf.

nameserver 192.168.5.10
nameserver 127.0.0.53
options edns0 trust-ad
search branch.company.local

1. **Configure `/etc/netplan/01-netcfg.yaml`**

   Edit the netplan configuration to use the branch server for DNS:

   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

   Ensure it contains:

   ```plaintext

network:
  version: 2
  ethernets:
    ens33:
      dhcp4: yes
      routes:
        - to: default
          via: 192.168.5.10
      nameservers:
        addresses:
          - 192.168.5.10
          - 8.8.8.8


   ```

2. **Apply the Netplan Configuration**

   ```bash
   sudo netplan apply
   ```

### Verify the Setup

1. **Check DNS Resolution**

   On the client, verify that DNS resolution works:

   ```bash
   ping main.abc.local
   ping branch.abc.local 
   nslookup main.abc.local
   nslookup branch.abc.local 
   nslookup client.abc.local
   ```
   ![Screenshot 2024-07-31 210605](https://github.com/user-attachments/assets/e5710539-c2e5-4eba-9bcb-3fecb917639d)

   - **External DNS Resolution:**

     ```bash
     nslookup google.com
     ```
    ![Screenshot 2024-07-31 200307](https://github.com/user-attachments/assets/fd0fb5d1-8bf5-40fc-9f7f-e0ec510d7cb5)


2. **Test External DNS Resolution**

   On the branch server, test that external domain resolution works:

   ```bash
   dig google.com
   ```

3. **Check BIND9 Status**

   Ensure that BIND9 is running correctly on both the main and branch servers:

   ```bash
   sudo systemctl status bind9
   ```

By following these steps, you should have a working DNS setup where the branch office can resolve both local and external domains through the main office DNS server.


### Steps to Deploy Tinc VPN

**Tinc VPN** is a flexible and powerful VPN daemon that supports full-mesh routing and dynamic links between nodes. Here's how to deploy Tinc VPN on both the main server and branch server.

### Prerequisites
- Two Linux servers (main and branch) with root or sudo access.
- Ensure that both servers have open network ports for Tinc (default is 655 for TCP and UDP).

### Step 1: Install Tinc VPN

**On Both Servers:**

1. **Update the package list and install Tinc:**
   ```bash
   sudo apt update
   sudo apt install tinc
   ```

### Step 2: Create Tinc Configuration Directories

**On Both Servers:**

1. **Create the main configuration directory for Tinc:**
   ```bash
   sudo mkdir -p /etc/tinc/vpn/hosts
   ```

2. **Navigate to the Tinc directory:**
   ```bash
   cd /etc/tinc/vpn
   ```

### Step 3: Generate Tinc Configuration Files

**On Both Servers:**

1. **Create the `tinc.conf` file:**
   ```bash
   sudo nano tinc.conf
   ```

2. **Add the following configuration (replace `MainServer` and `BranchServer` with appropriate hostnames):**

   **Main Server (`main`):**
   ```ini
	Name = main
	AddressFamily = ipv4
	Interface = tun0
   ```
  <img width="324" alt="image" src="https://github.com/user-attachments/assets/ef0ca51f-28a5-4ff3-9a8c-a56c9933e239">

   **Branch Server (`branch`):**
   ```ini
	Name = branch
	AddressFamily = ipv4
	Interface = tun0
	ConnectTo = main
   ```

3. **Create the `tinc-up` script:**
   ```bash
   sudo nano tinc-up
   ```

4. **Add the following content (adjust IP addresses as needed):**

   **Main Server:**
   ```bash
   #!/bin/sh
   ifconfig $INTERFACE 10.0.0.1 netmask 255.255.255.0
   ```
   <img width="345" alt="image" src="https://github.com/user-attachments/assets/07d50d4a-1925-4a74-9201-d2128d63ad83">

   **Branch Server:**
   ```bash
   #!/bin/sh
   ifconfig $INTERFACE 10.0.0.2 netmask 255.255.255.0
   ```

5. **Make the `tinc-up` script executable:**
   ```bash
   sudo chmod +x tinc-up
   ```

6. **Create the `tinc-down` script:**
   ```bash
   sudo nano tinc-down
   ```

7. **Add the following content:**
   ```bash
   #!/bin/sh
   ifconfig $INTERFACE down
   ```

8. **Make the `tinc-down` script executable:**
   ```bash
   sudo chmod +x tinc-down
   ```

### Step 4: Configure Host Files

**On Both Servers:**

1. **Create a host configuration file for each server:**

   **Main Server:**
   ```bash
   sudo nano hosts/main
   ```

   **Branch Server:**
   ```bash
   sudo nano hosts/branch
   ```

2. **Add the following content:**

   **Main Server:**
   ```ini
   Address = 192.168.1.10
   Subnet = 10.0.0.1/32
   ```

   **Branch Server:**
   ```ini
   Address = 192.168.5.10
   Subnet = 10.0.0.2/32
   ```
   <img width="352" alt="image" src="https://github.com/user-attachments/assets/8482ccf0-b42c-4095-9850-1aba239b49cb">

### Step 5: Generate Tinc Keys

**On Both Servers:**

1. **Generate the Tinc RSA key pair:**
   ```bash
   sudo tincd -n vpn -K4096
   ```

2. **This will generate `rsa_key.priv` and `hosts/<hostname>` files. Share the contents of these files between the servers:**

   **Main Server:**
   ```bash
   sudo cat /etc/tinc/vpn/hosts/main
   ```

   **Branch Server:**
   ```bash
   sudo cat /etc/tinc/vpn/hosts/branch
   ```

3. **Copy the public key portion (the lines starting with `-----BEGIN RSA PUBLIC KEY-----` to `-----END RSA PUBLIC KEY-----`) from each server and add it to the corresponding host file on the other server:**

   **On Main Server (`/etc/tinc/vpn/hosts/branch`):**
   ```ini
   -----BEGIN RSA PUBLIC KEY-----
   (Branch Server public key here)
   -----END RSA PUBLIC KEY-----
   ```
   
   **On Branch Server (`/etc/tinc/vpn/hosts/main`):**
   ```ini
   -----BEGIN RSA PUBLIC KEY-----
   (Main Server public key here)
   -----END RSA PUBLIC KEY-----
   ```
   <img width="419" alt="image" src="https://github.com/user-attachments/assets/051c3552-c09f-4568-a85b-ca40cb9c03c8">

   Ensure the hosts files, main and branch exists on both servers. [copy them all to each other]
   <img width="322" alt="image" src="https://github.com/user-attachments/assets/67d259aa-81c7-472f-9331-b6191fa25203">

### Step 6: Start Tinc VPN

**On Both Servers:**

1. **Enable and start the Tinc service:**
   ```bash
   sudo systemctl enable tinc@vpn
   sudo systemctl start tinc@vpn
   sudo systemctl status tinc@vpn
   ```

2. **Check the status of the Tinc service:**
   ```bash
   sudo systemctl status tinc@vpn
   ```
   <img width="536" alt="image" src="https://github.com/user-attachments/assets/e75df1be-41c1-4c8e-b9dc-dd56ca2bc99b">

### Step 7: Verify the Connection

1. **Verify that the `tun0` interface is up and configured correctly on both servers:**
   ```bash
   ifconfig tun0
   ```
   <img width="537" alt="image" src="https://github.com/user-attachments/assets/5cdb4af7-ceae-4487-8c52-f149416125e7">


2. **Check the connectivity between the servers:**
   ```bash
   ping 10.0.0.2  # From Main Server
   ping 10.0.0.1  # From Branch Server
   ssh osboxes@10.0.0.2 # From Main Server
    ifconfig tun0
   ```
   <img width="426" alt="image" src="https://github.com/user-attachments/assets/c1b85459-493e-4329-a1b4-5d2bd4a0e6b4">
   <img width="458" alt="image" src="https://github.com/user-attachments/assets/737c995e-818b-40cb-82bb-a9f8c0eb8763">

### Setup DHCP on Branch Office:

1. **Install DHCP (isc-dhcp-server):**

   ```bash
   sudo apt update
   sudo apt install isc-dhcp-server
   ```

2. **Configure DHCP Server:**

   Edit the DHCP server configuration file:

   ```bash
   sudo nano /etc/dhcp/dhcpd.conf
   ```

   Add or modify the following lines to configure the DHCP server:

   ```plaintext
   # Optionally specify a domain name
   option domain-name "company.local";

   # Specify the default lease time (in seconds)
   default-lease-time 600;

   # Specify the maximum lease time (in seconds)
   max-lease-time 7200;

   # Specify the network and subnet
   subnet 192.168.5.0 netmask 255.255.255.0 {
       range 192.168.5.50 192.168.5.100;  # IP range to be assigned to clients
       option routers 192.168.5.10;       # Gateway
       option domain-name-servers 192.168.5.10, 8.8.8.8;  # DNS servers
   }
   ```

3. **Specify Network Interface for DHCP:**

   Edit the file `/etc/default/isc-dhcp-server` to specify the network interface that the DHCP server should listen on. For example:

   ```plaintext
   INTERFACESv4="ens33"
   ```

   Replace `ens33` with the name of the network interface connected to your local network.

4. **Restart DHCP Server:**

   Activate and start the DHCP server to apply the changes:

   ```bash
   sudo systemctl start isc-dhcp-server
   systemctl status isc-dhcp-server
   systemctl enable isc-dhcp-server
   ```

### On the Client Server

#### 1. **Configure Netplan for DHCP**

Edit the Netplan configuration file to use DHCP for obtaining an IP address:

1. **Edit Netplan Configuration:**

   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

2. **Modify the Configuration to Use DHCP:**

   Update the file to use DHCP for the `ens33` interface:

   ```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: yes
      routes:
        - to: default
          via: 192.168.5.10
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
   ```

3. **Apply the Netplan configuration:**

   ```bash
   sudo netplan apply
   ```
   
   A new network will be created and will then need to be attached to ens33, then reboot so that it can pickup an IP. Or you can use below command to configure the network to be auto assigned an IP.

```
sudo nmcli con add type ethernet ifname ens33 con-name netplan-ens33 ipv4.method auto
```

### Verify the Setup

1. **Check DHCP Lease on Client:**

nmcli con show
sudo nmcli con up netplan-ens33
nmcli device show ens33
nmcli device status
nmcli con show netplan-ens33


   Once the client server is powered up, it should automatically obtain an IP address from the branch server. Verify the IP address on the client:

   ```bash
   ip addr show ens33
   ```

   You should see an IP address within the range specified in the DHCP server configuration (e.g., `192.168.5.15` to `192.168.5.100`).

By following these steps, your branch server will act as a DHCP server, and the client server will automatically receive its IP address from the branch server when powered up.


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
sudo iptables -A FORWARD -i ens38 -o ens33 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens38 -j ACCEPT
```

Replace `<internet_interface>` with the name of the network interface connected to the internet (e.g., `ens33`) and `<client_interface>` with the name of the network interface connected to the client system (e.g., `ens34`).

## 3. Save the `iptables` Configuration

To ensure that your `iptables` rules persist after a reboot, you need to save them. On Ubuntu, you can use the `iptables-save` command and store the rules in a file.

Save the rules:

Install `iptables-persistent` to load the rules at boot [if not already installed]:

```bash
sudo apt-get install iptables-persistent
```

```bash
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
OR
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

Check that the NAT rule is in place:

```bash
sudo iptables -t nat -L -v
```

## 4. Configure the Client System

Ensure that the client system has its default gateway set to the branch office server’s IP address on the local network.

On the client system, you can set the default gateway by editing the `/etc/netplan/01-netcfg.yaml` file or using the `ip route` command. For example:


sudo nano /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: yes
      routes:
        - to: default
          via: 192.168.5.10
      nameservers:
        addresses:
#         - 8.8.8.8
	  - 192.168.5.10
          - 8.8.4.4

```
sudo netplan apply
```

## 5. Test the Configuration

Test the internet connectivity from the client system by pinging an external website or using a web browser.

```bash
ping google.com
```

### NTP setup

I will be using `chrony` set up the NTP service, where the client system receives time sychronization from the branch server, following these steps:

### On the Branch Server

1. **Install Chrony:**

```bash
sudo apt-get update
sudo apt-get install chrony -y
```

2. **Configure Chrony:**

Edit the Chrony configuration file:

```bash
sudo nano /etc/chrony/chrony.conf
```

Add the following lines to allow the client server to get time from the branch server:

```plaintext
allow 192.168.5.0/24
```

This `allow` directive permits the specified network to access the time service. Adjust the network range if necessary.

Configure IPtables to permit NTP service syncronization on port 123 from client systems:

```
sudo iptables -A INPUT -p udp --dport 123 -j ACCEPT
sudo iptables -A OUTPUT -p udp --sport 123 -j ACCEPT

sudo iptables-save | sudo tee /etc/iptables/rules.v4
sudo iptables -L -v -n
```

3. **Start Chrony:**

```bash
sudo systemctl start chrony
sudo systemctl enable chrony
```

4. **Verify Chrony Status:**

```bash
sudo systemctl status chrony
```

### On the Client Server

1. **Install Chrony:**

```bash
sudo apt-get update
sudo apt-get install chrony -y
```

2. **Configure Chrony:**

Edit the Chrony configuration file:

```bash
sudo nano /etc/chrony/chrony.conf
```

Add the branch server's IP address as the NTP server:

```plaintext
server 192.168.5.10 iburst #Branch server IP address
```

3. **Start Chrony:**

```bash
sudo systemctl start chrony
sudo systemctl enable chrony
```

4. **Verify Chrony Status:**

```bash
sudo systemctl status chrony
```

### Verification

On the brannch server, run 'netplan apply'

To ensure that the client server is correctly synchronizing its time from the branch server, you can use the `chronyc` command.

**On the Client Server:**

1. **Check Chrony Sources:**

```bash
chronyc sources
```

You should see the branch server (`192.168.5.10` or its `hostname`) listed as a source.

2. **Other chrony commands are listed below:**

```bash
chronyc tracking
sudo chronyc -a makestep
chronyc activity
chronyc serverstats
chronyc sources -v
```
![Screenshot 2024-07-31 210845](https://github.com/user-attachments/assets/a5424a75-ad98-450a-9dfd-d096720c97e7)


## To set up Snort to monitor for suspicious activity on your servers, follow these steps:

### 1. Install Snort

#### On Debian/Ubuntu:

```bash
sudo apt update
sudo apt install snort
```
![Screenshot 2024-07-31 115810](https://github.com/user-attachments/assets/e500df3c-cfb7-4458-8c5a-437a4cd9c108)

### 2. Configure Snort

#### Initial Configuration:

Snort’s main configuration file is located at `/etc/snort/snort.conf`. Open this file to configure it according to your network setup.

```bash
sudo nano /etc/snort/snort.conf
```

#### Configure Network Variables:

Set your HOME_NET and EXTERNAL_NET variables. set up Snort to monitor three interfaces with different subnet, set it as follows:

```plaintext
var HOME_NET [192.168.1.0/24,10.0.0.0/24,192.168.79.0/24]
var EXTERNAL_NET !$HOME_NET
```

for branch server:
var HOME_NET [192.168.5.0/24,10.0.0.0/24,192.168.79.0/24]
var EXTERNAL_NET !$HOME_NET

for client:
var HOME_NET 192.168.5.0/24
var EXTERNAL_NET !$HOME_NET

#### Include Rule Files:

Ensure the rule paths are correctly specified:

```plaintext
echo 'include $RULE_PATH/local.rules' >> /etc/snort/snort.conf
```

### 4. Create Local Rules

You can create custom rules specific to your network in the `local.rules` file:

```bash
sudo nano /etc/snort/rules/local.rules
```

Add some basic rules:

```plaintext
# Alert on any ICMP traffic
alert icmp any any -> any any (msg:"ICMP Traffic Detected"; sid:1000001; rev:1;)

# Alert on any TCP traffic to port 123 (NTP)
alert tcp any any -> any 123 (msg:"TCP Traffic to NTP port 123 detected"; sid:1000002; rev:1;)

# Alert on any TCP traffic to port 53 (DNS)
alert tcp any any -> any 53 (msg:"TCP Traffic to DNS port 53 detected"; sid:1000003; rev:1;)

# Alert on any UDP traffic to port 53 (DNS)
alert udp any any -> any 53 (msg:"UDP Traffic to DNS port 53 detected"; sid:1000004; rev:1;)

# Alert on any TCP traffic to port 22 (SSH)
alert tcp any any -> any 22 (msg:"TCP Traffic to SSH port 22 detected"; sid:1000005; rev:1;)
```

### 5. Test Snort Configuration

Test the configuration to ensure there are no syntax errors:

```bash
sudo snort -T -c /etc/snort/snort.conf
```

### 6. Run Snort

Run Snort in IDS mode:

To monitor multiple network interfaces with Snort, 
You can run Snort multiple times, each time specifying a different interface:

```sh
snort -A console -c /etc/snort/snort.conf -i <network-interface>
snort -A console -c /etc/snort/snort.conf -i ens33 &
snort -A console -c /etc/snort/snort.conf -i eth1 &
```

Replace `<network-interface>` with your network interface, for example, `ens33`.

- Sample output of result
  ![Screenshot 2024-07-31 133333](https://github.com/user-attachments/assets/910e76a8-54aa-41bd-a924-316943fc3556)


### 7. Automate Snort Startup

To ensure Snort starts on boot, create a systemd service file.

#### Create Systemd Service File:

```bash
sudo nano /etc/systemd/system/snort33.service
```

Add the following content:

```ini
[Unit]
Description=Snort NIDS
After=network.target

[Service]
ExecStart=/usr/sbin/snort -c /etc/snort/snort.conf -i ens33
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### Create another Systemd Service File for vpn tunnel [connecting branch and main servers]:
```bash
sudo nano /etc/systemd/system/tun.service
```

Add the following content:

```ini
[Unit]
Description=Snort tunnel NIDS
After=network.target

[Service]
ExecStart=/usr/sbin/snort -c /etc/snort/snort.conf -i tun0
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### Start the services
```bash
systemctl daemon-reload
systemctl enable tun.service
systemctl start tun.service

systemctl enable snort33.service
systemctl start snort33.service
systemctl status snort33.service
```
![Screenshot 2024-07-31 140533](https://github.com/user-attachments/assets/9cdbabb8-2c1d-4799-a003-8e95832c8632)

### 8. Monitor Snort Logs

Snort logs its alerts to `/var/log/snort/snort.alert.fast` by default. You can monitor this file for suspicious activity.

```bash
tail -f /var/log/snort/snort.alert.fast
```
![Screenshot 2024-07-31 140757](https://github.com/user-attachments/assets/34a388b7-fc6b-4026-b9e9-a2d21c456464)

By following these steps, Snort will monitor your network traffic for suspicious activity and log alerts based on the rules defined. You can optionally adjust and expand the rules and configuration to match the specific requirements and threats relevant to your environment.

## COnfigure NMAP:
To use `nmap` for network scanning and to ensure that all expected services are running and accessible, follow these steps:

### 1. **Install Nmap on branch server**

First, make sure `nmap` is installed on your system. You can install it using the package manager for your distribution:

For Debian/Ubuntu-based systems:
```bash
sudo apt-get update
sudo apt-get install nmap
```

### 2. **Basic Network Scan**

To scan an entire network for live hosts and open ports, use:

```bash
nmap -sP 192.168.5.0/24
```
or
```bash
nmap -sn 192.168.5.0/24
```

This will perform a "ping scan" to determine which hosts are online.
![Screenshot 2024-07-31 155530](https://github.com/user-attachments/assets/bb5c9cb4-3735-46c8-9f30-7ea6f76e25d6)


### 3. **Service Detection**

To detect services running on specific hosts or an entire network, use the following command:

```bash
nmap -sV 192.168.5.0/24
```

This performs a service/version detection scan to determine what services and versions are running on open ports.

### 4. **Port Scanning**

To scan for open ports on a specific host:

```bash
nmap -p- 192.168.5.10
```
This scans all 65535 ports. You can specify a range of ports:
![Screenshot 2024-07-31 161358](https://github.com/user-attachments/assets/5a27ea6e-c51a-49bb-920b-a2090e0ff340)

This scans for port 22 and port 80 on the server
```bash
nmap -p 22,80,443 192.168.1.10
```
![Screenshot 2024-07-31 161907](https://github.com/user-attachments/assets/e080a6c0-853a-4130-9054-7c01f794d978)

### 5. **Aggressive Scan**

An aggressive scan performs a detailed scan including service detection, OS detection, and more:

```bash
nmap -A 192.168.5.15
```

This will give you extensive information about the host, including open ports, services, OS, and more.

### 6. **Checking Specific Services**

To check if specific services (like HTTP on port 80 and SSH on port 22) are running:

```bash
nmap -p 80,22 192.168.5.10
```

### 7. **Scan Multiple Hosts**

To scan multiple hosts or subnets, list them in the command:

```bash
sudo nmap -p 80,22 192.168.5.10 192.168.5.15 192.168.1.10
```
![Screenshot 2024-07-31 162143](https://github.com/user-attachments/assets/d153cbfd-6e2d-4d17-8195-0a63399fc20b)

### 8. **Output Formats**

For easier analysis, you can save the results in various formats:

- **Normal Output**:
  ```bash
  sudo nmap -oN scan_results.txt 192.168.5.15 192.168.5.10
  ```

### 9. **Schedule Regular Scans**

To ensure services are consistently monitored, you can set up a cron job to run `nmap` regularly.

Edit the crontab with:

```bash
crontab -e
```

Add an entry to run `nmap` at a regular interval, e.g., daily at midnight:

```bash
0 0 * * * /usr/bin/nmap -sP 192.168.5.0/24 > /var/log/nmap_daily_scan.log
```

These steps will help ensure all expected services are running and accessible on your network.

### Capture network packets with `tcpdump`
Capturing and analyzing network traffic using `tcpdump` and `Wireshark` can help you verify that your VPN is working correctly and identify any potential issues. Below are the steps to perform this task on both the main and branch servers.

#### Install `tcpdump` if not already pre-installed.

For Debian/Ubuntu-based systems:
```bash
sudo apt-get update
sudo apt-get install tcpdump
```

#### Capture Traffic

You need to capture traffic on the interfaces involved in the VPN connection. For example, my VPN setup interface is `tun0`, I can use the following command:

```bash
sudo tcpdump -i tun0 -w vpn_traffic.pcap
```
![Screenshot 2024-07-31 172143](https://github.com/user-attachments/assets/ad109b60-2f4a-4fe6-985c-9c953124f21f)

- `-i tun0`: Specifies the interface to capture traffic on (replace `tun0` with your own actual VPN interface).
- `-w vpn_traffic.pcap`: Writes the captured packets to a file named `vpn_traffic.pcap`.

You can also filter the traffic by IP address or port if you want to narrow down the capture:

```bash
sudo tcpdump -i tun0 host 10.0.0.1 -w vpn_traffic.pcap
```

#### Capture on Both Servers

Run the above `tcpdump` commands on both the main and branch servers to capture the relevant traffic.

I will also run a ping and ssh commands accross both servers in another session so that ICMP and SSH packets can be captured in the tcpdump process.
   ping 10.0.0.2 # From the main server
   ping 10.0.0.1 # From the branch server
   ssh user@10.0.0.2 # From the main server
   ssh user@10.0.0.1 # From the branch server

### Step 2: Transfer the Capture Files

After capturing the traffic, transfer the `.pcap` files to your local machine for analysis. You can use `scp` (secure copy) to do this:

```bash
scp user@branch_server:/path/to/vpn_traffic.pcap /local/path/
scp user@main_server:/path/to/vpn_traffic.pcap /local/path/
```

### Step 3: Analyze Traffic with Wireshark

#### Install Wireshark

Wireshark is available for Windows, macOS, and Linux. You can download it from the official [Wireshark website](https://www.wireshark.org/).

**Open captured files in Wireshark and analyze:**

1. Open Wireshark.
2. Click `File -> Open` and select the transferred .pcap files to open.
3. Use Wireshark's display filters to focus on specific traffic. For example, to display only traffic between two IP addresses, enter below strings in the search bar display fileter, and click on the blue arrow to the right to apply.

   ```wireshark
   ip.addr == 10.0.0.1 && ip.addr == 10.0.0.2
   ```
   ![Screenshot 2024-07-31 181912](https://github.com/user-attachments/assets/7b803bbc-94ab-4003-a7e8-526cf9580fd8)

2. **Inspect VPN Traffic**: Look at the traffic on the `tun0` interface (or your VPN interface) to ensure packets are being transmitted and received correctly.
3. **Check for Anomalies**: Look for retransmissions, malformed packets, or any other unusual activity.

By following these steps, you can capture and analyze the network traffic on both your main and branch servers, verify the VPN functionality, and troubleshoot any potential issues using `tcpdump` and Wireshark.

## AUTOMATION
###  Configure Ansible Playbooks
I will also install ansible and Based on the manual steps I have used to initially setup the configuration, I will create Ansible playbooks to automate the setup for DHCP, DNS, NTP, and basic firewall rules. 

Here is a code file containing the steps to install and configure Ansible on an Ubuntu system for this environment.

```bash
#!/bin/bash

# Update package index
echo "Updating package index..."
sudo apt update

# Install Ansible
echo "Installing Ansible..."
sudo apt install ansible -y

# Optionally, install the latest version of Ansible via PPA
echo "Adding Ansible PPA..."
sudo add-apt-repository ppa:ansible/ansible -y
sudo apt update
sudo apt install ansible -y

# Verify Ansible installation
echo "Verifying Ansible installation..."
ansible --version

# Create and configure the inventory file
echo "Configuring Ansible inventory file..."
sudo tee /etc/ansible/hosts > /dev/null <<EOL
[main_server]
192.168.1.10

[branch_server]
192.168.5.10

[client_server]
192.168.5.15
EOL

# Edit the Ansible configuration file
echo "Editing Ansible configuration file..."
sudo tee /etc/ansible/ansible.cfg > /dev/null <<EOL
[defaults]
inventory = /etc/ansible/hosts
remote_user = your_user
private_key_file = /path/to/private/key
EOL

# Test Ansible configuration
echo "Testing Ansible configuration..."
ansible all -m ping

echo "Ansible installation and configuration complete!"
```

### Instructions for Using the Script

1. **Save the Script:**

   Save the above script to a file, e.g., `install_ansible.sh`.

2. **Make the Script Executable:**

   ```bash
   chmod +x install_ansible.sh
   ```

3. **Run the Script:**

   ```bash
   ./install_ansible.sh
   ```

This script handles the installation of Ansible, adds the Ansible PPA if desired, sets up the inventory file, and configures the Ansible configuration file. Adjust the `remote_user` and `private_key_file` in the Ansible configuration file according to your setup.

Below are the Ansible playbooks for each service:

### 1. DHCP Configuration

**Playbook: `dhcp.yml`**

```yaml
---
- name: Configure DHCP Server
  hosts: branch_server
  become: yes
  tasks:
    - name: Install DHCP server
      apt:
        name: isc-dhcp-server
        state: present
        update_cache: yes

    - name: Configure DHCP server
      copy:
        dest: /etc/dhcp/dhcpd.conf
        content: |
          option domain-name "company.local";
          default-lease-time 600;
          max-lease-time 7200;

          subnet 192.168.5.0 netmask 255.255.255.0 {
              range 192.168.5.50 192.168.5.100;
              option routers 192.168.5.10;
              option domain-name-servers 192.168.5.10, 8.8.8.8;
          }

    - name: Specify network interface for DHCP
      lineinfile:
        path: /etc/default/isc-dhcp-server
        regexp: '^INTERFACESv4='
        line: 'INTERFACESv4="ens33"'

    - name: Restart DHCP server
      service:
        name: isc-dhcp-server
        state: restarted
        enabled: yes
```

### 2. DNS Configuration

**Playbook: `dns.yml`**

```yaml
---
- name: Configure DNS Server
  hosts: main_server
  become: yes
  tasks:
    - name: Install BIND9
      apt:
        name: "{{ item }}"
        state: present
        update_cache: yes
      with_items:
        - bind9
        - bind9utils
        - bind9-doc

    - name: Configure BIND9 named.conf.local
      copy:
        dest: /etc/bind/named.conf.local
        content: |
          zone "abc.local" {
              type master;
              file "/etc/bind/db.abc.local";
          };

          zone "5.168.192.in-addr.arpa" {
              type master;
              file "/etc/bind/db.192.168.5";
          };

    - name: Create forward zone file for abc.local
      copy:
        dest: /etc/bind/db.abc.local
        content: |
          $TTL    604800
          @       IN      SOA     main.abc.local. admin.abc.local. (
                                   2         ; Serial
                              604800         ; Refresh
                               86400         ; Retry
                             2419200         ; Expire
                              604800 )       ; Negative Cache TTL
          ;
          @       IN      NS      main.abc.local.
          main    IN      A       192.168.1.10
          branch  IN      A       192.168.5.10
          client  IN      A       192.168.5.15

    - name: Create reverse zone file for 192.168.5.x
      copy:
        dest: /etc/bind/db.192.168.5
        content: |
          $TTL    604800
          @       IN      SOA     main.abc.local. admin.abc.local. (
                                   2         ; Serial
                              604800         ; Refresh
                               86400         ; Retry
                             2419200         ; Expire
                              604800 )       ; Negative Cache TTL
          ;
          @       IN      NS      main.abc.local.
          10      IN      PTR     branch.abc.local.
          15      IN      PTR     client.abc.local.

    - name: Configure BIND9 forwarders
      lineinfile:
        path: /etc/bind/named.conf.options
        insertafter: '{'
        line: |
          forwarders {
              8.8.8.8;  // Google's DNS
              8.8.4.4;  // Google's DNS
          };

    - name: Restart BIND9
      service:
        name: bind9
        state: restarted
        enabled: yes

- name: Configure DNS Server
  hosts: branch_server
  become: yes
  tasks:
    - name: Install BIND9
      apt:
        name: "{{ item }}"
        state: present
        update_cache: yes
      with_items:
        - bind9
        - bind9utils
        - bind9-doc

    - name: Configure BIND9 named.conf.local as slave
      copy:
        dest: /etc/bind/named.conf.local
        content: |
          zone "abc.local" {
              type slave;
              file "/var/cache/bind/db.abc.local";
              masters { 192.168.1.10; };
          };

          zone "5.168.192.in-addr.arpa" {
              type slave;
              file "/var/cache/bind/db.192.168.5";
              masters { 192.168.1.10; };
          };

    - name: Configure BIND9 forwarders
      lineinfile:
        path: /etc/bind/named.conf.options
        insertafter: '{'
        line: |
          forwarders {
              192.168.1.10;  // Main server's IP
          };

    - name: Restart BIND9
      service:
        name: bind9
        state: restarted
        enabled: yes
```


### 3. NTP Configuration (using chrony) with iptables rules

**Playbook: `ntp.yml`**

```yaml
---
- name: Configure NTP with Chrony
  hosts: branch_server
  become: yes
  tasks:
    - name: Install chrony
      apt:
        name: chrony
        state: present
        update_cache: yes

    - name: Configure chrony to allow network
      lineinfile:
        path: /etc/chrony/chrony.conf
        line: 'allow 192.168.5.0/24'
        state: present

    - name: Restart chrony
      service:
        name: chrony
        state: restarted
        enabled: yes

    - name: Add iptables rules for chrony
      iptables:
        chain: INPUT
        protocol: udp
        destination_port: 123
        jump: ACCEPT

    - name: Add iptables rules for chrony output
      iptables:
        chain: OUTPUT
        protocol: udp
        source_port: 123
        jump: ACCEPT

    - name: Save iptables rules
      command: iptables-save > /etc/iptables/rules.v4

- name: Configure NTP with Chrony on Client Server
  hosts: client_server
  become: yes
  tasks:
    - name: Install chrony
      apt:
        name: chrony
        state: present
        update_cache: yes

    - name: Configure chrony to use branch server
      lineinfile:
        path: /etc/chrony/chrony.conf
        line: 'server 192.168.5.10 iburst'
        state: present

    - name: Restart chrony
      service:
        name: chrony
        state: restarted
        enabled: yes

    - name: Add iptables rules for chrony
      iptables:
        chain: INPUT
        protocol: udp
        destination_port: 123
        jump: ACCEPT

    - name: Add iptables rules for chrony output
      iptables:
        chain: OUTPUT
        protocol: udp
        source_port: 123
        jump: ACCEPT

    - name: Save iptables rules
      command: iptables-save > /etc/iptables/rules.v4
```

### 4. Basic Firewall Rules with additional iptables rules

**Playbook: `firewall.yml`**

```yaml
---
- name: Configure basic firewall rules and iptables
  hosts: branch_server
  become: yes
  tasks:
    - name: Install UFW
      apt:
        name: ufw
        state: present
        update_cache: yes

    - name: Allow SSH
      ufw:
        rule: allow
        name: 'OpenSSH'

    - name: Allow DHCP
      ufw:
        rule: allow
        port: 67
        proto: udp

    - name: Allow DNS
      ufw:
        rule: allow
        port: 53
        proto: udp

    - name: Allow NTP
      ufw:
        rule: allow
        port: 123
        proto: udp

    - name: Enable UFW
      ufw:
        state: enabled

    - name: Add NAT iptables rule
      iptables:
        chain: POSTROUTING
        table: nat
        out_interface: ens38
        jump: MASQUERADE

    - name: Add forward rule for related/established connections
      iptables:
        chain: FORWARD
        in_interface: ens38
        out_interface: ens33
        match: state
        state: RELATED,ESTABLISHED
        jump: ACCEPT

    - name: Add forward rule to allow traffic from ens33 to ens38
      iptables:
        chain: FORWARD
        in_interface: ens33
        out_interface: ens38
        jump: ACCEPT

    - name: Save iptables rules
      command: iptables-save > /etc/iptables/rules.v4

#

### Inventory File

**File: `hosts`**

```ini
[main_server]
192.168.1.10

[branch_server]
192.168.5.10

[client_server]
192.168.5.15

[all]
192.168.1.10
192.168.5.10
192.168.5.15
```

### Run the Playbooks

To execute the playbooks, use the following commands:

```bash
ansible-playbook -i hosts dhcp.yml
ansible-playbook -i hosts dns.yml
ansible-playbook -i hosts ntp.yml
ansible-playbook -i hosts firewall.yml
```

These playbooks will automate the configuration of DHCP, DNS, NTP with iptables rules, and basic firewall rules on the specified servers.

### Infrastucture as code:
To use Vagrant to provision a VM on VMware Workstation, simulating an additional branch server and client system, follow these steps. This guide will cover installing the necessary tools, setting up Vagrant, and creating Vagrantfiles to provision the VMs.

### Prerequisites

1. **VMware Workstation**: Ensure VMware Workstation is installed.
2. **Vagrant**: Install Vagrant on your system.
   - [Vagrant Installation Guide](https://developer.hashicorp.com/vagrant/install)
3. **Vagrant VMware Utility**: Install the Vagrant VMware Utility.
   - [Vagrant VMware Utility Installation Guide](https://developer.hashicorp.com/vagrant/docs/providers/vmware/vagrant-vmware-utility)
4. **Vagrant VMware Desktop Plugin**: Install the Vagrant VMware Desktop plugin.
   - Run the following command in your terminal:
     ```sh
     vagrant plugin install vagrant-vmware-desktop
     ```

### Steps to Provision VMs with Vagrant

#### 1. Create a Directory for Your Vagrant Project

Create a directory for your Vagrant project, and navigate into it:

```sh
mkdir vagrant
cd vagrant
```

#### 2. Initialize Vagrant

Initialize Vagrant in the directory:

```sh
vagrant init
```
![Screenshot 2024-07-31 210845](https://github.com/user-attachments/assets/73654d6e-0ec5-4ba9-9609-01ac52e71d37)

This will create a `Vagrantfile` in the directory. You will modify the `Vagrantfile`s to contain the resource creation for the 3 VMs.
Check for latest OS version at https://app.vagrantup.com/generic

To add additional configurations to each of the VMs, you can use a shell provisioner in the Vagrantfile. Below, I've created a script to set up DHCP, iptables, DNS, Chrony, and firewall rules. The script will be called provision.sh, and it will be referenced in the Vagrantfile for each VM.

```ruby
Vagrant.configure("2") do |config|
  # Define the "main" VM
  config.vm.define "main" do |main|
    main.vm.box = "generic/ubuntu2310"
    main.vm.network "private_network", ip: "192.168.1.10"
    main.vm.hostname = "main"

    main.vm.provider "vmware_desktop" do |v|
      v.vmx["memsize"] = "2048"
      v.vmx["numvcpus"] = "2"
      v.vmx["disk.size"] = "20000"
    end

    main.vm.provision "shell", path: "C:/Users/balog/Desktop/vagrant-branch-client/provision.sh"
  end

  # Define the "branch" VM
  config.vm.define "branch" do |branch|
    branch.vm.box = "generic/ubuntu2310"
    branch.vm.network "private_network", ip: "192.168.5.10"
    branch.vm.hostname = "branch"

    branch.vm.provider "vmware_desktop" do |v|
      v.vmx["memsize"] = "2048"
      v.vmx["numvcpus"] = "2"
      v.vmx["disk.size"] = "20000"
    end

    branch.vm.provision "shell", path: "C:/Users/balog/Desktop/vagrant-branch-client/provision.sh"
  end

  # Define the "client" VM
  config.vm.define "client" do |client|
    client.vm.box = "generic/ubuntu2310"
    client.vm.network "private_network", type: "dhcp"
    client.vm.hostname = "client"

    client.vm.provider "vmware_desktop" do |v|
      v.vmx["memsize"] = "2048"
      v.vmx["numvcpus"] = "2"
      v.vmx["disk.size"] = "20000"
    end

    client.vm.provision "shell", path: "C:/Users/balog/Desktop/vagrant-branch-client/provision.sh"
  end
end
```

#### Script provision for vagrant file

```bash
#!/bin/bash

# Update and install necessary packages
sudo apt-get update

# Install packages based on hostname
if [[ $(hostname) == "main" || $(hostname) == "branch" ]]; then
    sudo apt-get install -y bind9 bind9utils bind9-doc tinc

    if [[ $(hostname) == "branch" ]]; then
        sudo apt-get install -y isc-dhcp-server chrony
    fi
elif [[ $(hostname) == "client" ]]; then
    sudo apt-get install -y tinc chrony
fi

# Configure BIND9 for Main Server
if [[ $(hostname) == "main" ]]; then
    # BIND9 Configuration
    cat <<EOF | sudo tee /etc/bind/named.conf.local
zone "abc.local" {
    type master;
    file "/etc/bind/db.abc.local";
};

zone "5.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.5";
};
EOF

    cat <<EOF | sudo tee /etc/bind/db.abc.local
\$TTL    604800
@       IN      SOA     main.abc.local. admin.abc.local. (
                           2         ; Serial
                      604800         ; Refresh
                       86400         ; Retry
                     2419200         ; Expire
                      604800 )       ; Negative Cache TTL
;
@       IN      NS      main.abc.local.
main    IN      A       192.168.1.10
branch  IN      A       192.168.5.10
client  IN      A       192.168.5.15
EOF

    cat <<EOF | sudo tee /etc/bind/db.192.168.5
\$TTL    604800
@       IN      SOA     main.abc.local. admin.abc.local. (
                           2         ; Serial
                      604800         ; Refresh
                       86400         ; Retry
                     2419200         ; Expire
                      604800 )       ; Negative Cache TTL
;
@       IN      NS      main.abc.local.
10      IN      PTR     branch.abc.local.
15      IN      PTR     client.abc.local.
EOF

    cat <<EOF | sudo tee /etc/bind/named.conf.options
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    dnssec-validation auto;

    listen-on-v6 { any; };
};
EOF

    sudo systemctl start bind9

# Configure BIND9 for Branch Server
elif [[ $(hostname) == "branch" ]]; then
    # BIND9 Configuration
    cat <<EOF | sudo tee /etc/bind/named.conf.local
zone "abc.local" {
    type slave;
    file "/var/cache/bind/db.abc.local";
    masters { 192.168.1.10; };
};

zone "5.168.192.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.192.168.5";
    masters { 192.168.1.10; };
};
EOF

    cat <<EOF | sudo tee /etc/bind/named.conf.options
options {
    directory "/var/cache/bind";

    forwarders {
        192.168.1.10;
    };

    dnssec-validation auto;

    listen-on-v6 { any; };
};
EOF

    sudo systemctl start bind9

    # Configure DHCP Server
    cat <<EOF | sudo tee /etc/dhcp/dhcpd.conf
option domain-name "company.local";
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.5.0 netmask 255.255.255.0 {
    range 192.168.5.15 192.168.5.50;
    option routers 192.168.5.10;
    option domain-name-servers 192.168.5.10, 8.8.8.8;
}
EOF

    sudo sed -i 's/INTERFACESv4=""/INTERFACESv4="ens33"/' /etc/default/isc-dhcp-server

    sudo systemctl start isc-dhcp-server

    # Enable IP Forwarding and configure NAT
    sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
    sudo sysctl -p
    sudo iptables -t nat -A POSTROUTING -o ens38 -j MASQUERADE
    sudo iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
    sudo iptables -A FORWARD -j ACCEPT
    sudo sh -c "iptables-save > /etc/iptables/rules.v4"
#    sudo apt-get install iptables-persistent -y

    # Configure Tinc VPN for Branch Server
    sudo mkdir -p /etc/tinc/vpn/hosts

    cat <<EOF | sudo tee /etc/tinc/vpn/tinc.conf
Name = branch
AddressFamily = ipv4
Interface = tun0
ConnectTo = main
EOF

    cat <<EOF | sudo tee /etc/tinc/vpn/tinc-up
#!/bin/sh
ifconfig \$INTERFACE 10.0.0.2 netmask 255.255.255.0
EOF

    cat <<EOF | sudo tee /etc/tinc/vpn/tinc-down
#!/bin/sh
ifconfig \$INTERFACE down
EOF

    sudo chmod +x /etc/tinc/vpn/tinc-up /etc/tinc/vpn/tinc-down

    cat <<EOF | sudo tee /etc/tinc/vpn/hosts/branch
Address = 192.168.5.10
Subnet = 10.0.0.2/32
EOF

    sudo tincd -n vpn -K4096
    sudo systemctl enable tinc@vpn
    sudo systemctl start tinc@vpn

# Client Configuration
elif [[ $(hostname) == "client" ]]; then
    # Configure Netplan
    cat <<EOF | sudo tee /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: yes
      routes:
        - to: default
          via: 192.168.5.10
      nameservers:
        addresses:
          - 192.168.5.10
          - 8.8.8.8
EOF

    sudo netplan apply

    # Configure Chrony
    sudo sed -i '/pool /d' /etc/chrony/chrony.conf
    echo "server branch.abc.local iburst" | sudo tee -a /etc/chrony/chrony.conf
    sudo systemctl start chrony
    sudo systemctl enable chrony
fi

# Configure Chrony on Branch Server
if [[ $(hostname) == "branch" ]]; then
    sudo sed -i '/pool /d' /etc/chrony/chrony.conf
    echo "server main.abc.local prefer iburst" | sudo tee -a /etc/chrony/chrony.conf
    sudo systemctl start chrony
    sudo systemctl enable chrony
fi

# Common Configuration
# Set hostname and update /etc/hosts
sudo hostnamectl set-hostname $(hostname).abc.local
cat <<EOF | sudo tee -a /etc/hosts
192.168.1.10 main.abc.local
192.168.5.10 branch.abc.local
192.168.5.15 client.abc.local
EOF
```

This provisioner will set up BIND9 for both the main and branch servers, configure Tinc VPN, set up DHCP on the branch server, and provide internet access to the client.


#### 5. Start the VMs

```
vagrant up 
```
![Screenshot 2024-08-01 154008](https://github.com/user-attachments/assets/8ede35a6-3fc9-40bf-9cfc-1b8db29ab240)

#### 6. Verify the VMs

```
vagrant status
```
![Screenshot 2024-08-01 194413](https://github.com/user-attachments/assets/c93926c7-f9a8-4002-9f78-acae092b6325)

#### Use vmware workstation to discover and manage the running VMs
- Open vmware workstation, click on `File`, then select option `scan for virtual machines`
- Browse for the installation location of the created VMs, follow the instructions on screen and launch it.
  ![Screenshot 2024-08-01 194521](https://github.com/user-attachments/assets/3c6ff113-1174-4a2c-8bf5-0ba05e5c1844)
  ![Screenshot 2024-08-01 194538](https://github.com/user-attachments/assets/2d8d149c-2882-4405-9a31-ad04d11ae56d)

#### Verify the VMs on the vmware workstation
   ![Screenshot 2024-08-01 195604](https://github.com/user-attachments/assets/3adaff73-3711-4de8-9bdb-eab95b583c10)
   ![Screenshot 2024-08-01 200203](https://github.com/user-attachments/assets/706c419b-3b11-46a7-8297-97be7f4c53b0)


#### Other vagrant commands

```bash
vagrant halt #to halt the VM
vagrant destroy #to remove the VM setup
vagrant validate #to validate the configuration files in the vagrant directory
vagrant status #show status of vagrant VMs
```

### In Conclusion
Completing this scenario will provide you with practical experience in network administration, enabling you to apply the concepts and commands learned to address real-world challenges faced by DevOps engineers and system administrators.
