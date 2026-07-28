--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION (PC EXCLUSIVE - DYNAMIC HEIGHT)
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
    
    AutoFarm = false,
    AutoQuest = false,
    DynamicHeight = false, -- NEU: Geht höher, wenn man getroffen wird
    
    TargetMob = DynamicEnemies[1],
    TargetIsland = "Town of Beginnings",
    TargetNPC = DynamicQuests[1],
    TargetWeapon = "Combat",
    
    TweenSpeed = 55, 
}

local GPOIslands = {
    "Town of Beginnings", "Sandora", "Shell's Town", "Orange Town", 
    "Restaurant Baratie", "Roca Island", "Sphinx Island", "Marine Fort F-1", 
    "Fishman Island", "Colosseum", "Land of the Sky", "Marine Base G-1",
    "Logue Town", "Kori Island", "Island Of Zou", "Gravito's Fort",
    "Shark Park"
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

--// UI SETUP (Reskalierbar)
local Theme = { Background = Color3.fromRGB(12, 12, 14), Sidebar = Color3.fromRGB(18, 18, 20), SectionBG = Color3.fromRGB(24, 24, 26), Text = Color3.fromRGB(250, 250, 250), SubText = Color3.fromRGB(130, 130, 135), Accent = Color3.fromRGB(255, 255, 255), ToggleOff = Color3.fromRGB(35, 35, 38), ToggleOn = Color3.fromRGB(255, 255, 255), Stroke = Color3.fromRGB(45, 45, 50) }
local currentMainSize = UDim2.new(0, 550, 0, 380) 
local SidebarWidth = 150

local RyuHub = Instance.new("ScreenGui"); RyuHub.Name = "RyuHubPremium"; RyuHub.ResetOnSpawn = false; RyuHub.IgnoreGuiInset = true; RyuHub.Parent = guiParent
local MainFrame = Instance.new("Frame"); MainFrame.Size = currentMainSize; MainFrame.Position = UDim2.new(0.5, -currentMainSize.X.Offset/2, 0.5, -currentMainSize.Y.Offset/2); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.Active = true; MainFrame.Parent = RyuHub; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextColor3 = Theme.Text; Title.TextXAlignment = Enum.TextXAlignment.Left
local SubTitle = Instance.new("TextLabel", Topbar); SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 38); SubTitle.BackgroundTransparency = 1; SubTitle.Text = "PC Exclusive Edition"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 11; SubTitle.TextXAlignment = Enum.TextXAlignment.Left

local ResizeGrip = Instance.new("TextButton", MainFrame)
ResizeGrip.Size = UDim2.new(0, 20, 0, 20)
ResizeGrip.Position = UDim2.new(1, -20, 1, -20)
ResizeGrip.BackgroundTransparency = 1
ResizeGrip.Text = "◢"
ResizeGrip.TextColor3 = Theme.SubText
ResizeGrip.TextSize = 16
ResizeGrip.Font = Enum.Font.GothamBold

local CloseBtn = Instance.new("TextButton", Topbar); CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG; CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.SubText; CloseBtn.Font = Enum.Font.GothamBold; Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

local ToggleBtn = Instance.new("TextButton"); ToggleBtn.Size = UDim2.new(0, 50, 0, 50); ToggleBtn.Position = UDim2.new(0, 25, 0, 25); ToggleBtn.BackgroundColor3 = Theme.Sidebar; ToggleBtn.Text = "R"; ToggleBtn.Font = Enum.Font.GothamBlack; ToggleBtn.TextColor3 = Theme.Accent; ToggleBtn.TextSize = 20; ToggleBtn.Parent = RyuHub; Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
Instance.new("UIStroke", ToggleBtn).Color = Theme.Accent; Instance.new("UIStroke", ToggleBtn).Thickness = 2

local tDragStart, tStartPos, isDraggingBtn = nil, nil, false
ToggleBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then isDraggingBtn = false; tDragStart = input.Position; tStartPos = ToggleBtn.Position end end)
UserInputService.InputChanged:Connect(function(input) if tDragStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - tDragStart; if delta.Magnitude > 5 then isDraggingBtn = true; ToggleBtn.Position = UDim2.new(tStartPos.X.Scale, tStartPos.X.Offset + delta.X, tStartPos.Y.Scale, tStartPos.Y.Offset + delta.Y) end end end)

local rDragging, rDragStart, rStartSize = false, nil, nil
ResizeGrip.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        rDragging = true; rDragStart = input.Position; rStartSize = MainFrame.AbsoluteSize
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if rDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - rDragStart
        local newWidth = math.max(450, rStartSize.X + delta.X)
        local newHeight = math.max(300, rStartSize.Y + delta.Y)
        currentMainSize = UDim2.new(0, newWidth, 0, newHeight)
        MainFrame.Size = currentMainSize
    end
end)

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

--// ANTI-FALL HOVER SYSTEM
local function ToggleHover(state)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    if state then
        local bv = root:FindFirstChild("RyuHover")
        if not bv then
            bv = Instance.new("BodyVelocity")
            bv.Name = "RyuHover"
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Velocity = Vector3.new(0, 0, 0)
            bv.Parent = root
        end
    else
        local bv = root:FindFirstChild("RyuHover")
        if bv then bv:Destroy() end
    end
end

-- UI AUFBAU
local TabFarm = CreateMainTab("Farm")
local SubLeveling = CreateSubTab(TabFarm, "Leveling")

local SecAutoFarmMain = CreateSection(SubLeveling, "Master Auto Farm (Kite & Kill)")
CreateToggle(SecAutoFarmMain, "Enable Auto Farm", RyuConfig.AutoFarm, function(state) 
    RyuConfig.AutoFarm = state 
    if not state then ToggleHover(false) end 
end)

-- NEU: Dynamic Height Option!
CreateToggle(SecAutoFarmMain, "Dynamic Height (Anti-Hit)", RyuConfig.DynamicHeight, function(state) 
    RyuConfig.DynamicHeight = state 
end)

CreateToggle(SecAutoFarmMain, "Auto Quest", RyuConfig.AutoQuest, function(state) 
    RyuConfig.AutoQuest = state 
end)

local SecAutoFarmConfig = CreateSection(SubLeveling, "Farm Setup")
CreateDropdown(SecAutoFarmConfig, "Select Weapon/Style", GPOWeapons, "TargetWeapon")
CreateDropdown(SecAutoFarmConfig, "Select Enemy", DynamicEnemies, "TargetMob")
CreateDropdown(SecAutoFarmConfig, "Select Quest NPC", DynamicQuests, "TargetNPC")

local SecFarmAdvanced = CreateSection(SubLeveling, "Advanced Options")
CreateSlider(SecFarmAdvanced, "Movement Speed (Tween)", 30, 150, RyuConfig.TweenSpeed, function(val) 
    RyuConfig.TweenSpeed = val 
end)

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

local function EquipCombat()
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    
    local tool = char:FindFirstChild(RyuConfig.TargetWeapon) or LocalPlayer.Backpack:FindFirstChild(RyuConfig.TargetWeapon)
    if not tool then
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
            if item:IsA("Tool") and (item.Name:lower():find("combat") or item.Name:lower():find("melee") or item:GetAttribute("MeleeTool")) then
                tool = item; break
            end
        end
    end
    if tool and tool.Parent == LocalPlayer.Backpack then
        hum:EquipTool(tool)
        task.wait(0.1)
    end
end

local function PerformAttack()
    local inputModule = GetInputCallbacks()
    pcall(function()
        if inputModule and inputModule.Utils.canAutoM1() then
            inputModule.Callbacks.Attack:PC_Activate()
        else
            VirtualUser:CaptureController()
            VirtualUser:ClickButton1(Vector2.new())
        end
    end)
end

--// ============================================================================
--// UNBANNABLE MICRO-STEP TWEEN ENGINE
--// ============================================================================
local function SafeTween(targetCFrame)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local startPos = root.Position
    local targetPos = targetCFrame.Position
    local dist = (targetPos - startPos).Magnitude
    
    local speed = RyuConfig.TweenSpeed 
    local timeToTake = dist / speed
    
    if timeToTake < 0.1 then 
        root.CFrame = targetCFrame
        return 
    end

    local startTime = tick()
    while tick() - startTime < timeToTake do
        if not RyuConfig.AutoFarm and not RyuConfig.AutoQuest then break end
        local alpha = (tick() - startTime) / timeToTake
        
        local intermediatePos = startPos:Lerp(targetPos, alpha)
        root.CFrame = CFrame.lookAt(intermediatePos, targetPos)
        RunService.Heartbeat:Wait()
    end
    
    root.CFrame = targetCFrame
end

RunService.Stepped:Connect(function()
    if RyuConfig.AutoFarm or RyuConfig.AutoQuest then
        local char = LocalPlayer.Character
        if char then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end
    end
end)

--// ============================================================================
--// GPO MASTER KITE FARM (7 STUDS + DYNAMIC ANTI-HIT)
--// ============================================================================
local function GetCurrentQuest()
    local q = LocalPlayer:FindFirstChild("Quest")
    return q and q:FindFirstChild("CurrentQuest") and q.CurrentQuest.Value or "None"
end

task.spawn(function()
    while true do
        task.wait(0.1)
        
        if RyuConfig.AutoQuest and RyuConfig.TargetNPC and RyuConfig.TargetNPC ~= "" then
            if GetCurrentQuest() == "None" or GetCurrentQuest() == "" then
                local npc = Workspace:FindFirstChild(RyuConfig.TargetNPC, true)
                if npc then
                    ToggleHover(true)
                    local npcPos = npc:IsA("Model") and npc:GetPivot() or npc.CFrame
                    SafeTween(npcPos * CFrame.new(0, 0, 5))
                    local events = ReplicatedStorage:FindFirstChild("Events")
                    if events and events:FindFirstChild("Quest") then 
                        pcall(function() events.Quest:InvokeServer("getNPCQuestLocations") end)
                        task.wait(0.2)
                        pcall(function() events.Quest:InvokeServer({{"npcChat", true}}) end)
                        task.wait(0.2)
                        pcall(function() events.Quest:InvokeServer("takequest") end)
                    end
                end
            end
        end

        if RyuConfig.AutoFarm and RyuConfig.TargetMob and RyuConfig.TargetMob ~= "" then
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            if root and hum and hum.Health > 0 then
                ToggleHover(true) 
                
                local npcs = Workspace:FindFirstChild("NPCs")
                if npcs then
                    local validMobs = {}
                    for _, npc in pairs(npcs:GetChildren()) do
                        if npc.Name == RyuConfig.TargetMob then
                            local mHum = npc:FindFirstChildOfClass("Humanoid")
                            local mRoot = npc:FindFirstChild("HumanoidRootPart")
                            if mHum and mRoot and mHum.Health == mHum.MaxHealth then
                                table.insert(validMobs, npc)
                            end
                        end
                    end
                    
                    local aggroedMobs = {}
                    
                    -- PHASE 1: AGGRO SAMMELN
                    EquipCombat()
                    
                    for _, mob in pairs(validMobs) do
                        if not RyuConfig.AutoFarm or hum.Health <= 0 then break end
                        if #aggroedMobs >= 5 then break end
                        
                        local mRoot = mob:FindFirstChild("HumanoidRootPart")
                        local mHum = mob:FindFirstChildOfClass("Humanoid")
                        
                        if mRoot and mHum and mHum.Health > 0 then
                            local flatDir = Vector3.new(root.Position.X - mRoot.Position.X, 0, root.Position.Z - mRoot.Position.Z)
                            if flatDir.Magnitude < 0.1 then flatDir = Vector3.new(1, 0, 0) end
                            
                            -- Anflug fürs Aggro bleibt bei 2 Studs davor (damit Hit connected)
                            local attackPos = mRoot.Position + (flatDir.Unit * 2)
                            SafeTween(CFrame.lookAt(attackPos, mRoot.Position))
                            
                            local startHp = mHum.Health
                            local timeout = tick()
                            
                            while RyuConfig.AutoFarm and hum.Health > 0 and mHum.Health >= startHp and mHum.Health > 0 do
                                if tick() - timeout > 3 then break end 
                                
                                root.CFrame = CFrame.lookAt(mRoot.Position + (flatDir.Unit * 2), mRoot.Position)
                                PerformAttack()
                                task.wait(0.05)
                            end
                            
                            if mHum.Health > 0 and mHum.Health < startHp then
                                table.insert(aggroedMobs, mob)
                            end
                        end
                    end
                    
                    -- PHASE 2: TÖTEN (Start bei 7 Studs + Dynamic Height Option)
                    if #aggroedMobs > 0 and RyuConfig.AutoFarm then
                        EquipCombat()
                        
                        local firstMobRoot = aggroedMobs[1]:FindFirstChild("HumanoidRootPart")
                        if firstMobRoot then
                            local currentHeightOffset = 7 -- STANDARD: 7 Studs über dem Feind
                            local skyPos = firstMobRoot.Position + Vector3.new(0, currentHeightOffset, 0)
                            
                            SafeTween(CFrame.new(skyPos))
                            
                            local killTimeout = tick()
                            local myLastHealth = hum.Health
                            
                            while RyuConfig.AutoFarm and hum.Health > 0 do
                                if tick() - killTimeout > 25 then break end 
                                
                                -- DYNAMIC HEIGHT CHECK
                                if hum.Health < myLastHealth then
                                    if RyuConfig.DynamicHeight then
                                        currentHeightOffset = currentHeightOffset + 1.5 -- Drückt dich 1.5 Studs höher wenn getroffen
                                        RyuNotify:Send("Anti-Hit", "Schaden erkannt! Gehe höher: " .. currentHeightOffset .. " Studs", 2)
                                    end
                                    myLastHealth = hum.Health
                                elseif hum.Health > myLastHealth then
                                    myLastHealth = hum.Health -- HP Reset (z.B. durch Regeneration)
                                end
                                
                                -- Update Position mit aktuellem Height Offset
                                skyPos = firstMobRoot.Position + Vector3.new(0, currentHeightOffset, 0)

                                local aliveCount = 0
                                local targetLook = nil
                                
                                for _, mob in pairs(aggroedMobs) do
                                    if mob and mob.Parent then
                                        local mHum = mob:FindFirstChildOfClass("Humanoid")
                                        local mRoot = mob:FindFirstChild("HumanoidRootPart")
                                        
                                        if mHum and mHum.Health > 0 and mRoot then
                                            aliveCount = aliveCount + 1
                                            if not targetLook then targetLook = mRoot.Position end
                                            
                                            -- Hitbox anpassen, damit man alle von oben trifft
                                            if mRoot.Size.Y < 30 then
                                                mRoot.Size = Vector3.new(30, 30, 30)
                                                mRoot.CanCollide = false
                                            end
                                        end
                                    end
                                end
                                
                                if aliveCount == 0 then break end 
                                
                                if targetLook then
                                    root.CFrame = CFrame.lookAt(skyPos, Vector3.new(targetLook.X, skyPos.Y, targetLook.Z))
                                end
                                
                                PerformAttack()
                                RunService.Heartbeat:Wait()
                            end
                            
                            for _, mob in pairs(aggroedMobs) do
                                local mRoot = mob and mob:FindFirstChild("HumanoidRootPart")
                                if mRoot then mRoot.Size = Vector3.new(2, 2, 1) end
                            end
                        end
                    end
                    
                end
            end
        end
    end
end)

task.wait(0.5)
RyuNotify:Send("RYU HUB", "PC Edition: Dynamic Anti-Hit System Active!", 4)
