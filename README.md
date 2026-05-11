# CardioSync — Analyse avancée de la cohérence cardiaque

> **Application de biofeedback physiologique destinée aux utilisateurs professionnels habitués aux outils d'analyse physiologique.**
> ⚠️ Cette application n'est pas destinée à un usage avec des patients.

---

## Table des matières

- [Aperçu](#aperçu)
- [Fonctionnement](#fonctionnement)
  - [Module A — Capteur flash iPhone (PPG)](#module-a--capteur-flash-iphone-ppg)
  - [Module B — Score de cohérence cardiaque](#module-b--score-de-cohérence-cardiaque)
  - [Module C — Guide respiratoire](#module-c--guide-respiratoire)
- [Fonctionnalités](#fonctionnalités)
- [Stack technique](#stack-technique)
- [Structure du projet](#structure-du-projet)
- [Installation et lancement](#installation-et-lancement)
- [Configuration](#configuration)
- [Algorithmes](#algorithmes)
- [Compatibilité](#compatibilité)
- [Avertissements et limitations](#avertissements-et-limitations)
- [Licence](#licence)

---

## Aperçu

**CardioSync** est une application d'analyse avancée de la cohérence cardiaque qui exploite le **flash et la caméra arrière de l'iPhone comme capteur biométrique optique** (photopléthysmographie, PPG). Elle est conçue pour les professionnels du biofeedback, chercheurs en physiologie et praticiens du bien-être souhaitant accéder à des données cardiaques de haute précision sans matériel dédié supplémentaire.

Le capteur CMOS de l'iPhone capte les **micro-variations de densité optique** de la microcirculation cutanée à plus de 30 fps. L'application en extrait la série temporelle des intervalles RR, calcule la variabilité de la fréquence cardiaque (HRV) et produit un **score de cohérence cardiaque en temps réel**.

```
Doigt sur le flash  →  Signal PPG brut  →  Extraction RR  →  Analyse HRV  →  Score cohérence
```

---

## Fonctionnement

### Module A — Capteur flash iPhone (PPG)

Le principe repose sur la **photopléthysmographie** (PPG) :

1. Le flash LED illumine en continu les capillaires du bout du doigt.
2. Le capteur CMOS mesure l'intensité lumineuse réfléchie image par image.
3. Chaque systole cardiaque provoque une variation de la concentration en hémoglobine oxygénée, modifiant l'absorption lumineuse.
4. Ces micro-variations constituent le **signal PPG brut**, dont on extrait les pics systoliques pour reconstituer la série des intervalles RR.

**Métriques produites en temps réel :**

| Métrique | Description | Unité |
|----------|-------------|-------|
| `HR` | Fréquence cardiaque instantanée | BPM |
| `RR` | Intervalle inter-battement | ms |
| `HRV` | Variabilité globale de la fréquence cardiaque | ms |
| `SDNN` | Écart-type des intervalles RR normaux | ms |
| `RMSSD` | Racine carrée de la moyenne des différences RR au carré | ms |

---

### Module B — Score de cohérence cardiaque

L'algorithme applique une **transformée de Fourier rapide (FFT)** sur la série temporelle des intervalles RR pour quantifier la puissance spectrale dans trois bandes :

- **VLF** — 0.003–0.04 Hz (très basse fréquence)
- **LF** — 0.04–0.15 Hz (basse fréquence, résonance baroréflexe)
- **HF** — 0.15–0.40 Hz (haute fréquence, activité vagale)

Le **score de cohérence** est calculé comme la proportion de la puissance spectrale totale concentrée dans la bande LF lors d'un guidage respiratoire à 5–6 cycles/minute.

```
Score (%) = P_LF / (P_VLF + P_LF + P_HF) × 100
```

> **Seuil clinique :** Lorsque le score dépasse **80 %**, les effets positifs sur l'état physiologique deviennent particulièrement significatifs, notamment sur la régulation du système nerveux autonome et la balance sympatho-vagale.

---

### Module C — Guide respiratoire

Le guide respiratoire visuel est réglé sur la **fréquence de résonance du baroréflexe** (≈ 0.1 Hz, soit 6 cycles/minute). Ce protocole :

- Maximise l'amplitude de la variabilité sinusale respiratoire (RSA).
- Synchronise les oscillations cardiovasculaires avec la fenêtre de gain maximal du barorécepteur carotidien.
- Optimise le ratio LF/HF et donc le score de cohérence.

**Phases respiratoires par défaut :**

| Phase | Durée | Mode relaxant | Mode dynamisant |
|-------|-------|---------------|-----------------|
| Inspiration | 5 s | 5 s | 4 s |
| Expiration | 5 s | 6 s | 4 s |

---

## Fonctionnalités

- **Mesure PPG en temps réel** via le flash et la caméra arrière de l'iPhone
- **Score de cohérence cardiaque dynamique** avec seuil visuel à 80 %
- **Graphique interactif** de la courbe de cohérence avec zoom et marqueurs
- **Analyse spectrale HRV** (FFT) avec visualisation des bandes LF / HF / VLF
- **Historique complet des séances** — consultation à tout moment
- **Export des données brutes** au format CSV (séries RR, signal PPG)
- **Partage des résultats** — mail, Facebook, Twitter
- **Programmes personnalisés** :
  - 🌊 *Mode Relaxant* — cycles lents, priorité récupération parasympathique
  - ⚡ *Mode Dynamisant* — cycles courts, activation cognitive ciblée
- **Interface biométrique premium** — design health-tech, fond sombre, animations fluides

---

## Stack technique

| Composant | Technologie |
|-----------|-------------|
| Frontend | HTML5 · CSS3 · JavaScript (Vanilla) |
| Capteur | `AVCaptureSession` (iOS) — accès caméra/flash natif |
| Traitement signal | FFT maison + filtrage passe-bande Butterworth |
| Stockage | `CoreData` (iOS) / `IndexedDB` (web) |
| Export | CSV · PNG (graphiques) |
| Animations | CSS Animations · Canvas 2D API |
| Responsive | Mobile-first · CSS Grid · Flexbox |

> Le projet de démonstration (`coherence-cardiaque.html`) est un fichier HTML autonome sans aucune dépendance externe.

---

## Structure du projet

```
cardiosync/
├── index.html                  # Landing page de présentation
├── coherence-cardiaque.html    # Application principale (démo navigateur)
├── README.md
│
├── src/
│   ├── capture/
│   │   ├── ppg-reader.js       # Lecture du signal caméra / flash
│   │   └── peak-detector.js    # Détection des pics systoliques
│   ├── signal/
│   │   ├── fft.js              # Transformée de Fourier rapide
│   │   ├── butterworth.js      # Filtres passe-bande
│   │   └── hrv-metrics.js      # Calcul SDNN, RMSSD, LF/HF
│   ├── ui/
│   │   ├── ecg-canvas.js       # Rendu ECG temps réel
│   │   ├── gauge.js            # Jauge circulaire de cohérence
│   │   └── breath-guide.js     # Animateur du guide respiratoire
│   └── storage/
│       ├── session-store.js    # Sauvegarde des séances
│       └── export.js           # Export CSV / partage
│
├── assets/
│   ├── icons/                  # Icônes SVG
│   └── fonts/                  # Polices (optionnel)
│
└── tests/
    ├── signal/                 # Tests unitaires traitement du signal
    └── ui/                     # Tests composants UI
```

---

## Installation et lancement

### Version navigateur (démo)

Aucune installation requise. Ouvrir directement :

```bash
open coherence-cardiaque.html
```

### Version iOS native

```bash
# Prérequis : Xcode 15+, iOS 16+, compte développeur Apple
git clone https://github.com/cardiosync/app.git
cd app
open CardioSync.xcodeproj
```

Sélectionner un device physique (le simulateur ne supporte pas la caméra), puis `⌘ + R`.

> **Important :** L'accès au flash et à la caméra nécessite un iPhone physique. Le simulateur Xcode ne donne pas accès au matériel optique.

### Permissions iOS requises

Ajouter dans `Info.plist` :

```xml
<key>NSCameraUsageDescription</key>
<string>CardioSync utilise la caméra et le flash pour mesurer votre fréquence cardiaque par photopléthysmographie (PPG).</string>
```

---

## Configuration

Les paramètres principaux sont accessibles dans `src/config.js` :

```js
export default {
  // Capture
  captureFrameRate: 30,         // fps — minimum recommandé pour signal PPG fiable
  flashIntensity: 1.0,          // 0.0–1.0

  // Traitement signal
  rrWindowSize: 60,             // Nombre d'intervalles RR pour la FFT
  bandLF: [0.04, 0.15],         // Hz
  bandHF: [0.15, 0.40],         // Hz
  coherenceThreshold: 80,       // % — seuil d'affichage du feedback positif

  // Guide respiratoire (secondes)
  breathModes: {
    relax:   { inhale: 5, exhale: 6 },
    dynamic: { inhale: 4, exhale: 4 },
  },

  // Stockage
  maxSessionHistory: 90,        // jours de rétention
};
```

---

## Algorithmes

### Détection des pics PPG

```
Signal brut  →  Normalisation  →  Filtre passe-bande [0.5–4 Hz]  →  Seuillage adaptatif  →  Pics RR
```

Le seuil adaptatif est recalculé sur une fenêtre glissante de 5 s afin de compenser les variations d'amplitude dues aux mouvements.

### Calcul FFT (cohérence)

```
Série RR (N points)  →  Fenêtrage Hanning  →  FFT  →  Densité spectrale de puissance (PSD)
                                                           ↓
                                             Intégration P_LF / P_total → Score %
```

La PSD est recalculée toutes les 5 s sur une fenêtre glissante de 60 intervalles RR (≈ 1 minute).

---

## Compatibilité

| Plateforme | Support |
|------------|---------|
| iPhone XS et ultérieur (iOS 16+) | ✅ Complet |
| iPhone SE 2e/3e génération | ✅ Complet |
| iPad avec caméra arrière | ✅ Partiel (flash moins puissant) |
| Navigateur desktop (démo) | ✅ Simulation (pas de capteur réel) |
| Android | ❌ Non supporté (cette version) |

---

## Avertissements et limitations

- **Usage non médical.** CardioSync est un outil de biofeedback à usage personnel et professionnel. Il n'est **pas un dispositif médical** et ne doit pas être utilisé à des fins diagnostiques ou thérapeutiques.
- **Pas d'usage avec des patients.** Cette application n'est pas conçue pour un contexte clinique ou paramédical.
- **Précision conditionnelle.** La qualité du signal PPG dépend de l'immobilité du doigt, de la pression d'appui, et de la luminosité ambiante. Des artéfacts de mouvement peuvent dégrader le score de cohérence.
- **Non CE / FDA.** L'application n'a fait l'objet d'aucune certification en tant que dispositif médical.

---

## Licence

```
MIT License — CardioSync © 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software.
```

---

*CardioSync — Précision physiologique. Interface premium. Aucun capteur externe requis.*
