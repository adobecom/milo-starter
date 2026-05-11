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

No path rewriting needed. Use `aem.page` instead of `aem.live` for preview/stage.

> If your CDN can't proxy `/libs/*` to a different origin, set `LIBS` to the absolute URL in `scripts/scripts.js` and `head.html`:
> ```js
> const LIBS = 'https://milo.adobe.com/libs';
> ```

### Case 2 — project lives under a sub-path (e.g. `/foo/docs`)

| What | From | To | Strip prefix? |
|------|------|----|---------------|
| Milo libs | `your-domain.com/foo/docs/libs/*` | `main--milo--adobecom.aem.live/libs/*` | Yes |
| Project code | `your-domain.com/foo/docs/{scripts,styles,blocks,...}/*` | `main--{repo}--adobecom.aem.live/{scripts,...}/*` | Yes |
| Project content | `your-domain.com/foo/docs/*` | `main--{repo}--adobecom.aem.live/foo/docs/*` | **No** |

Content must keep the full path — EDS derives `.plain.html` URLs from it. Stripping the prefix from content requests causes 404s.

**SharePoint:** author all documents under the same path they'll appear at publicly (`/foo/docs/…`), not at the root.

Example CloudFront viewer-request function:

```js
const PREFIX = '/foo/docs';

function handler(event) {
  const req = event.request;
  if (
    req.uri.startsWith(PREFIX + '/libs/')
    || req.uri.endsWith('.js')
    || req.uri.endsWith('.css')
  ) {
    req.uri = req.uri.slice(PREFIX.length) || '/';
  }
  return req;
}
```

**Debugging:**
- JS/CSS has wrong `Content-Type` → prefix stripping is missing; CDN is returning a 404 page instead of the file.
- `.plain.html` returns 404 → prefix is being stripped from content; remove it from that rule.
- Links go to the wrong URL → documents in SharePoint aren't authored with the full public path.

## Security
1. Create a Service Now ID for your project via [Service Registry Portal](https://adobe.service-now.com/service_registry_portal.do#/search)
2. Update the `.kodiak/config.yaml` file to make sure valid team members are assigned security vulnerability Jira tickets.
