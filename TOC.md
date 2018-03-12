# Synopsis
Fabrikam, Inc. is starting a drone delivery service, and is taking an approach based on microservices to implement the technical components of the service.

The reference guide for the microservice design can be found at the Azure Architecture Center:[https://docs.microsoft.com/en-us/azure/architecture/microservices]

As the guide focuses on microservices there are some "missing pieces" to have a fully built out Drone Delivery Service. This guide builds upon the work already done, but focuses on the Identity and Access Management architecture needed to implement authentication and authorization for such a solution.

Detailed knowledge of the microservices guide is not a pre-requisite, but familiarity is assumed. Topics already discussed will not be repeated here.

## Audience
This intent of this guide is to provide an architectural overview of Azure Active Directory as it relates to delivering an "app" (with "app" being applied in a broad term here), with some details targeting developers.

# The Azure Active Directory Landscape
##	AAD "Classic"
##	AAD B2B
##	AAD B2C
# OAuth basics
# The different login components
## Login for employees (AAD Classic)
## Login for partners (B2B/B2C)
## Login for customers (B2C)
# Protecting APIs (API Management, JWT Validation)
## AuthN/AuthZ for back-end resources (SAS vs OAuth)
# Identity inside a K8S cluster vs outside the cluster