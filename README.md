--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION
--// Platform: PC & Mobile
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() 
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end 
end)

for _, v in pairs(guiParent:GetChildren()) do 
    if v.Name == "RyuHubPremium" or v.Name == "RyuNotifications" then v:Destroy() end 
end

--// RYU CONFIGURATION (GLOBAL STATE)
local RyuConfig = {
    -- Player Mods
    SpeedHack = false, SpeedValue = 35, 
    HighJump = false, JumpValue = 50, 
    LowGravity = false, GravityValue = 100,
    FOVChanger = false, FOVValue = 90,
    ESP = false, ESPTransparency = 50,
    
    -- GPO Farm
    AutoFarm = false,
    AutoQuest = false,
    IsTweening = false, 
    TargetMob = "Shell's Bandit",
    TargetIsland = "Town of Beginnings",
    TargetNPC = "Ash the Tailor",
    TargetWeapon = "Melee",
    FarmPosition = "Behind (Safe)", 
    TPMethod = "Ground Tween",
    
    -- Advanced Bypass Settings
    TweenSpeed = 150,       
    TPDistance = 150,       
    FarmOffset = 5          
}

--// GPO DATA LISTS
local GPOIslands = {
    "Town of Beginnings", "Sandora", "Shell's Town", "Orange Town", 
    "Restaurant Baratie", "Roca Island", "Sphinx Island", "Marine Fort F-1", 
    "Fishman Island", "Colosseum", "Land of the Sky", "Marine Base G-1",
    "Logue Town", "Kori Island", "Island Of Zou", "Gravito's Fort",
    "Fishman Cave", "Coco Island", "A rock", "???? Shrine", "Mysterious Cliff",
    "Reverse Mountain", "Shark Park"
}

local GPOEnemies = {
    "Bandit", "Shell's Bandit", "Corrupt Marine", "Fishman", "Chess Soldier", 
    "Marine Scout", "Monkey", "Gorilla", "Yeti"
}

local GPONPCs = {
    "Ash the Tailor", "Tyson", "Robo", "Robert", "Kevin", "Helen", "Gozen", 
    "Axe Hand Logan", "Captain Zhen", "Pharaoh Akshan", "Moria"
}

local GPOWeapons = {
    "Melee", "Sword", "Katana", "Rifle", "Pistol"
}

local FarmPositions = {
    "Behind (Safe)", "Distance (Gun)", "Below (Underground)", "Above (Max 12)"
}

local TPMethods = {
    "Ground Tween", "Sky Tween"
}

--// TWEEN ENGINE 1: SMOOTH SKY TWEEN (Youtube Method)
local function SmoothTween(targetCFrame, speed)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    RyuConfig.IsTweening = true 
    
    local dist = (root.Position - targetCFrame.Position).Magnitude
    if dist <= RyuConfig.TPDistance then
        root.CFrame = targetCFrame
        RyuConfig.IsTweening = false
        return
    end

    local tweenInfo = TweenInfo.new(dist / speed, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(root, tweenInfo, {CFrame = targetCFrame})
    
    tween:Play()
    
    local checkLoop
    checkLoop = RunService.Heartbeat:Connect(function()
        if not root or not root.Parent then
            tween:Cancel()
            checkLoop:Disconnect()
            return
        end
        
        local currentDist = (root.Position - targetCFrame.Position).Magnitude
        if currentDist <= RyuConfig.TPDistance then
            tween:Cancel()
            root.CFrame = targetCFrame
            checkLoop:Disconnect()
        end
    end)
    
    tween.Completed:Wait()
    if checkLoop then checkLoop:Disconnect() end
    if root then root.CFrame = targetCFrame end
    RyuConfig.IsTweening = false 
end

local function SkyTween(targetCFrame)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    -- 1. Unauffällig in die Luft steigen
    local upCFrame = root.CFrame + Vector3.new(0, 350, 0)
    SmoothTween(upCFrame, RyuConfig.TweenSpeed)
    
    -- 2. Über die Zielinsel fliegen
    local overTargetCFrame = CFrame.new(targetCFrame.Position.X, upCFrame.Position.Y, targetCFrame.Position.Z)
    SmoothTween(overTargetCFrame, RyuConfig.TweenSpeed)
    
    -- 3. Zur Zielposition absinken (mit Auto-TP am Ende)
    SmoothTween(targetCFrame, RyuConfig.TweenSpeed)
end

--// TWEEN ENGINE 2: STRICT GROUND TWEEN (Anti-Cheat Safe)
local function SafeGroundTween(targetCFrame, speed)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    RyuConfig.IsTweening = true 
    
    local currentPos = root.Position
    local targetPos = targetCFrame.Position
    local dist = (currentPos - targetPos).Magnitude
    
    -- Berechne die Y-Achsen-Differenz (nur nach oben relevant für Kick)
    local yDist = math.max(0, targetPos.Y - currentPos.Y) 
    
    -- GPO +Y Axis Limit ist 17. Wir nutzen max 14 Studs/Sekunde nach oben, um 100% sicher zu sein.
    local timeHorizontal = dist / speed
    local timeVertical = yDist / 14 
    local finalTweenTime = math.max(timeHorizontal, timeVertical)
    
    if dist <= RyuConfig.TPDistance then
        root.CFrame = targetCFrame
        RyuConfig.IsTweening = false
        return
    end

    local tweenInfo = TweenInfo.new(finalTweenTime, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(root, tweenInfo, {CFrame = targetCFrame})
    
    tween:Play()
    
    local checkLoop
    checkLoop = RunService.Heartbeat:Connect(function()
        if not root or not root.Parent then
            tween:Cancel()
            checkLoop:Disconnect()
            return
        end
        
        local currentDist = (root.Position - targetCFrame.Position).Magnitude
        if currentDist <= RyuConfig.TPDistance then
            tween:Cancel()
            root.CFrame = targetCFrame
            checkLoop:Disconnect()
        end
    end)
    
    tween.Completed:Wait()
    if checkLoop then checkLoop:Disconnect() end
    
    if root then root.CFrame = targetCFrame end
    RyuConfig.IsTweening = false 
end

--// NOCLIP ENGINE (Für Durchqueren von Bergen/Wänden während des Tweens)
RunService.Stepped:Connect(function()
    if RyuConfig.AutoFarm or RyuConfig.AutoQuest or RyuConfig.IsTweening then
        local char = LocalPlayer.Character
        if char then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = false
                end
            end
        end
    end
end)

--// ANTI-CHEAT PLATFORM ENGINE (Velocity Reset & Fester Boden)
RunService.Heartbeat:Connect(function()
    if RyuConfig.AutoFarm or RyuConfig.AutoQuest or RyuConfig.IsTweening then
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            -- Tötet Momentum: Verhindert, dass das Anti-Cheat einen Fall/Flug berechnet
            root.Velocity = Vector3.new(0, 0, 0)
            
            local plat = Workspace:FindFirstChild("RyuSafePlatform")
            if not plat then
                plat = Instance.new("Part", Workspace)
                plat.Name = "RyuSafePlatform"
                plat.Size = Vector3.new(15, 2, 15)
                plat.Anchored = true
                plat.Transparency = 1
                plat.CanCollide = true 
            end
            plat.CFrame = root.CFrame * CFrame.new(0, -3.5, 0)
        end
    else
        local plat = Workspace:FindFirstChild("RyuSafePlatform")
        if plat then plat:Destroy() end
    end
end)

--// NOTIFICATION SYSTEM
local NotificationContainer = Instance.new("Frame")
NotificationContainer.Name = "RyuNotifications"
NotificationContainer.Size = UDim2.new(0, 260, 1, -40)
NotificationContainer.Position = UDim2.new(1, -280, 0, 20)
NotificationContainer.BackgroundTransparency = 1
NotificationContainer.Parent = guiParent

local NotifLayout = Instance.new("UIListLayout", NotificationContainer)
NotifLayout.SortOrder = Enum.SortOrder.LayoutOrder
NotifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
NotifLayout.Padding = UDim.new(0, 8)

local RyuNotify = {}
function RyuNotify:Send(title, text, duration)
    duration = duration or 3
    local NotifFrame = Instance.new("Frame", NotificationContainer)
    NotifFrame.Size = UDim2.new(1, 0, 0, 60); NotifFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 20); NotifFrame.BackgroundTransparency = 1; NotifFrame.BorderSizePixel = 0
    Instance.new("UICorner", NotifFrame).CornerRadius = UDim.new(0, 8)
    
    local Stroke = Instance.new("UIStroke", NotifFrame); Stroke.Color = Color3.fromRGB(255, 255, 255); Stroke.Transparency = 1; Stroke.Thickness = 1.5
    local AccentLine = Instance.new("Frame", NotifFrame); AccentLine.Size = UDim2.new(0, 3, 0.8, 0); AccentLine.Position = UDim2.new(0, 4, 0.1, 0); AccentLine.BackgroundColor3 = Color3.fromRGB(255, 255, 255); AccentLine.BackgroundTransparency = 1; Instance.new("UICorner", AccentLine).CornerRadius = UDim.new(1, 0)
    local TitleLabel = Instance.new("TextLabel", NotifFrame); TitleLabel.Size = UDim2.new(1, -20, 0, 20); TitleLabel.Position = UDim2.new(0, 15, 0, 8); TitleLabel.BackgroundTransparency = 1; TitleLabel.Text = title; TitleLabel.TextColor3 = Color3.fromRGB(250, 250, 250); TitleLabel.TextTransparency = 1; TitleLabel.Font = Enum.Font.GothamBold; TitleLabel.TextSize = 13; TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    local DescLabel = Instance.new("TextLabel", NotifFrame); DescLabel.Size = UDim2.new(1, -20, 0, 20); DescLabel.Position = UDim2.new(0, 15, 0, 28); DescLabel.BackgroundTransparency = 1; DescLabel.Text = text; DescLabel.TextColor3 = Color3.fromRGB(130, 130, 135); DescLabel.TextTransparency = 1; DescLabel.Font = Enum.Font.Gotham; DescLabel.TextSize = 11; DescLabel.TextXAlignment = Enum.TextXAlignment.Left

    TweenService:Create(NotifFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0.1}):Play()
    TweenService:Create(Stroke, TweenInfo.new(0.3), {Transparency = 0.5}):Play()
    TweenService:Create(AccentLine, TweenInfo.new(0.3), {BackgroundTransparency = 0}):Play()
    TweenService:Create(TitleLabel, TweenInfo.new(0.3), {TextTransparency = 0}):Play()
    TweenService:Create(DescLabel, TweenInfo.new(0.3), {TextTransparency = 0}):Play()

    task.spawn(function()
        task.wait(duration)
        local fadeOut = TweenService:Create(NotifFrame, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 0)})
        TweenService:Create(Stroke, TweenInfo.new(0.3), {Transparency = 1}):Play(); TweenService:Create(AccentLine, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play(); TweenService:Create(TitleLabel, TweenInfo.new(0.3), {TextTransparency = 1}):Play(); TweenService:Create(DescLabel, TweenInfo.new(0.3), {TextTransparency = 1}):Play()
        fadeOut:Play(); fadeOut.Completed:Wait(); NotifFrame:Destroy()
    end)
end

--// PLAYER MODS (ENFORCER & ESP)
local PlayerMods = { SpeedEnabled = false, JumpEnabled = false, GravityEnabled = false, FOVEnabled = false, EnforceLoop = nil }
function PlayerMods:StartEnforcing()
    if PlayerMods.EnforceLoop then return end
    PlayerMods.EnforceLoop = RunService.Heartbeat:Connect(function()
        local character = LocalPlayer.Character
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        if PlayerMods.SpeedEnabled and humanoid.WalkSpeed ~= RyuConfig.SpeedValue then humanoid.WalkSpeed = RyuConfig.SpeedValue end
        if PlayerMods.JumpEnabled then humanoid.UseJumpPower = true; if humanoid.JumpPower ~= RyuConfig.JumpValue then humanoid.JumpPower = RyuConfig.JumpValue end end
        if PlayerMods.GravityEnabled and Workspace.Gravity ~= RyuConfig.GravityValue then Workspace.Gravity = RyuConfig.GravityValue end
        if PlayerMods.FOVEnabled and camera.FieldOfView ~= RyuConfig.FOVValue then camera.FieldOfView = RyuConfig.FOVValue end
    end)
end

function PlayerMods:SetSpeed(v, enabled) PlayerMods.SpeedEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then LocalPlayer.Character.Humanoid.WalkSpeed = 16 end end
function PlayerMods:SetJumpPower(v, enabled) PlayerMods.JumpEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then LocalPlayer.Character.Humanoid.JumpPower = 50 end end
function PlayerMods:SetGravity(v, enabled) PlayerMods.GravityEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled then Workspace.Gravity = 196.2 end end
function PlayerMods:SetFOV(v, enabled) PlayerMods.FOVEnabled = enabled; PlayerMods:StartEnforcing(); if not enabled then camera.FieldOfView = 70 end end

--// PREMIUM MONOCHROME THEME & UI SETUP
local Theme = { Background = Color3.fromRGB(12, 12, 14), Sidebar = Color3.fromRGB(18, 18, 20), SectionBG = Color3.fromRGB(24, 24, 26), Text = Color3.fromRGB(250, 250, 250), SubText = Color3.fromRGB(130, 130, 135), CloudLight = Color3.fromRGB(255, 255, 255), CloudDark = Color3.fromRGB(60, 60, 65), Accent = Color3.fromRGB(255, 255, 255), ToggleOff = Color3.fromRGB(35, 35, 38), ToggleOn = Color3.fromRGB(255, 255, 255), Stroke = Color3.fromRGB(45, 45, 50), Warning = Color3.fromRGB(255, 75, 75) }
local MainSize = UDim2.new(0, math.min(750, camera.ViewportSize.X - 40), 0, math.min(480, camera.ViewportSize.Y - 40))
local SidebarWidth = 160

local RyuHub = Instance.new("ScreenGui"); RyuHub.Name = "RyuHubPremium"; RyuHub.ResetOnSpawn = false; RyuHub.IgnoreGuiInset = true; RyuHub.Parent = guiParent

local function AddClickPop(element)
    local orig = element.Size
    element.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then TweenService:Create(element, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(orig.X.Scale, orig.X.Offset-4, orig.Y.Scale, orig.Y.Offset-4)}):Play() end end)
    element.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then TweenService:Create(element, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Size = orig}):Play() end end)
end

local ToggleBtn = Instance.new("TextButton"); ToggleBtn.Size = UDim2.new(0, 50, 0, 50); ToggleBtn.Position = UDim2.new(0, 25, 0, 25); ToggleBtn.BackgroundColor3 = Theme.Sidebar; ToggleBtn.Text = ""; ToggleBtn.Parent = RyuHub; Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local btnStroke = Instance.new("UIStroke", ToggleBtn); btnStroke.Color = Theme.Accent; btnStroke.Thickness = 2; btnStroke.Transparency = 0.5
local Katana = Instance.new("Frame", ToggleBtn); Katana.Size = UDim2.new(1, 0, 1, 0); Katana.BackgroundTransparency = 1; Katana.Rotation = 45
local Blade = Instance.new("Frame", Katana); Blade.Size = UDim2.new(0, 2, 0, 24); Blade.Position = UDim2.new(0.5, -1, 0.5, -18); Blade.BackgroundColor3 = Theme.CloudLight; Blade.BorderSizePixel = 0
local Guard = Instance.new("Frame", Katana); Guard.Size = UDim2.new(0, 12, 0, 2); Guard.Position = UDim2.new(0.5, -6, 0.5, 6); Guard.BackgroundColor3 = Theme.CloudDark; Guard.BorderSizePixel = 0
local Handle = Instance.new("Frame", Katana); Handle.Size = UDim2.new(0, 4, 0, 10); Handle.Position = UDim2.new(0.5, -2, 0.5, 8); Handle.BackgroundColor3 = Color3.fromRGB(40, 45, 50); Handle.BorderSizePixel = 0
Instance.new("UICorner", Blade).CornerRadius = UDim.new(1, 0); Instance.new("UICorner", Guard).CornerRadius = UDim.new(1, 0); Instance.new("UICorner", Handle).CornerRadius = UDim.new(0, 1)
AddClickPop(ToggleBtn)

local tDragStart, tStartPos, isDraggingBtn = nil, nil, false
ToggleBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then isDraggingBtn = false; tDragStart = input.Position; tStartPos = ToggleBtn.Position end end)
UserInputService.InputChanged:Connect(function(input) if tDragStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - tDragStart; if delta.Magnitude > 5 then isDraggingBtn = true; ToggleBtn.Position = UDim2.new(tStartPos.X.Scale, tStartPos.X.Offset + delta.X, tStartPos.Y.Scale, tStartPos.Y.Offset + delta.Y) end end end)

local MainFrame = Instance.new("Frame"); MainFrame.Size = UDim2.new(0, 0, 0, 0); MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.BorderSizePixel = 0; MainFrame.Active = true; MainFrame.Visible = false; MainFrame.ClipsDescendants = true; MainFrame.Parent = RyuHub; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local mainStroke = Instance.new("UIStroke", MainFrame); mainStroke.Color = Theme.Stroke; mainStroke.Transparency = 0.2; mainStroke.Thickness = 1.5

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if tDragStart then
            if not isDraggingBtn then
                if MainFrame.Visible then TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play(); task.wait(0.25); MainFrame.Visible = false
                else MainFrame.Visible = true; TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = MainSize, Position = UDim2.new(0.5, -MainSize.X.Offset/2, 0.5, -MainSize.Y.Offset/2)}):Play() end
            end
            tDragStart = nil
        end
    end
end)

local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextXAlignment = Enum.TextXAlignment.Left
local SubTitle = Instance.new("TextLabel", Topbar); SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 38); SubTitle.BackgroundTransparency = 1; SubTitle.Text = "Battle Royale & GPO Edition"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 11; SubTitle.TextXAlignment = Enum.TextXAlignment.Left
local CloseBtn = Instance.new("TextButton", Topbar); CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG; CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.SubText; CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 14; Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", CloseBtn).Color = Theme.Stroke
CloseBtn.Activated:Connect(function() TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play(); task.wait(0.25); MainFrame.Visible = false end)

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end end)
Topbar.InputChanged:Connect(function(input) if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - mDragStart; MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y) end end)
Topbar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end end)

local Line = Instance.new("Frame", MainFrame); Line.Size = UDim2.new(1, -40, 0, 1); Line.Position = UDim2.new(0, 20, 0, 65); Line.BackgroundColor3 = Theme.Stroke; Line.BorderSizePixel = 0
local Sidebar = Instance.new("ScrollingFrame", MainFrame); Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar); SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; SideLayout.SortOrder = Enum.SortOrder.LayoutOrder
local ContentContainer = Instance.new("Frame", MainFrame); ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1

local Tabs = {}
local sidebarOrderCounter = 0
local itemOrderCounter = 0

local function UpdateSidebarCanvas()
    local totalH = 10
    for _, t in pairs(Tabs) do totalH = totalH + 36 + 6; if t.IsOpen then totalH = totalH + t.SubLayout.AbsoluteContentSize.Y + 6 end end
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH)
end

local function CreateMainTab(name)
    local tabObj = { Btn = nil, Arrow = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {} }
    sidebarOrderCounter = sidebarOrderCounter + 1
    local tabBtn = Instance.new("TextButton", Sidebar); tabBtn.LayoutOrder = sidebarOrderCounter; tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name); tabBtn.TextColor3 = Theme.SubText; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 13; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8); tabObj.Btn = tabBtn
    local arrow = Instance.new("TextLabel", tabBtn); arrow.Size = UDim2.new(0, 20, 1, 0); arrow.Position = UDim2.new(1, -25, 0, 0); arrow.BackgroundTransparency = 1; arrow.Text = "v"; arrow.TextColor3 = Theme.SubText; arrow.Font = Enum.Font.GothamBold; arrow.TextSize = 12; tabObj.Arrow = arrow
    
    sidebarOrderCounter = sidebarOrderCounter + 1
    local subContainer = Instance.new("Frame", Sidebar); subContainer.LayoutOrder = sidebarOrderCounter; subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer); subLayout.Padding = UDim.new(0, 2); subLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; subLayout.SortOrder = Enum.SortOrder.LayoutOrder; tabObj.SubLayout = subLayout

    tabBtn.Activated:Connect(function()
        tabObj.IsOpen = not tabObj.IsOpen
        local targetSize = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)
        TweenService:Create(subContainer, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = targetSize}):Play()
        if tabObj.IsOpen then arrow.Text = "^"; TweenService:Create(tabBtn, TweenInfo.new(0.25), {TextColor3 = Theme.Text, BackgroundColor3 = Theme.SectionBG}):Play(); TweenService:Create(arrow, TweenInfo.new(0.25), {TextColor3 = Theme.Text}):Play()
        else arrow.Text = "v"; TweenService:Create(tabBtn, TweenInfo.new(0.25), {TextColor3 = Theme.SubText, BackgroundColor3 = Theme.Sidebar}):Play(); TweenService:Create(arrow, TweenInfo.new(0.25), {TextColor3 = Theme.SubText}):Play() end
        task.delay(0.26, UpdateSidebarCanvas); UpdateSidebarCanvas()
    end)
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end; UpdateSidebarCanvas() end)
    table.insert(Tabs, tabObj)
    return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subObj = { Btn = nil, Page = nil, Indicator = nil }
    local subBtn = Instance.new("TextButton", tabObj.SubContainer); subBtn.LayoutOrder = #tabObj.SubTabs + 1; subBtn.Size = UDim2.new(1, 0, 0, 28); subBtn.BackgroundTransparency = 1; subBtn.Text = "     " .. subName; subBtn.TextColor3 = Theme.SubText; subBtn.Font = Enum.Font.GothamMedium; subBtn.TextSize = 12; subBtn.TextXAlignment = Enum.TextXAlignment.Left; subObj.Btn = subBtn
    local indicator = Instance.new("Frame", subBtn); indicator.Size = UDim2.new(0, 16, 0, 2); indicator.Position = UDim2.new(0, 20, 1, -4); indicator.BackgroundColor3 = Theme.Accent; indicator.BorderSizePixel = 0; indicator.BackgroundTransparency = 1; Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0); subObj.Indicator = indicator
    local page = Instance.new("ScrollingFrame", ContentContainer); page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.ScrollBarThickness = 2; page.ScrollBarImageColor3 = Theme.Accent; page.Visible = false; subObj.Page = page
    local pageLayout = Instance.new("UIListLayout", page); pageLayout.Padding = UDim.new(0, 12); pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20) end)

    subBtn.Activated:Connect(function()
        for _, t in pairs(Tabs) do for _, st in pairs(t.SubTabs) do st.Page.Visible = false; TweenService:Create(st.Btn, TweenInfo.new(0.2), {TextColor3 = Theme.SubText}):Play(); TweenService:Create(st.Indicator, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play() end end
        page.Visible = true; TweenService:Create(subBtn, TweenInfo.new(0.2), {TextColor3 = Theme.Text}):Play(); TweenService:Create(indicator, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end)
    table.insert(tabObj.SubTabs, subObj)
    return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page); section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; section.BackgroundTransparency = 0; Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    local sStroke = Instance.new("UIStroke", section); sStroke.Color = Theme.Stroke; sStroke.Transparency = 0.2
    local secLayout = Instance.new("UIListLayout", section); secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local secPadding = Instance.new("UIPadding", section); secPadding.PaddingTop = UDim.new(0, 12); secPadding.PaddingBottom = UDim.new(0, 12)
    local title = Instance.new("TextLabel", section); title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText; title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end)
    return section
end

local function CreateToggle(section, text, descText, defaultState, callback)
    if type(defaultState) == "function" then callback = defaultState; defaultState = false end; itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section); frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, descText and 52 or 34); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(0.7, 0, 0, 34); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = defaultState and Theme.Text or Theme.SubText; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    if descText then local descLabel = Instance.new("TextLabel", frame); descLabel.Size = UDim2.new(0.7, 0, 0, 15); descLabel.Position = UDim2.new(0, 0, 0, 30); descLabel.BackgroundTransparency = 1; descLabel.Text = descText; descLabel.TextColor3 = Theme.SubText; descLabel.Font = Enum.Font.Gotham; descLabel.TextSize = 11; descLabel.TextXAlignment = Enum.TextXAlignment.Left end
    
    local tBtn = Instance.new("TextButton", frame); tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff; tBtn.Text = ""; Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local bStroke = Instance.new("UIStroke", tBtn); bStroke.Color = defaultState and Theme.ToggleOn or Theme.Stroke; bStroke.Transparency = 0.2; AddClickPop(tBtn)
    local circle = Instance.new("Frame", tBtn); circle.Size = UDim2.new(0, 16, 0, 16); circle.Position = defaultState and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8); circle.BackgroundColor3 = defaultState and Theme.Background or Color3.fromRGB(150, 150, 150); Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultState or false
    tBtn.Activated:Connect(function()
        isOn = not isOn
        if isOn then TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Theme.ToggleOn}):Play(); TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(1, -19, 0.5, -8), BackgroundColor3 = Theme.Background}):Play(); TweenService:Create(label, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextColor3 = Theme.Text}):Play(); bStroke.Color = Theme.ToggleOn
        else TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Theme.ToggleOff}):Play(); TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(0, 3, 0.5, -8), BackgroundColor3 = Color3.fromRGB(150, 150, 150)}):Play(); TweenService:Create(label, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextColor3 = Theme.SubText}):Play(); bStroke.Color = Theme.Stroke end
        if callback then callback(isOn) end
    end)
end

local function CreateSlider(section, text, min, max, default, callback)
    itemOrderCounter = itemOrderCounter + 1; local frame = Instance.new("Frame", section); frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(1, 0, 0, 20); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.SubText; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    local valLabel = Instance.new("TextLabel", frame); valLabel.Size = UDim2.new(0, 40, 0, 20); valLabel.Position = UDim2.new(1, -40, 0, 0); valLabel.BackgroundTransparency = 1; valLabel.Text = tostring(default); valLabel.TextColor3 = Theme.Accent; valLabel.Font = Enum.Font.GothamBold; valLabel.TextSize = 13; valLabel.TextXAlignment = Enum.TextXAlignment.Right
    local sliderBg = Instance.new("Frame", frame); sliderBg.Size = UDim2.new(1, 0, 0, 4); sliderBg.Position = UDim2.new(0, 0, 0, 32); sliderBg.BackgroundColor3 = Theme.ToggleOff; Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    local sliderFill = Instance.new("Frame", sliderBg); local percentage = (default - min) / (max - min); sliderFill.Size = UDim2.new(percentage, 0, 1, 0); sliderFill.BackgroundColor3 = Theme.Accent; Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    local knob = Instance.new("TextButton", sliderFill); knob.Size = UDim2.new(0, 14, 0, 14); knob.Position = UDim2.new(1, -7, 0.5, -7); knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255); knob.Text = ""; Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    
    local dragging = false
    local function setSlider(value) local relative = math.clamp((value - min) / (max - min), 0, 1); valLabel.Text = tostring(value); TweenService:Create(sliderFill, TweenInfo.new(0.08, Enum.EasingStyle.Quad), {Size = UDim2.new(relative, 0, 1, 0)}):Play(); if callback then callback(value) end end
    knob.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(1, -9, 0.5, -9)}):Play() end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 14, 0, 14), Position = UDim2.new(1, -7, 0.5, -7)}):Play() end end)
    UserInputService.InputChanged:Connect(function(input) if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local relative = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1); setSlider(math.floor(min + (max - min) * relative)) end end)
end

local function CreateDropdown(section, headerText, itemsList, targetConfigKey, callback)
    itemOrderCounter = itemOrderCounter + 1; local frame = Instance.new("Frame", section); frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 160); frame.BackgroundTransparency = 1
    local header = Instance.new("TextLabel", frame); header.Size = UDim2.new(1, 0, 0, 20); header.BackgroundTransparency = 1; header.Text = headerText .. ": " .. tostring(RyuConfig[targetConfigKey] or "None"); header.TextColor3 = Theme.SubText; header.Font = Enum.Font.GothamMedium; header.TextSize = 12; header.TextXAlignment = Enum.TextXAlignment.Left
    local scroll = Instance.new("ScrollingFrame", frame); scroll.Size = UDim2.new(1, 0, 0, 130); scroll.Position = UDim2.new(0, 0, 0, 25); scroll.BackgroundColor3 = Theme.Background; scroll.ScrollBarThickness = 4; scroll.ScrollBarImageColor3 = Theme.Accent; Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", scroll).Color = Theme.Stroke
    local listLayout = Instance.new("UIListLayout", scroll); listLayout.Padding = UDim.new(0, 4); listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    local buttons = {}
    for _, itemName in ipairs(itemsList) do
        local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); local isSelected = (RyuConfig[targetConfigKey] == itemName); btn.BackgroundColor3 = isSelected and Theme.Accent or Theme.SectionBG; btn.Text = "  " .. itemName; btn.TextColor3 = isSelected and Color3.fromRGB(10,10,12) or Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
        buttons[itemName] = btn
        btn.Activated:Connect(function()
            RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. itemName
            for name, b in pairs(buttons) do if name == itemName then b.BackgroundColor3 = Theme.Accent; b.TextColor3 = Color3.fromRGB(10,10,12) else b.BackgroundColor3 = Theme.SectionBG; b.TextColor3 = Theme.Text end end
            if callback then callback(itemName) end
        end)
    end
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
end

local function CreateButton(section, text, callback)
    itemOrderCounter = itemOrderCounter + 1; local btn = Instance.new("TextButton", section); btn.LayoutOrder = itemOrderCounter; btn.Size = UDim2.new(0.92, 0, 0, 36); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = text; btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 13; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6); local bStroke = Instance.new("UIStroke", btn); bStroke.Color = Theme.Stroke; bStroke.Transparency = 0.2; AddClickPop(btn)
    btn.Activated:Connect(function() if callback then callback() end end)
end

--// ============================================================================
--// 7. POPULATING TABS
--// ============================================================================

-- =======================
-- CATEGORY 1: BATTLE ROYALE
-- =======================
local TabBattleRoyale = CreateMainTab("Battle Royale")
local SubCharacter = CreateSubTab(TabBattleRoyale, "Character")

local SecMovement = CreateSection(SubCharacter, "Movement Settings")
CreateToggle(SecMovement, "Speed Hack", "Locks walk speed to slider value", RyuConfig.SpeedHack, function(state) RyuConfig.SpeedHack = state; PlayerMods:SetSpeed(RyuConfig.SpeedValue, state) end)
CreateSlider(SecMovement, "Speed Value", 16, 150, RyuConfig.SpeedValue, function(val) RyuConfig.SpeedValue = val; if RyuConfig.SpeedHack then PlayerMods:SetSpeed(val, true) end end)
CreateToggle(SecMovement, "High Jump", "Locks jump height to slider value", RyuConfig.HighJump, function(state) RyuConfig.HighJump = state; PlayerMods:SetJumpPower(RyuConfig.JumpValue, state) end)
CreateSlider(SecMovement, "Jump Value", 50, 200, RyuConfig.JumpValue, function(val) RyuConfig.JumpValue = val; if RyuConfig.HighJump then PlayerMods:SetJumpPower(val, true) end end)

local SecPhysics = CreateSection(SubCharacter, "Physics & Camera")
CreateToggle(SecPhysics, "Low Gravity", "Reduces world gravity", RyuConfig.LowGravity, function(state) RyuConfig.LowGravity = state; PlayerMods:SetGravity(RyuConfig.GravityValue, state) end)
CreateSlider(SecPhysics, "Gravity Value", 0, 196, RyuConfig.GravityValue, function(val) RyuConfig.GravityValue = val; if RyuConfig.LowGravity then PlayerMods:SetGravity(val, true) end end)
CreateToggle(SecPhysics, "FOV Changer", "Modifies camera view angle", RyuConfig.FOVChanger, function(state) RyuConfig.FOVChanger = state; PlayerMods:SetFOV(RyuConfig.FOVValue, state) end)
CreateSlider(SecPhysics, "FOV Value", 70, 120, RyuConfig.FOVValue, function(val) RyuConfig.FOVValue = val; if RyuConfig.FOVChanger then PlayerMods:SetFOV(val, true) end end)

local SecVisuals = CreateSection(SubCharacter, "Visuals")
CreateToggle(SecVisuals, "Player ESP", "Shows players through walls", RyuConfig.ESP, function(state) RyuConfig.ESP = state; ESPModule:Toggle(state) end)
CreateSlider(SecVisuals, "ESP Transparency", 0, 90, RyuConfig.ESPTransparency, function(val) RyuConfig.ESPTransparency = val; if RyuConfig.ESP then ESPModule:UpdateTransparency(val) end end)

-- =======================
-- CATEGORY 2: FARM
-- =======================
local TabFarm = CreateMainTab("Farm")
local SubLeveling = CreateSubTab(TabFarm, "Leveling")

-- Controls (Top)
local SecAutoFarmMain = CreateSection(SubLeveling, "Farm Controls")
CreateToggle(SecAutoFarmMain, "Auto Farm", "Tweens safely to enemy & auto attacks", RyuConfig.AutoFarm, function(state) RyuConfig.AutoFarm = state end)
CreateToggle(SecAutoFarmMain, "Auto Quest", "Automatically takes quests", RyuConfig.AutoQuest, function(state) RyuConfig.AutoQuest = state end)

-- Configurations (Bottom)
local SecAutoFarmConfig = CreateSection(SubLeveling, "Farm Configuration")
CreateDropdown(SecAutoFarmConfig, "Select Weapon", GPOWeapons, "TargetWeapon")
CreateDropdown(SecAutoFarmConfig, "Select Enemy", GPOEnemies, "TargetMob")
CreateDropdown(SecAutoFarmConfig, "Select Quest NPC", GPONPCs, "TargetNPC")

-- Farm Positioning
local SecFarmAdvanced = CreateSection(SubLeveling, "Advanced Bypass Settings")
CreateDropdown(SecFarmAdvanced, "Farm Position", FarmPositions, "FarmPosition")
CreateSlider(SecFarmAdvanced, "Farm Offset / Distance", 0, 35, RyuConfig.FarmOffset, function(val) RyuConfig.FarmOffset = val end)
CreateSlider(SecFarmAdvanced, "Tween Speed", 50, 400, RyuConfig.TweenSpeed, function(val) RyuConfig.TweenSpeed = val end)

-- Teleports
local SecTeleports = CreateSection(SubLeveling, "Safe Teleports")
CreateDropdown(SecTeleports, "Select TP Method", TPMethods, "TPMethod")
CreateDropdown(SecTeleports, "Select Island", GPOIslands, "TargetIsland")

CreateButton(SecTeleports, "Teleport To Island", function()
    local target = Workspace:FindFirstChild(RyuConfig.TargetIsland, true)
    if target then
        local pos = target:IsA("Model") and target:GetPivot() or target.CFrame
        task.spawn(function()
            if RyuConfig.TPMethod == "Sky Tween" then
                SkyTween(pos * CFrame.new(0, 50, 0))
            else
                SafeGroundTween(pos * CFrame.new(0, 10, 0), RyuConfig.TweenSpeed)
            end
            RyuNotify:Send("Teleport", "Arrived at " .. RyuConfig.TargetIsland, 3)
        end)
    end
end)

CreateButton(SecTeleports, "Teleport To NPC", function()
    local target = Workspace:FindFirstChild(RyuConfig.TargetNPC, true)
    if target then
        local pos = target:IsA("Model") and target:GetPivot() or target.CFrame
        task.spawn(function()
            if RyuConfig.TPMethod == "Sky Tween" then
                SkyTween(pos * CFrame.new(0, 0, 5))
            else
                SafeGroundTween(pos * CFrame.new(0, 0, 5), RyuConfig.TweenSpeed)
            end
            RyuNotify:Send("Teleport", "Arrived at " .. RyuConfig.TargetNPC, 3)
        end)
    end
end)


--// ============================================================================
--// 8. GPO AUTO FARM ENGINE (STRICT GROUND WORKFLOW + NEW REMOTES)
--// ============================================================================

local function GetCurrentQuest()
    local playerQuestData = LocalPlayer:FindFirstChild("Quest")
    if playerQuestData and playerQuestData:FindFirstChild("CurrentQuest") then
        return playerQuestData.CurrentQuest.Value
    end
    return "None"
end

local function GetGPOMob(mobName)
    local npcs = Workspace:FindFirstChild("NPCs")
    if not npcs then return nil end
    local closestTarget, shortestDistance = nil, math.huge
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end

    for _, npc in pairs(npcs:GetChildren()) do
        if npc.Name:lower():find(mobName:lower()) then
            local hum = npc:FindFirstChildOfClass("Humanoid")
            local targetRoot = npc:FindFirstChild("HumanoidRootPart")
            if hum and targetRoot and hum.Health > 0 then
                local dist = (targetRoot.Position - root.Position).Magnitude
                if dist < shortestDistance then shortestDistance = dist; closestTarget = npc end
            end
        end
    end
    return closestTarget
end

local lastReloadTick = 0

-- Workflow Loop
task.spawn(function()
    while true do
        task.wait(0.2)
        
        -- 1. Auto Quest Loop
        if RyuConfig.AutoQuest and RyuConfig.TargetNPC and RyuConfig.TargetNPC ~= "" then
            local currentQuest = GetCurrentQuest()
            if currentQuest == "None" or currentQuest == "" then
                local npcTarget = Workspace:FindFirstChild(RyuConfig.TargetNPC, true)
                if npcTarget then
                    local npcPos = npcTarget:IsA("Model") and npcTarget:GetPivot() or npcTarget.CFrame
                    -- 100% Sicherer Boden-Tween zur Quest
                    SafeGroundTween(npcPos * CFrame.new(0, 0, 5), RyuConfig.TweenSpeed)
                    task.wait(0.5)
                    pcall(function()
                        local questEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("Quest")
                        if questEvent then 
                            -- NEU: npcChat Bypass für die Quest-Sicherheit
                            questEvent:InvokeServer(unpack({{"npcChat", true}}))
                            task.wait(0.2)
                            questEvent:InvokeServer("takequest") 
                        end
                    end)
                end
            end
        end

        -- 2. Auto Farm Loop
        if RyuConfig.AutoFarm and RyuConfig.TargetMob and RyuConfig.TargetMob ~= "" then
            local target = GetGPOMob(RyuConfig.TargetMob)
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            if target and root and hum then
                local targetRoot = target:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    
                    -- Auto Equip Weapon
                    local weapon = char:FindFirstChild(RyuConfig.TargetWeapon)
                    if not weapon then
                        local packWeap = LocalPlayer.Backpack:FindFirstChild(RyuConfig.TargetWeapon)
                        if packWeap then hum:EquipTool(packWeap) end
                    end
                    
                    weapon = char:FindFirstChild(RyuConfig.TargetWeapon)
                    
                    -- Sicheres Farm Positioning ermitteln
                    local farmPos
                    if RyuConfig.FarmPosition == "Behind (Safe)" then
                        farmPos = targetRoot.CFrame * CFrame.new(0, 0, RyuConfig.FarmOffset)
                    elseif RyuConfig.FarmPosition == "Distance (Gun)" then
                        farmPos = targetRoot.CFrame * CFrame.new(0, 0, RyuConfig.FarmOffset + 15) -- Fest auf Distanz + Offset
                    elseif RyuConfig.FarmPosition == "Below (Underground)" then
                        farmPos = targetRoot.CFrame * CFrame.new(0, -RyuConfig.FarmOffset, 0)
                    elseif RyuConfig.FarmPosition == "Above (Max 12)" then
                        local safeY = math.clamp(RyuConfig.FarmOffset, 0, 12)
                        farmPos = targetRoot.CFrame * CFrame.new(0, safeY, 0)
                    end
                    
                    SafeGroundTween(farmPos, RyuConfig.TweenSpeed)
                    
                    -- Auto Aim
                    camera.CFrame = CFrame.new(camera.CFrame.Position, targetRoot.Position)
                    root.CFrame = CFrame.new(root.Position, Vector3.new(targetRoot.Position.X, root.Position.Y, targetRoot.Position.Z))
                    
                    -- Attack Spam & Reload
                    while target and target.Parent and targetRoot and targetRoot.Parent and RyuConfig.AutoFarm do
                        pcall(function()
                            if weapon and weapon:FindFirstChild("Muzzle") then
                                local args = {
                                    "fire",
                                    {
                                        Start = weapon.Muzzle.CFrame,
                                        Gun = weapon.Name,
                                        joe = "true",
                                        Position = targetRoot.Position
                                    }
                                }
                                ReplicatedStorage:WaitForChild("Events"):WaitForChild("GunManager"):FireServer(unpack(args))
                            else
                                -- NEU: 100% Sicherer Melee Combat Register
                                local combatReg = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("CombatRegister")
                                if combatReg and (not weapon or weapon.Name == "Melee") then
                                    local animFolder = ReplicatedStorage:FindFirstChild("CombatAnimations")
                                    local punchAnim = animFolder and animFolder:FindFirstChild("Melee") and animFolder.Melee:FindFirstChild("Punch2")
                                    if punchAnim then
                                        local meleeArgs = {
                                            {
                                                "swingsfx",
                                                "Melee",
                                                2,
                                                "Ground",
                                                false,
                                                punchAnim,
                                                2,
                                                1.5
                                            }
                                        }
                                        combatReg:InvokeServer(unpack(meleeArgs))
                                    end
                                end
                                
                                VirtualUser:CaptureController()
                                VirtualUser:ClickButton1(Vector2.new())
                            end
                            
                            -- RELOAD SIMULATOR (Alle 4s)
                            if tick() - lastReloadTick > 4 then
                                lastReloadTick = tick()
                                task.spawn(function()
                                    if weapon then
                                        local args = { "reload", { Gun = weapon.Name } }
                                        ReplicatedStorage:WaitForChild("Events"):WaitForChild("GunManager"):WaitForChild("gunFunctions"):InvokeServer(unpack(args))
                                    end
                                end)
                            end
                        end)
                        task.wait(0.1)
                    end
                end
            end
        end
    end
end)

--// ============================================================================
--// 9. INITIALIZATION / STARTUP
--// ============================================================================
task.wait(0.5)
RyuNotify:Send("RYU HUB", "Battle Royale & GPO Edition loaded! Stay safe.", 4)
