// 安全监测任务
void SafetyMonitoringTask(void) {
    static uint32_t last_check_time = 0;
    uint32_t current_time = HAL_GetTime();
    
    // 每500ms检查一次
    if (current_time - last_check_time < 500) return;
    last_check_time = current_time;
    
    // 1. 过热保护
    float current_temp = HAL_ReadTemperature();
    if (current_temp > 300.0) { // 300°C安全阈值
        EmergencyShutdown("过热保护触发");
        return;
    }
    
    // 2. 门开关检测
    if (Door_IsOpen() && current_state != OVEN_OFF) {
        // 根据设置决定是否暂停或停止
        if (current_settings.pause_on_door_open) {
            PauseOven();
        } else {
            StopOven();
        }
    }
    
    // 3. 加热元件故障检测
    static float last_temp = 0;
    float temp_change = fabs(current_temp - last_temp);
    last_temp = current_temp;
    
    // 如果温度在全功率下不上升，可能有故障
    if (HAL_GetHeaterPower() > 90.0 && temp_change < 2.0) {
        Fault_Log(FAULT_HEATER_FAILURE);
        EmergencyShutdown("加热元件故障");
    }
}
