--// ==========================================
--// IMPEL DOWN SCRIPT (ULTIMATE PREMIUM UI WITH AUTO-SAVE & IMPEL ENGINE V2)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "RyuHubPremium" then v:Destroy() end end

--// CONFIG SAVE SYSTEM
local configFileName = "RyuHub_ImpelDown_Config.json"
local RyuSavedConfig = {
    GlassMode = false,
    WorldBlur = false,
    AccentColor = {255, 255, 255},
    BgColor = {12, 12, 14},
    Font = "Gotham",
    HideBorders = false,
    Roundness = 12,
    BgImage = "",
    BgOpacity = 0.6,
    ToggleIcon = "rbxthumb://type=Asset&id=6050149849&w=150&h=150",
    ToggleSize = 50,
    ToggleGlow = 0.5,
    RainbowMode = false,
    FloatingIcon = false,
    AutoImpelDown = false,
    ImpelFarmDistance = 15
}

if readfile and isfile and isfile(configFileName) then
    pcall(function()
        local data = HttpService:JSONDecode(readfile(configFileName))
        for k, v in pairs(data) do RyuSavedConfig[k] = v end
    end)
end

local function SaveConfig()
    if writefile then
        pcall(function() writefile(configFileName, HttpService:JSONEncode(RyuSavedConfig)) end)
    end
end

--// PREMIUM MONOCHROME THEME
local Theme = {
    Background = Color3.fromRGB(RyuSavedConfig.BgColor[1], RyuSavedConfig.BgColor[2], RyuSavedConfig.BgColor[3]),
    Sidebar = Color3.fromRGB(18, 18, 20),
    SectionBG = Color3.fromRGB(24, 24, 26),
    Text = Color3.fromRGB(250, 250, 250),
    SubText = Color3.fromRGB(130, 130, 135),
    Accent = Color3.fromRGB(RyuSavedConfig.AccentColor[1], RyuSavedConfig.AccentColor[2], RyuSavedConfig.AccentColor[3]),
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

local UIBlur = Instance.new("BlurEffect")
UIBlur.Size = 0
UIBlur.Parent = Lighting

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

--// IMAGE BUTTON TOGGLE
local ToggleBtn = Instance.new("ImageButton")
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 25, 0, 25)
ToggleBtn.BackgroundColor3 = Theme.Sidebar
ToggleBtn.Image = RyuSavedConfig.ToggleIcon 
ToggleBtn.Parent = RyuHub
ToggleBtn.ScaleType = Enum.ScaleType.Crop
ToggleBtn.ClipsDescendants = true
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
local MainCorner = Instance.new("UICorner", MainFrame)
MainCorner.CornerRadius = UDim.new(0, 12)

local MainBgImage = Instance.new("ImageLabel", MainFrame)
MainBgImage.Size = UDim2.new(1, 0, 1, 0); MainBgImage.BackgroundTransparency = 1; MainBgImage.ImageTransparency = 1 
MainBgImage.ScaleType = Enum.ScaleType.Crop; MainBgImage.ZIndex = 0

local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Theme.Stroke; mainStroke.Transparency = 0.2; mainStroke.Thickness = 1.5

local isUIOpen = false
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if tDragStart then
            if not isDraggingBtn then
                if MainFrame.Visible then
                    isUIOpen = false
                    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play()
                    TweenService:Create(UIBlur, TweenInfo.new(0.3), {Size = 0}):Play()
                    task.wait(0.3); MainFrame.Visible = false
                else
                    isUIOpen = true; MainFrame.Visible = true
                    TweenService:Create(MainFrame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = MainSize, Position = UDim2.new(0.5, -MainSize.X.Offset/2, 0.5, -MainSize.Y.Offset/2)}):Play()
                    if _G.BlurEnabled then TweenService:Create(UIBlur, TweenInfo.new(0.35), {Size = 15}):Play() end
                end
            end
            tDragStart = nil
        end
    end
end)

local Topbar = Instance.new("Frame", MainFrame)
Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1; Topbar.ZIndex = 2

local Title = Instance.new("TextLabel", Topbar)
Title.Size = UDim2.new(0, 300, 0, 24); Title.Position = UDim2.new(0, 20, 0, 12); Title.BackgroundTransparency = 1
Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextXAlignment = Enum.TextXAlignment.Left; Title.ZIndex = 2
local TitleGradient = Instance.new("UIGradient", Title)
TitleGradient.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 180, 185)), ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)), ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 180, 185))}
TitleGradient.Offset = Vector2.new(-1, 0)
task.spawn(function() TweenService:Create(TitleGradient, TweenInfo.new(2.0, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Offset = Vector2.new(1, 0)}):Play() end)

local SubTitle = Instance.new("TextLabel", Topbar)
SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 36); SubTitle.BackgroundTransparency = 1
SubTitle.Text = "IMPEL DOWN SCRIPT"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 12; SubTitle.TextXAlignment = Enum.TextXAlignment.Left; SubTitle.ZIndex = 2

local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -40, 0, 15); CloseBtn.BackgroundColor3 = Theme.SectionBG
CloseBtn.Text = "X"; CloseBtn.TextColor3 = Theme.SubText; CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 14; CloseBtn.ZIndex = 2
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
local closeStroke = Instance.new("UIStroke", CloseBtn); closeStroke.Color = Theme.Stroke
AddHoverEffect(CloseBtn, Theme.SectionBG, Theme.Warning)
CloseBtn.MouseButton1Click:Connect(function() 
    isUIOpen = false
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play()
    TweenService:Create(UIBlur, TweenInfo.new(0.3), {Size = 0}):Play()
    task.wait(0.3); MainFrame.Visible = false 
end)

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end end)
Topbar.InputChanged:Connect(function(input) if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - mDragStart; MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y) end end)
Topbar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end end)

local Line = Instance.new("Frame", MainFrame)
Line.Size = UDim2.new(1, -40, 0, 1); Line.Position = UDim2.new(0, 20, 0, 65); Line.BackgroundColor3 = Theme.Stroke; Line.BorderSizePixel = 0; Line.ZIndex = 2

local Sidebar = Instance.new("ScrollingFrame", MainFrame)
Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0; Sidebar.ZIndex = 2
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; SideLayout.SortOrder = Enum.SortOrder.LayoutOrder

local ContentContainer = Instance.new("Frame", MainFrame)
ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1; ContentContainer.ZIndex = 2

local DiscordLabel = Instance.new("TextLabel", MainFrame)
DiscordLabel.Size = UDim2.new(0, 150, 0, 20); DiscordLabel.Position = UDim2.new(0, 15, 1, -30); DiscordLabel.BackgroundTransparency = 1
DiscordLabel.Text = "DISCORD.GG/RYUHUB"; DiscordLabel.Font = Enum.Font.GothamBold; DiscordLabel.TextSize = 11; DiscordLabel.TextXAlignment = Enum.TextXAlignment.Left; DiscordLabel.TextTransparency = 0.05; DiscordLabel.ZIndex = 2
local DiscordGradient = Instance.new("UIGradient", DiscordLabel)
DiscordGradient.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 180, 185)), ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)), ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 180, 185))}
DiscordGradient.Offset = Vector2.new(-1, 0)
task.spawn(function() TweenService:Create(DiscordGradient, TweenInfo.new(2.0, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Offset = Vector2.new(1, 0)}):Play() end)

--// UI SYSTEM BUILDERS
local Tabs = {}
local sidebarOrderCounter, itemOrderCounter = 0, 0

local function UpdateSidebarCanvas()
    local totalH = 10
    for _, t in pairs(Tabs) do totalH = totalH + 36 + 6; if t.IsOpen then totalH = totalH + t.SubLayout.AbsoluteContentSize.Y + 6 end end
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH)
end

local function CreateMainTab(name)
    local tabObj = { Btn = nil, Arrow = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {}, Toggle = nil }
    sidebarOrderCounter = sidebarOrderCounter + 1
    local tabBtn = Instance.new("TextButton", Sidebar)
    tabBtn.LayoutOrder = sidebarOrderCounter; tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name)
    tabBtn.TextColor3 = Theme.SubText; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 13; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; tabBtn.ZIndex = 2
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8); tabObj.Btn = tabBtn
    local arrow = Instance.new("TextLabel", tabBtn)
    arrow.Size = UDim2.new(0, 20, 1, 0); arrow.Position = UDim2.new(1, -25, 0, 0); arrow.BackgroundTransparency = 1; arrow.Text = "v"; arrow.TextColor3 = Theme.SubText; arrow.Font = Enum.Font.GothamBold; arrow.TextSize = 12; arrow.ZIndex = 2; tabObj.Arrow = arrow
    sidebarOrderCounter = sidebarOrderCounter + 1
    local subContainer = Instance.new("Frame", Sidebar)
    subContainer.LayoutOrder = sidebarOrderCounter; subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; subContainer.ZIndex = 2; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 2); subLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; subLayout.SortOrder = Enum.SortOrder.LayoutOrder; tabObj.SubLayout = subLayout
    local function toggleTab()
        tabObj.IsOpen = not tabObj.IsOpen
        TweenService:Create(subContainer, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)}):Play()
        if tabObj.IsOpen then
            arrow.Text = "^"; TweenService:Create(tabBtn, TweenInfo.new(0.25), {TextColor3 = Theme.Text, BackgroundColor3 = Theme.SectionBG}):Play(); TweenService:Create(arrow, TweenInfo.new(0.25), {TextColor3 = Theme.Text}):Play()
        else
            arrow.Text = "v"; TweenService:Create(tabBtn, TweenInfo.new(0.25), {TextColor3 = Theme.SubText, BackgroundColor3 = Theme.Sidebar}):Play(); TweenService:Create(arrow, TweenInfo.new(0.25), {TextColor3 = Theme.SubText}):Play()
        end
        task.delay(0.26, UpdateSidebarCanvas); UpdateSidebarCanvas()
    end
    tabBtn.MouseButton1Click:Connect(toggleTab); tabObj.Toggle = toggleTab
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end; UpdateSidebarCanvas() end)
    table.insert(Tabs, tabObj); return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subObj = { Btn = nil, Page = nil, Indicator = nil, Open = nil }
    local subBtn = Instance.new("TextButton", tabObj.SubContainer)
    subBtn.LayoutOrder = #tabObj.SubTabs + 1; subBtn.Size = UDim2.new(1, 0, 0, 28); subBtn.BackgroundTransparency = 1; subBtn.Text = "     " .. subName; subBtn.TextColor3 = Theme.SubText; subBtn.Font = Enum.Font.GothamMedium; subBtn.TextSize = 12; subBtn.TextXAlignment = Enum.TextXAlignment.Left; subBtn.ZIndex = 2; subObj.Btn = subBtn
    local indicator = Instance.new("Frame", subBtn)
    indicator.Size = UDim2.new(0, 16, 0, 2); indicator.Position = UDim2.new(0, 20, 1, -4); indicator.BackgroundColor3 = Theme.Accent; indicator.BorderSizePixel = 0; indicator.BackgroundTransparency = 1; indicator.ZIndex = 2; Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0); subObj.Indicator = indicator
    local page = Instance.new("ScrollingFrame", ContentContainer)
    page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.ScrollBarThickness = 2; page.ScrollBarImageColor3 = Theme.Accent; page.Visible = false; page.ZIndex = 2; subObj.Page = page
    local pageLayout = Instance.new("UIListLayout", page)
    pageLayout.Padding = UDim.new(0, 12); pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20) end)
    local function openSubTab()
        for _, t in pairs(Tabs) do for _, st in pairs(t.SubTabs) do st.Page.Visible = false; TweenService:Create(st.Btn, TweenInfo.new(0.2), {TextColor3 = Theme.SubText}):Play(); TweenService:Create(st.Indicator, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play() end end
        page.Visible = true; TweenService:Create(subBtn, TweenInfo.new(0.2), {TextColor3 = Theme.Text}):Play(); TweenService:Create(indicator, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end
    subBtn.MouseButton1Click:Connect(openSubTab); subObj.Open = openSubTab
    table.insert(tabObj.SubTabs, subObj); return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page)
    section.Name = "SectionContainer"; section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; section.BackgroundTransparency = 0; section.ZIndex = 2; Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    local sStroke = Instance.new("UIStroke", section); sStroke.Color = Theme.Stroke; sStroke.Transparency = 0.2
    local secLayout = Instance.new("UIListLayout", section)
    secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local secPadding = Instance.new("UIPadding", section)
    secPadding.PaddingTop = UDim.new(0, 12); secPadding.PaddingBottom = UDim.new(0, 12)
    local title = Instance.new("TextLabel", section)
    title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText; title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left; title.ZIndex = 2
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end)
    return section
end

local function CreateToggle(section, text, descText, defaultState, callback)
    if type(defaultState) == "function" then callback = defaultState; defaultState = false end
    itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section)
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, descText and 52 or 34); frame.BackgroundTransparency = 1; frame.ZIndex = 2
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 0, 34); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = defaultState and Theme.Text or Theme.SubText; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left; label.ZIndex = 2
    if descText then
        local descLabel = Instance.new("TextLabel", frame)
        descLabel.Size = UDim2.new(0.7, 0, 0, 15); descLabel.Position = UDim2.new(0, 0, 0, 30); descLabel.BackgroundTransparency = 1; descLabel.Text = descText; descLabel.TextColor3 = Theme.SubText; descLabel.Font = Enum.Font.Gotham; descLabel.TextSize = 11; descLabel.TextXAlignment = Enum.TextXAlignment.Left; descLabel.ZIndex = 2
    end
    local tBtn = Instance.new("TextButton", frame)
    tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff; tBtn.Text = ""; tBtn.ZIndex = 2; Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local bStroke = Instance.new("UIStroke", tBtn); bStroke.Color = defaultState and Theme.ToggleOn or Theme.Stroke; bStroke.Transparency = 0.2; AddClickPop(tBtn)
    local circle = Instance.new("Frame", tBtn)
    circle.Size = UDim2.new(0, 16, 0, 16); circle.Position = defaultState and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8); circle.BackgroundColor3 = defaultState and Theme.Background or Color3.fromRGB(150, 150, 150); circle.ZIndex = 2; Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    local isOn = defaultState or false
    tBtn.MouseButton1Click:Connect(function()
        isOn = not isOn
        if isOn then
            TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Theme.ToggleOn}):Play()
            TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(1, -19, 0.5, -8), BackgroundColor3 = Theme.Background}):Play()
            TweenService:Create(label, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextColor3 = Theme.Text}):Play(); bStroke.Color = Theme.ToggleOn
        else
            TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Theme.ToggleOff}):Play()
            TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(0, 3, 0.5, -8), BackgroundColor3 = Color3.fromRGB(150, 150, 150)}):Play()
            TweenService:Create(label, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {TextColor3 = Theme.SubText}):Play(); bStroke.Color = Theme.Stroke
        end
        if callback then callback(isOn) end
    end)
end

local function CreateSlider(section, text, min, max, default, callback)
    itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section)
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1; frame.ZIndex = 2
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, 0, 0, 20); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.SubText; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left; label.ZIndex = 2
    local valLabel = Instance.new("TextLabel", frame)
    valLabel.Size = UDim2.new(0, 40, 0, 20); valLabel.Position = UDim2.new(1, -40, 0, 0); valLabel.BackgroundTransparency = 1; valLabel.Text = tostring(default); valLabel.TextColor3 = Theme.Accent; valLabel.Font = Enum.Font.GothamBold; valLabel.TextSize = 13; valLabel.TextXAlignment = Enum.TextXAlignment.Right; valLabel.ZIndex = 2
    local sliderBg = Instance.new("Frame", frame)
    sliderBg.Size = UDim2.new(1, 0, 0, 4); sliderBg.Position = UDim2.new(0, 0, 0, 32); sliderBg.BackgroundColor3 = Theme.ToggleOff; sliderBg.ZIndex = 2; Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    local sliderFill = Instance.new("Frame", sliderBg)
    local percentage = (default - min) / (max - min); sliderFill.Size = UDim2.new(percentage, 0, 1, 0); sliderFill.BackgroundColor3 = Theme.Accent; sliderFill.ZIndex = 2; Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    local knob = Instance.new("TextButton", sliderFill)
    knob.Size = UDim2.new(0, 14, 0, 14); knob.Position = UDim2.new(1, -7, 0.5, -7); knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255); knob.Text = ""; knob.ZIndex = 2; Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    local dragging = false
    local function setSlider(value)
        local relative = math.clamp((value - min) / (max - min), 0, 1); valLabel.Text = tostring(value); TweenService:Create(sliderFill, TweenInfo.new(0.08, Enum.EasingStyle.Quad), {Size = UDim2.new(relative, 0, 1, 0)}):Play(); if callback then callback(value) end
    end
    knob.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(1, -9, 0.5, -9)}):Play() end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 14, 0, 14), Position = UDim2.new(1, -7, 0.5, -7)}):Play() end end)
    UserInputService.InputChanged:Connect(function(input) if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local relative = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1); setSlider(math.floor(min + (max - min) * relative)) end end)
end

local function CreateButton(section, text, color, callback)
    itemOrderCounter = itemOrderCounter + 1
    local btn = Instance.new("TextButton", section)
    btn.LayoutOrder = itemOrderCounter; btn.Size = UDim2.new(0.92, 0, 0, 34); btn.BackgroundColor3 = color; btn.Name = "CustomButton"; btn.Text = text; btn.TextColor3 = Color3.fromRGB(255,255,255); btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.ZIndex = 2
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", btn).Color = Theme.Stroke; Instance.new("UIStroke", btn).Transparency = 0.5; AddClickPop(btn)
    btn.MouseButton1Click:Connect(callback); return btn
end

local function CreateTextBox(section, placeholder, callback)
    itemOrderCounter = itemOrderCounter + 1
    local box = Instance.new("TextBox", section)
    box.LayoutOrder = itemOrderCounter; box.Size = UDim2.new(0.92, 0, 0, 34); box.BackgroundColor3 = Theme.Background; box.Name = "CustomTextBox"; box.Text = ""; box.PlaceholderText = placeholder; box.TextColor3 = Theme.Text; box.Font = Enum.Font.GothamMedium; box.TextSize = 12; box.ClearTextOnFocus = false; box.ClipsDescendants = true; box.ZIndex = 2
    Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", box).Color = Theme.Stroke
    if callback then box.FocusLost:Connect(function() callback(box.Text) end) end
    return box
end

local function CreateDropdown(section, headerText, itemsList, callback)
    itemOrderCounter = itemOrderCounter + 1
    local frame = Instance.new("Frame", section)
    frame.LayoutOrder = itemOrderCounter; frame.Size = UDim2.new(0.92, 0, 0, 160); frame.BackgroundTransparency = 1; frame.ZIndex = 2
    local header = Instance.new("TextLabel", frame)
    header.Size = UDim2.new(1, 0, 0, 20); header.BackgroundTransparency = 1; header.Text = headerText .. ": " .. tostring(itemsList[1]); header.TextColor3 = Theme.SubText; header.Font = Enum.Font.GothamMedium; header.TextSize = 12; header.TextXAlignment = Enum.TextXAlignment.Left; header.ZIndex = 2
    local scroll = Instance.new("ScrollingFrame", frame)
    scroll.Name = "DropdownContainer"; scroll.Size = UDim2.new(1, 0, 0, 130); scroll.Position = UDim2.new(0, 0, 0, 25); scroll.BackgroundColor3 = Theme.Background; scroll.ScrollBarThickness = 4; scroll.ZIndex = 2
    Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 6)
    local listLayout = Instance.new("UIListLayout", scroll)
    listLayout.Padding = UDim.new(0, 4); listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    for _, itemName in ipairs(itemsList) do
        local btn = Instance.new("TextButton", scroll)
        btn.Name = "DropButton"; btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. tostring(itemName); btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; btn.ZIndex = 2
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
        btn.MouseButton1Click:Connect(function() header.Text = headerText .. ": " .. tostring(itemName); if callback then callback(itemName) end end)
    end
    task.defer(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
    return frame
end

--// =======================
--// UI POPULATION 
--// =======================

local function Ryuhub() end

-- TAB 1: AUTO PLAY
local TabAutoPlay = CreateMainTab("Auto Play")
local SubAutoMain = CreateSubTab(TabAutoPlay, "Main")
local SecAutoPlay = CreateSection(SubAutoMain, "Impel Down Engine")
CreateToggle(SecAutoPlay, "Enable Auto Impel Down", "Automatically clear all stages", RyuSavedConfig.AutoImpelDown, function(state) RyuSavedConfig.AutoImpelDown = state end)
CreateToggle(SecAutoPlay, "Auto Next Stage", "Proceeds to the next floor automatically", false, Ryuhub)
CreateToggle(SecAutoPlay, "Auto Boss Farm", "Targets boss NPCs prioritized", false, Ryuhub)
CreateSlider(SecAutoPlay, "Farm Distance", 5, 50, RyuSavedConfig.ImpelFarmDistance, function(val) RyuSavedConfig.ImpelFarmDistance = val end)

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
    if state then _G.AntiAfkConnection = LocalPlayer.Idled:Connect(function() game:GetService("VirtualUser"):CaptureController(); game:GetService("VirtualUser"):ClickButton2(Vector2.new()) end)
    else if _G.AntiAfkConnection then _G.AntiAfkConnection:Disconnect() end end
end)

local SubTheme = CreateSubTab(TabSettings, "Theme & UI")
local SecWindow = CreateSection(SubTheme, "Window Personalization")
CreateToggle(SecWindow, "Glass Mode", "Transparent frosted glass UI", RyuSavedConfig.GlassMode, function(state)
    RyuSavedConfig.GlassMode = state
    local mainTrans = state and 0.4 or 0; local secTrans = state and 0.5 or 0
    TweenService:Create(MainFrame, TweenInfo.new(0.3), {BackgroundTransparency = mainTrans, BackgroundColor3 = state and Color3.fromRGB(5,5,5) or Theme.Background}):Play()
    for _, obj in pairs(MainFrame:GetDescendants()) do
        if obj:IsA("Frame") and (obj.Name == "SectionContainer" or obj.Name == "DropdownContainer") then TweenService:Create(obj, TweenInfo.new(0.3), {BackgroundTransparency = secTrans}):Play()
        elseif obj:IsA("TextBox") and obj.Name == "CustomTextBox" then TweenService:Create(obj, TweenInfo.new(0.3), {BackgroundTransparency = secTrans}):Play() end
    end
end)

CreateToggle(SecWindow, "World UI Blur", "Blurs the game when UI is open", RyuSavedConfig.WorldBlur, function(state)
    RyuSavedConfig.WorldBlur = state; _G.BlurEnabled = state
    if isUIOpen and state then TweenService:Create(UIBlur, TweenInfo.new(0.3), {Size = 15}):Play() elseif not state then TweenService:Create(UIBlur, TweenInfo.new(0.3), {Size = 0}):Play() end
end)

local PremiumColors = {["White"] = Color3.fromRGB(255, 255, 255), ["Crimson Red"] = Color3.fromRGB(220, 20, 60), ["Neon Blue"] = Color3.fromRGB(0, 150, 255), ["Emerald Green"] = Color3.fromRGB(46, 204, 113), ["Royal Purple"] = Color3.fromRGB(155, 89, 182), ["Hot Pink"] = Color3.fromRGB(255, 105, 180), ["Gold"] = Color3.fromRGB(241, 196, 15)}
CreateDropdown(SecWindow, "Accent Color", {"White", "Crimson Red", "Neon Blue", "Emerald Green", "Royal Purple", "Hot Pink", "Gold"}, function(colorName)
    local newColor = PremiumColors[colorName]
    if newColor then
        RyuSavedConfig.AccentColor = {math.floor(newColor.R*255), math.floor(newColor.G*255), math.floor(newColor.B*255)}
        Theme.Accent = newColor
        for _, obj in pairs(RyuHub:GetDescendants()) do
            if obj:IsA("UIStroke") and obj.Color ~= Theme.Stroke and obj.Color ~= Theme.Warning then obj.Color = newColor end
            if obj:IsA("Frame") and obj.BackgroundColor3 ~= Theme.Background and obj.BackgroundColor3 ~= Theme.SectionBG and obj.BackgroundColor3 ~= Theme.ToggleOff and obj.BackgroundColor3 ~= Theme.Sidebar and obj.BackgroundColor3 ~= Color3.fromRGB(150, 150, 150) then obj.BackgroundColor3 = newColor end
            if obj:IsA("TextLabel") and obj.Text == "Target: None" then obj.TextColor3 = newColor end
        end
    end
end)

local BgColors = {["Default Dark"] = Color3.fromRGB(12, 12, 14), ["Pitch Black"] = Color3.fromRGB(0, 0, 0), ["Midnight Blue"] = Color3.fromRGB(5, 10, 20), ["Deep Blood"] = Color3.fromRGB(20, 5, 5)}
CreateDropdown(SecWindow, "Background Tint", {"Default Dark", "Pitch Black", "Midnight Blue", "Deep Blood"}, function(colorName)
    local newBg = BgColors[colorName]
    if newBg then
        RyuSavedConfig.BgColor = {math.floor(newBg.R*255), math.floor(newBg.G*255), math.floor(newBg.B*255)}; Theme.Background = newBg
        if not RyuSavedConfig.GlassMode then TweenService:Create(MainFrame, TweenInfo.new(0.3), {BackgroundColor3 = newBg}):Play() end
    end
end)

CreateDropdown(SecWindow, "UI Font Style", {"Gotham", "Code", "Arcade", "SciFi", "Cartoon", "Fantasy", "Oswald"}, function(fontName)
    RyuSavedConfig.Font = fontName; local targetFont = Enum.Font.Gotham
    if fontName == "Code" then targetFont = Enum.Font.Code elseif fontName == "Arcade" then targetFont = Enum.Font.Arcade elseif fontName == "SciFi" then targetFont = Enum.Font.Michroma elseif fontName == "Cartoon" then targetFont = Enum.Font.Cartoon elseif fontName == "Fantasy" then targetFont = Enum.Font.Fantasy elseif fontName == "Oswald" then targetFont = Enum.Font.Oswald end
    for _, obj in pairs(MainFrame:GetDescendants()) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
            if obj.Font == Enum.Font.GothamBold then obj.Font = targetFont elseif obj.Font == Enum.Font.GothamMedium then obj.Font = targetFont else obj.Font = targetFont end
        end
    end
end)

CreateToggle(SecWindow, "Hide Window Borders", "Removes the outer window line", RyuSavedConfig.HideBorders, function(state) RyuSavedConfig.HideBorders = state; mainStroke.Enabled = not state end)
CreateSlider(SecWindow, "Window Roundness", 0, 24, RyuSavedConfig.Roundness, function(val) RyuSavedConfig.Roundness = val; MainCorner.CornerRadius = UDim.new(0, val) end)

CreateTextBox(SecWindow, "Custom Background URL (Asset ID)...", function(txt)
    if txt and txt ~= "" then
        local num = txt:match("%d+")
        if num then local url = "rbxthumb://type=Asset&id="..num.."&w=768&h=432"; MainBgImage.Image = url; RyuSavedConfig.BgImage = url; MainBgImage.ImageTransparency = RyuSavedConfig.BgOpacity end
    else MainBgImage.Image = ""; RyuSavedConfig.BgImage = ""; MainBgImage.ImageTransparency = 1 end
end)

CreateSlider(SecWindow, "Background Image Opacity", 0, 100, (1 - RyuSavedConfig.BgOpacity) * 100, function(val)
    local op = 1 - (val / 100); RyuSavedConfig.BgOpacity = op
    if MainBgImage.Image ~= "" then MainBgImage.ImageTransparency = op end
end)

local SecToggleUI = CreateSection(SubTheme, "Toggle Button Personalization")
CreateTextBox(SecToggleUI, "Custom Icon ID (e.g. 6050149849)", function(txt)
    if txt and txt ~= "" then local num = txt:match("%d+"); if num then local url = "rbxthumb://type=Asset&id="..num.."&w=150&h=150"; ToggleBtn.Image = url; RyuSavedConfig.ToggleIcon = url end end
end)

CreateSlider(SecToggleUI, "Toggle Button Size", 30, 80, RyuSavedConfig.ToggleSize, function(val)
    RyuSavedConfig.ToggleSize = val; TweenService:Create(ToggleBtn, TweenInfo.new(0.2), {Size = UDim2.new(0, val, 0, val)}):Play()
end)

CreateSlider(SecToggleUI, "Toggle Glow Intensity", 0, 100, (1 - RyuSavedConfig.ToggleGlow) * 100, function(val)
    local glow = 1 - (val / 100); RyuSavedConfig.ToggleGlow = glow; btnStroke.Transparency = glow
end)

local rainbowToggle = RyuSavedConfig.RainbowMode
CreateToggle(SecToggleUI, "RGB Rainbow Ring", "Animates the toggle button border", RyuSavedConfig.RainbowMode, function(state)
    rainbowToggle = state; RyuSavedConfig.RainbowMode = state; if not state then btnStroke.Color = Theme.Accent end
end)
task.spawn(function() while true do if rainbowToggle then btnStroke.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1) end; task.wait(0.1) end end)

CreateToggle(SecToggleUI, "Floating Icon Mode", "Removes the toggle button background", RyuSavedConfig.FloatingIcon, function(state)
    RyuSavedConfig.FloatingIcon = state; if state then TweenService:Create(ToggleBtn, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play() else TweenService:Create(ToggleBtn, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play() end
end)

local SecSave = CreateSection(SubTheme, "Config & System")
CreateButton(SecSave, "Save Settings To File", Color3.fromRGB(50, 150, 50), function() SaveConfig() end)
CreateButton(SecSave, "Reset UI Settings", Theme.Warning, function()
    if delfile and isfile and isfile(configFileName) then pcall(function() delfile(configFileName) end) end
    RyuSavedConfig = { GlassMode = false, WorldBlur = false, AccentColor = {255, 255, 255}, BgColor = {12, 12, 14}, Font = "Gotham", HideBorders = false, Roundness = 12, BgImage = "", BgOpacity = 0.6, ToggleIcon = "rbxthumb://type=Asset&id=6050149849&w=150&h=150", ToggleSize = 50, ToggleGlow = 0.5, RainbowMode = false, FloatingIcon = false, AutoImpelDown = false, ImpelFarmDistance = 15 }
    Theme.Background = Color3.fromRGB(12, 12, 14); Theme.Accent = Color3.fromRGB(255, 255, 255)
    TweenService:Create(MainFrame, TweenInfo.new(0.3), {BackgroundColor3 = Theme.Background, BackgroundTransparency = 0}):Play()
    MainBgImage.Image = ""; mainStroke.Enabled = true; UIBlur.Size = 0; _G.BlurEnabled = false; btnStroke.Color = Theme.Accent; rainbowToggle = false; MainCorner.CornerRadius = UDim.new(0, 12); ToggleBtn.Image = RyuSavedConfig.ToggleIcon; ToggleBtn.Size = UDim2.new(0, 50, 0, 50); btnStroke.Transparency = 0.5; ToggleBtn.BackgroundTransparency = 0
    for _, obj in pairs(MainFrame:GetDescendants()) do
        if obj:IsA("UIStroke") and obj.Color ~= Theme.Stroke and obj.Color ~= Theme.Warning then obj.Color = Theme.Accent end
        if obj:IsA("Frame") and obj.Name == "SectionContainer" then obj.BackgroundTransparency = 0 end
        if obj:IsA("Frame") and obj.Name == "DropdownContainer" then obj.BackgroundTransparency = 0 end
        if obj:IsA("TextBox") and obj.Name == "CustomTextBox" then obj.BackgroundTransparency = 0 end
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then if obj.Font ~= Enum.Font.GothamBold and obj.Font ~= Enum.Font.GothamMedium then obj.Font = Enum.Font.Gotham end end
    end
end)

task.spawn(function() Tabs[1].Toggle(); Tabs[1].SubTabs[1].Open() end)

local function ApplyLoadedSettings()
    if RyuSavedConfig.GlassMode then
        MainFrame.BackgroundTransparency = 0.4; MainFrame.BackgroundColor3 = Color3.fromRGB(5,5,5)
        for _, obj in pairs(MainFrame:GetDescendants()) do
            if obj:IsA("Frame") and (obj.Name == "SectionContainer" or obj.Name == "DropdownContainer") then obj.BackgroundTransparency = 0.5 elseif obj:IsA("TextBox") and obj.Name == "CustomTextBox" then obj.BackgroundTransparency = 0.5 end
        end
    end
    _G.BlurEnabled = RyuSavedConfig.WorldBlur
    local targetFont = Enum.Font.Gotham
    if RyuSavedConfig.Font == "Code" then targetFont = Enum.Font.Code elseif RyuSavedConfig.Font == "Arcade" then targetFont = Enum.Font.Arcade elseif RyuSavedConfig.Font == "SciFi" then targetFont = Enum.Font.Michroma elseif RyuSavedConfig.Font == "Cartoon" then targetFont = Enum.Font.Cartoon elseif RyuSavedConfig.Font == "Fantasy" then targetFont = Enum.Font.Fantasy elseif RyuSavedConfig.Font == "Oswald" then targetFont = Enum.Font.Oswald end
    for _, obj in pairs(MainFrame:GetDescendants()) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then if obj.Font == Enum.Font.GothamBold then obj.Font = targetFont elseif obj.Font == Enum.Font.GothamMedium then obj.Font = targetFont else obj.Font = targetFont end end
    end
    mainStroke.Enabled = not RyuSavedConfig.HideBorders; MainCorner.CornerRadius = UDim.new(0, RyuSavedConfig.Roundness)
    if RyuSavedConfig.BgImage ~= "" then MainBgImage.Image = RyuSavedConfig.BgImage; MainBgImage.ImageTransparency = RyuSavedConfig.BgOpacity end
    ToggleBtn.Image = RyuSavedConfig.ToggleIcon; ToggleBtn.Size = UDim2.new(0, RyuSavedConfig.ToggleSize, 0, RyuSavedConfig.ToggleSize); btnStroke.Transparency = RyuSavedConfig.ToggleGlow
    if RyuSavedConfig.FloatingIcon then ToggleBtn.BackgroundTransparency = 1 end
end
ApplyLoadedSettings()

--// ============================================================================
--// IMPEL DOWN AUTO FARM ENGINE (V2: VERA LOCK, CHESTS, WAYPOINTS, GUARDS, FAILSAFE)
--// ============================================================================

local currentComboIndex = 1
local lastSwing = 0

local function EquipTargetWeapon()
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    
    local targetWep = nil
    for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
        if item:IsA("Tool") and (item:GetAttribute("MeleeTool") or item.Name:lower():find("combat") or item.Name:lower():find("sword")) then
            targetWep = item; break
        end
    end
    if not targetWep then
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do if item:IsA("Tool") then targetWep = item break end end
    end
    
    if targetWep then
        pcall(function() ReplicatedStorage.Events.Tools:InvokeServer("equip", targetWep.Name) end)
        task.wait(0.1)
        hum:EquipTool(targetWep)
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
        if now - lastSwing >= 0.5 then
            lastSwing = now
            task.spawn(function()
                local hitParts = {}
                if type(targets) == "table" then
                    for _, npc in ipairs(targets) do
                        local mRoot = npc:FindFirstChild("HumanoidRootPart")
                        local mHum = npc:FindFirstChildOfClass("Humanoid")
                        if mRoot and mHum and mHum.Health > 0 then table.insert(hitParts, mRoot) end
                    end
                end
                
                local animName = "Punch" .. currentComboIndex
                if currentComboIndex == 1 then animName = "Dash" end
                if currentComboIndex == 4 then animName = "GroundPunch4" end
                
                local animObj = ReplicatedStorage:FindFirstChild("CombatAnimations") and ReplicatedStorage.CombatAnimations:FindFirstChild("Melee") and ReplicatedStorage.CombatAnimations.Melee:FindFirstChild(animName)
                if animObj then
                    local argsAnim = {"swingsfx", "Melee", currentComboIndex, "Ground", currentComboIndex == 1, animObj, 2, 1.5}
                    if ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("CombatRegister") then
                        pcall(function() ReplicatedStorage.Events.CombatRegister:InvokeServer(argsAnim) end)
                    end
                end
                
                if #hitParts > 0 then
                    local argsDamage = {"damage", hitParts, "Melee", {currentComboIndex, "Ground", "Melee"}, true, root.CFrame, ["aircombo"] = "Ground"}
                    if ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("CombatRegister") then
                        pcall(function() ReplicatedStorage.Events.CombatRegister:InvokeServer(argsDamage) end)
                    end
                end
                
                currentComboIndex = currentComboIndex + 1
                if currentComboIndex > 4 then currentComboIndex = 1 end
            end)
        end
    end)
end

-- TWEEN FUNCTION MIT ANTI-RUBBERBAND CHECK
local function SafeTween(targetCFrame, customSpeed, isKeyMove)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local startPos = root.Position
    local targetPos = targetCFrame.Position
    local dist = (targetPos - startPos).Magnitude
    
    local speed = customSpeed or 50 
    local timeToTake = dist / speed
    
    if timeToTake < 0.1 then 
        root.CFrame = targetCFrame
        return 
    end

    local startTime = tick()
    
    local bp = root:FindFirstChild("RyuHover")
    if not bp then
        bp = Instance.new("BodyPosition")
        bp.Name = "RyuHover"
        bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bp.D = 500
        bp.P = 50000
        bp.Parent = root
    end

    local lastRealPos = root.Position

    while tick() - startTime < timeToTake do
        if not RyuSavedConfig.AutoImpelDown then break end
        
        -- TP CHECK BYPASS: Server Rubberband Erkennung
        if isKeyMove and (root.Position - lastRealPos).Magnitude > 20 then
            bp.Position = root.Position
            task.wait(1) 
            
            startPos = root.Position
            dist = (targetPos - startPos).Magnitude
            timeToTake = dist / speed
            startTime = tick()
        end
        lastRealPos = root.Position

        local alpha = (tick() - startTime) / timeToTake
        local intermediatePos = startPos:Lerp(targetPos, alpha)
        
        bp.Position = intermediatePos
        root.CFrame = CFrame.new(intermediatePos) * targetCFrame.Rotation
        RunService.Heartbeat:Wait()
    end
    
    bp.Position = targetPos
    root.CFrame = targetCFrame
end

-- BODYGYRO FÜR 90 GRAD LOCK UND ANTI-ZAPPELN
local function ToggleHover(state, targetCFrame)
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
        local bg = root:FindFirstChild("RyuGyro")
        if not bg then
            bg = Instance.new("BodyGyro")
            bg.Name = "RyuGyro"
            bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bg.D = 100
            bg.P = 10000
            bg.Parent = root
        end
        bp.Position = targetCFrame and targetCFrame.Position or root.Position
        bg.CFrame = targetCFrame or root.CFrame
    else
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp:Destroy() end
        local bg = root:FindFirstChild("RyuGyro")
        if bg then bg:Destroy() end
    end
end

-- IMPEL DOWN ENGINE STATE MACHINE
_G.ImpelState = 1
_G.ChestsLooted = 0
_G.InFailsafe = false

task.spawn(function()
    while true do
        task.wait(0.1)
        if not RyuSavedConfig.AutoImpelDown then 
            _G.ImpelState = 1
            _G.ChestsLooted = 0
            _G.WP1_Reached = false
            _G.WP2_Reached = false
            _G.KeyWaitTriggered = false
            ToggleHover(false)
            continue 
        end

        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not root or not hum or hum.Health <= 0 then continue end

        -- SIDE FEATURE: AUTO STATS
        pcall(function()
            if ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("stats") then
                ReplicatedStorage.Events.stats:FireServer("Strength", nil, 700)
                ReplicatedStorage.Events.stats:FireServer("Defense", nil, 800)
            end
        end)

        -- FAILSAFE (HP < 80%)
        if hum.Health / hum.MaxHealth < 0.8 then
            if not _G.InFailsafe then
                _G.InFailsafe = true
                _G.SafePos = root.Position + Vector3.new(0, 14, 0)
            end
        end

        if _G.InFailsafe then
            ToggleHover(true, CFrame.new(_G.SafePos, _G.SafePos + Vector3.new(0, -1, 0)))
            pcall(function() ReplicatedStorage.Events.Block:InvokeServer(true, "Melee", true) end)
            
            -- Warten bis fast voll geheilt
            if hum.Health / hum.MaxHealth >= 0.95 then 
                _G.InFailsafe = false
                pcall(function() ReplicatedStorage.Events.Block:InvokeServer(false, "Melee", true) end)
            end
            continue
        end

        -- STATE 1: Auto Nightmare Difficulty
        if _G.ImpelState == 1 then
            local diffChooser = LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("DiffChooser")
            if diffChooser and diffChooser.Enabled then
                pcall(function() diffChooser.Replication.RemoteEvent:FireServer("Nightmare", "check!") end)
                task.wait(0.5)
                continue 
            else
                _G.ImpelState = 2
            end
        end

        -- STATE 2: Töte Vera (90 Grad Lock)
        if _G.ImpelState == 2 then
            local vera = Workspace:FindFirstChild("NPCs") and Workspace.NPCs:FindFirstChild("Vera")
            if vera then
                local vHum = vera:FindFirstChildOfClass("Humanoid")
                local vRoot = vera:FindFirstChild("HumanoidRootPart")
                if vHum and vRoot and vHum.Health > 0 then
                    local attackPos = vRoot.Position + Vector3.new(0, 7.5, 0)
                    local targetCFrame = CFrame.new(attackPos, attackPos + Vector3.new(0, -1, 0))
                    
                    if (root.Position - attackPos).Magnitude > 15 then
                        SafeTween(targetCFrame, 50, false)
                    else
                        ToggleHover(true, targetCFrame)
                    end

                    EquipTargetWeapon()
                    PerformMeleeAttack({vera})
                    task.wait(0.05)
                    continue 
                end
            else
                _G.ImpelState = 3
            end
        end

        -- STATE 3: Schlüssel sammeln
        if _G.ImpelState == 3 then
            local keyPart = nil
            pcall(function()
                local effects = Workspace:FindFirstChild("Effects")
                if effects then
                    local kModel = effects:FindFirstChild("Key")
                    if kModel then
                        if kModel:IsA("BasePart") then keyPart = kModel 
                        elseif kModel:FindFirstChild("Key") then keyPart = kModel.Key end
                    end
                end
                if not keyPart then
                    local islands = Workspace:FindFirstChild("Islands")
                    if islands then
                        for _, isl in pairs(islands:GetChildren()) do
                            if isl.Name:find("Impel Base") then
                                local spawns = isl:FindFirstChild("KeySpawns")
                                if spawns then
                                    for _, k in pairs(spawns:GetChildren()) do
                                        if k.Name == "Key" and k:IsA("BasePart") and k.Transparency < 1 then
                                            keyPart = k
                                            break
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end)

            if keyPart then
                if not _G.KeyWaitTriggered then
                    _G.KeyWaitTriggered = true
                    task.wait(4)
                end

                pcall(function()
                    if ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("sprint") then
                        ReplicatedStorage.Events.sprint:FireServer("rbxassetid://15382065457")
                    end
                end)

                local dist = (root.Position - keyPart.Position).Magnitude
                if dist > 4 then
                    ToggleHover(false)
                    hum.WalkSpeed = 45
                    hum:MoveTo(keyPart.Position)
                    SafeTween(keyPart.CFrame, 45, true) 
                else
                    root.CFrame = keyPart.CFrame
                    root.Velocity = Vector3.new(0,0,0)
                    
                    local prompt = keyPart:FindFirstChildOfClass("ProximityPrompt")
                    if prompt then
                        if fireproximityprompt then fireproximityprompt(prompt, 1)
                        else
                            prompt:InputHoldBegin()
                            task.wait(prompt.HoldDuration + 0.1)
                            prompt:InputHoldEnd()
                        end
                    else
                        pcall(function()
                            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                            task.wait(0.1)
                            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
                        end)
                    end
                    task.wait(0.5)
                end
                continue
            else
                if _G.KeyWaitTriggered then 
                    _G.ImpelState = 4 
                    task.wait(2)
                end
            end
        end

        -- STATE 4: 3 Blaue Chests
        if _G.ImpelState == 4 then
            if _G.ChestsLooted >= 3 then
                _G.ImpelState = 5
                continue
            end

            local targetChest, promptFound = nil, nil
            pcall(function()
                for _, v in pairs(Workspace:GetDescendants()) do
                    if v:IsA("Model") and v:FindFirstChild("Lock") and v:FindFirstChild("Top") then
                        local p = v:FindFirstChildOfClass("ProximityPrompt", true)
                        if p and p.Enabled then
                            targetChest = v
                            promptFound = p
                            break
                        end
                    end
                end
            end)

            if targetChest and promptFound then
                local cPos = targetChest:GetPivot().Position
                local dist = (root.Position - cPos).Magnitude
                if dist > 5 then
                    ToggleHover(false)
                    hum.WalkSpeed = 45
                    hum:MoveTo(cPos)
                    SafeTween(CFrame.new(cPos), 45, true)
                else
                    root.CFrame = CFrame.new(cPos)
                    root.Velocity = Vector3.new(0,0,0)
                    if fireproximityprompt then fireproximityprompt(promptFound, 1)
                    else
                        promptFound:InputHoldBegin()
                        task.wait(promptFound.HoldDuration + 0.1)
                        promptFound:InputHoldEnd()
                    end
                    task.wait(1)
                    _G.ChestsLooted = _G.ChestsLooted + 1
                end
                continue
            else
                _G.ImpelState = 5
            end
        end

        -- STATE 5: Waypoints verfolgen
        if _G.ImpelState == 5 then
            local wp1 = Vector3.new(2941.29, 2075.25, -13574.59)
            local wp2 = Vector3.new(2952.60, 2075.15, -13848.57)

            if not _G.WP1_Reached then
                local dist = (root.Position - wp1).Magnitude
                if dist > 5 then
                    ToggleHover(false)
                    hum.WalkSpeed = 45
                    SafeTween(CFrame.new(wp1), 45, true)
                else
                    root.CFrame = CFrame.new(wp1)
                    root.Velocity = Vector3.new(0,0,0)
                    _G.WP1_Reached = true
                    task.wait(1)
                end
                continue
            end

            if not _G.WP2_Reached then
                local dist = (root.Position - wp2).Magnitude
                if dist > 5 then
                    ToggleHover(false)
                    hum.WalkSpeed = 45
                    SafeTween(CFrame.new(wp2), 45, true)
                else
                    root.CFrame = CFrame.new(wp2)
                    root.Velocity = Vector3.new(0,0,0)
                    _G.WP2_Reached = true
                    _G.ImpelState = 6
                end
                continue
            end
        end

        -- STATE 6: Impel Guards Group Farm
        if _G.ImpelState == 6 then
            local guards = {}
            if Workspace:FindFirstChild("NPCs") then
                for _, v in pairs(Workspace.NPCs:GetChildren()) do
                    if v.Name:find("Impel Guard") then
                        local gHum = v:FindFirstChildOfClass("Humanoid")
                        local gRoot = v:FindFirstChild("HumanoidRootPart")
                        if gHum and gRoot and gHum.Health > 0 then
                            table.insert(guards, v)
                        end
                    end
                end
            end

            if #guards > 0 then
                local sumPos = Vector3.new(0, 0, 0)
                for _, g in ipairs(guards) do
                    sumPos = sumPos + g.HumanoidRootPart.Position
                end
                local centerPos = sumPos / #guards
                
                local attackPos = centerPos + Vector3.new(0, 7.5, 0)
                local targetCFrame = CFrame.new(attackPos, attackPos + Vector3.new(0, -1, 0))
                
                if (root.Position - attackPos).Magnitude > 15 then
                    SafeTween(targetCFrame, 50, false)
                else
                    ToggleHover(true, targetCFrame)
                end
                
                EquipTargetWeapon()
                PerformMeleeAttack(guards)
                task.wait(0.05)
                continue
            end
        end

    end
end)
