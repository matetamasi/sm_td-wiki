## Emulate LT(LAYER, KEY)

```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case emulate_lt_macro_key: {
            switch (action) {
                case SMTD_ACTION_TOUCH:
                    break;

                case SMTD_ACTION_TAP:
                    tap_code(KEY);
                    break;
                    
                case SMTD_ACTION_HOLD:
                    switch (tap_count) {
                        case 0:
                        case 1:
                            layer_move(LAYER);
                            break;
                        default:
                            register_code(KEY); 
                            break;
                    }
                    break;
                    
                case SMTD_ACTION_RELEASE:
                    switch (tap_count) {
                        case 0:
                        case 1:
                            layer_move(0);
                            break;
                        default:
                            unregister_code(KEY);
                            break;
                    }
                    break;
            } // end of switch (action)
            break;
                    
    } // end of switch (keycode)
} // end of on_smtd_action function
```

So, if sm_td register a tap, it will send KEY tap.
If sm_td registers a hold, it will switch to LAYER.
But if it's a hold after two sequential taps, it will send KEY press and will be pressing until a macro key will be physically released.




## Emulate MT(MOD, KEY)

```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case emulate_mt_macro_key: {                                           
            switch (action) {                                     
                case SMTD_ACTION_TOUCH:                           
                    break;    
                                                        
                case SMTD_ACTION_TAP:                             
                    tap_code16(KEY);                          
                    break;

                case SMTD_ACTION_HOLD:
                    switch (tap_count) {
                        case 0:
                        case 1:
                            register_mods(MOD_BIT(MOD)); 
                            break;
                        default:
                            register_code16(KEY);                 
                            break;
                    }                            
                    break;

                case SMTD_ACTION_RELEASE:                         
                    switch (tap_count) {
                        case 0:
                        case 1:
                            unregister_mods(MOD_BIT(MOD));
                            break;
                        default:
                            unregister_code16(KEY);
                            break;
                    }             
                    break;                                        
              } // end of switch (keycode)
} // end of on_smtd_action function
```

Almost the same with previous example



## Responsive MT(MOD, KEY)

```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case resposive_mt_macro_key: {                                           
            switch (action) {                                     
                case SMTD_ACTION_TOUCH:                           
                    switch (tap_count) {
                        case 0:
                        case 1:
                            register_mods(MOD_BIT(MOD)); 
                            break;
                        default:
                            register_code16(KEY);
                            break;
                    }
                    break;    

                case SMTD_ACTION_TAP: 
                    unregister_mods(MOD_BIT(MOD));
                    tap_code16(KEY);           
                    break;

                case SMTD_ACTION_HOLD:
                    // no need to register anything since it was already registered in SMTD_ACTION_TOUCH
                    break;

                case SMTD_ACTION_RELEASE:                         
                    switch (tap_count) {
                        case 0:
                        case 1:
                            unregister_mods(MOD_BIT(MOD));
                            break;
                        default:
                            unregister_code16(KEY);
                            break;
                    }             
                    break;                                        
              } // end of switch (keycode)
} // end of on_smtd_action function
```

In this case MOD is registered as soon as key was pressed. That might be handy, if you MOD with mouse click, so you don't need before keyboard will interpret that key is held.
So, on touch action sm_td will register MOD, and it should be unregistered on tap action.



## Responsive symbol change on sequential tapping

For example, you to alternate between `:`, `;` and `#` on each macro key press.
```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case resposive_seq_macro_key: {                                           
            switch (action) {                                     
                case SMTD_ACTION_TOUCH: 
                    if (tap_count > 0) {
                        tap_code16(KC_BSPACE);
                    }                         

                    switch (tap_count % 3) {
                        case 0: register_code16(KC_COLON); break;
                        case 1: register_code16(KC_SEMICOLON); break;
                        case 2: register_code16(KC_HASH); break;
                        default: break;
                    }
                    break;    

                case SMTD_ACTION_TAP: 
                case SMTD_ACTION_HOLD:
                case SMTD_ACTION_RELEASE:                         
                    break;                                        
              } // end of switch (keycode)
} // end of on_smtd_action function
```

So, the first press will send `:` and sequential presses will delete just entered symbol and send next


## Clumsy symbol pick after sequential tapping (same as QMK Tap Dance)

```c
void on_smtd_action(uint16_t keycode, smtd_action action, uint8_t tap_count) {
    switch (keycode) {
        case clumsy_seq_macro_key: {                                           
            switch (action) {                                     
                case SMTD_ACTION_TOUCH: 
                    break;    
                case SMTD_ACTION_TAP: 
                    switch (tap_count % 3) {
                        case 0: register_code16(KC_COLON); break;
                        case 1: register_code16(KC_SEMICOLON); break;
                        case 2: register_code16(KC_HASH); break;
                        default: break;
                    }
                    break
                case SMTD_ACTION_HOLD:
                case SMTD_ACTION_RELEASE:                         
                    break;                                        
              } // end of switch (keycode)
} // end of on_smtd_action function

bool smtd_feature_enabled_default(uint16_t keycode, smtd_feature feature) {
    if (keycode == clumsy_seq_macro_key && feature == SMTD_FEATURE_AGGREGATE_TAPS) return true;
    
    return smtd_feature_enabled_default(keycode, feature); 
}
```
In this case SMTD_ACTION_TAP wouldn't be sent until sm_td is 100% sure that you have finished your tap sequence. So SMTD_ACTION_TAP will be executed exactly once for all sequence.