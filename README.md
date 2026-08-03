--// ==========================================
--// IMPEL DOWN SCRIPT (PREMIUM UI WITH 10+ SETTINGS)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "RyuHubPremium" then v:Destroy() end end

--// PREMIUM MONOCHROME THEME
local Theme = {
    Background = Color3.fromRGB(12, 12, 14),
    Sidebar = Color3.fromRGB(18, 18, 20),
    SectionBG = Color3.fromRGB(24, 24, 26),
    Text = Color3.fromRGB(250, 250, 250),
    SubText = Color3.fromRGB(130, 130, 135),
    Accent = Color3.fromRGB(255, 255, 255),
    ToggleOff = Color3.fromRGB(35, 35, 38),
    ToggleOn = Color3.fromRGB(255, 255, 255),
    Stroke = Color3.fromRGB(45, 45, 50),
    Warning = Color3.fromRGB(255, 75, 75)
}

local MainSize = UDim2.new(0, math.min(750, camera.ViewportSize.X - 40), 0, math.min(480, camera.ViewportSize.Y - 40))
local SidebarWidth = 160

local RyuHub = Instance.new("ScreenGui")
RyuHub.Name = "RyuHubPremium"
RyuHub.ResetOnSpawn = false
RyuHub.IgnoreGuiInset = true
RyuHub.Parent = guiParent

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

--// IMAGE BUTTON TOGGLE (GEFIXT MIT RBXTHUMB)
local ToggleBtn = Instance.new("ImageButton")
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 25, 0, 25)
ToggleBtn.BackgroundColor3 = Theme.Sidebar
-- Der rbxthumb Fix zwingt Roblox dazu, auch Decal-IDs in Bilder umzuwandeln
ToggleBtn.Image = "rbxthumb://type=Asset&id=6050149849&w=150&h=150" 
ToggleBtn.Parent = RyuHub
ToggleBtn.ScaleType = Enum.ScaleType.Crop
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local btnStroke = Instance.new("UIStroke", ToggleBtn)
btnStroke.Color = Theme.Accent; btnStroke.Thickness = 2; btnStroke.Transparency = 0.5

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

-- Custom Background Image
local MainBgImage = Instance.new("ImageLabel", MainFrame)
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = 1 
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.ZIndex = 0

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
Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1; Topbar.ZIndex = 2

-- RYU HUB OBEN
local Title = Instance.new("TextLabel", Topbar)
Title.Size = UDim2.new(0, 300, 0, 24); Title.Position = UDim2.new(0, 20, 0, 12); Title.BackgroundTransparency = 1
Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextXAlignment = Enum.TextXAlignment.Left; Title.ZIndex = 2

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

-- IMPEL DOWN SCRIPT UNTEN
local SubTitle = Instance.new("TextLabel", Topbar)
SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 36); SubTitle.BackgroundTransparency = 1
SubTitle.Text = "IMPEL DOWN SCRIPT"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 12; SubTitle.TextXAlignment = Enum.TextXAlignment.Left; SubTitle.ZIndex = 2

local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG
CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.SubText; CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 14; CloseBtn.ZIndex = 2
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
Line.Size = UDim2.new(1, -40, 0, 1); Line.Position = UDim2.new(0, 20, 0, 65); Line.BackgroundColor3 = Theme.Stroke; Line.BorderSizePixel = 0; Line.ZIndex = 2

-- SIDEBAR (LINKS)
local Sidebar = Instance.new("ScrollingFrame", MainFrame)
Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0; Sidebar.ZIndex = 2
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
SideLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- CONTENT CONTAINER (RECHTS)
local ContentContainer = Instance.new("Frame", MainFrame)
ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1; ContentContainer.ZIndex = 2

local DiscordLabel = Instance.new("TextLabel", MainFrame)
DiscordLabel.Size = UDim2.new(0, 150, 0, 20)
DiscordLabel.Position = UDim2.new(0, 15, 1, -30)
DiscordLabel.BackgroundTransparency = 1
DiscordLabel.Text = "DISCORD.GG/RYUHUB"
DiscordLabel.Font = Enum.Font.GothamBold
DiscordLabel.TextSize = 11
DiscordLabel.TextXAlignment = Enum.TextXAlignment.Left
DiscordLabel.TextTransparency = 0.05
DiscordLabel.ZIndex = 2

local DiscordGradient = Instance.new("UIGradient", DiscordLabel)
DiscordGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 180, 185)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 180, 185))
}
DiscordGradient.Offset = Vector2.new(-1, 0)

task.spawn(function()
    TweenService:Create(DiscordGradient, TweenInfo.new(2.0, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Offset = Vector2.new(1, 0)}):Play()
end)

--// HIERARCHISCHES ACCORDEON-SYSTEM
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

local function CreateMainTab(name)
    local tabObj = { Btn = nil, Arrow = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {}, Toggle = nil }

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
    tabBtn.ZIndex = 2
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
    arrow.ZIndex = 2
    tabObj.Arrow = arrow

    sidebarOrderCounter = sidebarOrderCounter + 1
    local subContainer = Instance.new("Frame", Sidebar)
    subContainer.LayoutOrder = sidebarOrderCounter
    subContainer.Size = UDim2.new(1, 0, 0, 0)
    subContainer.BackgroundTransparency = 1
    subContainer.ClipsDescendants = true
    subContainer.ZIndex = 2
    tabObj.SubContainer = subContainer

    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 2)
    subLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
    subLayout.SortOrder = Enum.SortOrder.LayoutOrder
    tabObj.SubLayout = subLayout

    local function toggleTab()
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

    tabBtn.MouseButton1Click:Connect(toggleTab)
    tabObj.Toggle = toggleTab

    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end
        UpdateSidebarCanvas()
    end)

    table.insert(Tabs, tabObj)
    return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subObj = { Btn = nil, Page = nil, Indicator = nil, Open = nil }

    local subBtn = Instance.new("TextButton", tabObj.SubContainer)
    subBtn.LayoutOrder = #tabObj.SubTabs + 1
    subBtn.Size = UDim2.new(1, 0, 0, 28)
    subBtn.BackgroundTransparency = 1
    subBtn.Text = "     " .. subName
    subBtn.TextColor3 = Theme.SubText
    subBtn.Font = Enum.Font.GothamMedium
    subBtn.TextSize = 12
    subBtn.TextXAlignment = Enum.TextXAlignment.Left
    subBtn.ZIndex = 2
    subObj.Btn = subBtn

    local indicator = Instance.new("Frame", subBtn)
    indicator.Size = UDim2.new(0, 16, 0, 2)
    indicator.Position = UDim2.new(0, 20, 1, -4)
    indicator.BackgroundColor3 = Theme.Accent
    indicator.BorderSizePixel = 0
    indicator.BackgroundTransparency = 1
    indicator.ZIndex = 2
    Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0)
    subObj.Indicator = indicator

    local page = Instance.new("ScrollingFrame", ContentContainer)
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.ScrollBarThickness = 2
    page.ScrollBarImageColor3 = Theme.Accent
    page.Visible = false
    page.ZIndex = 2
    subObj.Page = page

    local pageLayout = Instance.new("UIListLayout", page)
    pageLayout.Padding = UDim.new(0, 12)
    pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20)
    end)

    local function openSubTab()
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

    subBtn.MouseButton1Click:Connect(openSubTab)
    subObj.Open = openSubTab

    table.insert(tabObj.SubTabs, subObj)
    return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page)
    section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; section.BackgroundTransparency = 0
    section.ZIndex = 2
    Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    local sStroke = Instance.new("UIStroke", section); sStroke.Color = Theme.Stroke; sStroke.Transparency = 0.2
    
    local secLayout = Instance.new("UIListLayout", section)
    secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local secPadding = Instance.new("UIPadding", section)
    secPadding.PaddingTop = UDim.new(0, 12); secPadding.PaddingBottom = UDim.new(0, 12)
    
    local title = Instance.new("TextLabel", section)
    title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText
    title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left; title.ZIndex = 2
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
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, descText and 52 or 34); frame.BackgroundTransparency = 1; frame.ZIndex = 2
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 0, 34); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = defaultState and Theme.Text or Theme.SubText
    label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left; label.ZIndex = 2
    
    if descText then
        local descLabel = Instance.new("TextLabel", frame)
        descLabel.Size = UDim2.new(0.7, 0, 0, 15); descLabel.Position = UDim2.new(0, 0, 0, 30); descLabel.BackgroundTransparency = 1
        descLabel.Text = descText; descLabel.TextColor3 = Theme.SubText; descLabel.Font = Enum.Font.Gotham; descLabel.TextSize = 11; descLabel.TextXAlignment = Enum.TextXAlignment.Left; descLabel.ZIndex = 2
    end
    
    local tBtn = Instance.new("TextButton", frame)
    tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff; tBtn.Text = ""; tBtn.ZIndex = 2
    Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local bStroke = Instance.new("UIStroke", tBtn); bStroke.Color = defaultState and Theme.ToggleOn or Theme.Stroke; bStroke.Transparency = 0.2
    AddClickPop(tBtn)
    
    local circle = Instance.new("Frame", tBtn)
    circle.Size = UDim2.new(0, 16, 0, 16); circle.Position = defaultState and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
    circle.BackgroundColor3 = defaultState and Theme.Background or Color3.fromRGB(150, 150, 150); circle.ZIndex = 2
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultState or false
    tBtn.MouseButton1Click:Connect(function()
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
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1; frame.ZIndex = 2
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, 0, 0, 20); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.SubText
    label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left; label.ZIndex = 2
    
    local valLabel = Instance.new("TextLabel", frame)
    valLabel.Size = UDim2.new(0, 40, 0, 20); valLabel.Position = UDim2.new(1, -40, 0, 0); valLabel.BackgroundTransparency = 1
    valLabel.Text = tostring(default); valLabel.TextColor3 = Theme.Accent; valLabel.Font = Enum.Font.GothamBold; valLabel.TextSize = 13; valLabel.TextXAlignment = Enum.TextXAlignment.Right; valLabel.ZIndex = 2
    
    local sliderBg = Instance.new("Frame", frame)
    sliderBg.Size = UDim2.new(1, 0, 0, 4); sliderBg.Position = UDim2.new(0, 0, 0, 32); sliderBg.BackgroundColor3 = Theme.ToggleOff; sliderBg.ZIndex = 2
    Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    
    local sliderFill = Instance.new("Frame", sliderBg)
    local percentage = (default - min) / (max - min)
    sliderFill.Size = UDim2.new(percentage, 0, 1, 0); sliderFill.BackgroundColor3 = Theme.Accent; sliderFill.ZIndex = 2
    Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    
    local knob = Instance.new("TextButton", sliderFill)
    knob.Size = UDim2.new(0, 14, 0, 14); knob.Position = UDim2.new(1, -7, 0.5, -7); knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255); knob.Text = ""; knob.ZIndex = 2
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
    btn.Text = text; btn.TextColor3 = Color3.fromRGB(255,255,255); btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.ZIndex = 2
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    Instance.new("UIStroke", btn).Color = Theme.Stroke; Instance.new("UIStroke", btn).Transparency = 0.5
    AddClickPop(btn)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function CreateTextBox(section, placeholder, callback)
    itemOrderCounter = itemOrderCounter + 1
    local box = Instance.new("TextBox", section)
    box.LayoutOrder = itemOrderCounter; box.Size = UDim2.new(0.92, 0, 0, 34); box.BackgroundColor3 = Theme.Background
    box.Text = ""; box.PlaceholderText = placeholder; box.TextColor3 = Theme.Text; box.Font = Enum.Font.GothamMedium; box.TextSize = 12
    box.ClearTextOnFocus = false; box.ClipsDescendants = true; box.ZIndex = 2
    Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
    Instance.new("UIStroke", box).Color = Theme.Stroke
    if callback then box.FocusLost:Connect(function() callback(box.Text) end) end
    return box
end

--// =======================
--// UI POPULATION 
--// =======================

local function Ryuhub()
    -- Platzhalter für Logik
end

-- TAB 1: AUTO PLAY
local TabAutoPlay = CreateMainTab("Auto Play")

local SubAutoMain = CreateSubTab(TabAutoPlay, "Main")
local SecAutoPlay = CreateSection(SubAutoMain, "Impel Down Engine")
CreateToggle(SecAutoPlay, "Enable Auto Impel Down", "Automatically clear all stages", false, Ryuhub)
CreateToggle(SecAutoPlay, "Auto Next Stage", "Proceeds to the next floor automatically", false, Ryuhub)
CreateToggle(SecAutoPlay, "Auto Boss Farm", "Targets boss NPCs prioritized", false, Ryuhub)
CreateSlider(SecAutoPlay, "Farm Distance", 5, 50, 15, Ryuhub)

local SecAutoSkills = CreateSection(SubAutoMain, "Auto Skills")
CreateToggle(SecAutoSkills, "Auto Use Skill 1", nil, false, Ryuhub)
CreateToggle(SecAutoSkills, "Auto Use Skill 2", nil, false, Ryuhub)
CreateToggle(SecAutoSkills, "Auto Use Ultimate", "Uses ultimate when available", false, Ryuhub)


-- TAB 2: PLAYER
local TabPlayer = CreateMainTab("Player")

local SubMovement = CreateSubTab(TabPlayer, "Movement")
local SecMovement = CreateSection(SubMovement, "Local Player Settings")
CreateToggle(SecMovement, "Enable WalkSpeed", nil, false, Ryuhub)
CreateSlider(SecMovement, "Walk Speed", 16, 150, 35, Ryuhub)
CreateToggle(SecMovement, "Enable JumpPower", nil, false, Ryuhub)
CreateSlider(SecMovement, "Jump Power", 50, 250, 50, Ryuhub)

local SecVisuals = CreateSection(SubMovement, "Visuals & Utility")
CreateToggle(SecVisuals, "Noclip", "Walk through walls", false, Ryuhub)
CreateToggle(SecVisuals, "Infinite Stamina", nil, false, Ryuhub)
CreateToggle(SecVisuals, "Item ESP", "Shows rare items in Impel Down", false, Ryuhub)


-- TAB 3: SETTINGS
local TabSettings = CreateMainTab("Settings")

local SubClient = CreateSubTab(TabSettings, "Client")
local SecClient = CreateSection(SubClient, "System Configuration")
CreateToggle(SecClient, "Anti-AFK Protection", "Prevents Roblox from kicking you for idling", false, function(state)
    -- Anti AFK Logik Platzhalter
end)

local SubTheme = CreateSubTab(TabSettings, "Theme & UI")

--// THEME SETTINGS: WINDOW
local SecWindow = CreateSection(SubTheme, "Window Personalization")

CreateToggle(SecWindow, "Glass Mode", "Beautiful frosted glass transparency", false, function(state)
    if state then
        TweenService:Create(MainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0.35, BackgroundColor3 = Color3.fromRGB(30, 30, 35)}):Play()
        TweenService:Create(mainStroke, TweenInfo.new(0.3), {Transparency = 0.6}):Play()
    else
        TweenService:Create(MainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0, BackgroundColor3 = Theme.Background}):Play()
        TweenService:Create(mainStroke, TweenInfo.new(0.3), {Transparency = 0.2}):Play()
    end
end)

CreateSlider(SecWindow, "Window Roundness", 0, 24, 12, function(val)
    for _, corner in pairs(MainFrame:GetChildren()) do
        if corner:IsA("UICorner") then corner.CornerRadius = UDim.new(0, val) end
    end
end)

CreateToggle(SecWindow, "Minimalist Borders", "Hides the outer window stroke", false, function(state)
    mainStroke.Enabled = not state
end)

CreateTextBox(SecWindow, "Background Image ID (e.g. 12345)", function(txt)
    if txt and txt ~= "" then
        local num = txt:match("%d+")
        if num then
            MainBgImage.Image = "rbxthumb://type=Asset&id="..num.."&w=768&h=432"
            MainBgImage.ImageTransparency = 0.6
        end
    else
        MainBgImage.ImageTransparency = 1
    end
end)

CreateSlider(SecWindow, "Background Visibility", 0, 100, 40, function(val)
    if MainBgImage.Image ~= "" then
        MainBgImage.ImageTransparency = 1 - (val / 100)
    end
end)

--// THEME SETTINGS: TOGGLE BUTTON
local SecToggleUI = CreateSection(SubTheme, "Toggle Button Personalization")

CreateTextBox(SecToggleUI, "Custom Icon ID (e.g. 6050149849)", function(txt)
    if txt and txt ~= "" then
        local num = txt:match("%d+")
        if num then ToggleBtn.Image = "rbxthumb://type=Asset&id="..num.."&w=150&h=150" end
    end
end)

CreateSlider(SecToggleUI, "Toggle Button Size", 30, 80, 50, function(val)
    TweenService:Create(ToggleBtn, TweenInfo.new(0.2), {Size = UDim2.new(0, val, 0, val)}):Play()
end)

CreateSlider(SecToggleUI, "Toggle Glow Intensity", 0, 100, 50, function(val)
    btnStroke.Transparency = 1 - (val / 100)
end)

local rainbowToggle = false
CreateToggle(SecToggleUI, "RGB Rainbow Ring", "Animates the toggle button border", false, function(state)
    rainbowToggle = state
    if not state then btnStroke.Color = Theme.Accent end
end)
task.spawn(function()
    while true do
        if rainbowToggle then
            btnStroke.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1)
        end
        task.wait(0.1)
    end
end)

CreateToggle(SecToggleUI, "Floating Icon Mode", "Removes the toggle button background", false, function(state)
    if state then
        TweenService:Create(ToggleBtn, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
    else
        TweenService:Create(ToggleBtn, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end
end)

local SecSave = CreateSection(SubTheme, "Save Config")
CreateButton(SecSave, "Save Theme & Settings", Color3.fromRGB(50, 150, 50), function()
    -- Save Logik Platzhalter
end)

-- INITIALISIERUNG
task.spawn(function()
    Tabs[1].Toggle()
    Tabs[1].SubTabs[1].Open()
end)

-- MOBILE FLY DOCK
local FlyDock = Instance.new("Frame", RyuHub)
FlyDock.Size = UDim2.new(0, 180, 0, 120); FlyDock.Position = UDim2.new(0.7, 0, 0.5, 0); FlyDock.BackgroundColor3 = Theme.Background
FlyDock.Visible = false; FlyDock.Active = true; FlyDock.Draggable = true
Instance.new("UICorner", FlyDock).CornerRadius = UDim.new(0, 8); Instance.new("UIStroke", FlyDock).Color = Theme.Accent
local function CreateDockBtn(txt, pos, size)
    local btn = Instance.new("TextButton", FlyDock)
    btn.Size = size; btn.Position = pos; btn.BackgroundColor3 = Theme.ToggleOff; btn.Font = Enum.Font.GothamBold; btn.Text = txt; btn.TextColor3 = Color3.fromRGB(255,255,255); btn.TextSize = 14
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    btn.MouseButton1Click:Connect(Ryuhub)
end
CreateDockBtn("W", UDim2.new(0.3, 0, 0.08, 0), UDim2.new(0, 32, 0, 32))
CreateDockBtn("S", UDim2.new(0.3, 0, 0.62, 0), UDim2.new(0, 32, 0, 32))
CreateDockBtn("A", UDim2.new(0.08, 0, 0.35, 0), UDim2.new(0, 32, 0, 32))
CreateDockBtn("D", UDim2.new(0.52, 0, 0.35, 0), UDim2.new(0, 32, 0, 32))
CreateDockBtn("UP", UDim2.new(0.78, 0, 0.12, 0), UDim2.new(0, 30, 0, 38))
CreateDockBtn("DOWN", UDim2.new(0.78, 0, 0.52, 0), UDim2.new(0, 30, 0, 38))
