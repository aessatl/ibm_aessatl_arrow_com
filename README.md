# PCC Hands on Lab #

![Overview](READMe_images/arrow.PNG)

**In our two hour lab session we will cover automation concepts and multiple tools to build an environement in Azure and later perform some automated configuration tasks**

**We will perform the following tasks**

* Create an Azure account if you do not have one to use already
* Use Powershell to pull down all the necessary components onto the Windows 10 workstation
* Clone the files from a pre-configured github repository
* Use Hashicorp Packer to create Azure images that contain customizations we will use later for configuration managagement 
* Use Terraform to build our infrastructure using our custom images
* Use Ansible to perform some configuration tasks against the target servers


## Prerequisites -  Azure set up ##

![azureportal](images/azureportal.PNG)

**If you do not already have an Azure account with a usable subscription please first visit portal.azure.com and select create account**

* This will require an email, phonenumber, and creditcard information


## Section 1 - Powershell set up 

**1.1 - Open powershell as an administrator and run the following command**

* This command will reach out to the git and download the powershell script found at https://github.com/kylejep/PCC_Labs/

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/kylejep/PCC_Labs/main/PCCinstall.ps1"
$file = "$env:USERPROFILE\Desktop\PCCinstall.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
```

**1.2 - Run the script from an administrator Powershell**
```Powershell
powershell.exe -ExecutionPolicy ByPass -File $file
```

**This script is used to pull down and install all the necessary software we will need for the lab namely:**

* **Chocolatey** - Windows based package manager used to install and manage software - https://chocolatey.org/
* **Git commandline**  - used to interact with git repos - https://community.chocolatey.org/packages/git
* **Packer** - image build manager used to create artifacts for server and infrastructrue builds expressed as code - https://community.chocolatey.org/packages/packer
* **Terraform** - Used to orchestrate the provisioning of infrastrcutre expressed as code - https://community.chocolatey.org/packages/terraform
* **Azure CLI** - azure command line tool that allows commands against Azure cloud - https://community.chocolatey.org/packages/azure-cli

<br/>

**1.3 - Change into a directory that you want to clone the repo into ie...**
```powershell
cd "~/desktop"
```
