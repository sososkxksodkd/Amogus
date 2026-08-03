--// ============================================================================
--// RYU HUB - DIAMOND EDITION (MINECRAFT DIAMOND THEME FRAMEWORK)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

--// SECURITY & PARENTING
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function()
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end
end)

for _, v in pairs(guiParent:GetChildren()) do
    if v.Name == "RyuHubDiamond" then v:Destroy() end
end

--// DIAMOND COLOR PALETTE
local DiamondTheme = {
    Background = Color3.fromRGB(10, 15, 22),       -- Sehr dunkles Diamant-Blau
    Sidebar = Color3.fromRGB(15, 22, 32),          -- Dunkelblau
    SectionBG = Color3.fromRGB(20, 30, 42),        -- Karten-Hintergrund
    Text = Color3.fromRGB(240, 248, 255),           -- Eisweiß
    SubText = Color3.fromRGB(130, 180, 205),       -- Pastellblau
    DiamondBlue = Color3.fromRGB(85, 235, 255),    -- Minecraft Diamant Hellblau
    DiamondGlow = Color3.fromRGB(120, 245, 255),   -- Strahlendes Weiß-Blau
    ToggleOff = Color3.fromRGB(30, 45, 60),
    Stroke = Color3.fromRGB(50, 120, 150)
}

--// MAIN SCREEN GUI
local RyuHubGui = Instance.new("ScreenGui")
RyuHubGui.Name = "RyuHubDiamond"
RyuHubGui.ResetOnSpawn = false
RyuHubGui.IgnoreGuiInset = true
RyuHubGui.Parent = guiParent

--// MAIN CONTAINER FRAME
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 680, 0, 420)
MainFrame.Position = UDim2.new(0.5, -340, 0.5, -210)
MainFrame.BackgroundColor3 = DiamondTheme.Background
MainFrame.ClipsDescendants = true
MainFrame.Active = true
MainFrame.Parent = RyuHubGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = DiamondTheme.DiamondBlue
MainStroke.Thickness = 1.5

--// TOPBAR
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Name = "Topbar"
Topbar.Size = UDim2.new(1, 0, 0, 50)
Topbar.BackgroundTransparency = 1

--// ANIMATED WAVE TITLE ("RYUHUB")
local TitleLabel = Instance.new("TextLabel", Topbar)
TitleLabel.Size = UDim2.new(0, 200, 1, 0)
TitleLabel.Position = UDim2.new(0, 20, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "RyuHub"
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextSize = 22
TitleLabel.TextColor3 = DiamondTheme.Text
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- DIAMOND SHINE WAVE EFFECT LOOP
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

-- DIAMOND BADGE (SUB TITLE)
local SubBadge = Instance.new("TextLabel", Topbar)
SubBadge.Size = UDim2.new(0, 100, 0, 20)
SubBadge.Position = UDim2.new(0, 120, 0.5, -10)
SubBadge.BackgroundColor3 = DiamondTheme.SectionBG
SubBadge.Text = "DIAMOND"
SubBadge.Font = Enum.Font.GothamBold
SubBadge.TextSize = 10
SubBadge.TextColor3 = DiamondTheme.DiamondBlue
Instance.new("UICorner", SubBadge).CornerRadius = UDim.new(0, 6)
local BadgeStroke = Instance.new("UIStroke", SubBadge)
BadgeStroke.Color = DiamondTheme.DiamondBlue
BadgeStroke.Thickness = 1

-- CLOSE BUTTON
local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28)
CloseBtn.Position = UDim2.new(1, -38, 0, 11)
CloseBtn.BackgroundColor3 = DiamondTheme.SectionBG
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = DiamondTheme.SubText
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 13
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

CloseBtn.Activated:Connect(function()
    MainFrame.Visible = false
end)

-- TOGGLE ICON BUTTON
local ToggleBtn = Instance.new("TextButton", RyuHubGui)
ToggleBtn.Size = UDim2.new(0, 45, 0, 45)
ToggleBtn.Position = UDim2.new(0, 20, 0.2, 0)
ToggleBtn.BackgroundColor3 = DiamondTheme.Background
ToggleBtn.Text = "💎"
ToggleBtn.TextSize = 20
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)
local ToggleStroke = Instance.new("UIStroke", ToggleBtn)
ToggleStroke.Color = DiamondTheme.DiamondBlue
ToggleStroke.Thickness = 1.5

ToggleBtn.Activated:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- DRAGGING ENGINE
local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - mDragStart
        MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        mDragging = false
    end
end)

--// 3-COLUMN CONTAINER SETUP
local BodyContainer = Instance.new("Frame", MainFrame)
BodyContainer.Size = UDim2.new(1, -20, 1, -65)
BodyContainer.Position = UDim2.new(0, 10, 0, 55)
BodyContainer.BackgroundTransparency = 1

-- SPALTE 1: SIDEBAR (MAIN TABS)
local Sidebar = Instance.new("ScrollingFrame", BodyContainer)
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 130, 1, 0)
Sidebar.BackgroundTransparency = 1
Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar)
SideLayout.Padding = UDim.new(0, 6)

-- SPALTE 2: SUB-TABS CONTAINER
local SubTabSidebar = Instance.new("Frame", BodyContainer)
SubTabSidebar.Name = "SubTabSidebar"
SubTabSidebar.Size = UDim2.new(0, 130, 1, 0)
SubTabSidebar.Position = UDim2.new(0, 140, 0, 0)
SubTabSidebar.BackgroundColor3 = DiamondTheme.Sidebar
Instance.new("UICorner", SubTabSidebar).CornerRadius = UDim.new(0, 8)

-- SPALTE 3: MAIN CONTENT AREA
local ContentArea = Instance.new("Frame", BodyContainer)
ContentArea.Name = "ContentArea"
ContentArea.Size = UDim2.new(1, -280, 1, 0)
ContentArea.Position = UDim2.new(0, 280, 0, 0)
ContentArea.BackgroundTransparency = 1

--// TAB SYSTEM CREATOR
local Categories = {}
local currentActiveMainTab = nil

function CreateMainTab(tabName, icon)
    icon = icon or "🔹"
    local tabObj = { Name = tabName, Button = nil, SubContainer = nil, SubTabs = {} }
    
    -- Main Tab Button
    local tabBtn = Instance.new("TextButton", Sidebar)
    tabBtn.Size = UDim2.new(1, 0, 0, 38)
    tabBtn.BackgroundColor3 = DiamondTheme.Sidebar
    tabBtn.Text = "  " .. icon .. "  " .. tabName
    tabBtn.TextColor3 = DiamondTheme.SubText
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 12
    tabBtn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)
    tabObj.Button = tabBtn
    
    -- Sub Tabs Container für diese Kategorie
    local subContainer = Instance.new("ScrollingFrame", SubTabSidebar)
    subContainer.Name = tabName .. "_Subs"
    subContainer.Size = UDim2.new(1, -10, 1, -10)
    subContainer.Position = UDim2.new(0, 5, 0, 5)
    subContainer.BackgroundTransparency = 1
    subContainer.ScrollBarThickness = 0
    subContainer.Visible = false
    local subLayout = Instance.new("UIListLayout", subContainer)
    subLayout.Padding = UDim.new(0, 4)
    tabObj.SubContainer = subContainer
    
    tabBtn.Activated:Connect(function()
        for _, cat in pairs(Categories) do
            cat.Button.BackgroundColor3 = DiamondTheme.Sidebar
            cat.Button.TextColor3 = DiamondTheme.SubText
            cat.SubContainer.Visible = false
        end
        tabBtn.BackgroundColor3 = DiamondTheme.SectionBG
        tabBtn.TextColor3 = DiamondTheme.DiamondBlue
        subContainer.Visible = true
        
        -- Ersten Subtab automatisch aktivieren
        if #tabObj.SubTabs > 0 then
            tabObj.SubTabs[1].Activate()
        end
    end)
    
    table.insert(Categories, tabObj)
    
    -- SUB TAB CREATOR
    function tabObj:CreateSubTab(subName)
        local subObj = { Name = subName, Page = nil }
        
        local subBtn = Instance.new("TextButton", subContainer)
        subBtn.Size = UDim2.new(1, 0, 0, 32)
        subBtn.BackgroundColor3 = DiamondTheme.Background
        subBtn.Text = "   " .. subName
        subBtn.TextColor3 = DiamondTheme.SubText
        subBtn.Font = Enum.Font.GothamMedium
        subBtn.TextSize = 11
        subBtn.TextXAlignment = Enum.TextXAlignment.Left
        Instance.new("UICorner", subBtn).CornerRadius = UDim.new(0, 6)
        
        -- Content Page für diesen Subtab
        local page = Instance.new("ScrollingFrame", ContentArea)
        page.Name = subName .. "_Page"
        page.Size = UDim2.new(1, 0, 1, 0)
        page.BackgroundTransparency = 1
        page.ScrollBarThickness = 2
        page.Visible = false
        local pageLayout = Instance.new("UIListLayout", page)
        pageLayout.Padding = UDim.new(0, 8)
        
        pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 10)
        end)
        
        local function ActivateSub()
            for _, s in pairs(subContainer:GetChildren()) do
                if s:IsA("TextButton") then
                    s.TextColor3 = DiamondTheme.SubText
                    s.BackgroundColor3 = DiamondTheme.Background
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            subBtn.TextColor3 = DiamondTheme.DiamondBlue
            subBtn.BackgroundColor3 = DiamondTheme.SectionBG
            page.Visible = true
        end
        
        subBtn.Activated:Connect(ActivateSub)
        subObj.Activate = ActivateSub
        
        table.insert(tabObj.SubTabs, subObj)
        return page
    end
    
    return tabObj
end

--// UI COMPONENT CREATORS FOR CONTENT PAGES
function CreateSection(page, titleText)
    local section = Instance.new("Frame", page)
    section.Size = UDim2.new(1, 0, 0, 40)
    section.BackgroundColor3 = DiamondTheme.Sidebar
    Instance.new("UICorner", section).CornerRadius = UDim.new(0, 8)
    
    local secLayout = Instance.new("UIListLayout", section)
    secLayout.Padding = UDim.new(0, 6)
    secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 8); Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 8)
    
    local title = Instance.new("TextLabel", section)
    title.Size = UDim2.new(0.92, 0, 0, 20)
    title.BackgroundTransparency = 1
    title.Text = titleText
    title.TextColor3 = DiamondTheme.DiamondBlue
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
    frame.Size = UDim2.new(0.92, 0, 0, 30)
    frame.BackgroundTransparency = 1
    
    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = DiamondTheme.Text
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local tBtn = Instance.new("TextButton", frame)
    tBtn.Size = UDim2.new(0, 38, 0, 20)
    tBtn.Position = UDim2.new(1, -38, 0.5, -10)
    tBtn.BackgroundColor3 = defaultState and DiamondTheme.DiamondBlue or DiamondTheme.ToggleOff
    tBtn.Text = ""
    Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    
    local isOn = defaultState
    tBtn.Activated:Connect(function()
        isOn = not isOn
        tBtn.BackgroundColor3 = isOn and DiamondTheme.DiamondBlue or DiamondTheme.ToggleOff
        if callback then callback(isOn) end
    end)
end

function CreateButton(section, text, callback)
    local btn = Instance.new("TextButton", section)
    btn.Size = UDim2.new(0.92, 0, 0, 30)
    btn.BackgroundColor3 = DiamondTheme.SectionBG
    btn.Text = text
    btn.TextColor3 = DiamondTheme.Text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = DiamondTheme.Stroke
    stroke.Thickness = 1
    
    btn.Activated:Connect(function()
        if callback then callback() end
    end)
end

--// ============================================================================
--// EXAMPLE STRUCTURE (IMPEL DOWN & MAIN CATEGORIES)
--// ============================================================================

local TabMain = CreateMainTab("Main", "🏠")
local SubGeneral = TabMain:CreateSubTab("General")
local SecGeneral = CreateSection(SubGeneral, "General Features")
CreateToggle(SecGeneral, "Enable Speed Boost", false, function(s) end)

local TabImpelDown = CreateMainTab("Impel Down", "💎")
local SubFarm = TabImpelDown:CreateSubTab("Auto Farm")
local SecImpelFarm = CreateSection(SubFarm, "Impel Down Farm")
CreateToggle(SecImpelFarm, "Auto Clear Floor", false, function(s) end)
CreateButton(SecImpelFarm, "Start Impel Down TP", function() print("Started!") end)

-- Initial ersten Tab aktivieren
Categories[1].Button.BackgroundColor3 = DiamondTheme.SectionBG
Categories[1].Button.TextColor3 = DiamondTheme.DiamondBlue
Categories[1].SubContainer.Visible = true
if #Categories[1].SubTabs > 0 then Categories[1].SubTabs[1].Activate() end
