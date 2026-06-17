666, the number of the beast.

---
brigata_id: 16b9e43f-bc5d-4d61-a18c-63691e354ec0
title: "Wasabi: New Bucket + API User Policy"
---

# Wasabi: New Bucket + API User Policy

A checklist for creating a Wasabi bucket for a company and locking it down to a single API user.

---

## 1. Create the Bucket

- [ ] Log in to the Wasabi Console
- [ ] Navigate to **Buckets** → **Create Bucket**
- [ ] Enter bucket name (use a clear naming convention, e.g. `company-name-purpose`)
- [ ] Select the appropriate **region**
- [ ] Leave versioning/logging at defaults (or configure as needed)
- [ ] Click **Create Bucket**

---

## 2. Create the API User

- [ ] Navigate to **IAM** → **Users** → **Create User**
- [ ] Enter a username (e.g. `company-name-api`)
- [ ] Check **Programmatic (API) access**
- [ ] Save the **Access Key ID** and **Secret Access Key** securely (you won't see the secret again)

---

## 3. Create a Bucket Policy (Restrict to This User Only)

- [ ] Navigate to **Buckets** → select the bucket → **Policies** tab
- [ ] Add the following bucket policy, replacing the placeholders:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:user/USERNAME"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME",
        "arn:aws:s3:::BUCKET_NAME/*"
      ]
    }
  ]
}
```

- [ ] Replace `ACCOUNT_ID` with your Wasabi account ID
- [ ] Replace `USERNAME` with the API user's username
- [ ] Replace `BUCKET_NAME` with the bucket name
- [ ] Save the policy

---

## 4. Verify Access

- [ ] Use a tool (e.g. `aws s3 ls s3://BUCKET_NAME` with Wasabi endpoint, or CyberDuck/S3Browser) to confirm the API user can access the bucket
- [ ] Confirm no other users can access the bucket

---

## Notes

- Wasabi endpoint format: `s3.REGION.wasabisys.com` (e.g. `s3.us-east-1.wasabisys.com`)
- To find your Account ID: top-right menu → **Account** in the Wasabi console
- If you want to restrict to specific actions instead of `s3:*`, swap in actions like `s3:GetObject`, `s3:PutObject`, etc.
