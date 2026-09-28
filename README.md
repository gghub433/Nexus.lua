-- [[ NEXUS V12 PRIME ]] --
-- Версия: 12.6.0
-- Тип: Агрессивный / Кастомизируемый
-- Особенности: Aimbot, Silent Aim, ESP, Chams, Tracers, SpeedHack, InfJump, AntiAFK, TeamCheck

local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local CoreGui          = game:GetService("CoreGui")
local HttpService      = game:GetService("HttpService")
local Stats            = game:GetService("Stats")

local LocalPlayer = Players.LocalPlayer
local Camera      = workspace.CurrentCamera

-- [ ЦВЕТА ]
local ACCENT    = Color3.fromRGB(191, 64, 255)
local BG_MAIN   = Color3.fromRGB(12, 12, 18)
local BG_ITEM   = Color3.fromRGB(22, 22, 30)
local BG_HOVER  = Color3.fromRGB(32, 32, 44)
local TEXT_ON   = Color3.new(1, 1, 1)
local TEXT_OFF  = Color3.fromRGB(145, 145, 165)

-- [ КОНФИГУРАЦИЯ ]
local Config = {
    Combat = {
        Aimbot      = false,
        SilentAim   = false,
        AimPart     = "Head",
        Smoothness  = 3,
        FOV         = 150,
        Prediction  = 16,
        WallCheck   = false,
        TeamCheck   = true,
        TargetMode  = "Closest",
    },
    Visuals = {
        ESP         = false,
        ESPNames    = false,
        ESPTracers  = false,
        Chams       = false,
        Crosshair   = false,
        Hitbox      = false,
        HitboxSize  = 25,
        CameraFOV   = 90,
    },
    Movement = {
        Bhop        = false,
        BhopPower   = 25,
        SpeedHack   = false,
        Speed       = 30,
        InfJump     = false,
        NoFall      = false,
        AntiAFK     = false,
    },
}

local Minimized = false
local CFG_FILE  = "NexusV12_Config.json"

local function SaveSettings()
    if writefile then writefile(CFG_FILE, HttpService:JSONEncode(Config)) end
end
local function LoadSettings()
    if isfile and isfile(CFG_FILE) then
        local ok, data = pcall(function() return HttpService:JSONDecode(readfile(CFG_FILE)) end)
        if ok then for k, v in pairs(data) do Config[k] = v end end
    end
end

-- [ GUI ROOT ]
if CoreGui:FindFirstChild("NEXUS_V12") then CoreGui:FindFirstChild("NEXUS_V12"):Destroy() end
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "NEXUS_V12"
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true

-- [ УВЕДОМЛЕНИЯ ]
local function Notify(title, body, dur)
    task.spawn(function()
        local f = Instance.new("Frame", ScreenGui)
        f.Size = UDim2.new(0, 295, 0, 72)
        f.Position = UDim2.new(1, 20, 1, -90)
        f.BackgroundColor3 = BG_MAIN; f.BorderSizePixel = 0
        Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
        local bar = Instance.new("Frame", f)
        bar.Size = UDim2.new(0, 3, 1, -14); bar.Position = UDim2.new(0, 0, 0, 7)
        bar.BackgroundColor3 = ACCENT; bar.BorderSizePixel = 0
        Instance.new("UICorner", bar).CornerRadius = UDim.new(0, 3)
        local t = Instance.new("TextLabel", f)
        t.Size = UDim2.new(1, -16, 0, 28); t.Position = UDim2.new(0, 13, 0, 4)
        t.BackgroundTransparency = 1; t.Text = title
        t.TextColor3 = ACCENT; t.Font = Enum.Font.GothamBold; t.TextSize = 13
        t.TextXAlignment = Enum.TextXAlignment.Left
        local c = Instance.new("TextLabel", f)
        c.Size = UDim2.new(1, -16, 0, 28); c.Position = UDim2.new(0, 13, 0, 34)
        c.BackgroundTransparency = 1; c.Text = body
        c.TextColor3 = TEXT_OFF; c.Font = Enum.Font.Gotham; c.TextSize = 12
        c.TextXAlignment = Enum.TextXAlignment.Left
        f:TweenPosition(UDim2.new(1, -315, 1, -90), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.4, true)
        task.wait(dur or 3)
        f:TweenPosition(UDim2.new(1, 20, 1, -90), Enum.EasingDirection.In, Enum.EasingStyle.Quart, 0.38, true)
        task.wait(0.45); f:Destroy()
    end)
end

-- [ HUD ]
local HUD = Instance.new("Frame", ScreenGui)
HUD.Size = UDim2.new(0, 158, 0, 76)
HUD.Position = UDim2.new(0, 18, 0, 18)
HUD.BackgroundColor3 = BG_MAIN; HUD.BackgroundTransparency = 0.12; HUD.BorderSizePixel = 0
Instance.new("UICorner", HUD).CornerRadius = UDim.new(0, 8)
local hStroke = Instance.new("UIStroke", HUD)
hStroke.Thickness = 1.5; hStroke.Color = ACCENT; hStroke.Transparency = 0.35
local hLabel = Instance.new("TextLabel", HUD)
hLabel.Size = UDim2.new(1, -14, 1, -6); hLabel.Position = UDim2.new(0, 9, 0, 3)
hLabel.BackgroundTransparency = 1; hLabel.TextColor3 = TEXT_ON
hLabel.Font = Enum.Font.Code; hLabel.TextSize = 11; hLabel.TextXAlignment = Enum.TextXAlignment.Left
hLabel.RichText = true; hLabel.Text = "FPS: --  PING: --\nSPD: --\nTARGET: NONE"

-- [ ГЛАВНОЕ ОКНО ]
local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 545, 0, 445)
Main.Position = UDim2.new(0.5, -272, 0.5, -222)
Main.BackgroundColor3 = BG_MAIN; Main.BorderSizePixel = 0; Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
local MainStroke = Instance.new("UIStroke", Main)
MainStroke.Thickness = 1.5; MainStroke.Color = ACCENT; MainStroke.Transparency = 0.15

-- ШАПКА
local Header = Instance.new("Frame", Main)
Header.Size = UDim2.new(1, 0, 0, 52)
Header.BackgroundColor3 = BG_ITEM; Header.BorderSizePixel = 0
local hLine = Instance.new("Frame", Header)
hLine.Size = UDim2.new(1, 0, 0, 2); hLine.Position = UDim2.new(0, 0, 1, -2)
hLine.BackgroundColor3 = ACCENT; hLine.BorderSizePixel = 0

local TitleLbl = Instance.new("TextLabel", Header)
TitleLbl.Size = UDim2.new(0, 200, 1, 0); TitleLbl.Position = UDim2.new(0, 16, 0, 0)
TitleLbl.BackgroundTransparency = 1; TitleLbl.Text = "NEXUS  V12"
TitleLbl.TextColor3 = TEXT_ON; TitleLbl.Font = Enum.Font.GothamBold; TitleLbl.TextSize = 20
TitleLbl.TextXAlignment = Enum.TextXAlignment.Left

local BadgeLbl = Instance.new("TextLabel", Header)
BadgeLbl.Size = UDim2.new(0, 58, 0, 18); BadgeLbl.Position = UDim2.new(0, 16, 1, -22)
BadgeLbl.BackgroundColor3 = ACCENT; BadgeLbl.BorderSizePixel = 0
Instance.new("UICorner", BadgeLbl).CornerRadius = UDim.new(0, 4)
BadgeLbl.Text = "  PRIME  "; BadgeLbl.TextColor3 = TEXT_ON
BadgeLbl.Font = Enum.Font.GothamBold; BadgeLbl.TextSize = 9

local MinBtn = Instance.new("TextButton", Header)
MinBtn.Size = UDim2.new(0, 34, 0, 34); MinBtn.Position = UDim2.new(1, -80, 0.5, -17)
MinBtn.Text = "−"; MinBtn.TextColor3 = TEXT_OFF
MinBtn.Font = Enum.Font.GothamBold; MinBtn.TextSize = 22
MinBtn.BackgroundColor3 = BG_HOVER; MinBtn.BorderSizePixel = 0
Instance.new("UICorner", MinBtn).CornerRadius = UDim.new(0, 6)

local CloseBtn = Instance.new("TextButton", Header)
CloseBtn.Size = UDim2.new(0, 34, 0, 34); CloseBtn.Position = UDim2.new(1, -40, 0.5, -17)
CloseBtn.Text = "×"; CloseBtn.TextColor3 = TEXT_ON
CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 22
CloseBtn.BackgroundColor3 = Color3.fromRGB(175, 38, 38); CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

CloseBtn.MouseButton1Click:Connect(function() Main.Visible = false end)
MinBtn.MouseButton1Click:Connect(function()
    Minimized = not Minimized
    local ts = Minimized and UDim2.new(0, 545, 0, 52) or UDim2.new(0, 545, 0, 445)
    TweenService:Create(Main, TweenInfo.new(0.38, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = ts}):Play()
    MinBtn.Text = Minimized and "□" or "−"
end)

-- DRAG
local isDrag, dragInp, dragStart, dragPos = false, nil, nil, nil
Header.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        isDrag = true; dragStart = i.Position; dragPos = Main.Position
    end
end)
UserInputService.InputChanged:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseMovement then dragInp = i end
end)
RunService.RenderStepped:Connect(function()
    if isDrag and dragInp then
        local d = dragInp.Position - dragStart
        Main.Position = UDim2.new(dragPos.X.Scale, dragPos.X.Offset + d.X, dragPos.Y.Scale, dragPos.Y.Offset + d.Y)
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then isDrag = false end
end)

-- SIDEBAR
local Sidebar = Instance.new("Frame", Main)
Sidebar.Size = UDim2.new(0, 138, 1, -62); Sidebar.Position = UDim2.new(0, 10, 0, 58)
Sidebar.BackgroundTransparency = 1
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 5)

local SideDiv = Instance.new("Frame", Main)
SideDiv.Size = UDim2.new(0, 1, 1, -62); SideDiv.Position = UDim2.new(0, 155, 0, 58)
SideDiv.BackgroundColor3 = BG_HOVER; SideDiv.BorderSizePixel = 0

-- CONTENT
local ContentArea = Instance.new("Frame", Main)
ContentArea.Size = UDim2.new(1, -170, 1, -68); ContentArea.Position = UDim2.new(0, 162, 0, 60)
ContentArea.BackgroundTransparency = 1

-- ВКЛАДКИ
local Tabs = {}
local TabBtns = {}
local ActiveTab = nil

local function MakeTab(name)
    local sf = Instance.new("ScrollingFrame", ContentArea)
    sf.Size = UDim2.new(1, 0, 1, 0); sf.BackgroundTransparency = 1; sf.Visible = false
    sf.ScrollBarThickness = 3; sf.ScrollBarImageColor3 = ACCENT
    sf.CanvasSize = UDim2.new(0, 0, 0, 0); sf.AutomaticCanvasSize = Enum.AutomaticSize.Y
    local ul = Instance.new("UIListLayout", sf); ul.Padding = UDim.new(0, 7)
    local pad = Instance.new("UIPadding", sf)
    pad.PaddingTop = UDim.new(0, 3); pad.PaddingRight = UDim.new(0, 4)
    Tabs[name] = sf
end

MakeTab("COMBAT"); MakeTab("VISUALS"); MakeTab("MOVEMENT"); MakeTab("SETTINGS")

local function SetTab(name)
    if ActiveTab then
        Tabs[ActiveTab].Visible = false
        local b = TabBtns[ActiveTab]
        if b then b.BackgroundColor3 = BG_ITEM; b.TextColor3 = TEXT_OFF end
    end
    ActiveTab = name; Tabs[name].Visible = true
    local b = TabBtns[name]
    if b then b.BackgroundColor3 = ACCENT; b.TextColor3 = TEXT_ON end
end

local function MakeTabBtn(name)
    local b = Instance.new("TextButton", Sidebar)
    b.Size = UDim2.new(1, 0, 0, 38); b.BackgroundColor3 = BG_ITEM; b.BorderSizePixel = 0
    b.Text = name; b.TextColor3 = TEXT_OFF; b.Font = Enum.Font.GothamBold; b.TextSize = 12
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 7)
    TabBtns[name] = b
    b.MouseButton1Click:Connect(function() SetTab(name) end)
end

MakeTabBtn("COMBAT"); MakeTabBtn("VISUALS"); MakeTabBtn("MOVEMENT"); MakeTabBtn("SETTINGS")

-- [ КОНСТРУКТОРЫ UI ]
local function CreateLabel(parent, text)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1, -4, 0, 20); f.BackgroundTransparency = 1
    local lbl = Instance.new("TextLabel", f)
    lbl.Size = UDim2.new(1, 0, 1, 0); lbl.BackgroundTransparency = 1
    lbl.Text = "▸ " .. text; lbl.TextColor3 = ACCENT
    lbl.Font = Enum.Font.GothamBold; lbl.TextSize = 10; lbl.TextXAlignment = Enum.TextXAlignment.Left
end

local function CreateToggle(parent, text, tbl, key)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -4, 0, 40); b.BackgroundColor3 = BG_ITEM; b.BorderSizePixel = 0
    b.TextXAlignment = Enum.TextXAlignment.Left; b.Font = Enum.Font.GothamMedium; b.TextSize = 13
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 7)
    local pad = Instance.new("UIPadding", b); pad.PaddingLeft = UDim.new(0, 12)

    local ind = Instance.new("TextLabel", b)
    ind.Size = UDim2.new(0, 42, 0, 20); ind.Position = UDim2.new(1, -50, 0.5, -10)
    ind.Font = Enum.Font.GothamBold; ind.TextSize = 10; ind.BorderSizePixel = 0
    ind.TextXAlignment = Enum.TextXAlignment.Center
    Instance.new("UICorner", ind).CornerRadius = UDim.new(0, 5)

    local function Refresh()
        local on = tbl[key]
        b.Text = text; b.TextColor3 = on and TEXT_ON or TEXT_OFF
        b.BackgroundColor3 = on and Color3.fromRGB(40, 24, 58) or BG_ITEM
        ind.Text = on and "ON" or "OFF"
        ind.BackgroundColor3 = on and ACCENT or BG_HOVER
        ind.TextColor3 = on and TEXT_ON or TEXT_OFF
    end
    Refresh()
    b.MouseButton1Click:Connect(function() tbl[key] = not tbl[key]; Refresh() end)
    return b
end

local function CreateSlider(parent, text, min, max, tbl, key)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1, -4, 0, 58); f.BackgroundTransparency = 1

    local lbl = Instance.new("TextLabel", f)
    lbl.Size = UDim2.new(1, -52, 0, 22); lbl.BackgroundTransparency = 1
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Font = Enum.Font.GothamMedium; lbl.TextSize = 12; lbl.TextColor3 = TEXT_OFF

    local valLbl = Instance.new("TextLabel", f)
    valLbl.Size = UDim2.new(0, 48, 0, 22); valLbl.Position = UDim2.new(1, -48, 0, 0)
    valLbl.BackgroundTransparency = 1; valLbl.TextXAlignment = Enum.TextXAlignment.Right
    valLbl.Font = Enum.Font.GothamBold; valLbl.TextSize = 13; valLbl.TextColor3 = ACCENT

    local track = Instance.new("Frame", f)
    track.Size = UDim2.new(1, 0, 0, 7); track.Position = UDim2.new(0, 0, 0, 38)
    track.BackgroundColor3 = BG_HOVER; track.BorderSizePixel = 0
    Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

    local fill = Instance.new("Frame", track)
    fill.BackgroundColor3 = ACCENT; fill.BorderSizePixel = 0; fill.Size = UDim2.new(0, 0, 1, 0)
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

    local dot = Instance.new("Frame", track)
    dot.Size = UDim2.new(0, 13, 0, 13); dot.AnchorPoint = Vector2.new(0.5, 0.5)
    dot.BackgroundColor3 = TEXT_ON; dot.BorderSizePixel = 0
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local function Refresh(val)
        val = math.clamp(math.floor(val), min, max); tbl[key] = val
        local pct = (val - min) / (max - min)
        fill.Size = UDim2.new(pct, 0, 1, 0)
        dot.Position = UDim2.new(pct, 0, 0.5, 0)
        lbl.Text = text; valLbl.Text = tostring(val)
    end
    Refresh(tbl[key])

    local sliding = false
    track.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then sliding = true end end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then sliding = false end end)
    RunService.RenderStepped:Connect(function()
        if sliding then
            local pct = math.clamp((UserInputService:GetMouseLocation().X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
            Refresh(min + (max - min) * pct)
        end
    end)
    return f
end

local function CreateCycle(parent, text, tbl, key, opts)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -4, 0, 40); b.BackgroundColor3 = BG_ITEM; b.BorderSizePixel = 0
    b.TextXAlignment = Enum.TextXAlignment.Left; b.Font = Enum.Font.GothamMedium; b.TextSize = 13
    b.TextColor3 = TEXT_OFF
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 7)
    Instance.new("UIPadding", b).PaddingLeft = UDim.new(0, 12)

    local chip = Instance.new("TextLabel", b)
    chip.Size = UDim2.new(0, 110, 0, 22); chip.Position = UDim2.new(1, -118, 0.5, -11)
    chip.BackgroundColor3 = BG_HOVER; chip.BorderSizePixel = 0
    chip.Font = Enum.Font.GothamBold; chip.TextSize = 11; chip.TextColor3 = ACCENT
    chip.TextXAlignment = Enum.TextXAlignment.Center
    Instance.new("UICorner", chip).CornerRadius = UDim.new(0, 5)

    local function Refresh()
        b.Text = text; chip.Text = "◀  " .. tbl[key] .. "  ▶"
    end
    Refresh()
    b.MouseButton1Click:Connect(function()
        local i = table.find(opts, tbl[key]) or 1
        tbl[key] = opts[i >= #opts and 1 or i + 1]; Refresh()
    end)
    return b
end

local function CreateButton(parent, text, color, cb)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, -4, 0, 40); b.BackgroundColor3 = color or ACCENT; b.BorderSizePixel = 0
    b.Text = text; b.TextColor3 = TEXT_ON; b.Font = Enum.Font.GothamBold; b.TextSize = 13
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 7)
    b.MouseButton1Click:Connect(cb)
    return b
end

-- [ COMBAT TAB ]
CreateLabel(Tabs["COMBAT"], "AIMBOT")
CreateToggle(Tabs["COMBAT"], "Aimbot", Config.Combat, "Aimbot")
CreateToggle(Tabs["COMBAT"], "Silent Aim", Config.Combat, "SilentAim")
CreateToggle(Tabs["COMBAT"], "Wall Check (видит сквозь)", Config.Combat, "WallCheck")
CreateToggle(Tabs["COMBAT"], "Team Check", Config.Combat, "TeamCheck")
CreateCycle(Tabs["COMBAT"], "Aim Part", Config.Combat, "AimPart", {"Head", "Torso", "HumanoidRootPart"})
CreateCycle(Tabs["COMBAT"], "Target Mode", Config.Combat, "TargetMode", {"Closest", "Lowest HP", "Random"})
CreateLabel(Tabs["COMBAT"], "ПАРАМЕТРЫ")
CreateSlider(Tabs["COMBAT"], "FOV Radius", 50, 600, Config.Combat, "FOV")
CreateSlider(Tabs["COMBAT"], "Smoothness", 1, 20, Config.Combat, "Smoothness")
CreateSlider(Tabs["COMBAT"], "Prediction", 1, 50, Config.Combat, "Prediction")

-- [ VISUALS TAB ]
CreateLabel(Tabs["VISUALS"], "ESP")
CreateToggle(Tabs["VISUALS"], "ESP Highlight", Config.Visuals, "ESP")
CreateToggle(Tabs["VISUALS"], "ESP Names", Config.Visuals, "ESPNames")
CreateToggle(Tabs["VISUALS"], "ESP Tracers", Config.Visuals, "ESPTracers")
CreateToggle(Tabs["VISUALS"], "Chams (красный силуэт)", Config.Visuals, "Chams")
CreateLabel(Tabs["VISUALS"], "ПРОЧЕЕ")
CreateToggle(Tabs["VISUALS"], "Crosshair", Config.Visuals, "Crosshair")
CreateToggle(Tabs["VISUALS"], "Hitbox Expander", Config.Visuals, "Hitbox")
CreateSlider(Tabs["VISUALS"], "Hitbox Size", 2, 100, Config.Visuals, "HitboxSize")
CreateSlider(Tabs["VISUALS"], "Camera FOV", 60, 130, Config.Visuals, "CameraFOV")

-- [ MOVEMENT TAB ]
CreateLabel(Tabs["MOVEMENT"], "СКОРОСТЬ")
CreateToggle(Tabs["MOVEMENT"], "BunnyHop", Config.Movement, "Bhop")
CreateSlider(Tabs["MOVEMENT"], "Bhop Power", 1, 100, Config.Movement, "BhopPower")
CreateToggle(Tabs["MOVEMENT"], "SpeedHack", Config.Movement, "SpeedHack")
CreateSlider(Tabs["MOVEMENT"], "Speed Value", 16, 250, Config.Movement, "Speed")
CreateLabel(Tabs["MOVEMENT"], "ПРЫЖОК")
CreateToggle(Tabs["MOVEMENT"], "Infinite Jump", Config.Movement, "InfJump")
CreateToggle(Tabs["MOVEMENT"], "No Fall Damage", Config.Movement, "NoFall")
CreateLabel(Tabs["MOVEMENT"], "ПРОЧЕЕ")
CreateToggle(Tabs["MOVEMENT"], "Anti-AFK", Config.Movement, "AntiAFK")

-- [ SETTINGS TAB ]
CreateLabel(Tabs["SETTINGS"], "КОНФИГ")
CreateButton(Tabs["SETTINGS"], "SAVE CONFIG", Color3.fromRGB(26, 74, 26), function()
    SaveSettings(); Notify("NEXUS", "Конфиг сохранён", 2)
end)
CreateButton(Tabs["SETTINGS"], "LOAD CONFIG", Color3.fromRGB(26, 38, 80), function()
    LoadSettings(); Notify("NEXUS", "Конфиг загружен", 2)
end)
CreateButton(Tabs["SETTINGS"], "RESET DEFAULTS", Color3.fromRGB(72, 22, 22), function()
    Notify("NEXUS", "Перезапусти скрипт для сброса", 3)
end)
CreateLabel(Tabs["SETTINGS"], "ИНТЕРФЕЙС")
CreateButton(Tabs["SETTINGS"], "TOGGLE HUD", BG_HOVER, function()
    HUD.Visible = not HUD.Visible
end)
CreateButton(Tabs["SETTINGS"], "TOGGLE WINDOW  (RShift)", BG_HOVER, function()
    Main.Visible = not Main.Visible
end)

SetTab("COMBAT")

-- [ DRAWING OBJECTS ]
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5; FOVCircle.NumSides = 64
FOVCircle.Color = ACCENT; FOVCircle.Filled = false; FOVCircle.Visible = false

local CrossH = Drawing.new("Line"); CrossH.Color = ACCENT; CrossH.Thickness = 1.5; CrossH.Visible = false
local CrossV = Drawing.new("Line"); CrossV.Color = ACCENT; CrossV.Thickness = 1.5; CrossV.Visible = false

-- ESP Name labels pool
local NameLabels = {}
local function GetNameLabel(p)
    if not NameLabels[p] then
        local lbl = Drawing.new("Text")
        lbl.Size = 14; lbl.Center = true; lbl.Outline = true
        lbl.Color = ACCENT; lbl.OutlineColor = Color3.new(0, 0, 0)
        lbl.Font = 2; lbl.Visible = false
        NameLabels[p] = lbl
    end
    return NameLabels[p]
end

-- ESP Tracer lines pool
local TracerLines = {}
local function GetTracer(p)
    if not TracerLines[p] then
        local ln = Drawing.new("Line")
        ln.Color = ACCENT; ln.Thickness = 1; ln.Visible = false
        TracerLines[p] = ln
    end
    return TracerLines[p]
end

Players.PlayerRemoving:Connect(function(p)
    if NameLabels[p] then NameLabels[p]:Remove(); NameLabels[p] = nil end
    if TracerLines[p] then TracerLines[p]:Remove(); TracerLines[p] = nil end
end)

-- [ ЛОГИКА ПОИСКА ЦЕЛИ ]
local function GetTarget()
    local best, bestDist = nil, Config.Combat.FOV
    local mousePos = UserInputService:GetMouseLocation()
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        if Config.Combat.TeamCheck and p.Team == LocalPlayer.Team then continue end
        if not p.Character then continue end
        local part = p.Character:FindFirstChild(Config.Combat.AimPart)
        if not part then continue end
        local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
        if not onScreen then continue end
        local dist = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
        if dist >= bestDist then continue end
        if Config.Combat.WallCheck then
            local hit = workspace:FindPartOnRayWithIgnoreList(
                Ray.new(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).Unit * 2000),
                {LocalPlayer.Character, Camera}
            )
            if not (hit and hit:IsDescendantOf(p.Character)) then continue end
        end
        best = part; bestDist = dist
    end
    return best
end

-- [ INFINITE JUMP ]
UserInputService.JumpRequest:Connect(function()
    if not Config.Movement.InfJump then return end
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- [ MAIN LOOP ]
local afkTimer = 0
RunService.RenderStepped:Connect(function(delta)
    -- HUD info
    local fps  = math.floor(1 / delta)
    local ping = "?"; pcall(function() ping = Stats.Network.ServerStatsItem["Data Ping"]:GetValueString() end)
    local spd  = 0
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        spd = math.floor(char.HumanoidRootPart.Velocity.Magnitude)
    end

    -- Aimbot
    local currentTarget = nil
    if Config.Combat.Aimbot and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        currentTarget = GetTarget()
        if currentTarget then
            local predPos = currentTarget.Position + (currentTarget.Velocity * (Config.Combat.Prediction * 0.01))
            local targetCF = CFrame.new(Camera.CFrame.Position, predPos)
            local s = math.max(1, Config.Combat.Smoothness)
            Camera.CFrame = Camera.CFrame:Lerp(targetCF, 1 / s)
        end
    end

    hLabel.Text = string.format(
        "FPS: %d   PING: %s\nSPD: %d\nTARGET: %s",
        fps, ping, spd,
        currentTarget and currentTarget.Parent.Name or "NONE"
    )

    -- FOV Circle
    FOVCircle.Visible = Config.Combat.Aimbot
    if Config.Combat.Aimbot then
        FOVCircle.Radius = Config.Combat.FOV
        FOVCircle.Position = UserInputService:GetMouseLocation()
    end

    -- Crosshair
    local mx = UserInputService:GetMouseLocation()
    CrossH.Visible = Config.Visuals.Crosshair
    CrossV.Visible = Config.Visuals.Crosshair
    if Config.Visuals.Crosshair then
        CrossH.From = Vector2.new(mx.X - 11, mx.Y); CrossH.To = Vector2.new(mx.X + 11, mx.Y)
        CrossV.From = Vector2.new(mx.X, mx.Y - 11); CrossV.To = Vector2.new(mx.X, mx.Y + 11)
    end

    -- Camera FOV
    Camera.FieldOfView = Config.Visuals.CameraFOV

    -- Movement
    if char then
        local hum  = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if hum and root then
            if Config.Movement.Bhop and UserInputService:IsKeyDown(Enum.KeyCode.Space)
               and hum.FloorMaterial ~= Enum.Material.Air then
                hum:ChangeState(Enum.HumanoidStateType.Jumping)
            end
            hum.WalkSpeed = Config.Movement.SpeedHack and Config.Movement.Speed or 16
            if Config.Movement.NoFall then
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            end
        end
    end

    -- Anti-AFK
    if Config.Movement.AntiAFK then
        afkTimer = afkTimer + delta
        if afkTimer >= 60 then
            afkTimer = 0
            local VU = game:GetService("VirtualUser")
            pcall(function() VU:CaptureController(); VU:ClickButton2(Vector2.new()) end)
        end
    else
        afkTimer = 0
    end

    -- ESP
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer or not p.Character then continue end

        -- Highlight ESP
        local hl = p.Character:FindFirstChild("Nexus_HL")
        if Config.Visuals.ESP then
            if not hl then
                hl = Instance.new("Highlight", p.Character)
                hl.Name = "Nexus_HL"; hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            end
            hl.FillColor = ACCENT; hl.FillTransparency = 0.55
            hl.OutlineColor = ACCENT; hl.OutlineTransparency = 0
        elseif hl then hl:Destroy() end

        -- Chams
        local chams = p.Character:FindFirstChild("Nexus_Chams")
        if Config.Visuals.Chams then
            if not chams then
                chams = Instance.new("Highlight", p.Character)
                chams.Name = "Nexus_Chams"; chams.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            end
            chams.FillColor = Color3.fromRGB(255, 70, 70)
            chams.FillTransparency = 0.15; chams.OutlineTransparency = 1
        elseif chams then chams:Destroy() end

        -- Name Labels
        local nameLbl = GetNameLabel(p)
        local hrp = p.Character:FindFirstChild("HumanoidRootPart")
        if Config.Visuals.ESPNames and hrp then
            local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 3.2, 0))
            nameLbl.Visible = onScreen
            if onScreen then
                nameLbl.Position = Vector2.new(screenPos.X, screenPos.Y)
                nameLbl.Text = p.DisplayName
            end
        else nameLbl.Visible = false end

        -- Tracers
        local tracer = GetTracer(p)
        if Config.Visuals.ESPTracers and hrp then
            local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            tracer.Visible = onScreen
            if onScreen then
                local vp = Camera.ViewportSize
                tracer.From = Vector2.new(vp.X / 2, vp.Y)
                tracer.To   = Vector2.new(screenPos.X, screenPos.Y)
            end
        else tracer.Visible = false end

        -- Hitbox
        local head = p.Character:FindFirstChild("Head")
        if head then
            if Config.Visuals.Hitbox then
                local s = Config.Visuals.HitboxSize
                head.Size = Vector3.new(s, s, s)
                head.Transparency = 0.8; head.CanCollide = false
            else
                head.Size = Vector3.new(2, 1, 1); head.Transparency = 0
            end
        end
    end
end)

-- [ TOGGLE WINDOW ]
UserInputService.InputBegan:Connect(function(i, g)
    if g then return end
    if i.KeyCode == Enum.KeyCode.RightShift then Main.Visible = not Main.Visible end
end)

Notify("NEXUS V12", "Team Check ON  ·  RShift — показать/скрыть", 4)
