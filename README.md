# Drone Autonome à Évitement d'Obstacle (ROBO4 2025/2026)

## 🖼️ Aperçu

<p align="center">
  <img src="images/Drone_chassis_front.jpeg" width="400"/>
  <img src="images/Drone_chassis_side.jpeg" width="400"/>
</p>

Ce dépôt contient le code et la configuration d'un drone quadrirotor capable de navigation autonome et de perception 3D sans GPS. [cite_start]Ce projet a été réalisé par **Aziz Zouari** et **Tarek Abdrabo** dans le cadre du module ROBO4[cite: 2, 5].

## 📌 Présentation du Projet

[cite_start]L'objectif principal est de concevoir un drone capable de se localiser et de cartographier son environnement en temps réel (SLAM visuel) à l'aide d'une caméra RGB-D[cite: 14, 16]. 

### Caractéristiques principales :
* [cite_start]**Navigation sans GPS** : Utilisation du SLAM visuel pour l'évolution en intérieur[cite: 19].
* [cite_start]**Perception 3D** : Intégration d'une caméra Intel RealSense D435[cite: 21].
* [cite_start]**Traitement Embarqué** : Calculs effectués sur une NVIDIA Jetson Nano[cite: 22].
* [cite_start]**Simulation** : Environnement de test sécurisé sous Gazebo[cite: 23].

## 🛠️ Architecture Matérielle

| Composant | Rôle |
| :--- | :--- |
| **Châssis DJI F450** | [cite_start]Structure mécanique quadrirotor[cite: 36]. |
| **Pixhawk 2.4.8** | [cite_start]Contrôleur de vol exécutant le firmware PX4[cite: 36]. |
| **NVIDIA Jetson Nano** | [cite_start]Calculateur embarqué pour ROS 2 et le SLAM[cite: 36]. |
| **Intel RealSense D435** | [cite_start]Capteur de profondeur et flux couleur[cite: 36]. |
| **Batterie LiPo 3S** | [cite_start]Source d'énergie (12,6 V)[cite: 36]. |

## 💻 Architecture Logicielle

[cite_start]Le système repose sur une communication entre trois piliers majeurs[cite: 43]:
1.  [cite_start]**PX4** : Gestion du vol bas niveau et de la stabilisation[cite: 44].
2.  [cite_start]**ROS 2 Humble** : Middleware assurant la communication entre les nœuds de perception et de contrôle[cite: 45].
3.  [cite_start]**Gazebo** : Jumeau numérique pour la validation des algorithmes[cite: 46].

### Bridge de communication
[cite_start]La communication entre PX4 (uORB) et ROS 2 (DDS) est assurée par le bridge **Micro XRCE-DDS**[cite: 52]. [cite_start]Le client tourne sur le Pixhawk et l'agent sur la Jetson Nano via une liaison UART/UDP[cite: 53, 54, 55].

## 🚀 Installation et Utilisation

### Acquisition des données (RealSense)
[cite_start]Pour lancer le driver de la caméra avec l'alignement de la profondeur (requis pour le SLAM)[cite: 99, 104]:
```bash
ros2 launch realsense2_camera rs_launch.py align_depth.enable:=true
