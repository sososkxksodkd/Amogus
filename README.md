--// ============================================================================
--// RYU HUB - DIAMOND KATANA EDITION (ULTRA MODERN GLASS & AUTO-SCALE)
--// ============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// 100% SAFE CLEANUP (NUR PLAYERGUI)
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

for _, v in ipairs(PlayerGui:GetChildren()) do
    if v.Name == "RyuHubModernGlass" then
        v:Destroy()
    end
end

--// ULTRA MODERN GLASS THEME
local Theme = {
    Background    = Color3.fromRGB(8, 11, 18),         -- Deep Obsidian Blue
    Sidebar       = Color3.fromRGB(13, 18, 28),        -- Darker Contrast
    Section       = Color3.fromRGB(18, 24, 38),        -- Card Color
    Accent        = Color3.fromRGB(0, 229, 255),       -- Neon Cyan (Diamond)
    AccentGlow    = Color3.fromRGB(140, 245, 255),     -- Soft Diamond Light
    TextWhite     = Color3.fromRGB(245, 250, 255),
    TextGray      = Color3.fromRGB(140, 160, 185),
    Border        = Color3.fromRGB(35, 50, 75),
    ToggleOff     = Color3.fromRGB(28, 35, 50),
    RedBtn        = Color3.fromRGB(255, 60, 60),
    
    GlassTransp   = 0.25, -- Milchglas-Effekt
    SecTransp     = 0.4
}

--// SMART AUTO-SCALING (MOBILE / PC)
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local screenY = Camera.ViewportSize.Y
-- Automatische Größenberechnung basierend auf dem Bildschirm
local baseScale = isMobile and math.clamp(screenY / 800, 0.5, 0.85) or 1.0

--// MAIN GUI INSTANCE
local RyuGui = Instance.new("ScreenGui")
RyuGui.Name = "RyuHubModernGlass"
RyuGui.ResetOnSpawn = false
RyuGui.IgnoreGuiInset = true
RyuGui.Parent = PlayerGui

local ScaleMod = Instance.new("UIScale", RyuGui)
ScaleMod.Scale = baseScale

--// MAIN FRAME (GLASS CONTAINER)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 680, 0, 440)
MainFrame.Position = UDim2.new(0.5, -340, 0.5, -220)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BackgroundTransparency = Theme.GlassTransp
MainFrame.Active = true
MainFrame.Parent = RyuGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainBorder = Instance.new("UIStroke", MainFrame)
MainBorder.Thickness = 1.5
MainBorder.Color = Theme.TextWhite
MainBorder.Transparency = 0.5

local MainGradient = Instance.new("UIGradient", MainBorder)
MainGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Accent),
    ColorSequenceKeypoint.new(0.5, Theme.TextWhite),
    ColorSequenceKeypoint.new(1, Theme.Accent)
})
MainGradient.Rotation = 45

-- Animierter Rand
task.spawn(function()
    while MainGradient.Parent do
        MainGradient.Rotation = (MainGradient.Rotation + 1) % 360
        task.wait(0.02)
    end
end)

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
Title.TextSize = 22
Title.TextColor3 = Theme.TextWhite
Title.TextXAlignment = Enum.TextXAlignment.Left

local TitleGrad = Instance.new("UIGradient", Title)
TitleGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Accent),
    ColorSequenceKeypoint.new(0.5, Theme.TextWhite),
    ColorSequenceKeypoint.new(1, Theme.Accent)
})
TitleGrad.Rotation = 0
TitleGrad.Offset = Vector2.new(-1, 0)

TweenService:Create(TitleGrad, TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Offset = Vector2.new(1, 0)}):Play()

--// CLOSE BUTTON
local CloseBtn = Instance.new("TextButton", Topbar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28)
CloseBtn.Position = UDim2.new(1, -40, 0.5, -14)
CloseBtn.BackgroundColor3 = Theme.RedBtn
CloseBtn.BackgroundTransparency = 0.15
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextWhite
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

CloseBtn.Activated:Connect(function()
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
    task.wait(0.3)
    MainFrame.Visible = false
end)

--// DYNAMIC RESIZE GRIP (Unten Rechts ziehen zum Vergrößern/Verkleinern)
local ResizeGrip = Instance.new("TextButton", MainFrame)
ResizeGrip.Size = UDim2.new(0, 25, 0, 25)
ResizeGrip.Position = UDim2.new(1, -25, 1, -25)
ResizeGrip.BackgroundTransparency = 1
ResizeGrip.Text = "◢"
ResizeGrip.TextColor3 = Theme.Accent
ResizeGrip.TextSize = 18
ResizeGrip.Font = Enum.Font.GothamBold

local isResizing = false
local dragStartResize, startSize
ResizeGrip.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isResizing = true
        dragStartResize = input.Position
        startSize = MainFrame.AbsoluteSize
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if isResizing and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStartResize
        -- Begrenze minimale und maximale GUI-Größe
        local newX = math.clamp(startSize.X + (delta.X / ScaleMod.Scale), 500, 1200)
        local newY = math.clamp(startSize.Y + (delta.Y / ScaleMod.Scale), 350, 800)
        MainFrame.Size = UDim2.new(0, newX, 0, newY)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isResizing = false
    end
end)

--// FLOATING TOGGLE BUTTON (DIAMOND KATANA)
local ToggleBtn = Instance.new("TextButton", RyuGui)
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 20, 0.15, 0)
ToggleBtn.BackgroundColor3 = Theme.Background
ToggleBtn.BackgroundTransparency = 0.2
ToggleBtn.Text = "💎🗡️"
ToggleBtn.TextSize = 20
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local ToggleBorder = Instance.new("UIStroke", ToggleBtn)
ToggleBorder.Color = Theme.Accent
ToggleBorder.Thickness = 1.5

ToggleBtn.Activated:Connect(function()
    if MainFrame.Visible then
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0,0,0,0), BackgroundTransparency = 1}):Play()
        task.wait(0.3)
        MainFrame.Visible = false
    else
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 680, 0, 440), BackgroundTransparency = Theme.GlassTransp}):Play()
    end
end)

--// DRAG SYSTEM (MAIN FRAME & TOGGLE)
local function CreateDrag(frame, handle)
    local dragging, dragStart, startPos
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and not isResizing and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + (delta.X / ScaleMod.Scale), startPos.Y.Scale, startPos.Y.Offset + (delta.Y / ScaleMod.Scale))
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

CreateDrag(MainFrame, Topbar)
CreateDrag(ToggleBtn, ToggleBtn)

--// CONTENT LAYOUT
local Body = Instance.new("Frame", MainFrame)
Body.Size = UDim2.new(1, -20, 1, -60)
Body.Position = UDim2.new(0, 10, 0, 50)
Body.BackgroundTransparency = 1

local Sidebar = Instance.new("ScrollingFrame", Body)
Sidebar.Size = UDim2.new(0, 170, 1, 0)
Sidebar.BackgroundTransparency = 1
Sidebar.ScrollBarThickness = 0
local SideList = Instance.new("UIListLayout", Sidebar)
SideList.Padding = UDim.new(0, 8)

SideList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, SideList.AbsoluteContentSize.Y + 10)
end)

local ContentArea = Instance.new("Frame", Body)
ContentArea.Size = UDim2.new(1, -190, 1, 0)
ContentArea.Position = UDim2.new(0, 190, 0, 0)
ContentArea.BackgroundTransparency = 1

--// FLAWLESS ACCORDION TAB SYSTEM
local Tabs = {}

local function CreateMainTab(name, icon)
    local tabData = {IsOpen = false, SubTabs = {}}
    
    local Wrapper = Instance.new("Frame", Sidebar)
    Wrapper.Size = UDim2.new(1, 0, 0, 42)
    Wrapper.BackgroundTransparency = 1
    Wrapper.ClipsDescendants = true
    
    local Btn = Instance.new("TextButton", Wrapper)
    Btn.Size = UDim2.new(1, 0, 0, 42)
    Btn.BackgroundColor3 = Theme.Sidebar
    Btn.BackgroundTransparency = Theme.SecTransp
    Btn.Text = "   " .. icon .. "  " .. name
    Btn.TextColor3 = Theme.TextGray
    Btn.Font = Enum.Font.GothamBold
    Btn.TextSize = 13
    Btn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 8)
    
    local BtnBorder = Instance.new("UIStroke", Btn)
    BtnBorder.Color = Theme.Border
    
    -- SubBox: Y-Size ist flexibel, aber wir setzen sie auf eine sichere Höhe, damit die Layouts berechnet werden können
    local SubBox = Instance.new("Frame", Wrapper)
    SubBox.Size = UDim2.new(1, 0, 0, 1000) 
    SubBox.Position = UDim2.new(0, 0, 0, 44)
    SubBox.BackgroundTransparency = 1
    
    local SubList = Instance.new("UIListLayout", SubBox)
    SubList.Padding = UDim.new(0, 4)
    SubList.HorizontalAlignment = Enum.HorizontalAlignment.Right
    
    tabData.Wrapper = Wrapper
    tabData.Btn = Btn
    tabData.SubBox = SubBox
    tabData.SubList = SubList
    
    Btn.Activated:Connect(function()
        local wasOpen = tabData.IsOpen
        for _, t in pairs(Tabs) do
            t.IsOpen = false
            TweenService:Create(t.Btn, TweenInfo.new(0.3), {TextColor3 = Theme.TextGray}):Play()
            TweenService:Create(t.Btn:FindFirstChildOfClass("UIStroke"), TweenInfo.new(0.3), {Color = Theme.Border}):Play()
            TweenService:Create(t.Wrapper, TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 42)}):Play()
        end
        if not wasOpen then
            tabData.IsOpen = true
            TweenService:Create(Btn, TweenInfo.new(0.3), {TextColor3 = Theme.Accent}):Play()
            TweenService:Create(BtnBorder, TweenInfo.new(0.3), {Color = Theme.Accent}):Play()
            
            local tHeight = 42 + SubList.AbsoluteContentSize.Y + 8
            TweenService:Create(Wrapper, TweenInfo.new(0.4, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, tHeight)}):Play()
            
            if #tabData.SubTabs > 0 then tabData.SubTabs[1]() end
        end
    end)
    
    SubList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if tabData.IsOpen then
            Wrapper.Size = UDim2.new(1, 0, 0, 42 + SubList.AbsoluteContentSize.Y + 8)
        end
    end)
    
    table.insert(Tabs, tabData)
    
    function tabData:AddSub(subName)
        local sBtn = Instance.new("TextButton", SubBox)
        sBtn.Size = UDim2.new(0.85, 0, 0, 30)
        sBtn.BackgroundTransparency = 1
        sBtn.Text = "•  " .. subName
        sBtn.TextColor3 = Theme.TextGray
        sBtn.Font = Enum.Font.GothamMedium
        sBtn.TextSize = 12
        sBtn.TextXAlignment = Enum.TextXAlignment.Left
        
        local Page = Instance.new("ScrollingFrame", ContentArea)
        Page.Size = UDim2.new(1, 0, 1, 0)
        Page.BackgroundTransparency = 1
        Page.ScrollBarThickness = 2
        Page.ScrollBarImageColor3 = Theme.Accent
        Page.Visible = false
        local pList = Instance.new("UIListLayout", Page)
        pList.Padding = UDim.new(0, 12)
        pList.HorizontalAlignment = Enum.HorizontalAlignment.Center
        
        pList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            Page.CanvasSize = UDim2.new(0, 0, 0, pList.AbsoluteContentSize.Y + 20)
        end)
        
        local function activate()
            for _, t in pairs(Tabs) do
                for _, b in pairs(t.SubBox:GetChildren()) do
                    if b:IsA("TextButton") then 
                        TweenService:Create(b, TweenInfo.new(0.2), {TextColor3 = Theme.TextGray}):Play() 
                    end
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            TweenService:Create(sBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextWhite}):Play()
            
            -- Schöner Page Fade-In
            Page.Visible = true
            Page.CanvasPosition = Vector2.new(0,0)
        end
        
        sBtn.Activated:Connect(activate)
        table.insert(tabData.SubTabs, activate)
        return Page
    end
    
    return tabData
end

--// UI ELEMENTS
local function AddCategory(page, titleText)
    local sec = Instance.new("Frame", page)
    sec.Size = UDim2.new(0.98, 0, 0, 40)
    sec.BackgroundColor3 = Theme.Section
    sec.BackgroundTransparency = Theme.SecTransp
    Instance.new("UICorner", sec).CornerRadius = UDim.new(0, 10)
    
    local secBor = Instance.new("UIStroke", sec)
    secBor.Color = Theme.Border
    secBor.Transparency = 0.5
    
    local sList = Instance.new("UIListLayout", sec)
    sList.Padding = UDim.new(0, 10)
    sList.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", sec).PaddingTop = UDim.new(0, 14)
    Instance.new("UIPadding", sec).PaddingBottom = UDim.new(0, 14)
    
    local t = Instance.new("TextLabel", sec)
    t.Size = UDim2.new(0.92, 0, 0, 20)
    t.BackgroundTransparency = 1
    t.Text = titleText
    t.TextColor3 = Theme.AccentGlow
    t.Font = Enum.Font.GothamBold
    t.TextSize = 14
    t.TextXAlignment = Enum.TextXAlignment.Left
    
    sList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        sec.Size = UDim2.new(0.98, 0, 0, sList.AbsoluteContentSize.Y + 28)
    end)
    return sec
end

local function AddToggle(sec, text, state, cb)
    local f = Instance.new("Frame", sec)
    f.Size = UDim2.new(0.92, 0, 0, 36)
    f.BackgroundTransparency = 1
    
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(0.7, 0, 1, 0)
    l.BackgroundTransparency = 1
    l.Text = text
    l.TextColor3 = Theme.TextWhite
    l.Font = Enum.Font.GothamMedium
    l.TextSize = 12
    l.TextXAlignment = Enum.TextXAlignment.Left
    
    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(0, 44, 0, 24)
    btn.Position = UDim2.new(1, -44, 0.5, -12)
    btn.BackgroundColor3 = state and Theme.Accent or Theme.ToggleOff
    btn.Text = ""
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    
    local ind = Instance.new("Frame", btn)
    ind.Size = UDim2.new(0, 20, 0, 20)
    ind.Position = state and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
    ind.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", ind).CornerRadius = UDim.new(1, 0)
    
    local active = state
    btn.Activated:Connect(function()
        active = not active
        TweenService:Create(btn, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {BackgroundColor3 = active and Theme.Accent or Theme.ToggleOff}):Play()
        TweenService:Create(ind, TweenInfo.new(0.25, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Position = active and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)}):Play()
        if cb then cb(active) end
    end)
end

local function AddButton(sec, text, cb)
    local btn = Instance.new("TextButton", sec)
    btn.Size = UDim2.new(0.92, 0, 0, 38)
    btn.BackgroundColor3 = Theme.Section
    btn.BackgroundTransparency = 0.2
    btn.Text = text
    btn.TextColor3 = Theme.TextWhite
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    
    local str = Instance.new("UIStroke", btn)
    str.Color = Theme.Border
    str.Thickness = 1
    
    btn.MouseEnter:Connect(function() TweenService:Create(str, TweenInfo.new(0.2), {Color = Theme.Accent}):Play() end)
    btn.MouseLeave:Connect(function() TweenService:Create(str, TweenInfo.new(0.2), {Color = Theme.Border}):Play() end)
    
    btn.Activated:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Theme.Accent, TextColor3 = Color3.new(0,0,0)}):Play()
        task.wait(0.1)
        TweenService:Create(btn, TweenInfo.new(0.3), {BackgroundColor3 = Theme.Section, TextColor3 = Theme.TextWhite}):Play()
        if cb then cb() end
    end)
end

--// INITIALIZATION (Demo Tabs)
local tMain = CreateMainTab("Main", "🏠")
local pGen = tMain:AddSub("General Setup")
local sGen = AddCategory(pGen, "Frontend Options")
AddToggle(sGen, "Demo Toggle", false, function() end)
AddButton(sGen, "Execute Function", function() end)

local tCombat = CreateMainTab("Combat", "⚔️")
local pFarm = tCombat:AddSub("Auto Farm")
local sFarm = AddCategory(pFarm, "Farming Settings")
AddToggle(sFarm, "Enable Auto Farm", false, function() end)

-- First Tab Open Initializer
task.delay(0.1, function()
    Tabs[1].IsOpen = true
    Tabs[1].Btn.TextColor3 = Theme.Accent
    Tabs[1].Btn:FindFirstChildOfClass("UIStroke").Color = Theme.Accent
    Tabs[1].Wrapper.Size = UDim2.new(1, 0, 0, 42 + Tabs[1].SubList.AbsoluteContentSize.Y + 8)
    if #Tabs[1].SubTabs > 0 then Tabs[1].SubTabs[1]() end
end)
