There are two feature flags with type `smtd_feature`:

- `SMTD_FEATURE_AGGREGATE_TAPS`

  Default behavior of sm_td library is to call tap action every time it's considered as a tap. This option allows to aggregate taps and call tap action only once after tap sequence is finished (same as original QMK Tap Dance).


- `SMTD_FEATURE_MODS_RECALL`
  
  Since tap action may be executed after a small delay (not immediately after key press), modifiers might be changed in that period. This option saves modifiers on key press and restores in on tap action.


Each of that feature flags has it's default values:
- `SMTD_GLOBAL_MODS_RECALL` (default is true) 
- `SMTD_GLOBAL_AGGREGATE_TAPS` (default is false) 

There is also a handy function to recall default value for a feature `bool smtd_feature_enabled_default(smtd_feature feature)`

And you may also override feature flags per key with adding `bool smtd_feature_enabled(uint16_t keycode, smtd_feature feature)` function to your keymap.c:

```c
bool smtd_feature_enabled(uint16_t keycode, smtd_feature feature);
    switch (keycode) {
        case MACRO_KEY_CODE:
            if (timeout == SMTD_FEATURE_MODS_RECALL) return false;
    }

    return smtd_feature_enabled_default(feature);
}
```


Let's examine each of this features more closely.

## SMTD_FEATURE_AGGREGATE_TAPS

That emulates QMK's Tap Dance. If SMTD_FEATURE_AGGREGATE_TAPS = true, `on_smtd_action(CKC, SMTD_ACTION_TAP, XXX)` will be called once in the end of sequence of taps. So sm_td need a little pause after last tap to ensure, that sequence has finished, and sm_td may send final SMTD_ACTION_TAP event. See a table to see the difference:

```								 
                     | SMTD_FEATURE_AGGREGATE_TAPS = false         | SMTD_FEATURE_AGGREGATE_TAPS = true          |
                     |                                             |                                             |
0ms  - - - ┌—————┐ - | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
           │macro│   | on_smtd_action(macro, SMTD_ACTION_TOUCH, 0) | on_smtd_action(macro, SMTD_ACTION_TOUCH, 0) |
           │ key │   |                                             |                                             |
           │     │   |                                             |                                             |
10ms - - - └—————┘ - | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
                     | on_smtd_action(macro, SMTD_ACTION_TAP, 0)   |                                             |
                     |                                             |                                             |
20ms - - - ┌—————┐ - | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
           │macro│   | on_smtd_action(macro, SMTD_ACTION_TOUCH, 1) | on_smtd_action(macro, SMTD_ACTION_TOUCH, 1) |
           │ key │   |                                             |                                             |
           │     │   |                                             |                                             |
30ms - - - └—————┘ - | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
                     | on_smtd_action(macro, SMTD_ACTION_TAP, 1)   |                                             |
                     |                                             |                                             |
                     |                                             |                                             |
40ms - - - ┌—————┐ - | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
           │macro│   | on_smtd_action(macro, SMTD_ACTION_TOUCH, 2) | on_smtd_action(macro, SMTD_ACTION_TOUCH, 2) |
           │ key │   |                                             |                                             |
           │     │   |                                             |                                             |
50ms - - - └—————┘ - | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
                     | on_smtd_action(macro, SMTD_ACTION_TAP, 2)   |                                             |
                     |                                             |                                             |
                     |                                             |                                             |
50ms + SEQUENCE_TERM | - - - - - - - - - - - - - - - - - - - - - - | - - - - - - - - - - - - - - - - - - - - - - |
                     |                                             | on_smtd_action(macro, SMTD_ACTION_TAP, 2)   |
```

## SMTD_FEATURE_MODS_RECALL

Since sm_td send key press a bit later after real physical key press has occurred, that may lead to unexpected behaviour with mod keys. So, if you press a regular shift key (witch are not a part of sm_td) and press

