### [👉👉点此进入♥观看入口👈👈](http://a.d44k.cc/hl.html)
<br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br><br></br>// 预设菜单项结构体
typedef struct {
    char name[20];
    float temperature;
    uint32_t duration; // 秒
    uint8_t preheat;   // 是否需要预热
} PresetMenuItem;

// 预设菜单数组
const PresetMenuItem preset_menu[] = {
    {"披萨", 220.0, 900, 1},
    {"饼干", 180.0, 720, 1},
    {"烤鸡", 200.0, 3600, 1},
    {"解冻", 50.0, 1800, 0},
    // 更多预设...
};

#define NUM_PRESETS (sizeof(preset_menu) / sizeof(preset_menu[0]))

// 获取预设菜单项
const PresetMenuItem* GetPresetMenuItem(uint8_t index) {
    if (index < NUM_PRESETS) {
        return &preset_menu[index];
    }
    return NULL;
}
