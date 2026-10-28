# PhotonPath — Documentation

> [Français](DOCUMENTATION.md) · **English** · [Interactive Offline Portal (with EN/FR toggle)](index.html) · [Scientific Paper](article.html)

PhotonPath is a **modeling** and **dimensioning** tool for the optical-to-detector chain in observational astronomy — "from photon to pixel". Before observing at night, it answers five key questions:

- Does my sensor adequately sample the optical image?
- What signal-to-noise ratio (SNR) will I achieve?
- What physical noise source is limiting my observation?
- Will target stars saturate the detector full well?
- How many calibration frames (bias, dark, flat) are mathematically required?

PhotonPath does not control telescope hardware and does not process raw FITS scientific exposures. It is the Python port of the PhotonPath Android application: all numerical values are strictly identical (see [Frequently Asked Questions](#frequently-asked-questions)).

For a step-by-step hands-on guide, read the [Tutorial](TUTORIEL_EN.md).

## Installation and Execution

Prerequisites: **Python 3.14** or newer and [uv](https://docs.astral.sh/uv/).

```bash
cd ~/Code/python/PhotonPath
uv venv --python 3.14 .venv
uv pip install --python .venv/bin/python -e ".[gui]"
.venv/bin/photonpath                 # or: .venv/bin/python -m photonpath
```

CLI Options:

| Option | Effect |
|---|---|
| `--lang en` | Interface in English (default). |
| `--lang fr` | Interface in French. |
| `--version` | Displays version and exits. |

On Linux, Qt requires standard system runtime libraries:
`libegl1 libgl1 libxkbcommon0 libfontconfig1 libdbus-1-3`.

### Standalone Application (Zero Setup)

The PyInstaller standalone bundle packages Python, Qt, and NumPy with zero external installation:

```bash
uv pip install --python .venv/bin/python -e ".[gui,package]"
.venv/bin/pyinstaller packaging/photonpath.spec --noconfirm
open dist/PhotonPath.app             # macOS; dist/PhotonPath/ on Windows/Linux
```

The application icon originates from `src/photonpath/resources/icons/photonpath.svg`. `tools/generate_icon.py` produces the window `.png`, and the `.icns` and `.ico` bundle icons.

## The Interface

![Screen 1 · Calculator](images/calculator.png)

- **Navigation Rail** (left): *Calculator* (optical chain), *Noise* (noise budget), *Calibration*. Keyboard shortcuts ⌘1, ⌘2, and ⌘3 (Ctrl on Linux/Windows) navigate between them.
- **Linked Screens**: Optical geometry cascades from Screen 1 to Screen 2; session parameters cascade from Screen 2 to Screen 3; atmospheric seeing is shared bidirectionally between Screens 1 and 2.
- **Input Column**: Every field displays its measurement unit with a dropdown selector when multiple units exist. Changing units automatically converts the displayed number while preserving the physical SI value.
- **Results Column**: High-contrast metric tiles, color-coded verdict chips, and schematic previews. Results are displayed as long as all required inputs are valid.
- **Adaptive Layout**: Above 720 px width, inputs and results are arranged in a balanced two-column desktop layout. Below 720 px, it collapses into a single scrolling column.
- **"?" Manual Button**: Opens the modal manual dialog displaying exact SI formulas, units, and live values evaluated for the current instrument. The dialog is dismissed via *Close*, Escape, or ⌘W. On Screen 2, the *Decision aid* drawer provides sensitivity curves and action plans.

Screen 3 only computes frame synthesis upon being displayed: this avoids background compute when the tab is hidden.

## Error Messages

| Interface Message | Cause | Example |
|---|---|---|
| Required | Field is empty | `""` |
| Not a number | Unparseable text | `abc` · `1_000` |
| Whole number expected | Fractional value in integer count | 1.5 pixels (while `12.0` is accepted) |
| Must be > 0 | Strict positivity required | focal length = 0 |
| Out of valid range | Outside physical bounds | seeing = 45″ |

- Both decimal points (`.`) and decimal commas (`,`) are accepted.
- Invalid inputs never revert silently to defaults: the associated result tile clears until corrected.

## Conventions to Keep in Mind

- **Catalog Magnitude = Extraterrestrial.** Atmospheric extinction `Δm = k·X` is subtracted only from the celestial target. Sky background is entered *as observed* from the site; a direct measured electron rate is also entered as observed.
- **Seeing = Atmospheric Turbulence Only.** The pipeline quadratically adds the diffraction element λ/D. If you enter the FWHM measured on stars, deconvolve first: `seeing = √(FWHM² − (λ/D)²)`. The Screen 2 manual indicates the aperture threshold where this omission becomes negligible (< 5%).
- **Electrons, Not ADU.** Dark current, read noise, and full well capacity are expressed in physical electrons: `e⁻ = ADU × gain`.
- **Dark Current at Operating Temperature.** Dark current generation roughly doubles every 5–7 °C; no automatic thermal law is applied. Enter the value measured at your true sensor operating temperature.
- **Emission Line Sources (Hα, [O III]).** Extended mode with custom bandwidth assumes a continuum spectral distribution. For pure emission line targets, prefer the *Rate e⁻/s* mode with a calibrated flux rate.
- **Screen 3 Heatmaps:** Numerical metrics are calculated analytically for your specific session; the six heatmap tiles are illustrative (simulated on a fixed 64 × 64 scene).

## Interpreting Verdicts

| Verdict | Physical Meaning | Operational Guidance |
|---|---|---|
| Undersampled | Fewer than 2 pixels per resolution element | Longer focal length or smaller pixels — or dithering + drizzle integration |
| Critical | Within ±5% of Nyquist criterion | Optimal spatial sampling |
| Oversampled | More than 2 pixels per resolution element | Pixel binning or shorter focal length possible |
| Shot-limited | Target Poisson photon statistics dominate | Collect more light: larger aperture or longer total exposure |
| Sky-limited | Sky background Poisson noise dominates | Darker observing site or narrowband filtering; lengthening sub-exposures no longer alters regime |
| Dark-limited | Sensor thermal dark current dominates | Cool detector, regulate temperature, or shorten sub-exposure |
| Read-limited | Readout amplifier electronic noise dominates | Lengthen sub-exposures, or switch to low-noise high-conversion-gain mode |
| Scintillation-limited | Atmospheric wavefront turbulence dominates | Bright star on small aperture. Lengthen sub-exposure (∝ 1/√t), observe near zenith, increase aperture (∝ D^−2/3) |

## Scientific Glossary

### Optics
- **λ/D** — Angular diffraction resolution limit [rad].
- **Airy Disc** — Diffraction pattern of a circular pupil, with first zero radius 1.21967 λ/D.
- **Pixel Scale** — Angle subtended by a pixel on the sky: *s* = *p*/*F*.
- **Nyquist Pitch** — Critical sampling pitch: Δθ<sub>c</sub> = λ / (2*D*).
- **Effective FWHM** — Combined atmospheric seeing and diffraction width: FWHM<sub>eff</sub> = √(θ<sub>seeing</sub>² + (λ/*D*)²).

### Photometry
- **Zero-Point F₀** — Spectral photon flux density of an *m* = 0 star [photons·s⁻¹·m⁻²·m⁻¹].
- **μ** — Diffuse surface brightness [mag·arcsec⁻²].
- **η, QE** — Total optical system transmittance, sensor quantum efficiency.
- **ε** — Linear secondary central obstruction ratio *d*/*D*.
- **k·X** — Atmospheric extinction across airmass *X*.

### Noise
- **CCD Equation** — SNR = *S* / √( *S* + *n*<sub>pix</sub>(*R*<sub>sky</sub> + *D*)*T* + *N* *n*<sub>pix</sub> *R*² + σ²<sub>scint</sub> ).
- **k** — Photometric aperture radius multiplier in units of effective FWHM.
- **E(k)** — Enclosed energy fraction of the composite PSF within aperture *k*.
- **Full Well** — Saturation electron storage capacity per pixel.
- **Optimal Sub-Exposure** — Duration where background noise overwhelms readout noise by 10×.

### Calibration
- **Bias** — Electronic baseline offset.
- **Dark** — Bias + thermal dark accumulation + hot pixels.
- **Flat** — Pixel-to-pixel optical and quantum response non-uniformity.
- **Master** — Stacking average of *N* calibration frames (noise decreases as 1/√*N*).
- **Calibration Floor** — Residual master frame noise that does not diminish when stacking science frames.

## Frequently Asked Questions

### Why does PhotonPath report two sampling verdicts?
Diffraction sampling assumes an aberration-free telescope outside Earth's atmosphere (space telescope or perfect adaptive optics). Ground-based observations are broadened by atmospheric seeing: only the *realized* verdict governs physical resolution. An instrument that is theoretically undersampled in diffraction can be heavily oversampled in practice (e.g., a 200/1000 mm Newtonian provides 0.73 px per λ/D, but 3.31 px per FWHM under 2.5″ seeing).

### Why does SNR drop when enlarging the photometric aperture?
Beyond an optimal radius (*k* ≈ 0.7 in background-limited regimes), enclosed energy *E*(*k*) plateaus near 1, whereas pixel count *n*<sub>pix</sub> continues to grow quadratically as *k*², accumulating background noise.

### Why does calibration require so many dark frames?
The master dark is subtracted identically from every science sub-exposure. Its noise is correlated across the entire stack and does not diminish with science frame stacking. As you stack more science frames, the calibration master must be cleaner; otherwise, the master becomes the noise floor limiting your image.

### Are numerical calculations identical to the Android application?
Yes. A dedicated golden parity test suite validates 1,531 values across 36 scenarios: 1,529 are identical bit-for-bit (0 ULP difference), and the remaining 2 match within 1 ULP (hardware summation ordering).

---
