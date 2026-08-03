--// ============================================================================
--// RYU HUB - DIAMOND GLASS EDITION (ULTRA PREMIUM & SMART SCALING)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

--// SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function()
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end
end)

for _, v in pairs(guiParent:GetChildren()) do
    if v.Name == "RyuHubDiamondGlass" then v:Destroy() end
end

--// DIAMOND GLASS THEME
local Theme = {
    -- Transparenz macht den Glass-Effekt aus
    Background    = Color3.fromRGB(8, 12, 20),         
    BgTransp      = 0.25,                              -- Glassmorphism Transparenz
    Sidebar       = Color3.fromRGB(12, 18, 30),
    SectionBG     = Color3.fromRGB(18, 28, 45),
    SectionTransp = 0.3,
    
    CardBorder    = Color3.fromRGB(60, 140, 180),      -- Dunklerer Rahmen
    TextPrimary   = Color3.fromRGB(250, 255, 255),     -- Kristallweiß
    TextSecondary = Color3.fromRGB(140, 200, 235),     -- Eisblau
    
    DiamondBlue   = Color3.fromRGB(85, 235, 255),      -- Minecraft Diamond
    DiamondGlow   = Color3.fromRGB(160, 250, 255),     -- Heller Glow
    ToggleOff     = Color3.fromRGB(25, 40, 60),
    
    -- Gradient für die Welle
    WaveColors = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(85, 235, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)), -- Pure White Center
        ColorSequenceKeypoint.new(1, Color3.fromRGB(85, 235, 255))
    })
}

--// SMART MOBILE SCALING
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local screenY = Camera.ViewportSize.Y
local baseScale = 1.0

-- Wenn auf Handy, skaliere dynamisch auf Basis der Bildschirmhöhe
if isMobile then
    baseScale = math.clamp(screenY / 700, 0.45, 0.75) -- Perfekte Passform
end

--// MAIN SCREEN GUI
local RyuHubGui = Instance.new("ScreenGui")
RyuHubGui.Name = "RyuHubDiamondGlass"
RyuHubGui.ResetOnSpawn = false
RyuHubGui.IgnoreGuiInset = true
RyuHubGui.Parent = guiParent

local GlobalScale = Instance.new("UIScale", RyuHubGui)
GlobalScale.Scale = baseScale

--// MAIN CONTAINER (GLASS)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 640, 0, 400) -- Kompaktere, edle Basis-Größe
MainFrame.Position = UDim2.new(0.5, -320, 0.5, -200)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BackgroundTransparency = Theme.BgTransp
MainFrame.Active = true
MainFrame.Parent = RyuHubGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

-- LEUCHTENDER DIAMANT RAHMEN
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 1.5
MainStroke.Color = Theme.DiamondBlue
MainStroke.Transparency = 0.2

--// TOPBAR
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Name = "Topbar"
Topbar.Size = UDim2.new(1, 0, 0, 50)
Topbar.BackgroundTransparency = 1

--// PERFECT WAVE ANIMATION TITLE
local TitleLabel = Instance.new("TextLabel", Topbar)
TitleLabel.Size = UDim2.new(0, 150, 1, 0)
TitleLabel.Position = UDim2.new(0, 20, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "RYU HUB"
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextSize = 24
TitleLabel.TextColor3 = Color3.new(1,1,1)
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- UIGradient für den weichen Shine-Effekt
local TitleGradient = Instance.new("UIGradient", TitleLabel)
TitleGradient.Color = Theme.WaveColors
TitleGradient.Rotation = 25
TitleGradient.Offset = Vector2.new(-1, 0)

-- Tween für die Welle
local waveTweenInfo = TweenInfo.new(2.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, false)
local waveTween = TweenService:Create(TitleGradient, waveTweenInfo, {Offset = Vector2.new(1, 0)})
waveTween:Play()

-- DIAMOND BADGE
local Badge = Instance.new("TextLabel", Topbar)
Badge.Size = UDim2.new(0, 80, 0, 18)
Badge.Position = UDim2.new(0, 135, 0.5, -9)
Badge.BackgroundColor3 = Theme.SectionBG
Badge.BackgroundTransparency = Theme.SectionTransp
Badge.Text = "DIAMOND"
Badge.Font = Enum.Font.GothamBold
Badge.TextSize = 10
Badge.TextColor3 = Theme.DiamondGlow
Instance.new("UICorner", Badge).CornerRadius = UDim.new(0, 4)
local BadgeStroke = Instance.new("UIStroke", Badge)
BadgeStroke.Color = Theme.DiamondBlue
BadgeStroke.Thickness = 1

-- CLOSE BUTTON
local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -36, 0.5, -13)
CloseBtn.BackgroundColor3 = Theme.SectionBG
CloseBtn.BackgroundTransparency = Theme.SectionTransp
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextSecondary
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 13
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
local CloseStroke = Instance.new("UIStroke", CloseBtn)
CloseStroke.Color = Theme.CardBorder

CloseBtn.Activated:Connect(function()
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
    task.wait(0.3)
    MainFrame.Visible = false
end)

-- FLOATING TOGGLE BUTTON (💎)
local ToggleBtn = Instance.new("TextButton", RyuHubGui)
ToggleBtn.Name = "RyuToggleBtn"
ToggleBtn.Size = UDim2.new(0, 46, 0, 46)
ToggleBtn.Position = UDim2.new(0, 20, 0.15, 0)
ToggleBtn.BackgroundColor3 = Theme.Background
ToggleBtn.BackgroundTransparency = 0.1
ToggleBtn.Text = "💎"
ToggleBtn.TextSize = 20
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local ToggleStroke = Instance.new("UIStroke", ToggleBtn)
ToggleStroke.Color = Theme.DiamondBlue
ToggleStroke.Thickness = 1.5

ToggleBtn.Activated:Connect(function()
    if MainFrame.Visible then
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
        task.wait(0.3)
        MainFrame.Visible = false
    else
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 640, 0, 400), BackgroundTransparency = Theme.BgTransp}):Play()
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

--// 2-COLUMN ACCORDION LAYOUT
local BodyContainer = Instance.new("Frame", MainFrame)
BodyContainer.Size = UDim2.new(1, -20, 1, -60)
BodyContainer.Position = UDim2.new(0, 10, 0, 50)
BodyContainer.BackgroundTransparency = 1

-- SPALTE 1: ACCORDION SIDEBAR
local Sidebar = Instance.new("ScrollingFrame", BodyContainer)
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 150, 1, 0)
Sidebar.BackgroundTransparency = 1
Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 6)

-- SPALTE 2: CONTENT AREA
local ContentArea = Instance.new("Frame", BodyContainer)
ContentArea.Name = "ContentArea"
ContentArea.Size = UDim2.new(1, -165, 1, 0)
ContentArea.Position = UDim2.new(0, 165, 0, 0)
ContentArea.BackgroundTransparency = 1

-- UI LIST LAYOUT UPDATER
local function UpdateSidebarCanvas()
    local totalH = 0
    for _, child in pairs(Sidebar:GetChildren()) do
        if child:IsA("GuiObject") and child.Visible then
            totalH = totalH + child.AbsoluteSize.Y + 6
        end
    end
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH + 10)
end

--// TAB SYSTEM (ACCORDION)
local Categories = {}

function CreateMainTab(tabName, icon)
    icon = icon or "🔹"
    local tabObj = { Name = tabName, Button = nil, SubContainer = nil, IsOpen = false, SubTabs = {} }
    
    -- Main Button
    local tabBtn = Instance.new("TextButton", Sidebar)
    tabBtn.Size = UDim2.new(1, 0, 0, 36)
    tabBtn.BackgroundColor3 = Theme.Sidebar
    tabBtn.BackgroundTransparency = Theme.SectionTransp
    tabBtn.Text = "  " .. icon .. "  " .. tabName
    tabBtn.TextColor3 = Theme.TextSecondary
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 12
    tabBtn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)
    local btnStroke = Instance.new("UIStroke", tabBtn)
    btnStroke.Color = Theme.CardBorder
    tabObj.Button = tabBtn
    
    -- Sub Container (Fährt nach unten aus)
    local subContainer = Instance.new("Frame", Sidebar)
    subContainer.Size = UDim2.new(1, 0, 0, 0)
    subContainer.BackgroundTransparency = 1
    subContainer.ClipsDescendants = true
    
    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 4)
    tabObj.SubContainer = subContainer
    
    -- Accordion Logic
    tabBtn.Activated:Connect(function()
        tabObj.IsOpen = not tabObj.IsOpen
        
        -- Style Update
        tabBtn.TextColor3 = tabObj.IsOpen and Theme.DiamondBlue or Theme.TextSecondary
        btnStroke.Color = tabObj.IsOpen and Theme.DiamondBlue or Theme.CardBorder
        
        -- Tween Container Size
        local targetHeight = tabObj.IsOpen and subLayout.AbsoluteContentSize.Y or 0
        local tw = TweenService:Create(subContainer, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, targetHeight)})
        tw:Play()
        
        -- Dynamically update sidebar canvas while tweening
        task.spawn(function()
            for i = 1, 15 do
                UpdateSidebarCanvas()
                task.wait(0.02)
            end
        end)
    end)
    
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if tabObj.IsOpen then
            subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y)
            UpdateSidebarCanvas()
        end
    end)
    
    table.insert(Categories, tabObj)
    
    -- CREATE SUB TABS
    function tabObj:CreateSubTab(subName)
        local subObj = { Name = subName, Page = nil }
        
        local subBtn = Instance.new("TextButton", subContainer)
        subBtn.Size = UDim2.new(1, -15, 0, 28)
        subBtn.Position = UDim2.new(0, 15, 0, 0) -- Eingerückt
        subBtn.BackgroundTransparency = 1
        subBtn.Text = "•  " .. subName
        subBtn.TextColor3 = Theme.TextSecondary
        subBtn.Font = Enum.Font.GothamMedium
        subBtn.TextSize = 11
        subBtn.TextXAlignment = Enum.TextXAlignment.Left
        
        local page = Instance.new("ScrollingFrame", ContentArea)
        page.Name = subName .. "_Page"
        page.Size = UDim2.new(1, 0, 1, 0)
        page.BackgroundTransparency = 1
        page.ScrollBarThickness = 2
        page.ScrollBarImageColor3 = Theme.DiamondBlue
        page.Visible = false
        
        local pageLayout = Instance.new("UIListLayout", page)
        pageLayout.Padding = UDim.new(0, 10)
        pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        
        pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 15)
        end)
        
        local function ActivateSub()
            -- Reset all subtabs
            for _, cat in pairs(Categories) do
                for _, btn in pairs(cat.SubContainer:GetChildren()) do
                    if btn:IsA("TextButton") then btn.TextColor3 = Theme.TextSecondary end
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            
            subBtn.TextColor3 = Theme.TextPrimary
            page.Visible = true
        end
        
        subBtn.Activated:Connect(ActivateSub)
        subObj.Activate = ActivateSub
        
        table.insert(tabObj.SubTabs, subObj)
        return page
    end
    
    return tabObj
end

--// UI COMPONENTS (GLASS SECTIONS)
function CreateSection(page, titleText)
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
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 10); Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 10)
    
    local title = Instance.new("TextLabel", section)
    title.Size = UDim2.new(0.92, 0, 0, 18)
    title.BackgroundTransparency = 1
    title.Text = titleText
    title.TextColor3 = Theme.DiamondGlow
    title.Font = Enum.Font.GothamBold
    title.TextSize = 12
    title.TextXAlignment = Enum.TextXAlignment.Left
    
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        section.Size = UDim2.new(0.98, 0, 0, secLayout.AbsoluteContentSize.Y + 20)
    end)
    return section
end

function CreateToggle(section, text, defaultState, callback)
    local frame = Instance.new("Frame", section)
    frame.Size = UDim2.new(0.92, 0, 0, 32)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Theme.TextPrimary
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local tBtn = Instance.new("TextButton", frame)
    tBtn.Size = UDim2.new(0, 40, 0, 20)
    tBtn.Position = UDim2.new(1, -40, 0.5, -10)
    tBtn.BackgroundColor3 = defaultState and Theme.DiamondBlue or Theme.ToggleOff
    tBtn.Text = ""
    Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultState
    tBtn.Activated:Connect(function()
        isOn = not isOn
        TweenService:Create(tBtn, TweenInfo.new(0.2), {BackgroundColor3 = isOn and Theme.DiamondBlue or Theme.ToggleOff}):Play()
        if callback then callback(isOn) end
    end)
end

function CreateSlider(section, text, min, max, default, callback)
    local frame = Instance.new("Frame", section)
    frame.Size = UDim2.new(0.92, 0, 0, 45)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 0, 18)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Theme.TextSecondary
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local valLabel = Instance.new("TextLabel", frame)
    valLabel.Size = UDim2.new(0.3, 0, 0, 18)
    valLabel.Position = UDim2.new(0.7, 0, 0, 0)
    valLabel.BackgroundTransparency = 1
    valLabel.Text = tostring(default)
    valLabel.TextColor3 = Theme.DiamondGlow
    valLabel.Font = Enum.Font.GothamBold
    valLabel.TextSize = 11
    valLabel.TextXAlignment = Enum.TextXAlignment.Right
    
    local sliderBg = Instance.new("Frame", frame)
    sliderBg.Size = UDim2.new(1, 0, 0, 6)
    sliderBg.Position = UDim2.new(0, 0, 0, 26)
    sliderBg.BackgroundColor3 = Theme.ToggleOff
    Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    
    local sliderFill = Instance.new("Frame", sliderBg)
    local rel = (default - min) / (max - min)
    sliderFill.Size = UDim2.new(rel, 0, 1, 0)
    sliderFill.BackgroundColor3 = Theme.DiamondBlue
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

function CreateButton(section, text, callback)
    local btn = Instance.new("TextButton", section)
    btn.Size = UDim2.new(0.92, 0, 0, 32)
    btn.BackgroundColor3 = Theme.SectionBG
    btn.BackgroundTransparency = 0.1
    btn.Text = text
    btn.TextColor3 = Theme.TextPrimary
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = Theme.CardBorder
    stroke.Thickness = 1
    
    btn.Activated:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Theme.DiamondBlue, TextColor3 = Color3.new(0,0,0)}):Play()
        task.wait(0.1)
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.SectionBG, TextColor3 = Theme.TextPrimary}):Play()
        if callback then callback() end
    end)
end

--// ============================================================================
--// GUI INITIALIZATION (EXAMPLE TABS)
--// ============================================================================

-- Main
local TabMain = CreateMainTab("Main", "🏠")
local SubGeneral = TabMain:CreateSubTab("General")
local SecGeneral = CreateSection(SubGeneral, "Player Setup")
CreateToggle(SecGeneral, "Enable ESP", false, function() end)

-- Impel Down
local TabImpel = CreateMainTab("Impel Down", "💎")
local SubFarm = TabImpel:CreateSubTab("Auto Farm")
local SecFarm = CreateSection(SubFarm, "Dungeon Farm")
CreateToggle(SecFarm, "Auto Clear Floor", false, function() end)
CreateButton(SecFarm, "Start Impel Down TP", function() end)

-- Settings
local TabSet = CreateMainTab("Settings", "⚙️")
local SubUI = TabSet:CreateSubTab("UI Scale")
local SecScale = CreateSection(SubUI, "Custom Scale Adjuster")
CreateSlider(SecScale, "GUI Scale Size", 40, 120, math.floor(baseScale * 100), function(v)
    GlobalScale.Scale = v / 100
end)

-- SET DEFAULT TAB
task.delay(0.1, function()
    -- Tab Klicken simulieren
    Categories[1].Button.BackgroundColor3 = Theme.SectionBG
    Categories[1].Button.TextColor3 = Theme.DiamondBlue
    Categories[1].Button:FindFirstChildOfClass("UIStroke").Color = Theme.DiamondBlue
    
    Categories[1].IsOpen = true
    local targetHeight = Categories[1].SubContainer:FindFirstChildOfClass("UIListLayout").AbsoluteContentSize.Y
    Categories[1].SubContainer.Size = UDim2.new(1, 0, 0, targetHeight)
    Categories[1].SubContainer.Visible = true
    
    if #Categories[1].SubTabs > 0 then Categories[1].SubTabs[1].Activate() end
    UpdateSidebarCanvas()
end)
