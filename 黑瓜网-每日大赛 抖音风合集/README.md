// MQTT主题定义
#define OVEN_CMD_TOPIC      "home/oven/cmd"
#define OVEN_STATUS_TOPIC   "home/oven/status"

// MQTT状态消息结构
typedef struct {
    char state[16];
    float current_temp;
    float target_temp;
    uint32_t time_remaining;
} OvenStatusMessage;

// 处理来自应用的命令
void ProcessAppCommand(const char* command) {
    // 解析JSON命令（简化示例）
    if (strstr(command, "\"start\"")) {
        // 提取参数并启动烤箱
        float temp = ExtractFloatParam(command, "temperature");
        uint32_t time = ExtractIntParam(command, "duration");
        
        if (temp > 0 && time > 0) {
            current_settings.target_temp = temp;
            current_settings.cook_time = time;
            StartOven();
        }
    } 
    else if (strstr(command, "\"stop\"")) {
        StopOven();
    }
    // 其他命令处理...
}

// 发送状态更新到应用
void SendStatusUpdate(void) {
    OvenStatusMessage status;
    
    switch (current_state) {
        case OVEN_OFF:
            strncpy(status.state, "off", sizeof(status.state));
            break;
        case OVEN_PREHEATING:
            strncpy(status.state, "preheating", sizeof(status.state));
            break;
        case OVEN_COOKING:
            strncpy(status.state, "cooking", sizeof(status.state));
            break;
        // 其他状态...
    }
    
    status.current_temp = HAL_ReadTemperature();
    status.target_temp = current_settings.target_temp;
    
    if (current_state == OVEN_COOKING) {
        uint32_t elapsed = HAL_GetTime() - cooking_start_time;
        status.time_remaining = current_settings.cook_time - elapsed;
    } else {
        status.time_remaining = 0;
    }
    
    // 序列化为JSON并发布（伪代码）
    char json[256];
    snprintf(json, sizeof(json), 
             "{\"state\":\"%s\",\"current_temp\":%.1f,\"target_temp\":%.1f,\"time_remaining\":%lu}",
             status.state, status.current_temp, status.target_temp, status.time_remaining);
    
    MQTT_Publish(OVEN_STATUS_TOPIC, json);
}
