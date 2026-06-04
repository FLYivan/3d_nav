# go2真机部署

## dddnav 部署
1、确保已安装docker


2、启动容器
cd ~/dog_robot/lib/3d_nav/dddmr_docker/docker_file && ./run_go2_gpu.bash



3、容器内编译


cd dddmr_navigation/ && source /opt/ros/humble/setup.bash && colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release

cd dddmr_navigation/ && source /opt/ros/humble/setup.bash && ws_build


4、启动建图
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


/cmd_vel geometry_msgs/msg/Twist
/lidar_point_cloud sensor_msgs/msg/PointCloud2
/odom nav_msgs/msg/Odometry
/tf 	tf2_msgs/msg/TFMessage

切换到go2_pc2分支
    ros2 launch go2_nav 3d_nav_real_go2.launch.py



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