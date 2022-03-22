---
layout: post
title: Deploying Windows Hello Hybrid Cloud Trust
date: 2022-03-22 07:26 -0400
categories: Intune
tags: intune azure hello 
---

### Late February Microsoft announced that they were releasing Windows Hello for Business Cloud Trust into Preview
In the past we had to deploy Hello utilizing a Key Trust model or Certificate. This required public key infrastructure (PKI) and enough 2016 DC's to support your environment's authentication needs. With "Cloud Trust" we no longer need to do any of this. After we have configured and deployed using this method users will be able to access on-prem resources without any delay between provisioning and authentication.  

*Prerequisites*  
- Windows 10 21H2 or Windows 11  
- MFA - Any provider  
- Windows Server 2016 or later DC's  

## Outline of steps  
1.	Configuring Azure AD Kerberos   
    a.	Install required powershell module  
    b.	Create the Kerberos object  
    c.  Set Kerberos connection  
2.	Configure Intune Policy  
    ```*NOTE – we will use configuration policies here so they are all in one spot, you may read documentation that WHB can be set under device enrollment and this is true, they do the same things, but you it puts config in two different places.```  
    a.	Create a Windows Configuration Policy for Windows Hello for Business configuration.  
	b.  Create a configuration profile for cloud trust setup    
3.  Testing  
    a.  Target test group  
    b.  Test devices via force sync from client  
    ```*NOTE - Intune tries to notify devices of policy updates within 5 minutes of publishing, but if there is a delay or issue for any reason it may mean you have to wait 24 hours for the device to check-in, this way we force that update on the client.```   
    c.	Verify status via logs  

