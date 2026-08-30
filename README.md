# Dino Party — Offline Co-op Mod

Mod for **Dino Party** (Unity 6000 IL2CPP + Photon Fusion 2 + Steamworks) that enables **Couch Co-op offline play** (2–4 players on one machine) without Steam authentication or Photon Cloud.

**Status: ✅ WORKING** — Custom Party button is redirected to a hidden offline mode that the developers shipped but never exposed in the UI.

---

## Table of Contents

1. [What This Project Does](#what-this-project-does)
2. [How to Play (Offline Couch Co-op)](#how-to-play-offline-couch-co-op)
3. [How to Play Over the Internet (Steam Remote Play Together)](#how-to-play-over-the-internet-steam-remote-play-together)
4. [How the Mod Works](#how-the-mod-works)
5. [Project Structure](#project-structure)
6. [What Was Achieved (5 Sessions)](#what-was-achieved-5-sessions)
7. [Known Limitations](#known-limitations)
8. [Credits](#credits)
9. [Reverse Engineering Playbook](#reverse-engineering-playbook)

---

## What This Project Does

Dino Party is a Unity IL2CPP game that uses **Photon Fusion 2** for online multiplayer. All online modes (Public + Custom parties) route through Photon Cloud, which requires:
- Steam authentication (game sends a Steam session ticket as custom auth)
- Photon Cloud server-side plugin `FusionPlugin21` (not available for self-hosting)

This project bypasses both by:
1. **Goldberg Steam Emulator** — fakes the Steamworks API so the game boots without Steam
2. **BepInEx + Harmony mod** — hooks the Custom Party button and redirects it to a **hidden offline mode** (`CreateOfflineGame()`) that exists in the game code but was never exposed in the UI

Result: **2–4 players can play couch co-op on a single machine with controllers, no internet required.**

---

## 📥 Download

> ### go to my github profile, on readme.md access my page there you will have the download link of full game playable
>
> Full game + mod v0.11.0 — content all file to run.
> please read file **This file** before playing.

---

## How to Play (Offline Couch Co-op)

### Requirements
- 1 machine with Dino Party (modded, see below)
- 2–4 USB controllers (Xbox / PlayStation / Switch Pro)
- No internet connection needed

### Setup (one-time)

**Prerequisites:** the game folder must already contain:
- `Dino Party_Data/Plugins/x86_64/steam_api64.dll` — Goldberg Steam Emulator build
- `Dino Party_Data/Plugins/x86_64/steam_settings/steam_appid.txt` — contains `1939390`
- `Dino Party_Data/Plugins/x86_64/steam_settings/account_name.txt` — any name (e.g. `PlayerOne`)
- `BepInEx/plugins/DinoLANMod.dll` — the mod (v0.11.0-offline)
- `dotnet/` — .NET 6 runtime required by BepInEx

### Steps

1. Plug 2–4 USB controllers into the machine
2. Launch `Dino Party.exe`
3. Wait for Goldberg to load → log shows `Success, Steam is ready to use`
4. Main Menu → **Party** → **Host Party** → **Custom** (purple button)
5. The mod redirects into **"Private - OFFLINE"** lobby
6. Screen shows 4 slots: `Press (A) or [Esc] to join`
7. Each controller presses **A** (Xbox) / **Cross** (PS) to join a slot
8. Pick a dino → Starter Outfit → play together on one screen

---

## How to Play Over the Internet (Steam Remote Play Together)

> Steam blocks Remote Play Together for non-Steam games. Use the tool below to bypass this.

### Tool
**[Remote-Play-Together-for-nonSteam-games](https://github.com/Counter-clock/Remote-Play-Together-for-nonSteam-games)** — lets you use Steam Remote Play Together with *any* game on your PC by masquerading as a free Steam game.

### Setup

**Step 1:** Download `RemotePlayTogether.exe` from the [Releases](https://github.com/Counter-clock/Remote-Play-Together-for-nonSteam-games/releases) page.

**Step 2:** Install **any Steam game that supports Remote Play Together** — the tool only needs it as a "host" (you never actually play it). A free, small option is RetroArch: https://store.steampowered.com/app/1118310/RetroArch/

**Step 3:** Replace that game's main executable with the tool (example uses RetroArch's `retroarch.exe` — substitute with your chosen game's exe):
1. Steam → Library → the game → right-click → Properties → Installed Files → Browse
2. Rename the original main executable → `<original>.exe.bak` (e.g. `retroarch.exe` → `retroarch.exe.bak`)
3. Copy `RemotePlayTogether.exe` into that folder
4. Rename `RemotePlayTogether.exe` → the original executable's name (e.g. `retroarch.exe`)

**Step 4:** Create `remoteplay.txt` in the same folder:
```
D:\ProjectStorage\Dino Party\Dino Party.exe | Dino Party
```
Format: one game per line — `<path to exe> | <display name>` (the `| Name` part is optional).

**Step 5:** Launch the host game from Steam:
1. Steam → Library → the host game → **Play**
2. The tool creates `remoteplay.txt` automatically (if missing) and shows a menu
3. Select **Dino Party**
4. The game launches (Goldberg loads automatically)

**Step 6:** Invite your friend:
1. In-game: **Shift+Tab** to open Steam Overlay
2. Right-click a friend → **Remote Play Together** → **Send Invite**
3. Friend accepts → Steam streams your screen and forwards their input

**Step 7:** Play together:
1. Host: Main Menu → Party → Host Party → **Custom** (mod redirects to offline mode)
2. Host uses controller 1 (or keyboard); friend uses controller 2 via Steam Input

### Troubleshooting

| Problem | Fix |
|---------|-----|
| "Remote Play Together" grayed out | Launch the host game from Steam Library, not the exe directly |
| Steam says game doesn't support RPT | Verify the host game's exe was actually replaced with the tool (~447 KB binary) |
| Tool menu appears but Dino Party won't launch | Check the path in `remoteplay.txt` |
| Friend doesn't receive invite | Add them as Steam Friend first |
| Stream OK but input doesn't reach game | Steam → Settings → Controller → enable Steam Input |
| High latency | Use LAN if possible; lower streaming quality; enable GPU encoding |
| Game requires admin rights | Run Steam as administrator |

---

## How the Mod Works

### Original flow (no mod)
```
Main Menu → Party → Host Party → Custom
    ↓
Networking.HostGame(isPublic=false)
    ↓
Photon Cloud → Auth → JoinLobby → CreateRoom → Fusion handshake
    ↓
FAIL (no Steam auth + no FusionPlugin21 on self-hosted servers)
```

### Flow with mod v0.11.0-offline
```
Main Menu → Party → Host Party → Custom
    ↓
Networking.HostGame(isPublic=false)
    ↓ (Harmony prefix intercept)
RedirectPatch.OfflineRedirect:
    ├── isPublic=true  → return true (Public button unchanged)
    └── isPublic=false → return false (skip) + call CreateOfflineGame()
    ↓
Couch co-op local mode (OfflineParty)
    ↓
"Private - OFFLINE" lobby → 4 slots → 2–4 players, one screen
```

### Core patch (simplified)
```csharp
// Tools/DinoLANMod/Plugin.cs
public static bool OfflineRedirect(ref bool isPublic) {
    if (isPublic) return true;  // Public button: keep original behavior
    var networking = UnityEngine.Object.FindObjectOfType<Networking>();
    networking.CreateOfflineGame();  // Hidden offline mode shipped but unused!
    return false;  // skip original
}
```

**Key insight:** the game developers shipped `Networking.CreateOfflineGame()` and an `OfflineParty` game-type enum, but the UI never exposes them — the Custom button always calls `HostGame(isPublic=false)` which goes online. The mod simply routes the Custom button to the hidden offline path.

---

## Project Structure

```
Dino Party/
├── Dino Party.exe                          ← Game executable
├── Dino Party_Data/
│   └── Plugins/x86_64/
│       ├── steam_api64.dll                 ← Goldberg Steam Emulator (bypasses Steam)
│       └── steam_settings/
│           ├── steam_appid.txt             ← 1939390 (Dino Party app ID)
│           └── account_name.txt            ← PlayerOne
├── BepInEx/
│   ├── LogOutput.log                       ← Mod log
│   └── plugins/
│       └── DinoLANMod.dll                  ← Mod v0.11.0-offline
├── dotnet/                                 ← .NET 6 runtime (BepInEx needs it)
├── Session1.md … Session5.md               ← Session logs (Vietnamese)
├── REMOTE_PLAY_GUIDE.md                    ← Remote Play setup guide
└── Tools/
    ├── DinoLANMod/
    │   ├── Plugin.cs                       ← Mod source
    │   └── DinoLANMod.csproj
    └── goldberg_emulator/                  ← Goldberg source (builds steam_api64.dll)
```

---

## What Was Achieved (5 Sessions)

### Session 1: Goldberg Steam Emulator
- Built Goldberg for Steamworks.NET 2025 (Flat API)
- Patches: `SteamInternal_SteamAPI_Init`, ~33 stub flat exports
- Game boots + `Success, Steam is ready to use`
- Discovered game uses Photon Fusion 2; party code = Fusion lobby code

### Session 2: Photon Server survey
- Redirected game to `127.0.0.1:5055` via mod
- Photon Server v4.0.28: patched protocol check (accepts 1.8) — but no GpBinaryV18 codec
- Photon Server v5: license blocked (strong-name signing) — abandoned
- LuxonServer (clean-room): game connects, auths, joins lobby, lists games — works!

### Session 3: LuxonServer + visualizer
- Built LuxonServer Debug with visualizer — sees every op code
- Corrected packet parsing (`0xF3` = GP_MAGIC, not op code 243)
- Room creation works on Master (`Creating game: ASxxx`)
- Game connects GameServer + sends Fusion Join — blocked at Fusion handshake

### Session 4: Fusion handshake root cause
- Mod hooking `SendJoin` (value-type params) crashes `NetworkProjectConfig.get_Global()` — key lesson
- After removing the hook: flow reaches Fusion Join echo; game receives it but sends Leave
- Reverse-engineered: game needs **FusionPlugin21** (native C++ plugin, Photon Cloud only) — not in any self-host SDK
- Discovered hidden `Networking.CreateOfflineGame()` + `NetworkGameType.OfflineParty` enum

### Session 5: SUCCESS
- Mod v0.11.0-offline hooks `HostGame(false)` → skip → `CreateOfflineGame()` → couch co-op local mode
- Tested: 2 players (Noob + Noob #2) playing together in the lava arena ✅

---

## Known Limitations

- ❌ **LAN multiplayer (two separate machines, real networking)** — impossible: Photon Fusion 2 requires `FusionPlugin21`, which exists only on Photon Cloud (licensed). No self-hosted Photon Server includes it.
- ❌ **Steam Remote Play Together directly** — Steam blocks non-Steam games (use the bypass tool above)
- ✅ **Couch co-op on one machine, 2–4 controllers** — working
- ✅ **Remote Play Together via a host Steam game stub** — works by design of the bypass tool

---

## Credits

| Tool | Author | License | Used for |
|------|--------|---------|----------|
| [BepInEx](https://github.com/BepInEx/BepInEx) | BepInEx Team | LGPL-2.1 | IL2CPP mod framework |
| [Harmony](https://github.com/pardeike/Harmony) | Andreas Pardeike | MIT | Runtime method patching |
| [Il2CppDumper](https://github.com/Perfare/Il2CppDumper) | Perfare | MIT | IL2CPP metadata dump |
| [Cpp2IL](https://github.com/SamboyCoding/Cpp2IL) | SamboyCoding | MIT | IL2CPP reversing |
| [Ghidra](https://github.com/NationalSecurityAgency/ghidra) | NSA | Apache-2.0 | Native code decompiler |
| [Goldberg Emulator](https://github.com/inflation/goldberg_emulator) | Mr_Goldberg | LGPL-3.0 | Steam API emulation |
| [LuxonServer](https://github.com/niansa/LuxonServer) | niansa | BSD-3-Clause | Clean-room Photon server |
| [Remote-Play-Together-for-nonSteam-games](https://github.com/Counter-clock/Remote-Play-Together-for-nonSteam-games) | Counter-clock | MIT | RPT for non-Steam games |
| [RetroArch](https://www.retroarch.com/) | Libretro Team | GPL-3.0 | Recommended free host game for RPT |
| [vcpkg](https://github.com/microsoft/vcpkg) | Microsoft | MIT | C++ package manager |

**Huge thanks to all open-source authors — this project wouldn't exist without them.**

---

## Reverse Engineering Playbook

> Condensed knowledge from 5 sessions of reverse-engineering a Unity IL2CPP + Steamworks + Photon game. Use as a playbook for future games.

### 1. General workflow checklist

1. **Identify the networking architecture** — Photon? Steam P2P? Custom?
2. **Dump IL2CPP metadata** — class/method/field names + RVAs
3. **Decompile with Ghidra headless** — read native logic by RVA
4. **Set up BepInEx + Harmony** — hook game code at runtime
5. **Bypass Steam auth** — Goldberg emulator if needed
6. **Analyze the network protocol** — server visualizer / packet capture
7. **Hunt for "back doors"** — hidden offline modes, unused code paths
8. **Mod-redirect** — switch the game to the hidden path

### 2. Dumping IL2CPP metadata

- **Il2CppDumper** (Perfare) — dumps from `GameAssembly.dll` + `global-metadata.dat`
- **Cpp2IL** — modern alternative, auto-finds metadata

Key outputs:
| File | Use |
|------|-----|
| `dump.cs` | All classes/methods/fields + **RVA per method** |
| `il2cpp.h` | C++ struct layouts (exact field offsets — read this instead of guessing from decompile) |
| `script.json` | method → RVA map (fast grep) |
| `stringliteral.json` | All strings + addresses (find error messages → trace to throwing code) |

### 3. Ghidra headless decompiling

```bash
analyzeHeadless.bat <projDir> <projName> -process "GameAssembly.dll" \
  -noanalysis -scriptPath <scriptDir> -postScript DecompileAddr.java 0x<addr>
```

- Address = `0x180000000 + RVA` (from `dump.cs`/`script.json`)
- `-noanalysis` skips full analysis (much faster, decompile still works)
- **Kill the Ghidra GUI first** (project lock!) and don't run 2 headless instances at once
- Keep your scripts in a dedicated folder (avoids compiling Ghidra's bundled scripts → duplicate class errors)

Reading decompile efficiently: match struct field accesses against `il2cpp.h`, strings against `stringliteral.json`, and calls against `dump.cs` names.

### 4. BepInEx 6 + Harmony — hard-won lessons

**Setup** (Unity 6000 IL2CPP): BepInEx 6 `be.785` from builds.bepinex.dev → `winhttp.dll` doorstop + `BepInEx/` + `dotnet/` into the game folder.

**⚠️ Crash lessons (learned painfully):**

1. **NEVER Harmony-detour Fusion/Photon IL2CPP methods with value-type params** → native crash. Real example: hooking `CloudServices.SendJoin` (`long, int, int, byte[]`) corrupted `NetworkProjectConfig.get_Global()`.
   - Rule: only hook methods whose params are reference types (object, string, class). For value-type params, use skip-prefixes instead of reading/writing them.
2. **NEVER use `object[] __args`** on methods with value-type params → crash. Use typed parameters matching the real signature.
3. **Il2Cpp Task ≠ managed Task** — `Il2CppSystem.Threading.Tasks.Task<bool>` is not `System.Threading.Tasks.Task<bool>`. Use `Il2CppSystem.Threading.Tasks.Task.FromResult(true)` + `return false` (skip original).
4. **Skip-prefix pattern**: `public static bool Prefix(...) { return false; }` skips the original method — use it to prevent NREs or redirect flow.
5. **Don't copy DLLs into `BepInEx/plugins` while the game runs** → "Device or resource busy". Kill the game first.
6. Use `AccessTools.Method` (not `GetMethod`) and pick the correct overload with explicit `Type[]`.
7. `FindObjectOfType<T>()` to grab a singleton MonoBehaviour that has no static Instance.

### 5. Identifying networking architecture

- **Inspect game files**: `PhotonAppSettings` assets, `Photon.Realtime.dll`, `Fusion.Realtime.dll` in interop → Photon. `steam_api64.dll` + Steamworks.NET → Steam.
- **Read the Steam page**: "Couch co-op", "Online co-op", "Remote Play Together" badges hint at modes.
- **Grep the dump**: `OperationCode` class lists every op code (e.g. Auth=230, JoinLobby=229, CreateGame=227).
- **Topology enums**: GameMode `Single=1, Shared=2, Server=3, Host=4, Client=5`.

**Critical blind spot:** Fusion 2 games always use `StartGameModeCloud` for multiplayer — there is no LAN direct mode. And Fusion 2 needs the server-side **FusionPlugin21** (native C++, Photon Cloud only). Self-hosted Photon (even licensed v5 SDK) cannot run Fusion 2 multiplayer.

### 6. Goldberg Steam Emulator

**When:** game uses Steamworks.NET and requires Steam init / Steam-ticket auth.

**Build:** clone [Goldberg](https://github.com/inflation/goldberg_emulator) + vcpkg (`protobuf x64-windows-static`) + MSVC.

**Deploy:**
```
Game_Data/Plugins/x86_64/
├── steam_api64.dll          ← Goldberg build (replaces original)
└── steam_settings/
    ├── steam_appid.txt      ← appid
    └── account_name.txt     ← fake username
```

**Required patches for Steamworks.NET 2025 (Flat API):** `SteamInternal_SteamAPI_Init`, `SteamInternal_GameServer_Init_V2`, `Steam_Timeline` class, ~33 stub flat exports. `steam_settings` must sit next to the DLL (`get_program_path()` = DLL dir). Note: Goldberg's `steam_remoteplay.h` is a stub — it does NOT emulate Steam Remote Play for the real Steam client.

### 7. Network protocol analysis

**Packet format (UDP → ENet → GpBinaryV18):**
```
UDP header (12B) → enet command header (12B: type/channel/flags/reserved/length/seq)
→ GpBinaryV18: magic 0xF3 + kind byte + payload
Kind: Init=0, InitResponse=1, Operation=2, OperationResponse=3, Event=4,
      Disconnect=5, InternalOperationRequest=6
```
**Painful lesson:** `f3` hex is **GP_MAGIC (0xF3)**, not op code 243!

**Server visualizer is the way out of guessing** — build LuxonServer with `LUXON_SERVER_ENABLE_VISUALIZER` and see every op code + decoded params.

**Fusion 2 protocol messages (bitstream):**
```
Join: Type(1B) GameMode(1B) PeerMode(1B) JoinRequests(4B) UniqueId(byte[])
      PlayerRef(4B) EncryptionKey(byte[]) EncryptionKeySecret(byte[]) PlayerCounterRef(4B)
      JoinMessageType: Request=1, Confirmation=2, Rejoin=3
NetworkConfigSync: Type(1B) NetworkConfig(string)
      SyncType: Request=1, Response=2, Override=3
```

### 8. Hunting for hidden offline modes

1. Grep `dump.cs` for `Offline`, `Local`, `CreateOffline`, `Tutorial`
2. Decompile UI button handlers → see what they actually call
3. Check enums for unused values

**The Dino Party win:** grep found `Networking.CreateOfflineGame()` — a method no UI button calls. The enum `NetworkGameType.OfflineParty = 3` existed but was never exposed. The Custom button always called `HostGame(isPublic=false)` (online). The fix was one hook redirecting the button to the hidden method.

**General pattern:**
```
UI button → public method X(bool flag)
                ↓ hook
     if (flag == private) → call hidden method Y() instead
```

### 9. Mod-redirect pattern (proven)

```csharp
// Hook the method the UI button calls
public static bool OfflineRedirect(ref bool isPublic) {
    if (isPublic) return true;   // keep original behavior for one branch
    var networking = UnityEngine.Object.FindObjectOfType<Networking>();
    networking.CreateOfflineGame();
    return false;                 // skip original for the other branch
}
```

- `return false` from a prefix skips the original method
- `return true` keeps it
- Log every interception for debugging
- Test with real controllers

### 10. Do / Don't — condensed from 5 sessions

**🚫 DON'T**
1. Don't Harmony-detour Fusion/Photon methods with value-type params → native crash
2. Don't use `object[] __args` on value-type params → crash
3. Don't patch Il2Cpp-Task-returning methods with managed Tasks → type mismatch
4. Don't guess op codes from raw hex without understanding the packet format
5. Don't run multiple server instances simultaneously → mixed-up logs
6. Don't run Ghidra GUI while using headless → project lock
7. Don't leave old servers running when rebuilding → LNK1168
8. Don't keep unnecessary trace hooks → crash risk + log noise
9. Don't trust old session notes 100% — re-verify each session
10. Don't ignore null-field checks in decompiled code

**✅ DO**
1. Build a server with visualizer — escape the guessing loop
2. Use mod trace hooks (typed signatures) + server visualizer = both directions covered
3. Decompile with Ghidra headless — fast and reusable
4. Fix one layer at a time — each fix moves the error to the next step
5. Back up every file before patching + verify the build after each change
6. Use `il2cpp.h` field structs — exact offsets without a decompiler
7. Watch `Player.log` + `LogOutput.log` continuously
8. Dig deep into game code before choosing an approach — hidden offline modes exist
9. Kill the game before writing DLLs (file locks)
10. Verify op codes match between game and server before debugging

### 11. Toolbox for the next game

| Tool | Purpose | Link |
|------|---------|------|
| BepInEx 6 | IL2CPP mod framework | https://github.com/BepInEx/BepInEx |
| HarmonyLib | Runtime patching | https://github.com/pardeike/Harmony |
| Il2CppDumper | IL2CPP metadata dump | https://github.com/Perfare/Il2CppDumper |
| Cpp2IL | IL2CPP reversing | https://github.com/SamboyCoding/Cpp2IL |
| Ghidra | Decompiler | https://github.com/NationalSecurityAgency/ghidra |
| Goldberg Emu | Steam emulation | https://github.com/inflation/goldberg_emulator |
| LuxonServer | Clean-room Photon server | https://github.com/niansa/LuxonServer |
| RemotePlayTogether | RPT for non-Steam games | https://github.com/Counter-clock/Remote-Play-Together-for-nonSteam-games |
| RetroArch | Recommended free host game for RPT | https://store.steampowered.com/app/1118310/ |
| vcpkg | C++ packages | https://github.com/microsoft/vcpkg |

### 12. Time estimates (from Dino Party experience)

| Task | Time |
|------|------|
| Dump + decompile one method | 5–15 min |
| BepInEx + basic mod setup | 1–2 h |
| Goldberg build for a new game | 2–4 h (first time) |
| Server with visualizer | 2–3 h |
| Analyze one network protocol | 4–8 h |
| Find a hidden offline mode | 1–4 h |
| Complete mod redirect | 2–4 h |

---

## License

This project is a collection of mods, patches, and documentation. The individual tools listed in [Credits](#credits) retain their own licenses. All custom code in this repository is provided as-is for educational purposes.

---

## References

- [Session logs](Session1.md) through [Session5.md](Session5.md) — detailed technical notes per session
- [REMOTE_PLAY_GUIDE.md](REMOTE_PLAY_GUIDE.md) — extra Remote Play Together details
