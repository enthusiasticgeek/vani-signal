# vani-signal — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `sin` `cos` `tan` `exp` `log` `sqrt` `abs` `atan2` `f64_pi()` `f64_hypot`
> `push` `pop` `len` `set` `vec`
>
> Depends on vani-complex (`Complex`, `complex_*`) -- v0.1.0's only Kosh dependency.

---

## v0.1.0 — Implemented ✓

### Discrete Fourier Transform, naive (2 functions)
- [x] `dft`, `idft` -- O(n^2) direct summation, any length. Reference
      implementation `fft` is checked against in tests.

### Fast Fourier Transform (3 functions)
- [x] `fft` -- Cooley-Tukey radix-2 decimation-in-time, recursive,
      `#[bounded(16)]` (max depth 16, n up to 65536)
- [x] `ifft` -- via the conjugate trick `(1/N) * conj(fft(conj(X)))`, avoids
      a second recursive implementation
- [x] `fft_real` -- lifts `Vec<f64>` to `Vec<Complex>` (im=0) and calls `fft`

### Spectrum helpers (3 functions)
- [x] `dft_magnitude`, `dft_power` -- per-bin `|X|` / `|X|^2`
- [x] `fft_freq` -- bin-to-Hz mapping, standard fftfreq convention (bins past
      n/2 are negative frequencies)

### Padding utilities (3 functions)
- [x] `next_power_of_two` -- smallest power of 2 >= n
- [x] `zero_pad`, `zero_pad_complex` -- right-pad with zeros to a target
      length, for preparing arbitrary-length input for `fft`/`fft_real`

### Convolution and correlation (3 functions)
- [x] `conv_linear` -- full linear convolution, O(n*m), validated against a
      hand-computed 3-tap example
- [x] `conv_circular` -- equal-length circular convolution (direct
      wraparound-index definition, not FFT-based)
- [x] `correlate` -- cross-correlation via `conv_linear(a, reverse(b))`

### Windowing functions (5 functions)
- [x] `window_rectangular`, `window_hann`, `window_hamming`, `window_blackman`
- [x] `apply_window` -- elementwise multiply

### Numeric Laplace and Z transforms (2 functions)
- [x] `laplace_transform_numeric` -- trapezoidal integration of
      `x(t) * e^(-st)` over sampled (possibly non-uniform) time points;
      validated against `L{e^(-2t)}(s) = 1/(s+2)` and `L{1}(s) = 1/s`
- [x] `z_transform_numeric` -- direct summation `sum x[n] * z^(-n)`; validated
      against a hand-computed value AND cross-checked against `dft` on the
      unit circle (composed test, not just isolated -- see
      `tests/test_transforms.vani`)

### Tests and examples
- [x] `tests/test_dft_fft.vani` -- dft/idft, fft/ifft round trips, fft vs dft
      agreement, fft_real, magnitude/power spectra, fft_freq, padding utilities
- [x] `tests/test_convolution_windows.vani` -- conv_linear, conv_circular,
      correlate, all four windows, apply_window
- [x] `tests/test_transforms.vani` -- z_transform_numeric, the dft cross-check,
      laplace_transform_numeric against two closed forms
- [x] `examples/spectrum_demo.vani` -- FFT of a two-tone signal, reports the
      dominant frequency bins
- [x] `examples/filter_and_window_demo.vani` -- moving-average FIR filter via
      convolution, plus a rectangular-vs-Hann spectral-leakage comparison

### Safety annotations
- [x] `#[bounded_stack(bytes=N)]` on every function, budgets set to `vanic
      check`'s exact reported worst-case (largest: `ifft` at 8072 bytes, since
      it composes `_conj_vec` + the recursive `fft` chain)
- [x] `#[bounded(16)]` on `fft` (the only recursive function in this library)

### Compiler quirk found along the way
- [x] Reusing a local variable name (e.g. `out`) in two non-overlapping
      scopes of the same function -- one inside an early-return `if` block,
      one after it -- causes an LLVM backend codegen collision
      (`multiple definition of local value named 'out.addr'`) even though
      the scopes never execute together. Not a vāṇī language bug per se, but
      worth remembering: give locals in sibling/successive blocks distinct
      names rather than relying on scoping alone.

---

## v0.1.4 (2026-07-27)

- [x] `window_bartlett(n)` -- triangular window, `1 - |2i/(n-1) - 1|`.
- [x] `window_kaiser(n, beta)` -- `I0(beta*sqrt(1-(2i/(n-1)-1)^2)) /
      I0(beta)`, via a new private `_bessel_i0` helper (power-series
      evaluation of the zeroth-order modified Bessel function, capped at
      100 iterations with early-exit on convergence -- same pattern as
      `vani-probability`'s `_gamma_reg_series`).
- [x] `window_tukey(n, alpha)` -- tapered-cosine window; `alpha=0` is
      rectangular, `alpha=1` is equivalent to `window_hann` (both checked
      directly in the new test).
- [x] `tests/test_convolution_windows.vani` extended: Bartlett's
      hand-computed triangular shape at n=5; Kaiser (beta=2.0, n=5)
      against reference values from the same I0-series formula evaluated
      in Python; Tukey (alpha=0.5, n=5)'s flat-top/tapered-zero shape,
      plus the alpha=0/alpha=1 identities. `#[bounded_stack(bytes=N)]`
      on all four new functions is `vanic check`'s exact reported
      worst-case. Full suite + `vanic audit-safety` re-verified on both
      backends.

## v0.1.5 (2026-07-27)

- [x] `z_transform_inverse(X_samples, r)` -- recovers a length-n real
      sequence from n samples of `X(z)` taken evenly around a circle of
      radius `r` (e.g. produced by calling `z_transform_numeric` at
      those points). On that circle, the samples are exactly the
      forward DFT of `y[j] = x[j]*r^(-j)`, so `idft` recovers `y`, and
      `x[j] = y[j] * r^j` undoes the scaling; `r = 1` is the ordinary
      IDFT case. `#[bounded_stack(bytes = 512)]`, `vanic check`'s exact
      reported worst-case (chain through `idft`). Added `pow` to the
      module header's known-builtins list (first use in this file).
- [x] `tests/test_transforms.vani` extended with a composed round-trip
      check: sample `X(z)` via `z_transform_numeric`, then recover the
      original sequence via `z_transform_inverse`, for both `r=1` (unit
      circle) and `r=2` (exercises the `r^n` rescaling). Full suite +
      `vanic audit-safety` re-verified on both backends.

## Future

No v0.2.0 is currently planned. Candidates if a concrete need shows up:
higher-radix or mixed-radix FFT (currently radix-2 only, so non-power-of-2
lengths fall back to O(n^2) `dft`), and an FFT-based
`conv_circular`/`conv_linear` fast path for large inputs.
