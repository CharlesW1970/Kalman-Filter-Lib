******************************************************************************
  * @name    kalman_filter implementation
  * @author  Hongxi Wang <hongxiw0x7d1@foxmail.com>
  * @version V1.1.5
  * @date    2020/2/16
  * @brief   C implementation of kalman filter
******************************************************************************
该卡尔曼滤波器可以在传感器采样频率不同的情况下，动态调整矩阵H R和K的维数与数值。

因此矩阵H和R的初始化会与矩阵P A和Q有所不同。另外的，在初始化量测向量z时需要额外写入传感器量测所对应的状态与这个量测的方式，详情请见例程。

若不需要动态调整量测向量z，可简单将结构体中的Use_Auto_Adjustment初始化为0，并像初始化矩阵P那样用常规方式初始化z H R即可。

要求量测向量z与控制向量u在传感器回调函数中更新。整数0意味着量测无效，即自上次卡尔曼滤波更新后无传感器数据更新。因此量测向量z与控制向量u会在卡尔曼滤波更新过程中被清零

此外，矩阵P过度收敛后滤波器将难以再适应状态的缓慢变化，从而产生滤波估计偏差。该算法通过限制矩阵P最小值的方法，可有效抑制滤波器的过度收敛，详情请见例程

******************************************************************************

**Example：**

```c
// x = 
//   |   height   |
//   |  velocity  |
//   |acceleration|

KalmanFilter_t Height_KF;

void Height_KF_Init(void)
{
    static float P_Init[9] =
        {
            10, 0,  0,
            0, 30,  0,
            0,  0, 10,
        };
    static float F_Init[9] =
        {
            1, dt,  0.5*dt*dt,
            0,  1,         dt,
            0,  0, 		    1,
        }; 
    static float Q_Init[9] =
        {
            0.25*dt*dt*dt*dt, 0.5*dt*dt*dt, 0.5*dt*dt,
            0.5*dt*dt*dt,            dt*dt,        dt,
            0.5*dt*dt,                  dt,         1,
        };
    
    // 设置最小方差
    static float state_min_variance[3] = {0.03, 0.005, 0.1};
    
    // 开启自动调整
    Height_KF.Use_Auto_Adjustment = 1;
    
    // 气压测得高度 GPS测得高度 加速度计测得z轴运动加速度
    static uint8_t measurement_reference[3] = {1, 1, 3};
    
    static float measurement_degree[3] = {1, 1, 1};
    // 根据measurement_reference与measurement_degree生成H矩阵如下
    //（在当前周期全部测量数据有效情况下）
    //| 1 0 0 |
    //| 1 0 0 |
    //| 0 0 1 |
    
    static float mat_R_diagonal_elements[3] = {30, 25, 35};
    //根据mat_R_diagonal_elements生成R矩阵如下
    //（在当前周期全部测量数据有效情况下）
    //| 30 0 0 |
    //| 0 25 0 |
    //| 0 0 35 |
    
    Kalman_Filter_Init(&Height_KF, 3, 0, 3);
    
    // 设置矩阵值
    memcpy(Height_KF.P_data, P_Init, sizeof(P_Init));
    memcpy(Height_KF.F_data, F_Init, sizeof(F_Init));
    memcpy(Height_KF.Q_data, Q_Init, sizeof(Q_Init));
    memcpy(Height_KF.Measurement_Reference, 
           measurement_reference, sizeof(measurement_reference));
    memcpy(Height_KF.Measurement_Degree, measurement_degree, sizeof(measurement_degree));
    memcpy(Height_KF.Mat_R_Diagonal_Elements, 
           mat_R_diagonal_elements, sizeof(mat_R_diagonal_elements));
    memcpy(Height_KF.State_Min_Variance, state_min_variance, sizeof(state_min_variance));
}
void INS_Task(void const *pvParameters)
{
    // 循环更新
    Kalman_Filter_Update(&Height_KF);
    vTaskDelay(ts);
}

// 测量数据更新应按照以下形式 即向MeasuredVector赋值
void Barometer_Read_Over(void)
{
    //......
    INS_KF.Measured_Vector[0] = baro_height;
}
void GPS_Read_Over(void)
{
    //......
    INS_KF.Measured_Vector[1] = GPS_height;
}
void Acc_Data_Process(void)
{
    //......
    INS_KF.Measured_Vector[2] = acc.z;
}
```

