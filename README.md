# Google Maps-Based Travel Planner Chrome Extension

This Chrome extension allows users to save places while browsing travel-related websites and organize them into a travel itinerary.

The Google Maps interface is hosted as a static website on AWS.

## Architecture

The static website is hosted using the following AWS services:

- **Amazon S3** — Stores and serves the static website files.
- **Amazon CloudFront** — Distributes the website through a CDN and provides HTTPS access.
- **AWS Certificate Manager (ACM)** — Issues and manages the TLS certificate for the custom domain.
- **Amazon Route 53** — Manages DNS records and routes the custom domain to CloudFront.

The domain is registered through an external domain registrar, while its authoritative DNS is managed by Route 53.

### Request Flow

```text
Browser
   |
   | DNS lookup
   v
Route 53
   |
   | A / AAAA Alias
   v
CloudFront
   |
   | HTTPS / CDN
   | TLS certificate managed by ACM
   v
Amazon S3
   |
   | Static files
   v
HTML / CSS / JavaScript
```

When a user accesses the website:

1. The browser resolves the custom domain through DNS.
2. The domain's authoritative name servers point to the Route 53 Hosted Zone.
3. Route 53 resolves the domain to the CloudFront distribution using an Alias record.
4. CloudFront establishes an HTTPS connection using the TLS certificate managed by ACM.
5. CloudFront serves cached content when available; otherwise, it retrieves the static files from S3.

### DNS and HTTPS

A **Route 53 Hosted Zone** stores the DNS records for the custom domain. When the Hosted Zone is created, Route 53 assigns authoritative name servers to it. The domain registrar delegates DNS management to these Route 53 name servers.

The custom domain is then mapped to the CloudFront distribution using a Route 53 **Alias record**.

For HTTPS, a TLS certificate for the custom domain is issued by **AWS Certificate Manager (ACM)**. Domain ownership is verified through DNS validation by adding the CNAME record provided by ACM to the Route 53 Hosted Zone.

The issued certificate is attached to the CloudFront distribution. When a browser connects to the custom domain, CloudFront presents this certificate during the TLS handshake, allowing the browser to verify the server's identity and establish an encrypted connection.

## Setup

Open the Chrome Extensions page and load the `maps` directory as an unpacked extension:

1. Open **Extensions**
2. Select **Manage Extensions**
3. Enable **Developer mode**
4. Click **Load unpacked**
5. Select the `maps` directory

## Demo

See the project demo in the Google Slides presentation:

https://docs.google.com/presentation/d/1uVr59mahwf3xTrPJPh-eGZ-YFghic4-fhgrPOhSTplM/edit?slide=id.g3c4642b03e5_0_45#slide=id.g3c4642b03e5_0_45

## Demo
https://docs.google.com/presentation/d/1uVr59mahwf3xTrPJPh-eGZ-YFghic4-fhgrPOhSTplM/edit?slide=id.g3c4642b03e5_0_45#slide=id.g3c4642b03e5_0_45
