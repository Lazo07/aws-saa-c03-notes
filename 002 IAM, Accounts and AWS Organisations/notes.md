# IAM, Accounts and AWS Organisations

## Identity Policies

### Identities
- IAM users
- IAM groups
- IAM Roles

### IAM Policies/ IAM Identity Policies
- Type of policy which gets attached to identities inside AWS.
- A set of security statements to AWS.
    - it grants or denies access to AWS products and features to any identity that uses that policy.
- Can also be attached to resources (ex. S3 bucket) by referencing IAM users and IAM roles using ARNs.

### Identity policies/ policy documents
- created using json:

![Alt text](pic/JN-0001.png)

- What makes up a statement:
    - Sid (Statement ID)
        - optional field
        - lets you identify a statement and what it does.
        - best practice is to use it.
    - Action
        - format #1: \<service> : \<operation>
        - format #2: \<service> : *
        - format #3: list of multiple independent actions.
    - Resources
        - matches AWS resources.
        - format #1: [*]
        - format #2: [arn, arn]
        - ARN = Amazon Resource Name
    - Effect
        - either allow or deny.
        - controls what AWS does if the action and the resource part of the statement match the operation that you're attempting to do with AWS.

### Policy Rules
- First priority: Explicit DENY
    - If you have a statement which explicitly denies access, nothing can overrule that.
- 2nd priority: Explicit ALLOW
    - If there's both explicit DENY and ALLOW policies, explicit DENY will take priority.
- 3rd priority: Default DENY (implicit)
    - If there are no explicit policies, then implicit DENY will take effect. 
- All identities start off with no access to any AWS resources except for Account root user. <i>If they're not given access, then they are not allowed access.</i>
- deny, allow, deny.

### Policy Types
- Inline policies
    - assign json to 3 separate accounts/users.
    - when to use?
        - For special or exceptional allows or denies.
        - Generally, this is for exceptions to the normal access rights that you want to grant.
- Managed policies
    - created as their own object.
    - attach one object to multiple accounts/users.
    - advantages:
        - Reusable
        - Low management overhead
    - 2 types of managed policies:
        - AWS managed policies = created and managed by AWS.
        - Customer managed policies = you can create and manage so you can define them as per the exact requirement of your business.

<br>

## IAM Users and ARNs

### IAM Users
- An identity used for anything requiring long-term AWS access (ex. humans, application, service accounts)
- Starts with Principal = unknown identity
    - needs to be authenticated by IAM
        - via user/pass if it's a human
        - via access key if it's a application or someone trying to use the CLI.
    - Once authenticated, the principal is now known as an authenticated idenity.
    - once known as authenticated indentity and the principal tries to do an action (ex. upload to an S3 bucket), AWS checks that that identity is authorized to do so by checking policies.

![Alt text](pic/IAMUsers-1.png)

### Amazon Resource Name (ARN)
- Uniquely identify resources within any AWS accounts.

![Alt text](pic/IAMUsers-2.png)

### ARN fields
- Partition
    - the partition that the resources is in.
    - for standard AWS regions, partition = AWS.
    - If you have resources in other partitions, partition = AWS-\<partition name>
        - sample: china beijing region = AWS-cn
- Service 
    - Service namespace that identifies the AWS product (ex. S3, IAM, RDS)
- Region
    - The region that the resources your referring to resides in.
- Account ID
    - Account ID of the AWS account that owns the resource.
- Resource/ Resource Type
    - Varies depending on the service.
    - resource id can be the ID or the name of an object.

### IAM Users limitations
- Only allowed max 5000 IAM users per account.
    - IAM is a global service so this is a per account limit, not region.
- An IAM user can be a member of max 10 IAM groups.
- If you have a project/scenario that needs more than 5000 identifiable users/identity, then it's likely that IAM users are not the right identity to pick for that solution. However, this can still be fixed by using IAM Roles and Identity Federation (to be discussed).

## IAM Groups
- Containers for IAM users.
- exists to manage large sets of IAM users easier.
- you can't login to IAM groups (have no credentials of their own).
- Used solely for organizing IAM users.
- an IAM user can be a member of multiple IAM groups.

### Benefits of groups
- allow effective administration-style management of users.
    - ex. groups can represent team, projects, functional groups in the business
- Groups can have both inline and managed policies attached to it.
- IAM group does not have limit for memberships. 
    - adding 5000 IAM users (max allowed count of users per account) to a single IAM group is possible.

### Limitations of groups
- Limit = 300 groups per account, but this can be increased with a support ticket.
- There is no concept of all-users group.
- You can create a group and then add all IAM users in it, but you have to create and manage it yourself and it doesn't exist natively unlike other identity management solutions. 
- IAM groups cannot have nestings (groups within groups)
- IAM groups can have users and permissions attached.
- IAM groups do not have login credentials
- Groups are NOT a true identity. They can't be referenced as a principal in a policy.
- A resource policy cannot grant access to an IAM group. You cannot reference them in a resource policy.
