---
layout: post
title: Enable OneDrive file sync on shared workstations
date: 2022-03-17 09:30 -0400
categories: Intune
tags: intune onedrive
---
### So you want OneDrive to work for Shared multi-user devices?  
By default devices configured with a shared multi-user device profile will not be able to launch or access OneDrive via the desktop application.
To enable this we will need to create a CSP that enables OneDrive sync which is required for the application to launch, Followed by a secondary CSP that forces the device to use MDM policy over local group policy.

### Creating the OneDrive Filesync OMA-URI.
Open Endpoint Manager portal and create a new Windows Configuration profile. Select "Template" and then "Custom".  

Fill in your profile's details:
 ![Desktop View](/assets/post/OneDrive_Sync/name_template.png){: w="700" h="400" }{: .shadow }  
 
 Create a OMA-URI for our settings:  

 ``` ./Device/Vendor/MSFT/Policy/Config/System/DisableOneDriveFileSync (Integer = 0)```  
 
 ![Desktop View](/assets/post/OneDrive_Sync/OMARURI.png){: w="700" h="400" }{: .shadow } 

Continue through the profile configuration and assign to your device group.  

### Creating the MDMWins OMA-URI.  

 We care going to replicate the process from above creating a new configuration profile as "Template" and "Custom".  
 Once named and description given, we will be creating the OMA-URI for this setting.  
 
```./Device/Vendor/MSFT/Policy/Config/ControlPolicyConflict/MDMWinsOverGP (Integer = 1)```

![Desktop View](/assets/post/OneDrive_Sync/MDMWins.png){: w="700" h="400" }{: .shadow }  

Continue through the profile creation prompts and assign to your device group.  
After a sync on the endpoints One Drive should now be launching properly on Shared Profile workstations.


