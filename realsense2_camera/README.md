# Acquisition des Données de la Caméra

Ce document décrit comment configurer et lancer le pipeline d'acquisition des données de la caméra Intel RealSense D435 sous ROS 2, ainsi que le SLAM visuel avec RTAB-Map.

---

## Prérequis

- Caméra Intel RealSense D435 connectée en USB 3.0
- Package `realsense2_camera` installé (ou compilé depuis les sources sur Jetson Nano)
- Package `rtabmap_launch` installé (ou compilé depuis les sources sur Jetson Nano)
- ROS 2 Humble sourcé dans le terminal

```bash
source /opt/ros/humble/setup.bash
source install/setup.bash
```

---

## 1. Lancement du driver RealSense

Le nœud `realsense2_camera` publie les flux couleur et profondeur synchronisés à **640×480 @ 15 fps**.

```bash
ros2 launch realsense2_camera rs_launch.py \
  align_depth.enable:=true \
  depth_module.profile:=640x480x15 \
  rgb_camera.profile:=640x480x15 \
  enable_sync:=true \
  initial_reset:=true
```

## 2. Lancement de RTAB-Map

RTAB-Map souscrit aux topics publiés par le driver RealSense pour construire la carte 3D dense.

```bash
ros2 launch rtabmap_launch rtabmap.launch.py \
  rtabmap_args:="--delete_db_on_start" \
  rgb_topic:=/camera/camera/color/image_raw \
  depth_topic:=/camera/camera/aligned_depth_to_color/image_raw \
  camera_info_topic:=/camera/camera/color/camera_info \
  frame_id:=camera_link \
  approx_sync:=true \
  qos:=2 \
  rtabmap_viz:=false \
  rviz:=true
```

## 3. Topics ROS 2 publiés

Une fois les deux nœuds lancés, les topics suivants sont disponibles :

| Topic | Type | Description |
|---|---|---|
| `/camera/camera/color/image_raw` | `sensor_msgs/Image` | Flux vidéo RGB |
| `/camera/camera/aligned_depth_to_color/image_raw` | `sensor_msgs/Image` | Carte de profondeur alignée sur le RGB |
| `/camera/camera/color/camera_info` | `sensor_msgs/CameraInfo` | Paramètres intrinsèques de la caméra |
| `/rtabmap/cloud_map` | `sensor_msgs/PointCloud2` | Nuage de points 3D dense généré par RTAB-Map |

### Vérifier que les topics sont bien publiés

```bash
ros2 topic list
ros2 topic hz /camera/camera/color/image_raw
ros2 topic hz /rtabmap/cloud_map
```

---

## 4. Visualisation dans RViz

Une fois RViz ouvert, ajouter les displays suivants :

- **Image** → topic `/camera/camera/color/image_raw` — flux RGB en direct
- **Image** → topic `/camera/camera/aligned_depth_to_color/image_raw` — carte de profondeur
- **PointCloud2** → topic `/rtabmap/cloud_map` — nuage de points 3D

---

## Notes

- Sur **Jetson Nano**, `librealsense2` et `rtabmap` doivent être compilés depuis les sources car les packages binaires ne sont pas disponibles pour JetPack 4.6 / Ubuntu 18.04.
