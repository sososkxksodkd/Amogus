--// ============================================================================
--// RYU HUB - CLEAN MASTER BUILD (V4 - FLING FIX & DUAL SLIDERS)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function() 
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end 
end)
for _, v in pairs(guiParent:GetChildren()) do 
    if v.Name == "RyuHubPremium" then v:Destroy() end 
end

--// EINFACHE KONFIGURATION
local RyuConfig = {
    AutoFarm = false,
    TargetMob = "Bandit",
    TargetWeapon = "Combat",
    TweenSpeed = 45,
    KillHeight = 5,    -- Höhe über dem Feind (3 bis 15 Studs)
    LookAngle = 50     -- Neigungswinkel zum Feind (0% bis 100%)
}

-- Mobs (nur echte Feinde)
local GPOEnemies = {
    "Bandit", "Bandit Boss", "Marine", "Corrupt Marine", 
    "Sword Bandit", "Fishman", "Skypiean", "Snow Bandit", "Zombie"
}
local GPOWeapons = { "Combat", "Melee", "Sword", "Katana" }

--// UI SETUP
local RyuHub = Instance.new("ScreenGui")
RyuHub.Name = "RyuHubPremium"
RyuHub.ResetOnSpawn = false
RyuHub.Parent = guiParent

local MainFrame = Instance.new("Frame", RyuHub)
MainFrame.Size = UDim2.new(0, 400, 0, 430) 
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -215)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
MainFrame.Active = true
MainFrame.Draggable = false -- DEAKTIVIERT: Verhindert, dass das Fenster die Mausklicks der Slider stiehlt!
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)

-- TOPBAR DRAGGING (Ermöglicht sauberes Verschieben nur über die Titelleiste)
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Size = UDim2.new(1, 0, 0, 40)
Topbar.BackgroundTransparency = 1

local draggingFrame, dragStart, startPos = false, nil, nil
Topbar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingFrame = true; dragStart = input.Position; startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if draggingFrame and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingFrame = false
    end
end)

local Title = Instance.new("TextLabel", Topbar)
Title.Size = UDim2.new(1, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.Text = "RYU HUB - STABLE FARM"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

-- Oben: Option zum Starten
local StartBtn = Instance.new("TextButton", MainFrame)
StartBtn.Size = UDim2.new(0.9, 0, 0, 36)
StartBtn.Position = UDim2.new(0.05, 0, 0, 45)
StartBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
StartBtn.Text = "AUTO FARM: OFF"
StartBtn.TextColor3 = Color3.fromRGB(200, 50, 50)
StartBtn.Font = Enum.Font.GothamBold
StartBtn.TextSize = 14
Instance.new("UICorner", StartBtn).CornerRadius = UDim.new(0, 6)

StartBtn.Activated:Connect(function()
    RyuConfig.AutoFarm = not RyuConfig.AutoFarm
    if RyuConfig.AutoFarm then
        StartBtn.Text = "AUTO FARM: ON"
        StartBtn.TextColor3 = Color3.fromRGB(50, 200, 50)
    else
        StartBtn.Text = "AUTO FARM: OFF"
        StartBtn.TextColor3 = Color3.fromRGB(200, 50, 50)
    end
end)

-- Feind-Auswahl
local MobLabel = Instance.new("TextLabel", MainFrame)
MobLabel.Size = UDim2.new(0.9, 0, 0, 18)
MobLabel.Position = UDim2.new(0.05, 0, 0, 90)
MobLabel.BackgroundTransparency = 1
MobLabel.Text = "Wähle Feind (Aktuell: " .. RyuConfig.TargetMob .. ")"
MobLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
MobLabel.Font = Enum.Font.Gotham
MobLabel.TextSize = 11

local MobScroll = Instance.new("ScrollingFrame", MainFrame)
MobScroll.Size = UDim2.new(0.9, 0, 0, 65)
MobScroll.Position = UDim2.new(0.05, 0, 0, 110)
MobScroll.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MobScroll.ScrollBarThickness = 4
local MobLayout = Instance.new("UIListLayout", MobScroll)

for _, mobName in ipairs(GPOEnemies) do
    local btn = Instance.new("TextButton", MobScroll)
    btn.Size = UDim2.new(1, 0, 0, 22)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    btn.Text = " " .. mobName
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 11
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Activated:Connect(function()
        RyuConfig.TargetMob = mobName
        MobLabel.Text = "Wähle Feind (Aktuell: " .. mobName .. ")"
    end)
end
MobLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() MobScroll.CanvasSize = UDim2.new(0, 0, 0, MobLayout.AbsoluteContentSize.Y) end)

-- Waffen-Auswahl
local WepLabel = Instance.new("TextLabel", MainFrame)
WepLabel.Size = UDim2.new(0.9, 0, 0, 18)
WepLabel.Position = UDim2.new(0.05, 0, 0, 180)
WepLabel.BackgroundTransparency = 1
WepLabel.Text = "Wähle Waffe (Aktuell: " .. RyuConfig.TargetWeapon .. ")"
WepLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
WepLabel.Font = Enum.Font.Gotham
WepLabel.TextSize = 11

local WepScroll = Instance.new("ScrollingFrame", MainFrame)
WepScroll.Size = UDim2.new(0.9, 0, 0, 65)
WepScroll.Position = UDim2.new(0.05, 0, 0, 200)
WepScroll.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
WepScroll.ScrollBarThickness = 4
local WepLayout = Instance.new("UIListLayout", WepScroll)

for _, wepName in ipairs(GPOWeapons) do
    local btn = Instance.new("TextButton", WepScroll)
    btn.Size = UDim2.new(1, 0, 0, 22)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    btn.Text = " " .. wepName
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 11
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Activated:Connect(function()
        RyuConfig.TargetWeapon = wepName
        WepLabel.Text = "Wähle Waffe (Aktuell: " .. wepName .. ")"
    end)
end
WepLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() WepScroll.CanvasSize = UDim2.new(0, 0, 0, WepLayout.AbsoluteContentSize.Y) end)

--// SLIDER FUNKTIONS-BAUSTEIN (Universal & 100% Repariert)
local function CreateUISlider(posY, titleText, minVal, maxVal, defaultVal, callback)
    local label = Instance.new("TextLabel", MainFrame)
    label.Size = UDim2.new(0.9, 0, 0, 18)
    label.Position = UDim2.new(0.05, 0, 0, posY)
    label.BackgroundTransparency = 1
    label.Text = titleText .. ": " .. defaultVal
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left

    local bg = Instance.new("Frame", MainFrame)
    bg.Size = UDim2.new(0.9, 0, 0, 12)
    bg.Position = UDim2.new(0.05, 0, 0, posY + 20)
    bg.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)

    local fill = Instance.new("Frame", bg)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

    local isSliderDragging = false

    local function UpdateSlider(input)
        local relative = math.clamp((input.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(relative, 0, 1, 0)
        local calculatedVal = math.floor(minVal + (maxVal - minVal) * relative)
        label.Text = titleText .. ": " .. calculatedVal
        callback(calculatedVal)
    end

    bg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            isSliderDragging = true
            UpdateSlider(input)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if isSliderDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            UpdateSlider(input)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            isSliderDragging = false
        end
    end)
end

-- SLIDER 1: ABSTAND ZUM GEGNER (Höhe)
CreateUISlider(275, "Abstand zum Gegner (Höhe)", 3, 15, RyuConfig.KillHeight, function(val)
    RyuConfig.KillHeight = val
end)

-- SLIDER 2: KÖRPER-NEIGUNGSWINKEL (Guck-Funktion)
CreateUISlider(335, "Körper-Blickwinkel (Schräg gucken)", 0, 100, RyuConfig.LookAngle, function(val)
    RyuConfig.LookAngle = val
end)

--// UNSICHTBARER BODEN (PLATFORM SYSTEM)
local RyuPlatform = Instance.new("Part")
RyuPlatform.Name = "RyuSafePlatform"
RyuPlatform.Size = Vector3.new(12, 1, 12) 
RyuPlatform.Anchored = true
RyuPlatform.Transparency = 1 
RyuPlatform.CanCollide = true 
RyuPlatform.Parent = Workspace

-- Status-Manager für den Boden
RunService.Heartbeat:Connect(function()
    if not RyuConfig.AutoFarm then
        RyuPlatform.CFrame = CFrame.new(0, 99999, 0)
    end
end)

--// SICHERER TWEEN
local function SafeTween(targetPos)
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local dist = (hrp.Position - targetPos).Magnitude
    local timeToTake = dist / RyuConfig.TweenSpeed
    
    if timeToTake < 0.1 then 
        hrp.CFrame = CFrame.new(targetPos)
        return 
    end

    local twInfo = TweenInfo.new(timeToTake, Enum.EasingStyle.Linear)
    local tw = TweenService:Create(hrp, twInfo, {CFrame = CFrame.new(targetPos)})
    tw:Play()
    tw.Completed:Wait()
end

--// AUTO ATTACK
local function EquipAndAttack()
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    
    local tool = char:FindFirstChild(RyuConfig.TargetWeapon) or LocalPlayer.Backpack:FindFirstChild(RyuConfig.TargetWeapon)
    if tool and tool.Parent == LocalPlayer.Backpack then
        hum:EquipTool(tool)
    end
    
    VirtualUser:CaptureController()
    VirtualUser:ClickButton1(Vector2.new())
end

--// AUTO FARM CORE LOOP (Gefixtes Rotations- & Void-System)
task.spawn(function()
    while true do
        task.wait(0.1)
        
        if RyuConfig.AutoFarm then
            local char = LocalPlayer.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            -- VOID NOTBREMSE: Verhindert Stürze unter die Map
            if hrp and hrp.Position.Y < -50 then
                hrp.CFrame = CFrame.new(0, 150, 0)
                task.wait(1)
            end
            
            if hrp and hum and hum.Health > 0 then
                local npcs = Workspace:FindFirstChild("NPCs")
                if npcs then
                    local targetMobs = {}
                    for _, npc in pairs(npcs:GetChildren()) do
                        if npc.Name == RyuConfig.TargetMob then
                            local mHum = npc:FindFirstChildOfClass("Humanoid")
                            local mRoot = npc:FindFirstChild("HumanoidRootPart")
                            if mHum and mRoot and mHum.Health > 0 and mRoot.Position.Y > 0 then
                                table.insert(targetMobs, npc)
                            end
                        end
                    end
                    
                    if #targetMobs > 0 then
                        local mainTarget = targetMobs[1]
                        local mRoot = mainTarget:FindFirstChild("HumanoidRootPart")
                        local mHum = mainTarget:FindFirstChildOfClass("Humanoid")
                        
                        if mRoot and mHum then
                            local targetSkyPos = mRoot.Position + Vector3.new(0, RyuConfig.KillHeight, 0)
                            
                            -- Plattform vor dem Flug ausrichten
                            RyuPlatform.CFrame = CFrame.new(targetSkyPos.X, targetSkyPos.Y - 3.2, targetSkyPos.Z)
                            SafeTween(targetSkyPos)
                            
                            while RyuConfig.AutoFarm and mHum and mHum.Health > 0 and hum.Health > 0 do
                                targetSkyPos = mRoot.Position + Vector3.new(0, RyuConfig.KillHeight, 0)
                                
                                -- 1. DIE PLATFORM BLEIBT IMMER 100% WAAGERECHT FLACH UNTER DEN FÜSSEN!
                                RyuPlatform.CFrame = CFrame.new(targetSkyPos.X, targetSkyPos.Y - 3.2, targetSkyPos.Z)
                                
                                -- 2. DYNAMIC PITCH ROTATION (Charakter schaut schräg nach unten zum Feind)
                                local flatLook = CFrame.lookAt(targetSkyPos, Vector3.new(mRoot.Position.X, targetSkyPos.Y, mRoot.Position.Z))
                                local fullLook = CFrame.lookAt(targetSkyPos, mRoot.Position)
                                
                                -- Blendet stufenlos über Slider (0% bis 100%) zwischen flachem und steilem Blick
                                local charRotation = flatLook:Lerp(fullLook, RyuConfig.LookAngle / 100)
                                hrp.CFrame = charRotation
                                
                                -- Stopper für ungewollte Physik-Impulse
                                hrp.Velocity = Vector3.new(0, 0, 0)
                                
                                -- Massen-Hitbox (Bis zu 5 Mobs)
                                local aggroCount = 0
                                for _, npc in pairs(targetMobs) do
                                    if npc and npc.Parent and aggroCount < 5 then
                                        local subRoot = npc:FindFirstChild("HumanoidRootPart")
                                        local subHum = npc:FindFirstChildOfClass("Humanoid")
                                        if subRoot and subHum and subHum.Health > 0 then
                                            local dist = (subRoot.Position - mRoot.Position).Magnitude
                                            if dist < 40 then 
                                                aggroCount = aggroCount + 1
                                                subRoot.Size = Vector3.new(15, 15, 15)
                                                subRoot.CanCollide = false
                                                subRoot.Transparency = 0.8 
                                            end
                                        end
                                    end
                                end
                                
                                EquipAndAttack()
                                task.wait(0.1)
                            end
                            
                            RyuPlatform.CFrame = CFrame.new(0, 99999, 0)
                            
                            for _, npc in pairs(targetMobs) do
                                local subRoot = npc and npc:FindFirstChild("HumanoidRootPart")
                                if subRoot then
                                    subRoot.Size = Vector3.new(2, 2, 1)
                                    subRoot.Transparency = 0
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end)
