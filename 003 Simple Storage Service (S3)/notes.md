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

## Object Versioning and MFA Delete

### Object versioning
- controlled at a bucket level.
- Starts with disabled state. You can optionally set to enable but you can NOT make it disabled again.
- Once enabled, you can set it to suspended. Unlike disabled state, you CAN switch a suspended bucket back to enabled again.
- Once supended, it stops new versions from being created but it does nothing with the existing versions.
- key = file name. this is unique.
- id = always null if versioning is disabled. If enabled, AWS will set an id value.
- when an object is modified: 
    - disabled versioning: it will be replaced by a new object. 
    - enabled versioning: AWS will create a new object with different id value (version number) higher than orig value to denote latest version.
    - You can specify the version to interact with using a particular id. If none supplied, then it's implied that you want to interact with the current latest version.
- When an object is deleted:
    - Objects are not really deleted, instead, a Delete Marker will be created.
        - Delete marker is basically a copy of the current object but hides all previous versions of that object.
        - You can remove Delete Markers. The object will then be "undeleted" and will contain back all of its previous versions. The latest version of the "undeleted" object will also be set back as its latest version.
    - There's a way on how to "fully" delete an object (as in the object itself will now be gone) and this is via Version Delete.
        - To do that, delete an object and specify the particular version id.
        - If what you're deleting is the object's most recent version, then the next most recent will be set as the current version.
- Unlike "objects", When working with object "versions", everything is permanent.

### MFA Delete
- Something that is enabled within the versioning configuration on a bucket.
- If enabled:
    - it means that MFA is required to change bucket versioning state (enable, disable, suspend).
    - MFA is required to fully delete any versions.
- When changing bucket's versioning state or deleting a particular version of an object via API call, you need to pass  this concatenated string along with any API calls to interact: `[Serial number of MFA token] + [Code generated by MFA token]`

<br>

## S3 Performance Optimization 

### Single PUT Upload
- single data stream to S3.
- if Stream fails, then the whole upload fails.
- It requires full restart.
- Limited to 5GB upload max.
- Slow upload speed.


### Multipart upload
- Faster upload.
- breaking data up to different individual parts.
- 100MB is the minimum data size to use multipart.
- Multipart can have a max of 10,000 parts.
- 1 part can have 5MB - 5GB size.
    - last part is a leftover, so it can be lower than 5MB if needed.
- parts can fail and be restarted, there's no need to restart from scratch.
- It improves transfer rates. Transfer rate = speed of all parts
    - if there's a single stream limitation on ISP or any network inefficiencies, then you more effectively use the internet bandwidth by splitting the original blob of data.

### S3 Transfer Acceleration
- Uses the network of AWS edge locations.
    - this connects directly to S3 location which makes it faster.
- default = switched off
- to enable it, there are some restrictions:
    - the bucket name cannot contain periods
    - needs to be DNS compatible in its naming.

## Key Management Service (KMS)
- used for encryption.
- Regional and public service.
- Capable of handling different key architectures (ex. symmetric and asymmetric keys)
- Capable of cryptographic operations (ex. encrypt, decrypt)
- cryptographic keys never leave KMS. It ensures that the key never leaves and are held securely within the service.
    - `FIPS 140-2 (L2)` compliant service (US security standard).

### KMS Keys
- Used by KMS
- KMS keys can be used for up to 4KB of data.
    - low limitation, because it's usually used on small bits of data or to generate other keys.
    - how KMS gets around this:
        - using Data Encryption keys (DEKS).
- container of the <u>physical key material</u>.
    - data that makes up the key.
    - date held by KMS which is used to encrypt and decrypt things that you give to KMS.
    - Can be generated by KMS or imported into KMS.
- KMS keys contain:
    - Id, 
    - creation date, 
    - key policy (resource policy), 
    - descriptions, 
    - state of the key
- role separation: encrypt, decrypt, manage keys have their own separate permissions.
- Key Concepts:
    - default: KMS keys are isolated to a region and never leave KMS.
    - can support multi-region keys.
    - can be AWS owned
        - collection of KMS keys for use in multiple AWS accounts.
        - Operates in the background and you largely don't need to worry about them.
    - can be Customer owned
        - AWS managed keys
            - created automatically by AWS when you use a service such as S3 which integrates with KMS.
            - can't be customized.
            - Rotation can't be disabled for AWS managed keys.
                - Set to rotate approx. 1 time per year.
        - Customer managed keys
            - created explicitly by the customer to use directly in an application or within an AWS service.
            - more configurable (ex. You can edit key policy)
            - Rotation is optional for customer managed keys.
                - enabled by default.
                - happens approx. 1 time per year.
        - Both AWS and customer managed keys can support <u>Rotation</u>.
            - Rotation is where physical backing material (data used to actually do cryptographic operations) is changed.
    - KMS key contains the backing key, the physical key material and all previous backing key caused by rotation.
        - means as the key is rotated, data encrypted with old versions can still be decrypted.
    - You can create aliases (shortcuts to keys).
        - Aliases aer also per region.


### Data Encryption Keys (DEKs)
- another type of key that KMS can generate.
- generated using KMS key, using the GenerateDataKey operation.
    - Generates a data encryption key which can be used to encrypt/decrypt data more than 4KB in size.
- DEKs are linked to the KMS key which created them, so KMS can tell which KMS key was used to genereate a specific DEK.
- KMS does not store DEK in any way.
    - KMS provides it to you or the service using KMS then discards it.
    - It gets discarded because it doesn't actually do the encryption and decryption of data. It's either you or the service using KMS that performs those operations.
- when DEK is generated, KMS provides you with 2 versions:
    - plaintext version = can be used immediately to perform cryptographic operations.
    - ciphertext/encrypted version = encrypted using the KMS key that generated it. Can be given back to KMS for decryption.
- architecture:
    - you would generate a data encryption key immediately before you wanted to encrypt something.
    - You would encrypt the data using the plaintext version.
    - Once encryption is done, discard the plaintext version.
    - Store the encrypted key version along with the encrypted data.
    - to decrypt the data, pass the encrypted DEK back to KMS and ask for it to decrypt it using the same KMS key used to generate it.
    - once the encrypted DEK is decrypted, decrypt the data with it.
    - Discard the decrypted DEK.

### Key policies and Security
- unlike other services, KMS does not automatically trust identities under an account (ex. IAM users), so you need to explicitly trust the identities.
- Permissions on keys are controlled in a few ways.
    - via Key policies (a resource policy)
    - via combining Key Policies + IAM Policies (not ideal to split into two, it's better to focus on using Key policy only)
    - via Key Policies + grants (to be discussed if needed)

## S3 Server-side Encryption (SSE)
- Buckets are not encrypted, objects are.
- 2 main encryption architectures
    1. Client-side encryption
        - objects being uploaded are encrypted by the client before they leave.
        - The data is ciphertexted the entire time.
        - data is received in a scrambled form and then stored in a scrambled form.
        - you handle everything (keys, process, tooling)
    2. Server-side encryption (SSE)
        - Even though the data is encrypted in transit using HTTPS, the objects themselves aren't initially encrypted.
        - Inside the tunnel, the data is in its original form.
        - When reaching S3 endpoint, the objbect will be encrypted and then sent to S3 storage.
        - you allow S3 to handle some processes.
        - mandatory by AWS. You can no longer store unencrypted objects via S3.
- 2 types of encryption
    1. at rest
        - how data is stored on disk in an encrypted way.
        - This will be the focus in current lesson.
    2. in transit
        - used when transferring from user side to S3 (vise versa).
        - uses an encrypted tunnel
            - you can't see raw data inside the tunnel.
            - uses this as default but there are exceptions (will be discussed elsewhere in the course)

### 3 types of server-side encryption (SSE) for S3 objects
1. SSE-C
    - server-side encryption with customer provided keys.
2. SSE-S3
    - server-side encryption with Amazon S3 managed keys.
    - this is default.
3. SSE-KMS
    - enhancements on SSE-S3
    - server-side encryption with KMS keys stored inside AWS Key Management Service (KMS).

### 2 components of server-side encryption
- encryption process and decryption process
    - symmetrical so you use the same key for both encryption and decryption.
- generation and management of the cryptographic keys
    - used as part of the encryption and decryption processes.

### How does the 3 types of SSE handle the 2 components?
- SSE-C
    - customer is responsible for the key generation and management.
    - S3 manages the encryption and decryption processes.
        - essentially offloading the CPU requirement of cryptography to AWS.
- SSE-S3 (AES256)
    - AWS is responsible for the key generation and management.
    - AWS handles the encryption and decryption processes
    - Uses S3 key for encrypting the object key, which is used to encrypt the object
        - 100% managed by AWS, you can't control this key.
    - uses AES-256 algorithm.
    - not suitable for you if you need the following:
        - control for access of key, rotation of key, role separation (S3 admin is not limited).
- SSE-KMS
    - AWS (S3 + KMS) is responsible for the key generation and management.
    - AWS (S3) handles the encryption and decryption processes
    - Involving KMS.
    - Uses KMS Key isntead of S3 key.
    - Customer can create managed KMS keys.
    - advantage:
        - control for access of key
        - rotation of key
        - role separation.

<br>

## S3 Bucket Keys
- There's a limit on how many PUTs per second you can do with S3 + KMS.
- Use KMS key to generate a time limited bucket key for encryption/decryption inside S3.
    - offloads the work from KMS to S3.
    - reduces the costs and improves scalability because it significantly reduces KMS API calls.
- Not retroactive
    - only affects objects or object encryption process after it's enabled on a bucket.

### Things to keep in mind when using S3 bucket keys
- after enabling the key, whne using cloudtrail to view KMS logs, then those logs will show the bucket ARN instead of your object ARN.
- Since offloading the work from KMS to S3, you're going to see fewer CloudTrail events for KMS.
- Bucket key works with same region replication and cross region replication.
    - When S3 replicates an encrypted object, it preserves the encryption settings of that encrypted object.
    - if replicating plaintext to a bucket that uses encryption, then S3 encrypts that object on it's way through to the destination with the destination's encryption configuration (will have ETAG changes).

<br>

## S3 Object Storage Classes
1. S3 standard
    - default storage class.
    - Should be used for frequently accessed data, which is important and non-replaceable.
    - Should always be the default choice. Only consider moving to other storage classes when there is a specific reason to do so.
    - when user calls API to upload, objects are replicated acress 3 different AZs in the AWS region.
    - S3 API endpoint responds with HTTP/1.1 200 OK = your data has been stored durably within the product.
    - durability:
        - 99.999999999% durability
        - for 10,000,000 objects, 1 object loss per 10,000 years.
        - Used to detect and fix any data corruption
            - Replication over 3 AZ's
            - Content-MD5 Checksums
            - Cyclic Redundancy checks (CRCs)
    - most balanced.
2. S3 Standard-IA (Standard Infrequent access)
    - process, replication, durability the same as with S3 Standard.
    - Cheaper than S3 standard.
    - In exchange of the cheaper price, there are compromises:
        - Has new cost component: Retrieval fee
            - Has a per gigabyte of data retrieval fee on top of transfer fee.
            - overall cost increases with frequent data access.
        - Has minimum duration charge of 30 days.
        - has minimum capacity charge of 128KB per object.
    - This class is designed for infrequently accessed data, long-lived data which are important but where access is infrequent.
    - Don't use for lots of small files, temporary data, data that are constantly accessed, unimportant data or data that can easily be replaced.
3. S3 One Zone-IA (One Zone Infrequent Access)
    - Cheaper than S3 Standard-IA and S3 Standard with compromise.
    - has the same charges and durability as with S3 Standard-IA.
    - The diffence is it does not provide multi-AZ resilience model. It only uses one AZ within the region.
    - This class is used for long-lived data as well with infrequent access.
    - Use this class for non-critical data or data which can easily be replaced.
        - replica copies, intermediate data.