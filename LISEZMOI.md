<!--
================================================================================
 Nom          : Khéops - LISEZMOI (dépôt GitHub One-Page-One-Mission)
 Fichier      : LISEZMOI.md
 Version      : 4.3
 Date         : 17/10/2026
 Auteur       : Eric PERRET
 Objet        : présentation, algorithmes et choix arbitrés du simulateur
                kheops_v43.html (version anglaise : README.md)
================================================================================
-->

# 🎯 One-Page-One-Mission — Khéops v4.3

▶️ **[Lancer le simulateur](https://ericperret.github.io/One-Page-One-Mission/kheops_v43.html)** — la simulation démarre seule au chargement.

[![Licence : CC BY-NC 4.0](https://img.shields.io/badge/Licence-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
[![HTML/CSS/JS pur](https://img.shields.io/badge/Pur-HTML%2FCSS%2FJS-blue.svg)](#technologie)
[![Aucune librairie](https://img.shields.io/badge/Librairies-aucune-green.svg)](#technologie)
[![Un seul fichier](https://img.shields.io/badge/Fichier-unique-orange.svg)](kheops_v43.html)

**Simulateur de goniométrie acoustique passive de drone** par écarts de temps d'arrivée (TDOA) sur une pyramide de 5 micros, avec caméra d'identification orientée par l'acoustique.

Le simulateur sert à **dimensionner la base de la pyramide** : il trace le *temps de mise au point à 1°* en fonction du côté de la pyramide. La démonstration temps réel autour de cette étude est volontairement ludique.

🇬🇧 English version: [README.md](README.md)

![Khéops v4.3 en fonctionnement](screenshot.png)

---

## Sommaire

1. [Ce que fait Khéops](#1-ce-que-fait-khéops)
2. [Utilisation](#2-utilisation)
3. [Résultat de l'étude de dimensionnement](#3-résultat-de-létude-de-dimensionnement)
4. [Architecture : vérité terrain et estimateur séparés](#4-architecture--vérité-terrain-et-estimateur-séparés)
5. [Modèle physique (vérité terrain)](#5-modèle-physique-vérité-terrain)
6. [Chaîne de mesure : ADC 1 MHz et GCC-PHAT](#6-chaîne-de-mesure--adc-1-mhz-et-gcc-phat)
7. [Estimateur](#7-estimateur)
8. [Caméra d'identification](#8-caméra-didentification)
9. [Étude de dimensionnement](#9-étude-de-dimensionnement)
10. [Choix arbitrés](#10-choix-arbitrés)
11. [Limites connues](#11-limites-connues)
12. [Technologie](#technologie)
13. [Historique](#historique)

---

## 1. Ce que fait Khéops

| Fonction | Méthode |
|---|---|
| Mesure | Écarts de temps d'arrivée sur les 10 paires de micros, GCC-PHAT sur une chaîne ADC 16 bits à 1 MHz simulée échantillon par échantillon |
| Direction | Normale au front d'onde, ajustée par Levenberg-Marquardt, aberrants rejetés par RANSAC sur les 125 arbres couvrants |
| Distance | Courbure du front d'onde, écarts de niveau entre micros (1/r), retard de l'écho sol sur le micro du sommet — **jamais** le niveau absolu |
| Trajectoire | Moindres carrés sur horizon glissant (1 s), modèle à accélération constante, retard de propagation intégré, pondération de Huber — **aucun filtre récursif** |
| Atmosphère | Vitesse du son ISO 9613-1 / CIPM-2007 à partir de la météo en direct au point de Khéops |
| Vent | Profil logarithmique, propagation exacte en vent uniforme, rafales réelles de la station |
| Bruit de fond | LA90 sur 30 s mesuré au micro de l'appareil (si autorisé), sinon 35 dB(A) |
| Caméra | Orientée uniquement par l'estimation acoustique ; drone « identifié » après 3 s d'alignement continu à moins de 1° |
| Étude | Temps de mise au point à 1° pour 8 tailles de pyramide × 40 passages reproductibles |

Khéops est un système **d'observation** : il n'y a ni arme ni conduite de tir.

---

## 2. Utilisation

1. Ouvrir `kheops_v43.html` dans un navigateur récent (Chrome, Edge, Firefox) — en local (`file://`) ou par le lien GitHub Pages.
2. La simulation démarre automatiquement. Le navigateur demande l'accès au micro : il sert **uniquement** à mesurer le bruit de fond (LA90). En cas de refus, le bruit de fond vaut 35 dB(A). Si le navigateur suspend l'audio au chargement, la mesure reprend au premier clic.
3. **Clic sur la carte** : déplace Khéops à cet endroit. La météo est relue dès que le déplacement dépasse 2 km.
4. Onglet **Étude** → **Lancer l'étude** : calcule la courbe de dimensionnement (environ 2 minutes dans un navigateur de bureau).
5. Onglets **Direction** et **Distance** : erreurs cumulées de la démonstration par tranche de distance ; infobulle au survol.
6. **PAUSE / RESET / RESET STATS** en bas de l'écran. Les panneaux de droite se replient ; leur état est mémorisé dans le navigateur.

**Aucun réglage n'est proposé.** Tout ce qui est réglable dans la réalité vient de la météo, du micro ou du clic sur la carte.

Bandeau supérieur, moyennes sur les drones identifiés :

| Indicateur | Définition |
|---|---|
| Identifiés | Nombre de drones identifiés depuis le dernier RESET STATS |
| T. align. moy. | Temps entre le début du suivi et le premier alignement à moins de 1° |
| T. suivi moy. | Durée de suivi jusqu'à l'identification |
| Aligné moy. | Part du temps de suivi passée alignée à moins de 1° |

Connexions externes (les seules) :

| Service | Usage | Hors ligne |
|---|---|---|
| Esri World Imagery | Fond de carte (zoom natif ≤ 17) | carte vide, simulation intacte |
| CARTO `dark_only_labels` | Étiquettes de la carte | sans étiquettes |
| open-meteo.com | Température, humidité, pression sol, vent et rafales à 10 m ; relu toutes les 15 min | atmosphère standard (20 °C, 50 %, 1013,25 hPa), vent nul |

---

## 3. Résultat de l'étude de dimensionnement

Deux exécutions indépendantes de l'étude dans le navigateur (8 tailles × 40 passages, mêmes graines de trajectoire ; météo simulée pour le test : vent 22 km/h, rafales 38,5 km/h, 28 °C).

| Côté (m) | Focus médian (s) A / B | Focus 90 % (s) A / B | Mis au point | Bruit ADC / modèle A / B |
|---:|:---:|:---:|:---:|:---:|
| 1 | 2,48 / 2,50 | 5,38 / 5,60 | 38/40 | ×1,05 / ×1,04 |
| 2 | 2,36 / 2,34 | 5,74 / 5,56 | 38/40 | ×1,06 / ×1,01 |
| 3 | 2,38 / 2,30 | 5,70 / 5,02 | 38/40 | ×1,06 / ×0,95 |
| 5 | 2,06 / 1,98 | 5,58 / 3,92 | 38/40 | ×1,12 / ×1,22 |
| 7 | 1,98 / 2,10 | 3,12 / 6,82 | 38/40 | ×1,04 / ×1,34 |
| 10 | 2,08 / 2,10 | 5,66 / 4,24 | 38/40 | ×1,68 / ×1,54 |
| 15 | 2,10 / 1,86 | 5,50 / 5,16 | 38/40 | ×2,15 / ×1,95 |
| 20 | 1,78 / 1,82 | 4,76 / 3,96 | 38/40 | ×4,68 / ×3,23 |

![Courbe temps de focus à 1° selon le côté de la pyramide](study.png)

**Lecture :**

- **Médiane** : elle baisse de 2,5 s (1 m) à 2,0 s (5 m), puis reste entre 1,8 et 2,1 s jusqu'à 20 m. La tendance est la même dans les deux essais.
- **90 %** : entre 3 et 7 s sans tendance nette ; 40 passages ne suffisent pas à stabiliser ce percentile (il repose sur 4 passages).
- **Échecs** : 2 passages sur 40 ne sont jamais mis au point, à toutes les tailles et dans les deux essais ; la cause est la trajectoire (identique d'une taille à l'autre), pas la base.
- **Bruit de corrélation** : il croît avec la taille (×1,0 à 3 m, ×1,5–1,7 à 10 m, ×3,2–4,7 à 20 m). Sur une grande base, l'écart entre micros dérive pendant les 16,4 ms d'intégration (source mobile) et le pic de corrélation s'étale : la géométrie s'améliore, la mesure se dégrade.
- **Conclusion** : l'essentiel du gain est acquis à **5 m** ; de 5 à 10 m le temps de focus ne bouge plus et le bruit de mesure reste modéré (≤ ×1,7) ; au-delà de 10 m, le gain sur la médiane (≤ 0,3 s) est payé par un bruit de corrélation multiplié par 2 à 5. **Base recommandée : 5 à 10 m de côté.**

---

## 4. Architecture : vérité terrain et estimateur séparés

Le code est découpé en deux mondes qui ne partagent que la mesure :

```
VÉRITÉ TERRAIN (ce qui se passe)            ESTIMATEUR (ce que Khéops sait)
───────────────────────────────            ────────────────────────────────
drone : trajectoire, manœuvres, niveau      météo de la station (moyenne)
atmosphère réelle = station + écart local   vitesse du son de la station
vent réel = moyenne + rafales               vent moyen de la station
bruit réel = modèle × facteurs inconnus     modèle de bruit nominal
                │                                       ▲
                └──── mesure : 10 TDOA, 5 niveaux, ─────┘
                      retard d'écho, SNR, décalages
```

L'estimateur **ne lit jamais** `sim.drone`, l'atmosphère réelle, les rafales, le facteur de bruit du drone, ni son niveau source. La position vraie n'est utilisée que pour juger l'alignement de la caméra et tracer les statistiques. C'est ce qui permet d'affirmer que la simulation est « sans triche ».

| Grandeur | Vérité terrain | Connu de l'estimateur |
|---|---|---|
| Température, humidité, pression | station + (−0,3 °C ; +3 % HR ; −0,8 hPa) | station |
| Vent | moyenne + rafales de la station (Poisson 0,1 /s, 1 à 4 s) | moyenne |
| Bruit TDOA | modèle × étalonnage ADC × facteur du drone (×0,8 à ×1,25) | estimé à partir des résidus |
| Niveau source | 79 dB ± 3 dB par drone, +2 dB en manœuvre, ±1 dB commun par mesure | inconnu (n'est pas utilisé) |
| Écart entre micros | 0,2 dB indépendants par micro | σ supposé 0,25 dB |

---

## 5. Modèle physique (vérité terrain)

### 5.1 Réseau de micros

Pyramide à base carrée de côté **a** : 4 micros au sol aux coins, 1 micro au sommet au-dessus du centre, à la hauteur **h = 0,636·a** (proportions de la pyramide de Khéops). Les 10 paires (i < j) sont toutes exploitées. Le côté **a** est le seul paramètre de dimensionnement ; la démonstration utilise a = 5 m.

### 5.2 Vitesse du son (ISO 9613-1 / CIPM-2007)

- Pression de vapeur saturante (ISO 9613-1) : $p_{sat} = p_r \cdot 10^{C}$, $C = -6{,}8346\,(T_{01}/T)^{1{,}261} + 4{,}6151$, $T_{01} = 273{,}16$ K, $p_r = 101\,325$ Pa.
- Facteur d'augmentation (CIPM-2007) : $f = 1{,}00062 + 3{,}14\cdot10^{-8}\,p + 5{,}6\cdot10^{-7}\,t^2$.
- Fraction molaire d'eau : $x_w = \frac{HR}{100}\, f\, p_{sat} / p$ ; CO₂ : $x_c = 420$ ppm.
- Masse molaire du mélange : air sec sans CO₂, CO₂ et eau pondérés par leur fraction molaire (air sec de référence CIPM-2007 à 400 ppm, CO₂ retiré pour ne pas le compter deux fois).
- Rapport γ du mélange à partir des $c_v$ de l'air sec (γ = 1,40), de l'eau (1,33) et du CO₂ (1,30).
- $c = \sqrt{\gamma R T / M}$. La formule de Cramer (JASA 93, 1993) est calculée en parallèle et l'écart est affiché.

### 5.3 Vent

- Profil logarithmique : $U(z) = U_{10}\,\ln(z/z_0)/\ln(10/z_0)$, $z_0 = 0{,}1$ m (herbe, cultures basses).
- Vent vu par un trajet : moyenne exacte du profil entre l'altitude de la source et celle du réseau, $\overline{\ln(z/z_0)} = [F(b)-F(a)]/(b-a)$ avec $F(z) = z\ln(z/z_0) - z$.
- Propagation exacte en vent uniforme **w** : le temps de trajet τ vérifie $|\mathbf d - \mathbf w\tau| = c\,\tau$, soit
  $\tau = \dfrac{-\mathbf w\cdot\mathbf d + \sqrt{(\mathbf w\cdot\mathbf d)^2 + (c^2 - |\mathbf w|^2)\,|\mathbf d|^2}}{c^2 - |\mathbf w|^2}$.
- Vent réel : direction et vent moyen de la station ; rafales à la valeur de rafale de la station, déclenchées par un processus de Poisson (0,1 /s), durée 1 à 4 s. Aucune autre variation n'est inventée.

### 5.4 Drone

- Apparition à 142,5 m (0,95 × rayon de la bulle de 150 m), altitude 25 à 55 m, cap vers Khéops ± 22,5°, vitesse 5 à 18 m/s.
- Manœuvres : processus de Poisson 0,5 /s, accélération latérale 0,5 à 2 g (composante verticale aléatoire), durée 0,5 à 2 s, retour à accélération nulle avec τ = 0,19 s.
- Historique de position à 50 Hz sur 5 s : l'instant d'émission retardé $t_e = t - \tau$ est résolu par itération sur la trajectoire vraie interpolée.

### 5.5 Acoustique

- Niveau source « Consumer 600 g » : 79 dB à 1 m ; ± 3 dB par drone (dispersion de modèle et d'état) ; + 2 dB à 2 g de manœuvre ; fluctuation commune ± 1 dB par mesure.
- Niveau reçu par micro : $L_i = L_s - 20\log_{10} r_i - \alpha_w r_i$ ; atténuation due au vent $\alpha_w$ : −4 dB/100 m sous le vent, +5 dB/100 m contre le vent, pour 50 km/h (proportionnelle, nulle sous 5 km/h).
- Seuil de détection : SNR moyen des 5 micros ≥ 3 dB au-dessus du bruit de fond.
- Bruit de fond : LA90 sur 30 s, pondération A (CEI 61672) appliquée au spectre, sensibilité MEMS −26 dBFS à 94 dB SPL, borné 20–90 dB(A) ; 35 dB(A) sans micro.
- Écho sol : sol plan, coefficient de réflexion ρ = 0,5 (herbe rase, terre). Seul le micro du sommet reçoit un écho distinct ; pour les micros posés au sol, son direct et réfléchi arrivent ensemble. Retard : $\tau_{écho} = \tau(\mathbf p', \mathbf x_5) - \tau(\mathbf p, \mathbf x_5)$ avec $\mathbf p' = (x, y, -z)$, soit environ $2h\sin(\text{site})/c$.

### 5.6 Signal source (chaîne ADC)

- Raies : fréquence de passage des pales 4 × 10 000 tr/min / 60 = 666,7 Hz, 5 harmoniques en 1/h.
- Large bande : bruit de phase aléatoire, amplitude $1/\sqrt{f/500 + 1}$, bande 100–8000 Hz, table périodique de 0,52 s.
- Puissances égales raies / large bande.

---

## 6. Chaîne de mesure : ADC 1 MHz et GCC-PHAT

### 6.1 Ce que mesure une corrélation

La corrélation d'une paire (i, j) prend un **segment** de la voie j et cherche son décalage dans le **tampon** de la voie i. Elle mesure donc l'arrivée en $\mathbf x_i$ du front d'onde qui arrive en $\mathbf x_j$ au centre du segment :

$$\Delta_{ij} = \tau_i(\mathbf p_j) - \tau_j(\mathbf p_j)$$

où $\mathbf p_j$ est le point d'émission du son reçu en $\mathbf x_j$ à cet instant. Chaque paire a donc son propre point d'émission. L'ancienne convention « même instant de réception sur tous les micros » faussait les écarts d'un facteur v/c (jusqu'à 5 % à 18 m/s) ; vérité terrain, estimateur et étalonnage utilisent tous la convention ci-dessus depuis la v4.2.

### 6.2 Acquisition

| Étape | Valeur |
|---|---|
| Échantillonnage | 1 MHz, 16 bits, pleine échelle ajustée à 4 σ du signal |
| Retards | variables dans le tampon (source mobile : retard et dérivée par voie), interpolation linéaire dans les tables du signal |
| Bruit | blanc gaussien sur 0–500 kHz, réglé pour le SNR mesuré dans la bande utile |
| Décimation | ×16 par moyenne de bloc → 62,5 kHz |
| Segment d'intégration | 1024 échantillons décimés = 16,4 ms |
| Tampon | segment + 2 × retard maximal + 16, retard maximal = ⌈√2·a / 300 · fs⌉ + 4 |
| FFT | radix-2 en place, taille = puissance de 2 ≥ tampon |

### 6.3 GCC-PHAT

$R_{ij}(\ell) = \mathcal F^{-1}\!\left[\dfrac{X_i\,S_j^*}{|X_i\,S_j^*|}\right]$ sur la bande 100–8000 Hz, $X_i$ : spectre du tampon de i, $S_j$ : spectre du segment de j. Recherche du pic sur les décalages valides (le segment décalé doit rester dans le tampon), puis interpolation parabolique sur 3 points (correction bornée à ± 0,5 échantillon).

### 6.4 Segments calés par la piste

Quand une trajectoire est suivie, chaque voie j décale son segment de $s_j$ (en échantillons décimés entiers) pour que toutes les voies corrèlent **le même front d'onde** : celui qui arrive au centre du réseau à l'instant de la mesure.

$s_j = t_e^{(c)} + \tau(\mathbf p_c, \mathbf x_j) - t_{rx}$

Sans piste, les segments sont centrés et l'écart de convention est pris comme incertitude supplémentaire (§ 7.1).

### 6.5 Écho par cepstre

Cepstre de puissance du tampon du micro du sommet : $c(q) = \mathcal F^{-1}[\log|X_5|^2 - \overline{\log|X_5|^2}]$ sur la bande utile. Recherche du pic entre 0,3 ms et $2h/300 + 0{,}5$ ms ; écho retenu si le pic dépasse la moyenne de 5 écarts-types ; interpolation parabolique.

### 6.6 Mode temps réel et étalonnage

Simuler la chaîne ADC complète coûte cher : la démonstration ne l'exécute en direct que si le tampon tient dans 2048 échantillons décimés (côté ≲ 1,7 m). Au-delà, et dans toute l'étude, la mesure est tirée d'un **modèle de bruit étalonné par la chaîne ADC** pour la taille courante :

- **Modèle** : borne de Cramér-Rao de l'estimation de retard, $\sigma_\tau = \dfrac{1}{2\pi f_{rms}\sqrt{2\,B\,T\,\mathrm{SNR}}}$ avec $f_{rms}$ = 1000 Hz, B = 7900 Hz, T = 16 ms, combinée quadratiquement à un plancher de 1 µs (synchronisation, phase des capteurs). SNR d'une paire : $10\log_{10}\dfrac{2}{10^{-S_i/10} + 10^{-S_j/10}}$. Aberrants : 2 %, uniformes sur ± 0,3 ms.
- **Étalonnage** (au chargement et pour chaque taille de l'étude) : 80 trames ADC complètes, drone tiré entre 30 et 100 m, site 10–60°, vitesse 5–18 m/s, segments calés sur la géométrie vraie. On compare chaque écart mesuré à la convention exacte :

| Sortie | Calcul |
|---|---|
| `factor` | 1,4826 × médiane de \|z\| (estimateur robuste du σ réel / σ modèle) |
| `outlier` | part des \|z\| > 6 σ, hors écho |
| `echoLock` | part des paires avec le sommet accrochées sur l'écho au lieu du son direct |
| `echoDet`, `echoFactor`, `echoFalse` | taux de détection, précision et faux pics du cepstre |

En mode modèle, l'écart d'une paire vaut l'écart vrai + $\sigma_\tau \times$ `factor` × facteur du drone × N(0,1), avec les aberrants et accrochages d'écho étalonnés. L'estimateur ne connaît aucun de ces facteurs.

---

## 7. Estimateur

Cadence : une mesure toutes les 20 ms (50 Hz), à pas de temps fixe indépendant de l'affichage.

### 7.1 Par trame : direction et courbure

Modèle de front d'onde sphérique, en mètres de trajet, paramètres $\theta = [\mathbf q, \kappa]$ avec $\mathbf q = s\,\mathbf n$ (**n** : normale au front d'onde vers la source, s : lenteur relative) et κ = 1/R :

$$d_{ij} = s\,(f_i - f_j), \qquad f_i = \frac{\sqrt{1 - 2\kappa\,a_i + \kappa^2 b_i} - 1}{\kappa}, \quad a_i = \mathbf n\cdot\mathbf x_i,\ b_i = |\mathbf x_i|^2$$

(forme stable quand κ → 0, positions relatives au centre du réseau).

1. **Initialisation onde plane** : $d_{ij} = \mathbf q\cdot(\mathbf x_j - \mathbf x_i)$ par moindres carrés.
2. **Levenberg-Marquardt** sur les 4 paramètres, avec un résidu d'a priori sur la lenteur attendue en vent connu : $s_{att} = c/(c - \mathbf w\cdot\mathbf n)$, σ = 0,01 (les rafales sont inconnues). Un pas n'est accepté que s'il réduit le coût.
3. **RANSAC** : 24 tirages parmi les 125 arbres couvrants (4 paires indépendantes reliant les 5 micros, dénombrés par union-find) ; inliers à moins de 4 σ ; arrêt anticipé si les 10 paires sont inliers. Ajustement final sur les inliers, puis une passe de réaffectation. Un micro impliqué dans au moins 2 paires rejetées est signalé « suspect ».
4. **Correction de convention** : si une trajectoire est suivie, les écarts mesurés sont ramenés au front d'onde commun par le modèle de trajectoire ; sinon l'incertitude est augmentée (somme quadratique) de $D^2 \cdot 20 / (c \cdot 50)$, D étant la plus grande distance entre deux micros (drone à 20 m/s et 50 m).

Sorties : direction **n** et son incertitude (projection de la covariance orthogonale à **n**), courbure κ et son incertitude, paires retenues, indice de cohérence.

### 7.2 Trajectoire : moindres carrés sur horizon glissant

Modèle à **accélération constante** sur la fenêtre, 9 paramètres :

$$\mathbf p(t) = \mathbf p_0 + \mathbf v\,(t - t_0) + \tfrac12\mathbf a\,(t - t_0)^2$$

Pour chaque mesure de la fenêtre, le modèle recalcule l'instant d'émission de chaque front d'onde (vent moyen de la station, propagation exacte) et prédit directement les observables. **Résidus** :

| Résidu | Prédiction | σ |
|---|---|---|
| Écart de trajet de chaque paire retenue | $c\,[\tau_i(\mathbf p_j) - \tau_j(\mathbf p_j)]$ avec segments calés | Cramér-Rao par paire ⊕ 0,1 mm, × facteur de bruit estimé |
| Écart de niveau entre micros (centré) | $-20\log_{10} R_i - \alpha_w R_i$, à la constante près | 0,25 dB |
| Retard de l'écho sol | $\tau(\mathbf p', \mathbf x_5) - \tau(\mathbf p, \mathbf x_5)$ | Cramér-Rao au SNR de l'écho × √2, × échelle d'écho estimée |

Les écarts de niveau sont centrés sur leur moyenne : le niveau source inconnu disparaît, seule la décroissance 1/r entre micros reste. Elle porte une information de distance indépendante de la puissance du drone.

**A priori faibles** : vitesse σ = 20 m/s, accélération σ = 15 m/s² (≤ 2 g), distance 100 ± 150 m.

**Optimisation** :

- Pondération de **Huber** (k = 1,5 σ).
- **Levenberg-Marquardt** : jusqu'à 6 itérations, 6 essais de λ par itération, pas en position limité à 30 m, pas accepté seulement s'il réduit l'objectif.
- **Jacobien exact de l'instant d'émission** : la position d'émission dépend elle-même des paramètres, puisque $t_e = t_{rx} - |\mathbf p(t_e) - \mathbf x|/c$. En différentiant : $d\mathbf p = \left(\mathbf I - \dfrac{\mathbf V\,\mathbf g^\top}{c + \mathbf g\cdot\mathbf V}\right)\dfrac{\partial\mathbf p}{\partial\theta}\,d\theta$, avec **g** la direction unitaire micro → source et **V** la vitesse à l'émission. Chaque ligne du Jacobien vaut donc $\mathbf u^\top(\mathbf I - \mathbf V\mathbf g^\top/(c + \mathbf g\cdot\mathbf V))\,\partial\mathbf p/\partial\theta$, **u** étant le gradient géométrique du résidu.
- **Fenêtre adaptative** : 1 s par défaut ; si le χ² réduit dépasse 4 (manœuvre), réduction × 0,6 jusqu'à 0,3 s ; regain 0,5 s par seconde de fit cohérent.
- **Nouvelle piste** : départs multiples le long de la direction mesurée, à 10, 20, 40, 80, 160 et 320 m (10 itérations chacun) ; le meilleur χ² l'emporte.
- **Vérification** toutes les 2 s : les départs multiples sont relancés ; une solution concurrente remplace la piste si elle réduit le χ² d'au moins 20 %.
- **Extrapolation** : 2 s sans mesure, puis la piste est abandonnée ; 3,2 s d'historique conservé.

**Facteurs d'échelle estimés** (le bruit réel est inconnu) :

$$k \leftarrow \exp\!\big[(1-\alpha)\ln k + \alpha \ln\!\big(k\sqrt{\operatorname{médiane}(\chi^2_T)}\big)\big]$$

sur les 50 derniers χ² des écarts de temps, α = 0,03 (≈ 2 s à 50 Hz), borné à [0,5 ; 8]. Même principe pour l'écho (borne 30). Ce facteur absorbe aussi les erreurs de modèle non représentées (rafales inconnues, quantification des décalages).

**Sorties** : position et vitesse à l'instant présent (retard acoustique compensé), distance, gisement, site, vitesse radiale, covariance (multipliée par max(1, χ²)), incertitude le long de la visée, part de l'information de distance apportée par les niveaux.

---

## 8. Caméra d'identification

- Orientée **uniquement** par l'estimation Khéops ; aucun pistage vidéo.
- Consigne : direction de la position estimée à l'instant présent ; vitesse angulaire anticipée à partir de la vitesse estimée (anticipation), plus rattrapage d'écart du premier ordre (τ = 0,3 s) :
  $\Delta\theta = \dot\theta_{ff}\,\Delta t + \varepsilon\,(1 - e^{-\Delta t/\tau})$ (ε : écart de consigne), saturé à 120 °/s en gisement et 90 °/s en site.
- Suivi dès que la piste est valide ; l'alignement est jugé sur la position **vraie** (écart grand cercle < 1°).
- **Identifié** après 3 s d'alignement continu ; affichage 1,5 s puis nouveau drone.
- Vue « centrage » : écart axe caméra / drone, cercles à 1°, 2°, 3°.

---

## 9. Étude de dimensionnement

| Paramètre | Valeur |
|---|---|
| Côtés testés | 1, 2, 3, 5, 7, 10, 15, 20 m |
| Passages par taille | 40, trajectoires identiques d'une taille à l'autre (graine mulberry32 : 1000 + 7919·p) |
| Durée maximale d'un passage | 30 s simulées |
| Mesure | mode modèle, étalonné par la chaîne ADC pour chaque taille |
| Météo | celle de la station au moment du lancement |
| Caméra | même orientation de départ pour toutes les tailles |
| **Temps de focus** | début de la première période d'alignement < 1° tenue au moins 1 s, compté depuis le début du suivi |
| Résultat | médiane, 90 %, nombre de passages jamais mis au point, facteur de bruit ADC et aberrants (infobulle) |

Le calcul est découpé en tranches de 25 ms pour garder l'interface fluide. Les trajectoires de drone et les rafales sont reproductibles ; le bruit de mesure et l'étalonnage ne le sont pas (tirages indépendants), d'où de légères différences d'un essai à l'autre.

---

## 10. Choix arbitrés

| Sujet | Choix | Raison |
|---|---|---|
| Filtrage | **Aucun filtre récursif** (ni Kalman, ni α-β) : moindres carrés sur horizon glissant | Chaque estimation s'explique par les seules mesures de la fenêtre ; pas d'état caché ni de réglage de bruit de processus |
| Distance | Courbure + écarts de niveau entre micros + écho ; **jamais le niveau absolu** | La puissance d'un drone est inconnue (± 3 dB = facteur 1,4 sur la distance) ; les écarts entre micros et la courbure n'en dépendent pas |
| Nombre de stations | **Une seule pyramide**, agrandie si nécessaire | Une deuxième station donnerait la distance par triangulation, mais le but est de dimensionner une station autonome |
| Géométrie | Pyramide 4 + 1, sommet à 0,636·a | Le micro du sommet donne la résolution en site et l'écho sol ; un seul paramètre (a) à dimensionner |
| Réglages | **Zéro paramètre utilisateur** | Météo, bruit de fond et position viennent du terrain ; le réalisme passe par des variations crédibles, pas par des curseurs |
| Météo | Station supposée **sur site** (écart local faible) | Cas favorable assumé ; le vent est celui de la station, sans effet « exotique » |
| Micros | Supposés étalonnés sur une source locale (0,2 dB) | Pratique de terrain standard |
| Mesure | Convention de corrélation exacte, segments calés par la piste | La convention « même instant de réception » faussait les écarts de v/c |
| Coût de calcul | Chaîne ADC en direct pour les petites bases ; modèle étalonné par l'ADC sinon | Temps réel garanti, sans perdre la fidélité de la chaîne réelle |
| Bruit inconnu | Facteur par drone (×0,8–×1,25) estimé par l'estimateur à partir de ses résidus | Le système réel ne connaît pas son bruit à l'avance |
| Robustesse | RANSAC par trame + Huber sur la trajectoire + départs multiples | Aberrants de multitrajet, accrochage sur l'écho, ambiguïté de distance au démarrage |
| Finalité | **Observation / identification, pas d'arme** | Système défensif de détection ; la conduite de tir a été retirée en v3.6 |
| Carte | Canvas natif (Leaflet retiré), clic = position de Khéops | Aucune librairie ; pas de GPS de l'appareil |
| Reproductibilité | mulberry32 pour l'environnement de l'étude | Mêmes trajectoires pour toutes les tailles : comparaison appariée |

---

## 11. Limites connues

- Sol plan et homogène ; un seul écho (sol), pas de bâtiment ni de végétation.
- Vent uniforme le long du trajet (moyenne du profil logarithmique) ; pas de turbulence ni de réfraction par gradient de température.
- Une seule source à la fois ; pas de séparation de plusieurs drones.
- L'écho n'est exploitable qu'à SNR élevé (drone à moins d'environ 45 m pour 79 dB) : il sert dans 0 à 7 % des mesures.
- Les écarts de niveau entre micros n'apportent qu'une faible part de l'information de distance (quelques % à 5 m de côté, affichée dans le panneau Distance) ; ils sont conservés car ils ne dépendent pas de la puissance du drone.
- La démonstration à 5 m tourne en mode modèle étalonné ; la chaîne ADC complète ne tourne en direct qu'en dessous de 1,7 m de côté.

---

## Technologie

- **Un seul fichier** HTML/CSS/JavaScript (≈ 4 700 lignes), aucune librairie, aucun outil de construction.
- Carte, statistiques, caméra et réticule dessinés en Canvas 2D.
- FFT radix-2 et algèbre linéaire (élimination de Gauss, inversion) écrites à la main.
- Boucle physique à pas fixe de 20 ms (au plus 5 pas rattrapés par image), interface rafraîchie à 10 Hz.
- Fonctionne hors ligne (sans carte ni météo en direct).
- Interface bilingue : français si le navigateur est en français, anglais sinon.

## Historique

| Version | Apport principal |
|---|---|
| 4.3 | Interface bilingue (langue du navigateur : français ou anglais) |
| 4.2 | Démarrage automatique ; convention de corrélation exacte ; segments calés par la piste ; Jacobien exact de l'instant d'émission ; départs multiples ; facteur de bruit estimé ; écho sol par cepstre ; étalonnage ADC par taille |
| 4.1 | Niveau propre à chaque micro ; écarts de niveau dans la trajectoire |
| 4.0 | Pyramide ; distance par la courbure ; trajectoire par moindres carrés glissants ; onglet Étude |
| 3.9 | Statistiques à onglets ; échelle graphique |
| 3.7 | Fond de carte Esri + CARTO ; météo en direct ; bruit de fond au micro ; plus aucun réglage |
| 3.6 | Partie arme supprimée ; caméra d'identification et chronomètres |
| 3.5 | Carte canvas (Leaflet retiré) ; ISO 9613-1 corrigée ; Kalman supprimé ; vérité terrain isolée |
| 3.4 | ADC 1 MHz, RANSAC, vent, ISO 9613-1 |

Le détail de chaque version figure dans le cartouche en tête de `kheops_v43.html`.

## Licence

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — utilisation non commerciale, avec attribution.

## Auteur

**Eric PERRET** — [GitHub](https://github.com/ericperret)
