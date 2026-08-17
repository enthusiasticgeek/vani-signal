# vani-signal

Digital signal processing library for the [vāṇī compiler](https://github.com/enthusiasticgeek/vani-compiler).

Depends on [vani-complex](https://github.com/enthusiasticgeek/vani-complex) for
`Complex` arithmetic -- every frequency-domain function takes or returns
`Vec<Complex>`. Time/sample-domain data is plain `Vec<f64>`.

**API reference / tutorial:** <https://enthusiasticgeek.github.io/vani-signal/>

## Add to your project

```toml
# vani.toml
[deps]
signal = { registry = "kosh", version = "^0.1" }
```

```sh
vanic add signal
vanic build
```

## What's included (v0.1.0 — complete; see TODO.md)

| Module | Functions |
|---|---|
| DFT (naive, any length) | `dft`, `idft` |
| FFT (Cooley-Tukey radix-2, power-of-2 length) | `fft`, `ifft`, `fft_real` |
| Spectrum helpers | `dft_magnitude`, `dft_power`, `fft_freq` |
| Padding utilities | `next_power_of_two`, `zero_pad`, `zero_pad_complex` |
| Convolution / correlation | `conv_linear`, `conv_circular`, `correlate` |
| Windowing | `window_rectangular`, `window_hann`, `window_hamming`, `window_blackman`, `apply_window` |
| Numeric transforms | `laplace_transform_numeric`, `z_transform_numeric` |

## fft() vs dft()

`fft`/`ifft`/`fft_real` are the fast O(n log n) Cooley-Tukey radix-2 path and
**require `len(x)` to be a power of 2**. Use `next_power_of_two()` +
`zero_pad()`/`zero_pad_complex()` to prepare arbitrary-length input, or fall
back to the O(n²) `dft`/`idft` (any length -- also useful as a reference
implementation to check `fft` against, which is exactly how this library's
tests validate `fft`).

`z_transform_numeric` evaluated on the unit circle (`|z| = 1`, angle
`2*pi*k/n`) gives the same values as `dft` -- see
`tests/test_transforms.vani` for the cross-check.

## What this library does NOT provide

These are already vāṇī compiler builtins — call them directly, no import needed:

`sin` `cos` `tan` `exp` `log` `sqrt` `abs` `atan2` `f64_pi()` `f64_hypot`
`push` `pop` `len` `set` `vec`

## License

MIT
