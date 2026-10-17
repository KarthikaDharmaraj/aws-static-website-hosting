# AWS Static Website Hosting with Amazon S3 and CloudFront

## Project Overview

This project demonstrates how to host a personal portfolio website using Amazon S3 and Amazon CloudFront.

The goal was to build a simple, secure and cost-conscious website while gaining hands-on experience with AWS storage, content delivery, HTTPS and access control.

**Live Website:** `https://YOUR_CLOUDFRONT_DOMAIN`

## Architecture

![AWS Static Website Hosting Architecture](docs/architecture/architecture.png)

**Architecture source:** [Editable draw.io diagram](docs/architecture/architecture.drawio)

### Request Flow

1. A visitor opens the portfolio website using the CloudFront distribution URL.
2. CloudFront delivers the website over HTTPS.
3. When content is needed from the origin, CloudFront requests it from the S3 bucket.
4. Origin Access Control (OAC) signs requests from CloudFront to S3.
5. The S3 bucket policy permits the configured CloudFront distribution to read website objects.

## AWS Services Used

| Service | Purpose |
|---|---|
| Amazon S3 | Stores the website's static files |
| Amazon CloudFront | Distributes website content and supports HTTPS |
| Origin Access Control (OAC) | Secures access from CloudFront to the S3 origin |
| AWS IAM and S3 bucket policy | Control permissions for accessing website objects |

## Implementation Steps

1. Created an S3 bucket and uploaded the portfolio website's `index.html` file.
2. Created a CloudFront distribution using the regular S3 bucket origin.
3. Configured Origin Access Control (OAC) for requests to S3.
4. Configured the S3 bucket policy to allow the CloudFront distribution to read website objects.
5. Set `index.html` as the CloudFront default root object.
6. Configured the viewer protocol policy to redirect HTTP requests to HTTPS.
7. Tested the website through the CloudFront distribution domain.

## Security Considerations

- Keep S3 Block Public Access enabled.
- Use Origin Access Control to restrict origin access to CloudFront.
- Grant only the S3 permissions required to read website objects.
- Avoid granting anonymous users public read access to the bucket.
- Do not commit AWS access keys, secret keys, passwords or other credentials to GitHub.

## Testing

- Verified that the website loads through the CloudFront distribution.
- Verified that the CloudFront URL uses HTTPS.
- Check that all four S3 Block Public Access settings remain enabled.
- Check that the bucket policy restricts read access to the intended CloudFront distribution.
- Test the direct S3 object URL in a signed-out browser session and confirm that anonymous access is denied.

Record the actual results of the security checks before marking them complete.

## Repository Structure

```text
aws-static-website-hosting/
├── README.md
├── website/
│   └── index.html
└── docs/
    └── architecture/
        ├── README.md
        ├── architecture.drawio
        └── architecture.png
```

## Cost Management

- Review AWS billing and available credits regularly.
- Check CloudFront and S3 usage to avoid unexpected charges.
- Remove unused resources when they are no longer needed.
- Before deleting resources, review the impact on the live website.

## What I Learned

- Hosting static website files with Amazon S3.
- Using CloudFront to deliver content over HTTPS.
- Configuring Origin Access Control for a private S3 origin.
- Applying least-privilege access through an S3 bucket policy.
- Testing and documenting an AWS deployment.

## Future Improvements

- Configure a custom domain and HTTPS certificate, if needed.
- Add automated deployment using GitHub Actions.
- Improve website monitoring and operational documentation.

---

**Project status:** Deployed and accessible through Amazon CloudFront. Security checks should be documented according to their verified results.
