// 简单的LCD显示函数
void DisplayOvenStatus(void) {
    char line1[20];
    char line2[20];
    
    // 根据当前状态显示不同信息
    switch (current_state) {
        case OVEN_OFF:
            snprintf(line1, sizeof(line1), "烤箱已关闭");
            snprintf(line2, sizeof(line2), "");
            break;
            
        case OVEN_PREHEATING:
            snprintf(line1, sizeof(line1), "预热中: %.1f°C", HAL_ReadTemperature());
            snprintf(line2, sizeof(line2), "目标: %.1f°C", current_settings.target_temp);
            break;
            
        case OVEN_COOKING:
            {
                uint32_t elapsed = HAL_GetTime() - cooking_start_time;
                uint32_t remaining = current_settings.cook_time - elapsed;
                uint32_t minutes = remaining / 60;
                uint32_t seconds = remaining % 60;
                
                snprintf(line1, sizeof(line1), "烹饪中: %.1f°C", HAL_ReadTemperature());
                snprintf(line2, sizeof(line2), "%02lu:%02lu 剩余", minutes, seconds);
            }
            break;
            
        // 其他状态...
    }
    
    // 实际LCD显示代码（取决于具体硬件）
    LCD_Clear();
    LCD_Print(0, 0, line1);
    LCD_Print(0, 1, line2);
}

// 按钮处理函数
void HandleButtons(void) {
    static uint32_t last_button_time = 0;
    uint32_t current_time = HAL_GetTime();
    
    // 防抖处理
    if (current_time - last_button_time < 200) return; // 200ms防抖
    
    if (Button_IsPressed(BUTTON_START)) {
        last_button_time = current_time;
        if (current_state == OVEN_OFF) {
            // 从预设或当前设置启动
            StartOven();
        } else {
            // 暂停/继续
            ToggleOvenPause();
        }
    }
    
    if (Button_IsPressed(BUTTON_STOP)) {
        last_button_time = current_time;
        StopOven();
    }
    
    // 其他按钮处理...
}
