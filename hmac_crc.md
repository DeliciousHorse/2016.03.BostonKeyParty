There was given following hmac function:

```python
CRC_POLY = to_bits(65, (2**64) + 0xeff67c77d13835f7)
CONST = to_bits(64, 0xabaddeadbeef1dea)

def crc(mesg):
  mesg += CONST
  shift = 0
  while shift < len(mesg) - 64:
    if mesg[shift]:
      for i in range(65):
    mesg[shift + i] ^= CRC_POLY[i]
    shift += 1
  return mesg[-64:]

INNER = to_bits(8, 0x36) * 8
OUTER = to_bits(8, 0x5c) * 8

def hmac(h, key, mesg):
  return h(xor(key, OUTER) + h(xor(key, INNER) + mesg))
```

Given string "zupe zecret", that gives hmac: 0xa57d43a032feb286, task was to find hmac for a string "BKPCTF".
So we need to find a key.

Let's start by rewriting hmac function a bit, "||" is concatenation:
```
HMAC(KEY, m) = CRC( (KEY xor O) || CRC( (KEY xor I) || m))
```

PRE and POST inverts are 0 in crc, so we *COULD* rewrite inner CRC as a polynomial mod `CRC_POLY`, *normally* it would be
```
M = len(m) * 8 -- length of message m in bits
CRC( (K xor I) || m) = m*x^64 + K*x^(64+M) + I*x^(64+M) mod CRC_POLY
```
but you need to notice, that there is `CONST` added right at the beginning, to the polynomial actually looks like:
```
CRC( (K xor I) || m) = m*x^64 + K*x^(64+M) + I*x^(64+M) + CONST mod CRC_POLY
```

Now we can rewrite whole HMAC as a polynomial:
```
HMAC(key, m) = m*x^128 + K*x^(128+M) + I*x^(128+M) + CONST*x^64 + K*x^128 + O*x^128 + CONST mod CRC_POLY
```

Since we know HMAC (0xa57d43a032feb286), let's move most of the things:
```
K*(x^(128+M) + x^128) = HMAC(key, m) - ( m*x^128 + I*x^(128+M) + O*x^128 + CONST*x^64 + CONST) mod CRC_POLY
```

We can calculate right side without any problems.
Luckily `gcd(CRC_POLY, x^(128+M) + x^128)) == 1`, so our solution for key is given by:

```
K = (right part) * inverse( x^(128+M) + x^128 ) mod CRC_POLY
```

You can read more about arithmetic on wikipedia: https://en.wikipedia.org/wiki/Finite_field_arithmetic
