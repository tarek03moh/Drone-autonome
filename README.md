# 🚁 Drone Autonome avec Perception 3D — ROBO4

## 🖼️ Aperçu

<p align="center">
  <img src="images/Drone_chassis_front.jpeg" width="400"/>
  <img src="images/Drone_chassis_side.jpeg" width="400"/>
</p>

> Projet de drone quadrirotor autonome capable de naviguer sans GPS en environnement intérieur,
> équipé d'un SLAM visuel 3D via une caméra RGB-D Intel RealSense D435.
 
**Auteurs :** Aziz Zouari & Tarek Abdrabo
**Module :** ROBO4 — Projet Robotique
**Année :** 2025 / 2026
 
---
 
## 📋 Table des matières
 
- [Aperçu du projet](#aperçu-du-projet)
- [Architecture du système](#architecture-du-système)
- [Matériel requis](#matériel-requis)
- [Prérequis logiciels](#prérequis-logiciels)
- [Installation](#installation)
- [Lancement](#lancement)
- [Structure du dépôt](#structure-du-dépôt)
- [État d'avancement](#état-davancement)
- [Perspectives](#perspectives)
- [Références](#références)
 
---
 
## Aperçu du projet
 
Ce projet vise à concevoir un drone quadrirotor autonome sur châssis **DJI F450** capable de :
 
- naviguer en **environnement intérieur sans GPS**,
- construire une **carte 3D dense** de son environnement en temps réel (SLAM visuel),
- détecter et éviter les obstacles grâce aux nuages de points générés par **RTAB-Map**.
 
Le flux de traitement principal est le suivant :
 
```
RealSense D435 ──► Jetson Nano (ROS 2 + RTAB-Map) ──► Pixhawk (PX4) ──► ESC + Moteurs
```
 
---
 
## Architecture du système
 
```
Gazebo ◄──── GZ Bridge ────► ROS 2 ◄──── uXRCE-DDS ────► PX4 / Pixhawk
                                │
                          RTAB-Map SLAM
                                │
                        Intel RealSense D435
```
 
Le bridge **Micro XRCE-DDS** assure la communication entre :
- le **client µXRCE-DDS** sur le Pixhawk (topics uORB)
- l'**agent µXRCE-DDS** sur la Jetson Nano (topics ROS 2)
 
---
 
## Matériel requis
 
| Composant | Description |
|---|---|
| Châssis DJI F450 | Structure quadrirotor, empattement 450 mm |
| 4× Moteurs Brushless | 1920 KV, commandés via PWM |
| 4× ESCs | Convertissent les signaux PWM en puissance moteur |
| Pixhawk 2.4.8 | Flight controller, firmware PX4 |
| Intel RealSense D435 | Caméra RGB-D stéréo, 640×480 px, interface USB 3.0 |
| Jetson Nano (NVIDIA) | Calculateur embarqué, exécute ROS 2 + RTAB-Map |
| Batterie LiPo 3S (12,6 V) | Source d'alimentation principale |
| Radio commande | Contrôle manuel via protocole SBUS (Spektrum) |
 
---
 
## Prérequis logiciels
 
- **OS :** Ubuntu 22.04 (machine de développement) / JetPack 4.6 (Jetson Nano)
- **ROS 2 :** Humble Hawksbill
- **PX4 :** Firmware open-source (v1.14+)
- **Gazebo :** Harmonic (simulation)
- **RTAB-Map :** compilé depuis les sources sur Jetson Nano
- **librealsense2 :** compilé depuis les sources sur Jetson Nano
- **Micro XRCE-DDS Agent**
 
> ⚠️ La Jetson Nano (JetPack 4.6 / Ubuntu 18.04) ne supporte pas officiellement ROS 2 Humble.
> `librealsense2` et `rtabmap` doivent être compilés depuis les sources.
> Voir [`docs/installation_jetson.md`](docs/installation_jetson.md) pour le guide complet.
 
---
 
## Installation
 
### 1. Cloner le dépôt
 
```bash
git clone https://github.com/<votre-username>/drone-autonome-robo4.git
cd drone-autonome-robo4
```
 
### 2. Installer les dépendances ROS 2
 
```bash
sudo apt update
rosdep install --from-paths src --ignore-src -r -y
```
 
### 3. Compiler le workspace
 
```bash
colcon build --symlink-install
source install/setup.bash
```
 
Voir [`docs/installation_jetson.md`](docs/installation_jetson.md) pour l'installation spécifique sur Jetson Nano.
 
---
 
## Lancement
 
### Simulation Gazebo
 
```bash
# Lancer PX4 + Gazebo
ros2 launch simulation gazebo_px4.launch.py
 
# Lancer le GZ Bridge
ros2 run ros_gz_bridge parameter_bridge ...
```
 
### Driver RealSense (caméra physique)
 
```bash
ros2 launch realsense2_camera rs_launch.py \
  align_depth.enable:=true \
  depth_module.profile:=640x480x15 \
  rgb_camera.profile:=640x480x15 \
  enable_sync:=true \
  initial_reset:=true
```
 
### SLAM visuel avec RTAB-Map
 
```bash
ros2 launch rtabmap_launch rtabmap.launch.py \
  rtabmap_args:="--delete_db_on_start" \
  rgb_topic:=/camera/camera/color/image_raw \
  depth_topic:=/camera/camera/aligned_depth_to_color/image_raw \
  camera_info_topic:=/camera/camera/color/camera_info \
  frame_id:=camera_link \
  approx_sync:=true \
  qos:=2 \
  rviz:=true
```
 
### Agent µXRCE-DDS (liaison Jetson ↔ Pixhawk)
 
```bash
MicroXRCEAgent serial --dev /dev/ttyTHS1 -b 921600
```
 
---
 
## Structure du dépôt
 
```
drone-autonome-robo4/
│
├── README.md
│
├── docs/                        # Documentation technique
│   ├── installation_jetson.md   # Guide d'installation sur Jetson Nano
│   ├── calibration_pixhawk.md   # Procédure de calibration Pixhawk / QGroundControl
│   ├── architecture.md          # Détail de l'architecture logicielle
│   └── wiring_diagram.md        # Schéma de câblage électronique
│
├── hardware/                    # Documentation matérielle
│   ├── bom.md                   # Bill of Materials (liste des composants)
│   ├── assembly_guide.md        # Guide d'assemblage du châssis F450
│   └── motor_config.md          # Configuration des moteurs (sens de rotation)
│
├── simulation/                  # Fichiers de simulation Gazebo
│   ├── worlds/                  # Scènes Gazebo (.world / .sdf)
│   ├── models/                  # Modèles 3D du drone et obstacles
│   └── launch/                  # Fichiers de lancement simulation
│
├── perception/                  # Pipeline de perception 3D
│   ├── launch/                  # Lancement RealSense + RTAB-Map
│   ├── config/                  # Paramètres RTAB-Map, filtres de nuages de points
│   └── scripts/                 # Scripts utilitaires (visualisation, export de cartes)
│
├── navigation/                  # Algorithmes de navigation et évitement d'obstacles
│   ├── launch/
│   ├── src/
│   └── config/
│
├── control/                     # Interface de contrôle PX4 / mode Offboard
│   ├── launch/
│   ├── src/                     # Nœuds ROS 2 de commande (setpoints, modes de vol)
│   └── config/                  # Paramètres PID Pixhawk exportés
│
├── tests/                       # Résultats et logs de tests
│   ├── flight_logs/             # Logs de vols (fichiers .ulg PX4)
│   ├── slam_maps/               # Cartes 3D générées par RTAB-Map (.db)
│   └── reports/                 # Comptes-rendus des séances de test
│
└── assets/                      # Médias et ressources
    ├── images/                  # Photos du drone, captures RViz, screenshots Gazebo
    └── rapport/                 # Rapport de projet PDF
```
 
---
 
## État d'avancement
 
| Étape | Statut |
|---|---|
| Assemblage mécanique et électronique (F450) | ✅ Terminé |
| Configuration Pixhawk + firmware PX4 | ✅ Terminé |
| Bridge uXRCE-DDS PX4 ↔ ROS 2 | ✅ Terminé |
| Acquisition RealSense D435 sous ROS 2 | ✅ Terminé |
| SLAM visuel RTAB-Map (tests statiques) | ✅ Validé en statique |
| Simulation Gazebo + jumeau numérique | ⚠️ Fonctionnel (contraintes GPU) |
| Test de vol manuel (mode Stabilized) | 🔄 En cours |
| Navigation autonome en boucle fermée (mode Offboard) | 🔄 En cours |
 
---
 
## Perspectives
 
- Intégration complète du pipeline SLAM visuel embarqué à bord
- Algorithme d'évitement d'obstacles basé sur les nuages de points RTAB-Map
- Filtre de Kalman pour l'estimation de position lors de saturation caméra
- Réglage optimal des correcteurs PID
- SLAM collaboratif multi-robot (long terme)
 
---
 
## Références
 
- [PX4 Autopilot](https://px4.io)
- [ROS 2 Humble](https://docs.ros.org/en/humble/)
- [RTAB-Map](https://introlab.github.io/rtabmap/)
- [Intel RealSense D435](https://www.intel.com/content/www/us/en/support/articles/000028662)
- [Micro XRCE-DDS — eProsima](https://micro-xrce-dds.docs.eprosima.com)
- [realsense-ros](https://github.com/IntelRealSense/realsense-ros)
 
---
 
*Projet réalisé dans le cadre du module ROBO4 — 2025/2026*
