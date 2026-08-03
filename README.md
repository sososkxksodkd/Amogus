--// ============================================================================
--// RYU HUB - DIAMOND EDITION (ULTRA PREMIUM & RESPONSIVE MOBILE/PC)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")

local LocalPlayer = Players.LocalPlayer

--// CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function()
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end
end)

for _, v in pairs(guiParent:GetChildren()) do
    if v.Name == "RyuHubDiamondUltra" then v:Destroy() end
end

--// CENTRAL CONFIGURATION & THEME
local Theme = {
    Background    = Color3.fromRGB(11, 16, 25),        -- Deep Diamond Void
    Sidebar       = Color3.fromRGB(16, 24, 38),        -- Dark Aquamarine
    SubSidebar    = Color3.fromRGB(20, 31, 48),        -- Ice Panel
    SectionBG     = Color3.fromRGB(26, 40, 60),        -- Card Color
    CardBorder    = Color3.fromRGB(45, 90, 125),       -- Card Stroke
    TextPrimary   = Color3.fromRGB(245, 252, 255),      -- Crystal White
    TextSecondary = Color3.fromRGB(135, 190, 220),      -- Soft Ice Blue
    DiamondBlue   = Color3.fromRGB(85, 235, 255),       -- Minecraft Diamond Cyan
    DiamondGlow   = Color3.fromRGB(160, 245, 255),      -- Pure Diamond Light
    ToggleOff     = Color3.fromRGB(35, 50, 70),
    AccentGradients = {
        ColorSequenceKeypoint.new(0, Color3.fromRGB(85, 235, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(180, 250, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(40, 180, 230))
    }
}

--// MOBILE vs PC DETECTOR & AUTO-SCALE
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local defaultScale = isMobile and 0.72 or 1.0 -- Kleiner für Handy, Normal für PC

--// MAIN SCREEN GUI
local RyuHubGui = Instance.new("ScreenGui")
RyuHubGui.Name = "RyuHubDiamondUltra"
RyuHubGui.ResetOnSpawn = false
RyuHubGui.IgnoreGuiInset = true
RyuHubGui.Parent = guiParent

-- UI SCALE CONTROLLER
local GlobalScale = Instance.new("UIScale")
GlobalScale.Scale = defaultScale
GlobalScale.Parent = RyuHubGui

--// MAIN CONTAINER FRAME
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 720, 0, 440)
MainFrame.Position = UDim2.new(0.5, -360, 0.5, -220)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.ClipsDescendants = false
MainFrame.Active = true
MainFrame.Parent = RyuHubGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)

-- GLOW STROKE EFFECT
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 2
MainStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

local MainGradient = Instance.new("UIGradient", MainStroke)
MainGradient.Color = ColorSequence.new(Theme.AccentGradients)
MainGradient.Rotation = 45

-- ANIMATE STROKE GRADIENT
task.spawn(function()
    while MainGradient and MainGradient.Parent do
        MainGradient.Rotation = (MainGradient.Rotation + 1) % 360
        task.wait(0.03)
    end
end)

--// TOPBAR
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Name = "Topbar"
Topbar.Size = UDim2.new(1, 0, 0, 52)
Topbar.BackgroundTransparency = 1

-- ANIMATED WAVE TITLE ("RyuHub")
local TitleLabel = Instance.new("TextLabel", Topbar)
TitleLabel.Size = UDim2.new(0, 160, 1, 0)
TitleLabel.Position = UDim2.new(0, 20, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "RyuHub"
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextSize = 22
TitleLabel.TextColor3 = Theme.TextPrimary
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

task.spawn(function()
    local textStr = "RyuHub"
    local length = #textStr
    local waveIndex = 1
    
    while TitleLabel and TitleLabel.Parent do
        local formattedText = ""
        for i = 1, length do
            local char = textStr:sub(i, i)
            if i == waveIndex then
                formattedText = formattedText .. string.format('<font color="rgb(255, 255, 255)"><b>%s</b></font>', char)
            elseif i == waveIndex - 1 or i == waveIndex + 1 then
                formattedText = formattedText .. string.format('<font color="rgb(180, 245, 255)">%s</font>', char)
            else
                formattedText = formattedText .. string.format('<font color="rgb(85, 235, 255)">%s</font>', char)
            end
        end
        TitleLabel.RichText = true
        TitleLabel.Text = formattedText
        
        waveIndex = waveIndex + 1
        if waveIndex > length + 2 then waveIndex = 1 end
        task.wait(0.12)
    end
end)

-- DIAMOND EDITION BADGE
local Badge = Instance.new("Frame", Topbar)
Badge.Size = UDim2.new(0, 110, 0, 22)
Badge.Position = UDim2.new(0, 125, 0.5, -11)
Badge.BackgroundColor3 = Theme.SectionBG
Instance.new("UICorner", Badge).CornerRadius = UDim.new(0, 6)

local BadgeStroke = Instance.new("UIStroke", Badge)
BadgeStroke.Color = Theme.DiamondBlue
BadgeStroke.Thickness = 1

local BadgeText = Instance.new("TextLabel", Badge)
BadgeText.Size = UDim2.new(1, 0, 1, 0)
BadgeText.BackgroundTransparency = 1
BadgeText.Text = "💎 DIAMOND"
BadgeText.Font = Enum.Font.GothamBold
BadgeText.TextSize = 10
BadgeText.TextColor3 = Theme.DiamondBlue

-- CLOSE & MINIMIZE BUTTONS
local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -40, 0, 11)
CloseBtn.BackgroundColor3 = Theme.SectionBG
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextSecondary
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

local CloseStroke = Instance.new("UIStroke", CloseBtn)
CloseStroke.Color = Theme.CardBorder
CloseStroke.Thickness = 1

CloseBtn.Activated:Connect(function()
    MainFrame.Visible = false
end)

-- TOGGLE FLOATING BUTTON (Für Mobile & PC)
local ToggleBtn = Instance.new("TextButton", RyuHubGui)
ToggleBtn.Name = "RyuToggleBtn"
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 25, 0.15, 0)
ToggleBtn.BackgroundColor3 = Theme.Background
ToggleBtn.Text = "💎"
ToggleBtn.TextSize = 22
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local ToggleStroke = Instance.new("UIStroke", ToggleBtn)
ToggleStroke.Color = Theme.DiamondBlue
ToggleStroke.Thickness = 2

ToggleBtn.Activated:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- DRAGGING ENGINE FOR TOGGLE BUTTON & MAINFRAME
local function MakeDraggable(guiObject, handleObject)
    local dragging, dragStart, startPos
    handleObject.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = guiObject.Position
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

--// 3-COLUMN LAYOUT SETUP
local BodyContainer = Instance.new("Frame", MainFrame)
BodyContainer.Size = UDim2.new(1, -24, 1, -66)
BodyContainer.Position = UDim2.new(0, 12, 0, 56)
BodyContainer.BackgroundTransparency = 1

-- SPALTE 1: SIDEBAR (MAIN CATEGORIES)
local Sidebar = Instance.new("ScrollingFrame", BodyContainer)
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 140, 1, 0)
Sidebar.BackgroundTransparency = 1
Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 6)

-- SPALTE 2: SUB-TABS PANEL
local SubTabSidebar = Instance.new("Frame", BodyContainer)
SubTabSidebar.Name = "SubTabSidebar"
SubTabSidebar.Size = UDim2.new(0, 140, 1, 0)
SubTabSidebar.Position = UDim2.new(0, 148, 0, 0)
SubTabSidebar.BackgroundColor3 = Theme.Sidebar
Instance.new("UICorner", SubTabSidebar).CornerRadius = UDim.new(0, 10)
local SubSideStroke = Instance.new("UIStroke", SubTabSidebar)
SubSideStroke.Color = Theme.CardBorder
SubSideStroke.Thickness = 1

-- SPALTE 3: CONTENT PANEL
local ContentArea = Instance.new("Frame", BodyContainer)
ContentArea.Name = "ContentArea"
ContentArea.Size = UDim2.new(1, -298, 1, 0)
ContentArea.Position = UDim2.new(0, 298, 0, 0)
ContentArea.BackgroundTransparency = 1

--// DYNAMIC CATEGORY & SUB-TAB SYSTEM
local Categories = {}

function CreateMainTab(tabName, icon)
    icon = icon or "🔹"
    local tabObj = { Name = tabName, Button = nil, SubContainer = nil, SubTabs = {} }
    
    local tabBtn = Instance.new("TextButton", Sidebar)
    tabBtn.Size = UDim2.new(1, 0, 0, 40)
    tabBtn.BackgroundColor3 = Theme.Sidebar
    tabBtn.Text = "   " .. icon .. "  " .. tabName
    tabBtn.TextColor3 = Theme.TextSecondary
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 12
    tabBtn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)
    
    local btnStroke = Instance.new("UIStroke", tabBtn)
    btnStroke.Color = Theme.CardBorder
    btnStroke.Thickness = 1
    tabObj.Button = tabBtn
    
    local subContainer = Instance.new("ScrollingFrame", SubTabSidebar)
    subContainer.Name = tabName .. "_Subs"
    subContainer.Size = UDim2.new(1, -12, 1, -12)
    subContainer.Position = UDim2.new(0, 6, 0, 6)
    subContainer.BackgroundTransparency = 1
    subContainer.ScrollBarThickness = 0
    subContainer.Visible = false
    
    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 5)
    tabObj.SubContainer = subContainer
    
    tabBtn.Activated:Connect(function()
        for _, cat in pairs(Categories) do
            cat.Button.BackgroundColor3 = Theme.Sidebar
            cat.Button.TextColor3 = Theme.TextSecondary
            cat.Button:FindFirstChildOfClass("UIStroke").Color = Theme.CardBorder
            cat.SubContainer.Visible = false
        end
        tabBtn.BackgroundColor3 = Theme.SectionBG
        tabBtn.TextColor3 = Theme.DiamondBlue
        btnStroke.Color = Theme.DiamondBlue
        subContainer.Visible = true
        
        if #tabObj.SubTabs > 0 then
            tabObj.SubTabs[1].Activate()
        end
    end)
    
    table.insert(Categories, tabObj)
    
    function tabObj:CreateSubTab(subName)
        local subObj = { Name = subName, Page = nil }
        
        local subBtn = Instance.new("TextButton", subContainer)
        subBtn.Size = UDim2.new(1, 0, 0, 32)
        subBtn.BackgroundColor3 = Theme.Background
        subBtn.Text = "   " .. subName
        subBtn.TextColor3 = Theme.TextSecondary
        subBtn.Font = Enum.Font.GothamMedium
        subBtn.TextSize = 11
        subBtn.TextXAlignment = Enum.TextXAlignment.Left
        Instance.new("UICorner", subBtn).CornerRadius = UDim.new(0, 6)
        
        local subBtnStroke = Instance.new("UIStroke", subBtn)
        subBtnStroke.Color = Theme.CardBorder
        subBtnStroke.Thickness = 1
        
        local page = Instance.new("ScrollingFrame", ContentArea)
        page.Name = subName .. "_Page"
        page.Size = UDim2.new(1, 0, 1, 0)
        page.BackgroundTransparency = 1
        page.ScrollBarThickness = 2
        page.ScrollBarImageColor3 = Theme.DiamondBlue
        page.Visible = false
        
        local pageLayout = Instance.new("UIListLayout", page)
        pageLayout.Padding = UDim.new(0, 8)
        
        pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 12)
        end)
        
        local function ActivateSub()
            for _, s in pairs(subContainer:GetChildren()) do
                if s:IsA("TextButton") then
                    s.TextColor3 = Theme.TextSecondary
                    s.BackgroundColor3 = Theme.Background
                    s:FindFirstChildOfClass("UIStroke").Color = Theme.CardBorder
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            subBtn.TextColor3 = Theme.DiamondBlue
            subBtn.BackgroundColor3 = Theme.SectionBG
            subBtnStroke.Color = Theme.DiamondBlue
            page.Visible = true
        end
        
        subBtn.Activated:Connect(ActivateSub)
        subObj.Activate = ActivateSub
        
        table.insert(tabObj.SubTabs, subObj)
        return page
    end
    
    return tabObj
end

--// UI COMPONENT BUILDERS
function CreateSection(page, titleText)
    local section = Instance.new("Frame", page)
    section.Size = UDim2.new(1, 0, 0, 40)
    section.BackgroundColor3 = Theme.Sidebar
    Instance.new("UICorner", section).CornerRadius = UDim.new(0, 8)
    
    local secStroke = Instance.new("UIStroke", section)
    secStroke.Color = Theme.CardBorder
    secStroke.Thickness = 1
    
    local secLayout = Instance.new("UIListLayout", section)
    secLayout.Padding = UDim.new(0, 6)
    secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 8)
    Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 8)
    
    local title = Instance.new("TextLabel", section)
    title.Size = UDim2.new(0.92, 0, 0, 18)
    title.BackgroundTransparency = 1
    title.Text = titleText
    title.TextColor3 = Theme.DiamondBlue
    title.Font = Enum.Font.GothamBold
    title.TextSize = 12
    title.TextXAlignment = Enum.TextXAlignment.Left
    
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 16)
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

function CreateButton(section, text, callback)
    local btn = Instance.new("TextButton", section)
    btn.Size = UDim2.new(0.92, 0, 0, 32)
    btn.BackgroundColor3 = Theme.SectionBG
    btn.Text = text
    btn.TextColor3 = Theme.TextPrimary
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = Theme.CardBorder
    stroke.Thickness = 1
    
    btn.Activated:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Theme.DiamondBlue}):Play()
        task.wait(0.1)
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.SectionBG}):Play()
        if callback then callback() end
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
    valLabel.TextColor3 = Theme.DiamondBlue
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
        sliderFill.Size = UDim2.new(pos, 0, 1, 0)
        if callback then callback(val) end
    end
    
    sliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            setVal(input)
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

--// ============================================================================
--// CATEGORIES & GUI SETTINGS SETUP
--// ============================================================================

-- MAIN CATEGORY
local TabMain = CreateMainTab("Main", "🏠")
local SubGeneral = TabMain:CreateSubTab("General")
local SecGeneral = CreateSection(SubGeneral, "Player Settings")
CreateToggle(SecGeneral, "Enable Speed Boost", false, function(s) end)

-- IMPEL DOWN CATEGORY
local TabImpelDown = CreateMainTab("Impel Down", "💎")
local SubImpelFarm = TabImpelDown:CreateSubTab("Auto Farm")
local SecImpelFarm = CreateSection(SubImpelFarm, "Impel Down Dungeon")
CreateToggle(SecImpelFarm, "Auto Clear Floor", false, function(s) end)
CreateButton(SecImpelFarm, "Start Dungeon TP", function() end)

-- SETTINGS CATEGORY (GUI SCALE & CONFIGS)
local TabSettings = CreateMainTab("Settings", "⚙️")
local SubUIConfig = TabSettings:CreateSubTab("UI Settings")
local SecScale = CreateSection(SubUIConfig, "GUI Scaling (Mobile / PC)")

CreateSlider(SecScale, "GUI Scale Size", 50, 120, math.floor(defaultScale * 100), function(val)
    GlobalScale.Scale = val / 100
end)

-- DEFAULT ACTIVE TAB
Categories[1].Button.BackgroundColor3 = Theme.SectionBG
Categories[1].Button.TextColor3 = Theme.DiamondBlue
Categories[1].SubContainer.Visible = true
if #Categories[1].SubTabs > 0 then Categories[1].SubTabs[1].Activate() end
