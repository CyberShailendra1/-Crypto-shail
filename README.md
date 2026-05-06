# 🔐 CryptoCTF Solver

> **Automatic flag extraction from CTF cryptography challenges**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![CTF](https://img.shields.io/badge/Category-CTF%20Tool-purple)](.)

A production-ready, extensible Python tool that **auto-detects** encryption types and **automatically applies** the right attacks to extract flags from Capture The Flag cryptography challenges.

---

## ✨ Features at a Glance

| Category | Attacks |
|----------|---------|
| **RSA** | Small-e, Wiener, Hastad broadcast, Fermat, FactorDB, p-1 smooth, common modulus |
| **AES** | ECB detection, byte-at-a-time, CBC bit-flip, padding oracle, key brute-force |
| **XOR** | Single-byte (freq analysis), multi-byte (Hamming), crib dragging, many-time pad |
| **Classical** | Caesar (all 25), Vigenere (IC/Kasiski), Affine (all a,b), Atbash, ROT-n, Rail Fence, Bacon |
| **Encoding** | Base16/32/58/64/85, Hex, Binary, Morse, URL, ASCII decimal/octal, HTML entities |
| **Hash** | MD5/SHA1/SHA256/SHA512 — wordlist, brute-force, online lookup, SHA1 length extension |
| **ECC** | Baby-step giant-step DLOG, singular curve, Pohlig-Hellman |
| **DH** | Small subgroup, BSGS, Pohlig-Hellman |
| **Remote** | Netcat / TCP service interaction (pwntools + raw socket fallback) |

---

## 🚀 Quick Start

```bash
git clone https://github.com/cybershailendra1/Crypto-shail
cd Crypto-shail


# Auto-detect and solve
python crypto_solver.py -f ciphertext.txt

# Inline ciphertext
python Crypto-shail.py -c "SGVsbG8gV29ybGQ="

# Connect to remote service
python Crypto-shail.py -r ctf.example.com:1337

# Force RSA attack with known e
python Crypto-shail.py -f rsa.txt -t rsa -e 3

# Hash cracking with wordlist
python Crypto-shail.py -f hash.txt --wordlist /path/to/rockyou.txt

# XOR with known crib
python Crypto-shail.py -f ct.bin -t xor --crib "flag{"
```

---

## 📦 Installation

### Requirements

```
Python 3.9+
pycryptodome    # AES, padding
gmpy2           # big integer operations
requests        # FactorDB, online hash lookup
pwntools        # remote CTF interaction
sympy           # prime generation (p-1 attack)
```

```bash
pip install -r requirements.txt
```

---

## 🛠 Command-Line Reference

```
python Crypto-shail.py [options]

Input:
  -f FILE           Input file (ciphertext, RSA params, hash, etc.)
  -c TEXT           Inline ciphertext string
  -r HOST:PORT      Connect to remote CTF service
  --extra FILE...   Extra files (Hastad broadcast / many-time pad)

Attack selection:
  -t TYPE           Force attack type (see list below)

RSA parameters:
  -n N              Modulus
  -e E              Public exponent
  --cipher C        Ciphertext integer
  -p P              Prime p (if known)
  -q Q              Prime q (if known)
  -d D              Private exponent (if known)

AES parameters:
  --key HEX         AES key (hex-encoded)
  --iv  HEX         Initialization vector (hex-encoded)
  --mode MODE       ECB | CBC | CTR | GCM

XOR:
  --crib TEXT       Known-plaintext crib for XOR / crib dragging

Hash cracking:
  --wordlist FILE   Dictionary file (rockyou.txt etc.)
  --brute-len N     Max brute-force length [default: 5]
  --offline         Disable online hash lookup

Flag options:
  --flag-pattern R  Custom flag regex (default: flag{...})

Remote:
  --rounds N        Max interaction rounds [default: 50]

Output:
  -v, --verbose     Show all intermediate decryption attempts
  -o FILE           Save results to file
  --log-level LVL   DEBUG | INFO | WARNING | ERROR
```

### Supported `-t` values

```
rsa, aes, xor, caesar, vigenere, affine, classical,
base64, base32, base16, hex, binary, morse, url,
rot13, atbash, base58, md5, sha1, sha256, hash,
ecc, dh
```

---

## 🧠 Architecture

```
cryptoctf/
├── Crypto-shail.py          ← Main CLI + orchestrator
├── requirements.txt
├── README.md
└── modules/
    ├── __init__.py
    ├── utils.py              ← Number theory, flag extraction, helpers
    ├── detector.py           ← Auto-detection (entropy, IC, regex, JSON)
    ├── encoding.py           ← Base*/Hex/Binary/Morse/ROT/Atbash/URL
    ├── classical.py          ← Caesar/Affine/Vigenere/RailFence/Bacon
    ├── xor_attacks.py        ← Single-byte/multi-byte/crib drag/MTP
    ├── rsa_attacks.py        ← Full RSA attack suite + FactorDB
    ├── aes_attacks.py        ← ECB/CBC/padding oracle/brute-force
    ├── hash_cracker.py       ← Dict/brute/online/length extension
    ├── ecc_dh.py             ← BSGS/Pohlig-Hellman/singular/small-subgroup
    └── remote.py             ← pwntools + raw socket remote interaction
```

---

## 📖 Usage Examples

### RSA — Small Exponent (e=3)

Given a file `rsa.txt`:
```
n = 179769313486231590...
e = 3
c = 857359647834512...
```

```bash
python Crypto-shail.py -f rsa.txt -t rsa
```

### RSA — Hastad Broadcast (e=3, multiple ciphertexts)

```bash
python Crypto-shail.py -f rsa1.txt --extra rsa2.txt rsa3.txt -t rsa -e 3
```

### Vigenere Cipher

```bash
python Crypto-shail.py -c "LXFOPVEFRNHR" -t vigenere
```

### XOR Many-Time Pad

```bash
python Crypto-shail.py -f ct1.bin --extra ct2.bin ct3.bin ct4.bin -t xor
```

### SHA1 Length Extension Attack

```python
from modules.hash_cracker import sha1_length_extension

new_hash, forged = sha1_length_extension(
    original_hash="da39a3ee5e6b4b0d3255bfef95601890afd80709",
    original_msg_len=10,
    append_data=b";admin=true",
    secret_len=16,
)
```

### Padding Oracle (POODLE)

```python
from modules.aes_attacks import padding_oracle_decrypt

def my_oracle(iv: bytes, ct: bytes) -> bool:
    # return True if server responds without padding error
    ...

plaintext = padding_oracle_decrypt(ciphertext, my_oracle, iv)
```

### Remote Challenge (Auto-solve loop)

```bash
python crypto_solver.py -r crypto.ctf.example.com:4433 --rounds 20 -v
```

---

## 🔬 Attack Details

### RSA Attacks

| Attack | When | Complexity |
|--------|------|------------|
| Small-e | e ∈ {3,5,...} and m^e < n | O(1) |
| Wiener | d < n^0.25 | O(log n) |
| Fermat | p,q close (|p-q| small) | O(√Δ) |
| Hastad | Same m, same e, different n×e | O(1) after CRT |
| FactorDB | n is pre-factored in DB | Network I/O |
| p-1 smooth | p-1 is B-smooth | O(B log n) |
| Common modulus | Same n, different e | O(1) via ext-gcd |

### XOR Key Length Detection

Uses **normalized Hamming distance** between consecutive blocks:

```
NKSD(ks) = mean( hamming_dist(block_i, block_i+1) / ks )
```

Correct key size minimises this value (blocks encrypted with same bytes are more similar).

### Vigenere Key Recovery

1. **Index of Coincidence (IC)** to estimate key length
2. **Chi-squared frequency analysis** per column to find each key byte
3. Falls back to **Kasiski examination** for repeated trigrams

---

## 🧩 Extending the Solver

### Add a Custom Attack

```python
# modules/my_cipher.py
def solve_my_cipher(ciphertext: str) -> Optional[str]:
    ...

# In Crypto-shail.py, add to _try_everything():
from modules.my_cipher import solve_my_cipher
result = solve_my_cipher(raw)
if result:
    self._record("my_cipher", result)
```

### Custom Flag Pattern

```bash
python Crypto-shail.py -f data.txt --flag-pattern "MYCTF\{[^}]+\}"
```

---

## ⚡ Performance Notes

- **Brute-force** (hash, affine): bounded by `--brute-len`; 5-char alphanumeric ≈ 60M candidates
- **BSGS DLOG**: feasible for groups up to ~2^40 (1M baby steps ≈ 100MB RAM)
- **FactorDB/online**: network calls; use `--offline` for air-gapped environments
- **Vigenere**: key lengths up to 20 tried by default; IC reliable for text > 200 chars

---

## 🛡 Responsible Use

This tool is built for **legal CTF competitions** and **educational purposes**.  
Never use it against systems you do not own or have explicit permission to test.

---

## 📜 License

MIT — see [LICENSE](LICENSE)

---

## 🤝 Contributing

PRs welcome! Especially:
- Lattice-based attacks (LLL, coppersmith)
- Stream cipher attacks (RC4, ChaCha)
- PKCS#1 OAEP / PSS edge cases
- Better ECC curve detection
