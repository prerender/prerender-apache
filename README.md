# prerender-apache
This is the Prerender.io middleware configuration for an Apache server to allow Google (and other crawlers) to crawl your javascript website.

The `httpd.conf` and `.htaccess` in this repository are full files for reference, but I will discuss the modifications made to each file here.

# httpd.conf
Uncomment these lines to allow the rewrite in `.htaccess` and to allow the proxy over https:
```
LoadModule headers_module libexec/apache2/mod_headers.so
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_http_module libexec/apache2/mod_proxy_http.so
LoadModule ssl_module libexec/apache2/mod_ssl.so
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
```

And add this to the bottom of the file to allow the proxy over https:
```
# Prerender.io configuration

# Allow proxying over https
SSLProxyEngine on

# These two commands ignore the validity of the SSL Certificate of the Prerender server
# Only uncomment the two SSLProxy lines if you are testing with a local prerender server over https with a self-signed cert
# A hosted Prerender server should have a valid SSL Certificate so the next two lines can stay commented in production
#SSLProxyCheckPeerName off
#SSLProxyVerify none
```

# .htaccess
Here is the actual config that sends the request to Prerender.io:
```
# Change YOUR_TOKEN to your prerender token
# Change https://service.prerender.io/ (at the end of the last RewriteRule)
# to http://localhost:3000/ when testing with a local prerender server

<IfModule mod_headers.c>
    RequestHeader set X-Prerender-Token "YOUR_TOKEN"
    RequestHeader set X-Prerender-Version "prerender-apache@2.0.0"
</IfModule>

<IfModule mod_rewrite.c>
    RewriteEngine On

    <IfModule mod_proxy_http.c>
        RewriteCond %{HTTP_USER_AGENT} googlebot|bingbot|yandex|baiduspider|facebookexternalhit|twitterbot|rogerbot|linkedinbot|embedly|quora\ link\ preview|showyoubot|outbrain|pinterest\/0\.|pinterestbot|slackbot|vkShare|W3C_Validator [NC,OR]
        RewriteCond %{QUERY_STRING} _escaped_fragment_
        RewriteCond %{REQUEST_URI} ^(?!.*?(\.js|\.css|\.xml|\.less|\.png|\.jpg|\.jpeg|\.gif|\.pdf|\.doc|\.txt|\.ico|\.rss|\.zip|\.mp3|\.rar|\.exe|\.wmv|\.doc|\.avi|\.ppt|\.mpg|\.mpeg|\.tif|\.wav|\.mov|\.psd|\.ai|\.xls|\.mp4|\.m4a|\.swf|\.dat|\.dmg|\.iso|\.flv|\.m4v|\.torrent|\.ttf|\.woff|\.svg))

        RewriteRule ^(index\.html|index\.php)?(.*) https://service.prerender.io/%{REQUEST_SCHEME}://%{HTTP_HOST}/$2 [P,END]
    </IfModule>
</IfModule>
```