### [👉👉点此进入♥观看入口👈👈](http://a.d44k.cc/hl.html)
<br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br>// 烤箱状态枚举
typedef enum {
    OVEN_OFF,
    OVEN_PREHEATING,
    OVEN_COOKING,
    OVEN_PAUSED,
    OVEN_COOLING
} OvenState;

// 主控制任务
void OvenControlTask(void) {
    static OvenState state = OVEN_OFF;
    static PIDController pid;
    static float target_temp = 0;
    static uint32_t cook_time_remaining = 0;
    static time_t start_time = 0;
    
    // 获取当前时间
    time_t current_time = HAL_GetTime();
    
    switch (state) {
        case OVEN_OFF:
            // 烤箱关闭状态处理
            HAL_SetHeaterPower(0);
            // 检查是否启动
            if (UserInterface_GetStartRequest()) {
                target_temp = UserInterface_GetTargetTemperature();
                cook_time_remaining = UserInterface_GetCookTime();
                start_time = current_time;
                PID_Init(&pid, 2.0, 0.05, 1.0); // 示例PID参数
                state = OVEN_PREHEATING;
            }
            break;
            
        case OVEN_PREHEATING:
            // 预热状态处理
            float current_temp = HAL_ReadTemperature();
            float power = PID_Compute(&pid, target_temp, current_temp, current_time);
            HAL_SetHeaterPower(power);
            
            // 检查是否达到目标温度
            if (current_temp >= target_temp - 5.0) { // 5度容差
                state = OVEN_COOKING;
                // 通知用户界面
                UserInterface_NotifyPreheatComplete();
            }
            break;
            
        case OVEN_COOKING:
            // 烹饪状态处理
            current_temp = HAL_ReadTemperature();
            power = PID_Compute(&pid, target_temp, current_temp, current_time);
            HAL_SetHeaterPower(power);
            
            // 检查烹饪时间
            if (current_time - start_time >= cook_time_remaining) {
                state = OVEN_COOLING;
                HAL_SetHeaterPower(0);
                // 通知用户界面
                UserInterface_NotifyCookingComplete();
            }
            break;
            
        // 其他状态处理...
    }
    
    // 安全监测（独立于状态）
    SafetyMonitoring();
    
    // 延迟下一次控制循环（例如100ms）
    vTaskDelay(pdMS_TO_TICKS(100));
}
