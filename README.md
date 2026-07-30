--// ============================================================================
--// RYU HUB - CLEAN MASTER BUILD (V2 - PERFECT HITBOX & SLIDER)
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
    KillHeight = 5 -- Standardmäßig 5 Studs Abstand
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
MainFrame.Size = UDim2.new(0, 400, 0, 390) -- Frame etwas größer gemacht für den Slider
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -195)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
MainFrame.Active = true
MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "RYU HUB - STABLE FARM"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

-- Oben: Option zum Starten
local StartBtn = Instance.new("TextButton", MainFrame)
StartBtn.Size = UDim2.new(0.9, 0, 0, 40)
StartBtn.Position = UDim2.new(0.05, 0, 0, 50)
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

-- Unten: Auswahl Tabellen
local MobLabel = Instance.new("TextLabel", MainFrame)
MobLabel.Size = UDim2.new(0.9, 0, 0, 20)
MobLabel.Position = UDim2.new(0.05, 0, 0, 100)
MobLabel.BackgroundTransparency = 1
MobLabel.Text = "Wähle Feind (Aktuell: " .. RyuConfig.TargetMob .. ")"
MobLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
MobLabel.Font = Enum.Font.Gotham
MobLabel.TextSize = 12

local MobScroll = Instance.new("ScrollingFrame", MainFrame)
MobScroll.Size = UDim2.new(0.9, 0, 0, 75)
MobScroll.Position = UDim2.new(0.05, 0, 0, 120)
MobScroll.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MobScroll.ScrollBarThickness = 4
local MobLayout = Instance.new("UIListLayout", MobScroll)

for _, mobName in ipairs(GPOEnemies) do
    local btn = Instance.new("TextButton", MobScroll)
    btn.Size = UDim2.new(1, 0, 0, 25)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    btn.Text = " " .. mobName
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 12
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Activated:Connect(function()
        RyuConfig.TargetMob = mobName
        MobLabel.Text = "Wähle Feind (Aktuell: " .. mobName .. ")"
    end)
end
MobLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() MobScroll.CanvasSize = UDim2.new(0, 0, 0, MobLayout.AbsoluteContentSize.Y) end)

local WepLabel = Instance.new("TextLabel", MainFrame)
WepLabel.Size = UDim2.new(0.9, 0, 0, 20)
WepLabel.Position = UDim2.new(0.05, 0, 0, 205)
WepLabel.BackgroundTransparency = 1
WepLabel.Text = "Wähle Waffe (Aktuell: " .. RyuConfig.TargetWeapon .. ")"
WepLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
WepLabel.Font = Enum.Font.Gotham
WepLabel.TextSize = 12

local WepScroll = Instance.new("ScrollingFrame", MainFrame)
WepScroll.Size = UDim2.new(0.9, 0, 0, 75)
WepScroll.Position = UDim2.new(0.05, 0, 0, 225)
WepScroll.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
WepScroll.ScrollBarThickness = 4
local WepLayout = Instance.new("UIListLayout", WepScroll)

for _, wepName in ipairs(GPOWeapons) do
    local btn = Instance.new("TextButton", WepScroll)
    btn.Size = UDim2.new(1, 0, 0, 25)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    btn.Text = " " .. wepName
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 12
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Activated:Connect(function()
        RyuConfig.TargetWeapon = wepName
        WepLabel.Text = "Wähle Waffe (Aktuell: " .. wepName .. ")"
    end)
end
WepLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() WepScroll.CanvasSize = UDim2.new(0, 0, 0, WepLayout.AbsoluteContentSize.Y) end)

--// ABSTAND SLIDER (NEU)
local DistLabel = Instance.new("TextLabel", MainFrame)
DistLabel.Size = UDim2.new(0.9, 0, 0, 20)
DistLabel.Position = UDim2.new(0.05, 0, 0, 315)
DistLabel.BackgroundTransparency = 1
DistLabel.Text = "Abstand zum Gegner: " .. RyuConfig.KillHeight .. " Studs"
DistLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
DistLabel.Font = Enum.Font.GothamBold
DistLabel.TextSize = 12
DistLabel.TextXAlignment = Enum.TextXAlignment.Left

local DistSliderBg = Instance.new("TextButton", MainFrame)
DistSliderBg.Size = UDim2.new(0.9, 0, 0, 12)
DistSliderBg.Position = UDim2.new(0.05, 0, 0, 340)
DistSliderBg.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
DistSliderBg.Text = ""
Instance.new("UICorner", DistSliderBg).CornerRadius = UDim.new(1, 0)

local DistSliderFill = Instance.new("Frame", DistSliderBg)
DistSliderFill.Size = UDim2.new((RyuConfig.KillHeight - 3) / (15 - 3), 0, 1, 0) -- Min 3, Max 15
DistSliderFill.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
Instance.new("UICorner", DistSliderFill).CornerRadius = UDim.new(1, 0)

local dragging = false
DistSliderBg.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local relative = math.clamp((input.Position.X - DistSliderBg.AbsolutePosition.X) / DistSliderBg.AbsoluteSize.X, 0, 1)
        DistSliderFill.Size = UDim2.new(relative, 0, 1, 0)
        RyuConfig.KillHeight = math.floor(3 + (15 - 3) * relative)
        DistLabel.Text = "Abstand zum Gegner: " .. RyuConfig.KillHeight .. " Studs"
    end
end)

--// UNSICHTBARER BODEN (PLATFORM SYSTEM)
local RyuPlatform = Instance.new("Part")
RyuPlatform.Name = "RyuSafePlatform"
RyuPlatform.Size = Vector3.new(10, 1, 10) 
RyuPlatform.Anchored = true
RyuPlatform.Transparency = 1 
RyuPlatform.CanCollide = true 
RyuPlatform.Parent = Workspace

-- Verstecke Plattform, wenn Farmen aus ist
RunService.Heartbeat:Connect(function()
    if not RyuConfig.AutoFarm then
        RyuPlatform.CFrame = CFrame.new(0, 99999, 0)
    end
end)

--// SICHERER TWEEN (Bewegt den Charakter sanft zum Ziel)
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

--// AUTO FARM CORE LOOP (Zieht Mobs zusammen)
task.spawn(function()
    while true do
        task.wait(0.1)
        
        if RyuConfig.AutoFarm then
            local char = LocalPlayer.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            if hrp and hum and hum.Health > 0 then
                local npcs = Workspace:FindFirstChild("NPCs")
                if npcs then
                    -- 1. Finde Feinde
                    local targetMobs = {}
                    for _, npc in pairs(npcs:GetChildren()) do
                        if npc.Name == RyuConfig.TargetMob then
                            local mHum = npc:FindFirstChildOfClass("Humanoid")
                            local mRoot = npc:FindFirstChild("HumanoidRootPart")
                            if mHum and mRoot and mHum.Health > 0 then
                                table.insert(targetMobs, npc)
                            end
                        end
                    end
                    
                    if #targetMobs > 0 then
                        local mainTarget = targetMobs[1]
                        local mRoot = mainTarget:FindFirstChild("HumanoidRootPart")
                        local mHum = mainTarget:FindFirstChildOfClass("Humanoid")
                        
                        if mRoot and mHum then
                            -- ELEVATOR FIX: Ziel-Position basiert IMMER auf dem Gegner, NICHT auf dir!
                            local targetSkyPos = mRoot.Position + Vector3.new(0, RyuConfig.KillHeight, 0)
                            
                            -- Fliege zur Position
                            SafeTween(targetSkyPos)
                            
                            while RyuConfig.AutoFarm and mHum and mHum.Health > 0 and hum.Health > 0 do
                                -- Aktualisiere die Position dynamisch (falls der Gegner läuft)
                                targetSkyPos = mRoot.Position + Vector3.new(0, RyuConfig.KillHeight, 0)
                                
                                -- Setze die Plattform genau 3.1 Studs unter deine gewünschte Standposition
                                RyuPlatform.CFrame = CFrame.new(targetSkyPos - Vector3.new(0, 3.1, 0))
                                
                                -- Richte Charakter starr zum Gegner nach unten aus
                                hrp.CFrame = CFrame.lookAt(targetSkyPos, mRoot.Position)
                                
                                -- MASSEN-HITBOX SYSTEM (Zieht bis zu 5 Gegner zusammen)
                                local aggroCount = 0
                                for _, npc in pairs(targetMobs) do
                                    if npc and npc.Parent and aggroCount < 5 then
                                        local subRoot = npc:FindFirstChild("HumanoidRootPart")
                                        local subHum = npc:FindFirstChildOfClass("Humanoid")
                                        if subRoot and subHum and subHum.Health > 0 then
                                            local dist = (subRoot.Position - mRoot.Position).Magnitude
                                            if dist < 40 then -- Wenn sie nah am Hauptziel sind
                                                aggroCount = aggroCount + 1
                                                -- Erweitere die Hitbox, damit deine Fäuste alle gleichzeitig treffen
                                                subRoot.Size = Vector3.new(15, 15, 15)
                                                subRoot.CanCollide = false
                                                subRoot.Transparency = 0.8 -- Leicht unsichtbar, damit es nicht stört
                                            end
                                        end
                                    end
                                end
                                
                                EquipAndAttack()
                                task.wait(0.1)
                            end
                            
                            -- Resette die Plattform kurz nach dem Kill
                            RyuPlatform.CFrame = CFrame.new(0, 99999, 0)
                            
                            -- Mache tote Gegner wieder normal
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
