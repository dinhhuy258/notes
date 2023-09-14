# Web Security

## Clickjacking

Clickjacking is a method of tricking website users into clicking on a harmful link, by disguising the link as something else.

[Demo](https://www.hacksplaining.com/exercises/click-jacking)

### Protection

**X-Frame-Options**

The `X-Frame-Options` HTTP header is an older standard that can be used to indicate whether or not a browser should be allowed to render a page in a `<frame>`, `<iframe>` or `<object>` tag. It was designed specifically to help protect against clickjacking, but has since between made obsolete by content security polices.

The supported values are:

- `DENY`: This web page cannot be embedded anywhere. This is the highest level of protection as it doesn’t allow anyone to embed our content.
- `SAMEORIGIN`: Only pages from the same domain as the current one can embed this page. This means that `example.com/embedder` can load `example.com/embedded` so long as its policy is set to `SAMEORIGIN`. This is a more relaxed policy that allows owners of a particular website to embed their own pages across their application.
- `ALLOW-FROM uri`: Embedding is allowed from the specified URI. We could, for example, let an external, authorized website embed our content by using `ALLOW-FROM https://external.com`. This is generally used when you intend to allow a third party to embed your content through an iframe.

**Content Security Policy**

The `Content-Security-Policy` HTTP header is part of the HTML5 standard, and provides a broader range of protection than the `X-Frame-Options` header (which it replaces). It is designed in such a way that website authors can enumerate individual domains from which resources (like scripts, stylesheets, and fonts) can be loaded, and also domains that are permitted to embed a page.

To control where your site can be embedded, use the `frame-ancestors` directive:

- `Content-Security-Policy: frame-ancestors 'none'`
- `Content-Security-Policy: frame-ancestors 'self'`
- `Content-Security-Policy: frame-ancestors *uri*`
