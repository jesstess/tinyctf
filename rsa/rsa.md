WTC RSA BBQ
===========

* Flag: **how_d0_you_7urn_this_0n?**
* File: [cry200.zip](data/cry200.zip "cry200.zip")

Unzipping the challenge file produces 2 files: `key.pem` and `cipher.bin`.

`key.pem` is a base-64 encoded public key with an unusual number of
slashes:

```
-----BEGIN PUBLIC KEY-----
MIIEUzANBgkqhkiG9w0BAQEFAAOCBEAAMIIEOwKCBDIGLT1hySRSYwFH6JZw////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////
//8CAwEAAQ==
-----END PUBLIC KEY-----
```

Given the "RSA" in the title of the challenge, it seems like the goal
is for us to use a special property of the unusual public key to be
able to derive the private RSA exponent, allowing us to decrypt
`cipher.bin` without having the private key.

To get ourselves into the RSA mindset, let's remind ourselves what
information we have by having a public key, and what information we
need to decrypt `cipher.bin`.

1. Pick two primes, `p`, and `q`.

2. `n = pq`. `n` will be our modulus.

3. `φ(n) = (p − 1)(q − 1)`

4. `e` = public key exponent. `1 < e < φ(n)`, and `e` and `φ(n)` are coprime.

5. `d ≡ e^−1 (mod φ(n))`. (`d` is the multiplicative inverse of `e mod φ(n)`)
`d` = private key exponent.

The public key has 2 parts: our modulus `n` (`n = pq`), and `e`, the public key exponent.

The private key has 2 parts: our modulus `n`, and `d`, the private key exponent.

If we had `p` and `q`, we could calculate `d` (because we could
compute `φ(n)` efficiently). Typically, `n` is too large to factor `p` and `q`.

Our challenge is likely to figure out what is special about this
public key such that we are able to determine `p` and `q` by a method
other than factoring.

First, let's take a look at the information in the public key:

```
$ openssl rsa -pubin -inform PEM -text -noout -in key.pem
Modulus (8587 bit):
    06:2d:3d:61:c9:24:52:63:01:47:e8:96:70:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:ff:
    ff:ff:ff:ff:ff:ff:ff:ff:ff
Exponent: 65537 (0x10001)
```

Our exponent, `e`, is the standard `65537`. Our modulus is a really long
number that ends in a bunch of `0xff`.

At this point, we might:

1. Noodle around on the internet for a while looking for special
properties of public keys.

2. Connect the "WTC" in the title back to the World Trade Center: the
Twin Towers.

3. Talk to a mathy friend or observe that the modulus prefix
`06:2d:3d:61:c9:24:52:63:01:47:e8:96:70 = 699549860111849`, which is a
perfect square.

Whichever route, we'd come to the conclusion that this modulus is
special because it is a product of [twin
primes](http://en.wikipedia.org/wiki/Twin_prime): `q = p + 2`.

That means `n = pq = p(p+2) = (p+1 + 1)(p+1 - 1) = (p+1)^2 - 1`.

If we add 1 to the modulus (add 1 to a number that ends in a bunch of
`0xff`s), we get `699549860111847^2` followed by 1061 zero bytes.

`n = 699549860111847^2 * 256^1061 = (699549860111847 * 2^4244)^2 = pq`

Because `p` and `q` are twin primes, recovering them from `n`
expressed as a square is easy: they are just above and below the
square root: `p = (699549860111847 * 2^4244) - 1` and
`q = (699549860111847 * 2^42441) + 1`.

We wouldn't normally be able to calculate `φ(n)` for such a large `n`,
but now that we have `p` and `q`, we can readily recover `d` given
that `de = 1 mod φ(n) = 1 mod (p-1)(q-1)`. Here's the full decryption:

```python
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

cipher = int(open("cipher.bin", "rb").read().encode("hex"), 16)

e = 65537
p = 699549860111847 * 2**4244 - 1
q = p + 2
n = p * q
fi = (p-1)*(q-1)
d = modinv(e, fi)

flag = pow(cipher, d, n)
print ("%x" % flag).decode("hex")
```

Running this script against `cipher.bin` reveals the flag:

`Congratulations! Here is a treat for you: flag{how_d0_you_7urn_this_0n?}`

## Bonus: walking through constructing the problem

A great way to ensure that we truly understand this challenge is to
walk through the steps necessary to create it.

### 1. Pick our twin primes.

We need twin primes that are +/- 1 from a large power of 2. Not all
twin primes have this property, but if we have a list of twin primes,
we can compute numbers of the form `k(2^x)` until we find a valid pair
(where `x` is the number of zeros we want):


```python
twins = {int(elt) + 1: True for elt in file("twins.txt").read().strip().split()}

x = 16

for k in range(10000):
    if k * 2**x in twins:
        print k, k * 2**x
```

(I took [twins.txt](data/twins.txt) from [this website](https://primes.utm.edu/lists/small/100ktwins.txt).)

Running the above script, for `x = 16`, we have two pairs of twin
primes that are also +/- 1 from a power of 2 in the range captured in
the twins file:

`786431` and `786433`

`7667711` and `7667713`

Let's choose `7667711` for this exercise.

### 2. Encrypt a message using the public key

```python
import math

def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

# p and q found by searching for twin primes.

p = 7667711
q = p + 2
n = p * q # '0x3578ffffffff'
e = 65537
plaintext = "Hi"
m = int(plaintext.encode("hex"), 16) # 18537
c = pow(m, e, n)
```

### 3. Decrypt the message using the private key

```python
# Decrypting using the private key.

fi = (p-1)*(q-1)
d = modinv(e, fi)
print ("%x" % pow(c, d, n)).decode("hex") # prints 'Hi'
```

### 4. Decrypt the message using the public key

```python
# Decrypting using only the public key (n, e) and knowledge that the
# modulus is the product of twin primes.

e = 65537

# modulus = 0x3578ffffffff
# modulus = (sqrt(prefix) * 2^x)^2
# sqrt(prefix) * 2^x - 1 = p

prefix = 0x3578
sqrt_of_prefix = math.sqrt(prefix + 1)

# 4 sets of ff => 256^4 = 2^8^4 = 2^32 = (2^16)^2
# sqrt of modulus = sqrt(prefix) * sqrt(2^16)
sqrt_of_modulus = sqrt_of_prefix * 2**16
p, q = sqrt_of_modulus - 1, sqrt_of_modulus + 1
fi = (p-1)*(q-1)
d = int(modinv(e, fi))
print ("%x" % pow(c, d, n)).decode("hex") # Also prints 'Hi'
```

[« Return to challenge board](../README.md "Return to challenge board")
