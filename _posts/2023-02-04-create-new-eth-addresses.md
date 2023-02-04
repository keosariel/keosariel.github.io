---
published: true
layout: post
title: Generating eth addresses using eth_account (python)
---

This is quite straight forward...

```bash $ pip install eth-account```

```python
from eth_account import Account

acc = Account.create("EXTRA RANDOMNESS")
print(acc.address) # Account address
print(acc.key.hex()) # Account private key
```
