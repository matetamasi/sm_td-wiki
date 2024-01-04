## Three fingers roll

If you press 3 keys simultaneously smtd will interpret this as hold + hold + top (read 3.3: Deep Explanation: Three keys and states stack). That's what usually expected in 99% cases, but anyway that breaks fast 3+ fingers rolls while typing. It will be fixed in verison v.1.1.0


## Rattle modification key with Mods Recall feature

For example, you hit `↓shift` (that is not a part of smtd), `↓smtd_macro`, `↑shift` and `↑smtd_macro`. Actual tap of smtd_macro wouldn't be sent to OS until you physically release that key. So OS will receive that sequence: `↓shift`, `↑shift` (from physical pressing and releasing shift key), then `↓shift` (from mod recall), `↓↑smtd_macro_tap_action` and `↑shift`.

To overcome that issue you should put shift modifier as another macro for smtd, so smtd would be able to correctly handle intercepting states for two keys.


## Twice key pressed processing in pre_users phase by quantum