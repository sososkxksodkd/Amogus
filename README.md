--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION (PORTAL WALK-SIMULATION & MAX SPEED 60)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")

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
    KillHeight = 7, 
    FishmanSpeed = 85, 
    ElevatorSpeed = 85,
    
    TargetIsland = IslandList[1],
    IslandSpeed = 60, -- Standard Speed auf 60 angepasst
    
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

--// UI AUFBAU: FARM (FISHMAN CAVE FARM)
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

local SecMovement = CreateSection(SubLeveling, "Fishman Cave Movement")
CreateSlider(SecMovement, "Cave Travel Speed", 50, 85, RyuConfig.FishmanSpeed, function(val)
    RyuConfig.FishmanSpeed = val
end)
CreateSlider(SecMovement, "Aufzug Geschw. (Y-Achse)", 5, 85, RyuConfig.ElevatorSpeed, function(val)
    RyuConfig.ElevatorSpeed = val
end)

CreateButton(SecMovement, "Smart Sky-TP to Fishman Cave", function()
    task.spawn(function()
        local cave = Workspace:FindFirstChild("Fishman Cave", true) or Workspace:FindFirstChild("FishmanIsland", true)
        if not cave then return end
        
        local targetPos = cave:IsA("Model") and cave:GetPivot().Position or cave.CFrame.Position
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hipHeight = hum and hum.HipHeight or 2.15
        local floorOffset = hipHeight + (root.Size.Y / 2)
        
        local platform = Instance.new("Part")
        platform.Name = "Part" 
        platform.Size = Vector3.new(40, 3, 40) 
        platform.Anchored = true
        platform.CanCollide = true
        platform.Transparency = 0.5
        platform.Material = Enum.Material.ForceField
        platform.Color = Color3.fromRGB(0, 255, 255)
        platform.CFrame = CFrame.new(root.Position - Vector3.new(0, floorOffset, 0))
        platform.Parent = Workspace
        
        ToggleHover(true)
        
        local function CustomLerp(tPos, currentSpeed, isHoverMode)
            local totalDist = (root.Position - tPos).Magnitude
            local t = totalDist / currentSpeed
            if t < 0.1 then return end
            
            local startPos = root.Position
            local startTime = tick()
            local lastDrop = tick()
            
            char:SetAttribute("evading", true)
            _G.soruDashing = true
            
            while tick() - startTime < t do
                local alpha = (tick() - startTime) / t
                local intermediatePos = startPos:Lerp(tPos, alpha)
                
                local lookPos = Vector3.new(tPos.X, intermediatePos.Y, tPos.Z)
                if (lookPos - intermediatePos).Magnitude < 0.1 then 
                    lookPos = intermediatePos + root.CFrame.LookVector 
                end
                
                if (root.Position - intermediatePos).Magnitude > 20 or (not isHoverMode and tick() - lastDrop >= 2.5) then
                    local isDrop = (tick() - lastDrop >= 2.5)
                    
                    ToggleHover(false)
                    platform.CFrame = CFrame.new(0, 99999, 0) 
                    
                    if hum then hum.Jump = true end
                    root.Velocity = Vector3.new(0, 50, 0)
                    
                    task.wait(isDrop and 0.7 or 1)
                    
                    ToggleHover(true)
                    startPos = root.Position
                    totalDist = (startPos - tPos).Magnitude
                    currentSpeed = currentSpeed > 0 and currentSpeed or 85
                    t = totalDist / currentSpeed
                    startTime = tick()
                    lastDrop = tick()
                else
                    local bp = root:FindFirstChild("RyuHover")
                    if bp then bp.Position = intermediatePos end
                    
                    root.CFrame = CFrame.lookAt(intermediatePos, lookPos)
                    root.Velocity = Vector3.new(0, 0, 0) 
                    
                    platform.CFrame = CFrame.new(intermediatePos.X, intermediatePos.Y - floorOffset, intermediatePos.Z)
                end
                RunService.Heartbeat:Wait()
            end
            root.CFrame = CFrame.new(tPos)
            
            char:SetAttribute("evading", nil)
            _G.soruDashing = nil
        end
        
        local safeY = 1500
        -- FIX: Direktes Tween zur Höhle statt Koordinaten
        CustomLerp(Vector3.new(root.Position.X, safeY, root.Position.Z), RyuConfig.ElevatorSpeed, false)
        CustomLerp(Vector3.new(targetPos.X, safeY, targetPos.Z), RyuConfig.FishmanSpeed, false)
        CustomLerp(targetPos + Vector3.new(0, 50, 0), RyuConfig.ElevatorSpeed, false)
        
        if hum then hum.Jump = true end
        root.Velocity = Vector3.new(0, 0, 0)
        
        -- FIX: Exakt 5 Sekunden warten vor dem Teleport
        RyuNotify:Send("Smart TP", "Warte 5 Sekunden für Portal-TP...", 5)
        task.wait(5)
        
        local areaTp = Workspace:FindFirstChild("AreaTeleporters")
        if areaTp and areaTp:FindFirstChild("FirstSea") and areaTp.FirstSea:FindFirstChild("Fishman") and areaTp.FirstSea.Fishman:FindFirstChild("Part") then
            local portal = areaTp.FirstSea.Fishman.Part
            
            local tpSuccess = false
            local isBlack = false
            
            -- FIX: Smart Retry Loop & WALK SIMULATION
            while not tpSuccess do
                ToggleHover(false)
                root.Velocity = Vector3.new(0, 0, 0)
                -- Teleportiere direkt AUF das Portal (mit leichtem Y-Abstand)
                root.CFrame = portal.CFrame * CFrame.new(0, 3, 0)
                
                RyuNotify:Send("Smart TP", "Versuche Portal-Teleport...", 3)
                
                local checkStart = tick()
                isBlack = false
                
                while tick() - checkStart < 4 do
                    if char and root and portal and (root.Position - portal.Position).Magnitude > 1000 then
                        tpSuccess = true
                        break
                    end
                    
                    -- PORTAL WALK-SIMULATION: Charakter läuft im Kreis, um das Portal zu triggern
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
                    RyuNotify:Send("Smart TP", "Teleport verzögert! Versuche es in 3 Sek. nochmal...", 3)
                    task.wait(3)
                end
            end
            
            if isBlack then
                RyuNotify:Send("Smart TP", "Ladebildschirm erkannt! Warte auf Map...", 3)
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
            
            ToggleHover(true)
            RyuNotify:Send("Smart TP", "Erfolgreich in Fishman Cave angekommen!", 3)
        end
        
        platform:Destroy()
        ToggleHover(false)
    end)
end)

CreateButton(SecMovement, "Boden-TP to Fishman Cave (Direkt)", function()
    task.spawn(function()
        local cave = Workspace:FindFirstChild("Fishman Cave", true) or Workspace:FindFirstChild("FishmanIsland", true)
        if not cave then return end
        
        local targetPos = cave:IsA("Model") and cave:GetPivot().Position or cave.CFrame.Position
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hipHeight = hum and hum.HipHeight or 2.15
        local floorOffset = hipHeight + (root.Size.Y / 2)
        
        local platform = Instance.new("Part")
        platform.Name = "Part"
        platform.Size = Vector3.new(40, 3, 40) 
        platform.Anchored = true
        platform.CanCollide = true
        platform.Transparency = 0.5
        platform.Material = Enum.Material.ForceField
        platform.Color = Color3.fromRGB(0, 255, 255)
        platform.CFrame = CFrame.new(root.Position - Vector3.new(0, floorOffset, 0))
        platform.Parent = Workspace
        
        ToggleHover(true)
        
        local function CustomLerp(tPos, currentSpeed, isHoverMode)
            local totalDist = (root.Position - tPos).Magnitude
            local t = totalDist / currentSpeed
            if t < 0.1 then return end
            
            local startPos = root.Position
            local startTime = tick()
            local lastDrop = tick() 
            
            char:SetAttribute("evading", true)
            _G.soruDashing = true
            
            while tick() - startTime < t do
                local alpha = (tick() - startTime) / t
                local intermediatePos = startPos:Lerp(tPos, alpha)
                
                local lookPos = Vector3.new(tPos.X, intermediatePos.Y, tPos.Z)
                if (lookPos - intermediatePos).Magnitude < 0.1 then 
                    lookPos = intermediatePos + root.CFrame.LookVector 
                end
                
                if (root.Position - intermediatePos).Magnitude > 20 or (not isHoverMode and tick() - lastDrop >= 2.5) then
                    local isDrop = (tick() - lastDrop >= 2.5)
                    
                    ToggleHover(false)
                    platform.CFrame = CFrame.new(0, 99999, 0) 
                    
                    if hum then hum.Jump = true end
                    root.Velocity = Vector3.new(0, 50, 0)
                    
                    task.wait(isDrop and 0.7 or 1)
                    
                    ToggleHover(true)
                    startPos = root.Position
                    totalDist = (startPos - tPos).Magnitude
                    currentSpeed = currentSpeed > 0 and currentSpeed or 85
                    t = totalDist / currentSpeed
                    startTime = tick()
                    lastDrop = tick()
                else
                    local bp = root:FindFirstChild("RyuHover")
                    if bp then bp.Position = intermediatePos end
                    
                    root.CFrame = CFrame.lookAt(intermediatePos, lookPos)
                    root.Velocity = Vector3.new(0, 0, 0)
                    
                    platform.CFrame = CFrame.new(intermediatePos.X, intermediatePos.Y - floorOffset, intermediatePos.Z)
                end
                RunService.Heartbeat:Wait()
            end
            root.CFrame = CFrame.new(tPos)
            
            char:SetAttribute("evading", nil)
            _G.soruDashing = nil
        end
        
        -- FIX: Direktes Tween zur Höhle statt Koordinaten
        CustomLerp(targetPos + Vector3.new(0, 50, 0), RyuConfig.FishmanSpeed, false)
        
        if hum then hum.Jump = true end
        root.Velocity = Vector3.new(0, 0, 0)
        
        -- FIX: Exakt 5 Sekunden warten vor dem Teleport
        RyuNotify:Send("Smart TP", "Warte 5 Sekunden für Portal-TP...", 5)
        task.wait(5)
        
        local areaTp = Workspace:FindFirstChild("AreaTeleporters")
        if areaTp and areaTp:FindFirstChild("FirstSea") and areaTp.FirstSea:FindFirstChild("Fishman") and areaTp.FirstSea.Fishman:FindFirstChild("Part") then
            local portal = areaTp.FirstSea.Fishman.Part
            
            local tpSuccess = false
            local isBlack = false
            
            -- FIX: Smart Retry Loop & WALK SIMULATION
            while not tpSuccess do
                ToggleHover(false)
                root.Velocity = Vector3.new(0, 0, 0)
                -- Teleportiere direkt AUF das Portal
                root.CFrame = portal.CFrame * CFrame.new(0, 3, 0)
                
                RyuNotify:Send("Smart TP", "Versuche Portal-Teleport...", 3)
                
                local checkStart = tick()
                isBlack = false
                
                while tick() - checkStart < 4 do
                    if char and root and portal and (root.Position - portal.Position).Magnitude > 1000 then
                        tpSuccess = true
                        break
                    end
                    
                    -- PORTAL WALK-SIMULATION
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
                    RyuNotify:Send("Smart TP", "Teleport verzögert! Versuche es in 3 Sek. nochmal...", 3)
                    task.wait(3)
                end
            end
            
            if isBlack then
                RyuNotify:Send("Smart TP", "Ladebildschirm erkannt! Warte auf Map...", 3)
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
            
            ToggleHover(true)
            RyuNotify:Send("Smart TP", "Erfolgreich in Fishman Cave angekommen!", 3)
        end
        
        platform:Destroy()
        ToggleHover(false)
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
local SubTransport = CreateSubTab(TabMobility, "Transportation")
local SubAutoBuy = CreateSubTab(TabMobility, "Auto Buy")

local SecIslandTP = CreateSection(SubTransport, "Island Teleportation")
CreateDropdown(SecIslandTP, "Select Island", IslandList, "TargetIsland")
-- FIX: Slider maximal auf 60 gesetzt
CreateSlider(SecIslandTP, "Travel Speed", 50, 60, RyuConfig.IslandSpeed, function(val)
    RyuConfig.IslandSpeed = val
end)

CreateButton(SecIslandTP, "Smart Sky-TP to Island", function()
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
            RyuNotify:Send("Error", "Insel '" .. targetIslandName .. "' nicht in der Map gefunden!", 3)
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
            RyuNotify:Send("Error", "Konnte Zielkoordinaten nicht finden!", 3)
            return
        end
        
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hipHeight = hum and hum.HipHeight or 2.15
        local floorOffset = hipHeight + (root.Size.Y / 2)
        
        local targetPos = rawPos
        local rayParams = RaycastParams.new()
        rayParams.FilterDescendantsInstances = {island, Workspace:FindFirstChild("Terrain")}
        rayParams.FilterType = Enum.RaycastFilterType.Include
        
        local groundHit = Workspace:Raycast(rawPos + Vector3.new(0, 1000, 0), Vector3.new(0, -2000, 0), rayParams)
        
        if groundHit and groundHit.Position.Y >= -1 then
            targetPos = Vector3.new(groundHit.Position.X, groundHit.Position.Y + hipHeight + 2, groundHit.Position.Z)
        else
            targetPos = Vector3.new(rawPos.X, 1 + hipHeight + 5, rawPos.Z)
        end
        
        local platform = Instance.new("Part")
        platform.Name = "Part" 
        platform.Size = Vector3.new(40, 3, 40) 
        platform.Anchored = true
        platform.CanCollide = true
        platform.Transparency = 0.5
        platform.Material = Enum.Material.ForceField
        platform.Color = Color3.fromRGB(0, 255, 0) 
        platform.CFrame = CFrame.new(root.Position - Vector3.new(0, floorOffset, 0))
        platform.Parent = Workspace
        
        ToggleHover(true)
        
        local function IslandLerp(tPos, currentSpeed)
            local totalDist = (root.Position - tPos).Magnitude
            if totalDist < 5 then return true end 
            
            currentSpeed = currentSpeed > 0 and currentSpeed or 85
            local t = totalDist / currentSpeed
            if t < 0.1 then return true end
            
            local startPos = root.Position
            local startTime = tick()
            local lastClipCheck = tick()
            local clipped = false
            
            char:SetAttribute("evading", true)
            _G.soruDashing = true
            
            local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
            
            while tick() - startTime < t do
                
                if tick() - lastClipCheck > 0.1 then
                    lastClipCheck = tick()
                    pcall(function()
                        local op = OverlapParams.new()
                        op.FilterDescendantsInstances = {island}
                        op.FilterType = Enum.RaycastFilterType.Include
                        local hits = Workspace:GetPartsInPart(root, op)
                        for _, hitPart in ipairs(hits) do
                            if hitPart:IsA("BasePart") and hitPart.CanCollide then
                                clipped = true
                                break
                            end
                        end
                    end)
                    if clipped then break end
                end
                
                local alpha = (tick() - startTime) / t
                local intermediatePos = startPos:Lerp(tPos, alpha)
                
                local lookPos = Vector3.new(tPos.X, intermediatePos.Y, tPos.Z)
                if (lookPos - intermediatePos).Magnitude < 0.1 then 
                    lookPos = intermediatePos + root.CFrame.LookVector 
                end
                
                -- RAYCAST NACH UNTEN (Terrain abtasten)
                local rayParams = RaycastParams.new()
                rayParams.FilterDescendantsInstances = {char, platform, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles")}
                rayParams.FilterType = Enum.RaycastFilterType.Exclude

                local tempGroundHit = Workspace:Raycast(Vector3.new(intermediatePos.X, 2500, intermediatePos.Z), Vector3.new(0, -3000, 0), rayParams)
                
                local finalY
                if tempGroundHit and tempGroundHit.Position.Y >= -1 then
                    finalY = tempGroundHit.Position.Y + floorOffset + 5
                else
                    finalY = floorOffset + 1
                end
                
                finalY = math.max(finalY, 1)

                local finalPos = Vector3.new(intermediatePos.X, finalY, intermediatePos.Z)
                
                local lookPos = Vector3.new(tPos.X, finalPos.Y, tPos.Z)
                if (lookPos - finalPos).Magnitude > 0.1 then 
                    root.CFrame = CFrame.lookAt(finalPos, lookPos)
                else
                    root.CFrame = CFrame.new(finalPos)
                end
                
                root.Velocity = Vector3.new(0, 0, 0)
                platform.CFrame = CFrame.new(finalPos.X, finalPos.Y - floorOffset, finalPos.Z)
                
                -- ANTI-CHEAT BYPASS: FOOTSTEP SPAM
                if footstepEvent then
                    pcall(function() footstepEvent:FireServer() end)
                end
                
                RunService.Heartbeat:Wait()
            end
            
            if not clipped then
                local finalDist = (root.Position - tPos).Magnitude
                if finalDist > 20 then
                    root.CFrame = CFrame.new(tPos)
                end
            else
                RyuNotify:Send("Island TP", "Noclip erkannt! Ziel erfolgreich erreicht.", 2)
            end
            
            char:SetAttribute("evading", nil)
            _G.soruDashing = nil
            
            return clipped
        end
        
        RyuNotify:Send("Island TP", "Gleite nach " .. targetIslandName .. "...", 3)
        IslandLerp(rawPos, RyuConfig.IslandSpeed)
        
        if hum then hum.Jump = true end
        root.Velocity = Vector3.new(0, 0, 0)
        
        platform:Destroy()
        ToggleHover(false)
        RyuNotify:Send("Island TP", "Ziel erreicht!", 3)
    end)
end)

CreateButton(SecIslandTP, "Boden-TP to Island (Direkt)", function()
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
            RyuNotify:Send("Error", "Insel '" .. targetIslandName .. "' nicht in der Map gefunden!", 3)
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
            RyuNotify:Send("Error", "Konnte Zielkoordinaten nicht finden!", 3)
            return
        end
        
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hipHeight = hum and hum.HipHeight or 2.15
        local floorOffset = hipHeight + (root.Size.Y / 2)
        
        local targetPos = rawPos
        local rayParams = RaycastParams.new()
        rayParams.FilterDescendantsInstances = {island, Workspace:FindFirstChild("Terrain")}
        rayParams.FilterType = Enum.RaycastFilterType.Include
        
        local groundHit = Workspace:Raycast(rawPos + Vector3.new(0, 1000, 0), Vector3.new(0, -2000, 0), rayParams)
        
        if groundHit and groundHit.Position.Y >= -1 then
            targetPos = Vector3.new(groundHit.Position.X, groundHit.Position.Y + hipHeight + 2, groundHit.Position.Z)
        else
            targetPos = Vector3.new(rawPos.X, 1 + hipHeight + 5, rawPos.Z)
        end
        
        local platform = Instance.new("Part")
        platform.Name = "Part" 
        platform.Size = Vector3.new(40, 3, 40) 
        platform.Anchored = true
        platform.CanCollide = true
        platform.Transparency = 0.5
        platform.Material = Enum.Material.ForceField
        platform.Color = Color3.fromRGB(0, 255, 0) 
        platform.CFrame = CFrame.new(root.Position - Vector3.new(0, floorOffset, 0))
        platform.Parent = Workspace
        
        ToggleHover(true)
        
        local function IslandLerp(tPos, currentSpeed)
            local totalDist = (root.Position - tPos).Magnitude
            if totalDist < 5 then return true end 
            
            currentSpeed = currentSpeed > 0 and currentSpeed or 85
            local t = totalDist / currentSpeed
            if t < 0.1 then return true end
            
            local startPos = root.Position
            local startTime = tick()
            local lastClipCheck = tick()
            local clipped = false
            
            char:SetAttribute("evading", true)
            _G.soruDashing = true
            
            local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
            
            while tick() - startTime < t do
                
                if tick() - lastClipCheck > 0.1 then
                    lastClipCheck = tick()
                    pcall(function()
                        local op = OverlapParams.new()
                        op.FilterDescendantsInstances = {island}
                        op.FilterType = Enum.RaycastFilterType.Include
                        local hits = Workspace:GetPartsInPart(root, op)
                        for _, hitPart in ipairs(hits) do
                            if hitPart:IsA("BasePart") and hitPart.CanCollide then
                                clipped = true
                                break
                            end
                        end
                    end)
                    if clipped then break end
                end
                
                local alpha = (tick() - startTime) / t
                local intermediatePos = startPos:Lerp(tPos, alpha)
                
                local lookPos = Vector3.new(tPos.X, intermediatePos.Y, tPos.Z)
                if (lookPos - intermediatePos).Magnitude < 0.1 then 
                    lookPos = intermediatePos + root.CFrame.LookVector 
                end
                
                -- RAYCAST NACH UNTEN (Terrain abtasten)
                local rayParams = RaycastParams.new()
                rayParams.FilterDescendantsInstances = {char, platform, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles")}
                rayParams.FilterType = Enum.RaycastFilterType.Exclude

                local tempGroundHit = Workspace:Raycast(Vector3.new(intermediatePos.X, 2500, intermediatePos.Z), Vector3.new(0, -3000, 0), rayParams)
                
                local finalY
                if tempGroundHit and tempGroundHit.Position.Y >= -1 then
                    -- Auf einer Insel (Boden ist über oder nah am Meeresspiegel)
                    finalY = tempGroundHit.Position.Y + floorOffset + 5
                else
                    -- Im Meer (Boden ist tief im Minusbereich)
                    finalY = floorOffset + 1 -- Gleitet exakt auf Wasserhöhe (knapp über Y=0)
                end
                
                finalY = math.max(finalY, 1) -- Sicherheitsnetz: Niemals ins Minus!

                -- EXAKT 5 STUDS ÜBER DEM BODEN KLEBEN (Oder über dem Wasser)
                local finalPos = Vector3.new(intermediatePos.X, finalY, intermediatePos.Z)
                
                local lookPos = Vector3.new(tPos.X, finalPos.Y, tPos.Z)
                if (lookPos - finalPos).Magnitude > 0.1 then 
                    root.CFrame = CFrame.lookAt(finalPos, lookPos)
                else
                    root.CFrame = CFrame.new(finalPos)
                end
                
                root.Velocity = Vector3.new(0, 0, 0)
                platform.CFrame = CFrame.new(finalPos.X, finalPos.Y - floorOffset, finalPos.Z)
                
                -- ANTI-CHEAT BYPASS: FOOTSTEP SPAM
                if footstepEvent then
                    pcall(function() footstepEvent:FireServer() end)
                end
                
                RunService.Heartbeat:Wait()
            end
            
            if not clipped then
                local finalDist = (root.Position - tPos).Magnitude
                if finalDist > 20 then
                    root.CFrame = CFrame.new(tPos)
                end
            else
                RyuNotify:Send("Island TP", "Noclip erkannt! Ziel erfolgreich erreicht.", 2)
            end
            
            char:SetAttribute("evading", nil)
            _G.soruDashing = nil
            
            return clipped
        end
        
        RyuNotify:Send("Island TP", "Gleite direkt nach " .. targetIslandName .. "...", 3)
        IslandLerp(rawPos, RyuConfig.IslandSpeed)
        
        if hum then hum.Jump = true end
        root.Velocity = Vector3.new(0, 0, 0)
        
        platform:Destroy()
        ToggleHover(false)
        RyuNotify:Send("Island TP", "Ziel erreicht!", 3)
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
        RyuNotify:Send("Auto Buy", "Kaufanfrage für Geppo gesendet!", 3)
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
--// MODULE HOOKING: PURE RAW COMBAT 
--// ============================================================================
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

local function PerformMeleeAttack()
    pcall(function()
        local char = LocalPlayer.Character
        local tool = char and char:FindFirstChildOfClass("Tool")
        if tool then tool:Activate() end
        
        if mouse1click then mouse1click() end
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
        task.wait()
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
    end)
end

--// ============================================================================
--// UNBANNABLE MICRO-STEP TWEEN ENGINE
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
        
        root.CFrame = CFrame.lookAt(intermediatePos, targetPos)
        RunService.Heartbeat:Wait()
    end
    
    local bpFinal = root:FindFirstChild("RyuHover")
    if bpFinal then bpFinal.Position = targetPos end
    root.CFrame = targetCFrame
end

RunService.Stepped:Connect(function()
    if RyuConfig.Noclip or RyuConfig.AutoFarm then
        local char = LocalPlayer.Character
        if char then
            for _, v in pairs(char:GetChildren()) do
                if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" and v.CanCollide then 
                    v.CanCollide = false 
                end
            end
        end
    end
end)

--// ============================================================================
--// HARMONY CORE: 1-BY-1 FARM, QUEST FUSION & FAIL-SAFE
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
            SafeTween(npcPos * CFrame.new(0, 0, 3.5))
            root.CFrame = CFrame.lookAt(root.Position, Vector3.new(npcPos.Position.X, root.Position.Y, npcPos.Position.Z))
            
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
                RyuNotify:Send("Fail-Safe", "Quest-Sicherung greift ein!", 2)
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

--// MAIN FARM LOOP
task.spawn(function()
    while true do
        task.wait(0.1)
        
        if not RyuConfig.AutoFarm then
            continue
        end

        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        
        if not root or not hum or hum.Health <= 0 then continue end

        ToggleHover(true)
        
        --// 1. QUEST CHECK
        if RyuConfig.TargetNPC and RyuConfig.TargetNPC ~= "" then
            if not CheckQuestActive() then
                FetchQuest()
                continue
            end
        end

        --// 2. 1-BY-1 MOB HUNT
        if RyuConfig.TargetMob and RyuConfig.TargetMob ~= "" then
            local npcs = Workspace:FindFirstChild("NPCs")
            if not npcs then continue end
            
            local targetMob = nil
            local closestDist = math.huge
            
            for _, npc in pairs(npcs:GetChildren()) do
                if npc.Name == RyuConfig.TargetMob then
                    local mHum = npc:FindFirstChildOfClass("Humanoid")
                    local mRoot = npc:FindFirstChild("HumanoidRootPart")
                    
                    local isRagdolled = npc:FindFirstChild("Rag") or (npc.Parent and npc.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                    
                    if mHum and mRoot and mHum.Health > 0 and not isRagdolled then
                        local d = (root.Position - mRoot.Position).Magnitude
                        
                        if d < closestDist then
                            closestDist = d
                            targetMob = npc
                        end
                    end
                end
            end
            
            if targetMob then
                local mRoot = targetMob:FindFirstChild("HumanoidRootPart")
                local mHum = targetMob:FindFirstChildOfClass("Humanoid")
                
                EquipTargetWeapon()
                
                while RyuConfig.AutoFarm and mHum and mHum.Health > 0 do
                    local isRagdolled = targetMob:FindFirstChild("Rag") or (targetMob.Parent and targetMob.Parent.Name == "Ragdolls") or (mHum and mHum:GetAttribute("isRagdolled"))
                    
                    if isRagdolled then
                        mRoot.Size = Vector3.new(2, 2, 1)
                        task.wait(0.2)
                        continue 
                    end
                    
                    if not CheckQuestActive() then
                        break 
                    end
                    
                    mRoot.Size = Vector3.new(20, 20, 20)
                    mRoot.CanCollide = false
                    mRoot.Velocity = Vector3.new(0, 0, 0)
                    mRoot.RotVelocity = Vector3.new(0, 0, 0)
                    
                    local curFlatDir = Vector3.new(root.Position.X - mRoot.Position.X, 0, root.Position.Z - mRoot.Position.Z)
                    if curFlatDir.Magnitude < 0.1 then curFlatDir = Vector3.new(1, 0, 0) end
                    local attackPos = mRoot.Position + (curFlatDir.Unit * 3) + Vector3.new(0, RyuConfig.KillHeight, 0)
                    
                    local distToPos = (root.Position - attackPos).Magnitude
                    if distToPos > 5 then
                        SafeTween(CFrame.lookAt(attackPos, mRoot.Position))
                    end
                    
                    local bp = root:FindFirstChild("RyuHover")
                    if bp then bp.Position = attackPos end
                    root.CFrame = CFrame.lookAt(root.Position, Vector3.new(mRoot.Position.X, root.Position.Y, mRoot.Position.Z))
                    
                    PerformMeleeAttack()
                    task.wait(0.05)
                end
                
                if mRoot then mRoot.Size = Vector3.new(2, 2, 1) end
            end
        end
    end
end)

task.wait(0.5)
RyuNotify:Send("RYU HUB", "PC Edition: Portal Walk-Sim Active!", 4)
