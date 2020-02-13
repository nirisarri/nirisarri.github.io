---
layout: post
published: false
title: >-
  Install and run Minikube on a Virtual machine - Install Hyper-V on Windows 10
  and create an Ubuntu 18 VM
date: '2020-02-13'
---
This is the second post of the Install and run Minikube on a Virtual machine series.

### Install Hyper-V on Windows 10

Reference the article [Install Hyper-V On Windows 10](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v). 

As I mentioned before, the base platform I will be using is Microsoft's Hyper-V. You need windows 10 Enterprise, pro or Education in order to install and run Hyper-V. Of course you can do it on windows Server, but I will be focusing on Win10.

1. Ensure your BIOS has Virtualization enabled. This is different for every manufacturer. Check with yours.
2. Open a PowerShell console as administrator
3. Run the following Command:

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

4. When completed, reboot.

This should install the Hyper-V Manager console, where you can create virtual machines.

### Create an Ubuntu 18 Virtual Machine

I am still looking for a way to create it using powerShell, but here is the method using the Hyper-V Manager.

1. Download a .ISO for Ubuntu Server 18 from the [Ubuntu website](https://ubuntu.com/download/server). at this time, the latest LTS (Long term support) version is 18.04.4 LTS

2. The Hyper-V manager, create an external virtual switch by

   - Click on Virtual Switch Manager

   - In _what type of virtual switch do you want to create_ Select **external** and click on **Create Virtual Switch**.

   - In **name** put something memorable, for example `minikube-external-switch`.

   - In **External Network** select an active external network card you want to use for your VM. 

     > **NOTE:** This has an issue: lets say you select a wired adapter and you connect then on other place through wireless, then your VM will be disconnected. You have to keep this in mind, and adjust depending on your environment. If you have a better idea on how to be able to have internet connection and at the same time access to the internal ports of the VM from the host, please drop me a line 😊) 

​	2. Back in the Hyper-V Manager click on **New ▶ virtual Machine**.

3. Fill up a Name, for example `nic-minikube`, and change the VM's location, If you wish, in the **Name and location** section. This name is only to identify your VM in the Hyper-V Manager, and the location will depend on how you want to organize your VMS. The only recommendation is that the faster the hard drive, the better, so prefer an SSD to an HHD.
4. In **Specify Generation ** select **Generation 1** as it is more common and portable.
5. In Assign Memory, select 8192 MB (8 GB - I know: it is a guzzler.). Leave checked the **Use dynamic memory for this Virtual Machine** checkbox.
6. In **Configure Networking** choose the switch created in the step 2. In this example it was `minikube-external-switch`.
7. In **Connect virtual Hard Disk** select **Create a virtual hard disk** and leave the defaults.
8. In **Installation Options** select **Install an operating system from a bootable CD/DVD-ROM** and in **media** point to the ISO downloaded in the step 1.
9. Click on **Finish** to kick off the process.  Once it finishes, you will see the VM in your **Virtual Machines** list.
10. Select the VM from the Virtual Machines list and in Actions select under _&lt;your vm name>_ ▲ the **Settings...** button.
11. Select the Processor element under Hardware and change the **Number of virtual processors** to **4** and click **Ok** to save the changes.
12. Right-click on your VM and select **Connect**. A black window should appear.
13. Click on the **Start** button in the middle of the screen to start the VM.
14. In the virtual screen select your OS language and press **Enter**.
15. Select **Install Ubuntu Server** and press **Enter**.
16. Select the language you wish
17. select the keyboard layout you wish
18. select **Install Ubuntu**.
19. In the **Network Connections** select the **eth0** connection and press 
20. Enter to open the menu. then select **Edit IPv4**

    - Select **Manual** in **IPV4 Method**

    - Select a subnet that is part of your network but outside your DHCP range. I have my DHCP configured to use `10.0.0.2 - 10.0.0.100` and my gateway is running on `10.0.0.1`, so I will be selecting a `10.0.0.140` address in a `10.0.0.0/24` CIDR subnet, with `10.0.0.1` as gateway and my local DNSs `10.0.0.110, 10.0.1.111`
21. In **Configure Proxy** leave Proxy address blank
22. In **Configure Ubuntu** archive mirror leave the default
23. In **FileSystem setup** leave Use an entire disk selected.

    - in the **Choose the disk to install to** select the default.

- In the **FILE SYSTEM SUMMARY** leave all by default and select **Done** and **Continue**

24. In the **Profile Setup** Enter the following Information:
    - **Your Name**: your name
    - **Your Server's name**: your server name (I choose nic-minikube)
    - **Pick a Username**: a username to use by default. I always choose 'nic'
    - **Choose a password**: a password. This will be the root user so choose wisely.
25. In **SSH Setup** select Install OpenSSH server by pressing space.
26. In **featured Server Snaps** do not select anything as we will be using APT to install the packages.
27. When it finishes click **Reboot now**. Probably the first time it reboots will fail and will ask you to remove the installation media. Just click OK and watch it reboot again.
28. You should be greeted by a bash login prompt. Use your login/password to log in.
29. Now you have a running Ubuntu server.
30. Type the following command to download and apply  the latest patches:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

31. OPTIONAL: You could SSH into it from a WSL or bash window in your host 😎 I usually edit my \windows\system32\drivers\etc\hosts file and add the following line to it:

   ```
   # [your VM IP] [A name to identify your IP] 
   10.0.0.140 nic-minikube
   ```

   so I can SSH by typing  `ssh nic@nic-minikube` from my WSL terminal.
Next step: turn nested virtualization on.
