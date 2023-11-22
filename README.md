# Active Directory HomeLab

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

![DC-settings-adapter-1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sexsc99athq9oh059bb2.png)
![DC-settings-adapter-2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bft9q1mrokl8trny7dou.png)

After completing the network adapter configuration, we will need to complete the initial setup of Windows Server 2019 on our DC.

![DC-installation-media](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bm4cywgz481s2z2o0xr6.png)
![DC-installation-start](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/d8kehset9uwd0x2vdwis.png)

After initial setup is completed, we need to configure the network adapters in OS. We can identify which one is going to be the internal adapter by checking the IPv4 addresses their respective connection details.

![Windows-server-install-screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/00tbxabnsp05zjk5sopd.png)

The one with `10.0.2.15` IPv4 is our internet facing adapter, whereas the other is our internal one since the IPv4 is autoconfigured, so we can now label them as _INTERNET_ and X_Internal_X respectively.

![adapter-settings](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/c2a4a3e4-9638-4909-b9a7-2a47787e6086)

Renaming them will be easier for the configuration we'll be doing throughout the project.

### Setting up IP addressing

Now we will be setting up the IP addressing for our internal adapter with the following configuration:

- IP address: `172.16.0.1`
- Subnet mask: `255.255.255.0`
- Default gateway:  `. . .`
- Preferred DNS server: `127.0.0.1`

![Internal-properties](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u44nmjjool2j4zreinyz.png)

Note: When we install AD, we will configure the DC to use itself as the primary DNS server, so that's why we have a loopback IP, `127.0.0.1` in the Preferred DNS Server field.

Last thing is to rename the PC to `DC` and restart before we install our Active Directory Domain Services.

![Rename-pc](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hrsuwt5drjjhm379if1y.png)

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

### Setup DHCP server

Doing this will allow our Windows 11 client to be auto assigned an IP address and allow our client to browse the internet, even though the VM is on a virtual private network.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/457afa12-5791-48ec-bbeb-18927daaaa74

After installation, it's time to configure the DHCP and setup a scope. Again, the purpose of DHCP is to allow clients on the network to automatically be assigned an IP address. Referencing our network diagram, I will create a scope that will give IP addresses in a range of `172.16.0.100-200`, so a range of 100 addresses that the DHCP server can give out. The DHCP lease time will be kept at the default 8 days. If this were a cafe, for example, I would want to probably use a lease period of 2hrs, since new clients will be logging into our wifi network frequently. We don't want to lock out IP address with a long lease time like 20 days. If we did that, we'll run out of IP's if the new client connection volume exceeds our IP cycle rate set by the DHCP lease time. This effectively prevents new clients from connecting to the internet through our network, since new IP's cannot be assigned. A better solution for the cafe situation would be to have a large IP range with short DHCP lease time. However, we are working with a homelab setup, so the default values will work fine for this situation.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/00275584-d3db-490d-bd82-3df7d0e7ff2e

### Config to allow us to browse internet from domain controller

In order to get the powershell script from the internet and execute it on our domain controller, we'll need to do some more confiruation. We need to disable the IE Enhanced Security Config setting in our domain controller.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/f3b11a94-b2a4-4d58-a472-14478c12fccb

With the IE security feature disabled, we can download the script to the server [here](https://github.com/joshmadakor1/AD_PS/archive/master.zip).

### Powershell script to create 1000 users

Once we've downloaded and extracted the script files, we're ready to run it using PowerShell ISE in administrator mode to create our users. Before running the script though, open the text file, we'll add our first and last name to the `names.txt` file, just to make it easy to remember for when we log into the client computer after we're done with our server.

![PowerShell-ISE](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/22d7b7bb-0eaa-4ed0-9f15-724074d33d2c)

By default Windows won't allow us to execute unknown scripts from the internet, so we need to enable execution of our script by running the following command: `Set-ExecutionPolicy Unrestricted` and then click "Yes to All".

![Screenshot 2023-11-21 161052](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/0256a33c-bf07-4c2c-8377-b74a0668abe8)

Now, we run the script. There will be some visible errors during execution, but that's because of duplicates in the names.txt file, which shouldn't mess with the script's execution.

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/704967d1-2018-4636-9c76-121bf38d4908

Confirming our users have been created in AD:

https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/0efe3457-81bf-40a1-9260-87733b9c8cf2

And we're done with our domain controller setup.

### Setup client virtual machine

Finally we can create our client machine, which will act as a user in our domain we created. We will call our machine `CLIENT1`. This will simulate an employee machine on our domain.

![Screenshot 2023-11-20 195901](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/b10b92f4-d992-4d24-8682-3830a1fa3ea0)

We'll set our network adapter to the internal network we configured when initally setting up our domain controller:

![Screenshot 2023-11-20 195956](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/34999bb5-98d2-41e4-8cc9-b5927cd5506f)

On initial setup, we can name our computer `CLIENT`, so when we get to the desktop, all we need to do is add our computer to our domain and authenticate the change with our domain admin credentials

![Screenshot 2023-11-20 203350](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/587e9b3e-67bd-4ed0-9dd7-64622b936529)
![Screenshot 2023-11-20 203541](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/9e510d4b-eaac-41cd-ad87-0e243511e50d)

Let's logout and log in as our generate user `dnguyen`.

![Screenshot 2023-11-20 203915](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/79581328-de8a-4f9d-9b37-d50958ea7d4e)

After a successful login, let's run `whoami` to confirm my domain\user.

![Screenshot 2023-11-20 204123](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/013a4f03-b357-4281-ab10-4797ede5e5f0)

Let's ping google.com to confirm we have access to the internet and for good measure, we can ping our domain: `mydomain.com`

![Screenshot 2023-11-20 203237](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/0c1e7816-7ad7-49c2-89f1-2b3006f2b927)

Back on our DC, we can take a look in our DHCP > dc.mydomai.com > IPv4 > Address Leases to see our client machine listed with its unique IP in our defined scope.

![Screenshot 2023-11-20 203707](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/bd277c67-4509-45e6-b81b-5fe80f6dbc52)

We can also confirm in Active Directory > mydomain.com > Computers that our client is listed there as well.

![Screenshot 2023-11-20 203803](https://github.com/dmnuggins/Active-Directory-HomeLab/assets/7257923/54047cf8-36bd-4c2e-a44e-df4e7d1d0c7e)

And success, that is the end of the lab! 🙌
