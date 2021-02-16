# Asymmetric Cryptography
[TOC]
###### tags: `Security` `Crypto`

## RSA
1. 選兩個質數 (p 和 q)
    1. N = p * q
    1. ϕ(n) = (p-1) * (q-1)
1. 選加密指數 e
    1. 算解密指數 d
        e*d ≡ 1 (mod ϕ(n))
        ```
        from Crypto.Util.number import inverse
        inverse(e, phi)
        ```
公鑰 = (N, e)
私鑰 = d

1. 加密
E(m) = m^e mod n
`c = pow(m, e, n)`

1. 解密
D(c) = c^d mod n
`m = pow(c, d, n)`

1. interger <=> string
```python=
from Crypto.Util.number import bytes_to_long, long_to_bytes
c = bytes_to_long(open("./flag.enc", "rb").read())
decrypt = long_to_bytes(m)
```

* 用openssl看金鑰
`openssl rsa -pubin -in pubkey.txt -text -modulus`
n = modulus
* 用python讀金鑰
```python=
from Crypto.PublicKey import RSA
key = RSA.importKey(open("pubkey.txt", "rb").read())
key.n
key.e
```
### 單純分解 N
online database : factordb.com
```python=
from Crypto.Util.number import inverse, bytes_to_long, long_to_bytes

n = 66473473500165594946611690873482355823120606837537154371392262259669981906291
p = 800644567978575682363895000391634967
q = 83024947846700869393771322159348359271173
e = 65537

phi = (p - 1) * (q - 1)
c = bytes_to_long(open("./flag.enc", "rb").read())
d = inverse(e, phi)
m = pow(c, d, n)
decrypt = long_to_bytes(m)
print(decrypt)
```
小技巧
```
qiwi CTF iRoot.txt
M^e mod N = C

M = 2
e = 3
N = 100

如果 M^e 小於 N    =>   C開e次方根等於M
```

### Twin Prime
```
定義:兩個公鑰分別為
n1 = p*q
n2 = (p+2)*(q+2)
```
```
公式
p+q = (n2 - n1 - 4)/2
n1_phi = n1 - (p + q) + 1
n2_phi = n1 + (p + q) + 1
```
### common factor attack
### 加密指數攻擊 - Hastad’s Broadcast Attack
### 模數攻擊 - common modulus attack
```python=
from Crypto.Util.number import inverse, bytes_to_long, long_to_bytes

# C1 = m^e1 mod n
# C2 = m^e2 mod n
# gcd(e1, e2) = 1
#     e1*s1+e2*s2 = 1
# C1^s1 * C2^s2 = m mod n

def xgcd(b, n):
    x0, x1, y0, y1 = 1, 0, 0, 1
    while n != 0:
        q, b, n = b // n, n, b % n
        x0, x1 = x1, x0 - q * x1
        y0, y1 = y1, y0 - q * y1
    return  b, x0, y0

def common_modulus_attack(c1, c2, e1, e2, n):
    _, s1, s2 = xgcd(e1, e2)
    if s1 < 0:
        s1 = -s1
        c1 = inverse(c1, n)
    if s2 < 0:
        s2 = -s2
        c2 = inverse(c2, n)
    c1s1 = pow(c1, s1, n)
    c2s2 = pow(c2, s2, n)
    m = (c1s1 * c2s2) % n
    return m
```