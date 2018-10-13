---
title: "python获取mac地址"
date: 2013-01-03
description: "python获取mac地址"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2013/01/03/python-get-mac/]
---

```python
import uuid
node = uuid.getnode()
mac = uuid.UUID(int = node).hex[-12:]
print mac
```
