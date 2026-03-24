# Lab 01 – Entra Basics

## Purpose
The purpose of this lab was to build practical foundational understanding of **Microsoft Entra ID** and core concepts within **Identity and Access Management (IAM)**. The focus was on working with users, groups, administrative units, authentication-related settings, and built-in administrative roles to understand how identities are organized, how access can be structured, and how privileges should be handled in a modern cloud-based identity platform.

## Environment
The lab was carried out in a dedicated **Microsoft Entra test tenant**, which made it possible to work in a safe and controlled lab environment without affecting production.

## Steps Performed
I started by creating five test users in **Identity → Users → All users** to simulate different roles in an organization:

- Anna HR
- Erik IT
- Sara Sales
- Admin Test
- Temp Consultant

I then created four **Security Groups** in **Identity → Groups → All groups** and assigned users based on role and purpose:

- **HR-Users** → Anna HR
- **IT-Admins** → Erik IT, Admin Test
- **MFA-Pilot** → Temp Consultant
- **Sales-Users** → Sara Sales

This structure was used to simulate how an organization can group identities based on department, administrative function, and security initiatives such as MFA rollout.

Next, I went to **Identity → Administrative units** and created an **Administrative Unit** named **Students**. I then added two test users to this administrative unit to understand how administration can be scoped to a specific part of the directory.

I also reviewed the following built-in roles in **Identity → Roles & administrators**:

- **Global Administrator**
- **User Administrator**
- **Security Administrator**
- **Privileged Role Administrator**

The role review focused on the apparent responsibility of each role, why the role is security-sensitive or important, and whether the role appears broad or more limited in scope.

Finally, I reviewed **Password reset → Properties** and confirmed that **Self-Service Password Reset** was set to **None** for standard end users. At the same time, I noted that administrators were still enabled for self-service password reset and were required to use two authentication methods for password reset.

I then reviewed **Authentication methods → Policies** and observed which authentication methods were enabled or disabled in the tenant. I noted that **Microsoft Authenticator**, **Temporary Access Pass**, **Software OATH tokens**, and **Email OTP** were enabled for **all users**.

## Results
The lab resulted in a working test environment where users, groups, group memberships, and an administrative unit could be verified in the Microsoft Entra portal. By creating a simple identity structure with clearly defined user types and group assignments, I gained practical experience in how Entra can be used to organize identities in a structured way.

I also gained a clearer understanding of the difference between **groups**, **roles**, and **Administrative Units**. Groups are used to organize users and simplify access management, roles are used to grant administrative privileges, and Administrative Units are used to scope administration to a specific part of the directory.

Reviewing the built-in roles also provided a clearer picture of how different administrative roles represent different levels of responsibility, risk, and privilege. Reviewing **Password reset** and **Authentication methods** also helped me understand how authentication and recovery settings contribute to identity security.

## Security Reflection
This lab clearly showed that **groups, administrative roles, and administrative units are not the same thing**, even though all three influence how identities are managed in Microsoft Entra. Groups are mainly used to organize users and simplify access, roles represent elevated privileges, and Administrative Units are used to scope administration.

One important lesson was why **Global Administrator** should be tightly limited. The role has very broad access and can affect large parts of the environment. If such an account is compromised or misused, the consequences can be severe. Because of this, highly privileged roles should only be assigned to a very small number of accounts with a clear operational need.

The lab also showed why **group-based access** is often better than assigning rights directly to individual users. When access is managed through groups, administration becomes more structured, easier to review, and more scalable over time.

Reviewing **Password reset** and **Authentication methods** also showed that secure identity management is not only about creating users and groups, but also about how users authenticate and recover access. Different authentication methods represent different security levels, and administrative accounts require stronger protection than standard users.

## Recommendation to Customer
I recommend that organizations build their access model with **groups as the foundation**, where users are organized by function, department, or security need. This is usually better than assigning rights directly to individual users, because it provides better structure, simpler administration, and stronger control over who has access to what.

I also recommend that administrative roles be handled separately and very carefully. In particular, roles such as **Global Administrator** should be limited and only assigned when there is a clear business need. Highly privileged roles should be reviewed regularly and protected by strong security controls to reduce the risk of overprivileged accounts.

For larger or more complex environments, it can also be useful to use **Administrative Units** to scope administration between different parts of the organization.

## What I Learned
- how users are created and managed in Microsoft Entra
- how groups can be used to structure identities by function and need
- how group membership supports a more controlled and scalable access model
- the difference between a group, an administrative role, and an Administrative Unit
- why **Global Administrator** is a highly sensitive role that should be limited
- why group-based access is often better than direct assignment to individual users
- how built-in Entra roles differ in responsibility, sensitivity, and privilege level
- why **least privilege** is a core principle in secure IAM governance
- that **Self-Service Password Reset** can be configured differently for end users and administrators
- that administrators often have stricter password reset security requirements
- that **Authentication methods** control how users are allowed to authenticate
- that **Microsoft Authenticator** and **Temporary Access Pass** appear to be important methods in modern Entra environments
