You need to put all handlers of you sm_td macros into `on_smtd_action` function. It's called with following arguments:
- uint16_t keycode - keycode of the macro key you pressed
- smtd_action action - result interpreted action (SMTD_ACTION_TOUCH, SMTD_ACTION_TAP, SMTD_ACTION_HOLD, SMTD_ACTION_RELEASE). tap, hold and release are self-explanatory. Touch action fired on every key press (without knowing if it is a tap or hold).
- uint8_t tap_count - number of sequential taps before current action. (will reset after hold, pause or any other key press)

There are only two execution flow for `on_smtd_action` function:
- touch → tap 
- touch → hold → release

You will never get a tap between hold and release or miss a touch action. So be sure, once touch has been executed, it will be finished with tap or hold+release.

Possible structure to describe macros behaviours is nested switch-cases. Top level for choosing macro key, second for action, third (optional) for tap_count. For example:


```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case CUSTOM_KEYCODE_1: {
            switch (action) {
                case SMTD_ACTION_TOUCH:
                    break; // touch action are not used in this example  
                    
                case SMTD_ACTION_TAP: 
                    // this is a tap action for CUSTOM_KEYCODE_1
                    tap_code(KC_SPACE);
                    break;
                    
                case SMTD_ACTION_HOLD:
                    // this is a hold action for CUSTOM_KEYCODE_1
                    switch (tap_count) {
                        case 0:
                        case 1:
                            layer_move(4);
                            break;
                        default:
                            // sending hold for OS
                            register_code(KC_SPACE); 
                            break;
                    }
                    break;
                    
                case SMTD_ACTION_RELEASE:
                    // this is a release action for CUSTOM_KEYCODE_1
                    switch (tap_count) {
                        case 0:
                        case 1:
                            layer_move(0);
                            break;
                        default:
                            // releasing hold for OS
                            unregister_code(KC_SPACE);
                            break;
                    }
                    break;
            } // end of switch (action)
            break;
        } // end of case CUSTOM_KEYCODE_1
            
        // put all your custom keycodes here
        
    } // end of switch (keycode)
} // end of on_smtd_action function
```