--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION (FULL SPIDER TP + TP-CHECK BYPASS)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function() 
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end 
end)

for _, v in pairs(guiParent:GetChildren()) do 
    if v.Name == "RyuHubPremium" or v.Name == "RyuNotifications" then v:Destroy() end 
end

--// ANTI-ANNOYING MESSAGE HIDER
task.spawn(function()
    local pg = LocalPlayer:WaitForChild("PlayerGui", 10)
    if pg then
        pg.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
                task.delay(0.01, function()
                    if descendant.Parent and descendant.Text then
                        local txt = descendant.Text:lower()
                        if txt:match("cd") or txt:match("cooldown") or txt:match("climb") or txt:match("error") then
                            descendant.Visible = false
                            descendant:Destroy()
                        end
                    end
                end)
            end
        end)
    end
end)

--// FESTE GPO DATEN
local StaticGPO = {
    Sea1 = {
        Mobs = {
            "Bandit", "Bandit Boss", "Desert Bandit", "Lucid", "Corrupt Marine", 
            "Shell's Bandit", "Axe Hand Logan", "Krieg Pirate", "Star Clown", 
            "Monkey", "Gorilla King", "Saw Shark Pirate", "Saw Shark", "Fishman Karate User", 
            "Ryu", "Neptune", "Sky Bandit", "Castle Guard", "Head Guardian", "Enel"
        },
        Quests = {
            "Daph", "Ronny", "Robert", "Kevin", "Gozen", "Waby", "Vi", "Becky", 
            "Jenny", "Janny", "Vego", "Bibby", "Viva"
        },
        Islands = {
            "Town of Beginnings", "Sandora", "Shell's Town", "Island Of Zou", "Baratie", 
            "Orange Town", "Sphinx Island", "Arlong Park", "Kori Island", "Land of the Sky", 
            "Gravito's Fort", "Fishman Island", "Fishman Cave", "Marine Base G-1"
        }
    },
    Sea2 = {
        Mobs = {
            "Desert Kingdom Bandit", "Pharaoh Akshan", "Moria Guard", "Borj",
            "Ryuma", "Musashi", "Donmingo", "Lucy", "Pica", "Kraken"
        },
        Quests = {
            "Rovo Quest NPC", "Sashi Quest NPC", "Desert Kingdom Quest NPC", "Foro Quest NPC"
        },
        Islands = {
            "Rovo Island", "Desert Kingdom", "Sphinx Island", "Sashi Island", 
            "Reverse Mountain", "Foro Island", "Thriller Bark"
        }
    }
}

--// SEA DETECTOR
local function GetCurrentSeaData()
    local islandsFolder = Workspace:FindFirstChild("Islands") or Workspace:FindFirstChild("Locations")
    if islandsFolder then
        if islandsFolder:FindFirstChild("Rovo Island") or islandsFolder:FindFirstChild("Desert Kingdom") then
            return StaticGPO.Sea2
        end
    end
    return StaticGPO.Sea1
end

--// HYBRIDER SCANNER
local function GetDynamicLists()
    local mobsDict, questsDict, islandsDict, weaponsDict = {}, {}, {}, {}
    local mobs, quests, islands, weapons = {}, {}, {}, {}

    local currentSea = GetCurrentSeaData()
    for _, v in ipairs(currentSea.Mobs) do mobsDict[v] = true end
    for _, v in ipairs(currentSea.Quests) do questsDict[v] = true end

    local npcsFolder = Workspace:FindFirstChild("NPCs") or Workspace:FindFirstChild("Live")
    if npcsFolder then
        for _, v in pairs(npcsFolder:GetChildren()) do
            if v:IsA("Model") and v ~= LocalPlayer.Character then
                local hum = v:FindFirstChildOfClass("Humanoid")
                if hum then
                    local isQuest = v:FindFirstChild("QuestMark") or v:FindFirstChild("Quest") or (v:FindFirstChild("Head") and v.Head:FindFirstChild("Quest"))
                    if isQuest then
                        questsDict[v.Name] = true
                    else
                        mobsDict[v.Name] = true
                    end
                end
            end
        end
    end

    local islandsFolder = Workspace:FindFirstChild("Islands") or Workspace:FindFirstChild("Locations")
    if islandsFolder then
        for _, v in pairs(islandsFolder:GetChildren()) do
            islandsDict[v.Name] = true
        end
    end
    
    if next(islandsDict) == nil then
        for _, v in ipairs(currentSea.Islands) do
            islandsDict[v] = true
        end
    end

    local function scanTools(container)
        if not container then return end
        for _, item in pairs(container:GetChildren()) do
            if item:IsA("Tool") then
                weaponsDict[item.Name] = true
            elseif item:IsA("Folder") then
                scanTools(item)
            end
        end
    end

    scanTools(LocalPlayer:FindFirstChild("Backpack"))
    scanTools(LocalPlayer.Character)
    if next(weaponsDict) == nil then weaponsDict["Combat"] = true end

    for k in pairs(mobsDict) do table.insert(mobs, k) end
    for k in pairs(questsDict) do table.insert(quests, k) end
    for k in pairs(islandsDict) do table.insert(islands, k) end
    for k in pairs(weaponsDict) do table.insert(weapons, k) end

    table.sort(mobs)
    table.sort(quests)
    table.sort(islands)
    table.sort(weapons)

    return mobs, quests, islands, weapons
end

local InitMobs, InitQuests, InitIslands, InitWeapons = GetDynamicLists()

--// RYU CONFIGURATION
local RyuConfig = {
    AutoFarm = false,
    AutoQuest = false,
    
    TargetMob = InitMobs[1], 
    TargetNPC = InitQuests[1],               
    TargetWeapon = InitWeapons[1],           
    
    TweenSpeed = 45, -- Sicherer Speed-Wert unter Anticheat-Limit
    KillHeight = 8, 
    FishmanSpeed = 65, 
    
    TargetIsland = InitIslands[1],
    IslandSpeed = 50, 
    
    AutoStrength = false, StrengthCap = 1500,
    AutoStamina = false,  StaminaCap = 1500,
    AutoDefense = false,  DefenseCap = 1500,
    AutoSword = false,    SwordCap = 1500,
    AutoGun = false,      GunCap = 1500
}

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
                task.wait(0.1) 
            end
        end)
    end
end
if LocalPlayer.Character then AddRainbowTag(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(AddRainbowTag)

--// UI SETUP
local Theme = { Background = Color3.fromRGB(12, 12, 14), Sidebar = Color3.fromRGB(18, 18, 20), SectionBG = Color3.fromRGB(24, 24, 26), Text = Color3.fromRGB(250, 250, 250), SubText = Color3.fromRGB(130, 130, 135), Accent = Color3.fromRGB(255, 255, 255), ToggleOff = Color3.fromRGB(35, 35, 38), ToggleOn = Color3.fromRGB(255, 255, 255), Stroke = Color3.fromRGB(45, 45, 50) }
local currentMainSize = UDim2.new(0, 550, 0, 380) 
local SidebarWidth = 150

local RyuHub = Instance.new("ScreenGui"); RyuHub.Name = "RyuHubPremium"; RyuHub.ResetOnSpawn = false; RyuHub.IgnoreGuiInset = true; RyuHub.Parent = guiParent
local MainFrame = Instance.new("Frame"); MainFrame.Size = currentMainSize; MainFrame.Position = UDim2.new(0.5, -currentMainSize.X.Offset/2, 0.5, -currentMainSize.Y.Offset/2); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.Active = true; MainFrame.Parent = RyuHub; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextColor3 = Theme.Text; Title.TextXAlignment = Enum.TextXAlignment.Left

local ResizeGrip = Instance.new("TextButton", MainFrame); ResizeGrip.Size = UDim2.new(0, 20, 0, 20); ResizeGrip.Position = UDim2.new(1, -20, 1, -20); ResizeGrip.BackgroundTransparency = 1; ResizeGrip.Text = "◢"; ResizeGrip.TextColor3 = Theme.SubText; ResizeGrip.TextSize = 16; ResizeGrip.Font = Enum.Font.GothamBold

local CloseBtn = Instance.new("TextButton", Topbar); CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG; CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.SubText; CloseBtn.Font = Enum.Font.GothamBold; Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

local ToggleBtn = Instance.new("TextButton"); ToggleBtn.Size = UDim2.new(0, 50, 0, 50); ToggleBtn.Position = UDim2.new(0, 25, 0, 25); ToggleBtn.BackgroundColor3 = Theme.Sidebar; ToggleBtn.Text = "R"; ToggleBtn.Font = Enum.Font.GothamBlack; ToggleBtn.TextColor3 = Theme.Accent; ToggleBtn.TextSize = 20; ToggleBtn.Parent = RyuHub; Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0); Instance.new("UIStroke", ToggleBtn).Color = Theme.Accent; Instance.new("UIStroke", ToggleBtn).Thickness = 2

local tDragStart, tStartPos, isDraggingBtn = nil, nil, false
ToggleBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then isDraggingBtn = false; tDragStart = input.Position; tStartPos = ToggleBtn.Position end end)
UserInputService.InputChanged:Connect(function(input) if tDragStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - tDragStart; if delta.Magnitude > 5 then isDraggingBtn = true; ToggleBtn.Position = UDim2.new(tStartPos.X.Scale, tStartPos.X.Offset + delta.X, tStartPos.Y.Scale, tStartPos.Y.Offset + delta.Y) end end end)

local rDragging, rDragStart, rStartSize = false, nil, nil
ResizeGrip.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then rDragging = true; rDragStart = input.Position; rStartSize = MainFrame.AbsoluteSize end end)
UserInputService.InputChanged:Connect(function(input) if rDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - rDragStart; currentMainSize = UDim2.new(0, math.max(450, rStartSize.X + delta.X), 0, math.max(300, rStartSize.Y + delta.Y)); MainFrame.Size = currentMainSize end end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if tDragStart then
            if not isDraggingBtn then
                if MainFrame.Visible then 
                    TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play(); task.wait(0.25); MainFrame.Visible = false
                else 
                    MainFrame.Visible = true; TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = currentMainSize, Position = UDim2.new(0.5, -currentMainSize.X.Offset/2, 0.5, -currentMainSize.Y.Offset/2)}):Play() 
                end
            end
            tDragStart = nil
        end
        rDragging = false
    end
end)

CloseBtn.Activated:Connect(function() TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play(); task.wait(0.25); MainFrame.Visible = false end)

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end end)
Topbar.InputChanged:Connect(function(input) if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - mDragStart; MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y) end end)
Topbar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end end)

local ContentContainer = Instance.new("Frame", MainFrame); ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1
local Sidebar = Instance.new("ScrollingFrame", MainFrame); Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar); SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; SideLayout.SortOrder = Enum.SortOrder.LayoutOrder

local Tabs = {}
local function UpdateSidebarCanvas()
    local totalH = 10
    for _, t in pairs(Tabs) do totalH = totalH + 36 + 6; if t.IsOpen then totalH = totalH + t.SubLayout.AbsoluteContentSize.Y + 6 end end
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH)
end

local function CreateMainTab(name)
    local tabObj = { Btn = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {} }
    local tabBtn = Instance.new("TextButton", Sidebar); tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name); tabBtn.TextColor3 = Theme.SubText; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 13; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8); tabObj.Btn = tabBtn
    local subContainer = Instance.new("Frame", Sidebar); subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer); subLayout.Padding = UDim.new(0, 2); tabObj.SubLayout = subLayout
    tabBtn.Activated:Connect(function() tabObj.IsOpen = not tabObj.IsOpen; TweenService:Create(subContainer, TweenInfo.new(0.25), {Size = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)}):Play(); tabBtn.TextColor3 = tabObj.IsOpen and Theme.Text or Theme.SubText; task.delay(0.26, UpdateSidebarCanvas); UpdateSidebarCanvas() end)
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end; UpdateSidebarCanvas() end)
    table.insert(Tabs, tabObj); return tabObj
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
    
    local function populate(list)
        for _, child in pairs(scroll:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        for _, itemName in ipairs(list) do
            local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. tostring(itemName); btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
            btn.Activated:Connect(function() RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. tostring(itemName) end)
        end
        task.defer(function()
            scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10)
        end)
    end
    
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
    populate(itemsList)
    
    return { Refresh = populate }
end

local function CreateSlider(section, text, min, max, default, callback)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1
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

local function CreateButton(section, text, callback)
    local btn = Instance.new("TextButton", section); btn.Size = UDim2.new(0.92, 0, 0, 34); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = text; btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 13; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    local stroke = Instance.new("UIStroke", btn); stroke.Color = Theme.Stroke; stroke.Thickness = 1
    btn.Activated:Connect(function() if callback then callback() end end)
end

--// ============================================================================
--// ANTI-FLING & STABILE HOVER LOGIK
--// ============================================================================
local function ToggleHover(state)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    if state then
        root.Anchored = false
        local bp = root:FindFirstChild("RyuHover")
        if not bp then
            bp = Instance.new("BodyPosition")
            bp.Name = "RyuHover"
            bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.D = 800
            bp.P = 15000
            bp.Parent = root
        end
        bp.Position = root.Position
    else
        root.Anchored = false
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp:Destroy() end
    end
end

--// UI AUFBAU
local TabFarm = CreateMainTab("Farm")
local SubLeveling = CreateSubTab(TabFarm, "Leveling")
local SubConfig = CreateSubTab(TabFarm, "Config")
local SubStats = CreateSubTab(TabFarm, "Stats") 

local SecAutoFarmMain = CreateSection(SubLeveling, "Auto Farm")
CreateToggle(SecAutoFarmMain, "Enable Auto Farm", RyuConfig.AutoFarm, function(state) 
    RyuConfig.AutoFarm = state 
    if not state then ToggleHover(false) end 
end)
CreateToggle(SecAutoFarmMain, "Auto Quest Link", RyuConfig.AutoQuest, function(state) 
    RyuConfig.AutoQuest = state 
end)

local SecFarmAdvanced = CreateSection(SubLeveling, "Advanced Options")
CreateSlider(SecFarmAdvanced, "Movement Speed (Tween)", 10, 65, RyuConfig.TweenSpeed, function(val) 
    RyuConfig.TweenSpeed = val 
end)
CreateSlider(SecFarmAdvanced, "Kill Height Offset", -20, 30, RyuConfig.KillHeight, function(val) 
    RyuConfig.KillHeight = val 
end)

local DropMob, DropNPC, DropWep, DropIsland

local SecFarmConfig = CreateSection(SubConfig, "Farm Config")
DropMob = CreateDropdown(SecFarmConfig, "Select Mob", InitMobs, "TargetMob")
DropNPC = CreateDropdown(SecFarmConfig, "Select Quest NPC", InitQuests, "TargetNPC")
DropWep = CreateDropdown(SecFarmConfig, "Select Weapon", InitWeapons, "TargetWeapon")

CreateButton(SecFarmConfig, "Refresh All Lists", function()
    local newMobs, newQuests, newIslands, newWeaps = GetDynamicLists()
    if DropMob then DropMob:Refresh(newMobs) end
    if DropNPC then DropNPC:Refresh(newQuests) end
    if DropWep then DropWep:Refresh(newWeaps) end
    if DropIsland then DropIsland:Refresh(newIslands) end
    RyuNotify:Send("Lists Refreshed", "Listen manuell aktualisiert!", 3)
end)

-- AUTO REFRESH LOOP
task.spawn(function()
    local function listsEqual(a, b)
        if #a ~= #b then return false end
        for i = 1, #a do if a[i] ~= b[i] then return false end end
        return true
    end
    local lastMobs, lastQuests, lastIslands, lastWeaps = InitMobs, InitQuests, InitIslands, InitWeapons
    while true do
        task.wait(3)
        local newMobs, newQuests, newIslands, newWeaps = GetDynamicLists()
        if not listsEqual(lastMobs, newMobs) then
            lastMobs = newMobs
            if DropMob then DropMob:Refresh(newMobs) end
        end
        if not listsEqual(lastQuests, newQuests) then
            lastQuests = newQuests
            if DropNPC then DropNPC:Refresh(newQuests) end
        end
        if not listsEqual(lastWeaps, newWeaps) then
            lastWeaps = newWeaps
            if DropWep then DropWep:Refresh(newWeaps) end
        end
        if not listsEqual(lastIslands, newIslands) then
            lastIslands = newIslands
            if DropIsland then DropIsland:Refresh(newIslands) end
        end
    end
end)

-- AUTO STATS
local SecAutoStats = CreateSection(SubStats, "Auto Stats System")
CreateToggle(SecAutoStats, "Auto Strength", RyuConfig.AutoStrength, function(state) RyuConfig.AutoStrength = state end)
CreateSlider(SecAutoStats, "Strength Cap", 1, 2000, 1500, function(val) RyuConfig.StrengthCap = val end)
CreateToggle(SecAutoStats, "Auto Stamina", RyuConfig.AutoStamina, function(state) RyuConfig.AutoStamina = state end)
CreateSlider(SecAutoStats, "Stamina Cap", 1, 2000, 1500, function(val) RyuConfig.StaminaCap = val end)
CreateToggle(SecAutoStats, "Auto Defense", RyuConfig.AutoDefense, function(state) RyuConfig.AutoDefense = state end)
CreateSlider(SecAutoStats, "Defense Cap", 1, 2000, 1500, function(val) RyuConfig.DefenseCap = val end)
CreateToggle(SecAutoStats, "Auto Sword", RyuConfig.AutoSword, function(state) RyuConfig.AutoSword = state end)
CreateSlider(SecAutoStats, "Sword Cap", 1, 2000, 1500, function(val) RyuConfig.SwordCap = val end)
CreateToggle(SecAutoStats, "Auto Gun", RyuConfig.AutoGun, function(state) RyuConfig.AutoGun = state end)
CreateSlider(SecAutoStats, "Gun Cap", 1, 2000, 1500, function(val) RyuConfig.GunCap = val end)

--// MOBILITY TAB
local TabMobility = CreateMainTab("Mobility")
SubTransport = CreateSubTab(TabMobility, "Spider TP")

local SecIslandTP = CreateSection(SubTransport, "Spider Teleportation")
DropIsland = CreateDropdown(SecIslandTP, "Select Island", InitIslands, "TargetIsland")
CreateSlider(SecIslandTP, "Travel Speed", 10, 65, RyuConfig.IslandSpeed, function(val)
    RyuConfig.IslandSpeed = val
end)

--// ============================================================================
--// VOLLSTÄNDIGER, ORIGINALER SPIDER-TP ENGINEMANAGER (MIT CLIMB-EVENTS & RAYCAST)
--// ============================================================================
local function FireClimbEvents()
    pcall(function()
        local eventsFolder = ReplicatedStorage:FindFirstChild("Events")
        if eventsFolder and eventsFolder:FindFirstChild("Climb") then
            eventsFolder.Climb:FireServer()
        end
    end)
end

local function FullSpiderLerp(targetCFrame, travelSpeed)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return false end

    local startPos = root.Position
    local targetPos = targetCFrame.Position
    local totalDist = (targetPos - startPos).Magnitude

    if totalDist < 5 then
        root.CFrame = targetCFrame
        return true
    end

    ToggleHover(true)

    -- 1. Sichere Flug-Höhe berechnen (Raycast & Sky Check)
    local highY = math.max(startPos.Y, targetPos.Y) + 35
    local ray = Ray.new(Vector3.new(startPos.X, highY, startPos.Z), Vector3.new(0, 150, 0))
    local hit = Workspace:FindPartOnRayWithIgnoreList(ray, {char})
    if hit then highY = startPos.Y + 20 end

    local waypoints = {
        Vector3.new(startPos.X, highY, startPos.Z),
        Vector3.new(targetPos.X, highY, targetPos.Z),
        targetPos
    }

    local currentPos = startPos
    for idx, nextPoint in ipairs(waypoints) do
        local segmentDist = (nextPoint - currentPos).Magnitude
        if segmentDist > 1 then
            local duration = segmentDist / travelSpeed
            local elapsed = 0
            
            while elapsed < duration do
                if not _G.RyuIsTweening and not RyuConfig.AutoFarm then break end
                local dt = RunService.Heartbeat:Wait()
                elapsed = elapsed + dt
                local alpha = math.clamp(elapsed / duration, 0, 1)
                
                -- Anticheat Distance Capping (Max 25 Studs pro Step -> Kein TP Check)
                local nextStep = currentPos:Lerp(nextPoint, alpha)
                if (nextStep - root.Position).Magnitude > 25 then
                    nextStep = root.Position + (nextStep - root.Position).Unit * 25
                end
                
                root.CFrame = CFrame.new(nextStep) * targetCFrame.Rotation
                FireClimbEvents()
            end
            currentPos = nextPoint
        end
    end

    -- Robo-Landing Safety Check
    local landingRay = Ray.new(targetPos + Vector3.new(0, 10, 0), Vector3.new(0, -50, 0))
    local landHit, landPoint = Workspace:FindPartOnRayWithIgnoreList(landingRay, {char})
    if landHit then
        root.CFrame = CFrame.new(landPoint + Vector3.new(0, 3.5, 0)) * targetCFrame.Rotation
    else
        root.CFrame = targetCFrame
    end

    ToggleHover(false)
    return true
end

CreateButton(SecIslandTP, "Start Spider TP", function()
    if _G.RyuIsTweening then return end
    _G.RyuIsTweening = true
    
    task.spawn(function()
        local targetIslandName = RyuConfig.TargetIsland
        local island = Workspace:FindFirstChild(targetIslandName, true)
        
        if not island then 
            RyuNotify:Send("Spider TP Error", "Insel nicht gefunden!", 3)
            _G.RyuIsTweening = false
            return 
        end
        
        local targetCFrame
        pcall(function()
            if island:IsA("Model") then
                targetCFrame = island:GetPivot()
            elseif island:IsA("BasePart") then
                targetCFrame = island.CFrame
            end
        end)
        
        if targetCFrame then
            RyuNotify:Send("Spider TP", "Reise nach " .. targetIslandName, 3)
            FullSpiderLerp(targetCFrame, RyuConfig.IslandSpeed)
            RyuNotify:Send("Spider TP", "Angekommen!", 3)
        end
        
        _G.RyuIsTweening = false
    end)
end)

--// ============================================================================
--// COMBAT & AGGRO MODULES
--// ============================================================================
local currentComboIndex = 1
local lastSwing = 0

local function EquipTargetWeapon()
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    
    local targetWep = RyuConfig.TargetWeapon
    
    pcall(function()
        local unequiped = LocalPlayer.Backpack:FindFirstChild("Unequiped")
        if unequiped and unequiped:FindFirstChild(targetWep) then
            ReplicatedStorage.Events.Tools:InvokeServer("equip", targetWep)
            task.wait(0.1)
        end
    end)
    
    if char:FindFirstChild(targetWep) then return true end
    for _, item in pairs(char:GetChildren()) do
        if item:IsA("Tool") and (item.Name:lower():find(targetWep:lower()) or item:GetAttribute("MeleeTool")) then
            return true 
        end
    end
    
    local tool = LocalPlayer.Backpack:FindFirstChild(targetWep)
    if not tool then
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
            if item:IsA("Tool") then tool = item; break end
        end
    end
    
    if tool and tool.Parent == LocalPlayer.Backpack then
        hum:EquipTool(tool)
        task.wait(0.1)
        return true
    end
    return false
end

local function PerformMeleeAttack(targets)
    pcall(function()
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local now = tick()
        if now - lastSwing >= 0.15 then
            lastSwing = now
            task.spawn(function()
                local hitParts = {}
                
                if type(targets) == "table" then
                    for _, npc in ipairs(targets) do
                        local mRoot = npc:FindFirstChild("HumanoidRootPart")
                        local mHum = npc:FindFirstChildOfClass("Humanoid")
                        if mRoot and mHum and mHum.Health > 0 then
                            table.insert(hitParts, mRoot)
                        end
                    end
                end
                
                local animName = "Punch" .. currentComboIndex
                if currentComboIndex == 1 then animName = "Dash" end
                if currentComboIndex == 4 then animName = "GroundPunch4" end
                
                local animObj = ReplicatedStorage:FindFirstChild("CombatAnimations") 
                    and ReplicatedStorage.CombatAnimations:FindFirstChild("Melee")
                    and ReplicatedStorage.CombatAnimations.Melee:FindFirstChild(animName)
                
                if animObj then
                    local argsAnim = {
                        "swingsfx", "Melee", currentComboIndex, "Ground", currentComboIndex == 1, animObj, 2, 1.5
                    }
                    ReplicatedStorage.Events.CombatRegister:InvokeServer(argsAnim)
                end
                
                if #hitParts > 0 then
                    local argsDamage = {
                        "damage", hitParts, "Melee", {currentComboIndex, "Ground", "Melee"}, true, root.CFrame, ["aircombo"] = "Ground"
                    }
                    ReplicatedStorage.Events.CombatRegister:InvokeServer(argsDamage)
                end
                
                currentComboIndex = currentComboIndex + 1
                if currentComboIndex > 4 then currentComboIndex = 1 end
            end)
        end
    end)
end

-- ANTI-CHEAT SAFE TWEEN FOR AUTO FARM (KEIN TP CHECK MEHR)
local function SafeTween(targetCFrame, customSpeed)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local startPos = root.Position
    local targetPos = targetCFrame.Position
    local dist = (targetPos - startPos).Magnitude
    
    local speed = customSpeed or RyuConfig.TweenSpeed 
    local timeToTake = dist / speed
    
    if timeToTake < 0.05 then 
        root.CFrame = targetCFrame
        return 
    end

    local startTime = tick()
    local lastPos = startPos
    
    while tick() - startTime < timeToTake do
        if not RyuConfig.AutoFarm and not _G.RyuIsTweening then break end
        local alpha = (tick() - startTime) / timeToTake
        local intermediatePos = startPos:Lerp(targetPos, alpha)
        
        -- Anticheat Step Cap (verhindert 'Tp Check Threshold: 36')
        if (intermediatePos - lastPos).Magnitude > 20 then
            intermediatePos = lastPos + (intermediatePos - lastPos).Unit * 20
        end
        lastPos = intermediatePos
        
        root.CFrame = CFrame.new(intermediatePos) * targetCFrame.Rotation
        FireClimbEvents()
        RunService.Heartbeat:Wait()
    end
    root.CFrame = targetCFrame
end

-- Anti-Fling & Noclip Loop
RunService.Stepped:Connect(function()
    if RyuConfig.Noclip or RyuConfig.AutoFarm then
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum and RyuConfig.AutoFarm then hum.AutoRotate = false end
            for _, v in pairs(char:GetChildren()) do
                if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" and v.CanCollide then 
                    v.CanCollide = false 
                end
            end
        end
    end
end)

--// ============================================================================
--// QUEST ENGINE & STATE MACHINE
--// ============================================================================
local function CheckQuestActive()
    local active = false
    pcall(function()
        local qFolder = LocalPlayer:FindFirstChild("Quest")
        if qFolder and qFolder:FindFirstChild("CurrentQuest") then
            local val = qFolder.CurrentQuest.Value
            if val and val ~= "" and val ~= "None" then 
                active = true 
            end
        end
        
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg then
            for _, v in pairs(pg:GetDescendants()) do
                if v:IsA("TextLabel") and v.Visible then
                    local txt = v.Text:lower()
                    if txt:find("completed") then
                        active = false
                        return
                    end
                    if not active and v.AbsolutePosition.X < 500 and v.AbsolutePosition.Y < 500 then
                        if txt:match("%d+/%d+") or txt:match("%d+%s*/%s*%d+") then
                            active = true
                        end
                    end
                end
            end
        end
    end)
    return active
end

local function FetchQuest()
    local targetNpcName = RyuConfig.TargetNPC
    if not targetNpcName or targetNpcName == "" or targetNpcName == "None" then return false end
    
    local npc = Workspace:FindFirstChild(targetNpcName, true)
    if npc then
        RyuNotify:Send("Auto Quest", "Fliege zu NPC: " .. targetNpcName, 2)
        local npcPos = npc:IsA("Model") and npc:GetPivot() or npc.CFrame
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            local targetPos = npcPos.Position + Vector3.new(0, 3, 3)
            local targetCFrame = CFrame.lookAt(targetPos, Vector3.new(npcPos.Position.X, targetPos.Y, npcPos.Position.Z))
            
            SafeTween(targetCFrame)
            root.CFrame = targetCFrame
            task.wait(0.2)
            
            pcall(function()
                local QuestEvent = ReplicatedStorage.Events.Quest
                QuestEvent:InvokeServer({"npcChat", true})
                QuestEvent:InvokeServer({"takequest", "Help " .. targetNpcName})
                QuestEvent:InvokeServer({"takequest", targetNpcName})
                QuestEvent:InvokeServer({"takequest"})
                QuestEvent:InvokeServer("takequest")
                QuestEvent:InvokeServer({"acceptquest"})
                QuestEvent:InvokeServer("acceptquest")
            end)
            task.wait(0.8)
            return CheckQuestActive()
        end
    end
    return false
end

-- AUTO STATS LOOP
task.spawn(function()
    while true do
        task.wait(3) 
        local function getStatVal(name)
            local statsFolder = LocalPlayer:FindFirstChild("Stats") or LocalPlayer:FindFirstChild("Data")
            if statsFolder and statsFolder:FindFirstChild(name) then return statsFolder[name].Value end
            return 0 
        end
        
        local function upgradeStat(statName, cap)
            if getStatVal(statName) < cap then
                for i = 1, 5 do 
                    pcall(function()
                        ReplicatedStorage:WaitForChild("Events"):WaitForChild("stats"):FireServer(statName, nil, 1)
                    end)
                end
            end
        end

        if RyuConfig.AutoStrength then upgradeStat("Strength", RyuConfig.StrengthCap) end
        if RyuConfig.AutoStamina then upgradeStat("Stamina", RyuConfig.StaminaCap) end
        if RyuConfig.AutoDefense then upgradeStat("Defense", RyuConfig.DefenseCap) end
        if RyuConfig.AutoSword then upgradeStat("SwordMastery", RyuConfig.SwordCap) end
        if RyuConfig.AutoGun then upgradeStat("GunMastery", RyuConfig.GunCap) end
    end
end)

--// ============================================================================
--// CORE AUTO-FARM ENGINE (SAFE AGGRO + AOE MULTI-KILL)
--// ============================================================================
local function GetTargetMobs()
    local npcs = Workspace:FindFirstChild("NPCs") or Workspace:FindFirstChild("Live")
    local mobs = {}
    if npcs and RyuConfig.TargetMob ~= "" then
        for _, npc in pairs(npcs:GetChildren()) do
            if npc.Name == RyuConfig.TargetMob then
                local mHum = npc:FindFirstChildOfClass("Humanoid")
                local mRoot = npc:FindFirstChild("HumanoidRootPart")
                local isRagdolled = npc:FindFirstChild("Rag") or (npc.Parent and npc.Parent.Name == "Ragdolls")
                if mHum and mRoot and mHum.Health > 0 and not isRagdolled then
                    table.insert(mobs, npc)
                end
            end
        end
    end
    return mobs
end

-- MAIN FARM LOOP
task.spawn(function()
    while true do
        task.wait(0.05)
        
        if not RyuConfig.AutoFarm then
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then 
                hum.AutoRotate = true 
                if char:FindFirstChild("HumanoidRootPart") then
                    char.HumanoidRootPart.Anchored = false
                end
            end
            continue
        end

        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        
        if not root or not hum or hum.Health <= 0 then continue end
        hum.AutoRotate = false 

        -- 1. QUEST AUTOMATION
        if RyuConfig.AutoQuest then
            if not CheckQuestActive() then
                root.Anchored = false
                ToggleHover(true)
                FetchQuest()
                task.wait(0.5)
                if not CheckQuestActive() then
                    continue
                end
            end
        end

        -- 2. TARGET SCANNING & SAFE AGGRO PULL
        local targetMobs = GetTargetMobs()
        
        if #targetMobs > 0 then
            root.Anchored = false
            ToggleHover(true)
            EquipTargetWeapon()

            -- AGGRO PHASE (Sicherer Flug zu jedem Mob ohne TP Check)
            RyuNotify:Send("Auto Farm", "Ziehe Aggro von " .. #targetMobs .. " Mobs...", 1.5)
            for _, mob in ipairs(targetMobs) do
                if not RyuConfig.AutoFarm or (RyuConfig.AutoQuest and not CheckQuestActive()) then break end
                
                local mRoot = mob:FindFirstChild("HumanoidRootPart")
                local mHum = mob:FindFirstChildOfClass("Humanoid")
                if mRoot and mHum and mHum.Health > 0 then
                    local hitPos = mRoot.Position + Vector3.new(0, 2, 2)
                    local safeTargetCFrame = CFrame.lookAt(hitPos, mRoot.Position)
                    
                    -- Sanfter Anflug statt Sprung-TP
                    SafeTween(safeTargetCFrame, RyuConfig.TweenSpeed)
                    PerformMeleeAttack({mob})
                    task.wait(0.12)
                end
            end

            -- STACKING & AOE KILL PHASE
            local centerPos = Vector3.new(0, 0, 0)
            local validCount = 0
            for _, mob in ipairs(targetMobs) do
                local mRoot = mob:FindFirstChild("HumanoidRootPart")
                if mRoot then
                    centerPos = centerPos + mRoot.Position
                    validCount = validCount + 1
                end
            end

            if validCount > 0 then
                centerPos = centerPos / validCount
                local attackPos = centerPos + Vector3.new(0, RyuConfig.KillHeight, 0)
                local attackCFrame = CFrame.new(attackPos)

                -- Fliege über die Mob-Mitte
                SafeTween(attackCFrame, RyuConfig.TweenSpeed)
                root.CFrame = attackCFrame
                root.Anchored = true -- Stabilisieren über den Mobs

                -- Multi-Kill Loop
                local farmTimer = tick()
                while RyuConfig.AutoFarm and (tick() - farmTimer < 8) do
                    if RyuConfig.AutoQuest and not CheckQuestActive() then break end
                    
                    local activeMobs = GetTargetMobs()
                    if #activeMobs == 0 then break end
                    
                    for _, mob in ipairs(activeMobs) do
                        local mRoot = mob:FindFirstChild("HumanoidRootPart")
                        if mRoot then
                            mRoot.Size = Vector3.new(25, 25, 25)
                            mRoot.CanCollide = false
                            mRoot.Velocity = Vector3.new(0, 0, 0)
                        end
                    end

                    PerformMeleeAttack(activeMobs)
                    task.wait(0.08)
                end
                
                root.Anchored = false
            end
        else
            task.wait(0.5)
        end
    end
end)
