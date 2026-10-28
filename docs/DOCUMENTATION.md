# PhotonPath — documentation

> **Français** · [English](DOCUMENTATION_EN.md) · [Portail interactif hors-ligne (avec switch EN/FR)](index.html) · [Article scientifique](article.html)

PhotonPath est un outil de **modélisation** et de **dimensionnement** d'une observation
astronomique. Il répond, avant la nuit, à cinq questions :

- Mon capteur échantillonne-t-il bien l'image ?
- Quel rapport signal/bruit vais-je obtenir ?
- Qu'est-ce qui me limite ?
- Mes étoiles vont-elles saturer ?
- Combien de poses de calibration faut-il ?

Il ne pilote aucun instrument et ne traite aucune image réelle. C'est le portage en
Python de l'application Android PhotonPath : tous les chiffres sont identiques (voir
[Questions fréquentes](#questions-fréquentes)).

Pour une prise en main guidée, lire le [tutoriel](TUTORIEL.md).

## Installation et lancement

Prérequis : **Python 3.14** ou plus récent et [uv](https://docs.astral.sh/uv/).

```bash
cd ~/Code/python/PhotonPath
uv venv --python 3.14 .venv
uv pip install --python .venv/bin/python -e ".[gui]"
.venv/bin/photonpath                 # ou : .venv/bin/python -m photonpath
```

Options :

| Option | Effet |
|---|---|
| `--lang fr` | Interface en français (anglais par défaut, comme l'Android). |
| `--version` | Affiche la version et quitte. |

Sous Linux, Qt demande quelques bibliothèques système :
`libegl1 libgl1 libxkbcommon0 libfontconfig1 libdbus-1-3`.

### Application autonome

Le paquet PyInstaller embarque Python, Qt et numpy : il ne demande aucune installation.

```bash
uv pip install --python .venv/bin/python -e ".[gui,package]"
.venv/bin/pyinstaller packaging/photonpath.spec --noconfirm
open dist/PhotonPath.app             # macOS ; dist/PhotonPath/ ailleurs
```

L'icône vient de `src/photonpath/resources/icons/photonpath.svg`, portée des vecteurs
de l'icône Android. `tools/generate_icon.py` en tire le `.png` de la fenêtre, et le
`.icns` et le `.ico` du paquet.

## L'interface

![Écran 1 · Calculator](images/calculator.png)

- **Rail de navigation** (à gauche) : *Calculator* (chaîne optique), *Noise* (budget
  de bruit), *Calibration*. ⌘1, ⌘2 et ⌘3 (Ctrl ailleurs) y mènent aussi.
- **Écrans liés** : l'optique passe de l'écran 1 à l'écran 2, la séance de l'écran 2 à
  l'écran 3, et le seeing est commun aux écrans 1 et 2.
- **Colonne des saisies** : chaque champ porte son unité, avec un menu déroulant quand
  plusieurs unités existent. Changer d'unité convertit la valeur ; la grandeur physique
  ne change pas.
- **Colonne des résultats** : tuiles de valeurs, verdicts colorés et schémas. Les
  résultats n'apparaissent que si toutes les saisies utiles sont valides.
- **Disposition** : au-delà de 720 px de large, saisies et résultats tiennent sur deux
  colonnes. En dessous, sur une seule colonne qui défile.
- **Bouton « ? »** : ouvre la notice de l'écran, avec les formules, les unités et les
  valeurs de l'instrument courant. Bouton *Close*, Échap ou ⌘W la referment. Sur
  l'écran 2, le volet *Decision aid* regroupe les barèmes et les courbes d'arbitrage.

L'écran 3 ne synthétise ses trames qu'une fois affiché : c'est le seul calcul coûteux
de l'application, et il ne tourne pas tant que l'écran est caché.

## Messages d'erreur

| Message (interface) | Cause | Exemple |
|---|---|---|
| Required | Champ vide | "" |
| Not a number | Texte illisible | `abc` · `1_000` |
| Whole number expected | Un comptage reçoit un décimal | 1,5 pixel (mais `12.0` est accepté) |
| Must be > 0 | Grandeur strictement positive attendue | focale = 0 |
| Out of valid range | Hors plage physique | seeing = 45″ |

- La virgule décimale est acceptée partout.
- Une saisie invalide n'est jamais remplacée par une valeur par défaut : le résultat
  disparaît tant que l'erreur reste.

## Conventions à connaître

- **Magnitude de catalogue = hors atmosphère.** L'extinction `Δm = k·X` est
  retranchée de la cible seulement. Le fond de ciel se saisit *tel qu'observé* ; un
  débit mesuré aussi.
- **Seeing = turbulence seule.** Le calcul ajoute λ/D. Si vous saisissez la FWHM
  mesurée sur vos étoiles, déconvoluez d'abord : `seeing = √(FWHM² − (λ/D)²)`. La
  notice de l'écran 2 dit à partir de quel diamètre l'omission devient négligeable
  (moins de 5 %).
- **Électrons, pas ADU.** Courant d'obscurité, bruit de lecture et puits sont en
  électrons : `e⁻ = ADU × gain`.
- **Courant d'obscurité à la température d'usage.** Il double environ tous les
  5–7 °C ; aucune loi thermique n'est appliquée.
- **Source à raies (Hα, OIII).** Le mode *Extended* avec bande personnalisée suppose
  un continuum et sous-estime le signal. Préférer le mode *Rate e⁻/s* avec un débit
  mesuré.
- **Écran 3 :** les chiffres valent pour votre séance ; les images sont
  illustratives (scène 64 × 64 fixe).

## Lire les verdicts

| Verdict | Signification | Levier (texte de l'application) |
|---|---|---|
| Undersampled | Moins de 2 px par élément de résolution | Focale plus longue ou pixel plus petit — ou dithering + drizzle |
| Critical | Proche de Nyquist (± 5 %) | — |
| Oversampled | Plus de 2 px par élément | Binning ou focale plus courte possible |
| Shot-limited | Le bruit vient de la source | Plus de lumière : diamètre, ou temps total |
| Sky-limited | Le fond de ciel domine | Ciel plus sombre ou bande plus étroite ; allonger ne change plus le régime |
| Dark-limited | Le courant d'obscurité domine | Refroidir le capteur, ou raccourcir la pose |
| Read-limited | Le bruit de lecture domine | Sous-poses plus longues, ou gain / caméra moins bruyants |
| Scintillation-limited | Étoile brillante, petit diamètre | Sous-pose plus longue (∝ 1/√t), diamètre (∝ D^−2/3), plus près du zénith |

## Glossaire

### Optique

- **λ/D** — élément de résolution de diffraction.
- **Tache d'Airy** — image d'une étoile par une pupille circulaire, de rayon 1,22 λ/D.
- **Échelle par pixel** — angle vu par un pixel, `p/F`.
- **Nyquist** — au moins 2 échantillons par élément de résolution.
- **FWHM effective** — `√(seeing² + (λ/D)²)`.

### Photométrie

- **Point zéro F₀** — flux de photons d'une source de magnitude 0, par m² et par
  mètre de bande.
- **μ** — brillance de surface, en mag/″².
- **η, QE** — transmission de l'optique, rendement quantique.
- **ε** — obstruction centrale linéaire.
- **k·X** — extinction atmosphérique.

### Bruit

- **Équation CCD** — `S/B = S / √(S + n_pix·(ciel + D)·T + N·n_pix·R²)`.
- **k** — rayon de l'ouverture photométrique, en FWHM.
- **E(k)** — fraction du flux dans l'ouverture.
- **Puits** — électrons maximum d'un pixel.
- **Sous-pose optimale** — quand le fond noie la lecture (×10).

### Calibration

- **Bias** — offset de lecture.
- **Dark** — bias + courant d'obscurité + pixels chauds.
- **Flat** — réponse pixel à pixel.
- **Master** — moyenne de N poses (bruit ÷ √N).
- **Plancher** — bruit des masters, qui ne baisse pas à l'empilement.

## Questions fréquentes

### Pourquoi deux verdicts d'échantillonnage ?

Celui de diffraction suppose une optique parfaite sans atmosphère : télescope
spatial, optique adaptative. Au sol, le seeing élargit l'image : seul le verdict
« réalisé » compte. Un « sous-échantillonné » de diffraction peut être
« sur-échantillonné » en pratique : un Newton 200/1000 donne 0,73 px par λ/D, mais
3,31 px par FWHM.

### Pourquoi le S/B baisse-t-il quand j'agrandis l'ouverture photométrique ?

Au-delà d'un certain rayon, `E(k)` plafonne à 1 alors que `n_pix` croît en k² : on
ramasse surtout du fond. Le maximum dépend du régime de bruit ; la courbe S/B(k) de
la notice le situe.

### Pourquoi l'application demande-t-elle des centaines de darks ?

Le même master est soustrait de chaque pose : son bruit ne s'atténue pas à
l'empilement. Plus vous empilez, plus le master doit être propre. Un nombre
impraticable signale qu'il faut agir sur le capteur : le refroidir, ou changer de gain.

### Les chiffres sont-ils identiques à ceux de l'application Android ?

Oui pour tout ce qui est calculé. Un test compare 1 531 grandeurs, issues de 36
scénarios, à une référence exportée par l'Android :

- 1 529 sont identiques au bit près ;
- les deux autres diffèrent d'une unité sur le dernier bit.

La vignette d'étoile simulée de l'écran 2 est la même, car le générateur de
`java.util.Random` est porté bit pour bit.

Seules les trames illustratives de l'écran 3 diffèrent : elles sont tirées par le
générateur de numpy. Il en va donc de même pour la non-uniformité, mesurée sur ces
trames, qui est marquée *illustrative scene*. Les autres métriques sont analytiques
et identiques : bruit du master, bruit final, plancher et nombre de poses recommandé.

## Guide du développeur

```bash
uv pip install --python .venv/bin/python -e ".[gui,dev]"
.venv/bin/ruff check . && .venv/bin/ruff format --check .   # style
.venv/bin/mypy                                              # types (strict)
.venv/bin/lint-imports                                      # règles de dépendance
.venv/bin/pytest                                            # tout, interface comprise (sans écran)
.venv/bin/pytest -m parity                                  # parité Android seule
.venv/bin/pytest -m gui                                     # interface seule
.venv/bin/python tools/check_consistency.py                 # documentation ↔ code
.venv/bin/python tools/capture_screens.py                   # captures de cette documentation
```

Les conventions complètes sont dans [AGENTS.md](../AGENTS.md) et les décisions
d'architecture dans [ARCHITECTURE.md](../ARCHITECTURE.md).

### Recette · ajouter une formule

1. Écrire une fonction pure dans `domain/physics/`, avec entrées et sorties en SI. Sa
   docstring donne la formule, les unités, la référence et le domaine de validité.
2. Ajouter une entrée dans `PHYSICS.md`, avec un cas-test chiffré.
3. Écrire le test de référence (priorité : Léna et al.).
4. Câbler la formule : cas d'usage → UiState → écran.
5. Faire de même côté Android, puis régénérer `reference_android.json`.

### Recette · ajouter un écran

1. Créer `presentation/<ecran>/` avec `ui_state.py` et `view_model.py` (sans Qt),
   `screen.py` et `components/`.
2. Ajouter les modèles et les cas d'usage dans `domain/`.
3. Construire le ViewModel dans `container.py`, puis déclarer l'écran dans
   `build_window` (`app.py`) : le rail le prend en charge.
4. Fournir les chaînes EN et FR (importées des `strings.xml` Android).
5. Réutiliser `common/` et `theme/` : aucune couleur en dur.
