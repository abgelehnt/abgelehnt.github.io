# 直立篇

## 陀螺仪

AHRS陀螺仪姿态解算算法大致分三类：Mahony互补滤波，扩展卡尔曼EKF姿态融合，梯度下降。其中，互补滤波的抖动实在恐怖，EKF的运算量过大，梯度下降算法是最好用的，更重要的是：已经有大佬开源了此算法[xioTechnologies/Fusion](https://github.com/xioTechnologies/Fusion)

