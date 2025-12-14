---
title: "{{ replace .Name "-" " " | title }}"
description: ""
date: {{ .Date }}
Lastmod: {{ .Date }}
summary: "示例摘要"
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/{{ substr (md5 (.Date)) 4 6 }}/1600/900.webp"
tags: ["自我折腾"]
series: ["建站技术"]
series_order: 1
---
