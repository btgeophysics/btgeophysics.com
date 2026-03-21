# Black Tusk Geophysics Website

Static website for [btgeophysics.com](https://btgeophysics.com), built with [Hugo](https://gohugo.io/) and deployed to GitHub Pages.

## Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) v0.158.0+
- Git

Install Hugo on macOS:

```bash
brew install hugo
```

## Local Development

Preview the site locally with live reload:

```bash
hugo server
```

Open [http://localhost:1313/](http://localhost:1313/) in your browser.

Build the production site:

```bash
hugo --minify
```

Output goes to `public/`.

## Project Structure

```
├── hugo.toml                  # Site configuration (title, contact info, API keys)
├── content/
│   ├── _index.md              # Homepage "About" section text
│   └── services/              # Service descriptions (one .md file per service)
│       ├── mpv-data-acquisition.md
│       ├── data-processing.md
│       ├── borehole-classification.md
│       └── marine-classification.md
├── data/
│   └── projects.json          # Project map marker data (location, description, image)
├── layouts/
│   ├── _default/baseof.html   # Base HTML template (head, scripts, body wrapper)
│   ├── index.html             # Homepage template (assembles all sections)
│   └── partials/              # Reusable template fragments
│       ├── ultratem.html          # UltraTEM product section
│       ├── service-modal.html     # Service detail modal (reused per service)
│       ├── project-map-modal.html # Google Maps modal container
│       └── project-map-scripts.html # Google Maps JS + project data injection
├── assets/
│   └── scss/
│       └── main.scss          # All custom styles (Bootstrap 5 overrides)
├── static/
│   ├── img/                   # All images
│   │   └── casestudies/       # Project location photos
│   ├── fonts/                 # Web fonts
│   ├── UltraTEMBasicDescriptionv2.pdf
│   ├── favicon.ico
│   └── CNAME                  # Custom domain for GitHub Pages
└── .github/
    └── workflows/
        └── deploy.yml         # GitHub Actions: build Hugo + deploy to Pages
```

## Editing Content

### Update the About section

Edit `content/_index.md`. This is standard Markdown.

### Update a service description

Edit the corresponding file in `content/services/`. Each file has YAML frontmatter followed by Markdown content:

```yaml
---
title: "MPV Data acquisition"          # Card label on homepage
modalTitle: "Data acquisition with the MPV sensor"  # Modal heading
image: "/img/acrossslopedynamic.JPG"   # Thumbnail image
weight: 1                              # Display order (1 = first)
modalId: "services_MPV"                # HTML anchor ID
---

Your content here in Markdown...
```

### Add a new service

1. Create a new `.md` file in `content/services/` (copy an existing one as a template)
2. Set a unique `modalId` and appropriate `weight` for ordering
3. Add the service image to `static/img/`
4. Commit and push

### Update contact information

Edit the `[params]` section in `hugo.toml`:

```toml
[params]
  email = "info@btgeophysics.com"
  phone = "604 428 3382"
  address = "#401-1755 Broadway W"
  city = "Vancouver, BC V6J 4S5"
  country = "Canada"
```

### Add a project to the map

Edit `data/projects.json`. Add a new entry to the JSON array:

```json
{
  "headline": "Project Name, State",
  "startDate": "2024",
  "endDate": "2025",
  "text": "Description of work performed.",
  "media": "/img/casestudies/your-image.jpg",
  "lat": 40.7128,
  "lon": -74.0060
}
```

Add the project image to `static/img/casestudies/`.

### Update styles

Edit `assets/scss/main.scss`. Hugo compiles SCSS automatically on build.

## Deployment

Deployment is fully automatic. Push to the `main` branch and GitHub Actions will:

1. Install Hugo Extended
2. Run `hugo --minify`
3. Deploy the `public/` directory to GitHub Pages

The workflow is defined in `.github/workflows/deploy.yml`.

## Initial Setup

### 1. Create the GitHub repository

```bash
gh repo create btgeophysics/btgeophysics.com --public --source=. --remote=origin --push
```

### 2. Enable GitHub Pages

1. Go to the repository on GitHub
2. Navigate to **Settings → Pages**
3. Under "Build and deployment", set Source to **GitHub Actions**
4. The first deployment will trigger automatically after the push

### 3. Verify deployment

After the workflow completes (1-2 minutes), the site will be available at:

```
https://btgeophysics.github.io/btgeophysics.com/
```

Once the custom domain is configured (next section), it will be at `https://btgeophysics.com`.

## DNS and Domain Setup (AWS Route 53)

The domain `btgeophysics.com` is managed in AWS Route 53. Follow these steps to point it to GitHub Pages.

### Step 1: Configure the custom domain in GitHub

1. Go to repository **Settings → Pages**
2. Under "Custom domain", enter: `btgeophysics.com`
3. Click **Save**
4. GitHub will show a DNS check — it will fail until you update Route 53 (next step)

### Step 2: Update DNS records in Route 53

1. Log in to the [AWS Console](https://console.aws.amazon.com/) and go to **Route 53**
2. Select the hosted zone for `btgeophysics.com`
3. **Delete or update** any existing A/AAAA records pointing to Azure

#### Create A records (apex domain)

Create an A record set for the apex domain (`btgeophysics.com`):

| Field       | Value                                                         |
|-------------|---------------------------------------------------------------|
| Name        | `btgeophysics.com`                                            |
| Type        | A                                                             |
| TTL         | 300                                                           |
| Value       | `185.199.108.153` `185.199.109.153` `185.199.110.153` `185.199.111.153` |

All four IP addresses go in a single record (one per line in the Route 53 console).

#### Create AAAA records (IPv6)

Create an AAAA record set for the apex domain:

| Field       | Value                                                         |
|-------------|---------------------------------------------------------------|
| Name        | `btgeophysics.com`                                            |
| Type        | AAAA                                                          |
| TTL         | 300                                                           |
| Value       | `2606:50c0:8000::153` `2606:50c0:8001::153` `2606:50c0:8002::153` `2606:50c0:8003::153` |

#### Create CNAME record (www subdomain)

| Field       | Value                          |
|-------------|--------------------------------|
| Name        | `www.btgeophysics.com`         |
| Type        | CNAME                          |
| TTL         | 300                            |
| Value       | `btgeophysics.github.io`       |

> **Important:** Do not modify DNS records for `btfield.btgeophysics.com` or any other subdomains. Those are separate services and should remain as-is.

### Step 3: Verify DNS propagation

After updating Route 53, verify the changes are propagating:

```bash
# Check A records
dig btgeophysics.com +short
# Should return: 185.199.108.153, 185.199.109.153, etc.

# Check CNAME
dig www.btgeophysics.com +short
# Should return: btgeophysics.github.io
```

DNS propagation typically takes 5-60 minutes with Route 53.

### Step 4: Enable HTTPS

1. Go back to repository **Settings → Pages**
2. The DNS check should now pass
3. Check the box **"Enforce HTTPS"**
4. GitHub will automatically provision and renew a Let's Encrypt TLS certificate

> This replaces the manually managed Let's Encrypt certificate from the Azure setup. No further certificate management is required.

### Step 5: Verify

- `https://btgeophysics.com` — loads the site with a valid certificate
- `http://btgeophysics.com` — redirects to HTTPS
- `https://www.btgeophysics.com` — redirects to the apex domain

### Step 6: Decommission Azure

After confirming the site works on GitHub Pages for a few days:

1. Shut down the Azure App Service / static site resource
2. Remove any Azure-related DNS records from Route 53 that are no longer needed
3. Cancel associated Azure billing

## External Dependencies

| Service | Purpose | Config location |
|---------|---------|-----------------|
| [Font Awesome Kit](https://fontawesome.com/) | Icons | `hugo.toml` → `fontAwesomeKit` |
| [Google Maps API](https://console.cloud.google.com/) | Project map | `hugo.toml` → `googleMapsKey` |
| [Google Fonts](https://fonts.google.com/) | Lora, Montserrat | `layouts/_default/baseof.html` |
| [Knight Lab Timeline](https://timeline.knightlab.com/) | Project timeline | `hugo.toml` → `timelineURL` |
| [Bootstrap 5 CDN](https://getbootstrap.com/) | CSS framework | `layouts/_default/baseof.html` |

### Google Maps API key

The API key in `hugo.toml` should be restricted in the [Google Cloud Console](https://console.cloud.google.com/apis/credentials) to only allow requests from `btgeophysics.com`.
