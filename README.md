# WebPerfLib.NET

WebPerfLib.NET is a modern web optimization library for ASP.NET MVC Razor views.

The library makes it simple to optimize pages for both HTTP 1.1 and HTTP/2 clients.  Performance "hacks" from the HTTP 1.1 era such as bundling, inlining, and domain sharding actually __degrade performance__ for HTTP/2 clients.  However, it will be important to keep these techniques in place for legacy clients during the long transition to HTTP/2.

Additionally, 

WebPerfLib.NET includes the following features:

- __HTTP/2 optimizations__, including:
	- Server Push (via headers to integrate with HTTP/2 reverse proxies like [h2o](https://github.com/h2o/h2o)).
	- Unbundling of resources to improve caching on multiplexed connections.
	- Unsharding of domains to reduce latency on multiplexed connecti ons.
- __Legacy optimizations for HTTP v1 clients__, including:
	- Script & Stylesheet bundling.
	- Resource inlining.
	- Domain sharding.
- __Versioned resource URLs__ to enable optimal caching for all clients.
- __HTML5 &lt;picture&gt; element__ support for responsive images ([supported by Chrome and Firefox](http://caniuse.com/#search=picture) as of June 2015).
- __Automatic use of pre-minified files__ (if a 

The protocol-specific features can be enabled via configuration for all requests, or based on configured request headers. 

## Usage

WebPerfLib.NET adds extension methods to `HtmlHelper`, so it can be used easily from within any MVC view.

### Server Push for HTTP/2, Inlining for HTTP 1.1 : `@Html.Attach`  



### Bundling for HTTP 1.1: `@Html.Scripts` and `@Html.Styles`

    @Html.Image("~/Content/img.jpg")
    @Html.Picture("~/Content/img.jpg")
	@Html.Script

### `@Html.Picture`: &lt;picture&gt; Element for Images

### `@Html.Version` and All Other Methods: Versioned URLs

### All Methods: Use Minified Version in Release Mode 

## Background: HTTP/2 and Obsolete Performance Hacks

As of June 2015, HTTP/2 is supported by most web browsers globally ([see caniuse.com](http://caniuse.com/#search=http2) for up-to-date information).  However many websites are still using HTTP 1.1-era optimization techniques that __degrade performance__ for HTTP/2 clients.

### HTTP 1.0/1.1 Performance Hacks

Modern web pages include dozens of files.  Before HTTP/2, each file required a dedicated connection to the server.  These could be reused (using TCP Keep-Alive), but browsers limited the number of active connections per host.  These factors caused TCP connection and TLS negotiation to be a significant performance bottleneck.

Over time, some hacks were developed to work around this problem:

- __Bundling__ - combining multiple files together to reduce the number of files on the page.
- __Inlining__ - embedding files directly into the HTML document; again, to reduce the number of files on the page. 
- __Domain Sharding__ - referencing files on multiple different domains to work around connection-per-host restrictions.  Another benefit to this technique is that these other domains can be cookie-less, reducing the size of the request.

Bundling and Inlining both come at the expense of cacheability.  Whenever any file in a bundle is changed, the entire bundle must be expired from cache.  And inlined resources cannot be cached at all.

### HTTP/2 Multiplexing, Server Push, and Header Compression

In HTTP/2, connections to the server allow _multiplexed_ downloads.  This means that a single TCP connection can download multiple files in parallel.  In other words, there is no longer any benefit to be gained from bundling.  It also makes domain sharding an _anti_-optimization, since it will prevent these parallel downloads.

HTTP/2 also adds support for _Server Push_.  This allows a server to respond to a single HTTP request with multiple files.  Using Server Push to send a related file has the same effect as inlining it, but allows caching of the file.

Additionally, HTTP/2 compresses request headers with a lookup table that persists throughout a connection.  In other words, making additional requests over the same connection does not require re-uploading cookies and other headers.  This eliminates the other minor benefit of domain sharding.

### WebPerfLib.NET - Supporting the HTTP/2 Transition

The rollout of HTTP/2 will take a long time.  Internet Explorer versions through 10 will be widely used for years, and they will probably never support it.
  
Web applications need a way to support the HTTP 1.1-specific optimization techniques without letting them degrade performance for HTTP/2 clients.  This is the goal of WebPerfLib.NET.

