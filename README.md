# AWS Static Website Hosting with Amazon S3 and CloudFront

A hands-on cloud project that hosts a static personal portfolio website on Amazon S3 and delivers it worldwide over HTTPS through Amazon CloudFront. The S3 bucket stays fully private: visitors can reach the files only through CloudFront, never directly from S3.

The project focuses on secure content delivery, least-privilege access, and clear documentation of the architecture and deployment.

**Project status:** Complete. All AWS resources were deleted after testing and documentation to avoid ongoing costs, so there is no live demo. The code, screenshots and documents in this repository show what was built and verified.

---

## Architecture

![AWS Static Website Hosting architecture](docs/architecture/AWS%20Static%20Website%20Hosting.png)

The editable diagram is available here: [AWS Static Website Hosting.drawio](docs/architecture/AWS%20Static%20Website%20Hosting.drawio)

| Component | Role |
|---|---|
| **Amazon S3** | Private bucket that stores the website files (origin) |
| **Amazon CloudFront** | Content delivery network that serves the site over HTTPS from edge locations |
| **Origin Access Control (OAC)** | Lets CloudFront sign its requests to S3, so only CloudFront can read the bucket |
| **S3 bucket policy** | Allows `s3:GetObject` only for this one CloudFront distribution |
| **S3 Block Public Access** | Stays enabled, so the bucket can never be made public by mistake |

The S3 bucket is in `eu-central-1` (Frankfurt). CloudFront is a global service, so it has no region: it serves the site from AWS edge locations around the world.

### How a request flows

1. A visitor opens the website through the CloudFront distribution URL.
2. CloudFront receives the request at the nearest edge location. HTTP requests are redirected to HTTPS.
3. If the file is already cached at that edge location, CloudFront returns it straight away.
4. If it isn't cached, CloudFront requests it from S3 and signs the request using Origin Access Control.
5. S3 checks the bucket policy, confirms the request comes from this distribution, and returns the file.
6. CloudFront caches the file at the edge and returns it to the visitor.

---

## Features

- Static portfolio website served over HTTPS
- Automatic HTTP-to-HTTPS redirection
- `index.html` served as the default root object
- Content cached at CloudFront edge locations for faster loading
- Private S3 bucket with no public access

---

## Security

- **No public bucket:** S3 Block Public Access stays enabled on the bucket.
- **Origin Access Control:** CloudFront signs every request to S3. OAC is the method AWS recommends, replacing the older Origin Access Identity (OAI).
- **Least privilege:** the bucket policy allows only one action (`s3:GetObject`), only on this bucket's objects, and only for this one distribution (checked with the `AWS:SourceArn` condition).
- **HTTPS only:** visitors who use HTTP are redirected to HTTPS.
- **No secrets in the repository:** the AWS account ID and distribution ID are masked in committed files and screenshots.

### Bucket policy

The policy used, with account-specific values replaced by placeholders:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<bucket-name>/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
        }
      }
    }
  ]
}
```

---

## Implementation

1. Created a private S3 bucket in `eu-central-1` and kept Block Public Access enabled.
2. Uploaded the website's `index.html` file.
3. Created a CloudFront distribution with the bucket's **S3 REST endpoint** as the origin. OAC only works with the REST endpoint, not the S3 static website endpoint, so static website hosting is not enabled on the bucket.
4. Created an Origin Access Control with request signing enabled ("Sign requests") and attached it to the origin.
5. Added the bucket policy above, so only this distribution can read the objects.
6. Set `index.html` as the default root object.
7. Set the viewer protocol policy to redirect HTTP to HTTPS.
8. Tested the website through the CloudFront distribution URL.

---

## Testing

| Test | Expected | Result |
|---|---|---|
| Open the CloudFront distribution URL (`http://<distribution-domain>.cloudfront.net`) | Website loads, with `index.html` served at the root URL | Pass |

---

## Screenshots

| Screenshot | What it shows |
|---|---|
| [CloudFront distribution](screenshots/01-cloudfront-distribution.png) | The distribution, its domain name and settings |
| [CloudFront OAC](screenshots/02-cloudfront-oac.png) | Origin Access Control with request signing |
| [S3 Block Public Access](screenshots/03-s3-block-public-access.png) | Public access blocked on the bucket |
| [S3 bucket policy](screenshots/04-s3-oac-bucket-policy.png) | Policy limited to the CloudFront distribution |

---

## Key Concepts and Lessons Learned

| Topic | What I learned |
|---|---|
| S3 REST endpoint vs website endpoint | OAC works only with the REST endpoint. The website endpoint would need a public bucket, which this project avoids. |
| OAC vs OAI | OAC is the current recommended method. It supports all S3 regions and SSE-KMS encryption, and uses signed requests. |
| Default root object | It applies only to the root URL (`/`). A subfolder like `/about/` would not load `/about/index.html` automatically; that would need a CloudFront Function. |
| 403 instead of 404 | Without `s3:ListBucket` permission, S3 returns `403` for a missing file. A CloudFront custom error response can show a friendly 404 page instead. |
| Caching | After changing a file in S3, CloudFront can keep serving the old version until the cache expires. A cache invalidation (for example `/*`) forces it to fetch the new version. |

---

## Cost Management and Cleanup

AWS resources may incur charges depending on usage and current pricing.

All resources were deleted after the project was completed, in this order:

1. Saved all screenshots for documentation.
2. **Disabled** the CloudFront distribution and waited for the change to deploy.
3. **Deleted** the CloudFront distribution. The distribution URL stopped working at this point.
4. Deleted the Origin Access Control (it can only be deleted once no distribution uses it).
5. Emptied the S3 bucket, then deleted it.

---

## Repository Structure

```
aws-static-website-hosting/
├── README.md
├── index.html             # Website file uploaded to Amazon S3
├── docs/
│   └── architecture/      # Architecture diagram (PNG + draw.io source)
└── screenshots/           # Evidence from the deployment
```

---

## Possible Improvements

- Add a custom domain with Amazon Route 53 and an SSL/TLS certificate from AWS Certificate Manager (the certificate must be in `us-east-1` for CloudFront)
- Add a custom 404 error page through CloudFront custom error responses
- Add security headers (such as HSTS) with a CloudFront response headers policy
- Automate deployment with GitHub Actions: sync files to S3, then invalidate the CloudFront cache
- Define the infrastructure as code (AWS CloudFormation or Terraform) instead of creating it in the console

---

## Technology Stack

Amazon S3 · Amazon CloudFront · Origin Access Control · AWS IAM (bucket policies) · HTML · Git and GitHub
