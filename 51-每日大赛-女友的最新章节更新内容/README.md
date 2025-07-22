// PID控制器结构体
typedef struct {
    float Kp;           // 比例增益
    float Ki;           // 积分增益
    float Kd;           // 微分增益
    float integral;     // 积分项
    float prev_error;   // 上一次误差
    time_t prev_time;   // 上一次采样时间
} PIDController;

// 初始化PID控制器
void PID_Init(PIDController *pid, float Kp, float Ki, float Kd) {
    pid->Kp = Kp;
    pid->Ki = Ki;
    pid->Kd = Kd;
    pid->integral = 0;
    pid->prev_error = 0;
    pid->prev_time = 0;
}

// 计算PID输出
float PID_Compute(PIDController *pid, float setpoint, float current_temp, time_t current_time) {
    float error = setpoint - current_temp;
    
    // 计算时间差（假设系统定期调用此函数）
    float dt = (pid->prev_time == 0) ? 1.0 : (current_time - pid->prev_time) / 1000.0;
    pid->prev_time = current_time;
    
    // 比例项
    float proportional = pid->Kp * error;
    
    // 积分项（带抗饱和）
    pid->integral += error * dt;
    // 积分限幅
    if (pid->integral > 100.0) pid->integral = 100.0;
    if (pid->integral < -100.0) pid->integral = -100.0;
    float integral = pid->Ki * pid->integral;
    
    // 微分项
    float derivative = pid->Kd * (error - pid->prev_error) / dt;
    pid->prev_error = error;
    
    // 计算总输出（限制在0-100%功率范围内）
    float output = proportional + integral + derivative;
    if (output > 100.0) output = 100.0;
    if (output < 0.0) output = 0.0;
    
    return output;
}
