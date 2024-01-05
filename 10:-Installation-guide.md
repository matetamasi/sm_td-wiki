1. Clone `sm_td.h` repository into your `keymaps/your_keymap` folder (next to your keymap.c)
2. Add `#include "sm_td.h"` to your `keymap.c` file
3. Check `!process_smtd` first in your `process_record_user` function like this
```c
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
    if (!process_smtd(keycode, record)) {
        return false;
    }
    // your code here
}
```
4. Add some custom keycodes to your `keymaps`, so sm_td library can use them without conflicts
5. Declare variable `smtd_states` your custom keycodes for sm_td library like this:
```c
smtd_state smtd_states[] = {
    SMTD(CUSTOM_KEYCODE_1),
    SMTD(CUSTOM_KEYCODE_2),
    // put all your custom keycodes here
}

// this is the size of your custom keycodes array, it is used for internal purposes. Do not delete this
size_t smtd_states_size = sizeof(smtd_states) / sizeof(smtd_states[0]);
```
6. Describe your custom keycodes in `on_smtd_action` function like this:
```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case CKC_SPACE: {
            switch (CUSTOM_KEYCODE_1) {
                case SMTD_ACTION_TOUCH:
                    break; // touch action are not used in this example  
                case SMTD_ACTION_TAP:
                    // this is a tap action for CUSTOM_KEYCODE_1
                    tap_code(KC_SPACE);
                    break;
                case SMTD_ACTION_HOLD:
                    // this is a hold action for CUSTOM_KEYCODE_1
                    if (tap_count == 0 || tap_count == 1) {
                        layer_move(4);
                    } else {
                        // sending hold for OS
                        register_code(KC_SPACE); 
                    }
                    break;
                case SMTD_ACTION_RELEASE:
                    // this is a release action for CUSTOM_KEYCODE_1
                    if (tap_count == 0 || tap_count == 1) {
                        layer_move(0);
                    } else {
                        // releasing hold for OS
                        unregister_code(KC_SPACE);
                    }
                    break;
            }
            break;
        } // end of case CKC_SPACE
            
        // put all your custom keycodes here
        
    } // end of switch (keycode)
} // end of on_smtd_action function
```