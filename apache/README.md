###### FAQ / APACHE
# Configuration

## Configure proxy rules with Variables  

> This configuration work wih Apache 2.2+. Hower to check if the rewrite rule is applyied 
> make sur to add the right directive to enable rewrite logging as follows


```
  <IfModule mod_rewrite.c>
        RewriteEngine on
        RewriteLogLevel 3
        RewriteLog /var/log/apache2/rewrite.log
  </IfModule>
```

### Problem

Suppose to want to forward your web traffic to different targets depending on the request URI.

* When the server find `cdn`   at the beginning of the request URI, it should forward the traffic to a AWS cloudfront endpoint
* When the server find `local` at the beginning of the request URI, it should serve the traffic to the root directory of our server

How can we achieve this ? 

### Solution 

To do this we need to use a feature of apache server called : RewriteMap.
We create a mapping file where we store our variable as key/value pair as follows : 

```bash
# File : /etc/apache2/extras/proxy-urls.mappings
#
# Key           # Value
CDN_ENDPOINT    blablablaxxx.cloudfront.net
LOCAL_ENDPOINT  /
```

This content should be put in a file named at your convinienc; I choosed to name mine like : `/etc/apache2/extras/proxy-urls.mappings`

The next is to add the following lines to your apache vhost or main configuration file : 

```
<IfModule mod_rewrite.c>
  RewriteEngine on
  RewriteLogLevel 3
  RewriteLog /var/log/apache2/rewrite.log

  RewriteMap  proxy-urls "txt:/etc/apache2/extras/proxy-urls.mappings"
  RewriteRule "^/cdn/(.*)"    "${proxy-urls:CDN_ENDPOINT}/$1" [P,L]
  RewriteRule "^/local/(.*)"  "${proxy-urls:LOCAL_ENDPOINT}/$1" [P,L]
</IfModule>
```  

Now if you want to change the value of your endpoints, you just have to change the value in the mapping file `/etc/apache2/extras/proxy-urls.mappings`


### Troubleshootings

> Make sure you have the mode rewrite activated : `$: a2enmod rewrite`

