# Setup for Hybrid Identity

Fabrikam, Inc. are already using Active Directory on-premises, and have decided to build upon this when moving into the cloud. While there are many options as to the details of how this is done there are three main routes one can go.

## Azure AD and AD are kept separate
In this setup you create separate identities for on-premises and cloud. In the basic scenario this means that you will have one set of users with @contoso.com as their domain suffix, and another set of users with @contoso.onmicrosoft.com. In general this is not a recommended configuration.

## Federated identities
The user objects are synchronized into Azure AD using a tool like AAD Connect. The actual authentication still happens in the local directory, and passwords never leave your own datacenter. This is a good setup if you want/need more fine-grained control over the auth process, but it requires you to build a highly available and redundant setup on-premises and has some extra maintenance effort to it so do consider things fully before choosing this.

## Syncronizing users and/or passwords
In addition to synchronizing the user objects you can choose to synchronize the password into the cloud as well. This is the easiest solution as it doesn't even require the local directory to have five nines of uptime. (You should not leave the domain controllers unavailable for extended periods of time though as that might cause issues.) As a compromise between keeping passwords on-premises and synching them into the cloud you can also use what is known as "pass-through authentication" where the AAD Connect server maintain an outbound connection to Azure AD which is used for auth purposes. This makes the user experince "cloud native", unlike ADFS which is visibly different, but doesn't add the complexity of federation. With pass-through auth the credentials flow through the cloud, but are not stored in Azure AD.

> Note: While a non-federated setup will require authentication and authorization flows to be maintained through the cloud by default it is entirely possible to maintain a federation service separately which is not part of the AAD Connect installation. For scenarios where you need to use features not offered by Azure AD you can still maintain an ADFS farm as a separate channel not interfering with the default.

Fabrikam, Inc. have decided to go all-in on the cloud and has configured their domain to do a full password sync into Azure AD. They still maintain an ADFS installation in case they need to set up legacy integrations, but it is not the preferred auth solution.

## More informations
For details on how to install Azure AD Connect see the following instructions:  
[https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect]