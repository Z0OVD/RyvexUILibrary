# Ryvex UI Library

A lightweight, single-file UI framework for Roblox script hubs — windows, tabs, and a full set of controls with a built-in **key system**, **tier badges**, **config profiles**, **live theming**, and **mobile support**. No dependencies. Drop one file on GitHub and `loadstring` it, Rayfield-style.

```lua
local Ryvex = loadstring(game:HttpGet("https://raw.githubusercontent.com/<user>/<repo>/main/Ryvex.lua"))()
```

---

## Features

- **Window** — draggable, minimise-to-pill, hide/show, toggle hotkey
- **Key system** — gate the hub behind Free / Premium keys, auto-badge, link buttons, saved-key auto-login
- **Controls** — Toggle, Button, Slider, Dropdown (single + multi), Keybind, Input, Color Picker, Label, Paragraph, Divider
- **Notifications** — stacked, auto-dismiss, progress countdown
- **Config profiles** — save/load all element values to the executor filesystem, plus a ready-made `CreateConfigManager()` panel
- **Live theming** — switch themes at runtime; 3 presets included (Dark / Midnight / Crimson)
- **Mobile** — auto floating toggle button on touch devices
- **Clean lifecycle** — one shared input dispatcher, all connections tracked, `Unload()` tears everything down (no ghost keybinds, no leaks on reload)

---

## Quick start

```lua
local Ryvex = loadstring(game:HttpGet("https://raw.githubusercontent.com/<user>/<repo>/main/Ryvex.lua"))()

local Window = Ryvex:CreateWindow({
    Title      = "Ryvex Hub",
    SubTitle   = "v1.0",
    ToggleKey  = Enum.KeyCode.RightShift,
})

local Combat = Window:CreateTab({ Name = "Combat" })
local aim    = Combat:CreateSection("Aimbot")

aim:CreateToggle({ Name = "Enabled", Default = false, Flag = "aim_on",
    Callback = function(state) print("aimbot:", state) end })

aim:CreateSlider({ Name = "FOV", Min = 0, Max = 500, Default = 120, Suffix = "px", Flag = "aim_fov",
    Callback = function(value) print("fov:", value) end })

Ryvex:Notify({ Title = "Ryvex", Content = "Loaded clean.", Duration = 4 })
```

---

## API reference

### `Ryvex:CreateWindow(opts) → Window | nil`

| Option | Type | Default | Notes |
|---|---|---|---|
| `Title` | string | `"Ryvex"` | Topbar title |
| `SubTitle` | string | – | Small line under the title |
| `Size` | UDim2 | `620×420` | Window size |
| `ToggleKey` | KeyCode | `RightShift` | Show/hide hotkey |
| `Tier` | string | – | `"Free"`/`"Premium"` badge; auto-pulled from the key system if omitted |
| `MobileButton` | bool | `false` | Force the floating toggle button (auto-on for touch devices) |
| `KeySystem` | table | – | If set, shows the key screen first and only builds the menu once a valid key resolves. Same options as `Ryvex:Key`. Returns `nil` if the user closes the prompt. |

```lua
local Window = Ryvex:CreateWindow({
    Title = "Ryvex",
    KeySystem = {
        Keys  = { Free = { "free-abc123" }, Premium = { "PREM-9981" } },
        Links = { { Name = "Free Key (Linkvertise)", Url = "https://..." } },
        SaveKey = true,
    },
})
if not Window then return end   -- user bailed on the key screen
```

**Window methods:** `CreateTab(opts)`, `SelectTab(tab)`, `Hide()`, `Show()`, `Toggle()`, `ToggleMinimize()`, `Destroy()`.

---

### `Ryvex:Key(opts) → tier | false`  *(yields)*

Blocks until a valid key is entered or the prompt is closed. Returns the tier string (`"Free"`/`"Premium"`) or `false`. Usually you just pass `KeySystem` to `CreateWindow` instead of calling this directly.

| Option | Type | Notes |
|---|---|---|
| `Title` | string | Header text |
| `Note` | string | Instruction line |
| `Keys` | table | `{ Free = {...}, Premium = {...} }` — exact match; premium wins ties |
| `Links` | table | `{ { Name = "...", Url = "..." }, ... }` — clicking copies the URL to clipboard |
| `SaveKey` | bool | Remember a valid key and auto-login next run |
| `SaveFileName` | string | Key file name (default `"key"`) |
| `OnSuccess` | function(tier) | Fired on a valid key |
| `OnFail` | function() | Fired on a bad key |

The resolved tier is stored on `Ryvex.KeyTier` and drives the window badge (**PREMIUM** = gold, **FREE** = blue).

---

### Tabs & Sections

```lua
local Tab     = Window:CreateTab({ Name = "Combat", Icon = "rbxassetid://..." })  -- Icon optional
local Section = Tab:CreateSection("Aimbot")   -- title optional
```

Elements can be created on a `Section`, or directly on a `Tab` (an implicit section is made for you).

---

### Elements

Every element is created on a section: `Section:CreateX(opts)`. Options common to most: `Name`, `Flag`, `Callback`, `Default`.

| Constructor | Key options | Handle methods |
|---|---|---|
| `CreateToggle` | `Default` (bool) | `:Set(v[, silent])`, `:Get()` |
| `CreateButton` | `SubText` | `:SetText(t)` |
| `CreateSlider` | `Min`, `Max`, `Default`, `Decimals`, `Suffix` | `:Set(v[, silent])`, `:Get()` |
| `CreateDropdown` | `Options` (array), `Multi` (bool), `Default` | `:Set(v)`, `:Get()`, `:Refresh(newOptions)` |
| `CreateKeybind` | `Default` (KeyCode), `Mode` (`"Hold"`/`"Toggle"`) | `:Set(key)`, `:Get()` |
| `CreateInput` | `Placeholder`, `Default` | `:Set(v)`, `:Get()` |
| `CreateColorPicker` | `Default` (Color3) | `:Set(c[, silent])`, `:Get()` |
| `CreateLabel` | `Text` | `:Set(text)` |
| `CreateParagraph` | `Title`, `Content` | – |
| `CreateDivider` | – | – |
| `CreateConfigManager` | – | Ready-made save/load/refresh + saved-config dropdown |

```lua
Section:CreateDropdown({
    Name = "ESP Modes", Multi = true,
    Options = { "Box", "Name", "Health", "Distance" },
    Default = { "Box", "Name" }, Flag = "esp_modes",
    Callback = function(selected) end,
})

Section:CreateKeybind({ Name = "Aim Key", Default = Enum.KeyCode.E, Mode = "Hold",
    Callback = function(down) end })
```

---

### Flags

Any element given a `Flag` registers into `Ryvex.Flags[flag]` (live value) and `Ryvex.Options[flag]` (the handle). Read state anywhere:

```lua
if Ryvex.Flags.aim_on then ... end
Ryvex.Options.aim_fov:Set(200)
```

---

### Notifications

```lua
Ryvex:Notify({ Title = "Heads up", Content = "Something happened.", Duration = 5 })
```

---

### Config profiles

```lua
Ryvex:SaveConfig("main")   -- writes all flagged values to <workspace>/Ryvex/main.json
Ryvex:LoadConfig("main")
Ryvex:ListConfigs()        -- -> { "main", "pvp", ... }
```

Or drop in the ready-made panel:

```lua
Tab:CreateSection("Config"):CreateConfigManager()
```

Requires an executor with filesystem functions (`writefile`/`readfile`/`listfiles`). No-ops safely without them.

---

### Theming

```lua
Ryvex:SetTheme("Midnight")          -- built-in: "Dark", "Midnight", "Crimson"
Ryvex:SetTheme({ Accent = Color3.fromRGB(0,255,150), Background = ... })  -- or a custom table
```

Switching re-skins the live UI. Custom theme tables should provide the same keys as the presets (`Accent`, `Background`, `Topbar`, `Sidebar`, `Element`, `ElementHover`, `ElementBorder`, `Section`, `Text`, `SubText`, `Placeholder`, `Shadow`).

---

### Lifecycle

```lua
Ryvex.OnUnload:Connect(function() end)
Ryvex.OnThemeChange:Connect(function(theme) end)
Ryvex:Unload()   -- destroys all windows, disconnects every input, clears state
```

Bind your cheat's cleanup to `OnUnload` so toggling the hub off stops your loops.

---

## Publishing to GitHub

1. Create a repo and add `Ryvex.lua` at the root (keep this `README.md` alongside it).
2. Grab the **raw** URL of `Ryvex.lua` (`raw.githubusercontent.com/...`).
3. Ship the one-liner:

```lua
local Ryvex = loadstring(game:HttpGet("https://raw.githubusercontent.com/<user>/<repo>/main/Ryvex.lua"))()
```

> Tip: for versioned releases, pin the loader to a tag/commit instead of `main` so an update never breaks live scripts.

---

## License

MIT — do what you want, no warranty.
