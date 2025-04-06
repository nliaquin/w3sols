# Challenge
Decrypt this RSA message:
```bash
309117097659990665453
125675338953457551017
524099092120785248852
772538252438953530955
547462544172248492882
028215860448757441963
543018082275730030658
585936545563088067075
131807465077304821584

N= 783340156742833416191

E= 653
```

# Guide

To decrypt the given RSA-encrypted message, we must complete the following steps:
1. Factorize N: Find the prime factors p and q such that N = p × q
2. Compute ϕ(N): Calculate Euler's totient function, ϕ(N) = (p − 1)×(q − 1)
3. Determine the private exponent d: Find 𝑑 d such that d ≡ e −1 modϕ(N)
4. Decrypt the ciphertext: Use d to decrypt the ciphertext and retrieve the original message


## 1. Factorize N

Given N = 783340156742833416191, the first task is to find its prime factors p and q. Since N is relatively small (21 digits), it is feasible to factorize using tools like msieve or YAFU.

Using YAFU, run the following:
```bash
$ yafu "factor(783340156742833416191)"

fac: factoring 783340156742833416191
fac: using pretesting plan: normal
fac: no tune info: using qs/gnfs crossover of 95 digits
fac: no tune info: using qs/snfs crossover of 95 digits
div: primes less than 10000
fmt: 1000000 iterations
Total factoring time = 0.0034 seconds


***factors found***
P11 = 28188776653
P11 = 27789079547

***factorization:***
783340156742833416191=28188776653*27789079547

ans = 1
```

In this output, YAFU factored N successfully, where p = 28188776653 and q = 27789079547.


## 2. Compute ϕ(N)
Calculate using Python, R, or any other language of your choice, (p - 1) * (q - 1)

```python
$ python
Python 3.13.2 (main, Feb  5 2025, 08:05:21) [GCC 14.2.1 20250128] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> p=28188776653
>>> q=27789079547
>>> phi_n = (p - 1) * (q - 1)
>>> print(phi_n)
783340156686855559992
>>> quit()
```

Now now we know ϕ(N) = 783340156686855559992


## 3. Determine the private exponent d
Given that e = 653 and ϕ(N) = 783340156686855559992, let's use the mod_inverse function from sympy in Python to calculate d.

```python
$ python
Python 3.13.2 (main, Feb  5 2025, 08:05:21) [GCC 14.2.1 20250128] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from sympy import mod_inverse
>>> e = 653
>>> phi_n = 783340156686855559992
>>> d = mod_inverse(e, phi_n)
>>> print(d)
334688979656405361773
>>> quit()
```

Now we know d = 334688979656405361773


## 4. Decrypt the ciphertext
Finally, using Python once more, we can decrypt the cipher text knowing everything above.

```python
from binascii import unhexlify
from sympy import mod_inverse

# RSA values from the challenge
N = 783340156742833416191
e = 653
d = 334688979656405361773

# Ciphertext blocks (as integers)
ciphertext_blocks = [
    309117097659990665453,
    125675338953457551017,
    524099092120785248852,
    772538252438953530955,
    547462544172248492882,
    28215860448757441963,
    543018082275730030658,
    585936545563088067075,
    131807465077304821584
]

# Decrypt each block, convert to bytes
plaintext_bytes = b''
for c in ciphertext_blocks:
    m = pow(c, d, N)
    
    # Convert decrypted integer to byte sequence
    # Determine minimum number of bytes needed
    block_size = (N.bit_length() + 7) // 8
    m_bytes = m.to_bytes(block_size, byteorder='big')
    
    plaintext_bytes += m_bytes

print(plaintext_bytes.decode('utf-8'))
```

#### Solution File: plaintext.txt


