---
layout: post
title: Configure local group membership for Intune devices
date: 2022-03-15 18:27 -0400
categories: Intune
tags: intune 
---

### No more OMA-URI's and hunting for SID's...  

This may come as a shock, But adding, updating, or removing members of the local groups on devices was not always straight forward. There was a time when you had to hunt down the SID for the GA role, or AAD device admin role, as well as any others you wanted to define on the local level. After aquiring these you would need to create or modify an existing configuration and assign them to your device groups.  


Thanks to the great Intune team up at Microsoft they ahve provided us with a new way achieve our goals.  
```*Devices MUST be on 20H2 or newer to support this feature*```  

In your Endpoint Management portal navigate to "Endpoint Security in the left pane"  and select "Account Protection"  

![Desktop View](/assets/img/endpointsecurity1.png){: w="700" h="400" }  

### Creating the policy  

You will begin by creating a new policy selecting your platform as Windows 10 and later and selecting "Local user group membership"  

![Desktop View](/assets/img/createlocal2.png){: w="700" h="400" }  

As you see in the screenshot above, you are now able to select the local groups you want to add to.  
- Administrators  
- Users  
- Guests  
- Power Users  
- Remote Desktop Users  
- Remote Management Users  

You are able to add multiple groups per policy. I would however suggest creating a policy for each Local group and/or device groups for different use cases.  

The middle field is were we will select our action.  
- Add (Update) -- Allows addition of members to the specified group while retaining current members  
- Remove (Update) -- Removes members of the specified group while retaining current members  
- Add (Replace) -- Replaces current members of specified group with new members.  

For our final configuration item we are able to define our User selection type.  
- Users/Groups -- Supported for AADJ devices only  
- Manual -- Supported for AADJ and Hybrid devices  

```*The manual method is helpful when it is necessary to allow your on-prem AD users into local groups*```  

Once you have selected your configuration and assigned the necessary Users/Groups you are now ready to test deploy this policy in your lab for validation.