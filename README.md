--// ============================================================================
--// RYU HUB - MONOCHROME MASTER EDITION (GPO FISHMAN + LAW RAID + BOSS + ANTI-STUN)
--// ============================================================================

--// 0. ULTRA-AGGRESSIVER CLEANUP
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local function DestroyOldGuis()
    local targets = {CoreGui, LocalPlayer:WaitForChild("PlayerGui")}
    if gethui then table.insert(targets, gethui()) end
    
    for _, container in ipairs(targets) do
        pcall(function()
            for _, v in pairs(container:GetChildren()) do
                if string.find(v.Name, "Ryu") then
                    v:Destroy()
                end
            end
        end)
    end
end
DestroyOldGuis()

--// 1. SERVICES & CORE SETUP
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")

local camera = Workspace.CurrentCamera
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() 
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end 
end)

--// 2. RYU CONFIGURATION (GLOBAL STATE)
local RyuConfig = {
    -- Basics
    SpeedHack = false, SpeedValue = 35, 
    HighJump = false, JumpValue = 50, 
    Fly = false, FlySpeed = 50, FlyKeybind = "LeftControl",
    LowGravity = false, GravityValue = 100,
    FOVChanger = false, FOVValue = 90,
    NoStun = false, 
    ESP = false, ESPTransparency = 50,
    ChestESP = false, ChestESPTransparency = 25,
    -- Automation: Boss & Law
    AutoBossFarm = false, BossName = "Juzo the Diamondback", BossHoverDistance = 13,
    AutoLawRaid = false, 
    SkillCooldown = 1.0, TweenSpeed = 45, FarmKeybind = "X", AutoServerHop = false,
    -- Automation: GPO Fishman
    AutoGPOFarm = false, AutoQuest = false, QuestInterval = 45,
    TargetMob = "Fishman Karate User", TargetNPC = "Becky", TargetWeapon = "Combat",
    GPOTweenSpeed = 50, KillHeight = 7, FishmanSpeed = 150, ElevatorSpeed = 150,
    -- Config System
    SelectedSlot = 1 
}

local UIRegistry = { Toggles = {}, Sliders = {}, TextBoxes = {} }

--// 3. MONOCHROME THEME (DEEP BLACK & PURE WHITE)
local Theme = {
    Background = Color3.fromRGB(10, 10, 10),
    Sidebar = Color3.fromRGB(15, 15, 15),
    SectionBG = Color3.fromRGB(20, 20, 20),
    Text = Color3.fromRGB(255, 255, 255),
    SubText = Color3.fromRGB(170, 170, 170),
    CloudLight = Color3.fromRGB(255, 255, 255),
    CloudDark = Color3.fromRGB(100, 100, 100),
    Accent = Color3.fromRGB(255, 255, 255),
    ToggleOff = Color3.fromRGB(40, 40, 40),
    ToggleOn = Color3.fromRGB(255, 255, 255),
    Stroke = Color3.fromRGB(45, 45, 45),
    Warning = Color3.fromRGB(255, 80, 80)
}

local RyuHub = Instance.new("ScreenGui")
RyuHub.Name = "RyuHubPremium"
RyuHub.ResetOnSpawn = false
RyuHub.IgnoreGuiInset = true
RyuHub.Parent = guiParent

--// RAINBOW OVERHEAD TITLE (GPO Feature)
local function AddRainbowTag(character)
    local head = character:WaitForChild("Head", 5)
    if head then
        if head:FindFirstChild("RyuHubTag") then head.RyuHubTag:Destroy() end
        local bg = Instance.new("BillboardGui"); bg.Name = "RyuHubTag"; bg.Size = UDim2.new(0, 200, 0, 50); bg.StudsOffset = Vector3.new(0, 3, 0); bg.AlwaysOnTop = true; bg.Parent = head
        local txt = Instance.new("TextLabel"); txt.Size = UDim2.new(1, 0, 1, 0); txt.BackgroundTransparency = 1; txt.Text = "RYUHUB"; txt.Font = Enum.Font.GothamBlack; txt.TextSize = 22; txt.TextStrokeTransparency = 0; txt.Parent = bg
        task.spawn(function()
            while bg.Parent do
                txt.TextColor3 = Color3.fromHSV(tick() % 5 / 5, 1, 1)
                task.wait(0.1) 
            end
        end)
    end
end
if LocalPlayer.Character then AddRainbowTag(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(AddRainbowTag)

--// MAJESTIC NOTIFICATION SYSTEM
local function MajesticNotify(title, text)
    local overlay = Instance.new("Frame", RyuHub); overlay.Size = UDim2.new(1, 0, 1, 0); overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0); overlay.BackgroundTransparency = 1; overlay.ZIndex = 100
    local box = Instance.new("Frame", overlay); box.Size = UDim2.new(0, 0, 0, 0); box.Position = UDim2.new(0.5, 0, 0.5, 0); box.AnchorPoint = Vector2.new(0.5, 0.5); box.BackgroundColor3 = Theme.Background; box.ClipsDescendants = true; box.ZIndex = 101; Instance.new("UICorner", box).CornerRadius = UDim.new(0, 12)
    local boxStroke = Instance.new("UIStroke", box); boxStroke.Color = Theme.Accent; boxStroke.Thickness = 2; boxStroke.Transparency = 1
    local titleLabel = Instance.new("TextLabel", box); titleLabel.Size = UDim2.new(1, 0, 0, 40); titleLabel.Position = UDim2.new(0, 0, 0, 20); titleLabel.BackgroundTransparency = 1; titleLabel.Text = string.upper(title); titleLabel.TextColor3 = Theme.Accent; titleLabel.Font = Enum.Font.GothamBlack; titleLabel.TextSize = 24; titleLabel.ZIndex = 102
    local descLabel = Instance.new("TextLabel", box); descLabel.Size = UDim2.new(1, 0, 0, 30); descLabel.Position = UDim2.new(0, 0, 0, 60); descLabel.BackgroundTransparency = 1; descLabel.Text = text; descLabel.TextColor3 = Theme.Text; descLabel.Font = Enum.Font.Gotham; descLabel.TextSize = 14; descLabel.ZIndex = 102
    
    TweenService:Create(overlay, TweenInfo.new(0.5), {BackgroundTransparency = 0.4}):Play(); TweenService:Create(box, TweenInfo.new(0.6, Enum.EasingStyle.Exponential, Enum.EasingDirection.Out), {Size = UDim2.new(0, 350, 0, 120)}):Play(); TweenService:Create(boxStroke, TweenInfo.new(0.5), {Transparency = 0}):Play()
    task.delay(2.5, function()
        TweenService:Create(overlay, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play(); TweenService:Create(boxStroke, TweenInfo.new(0.5), {Transparency = 1}):Play()
        local out = TweenService:Create(box, TweenInfo.new(0.5, Enum.EasingStyle.Exponential, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
        out:Play(); out.Completed:Wait(); overlay:Destroy()
    end)
end

local NotificationContainer = Instance.new("Frame"); NotificationContainer.Name = "RyuNotifications"; NotificationContainer.Size = UDim2.new(0, 260, 1, -40); NotificationContainer.Position = UDim2.new(1, -280, 0, 20); NotificationContainer.BackgroundTransparency = 1; NotificationContainer.Parent = guiParent
local NotifLayout = Instance.new("UIListLayout", NotificationContainer); NotifLayout.SortOrder = Enum.SortOrder.LayoutOrder; NotifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom; NotifLayout.Padding = UDim.new(0, 8)

local RyuNotify = {}
function RyuNotify:Send(title, text, duration)
    duration = duration or 3
    local NotifFrame = Instance.new("Frame", NotificationContainer); NotifFrame.Size = UDim2.new(1, 0, 0, 60); NotifFrame.BackgroundColor3 = Theme.SectionBG; NotifFrame.BackgroundTransparency = 1; NotifFrame.BorderSizePixel = 0; Instance.new("UICorner", NotifFrame).CornerRadius = UDim.new(0, 6)
    local Stroke = Instance.new("UIStroke", NotifFrame); Stroke.Color = Theme.Stroke; Stroke.Transparency = 1; Stroke.Thickness = 1
    local AccentLine = Instance.new("Frame", NotifFrame); AccentLine.Size = UDim2.new(0, 3, 0.8, 0); AccentLine.Position = UDim2.new(0, 4, 0.1, 0); AccentLine.BackgroundColor3 = Theme.Accent; AccentLine.BackgroundTransparency = 1; Instance.new("UICorner", AccentLine).CornerRadius = UDim.new(1, 0)
    local TitleLabel = Instance.new("TextLabel", NotifFrame); TitleLabel.Size = UDim2.new(1, -20, 0, 20); TitleLabel.Position = UDim2.new(0, 15, 0, 8); TitleLabel.BackgroundTransparency = 1; TitleLabel.Text = title; TitleLabel.TextColor3 = Theme.Accent; TitleLabel.TextTransparency = 1; TitleLabel.Font = Enum.Font.GothamBold; TitleLabel.TextSize = 13; TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    local DescLabel = Instance.new("TextLabel", NotifFrame); DescLabel.Size = UDim2.new(1, -20, 0, 20); DescLabel.Position = UDim2.new(0, 15, 0, 28); DescLabel.BackgroundTransparency = 1; DescLabel.Text = text; DescLabel.TextColor3 = Theme.Text; DescLabel.TextTransparency = 1; DescLabel.Font = Enum.Font.Gotham; DescLabel.TextSize = 11; DescLabel.TextXAlignment = Enum.TextXAlignment.Left

    TweenService:Create(NotifFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0}):Play(); TweenService:Create(Stroke, TweenInfo.new(0.3), {Transparency = 0}):Play(); TweenService:Create(AccentLine, TweenInfo.new(0.3), {BackgroundTransparency = 0}):Play(); TweenService:Create(TitleLabel, TweenInfo.new(0.3), {TextTransparency = 0}):Play(); TweenService:Create(DescLabel, TweenInfo.new(0.3), {TextTransparency = 0}):Play()
    task.spawn(function()
        task.wait(duration)
        local fadeOut = TweenService:Create(NotifFrame, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 0)})
        TweenService:Create(Stroke, TweenInfo.new(0.3), {Transparency = 1}):Play(); TweenService:Create(AccentLine, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play(); TweenService:Create(TitleLabel, TweenInfo.new(0.3), {TextTransparency = 1}):Play(); TweenService:Create(DescLabel, TweenInfo.new(0.3), {TextTransparency = 1}):Play()
        fadeOut:Play(); fadeOut.Completed:Wait(); NotifFrame:Destroy()
    end)
end

--// 4. SERVER HOP MODULE
local ServerHopModule = { IsHopping = false }
function ServerHopModule:Hop()
    if ServerHopModule.IsHopping then return end
    ServerHopModule.IsHopping = true; RyuNotify:Send("Server Hop", "Searching for available servers...", 4)
    task.spawn(function()
        local placeId = game.PlaceId; local jobId = game.JobId
        local url = "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"
        local success, result = pcall(function() return game:HttpGet(url) end)
        if success then
            local data = HttpService:JSONDecode(result)
            if data and data.data then
                for _, server in ipairs(data.data) do
                    if server.playing < server.maxPlayers and server.id ~= jobId then
                        RyuNotify:Send("Server Hop", "Server found! Initiating teleport...", 5); task.wait(1)
                        TeleportService:TeleportToPlaceInstance(placeId, server.id, LocalPlayer); return
                    end
                end
            end
        end
        RyuNotify:Send("Error", "No available servers found. Retrying later.", 3); ServerHopModule.IsHopping = false
    end)
end

--// 5. BACKEND MODULES (AGGRESSIVE ANTI-STUN & VELOCITY JUMP)
local PlayerMods = { DefaultSpeed = 16, DefaultGravity = 196.2, DefaultFOV = 70, SpeedEnabled = false, JumpEnabled = false, GravityEnabled = false, FOVEnabled = false, NoStunEnabled = false, EnforceLoop = nil }

-- VELOCITY JUMP HACK
UserInputService.JumpRequest:Connect(function()
    if PlayerMods.JumpEnabled and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        local root = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hum and root then
            local state = hum:GetState()
            if state == Enum.HumanoidStateType.Running or state == Enum.HumanoidStateType.RunningNoPhysics or state == Enum.HumanoidStateType.Landed then
                root.Velocity = Vector3.new(root.Velocity.X, RyuConfig.JumpValue, root.Velocity.Z)
            end
        end
    end
end)

function PlayerMods:StartEnforcing()
    if PlayerMods.EnforceLoop then return end
    PlayerMods.EnforceLoop = RunService.Heartbeat:Connect(function()
        local character = LocalPlayer.Character; if not character then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid"); local rootPart = character:FindFirstChild("HumanoidRootPart"); if not humanoid or not rootPart then return end
        
        if PlayerMods.SpeedEnabled and humanoid.WalkSpeed ~= RyuConfig.SpeedValue then humanoid.WalkSpeed = RyuConfig.SpeedValue end
        if PlayerMods.GravityEnabled and Workspace.Gravity ~= RyuConfig.GravityValue then Workspace.Gravity = RyuConfig.GravityValue end
        if PlayerMods.FOVEnabled and camera.FieldOfView ~= RyuConfig.FOVValue then camera.FieldOfView = RyuConfig.FOVValue end

        -- AGGRESSIVE M1 ANTI-STUN
        if PlayerMods.NoStunEnabled then
            if rootPart.Anchored and not RyuConfig.AutoBossFarm and not RyuConfig.AutoLawRaid and not RyuConfig.AutoGPOFarm then rootPart.Anchored = false end
            if humanoid.PlatformStand and not RyuConfig.Fly then humanoid.PlatformStand = false end
            if humanoid.Sit then humanoid.Sit = false end
            if not humanoid.AutoRotate then humanoid.AutoRotate = true end
            
            local targetSpeed = PlayerMods.SpeedEnabled and RyuConfig.SpeedValue or PlayerMods.DefaultSpeed
            if humanoid.WalkSpeed <= 2 then humanoid.WalkSpeed = targetSpeed end
            
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            
            for _, force in pairs(rootPart:GetChildren()) do
                if force:IsA("BodyVelocity") or force:IsA("BodyPosition") or force:IsA("LinearVelocity") or force:IsA("AlignPosition") or force:IsA("BodyGyro") then
                    if not string.find(force.Name, "Ryu") then force:Destroy() end
                end
            end
            
            for _, child in pairs(character:GetChildren()) do
                if child:IsA("ValueBase") or child:IsA("BoolValue") or child:IsA("StringValue") or child:IsA("NumberValue") or child:IsA("ObjectValue") then
                    local name = string.lower(child.Name)
                    if name:find("stun") or name:find("hit") or name:find("combo") or name:find("m1") or name:find("attack") or name:find("action") or name:find("busy") or name:find("snare") or name:find("freeze") or name:find("ragdoll") or name:find("cooldown") or name:find("stagger") then 
                        child:Destroy() 
                    end
                end
            end
        end
    end)
end

function PlayerMods:SetSpeed(enabled) PlayerMods.SpeedEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled and LocalPlayer.Character then local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid"); if hum then hum.WalkSpeed = PlayerMods.DefaultSpeed end end end
function PlayerMods:SetJumpPower(enabled) PlayerMods.JumpEnabled = enabled; PlayerMods:StartEnforcing() end
function PlayerMods:SetGravity(enabled) PlayerMods.GravityEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled then Workspace.Gravity = PlayerMods.DefaultGravity end end
function PlayerMods:SetFOV(enabled) PlayerMods.FOVEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled then camera.FieldOfView = PlayerMods.DefaultFOV end end
function PlayerMods:SetNoStun(enabled) PlayerMods.NoStunEnabled = enabled; PlayerMods:StartEnforcing() end

local FlyModule = { Enabled = false, FlyLoop = nil }
function FlyModule:Toggle(state)
    FlyModule.Enabled = state
    local character = LocalPlayer.Character; if not character then return end
    local root = character:FindFirstChild("HumanoidRootPart"); local hum = character:FindFirstChildOfClass("Humanoid"); if not root or not hum then return end
    if state then
        local bg = Instance.new("BodyGyro"); bg.Name = "RyuFlyGyro"; bg.P = 9e4; bg.maxTorque = Vector3.new(9e9, 9e9, 9e9); bg.cframe = root.CFrame; bg.Parent = root
        local bv = Instance.new("BodyVelocity"); bv.Name = "RyuFlyVelocity"; bv.velocity = Vector3.new(0,0,0); bv.maxForce = Vector3.new(9e9, 9e9, 9e9); bv.Parent = root
        hum.PlatformStand = true
        FlyModule.FlyLoop = RunService.RenderStepped:Connect(function()
            if not FlyModule.Enabled or not LocalPlayer.Character or not root or not hum then FlyModule:Toggle(false); return end
            hum.PlatformStand = true; bg.cframe = camera.CFrame
            local moveDir = Vector3.new()
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0, 1, 0) end
            if moveDir.Magnitude > 0 then moveDir = moveDir.Unit end; bv.velocity = moveDir * RyuConfig.FlySpeed
        end)
    else
        if FlyModule.FlyLoop then FlyModule.FlyLoop:Disconnect(); FlyModule.FlyLoop = nil end
        if root:FindFirstChild("RyuFlyGyro") then root.RyuFlyGyro:Destroy() end; if root:FindFirstChild("RyuFlyVelocity") then root.RyuFlyVelocity:Destroy() end; if hum then hum.PlatformStand = false end
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    local success, targetKey = pcall(function() return Enum.KeyCode[RyuConfig.FlyKeybind] end)
    if success and targetKey and input.KeyCode == targetKey then
        RyuConfig.Fly = not RyuConfig.Fly
        FlyModule:Toggle(RyuConfig.Fly)
        if UIRegistry.Toggles["Fly"] then UIRegistry.Toggles["Fly"](RyuConfig.Fly) end
        RyuNotify:Send("Fly Mode", RyuConfig.Fly and "Activated via Hotkey" or "Deactivated via Hotkey", 2)
    end
end)

local ESPModule = { Enabled = false, FillColor = Color3.fromRGB(255, 255, 255), OutlineColor = Color3.fromRGB(255, 255, 255), Highlights = {} }
function ESPModule:UpdateTransparency(val) local realVal = val / 100; for _, highlight in pairs(ESPModule.Highlights) do if highlight and highlight.Parent then highlight.FillTransparency = realVal end end end
function ESPModule:Toggle(state)
    ESPModule.Enabled = state; if not state then for _, highlight in pairs(ESPModule.Highlights) do if highlight and highlight.Parent then highlight:Destroy() end end; table.clear(ESPModule.Highlights); return end
    local function applyESP(player)
        if player == LocalPlayer then return end
        local function attachHighlight(character)
            if not ESPModule.Enabled then return end; if character:FindFirstChild("RyuESP") then character.RyuESP:Destroy() end
            local highlight = Instance.new("Highlight"); highlight.Name = "RyuESP"; highlight.Adornee = character; highlight.FillColor = ESPModule.FillColor; highlight.OutlineColor = ESPModule.OutlineColor; highlight.FillTransparency = RyuConfig.ESPTransparency / 100; highlight.OutlineTransparency = 0.1; highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop; highlight.Parent = character; table.insert(ESPModule.Highlights, highlight)
        end
        if player.Character then attachHighlight(player.Character) end; player.CharacterAdded:Connect(attachHighlight)
    end
    for _, player in pairs(Players:GetPlayers()) do applyESP(player) end; Players.PlayerAdded:Connect(applyESP)
end

local ChestESPModule = { Enabled = false, Highlights = {}, ScanLoop = nil }
function ChestESPModule:UpdateTransparency(val) local realVal = val / 100; for _, highlight in pairs(ChestESPModule.Highlights) do if highlight and highlight.Parent then highlight.FillTransparency = realVal end end end
function ChestESPModule:Toggle(state)
    ChestESPModule.Enabled = state
    if not state then if ChestESPModule.ScanLoop then ChestESPModule.ScanLoop:Disconnect(); ChestESPModule.ScanLoop = nil end; for _, highlight in pairs(ChestESPModule.Highlights) do if highlight and highlight.Parent then highlight:Destroy() end end; table.clear(ChestESPModule.Highlights); return end
    local function applyChestHighlight(targetModel, color)
        if targetModel:FindFirstChild("RyuChestESP") then return end
        local highlight = Instance.new("Highlight"); highlight.Name = "RyuChestESP"; highlight.Adornee = targetModel; highlight.FillColor = color; highlight.OutlineColor = Color3.fromRGB(255, 255, 255); highlight.FillTransparency = RyuConfig.ChestESPTransparency / 100; highlight.OutlineTransparency = 0.1; highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop; highlight.Parent = targetModel; table.insert(ChestESPModule.Highlights, highlight)
    end
    local function scanForChests()
        if not ChestESPModule.Enabled then return end
        local searchContainer = Workspace:FindFirstChild("Effects") or Workspace
        for _, obj in pairs(searchContainer:GetChildren()) do
            if obj:IsA("Model") or obj:IsA("Folder") then
                for _, desc in pairs(obj:GetDescendants()) do
                    local text = ""
                    if desc:IsA("ProximityPrompt") then text = string.lower(desc.ObjectText .. " " .. desc.ActionText)
                    elseif desc:IsA("TextLabel") or desc:IsA("StringValue") then text = string.lower(desc.Text or desc.Value or "")
                    elseif desc:IsA("BasePart") then text = string.lower(desc.Name) end
                    if text:find("mythical") then applyChestHighlight(obj, Color3.fromRGB(190, 0, 255)); break
                    elseif text:find("legendary") then applyChestHighlight(obj, Color3.fromRGB(255, 215, 0)); break
                    elseif text:find("rare") then applyChestHighlight(obj, Color3.fromRGB(150, 150, 150)); break
                    elseif text:find("chest") or text:find("crate") or text:find("box") then applyChestHighlight(obj, Color3.fromRGB(100, 100, 100)); break end
                end
            end
        end
    end
    scanForChests()
    ChestESPModule.ScanLoop = RunService.Heartbeat:Connect(function() if math.random(1, 120) == 1 then scanForChests() end end)
end

--// ============================================================================
--// GPO FISHMAN CAVE FARM MODULE
--// ============================================================================
local function ToggleHover(state)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    if state then
        local bp = root:FindFirstChild("RyuHover")
        if not bp then
            bp = Instance.new("BodyPosition"); bp.Name = "RyuHover"; bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bp.D = 500; bp.P = 50000; bp.Parent = root
        end
        bp.Position = root.Position
    else
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp:Destroy() end
    end
end

local function EquipTargetWeapon()
    local char = LocalPlayer.Character; local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    local targetWep = RyuConfig.TargetWeapon
    if char:FindFirstChild(targetWep) then return true end
    for _, item in pairs(char:GetChildren()) do
        if item:IsA("Tool") and (item.Name:lower():find(targetWep:lower()) or item:GetAttribute("MeleeTool")) then return true end
    end
    local tool = LocalPlayer.Backpack:FindFirstChild(targetWep)
    if not tool then
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
            if item:IsA("Tool") and (item.Name:lower():find(targetWep:lower()) or item:GetAttribute("MeleeTool") or item.Name:lower():find("melee") or item.Name:lower():find("sword") or item.Name:lower():find("combat")) then
                tool = item; break
            end
        end
    end
    if not tool then
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do if item:IsA("Tool") then tool = item; break end end
    end
    if tool and tool.Parent == LocalPlayer.Backpack then hum:EquipTool(tool); task.wait(0.1); return true end
    return false
end

local function PerformMeleeAttack()
    pcall(function()
        local char = LocalPlayer.Character; local tool = char and char:FindFirstChildOfClass("Tool")
        if tool then tool:Activate() end
        if mouse1click then mouse1click() end
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1); task.wait(); VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
    end)
end

local function SafeTween(targetCFrame, customSpeed)
    local char = LocalPlayer.Character; local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local startPos = root.Position; local targetPos = targetCFrame.Position; local dist = (targetPos - startPos).Magnitude
    local speed = customSpeed or RyuConfig.GPOTweenSpeed
    local timeToTake = dist / speed
    if timeToTake < 0.1 then root.CFrame = targetCFrame; return end
    local startTime = tick()
    while tick() - startTime < timeToTake do
        if not RyuConfig.AutoGPOFarm then break end
        local alpha = (tick() - startTime) / timeToTake
        local intermediatePos = startPos:Lerp(targetPos, alpha)
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp.Position = intermediatePos end
        root.CFrame = CFrame.lookAt(intermediatePos, targetPos)
        RunService.Heartbeat:Wait()
    end
    local bpFinal = root:FindFirstChild("RyuHover")
    if bpFinal then bpFinal.Position = targetPos end
    root.CFrame = targetCFrame
end

local function CheckQuestActive()
    local active = false
    pcall(function()
        local qFolder = LocalPlayer:FindFirstChild("Quest")
        if qFolder and qFolder:FindFirstChild("CurrentQuest") then
            local val = qFolder.CurrentQuest.Value
            if val and val ~= "" and val ~= "None" then active = true end
        end
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg then
            for _, v in pairs(pg:GetDescendants()) do
                if v:IsA("TextLabel") and v.Visible then
                    local txt = v.Text:lower()
                    if txt:find("completed") then active = false; return end
                    if not active and v.AbsolutePosition.X < 500 and v.AbsolutePosition.Y < 500 then
                        if txt:match("%d+/%d+") or txt:match("%d+%s*/%s*%d+") then active = true end
                    end
                end
            end
        end
    end)
    return active
end

local function FetchQuest()
    local npc = Workspace:FindFirstChild(RyuConfig.TargetNPC, true)
    if npc then
        local npcPos = npc:IsA("Model") and npc:GetPivot() or npc.CFrame
        local char = LocalPlayer.Character; local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            SafeTween(npcPos * CFrame.new(0, 0, 3.5))
            root.CFrame = CFrame.lookAt(root.Position, Vector3.new(npcPos.Position.X, root.Position.Y, npcPos.Position.Z))
            pcall(function()
                local QuestEvent = ReplicatedStorage.Events.Quest
                QuestEvent:InvokeServer({"npcChat", true})
                local questString = "Help " .. RyuConfig.TargetNPC
                QuestEvent:InvokeServer({"takequest", questString}); QuestEvent:InvokeServer({"takequest", RyuConfig.TargetNPC})
                QuestEvent:InvokeServer({"takequest"}); QuestEvent:InvokeServer("takequest")
                QuestEvent:InvokeServer({"acceptquest"}); QuestEvent:InvokeServer("acceptquest")
            end)
            task.wait(0.5)
        end
    end
end

task.spawn(function()
    while true do
        task.wait(180) 
        if RyuConfig.AutoGPOFarm and RyuConfig.TargetNPC ~= "" and RyuConfig.TargetNPC ~= "None" then
            if not CheckQuestActive() then RyuNotify:Send("Fail-Safe", "Quest-Sicherung greift ein!", 2); FetchQuest() end
        end
    end
end)

local GPOFarmModule = { FarmLoop = nil }
function GPOFarmModule:Toggle(state)
    RyuConfig.AutoGPOFarm = state
    if not state then 
        ToggleHover(false); if self.FarmLoop then self.FarmLoop:Disconnect(); self.FarmLoop = nil end 
        RyuNotify:Send("GPO Farm", "Fishman Cave Farm stopped.", 2)
        return 
    end
    
    -- Turn off other farms
    if RyuConfig.AutoBossFarm then BossFarmModule:Toggle(false); if UIRegistry.Toggles["AutoBossFarm"] then UIRegistry.Toggles["AutoBossFarm"](false) end end
    if RyuConfig.AutoLawRaid then LawRaidModule:Toggle(false); if UIRegistry.Toggles["AutoLawRaid"] then UIRegistry.Toggles["AutoLawRaid"](false) end end

    RyuNotify:Send("GPO Farm", "Fishman Cave Farm started!", 3)
    self.FarmLoop = RunService.Heartbeat:Connect(function()
        if not RyuConfig.AutoGPOFarm then return end
        local char = LocalPlayer.Character; local root = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not root or not hum or hum.Health <= 0 then return end

        ToggleHover(true)
        if RyuConfig.AutoQuest and RyuConfig.TargetNPC and RyuConfig.TargetNPC ~= "" then
            if not CheckQuestActive() then FetchQuest(); return end
        end

        if RyuConfig.TargetMob and RyuConfig.TargetMob ~= "" then
            local npcs = Workspace:FindFirstChild("NPCs"); if not npcs then return end
            local targetMob = nil; local closestDist = math.huge
            for _, npc in pairs(npcs:GetChildren()) do
                if npc.Name == RyuConfig.TargetMob then
                    local mHum = npc:FindFirstChildOfClass("Humanoid"); local mRoot = npc:FindFirstChild("HumanoidRootPart")
                    local isRagdolled = npc:FindFirstChild("Rag") or (npc.Parent and npc.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                    if mHum and mRoot and mHum.Health > 0 and not isRagdolled then
                        local d = (root.Position - mRoot.Position).Magnitude
                        if d < closestDist then closestDist = d; targetMob = npc end
                    end
                end
            end
            
            if targetMob then
                local mRoot = targetMob:FindFirstChild("HumanoidRootPart"); local mHum = targetMob:FindFirstChildOfClass("Humanoid")
                EquipTargetWeapon()
                while RyuConfig.AutoGPOFarm and mHum and mHum.Health > 0 do
                    local isRagdolled = targetMob:FindFirstChild("Rag") or (targetMob.Parent and targetMob.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                    if isRagdolled then mRoot.Size = Vector3.new(2, 2, 1); task.wait(0.2); continue end
                    if RyuConfig.AutoQuest and not CheckQuestActive() then break end
                    
                    mRoot.Size = Vector3.new(20, 20, 20); mRoot.CanCollide = false; mRoot.Velocity = Vector3.zero; mRoot.RotVelocity = Vector3.zero
                    local curFlatDir = Vector3.new(root.Position.X - mRoot.Position.X, 0, root.Position.Z - mRoot.Position.Z)
                    if curFlatDir.Magnitude < 0.1 then curFlatDir = Vector3.new(1, 0, 0) end
                    local attackPos = mRoot.Position + (curFlatDir.Unit * 3) + Vector3.new(0, RyuConfig.KillHeight, 0)
                    
                    if (root.Position - attackPos).Magnitude > 5 then SafeTween(CFrame.lookAt(attackPos, mRoot.Position)) end
                    local bp = root:FindFirstChild("RyuHover"); if bp then bp.Position = attackPos end
                    root.CFrame = CFrame.lookAt(root.Position, Vector3.new(mRoot.Position.X, root.Position.Y, mRoot.Position.Z))
                    PerformMeleeAttack(); task.wait(0.05)
                end
                if mRoot then mRoot.Size = Vector3.new(2, 2, 1) end
            end
        end
    end)
end

--// SMART TP HELPERS (GPO)
local function GPOSmartTP(useSky)
    task.spawn(function()
        local cave = Workspace:FindFirstChild("Fishman Cave", true) or Workspace:FindFirstChild("FishmanIsland", true)
        if not cave then return end
        local targetPos = cave:IsA("Model") and cave:GetPivot().Position or cave.CFrame.Position
        local char = LocalPlayer.Character; local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hipHeight = hum and hum.HipHeight or 2.15; local floorOffset = hipHeight + (root.Size.Y / 2)
        
        local platform = Instance.new("Part"); platform.Name = "RyuAntiAirPlatform"; platform.Size = Vector3.new(40, 3, 40); platform.Anchored = true; platform.CanCollide = true; platform.Transparency = 0.5; platform.Material = Enum.Material.ForceField; platform.Color = Color3.fromRGB(255, 255, 255); platform.CFrame = CFrame.new(root.Position - Vector3.new(0, floorOffset, 0)); platform.Parent = Workspace
        ToggleHover(true)
        
        local function CustomLerp(tPos, currentSpeed)
            local totalDist = (root.Position - tPos).Magnitude; local t = totalDist / currentSpeed
            if t < 0.1 then return end
            local startPos = root.Position; local startTime = tick(); local lastDrop = tick() 
            char:SetAttribute("evading", true); _G.soruDashing = true
            
            while tick() - startTime < t do
                local alpha = (tick() - startTime) / t; local intermediatePos = startPos:Lerp(tPos, alpha)
                local lookPos = Vector3.new(tPos.X, intermediatePos.Y, tPos.Z)
                if (lookPos - intermediatePos).Magnitude < 0.1 then lookPos = intermediatePos + root.CFrame.LookVector end
                
                if (root.Position - intermediatePos).Magnitude > 20 or (tick() - lastDrop >= 2.5) then
                    local isDrop = (tick() - lastDrop >= 2.5)
                    ToggleHover(false); platform.CFrame = CFrame.new(0, 99999, 0) 
                    if hum then hum.Jump = true end; root.Velocity = Vector3.new(0, 50, 0); task.wait(isDrop and 0.7 or 1)
                    ToggleHover(true); startPos = root.Position; totalDist = (startPos - tPos).Magnitude; t = totalDist / currentSpeed; startTime = tick(); lastDrop = tick()
                else
                    local bp = root:FindFirstChild("RyuHover"); if bp then bp.Position = intermediatePos end
                    root.CFrame = CFrame.lookAt(intermediatePos, lookPos); root.Velocity = Vector3.new(0, 0, 0)
                    platform.CFrame = CFrame.new(intermediatePos.X, intermediatePos.Y - floorOffset, intermediatePos.Z)
                    char:SetAttribute("Grounded", true); _G.grounded = true
                end
                RunService.Heartbeat:Wait()
            end
            root.CFrame = CFrame.new(tPos); char:SetAttribute("evading", nil); _G.soruDashing = nil
        end
        
        if useSky then
            local safeY = 1500
            CustomLerp(Vector3.new(root.Position.X, safeY, root.Position.Z), RyuConfig.ElevatorSpeed)
            CustomLerp(Vector3.new(targetPos.X, safeY, targetPos.Z), RyuConfig.FishmanSpeed)
            CustomLerp(targetPos + Vector3.new(0, 50, 0), RyuConfig.ElevatorSpeed)
        else
            CustomLerp(targetPos + Vector3.new(0, 50, 0), RyuConfig.FishmanSpeed)
        end
        
        if hum then hum.Jump = true end; root.Velocity = Vector3.new(0, 60, 0); task.wait(0.2); ToggleHover(false)
        RyuNotify:Send("Smart TP", "Warte 1 Sekunde für Portal-TP...", 3); task.wait(1)
        
        local areaTp = Workspace:FindFirstChild("AreaTeleporters")
        if areaTp and areaTp:FindFirstChild("FirstSea") and areaTp.FirstSea:FindFirstChild("Fishman") and areaTp.FirstSea.Fishman:FindFirstChild("Part") then
            root.CFrame = areaTp.FirstSea.Fishman.Part.CFrame; RyuNotify:Send("Smart TP", "Teleportiert durchs Portal!", 2); task.wait(2); ToggleHover(true)
            RyuNotify:Send("Smart TP", "Navigiere durch Fishman Cave...", 3)
            local caveRoute = { Vector3.new(8004, -2154, -17130), Vector3.new(7960, -2154, -17156), Vector3.new(7862, -2154, -17159), Vector3.new(7775, -2177, -17174) }
            for _, wp in ipairs(caveRoute) do CustomLerp(wp, RyuConfig.FishmanSpeed) end
            RyuNotify:Send("Smart TP", "Route in Fishman Cave abgeschlossen!", 3)
        end
        platform:Destroy(); ToggleHover(false)
    end)
end

--// AUTO BOSS FARM MODULE
local BossFarmModule = { Enabled = false, FarmLoop = nil, CurrentTarget = nil, LastAttackTime = 0, Platform = nil, ActiveTween = nil, IsTweening = false, SearchAttempts = 0 }

function BossFarmModule:UpdatePlatform(rootPart)
    if not BossFarmModule.Platform or not BossFarmModule.Platform.Parent then
        local part = Instance.new("Part"); part.Name = "RyuAntiAirPlatform"; part.Size = Vector3.new(6, 1, 6); part.Anchored = true; part.CanCollide = true; part.Transparency = 1; part.Material = Enum.Material.SmoothPlastic; part.Parent = Workspace; BossFarmModule.Platform = part
    end
    BossFarmModule.Platform.CFrame = rootPart.CFrame * CFrame.new(0, -3.2, 0)
end

function BossFarmModule:RemovePlatform() if BossFarmModule.Platform then BossFarmModule.Platform:Destroy(); BossFarmModule.Platform = nil end end
function BossFarmModule:StopTween() if BossFarmModule.ActiveTween then BossFarmModule.ActiveTween:Cancel(); BossFarmModule.ActiveTween = nil end; BossFarmModule.IsTweening = false end

function BossFarmModule:Toggle(state)
    BossFarmModule.Enabled = state; RyuConfig.AutoBossFarm = state
    if not state then
        if BossFarmModule.FarmLoop then BossFarmModule.FarmLoop:Disconnect(); BossFarmModule.FarmLoop = nil end
        BossFarmModule:StopTween(); BossFarmModule:RemovePlatform()
        local char = LocalPlayer.Character; if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.Velocity = Vector3.zero end
        RyuNotify:Send("Boss Farm", "Auto Boss Farm stopped.", 2)
        return
    end
    
    if RyuConfig.AutoLawRaid then LawRaidModule:Toggle(false); if UIRegistry.Toggles["AutoLawRaid"] then UIRegistry.Toggles["AutoLawRaid"](false) end end
    if RyuConfig.AutoGPOFarm then GPOFarmModule:Toggle(false); if UIRegistry.Toggles["AutoGPOFarm"] then UIRegistry.Toggles["AutoGPOFarm"](false) end end
    
    RyuNotify:Send("Boss Farm", "Farm started (Target: " .. RyuConfig.BossName .. ")", 3)
    BossFarmModule.SearchAttempts = 0
    BossFarmModule.FarmLoop = RunService.Heartbeat:Connect(function()
        if not BossFarmModule.Enabled or ServerHopModule.IsHopping then return end
        local char = LocalPlayer.Character; if not char then return end
        local myRoot = char:FindFirstChild("HumanoidRootPart"); local myHum = char:FindFirstChildOfClass("Humanoid")
        if not myRoot or not myHum or myHum.Health <= 0 then BossFarmModule:StopTween(); BossFarmModule:RemovePlatform(); return end
        
        if not BossFarmModule.CurrentTarget or not BossFarmModule.CurrentTarget.Parent or not BossFarmModule.CurrentTarget:FindFirstChildOfClass("Humanoid") or BossFarmModule.CurrentTarget:FindFirstChildOfClass("Humanoid").Health <= 0 then
            BossFarmModule:StopTween()
            local npcsFolder = Workspace:FindFirstChild("NPCs") or Workspace; local nearestBoss = nil; local shortestDist = math.huge
            for _, model in pairs(npcsFolder:GetChildren()) do
                if model:IsA("Model") and (string.find(string.lower(model.Name), string.lower(RyuConfig.BossName)) or RyuConfig.BossName == "All") then
                    local hum = model:FindFirstChildOfClass("Humanoid"); local root = model:FindFirstChild("HumanoidRootPart")
                    if hum and hum.Health > 0 and root then
                        local dist = (myRoot.Position - root.Position).Magnitude
                        if dist < shortestDist then shortestDist = dist; nearestBoss = model end
                    end
                end
            end
            BossFarmModule.CurrentTarget = nearestBoss
            
            if not BossFarmModule.CurrentTarget then
                BossFarmModule.SearchAttempts = BossFarmModule.SearchAttempts + 1
                if BossFarmModule.SearchAttempts > 120 and RyuConfig.AutoServerHop then
                    RyuNotify:Send("Boss Farm", "Boss not found. Hopping server...", 3); BossFarmModule.Enabled = false
                    if UIRegistry.Toggles["AutoBossFarm"] then UIRegistry.Toggles["AutoBossFarm"](false) end
                    ServerHopModule:Hop(); return
                end
            else BossFarmModule.SearchAttempts = 0 end
        end
        
        if BossFarmModule.CurrentTarget then
            local bossRoot = BossFarmModule.CurrentTarget:FindFirstChild("HumanoidRootPart")
            if bossRoot then
                local hoverPosition = bossRoot.Position + Vector3.new(0, RyuConfig.BossHoverDistance, 0)
                local targetCFrame = CFrame.new(hoverPosition, bossRoot.Position)
                local dist = (myRoot.Position - hoverPosition).Magnitude
                if dist > 5 then
                    if not BossFarmModule.IsTweening then
                        BossFarmModule.IsTweening = true
                        local duration = dist / (RyuConfig.TweenSpeed or 45)
                        local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
                        BossFarmModule.ActiveTween = TweenService:Create(myRoot, tweenInfo, {CFrame = targetCFrame}); BossFarmModule.ActiveTween:Play()
                        BossFarmModule.ActiveTween.Completed:Connect(function() BossFarmModule.IsTweening = false end)
                    end
                else
                    if BossFarmModule.IsTweening then BossFarmModule:StopTween() end
                    myRoot.CFrame = targetCFrame; myRoot.Velocity = Vector3.zero; BossFarmModule:UpdatePlatform(myRoot)
                    if tick() - BossFarmModule.LastAttackTime >= RyuConfig.SkillCooldown then
                        BossFarmModule.LastAttackTime = tick()
                        local targetKeyEnum = Enum.KeyCode[RyuConfig.FarmKeybind] or Enum.KeyCode.X
                        VirtualInputManager:SendKeyEvent(true, targetKeyEnum, false, game)
                        task.delay(0.05, function() VirtualInputManager:SendKeyEvent(false, targetKeyEnum, false, game) end)
                    end
                end
            end
        else BossFarmModule:StopTween(); BossFarmModule:RemovePlatform() end
    end)
end

--// LAW RAID MODULE (PRIORITY TARGETER)
local LawRaidModule = { Enabled = false, RaidLoop = nil, CurrentTarget = nil, LastAttackTime = 0, Platform = nil, ActiveTween = nil, IsTweening = false }

function LawRaidModule:UpdatePlatform(rootPart)
    if not LawRaidModule.Platform or not LawRaidModule.Platform.Parent then
        local part = Instance.new("Part"); part.Name = "RyuAntiAirPlatformRaid"; part.Size = Vector3.new(6, 1, 6); part.Anchored = true; part.CanCollide = true; part.Transparency = 1; part.Material = Enum.Material.SmoothPlastic; part.Parent = Workspace; LawRaidModule.Platform = part
    end
    LawRaidModule.Platform.CFrame = rootPart.CFrame * CFrame.new(0, -3.2, 0)
end
function LawRaidModule:RemovePlatform() if LawRaidModule.Platform then LawRaidModule.Platform:Destroy(); LawRaidModule.Platform = nil end end
function LawRaidModule:StopTween() if LawRaidModule.ActiveTween then LawRaidModule.ActiveTween:Cancel(); LawRaidModule.ActiveTween = nil end; LawRaidModule.IsTweening = false end

local function GetTargetPos(targetObj)
    if targetObj:IsA("Model") then
        local part = targetObj:FindFirstChild("HumanoidRootPart") or targetObj.PrimaryPart or targetObj:FindFirstChildWhichIsA("BasePart")
        if part then return part.Position end
    elseif targetObj:IsA("BasePart") then return targetObj.Position end
    return nil
end

function LawRaidModule:GetRaidTarget()
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end

    local function findClosest(nameFilter)
        local closest = nil; local shortest = math.huge
        local npcs = Workspace:FindFirstChild("NPCs")
        if npcs then
            for _, obj in pairs(npcs:GetChildren()) do
                if string.find(string.lower(obj.Name), string.lower(nameFilter)) then
                    local hum = obj:FindFirstChildOfClass("Humanoid")
                    if not hum or hum.Health > 0 then
                        local pos = GetTargetPos(obj)
                        if pos then
                            local dist = (myRoot.Position - pos).Magnitude
                            if dist < shortest then shortest = dist; closest = obj end
                        end
                    end
                end
            end
        end
        if not closest and nameFilter == "slime core" then
            for _, obj in pairs(Workspace:GetChildren()) do
                if string.find(string.lower(obj.Name), string.lower(nameFilter)) then
                    local hum = obj:FindFirstChildOfClass("Humanoid")
                    if not hum or hum.Health > 0 then
                        local pos = GetTargetPos(obj)
                        if pos then
                            local dist = (myRoot.Position - pos).Magnitude
                            if dist < shortest then shortest = dist; closest = obj end
                        end
                    end
                end
            end
        end
        return closest
    end

    local target = findClosest("Scientist")
    if target then return target end
    
    target = findClosest("Law")
    if target then return target end
    
    target = findClosest("slime core")
    if target then return target end

    return nil
end

function LawRaidModule:Toggle(state)
    LawRaidModule.Enabled = state; RyuConfig.AutoLawRaid = state
    if not state then
        if LawRaidModule.RaidLoop then LawRaidModule.RaidLoop:Disconnect(); LawRaidModule.RaidLoop = nil end
        LawRaidModule:StopTween(); LawRaidModule:RemovePlatform()
        local char = LocalPlayer.Character; if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.Velocity = Vector3.zero end
        RyuNotify:Send("Law Raid", "Raid Automation stopped.", 2)
        return
    end
    
    if RyuConfig.AutoBossFarm then BossFarmModule:Toggle(false); if UIRegistry.Toggles["AutoBossFarm"] then UIRegistry.Toggles["AutoBossFarm"](false) end end
    if RyuConfig.AutoGPOFarm then GPOFarmModule:Toggle(false); if UIRegistry.Toggles["AutoGPOFarm"] then UIRegistry.Toggles["AutoGPOFarm"](false) end end
    
    RyuNotify:Send("Law Raid", "Raid Automation started! Scanning targets...", 3)
    LawRaidModule.RaidLoop = RunService.Heartbeat:Connect(function()
        if not LawRaidModule.Enabled then return end
        local char = LocalPlayer.Character; if not char then return end
        local myRoot = char:FindFirstChild("HumanoidRootPart"); local myHum = char:FindFirstChildOfClass("Humanoid")
        if not myRoot or not myHum or myHum.Health <= 0 then LawRaidModule:StopTween(); LawRaidModule:RemovePlatform(); return end
        
        local targetInvalid = false
        if not LawRaidModule.CurrentTarget or not LawRaidModule.CurrentTarget.Parent then targetInvalid = true
        else
            local hum = LawRaidModule.CurrentTarget:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health <= 0 then targetInvalid = true end
        end

        if targetInvalid then
            LawRaidModule:StopTween()
            LawRaidModule.CurrentTarget = LawRaidModule:GetRaidTarget()
        end
        
        if LawRaidModule.CurrentTarget then
            local targetPos = GetTargetPos(LawRaidModule.CurrentTarget)
            if targetPos then
                local hoverPosition = targetPos + Vector3.new(0, RyuConfig.BossHoverDistance, 0)
                local targetCFrame = CFrame.new(hoverPosition, targetPos)
                local dist = (myRoot.Position - hoverPosition).Magnitude
                
                if dist > 5 then
                    if not LawRaidModule.IsTweening then
                        LawRaidModule.IsTweening = true
                        local duration = dist / (RyuConfig.TweenSpeed or 45)
                        local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
                        LawRaidModule.ActiveTween = TweenService:Create(myRoot, tweenInfo, {CFrame = targetCFrame}); LawRaidModule.ActiveTween:Play()
                        LawRaidModule.ActiveTween.Completed:Connect(function() LawRaidModule.IsTweening = false end)
                    end
                else
                    if LawRaidModule.IsTweening then LawRaidModule:StopTween() end
                    myRoot.CFrame = targetCFrame; myRoot.Velocity = Vector3.zero; LawRaidModule:UpdatePlatform(myRoot)
                    if tick() - LawRaidModule.LastAttackTime >= RyuConfig.SkillCooldown then
                        LawRaidModule.LastAttackTime = tick()
                        local targetKeyEnum = Enum.KeyCode[RyuConfig.FarmKeybind] or Enum.KeyCode.X
                        VirtualInputManager:SendKeyEvent(true, targetKeyEnum, false, game)
                        task.delay(0.05, function() VirtualInputManager:SendKeyEvent(false, targetKeyEnum, false, game) end)
                    end
                end
            end
        else LawRaidModule:StopTween(); LawRaidModule:RemovePlatform() end
    end)
end

--// 6. CONFIG SAVE/LOAD MANAGER
local function GetConfigFilename() return "RyuHub_Slot" .. tostring(RyuConfig.SelectedSlot) .. ".json" end

local function SaveConfig()
    if writefile then
        local success, json = pcall(function() return HttpService:JSONEncode(RyuConfig) end)
        if success then writefile(GetConfigFilename(), json); MajesticNotify("Config Saved", "Successfully saved to Slot " .. tostring(RyuConfig.SelectedSlot))
        else RyuNotify:Send("Error", "Failed to encode settings.", 3) end
    else RyuNotify:Send("Error", "Your executor does not support saving files.", 3) end
end

local function LoadConfig()
    local filename = GetConfigFilename()
    if readfile and isfile and isfile(filename) then
        local success, data = pcall(function() return HttpService:JSONDecode(readfile(filename)) end)
        if success and type(data) == "table" then
            local currentSlot = RyuConfig.SelectedSlot 
            for k, v in pairs(data) do
                RyuConfig[k] = v
                if UIRegistry.Toggles[k] then UIRegistry.Toggles[k](v) end
                if UIRegistry.Sliders[k] then UIRegistry.Sliders[k](v) end
                if UIRegistry.TextBoxes[k] then UIRegistry.TextBoxes[k](v) end
            end
            RyuConfig.SelectedSlot = currentSlot
            
            PlayerMods:SetSpeed(RyuConfig.SpeedHack); PlayerMods:SetJumpPower(RyuConfig.HighJump); PlayerMods:SetGravity(RyuConfig.LowGravity); PlayerMods:SetFOV(RyuConfig.FOVChanger); PlayerMods:SetNoStun(RyuConfig.NoStun)
            FlyModule:Toggle(RyuConfig.Fly); ESPModule:Toggle(RyuConfig.ESP); ChestESPModule:Toggle(RyuConfig.ChestESP)
            BossFarmModule:Toggle(RyuConfig.AutoBossFarm); LawRaidModule:Toggle(RyuConfig.AutoLawRaid); GPOFarmModule:Toggle(RyuConfig.AutoGPOFarm)
            MajesticNotify("Config Loaded", "Loaded profile from Slot " .. tostring(RyuConfig.SelectedSlot))
        else RyuNotify:Send("Error", "Failed to read config file.", 3) end
    else RyuNotify:Send("Notice", "No saved configuration found for Slot " .. tostring(RyuConfig.SelectedSlot), 3) end
end

--// 7. UI GENERATION
local MainSize = UDim2.new(0, math.min(750, camera.ViewportSize.X - 40), 0, math.min(480, camera.ViewportSize.Y - 40))
local SidebarWidth = 160

local function AddHoverEffect(element, def, hov) element.MouseEnter:Connect(function() TweenService:Create(element, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = hov}):Play() end); element.MouseLeave:Connect(function() TweenService:Create(element, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = def}):Play() end) end
local function AddClickPop(element) local orig = element.Size; element.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then TweenService:Create(element, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(orig.X.Scale, orig.X.Offset-4, orig.Y.Scale, orig.Y.Offset-4)}):Play() end end); element.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then TweenService:Create(element, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Size = orig}):Play() end end) end

local ToggleBtn = Instance.new("TextButton"); ToggleBtn.Size = UDim2.new(0, 50, 0, 50); ToggleBtn.Position = UDim2.new(0, 25, 0, 25); ToggleBtn.BackgroundColor3 = Theme.Sidebar; ToggleBtn.Text = ""; ToggleBtn.Parent = RyuHub; Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local btnStroke = Instance.new("UIStroke", ToggleBtn); btnStroke.Color = Theme.Accent; btnStroke.Thickness = 1.5; btnStroke.Transparency = 0.5
local Katana = Instance.new("Frame", ToggleBtn); Katana.Size = UDim2.new(1, 0, 1, 0); Katana.BackgroundTransparency = 1; Katana.Rotation = 45
local Blade = Instance.new("Frame", Katana); Blade.Size = UDim2.new(0, 2, 0, 24); Blade.Position = UDim2.new(0.5, -1, 0.5, -18); Blade.BackgroundColor3 = Theme.Accent; Blade.BorderSizePixel = 0
local Guard = Instance.new("Frame", Katana); Guard.Size = UDim2.new(0, 14, 0, 3); Guard.Position = UDim2.new(0.5, -7, 0.5, 6); Guard.BackgroundColor3 = Theme.Background; Guard.BorderSizePixel = 0
local Handle = Instance.new("Frame", Katana); Handle.Size = UDim2.new(0, 4, 0, 10); Handle.Position = UDim2.new(0.5, -2, 0.5, 8); Handle.BackgroundColor3 = Theme.Accent; Handle.BorderSizePixel = 0
Instance.new("UICorner", Blade).CornerRadius = UDim.new(1, 0); Instance.new("UICorner", Guard).CornerRadius = UDim.new(1, 0); Instance.new("UICorner", Handle).CornerRadius = UDim.new(0, 1)

AddClickPop(ToggleBtn); local tDragStart, tStartPos, isDraggingBtn = nil, nil, false
ToggleBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then isDraggingBtn = false; tDragStart = input.Position; tStartPos = ToggleBtn.Position end end)
UserInputService.InputChanged:Connect(function(input) if tDragStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - tDragStart; if delta.Magnitude > 5 then isDraggingBtn = true; ToggleBtn.Position = UDim2.new(tStartPos.X.Scale, tStartPos.X.Offset + delta.X, tStartPos.Y.Scale, tStartPos.Y.Offset + delta.Y) end end end)

local MainFrame = Instance.new("Frame"); MainFrame.Size = UDim2.new(0, 0, 0, 0); MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.BorderSizePixel = 0; MainFrame.Active = true; MainFrame.Visible = false; MainFrame.ClipsDescendants = true; MainFrame.Parent = RyuHub
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10); local mainStroke = Instance.new("UIStroke", MainFrame); mainStroke.Color = Theme.Stroke; mainStroke.Transparency = 0; mainStroke.Thickness = 1

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if tDragStart then
            if not isDraggingBtn then
                if MainFrame.Visible then TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play(); task.wait(0.3); MainFrame.Visible = false
                else MainFrame.Visible = true; TweenService:Create(MainFrame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = MainSize, Position = UDim2.new(0.5, -MainSize.X.Offset/2, 0.5, -MainSize.Y.Offset/2)}):Play() end
            end; tDragStart = nil
        end
    end
end)

local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextXAlignment = Enum.TextXAlignment.Left
local TitleGradient = Instance.new("UIGradient", Title); TitleGradient.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Theme.Accent), ColorSequenceKeypoint.new(0.5, Color3.fromRGB(150, 150, 150)), ColorSequenceKeypoint.new(1, Theme.Accent)}; TitleGradient.Offset = Vector2.new(-1, 0)
task.spawn(function() TweenService:Create(TitleGradient, TweenInfo.new(3.0, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Offset = Vector2.new(1, 0)}):Play() end)
local SubTitle = Instance.new("TextLabel", Topbar); SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 38); SubTitle.BackgroundTransparency = 1; SubTitle.Text = "Monochrome Master Edition"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 11; SubTitle.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", Topbar); CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG; CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.Text; CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 14; Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", CloseBtn).Color = Theme.Stroke; AddHoverEffect(CloseBtn, Theme.SectionBG, Theme.Warning)
CloseBtn.MouseButton1Click:Connect(function() TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play(); task.wait(0.3); MainFrame.Visible = false end)

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end end)
Topbar.InputChanged:Connect(function(input) if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - mDragStart; MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y) end end)
Topbar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end end)

local Line = Instance.new("Frame", MainFrame); Line.Size = UDim2.new(1, -40, 0, 1); Line.Position = UDim2.new(0, 20, 0, 65); Line.BackgroundColor3 = Theme.Stroke; Line.BorderSizePixel = 0
local Sidebar = Instance.new("ScrollingFrame", MainFrame); Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar); SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; SideLayout.SortOrder = Enum.SortOrder.LayoutOrder
local ContentContainer = Instance.new("Frame", MainFrame); ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1

local Tabs = {}; local sidebarOrderCounter = 0; local itemOrderCounter = 0
local function UpdateSidebarCanvas() local totalH = 10; for _, t in pairs(Tabs) do totalH = totalH + 36 + 6; if t.IsOpen then totalH = totalH + t.SubLayout.AbsoluteContentSize.Y + 6 end end; Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH) end

local function CreateMainTab(name)
    local tabObj = { Btn = nil, Arrow = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {} }; sidebarOrderCounter = sidebarOrderCounter + 1
    local tabBtn = Instance.new("TextButton", Sidebar); tabBtn.LayoutOrder = sidebarOrderCounter; tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name); tabBtn.TextColor3 = Theme.Text; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 12; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 6); tabObj.Btn = tabBtn
    local arrow = Instance.new("TextLabel", tabBtn); arrow.Size = UDim2.new(0, 20, 1, 0); arrow.Position = UDim2.new(1, -25, 0, 0); arrow.BackgroundTransparency = 1; arrow.Text = "v"; arrow.TextColor3 = Theme.Text; arrow.Font = Enum.Font.GothamBold; arrow.TextSize = 12; tabObj.Arrow = arrow
    sidebarOrderCounter = sidebarOrderCounter + 1
    local subContainer = Instance.new("Frame", Sidebar); subContainer.LayoutOrder = sidebarOrderCounter; subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer); subLayout.Padding = UDim.new(0, 2); subLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; subLayout.SortOrder = Enum.SortOrder.LayoutOrder; tabObj.SubLayout = subLayout
    tabBtn.MouseButton1Click:Connect(function()
        tabObj.IsOpen = not tabObj.IsOpen; local targetSize = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)
        TweenService:Create(subContainer, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = targetSize}):Play()
        if tabObj.IsOpen then arrow.Text = "^"; TweenService:Create(tabBtn, TweenInfo.new(0.25), {BackgroundColor3 = Theme.SectionBG}):Play() else arrow.Text = "v"; TweenService:Create(tabBtn, TweenInfo.new(0.25), {BackgroundColor3 = Theme.Sidebar}):Play() end
        task.delay(0.26, UpdateSidebarCanvas); UpdateSidebarCanvas()
    end)
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end; UpdateSidebarCanvas() end)
    table.insert(Tabs, tabObj); return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subObj = { Btn = nil, Page = nil, Indicator = nil }
    local subBtn = Instance.new("TextButton", tabObj.SubContainer); subBtn.LayoutOrder = #tabObj.SubTabs + 1; subBtn.Size = UDim2.new(1, 0, 0, 28); subBtn.BackgroundTransparency = 1; subBtn.Text = "     " .. subName; subBtn.TextColor3 = Theme.SubText; subBtn.Font = Enum.Font.GothamMedium; subBtn.TextSize = 11; subBtn.TextXAlignment = Enum.TextXAlignment.Left; subObj.Btn = subBtn
    local indicator = Instance.new("Frame", subBtn); indicator.Size = UDim2.new(0, 16, 0, 2); indicator.Position = UDim2.new(0, 20, 1, -4); indicator.BackgroundColor3 = Theme.Accent; indicator.BorderSizePixel = 0; indicator.BackgroundTransparency = 1; Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0); subObj.Indicator = indicator
    local page = Instance.new("ScrollingFrame", ContentContainer); page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.ScrollBarThickness = 2; page.ScrollBarImageColor3 = Theme.Accent; page.Visible = false; subObj.Page = page
    local pageLayout = Instance.new("UIListLayout", page); pageLayout.Padding = UDim.new(0, 12); pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20) end)
    subBtn.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do for _, st in pairs(t.SubTabs) do st.Page.Visible = false; TweenService:Create(st.Indicator, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play() end end
        page.Visible = true; TweenService:Create(indicator, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end)
    table.insert(tabObj.SubTabs, subObj); return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page); section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; section.BackgroundTransparency = 0; Instance.new("UICorner", section).CornerRadius = UDim.new(0, 8); local sStroke = Instance.new("UIStroke", section); sStroke.Color = Theme.Stroke; sStroke.Transparency = 0
    local secLayout = Instance.new("UIListLayout", section); secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 12); Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 12)
    local title = Instance.new("TextLabel", section); title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText; title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 13; title.TextXAlignment = Enum.TextXAlignment.Left
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end); return section
end

local function CreateToggle(section, text, descText, configKey, callback)
    itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section); frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, descText and 52 or 34); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(0.7, 0, 0, 34); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.Text; label.Font = Enum.Font.GothamMedium; label.TextSize = 12; label.TextXAlignment = Enum.TextXAlignment.Left
    if descText then
        local descLabel = Instance.new("TextLabel", frame); descLabel.Size = UDim2.new(0.7, 0, 0, 15); descLabel.Position = UDim2.new(0, 0, 0, 30); descLabel.BackgroundTransparency = 1; descLabel.Text = descText; descLabel.TextColor3 = Theme.SubText; descLabel.Font = Enum.Font.Gotham; descLabel.TextSize = 10; descLabel.TextXAlignment = Enum.TextXAlignment.Left
    end
    local tBtn = Instance.new("TextButton", frame); tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = Theme.ToggleOff; tBtn.Text = ""; Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local bStroke = Instance.new("UIStroke", tBtn); bStroke.Color = Theme.Stroke; bStroke.Transparency = 0; AddClickPop(tBtn)
    local circle = Instance.new("Frame", tBtn); circle.Size = UDim2.new(0, 16, 0, 16); circle.Position = UDim2.new(0, 3, 0.5, -8); circle.BackgroundColor3 = Theme.Background; Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    
    local isOn = RyuConfig[configKey] or false
    local function UpdateVisuals(state)
        isOn = state
        if state then
            TweenService:Create(tBtn, TweenInfo.new(0.25), {BackgroundColor3 = Theme.ToggleOn}):Play(); TweenService:Create(circle, TweenInfo.new(0.25), {Position = UDim2.new(1, -19, 0.5, -8), BackgroundColor3 = Theme.Background}):Play(); bStroke.Color = Theme.ToggleOn
        else
            TweenService:Create(tBtn, TweenInfo.new(0.25), {BackgroundColor3 = Theme.ToggleOff}):Play(); TweenService:Create(circle, TweenInfo.new(0.25), {Position = UDim2.new(0, 3, 0.5, -8), BackgroundColor3 = Color3.fromRGB(150, 150, 150)}):Play(); bStroke.Color = Theme.Stroke
        end
    end
    UpdateVisuals(isOn)
    UIRegistry.Toggles[configKey] = function(newState) UpdateVisuals(newState) end

    tBtn.MouseButton1Click:Connect(function() RyuConfig[configKey] = not isOn; UpdateVisuals(RyuConfig[configKey]); if callback then callback(RyuConfig[configKey]) end end)
end

local function CreateSlider(section, text, min, max, configKey, callback)
    itemOrderCounter = itemOrderCounter + 1; local default = RyuConfig[configKey] or min
    local frame = Instance.new("Frame", section); frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(1, 0, 0, 20); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.Text; label.Font = Enum.Font.GothamMedium; label.TextSize = 12; label.TextXAlignment = Enum.TextXAlignment.Left
    local valLabel = Instance.new("TextLabel", frame); valLabel.Size = UDim2.new(0, 40, 0, 20); valLabel.Position = UDim2.new(1, -40, 0, 0); valLabel.BackgroundTransparency = 1; valLabel.Text = tostring(default); valLabel.TextColor3 = Theme.Accent; valLabel.Font = Enum.Font.GothamBold; valLabel.TextSize = 12; valLabel.TextXAlignment = Enum.TextXAlignment.Right
    local sliderBg = Instance.new("Frame", frame); sliderBg.Size = UDim2.new(1, 0, 0, 4); sliderBg.Position = UDim2.new(0, 0, 0, 32); sliderBg.BackgroundColor3 = Theme.ToggleOff; Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0); local sliderStroke = Instance.new("UIStroke", sliderBg); sliderStroke.Color = Theme.Stroke; sliderStroke.Thickness = 1
    local sliderFill = Instance.new("Frame", sliderBg); local percentage = (default - min) / (max - min); sliderFill.Size = UDim2.new(percentage, 0, 1, 0); sliderFill.BackgroundColor3 = Theme.Accent; Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    local knob = Instance.new("TextButton", sliderFill); knob.Size = UDim2.new(0, 14, 0, 14); knob.Position = UDim2.new(1, -7, 0.5, -7); knob.BackgroundColor3 = Theme.Background; knob.Text = ""; Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    
    local function setSlider(value)
        local relative = math.clamp((value - min) / (max - min), 0, 1); valLabel.Text = tostring(value)
        TweenService:Create(sliderFill, TweenInfo.new(0.08, Enum.EasingStyle.Quad), {Size = UDim2.new(relative, 0, 1, 0)}):Play()
        if callback then callback(value) end
    end
    UIRegistry.Sliders[configKey] = function(newVal)
        local relative = math.clamp((newVal - min) / (max - min), 0, 1); valLabel.Text = tostring(newVal); TweenService:Create(sliderFill, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Size = UDim2.new(relative, 0, 1, 0)}):Play()
    end
    local dragging = false
    knob.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(1, -9, 0.5, -9)}):Play() end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 14, 0, 14), Position = UDim2.new(1, -7, 0.5, -7)}):Play() end end)
    UserInputService.InputChanged:Connect(function(input) if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local relative = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1); local newVal = math.floor(min + (max - min) * relative); RyuConfig[configKey] = newVal; setSlider(newVal) end end)
end

local function CreateTextBox(section, placeholder, configKey, callback)
    itemOrderCounter = itemOrderCounter + 1
    local box = Instance.new("TextBox", section); box.LayoutOrder = itemOrderCounter; box.Size = UDim2.new(0.92, 0, 0, 34); box.BackgroundColor3 = Theme.Sidebar; box.Text = RyuConfig[configKey] or ""; box.PlaceholderText = placeholder; box.TextColor3 = Theme.Text; box.Font = Enum.Font.GothamMedium; box.TextSize = 12; box.ClearTextOnFocus = false; box.ClipsDescendants = true; Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", box).Color = Theme.Stroke
    UIRegistry.TextBoxes[configKey] = function(newText) box.Text = newText end
    box.FocusLost:Connect(function() RyuConfig[configKey] = box.Text; if callback then callback(box.Text) end end)
    return box
end

local function CreateButton(section, text, color, callback)
    itemOrderCounter = itemOrderCounter + 1; local btn = Instance.new("TextButton", section); btn.LayoutOrder = itemOrderCounter; btn.Size = UDim2.new(0.92, 0, 0, 34); btn.BackgroundColor3 = color; btn.Text = text; btn.TextColor3 = (color == Theme.Accent) and Theme.Background or Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", btn).Color = Theme.Stroke; Instance.new("UIStroke", btn).Transparency = 0; AddClickPop(btn); btn.MouseButton1Click:Connect(callback); return btn
end

--// ============================================================================
--// 8. POPULATING TABS
--// ============================================================================

-- MAIN TAB 1: BATTLE ROYALE
local TabBattleRoyale = CreateMainTab("Battle Royale")
local SubCharacter = CreateSubTab(TabBattleRoyale, "Character Mods")

local SecMovement = CreateSection(SubCharacter, "Movement & Flight")
CreateToggle(SecMovement, "Speed Modifier", "Permanently increases your walk speed", "SpeedHack", function(state) PlayerMods:SetSpeed(state) end)
CreateSlider(SecMovement, "Walkspeed Value", 16, 150, "SpeedValue", nil)
CreateToggle(SecMovement, "Jump Modifier", "Ultimate Velocity Hack (Bypasses anti-cheat)", "HighJump", function(state) PlayerMods:SetJumpPower(state) end)
CreateSlider(SecMovement, "Jump Power Value", 50, 200, "JumpValue", nil)
CreateToggle(SecMovement, "Fly Mode", "Free flight (WASD + Space/Shift)", "Fly", function(state) FlyModule:Toggle(state) end)
CreateTextBox(SecMovement, "Fly Toggle Keybind (Default: LeftControl)", "FlyKeybind", function(text)
    local success = pcall(function() return Enum.KeyCode[text] end)
    if success and text ~= "" then RyuConfig.FlyKeybind = text; RyuNotify:Send("Keybind", "Fly Keybind set to [" .. text .. "]", 2)
    else RyuNotify:Send("Error", "Invalid keybind! Reverting to (LeftControl).", 3); RyuConfig.FlyKeybind = "LeftControl"; if UIRegistry.TextBoxes["FlyKeybind"] then UIRegistry.TextBoxes["FlyKeybind"]("LeftControl") end end
end)
CreateSlider(SecMovement, "Flight Speed", 10, 200, "FlySpeed", nil)

local SecCombat = CreateSection(SubCharacter, "Combat & Defense")
CreateToggle(SecCombat, "Anti-Stun / No Ragdoll", "Aggressively prevents M1 hits and stuns", "NoStun", function(state) PlayerMods:SetNoStun(state) end)

local SecPhysics = CreateSection(SubCharacter, "Physics & Camera")
CreateToggle(SecPhysics, "Low Gravity", "Decreases workspace gravity for moon jumps", "LowGravity", function(state) PlayerMods:SetGravity(state) end)
CreateSlider(SecPhysics, "Gravity Value", 0, 196, "GravityValue", nil)
CreateToggle(SecPhysics, "Field of View", "Modifies your camera FOV", "FOVChanger", function(state) PlayerMods:SetFOV(state) end)
CreateSlider(SecPhysics, "FOV Value", 70, 120, "FOVValue", nil)

local SecVisuals = CreateSection(SubCharacter, "Player & Loot ESP")
CreateToggle(SecVisuals, "Player ESP", "Highlights enemies through walls", "ESP", function(state) ESPModule:Toggle(state) end)
CreateSlider(SecVisuals, "Player ESP Transparency %", 0, 90, "ESPTransparency", function(val) if RyuConfig.ESP then ESPModule:UpdateTransparency(val) end end)
CreateToggle(SecVisuals, "Multi-Rarity Chest ESP", "Highlights nearby loot based on rarity", "ChestESP", function(state) ChestESPModule:Toggle(state) end)
CreateSlider(SecVisuals, "Chest ESP Transparency %", 0, 90, "ChestESPTransparency", function(val) if RyuConfig.ChestESP then ChestESPModule:UpdateTransparency(val) end end)

-- MAIN TAB 2: FARMING
local TabFarming = CreateMainTab("Automation")
local SubBossFarm = CreateSubTab(TabFarming, "Auto Boss Farm")
local SubLawRaid = CreateSubTab(TabFarming, "Law Raid Farm")
local SubGPOFarm = CreateSubTab(TabFarming, "GPO Fishman Farm")

-- BOSS FARM
local SecBossMain = CreateSection(SubBossFarm, "NPC Automation (Workspace.NPCs)")
CreateToggle(SecBossMain, "Enable Auto Boss Farm", "Tweens smoothly to the target and attacks", "AutoBossFarm", function(state) BossFarmModule:Toggle(state) end)
CreateTextBox(SecBossMain, "Target Name (Default: Juzo the Diamondback)", "BossName", function(text)
    if text == "" then RyuConfig.BossName = "All" end
    RyuNotify:Send("Automation", "Target updated to: " .. RyuConfig.BossName, 2)
end)

local SecServerHop = CreateSection(SubBossFarm, "Server Management")
CreateToggle(SecServerHop, "Auto Server Hop", "Switches server if the boss is dead", "AutoServerHop", nil)
CreateButton(SecServerHop, "Force Server Hop", Theme.Accent, function() ServerHopModule:Hop() end)

local SecBossSettings = CreateSection(SubBossFarm, "Tween & Attack Settings")
CreateTextBox(SecBossSettings, "Attack Keybind (e.g. X, E, Q, C)", "FarmKeybind", function(text)
    text = string.upper(text); local success = pcall(function() return Enum.KeyCode[text] end)
    if success and text ~= "" then RyuConfig.FarmKeybind = text; RyuNotify:Send("Automation", "Keybind set to [" .. text .. "]", 2)
    else RyuNotify:Send("Error", "Invalid keybind! Reverting to (X).", 3); RyuConfig.FarmKeybind = "X"; if UIRegistry.TextBoxes["FarmKeybind"] then UIRegistry.TextBoxes["FarmKeybind"]("X") end end
end)
CreateSlider(SecBossSettings, "Tween Speed (Studs/sec)", 10, 100, "TweenSpeed", nil)
CreateSlider(SecBossSettings, "Hover Distance (Studs)", 5, 30, "BossHoverDistance", nil)
CreateSlider(SecBossSettings, "Skill Cooldown (sec)", 0, 300, "SkillCooldown", nil)

-- LAW RAID
local SecLawMain = CreateSection(SubLawRaid, "Law Raid Progression")
CreateToggle(SecLawMain, "Enable Law Raid Farm", "Auto-targets Scientists -> Law -> Slime Core", "AutoLawRaid", function(state) LawRaidModule:Toggle(state) end)

-- GPO FISHMAN CAVE
local SecGPOFarmMain = CreateSection(SubGPOFarm, "GPO: Fishman Cave")
CreateToggle(SecGPOFarmMain, "Enable Cave Farm", "1-by-1 Combat. Safe for M1s.", "AutoGPOFarm", function(state) GPOFarmModule:Toggle(state) end)
CreateToggle(SecGPOFarmMain, "Auto Quest Link", "Automatically takes quest", "AutoQuest", nil)
CreateTextBox(SecGPOFarmMain, "Target Weapon (e.g. Combat)", "TargetWeapon", nil)
CreateTextBox(SecGPOFarmMain, "Target Mob (e.g. Fishman Karate User)", "TargetMob", nil)
CreateTextBox(SecGPOFarmMain, "Target Quest NPC (e.g. Becky)", "TargetNPC", nil)

local SecGPOMovement = CreateSection(SubGPOFarm, "Cave Movement & TP")
CreateSlider(SecGPOMovement, "Cave Travel Speed", 50, 200, "FishmanSpeed", nil)
CreateSlider(SecGPOMovement, "Elevator Speed (Y-Axis)", 10, 150, "ElevatorSpeed", nil)
CreateButton(SecGPOMovement, "Smart Sky-TP to Fishman Cave", Theme.Sidebar, function() GPOSmartTP(true) end)
CreateButton(SecGPOMovement, "Ground-TP to Fishman Cave", Theme.Sidebar, function() GPOSmartTP(false) end)


-- MAIN TAB 3: SETTINGS (3 SLOTS, SAVE & LOAD)
local TabSettings = CreateMainTab("Settings")
local SubConfig = CreateSubTab(TabSettings, "Configuration Slots")
local SecSaveLoad = CreateSection(SubConfig, "Profile Management (3 Slots)")

CreateSlider(SecSaveLoad, "Active Config Slot (1-3)", 1, 3, "SelectedSlot", function(val)
    RyuConfig.SelectedSlot = val
    RyuNotify:Send("Config Slot", "Switched to Slot " .. tostring(val), 2)
end)

CreateButton(SecSaveLoad, "Save Config to Current Slot", Theme.Accent, function() SaveConfig() end)
CreateButton(SecSaveLoad, "Load Config from Current Slot", Theme.Sidebar, function() LoadConfig() end)

--// 9. INITIALIZATION
task.wait(0.5)
RyuNotify:Send("RYU HUB", "Monochrome Master Edition loaded!", 4)
