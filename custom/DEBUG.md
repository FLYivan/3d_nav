# go2真机部署

## dddnav 部署
1、确保已安装docker

安装dds

apt-get update
apt-get install -y ros-humble-rmw-cyclonedds-cpp

确认是否安装
ls /opt/ros/humble/lib/librmw_cyclonedds_cpp.so


# 在宿主机
docker cp /home/unitree/cyclonedds_ws/cyclonedds.xml dddmr_humble_l4t_dev:/root/cyclonedds.xml

# 在容器内
export CYCLONEDDS_URI=file:///root/cyclonedds.xml
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
source /opt/ros/humble/setup.bash
ros2 daemon stop && ros2 topic list

2、启动容器
cd ~/dog_robot/lib/3d_nav/dddmr_docker/docker_file && ./run_go2_gpu.bash

docker rm -f dddmr_humble_l4t_dev

-v /home/unitree/cyclonedds_ws/cyclonedds.xml:/root/cyclonedds.xml:ro
-e ROS_DOMAIN_ID=0
-e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
-e CYCLONEDDS_URI=file:///root/cyclonedds.xml

3、容器内编译
cd dddmr_navigation/ && source /opt/ros/humble/setup.bash && ws_build

4、启动建图
cd dddmr_navigation/ && source /opt/ros/humble/setup.bash 
source install/setup.bash && ros2 launch dddmr_beginner_guide hesai_xt16_mapping.launch

5、保存地图
1）打开一个新终端
docker exec -it <容器名或ID> bash
# 进容器后
tmux
# Ctrl+b 再按 c  → 新窗口
# Ctrl+b 再按 %  → 左右分屏

ros2 service call /save_mapped_point_cloud std_srvs/srv/Empty


2）在宿主机创建地图保存位置
cd ~ && mkdir 3d_nav_map  (也可以用现有路径)

把保存在 /tmp对应文件夹的地图文件移动到 /root/dddmr_bags：

mv /tmp/2026_03_28_19_25_15/ /root/dddmr_bags/
（宿主机位置~/dddmr_bags/）


5、启动导航

ros2 launch dddmr_beginner_guide hesai_xt16_navigation.launch

上位机查看rviz


## 必要话题发布和订阅


/cmd_vel                    geometry_msgs/msg/Twist
/lidar_point_cloud          sensor_msgs/msg/PointCloud2
/odom                       nav_msgs/msg/Odometry
/tf 	                    tf2_msgs/msg/TFMessage

切换到go2_pc2分支
    ros2 launch go2_nav 3d_nav_real_go2.launch.py

colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release --packages-select dddmr_pcl

## 调试调参


经常报规划超时
略增 planner_patience

原地转圈很久
略减 oscillation_patience 或 oscillation_angle

局部规划老失败
略增 controller_patience，或检查感知/TF

CPU 吃紧
略降 controller_frequency、global_plan_query_frequency

只点到点、不需行驶中重规划
global_plan_query_frequency 可改小或参考 mapping 用 -1



用 GDB 拿到精确崩溃位置

source install/setup.bash
gdb -batch -ex run -ex bt --args \
  install/lego_loam_bor/lib/lego_loam_bor/lego_loam \
  --ros-args \
  --params-file install/dddmr_beginner_guide/share/dddmr_beginner_guide/config/hesai_xt16_mapping.yaml \
  -r /lslidar_point_cloud:=/lidar_point_cloud





# 调试问题
1、退出是容器没有执行关闭
2、容器内没有安装dds中间件，没有配置dds
3、增加一个3d里程计
4、voxel_grid_omp换成最原始的，试试看是否可以