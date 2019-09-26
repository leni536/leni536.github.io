---
layout: post
title:  "Implementations for Gray code encoding and decoding"
date:   2019-09-26 10:49:00 +0200
---

# Gray code encoding

Gray code encoding is a bijective function between natural numbers (including 0) in a way that the Gray code of subsequent integers only differ in a single binary digit.
This property doesn't uniquely define this function.
I use the following common definition:

$$
    \newcommand{\xor}{\wedge}
    \newcommand{\rshift}{\gg}
    \renewcommand{\neg}{\mathord{\sim}}
    a: \mathcal{N} \rightarrow \mathcal{N} \\
    a_i = i \xor (i \rshift 1),
$$

where $$\xor$$ denotes the ,,bitwise exclusive or'' and $$\rshift$$ denotes the ,,bitwise right shift'' operators.

# Gray code decoding

I denote the inverse of the function $$a$$ with $$b$$.
$$b_i$$ can be expressed in the following way:

$$
    b_i 
    = \begin{cases}
        0 & i = 0 \\
        i 
        \xor (i\rshift1)
        \xor (i\rshift2)
        \xor\dots\xor
        (i\rshift \lfloor \log_2 i \rfloor)
        & i > 0 \\
      \end{cases}
$$

It can be shown that $$b$$ is indeed the inverse of $$a$$ using the commutative and associative properties of $$\xor$$ and the following identities:

$$
    x \xor x = 0 \\
    x \xor 0 = x \\
    (x \xor y)\rshift n = (x \rshift n)\xor(y \rshift n) \\
    (x \rshift n) \rshift m = x \rshift (n+m) \\
    x \rshift (\lfloor \log_2 x \rfloor + 1) = 0
$$

# Implementations

Implementing Gray code encoding is rather simple:

```c
unsigned gray_encode(unsigned i) {
    return i^(i>>1);
}
```

One can implement Gray code decoding directly from the defining expression of $$b_i$$:

```c
unsigned gray_decode(unsigned i) {
    unsigned ret = 0;
    while (i != 0) {
        ret = (ret ^ i);
        i = (i >> 1);
    }
    return ret;
}
```

One can actually do better for a known fixed-width integral type.
The following algorithm is from [Wikipedia](https://en.wikipedia.org/wiki/Gray_code#Converting_to_and_from_Gray_code):

```c
uint32_t gray_decode(uint32_t i) {
    i ^= (i >> 16);
    i ^= (i >> 8);
    i ^= (i >> 4);
    i ^= (i >> 2);
    i ^= (i >> 1);
    return i;
}
```

One can show the correctness of this implementation using the identities I presented before and induction.

I also developed an alternative implementation that makes use of the `popcnt` and `pdep` x86_64 CPU instructions (part of the POPCNT and BMI2 extensions respectively).

```c
#include <x86intrin.h>
uint32_t gray_decode(uint32_t i) {
    uint32_t evens = _pdep_u32(0x55555555u,i);
    uint32_t odds  = _pdep_u32(0xAAAAAAAAu,i);
    uint32_t popcount = __builtin_popcount(i);
    return (~(-(popcount & 1)))
            ^ ((evens << 1) + (~(odds << 1)));
}
```

In the following sections I break down how this implementation works.
I stick with 32 bit numbers, but my reasoning works for any bitwidth.

If $$i = \mathbf{x}_{31} \mathbf{x}_{30} \dots \mathbf{x}_1 \mathbf{x}_0$$ 
  and $$b_i = \mathbf{y}_{31} \mathbf{y}_{30} \dots \mathbf{y}_1 \mathbf{y}_0$$ -- 
  where the bold letters denote binary digits -- then $$ \mathbf{y}_n = \mathbf{x}_n \xor \mathbf{x}_{n+1} \xor \dots \xor \mathbf{x}_{31}$$.
In other words one can get the $$n$$th digit in $$b_i$$ by xoring the digits from the $$n$$th to the most significant digit in $$i$$.
For a given $$i$$ where the 1 digits are sparsely laid out one can think of $$b_i$$ in the following way:

$$
\begin{align}
    i &= \verb|…010…010…010…010…| \\
  b_i &= \verb|…011…100…011…100…| \\
\end{align}
$$

The leading digits (from the most significant digits) are 0 up until the first 1 digit in $$i$$.
Then the following digits in $$b_i$$ are all 1 up until the next 1 digit in $$i$$, and so on.
The 1 digits in $$i$$ mark the transitions between strips of 0s and 1s in $$b_i$$.
For this description it's not required that the 1s in $$i$$ are sparsely laid out,
  it just makes the intuition easier (for me at least).

I define $$e_i$$ by taking every second 1 binary digit in $$i$$ starting from the least significant one.
I also define $$o_i$$ by taking every second 1 binary digit in $$1$$ starting from the second least significant one.
$$e_i$$ and $$o_i$$ correspond to the `evens` and `odds` variables in my `gray_decode` function above.

$$
\begin{align}
    i &= \verb|…010…010…010…010…| \\
  e_i &= \verb|…000…010…000…010…| \\
  o_i &= \verb|…010…000…010…000…| \\
\end{align}
$$

After adding $$e_i$$ and $$\neg o_i$$, where $$\neg$$ denotes bitwise negation, the result has similar strips of 0s and 1s as $$b_i$$:

$$
\begin{align}
           i &= \verb|…010…010…010…010…| \\
         e_i &= \verb|…000…010…000…010…| \\
    \neg o_i &= \verb|…101…111…101…111…| \\
e_i+\neg o_i &= \verb|…110…001…110…001…| \\
\end{align}
$$

There are two main differences to $$b_i$$:

1. The transitions between the strips of 0s and 1s start one place to the right compared to $$b_i$$.
2. The first strips from the left can be all 1s or all 0s depending if there are even or odd number of 1 digits in $$i$$.

The first issue can be fixed by shifting $$e_i$$ and $$o_i$$ to the left by one place beforehand.
The second difference can be addressed by first calculating if there are even or odd number of 1 bits in $$i$$.
Then one can flip the bits in the result for the even case.

I denote the number of 1 bits in $$i$$ as $$p_i$$, this corresponds to the `popcnt` variable in my `gray_decode` function.
By applying these fixes we get the following expression for $$b_i$$:

$$
    b_i = \begin{cases}
        (e_i \ll 1) + \neg(o_i \ll 1) & p_i\text{ is odd} \\
        \neg((e_i \ll 1) + \neg(o_i \ll 1)) & p_i\text{ is even} \\
    \end{cases}
$$

The branching between the even and odd cases can also be eliminated by xoring the intermediate result with `00…0` or `11…1` depending on the parity of $$p_i$$.
$$p_i \mathbin{\&} 1$$ is either 0 or 1 , $$-(p_i \mathbin{\&} 1)$$ is either `00…0` or `11…11` for the even and odd cases respectively.
Since bitwise negation is required for the even case an additional bitwise negation is necessary:

$$
    b_i = (\neg(-(p_i \mathbin{\&} 1))) \xor ((e_i \ll 1) + \neg(o_i \ll 1))
$$

The `popcnt` and `pdep` CPU instructions can be used to calculate $$e_i$$, $$o_i$$ and $$p_i$$ efficiently.

The `popcnt` instruction takes one source and one destination argument.
It writes the number of 1 binary digits in the source and returns the count in the destination.
$$p_i$$ can be calculated by applying `popcnt` on $$i$$.

`pdep` takes two source arguments (`src` and `mask`) and one destination argument (`dst`).
It takes the low bits from `src` and deposits them in the destination to the corresponding set bit locations in `mask`.

![pdep figure](/assets/pdep.svg)
{: style="text-align: center;"}

One can calculate $$e_i$$ by using `pdep` and setting `src` to `0101…01` and `mask` to $$i$$.
$$o_i$$ can be calculated similarly with `pdep` but with `1010…10` as `src`.
This is how `evens` and `odds` are calculated in my `gray_decode` function, the magic numbers are the `src` arguments.

# Benchmark

I think there is no correct way to objectively benchmark these Gray code decoding functions in isolation.
One can call these functions in a tight loop and compare their throughput that way, but one shouldn't take the resulting numbers too seriously. 

I used Google's benchmark library and the following setup:

```c++
static void Generic32(benchmark::State& state) {
  uint32_t i = 0;
  for (auto _ : state) {
    ++i;
    uint32_t x = gray_decode_generic(i);
    benchmark::DoNotOptimize(x);
  }
  state.SetBytesProcessed(i*sizeof(uint32_t));
}
BENCHMARK(Generic32);
```

And similarly for all the other functions for both `uint32_t` and `uint64_t`.
I compiled the benchmark with `gcc -O2`.

```
-------------------------------------------------
Benchmark          Time           CPU Iterations
-------------------------------------------------
Generic32          2 ns          2 ns   58150989   1.56283GB/s
PDEP32             2 ns          2 ns   82701968   2.18142GB/s
Generic64          2 ns          2 ns   75010161   4.67384GB/s
PDEP64             1 ns          1 ns  110827182   5.80519GB/s
```

The throughput of the `popcnt` and `pdep` method is somewhat faster in this specific benchmark.

# Notes

1. $$b_i$$ can also be calculated using [carry-less multiplying](https://en.wikipedia.org/wiki/Carry-less_product) $$i \ll 1$$ with `11…1` and fixing the result with `popcnt`.
The PCLMUL extension provides this operation, but it can only carry-less multiply 64bit arguments and it still requires XMM registers.
The performance of this method wasn't great the last time I tried.
There can easily be a CPU instruction that could help beating my method at least in a streaming case, but I'm not familiar with all of the vector instructions.
2. Fun fact: $$b_i \mathbin{\&} 1$$ is the [Thue--Morse sequence](https://en.wikipedia.org/wiki/Thue%E2%80%93Morse_sequence)
3. I use a variation of my algorithm in my [fast Hilbert-curve](https://https://github.com/leni536/fast_hilbert_curve) library.
For my purposes I don't need to shift $$e_i$$ and $$o_i$$.
