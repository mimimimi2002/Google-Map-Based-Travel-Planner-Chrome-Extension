## How It Works

The project consists of three main components:

1. **Chrome Extension** — Collects place names while the user browses travel-related websites.
2. **Map Page (`map/index.html`)** — Displays the collected places on Google Maps.
3. **Shiori Page (`shiori/index.html`)** — Generates a travel itinerary ("Shiori") from the collected places.

The Map and Shiori pages are static web applications hosted in the same Amazon S3 bucket.

```text
S3 Bucket: mikimiki.site
│
├── map/
│   └── index.html       # Google Maps page
│
└── shiori/
    ├── index.html       # Travel itinerary page
    ├── static/
    ├── fonts/
    └── ...
```

### Saving Places

While browsing a website, the user can select a place name, such as `Tokyo Tower`, and add it through the Chrome extension.

The selected place is stored in `chrome.storage.local`.

The extension also sends the place name to the Map page using `window.postMessage()`, allowing the location to be displayed on Google Maps.

```text
Travel Website
      |
      | Select a place name
      | e.g. "Tokyo Tower"
      v
Chrome Extension
(content.js)
      |
      +------> chrome.storage.local
      |        Save placeNames
      |
      +------> postMessage("ADD_PLACE")
                     |
                     v
              map/index.html
                     |
                     v
                Google Maps
                     |
                     v
               Show location
```

The Map page is loaded inside an iframe by the Chrome extension.

When the map is opened, previously saved places can also be retrieved from `chrome.storage.local` and sent to the Map page.

### Creating a Travel Itinerary

After collecting multiple places, the user can create a travel itinerary ("Shiori").

The extension retrieves the saved place names from `chrome.storage.local` and passes them to the Shiori page.

```text
chrome.storage.local

[
  "Tokyo Tower",
  "Shibuya",
  "Asakusa"
]

        |
        | Create Shiori
        v

shiori/index.html
        |
        v
Travel Itinerary
```

This creates the following overall workflow:

```text
Browse Travel Websites
        |
        | Select interesting places
        v
Chrome Extension
        |
        | Save
        v
chrome.storage.local
        |
        +----------------------+
        |                      |
        v                      v
 map/index.html         shiori/index.html
        |                      |
        v                      v
  Google Maps          Travel Itinerary
```

## AWS Architecture

Both the Map page and the Shiori page are hosted as static web applications on Amazon S3.

```text
                        mikimiki.site
                              |
                         Route 53
                              |
                              v
                         CloudFront
                         HTTPS / CDN
                              |
                              v
                     Amazon S3 Bucket
                              |
                 +------------+------------+
                 |                         |
                 v                         v
          map/index.html            shiori/index.html
                 |                         |
                 v                         v
           Google Maps              Travel Itinerary
```

The AWS services have the following roles:

- **Amazon S3** — Stores the static Map and Shiori web applications.
- **Amazon CloudFront** — Distributes the static content and provides an HTTPS endpoint.
- **AWS Certificate Manager (ACM)** — Issues and manages the TLS certificate for `mikimiki.site`.
- **Amazon Route 53** — Manages DNS and maps `mikimiki.site` to the CloudFront distribution.

### DNS and HTTPS

The domain is registered through an external domain registrar, while its authoritative DNS is managed by Route 53.

A Route 53 Hosted Zone stores the DNS records for `mikimiki.site`. The domain registrar delegates DNS management to the Route 53 authoritative name servers.

An Alias record maps:

```text
mikimiki.site
      |
      v
CloudFront Distribution
```

For HTTPS, AWS Certificate Manager issues a TLS certificate for `mikimiki.site`. Domain ownership is verified using DNS validation, and the certificate is attached to CloudFront.

When a browser accesses the application:

```text
Browser
   |
   | https://mikimiki.site/...
   v
Route 53
   |
   | DNS resolution
   v
CloudFront
   |
   | HTTPS
   | ACM TLS Certificate
   v
Amazon S3
   |
   +----> map/index.html
   |
   └----> shiori/index.html
```
