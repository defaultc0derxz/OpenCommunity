# Code Review §5 — Ofuscacao (branch feat/obfuscate-strings)

## 5.1 Seguranca — tolerancia zero

| Item | Verificacao | Resultado |
|------|-------------|-----------|
| Sem `std::string` em repouso p/ nomes | `EncString m_Name` (Module.h:152), `EncString name/buttonLabel/playerHeadName` (ModuleOption.h:12,26,29) | PASS |
| `MODULE_INFO` com `XOR()` | 15/15 usos com `XOR("Nome")`, `XOR("Desc")`; 0 literais crus | PASS |
| Sem `GetName()` legado | 0 ocorrencias em runtime/shared/launcher | PASS |
| `WipeAllXorStrings()` no unload | `Main.cpp:200` (DETACH, apos scrubs) + `Notifications.cpp` apos Send + `Screen.cpp:3839` | PASS |
| `ScrubAllTextBuffers` + `ScrubNotifications` + `ScrubRuntimeUiTextState` + `ScrubUiTextState` | Main.cpp:71,197-199; Notifications.h:69; Screen.h:53 / Screen.cpp:3831,3838 | PASS |
| `ClearString` vs `clear()` p/ strings com nome | Notifications usa `Clear()` (Wipe) + `ClearString` em transitorios; HUD entries transitorias por frame; module names nunca em `std::string` persistente | PASS |
| Dump limpo (estatico) | `validation_static.txt`: 0/33 identificadores no runtime.dll | PASS |
| Dump limpo (dinamico) | `validation_dynamic.txt`: identificadores unicos 0 hits em 1.5GB do javaw; residuos genericos provados como JVM (`java/util/ArrayList`) | PASS com ressalva |
| RTTI | `RuntimeTypeInfo=false` (runtime.vcxproj:71,94); sem `typeid/dynamic_cast` no codigo | PASS |

Ressalvas:
- `Target`, `ArrayList`, `Notifications` sao palavras genericas e colidem com strings da JVM (`java/util/ArrayList`); gate de 0 hits vale para identificadores unicos + `.rdata` do runtime.dll (0 hits comprovado).
- Runtime validado com `PlatformToolset=v143` (VS instalado); projeto mantido em `v145` (original). Codigo compativel C++20, sem dependencia de toolset.

## 5.2 Clean code
- Nomes: `ScopedName`, `EncString`, `Scrub*` — PASS.
- Funcoes pequenas: `GetNameInto` (1 coisa), `ScrubTextBuffers` (so Wipe) — PASS.
- Sem duplicacao: `HashName` (Module) + `Fnv1aHash` (Notifications) + `ModuleNameStr/OptionNameStr` (CommandManager) centralizados por uso — PASS.
- Sem abstracao especulativa: `EncString` concreto — PASS.
- Includes minimos: `Module.h` -> `ModuleOption.h` + security; `Main.cpp` + `imgui_internal.h` so p/ scrub — PASS.

## 5.3 Nao-regressao
- [x] Build Release passa (runtime.dll + launcher.exe, v143 override) — G1
- [ ] Menu/toggles/notificacoes funcionam igual — exige teste manual com injecao (nao executado aqui)
- [ ] `Tick` sem subir >1ms — exige profiling em jogo (nao executado aqui)
