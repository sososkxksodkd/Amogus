--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION (NO DISTANCE LIMIT / NO ERRORS)
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

--// ANTI-ANNOYING MESSAGE HIDER (Löscht die nervige rote Schrift)
task.spawn(function()
    local pg = LocalPlayer:WaitForChild("PlayerGui", 10)
    if pg then
        pg.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
                task.delay(0.01, function()
                    if descendant.Parent and descendant.Text then
                        local txt = descendant.Text:lower()
                        -- "error" hinzugefügt, um generische GPO-Fehlermeldungen zu blockieren
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

--// BEREINIGTE NPC & ENEMY LISTEN
local DynamicEnemies = {"Bandit", "Bandit Boss", "Fishman", "Fishman Karate User"}
local DynamicQuests = {"Becky", "Daph", "Tyson", "Helen"}

--// INSEL LISTE AUS WORKSPACE
local IslandList = {
    "???? Shrine", "Coco Island", "Colosseum", "Fishman Cave", "Gravito's Fort", 
    "Island Of Zou", "Kori Island", "Land of the Sky", "Logue Town", "Marine Base G-1", 
    "Marine Fort F-1", "Mysterious Cliff", "Orange Town", "Restaurant Baratie", 
    "Reverse Mountain", "Roca Island", "Sandora", "Shark Park", "Shell's Town", 
    "Sphinx Island", "Town of Beginnings"
}

--// RYU CONFIGURATION
local RyuConfig = {
    AutoFarm = false,
    AutoQuest = false,
    QuestInterval = 45, 
    
    TargetMob = "Fishman Karate User", 
    TargetNPC = "Becky",               
    TargetWeapon = "Combat",           
    
    TweenSpeed = 50, 
    KillHeight = 5, 
    FishmanSpeed = 65, 
    ElevatorSpeed = 65, 
    AttackDelay = 0.9, -- 0.9 Sekunden Delay
    
    TargetIsland = IslandList[1],
    IslandSpeed = 60, 
    
    AutoStrength = false,
    AutoStamina = false,
    AutoDefense = false,
    AutoSword = false,
    AutoGun = false
}

local GPOWeapons = { "Combat", "Melee", "Sword", "Katana" }

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
    for _, itemName in ipairs(itemsList) do
        local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. itemName; btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
        btn.Activated:Connect(function() RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. itemName end)
    end
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
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
--// ANTI-FLING HOVER SYSTEM 
--// ============================================================================
local function ToggleHover(state)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    if state then
        local bp = root:FindFirstChild("RyuHover")
        if not bp then
            bp = Instance.new("BodyPosition")
            bp.Name = "RyuHover"
            bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.D = 500
            bp.P = 50000
            bp.Parent = root
        end
        bp.Position = root.Position
    else
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp:Destroy() end
    end
end

--// UI AUFBAU: FARM
local TabFarm = CreateMainTab("Farm")
local SubLeveling = CreateSubTab(TabFarm, "Leveling")
local SubStats = CreateSubTab(TabFarm, "Stats") 

local SecAutoFarmMain = CreateSection(SubLeveling, "FISHMAN CAVE FARM")
CreateToggle(SecAutoFarmMain, "Enable Auto Farm", RyuConfig.AutoFarm, function(state) 
    RyuConfig.AutoFarm = state 
    if not state then ToggleHover(false) end 
end)
CreateToggle(SecAutoFarmMain, "Auto Quest Link", RyuConfig.AutoQuest, function(state) 
    RyuConfig.AutoQuest = state 
end)
CreateSlider(SecAutoFarmMain, "Quest Interval (Secs)", 10, 100, RyuConfig.QuestInterval, function(val) 
    RyuConfig.QuestInterval = val 
end)

local SecFarmAdvanced = CreateSection(SubLeveling, "Advanced Options")
CreateSlider(SecFarmAdvanced, "Movement Speed (Tween)", 10, 85, RyuConfig.TweenSpeed, function(val) 
    RyuConfig.TweenSpeed = val 
end)
CreateSlider(SecFarmAdvanced, "Kill Height Offset", -20, 30, RyuConfig.KillHeight, function(val) 
    RyuConfig.KillHeight = val 
end)
CreateSlider(SecFarmAdvanced, "Attack Delay (5=0.5s, 10=1s)", 5, 10, 9, function(val) 
    RyuConfig.AttackDelay = val / 10 
end)

local SecMovement = CreateSection(SubLeveling, "Auto Farm")

local tpCaveLabel = Instance.new("TextLabel", SecMovement)
tpCaveLabel.Size = UDim2.new(0.92, 0, 0, 20)
tpCaveLabel.BackgroundTransparency = 1
tpCaveLabel.Text = "USE IT ONLY IF YOU ARE IN THE FISHMAN CAVE"
tpCaveLabel.TextColor3 = Theme.SubText
tpCaveLabel.Font = Enum.Font.GothamBold
tpCaveLabel.TextSize = 11
tpCaveLabel.TextXAlignment = Enum.TextXAlignment.Center

CreateButton(SecMovement, "Fishman Cave tp", function()
    task.spawn(function()
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        
        local areaTp = Workspace:FindFirstChild("AreaTeleporters")
        if areaTp and areaTp:FindFirstChild("FirstSea") and areaTp.FirstSea:FindFirstChild("Fishman") and areaTp.FirstSea.Fishman:FindFirstChild("Part") then
            local portal = areaTp.FirstSea.Fishman.Part
            
            if (root.Position - portal.Position).Magnitude > 500 then
                RyuNotify:Send("Error", "You must be closer to the Fishman Cave!", 3)
                return
            end
            
            RyuNotify:Send("Fishman Cave TP", "Simuliere Bewegung...", 2)
            local moveStart = tick()
            while tick() - moveStart < 2 do
                if hum and root then
                    hum:Move(root.CFrame.LookVector, false)
                end
                RunService.Heartbeat:Wait()
            end
            if hum then hum:Move(Vector3.new(0,0,0), false) end
            
            local tpSuccess = false
            local isBlack = false
            
            while not tpSuccess do
                root.Velocity = Vector3.new(0, 0, 0)
                root.CFrame = portal.CFrame * CFrame.new(0, 3, 0)
                
                local checkStart = tick()
                isBlack = false
                
                while tick() - checkStart < 4 do
                    if char and root and portal and (root.Position - portal.Position).Magnitude > 1000 then
                        tpSuccess = true
                        break
                    end
                    
                    if hum and root and portal and (root.Position - portal.Position).Magnitude < 50 then
                        hum:Move(Vector3.new(math.sin(tick() * 10), 0, math.cos(tick() * 10)))
                        local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
                        if footstepEvent then pcall(function() footstepEvent:FireServer() end) end
                    end
                    
                    local pg = LocalPlayer:FindFirstChild("PlayerGui")
                    if pg then
                        for _, v in pairs(pg:GetDescendants()) do
                            if v:IsA("Frame") and v.Visible and v.BackgroundTransparency <= 0.1 then
                                if v.BackgroundColor3 == Color3.new(0, 0, 0) and v.AbsoluteSize.X > 500 and v.AbsoluteSize.Y > 500 then
                                    isBlack = true
                                    tpSuccess = true
                                    break
                                end
                            end
                        end
                    end
                    
                    if tpSuccess then 
                        if hum then hum:Move(Vector3.new(0,0,0)) end
                        break 
                    end
                    task.wait(0.1)
                end
                
                if not tpSuccess then
                    if hum then hum:Move(Vector3.new(0,0,0)) end
                    task.wait(3)
                end
            end
            
            if isBlack then
                local startClear = tick()
                while tick() - startClear < 15 do
                    local foundBlack = false
                    local pg = LocalPlayer:FindFirstChild("PlayerGui")
                    if pg then
                        for _, v in pairs(pg:GetDescendants()) do
                            if v:IsA("Frame") and v.Visible and v.BackgroundTransparency <= 0.1 then
                                if v.BackgroundColor3 == Color3.new(0, 0, 0) and v.AbsoluteSize.X > 500 and v.AbsoluteSize.Y > 500 then
                                    foundBlack = true
                                    break
                                end
                            end
                        end
                    end
                    if not foundBlack then break end
                    task.wait(0.1)
                end
                task.wait(1)
            else
                task.wait(2)
            end
            
            RyuNotify:Send("Fishman Cave TP", "Erfolgreich angekommen! Du hast die Kontrolle.", 4)
        else
            RyuNotify:Send("Error", "Portal nicht gefunden!", 3)
        end
    end)
end)

--// AUTO STATS UI
local SecAutoStats = CreateSection(SubStats, "Auto Stats System")
CreateToggle(SecAutoStats, "Auto Strength", RyuConfig.AutoStrength, function(state) 
    RyuConfig.AutoStrength = state 
end)
CreateToggle(SecAutoStats, "Auto Stamina", RyuConfig.AutoStamina, function(state) 
    RyuConfig.AutoStamina = state 
end)
CreateToggle(SecAutoStats, "Auto Defense", RyuConfig.AutoDefense, function(state) 
    RyuConfig.AutoDefense = state 
end)
CreateToggle(SecAutoStats, "Auto Sword Mastery", RyuConfig.AutoSword, function(state) 
    RyuConfig.AutoSword = state 
end)
CreateToggle(SecAutoStats, "Auto Gun Mastery", RyuConfig.AutoGun, function(state) 
    RyuConfig.AutoGun = state 
end)

--// MOBILITY TAB -> TRANSPORTATION & AUTO BUY
local TabMobility = CreateMainTab("Mobility")
local SubTransport = CreateSubTab(TabMobility, "Spider TP")
local SubAutoBuy = CreateSubTab(TabMobility, "Auto Buy")

local SecIslandTP = CreateSection(SubTransport, "Spider Teleportation")
CreateDropdown(SecIslandTP, "Select Island", IslandList, "TargetIsland")
CreateSlider(SecIslandTP, "Travel Speed", 10, 65, RyuConfig.IslandSpeed, function(val)
    RyuConfig.IslandSpeed = val
end)

--// DEIN 100% EXAKT UNBERÜHRTES ORIGINAL-TRANSPORT-SYSTEM
CreateButton(SecIslandTP, "Start Spider TP", function()
    if _G.RyuIsTweening then return end
    _G.RyuIsTweening = true
    
    task.spawn(function()
        local targetIslandName = RyuConfig.TargetIsland
        
        local island = nil
        for _, v in pairs(Workspace:GetChildren()) do
            if string.lower(v.Name) == string.lower(targetIslandName) then
                island = v
                break
            end
        end
        if not island then
            for _, v in pairs(Workspace:GetDescendants()) do
                if string.lower(v.Name) == string.lower(targetIslandName) then
                    island = v
                    break
                end
            end
        end
        
        if not island then 
            _G.RyuIsTweening = false
            return 
        end
        
        local rawPos
        pcall(function()
            if island:IsA("Model") then
                rawPos = island:GetPivot().Position
            elseif island:IsA("BasePart") then
                rawPos = island.Position
            else
                local tpPart = island:FindFirstChildWhichIsA("BasePart", true)
                if tpPart then
                    rawPos = tpPart.Position
                end
            end
        end)
        
        if not rawPos then
            _G.RyuIsTweening = false
            return
        end
        
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then _G.RyuIsTweening = false return end

        local targetPos = rawPos
        local closestRobo = nil
        local closestDist = math.huge 
        
        local islandDistFromPlayer = (rawPos - root.Position).Magnitude
        
        for _, v in pairs(Workspace:GetDescendants()) do
            if v.Name == "Robo" and v:IsA("Model") and v:FindFirstChild("HumanoidRootPart") then
                local distToTarget = (v.HumanoidRootPart.Position - rawPos).Magnitude
                local distToPlayer = (v.HumanoidRootPart.Position - root.Position).Magnitude
                
                local isStartIslandRobo = (distToPlayer < 1000 and islandDistFromPlayer > 1500)
                
                if not isStartIslandRobo and distToTarget <= 300 then
                    if distToTarget < closestDist then
                        closestDist = distToTarget
                        closestRobo = v
                    end
                end
            end
        end

        local isLookingForRobo = false
        if closestRobo and closestRobo:FindFirstChild("HumanoidRootPart") then
            targetPos = closestRobo.HumanoidRootPart.Position
        else
            isLookingForRobo = true
        end
        
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hipHeight = hum and hum.HipHeight or 2.15
        local floorOffset = hipHeight + (root.Size.Y / 2)
        
        ToggleHover(true)
        
        local climbEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("climb")
        local sprintEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("sprint")
        local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
        
        local function SpiderLerp(tPos, currentSpeed)
            local startPos = root.Position
            local flatStart = Vector3.new(startPos.X, 0, startPos.Z)
            local flatTarget = Vector3.new(tPos.X, 0, tPos.Z)
            local totalDist = (flatStart - flatTarget).Magnitude
            
            if totalDist < 5 then return true end 
            
            currentSpeed = currentSpeed > 0 and currentSpeed or RyuConfig.IslandSpeed
            local t = totalDist / currentSpeed
            if t < 0.1 then return true end
            
            local elapsedTime = 0
            local currentY = root.Position.Y
            local isClimbing = false
            local lastFootstep = tick()
            local nextRoboCheck = tick()
            local lastClimbFire = 0
            
            char:SetAttribute("evading", true)
            _G.soruDashing = true

            local rayParamsDown = RaycastParams.new()
            rayParamsDown.FilterDescendantsInstances = {char, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles")}
            rayParamsDown.FilterType = Enum.RaycastFilterType.Exclude

            if hum then hum.PlatformStand = false end

            while elapsedTime < t do
                local dt = RunService.Heartbeat:Wait()
                dt = math.clamp(dt, 0.001, 0.05)
                
                if isLookingForRobo and tick() - nextRoboCheck > 1 then
                    nextRoboCheck = tick()
                    local npcsFolder = Workspace:FindFirstChild("NPCs")
                    if npcsFolder then
                        for _, v in pairs(npcsFolder:GetChildren()) do
                            if v.Name == "Robo" and v:IsA("Model") and v:FindFirstChild("HumanoidRootPart") then
                                local distToTarget = (v.HumanoidRootPart.Position - rawPos).Magnitude
                                if (v.HumanoidRootPart.Position - rawPos).Magnitude < 1500 and distToTarget <= 300 then
                                    tPos = v.HumanoidRootPart.Position
                                    isLookingForRobo = false
                                    
                                    startPos = root.Position
                                    local newFlatStart = Vector3.new(startPos.X, 0, startPos.Z)
                                    local newFlatTarget = Vector3.new(tPos.X, 0, tPos.Z)
                                    totalDist = (newFlatStart - newFlatTarget).Magnitude
                                    t = totalDist / currentSpeed
                                    elapsedTime = 0
                                    
                                    break
                                end
                            end
                        end
                    end
                end
                
                local currentPos = root.Position
                if (currentPos - tPos).Magnitude <= 5 then break end
                
                local alpha = math.clamp(elapsedTime / t, 0, 1)
                local currentX = startPos.X + (tPos.X - startPos.X) * alpha
                local currentZ = startPos.Z + (tPos.Z - startPos.Z) * alpha
                
                local flatMoveDir = (Vector3.new(tPos.X, 0, tPos.Z) - Vector3.new(currentPos.X, 0, currentPos.Z))
                if flatMoveDir.Magnitude > 0.1 then flatMoveDir = flatMoveDir.Unit else flatMoveDir = root.CFrame.LookVector end
                
                local samplePos1 = Vector3.new(currentX, 0, currentZ)
                local samplePos2 = samplePos1 + (flatMoveDir * 6)
                
                local hit1 = Workspace:Raycast(Vector3.new(samplePos1.X, currentY + 15, samplePos1.Z), Vector3.new(0, -3000, 0), rayParamsDown)
                local y1 = hit1 and hit1.Position.Y or 0
                
                local hit2 = Workspace:Raycast(Vector3.new(samplePos2.X, 2500, samplePos2.Z), Vector3.new(0, -3000, 0), rayParamsDown)
                local y2 = hit2 and hit2.Position.Y or 0
                
                local forwardRayStart = currentPos + Vector3.new(0, 1.5, 0)
                local forwardHit = Workspace:Raycast(forwardRayStart, flatMoveDir * 6, rayParamsDown)
                
                local targetY = y1
                if forwardHit then
                    targetY = math.max(y1, y2)
                else
                    if math.abs(y2 - currentY) < 6 then
                        targetY = y2
                    end
                end
                
                targetY = math.max(targetY, 1) 
                
                local finalY = targetY + floorOffset
                local yVelocity = 0
                local addTime = dt
                
                local wallCheckHit = Workspace:Raycast(currentPos, flatMoveDir * 4.5, rayParamsDown)
                local isWallBlocking = wallCheckHit and wallCheckHit.Distance <= 4

                if finalY > currentY + 3 then 
                    if not isClimbing then
                        isClimbing = true
                    end
                    
                    if tick() - lastClimbFire > 0.3 then
                        lastClimbFire = tick()
                        task.spawn(function()
                            if climbEvent then pcall(function() climbEvent:InvokeServer(true) end) end
                        end)
                    end
                    if hum then hum:ChangeState(Enum.HumanoidStateType.Climbing) end
                    
                    local climbRate = currentSpeed * 0.8
                    currentY = math.min(currentY + (climbRate * dt), finalY)
                    yVelocity = climbRate
                    
                    if isWallBlocking then
                        addTime = 0 
                    elseif finalY - currentY > 5 then
                        addTime = dt * 0.3
                    end
                else
                    if isClimbing then
                        isClimbing = false
                        task.spawn(function()
                            if climbEvent then pcall(function() climbEvent:InvokeServer(false) end) end
                        end)
                        if hum then hum:ChangeState(Enum.HumanoidStateType.Running) end
                    end
                    
                    if finalY < currentY then
                        local fallRate = 150
                        currentY = math.max(currentY - (fallRate * dt), finalY)
                        yVelocity = -fallRate
                    else
                        currentY = finalY
                    end
                end
                
                currentY = math.max(currentY, 1)
                
                elapsedTime = elapsedTime + addTime
                
                local finalPos = Vector3.new(currentX, currentY, currentZ)
                local lookPos = Vector3.new(tPos.X, currentY, tPos.Z)
                
                local moveDir = (lookPos - finalPos).Unit
                if moveDir ~= moveDir then moveDir = root.CFrame.LookVector end
                
                if (lookPos - finalPos).Magnitude > 0.1 then 
                    root.CFrame = CFrame.lookAt(finalPos, lookPos)
                else
                    root.CFrame = CFrame.new(finalPos)
                end
                
                if hum then hum:Move(moveDir, false) end
                
                root.Velocity = Vector3.new(moveDir.X * currentSpeed, yVelocity, moveDir.Z * currentSpeed)
                
                local bp = root:FindFirstChild("RyuHover")
                if bp then bp.Position = finalPos end
                
                if tick() - lastFootstep > 0.3 then
                    lastFootstep = tick()
                    if not isClimbing then
                        if sprintEvent then pcall(function() sprintEvent:FireServer("rbxassetid://15382065457") end) end
                        if footstepEvent then pcall(function() footstepEvent:FireServer() end) end
                    end
                end
            end
            
            if hum then hum:Move(Vector3.new(0,0,0), false) end
            if isClimbing then
                task.spawn(function()
                    if climbEvent then pcall(function() climbEvent:InvokeServer(false) end) end
                end)
            end
            
            char:SetAttribute("evading", nil)
            _G.soruDashing = nil
            
            root.Velocity = Vector3.new(0, 0, 0)
            
            return true
        end
        
        SpiderLerp(targetPos, RyuConfig.IslandSpeed)
        
        if hum then hum.Jump = true end
        root.Velocity = Vector3.new(0, 0, 0)
        
        ToggleHover(false)
        _G.RyuIsTweening = false
    end)
end)

local SecAutoBuy = CreateSection(SubAutoBuy, "Auto Buy")
CreateButton(SecAutoBuy, "Buy Geppo", function()
    pcall(function()
        local inter = LocalPlayer:WaitForChild("InteractionsV2", 2)
        if inter then
            if inter:IsA("RemoteEvent") then
                inter:FireServer("Geppo")
            elseif inter:IsA("RemoteFunction") then
                inter:InvokeServer("Geppo")
            end
        end
    end)
end)

local suggestLabel = Instance.new("TextLabel", SecAutoBuy)
suggestLabel.Size = UDim2.new(0.92, 0, 0, 20)
suggestLabel.BackgroundTransparency = 1
suggestLabel.Text = "SUGGEST MORE IN THE DISCORD"
suggestLabel.TextColor3 = Theme.SubText
suggestLabel.Font = Enum.Font.GothamBold
suggestLabel.TextSize = 11
suggestLabel.TextXAlignment = Enum.TextXAlignment.Center

--// ============================================================================
--// MODULE HOOKING: PURE RAW COMBAT (ANIMATION + DAMAGE REMOTES)
--// ============================================================================
local currentComboIndex = 1
local lastSwing = 0

local function EquipTargetWeapon()
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    
    local targetWep = RyuConfig.TargetWeapon
    
    if char:FindFirstChild(targetWep) then return true end
    for _, item in pairs(char:GetChildren()) do
        if item:IsA("Tool") and (item.Name:lower():find(targetWep:lower()) or item:GetAttribute("MeleeTool")) then
            return true 
        end
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
        -- Nutzt jetzt den einstellbaren Delay aus dem UI
        if now - lastSwing >= RyuConfig.AttackDelay then
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
                -- Schlag 5 komplett entfernt, da er Knockback verursacht!
                
                local animObj = ReplicatedStorage:FindFirstChild("CombatAnimations") 
                    and ReplicatedStorage.CombatAnimations:FindFirstChild("Melee")
                    and ReplicatedStorage.CombatAnimations.Melee:FindFirstChild(animName)
                
                -- 1. SCHRITT: Sende die Animation an den Server
                if animObj then
                    local argsAnim = {
                        "swingsfx",
                        "Melee",
                        currentComboIndex,
                        "Ground",
                        currentComboIndex == 1,
                        animObj,
                        2,
                        1.5
                    }
                    ReplicatedStorage.Events.CombatRegister:InvokeServer(argsAnim)
                end
                
                -- 2. SCHRITT: Sende sofort danach den Schaden an den Server
                if #hitParts > 0 then
                    local argsDamage = {
                        "damage",
                        hitParts,
                        "Melee",
                        {currentComboIndex, "Ground", "Melee"},
                        true,
                        root.CFrame,
                        ["aircombo"] = "Ground"
                    }
                    ReplicatedStorage.Events.CombatRegister:InvokeServer(argsDamage)
                end
                
                currentComboIndex = currentComboIndex + 1
                -- Nach dem 4. Schlag direkt wieder bei 1 anfangen (kein 5. Schlag!)
                if currentComboIndex > 4 then currentComboIndex = 1 end
            end)
        end
    end)
end

--// ============================================================================
--// UNBANNABLE MICRO-STEP TWEEN ENGINE (NORMALES SCHAUEN)
--// ============================================================================
local function SafeTween(targetCFrame, customSpeed)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local startPos = root.Position
    local targetPos = targetCFrame.Position
    local dist = (targetPos - startPos).Magnitude
    
    local speed = customSpeed or RyuConfig.TweenSpeed 
    local timeToTake = dist / speed
    
    if timeToTake < 0.1 then 
        root.CFrame = targetCFrame
        return 
    end

    local startTime = tick()
    while tick() - startTime < timeToTake do
        if not RyuConfig.AutoFarm then break end
        
        local alpha = (tick() - startTime) / timeToTake
        local intermediatePos = startPos:Lerp(targetPos, alpha)
        
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp.Position = intermediatePos end
        
        root.CFrame = CFrame.new(intermediatePos) * targetCFrame.Rotation
        RunService.Heartbeat:Wait()
    end
    
    local bpFinal = root:FindFirstChild("RyuHover")
    if bpFinal then bpFinal.Position = targetPos end
    root.CFrame = targetCFrame
end

-- Anti-Fling & Flacker-Schutz
RunService.Stepped:Connect(function()
    if RyuConfig.Noclip or RyuConfig.AutoFarm then
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum and RyuConfig.AutoFarm then
                hum.AutoRotate = false 
            end
            
            for _, v in pairs(char:GetChildren()) do
                if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" and v.CanCollide then 
                    v.CanCollide = false 
                end
            end
        end
    else
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then hum.AutoRotate = true end
    end
end)

--// ============================================================================
--// HARMONY CORE: 1-BY-1 FARM (MOB GROUPING)
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
    local npc = Workspace:FindFirstChild(RyuConfig.TargetNPC, true)
    if npc then
        local npcPos = npc:IsA("Model") and npc:GetPivot() or npc.CFrame
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            local targetPos = npcPos.Position + Vector3.new(0, RyuConfig.KillHeight, 3.5)
            local targetCFrame = CFrame.lookAt(targetPos, Vector3.new(npcPos.Position.X, targetPos.Y, npcPos.Position.Z))
            
            SafeTween(targetCFrame)
            root.CFrame = targetCFrame
            
            pcall(function()
                local QuestEvent = ReplicatedStorage.Events.Quest
                QuestEvent:InvokeServer({"npcChat", true})
                local questString = "Help " .. RyuConfig.TargetNPC
                QuestEvent:InvokeServer({"takequest", questString})
                QuestEvent:InvokeServer({"takequest", RyuConfig.TargetNPC})
                QuestEvent:InvokeServer({"takequest"})
                QuestEvent:InvokeServer("takequest")
                QuestEvent:InvokeServer({"acceptquest"})
                QuestEvent:InvokeServer("acceptquest")
            end)
            task.wait(0.5)
        end
    end
end

--// FAIL-SAFE: QUEST SICHERUNG
task.spawn(function()
    while true do
        task.wait(180) 
        if RyuConfig.AutoFarm and RyuConfig.TargetNPC ~= "" and RyuConfig.TargetNPC ~= "None" then
            if not CheckQuestActive() then
                FetchQuest()
            end
        end
    end
end)

--// AUTO STATS LOOP
task.spawn(function()
    while true do
        task.wait(3) 
        
        local function upgradeStat(statName)
            for i = 1, 5 do 
                pcall(function()
                    ReplicatedStorage:WaitForChild("Events"):WaitForChild("stats"):FireServer(statName, nil, 1)
                end)
            end
        end

        if RyuConfig.AutoStrength then upgradeStat("Strength") end
        if RyuConfig.AutoStamina then upgradeStat("Stamina") end
        if RyuConfig.AutoDefense then upgradeStat("Defense") end
        if RyuConfig.AutoSword then upgradeStat("SwordMastery") end
        if RyuConfig.AutoGun then upgradeStat("GunMastery") end
    end
end)

--// MAIN FARM LOOP (KITING / GROUPING REWORK)
task.spawn(function()
    while true do
        task.wait(0.1)
        
        if not RyuConfig.AutoFarm then
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then hum.AutoRotate = true end
            continue
        end

        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        
        if not root or not hum or hum.Health <= 0 then continue end

        ToggleHover(true)
        hum.AutoRotate = false 
        
        --// 1. QUEST CHECK
        if RyuConfig.TargetNPC and RyuConfig.TargetNPC ~= "" then
            if not CheckQuestActive() then
                FetchQuest()
                continue
            end
        end

        --// 2. MOB GROUPING / KITING (ALLE PULLEN & IN DER MITTE TÖTEN)
        if RyuConfig.TargetMob and RyuConfig.TargetMob ~= "" then
            local npcs = Workspace:FindFirstChild("NPCs")
            if not npcs then continue end
            
            local targetMobs = {}
            local centerPos = Vector3.new(0, 0, 0)
            local mobCount = 0
            
            for _, npc in pairs(npcs:GetChildren()) do
                if npc.Name == RyuConfig.TargetMob then
                    local mHum = npc:FindFirstChildOfClass("Humanoid")
                    local mRoot = npc:FindFirstChild("HumanoidRootPart")
                    
                    local isRagdolled = npc:FindFirstChild("Rag") or (npc.Parent and npc.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                    
                    if mHum and mRoot and mHum.Health > 0 and not isRagdolled then
                        table.insert(targetMobs, npc)
                        centerPos = centerPos + mRoot.Position
                        mobCount = mobCount + 1
                    end
                end
            end
            
            if mobCount > 0 then
                centerPos = centerPos / mobCount
                local attackCenter = centerPos + Vector3.new(0, RyuConfig.KillHeight, 0)
                
                EquipTargetWeapon()
                
                -- Phase 1: Jeden Mob einmal anhitten (Pulling)
                for _, npc in ipairs(targetMobs) do
                    if not RyuConfig.AutoFarm or not CheckQuestActive() then break end
                    
                    local mHum = npc:FindFirstChildOfClass("Humanoid")
                    local mRoot = npc:FindFirstChild("HumanoidRootPart")
                    local isRagdolled = npc:FindFirstChild("Rag") or (npc.Parent and npc.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                    
                    if mHum and mRoot and mHum.Health > 0 and not isRagdolled then
                        local curFlatDir = Vector3.new(root.Position.X - mRoot.Position.X, 0, root.Position.Z - mRoot.Position.Z)
                        if curFlatDir.Magnitude < 0.1 then curFlatDir = Vector3.new(1, 0, 0) end
                        
                        local attackPos = mRoot.Position + (curFlatDir.Unit * 3) + Vector3.new(0, RyuConfig.KillHeight, 0)
                        local targetCFrame = CFrame.lookAt(attackPos, Vector3.new(mRoot.Position.X, attackPos.Y, mRoot.Position.Z))
                        
                        local distToPos = (root.Position - attackPos).Magnitude
                        if distToPos > 5 then
                            SafeTween(targetCFrame)
                        end
                        
                        local bp = root:FindFirstChild("RyuHover")
                        if bp then bp.Position = attackPos end
                        root.CFrame = targetCFrame
                        
                        PerformMeleeAttack({npc})
                        task.wait(0.05)
                    end
                end
                
                -- Phase 2: In die Mitte fliegen und Spammen
                if RyuConfig.AutoFarm and CheckQuestActive() then
                    local targetCFrameCenter = CFrame.new(attackCenter)
                    
                    if (root.Position - attackCenter).Magnitude > 5 then
                        SafeTween(targetCFrameCenter)
                    end
                    
                    local anyAlive = true
                    while RyuConfig.AutoFarm and anyAlive do
                        if not CheckQuestActive() then break end
                        anyAlive = false
                        
                        for _, npc in ipairs(targetMobs) do
                            local mHum = npc:FindFirstChildOfClass("Humanoid")
                            local mRoot = npc:FindFirstChild("HumanoidRootPart")
                            local isRagdolled = npc:FindFirstChild("Rag") or (npc.Parent and npc.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                            
                            if mHum and mRoot and mHum.Health > 0 then
                                anyAlive = true
                                
                                for _, part in pairs(npc:GetChildren()) do
                                    if part:IsA("BasePart") then
                                        part.CanCollide = false
                                        part.Velocity = Vector3.new(0, 0, 0)
                                        part.RotVelocity = Vector3.new(0, 0, 0)
                                    end
                                end
                                
                                if isRagdolled then
                                    mRoot.Size = Vector3.new(2, 2, 1)
                                else
                                    mRoot.Size = Vector3.new(20, 20, 20)
                                    mRoot.CanCollide = false
                                    mRoot.Velocity = Vector3.new(0, 0, 0)
                                    mRoot.RotVelocity = Vector3.new(0, 0, 0)
                                end
                            end
                        end
                        
                        if anyAlive then
                            local bp = root:FindFirstChild("RyuHover")
                            if bp then bp.Position = attackCenter end
                            root.CFrame = targetCFrameCenter
                            
                            PerformMeleeAttack(targetMobs)
                            task.wait(0.05)
                        end
                    end
                    
                    -- Reset Hitboxes
                    for _, npc in ipairs(targetMobs) do
                        local mRoot = npc:FindFirstChild("HumanoidRootPart")
                        if mRoot then mRoot.Size = Vector3.new(2, 2, 1) end
                    end
                end
            end
        end
    end
end)
