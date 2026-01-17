# 🎯 One-Page-One-Mission (Khéops)

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
[![Pure HTML/JS](https://img.shields.io/badge/Pure-HTML%2FJS-blue.svg)](#)
[![No Dependencies](https://img.shields.io/badge/Dependencies-None-green.svg)](#)

**Simulateur de détection acoustique de drone** avec localisation 3D temps réel et conduite de tir.

![Screenshot](screenshot.png)

## 🚀 Fonctionnalités

- **Multilatération TDOA** — Algorithme de Chan pour estimation de position 3D
- **Filtre de Kalman** — Fusion 6 états (position + vitesse)
- **RANSAC** — Rejet des mesures aberrantes
- **ISO 9613-1** — Calcul précis de la vitesse du son (température, humidité, pression, CO₂)
- **Intégration Doppler** — Estimation de vitesse radiale avec zone morte
- **Simulation de vent** — Modèle de perturbation atmosphérique réaliste
- **Conduite de tir** — Handover automatique acoustique → optique
- **Visualisation temps réel** — Carte Leaflet, vue latérale, caméra, viseur

## 🛠️ Technologies

- HTML/CSS/JavaScript pur
- Aucune dépendance externe (sauf Leaflet pour les tuiles de carte)
- Fonctionne entièrement dans le navigateur
- Fichier unique, ~3000 lignes

## 📖 Utilisation

1. Ouvrir \`index.html\` dans un navigateur
2. Cliquer sur **START** pour lancer la simulation
3. Ajuster les paramètres dans la barre latérale (vent, vitesse drone, température...)
4. Observer le système acoustique traquer le drone et transférer à la conduite de tir optique

## 🎓 But pédagogique

Ce simulateur démontre :
- Les techniques de localisation acoustique passive
- La fusion de capteurs avec filtrage de Kalman
- Les concepts anti-drone (Counter-UAS)
- Le traitement du signal temps réel en JavaScript

## 📄 Licence

CC BY-NC 4.0 — Usage non commercial uniquement.

## 👤 Auteur

**Eric PERRET** — [GitHub](https://github.com/ericperret)