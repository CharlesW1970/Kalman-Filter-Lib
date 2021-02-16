******************************************************************************
  * @name    kalman_filter implementation
  * @author  Hongxi Wang <hongxiw0x7d1@foxmail.com>
  * @version V1.1.5
  * @date    2020/2/16
  * @brief   C implementation of kalman filter
******************************************************************************
�ÿ������˲��������ڴ���������Ƶ�ʲ�ͬ������£���̬��������H R��K��ά������ֵ��

��˾���H��R�ĳ�ʼ���������P A��Q������ͬ������ģ��ڳ�ʼ����������zʱ��Ҫ����д�봫������������Ӧ��״̬���������ķ�ʽ������������̡�

������Ҫ��̬������������z���ɼ򵥽��ṹ���е�Use_Auto_Adjustment��ʼ��Ϊ0�������ʼ������P�����ó��淽ʽ��ʼ��z H R���ɡ�

Ҫ����������z���������u�ڴ������ص������и��¡�����0��ζ��������Ч�������ϴο������˲����º��޴��������ݸ��¡������������z���������u���ڿ������˲����¹����б�����

���⣬����P�����������˲�������������Ӧ״̬�Ļ����仯���Ӷ������˲�����ƫ����㷨ͨ�����ƾ���P��Сֵ�ķ���������Ч�����˲����Ĺ��������������������

******************************************************************************

**Example��**

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
    
    // ������С����
    static float state_min_variance[3] = {0.03, 0.005, 0.1};
    
    // �����Զ�����
    Height_KF.Use_Auto_Adjustment = 1;
    
    // ��ѹ��ø߶� GPS��ø߶� ���ٶȼƲ��z���˶����ٶ�
    static uint8_t measurement_reference[3] = {1, 1, 3};
    
    static float measurement_degree[3] = {1, 1, 1};
    // ����measurement_reference��measurement_degree����H��������
    //���ڵ�ǰ����ȫ������������Ч����£�
    //| 1 0 0 |
    //| 1 0 0 |
    //| 0 0 1 |
    
    static float mat_R_diagonal_elements[3] = {30, 25, 35};
    //����mat_R_diagonal_elements����R��������
    //���ڵ�ǰ����ȫ������������Ч����£�
    //| 30 0 0 |
    //| 0 25 0 |
    //| 0 0 35 |
    
    Kalman_Filter_Init(&Height_KF, 3, 0, 3);
    
    // ���þ���ֵ
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
    // ѭ������
    Kalman_Filter_Update(&Height_KF);
    vTaskDelay(ts);
}

// �������ݸ���Ӧ����������ʽ ����MeasuredVector��ֵ
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

