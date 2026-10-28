Ticket 1 – Virtual Machine Setup
1. What was asked
The task was to prepare two virtual machines for a new developer: one Linux machine and one Windows machine. Both machines should have internet access and be able to communicate with each other. The setup should also be documented so another person can repeat the process.
2. What was actually needed
I needed to choose the operating systems, allocate hardware resources and configure the network connection.
I used Ubuntu Server 24.04 ARM64 and Windows 11. Both virtual machines were running on my Mac using UTM.
3. What I did
First, I created a new Ubuntu virtual machine.
Ubuntu:
Name: lab03-ubuntu
RAM: 2048 MB
CPU: 2 cores
Disk: 25 GB
Network: Shared Network
Windows:
Name: lab02-win11
RAM: 4096 MB
CPU: 2 cores
Disk: 80 GB
Network: Shared Network
I checked the network connection using the ping command.
At first, Windows Firewall blocked incoming ping requests. I added an inbound firewall rule:
netsh advfirewall firewall add rule name="Allow Ping IPv4" protocol=icmpv4:8,any dir=in action=allow
After that, both virtual machines could communicate successfully.
Ubuntu to Windows: 4 packets received, 0% packet loss.
Windows to Ubuntu: 4 packets received, 0% packet loss.
4. Evidence
The evidence folder contains screenshots of the network tests and text files with Linux system information and ping results.
5. What I would do differently
Next time, I would collect all screenshots and configuration details during the installation process to make the documentation more complete.
Snapshot Limitation
I initially tried to use my existing Ubuntu and Windows virtual machines in VirtualBox. However, I could not establish a connection between them because they were using different virtual network environments.
To solve this problem, I created a new Ubuntu virtual machine in UTM and used my Windows 11 virtual machine in UTM. After configuring the network, I successfully tested communication between both machines in both directions.
I could not create snapshots because my virtual machines use Apple Virtualization, which does not support snapshots on macOS Sequoia 15.2. I documented this limitation instead of claiming that snapshots were created.
