******************************************************************************
  * @name    kalman_filter implementation
  * @author  Hongxi Wong <hongxiw0x7d1@foxmail.com>
  * @version V1.0.7
  * @date    2020/5/28
  * @brief   C implementation of kalman filter
******************************************************************************
�ÿ������˲��������ڴ���������Ƶ�ʲ�ͬ������£���̬��������H R��K��ά������ֵ��
This implementation of kalman filter can dynamically adjust dimension and value of matrix H R and K according to the measurement validity under any circumstance that the sampling rate of component sensors are different.

��˾���H��R�ĳ�ʼ���������P A��Q������ͬ������ģ��ڳ�ʼ����������zʱ��Ҫ����д�봫������������Ӧ��״̬���������ķ�ʽ������������̡�
Therefore, the initialization of matrix P, A, and Q is sometimes different from that of matrix H R. when initialization. Additionally, the corresponding state and the method of the measurement should be provided when initializing measurement vector z. For more details, please see the example.

������Ҫ��̬������������z���ɼ򵥽��ṹ���е�Use_Auto_Adjustment��ʼ��Ϊ0�������ʼ������P�����ó��淽ʽ��ʼ��z H R���ɡ�
If automatic adjustment is not required, assign zero to the Use_Auto_Adjustment and initialize z H R in the normal way as matrix P.

Ҫ����������z���������u�ڴ������ص������и��¡�����0��ζ��������Ч�������ϴο������˲����º��޴��������ݸ��¡������������z���������u���ڿ������˲����¹����б�����
Measured_Vector and Control_Vector are required to be updated in the sensor callback function. Integer 0 in measurement vector z indicates the invalidity of current measurement, so Measured_Vector and Control_Vector will be reset (to 0) during each update. 

���⣬����P�����������˲�������������Ӧ״̬�Ļ����仯���Ӷ������˲�����ƫ����㷨ͨ�����ƾ���P��Сֵ�ķ���������Ч�����˲����Ĺ�������������������̡�

Additionally, the excessive convergence of matrix P will make filter incapable of adopting the slowly changing state. This implementation can effectively suppress filter excessive convergence through boundary limiting for matrix P. For more details, please see the example.

******************************************************************************

**Example��**

```c
void Height_KF_Init(void)
{
    static float P_Init[9] =
        {
            10, 0,  0,
            0, 30,  0,
            0,  0, 10,
        };
    static float A_Init[9] =
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
    
    //boundary limiting for matrix P
    static float state_min_variance[3] = {0.03, 0.005, 0.1};
    
    //use auto adjustment for matrix H R K
    Height_KF.Use_Auto_Adjustment = 1;
    
    //baro for height  GPS for height  IMU for acc
    static uint8_t measurement_reference[3] = {1, 1, 3};
    
    //barometer measures height indirectly
    static float measurement_degree[3] = {0.8, 1, 1};
    //according to measurement_reference and measurement_degree
    //matrix H will be like this:
    //| 0.8 0 0 |
    //|   1 0 0 |
    //|   0 0 1 |
    
    static float mat_R_diagonal_elements[3] = {30, 25, 35};
    //according to mat_R_diagonal_elements
    //matrix R will be like this:
    //| 30 0 0 |
    //| 0 25 0 |
    //| 0 0 35 |
    
    Kalman_Filter_Init(&Height_KF, 3, 0, 3);
    
    memcpy(Height_KF.P_data, P_Init, sizeof(P_Init));
    memcpy(Height_KF.A_data, A_Init, sizeof(A_Init));
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
    //infinite loop
    Kalman_Filter_Update(&Height_KF);
    vTaskDelay(ts);
}
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

