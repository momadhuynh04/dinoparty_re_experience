# Guide to Playing Dino Party (2 Players) via Steam Remote Play Together

## Overview

Dino Party offers **3 ways to play together**:
1. **Couch co-op** (local; 2–4 players on one machine) ← **SUPPORTED (Mod 0.11.0-offline)**
2. **Private online lobby** (requires Photon Cloud; difficult to mod due to Fusion 2)
3. **Steam Remote Play Together** — the host streams the game to you via Steam

**Steam Remote Play Together** allows you to play with friends who are far away (using two separate machines). The host streams their screen and receives remote inputs.

> ⚠️ **Note**: Steam BLOCKS Remote Play Together for non-Steam games. You need to use the **RemotePlayTogether.exe** tool to bypass this restriction (instructions below).

---

## Part A: RemotePlayTogether.exe — Bypass tool for non-Steam games

### What is this tool?
[Remote-Play-Together-for-nonSteam-games](https://github.com/Counter-clock/Remote-Play-Together-for-nonSteam-games) — a small program that enables Steam Remote Play Together for **any PC game** (including non-Steam games like Dino Party). ### Operating Principle
1. Steam only enables Remote Play Together for **actual Steam games** (those in your library).
2. Emulation tool: replaces the executable of a free Steam game (RetroArch) with a "stub" file.
3. When you launch RetroArch from Steam → the stub displays a menu allowing you to select a non-Steam game.
4. You select Dino Party → the stub launches the actual game → Steam still thinks it is streaming RetroArch.

### Detailed Setup

#### Step 1: Download RemotePlayTogether.exe
```
https://github.com/Counter-clock/Remote-Play-Together-for-nonSteam-games/releases
→ Download the RemotePlayTogether.exe file (latest version)
```

#### Step 2: Download RetroArch (free Steam game)
```
Steam → Store → search for "RetroArch" → Install (free)
Or direct link: https://store.steampowered.com/app/1118310/RetroArch/
```
RetroArch is a **free** Steam game that **supports Remote Play Together** and is **lightweight**—making it perfect to act as the emulation "host."

#### Step 3: Replace retroarch.exe with RemotePlayTogether.exe
```
1. Steam → Library → RetroArch → Right-click → Properties → Installed Files → Browse
2. Navigate to the folder containing retroarch.exe (e.g., D:\SteamLibrary\steamapps\common\RetroArch\)
3. Backup: rename retroarch.exe to retroarch.exe.bak
4. Copy RemotePlayTogether.exe into this folder
5. Rename RemotePlayTogether.exe to retroarch.exe
```

#### Step 4: Create remoteplay.txt
Create a file named `remoteplay.txt` in the SAME folder (D:\SteamLibrary\steamapps\common\RetroArch\):
```
D:\ProjectStorage\Dino Party\Dino Party.exe | Dino Party
```

**Format**:
```
# One game per line: <path to game exe> |
``` <display name>
D:\Games\Game1\game1.exe
C:\SteamLibrary\steamapps\common\Game2\game2.exe | Game 2
```
- The `| `Name` is optional — sets the display name in the menu
- Path must use the correct Windows format (backslashes)

#### Step 5: Launch RetroArch from Steam
1. Steam → Library → RetroArch → **Play**
2. The tool automatically creates `remoteplay.txt` (if missing) and displays the menu
3. The menu shows the game list from `remoteplay.txt`
4. Select **Dino Party** (press the corresponding number or click)
5. Dino Party launches (Goldberg Steam emulator loads automatically)

#### Step 6: Invite friends via Remote Play Together
1. Inside Dino Party: Press **Shift+Tab** to open the Steam Overlay
2. Go to the **Friends** tab → Right-click a friend
3. Select **"Remote Play Together"** → **"Send Invite"**
4. Friend receives an invite popup → Click **Accept**
5. Steam streams the host's screen → friend sends remote input

#### Step 7: 2-player gameplay
1. Host: Go to Main Menu → Party → Host Party → **Custom** (mod redirects to offline mode)
2. Host uses controller 1 (or keyboard)
3. Friend uses controller 2 (via Steam Input)
4. Both play couch co-op

---

## Part B: Troubleshooting

### 1. "Remote Play Together" is disabled/grayed out
- **Cause**: RetroArch was not launched correctly from Steam
- **Fix**: Ensure you launch RetroArch from the Steam Library (do not run the .exe file directly)

### 2. Steam reports "This game doesn't support Remote Play Together"
- **Cause**: retroarch.exe was not replaced correctly
- **Fix**: Verify that the retroarch.exe file in the RetroArch folder is actually RemotePlayTogether.exe (check size: ~447KB)

### 3. Tool menu appears, but selecting Dino Party doesn't launch it
- **Cause**: Incorrect path in remoteplay.txt
- **Fix**: Check that the path `D:\ProjectStorage\Dino Party\Dino Party.exe` exists

### 4. Friend - Friend does not receive the invite
- **Cause**: Not yet Steam Friends
- **Fix**: Add each other as Steam Friends first, then send the invite

### 5. Streaming works, but inputs aren't reaching the game
- **Cause**: Steam Input is not enabled
- **Fix**: Steam → Settings → Controller → Enable Steam Input for the friend's controller

### 6. High latency (game lag)
- **Fix**:
- Use LAN (same network) if possible
- Steam → Settings → Remote Play → Streaming quality = Balanced/Low
- Host uses GPU encoding (NVENC/QuickSync)

### 7. Game requires administrator privileges
- **Fix**: Run Steam as administrator (Right-click Steam → Run as administrator)

---

## Part C: Comparison of play methods

| Method | PC | Controller | Internet Required | Photon | Tested |
|--------|-----|-----------|-------------------|--------|--------|
| **Couch co-op (Mod 0.11.0)** | 1 PC | 2-4 USB | ❌ | ❌ | ✅ SUCCESSFUL