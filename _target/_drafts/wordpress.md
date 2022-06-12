---
layout: post
title: wordpress
tags: wordpress web cms wpscan xmlrpc
created: "2022-06-10"
updated: "2022-06-10"
---

# wordpress

## xmlrpc

curl -d try_xmlrpc2.xml http://internal.thm/wordpress/xmlrpc.php

wp-config
wp-login
wp-cron

theme editor > 404.php -> reverse shell
curl http://internal.thm/blog/index.php/wp-json/wp/v2/users/?per_page=100