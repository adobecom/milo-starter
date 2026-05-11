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

### Case 1 — project owns the root of the domain

| What | From | To |
|------|------|----|
| Milo libs | `your-domain.com/libs/*` | `main--milo--adobecom.aem.live/libs/*` |
| Project | `your-domain.com/*` | `main--{repo}--adobecom.aem.live/*` |

Use `aem.page` instead of `aem.live` for preview/stage. No path rewriting needed.

The starter resolves Milo libs as `/libs` on production, relying on the CDN rule above to proxy the requests. If your CDN can't route `/libs/*` to a different origin, hardcode the absolute URL in both of these places:

- `scripts/scripts.js`: `const LIBS = 'https://www.adobe.com/libs'`
- `head.html` inline script: change `return '/libs'` → `return 'https://www.adobe.com/libs'`

### Case 2 — project lives under a sub-path

Example: site is at `your-domain.com/acme/blog`.

| What | From | To | Strip prefix? |
|------|------|----|---------------|
| Milo libs | `your-domain.com/acme/blog/libs/*` | `main--milo--adobecom.aem.live/libs/*` | Yes |
| Project code | `your-domain.com/acme/blog/{scripts,styles,blocks,...}/*` | `main--{repo}--adobecom.aem.live/{scripts,...}/*` | Yes |
| Project content | `your-domain.com/acme/blog/*` | `main--{repo}--adobecom.aem.live/acme/blog/*` | **No** |

Use `aem.page` instead of `aem.live` for preview/stage.

**Content must keep the full path.** EDS derives `.plain.html` URLs from the canonical path — stripping the prefix from content requests breaks page rendering.

**SharePoint:** author all documents under the same sub-path they'll appear at publicly (`/acme/blog/…`), not at the root. This keeps AEM-generated links correct.

Example CloudFront viewer-request function:

```js
const PREFIX = '/acme/blog';
const CODE = ['/libs/', '/scripts/', '/styles/', '/blocks/', '/tools/', '/img/'];

function handler(event) {
  const req = event.request;
  const path = req.uri.slice(PREFIX.length);
  if (CODE.some(p => path.startsWith(p))) {
    req.uri = path || '/';
  }
  return req;
}
```

**Debugging:**
- JS/CSS wrong `Content-Type` → prefix not being stripped; CDN is serving a 404 error page as the file.
- `.plain.html` returns 404 → prefix is being stripped from content requests; restrict the rule to code paths only.
- Links navigate to wrong URL → documents in SharePoint aren't authored with the full public path (e.g. `/acme/blog/page`, not `/page`).

## Security
1. Create a Service Now ID for your project via [Service Registry Portal](https://adobe.service-now.com/service_registry_portal.do#/search)
2. Update the `.kodiak/config.yaml` file to make sure valid team members are assigned security vulnerability Jira tickets.
