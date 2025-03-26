---
title: ros-dolly分析
top_img: /img/background.png
cover: /img/unhappy.jpg
date: 2024-12-24 21:48:26
tags: ros-rt
---

# 理论分析
分析链：lidar -> /dolly/laser_scan -> /dolly/cmd_vel -> /tf
分析链的应用意义：激光雷达传感器扫描前方物体，找到最近的物体并控制车轮跟随。

## 基本参数
$T$：周期

$D$：deadline截止日期

$t_{s}$：开始执行时间

$t_r$：释放时间

$t_e$: 完成时间

$e$：执行时间

## 参数分析

### 周期
分析链中的lidar为激光雷达传感器，其在gezebo建模中设置了固定的更新频率。同时lidar也为分析链的
source event，其频率对应的周期可看作该链的周期。
在`dolly_gazebo/models/dolly/model.sdf`文件中里的`<sensor>`组件中设置了传感器更新频率为
`<update_rate>100.0</update_rate>`。

因此链周期$T$为10ms。

### 截止日期

```c++
 void OnSensorMsg (const sensor_msgs::msg::LaserScan::SharedPtr _msg)
  {
    // 1、Find closest hit
    float min_range = _msg->range_max + 1;
    int idx = -1;
    for (auto i = 0u; i < _msg->ranges.size(); ++i) {
      auto range = _msg->ranges[i];
      if (range > _msg->range_min && range < _msg->range_max && range < min_range) {
        min_range = range;
        idx = i;
      }
    }

    // 2、Calculate desired yaw change
    double turn = _msg->angle_min + _msg->angle_increment * idx;

    // 3、Populate command message, all weights have been calculated by trial and error
    auto cmd_msg = std::make_unique<geometry_msgs::msg::Twist>();

    // Bad readings, stop
    if (idx < 0) {
      cmd_msg->linear.x = 0;
      cmd_msg->angular.z = 0;
    } else if (min_range <= min_dist_) {
      // Too close, just rotate
      cmd_msg->linear.x = 0;
      cmd_msg->angular.z = turn * angular_k_;
    } else {
      cmd_msg->linear.x = linear_k_ / abs(turn);
      cmd_msg->angular.z = turn * angular_k_;
    }

    cmd_pub_->publish(std::move(cmd_msg));
  }
    /// \brief Laser messages subscriber
  rclcpp::Subscription<sensor_msgs::msg::LaserScan>::SharedPtr laser_sub_;

  /// \brief Velocity command publisher
  rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr cmd_pub_;

  /// \brief Minimum allowed distance from target
  double min_dist_ = 1.0;

  /// \brief Scale linear velocity, chosen by trial and error
  double linear_k_ = 0.02;

  /// \brief Scale angular velocity, chosen by trial and error
  double angular_k_ = 0.08;
```
一个回调的截止时间与其关联链的截止时间相同（PiCAS）。

首先需要明确，截止时间是特定于应用场景的。截止时间的出现伴随场景中实时性的要求。
在dolly follow场景中，对dolly的运动速度和运动时间并没有限制。

对其实际运动行为进行分析： /dolly/laser_scan接受传感器信息，并将其转化成dolly控制参数Twist,由/dolly/cmd_vel发布给两轮差速控制器控制dolly运动。

实际上是，在确定跟随目标后，dolly以尽可能大的线速度和角速度运动朝目标物体运动。代码中设置了`min_dist_`时，表示在与目标物体距离小于`min_dist_`，向dolly下发送刹车指令，即线速度为0，角速度正常，dolly需要在撞击到目标物体前将速度减小至0。

因为在距离目标物体`min_dist_`时，dolly的速度最大。在速度减为0前，/dolly/cmd_vel发布的控制指令中线速度值一直为0,两轮差速控制器控制dolly减速。
在此场景下，dolly运动具有实时约束，求得deadline_a为`min_dist_/linear_k_`，`linear_k_`为所有情况下的线速度最小值。

该deadline包括了从下刹车指令到运动结束整个过程，包含链计算和运动两个部分。因此链实例的截止时间`deadline=dealine_a-刹车到0的运动时间`。





