# PhotonPath

[![Python 3.14+](https://img.shields.io/badge/python-3.14+-blue.svg)](https://www.python.org/downloads/)
[![GUI PySide6](https://img.shields.io/badge/GUI-PySide6%20Qt-00E5FF.svg)](https://wiki.qt.io/Qt_for_Python)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-GitHub%20Pages-00FFB2.svg)](https://kundaastratta.github.io/PhotonPath-App/)
[![Scientific Paper](https://img.shields.io/badge/Scientific%20Paper-PASP%2FA%26A-7B61FF.svg)](https://kundaastratta.github.io/PhotonPath-App/article.html)

**Observational Astronomy Optical-to-Detector Modeling & Dimensioning Framework — "From Photon to Pixel".**

PhotonPath is an open-source, desktop modeling and dimensioning software for amateur and professional astronomers. It guides the preparation of observational campaigns by modeling the complete chain: optics, atmospheric turbulence, focal-plane sampling, sensor physics, full CCD noise budget, and calibration pipeline noise floor.

---

## 🌐 Online Documentation & Interactive Web Tools

- **[Interactive Documentation & Sizing Portal](https://kundaastratta.github.io/PhotonPath-App/)** — Complete offline-ready bilingual (EN/FR) documentation, interactive sizing calculators, and step-by-step generic tutorial.
- **[Peer-Reviewed Scientific Article](https://kundaastratta.github.io/PhotonPath-App/article.html)** — International journal format paper: *"PhotonPath: A Unified Optical-to-Detector Modeling and Dimensioning Framework for Observational Astronomy"*.

---

## 📦 Downloads (v0.1.0)

Pre-built standalone packages requiring zero programming setup:

| Platform | Package | Architecture | Description |
| :--- | :--- | :--- | :--- |
| **macOS** | [**PhotonPath-macOS.zip**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/PhotonPath-macOS.zip) | Apple Silicon & Intel (13+) | Standalone `PhotonPath.app` bundle |
| **Windows** | [**PhotonPath-Windows.zip**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/PhotonPath-Windows.zip) | 64-bit (Win 10/11) | Portable edition, double-click `lancer_photonpath.bat` |
| **Linux** | [**PhotonPath-Linux.zip**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/PhotonPath-Linux.zip) | x86_64 glibc | Native binary with desktop launch script |
| **Python** | [**photonpath-0.1.0-py3-none-any.whl**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/photonpath-0.1.0-py3-none-any.whl) | Pure Python ≥ 3.14 | Install via `pip install ".[gui]"` |

---

## 🔭 The Three Screens

### 1. Screen 1 · Optical Bench & Dual Sampling Verdicts
Models optical geometry, focal ratio, Airy disk, and solves the fundamental **dual-sampling paradox**: reporting both the theoretical **Nyquist diffraction cutoff** in space ($\Delta\theta_{\text{Nyquist}} = \lambda / 2D$) and the **realized atmospheric seeing** under ground turbulence ($\text{FWHM}_{\text{eff}} = \sqrt{\theta_{\text{seeing}}^2 + (\lambda/D)^2}$).

![Screen 1 - Optical Calculator](docs/images/calculator.png)

### 2. Screen 2 · Full CCD Noise Budget & Decision Aid
Integrates the complete CCD equation with target photon Poisson noise, sky background, sensor thermal dark current, electronic readout noise, and atmospheric scintillation. 
Automatically diagnoses the limiting regime among the 5 fundamental regimes (<span style="color:#00FFB2">SHOT_LIMITED</span>, <span style="color:#00E5FF">SKY_LIMITED</span>, <span style="color:#FFAB00">DARK_LIMITED</span>, <span style="color:#FF4444">READ_LIMITED</span>, <span style="color:#7B61FF">SCINTILLATION_LIMITED</span>) and computes the exact optimal sub-exposure duration $t_{\text{opt}}$.

![Screen 2 - Noise Budget](docs/images/noise.png)

### 3. Screen 3 · Calibration Pipeline & Calibration Floor Theorem
Simulates the reduction pipeline $(\text{Light} - \text{Dark}) / \text{Flat}$ with real-time heatmaps. Demonstrates and computes the required calibration frames to prevent master calibration noise from capping final stacked SNR.

![Screen 3 - Calibration Pipeline](docs/images/calibration.png)

---

## ⚡ Quick Start

### macOS
1. Download [**PhotonPath-macOS.zip**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/PhotonPath-macOS.zip).
2. Extract and move `PhotonPath.app` to your `/Applications` folder.
3. On first launch, right-click `PhotonPath.app` → **Open** → confirm **Open**.

### Windows
1. Download [**PhotonPath-Windows.zip**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/PhotonPath-Windows.zip).
2. Extract the archive.
3. Double-click `lancer_photonpath.bat` (or `PhotonPath\PhotonPath.exe`).

### Linux
1. Download [**PhotonPath-Linux.zip**](https://github.com/KundaAstratta/PhotonPath-App/releases/latest/download/PhotonPath-Linux.zip).
2. Extract the archive.
3. Open a terminal and run:
   ```bash
   chmod +x lancer_photonpath.sh
   ./lancer_photonpath.sh
   ```

### Python Wheel (Universal)
```bash
pip install photonpath-0.1.0-py3-none-any.whl[gui]
photonpath            # English interface (default)
photonpath --lang fr  # French interface
```

---

## 🛡️ Core Physical Principles

- **100% Offline & Private:** Zero internet connection required, zero telemetry, zero analytics, zero external web requests.
- **Strict SI Units:** All domain physics compute in pure SI units (meters, seconds, radians, Joules) before presentation conversions.
- **Cross-Platform Numerical Parity:** Validated against double-precision reference benchmarks with zero NaN tolerance.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
