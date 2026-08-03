--// ============================================================================
--// RYU HUB - PURE MODERN GLASS UI (100% SAFE - NO HOOKS, NO REMOTES)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

--// CLEANUP (Nur altes GUI entfernen, keine Anti-Cheat Manipulationen!)
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function()
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end
end)

for _, v in pairs(guiParent:GetChildren()) do
    if v.Name == "RyuHubModernGlass" then v:Destroy() end
end

--// ULTRA MODERN GLASS THEME
local Theme = {
    Background    = Color3.fromRGB(5, 7, 12),          
    BgTransp      = 0.35,                              
    Sidebar       = Color3.fromRGB(10, 14, 22),
    SectionBG     = Color3.fromRGB(15, 20, 30),
    SectionTransp = 0.4,
    
    CardBorder    = Color3.fromRGB(40, 60, 80),        
    TextPrimary   = Color3.fromRGB(255, 255, 255),     
    TextSecondary = Color3.fromRGB(150, 170, 190),     
    
    Accent        = Color3.fromRGB(0, 229, 255),       
    AccentGlow    = Color3.fromRGB(120, 245, 255),
    ToggleOff     = Color3.fromRGB(25, 30, 45),
    CloseRed      = Color3.fromRGB(255, 59, 48),       
    
    WaveColors = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 229, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 229, 255))
    })
}

--// SMART MOBILE SCALING
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local screenY = Camera.ViewportSize.Y
local baseScale = isMobile and math.clamp(screenY / 750, 0.45, 0.75) or 1.0

--// MAIN SCREEN GUI
local RyuHubGui = Instance.new("ScreenGui")
RyuHubGui.Name = "RyuHubModernGlass"
RyuHubGui.ResetOnSpawn = false
RyuHubGui.IgnoreGuiInset = true
RyuHubGui.Parent = guiParent

local GlobalScale = Instance.new("UIScale", RyuHubGui)
GlobalScale.Scale = baseScale

--// MAIN CONTAINER (GLASS)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 660, 0, 420)
MainFrame.Position = UDim2.new(0.5, -330, 0.5, -210)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BackgroundTransparency = Theme.BgTransp
MainFrame.Active = true
MainFrame.Parent = RyuHubGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)

-- GLOW STROKE
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 1.5
MainStroke.Color = Theme.Accent
MainStroke.Transparency = 0.3

--// TOPBAR
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Name = "Topbar"
Topbar.Size = UDim2.new(1, 0, 0, 50)
Topbar.BackgroundTransparency = 1

--// PERFECT WAVE ANIMATION TITLE
local TitleLabel = Instance.new("TextLabel", Topbar)
TitleLabel.Size = UDim2.new(0, 200, 1, 0)
TitleLabel.Position = UDim2.new(0, 20, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "RYU HUB"
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextSize = 22
TitleLabel.TextColor3 = Color3.new(1,1,1)
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

local TitleGradient = Instance.new("UIGradient", TitleLabel)
TitleGradient.Color = Theme.WaveColors
TitleGradient.Rotation = 25
TitleGradient.Offset = Vector2.new(-1, 0)

local waveTween = TweenService:Create(TitleGradient, TweenInfo.new(2.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, false), {Offset = Vector2.new(1, 0)})
waveTween:Play()

--// MODERN RED CLOSE BUTTON
local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28)
CloseBtn.Position = UDim2.new(1, -40, 0.5, -14)
CloseBtn.BackgroundColor3 = Theme.CloseRed
CloseBtn.BackgroundTransparency = 0.15
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

CloseBtn.Activated:Connect(function()
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
    task.wait(0.3)
    MainFrame.Visible = false
end)

-- FLOATING TOGGLE BUTTON
local ToggleBtn = Instance.new("TextButton", RyuHubGui)
ToggleBtn.Name = "RyuToggleBtn"
ToggleBtn.Size = UDim2.new(0, 48, 0, 48)
ToggleBtn.Position = UDim2.new(0, 20, 0.15, 0)
ToggleBtn.BackgroundColor3 = Theme.Background
ToggleBtn.BackgroundTransparency = 0.2
ToggleBtn.Text = "🌀"
ToggleBtn.TextSize = 22
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local ToggleStroke = Instance.new("UIStroke", ToggleBtn)
ToggleStroke.Color = Theme.Accent
ToggleStroke.Thickness = 1.5

ToggleBtn.Activated:Connect(function()
    if MainFrame.Visible then
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
        task.wait(0.3)
        MainFrame.Visible = false
    else
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 660, 0, 420), BackgroundTransparency = Theme.BgTransp}):Play()
    end
end)

-- DRAGGING
local function MakeDraggable(guiObject, handleObject)
    local dragging, dragStart, startPos
    handleObject.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true; dragStart = input.Position; startPos = guiObject.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            guiObject.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end
MakeDraggable(MainFrame, Topbar)
MakeDraggable(ToggleBtn, ToggleBtn)

--// 2-COLUMN LAYOUT
local BodyContainer = Instance.new("Frame", MainFrame)
BodyContainer.Size = UDim2.new(1, -20, 1, -60)
BodyContainer.Position = UDim2.new(0, 10, 0, 50)
BodyContainer.BackgroundTransparency = 1

-- SPALTE 1: ACCORDION SIDEBAR
local Sidebar = Instance.new("ScrollingFrame", BodyContainer)
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 160, 1, 0)
Sidebar.BackgroundTransparency = 1
Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 8)
SideLayout.SortOrder = Enum.SortOrder.LayoutOrder

SideLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, SideLayout.AbsoluteContentSize.Y + 10)
end)

-- SPALTE 2: CONTENT AREA
local ContentArea = Instance.new("Frame", BodyContainer)
ContentArea.Name = "ContentArea"
ContentArea.Size = UDim2.new(1, -180, 1, 0)
ContentArea.Position = UDim2.new(0, 180, 0, 0)
ContentArea.BackgroundTransparency = 1

--// PERFECT ACCORDION TAB SYSTEM
local Categories = {}

local function CreateMainTab(tabName, icon)
    icon = icon or "🔹"
    local tabObj = { Name = tabName, IsOpen = false, SubTabs = {} }
    
    local tabWrapper = Instance.new("Frame", Sidebar)
    tabWrapper.Size = UDim2.new(1, 0, 0, 40)
    tabWrapper.BackgroundTransparency = 1
    tabWrapper.ClipsDescendants = true
    
    local tabBtn = Instance.new("TextButton", tabWrapper)
    tabBtn.Size = UDim2.new(1, 0, 0, 40)
    tabBtn.BackgroundColor3 = Theme.Sidebar
    tabBtn.BackgroundTransparency = Theme.SectionTransp
    tabBtn.Text = "   " .. icon .. "  " .. tabName
    tabBtn.TextColor3 = Theme.TextSecondary
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 13
    tabBtn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)
    
    local btnStroke = Instance.new("UIStroke", tabBtn)
    btnStroke.Color = Theme.CardBorder
    tabObj.Button = tabBtn
    tabObj.Wrapper = tabWrapper
    
    local subContainer = Instance.new("Frame", tabWrapper)
    subContainer.Size = UDim2.new(1, 0, 1, -40)
    subContainer.Position = UDim2.new(0, 0, 0, 40)
    subContainer.BackgroundTransparency = 1
    
    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 4)
    subLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
    Instance.new("UIPadding", subContainer).PaddingTop = UDim.new(0, 4)
    
    tabBtn.Activated:Connect(function()
        local wasOpen = tabObj.IsOpen
        
        for _, cat in pairs(Categories) do
            cat.IsOpen = false
            TweenService:Create(cat.Button, TweenInfo.new(0.3), {TextColor3 = Theme.TextSecondary}):Play()
            TweenService:Create(cat.Button:FindFirstChildOfClass("UIStroke"), TweenInfo.new(0.3), {Color = Theme.CardBorder}):Play()
            TweenService:Create(cat.Wrapper, TweenInfo.new(0.35, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 40)}):Play()
        end
        
        if not wasOpen then
            tabObj.IsOpen = true
            TweenService:Create(tabBtn, TweenInfo.new(0.3), {TextColor3 = Theme.Accent}):Play()
            TweenService:Create(btnStroke, TweenInfo.new(0.3), {Color = Theme.Accent}):Play()
            
            local targetHeight = 40 + subLayout.AbsoluteContentSize.Y + 4
            TweenService:Create(tabWrapper, TweenInfo.new(0.35, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, targetHeight)}):Play()
            
            if #tabObj.SubTabs > 0 then
                tabObj.SubTabs[1].Activate()
            end
        end
    end)
    
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if tabObj.IsOpen then
            local targetHeight = 40 + subLayout.AbsoluteContentSize.Y + 4
            tabWrapper.Size = UDim2.new(1, 0, 0, targetHeight)
        end
    end)
    
    table.insert(Categories, tabObj)
    
    function tabObj:CreateSubTab(subName)
        local subObj = { Name = subName, Page = nil }
        
        local subBtn = Instance.new("TextButton", subContainer)
        subBtn.Size = UDim2.new(0.9, 0, 0, 30)
        subBtn.BackgroundTransparency = 1
        subBtn.Text = "•  " .. subName
        subBtn.TextColor3 = Theme.TextSecondary
        subBtn.Font = Enum.Font.GothamMedium
        subBtn.TextSize = 12
        subBtn.TextXAlignment = Enum.TextXAlignment.Left
        
        local page = Instance.new("ScrollingFrame", ContentArea)
        page.Name = subName .. "_Page"
        page.Size = UDim2.new(1, 0, 1, 0)
        page.BackgroundTransparency = 1
        page.ScrollBarThickness = 2
        page.ScrollBarImageColor3 = Theme.Accent
        page.Visible = false
        
        local pageLayout = Instance.new("UIListLayout", page)
        pageLayout.Padding = UDim.new(0, 10)
        pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        
        pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20)
        end)
        
        local function ActivateSub()
            for _, cat in pairs(Categories) do
                for _, btn in pairs(cat.Wrapper:GetDescendants()) do
                    if btn:IsA("TextButton") and btn.Text:match("•") then 
                        TweenService:Create(btn, TweenInfo.new(0.2), {TextColor3 = Theme.TextSecondary}):Play()
                    end
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            
            TweenService:Create(subBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextPrimary}):Play()
            page.Visible = true
        end
        
        subBtn.Activated:Connect(ActivateSub)
        subObj.Activate = ActivateSub
        
        table.insert(tabObj.SubTabs, subObj)
        return page
    end
    
    return tabObj
end

--// UI COMPONENTS
local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page)
    section.Size = UDim2.new(0.98, 0, 0, 40)
    section.BackgroundColor3 = Theme.SectionBG
    section.BackgroundTransparency = Theme.SectionTransp
    Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    
    local secStroke = Instance.new("UIStroke", section)
    secStroke.Color = Theme.CardBorder
    secStroke.Transparency = 0.5
    
    local secLayout = Instance.new("UIListLayout", section)
    secLayout.Padding = UDim.new(0, 8)
    secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 12); Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 12)
    
    local title = Instance.new("TextLabel", section)
    title.Size = UDim2.new(0.92, 0, 0, 18)
    title.BackgroundTransparency = 1
    title.Text = titleText
    title.TextColor3 = Theme.AccentGlow
    title.Font = Enum.Font.GothamBold
    title.TextSize = 13
    title.TextXAlignment = Enum.TextXAlignment.Left
    
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        section.Size = UDim2.new(0.98, 0, 0, secLayout.AbsoluteContentSize.Y + 24)
    end)
    return section
end

local function CreateToggle(section, text, defaultState, callback)
    local frame = Instance.new("Frame", section)
    frame.Size = UDim2.new(0.92, 0, 0, 34)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Theme.TextPrimary
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local tBtn = Instance.new("TextButton", frame)
    tBtn.Size = UDim2.new(0, 42, 0, 22)
    tBtn.Position = UDim2.new(1, -42, 0.5, -11)
    tBtn.BackgroundColor3 = defaultState and Theme.Accent or Theme.ToggleOff
    tBtn.Text = ""
    Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    
    local indicator = Instance.new("Frame", tBtn)
    indicator.Size = UDim2.new(0, 18, 0, 18)
    indicator.Position = defaultState and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)
    indicator.BackgroundColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultState
    tBtn.Activated:Connect(function()
        isOn = not isOn
        TweenService:Create(tBtn, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {BackgroundColor3 = isOn and Theme.Accent or Theme.ToggleOff}):Play()
        TweenService:Create(indicator, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Position = isOn and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)}):Play()
        if callback then callback(isOn) end
    end)
end

local function CreateSlider(section, text, min, max, default, callback)
    local frame = Instance.new("Frame", section)
    frame.Size = UDim2.new(0.92, 0, 0, 48)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 0, 18)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Theme.TextSecondary
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local valLabel = Instance.new("TextLabel", frame)
    valLabel.Size = UDim2.new(0.3, 0, 0, 18)
    valLabel.Position = UDim2.new(0.7, 0, 0, 0)
    valLabel.BackgroundTransparency = 1
    valLabel.Text = tostring(default)
    valLabel.TextColor3 = Theme.AccentGlow
    valLabel.Font = Enum.Font.GothamBold
    valLabel.TextSize = 12
    valLabel.TextXAlignment = Enum.TextXAlignment.Right
    
    local sliderBg = Instance.new("Frame", frame)
    sliderBg.Size = UDim2.new(1, 0, 0, 6)
    sliderBg.Position = UDim2.new(0, 0, 0, 28)
    sliderBg.BackgroundColor3 = Theme.ToggleOff
    Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    
    local sliderFill = Instance.new("Frame", sliderBg)
    local rel = (default - min) / (max - min)
    sliderFill.Size = UDim2.new(rel, 0, 1, 0)
    sliderFill.BackgroundColor3 = Theme.Accent
    Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    
    local dragging = false
    local function setVal(input)
        local pos = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
        local val = math.floor(min + (max - min) * pos)
        valLabel.Text = tostring(val)
        TweenService:Create(sliderFill, TweenInfo.new(0.1), {Size = UDim2.new(pos, 0, 1, 0)}):Play()
        if callback then callback(val) end
    end
    
    sliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true; setVal(input)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            setVal(input)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

local function CreateButton(section, text, callback)
    local btn = Instance.new("TextButton", section)
    btn.Size = UDim2.new(0.92, 0, 0, 36)
    btn.BackgroundColor3 = Theme.SectionBG
    btn.BackgroundTransparency = 0.1
    btn.Text = text
    btn.TextColor3 = Theme.TextPrimary
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    
    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = Theme.CardBorder
    stroke.Thickness = 1
    
    btn.Activated:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Theme.Accent, TextColor3 = Color3.new(0,0,0)}):Play()
        task.wait(0.1)
        TweenService:Create(btn, TweenInfo.new(0.25), {BackgroundColor3 = Theme.SectionBG, TextColor3 = Theme.TextPrimary}):Play()
        if callback then callback() end
    end)
end

--// ============================================================================
--// GUI INITIALIZATION (ONLY UI TABS)
--// ============================================================================

-- Main UI Elements Demo
local TabMain = CreateMainTab("Main", "🏠")
local SubGeneral = TabMain:CreateSubTab("General Options")
local SecGeneral = CreateSection(SubGeneral, "Frontend Options")
CreateToggle(SecGeneral, "Example Toggle 1", false, function() print("Toggle 1 clicked") end)
CreateToggle(SecGeneral, "Example Toggle 2", true, function() print("Toggle 2 clicked") end)
CreateButton(SecGeneral, "Click Me", function() print("Button clicked") end)

local SubVisuals = TabMain:CreateSubTab("Visuals")
local SecVis = CreateSection(SubVisuals, "Visual Adjustments")
CreateSlider(SecVis, "Brightness", 0, 100, 50, function(val) print(val) end)

-- Extra Demo Tab
local TabExtra = CreateMainTab("Extra", "✨")
local SubExtra1 = TabExtra:CreateSubTab("Settings")
local SecExtra = CreateSection(SubExtra1, "Misc Settings")
CreateToggle(SecExtra, "Enable Features", false, function() end)

-- Settings (Scale Adjuster included)
local TabSet = CreateMainTab("Settings", "⚙️")
local SubUI = TabSet:CreateSubTab("UI Scale")
local SecScale = CreateSection(SubUI, "Custom Scale Adjuster")

CreateSlider(SecScale, "GUI Scale Size", 40, 120, math.floor(baseScale * 100), function(v)
    GlobalScale.Scale = v / 100
end)

-- INIT FIRST TAB
task.delay(0.1, function()
    Categories[1].IsOpen = true
    Categories[1].Button.TextColor3 = Theme.Accent
    Categories[1].Button:FindFirstChildOfClass("UIStroke").Color = Theme.Accent
    
    local targetHeight = 40 + Categories[1].SubContainer:FindFirstChildOfClass("UIListLayout").AbsoluteContentSize.Y + 4
    Categories[1].Wrapper.Size = UDim2.new(1, 0, 0, targetHeight)
    
    if #Categories[1].SubTabs > 0 then Categories[1].SubTabs[1].Activate() end
end)
