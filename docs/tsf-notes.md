# TSF: what the shell does, and why

The text service is jis-moonshot's (team-mirae/jis-moonshot `windows/src/text_service.cpp`), whose
behaviour was measured on Windows 11 25H2 in three kinds of control. Read its
[`docs/troubleshooting/windows-tsf-notes.md`](https://github.com/team-mirae/jis-moonshot/blob/main/docs/troubleshooting/windows-tsf-notes.md)
before changing `src/tip/text_service.cpp`: each of these was "fixed" the obvious way first, and the
obvious way broke typing. FKeyIME's [`docs/issues/`](https://github.com/themrsung/fkeyime/tree/main/docs/issues)
(archived) has the problems before that.

## Kept as measured there

| What | Why (where) |
|---|---|
| The engine runs in `OnTestKeyDown`; a key it doesn't consume is committed there and reported as not eaten | A key eaten in `OnTestKeyDown` never reaches the app (moonshot) |
| `OnKeyDown` takes the cached result, comparing virtual key, scan code and extended flag only | Its lParam has the repeat count cleared; IMM-layer apps call it for keys not eaten (moonshot) |
| A key-down tested twice runs the engine once (`SameKeyDown`) | Message loops that peek and test (moonshot #5) |
| IMM-layer (transitory) contexts get the composition as an interim selection, set on the first character | A plain caret ends the composition at once there (FKeyIME, moonshot) |
| Text the app refuses or ends right after our update stays as plain text and is replaced next key | Apps that end every composition (moonshot #6) |
| Edit sessions hold the service, the context and their composition, and never call the engine | Queued sessions could run after `Deactivate` (moonshot #7) |
| No context: the key is the app's, nothing stays staged | moonshot #3 |
| `VK_PACKET` commits first, then reaches the app | moonshot #4 |
| A virtual key with scan code 0 gets `MapVirtualKeyW`'s | Injected keys without scan codes (moonshot #9) |
| Modifiers alone never commit | FKeyIME `shift-splits-the-composition.md` |
| The open/close compartment only mirrors the mode; changes from elsewhere are put back | Windows closes it when a profile activates (moonshot) |
| Settings are re-read only when `SettingsVersion` changed; the key grants ALL APPLICATION PACKAGES read | moonshot #9; Store apps couldn't see the user's key (moonshot) |
| Background work holds the DLL until it returns (`FreeLibraryWhenCallbackReturns`) | The DLL unloaded under the preload (moonshot) |
| The menu window is excluded from capture | A certificate password showed in a recording (moonshot) |
| Password fields: ES_PASSWORD, password character, disabled keyboard, empty context, input scopes | moonshot |
| Restart apps after installing | They keep the old DLL, or none (FKeyIME `apps-dont-load-fkeyime.md`) |

## New here, not measured yet

| What | Why | Risk |
|---|---|---|
| The engine is a DLL loaded at run time from the manifest, and freed in `DllCanUnloadNow` | Any bada-compatible engine plugs in | Loader-lock rules: never free it in `DllMain` |
| The mode is a session-wide global compartment keyed by the CLSID, with the profile | One TIP for both modes (docs/spec.md §3) | Apps that are busy follow late |
| The language keys by scan code under the Japanese layout | docs/spec.md §4 | Apple keyboards' 英数 / かな scan codes are a guess |
| Caps Lock set back with an injected press | docs/spec.md §5 | `GetKeyState` timing; UIPI |
| The state table and its API | docs/api.md | AppContainer apps; secure-field detection at focus change |
| One profile under ja-JP | docs/spec.md §2.2 | Windows' Japanese IME hotkeys (半角/全角 toggles open/close) fighting the mirror |

Add what you measure to this file, with the date and how.
