<div class="chapter-hero">
  <span class="chapter-badge">Chapter 08 • Hardware Input</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Controls & Input Mapping</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Polling front/rear capacitive touch surfaces, physical buttons, and dual analog sticks. Preventing silent heap corruption caused by Sony raw touch report IDs, and synthesizing touch events from physical gamepad inputs.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Level: Intermediate</span>
    <span class="chapter-meta-pill">🧠 Focus: sceTouch, sceCtrl, Touch-to-Pad, Heap Safety</span>
    <span class="chapter-meta-pill">🎮 Verified: Zenonia 2/3/4, Destinia, Hero of Sparta</span>
  </div>
</div>

---

## 🎮 Polling Hardware Controls in VitaOS

```mermaid
graph TD
    subgraph "PS Vita Hardware"
        FrontTouch["Front Capacitive OLED/LCD"]
        BackTouch["Rear Touchpad"]
        Buttons["Physical Buttons (D-Pad, Cross, Square, L/R)"]
        Sticks["Dual Analog Sticks"]
    end

    subgraph "VitaOS APIs"
        sceTouch["sceTouchPeek(SCE_TOUCH_PORT_FRONT)"]
        sceCtrl["sceCtrlPeekBufferPositive()"]
    end

    subgraph "Loader Abstraction Layer"
        Sanitizer["Touch ID Sanitizer (Prevents Heap Corruption)"]
        TouchMapper["Physical-to-Touch Synthetic Mapper"]
        Deadzone["Analog Deadzone Filter"]
    end

    subgraph "Game Engine"
        NativeTouch["nativeOnTouch(action, x, y, id)"]
        NativeKey["nativeOnKeyDown(keyCode)"]
    end

    FrontTouch --> sceTouch --> Sanitizer --> NativeTouch
    Buttons --> sceCtrl --> TouchMapper --> NativeTouch
    Buttons --> sceCtrl --> NativeKey
    Sticks --> sceCtrl --> Deadzone --> TouchMapper
```

### 1. Mandatory Touch Sampling Initialization
By default in VitaOS, touch sampling is **disabled** to save power. Without enabling it, `sceTouchPeek` will always report zero touches:

```c
#include <psp2/touch.h>

void init_touch_system(void) {
    sceTouchSetSamplingState(SCE_TOUCH_PORT_FRONT, SCE_TOUCH_SAMPLING_STATE_START);
    sceTouchSetSamplingState(SCE_TOUCH_PORT_BACK, SCE_TOUCH_SAMPLING_STATE_START);
}
```

---

## 💣 The Touch Report ID Heap Corruption Trap

### The Problem:
Sony's `SceTouchReport` struct exposes a hardware-assigned `id`:
```c
SceTouchData touch;
sceTouchPeek(SCE_TOUCH_PORT_FRONT, &touch, 1);
uint8_t raw_id = touch.report[i].id; // Can be 15, 48, 120...
```

Android engines (like Cocos2d-x or Gameloft engines) allocate a fixed-size internal array for active fingers:
```cpp
// Internal C++ game engine code:
Touch active_touches[5];

void onTouchDown(int pointer_id, float x, float y) {
    active_touches[pointer_id].x = x; // ❌ CRASH IF pointer_id > 4
}
```

If you pass Sony's raw `report.id` (e.g. 15) to the engine:
The engine accesses `&active_touches[15]`, **writing out of bounds and silently corrupting the application heap**. The game will not crash immediately, but will abort minutes later during unrelated heap operations.

### ✅ The Production Solution: Virtual Pointer ID Pool

=== "Virtual Touch ID Allocator (input.c)"
    ```c
    #define MAX_TOUCH_SLOTS 4

    static int virtual_touch_slots[MAX_TOUCH_SLOTS] = { -1, -1, -1, -1 };

    int get_virtual_touch_id(int sony_raw_id) {
        // 1. If already mapped, return existing slot
        for (int i = 0; i < MAX_TOUCH_SLOTS; i++) {
            if (virtual_touch_slots[i] == sony_raw_id) return i;
        }
        // 2. Allocate new free slot
        for (int i = 0; i < MAX_TOUCH_SLOTS; i++) {
            if (virtual_touch_slots[i] == -1) {
                virtual_touch_slots[i] = sony_raw_id;
                return i;
            }
        }
        return 0; // Fallback
    }

    void release_virtual_touch_id(int sony_raw_id) {
        for (int i = 0; i < MAX_TOUCH_SLOTS; i++) {
            if (virtual_touch_slots[i] == sony_raw_id) {
                virtual_touch_slots[i] = -1;
                break;
            }
        }
    }
    ```

=== "Polling Loop with Sanitization"
    ```c
    void poll_touch_events(void) {
        SceTouchData touch;
        sceTouchPeek(SCE_TOUCH_PORT_FRONT, &touch, 1);

        for (int i = 0; i < touch.reportNum; i++) {
            // Coordinate mapping: Vita touch sensor is 1920x1088
            float x = (float)touch.report[i].x * 960.0f / 1920.0f;
            float y = (float)touch.report[i].y * 544.0f / 1088.0f;
            
            // Map Sony raw ID to 0..3 safe slot
            int safe_id = get_virtual_touch_id(touch.report[i].id);

            // Dispatch to native engine (0 = ACTION_DOWN, 2 = ACTION_MOVE)
            nativeOnTouch(2, x, y, safe_id);
        }
    }
    ```

---

## 🕹️ Mapping Physical Buttons to On-Screen Touch

For mobile games without controller support:

=== "Touch-to-Pad Implementation"
    ```c
    #include <psp2/ctrl.h>

    typedef struct {
        uint32_t vita_button;
        float screen_x;
        float screen_y;
        int is_pressed;
    } VirtualButton;

    static VirtualButton v_buttons[] = {
        { SCE_CTRL_CROSS,    860.0f, 440.0f, 0 }, // Action / Attack
        { SCE_CTRL_SQUARE,   780.0f, 380.0f, 0 }, // Skill / Special
        { SCE_CTRL_TRIANGLE, 840.0f, 300.0f, 0 }, // Inventory / Menu
        { SCE_CTRL_CIRCLE,   910.0f, 370.0f, 0 }, // Dodge / Roll
    };

    void process_touch_emulation(uint32_t current_buttons) {
        for (int i = 0; i < sizeof(v_buttons)/sizeof(v_buttons[0]); i++) {
            VirtualButton *vb = &v_buttons[i];
            int pressed = (current_buttons & vb->vita_button) != 0;
            
            if (pressed && !vb->is_pressed) {
                // Synthetic touch down using slot 9+i to avoid slot collisions
                nativeOnTouch(0, vb->screen_x, vb->screen_y, 9 + i);
                vb->is_pressed = 1;
            } else if (!pressed && vb->is_pressed) {
                // Synthetic touch up
                nativeOnTouch(1, vb->screen_x, vb->screen_y, 9 + i);
                vb->is_pressed = 0;
            }
        }
    }
    ```

---

## 🎯 Analog Stick Deadzone Filtering

Vita analog sticks report unsigned bytes `0..255` centered at `128`:

=== "Analog Normalization"
    ```c
    #define ANALOG_DEADZONE 30

    void read_analog_stick(uint8_t raw_x, uint8_t raw_y, float *out_x, float *out_y) {
        int dx = (int)raw_x - 128;
        int dy = (int)raw_y - 128;
        
        if (abs(dx) < ANALOG_DEADZONE) dx = 0;
        if (abs(dy) < ANALOG_DEADZONE) dy = 0;
        
        *out_x = (float)dx / 128.0f;
        *out_y = (float)dy / 128.0f;
    }
    ```
