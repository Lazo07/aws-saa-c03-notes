# Simple Storage Service (S3)

## S3 Security (Resouce Policies & ACLs)

### S3
- Private by default.
- initial access is granted to Account root user (account which created it) by default.
- Granting access priviledge for other accounts will need to be done explicitly.
    - By using S3 bucket policies.

### Bucket Policies
- A type of <u>AWS Resource Policy</u>.
    - Just like an identity policy, but as the name suggests, they're attached to resources instead of identities.
    - Answer the question: who can access that resource?
- Identity polices or Resource Policy, which to use?
    - Limitation of identity policy is that you can't grant access/give identity policy to users outside your account. So for these kinds of scenarios, you can use resource policies instead.
    - Resource policy allows access for identities from the same account or different accounts.
    - Resource policy can allow or deny anonymous principals (principals not identified by AWS), unlike identity policy  that needs to be attached to an authenticated identity.
- Common uses of bucket policies (based on resource policy description):
    - Grant access to other AWS accounts.
    - Allow anonymouse access to a bucket.

### Parts of Bucket policies
- One major different with identity policy is the explicit <u>"Principal"</u> component.
    - defines which principals are affected by the resource policies.

![Alt text](pic/S3Security-1.png)

### Access Control List (ACLs) [LEGACY]
- subresource of buckets
- ACLs on objects and bucket
- Inflexible and allows simple permissions only
    - ex. Can't have conditions like bucket policies. Only has 5 possible permissions

### Block Public Access
- Additional settings/bounderies that only applies to anonymous principals.
- It will always apply no matter what the bucket policy contains.
- This is to avoid issues with unknowingly giving public access which results to data leak. This were caused by genuine lack of knowledge about the security model or config mistakes.

<br>

## S3 Static Website Hosting
- normal access of S3 objects is via AWS APIs.
- Static website hosting feature allows access via HTTP (ex. blogs)
- Static Website hosting common examples
    - creating static blogs.
        - Index and error documents needs to be set (HTML)
        - You can only use a custom domain (via Route53) with a bucket if the name of the bucket matches the domain.
    - Offloading
        - offload static media from a compute service/EC2 for example to an S3.
        - Compute services tend to be relatively expensive so it's ideal to offload what we can to S3.
        - needs to be placed in a bucket with static hosting enabled.
    - Out-of-band pages
        - a method of accessing something that is outside of the main way.
        - Error or status notification system. For example if a page is hosted in an EC2 and you want to show a page during its maintenance period, then it doesn't make sense to also put that page in the same EC2.
- S3 <- data : free
- S3 -> data : charged per GB
