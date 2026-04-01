-- [[ NEXUS V12 PRIME - (TEAM CHECK UPDATE) ]] --
-- Версия: 12.5.1
-- Тип: Агрессивный / Кастомизируемый
-- Особенности: Team Check, Система сохранений, Резкий Аим, Безлимитный ESP

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local Stats = game:GetService("Stats")

-- [ ИНИЦИАЛИЗАЦИЯ ПЕРЕМЕННЫХ ]
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()
local StartTime = tick()

-- [ ТАБЛИЦА КОНФИГУРАЦИИ ]
local NexusConfig = {
    Visuals = {
        ESP_Enabled = false,
        ESP_Outline = true,
        Hitbox_Enabled = false,
        Hitbox_Size = 25,
        Hitbox_Transparency = 0.6,
        FieldOfView = 90,
        RainbowSpeed = 4
    },
    Combat = {
        Aimbot_Enabled = false,
        Aimbot_Part = "Head",
        Aimbot_Smoothness = 1, 
        Aimbot_FOV = 150,
        WallCheck = false,
        Prediction = 0.165,
        TargetMode = "Closest",
        TeamCheck = true -- ВКЛЮЧЕНО ПО УМОЛЧАНИЮ
    },
    Movement = {
        Bhop_Enabled = false,
        Bhop_Power = 25,
        WalkSpeed = 16,
        AutoJump = true
    },
    Interface = {
        Visible = true,
        Minimized = false,
        RainbowUI = true,
        AccentColor = Color3.fromRGB(191, 64, 255),
        FontSize = 14
    }
}

-- [ СИСТЕМА СОХРАНЕНИЙ ]
local ConfigFileName = "NexusV12_Config.json"

local function SaveSettings()
    if writefile then
        local Data = HttpService:JSONEncode(NexusConfig)
        writefile(ConfigFileName, Data)
        print("[NEXUS] Настройки сохранены!")
    end
end

local function LoadSettings()
    if isfile and isfile(ConfigFileName) then
        local Data = readfile(ConfigFileName)
        local Decoded = HttpService:JSONDecode(Data)
        for i, v in pairs(Decoded) do NexusConfig[i] = v end
        print("[NEXUS] Настройки загружены!")
    end
end

if CoreGui:FindFirstChild("NEXUS_V12_MASTER") then
    CoreGui:FindFirstChild("NEXUS_V12_MASTER"):Destroy()
end

local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "NEXUS_V12_MASTER"

-- [ ФУНКЦИЯ УВЕДОМЛЕНИЙ ]
local function Notify(Title, Content, Duration)
    task.spawn(function()
        local nFrame = Instance.new("Frame", ScreenGui)
        nFrame.Size = UDim2.new(0, 280, 0, 80)
        nFrame.Position = UDim2.new(1, 40, 1, -120)
        nFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        Instance.new("UICorner", nFrame)
        local Stroke = Instance.new("UIStroke", nFrame)
        Stroke.Color = NexusConfig.Interface.AccentColor
        Stroke.Thickness = 2
        
        local T = Instance.new("TextLabel", nFrame)
        T.Size = UDim2.new(1, -20, 0, 35); T.Position = UDim2.new(0, 12, 0, 5)
        T.Text = Title; T.TextColor3 = NexusConfig.Interface.AccentColor
        T.Font = "GothamBold"; T.TextSize = 16; T.BackgroundTransparency = 1; T.TextXAlignment = "Left"
        
        local C = Instance.new("TextLabel", nFrame)
        C.Size = UDim2.new(1, -20, 0, 35); C.Position = UDim2.new(0, 12, 0, 35)
        C.Text = Content; C.TextColor3 = Color3.new(1, 1, 1)
        C.Font = "Gotham"; C.TextSize = 13; C.BackgroundTransparency = 1; C.TextXAlignment = "Left"
        
        nFrame:TweenPosition(UDim2.new(1, -300, 1, -120), "Out", "Quart", 0.5)
        task.wait(Duration or 3)
        nFrame:TweenPosition(UDim2.new(1, 40, 1, -120), "In", "Quart", 0.5)
        task.wait(0.6); nFrame:Destroy()
    end)
end

-- [ ИНФОРМАЦИОННЫЙ HUD ]
local HUD = Instance.new("Frame", ScreenGui)
HUD.Size = UDim2.new(0, 180, 0, 85); HUD.Position = UDim2.new(0, 30, 0, 30)
HUD.BackgroundColor3 = Color3.fromRGB(10, 10, 15); HUD.BackgroundTransparency = 0.2
Instance.new("UICorner", HUD)
local hStroke = Instance.new("UIStroke", HUD)
hStroke.Thickness = 2; hStroke.Color = NexusConfig.Interface.AccentColor

local hLabel = Instance.new("TextLabel", HUD)
hLabel.Size = UDim2.new(1, -20, 1, -10); hLabel.Position = UDim2.new(0, 12, 0, 5)
hLabel.BackgroundTransparency = 1; hLabel.TextColor3 = Color3.new(1, 1, 1)
hLabel.Font = "GothamBold"; hLabel.TextSize = 12; hLabel.TextXAlignment = "Left"
hLabel.Text = "FPS: 0\nPING: 0ms\nSPD: 0\nTARGET: NONE"

-- [ ГЛАВНОЕ ОКНО ]
local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 520, 0, 420); Main.Position = UDim2.new(0.5, -260, 0.5, -210)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 20); Main.ClipsDescendants = true
Instance.new("UICorner", Main)
local MainStroke = Instance.new("UIStroke", Main)
MainStroke.Thickness = 3; MainStroke.Color = NexusConfig.Interface.AccentColor

local Header = Instance.new("Frame", Main)
Header.Size = UDim2.new(1, 0, 0, 60); Header.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Header)
Title.Size = UDim2.new(1, 0, 1, 0); Title.Position = UDim2.new(0, 25, 0, 0)
Title.Text = "NEXUS V12 : MASTER UNIT"; Title.TextColor3 = Color3.new(1, 1, 1); Title.Font = "GothamBold"; Title.TextSize = 22; Title.BackgroundTransparency = 1; Title.TextXAlignment = "Left"

local MinBtn = Instance.new("TextButton", Header)
MinBtn.Size = UDim2.new(0, 45, 0, 45); MinBtn.Position = UDim2.new(1, -60, 0, 7)
MinBtn.Text = "_"; MinBtn.TextColor3 = Color3.new(1, 1, 1); MinBtn.Font = "GothamBold"; MinBtn.TextSize = 26; MinBtn.BackgroundTransparency = 1

MinBtn.MouseButton1Click:Connect(function()
    NexusConfig.Interface.Minimized = not NexusConfig.Interface.Minimized
    local TargetSize = NexusConfig.Interface.Minimized and UDim2.new(0, 520, 0, 60) or UDim2.new(0, 520, 0, 420)
    TweenService:Create(Main, TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = TargetSize}):Play()
end)

-- Драг
local drag, dInput, dStart, sPos
Header.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = true; dStart = i.Position; sPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement then dInput = i end end)
RunService.RenderStepped:Connect(function() if drag and dInput then
    local delta = dInput.Position - dStart
    Main.Position = UDim2.new(sPos.X.Scale, sPos.X.Offset + delta.X, sPos.Y.Scale, sPos.Y.Offset + delta.Y)
end end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end end)

-- Вкладки
local Sidebar = Instance.new("Frame", Main)
Sidebar.Size = UDim2.new(0, 150, 1, -80); Sidebar.Position = UDim2.new(0, 15, 0, 70); Sidebar.BackgroundTransparency = 1
Instance.new("UIListLayout", Sidebar).Padding = UDim.new(0, 12)

local ContentArea = Instance.new("Frame", Main)
ContentArea.Size = UDim2.new(1, -195, 1, -85); ContentArea.Position = UDim2.new(0, 180, 0, 70); ContentArea.BackgroundTransparency = 1

local CombatTab = Instance.new("ScrollingFrame", ContentArea); CombatTab.Size = UDim2.new(1,0,1,0); CombatTab.BackgroundTransparency = 1; CombatTab.Visible = true; CombatTab.ScrollBarThickness = 0
local VisualTab = Instance.new("ScrollingFrame", ContentArea); VisualTab.Size = UDim2.new(1,0,1,0); VisualTab.BackgroundTransparency = 1; VisualTab.Visible = false; VisualTab.ScrollBarThickness = 0
local SettingsTab = Instance.new("ScrollingFrame", ContentArea); SettingsTab.Size = UDim2.new(1,0,1,0); SettingsTab.BackgroundTransparency = 1; SettingsTab.Visible = false; SettingsTab.ScrollBarThickness = 0
Instance.new("UIListLayout", CombatTab).Padding = UDim.new(0, 12); Instance.new("UIListLayout", VisualTab).Padding = UDim.new(0, 12); Instance.new("UIListLayout", SettingsTab).Padding = UDim.new(0, 12)

-- [ КОНСТРУКТОРЫ ]
local function CreateTabButton(Name, Page)
    local B = Instance.new("TextButton", Sidebar)
    B.Size = UDim2.new(1, 0, 0, 45); B.BackgroundColor3 = Color3.fromRGB(25, 25, 35); B.Text = Name; B.TextColor3 = Color3.new(0.8, 0.8, 0.8); B.Font = "GothamBold"; B.TextSize = 14; Instance.new("UICorner", B)
    B.MouseButton1Click:Connect(function()
        CombatTab.Visible = false; VisualTab.Visible = false; SettingsTab.Visible = false; Page.Visible = true
        for _, v in pairs(Sidebar:GetChildren()) do if v:IsA("TextButton") then v.TextColor3 = Color3.new(0.8, 0.8, 0.8) end end
        B.TextColor3 = NexusConfig.Interface.AccentColor
    end)
end

local function CreateToggle(Parent, Text, FlagTable, FlagKey)
    local B = Instance.new("TextButton", Parent)
    B.Size = UDim2.new(1, -10, 0, 45); B.BackgroundColor3 = Color3.fromRGB(20, 20, 25); B.Text = "  " .. Text; B.TextColor3 = Color3.new(0.6, 0.6, 0.6); B.Font = "GothamMedium"; B.TextSize = 14; B.TextXAlignment = "Left"; Instance.new("UICorner", B)
    B.MouseButton1Click:Connect(function()
        FlagTable[FlagKey] = not FlagTable[FlagKey]
        B.BackgroundColor3 = FlagTable[FlagKey] and NexusConfig.Interface.AccentColor or Color3.fromRGB(20, 20, 25)
        B.TextColor3 = FlagTable[FlagKey] and Color3.new(1, 1, 1) or Color3.new(0.6, 0.6, 0.6)
    end)
end

local function CreateSlider(Parent, Text, Min, Max, FlagTable, FlagKey)
    local SFrame = Instance.new("Frame", Parent); SFrame.Size = UDim2.new(1, -10, 0, 60); SFrame.BackgroundTransparency = 1
    local L = Instance.new("TextLabel", SFrame); L.Size = UDim2.new(1, 0, 0, 25); L.Text = Text .. ": " .. FlagTable[FlagKey]; L.TextColor3 = Color3.new(0.9, 0.9, 0.9); L.Font = "Gotham"; L.BackgroundTransparency = 1; L.TextXAlignment = "Left"
    local SBar = Instance.new("TextButton", SFrame); SBar.Size = UDim2.new(1, 0, 0, 12); SBar.Position = UDim2.new(0, 0, 0, 35); SBar.BackgroundColor3 = Color3.fromRGB(35, 35, 45); SBar.Text = ""; Instance.new("UICorner", SBar)
    local Fill = Instance.new("Frame", SBar); Fill.Size = UDim2.new((FlagTable[FlagKey]-Min)/(Max-Min), 0, 1, 0); Fill.BackgroundColor3 = NexusConfig.Interface.AccentColor; Instance.new("UICorner", Fill)
    SBar.InputBegan:Connect(function(I) if I.UserInputType == Enum.UserInputType.MouseButton1 then
        local Conn; Conn = RunService.RenderStepped:Connect(function()
            local Perc = math.clamp((UserInputService:GetMouseLocation().X - SBar.AbsolutePosition.X) / SBar.AbsoluteSize.X, 0, 1)
            Fill.Size = UDim2.new(Perc, 0, 1, 0); local Val = math.floor(Min + (Max - Min) * Perc); L.Text = Text .. ": " .. Val; FlagTable[FlagKey] = Val
        end)
        UserInputService.InputEnded:Connect(function(E) if E.UserInputType == Enum.UserInputType.MouseButton1 then Conn:Disconnect() end end)
    end end)
end

-- [ СБОРКА ]
CreateTabButton("COMBAT", CombatTab); CreateTabButton("VISUALS", VisualTab); CreateTabButton("SETTINGS", SettingsTab)
CreateToggle(CombatTab, "Aimbot Aggressive", NexusConfig.Combat, "Aimbot_Enabled")
CreateToggle(CombatTab, "Team Check", NexusConfig.Combat, "TeamCheck")
CreateToggle(CombatTab, "Ignore Walls", NexusConfig.Combat, "WallCheck")
CreateSlider(CombatTab, "FOV Range", 50, 1200, NexusConfig.Combat, "Aimbot_FOV")
CreateToggle(CombatTab, "Auto-Bhop", NexusConfig.Movement, "Bhop_Enabled")
CreateSlider(CombatTab, "Bhop Velocity", 1, 100, NexusConfig.Movement, "Bhop_Power")

CreateToggle(VisualTab, "Master ESP (Unlimited)", NexusConfig.Visuals, "ESP_Enabled")
CreateToggle(VisualTab, "Hitbox Expander", NexusConfig.Visuals, "Hitbox_Enabled")
CreateSlider(VisualTab, "Hitbox Radius", 2, 100, NexusConfig.Visuals, "Hitbox_Size")
CreateSlider(VisualTab, "Render FOV", 70, 130, NexusConfig.Visuals, "FieldOfView")

local SaveBtn = Instance.new("TextButton", SettingsTab); SaveBtn.Size = UDim2.new(1, -10, 0, 45); SaveBtn.BackgroundColor3 = Color3.fromRGB(30, 80, 30); SaveBtn.Text = "SAVE SETTINGS"; SaveBtn.TextColor3 = Color3.new(1,1,1); SaveBtn.Font = "GothamBold"; Instance.new("UICorner", SaveBtn); SaveBtn.MouseButton1Click:Connect(SaveSettings)
local LoadBtn = Instance.new("TextButton", SettingsTab); LoadBtn.Size = UDim2.new(1, -10, 0, 45); LoadBtn.BackgroundColor3 = Color3.fromRGB(80, 30, 30); LoadBtn.Text = "LOAD SETTINGS"; LoadBtn.TextColor3 = Color3.new(1,1,1); LoadBtn.Font = "GothamBold"; Instance.new("UICorner", LoadBtn); LoadBtn.MouseButton1Click:Connect(LoadSettings)

-- [ ЛОГИКА (TEAM CHECK ОБНОВЛЕН) ]
local FOVCircle = Drawing.new("Circle"); FOVCircle.Thickness = 2; FOVCircle.NumSides = 70; FOVCircle.Visible = false

local function GetClosest()
    local Target = nil; local MinDist = NexusConfig.Combat.Aimbot_FOV
    for _, P in pairs(Players:GetPlayers()) do
        -- ПРОВЕРКА TEAM CHECK ЗДЕСЬ
        if P ~= LocalPlayer and (not NexusConfig.Combat.TeamCheck or P.Team ~= LocalPlayer.Team) and P.Character and P.Character:FindFirstChild(NexusConfig.Combat.Aimbot_Part) then
            local Body = P.Character[NexusConfig.Combat.Aimbot_Part]
            local Pos, OnScreen = Camera:WorldToViewportPoint(Body.Position)
            if OnScreen then
                local Dist = (Vector2.new(Pos.X, Pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                if Dist < MinDist then
                    if NexusConfig.Combat.WallCheck then MinDist = Dist; Target = Body
                    else
                        local Hit = workspace:FindPartOnRayWithIgnoreList(Ray.new(Camera.CFrame.Position, (Body.Position - Camera.CFrame.Position).Unit * 1000), {LocalPlayer.Character, Camera})
                        if Hit and Hit:IsDescendantOf(P.Character) then MinDist = Dist; Target = Body end
                    end
                end
            end
        end
    end
    return Target
end

RunService.RenderStepped:Connect(function(Delta)
    local FPS = math.floor(1/Delta); local Ping = Stats.Network.ServerStatsItem["Data Ping"]:GetValueString()
    local Speed = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) and math.floor(LocalPlayer.Character.HumanoidRootPart.Velocity.Magnitude) or 0
    
    if NexusConfig.Interface.RainbowUI then
        local C = Color3.fromHSV(tick() % 4 / 4, 0.8, 1); NexusConfig.Interface.AccentColor = C; MainStroke.Color = C; hStroke.Color = C; FOVCircle.Color = C
    end
    
    FOVCircle.Visible = NexusConfig.Combat.Aimbot_Enabled; FOVCircle.Radius = NexusConfig.Combat.Aimbot_FOV; FOVCircle.Position = UserInputService:GetMouseLocation()
    Camera.FieldOfView = NexusConfig.Visuals.FieldOfView
    
    local CurrentTarget = nil
    if NexusConfig.Combat.Aimbot_Enabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        CurrentTarget = GetClosest()
        if CurrentTarget then Camera.CFrame = CFrame.new(Camera.CFrame.Position, CurrentTarget.Position + (CurrentTarget.Velocity * NexusConfig.Combat.Prediction)) end
    end
    hLabel.Text = "FPS: " .. FPS .. "\nPING: " .. Ping .. "\nSPD: " .. Speed .. "\nTARGET: " .. (CurrentTarget and CurrentTarget.Parent.Name or "NONE")

    local Char = LocalPlayer.Character
    if NexusConfig.Movement.Bhop_Enabled and Char and Char:FindFirstChild("Humanoid") then
        local Hum = Char.Humanoid; local Root = Char.HumanoidRootPart
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) and Hum.FloorMaterial ~= Enum.Material.Air then
            Hum:ChangeState(Enum.HumanoidStateType.Jumping); NexusConfig.Movement.WalkSpeed = NexusConfig.Movement.WalkSpeed + 5
        elseif Hum.FloorMaterial ~= Enum.Material.Air then NexusConfig.Movement.WalkSpeed = 16 end
        Root.Velocity = Vector3.new(Hum.MoveDirection.X * NexusConfig.Movement.WalkSpeed, Root.Velocity.Y, Hum.MoveDirection.Z * NexusConfig.Movement.WalkSpeed)
    end

    for _, P in pairs(Players:GetPlayers()) do
        if P ~= LocalPlayer and P.Character then
            local Head = P.Character:FindFirstChild("Head")
            if Head then
                if NexusConfig.Visuals.Hitbox_Enabled then
                    Head.Size = Vector3.new(NexusConfig.Visuals.Hitbox_Size, NexusConfig.Visuals.Hitbox_Size, NexusConfig.Visuals.Hitbox_Size)
                    Head.Transparency = NexusConfig.Visuals.Hitbox_Transparency; Head.Color = NexusConfig.Interface.AccentColor; Head.CanCollide = false
                else Head.Size = Vector3.new(2, 1, 1); Head.Transparency = 0 end
            end
            local HL = P.Character:FindFirstChild("Nexus_HL")
            if NexusConfig.Visuals.ESP_Enabled then
                if not HL then HL = Instance.new("Highlight", P.Character); HL.Name = "Nexus_HL"; HL.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop end
                HL.FillColor = NexusConfig.Interface.AccentColor
            elseif HL then HL:Destroy() end
        end
    end
end)

Notify("NEXUS V12", "TEAM CHECK АКТИВИРОВАН", 4)
