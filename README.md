--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION (PC EXCLUSIVE - 1v1 HOVER FARM)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
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

--// SMART NPC & ENEMY SORTER
local KnownQuests = {
    "Ash the Tailor", "Tyson", "Robo", "Robert", "Kevin", "Helen", "Gozen", 
    "Axe Hand Logan", "Captain Zhen", "Pharaoh Akshan", "Moria", "Coby", "Bomi", "Haku"
}
local DynamicEnemies = {}
local DynamicQuests = {}

local function SortNPCs()
    if Workspace:FindFirstChild("NPCs") then
        for _, npc in pairs(Workspace.NPCs:GetChildren()) do
            local hum = npc:FindFirstChildOfClass("Humanoid")
            if hum then
                local isQuest = false
                if table.find(KnownQuests, npc.Name) or string.find(npc.Name:lower(), "quest") then
                    isQuest = true
                end
                
                if isQuest then
                    if not table.find(DynamicQuests, npc.Name) then table.insert(DynamicQuests, npc.Name) end
                else
                    if not table.find(DynamicEnemies, npc.Name) then table.insert(DynamicEnemies, npc.Name) end
                end
            end
        end
    end
    table.sort(DynamicEnemies)
    table.sort(DynamicQuests)
end
SortNPCs()

if #DynamicEnemies == 0 then DynamicEnemies = {"Bandit", "Bandit Boss", "Corrupt Marine"} end
if #DynamicQuests == 0 then DynamicQuests = {"Ash the Tailor", "Tyson"} end

--// RYU CONFIGURATION
local RyuConfig = {
    SpeedHack = false, SpeedValue = 35, 
    HighJump = false, JumpValue = 50, 
    
    AutoFarm = false,
    AutoQuest = false,
    IsTweening = false, 
    TargetMob = DynamicEnemies[1],
    TargetIsland = "Town of Beginnings",
    TargetNPC = DynamicQuests[1],
    TargetWeapon = "Combat",
    
    TweenSpeed = 120,       
    FarmOffset = 6 -- Wir schweben exakt 6 Studs über dem Gegner!
}

local GPOIslands = {
    "Town of Beginnings", "Sandora", "Shell's Town", "Orange Town", 
    "Restaurant Baratie", "Roca Island", "Sphinx Island", "Marine Fort F-1", 
    "Fishman Island", "Colosseum", "Land of the Sky", "Marine Base G-1",
    "Logue Town", "Kori Island", "Island Of Zou", "Gravito's Fort",
    "Shark Park"
}

local GPOWeapons = { "Combat", "Melee", "Sword", "Katana", "Rifle", "Pistol" }

--// RAINBOW OVERHEAD TITLE
local function AddRainbowTag(character)
    local head = character:WaitForChild("Head", 5)
    if head then
        if head:FindFirstChild("RyuHubTag") then head.RyuHubTag:Destroy() end
        local bg = Instance.new("BillboardGui")
        bg.Name = "RyuHubTag"
        bg.Size = UDim2.new(0, 200, 0, 50)
        bg.StudsOffset = Vector3.new(0, 3, 0)
        bg.AlwaysOnTop = true
        bg.Parent = head
        
        local txt = Instance.new("TextLabel")
        txt.Size = UDim2.new(1, 0, 1, 0)
        txt.BackgroundTransparency = 1
        txt.Text = "RYUHUB"
        txt.Font = Enum.Font.GothamBlack
        txt.TextSize = 22
        txt.TextStrokeTransparency = 0
        txt.Parent = bg
        
        task.spawn(function()
            while bg.Parent do
                txt.TextColor3 = Color3.fromHSV(tick() % 5 / 5, 1, 1)
                task.wait()
            end
        end)
    end
end
if LocalPlayer.Character then AddRainbowTag(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(AddRainbowTag)

--// NOTIFICATION SYSTEM
local NotificationContainer = Instance.new("Frame")
NotificationContainer.Name = "RyuNotifications"
NotificationContainer.Size = UDim2.new(0, 260, 1, -40)
NotificationContainer.Position = UDim2.new(1, -280, 0, 20)
NotificationContainer.BackgroundTransparency = 1
NotificationContainer.Parent = guiParent

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

--// UI SETUP (Verkürzt für Übersichtlichkeit, selbes System wie vorher)
local Theme = { Background = Color3.fromRGB(12, 12, 14), Sidebar = Color3.fromRGB(18, 18, 20), SectionBG = Color3.fromRGB(24, 24, 26), Text = Color3.fromRGB(250, 250, 250), SubText = Color3.fromRGB(130, 130, 135), Accent = Color3.fromRGB(255, 255, 255), ToggleOff = Color3.fromRGB(35, 35, 38), ToggleOn = Color3.fromRGB(255, 255, 255), Stroke = Color3.fromRGB(45, 45, 50) }
local MainSize = UDim2.new(0, math.min(750, camera.ViewportSize.X - 40), 0, math.min(480, camera.ViewportSize.Y - 40))
local SidebarWidth = 160
local RyuHub = Instance.new("ScreenGui"); RyuHub.Name = "RyuHubPremium"; RyuHub.ResetOnSpawn = false; RyuHub.IgnoreGuiInset = true; RyuHub.Parent = guiParent

local MainFrame = Instance.new("Frame"); MainFrame.Size = MainSize; MainFrame.Position = UDim2.new(0.5, -MainSize.X.Offset/2, 0.5, -MainSize.Y.Offset/2); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.BorderSizePixel = 0; MainFrame.Active = true; MainFrame.Visible = true; MainFrame.ClipsDescendants = true; MainFrame.Parent = RyuHub; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextXAlignment = Enum.TextXAlignment.Left
local SubTitle = Instance.new("TextLabel", Topbar); SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 38); SubTitle.BackgroundTransparency = 1; SubTitle.Text = "PC Exclusive Edition"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 11; SubTitle.TextXAlignment = Enum.TextXAlignment.Left

local Sidebar = Instance.new("ScrollingFrame", MainFrame); Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar); SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; SideLayout.SortOrder = Enum.SortOrder.LayoutOrder
local ContentContainer = Instance.new("Frame", MainFrame); ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1

-- UI Helper Functions
local function CreateMainTab(name)
    local tabObj = { Btn = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {} }
    local tabBtn = Instance.new("TextButton", Sidebar); tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name); tabBtn.TextColor3 = Theme.SubText; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 13; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8); tabObj.Btn = tabBtn
    local subContainer = Instance.new("Frame", Sidebar); subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer); subLayout.Padding = UDim.new(0, 2); tabObj.SubLayout = subLayout
    tabBtn.Activated:Connect(function() tabObj.IsOpen = not tabObj.IsOpen; subContainer.Size = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0); tabBtn.TextColor3 = tabObj.IsOpen and Theme.Text or Theme.SubText end)
    return tabObj
end
local function CreateSubTab(tabObj, subName)
    local subBtn = Instance.new("TextButton", tabObj.SubContainer); subBtn.Size = UDim2.new(1, 0, 0, 28); subBtn.BackgroundTransparency = 1; subBtn.Text = "     " .. subName; subBtn.TextColor3 = Theme.SubText; subBtn.Font = Enum.Font.GothamMedium; subBtn.TextSize = 12; subBtn.TextXAlignment = Enum.TextXAlignment.Left
    local page = Instance.new("ScrollingFrame", ContentContainer); page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.ScrollBarThickness = 2; page.Visible = false
    local pageLayout = Instance.new("UIListLayout", page); pageLayout.Padding = UDim.new(0, 12); pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20) end)
    subBtn.Activated:Connect(function() for _, v in pairs(ContentContainer:GetChildren()) do v.Visible = false end; page.Visible = true end)
    return page
end
local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page); section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    local secLayout = Instance.new("UIListLayout", section); secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 12); Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 12)
    local title = Instance.new("TextLabel", section); title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText; title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end)
    return section
end
local function CreateToggle(section, text, defaultState, callback)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 34); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(0.7, 0, 1, 0); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.Text; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    local tBtn = Instance.new("TextButton", frame); tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff; tBtn.Text = ""; Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local isOn = defaultState
    tBtn.Activated:Connect(function() isOn = not isOn; tBtn.BackgroundColor3 = isOn and Theme.ToggleOn or Theme.ToggleOff; if callback then callback(isOn) end end)
end
local function CreateDropdown(section, headerText, itemsList, targetConfigKey)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 160); frame.BackgroundTransparency = 1
    local header = Instance.new("TextLabel", frame); header.Size = UDim2.new(1, 0, 0, 20); header.BackgroundTransparency = 1; header.Text = headerText .. ": " .. tostring(RyuConfig[targetConfigKey] or "None"); header.TextColor3 = Theme.SubText; header.Font = Enum.Font.GothamMedium; header.TextSize = 12; header.TextXAlignment = Enum.TextXAlignment.Left
    local scroll = Instance.new("ScrollingFrame", frame); scroll.Size = UDim2.new(1, 0, 0, 130); scroll.Position = UDim2.new(0, 0, 0, 25); scroll.BackgroundColor3 = Theme.Background; scroll.ScrollBarThickness = 4; Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 6)
    local listLayout = Instance.new("UIListLayout", scroll); listLayout.Padding = UDim.new(0, 4); listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    for _, itemName in ipairs(itemsList) do
        local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. itemName; btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
        btn.Activated:Connect(function() RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. itemName end)
    end
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
end

--// UI AUFBAU
local TabFarm = CreateMainTab("Farm")
local SubLeveling = CreateSubTab(TabFarm, "Leveling")
local SecAutoFarmMain = CreateSection(SubLeveling, "Farm Controls")
CreateToggle(SecAutoFarmMain, "Auto Farm (1v1 Hover Mode)", RyuConfig.AutoFarm, function(state) RyuConfig.AutoFarm = state end)
CreateToggle(SecAutoFarmMain, "Auto Quest", RyuConfig.AutoQuest, function(state) RyuConfig.AutoQuest = state end)
local SecAutoFarmConfig = CreateSection(SubLeveling, "Farm Configuration")
CreateDropdown(SecAutoFarmConfig, "Select Weapon", GPOWeapons, "TargetWeapon")
CreateDropdown(SecAutoFarmConfig, "Select Enemy", DynamicEnemies, "TargetMob")
CreateDropdown(SecAutoFarmConfig, "Select Quest NPC", DynamicQuests, "TargetNPC")

--// ============================================================================
--// MODULE HOOKING: PC COMBAT ENGINE
--// ============================================================================
local function GetInputCallbacks()
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if backpack and backpack:FindFirstChild("InputCallbacks") then return require(backpack.InputCallbacks) end
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("InputCallbacks") then return require(char.InputCallbacks) end
    return nil
end

--// ============================================================================
--// ANTI-TP TWEEN ENGINE
--// ============================================================================
local function CustomSafeTween(targetCFrame)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    RyuConfig.IsTweening = true 
    local startPos = root.Position
    local targetPos = targetCFrame.Position
    local totalDist = (targetPos - startPos).Magnitude
    
    -- STRENGES BYPASS-LIMIT: 60 Studs/Sekunde (Server denkt, du rennst/dashest legal)
    local speed = 60 
    local finalTime = totalDist / speed
    if finalTime < 0.1 then finalTime = 0.1 end

    local tweenInfo = TweenInfo.new(finalTime, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(root, tweenInfo, {CFrame = targetCFrame})
    
    tween:Play()
    tween.Completed:Wait()

    if root then 
        root.Velocity = Vector3.new(0, 0, 0)
    end
    RyuConfig.IsTweening = false 
end

--// NOCLIP ENGINE (Stellt sicher, dass wir durch Wände zum Gegner gleiten können)
RunService.Stepped:Connect(function()
    if RyuConfig.AutoFarm or RyuConfig.IsTweening then
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

--// ============================================================================
--// NEUE 1V1 HOVER FARM ENGINE (100% DAMAGE, KEIN GHOST HIT)
--// ============================================================================
task.spawn(function()
    while true do
        task.wait(0.1)
        
        -- 1. Auto Quest Loop
        if RyuConfig.AutoQuest and RyuConfig.TargetNPC and RyuConfig.TargetNPC ~= "" then
            local playerQuestData = LocalPlayer:FindFirstChild("Quest")
            local currentQuest = playerQuestData and playerQuestData:FindFirstChild("CurrentQuest") and playerQuestData.CurrentQuest.Value or "None"
            
            if currentQuest == "None" or currentQuest == "" then
                local npcTarget = Workspace:FindFirstChild(RyuConfig.TargetNPC, true)
                if npcTarget then
                    local npcPos = npcTarget:IsA("Model") and npcTarget:GetPivot() or npcTarget.CFrame
                    CustomSafeTween(npcPos * CFrame.new(0, 0, 4))
                    task.wait(0.5)
                    local questEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("Quest")
                    if questEvent then 
                        pcall(function() questEvent:InvokeServer("getNPCQuestLocations") end)
                        task.wait(0.2)
                        pcall(function() questEvent:InvokeServer({{"npcChat", true}}) end)
                        task.wait(0.2)
                        pcall(function() questEvent:InvokeServer("takequest") end)
                    end
                end
            end
        end

        -- 2. Auto Farm Loop (1v1 Hover)
        if RyuConfig.AutoFarm and RyuConfig.TargetMob and RyuConfig.TargetMob ~= "" then
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            if root and hum and hum.Health > 0 then
                local npcs = Workspace:FindFirstChild("NPCs")
                if npcs then
                    -- Finde den NÄCHSTEN, LEBENDIGEN Gegner mit exaktem Namen
                    local targetMobObj = nil
                    local shortestDist = math.huge
                    
                    for _, npc in pairs(npcs:GetChildren()) do
                        if npc.Name == RyuConfig.TargetMob then
                            local mobHum = npc:FindFirstChildOfClass("Humanoid")
                            local mobRoot = npc:FindFirstChild("HumanoidRootPart")
                            if mobHum and mobRoot and mobHum.Health > 0 then
                                local dist = (mobRoot.Position - root.Position).Magnitude
                                if dist < shortestDist then
                                    shortestDist = dist
                                    targetMobObj = npc
                                end
                            end
                        end
                    end
                    
                    -- Greife diesen einen Gegner an, bis er stirbt
                    if targetMobObj then
                        local mobRoot = targetMobObj:FindFirstChild("HumanoidRootPart")
                        local mobHum = targetMobObj:FindFirstChildOfClass("Humanoid")
                        
                        if mobRoot and mobHum then
                            -- Waffe Ausrüsten (Combat Tool)
                            local combatTool = char:FindFirstChild(RyuConfig.TargetWeapon) or LocalPlayer.Backpack:FindFirstChild(RyuConfig.TargetWeapon)
                            if not combatTool then
                                for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
                                    if item:IsA("Tool") and (item.Name:lower():find("combat") or item.Name:lower():find("melee") or item:GetAttribute("MeleeTool")) then
                                        combatTool = item; break
                                    end
                                end
                            end
                            if combatTool and combatTool.Parent == LocalPlayer.Backpack then
                                hum:EquipTool(combatTool)
                                task.wait(0.2)
                            end

                            local inputModule = GetInputCallbacks()
                            
                            -- Sauberen Anflug machen (6 Studs ÜBER dem Gegner)
                            local hoverPos = mobRoot.Position + Vector3.new(0, RyuConfig.FarmOffset, 0)
                            CustomSafeTween(CFrame.new(hoverPos, mobRoot.Position))
                            
                            local timeout = tick()
                            local lastHealth = mobHum.Health

                            -- Kampf-Loop 1v1
                            while RyuConfig.AutoFarm and hum.Health > 0 and targetMobObj and targetMobObj.Parent and mobHum.Health > 0 do
                                -- Wenn der Mob keinen Damage kriegt (verbuggt ist), breche nach 8 Sekunden ab!
                                if mobHum.Health < lastHealth then
                                    lastHealth = mobHum.Health
                                    timeout = tick() -- Reset Timeout wenn Damage gemacht wurde
                                elseif tick() - timeout > 8 then 
                                    break 
                                end
                                
                                -- Minimaler Hitbox Buff (Y-Achse leicht strecken, damit wir von oben treffen)
                                -- Server akzeptiert dies als legal, weil X/Z unverändert bleiben!
                                if mobRoot.Size.Y < 8 then
                                    mobRoot.Size = Vector3.new(mobRoot.Size.X, 10, mobRoot.Size.Z)
                                    mobRoot.CanCollide = false
                                end
                                
                                -- Schwebe stabil über ihm und schaue nach unten
                                root.Velocity = Vector3.new(0, 0, 0)
                                local targetCFrame = CFrame.lookAt(mobRoot.Position + Vector3.new(0, RyuConfig.FarmOffset, 0), mobRoot.Position)
                                root.CFrame = targetCFrame
                                
                                -- Schlage ganz normal zu
                                pcall(function()
                                    if inputModule and inputModule.Utils.canAutoM1() then
                                        inputModule.Callbacks.Attack:PC_Activate()
                                    else
                                        VirtualUser:CaptureController()
                                        VirtualUser:ClickButton1(Vector2.new())
                                    end
                                end)
                                
                                task.wait(0.1) -- Humaner Rhythmus (Server registriert jeden Schlag sauber)
                            end
                            
                            -- Hitbox zurücksetzen wenn tot
                            if mobRoot then mobRoot.Size = Vector3.new(2, 2, 1); mobRoot.CanCollide = true end
                        end
                    end
                end
            end
        end
    end
end)

task.wait(0.5)
RyuNotify:Send("RYU HUB", "PC Exclusive Edition loaded! Safe 1v1 Hover Farm Active.", 4)
