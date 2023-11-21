## Active Directory HomeLab

This project is a walkthrough of how I created an Active Directory homelab environment in VirtualBox using PowerShell, Active Directory, Windows 11, and MS Server 2019. The network consists of two VM's (virtual machines) where one will be the DC (domain controller) and the other is a client machine. I will have Active Directory (AD) installed on the DC and will generate 1000 randomized users in AD, which can be used to log into the client machine once the domain is set up and the client is properly added. This lab is a simulation of an enterprise network environment, so there will be some configurations that optimize for time and should not be included in a production-enterprise environment.

### Downloads

- [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)
- [Microsoft Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 11 ISO](https://www.microsoft.com/en-us/software-download/windows11)

### Network Diagram

I will reference this diagram for the project configurations
![network-diagram](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0sd2tbs24w3kxasbbonw.png)

### Creating the domain controller
The first virtual machine will function as our domain controller and will require two network adapters. After creating our machine, using the Server 2019 ISO, we will configure the network adapters. In the VM's Settings > Network page, leave Adapter 1 with the default NAT configuration. Enable Adapter 2 and set Attached to: Internal Network.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sexsc99athq9oh059bb2.png)
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bft9q1mrokl8trny7dou.png)

After completing the network adapter configuration, we will need to complete the initial setup of Windows Server 2019 on our DC.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bm4cywgz481s2z2o0xr6.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/d8kehset9uwd0x2vdwis.png)

After initial setup is completed, we need to configure the network adapters in OS. We can identify which one is going to be the internal adapter by checking the IPv4 addresses their respective connection details.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/00tbxabnsp05zjk5sopd.png)

The one with `10.0.2.15` IPv4 is our internet facing adapter, whereas the other is our internal one since the IPv4 is autoconfigured, so we can now label them as _INTERNET_ and X_Internal_X respectively.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qi343h9ixok4wmdq8816.png)

Renaming them will be easier for the configuration we'll be doing throughout the project.

### Setting up IP addressing

Now we will be setting up the IP addressing for our internal adapter with the following configuration:

- IP address: `172.16.0.1`
- Subnet mask: `255.255.255.0`
- Default gateway:  `. . .`
- Preferred DNS server: `127.0.0.1`

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u44nmjjool2j4zreinyz.png)

Note: When we install AD, we will configure the DC to use itself as the primary DNS server, so that's why we have a loopback IP, `127.0.0.1` in the Preferred DNS Server field.

Last thing is to rename the PC to `DC` and restart before we install our Active Directory Domain Services.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hrsuwt5drjjhm379if1y.png)

### Install Active Directory Domain Services

After booting back into the DC, I install Active Directory Domain Services:

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/082ac489-242e-4b72-b908-5e54bae02b4f

### Promote the server to domain controller

Now to promote our server to a domain controller. This will auto restart the VM after the wizard completes the promotion.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/c446b041-bfa6-4a16-a55f-343f4f7f574d

Upon next login, we see that our VM is now part of MYDOMAIN.

![Screenshot 2023-11-21 150253](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/ad786f2f-8a2e-411c-a62d-2c41a098476c)

Instead of the defaul Administrator account, I create my own domain admin account and promote it to Domain Admins.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/4d9df50a-8ba2-42f8-afb3-2a14ffe2bf90

### RAS / NAT

Now to configure RAS/NAT to allow our client VM that is on the virtual private network to access the internet through the domain controller.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/88d1f4da-40c4-4d80-aead-edcdf1a5d037

