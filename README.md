# Never-Miss-a-Lead — standalone product site

Live at **https://nevermiss.northbeambuilds.com** — the standalone website for
Never-Miss-a-Lead, a product by [Northbeam Builds](https://northbeambuilds.com).
northbeambuilds.com is the company site; each product gets its own site like
this one (free on a subdomain first; graduates to its own .com when it proves out).

**This repo is the reference implementation.** Every future product site is a
copy-paste of this pattern — see "Replication pattern" below.

## Layout

```
index.html            Home: hero, problem + honest stats, 3 steps, sample, fit, price summary, demos, FAQ teaser, CTA
demos/index.html      Concept demos (links to northbeambuilds.com/demos/<name>/, labeled "Concept demo — not a client")
pricing/index.html    Full price box with totals, guarantee, buy button
faq/index.html        Full FAQ accordion
contact/index.html    northbeambuilds@gmail.com card + buy button
assets/site.css       Shared design system (extracted from the verified company-site tool page)
assets/site.js        Header nav toggle + FAQ chat widget (carried over from the company site)
CNAME                 nevermiss.northbeambuilds.com
robots.txt, sitemap.xml
```

Rules that hold on every page:

- **Every buy button points at the product's LIVE Stripe payment link** — no
  reserve/mailto CTAs for buying. For this product:
  `https://buy.stripe.com/eVqdR98AY8BKgMWcfAgYU00`
  (label: "Get Never-Miss-a-Lead — $199 setup + $99/mo")
- The product is the hero brand (header wordmark, titles). The footer carries
  "**<Product> — by Northbeam Builds**" + a link to https://northbeambuilds.com.
- Copy comes only from the verified product page on the company site
  (`northbeambuilds.com/tools/<tool>/`). Prices, stats, and claims must match
  it exactly — stats keep their source + caveat text.
- Demos are always labeled "Concept demo — not a client".
- The FAQ chat widget (assets/site.js) ships on every page.

## Replication pattern — spin up the next product site

Worked example values below are this product's; swap in the new product's.

### 1. DNS (Porkbun, domain northbeambuilds.com)

Add one record (TTL 600):

| Type  | Host       | Answer              |
| ----- | ---------- | ------------------- |
| CNAME | `nevermiss` | `timothyjt.github.io` |

(Host = the subdomain label, e.g. `nevermiss` for nevermiss.northbeambuilds.com.)

### 2. Repo

Create a public repo `TimothyjT/<product>-site` (GitHub API:
`POST /user/repos {"name":"<product>-site","private":false}`), then copy this
repo's layout, swapping in the new product's verified copy, subdomain, and
Stripe payment link. Commit a `CNAME` file containing the full subdomain
(e.g. `nevermiss.northbeambuilds.com`).

### 3. Enable GitHub Pages + custom domain (API)

```bash
# enable Pages from branch main, root path
curl -X POST -H "Authorization: Bearer $GH_TOKEN" \
  https://api.github.com/repos/TimothyjT/<product>-site/pages \
  -d '{"source":{"branch":"main","path":"/"}}'

# set the custom domain
curl -X PUT -H "Authorization: Bearer $GH_TOKEN" \
  https://api.github.com/repos/TimothyjT/<product>-site/pages \
  -d '{"cname":"<sub>.northbeambuilds.com"}'

# trigger a build
curl -X POST -H "Authorization: Bearer $GH_TOKEN" \
  https://api.github.com/repos/TimothyjT/<product>-site/pages/builds
```

### 4. HTTPS

GitHub provisions a Let's Encrypt certificate automatically once DNS resolves.
Check state via the API — **never trust a local `curl` from the build VM (its
proxy intercepts TLS and lies)**:

```bash
curl -H "Authorization: Bearer $GH_TOKEN" \
  https://api.github.com/repos/TimothyjT/<product>-site/pages
# look for: "status":"built", "https_certificate":{"state":"approved"}
```

When `https_certificate.state` is `approved`, enforce HTTPS:

```bash
curl -X PUT -H "Authorization: Bearer $GH_TOKEN" \
  https://api.github.com/repos/TimothyjT/<product>-site/pages \
  -d '{"https_enforced":true}'
```

If the certificate is still provisioning (states like `new`,
`authorization_pending`), wait and re-check — do not force anything.

### 5. Verify

- `GET /repos/TimothyjT/<product>-site/pages` → status `built`, cname correct,
  `https_enforced: true` once the cert is approved.
- Every page loads at `https://<sub>.northbeambuilds.com/<page>/`, contains the
  product's price figures, the footer "— by Northbeam Builds" line, and the
  chat widget.
- Every buy button on every page resolves to the product's live Stripe link.

## Product registry

| Product | Subdomain | Stripe payment link |
| ------- | --------- | ------------------- |
| Never-Miss-a-Lead | nevermiss.northbeambuilds.com | https://buy.stripe.com/eVqdR98AY8BKgMWcfAgYU00 |

(Add a row for each new product site as it launches.)
