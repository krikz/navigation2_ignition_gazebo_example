# Используем официальный образ ROS 2 Humble с Gazebo Fortress
FROM osrf/ros:humble-desktop

LABEL maintainer="you@example.com"
ENV DEBIAN_FRONTEND=noninteractive \
    ROS_DISTRO=humble \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8 \
    SHELL=/bin/bash \
    PYTHONUNBUFFERED=1

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    curl \
    git \
    wget \
    python3-pip \
    python3-vcstool \
    python3-colcon-common-extensions \
    ros-humble-navigation2 \
    ros-humble-nav2-bringup \
    ros-humble-cv-bridge \
    ros-humble-laser-geometry \
    ros-humble-sensor-msgs \
    ros-humble-xacro \
    ros-humble-gazebo-ros-pkgs \
    && rm -rf /var/lib/apt/lists/*

# Установка Python-зависимостей
RUN pip3 install --no-cache-dir \
    artefacts-toolkit \
    artefacts-cli \
    rerun-sdk \
    trimesh \
    pyquaternion \
    yourdfpy

# ============= ДАЛЬШЕ — ТОЛЬКО НЕ ЗАТРАГИВАТЬ ПРЕДЫДУЩИЕ СЛОИ =============

# Рабочая директория
WORKDIR /root/nav2_ignition_example

# Клонируем репозиторий
RUN git clone https://github.com/art-e-fact/navigation2_ignition_gazebo_example.git .

RUN apt-get update && apt-get install -y ros-humble-ros-gz-bridge \
ros-humble-ros-gz-sim \
ros-humble-ackermann-steering-controller \
ros-humble-control-msgs \
ros-humble-diff-drive-controller \
ros-humble-effort-controllers \
ros-humble-force-torque-sensor-broadcaster \
ros-humble-hardware-interface \
ros-humble-imu-sensor-broadcaster \
ros-humble-joint-state-broadcaster \
ros-humble-joint-trajectory-controller \
ros-humble-mecanum-drive-controller \
ros-humble-ros2controlcli \
ros-humble-tricycle-controller \
ros-humble-velocity-controllers \
ros-humble-joint-state-publisher \
ros-humble-ros2-control \
ros-humble-ros2-controllers \
ros-humble-robot-localization \
ros-humble-ros-ign-gazebo \
ros-humble-topic-tools \
libignition-gazebo6-dev \
libignition-plugin-dev \
ros-humble-controller-manager

# Импортируем дополнительные зависимости через vcs
RUN vcs import --input deps.repos src

RUN rosdep install -y -r -i  --from-paths .

# === УСТАНОВКА ROS-ЗАВИСИМОСТЕЙ: исправленная версия с нужными skip-keys ===
# Теперь мы добавили: ros-gz-bridge, ros_ign_gazebo, ign_ros2_control
# Устанавливаем ROS-зависимости, пропуская те, что будут собраны из src или уже доступны
# Устанавливаем ROS-зависимости, пропуская те, что будут собраны из src
RUN . /opt/ros/humble/setup.sh && \
    if [ ! -f /etc/ros/rosdep/sources.list.d/20-default.list ]; then \
        rosdep init || true; \
    fi && \
    rosdep update && \
    rosdep install -y \
        --from-paths src \
        --ignore-src \
        --rosdistro humble \
        --skip-keys="control_msgs,control-msgs,ros-gz-bridge,ros_ign_gazebo,ign_ros2_control,ros-ign-gazebo,joint_state_publisher,robot_state_publisher,xacro,robot_localization,topic_tools,controller_manager,hardware_interface,velocity_controllers"

# Собираем проект
RUN . /opt/ros/humble/setup.sh && \
    colcon build --symlink-install

# Настройка окружения
ENV ROS_PACKAGE_PATH=/root/nav2_ignition_example:$ROS_PACKAGE_PATH
ENV GAZEBO_MODEL_PATH=/root/nav2_ignition_example/models:${GAZEBO_MODEL_PATH}
ENV GAZEBO_RESOURCE_PATH=/root/nav2_ignition_example/models:${GAZEBO_RESOURCE_PATH}

# Sourcing setup.bash по умолчанию
SHELL ["/bin/bash", "-c"]
RUN echo ". /opt/ros/humble/setup.sh" >> ~/.bashrc && \
    echo ". /root/nav2_ignition_example/install/setup.bash" >> ~/.bashrc

# === Точка входа: запуск launch-файла или интерактивная оболочка ===
CMD ["bash"]