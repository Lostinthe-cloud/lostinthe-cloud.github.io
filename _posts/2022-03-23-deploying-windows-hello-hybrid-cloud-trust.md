---
layout: post
title: Deploying Windows Hello Hybrid Cloud Trust
date: 2022-03-23 020:00 -0400
categories: Intune 
tags: intune azure hello identity passwordless 
image: /assets/img/HelloConfig.png
---

### Accessing on prem resources is easier than ever!
----  

This is now the default recomendation for any environment where certificate issuance is not needed.  
The process we are outlining below will be for full AAD joined devices, if you are utilizing HAAD (Hybrid join), you can deploy this method via group policy for step 2.  

*Do note that for HAAD there is an added check before provisioning to verify DC connectivity.*

<ins>Prerequisites<ins>  
- Windows 10 21H2 or Windows 11  
- MFA - Any provider  
- Windows Server 2016 or later DC's  

#### Outline of steps  
1.	Configuring Azure AD Kerberos   
    a.	Install required powershell module  
    b.	Create the Kerberos object  
    c.  Set Kerberos connection  
2.	Configure Intune Policy  
    ```*NOTE – We will use configuration policies here so they are all in one spot, you may read documentation that WHB can be set under device enrollment and this is true, they achieve the same end result, but it puts the config in two different places.```  
    a.	Create a Windows Configuration Policy for Windows Hello for Business configuration.  
	b.  Create a configuration profile for cloud trust setup    
3.  Testing  
    a.  Target test group  
    b.  Test devices via force sync from client  
    ```*NOTE - Intune tries to notify devices of policy updates within 5 minutes of publishing, but if there is a delay or issue for any reason it may mean you have to wait 24 hours for the device to check-in, this way we force that update on the client.```   
    c.	Verify status via logs  


#### Creating the Azure AD Kerberos Object  
----  

* From an elevated powershell prompt execute:  
```powershell  
Install-Module -Name AzureADHybridAuthenticationManagement -AllowClobber
```  
Accept the prompt for untrusted repository 'PSGallery' to complete installation.  

![Desktop View](/assets/img/InstallADKerberos.png){: w="700" h="400" } 


* After installation we will create the kerberos object utilizing this command:  

```powershell
# Specify the on-premises Active Directory domain. A new Azure AD
# Kerberos Server object will be created in this Active Directory domain.
$domain = "LostInCloud.com"

# Enter a UPN of an Azure Active Directory global administrator
$userPrincipalName = "administrator@LostInCloud.onmicrosoft.com"

# Enter a domain administrator username and password.
$domainCred = Get-Credential

# Create the new Azure AD Kerberos Server object in Active Directory
# and then publish it to Azure Active Directory.
# Open an interactive sign-in prompt with given username to access the Azure AD.
Set-AzureADKerberosServer -Domain $domain -UserPrincipalName $userPrincipalName -DomainCredential $domainCred
```
*You will need to replace the values for $domain and $userPrincipalName with the correct details for your environment.*  

The above script is assuming that we are utilizing a form of modern auth. If you are not using modern auth refer to this [Microsoft documentation](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-security-key-on-premises#example-1-prompt-for-all-credentials) for other example scripts.  

#### Configuring Intune policies for enablement and Cloud Trust configuration 
---- 

 We will define a user group to target for our example "WHFBCloudTrustUsers". Both of the required policies will be assigned to this group.  

 * Begin by creating a new configuration profile Templates > Identity Protection. Give it a fitting name/description and continue.  

 ![Desktop View](/assets/img/HelloCreate.png){: w="700" h="400" }  

 * Lets move into defining our Hello config. Below is an example of what would be fitting for most environments. These settings are fairly straight forward, The option for Pin recovery should be discussed with your security team and additional config may be needed for other third-party MFA providers.

![Desktop View](/assets/img/HelloConfig.png){: w="700" h="400" }  
&ensp;&ensp;&ensp;&ensp;*Something to note here is that enabling "certificates for on-prem resources" will disable cloud trust.*    
  
  &ensp;&ensp;  
  
Now we will create another configuration profile this time choosing Templates > Custom. Fill in the name and description as needed and continue  

* Create the OMA-URI: 
```
./Device/Vendor/MSFT/PassportForWork/tenant ID/Policies/UseCloudTrustForOnPremAuth
```  
&ensp;&ensp; Replace tenant ID with the one that correlates to your environement. This can be found easily at <portal.azure.com>

* Set Data Type: Boolean and Value: True  

![Desktop View](/assets/img/HelloOMA.png){: w="900" h="700" }  

* Once these policies are defined and assigned to our target group, we will do a force sync on a device to quickly retrieve the policies and test provisioning. A sign out and sign in on the test device should present the user with steps to configure biometrics if applicable, and configure a pin.  

#### Validation 
----

If we hop over into event viewer and navigate down to Applications and Services Logs\Microsoft\Windows > User Device Registration we should be presented with the status's of the pre-req checks for the partial TGT before provisioning begins. This test has 3 states: Yes and No are self explanatory, Not tested is acceptable for devices that are AADJ only as this pre-req check is skipped for them as no line fo sight to the local DC is needed for signin.  

![Desktop View](/assets/img/devreg.png){: w="700" h="400" }

We can also verify these checks using 'dsregcmd /status'. Furthermore, if we execute 'klist' we should be able to verify our cached kerberosTGT tickets after provisioning is completed.  

Thats all there is to it. Utilizing this method puts your environment in a good place to begin looking at FIDO implementation for the future!









