---
published: true
layout: post
title: Generating eth addresses using eth_account (python)
---
## Generating eth addresses using `eth_account`

This is quite straight forward...

```python
from eth_account import Account

acc = Account.create("EXTRA RANDOMNESS")
print(acc.address) # Account address
print(acc.key.hex()) # Account private key
```