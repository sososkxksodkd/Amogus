--// ============================================================================
--// RYU HUB - CLEAN MASTER BUILD (V1 - STABLE AUTO FARM)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")

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
    KillHeight = 8
}

-- Mobs (nur echte Feinde, keine Shop-NPCs)
local GPOEnemies = {
    "Bandit", "Bandit Boss", "Marine", "Corrupt Marine", 
    "Sword Bandit", "Fishman", "Skypiean", "Snow Bandit", "Zombie"
}
local GPOWeapons = { "Combat", "Melee", "Sword", "Katana" }

--// UI SETUP (Strukturiert: Oben Start, Unten Auswahl)
local RyuHub = Instance.new("ScreenGui")
RyuHub.Name = "RyuHubPremium"
RyuHub.ResetOnSpawn = false
RyuHub.Parent = guiParent

local MainFrame = Instance.new("Frame", RyuHub)
MainFrame.Size = UDim2.new(0, 400, 0, 350)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -175)
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
MobScroll.Size = UDim2.new(0.9, 0, 0, 80)
MobScroll.Position = UDim2.new(0.05, 0, 0, 125)
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
WepLabel.Position = UDim2.new(0.05, 0, 0, 215)
WepLabel.BackgroundTransparency = 1
WepLabel.Text = "Wähle Waffe (Aktuell: " .. RyuConfig.TargetWeapon .. ")"
WepLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
WepLabel.Font = Enum.Font.Gotham
WepLabel.TextSize = 12

local WepScroll = Instance.new("ScrollingFrame", MainFrame)
WepScroll.Size = UDim2.new(0.9, 0, 0, 80)
WepScroll.Position = UDim2.new(0.05, 0, 0, 240)
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

--// UNSICHTBARER BODEN (PLATFORM SYSTEM)
local RyuPlatform = Instance.new("Part")
RyuPlatform.Name = "RyuSafePlatform"
RyuPlatform.Size = Vector3.new(10, 1, 10) -- 10x10 Platte
RyuPlatform.Anchored = true
RyuPlatform.Transparency = 1 -- Unsichtbar
RyuPlatform.CanCollide = true -- Spieler kann darauf stehen!
RyuPlatform.Parent = Workspace

-- Hält die Plattform immer genau unter dem Spieler, wenn gefarmt wird
RunService.Heartbeat:Connect(function()
    if RyuConfig.AutoFarm and LocalPlayer.Character then
        local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            -- Setzt die Plattform genau 3.5 Studs unter den Mittelpunkt des Charakters
            RyuPlatform.CFrame = CFrame.new(hrp.Position - Vector3.new(0, 3.5, 0))
        end
    else
        -- Versteckt die Plattform weit weg, wenn Auto Farm aus ist
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
    
    -- Simuliert simplen Mausklick für Angriffe
    VirtualUser:CaptureController()
    VirtualUser:ClickButton1(Vector2.new())
end

--// AUTO FARM CORE LOOP (Zieht 2-5 Gegner zusammen)
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
                    -- 1. Finde Feinde des ausgewählten Typs
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
                    
                    -- 2. Wenn Gegner gefunden wurden, schnapp dir bis zu 5 Stück
                    if #targetMobs > 0 then
                        local mainTarget = targetMobs[1]
                        local mRoot = mainTarget:FindFirstChild("HumanoidRootPart")
                        
                        if mRoot then
                            -- Fliege über den ersten Gegner
                            local farmPosition = mRoot.Position + Vector3.new(0, RyuConfig.KillHeight, 0)
                            SafeTween(farmPosition)
                            
                            -- Töte, solange der Hauptgegner lebt
                            local mHum = mainTarget:FindFirstChildOfClass("Humanoid")
                            while RyuConfig.AutoFarm and mHum and mHum.Health > 0 and hum.Health > 0 do
                                -- Richte Charakter nach unten zum Gegner aus
                                hrp.CFrame = CFrame.lookAt(hrp.Position, mRoot.Position)
                                
                                -- Erweitere die Hitbox leicht, damit die Kills sicher treffen
                                mRoot.Size = Vector3.new(15, 15, 15)
                                mRoot.CanCollide = false
                                
                                EquipAndAttack()
                                task.wait(0.1)
                            end
                        end
                    end
                end
            end
        end
    end
end)
