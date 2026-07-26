# Hello to everyone reading in English! 👋

In this repo I want to walk step by step through one of the most important and newest **post-quantum digital signature** algorithms — one designed to resist attacks from quantum computers.

If, like me, you don't have patience for long papers, you're in the right place. This takes less than **10 minutes** to read, but by the end you'll have a solid picture of what a digital signature is and why post-quantum signatures matter.

---

# What is a digital signature?

A digital signature is something attached to a message or file so anyone can check:

- Did this message really come from the sender?
- Has the message changed since it was sent?

That's exactly what digital signatures are designed to answer.

> **Note:** A digital signature does not hide or encrypt the message. Anyone can see the message's content — the signature only confirms who sent it and that it wasn't tampered with.

## An example

Suppose Alice wants to send a PDF file to Bob.

### 1. Key Generation (Alice does this once)

```
public_key, private_key = key_generation()
```

Alice keeps `private_key` secret and shares `public_key` with everyone.

### 2. Signing (Alice signs the file)

```
signature = sign(file, private_key)
```

Alice sends Bob: the `file`, the `signature`, and her `public_key`.

### 3. Verification (Bob checks the signature)

```
is_valid = verify(file, signature, public_key)
```

If the signature checks out, Bob can be confident that:

- The file was signed by whoever holds the private key (Alice).
- The file hasn't changed since it was signed.

If even one letter, digit, or bit of the file changes, the signature check fails and Bob knows the file isn't trustworthy.

---

## RSA in simple terms

RSA is the oldest and most widely used digital signature scheme. Here's how it works:

**`key_generation()`** — Pick two huge, secret prime numbers `p` and `q`, then multiply them to get `n = p × q`. This `n` is public — but factoring it back into `p` and `q` is brutally hard. Compute a secret number `d` using `p` and `q`. The math guarantees that `d` and a public exponent `E` are inverses: anything raised to `d` then `E` (mod `n`) comes back unchanged. Your **public key** is `(n, E)`, your **private key** is `(n, d)`.

**`sign()`** — Hash the message to get a fingerprint `h`, then compute `signature = h^d mod n`. Only you can do this because only you know `d`.

**`verify()`** — Compute `signature^E mod n` and check if it equals the hash of the message. If it matches, the signature is valid.

**So what's the problem?**
Shor's algorithm, run on a quantum computer, can factor `n` quickly. Once quantum computers are powerful enough, RSA stops being safe — that's the whole motivation for schemes like ML-DSA.

![RSA hardness: multiplying primes is easy, factoring n back apart is hard](assets/rsa-hardness.gif)

## ML-DSA in simple terms (post-quantum)

Instead of relying on "factoring big numbers," ML-DSA relies on a geometric problem called the **lattice problem** — one that, as far as we know, quantum computers have no fast trick for either.

**`key_generation()`** — There's a public matrix `A` that everyone can see — think of it as a big, public maze. Your **private key** is a short, secret vector `s1` (a simple path through that maze). Your **public key** `t` is just where that path ends up: `t = A·s1`. Given only the endpoint, finding the secret path is essentially impossible — even for a quantum computer.

**`sign()`** — Take a fresh random vector `y`, compute where it lands `w = A·y`, then build a challenge `c` from the message and `w`. Mix the challenge into your secret: `z = y + c·s1`. Return `(z, c)` — never the secret `s1` or the random `y`.

**`verify()`** — Rebuild `w` using only public data: `w = A·z - c·t`. Check if hashing the message with this `w` produces the same challenge `c`. If it matches, the signature is valid.

![ML-DSA hardness: walking the lattice with a short secret is easy, reversing it isn't](assets/ml-dsa-hardness.gif)

![ML-DSA sign & verify: only (z, c) ever cross from signer to verifier — s1 and y never do](assets/ml-dsa-sign-verify.gif)

## Why does ML-DSA survive quantum computers?

Both RSA and ML-DSA rely on a **hard problem** — easy to verify, brutally hard to reverse. The difference is *which* hard problem.

**RSA's weakness:** Shor's algorithm gives quantum computers a fast way to factor large numbers. Once you can factor `n` back into `p` and `q`, you can compute the private key `d` and forge any signature. RSA is broken.

**ML-DSA's strength:** No known quantum algorithm can efficiently solve lattice problems like finding a short secret vector `s1` from `t = A·s1`. Quantum computers offer no significant speedup here — the best attacks are still exponential, just like on classical computers.

That's why ML-DSA is called "post-quantum": it's designed to stay secure even after large-scale quantum computers exist.

That security doesn't come free — ML-DSA trades bigger keys and signatures for quantum resistance:

![RSA vs ML-DSA: the cost of quantum resistance in key and signature size](assets/rsa-vs-ml-dsa-tradeoff.gif)

## The code

- `rsa.py` — a minimal RSA implementation
- `ml_dsa.py` — a simplified ML-DSA implementation (for easier learning; it skips the error term and high-bit rounding that real ML-DSA uses)

*(نسخه‌ی فارسی: [readme.fa.md](readme.fa.md))*
