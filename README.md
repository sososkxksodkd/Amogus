--// ==========================================
--// RYU HUB - GPO EDITION (TWEEN & LOGIC)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// PLATZHALTER-FUNKTION
local function Ryuhub()
    -- Für zukünftige Features
end

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "RyuHubPremium" then v:Destroy() end end

--// DYNAMISCHER WORKSPACE SCANNER FÜR INSELN
local function GetDynamicLists()
    local islands = {}
    local islandsFolder = Workspace:FindFirstChild("Islands")
    if islandsFolder then
        for _, v in pairs(islandsFolder:GetChildren()) do
            table.insert(islands, v.Name)
        end
    else
        islands = {
            "???? Shrine", "A rock", "Coco Island", "Colosseum", "Colosseum of Arc", 
            "Desert Kingdom", "Dokkan Island", "Fishman Cave", "Fishman Island", 
            "Foro Island", "Impel Base", "Marine Base G-1", "Marine Fort F-1", 
            "Mirror World", "Mysterious Cliff", "Mysterious Reef", "Orange Town", 
            "Restaurant Baratie", "Reverse Mountain", "Roca Island", "Rose Kingdom", 
            "Rovo Island", "Sakura Stronghold", "Sandora", "Sashi Island", 
            "Sett's Arena", "Shark Park", "Shell's Town", "Sphinx Island", 
            "Spirit Island", "Thriller Bark", "Town of Beginnings", 
            "Turtleback Cave", "Umi Island", "Whole Cake Island"
        }
    end
    table.sort(islands)
    return islands
end

local InitIslands = GetDynamicLists()

--// RYU CONFIGURATION (GPO)
local RyuConfig = {
    TargetIsland = InitIslands[1] or "Town of Beginnings",
    IslandSpeed = 65, -- Max 65 limit
    GuiColor = Color3.fromRGB(255, 255, 255)
}

--// PREMIUM MONOCHROME THEME
local Theme = {
    Background = Color3.fromRGB(12, 12, 14),
    Sidebar = Color3.fromRGB(18, 18, 20),
    SectionBG = Color3.fromRGB(24, 24, 26),
    Text = Color3.fromRGB(250, 250, 250),
    SubText = Color3.fromRGB(130, 130, 135),
    CloudLight = Color3.fromRGB(255, 255, 255),
    CloudDark = Color3.fromRGB(60, 60, 65),
    Accent = RyuConfig.GuiColor,
    ToggleOff = Color3.fromRGB(35, 35, 38),
    ToggleOn = RyuConfig.GuiColor,
    Stroke = Color3.fromRGB(45, 45, 50),
    Warning = Color3.fromRGB(255, 75, 75)
}

local isMobile = camera.ViewportSize.X < 850
local MainSize = UDim2.new(0, math.min(750, camera.ViewportSize.X - 40), 0, math.min(480, camera.ViewportSize.Y - 40))
local SidebarWidth = 160

local RyuHub = Instance.new("ScreenGui")
RyuHub.Name = "RyuHubPremium"
RyuHub.ResetOnSpawn = false
RyuHub.IgnoreGuiInset = true
RyuHub.Parent = guiParent

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

--// ANIMATION & UI HELPERS
local function AddHoverEffect(element, def, hov)
    element.MouseEnter:Connect(function() TweenService:Create(element, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = hov}):Play() end)
    element.MouseLeave:Connect(function() TweenService:Create(element, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = def}):Play() end)
end

local function AddClickPop(element)
    local orig = element.Size
    element.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            TweenService:Create(element, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(orig.X.Scale, orig.X.Offset-4, orig.Y.Scale, orig.Y.Offset-4)}):Play()
        end
    end)
    element.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            TweenService:Create(element, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Size = orig}):Play()
        end
    end)
end

--// TRADITIONAL KATANA TOGGLE
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 25, 0, 25)
ToggleBtn.BackgroundColor3 = Theme.Sidebar
ToggleBtn.Text = ""
ToggleBtn.Parent = RyuHub
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local btnStroke = Instance.new("UIStroke", ToggleBtn)
btnStroke.Color = Theme.Accent; btnStroke.Thickness = 2; btnStroke.Transparency = 0.5

local Katana = Instance.new("Frame", ToggleBtn)
Katana.Size = UDim2.new(1, 0, 1, 0); Katana.BackgroundTransparency = 1; Katana.Rotation = 45
local Blade = Instance.new("Frame", Katana)
Blade.Size = UDim2.new(0, 2, 0, 24); Blade.Position = UDim2.new(0.5, -1, 0.5, -18); Blade.BackgroundColor3 = Theme.CloudLight; Blade.BorderSizePixel = 0
local BladeGlow = Instance.new("UIStroke", Blade)
BladeGlow.Color = Theme.Accent; BladeGlow.Thickness = 1; BladeGlow.Transparency = 0.5
local Guard = Instance.new("Frame", Katana)
Guard.Size = UDim2.new(0, 12, 0, 2); Guard.Position = UDim2.new(0.5, -6, 0.5, 6); Guard.BackgroundColor3 = Theme.CloudDark; Guard.BorderSizePixel = 0
local Handle = Instance.new("Frame", Katana)
Handle.Size = UDim2.new(0, 4, 0, 10); Handle.Position = UDim2.new(0.5, -2, 0.5, 8); Handle.BackgroundColor3 = Color3.fromRGB(40, 45, 50); Handle.BorderSizePixel = 0
Instance.new("UICorner", Blade).CornerRadius = UDim.new(1, 0)
Instance.new("UICorner", Guard).CornerRadius = UDim.new(1, 0)
Instance.new("UICorner", Handle).CornerRadius = UDim.new(0, 1)

AddClickPop(ToggleBtn)
local tDragStart, tStartPos, isDraggingBtn = nil, nil, false
ToggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDraggingBtn = false; tDragStart = input.Position; tStartPos = ToggleBtn.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if tDragStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - tDragStart
        if delta.Magnitude > 5 then isDraggingBtn = true; ToggleBtn.Position = UDim2.new(tStartPos.X.Scale, tStartPos.X.Offset + delta.X, tStartPos.Y.Scale, tStartPos.Y.Offset + delta.Y) end
    end
end)

--// MAIN WINDOW FRAME
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 0, 0, 0); MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0; MainFrame.Active = true; MainFrame.Visible = false; MainFrame.ClipsDescendants = true
MainFrame.Parent = RyuHub
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Theme.Stroke
mainStroke.Transparency = 0.2
mainStroke.Thickness = 1.5

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if tDragStart then
            if not isDraggingBtn then
                if MainFrame.Visible then
                    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play()
                    task.wait(0.3); MainFrame.Visible = false
                else
                    MainFrame.Visible = true
                    TweenService:Create(MainFrame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = MainSize, Position = UDim2.new(0.5, -MainSize.X.Offset/2, 0.5, -MainSize.Y.Offset/2)}):Play()
                end
            end
            tDragStart = nil
        end
    end
end)

local Topbar = Instance.new("Frame", MainFrame)
Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1

local Title = Instance.new("TextLabel", Topbar)
Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1
Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextXAlignment = Enum.TextXAlignment.Left

local TitleGradient = Instance.new("UIGradient", Title)
TitleGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 180, 185)),   
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)), 
    ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 180, 185))    
}
TitleGradient.Offset = Vector2.new(-1, 0)

task.spawn(function()
    local tweenInfo = TweenInfo.new(2.0, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
    TweenService:Create(TitleGradient, tweenInfo, {Offset = Vector2.new(1, 0)}):Play()
end)

local SubTitle = Instance.new("TextLabel", Topbar)
SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 38); SubTitle.BackgroundTransparency = 1
SubTitle.Text = "Grand Piece Online"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 11; SubTitle.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG
CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.SubText; CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 14
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", CloseBtn).Color = Theme.Stroke
AddHoverEffect(CloseBtn, Theme.SectionBG, Theme.Warning)
CloseBtn.MouseButton1Click:Connect(function() 
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play()
    task.wait(0.3); MainFrame.Visible = false 
end)

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end
end)
Topbar.InputChanged:Connect(function(input)
    if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - mDragStart
        MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y)
    end
end)
Topbar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end
end)

local Line = Instance.new("Frame", MainFrame)
Line.Size = UDim2.new(1, -40, 0, 1); Line.Position = UDim2.new(0, 20, 0, 65); Line.BackgroundColor3 = Theme.Stroke; Line.BorderSizePixel = 0

-- SIDEBAR (LINKS)
local Sidebar = Instance.new("ScrollingFrame", MainFrame)
Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
SideLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- CONTENT CONTAINER (RECHTS)
local ContentContainer = Instance.new("Frame", MainFrame)
ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1

local DiscordLabel = Instance.new("TextLabel", MainFrame)
DiscordLabel.Size = UDim2.new(0, 150, 0, 20)
DiscordLabel.Position = UDim2.new(0, 15, 1, -30)
DiscordLabel.BackgroundTransparency = 1
DiscordLabel.Text = "DISCORD.GG/RYUHUB"
DiscordLabel.Font = Enum.Font.GothamBold
DiscordLabel.TextSize = 11
DiscordLabel.TextXAlignment = Enum.TextXAlignment.Left
DiscordLabel.TextTransparency = 0.05

local DiscordGradient = Instance.new("UIGradient", DiscordLabel)
DiscordGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 180, 185)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 180, 185))
}
DiscordGradient.Offset = Vector2.new(-1, 0)

task.spawn(function()
    local tweenInfo = TweenInfo.new(2.0, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
    TweenService:Create(DiscordGradient, tweenInfo, {Offset = Vector2.new(1, 0)}):Play()
end)

--// ACCORDEON-SYSTEM & UI BUILDERS
local Tabs = {}
local sidebarOrderCounter = 0
local itemOrderCounter = 0

local function UpdateSidebarCanvas()
    local totalH = 10
    for _, t in pairs(Tabs) do
        totalH = totalH + 36 + 6
        if t.IsOpen then
            totalH = totalH + t.SubLayout.AbsoluteContentSize.Y + 6
        end
    end
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH)
end

local function SecureTrigger(button, func)
    button.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            func()
        end
    end)
end

local function CreateMainTab(name)
    local tabObj = { Btn = nil, Arrow = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {}, ToggleFunc = nil }

    sidebarOrderCounter = sidebarOrderCounter + 1
    local tabBtn = Instance.new("TextButton", Sidebar)
    tabBtn.LayoutOrder = sidebarOrderCounter
    tabBtn.Size = UDim2.new(1, 0, 0, 36)
    tabBtn.BackgroundColor3 = Theme.Sidebar
    tabBtn.Text = "  " .. string.upper(name)
    tabBtn.TextColor3 = Theme.SubText
    tabBtn.Font = Enum.Font.GothamBlack
    tabBtn.TextSize = 13
    tabBtn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)
    tabObj.Btn = tabBtn

    local arrow = Instance.new("TextLabel", tabBtn)
    arrow.Size = UDim2.new(0, 20, 1, 0)
    arrow.Position = UDim2.new(1, -25, 0, 0)
    arrow.BackgroundTransparency = 1
    arrow.Text = "v"
    arrow.TextColor3 = Theme.SubText
    arrow.Font = Enum.Font.GothamBold
    arrow.TextSize = 12
    tabObj.Arrow = arrow

    sidebarOrderCounter = sidebarOrderCounter + 1
    local subContainer = Instance.new("Frame", Sidebar)
    subContainer.LayoutOrder = sidebarOrderCounter
    subContainer.Size = UDim2.new(1, 0, 0, 0)
    subContainer.BackgroundTransparency = 1
    subContainer.ClipsDescendants = true
    tabObj.SubContainer = subContainer

    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 2)
    subLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
    subLayout.SortOrder = Enum.SortOrder.LayoutOrder
    tabObj.SubLayout = subLayout

    tabObj.ToggleFunc = function()
        tabObj.IsOpen = not tabObj.IsOpen
        local targetSize = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)
        TweenService:Create(subContainer, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = targetSize}):Play()
        if tabObj.IsOpen then
            arrow.Text = "^"
            TweenService:Create(tabBtn, TweenInfo.new(0.25), {TextColor3 = Theme.Text, BackgroundColor3 = Theme.SectionBG}):Play()
            TweenService:Create(arrow, TweenInfo.new(0.25), {TextColor3 = Theme.Text}):Play()
        else
            arrow.Text = "v"
            TweenService:Create(tabBtn, TweenInfo.new(0.25), {TextColor3 = Theme.SubText, BackgroundColor3 = Theme.Sidebar}):Play()
            TweenService:Create(arrow, TweenInfo.new(0.25), {TextColor3 = Theme.SubText}):Play()
        end
        task.delay(0.26, UpdateSidebarCanvas)
        UpdateSidebarCanvas()
    end

    SecureTrigger(tabBtn, tabObj.ToggleFunc)

    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end
        UpdateSidebarCanvas()
    end)

    table.insert(Tabs, tabObj)
    return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subObj = { Btn = nil, Page = nil, Indicator = nil, SelectFunc = nil }

    local subBtn = Instance.new("TextButton", tabObj.SubContainer)
    subBtn.LayoutOrder = #tabObj.SubTabs + 1
    subBtn.Size = UDim2.new(1, 0, 0, 28)
    subBtn.BackgroundTransparency = 1
    subBtn.Text = "     " .. subName
    subBtn.TextColor3 = Theme.SubText
    subBtn.Font = Enum.Font.GothamMedium
    subBtn.TextSize = 12
    subBtn.TextXAlignment = Enum.TextXAlignment.Left
    subObj.Btn = subBtn

    local indicator = Instance.new("Frame", subBtn)
    indicator.Size = UDim2.new(0, 16, 0, 2)
    indicator.Position = UDim2.new(0, 20, 1, -4)
    indicator.BackgroundColor3 = Theme.Accent
    indicator.BorderSizePixel = 0
    indicator.BackgroundTransparency = 1
    Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0)
    subObj.Indicator = indicator

    local page = Instance.new("ScrollingFrame", ContentContainer)
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.ScrollBarThickness = 2
    page.ScrollBarImageColor3 = Theme.Accent
    page.Visible = false
    subObj.Page = page

    local pageLayout = Instance.new("UIListLayout", page)
    pageLayout.Padding = UDim.new(0, 12)
    pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20)
    end)

    subObj.SelectFunc = function()
        for _, t in pairs(Tabs) do
            for _, st in pairs(t.SubTabs) do
                st.Page.Visible = false
                TweenService:Create(st.Btn, TweenInfo.new(0.2), {TextColor3 = Theme.SubText}):Play()
                TweenService:Create(st.Indicator, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
            end
        end
        page.Visible = true
        TweenService:Create(subBtn, TweenInfo.new(0.2), {TextColor3 = Theme.Text}):Play()
        TweenService:Create(indicator, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end

    SecureTrigger(subBtn, subObj.SelectFunc)

    table.insert(tabObj.SubTabs, subObj)
    return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page)
    section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; section.BackgroundTransparency = 0
    Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    local sStroke = Instance.new("UIStroke", section); sStroke.Color = Theme.Stroke; sStroke.Transparency = 0.2
    
    local secLayout = Instance.new("UIListLayout", section)
    secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local secPadding = Instance.new("UIPadding", section)
    secPadding.PaddingTop = UDim.new(0, 12); secPadding.PaddingBottom = UDim.new(0, 12)
    
    local title = Instance.new("TextLabel", section)
    title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText
    title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end)
    return section
end

local function CreateToggle(section, text, descText, defaultState, callback)
    if type(defaultState) == "function" then
        callback = defaultState
        defaultState = false
    end
    
    itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section)
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, descText and 52 or 34); frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 0, 34); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = defaultState and Theme.Text or Theme.SubText
    label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    
    if descText then
        local descLabel = Instance.new("TextLabel", frame)
        descLabel.Size = UDim2.new(0.7, 0, 0, 15); descLabel.Position = UDim2.new(0, 0, 0, 30); descLabel.BackgroundTransparency = 1
        descLabel.Text = descText; descLabel.TextColor3 = Theme.SubText; descLabel.Font = Enum.Font.Gotham; descLabel.TextSize = 11; descLabel.TextXAlignment = Enum.TextXAlignment.Left
    end
    
    local tBtn = Instance.new("TextButton", frame)
    tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff; tBtn.Text = ""
    Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local bStroke = Instance.new("UIStroke", tBtn); bStroke.Color = defaultState and Theme.ToggleOn or Theme.Stroke; bStroke.Transparency = 0.2
    AddClickPop(tBtn)
    
    local circle = Instance.new("Frame", tBtn)
    circle.Size = UDim2.new(0, 16, 0, 16); circle.Position = defaultState and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
    circle.BackgroundColor3 = defaultState and Theme.Background or Color3.fromRGB(150, 150, 150)
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultState or false
    SecureTrigger(tBtn, function()
        isOn = not isOn
        if isOn then
            TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Theme.ToggleOn}):Play()
            TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(1, -19, 0.5, -8), BackgroundColor3 = Theme.Background}):Play()
            TweenService:Create(label, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextColor3 = Theme.Text}):Play()
            bStroke.Color = Theme.ToggleOn
        else
            TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Theme.ToggleOff}):Play()
            TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(0, 3, 0.5, -8), BackgroundColor3 = Color3.fromRGB(150, 150, 150)}):Play()
            TweenService:Create(label, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextColor3 = Theme.SubText}):Play()
            bStroke.Color = Theme.Stroke
        end
        if callback then callback(isOn) end
    end)
end

local function CreateSlider(section, text, min, max, default, callback)
    itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section)
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, 0, 0, 20); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.SubText
    label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    
    local valLabel = Instance.new("TextLabel", frame)
    valLabel.Size = UDim2.new(0, 40, 0, 20); valLabel.Position = UDim2.new(1, -40, 0, 0); valLabel.BackgroundTransparency = 1
    valLabel.Text = tostring(default); valLabel.TextColor3 = Theme.Accent; valLabel.Font = Enum.Font.GothamBold; valLabel.TextSize = 13; valLabel.TextXAlignment = Enum.TextXAlignment.Right
    
    local sliderBg = Instance.new("Frame", frame)
    sliderBg.Size = UDim2.new(1, 0, 0, 4); sliderBg.Position = UDim2.new(0, 0, 0, 32); sliderBg.BackgroundColor3 = Theme.ToggleOff
    Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    
    local sliderFill = Instance.new("Frame", sliderBg)
    local percentage = (default - min) / (max - min)
    sliderFill.Size = UDim2.new(percentage, 0, 1, 0); sliderFill.BackgroundColor3 = Theme.Accent
    Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    
    local knob = Instance.new("TextButton", sliderFill)
    knob.Size = UDim2.new(0, 14, 0, 14); knob.Position = UDim2.new(1, -7, 0.5, -7); knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255); knob.Text = ""
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    
    local dragging = false
    local function setSlider(value)
        local relative = math.clamp((value - min) / (max - min), 0, 1)
        valLabel.Text = tostring(value)
        TweenService:Create(sliderFill, TweenInfo.new(0.08, Enum.EasingStyle.Quad), {Size = UDim2.new(relative, 0, 1, 0)}):Play()
        if callback then callback(value) end
    end
    
    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(1, -9, 0.5, -9)}):Play()
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 14, 0, 14), Position = UDim2.new(1, -7, 0.5, -7)}):Play()
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local relative = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
            local value = math.floor(min + (max - min) * relative)
            setSlider(value)
        end
    end)
end

local function CreateButton(section, text, color, callback)
    itemOrderCounter = itemOrderCounter + 1
    local btn = Instance.new("TextButton", section)
    btn.LayoutOrder = itemOrderCounter; btn.Size = UDim2.new(0.92, 0, 0, 34); btn.BackgroundColor3 = color
    btn.Text = text; btn.TextColor3 = Color3.fromRGB(255,255,255); btn.Font = Enum.Font.GothamBold; btn.TextSize = 12
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    Instance.new("UIStroke", btn).Color = Theme.Stroke; Instance.new("UIStroke", btn).Transparency = 0.5
    AddClickPop(btn)
    SecureTrigger(btn, callback)
    return btn
end

--// SEARCHABLE DROPDOWN (GPO ISLANDS)
local function CreateSearchableDropdown(section, headerText, itemsList, targetConfigKey)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 200); frame.BackgroundTransparency = 1
    local header = Instance.new("TextLabel", frame); header.Size = UDim2.new(1, 0, 0, 20); header.BackgroundTransparency = 1; header.Text = headerText .. ": " .. tostring(RyuConfig[targetConfigKey] or "None"); header.TextColor3 = Theme.SubText; header.Font = Enum.Font.GothamMedium; header.TextSize = 12; header.TextXAlignment = Enum.TextXAlignment.Left
    
    local searchBox = Instance.new("TextBox", frame)
    searchBox.Size = UDim2.new(1, 0, 0, 26); searchBox.Position = UDim2.new(0, 0, 0, 25); searchBox.BackgroundColor3 = Theme.ToggleOff; searchBox.TextColor3 = Theme.Text; searchBox.PlaceholderText = "Island name"; searchBox.Font = Enum.Font.Gotham; searchBox.TextSize = 12; Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 4)
    
    local scroll = Instance.new("ScrollingFrame", frame); scroll.Size = UDim2.new(1, 0, 0, 135); scroll.Position = UDim2.new(0, 0, 0, 60); scroll.BackgroundColor3 = Theme.Background; scroll.ScrollBarThickness = 4; Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 6)
    local listLayout = Instance.new("UIListLayout", scroll); listLayout.Padding = UDim.new(0, 4); listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    
    local function populate(filter)
        for _, child in pairs(scroll:GetChildren()) do if child:IsA("TextButton") then child:Destroy() end end
        for _, itemName in ipairs(itemsList) do
            if filter == "" or string.lower(itemName):find(string.lower(filter)) then
                local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. tostring(itemName); btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
                SecureTrigger(btn, function() RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. tostring(itemName) end)
            end
        end
        task.defer(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
    end
    
    searchBox:GetPropertyChangedSignal("Text"):Connect(function() populate(searchBox.Text) end)
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
    populate("")
end

--// =======================
--// TABS & SECTIONS (GPO)
--// =======================

-- TAB 1: FARM
local TabFarm = CreateMainTab("FARM")
local SubAutoLevel = CreateSubTab(TabFarm, "Auto Level")
local SecAutoLevel = CreateSection(SubAutoLevel, "Auto Leveling (Placeholder)")
local phLevel = Instance.new("TextLabel", SecAutoLevel); phLevel.Size = UDim2.new(1, 0, 0, 30); phLevel.BackgroundTransparency = 1; phLevel.Text = "Coming Soon..."; phLevel.TextColor3 = Theme.SubText; phLevel.Font = Enum.Font.Gotham; phLevel.TextSize = 12

-- TAB 2: PLAYER
local TabPlayer = CreateMainTab("PLAYER")
local SubBR = CreateSubTab(TabPlayer, "Battle Royale")
local SecBR = CreateSection(SubBR, "Battle Royale Settings")
local phBR = Instance.new("TextLabel", SecBR); phBR.Size = UDim2.new(1, 0, 0, 30); phBR.BackgroundTransparency = 1; phBR.Text = "Coming Soon..."; phBR.TextColor3 = Theme.SubText; phBR.Font = Enum.Font.Gotham; phBR.TextSize = 12

-- TAB 3: MOBILITY
local TabMobility = CreateMainTab("MOBILITY")
local SubTween = CreateSubTab(TabMobility, "Tween")
local SubTeleport = CreateSubTab(TabMobility, "Teleport")
local SubAutoBuy = CreateSubTab(TabMobility, "Auto Buy")

local SecIslandTP = CreateSection(SubTween, "Spider Tween (Islands)")
CreateSearchableDropdown(SecIslandTP, "Selected Island", InitIslands, "TargetIsland")
CreateSlider(SecIslandTP, "Tween Speed (Max 65)", 10, 65, 65, function(val) RyuConfig.IslandSpeed = val end)

-- TAB 4: SETTINGS
local TabSettings = CreateMainTab("SETTINGS")
local SubConfig = CreateSubTab(TabSettings, "Configs")
local SecGui = CreateSection(SubConfig, "GUI Recolour")
local phGui = Instance.new("TextLabel", SecGui); phGui.Size = UDim2.new(1, 0, 0, 30); phGui.BackgroundTransparency = 1; phGui.Text = "Color Picker Coming Soon..."; phGui.TextColor3 = Theme.SubText; phGui.Font = Enum.Font.Gotham; phGui.TextSize = 12

--// =======================
--// SPIDER TWEEN LOGIC (GPO)
--// =======================

local function ToggleHover(state, root)
    if state then
        local bp = root:FindFirstChild("RyuHover")
        if not bp then
            bp = Instance.new("BodyPosition")
            bp.Name = "RyuHover"; bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.D = 500; bp.P = 50000; bp.Parent = root
        end
        bp.Position = root.Position
        return bp
    else
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp:Destroy() end
        return nil
    end
end

CreateButton(SecIslandTP, "Start Spider Tween", Theme.SectionBG, function()
    if _G.RyuIsTweening then return end
    _G.RyuIsTweening = true
    
    task.spawn(function()
        local targetIslandName = RyuConfig.TargetIsland
        local island = nil
        
        local islandsFolder = Workspace:FindFirstChild("Islands")
        if islandsFolder then
            for _, v in pairs(islandsFolder:GetChildren()) do
                if string.lower(v.Name) == string.lower(targetIslandName) then
                    island = v; break
                end
            end
        end
        
        if not island then _G.RyuIsTweening = false; return end
        
        local rawPos
        if island:IsA("Model") then rawPos = island:GetPivot().Position
        elseif island:IsA("BasePart") then rawPos = island.Position
        else
            local part = island:FindFirstChildWhichIsA("BasePart", true)
            if part then rawPos = part.Position else _G.RyuIsTweening = false; return end
        end
        
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then _G.RyuIsTweening = false; return end

        local targetPos = rawPos
        local bp = ToggleHover(true, root)
        
        local climbEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("climb")
        local sprintEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("sprint")
        local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
        
        if sprintEvent then pcall(function() sprintEvent:FireServer("rbxassetid://15382065457") end) end
        
        --// FAKE FLOOR FÜR PERFEKTE LAUF-ANIMATION //--
        local fakeFloor = Instance.new("Part")
        fakeFloor.Name = "RyuFakeFloor"
        fakeFloor.Size = Vector3.new(4, 1, 4)
        fakeFloor.Anchored = true
        fakeFloor.CanCollide = true
        fakeFloor.Transparency = 1
        fakeFloor.Parent = Workspace
        
        local function GetTrueTopY(x, z)
            local rParams = RaycastParams.new()
            local currentFilter = {char, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles"), fakeFloor}
            rParams.FilterType = Enum.RaycastFilterType.Exclude
            rParams.IgnoreWater = true
            
            local origin = Vector3.new(x, 4000, z)
            
            for i = 1, 10 do
                rParams.FilterDescendantsInstances = currentFilter
                local hit = Workspace:Raycast(origin, Vector3.new(0, -5000, 0), rParams)
                if hit then 
                    if hit.Instance.Transparency < 1 then
                        return hit.Position.Y 
                    else
                        table.insert(currentFilter, hit.Instance)
                    end
                else
                    break
                end
            end
            return 0 
        end

        local currentSpeed = RyuConfig.IslandSpeed
        local floorOffset = 5 
        local lastFootstep = tick()
        local isClimbingState = false
        
        local foundRobo = false
        local nextRoboCheck = tick()
        
        char:SetAttribute("evading", true)
        
        local elapsedTime = 0
        local flatStart = Vector3.new(root.Position.X, 0, root.Position.Z)
        local flatTarget = Vector3.new(targetPos.X, 0, targetPos.Z)
        local totalDist = (flatStart - flatTarget).Magnitude
        local t = totalDist / currentSpeed
        local currentY = root.Position.Y
        
        while elapsedTime < t do
            local dt = RunService.Heartbeat:Wait()
            dt = math.clamp(dt, 0.001, 0.05)
            
            local currentFlat = Vector3.new(root.Position.X, 0, root.Position.Z)
            local targetFlat = Vector3.new(targetPos.X, 0, targetPos.Z)
            local distToTarget = (targetFlat - currentFlat).Magnitude
            
            -- STRIKTER ABBRUCH-CHECK: Stoppt erst, wenn wir wirklich am Ziel sind (< 3 Studs)
            if distToTarget < 3 then 
                break 
            end
            
            local islandFlat = Vector3.new(rawPos.X, 0, rawPos.Z)
            local distToIsland = (currentFlat - islandFlat).Magnitude
            
            -- Smarte Robo Suche: Startet erst wenn wir auf 1500 Studs der Insel nah sind
            if not foundRobo and distToIsland < 1500 and tick() - nextRoboCheck > 1 then
                nextRoboCheck = tick()
                local npcsFolder = Workspace:FindFirstChild("NPCs")
                if npcsFolder then
                    local bestRobo = nil
                    local bestDist = math.huge
                    for _, v in pairs(npcsFolder:GetChildren()) do
                        if v.Name == "Robo" and v:IsA("Model") and v:FindFirstChild("HumanoidRootPart") then
                            local d = (v.HumanoidRootPart.Position - rawPos).Magnitude
                            if d <= 500 and d < bestDist then
                                bestDist = d
                                bestRobo = v
                            end
                        end
                    end
                    if bestRobo then
                        foundRobo = true
                        targetPos = bestRobo.HumanoidRootPart.Position
                        targetFlat = Vector3.new(targetPos.X, 0, targetPos.Z)
                    end
                end
            end
            
            local moveDir = (targetFlat - currentFlat).Unit
            if targetFlat == currentFlat or (targetFlat - currentFlat).Magnitude < 0.1 then
                moveDir = root.CFrame.LookVector
            end
            
            local currentX = root.Position.X + (moveDir.X * currentSpeed * dt)
            local currentZ = root.Position.Z + (moveDir.Z * currentSpeed * dt)
            local calcPos = Vector3.new(currentX, currentY, currentZ)
            
            local roofY = GetTrueTopY(currentX, currentZ) + floorOffset
            local groundY = GetTrueTopY(currentX, currentZ)
            local targetY = math.max(groundY + floorOffset, roofY)
            
            -- HARTER Y-LIMIT CHECK (Verhindert das Eintauchen ins Wasser oder Fallen unter -1)
            targetY = math.max(targetY, 5)
            
            local yVelocity = 0
            local addTime = dt

            -- 3 Stud Wall Check
            local rayParamsDown = RaycastParams.new()
            rayParamsDown.FilterDescendantsInstances = {char, Workspace:FindFirstChild("Effects"), fakeFloor}
            rayParamsDown.FilterType = Enum.RaycastFilterType.Exclude
            local wallHit = Workspace:Raycast(calcPos, moveDir * 3, rayParamsDown)
            
            if wallHit then
                local hitName = wallHit.Instance.Name
                local parentName = wallHit.Instance.Parent and wallHit.Instance.Parent.Name or ""
                if wallHit.Instance.Transparency < 1 and hitName ~= "Ocean" and parentName ~= "WaterStuff" then
                    local wallTopY = GetTrueTopY(wallHit.Position.X, wallHit.Position.Z) + floorOffset
                    if wallTopY > currentY then
                        targetY = math.max(targetY, wallTopY)
                    end
                end
            end
            
            local isWallInFront = (targetY > currentY + 1)
            if isWallInFront and not isClimbingState then
                isClimbingState = true
                if climbEvent then pcall(function() climbEvent:InvokeServer(true) end) end
            elseif not isWallInFront and isClimbingState then
                isClimbingState = false
                if climbEvent then pcall(function() climbEvent:InvokeServer(false) end) end
            end

            -- Vorwärts-Stop, wenn Wand blockiert
            if isWallInFront then 
                currentX = root.Position.X 
                currentZ = root.Position.Z 
            end 

            -- Zack-Hoch (600) vs Sicher-Runter (60)
            local safeVerticalSpeedUp = 600
            local safeVerticalSpeedDown = 60

            if currentY < targetY - 0.5 then
                currentY = math.min(currentY + (safeVerticalSpeedUp * dt), targetY)
                yVelocity = 20
            elseif currentY > targetY + 0.5 then
                currentY = math.max(currentY - (safeVerticalSpeedDown * dt), targetY)
                if currentY < (groundY + floorOffset) then currentY = groundY + floorOffset end
                yVelocity = -20
            else
                currentY = targetY
                yVelocity = 0
            end
            
            -- Absolute Absicherung: currentY darf NIEMALS negativ sein
            currentY = math.max(currentY, 0)
            
            local finalPos = Vector3.new(currentX, currentY, currentZ)
            bp.Position = finalPos
            root.CFrame = CFrame.lookAt(root.Position, Vector3.new(targetPos.X, root.Position.Y, targetPos.Z))
            
            -- Setze Fake Floor exakt unter die Füße des Spielers
            if fakeFloor then
                fakeFloor.CFrame = root.CFrame * CFrame.new(0, -((hum.HipHeight or 2) + (root.Size.Y / 2) + 0.05), 0)
            end
            
            -- ERZWINGT LAUF-ANIMATION & VERHINDERT FALL-ANIMATION
            if hum then 
                hum:ChangeState(Enum.HumanoidStateType.Running)
                hum:Move(moveDir, false) 
            end
            
            root.Velocity = Vector3.new(moveDir.X * currentSpeed, 0, moveDir.Z * currentSpeed)
            
            if tick() - lastFootstep > 0.35 then
                lastFootstep = tick()
                if footstepEvent then pcall(function() footstepEvent:FireServer() end) end
            end
        end
        
        -- Cleanup nach dem TP
        if fakeFloor then fakeFloor:Destroy() end
        if hum then hum:Move(Vector3.new(0,0,0), false) end
        if climbEvent and isClimbingState then pcall(function() climbEvent:InvokeServer(false) end) end
        ToggleHover(false, root)
        char:SetAttribute("evading", nil)
        root.Velocity = Vector3.new(0, 0, 0)
        
        _G.RyuIsTweening = false
    end)
end)

-- INITIALISIERUNG (SICHER)
task.spawn(function()
    if Tabs[3] and Tabs[3].ToggleFunc then Tabs[3].ToggleFunc() end
    if Tabs[3].SubTabs[1] and Tabs[3].SubTabs[1].SelectFunc then Tabs[3].SubTabs[1].SelectFunc() end
end)
