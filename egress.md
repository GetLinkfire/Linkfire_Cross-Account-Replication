# HOWTO

## Linkfire Egress

### How to accept data delivery from Linkfire

#### 👩‍💼Client Setup

*ℹ️ This document should be sent to the client. It contians all of the steps required for the client to configure their S3 bucket to accept cross region replicated data from a Linkfire S3 bucket.*













## 📋 Required Information

### Source (Linkfire)

### Destination (Client)	



## Step 1

### Enable Versioning

Ensure versioning is enabled on the client owned source bucket.

```bash
aws s3api put-bucket-versioning \
--bucket <CLIENT BUCKET NAME> \
--versioning-configuration Status=Enabled
```

Linkfire will do the same on its buckets.

---

In the AWS Management Console you can enable Versioning by going to the Properties Tab of your bucket and Clicking Enable.

![Screen Shot 2020-10-22 at 12.00.40 PM](versioning.png?raw=true)

## Step 2

### Update Bucket Policy

Add the following policy to the client bucket to ensure Linkfire can deliver data to your s3 bucket.

```
{
    "Version": "2012-10-17",
    "Id": "LinkfireRawDataFeedDeliveryPolicy",
    "Statement": [
        {
            "Sid": "AllowLinkfireReportDelivery",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<LINKFIRE AWS ACCOUNT ID>:root"
            },
            "Action": [
                "s3:GetBucketVersioning",
                "s3:PutBucketVersioning",
                "s3:ReplicateObject",
                "s3:ReplicateDelete",
                "s3:ObjectOwnerOverrideToBucketOwner"
            ],
            "Resource": [
                "arn:aws:s3:::<CLIENT BUCKET NAME>",
                "arn:aws:s3:::<CLIENT BUCKET NAME>/*"
            ]
        }
    ]
}
```

## 3) Profit
Notify your Linkfire account manager or the security engineer that you are working with that you have enabled versioning and the s3 bucket policy, and we will the test and ensure replication/delivery is fully enabled.


## Questions

If you have any questions, please reach out to [security@linkfire.com](mailto:security@linkfire.com). We also provide means to communicate through a secure channel during implementation using [Keybase](https://keybase.io/linkfiresec). 
