<div align="center">

# LCG_Cracker

**Recover every parameter of a Linear Congruential Generator from as few as 6 consecutive outputs.**

![Python](https://img.shields.io/badge/Python-3.9%2B-38E1FF?style=flat-square&labelColor=0A1428&logo=python&logoColor=38E1FF)
![Topic](https://img.shields.io/badge/topic-PRNG%20cryptanalysis-38E1FF?style=flat-square&labelColor=0A1428)

</div>

---

## Overview

An LCG cracker written in pure Python that needs at least 6 outputs from the generator
(original note: *"Pembuatan lcg cracker menggunakan bahasa python dengan minimal input 6 output dari lcg generator"*).

The generator being attacked is

```
s_{i+1} = (x * s_i + c) mod m
```

where `x` is the multiplier, `c` the increment, `m` the modulus and `a` the hidden seed (the state before the first output).
Given the outputs `s_1 .. s_n`, `lcg_attack.attack()` returns `(a, x, m, c)` with nothing else known in advance.

## How the attack works

1. **Build equations.** Each pair of consecutive outputs gives one relation `s_{i+1} = x*s_i + c (mod m)` (`convert`).
2. **Eliminate `c`.** Subtracting two relations removes the increment: `(s_i - s_j) * x = s_{i+1} - s_{j+1} (mod m)` (`eliminationB`).
3. **Eliminate `x`.** Two of those equations are scaled to the LCM of their coefficients and subtracted, leaving an integer that is a multiple of `m` (`eliminationA`).
4. **Recover `m`.** The GCD of all those multiples gives the modulus (`gcdOnList`).
5. **Recover `x`, `c`, seed.** With `m` known, `x` comes from a modular inverse of an invertible coefficient, then `c = s_2 - x*s_1 mod m`, and the seed is `a = (s_1 - c) * x^-1 mod m`.

The author's notes in `lcg_attack.py` state the conditions for it to work well:

- at least 6 outputs are required;
- `m` should be larger than the other parameters (`a`, `x`, `c`); if it isn't, accuracy drops, but recovery can still succeed with some probability;
- more outputs raise the probability of full recovery. The comment estimates about 87% at the 6-output minimum.

## Files

| File | Description |
| --- | --- |
| `lcg_attack.py` | The `lcg_attack` class; `lcg_attack.attack(outputs)` returns `(a, x, m, c)` |
| `test.py` | Benchmark: builds random LCGs (prime parameters of 32/64/128/256 bits, with `m` 5 bits larger), runs 100 cases per size with 6 outputs each, and prints recovery rates for the full set and for each parameter |

## Requirements

- Python 3.9+ (uses `math.lcm` and `pow(x, -1, m)`)
- [`pycryptodome`](https://pypi.org/project/pycryptodome/), only for `test.py` (`Crypto.Util.number.getPrime`)

```bash
pip install pycryptodome
```

## Usage

Run the benchmark:

```bash
python test.py
```

Use it on your own outputs:

```python
from lcg_attack import lcg_attack

outputs = [...]  # at least 6 consecutive LCG outputs
a, x, m, c = lcg_attack.attack(outputs)
```

---

<div align="center">

Part of [wondping0](https://github.com/wondping0)'s cryptography notebooks · [wondping0.github.io](https://wondping0.github.io)

</div>
