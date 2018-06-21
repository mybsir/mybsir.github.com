---
layout: post
title:  Python脚本使用Django的Models
categories: Django
description: Python脚本使用Django的Models
keywords: Python脚本 , Django Models
---

# Python脚本使用Django的Models

```
import sys,os,django
sys.path.append(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "treasure.settings")
django.setup()
```
