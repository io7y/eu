// 用户设置结构体
typedef struct {
    float target_temp;
    uint32_t cook_time;
    uint8_t preset_index;
    uint8_t sound_enabled;
    uint8_t unit_celsius; // 1=摄氏度, 0=华氏度
} OvenSettings;

// 全局用户设置（实际应用中应使用EEPROM持久化）
static OvenSettings current_settings = {
    .target_temp = 180.0,
    .cook_time = 600, // 10分钟
    .preset_index = 0,
    .sound_enabled = 1,
    .unit_celsius = 1
};

// 加载默认设置
void LoadDefaultSettings(void) {
    // 从EEPROM或默认值加载
    // 这里简化为直接初始化
    memcpy(&current_settings, &(OvenSettings){
        .target_temp = 180.0,
        .cook_time = 600,
        .preset_index = 0,
        .sound_enabled = 1,
        .unit_celsius = 1
    }, sizeof(OvenSettings));
}

// 保存设置到EEPROM（伪代码）
void SaveSettingsToEEPROM(void) {
    // EEPROM_Write(EEPROM_SETTINGS_ADDR, &current_settings, sizeof(OvenSettings));
}
