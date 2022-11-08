# 循迹篇

由于我们在做比赛时仅仅是个大二学生，我们不会用任何高级的控制算法，直立控制部分我们使用串级PID，速度控制部分我们使用一阶RC滤波加PID，转向也是PID。

## 姿态拟合

AHRS陀螺仪姿态解算算法主流的有三类：Mahony互补滤波，扩展卡尔曼EKF姿态融合，梯度下降。其中，互补滤波的抖动实在恐怖，EKF的运算量过大，梯度下降算法是最好用的，更重要的是：已经有大佬开源了此算法[xioTechnologies/Fusion](https://github.com/xioTechnologies/Fusion)

## 串级PID调参经验

- 内环过小时：跑起来车会明显很软，还有可能车会贴在地上跑。

- 内环过大时：会有类似手机震动一样的高频振动。

- 电机的温度对车的影响很大，在电机温度较高时可以增大内环抵消。

## 速度环

由于速度环为正反馈，目标速度的大幅改变必然导致震荡，所以需要加一个RC滤波器减缓震荡。

``` c
#define Speed_EncoderCof 0.1f //一阶RC滤波器滤波系数
#define SPEED_LIMIT 30.0f // 加速角度阈值
#define SPEED_DECREASE_LIMIT 16.0f // 减速角度阈值
Speed_FilterEncoder = Speed_EncoderCof * Encoder_AvgValue + (1 - Speed_EncoderCof) * Speed_FilterEncoder;
float SpeedError = Speed_FilterEncoder - TargetSpeed;
SpeedOut = arm_pid_f32(&Speed, SpeedError);

// 防止输出角度太大导致车碰地
if (SpeedOut >= SPEED_DECREASE_LIMIT)
	SpeedOut = SPEED_DECREASE_LIMIT;
else if (SpeedOut <= -SPEED_LIMIT)
	SpeedOut = -SPEED_LIMIT;
```

## 转向环

``` c
float TurnSpeedPIDLoc() {
	float Turn_Out;

	if (!BlobFoundFlag) {
		Turn_Out = arm_pid_f32(&TurnSpeed, (float) IMU_Data.gyro_z - BlobNotFoundTurn);
		if (Turn_Out < 0)
			Turn_Out = 0;
	} else {
		float AngleError = DegreeK * (-TargetAngle);
		if (AngleError > TurnLimit) {
			AngleError = TurnLimit;
			BlobFoundFlag++;

			Turn_Out = arm_pid_f32(&TurnSpeed, (float) IMU_Data.gyro_z - AngleError);
			if(Turn_Out > 0)
				Turn_Out = 0;
		} else if (AngleError < -TurnLimit) {
			AngleError = -TurnLimit;
			BlobFoundFlag++;

			Turn_Out = arm_pid_f32(&TurnSpeed, (float) IMU_Data.gyro_z - AngleError);
			if(Turn_Out < 0)
				Turn_Out = 0;
		} else {
			BlobFoundFlag = 1;

			Turn_Out = arm_pid_f32(&TurnSpeed, (float) IMU_Data.gyro_z - AngleError);
		}
	}

	/// 关闭转向环
//	TargetTurnSpeed = 0;

	return Turn_Out;
}
```

