--// ============================================================================
--// RYU HUB - CLEAN MASTER BUILD (V7 - ULTIMATE VOID ANCHOR)
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
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)

-- TOPBAR DRAGGING (Sauberes Verschieben)
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


--// UNIVERSELLE SLIDER ENGINE
local function CreateUISlider(parent, posY, titleText, minVal, maxVal, defaultVal, callback)
    local label = Instance.new("TextLabel", parent)
    label.Size = UDim2.new(0.9, 0, 0, 18)
    label.Position = UDim2.new(0.05, 0, 0, posY)
    label.BackgroundTransparency = 1
    label.Text = titleText .. ": " .. defaultVal
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left

    local bg = Instance.new("TextButton", parent)
    bg.Size = UDim2.new(0.9, 0, 0, 12)
    bg.Position = UDim2.new(0.05, 0, 0, posY + 20)
    bg.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    bg.Text = ""
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)

    local fill = Instance.new("Frame", bg)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

    local isDragging = false

    local function updateSlider(input)
        local relative = math.clamp((input.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(relative, 0, 1, 0)
        local val = math.floor(minVal + (maxVal - minVal) * relative)
        label.Text = titleText .. ": " .. val
        callback(val)
    end

    bg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = true
            updateSlider(input)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            updateSlider(input)
        end
    end)
end

-- Beide Slider instanziieren
CreateUISlider(MainFrame, 275, "Abstand zum Gegner", 3, 15, RyuConfig.KillHeight, function(val)
    RyuConfig.KillHeight = val
end)

CreateUISlider(MainFrame, 330, "Körper-Blickwinkel (%)", 0, 100, RyuConfig.LookAngle, function(val)
    RyuConfig.LookAngle = val
end)


--// ANTI-GRAVITY HOVER SYSTEM (BODENLOS & SCHWERELOS)
local function ToggleHover(state)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not root then return end
    
    if state then
        if hum then hum.PlatformStand = true end 
        local bv = root:FindFirstChild("RyuHover")
        if not bv then
            bv = Instance.new("BodyVelocity")
            bv.Name = "RyuHover"
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Velocity = Vector3.new(0, 0, 0) 
            bv.Parent = root
        end
    else
        if hum then hum.PlatformStand = false end
        local bv = root:FindFirstChild("RyuHover")
        if bv then bv:Destroy() end
    end
end

--// ABSOLUTER FLING PROTECTOR
RunService.Stepped:Connect(function()
    if RyuConfig.AutoFarm then
        local char = LocalPlayer.Character
        if char then
            local root = char:FindFirstChild("HumanoidRootPart")
            if root then
                root.Velocity = Vector3.new(0, 0, 0)
                root.RotVelocity = Vector3.new(0, 0, 0)
            end
            
            for _, v in pairs(char:GetChildren()) do
                if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" then 
                    v.CanCollide = false 
                end
            end
        end
    end
end)


--// SICHERER LERP-TWEEN (ZITTERFREI!)
local function SafeLerp(targetCFrame)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local startCFrame = root.CFrame
    local dist = (startCFrame.Position - targetCFrame.Position).Magnitude
    local timeToTake = dist / RyuConfig.TweenSpeed
    
    if timeToTake < 0.1 then 
        root.CFrame = targetCFrame
        return 
    end

    local startTime = tick()
    while tick() - startTime < timeToTake do
        if not RyuConfig.AutoFarm then break end
        local alpha = (tick() - startTime) / timeToTake
        root.CFrame = startCFrame:Lerp(targetCFrame, alpha)
        RunService.Heartbeat:Wait()
    end
    root.CFrame = targetCFrame
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


--// AUTO FARM CORE LOOP (Y-AXIS ANCHOR)
task.spawn(function()
    while true do
        task.wait(0.1)
        
        if RyuConfig.AutoFarm then
            local char = LocalPlayer.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            if hrp and hum and hum.Health > 0 then
                ToggleHover(true) 

                local npcs = Workspace:FindFirstChild("NPCs")
                if npcs then
                    local targetMobs = {}
                    for _, npc in pairs(npcs:GetChildren()) do
                        if npc.Name == RyuConfig.TargetMob then
                            local mHum = npc:FindFirstChildOfClass("Humanoid")
                            local mRoot = npc:FindFirstChild("HumanoidRootPart")
                            -- VOID-BUGFIX: Wir prüfen NICHT mehr auf > 5, das hat Fishman Cave zerstört!
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
                            -- DER RETTENDE Y-ANKER: Wir merken uns die Höhe des Gegners BEVOR der Kampf losgeht!
                            local startY = mRoot.Position.Y 
                            
                            local targetSkyPos = Vector3.new(mRoot.Position.X, startY + RyuConfig.KillHeight, mRoot.Position.Z)
                            
                            -- Anflug
                            SafeLerp(CFrame.new(targetSkyPos))
                            
                            -- Kampf Loop
                            while RyuConfig.AutoFarm and mHum and mHum.Health > 0 and hum.Health > 0 do
                                -- VOID-NOTBREMSE: Wenn der Gegner > 15 Studs durch den Boden fällt, brich SOFORT ab!
                                if mRoot.Position.Y < (startY - 15) then break end 
                                
                                -- SICHERHEITS-Y: Zwingt deinen Charakter, niemals dem NPC tief in den Boden zu folgen!
                                local safeNPC_Y = math.max(mRoot.Position.Y, startY - 5)
                                targetSkyPos = Vector3.new(mRoot.Position.X, safeNPC_Y + RyuConfig.KillHeight, mRoot.Position.Z)
                                
                                -- DYNAMIC PITCH ROTATION (Ausrichtung)
                                local flatLook = CFrame.lookAt(targetSkyPos, Vector3.new(mRoot.Position.X, targetSkyPos.Y, mRoot.Position.Z))
                                local fullLook = CFrame.lookAt(targetSkyPos, Vector3.new(mRoot.Position.X, safeNPC_Y, mRoot.Position.Z))
                                
                                -- Blendet stufenlos zwischen flachem und steilem Blick
                                local charRotation = flatLook:Lerp(fullLook, RyuConfig.LookAngle / 100)
                                hrp.CFrame = charRotation
                                
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
            else
                ToggleHover(false)
            end
        else
            ToggleHover(false)
        end
    end
end)
