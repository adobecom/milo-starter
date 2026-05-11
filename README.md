# Milo goes to college
Use this project template to create a Milo site on DA! [milo-college](https://github.com/adobecom/milo-college) is the Sharepoint equivalent to this project.

## Steps

1. Copy existing [`college`](https://adobe.sharepoint.com/:f:/r/sites/adobecom/Shared%20Documents/demos/college) content folder to your sharepoint and give helix@adobe.com View access
2. Click "[Use this template](https://github.com/adobecom/milo-college/generate)" Github button on this project.
3. Install the [AEM Code Sync Bot](https://github.com/apps/aem-code-sync)

From your newly created project

1. Install the [Helix Bot](https://github.com/apps/helix-bot/installations/new).
2. Change the fstab.yaml file to point to your content.
3. Add the project to the [Helix Sidekick](https://github.com/adobe/helix-sidekick).
4. Start creating your content.

## Developing
1. Install the [Helix CLI](https://github.com/adobe/helix-cli): `sudo npm install -g @adobe/aem-cli`
1. Run `aem up` this repo's folder. (opens your browser at `http://localhost:3000`)
1. Open this repo's folder in your favorite editor and start coding.

## Testing
```sh
npm run test
```
or:
```sh
npm run test:watch
```
This will give you several options to debug tests. Note: coverage may not be accurate.

## CDN Configuration

### Simple case — project owns the root path

Route `your-domain.com/*` to `main--{repo}--adobecom.aem.live/*` (prod) or `main--{repo}--adobecom.aem.page/*` (preview/stage). No path rewriting needed. Use `aem.page` for non-prod environments so you can test content before publishing.

The starter maps `LIBS = '/libs'` in `scripts/scripts.js`. Any request to `/libs/*` must be proxied to the Milo origin (`main--milo--adobecom.aem.live`). If your CDN cannot do this, change the constant to the absolute URL instead and update the matching inline snippet in `head.html`:

```js
const LIBS = 'https://milo.adobe.com/libs';
```

### Sub-path case — project owns `/some/path` on a shared domain

This is the harder case (e.g. `your-domain.com/foo/docs` on a domain your team does not fully control). Follow these rules exactly to avoid the problems documented below.

#### Content must mirror the public path in SharePoint

All authored content must live under the same path in SharePoint as it will appear publicly. If the site lives at `/foo/docs`, every document must be at `/foo/docs/…` in the SharePoint folder — not at the root. This keeps AEM-generated links correct and prevents relative-link breakage.

#### CDN routing

Route `your-domain.com/foo/docs/*` → `main--{repo}--adobecom.aem.live/foo/docs/*`. **Do not strip the prefix from the content path.** EDS serves `.plain.html` by appending it to the canonical URL path; if the CDN strips the prefix the request arrives with the wrong path and returns a 404.

#### Milo libs and project code do need the prefix stripped

Project code (`/scripts/`, `/styles/`, `/blocks/`, etc.) and Milo libs (`/libs/`) live at the repo root, not under `/foo/docs`. When your CDN proxies `your-domain.com/foo/docs/scripts/foo.js` it must strip `/foo/docs` before forwarding to the AEM code origin. Apply stripping **only** for `.js` and `.css` requests (or the known code paths listed below) — never for HTML or plain.html.

Example CloudFront viewer-request function:

```js
const PREFIX = '/foo/docs';

function handler(event) {
  const request = event.request;
  if (
    request.uri.startsWith(PREFIX + '/libs/') // milo libs
    || request.uri.endsWith('.js')
    || request.uri.endsWith('.css')
  ) {
    request.uri = request.uri.slice(PREFIX.length) || '/';
  }
  return request;
}
```

Known code-bus paths that need the prefix stripped (for an explicit allowlist approach):

```
/scripts/
/styles/
/blocks/
/tools/
/img/
/404.html
```

#### Diagnosing CDN issues

- **Wrong `Content-Type` on a JS/CSS file** — the CDN is returning a 404 error page with a 200 status. The path being requested doesn't exist on the origin; check whether the prefix stripping is missing or mis-targeted.
- **`.plain.html` returns wrong content or 404** — the prefix is being stripped from content requests. Stripping should apply to code/libs only.
- **Links navigate to the wrong URL** — authored links in Word use the wrong base path. All links must be authored with the full public path (e.g. `/foo/docs/getting-started`, not `/getting-started`). gnav and footer docs are subject to the same rule.

#### Sidekick / preview workflow

After editing a Word doc in SharePoint, refresh the doc in SharePoint before clicking Preview in Sidekick to ensure SharePoint has flushed its internal cache before EDS fetches the content.

## Security
1. Create a Service Now ID for your project via [Service Registry Portal](https://adobe.service-now.com/service_registry_portal.do#/search)
2. Update the `.kodiak/config.yaml` file to make sure valid team members are assigned security vulnerability Jira tickets.
