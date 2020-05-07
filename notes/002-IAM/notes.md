# Identity and Access Management

Has Global scope across AWS.

Security for
* Users
* Groups
* Roles

Never use root account.

Policies are written in JSON. AWS has many existing policies you can use.

## User
* A user should be a *single* physical person, with their own unique account (not the root account). 
* Grant users the exact permissions they need for their role. Least privelage principle.
* Never commit credentials to a repo.


## Groups
* Groups contain users. Form groups based on functionality required for a set of users.

## Roles
* For internal AWS resources only.
* *One* Role per application.

##IAM Federation
Enterprise solution for using identity federaion and SAML. Like using Active Directory.


