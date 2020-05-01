/**
******************************************************************************
  * @name    kalman_filter implementation
  * @author  Hongxi Wong <hongxiw0x7d1@foxmail.com>
  * @version V1.0.5
  * @date    2020/5/1
  * @brief   
******************************************************************************
  * @attention 
  * �ÿ������˲��������ڴ���������Ƶ�ʲ�ͬ������£���̬��������H R��K��ά������ֵ��
  * This implementation of kalman filter can dynamically adjust dimension and  
  * value of matrix H R and K according to the measurement validity under any 
  * circumstance that the sampling rate of component sensors are different.
  * 
  * ��˾���H��R�ĳ�ʼ���������P A��Q������ͬ������ģ��ڳ�ʼ����������zʱ��Ҫ����д
  * �봫������������Ӧ��״̬���������ķ�ʽ���������c�ļ�������
  * Therefore, the initialization of matrix P, A, and Q is sometimes different 
  * from that of matrix H R. when initialization. Additionally, the corresponding 
  * state and the method of the measurement should be provided when initializing 
  * measurement vector z. For more details, please see the example in kalman filter.c 
  * 
  * ������Ҫ��̬������������z���ɼ򵥽��ṹ���е�Use_Auto_Adjustment��ʼ��Ϊ0�������
  * ʼ������P�����ó��淽ʽ��ʼ��z H R���ɡ�
  * If automatic adjustment is not required, assign zero to the Use_Auto_Adjustment 
  * and initialize z H R in the normal way as matrix P.
  * 
  * Ҫ����������z���������u�ڴ������ص������и��¡�����0��ζ��������Ч�������ϴο�����
  * �˲����º��޴��������ݸ��¡������������z���������u���ڿ������˲����¹����б�����
  * Measured_Vector and Control_Vector are required to be updated in the sensor 
  * callback function. Integer 0 in measurement vector z indicates the invalidity 
  * of current measurement, so Measured_Vector and Control_Vector will be reset 
  * (to 0) during each update. 
******************************************************************************
  */
