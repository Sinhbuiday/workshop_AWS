---
title: "Week 6 Worklog"
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

- Explore the security services on AWS.
- Amazon Identity and Access Management.

### Tasks to be carried out this week:

| Day | Task                                                  | Start Date | Completion Date | Reference Material                                                                            |
| --- | ----------------------------------------------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Amazon Cognito - AWS Organization                   |  8/6/2026 | 9/6/2026      |
| 2   | - AWS Identity Center - Amazon Key Management Service | 10/6/2026 | 11/6/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - AWS Security Hub                                    | 13/6/2026 | 14/6/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Week 6 Achievements:

- Shared Responsibility Model:

  - The shared responsibility model: When using cloud-based services, security responsibilities are shared between the customer and the service provider.
  - Customers are responsible for protecting their data, applying best practices, and using services to ensure the security of virtual machines, the hypervisor, or database servers.
  - Security responsibilities vary depending on the service layer:
    - Basic services **(IaaS)**
    - Managed server services **(PaaS)**
    - Fully managed services **(AWS)**

- Amazon Identity and Access Management (IAM):

  - Root account: This account has full access to all AWS services and resources, and can grant additional permissions to resources.
  - Account information: Linked (when registering the account), Not restricted in permissions.
  - Best Practices: Create and use an IAM Administrator User, lock the root user credentials (shared users), and ensure domain information is renewed via the root user’s email.
  - IAM is a service that helps you control access to services and resources within your AWS account. IAM allows you to create multiple user accounts (IAM users) with different credentials and permission levels.
  - IAM Principal is an entity that can perform actions on resources within your AWS account.

    - AWS account and root user
    - IAM users
    - Federated users (using web identity or SAML federation)
    - IAM roles
    - Assumed-role sessions
    - AWS services
    - Anonymous users (not recommended)

  - **IAM Users** are not separate AWS accounts. IAM users can be managed through the Management Console or by using an access key/secret key for programmatic access (AWS CLI or AWS SDK).

    - By default, an IAM User has no permissions when created.
    - IAM Users should not be used to manage access to other applications or systems.
    - Permissions for IAM Users are primarily assigned by attaching IAM Policies to the IAM User.
    - For easier management, permissions are often assigned by grouping multiple IAM Users into an IAM Group.
    - An IAM Group cannot be a member of another IAM Group.

  - IAM Policy: There are two types of IAM policies

    - **Identity-based Policy**: Attached to an **IAM Principal**.
    - **Resource-based Policy**: Attached to an **AWS Resource**.

  - The way IAM permissions are enforced always prioritizes Deny over Allow. If there is an explicit Deny, then even if another IAM Policy grants Allow, the result will still be Deny.

  - IAM Role:
    - Allows defining a set of permissions to access resources (by attaching an IAM Policy to an IAM Role). An IAM Role does not have credentials to access the Management Console or AWS CLI/SDK.
    - When an IAM User wants to use an IAM Role, the IAM User can assume the IAM Role. Once the role is assumed, the user’s permissions are replaced by the permissions granted to the IAM Role. In addition, temporary security credentials are issued to the IAM User or an AWS Service to access AWS services. Assuming a role works with **AWS STS – Security Token Service**, which generates temporary credentials (similar to an access key).
    - For a user to assume an IAM Role, the role must be attached to a resource-based IAM policy, called an IAM Role trust policy, which defines who can use the IAM Role.
    - IAM Roles are commonly used to follow the principle of least privilege, for example by granting permissions to other AWS accounts to access resources in the current AWS account.
    - Besides being used by IAM Users, IAM Roles can also be used to grant AWS Services access to AWS resources.
    - A common use case is granting permissions to applications running inside compute services (such as EC2) through IAM Roles.

- Amazon Cognito - AWS Organization:

  - Amazon Cognito: A managed AWS service that provides authentication, authorization, and user management for web and mobile applications. Users can sign in directly with a username and password or through a third party such as Facebook, Amazon, or Google.
  - The two main components of Amazon Cognito are User Pool and Identity Pool:
    - **User Pool** stores user information and provides sign-up and sign-in options for application users.
    - **Identity Pool** grants users access to other AWS services.
  - AWS Organizations: AWS Organizations helps centrally manage and govern an environment that consists of multiple AWS accounts.
    - AWS Organizations can create new AWS accounts, allocate resources, organize AWS accounts into Organizational Units (OU), and simplify centralized billing **(consolidated billing)**.
    - AWS Organizations can apply Service Control Policies (SCPs) to OUs or AWS accounts. SCPs establish permission boundaries for IAM Users or IAM Roles within those OUs or accounts.
    - SCPs provide the ability to implement deny-based policies.

- AWS Identity Center (SSO):
  - AWS Identity Center helps manage access to AWS accounts and external applications.
  - The identity source can be a user directory within AWS Identity Center or linked to Active Directory (AWS Managed Microsoft AD, on-premises Microsoft AD via two-way trust, or AD Connector).
  - Permission Sets define the access level of Users and Groups to AWS accounts within an AWS Organization. The permissions are stored in AWS Identity Center and provisioned to AWS accounts as IAM Roles. You can assign multiple permissions to a single User.
- Amazon Key Management Service(KMS):
  - AWS Key Management Service (KMS) helps create and manage encryption keys, used for encrypting and decrypting data on AWS.
  - Encryption keys always remain within AWS KMS, ensuring compliance with the FIPS 140-2 standard.
  - CMK (Customer Managed Key) serves as the primary resource in AWS KMS. A CMK can be up to 4KB in size. However, in practice, CMKs are mainly used to generate, encrypt, and decrypt Data Keys—keys used outside of AWS KMS to encrypt data.
- AWS Security Hub: A service that enables standardized security checks based on security standards and best practices.
  - Security Hub runs continuously, monitoring services in the AWS account and performing security checks based on AWS best practices and industry standards (e.g., PCI DSS).
  - Security Hub provides findings in the form of scores and helps standardize the identification of accounts and resources that may require attention.
