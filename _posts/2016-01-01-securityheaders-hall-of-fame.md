I recently found the securityheaders.io site.

The site, developed by Scott Helme, provides a URL based scanner to analyze the security header response of any given site. It then rates said sites based on what type of headers are permitted / blocked.

Today's modern web browsers are mostly secure (some more then others), but still a significant amount of the security relies upon websites being correctly configured to utilize the client side security mechanisms present in the browser. By giving browsers some guidelines on what can be trusted, you protect the users of your site against a lot of Cross Site Scripting and Man in the middle type attacks. Any individual running a site, owes it to their visitors, to do their best to insure they don't leave with more then they arrived with, namely malware.

When I first ran the scan against nfv.space, I will be honest, it needed attention, so I looked around and started to make the needed changes to my web server (nginx), until all holes were closed. A lot of these values were gleamed from different places (mostly the nginx documentation) and others were from Scott's site (which provides really good guidance, and does not leave you hanging dry). If you use apache, you will need to dig out your own configs, although again, I am sure Scott's site will help guide you.

For nginx its pretty simple to add the headers with a 'add_header' key followed by a value.

CRITERIA CHECKS

securityheaders.io checks for the following header response results (most descriptions care of developer.mozilla.org). For each value, I will also share the nginx.conf key / value I used.


# X-FRAME-OPTIONS

X-Frame-options is an HTTP response header and can be used to indicate whether or not a browser should be allowed to render a page in a <frame>, <iframe> or<object> . Sites can use this to avoid clickjacking attacks, by ensuring that their content is not embedded into other sites.

add_header X-Frame-Options "SAMEORIGIN";
The values available are:

DENY: The page cannot be displayed in a frame, regardless of the site attempting to do so.

SAMEORIGIN The page can only be displayed in a frame on the same origin as the page itself.

ALLOW-FROM uri The page can only be displayed in a frame on the specified origin.

I opted for SAMEORIGIN, meaning only allow framing from the same domain.


# HTTP STRICT TRANSPORT SECURITY

HTTP Strict Transport Security (often abbreviated as HSTS) is a security feature that lets a web site tell browsers that it should only be communicated with using HTTPS, instead of using HTTP. Using Strict Transport Security, lets a web site inform the browser that it should never load the site using HTTP, and should automatically convert all attempts to access the site using HTTP to HTTPS requests instead.

add_header Strict-Transport-Security "max-age=31536000; includeSubdomains";
For my setting, I elected for a max age of one year, and I include subdomains, as I plan to run another couple of sites under nfv.space.

# X-CONTENT-TYPE-OPTIONS

X-Content-Type-Options uses only one defined value, "nosniff", prevents Internet Explorer and Google Chrome from MIME-sniffing a response away from the declared content-type. This also applies to Google Chrome, when downloading extensions. This reduces exposure to drive-by download attacks and sites serving user uploaded content that, by clever naming, could be treated by MSIE as executable or dynamic HTML files.

add_header X-Content-Type-Options nosniff;

# X-XSS-PROTECTION

X-XSS-Protection header enables the Cross-site scripting (XSS) filter built into most recent web browsers. It's usually enabled by default anyway, so the role of this header is to re-enable the filter for this particular website if it was disabled by the user.

add_header X-XSS-Protection "1; mode=block";
The token mode=block will prevent browser (IE8+ and Webkit browsers) to render pages (instead of sanitizing) if a potential XSS reflection (= non-persistent) attack is detected.

# CONTENT SECURITY POLICY

Content Security Policy (often abbreviated as CSP) is a computer security standard introduced to prevent cross-site scripting (XSS), clickjacking and other code injection attacks resulting from execution of malicious content in the trusted web page context. CSP is built upon the principle of Same-origin Policy. A simple example of a CSP / Same Origin Policy would be that code from https://mybank.com should only have access to https://mybank.com's data, and not https://evil.com.

CSP's are typically built on a least privilege model, whereby you tell browsers what sites are allowed to run say for example java script, or load external images.

In my case I only needed to allow three sites:

https://fonts.gstatic.com (to allow my google fonts to be rendered on this page)
https://www.google-analytics.com (for google anayltics)
https://avatars.githubusercontent.com (for the small avatar images used to render my github profile).

Here is my complete entry:
'''
add_header Content-Security-Policy "default-src 'self'; script-src 'self' https://www.google-analytics.com https://avatars.githubusercontent.com; img-src *; style-src https: 'unsafe-inline'; font-src 'self' https://fonts.googleapis.com  https://fonts.gstatic.com; object-src 'none'";
'''

A tip here is to use 'insect element' or mozilla developer tools, to see what external sources are used to render the pages in your website. Doing this got me the google fonts API and the github calls.


# HTTP PUBLIC KEY PINNING (HPKP)

HTTP Public Key Pinning (HPKP) is a security mechanism which allows HTTPS websites to resist impersonation by attackers using mis-issued or otherwise fraudulent certificates. (For example, sometimes attackers can compromise certificate authorities, and then can mis-issue certificates for a web origin.) The HTTPS web server serves a list of public key hashes, and on subsequent connections clients expect that server to use 1 or more of those public keys in its certificate chain.

'''
add_header Public-Key-Pins 'pin-sha256="xXEEzomG2N6hdat2mh4ihZyg5HTtY/ooyW6ZiDVOpBg="; \
        pin-sha256="xMLI6oh+YtS/VUVneBYhuNpruxWj5TCtDN28Yb87rG4="; \
        pin-sha256="t8bKmXF6QhKHZSriAlXS3/l32+pvYfHBCGiCAPM4NqI="; \
        max-age=10';
'''

I found the best guide for setting this up, is right on securityheaders.io itself

RESULTS

First thing I checked was the actual headers themselves, doing this showed my config changes were active:

'''
HTTP/1.1 200 OK
Server: nginx/1.4.6 (Ubuntu)
Date: Fri, 01 Jan 2016 19:48:31 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/5.5.9-1ubuntu4.14
Link: https://nfv.space/wp-json/; rel=https://api.w.org/
X-Frame-Options: SAMEORIGIN
Strict-Transport-Security: max-age=31536000; includeSubdomains
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src self; script-src self https://ssl.google-analytics.com https://avatars.githubusercontent.com; img-src self https://ssl.google-analytics.com https://avatars.githubusercontent.com; style-src; font-src self https://fonts.googleapis.com  https://fonts.gstatic.com; object-src none
Public-Key-Pins: pin-sha256=xXEEzomG2N6hdat2mh4ihZyg5HTtY/ooyW6ZiDVOpBg=; \
	pin-sha256=xMLI6oh+YtS/VUVneBYhuNpruxWj5TCtDN28Yb87rG4=; \
	pin-sha256=t8bKmXF6QhKHZSriAlXS3/l32+pvYfHBCGiCAPM4NqI=; \
	max-age=10

'''

The result on securityheaders.io was an A+ 'hall of fame' rating.

![Multi-Tenancy](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/halloffame.png)

One further interesting point. I took a look at quite a few major banking / tech sites, as was shocked to see how vulnerable they were.
