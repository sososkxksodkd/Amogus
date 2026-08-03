--// ============================================================================
--// RYU HUB - PERFECT MINIMALIST UI (100% VANILLA & SAFE)
--// ============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--// CLEANUP
for _, v in ipairs(PlayerGui:GetChildren()) do
    if v.Name == "RyuHubPerfect" then
        v:Destroy()
    end
end

--// MINIMALIST DARK THEME (Based on Reference)
local Theme = {
    Background    = Color3.fromRGB(15, 15, 18),        -- Main Body
    Sidebar       = Color3.fromRGB(10, 10, 12),        -- Darker Sidebar
    Card          = Color3.fromRGB(22, 22, 26),        -- Right side content cards
    
    TextWhite     = Color3.fromRGB(240, 240, 240),     -- Active Text
    TextGray      = Color3.fromRGB(130, 130, 140),     -- Inactive Text
    TextDark      = Color3.fromRGB(80, 80, 90),        -- Sub-labels
    
    Accent        = Color3.fromRGB(255, 255, 255),     -- Pure White Accent
    ToggleOff     = Color3.fromRGB(35, 35, 40),
    Border        = Color3.fromRGB(30, 30, 35)
}

local screenY = Camera.ViewportSize.Y
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local baseScale = isMobile and math.clamp(screenY / 700, 0.5, 0.8) or 1.0

--// MAIN GUI
local RyuGui = Instance.new("ScreenGui")
RyuGui.Name = "RyuHubPerfect"
RyuGui.ResetOnSpawn = false
RyuGui.IgnoreGuiInset = true
RyuGui.Parent = PlayerGui

local ScaleMod = Instance.new("UIScale", RyuGui)
ScaleMod.Scale = baseScale

--// MAIN FRAME
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 750, 0, 480)
MainFrame.Position = UDim2.new(0.5, -375, 0.5, -240)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.Active = true
MainFrame.Parent = RyuGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = Theme.Border
MainStroke.Thickness = 1

--// TOPBAR
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Size = UDim2.new(1, 0, 0, 50)
Topbar.BackgroundTransparency = 1

local Title = Instance.new("TextLabel", Topbar)
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 20, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "RYU HUB"
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 20
Title.TextColor3 = Theme.TextWhite
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Welle (Subtil & Modern)
local TitleGrad = Instance.new("UIGradient", Title)
TitleGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.TextGray),
    ColorSequenceKeypoint.new(0.5, Theme.TextWhite),
    ColorSequenceKeypoint.new(1, Theme.TextGray)
})
TitleGrad.Rotation = 45
TitleGrad.Offset = Vector2.new(-1, 0)
TweenService:Create(TitleGrad, TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Offset = Vector2.new(1, 0)}):Play()

local SubTitle = Instance.new("TextLabel", Topbar)
SubTitle.Size = UDim2.new(0, 200, 0, 15)
SubTitle.Position = UDim2.new(0, 20, 0, 32)
SubTitle.BackgroundTransparency = 1
SubTitle.Text = "Diamond Edition"
SubTitle.Font = Enum.Font.Gotham
SubTitle.TextSize = 11
SubTitle.TextColor3 = Theme.TextGray
SubTitle.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -40, 0.5, -15)
CloseBtn.BackgroundColor3 = Theme.Card
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextGray
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", CloseBtn).Color = Theme.Border

CloseBtn.MouseEnter:Connect(function() TweenService:Create(CloseBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextWhite}):Play() end)
CloseBtn.MouseLeave:Connect(function() TweenService:Create(CloseBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextGray}):Play() end)
CloseBtn.Activated:Connect(function()
    TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
    task.wait(0.25); MainFrame.Visible = false
end)

--// CUSTOM TOGGLE BUTTON (No Emojis, Custom Sword-Diamond ASCII)
local ToggleBtn = Instance.new("TextButton", RyuGui)
ToggleBtn.Size = UDim2.new(0, 55, 0, 45)
ToggleBtn.Position = UDim2.new(0, 20, 0.15, 0)
ToggleBtn.BackgroundColor3 = Theme.Sidebar
ToggleBtn.Text = "♦|══>"
ToggleBtn.TextColor3 = Theme.TextWhite
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 14
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", ToggleBtn).Color = Theme.Border

ToggleBtn.Activated:Connect(function()
    if MainFrame.Visible then
        TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
        task.wait(0.25); MainFrame.Visible = false
    else
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.35, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(0, 750, 0, 480), BackgroundTransparency = 0}):Play()
    end
end)

--// DRAG SYSTEM
local function ApplyDrag(frame, handle)
    local dragToggle, dragStart, startPos
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragToggle = true; dragStart = input.Position; startPos = frame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragToggle and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + (delta.X/ScaleMod.Scale), startPos.Y.Scale, startPos.Y.Offset + (delta.Y/ScaleMod.Scale))
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragToggle = false end
    end)
end
ApplyDrag(MainFrame, Topbar)
ApplyDrag(ToggleBtn, ToggleBtn)

--// LAYOUT BODY
local Body = Instance.new("Frame", MainFrame)
Body.Size = UDim2.new(1, 0, 1, -60)
Body.Position = UDim2.new(0, 0, 0, 60)
Body.BackgroundTransparency = 1

--// LEFT SIDEBAR (DARK)
local Sidebar = Instance.new("ScrollingFrame", Body)
Sidebar.Size = UDim2.new(0, 180, 1, 0)
Sidebar.BackgroundColor3 = Theme.Sidebar
Sidebar.BorderSizePixel = 0
Sidebar.ScrollBarThickness = 0

local SidebarLine = Instance.new("Frame", Sidebar)
SidebarLine.Size = UDim2.new(0, 1, 1, 0)
SidebarLine.Position = UDim2.new(1, -1, 0, 0)
SidebarLine.BackgroundColor3 = Theme.Border
SidebarLine.BorderSizePixel = 0

local SideList = Instance.new("UIListLayout", Sidebar)
SideList.Padding = UDim.new(0, 2)
SideList.SortOrder = Enum.SortOrder.LayoutOrder

--// RIGHT CONTENT AREA
local ContentArea = Instance.new("Frame", Body)
ContentArea.Size = UDim2.new(1, -190, 1, -10)
ContentArea.Position = UDim2.new(0, 190, 0, 0)
ContentArea.BackgroundTransparency = 1

--// PERFECT ACCORDION TAB SYSTEM
local Categories = {}

local function CreateCategory(name)
    local cat = {IsOpen = false, SubBtns = {}}
    
    local Wrapper = Instance.new("Frame", Sidebar)
    Wrapper.Size = UDim2.new(1, 0, 0, 45)
    Wrapper.BackgroundTransparency = 1
    Wrapper.ClipsDescendants = true
    
    local MainBtn = Instance.new("TextButton", Wrapper)
    MainBtn.Size = UDim2.new(1, 0, 0, 45)
    MainBtn.BackgroundTransparency = 1
    MainBtn.Text = "    " .. string.upper(name)
    MainBtn.TextColor3 = Theme.TextGray
    MainBtn.Font = Enum.Font.GothamBold
    MainBtn.TextSize = 12
    MainBtn.TextXAlignment = Enum.TextXAlignment.Left
    
    local Chevron = Instance.new("TextLabel", MainBtn)
    Chevron.Size = UDim2.new(0, 20, 1, 0)
    Chevron.Position = UDim2.new(1, -30, 0, 0)
    Chevron.BackgroundTransparency = 1
    Chevron.Text = "v"
    Chevron.TextColor3 = Theme.TextDark
    Chevron.Font = Enum.Font.GothamBold
    Chevron.TextSize = 12
    
    local SubBox = Instance.new("Frame", Wrapper)
    SubBox.Size = UDim2.new(1, 0, 1, -45)
    SubBox.Position = UDim2.new(0, 0, 0, 45)
    SubBox.BackgroundTransparency = 1
    
    local SubList = Instance.new("UIListLayout", SubBox)
    SubList.Padding = UDim.new(0, 0)
    
    cat.Wrapper = Wrapper
    cat.MainBtn = MainBtn
    cat.SubBox = SubBox
    cat.SubList = SubList
    
    MainBtn.Activated:Connect(function()
        local wasOpen = cat.IsOpen
        
        -- Alle schließen
        for _, c in pairs(Categories) do
            c.IsOpen = false
            TweenService:Create(c.MainBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextGray}):Play()
            TweenService:Create(c.MainBtn:FindFirstChild("TextLabel"), TweenInfo.new(0.2), {Rotation = 0}):Play()
            TweenService:Create(c.Wrapper, TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 45)}):Play()
        end
        
        -- Wenn es vorher zu war, öffnen
        if not wasOpen then
            cat.IsOpen = true
            TweenService:Create(MainBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextWhite}):Play()
            TweenService:Create(Chevron, TweenInfo.new(0.2), {Rotation = 180}):Play()
            
            local targetY = 45 + SubList.AbsoluteContentSize.Y
            TweenService:Create(Wrapper, TweenInfo.new(0.35, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, targetY)}):Play()
        end
    end)
    
    SubList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if cat.IsOpen then
            Wrapper.Size = UDim2.new(1, 0, 0, 45 + SubList.AbsoluteContentSize.Y)
        end
        Sidebar.CanvasSize = UDim2.new(0, 0, 0, SideList.AbsoluteContentSize.Y + 20)
    end)
    
    table.insert(Categories, cat)
    
    function cat:AddSub(subName)
        local subData = {}
        
        local SBtn = Instance.new("TextButton", SubBox)
        SBtn.Size = UDim2.new(1, 0, 0, 35)
        SBtn.BackgroundTransparency = 1
        SBtn.Text = "        " .. subName
        SBtn.TextColor3 = Theme.TextDark
        SBtn.Font = Enum.Font.GothamMedium
        SBtn.TextSize = 12
        SBtn.TextXAlignment = Enum.TextXAlignment.Left
        
        local Indicator = Instance.new("Frame", SBtn)
        Indicator.Size = UDim2.new(0, 2, 0.4, 0)
        Indicator.Position = UDim2.new(0, 0, 0.3, 0)
        Indicator.BackgroundColor3 = Theme.Accent
        Indicator.BackgroundTransparency = 1
        Instance.new("UICorner", Indicator).CornerRadius = UDim.new(1, 0)
        
        local Page = Instance.new("ScrollingFrame", ContentArea)
        Page.Size = UDim2.new(1, 0, 1, 0)
        Page.BackgroundTransparency = 1
        Page.ScrollBarThickness = 2
        Page.ScrollBarImageColor3 = Theme.Border
        Page.Visible = false
        
        local pList = Instance.new("UIListLayout", Page)
        pList.Padding = UDim.new(0, 8)
        pList.HorizontalAlignment = Enum.HorizontalAlignment.Center
        
        pList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            Page.CanvasSize = UDim2.new(0, 0, 0, pList.AbsoluteContentSize.Y + 20)
        end)
        
        local function activate()
            -- Alles resetten
            for _, c in pairs(Categories) do
                for _, btn in pairs(c.SubBox:GetChildren()) do
                    if btn:IsA("TextButton") then
                        TweenService:Create(btn, TweenInfo.new(0.2), {TextColor3 = Theme.TextDark}):Play()
                        TweenService:Create(btn:FindFirstChild("Frame"), TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
                    end
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            
            -- Aktivieren
            TweenService:Create(SBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextWhite}):Play()
            TweenService:Create(Indicator, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
            Page.Visible = true
            Page.CanvasPosition = Vector2.new(0,0)
        end
        
        SBtn.Activated:Connect(activate)
        subData.Activate = activate
        table.insert(cat.SubBtns, subData)
        
        return Page
    end
    
    return cat
end

--// UI ELEMENTS BUILDER
local function CreateSectionCard(page)
    local card = Instance.new("Frame", page)
    card.Size = UDim2.new(0.98, 0, 0, 40)
    card.BackgroundColor3 = Theme.Card
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)
    
    local cStr = Instance.new("UIStroke", card)
    cStr.Color = Theme.Border
    cStr.Thickness = 1
    
    local lay = Instance.new("UIListLayout", card)
    lay.Padding = UDim.new(0, 0)
    lay.SortOrder = Enum.SortOrder.LayoutOrder
    
    lay:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        card.Size = UDim2.new(0.98, 0, 0, lay.AbsoluteContentSize.Y)
    end)
    return card
end

local function AddToggle(card, title, subText, default, callback)
    local f = Instance.new("Frame", card)
    f.Size = UDim2.new(1, 0, 0, 55)
    f.BackgroundTransparency = 1
    
    local t = Instance.new("TextLabel", f)
    t.Size = UDim2.new(0.7, 0, 0, 20)
    t.Position = UDim2.new(0, 15, 0, 10)
    t.BackgroundTransparency = 1
    t.Text = title
    t.TextColor3 = Theme.TextWhite
    t.Font = Enum.Font.GothamMedium
    t.TextSize = 13
    t.TextXAlignment = Enum.TextXAlignment.Left
    
    local st = Instance.new("TextLabel", f)
    st.Size = UDim2.new(0.7, 0, 0, 15)
    st.Position = UDim2.new(0, 15, 0, 30)
    st.BackgroundTransparency = 1
    st.Text = "Use: " .. subText
    st.TextColor3 = Theme.TextDark
    st.Font = Enum.Font.Gotham
    st.TextSize = 11
    st.TextXAlignment = Enum.TextXAlignment.Left
    
    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(0, 44, 0, 24)
    btn.Position = UDim2.new(1, -60, 0.5, -12)
    btn.BackgroundColor3 = default and Theme.Accent or Theme.ToggleOff
    btn.Text = ""
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    
    local circ = Instance.new("Frame", btn)
    circ.Size = UDim2.new(0, 20, 0, 20)
    circ.Position = default and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
    circ.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", circ).CornerRadius = UDim.new(1, 0)
    
    local isOn = default
    btn.Activated:Connect(function()
        isOn = not isOn
        TweenService:Create(btn, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {BackgroundColor3 = isOn and Theme.Accent or Theme.ToggleOff}):Play()
        TweenService:Create(circ, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Position = isOn and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)}):Play()
        if callback then callback(isOn) end
    end)
    
    -- Trennlinie, außer es ist das letzte Element
    local line = Instance.new("Frame", f)
    line.Size = UDim2.new(1, -30, 0, 1)
    line.Position = UDim2.new(0, 15, 1, -1)
    line.BackgroundColor3 = Theme.Border
    line.BorderSizePixel = 0
end

--// INIT INTERFACE
local catCombat = CreateCategory("COMBAT")
local pageSkills = catCombat:AddSub("Skills")
local pageTiming = catCombat:AddSub("Timing")
local pageDef    = catCombat:AddSub("Defense")

local cardSkills = CreateSectionCard(pageSkills)
AddToggle(cardSkills, "Grimmjow", "Panther King Claw", false, function() end)
AddToggle(cardSkills, "Joseph Joestar", "Jeep", false, function() end)
AddToggle(cardSkills, "Gaara", "Sand Tsunami / Great Tsunami", false, function() end)
AddToggle(cardSkills, "Josuke", "Ball / Rock", true, function() end)
AddToggle(cardSkills, "Kazuma", "Kazuma Arrow", false, function() end)

local catPlayer = CreateCategory("PLAYER")
local pageChar = catPlayer:AddSub("Character")
local cardChar = CreateSectionCard(pageChar)
AddToggle(cardChar, "Auto Respawn", "Instantly respawn on death", false, function() end)

local catVis = CreateCategory("VISUALS")
local pageEsp = catVis:AddSub("ESP")

-- Setup First Tab
task.delay(0.1, function()
    catCombat.IsOpen = true
    catCombat.MainBtn.TextColor3 = Theme.TextWhite
    catCombat.MainBtn:FindFirstChild("TextLabel").Rotation = 180
    catCombat.Wrapper.Size = UDim2.new(1, 0, 0, 45 + catCombat.SubList.AbsoluteContentSize.Y)
    if #catCombat.SubBtns > 0 then catCombat.SubBtns[1].Activate() end
end)
