# AprilTag ROS 2 tracker

This repository contains a ROS 2 wrapper around AprilTag 3, configured for the
`tag36h11` markers used by this project. It detects tags in a calibrated camera
stream, publishes their poses, and optionally broadcasts a TF frame for each
configured tag.

## Included printable tags

Print-ready SVG files for IDs 1 through 7 are in [`tags/tag36h11`](tags/tag36h11).
Each file has:

- AprilTag family: `tag36h11`
- black detection-square size: 40 mm
- total SVG size, including the white quiet zone: 50 mm

Print the SVG at **100% / actual size**. Disable "fit to page" and verify that
the outside edge of the black square measures 40 mm after printing. Do not crop
or cover the white border.

The same 40 mm measurement is configured as `0.04` metres in
[`apriltag_ros/config/tags.param.yaml`](apriltag_ros/config/tags.param.yaml).
If the printed black square is a different size, update that file or the
estimated distance will be scaled incorrectly.

## Requirements

- ROS 2 Humble
- A calibrated camera publishing a rectified image and `CameraInfo`
- The dependencies declared in
  [`apriltag_ros/package.xml`](apriltag_ros/package.xml), including the ROS 2
  `apriltag` package

## Clone and build

```bash
mkdir -p ~/apriltag_ws/src
cd ~/apriltag_ws/src
git clone --branch ros2-port https://github.com/KaliberAI/apriltag_ros.git

cd ~/apriltag_ws
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

The generated `build/`, `install/`, and `log/` directories are deliberately not
stored in Git.

## Run the detector

By default, the launch file expects these camera topics:

```text
/camera/color/image_rect_raw
/camera/color/camera_info
```

Start detection with:

```bash
source /opt/ros/humble/setup.bash
source ~/apriltag_ws/install/setup.bash

ros2 launch apriltag_ros continuous_detection.launch.xml \
  camera_name:=/camera/color \
  image_topic:=image_rect_raw
```

Change `camera_name` and `image_topic` to match the camera driver. The image
must be rectified, and its matching `CameraInfo` is required for pose
estimation.

## Outputs

The detector publishes:

```text
/apriltag_ros_continuous_detector_node/tag_detections
/apriltag_ros_continuous_detector_node/tag_detections_image
/tf
```

Inspect detections with:

```bash
ros2 topic echo /apriltag_ros_continuous_detector_node/tag_detections
```

Configured standalone tags also appear as TF child frames named `tag_1`
through `tag_7`. For example:

```bash
ros2 run tf2_ros tf2_echo CAMERA_OPTICAL_FRAME tag_7
```

Replace `CAMERA_OPTICAL_FRAME` with the `frame_id` from the camera's
`CameraInfo` message.

The supplied launch file also publishes a project-specific static transform
from `tag_7` to `track_target`. Its offset and frame names can be changed with
the `target_offset`, `source_frame_name`, and `target_frame_name` launch
arguments.

## Use from another ROS 2 project

Source this workspace before building the dependent workspace:

```bash
source /opt/ros/humble/setup.bash
source ~/apriltag_ws/install/setup.bash
cd ~/other_ws
colcon build
source install/setup.bash
```

Add this dependency to the consuming package's `package.xml`:

```xml
<depend>apriltag_ros</depend>
```

Then subscribe to `apriltag_ros/msg/AprilTagDetectionArray` on the detections
topic above, or use the published tag frames through TF.

## Configuration

- Detector family and tuning:
  [`apriltag_ros/config/settings.param.yaml`](apriltag_ros/config/settings.param.yaml)
- Recognized IDs and physical sizes:
  [`apriltag_ros/config/tags.param.yaml`](apriltag_ros/config/tags.param.yaml)
- Camera topic remapping and target frame:
  [`apriltag_ros/launch/continuous_detection.launch.xml`](apriltag_ros/launch/continuous_detection.launch.xml)

Rebuild and source the workspace after changing installed configuration:

```bash
cd ~/apriltag_ws
colcon build --symlink-install
source install/setup.bash
```

## Upstream and license

This project is based on
[AprilRobotics/apriltag_ros](https://github.com/AprilRobotics/apriltag_ros) and
the [AprilTag 3](https://github.com/AprilRobotics/apriltag) detector. See
[`LICENSE`](LICENSE) for licensing terms and
[`apriltag_ros/docs/tag_size_guide.svg`](apriltag_ros/docs/tag_size_guide.svg)
for the upstream tag-size diagram.
