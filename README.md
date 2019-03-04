# Linkfire Integrations
# Cross Region 🗑 Bucket Replication

## Details

### Source Account
* Account ID: **`<SOURCEOWNERID>`**: ${Integration-aws-account-id}
* Bucket Name **`<SOURCEBUCKET>`**: ${Integration-owned-bucket-name}

### Linkfire
* Account ID: **`<DESTINATIONOWNERID>`**: ${linkfire-aws-account-id}
* Bucket Name **`<DESTINATIONBUCKET>`**: ${linkfire-ingress-destination-bucket-name}

*⚠️ The above details will be shared between integration and Linkfire through a secure channel.*

## 1) Enable Versioning

Ensure versioning is enabled on the Integration owned source bucket.

```
aws s3api put-bucket-versioning \
--bucket <SOURCEBUCKET> \
--versioning-configuration Status=Enabled
```

## 2) Create Role & Permissions

Save the following file locally as: `S3-role-trust-policy.json`. 
This is your trust policy template.

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Principal":{
            "Service":"s3.amazonaws.com"
         },
         "Action":"sts:AssumeRole"
      }
   ]
}
```


Create Role and apply trust policy template.

```
aws iam create-role \
--role-name Linkfire_Replication_Role \
--assume-role-policy-document file://s3-role-trust-policy.json
```

*⚠️ Grab that ARN from the newly created role. you will need it later.*

---

The following step will create the `Linkfire_Replication_Role` policy which you will attach to the new role you just created.

Save the following file as: `S3-role-permissions-policy.json`  
This is the policy you will bind to the role:

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "s3:GetObjectVersionForReplication",
            "s3:GetObjectVersionAcl"
         ],
         "Resource":[
            "arn:aws:s3:::<SOURCEBUCKET>/*"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:ListBucket",
            "s3:GetReplicationConfiguration"
         ],
         "Resource":[
            "arn:aws:s3:::<SOURCEBUCKET>"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:ReplicateObject",
            "s3:ReplicateDelete",
            "s3:ReplicateTags",
            "s3:GetObjectVersionTagging"

         ],
         "Resource":"arn:aws:s3:::<DESTINATIONBUCKET>/*"
      }
   ]
}
```

Now bind the policy to the new role:

```
aws iam put-role-policy \
--role-name Linkfire_Replication_Role \
--policy-document file://s3-role-permissions-policy.json \
--policy-name Linkfire_Replication_Role_Policy
```

## 3) Add Replication to the bucket

Save the following file as: `replication.json` This will enable replication on the bucket as long as the roles and policies are in place:

*⚠️ Update the `Role` with the ARN of the role you created before this.*

```
{
  "Role": "aws:iam::<SOURCEOWNERID>:role/Linkfire_Replication_Role",
  "Rules": [
    {
      "Status": "Enabled",
      "Priority": "1",
      "DeleteMarkerReplication": { "Status": "Disabled" },
      "Filter" : {},
      "Destination": {
        "Bucket": "arn:aws:s3:::<DESTINATIONBUCKET>",
        "Account":"<DESTINATIONOWNERID>",
        "AccessControlTranslation":{"Owner":"Destination"}
      },
    }
  ]
}
```

Run the following command to add the replication configuration to your source bucket. Be sure to provide source-bucket name.

```
aws s3api put-bucket-replication \
--replication-configuration file://replication.json \
--bucket <SOURCEBUCKET>
```

Verify replication settings:

```
aws s3api get-bucket-replication \
--bucket <SOURCEBUCKET>
```

## Questions
Please provide Linkfire with the account number of the AWS account that this bucket is hosted in, so we can grant the permission to the account to replicate files to the Linkfire bucket. We will communicate this through a secure channel during implementation. 