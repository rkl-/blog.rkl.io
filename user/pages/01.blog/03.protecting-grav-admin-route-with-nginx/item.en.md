---
title: 'Protecting Grav admin route with Nginx'
media_order: security-2910624_1280.jpg
published: true
date: '28-12-2017 18:38'
taxonomy:
    category:
        - blog
    tag:
        - linux
        - server
        - nginx
        - grav
author: 'Romano Kleinwaechter'
---

[Grav](https://getgrav.org/) is a wonderfull and lightwight software for blogging and microsite setups. If you already use Grav or want to use it, you should consider this [Nginx](https://nginx.org/en/) setup for protecting the admin route.

**1. Protection by IP address**
```
location ~ ^/admin.*$ {
    try_files $uri $uri/ /index.php?_url=$uri&$query_string;
    
    allow 127.0.0.1;
    deny all;
}
```

**2. Using a special cookie (not really safe)**
```
location ~ ^/admin.*$ {
    if ($http_cookie !~ 'my-sepcial-cookie=admin-access') {
        return 403;
    }

    try_files $uri $uri/ /index.php?_url=$uri&$query_string;
}
```

The second part is not the safest one. However, in some cases it can be usefull.

You should place one of the blocks before the regular `location ~ \.php$` block.