# ISWS website

Source of the website of the **International Semantics and Knowledge Summer School (ISWS)**, published at **https://semanticwebschool.org**.

The site is built with [Jekyll](https://jekyllrb.com/) and the premium Jekyll theme *Snowlake* by Tortoiz Themes, and it is deployed to GitHub Pages by a GitHub Actions workflow. Every edition of the school has its own folder in this repository and its own path on the website.

| Edition | Folder | URL |
|---|---|---|
| 2027 | `2027/` | https://semanticwebschool.org/2027/ |
| 2028 (future) | `2028/` | https://semanticwebschool.org/2028/ |

Older editions (2019–2026) are **not** part of this repository. They run on their own subdomains, e.g. `2024.semanticwebschool.org`, and are hosted elsewhere (see [Domain and DNS](#domain-and-dns)).

> **Theme licence.** Snowlake is a paid theme. Its developer confirmed that it may be used in this public GitHub Pages repository. Do not reuse the theme files for other websites without checking the licence.


## Repository structure

```
isws-dev.github.io/
├── .github/workflows/deploy.yml   builds every year folder and deploys to GitHub Pages
├── index.html                     redirects semanticwebschool.org/ to the current edition
├── README.md                      this file
└── 2027/                          complete Jekyll site for ISWS 2027
    ├── _config.yml                site URL and base path (/2027)
    ├── _data/general_settings.yml site title, description, logos, favicon, footer text, social links
    ├── _data/navigation.yml       top navigation menu
    ├── index.html                 start page (save-the-date, important dates, directors)
    ├── keynotes.html              keynote speakers page (/2027/keynotes)
    ├── previous-editions.html     links to earlier editions (/2027/previous-editions)
    ├── _includes/layouts/header/isws-page-header.html   castle-image header for subpages
    ├── assets/images/             images, logos, director and keynote speaker photos
    └── ...                        theme files and theme demo pages
```

Each year folder is an independent Jekyll site. Changing one year never affects another.


## Deployment

Every push to the `main` branch starts the workflow **Build and deploy site** (`.github/workflows/deploy.yml`). It

1. builds every folder whose name is a year (`2027/`, `2028/`, …) with Jekyll into `_site/<year>/`,
2. copies the root `index.html` (the redirect) into `_site/`,
3. publishes the result to GitHub Pages.

The run takes about 2–4 minutes. Progress and errors are visible in the **Actions** tab of the repository. In the repository settings, **Settings → Pages → Source** must stay on **GitHub Actions**.

Nothing has to be built locally before pushing. Local builds are only for previewing changes.


## Working locally (macOS)

### One-time setup

macOS ships an old Ruby that is too old for Jekyll, so install a current Ruby with Homebrew:

```bash
brew install ruby
echo 'export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/$(ls /opt/homebrew/lib/ruby/gems | tail -1)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
ruby -v
gem install bundler
```

`ruby -v` must show version 3.x or 4.x. On an Intel Mac, replace `/opt/homebrew` with `/usr/local`. The site is currently built with Ruby 4.0, both locally and in the workflow (`ruby-version` in `deploy.yml`).

### Preview a year

```bash
cd 2027
bundle install
bundle exec jekyll serve
```

Then open http://127.0.0.1:4000/2027/. Jekyll rebuilds automatically when files change; reload the browser to see the result. Stop the server with `Ctrl+C`.

If Jekyll stops with `cannot load such file -- <name>`, a library is missing that newer Ruby versions no longer include by default. Add `gem "<name>"` to the `Gemfile` of that year and run `bundle install` again. `csv`, `base64`, `logger` and `webrick` were already added this way.


## Editing the 2027 site

### Common changes

| What | Where |
|---|---|
| Texts, dates and sections of the start page | `2027/index.html` |
| Site title, description, footer text | `2027/_data/general_settings.yml` |
| Social links in the footer (currently LinkedIn only) | `social_accounts` in `2027/_data/general_settings.yml` |
| Navigation menu | `2027/_data/navigation.yml` |
| Logos, favicon | `2027/assets/images/` and the paths in `general_settings.yml` |
| Header background image | `2027/assets/images/bertinoro-castle.webp` (replace the file, keep the name) |
| Custom styles of the start page | `<style>` block at the end of `2027/index.html` |
| Keynote speakers (names, bios, talk titles) | `2027/keynotes.html`; photos in `2027/assets/images/keynotes/` (square WebP, 480 × 480) |
| List of previous editions | `2027/previous-editions.html` (one `<li>` per edition, newest first) |

### `/2027/` rule

Because each edition lives under its own path, **every internal link and image path must start with the year**:

```html
<img src="/2027/assets/images/example.webp">
<a href="/2027/programme">Programme</a>
```

A path that starts with `/assets/...` or `/programme` points to the domain root and results in a broken image or a 404 page. The same applies to URLs in YAML files (`navigation.yml`, `general_settings.yml`, front matter of pages).

The only exceptions are `favicon`, `meta_og_image` and `meta_twitter_image` in `general_settings.yml`. The theme adds the base path to these itself, so they are written without the year (`/assets/images/...`).

### Adding a page

1. Create a file in `2027/`, e.g. `2027/programme.html`, with the following frame (front matter, navigation, footer):
   ```liquid
   ---
   layout: default
   title: "Programme – ISWS 2027"
   custom_font: font3
   ---

   {% include layouts/nav/nav-10.html default_logo="black" %}

   <div class="wrapper light-wrapper">
     <div class="container inner">
       <h1>Programme</h1>
       <!-- content -->
     </div>
   </div>

   {% include layouts/footer/footer-3.html %}

   <style>
     .navbar-nav .nav-link, .dropdown-item { font-family: 'Jost', sans-serif !important; }
   </style>
   ```
   `nav-10` is a navigation with dark text and the navy logo, suitable for pages without a dark header image. The start page uses `nav-1` instead, which is transparent with white text and only works on top of a dark header. The `<style>` line sets the menu font to Jost, as on the start page.

   For a page with the castle image as header (as on the keynote speakers and previous editions pages), replace the `nav-10` line with these two lines and drop the `<h1>`:
   ```liquid
   {% include layouts/nav/nav-1.html alignment_class="ms-auto" %}
   {% include layouts/header/isws-page-header.html title="Programme" lead="6 – 12 June 2027 · Bertinoro, Italy" %}
   ```

   Avoid the theme's `layout: page`: it uses a different navigation and a footer (`footer-1`) that still contains the theme's placeholder links.
2. Add it to the menu in `2027/_data/navigation.yml`, above the commented-out demo menu:
   ```yaml
   main_menu:
     - text: "Home"
       url: "/2027/"
       menu_type: "simple"
     - text: "Programme"
       url: "/2027/programme"
       menu_type: "simple"
   ```
   The commented-out block below it is the theme's original demo menu, kept for reference. It is not used.
3. Preview locally, then commit and push.

The theme's demo pages (`home-pages/`, `elements/`, `features/`, `about/`, `services/` …) are still in the folder and are a good source of ready-made sections. Open them locally under `http://127.0.0.1:4000/2027/elements/...`, copy the markup you need, and adapt the paths to the `/2027/` rule. Unused demo pages can be deleted at any time.

### Logos

The ISWS logo exists as SVG in four files in `2027/assets/images/`:

| File | Use |
|---|---|
| `isws-logo.svg` | navy letters, for light backgrounds |
| `isws-logo-light.svg` | white letters, for dark backgrounds (navigation, footer) |
| `isws-logo@2x.svg`, `isws-logo-light@2x.svg` | identical drawings, required by the theme for high-resolution screens |

The display size is defined by the `width` and `height` attributes at the start of each SVG file (`110` × `60` for the normal files, `220` × `120` for the `@2x` files). Without these attributes the theme shows the logo at full page width. To change the size, change both numbers in all four files in the same proportion, keeping the `@2x` values twice as large.

### Known theme quirks

The theme was written for sites at the domain root, not under a path like `/2027/`. When the theme was set up, its template variable `{{relative_url}}` (which is always empty) was replaced by `{{ site.baseurl }}`, and the year was added to all asset paths. When copying markup from the original theme download, check the paths again.

The files `github.toml`, `netlify.toml` and `cloudflare.toml` in `2027/` belong to other hosting providers and are ignored by GitHub Pages.


## Starting a new edition

1. Copy the folder of the previous year, e.g. `2027/` → `2028/`.
2. In the new folder, replace the year in all paths: in VS Code, use **Find and Replace in Files** (`Cmd+Shift+H`) with search `/2027/`, replace `/2028/`, and *files to include* `2028`. Check the preview before replacing.
3. In `2028/_config.yml`, set `baseurl: "/2028"`.
4. Update the content: texts, dates, directors, images, title and footer in `general_settings.yml`.
5. In the root `index.html`, change both `/2027/` references to `/2028/`, so the domain points to the new edition.
6. Commit and push.

The previous year stays online unchanged under its own path.


## Domain and DNS

The domain `semanticwebschool.org` is registered with GoDaddy. It is connected to this repository under **Settings → Pages → Custom domain**, with **Enforce HTTPS** enabled. No `CNAME` file is needed because the site is deployed via GitHub Actions.

Relevant DNS records at GoDaddy:

| Type | Name | Value | Purpose |
|---|---|---|---|
| A | `@` | `185.199.108.153` | GitHub Pages |
| A | `@` | `185.199.109.153` | GitHub Pages |
| A | `@` | `185.199.110.153` | GitHub Pages |
| A | `@` | `185.199.111.153` | GitHub Pages |
| CNAME | `www` | `isws-dev.github.io` | GitHub Pages |
| A | `2019` … `2023` | `150.146.207.67` | old editions on an external server |
| CNAME | `2024` | `semanticwebschool.github.io` | ISWS 2024 site |
| CNAME | `2025`, `2026` | `semanticwebschool2025.github.io` | ISWS 2025/2026 site |

Do not change or delete the records of the old editions, otherwise those sites go offline. The NS, SOA and `_domainconnect` records are managed by GoDaddy.


## Accounts and access

The repository belongs to the GitHub user account **isws-dev**. Pushing requires access to this account or collaborator rights on the repository. DNS changes require access to the GoDaddy account that holds `semanticwebschool.org`.
