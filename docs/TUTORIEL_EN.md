# Tutorial · End-to-End Instrumental Dimensioning Walkthrough

> [Français](TUTORIEL.md) · **English** · [Interactive Offline Portal (with EN/FR toggle)](index.html) · [Scientific Paper](article.html)

PhotonPath opens by default on the benchmark parameters of a compact portable instrument (35/150 mm).
This represents an instructive case study: its diffraction Airy disk exceeds typical atmospheric seeing, and its compact sensor operates uncooled.
However, this methodical workflow applies universally to **any astronomical optical train** (apochromatic refractor, Newtonian reflector, Schmidt-Cassegrain, or telephoto lens).

All numbers presented below are exactly those calculated and displayed by the application. By modifying the input parameters to match your own setup, the exact same physical dimensioning applies to any instrument.

## Step 0 — Launch the Application

Install and launch `photonpath` (refer to the [Documentation](DOCUMENTATION_EN.md#installation-and-execution)).
The *Calculator* screen is active by default. No inputs need to be modified for this initial walkthrough.

## Step 1 — Optical Baseline & Spatial Sampling (Calculator)

![Screen 1 · Calculator](images/calculator.png)

Inputs at launch: `35 mm aperture / 150 mm focal length / 550 nm wavelength / 2.0 µm pixel size / 3840 × 2160 pixels`, atmospheric seeing `2.5″`.

| Metric | Computed Value | Physical Meaning |
|---|---|---|
| Focal Ratio | ƒ/4.3 | N = 150 / 35 |
| Airy Radius | 3.95″ · 1.4 px | First zero radius of diffraction pattern |
| Pixel Scale | 2.750″/px | Angular scale on the sky per pixel |
| Field of View | 2.93° × 1.65° | Total true sky coverage |
| Diffraction Sampling | 1.18 px per λ/D · UNDERSAMPLED | Pixel (2.75″) exceeds critical Nyquist pitch (1.62″) |
| Realized Sampling (2.5″ seeing) | 1.5 px per FWHM · UNDERSAMPLED | Realized atmospheric sampling under 2.5″ seeing |

**Scientific Interpretation:**

- The pixel scale (2.75″) is broader than the Nyquist sampling pitch (λ/(2D) = 1.62″). The instrument cannot resolve fine diffraction details on a single exposure, but covers an expansive 2.93° × 1.65° field of view in a single pointing.
- Unique characteristic: the angular resolution element λ/D (3.24″) exceeds the atmospheric seeing disk (2.5″). Hence, the diffraction and realized verdicts closely match. On large apertures, the opposite occurs.

> **Hands-on experiment.** Enter a Newtonian reflector: `200 mm aperture / 1000 mm focal length` with a `3.76 µm` sensor.
> The pixel scale drops to 0.776″/px.
>
> - Diffraction Sampling: **0.73 px per λ/D → UNDERSAMPLED**.
> - Realized Sampling under 2.5″ seeing: **3.3 px per FWHM → OVERSAMPLED**.
>
> The exact same instrument is undersampled in pure diffraction theory and oversampled in practical ground observation. This is the fundamental reason for reporting both verdicts. Reset the inputs back to the DWARF 3 defaults.

## Step 2 — Exposure Planning & Limiting Regimes (Noise)

![Screen 2 · Noise](images/noise.png)

The *Noise* screen inherits the optical geometry from Screen 1. Baseline settings:

- **Magnitude** mode, star magnitude *m* = 15 in Johnson V band;
- Sky surface brightness µ = 20.5 mag/arcsec², optical transmission η = 0.6, detector QE = 0.8;
- Detector dark current *D* = 0.5 e⁻/s, readout noise *R* = 1.5 e⁻, full well 12,000 e⁻;
- Exposure plan: 40 frames × 15 s (600 s = 10 min total).

| Metric | Computed Value | Physical Insight |
|---|---|---|
| Signal-to-Noise Ratio (SNR) | 22.6 | Achieved across the 10-minute stack |
| Dominant Noise Regime | DARK-LIMITED | Sensor thermal generation dominates variance |
| Total Signal · Total Noise | 2,283 e⁻ · 101 e⁻ | Net signal in photometric aperture vs standard deviation |
| Contained Pixels · Enclosed Energy | 15.7 px · 93.6 % | Circular aperture radius *k* = 1.5 |

The variance budget partitions total variance among individual components:

| Target (Shot) | Sky Background | Thermal Dark | Readout Noise | Scintillation |
|---|---|---|---|---|
| 22 % (2,283 e⁻²) | 18 % (1,822 e⁻²) | 46 % (4,698 e⁻²) | 14 % (1,409 e⁻²) | 0 % (disabled) |

**Action Plan:**

- Optimal sub-exposure: **32 s**. The current 15 s sub-exposures still incur an unnecessary readout noise penalty.
- Doubling the SNR requires **160 frames** (40 minutes total integration).
- Detection limit at SNR = 5: magnitude **16.7**.

**Photometric Aperture Optimization:** Open the "?" manual dialog → *Decision aid* → *Aperture radius k*.

- The peak SNR occurs at ***k* ≈ 0.7**, boosting SNR to **31.4** (+39% gain compared to default *k* = 1.5).
- Because the diffraction disk is broad and diffuse on small apertures, an excessively large aperture mostly accumulates dark current and sky background.

| k Factor | Contained Pixels | Enclosed Energy E(k) | Stack SNR |
|---|---|---|---|
| 0.5 | 1.7 px | 57.0 % | 29.2 |
| 0.7 ▲ (Optimum) | 3.4 px | 77.6 % | 31.4 |
| 1.0 | 7.0 px | 87.5 % | 28.4 |
| 1.5 ● (Default) | 15.7 px | 93.6 % | 22.6 |
| 3.0 | 62.6 px | 96.7 % | 12.8 |

## Step 3 — Calibration Frame Planning (Calibration)

![Screen 3 · Calibration](images/calibration.png)

The *Calibration* screen inherits the observing session from Screen 2: *D* = 0.5 e⁻/s, *R* = 1.5 e⁻, 15 s × 40, and a sky background of 2.91 e⁻ per sub-exposure.

| Calibration Frames | Master Dark Noise | Calibrated Stack Noise | Master Variance Share |
|---|---|---|---|
| 16 (Default) | 0.78 e⁻ | 0.96 e⁻ | 66 % |
| **343 (Recommended)** | 0.17 e⁻ | 0.59 e⁻ | 8 % |

**The Calibration Floor Law:**
Because the identical master dark is subtracted from every science sub-exposure, its noise does not decrease with science frame stacking.
If you stack 40 science frames with only 16 dark frames, **two thirds (66%)** of the residual noise in your final image originates from the calibration master itself!
Increasing the calibration stack to 343 frames lowers master dark noise to 0.17 e⁻, reducing calibration floor variance to only 8%.

---
