---
title: "Hosting a highly available inexpensive static site"
date: 2020-12-04T19:19:41-05:00
draft: false 
---

### Deploying a static website with Hugo, B2 cloud and Cloudflare

#### Inexpensive hosting with high uptime and bandwidth capabilities

This website is built using Hugo, a static website building tool written in Go.  I've created a theme template and using Hugo I can easily generate a complete website. The generated website is then uploaded to a public B2 cloud bucket. Access to the website hosted in the B2 bucket is then provided through Cloudflare.

Taking advantage of the bandwidth alliance between B2 and Cloudflare gives free bandwidth and inexpensive storage of your data.  Currently B2 cloud charges $5 per month for 1TB of data. 

#### Creating the website bucket

After creating an account on B2 cloud create a new bucket and enable public access. Create an application key and give it access to the newly created bucket. Note down the application key and ID as they will be needed in the following steps.

#### Creating the Hugo website

Hugo is a powerful static site generation tool with a large community and many free themes available. It has a powerful template engine and content management tools.

I created this websites current theme using Hugo templates and found it very enjoyable once I got more familiar with Hugo template order precedence.

Once the Hugo static website is ready you can build the static files with the `hugo` command.

```
❯ hugo
Start building sites … 

                   | EN  
-------------------+-----
  Pages            | 22  
  Paginator pages  |  0  
  Non-page files   |  1  
  Static files     |  0  
  Processed images |  0  
  Aliases          |  0  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 53 ms
```

<br />

#### Using Github Actions to automatically upload the website files

Use Github Actions to upload the finished static website files. The CI jobs will check the spelling of the content, build the website, and then upload the public directory to the B2 cloud bucket.

In the Github Actions configuration I created 2 jobs, one to check the spelling and if that succeeds then another to create the website and upload to B2.

``` .github/workflows/main.yml
name: Build Hugo Static Site
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
  workflow_dispatch:

jobs:
  check-spelling:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        ref: ${{ github.head_ref }}
    - name: Check Spelling
      uses: rojopolis/spellcheck-github-actions@0.6.0
  deploy:
    needs: [check-spelling]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        submodules: true
        fetch-depth: 0

    - name: Hugo setup
      uses: peaceiris/actions-hugo@v2.4.13
      with:
        extended: true
    - name: Hugo version
      run: hugo env
    - name: Build
      run: hugo --minify
    - name: Backblaze B2 Sync
      uses: earendildev/backblaze-b2-action@v0.2.0
      env:
        SOURCE_DIR: './public'
        B2_BUCKET_PATH: 'b2://${{ secrets.B2_BUCKET }}/'
        B2_APPKEY_ID: ${{ secrets.B2_APPKEY_ID }}
        B2_APPKEY: ${{ secrets.B2_APPKEY }}
```
<br />

#### Configure Cloudflare Workers to handle the requests

By default B2 cloud serves files from a URL that includes some unnecessary routes. In order to match the routes created in the website we can use Cloudflare Workers to handle the inbound requests. This also gives the ability to modify the request to remove some B2 debugging headers and do some error handling such as serving the 404.html page.

I've based the worker code off of this great example found [here.](https://jross.me/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers/) Using this code with some small modifications allows you to use B2 as the web server with full caching in the Cloudflare network.

I've made some modifications to allow serving the index.html page without specifying it directly as one limitation of B2 cloud is that there isn't a concept of an index page for HTTP requests. I've also added some checks to see if the request to B2 gives a 404 error, if so it will return the 404 page included with the website. This prevents a user from seeing the 404 page directly from the B2 cloud bucket.

``` javascript
'use strict';
const b2Domains = ['dev.roote.ca', 'www.roote.ca', 'roote.ca'];
const b2Bucket = 'r00t-homepage';
const b2UrlPath = `/file/${b2Bucket}/`;

addEventListener('fetch', event => {
	return event.respondWith(handleRequest(event));
});

// define the file extensions we wish to add basic access control headers to
const corsFileTypes = ['png', 'jpg', 'gif', 'jpeg', 'webp'];

// backblaze returns some additional headers that are useful for debugging, but unnecessary in production. We can remove these to save some size
const removeHeaders = [
	'x-bz-content-sha1',
	'x-bz-file-id',
	'x-bz-file-name',
	'x-bz-info-src_last_modified_millis',
	'X-Bz-Upload-Timestamp',
	'Expires'
];
const expiration = 31536000; // override browser cache for images - 1 year

function fixHeaders(url, status, headers){
	let newHdrs = new Headers(headers);
	// add basic cors headers for images
	if(corsFileTypes.includes(url.pathname.split('.').pop())){
		newHdrs.set('Access-Control-Allow-Origin', '*');
	}
	// override browser cache for files when 200
	if(status === 200){
		newHdrs.set('Cache-Control', "public, max-age=" + expiration);
	}else{
		// only cache other things for 5 minutes
		newHdrs.set('Cache-Control', 'public, max-age=300');
	}
	// set ETag for efficient caching where possible
	const ETag = newHdrs.get('x-bz-content-sha1') || newHdrs.get('x-bz-info-src_last_modified_millis') || newHdrs.get('x-bz-file-id');
	if(ETag){
		newHdrs.set('ETag', ETag);
	}
	// remove unnecessary headers
	removeHeaders.forEach(header => {
		newHdrs.delete(header);
	});
	return newHdrs;
};

function fixUrl(url) {
    if(b2Domains.includes(url.host)) {
        const path = url.pathname;
        if (path.includes(b2UrlPath)) {
            return;
        }
        if (path.includes('404.html')) {
            url.pathname = b2UrlPath + url.pathname;
            return;
        }
		if(path.endsWith('/')) {
			url.pathname = b2UrlPath + url.pathname + 'index.html';
		} else if(path.slice(-5).includes('.')) {
			url.pathname = b2UrlPath + url.pathname;
		} else {
            url.pathname = b2UrlPath + url.pathname + '/' + 'index.html';
        }
	}
}

async function fetchAndStreamNotFoundPage(resp, base_domain) {
  const { status, statusText } = resp;
  const { readable, writable } = new TransformStream();

  const response = await fetch(`${base_domain}/404.html`);
  const resp_text = await response.text();
  const { headers } = response;

  return new Response(resp_text, {
    status,
    statusText,
    headers
  });
}

async function handleRequest(event){
	const cache = caches.default; // Cloudflare edge caching
	let url = new URL(event.request.url);

	fixUrl(url);

	let response = await cache.match(url); // try to find match for this request in the edge cache
	if(response){
		// use cache found on Cloudflare edge. Set X-Worker-Cache header for helpful debug
		let newHdrs = fixHeaders(url, response.status, response.headers);
		newHdrs.set('X-Worker-Cache', "true");
		return new Response(response.body, {
			status: response.status,
	    	statusText: response.statusText,
			headers: newHdrs
		});
	}

	// no cache, fetch request, apply Cloudflare lossless compression
	response = await fetch(url);
    let newHdrs = fixHeaders(url, response.status, response.headers);
    if(response.status === 404) {
		return fetchAndStreamNotFoundPage(response, url.origin);
    }
	
	response = new Response(response.body, {
		status: response.status,
		statusText: response.statusText,
		headers: newHdrs
	});

	event.waitUntil(cache.put(url, response.clone()));
	return response;
}
```
