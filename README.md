# OneCCTV

A Ring-style CCTV surveillance system for FiveM, built on OneUI. Multi-terminal support, live scripted-camera feeds, PTZ controls, motion detection with vehicle classification, role-based access control, Discord logging, and a full Windows 10-style desktop UI.

---

## ⚠️ REQUIRES OneUI — INSTALL IT FIRST ⚠️

**OneCCTV will NOT work without OneUI.**

Download and install OneUI before installing OneCCTV:
- **OneUI** is the UI library that powers all text prompts, notifications, and dialogs.
- Without it, OneCCTV will fail to start with the error: `OneUI must be started before this resource.`

**Installation order in your `server.cfg`:**
```
ensure OneUI
ensure OneCCTV
```

If you skip OneUI, nothing will work. Don't open a bug report — install OneUI first.

---

## Features

### Live Camera Feeds
- Real in-world scripted cameras at configured positions
- Live view through the camera lens (not a static image)
- Pan / Tilt / Zoom (PTZ) controls with configurable limits
- Single-view layout (clean, focused on one camera at a time)

### Multi-Terminal Support
- Each terminal (computer terminal) has its own set of cameras
- A terminal can ONLY view its own cameras — not cameras from other terminals
- Add unlimited terminals around the map (banks, police stations, warehouses, etc.)
- Per-terminal desktop cosmetics (PC name, domain)
- Map blips at each terminal location

### Motion Detection
- Define invisible motion zones on any camera
- Detects when a player enters the zone
- **Vehicle classification** — knows if the motion is:
  - 🚶 A player on foot (info alert)
  - 🚗 A player in a regular vehicle (warning alert, includes vehicle name)
  - 🚓 A player in an emergency vehicle — police, ambulance, fire, sheriff, FBI (critical alert, red)
- Per-zone cooldown to prevent spam
- Real-time alerts in the CCTV app's "Recent Alerts" panel
- Discord logging for all motion events

### Role-Based Access Control
- Restrict terminals to specific roles
- Supports 4 frameworks:
  - **ACE** (FiveM native ACL) — default
  - **ESX** — job name checks
  - **QBCore** — job name checks
  - **Custom** — provide your own `GetPlayerRoles()` function
- Admin bypass via `onecctv.admin` ACE permission
- Access-denied overlay when a player lacks the required role

### Discord Logging
- All major events logged to Discord via webhook
- Configurable per-event (toggle each on/off)
- Events:
  - 🔓 Terminal opened
  - 🔒 Terminal closed
  - 📹 Camera switched (off by default — can be noisy)
  - 🚨 Motion detected (on foot / vehicle / emergency vehicle)
- Player Discord mentions when available
- Embed colors match severity (blue=info, orange=warning, red=critical)

### Windows 10 Desktop UI
- Full Win10-style desktop with taskbar, start menu, and desktop icons
- Draggable / minimizable / maximizable / closable app windows
- File Explorer, Notepad, and Settings apps included
- Personalization (wallpapers, accent colors, dark/light theme)
- The CCTV app launches in a fullscreen window
- Wallpaper auto-hides when CCTV is visible (so the live camera shows through), restores when minimized

---

## Installation

### Step 1: Install OneUI (REQUIRED)

Download OneUI and place it in `resources/OneUI/`. Add to `server.cfg`:
```
ensure OneUI
```

### Step 2: Install OneCCTV

1. Download the `OneCCTV` folder.
2. Place it in `resources/OneCCTV/`.
3. Add to your `server.cfg` (after OneUI):
```
ensure OneUI
ensure OneCCTV
```
4. Restart your server.

---

## Configuration

All configuration is in `config.lua`. Key sections:

### Terminals & Cameras
```lua
Config.Terminals = {
    {
        id      = 'mrpd_front_desk',
        label   = 'Mission Row PD · Front Desk',
        pos     = vec3(434.18, -983.69, 30.71),
        heading = 90.0,
        desktop = { pcName = 'MRPD-CCTV-01', domain = 'mrpd.local' },
        roles   = { 'police' },  -- optional: restrict access
        cameras = {
            {
                id        = 'cam_001',
                label     = 'Front Entrance',
                pos       = vec3(428.0, -980.0, 33.5),
                rotation  = vec3(-25.0, 0.0, 90.0),
                fov       = 60.0,
                zone      = 'Perimeter',
                status    = 'online',
                motion    = {              -- optional: motion zone
                    box      = vec3(8, 8, 4),  -- width, length, height
                    cooldown = 30,             -- seconds between alerts
                },
            },
        },
    },
}
```

### Role-Based Access
```lua
Config.Roles = {
    framework    = 'ace',    -- 'none' | 'ace' | 'esx' | 'qb' | 'custom'
    adminBypass  = true,
}
```

### Discord Logging
```lua
Config.Discord = {
    webhook  = 'https://discord.com/api/webhooks/YOUR_WEBHOOK_URL',
    botName  = 'OneCCTV Logs',
    events   = {
        terminalOpen  = true,
        terminalClose = true,
        cameraSwitch  = false,
        motionDetect  = true,
    },
}
```

---

## Commands

| Command | Description |
|---------|-------------|
| `/cctv [terminalId]` | Open a terminal's CCTV viewer (default: first terminal) |
| `/cctvlist` | List all configured terminals and cameras (notification) |
| `/cctvview [terminalId] [cameraId]` | Debug: force camera view without NUI |
| `/cctvview stop` | Stop debug camera view |

---

## Bundled Utility: os_coords

OneCCTV includes a companion utility script — **os_coords** — for positioning cameras and terminals.

### os_coords Features

- **`/copyvec3`** — Copy your current position as `vec3(x, y, z)` to the clipboard
- **`/copyvec3h`** — Copy position + heading
- **`/copycam [label]`** — Copy a full camera config block (pos + rotation from where you're looking + fov)
- **`/copyterminal [id] [label]`** — Copy a full terminal config block
- **`/copymotion [w] [l] [h] [cooldown]`** — Copy a motion zone config block
- **`/showcoords`** — Toggle an on-screen live coordinate display
- **`/fly`** — Toggle noclip fly mode (WASD + Q/E + Shift + mouse)

### os_coords Installation

os_coords is a separate resource. Install it alongside OneCCTV:
```
ensure OneUI
ensure OneCCTV
ensure os_coords
```

### os_coords Workflow

The fastest way to add a new camera:

1. Type `/fly` to enter noclip fly mode
2. Fly to where you want the camera lens to be
3. Look in the direction the camera should face
4. Type `/copycam Front Door` — a full camera block is copied to your clipboard
5. Type `/fly` again to exit
6. Paste the block into your terminal's `cameras = {}` table in `config.lua`

For motion zones, stand where the camera is and type `/copymotion 10 10 4 30` — a motion zone block is copied, ready to paste into the camera definition.

---

## File Structure

```
OneCCTV/
├── fxmanifest.lua
├── config.lua                 # All configuration (terminals, cameras, roles, Discord, motion)
├── client/
│   ├── camera.lua            # Scripted camera management (create, PTZ, destroy)
│   └── main.lua              # Proximity loop, NUI callbacks, viewer open/close, alerts
├── server/
│   ├── logging.lua           # Discord webhook + role-based access control
│   ├── motion.lua            # Motion detection loop + vehicle classification
│   └── main.lua              # Server bootstrap
└── web/
    └── index.html            # Full Win10 desktop UI (inline CSS + JS)
```

---

## Requirements

- **OneUI** (REQUIRED — see warning above)
- FiveM server
- Lua 5.4

---

## Author

**OneSystems**

Part of the OneSystems script ecosystem. Built on [OneUI](#).
