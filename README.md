--// ============================================================================
--// RYU HUB - PURE VANILLA REBUILD (ZERO EXPLOIT TRACES)
--// ============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- WIR NUTZEN NUR DAS NORMALE PLAYERGUI (Kein CoreGui, kein gethui!)
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

for _, v in ipairs(PlayerGui:GetChildren()) do
    if v.Name == "RyuHubSafe" then
        v:Destroy()
    end
end

--// THEME SETTINGS (Modern Glass & Cyan)
local Theme = {
    Background    = Color3.fromRGB(5, 7, 12),
    Sidebar       = Color3.fromRGB(10, 14, 22),
    Section       = Color3.fromRGB(15, 20, 30),
    Accent        = Color3.fromRGB(0, 229, 255),
    AccentGlow    = Color3.fromRGB(120, 245, 255),
    TextWhite     = Color3.fromRGB(255, 255, 255),
    TextGray      = Color3.fromRGB(150, 170, 190),
    Border        = Color3.fromRGB(40, 60, 80),
    ToggleOff     = Color3.fromRGB(25, 30, 45),
    RedBtn        = Color3.fromRGB(255, 59, 48),
    
    GlassTransp   = 0.35,
    SecTransp     = 0.4
}

--// SCALE CALCULATION
local screenY = Camera.ViewportSize.Y
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local baseScale = isMobile and math.clamp(screenY / 750, 0.45, 0.75) or 1.0

--// MAIN GUI INSTANCE
local RyuGui = Instance.new("ScreenGui")
RyuGui.Name = "RyuHubSafe"
RyuGui.ResetOnSpawn = false
RyuGui.IgnoreGuiInset = true
RyuGui.Parent = PlayerGui

local ScaleMod = Instance.new("UIScale", RyuGui)
ScaleMod.Scale = baseScale

--// MAIN FRAME
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 660, 0, 420)
MainFrame.Position = UDim2.new(0.5, -330, 0.5, -210)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BackgroundTransparency = Theme.GlassTransp
MainFrame.Active = true
MainFrame.Parent = RyuGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainBorder = Instance.new("UIStroke", MainFrame)
MainBorder.Color = Theme.Accent
MainBorder.Thickness = 1.5
MainBorder.Transparency = 0.3

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

-- Einfacher Farbverlauf
local TitleGrad = Instance.new("UIGradient", Title)
TitleGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Accent),
    ColorSequenceKeypoint.new(0.5, Theme.TextWhite),
    ColorSequenceKeypoint.new(1, Theme.Accent)
})
TitleGrad.Rotation = 25
TitleGrad.Offset = Vector2.new(-1, 0)

TweenService:Create(TitleGrad, TweenInfo.new(2.5, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1), {Offset = Vector2.new(1, 0)}):Play()

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

--// FLOATING TOGGLE
local ToggleBtn = Instance.new("TextButton", RyuGui)
ToggleBtn.Size = UDim2.new(0, 48, 0, 48)
ToggleBtn.Position = UDim2.new(0, 20, 0.15, 0)
ToggleBtn.BackgroundColor3 = Theme.Background
ToggleBtn.BackgroundTransparency = 0.2
ToggleBtn.Text = "🌀"
ToggleBtn.TextSize = 22
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
        TweenService:Create(MainFrame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 660, 0, 420), BackgroundTransparency = Theme.GlassTransp}):Play()
    end
end)

--// DRAG SYSTEM (SAFE WAY)
local function CreateDrag(frame, handle)
    local dragToggle, dragStart, startPos
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragToggle = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragToggle and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragToggle = false
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
Sidebar.Size = UDim2.new(0, 160, 1, 0)
Sidebar.BackgroundTransparency = 1
Sidebar.ScrollBarThickness = 0
local SideList = Instance.new("UIListLayout", Sidebar)
SideList.Padding = UDim.new(0, 8)

SideList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, SideList.AbsoluteContentSize.Y + 10)
end)

local ContentArea = Instance.new("Frame", Body)
ContentArea.Size = UDim2.new(1, -180, 1, 0)
ContentArea.Position = UDim2.new(0, 180, 0, 0)
ContentArea.BackgroundTransparency = 1

--// TAB SYSTEM
local Tabs = {}

local function CreateMainTab(name, icon)
    local tabData = {IsOpen = false, SubTabs = {}}
    
    local Wrapper = Instance.new("Frame", Sidebar)
    Wrapper.Size = UDim2.new(1, 0, 0, 40)
    Wrapper.BackgroundTransparency = 1
    Wrapper.ClipsDescendants = true
    
    local Btn = Instance.new("TextButton", Wrapper)
    Btn.Size = UDim2.new(1, 0, 0, 40)
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
    
    local SubBox = Instance.new("Frame", Wrapper)
    SubBox.Size = UDim2.new(1, 0, 1, -40)
    SubBox.Position = UDim2.new(0, 0, 0, 40)
    SubBox.BackgroundTransparency = 1
    local SubList = Instance.new("UIListLayout", SubBox)
    SubList.Padding = UDim.new(0, 4)
    SubList.HorizontalAlignment = Enum.HorizontalAlignment.Right
    Instance.new("UIPadding", SubBox).PaddingTop = UDim.new(0, 4)
    
    tabData.Wrapper = Wrapper
    tabData.Btn = Btn
    tabData.SubBox = SubBox
    
    Btn.Activated:Connect(function()
        local wasOpen = tabData.IsOpen
        for _, t in pairs(Tabs) do
            t.IsOpen = false
            TweenService:Create(t.Btn, TweenInfo.new(0.3), {TextColor3 = Theme.TextGray}):Play()
            TweenService:Create(t.Btn:FindFirstChildOfClass("UIStroke"), TweenInfo.new(0.3), {Color = Theme.Border}):Play()
            TweenService:Create(t.Wrapper, TweenInfo.new(0.35, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 40)}):Play()
        end
        if not wasOpen then
            tabData.IsOpen = true
            TweenService:Create(Btn, TweenInfo.new(0.3), {TextColor3 = Theme.Accent}):Play()
            TweenService:Create(BtnBorder, TweenInfo.new(0.3), {Color = Theme.Accent}):Play()
            local tHeight = 40 + SubList.AbsoluteContentSize.Y + 4
            TweenService:Create(Wrapper, TweenInfo.new(0.35, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, tHeight)}):Play()
            if #tabData.SubTabs > 0 then tabData.SubTabs[1]() end
        end
    end)
    
    SubList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if tabData.IsOpen then
            Wrapper.Size = UDim2.new(1, 0, 0, 40 + SubList.AbsoluteContentSize.Y + 4)
        end
    end)
    
    table.insert(Tabs, tabData)
    
    function tabData:AddSub(subName)
        local sBtn = Instance.new("TextButton", SubBox)
        sBtn.Size = UDim2.new(0.9, 0, 0, 30)
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
        pList.Padding = UDim.new(0, 10)
        pList.HorizontalAlignment = Enum.HorizontalAlignment.Center
        
        pList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            Page.CanvasSize = UDim2.new(0, 0, 0, pList.AbsoluteContentSize.Y + 20)
        end)
        
        local function activate()
            for _, t in pairs(Tabs) do
                for _, b in pairs(t.SubBox:GetChildren()) do
                    if b:IsA("TextButton") then TweenService:Create(b, TweenInfo.new(0.2), {TextColor3 = Theme.TextGray}):Play() end
                end
            end
            for _, p in pairs(ContentArea:GetChildren()) do
                if p:IsA("ScrollingFrame") then p.Visible = false end
            end
            TweenService:Create(sBtn, TweenInfo.new(0.2), {TextColor3 = Theme.TextWhite}):Play()
            Page.Visible = true
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
    sList.Padding = UDim.new(0, 8)
    sList.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", sec).PaddingTop = UDim.new(0, 12)
    Instance.new("UIPadding", sec).PaddingBottom = UDim.new(0, 12)
    
    local t = Instance.new("TextLabel", sec)
    t.Size = UDim2.new(0.92, 0, 0, 18)
    t.BackgroundTransparency = 1
    t.Text = titleText
    t.TextColor3 = Theme.AccentGlow
    t.Font = Enum.Font.GothamBold
    t.TextSize = 13
    t.TextXAlignment = Enum.TextXAlignment.Left
    
    sList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        sec.Size = UDim2.new(0.98, 0, 0, sList.AbsoluteContentSize.Y + 24)
    end)
    return sec
end

local function AddToggle(sec, text, state, cb)
    local f = Instance.new("Frame", sec)
    f.Size = UDim2.new(0.92, 0, 0, 34)
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
    btn.Size = UDim2.new(0, 42, 0, 22)
    btn.Position = UDim2.new(1, -42, 0.5, -11)
    btn.BackgroundColor3 = state and Theme.Accent or Theme.ToggleOff
    btn.Text = ""
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    
    local ind = Instance.new("Frame", btn)
    ind.Size = UDim2.new(0, 18, 0, 18)
    ind.Position = state and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)
    ind.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", ind).CornerRadius = UDim.new(1, 0)
    
    local active = state
    btn.Activated:Connect(function()
        active = not active
        TweenService:Create(btn, TweenInfo.new(0.25), {BackgroundColor3 = active and Theme.Accent or Theme.ToggleOff}):Play()
        TweenService:Create(ind, TweenInfo.new(0.25), {Position = active and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)}):Play()
        if cb then cb(active) end
    end)
end

local function AddSlider(sec, text, min, max, default, cb)
    local f = Instance.new("Frame", sec)
    f.Size = UDim2.new(0.92, 0, 0, 48)
    f.BackgroundTransparency = 1
    
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(0.7, 0, 0, 18)
    l.BackgroundTransparency = 1
    l.Text = text
    l.TextColor3 = Theme.TextGray
    l.Font = Enum.Font.GothamMedium
    l.TextSize = 12
    l.TextXAlignment = Enum.TextXAlignment.Left
    
    local vL = Instance.new("TextLabel", f)
    vL.Size = UDim2.new(0.3, 0, 0, 18)
    vL.Position = UDim2.new(0.7, 0, 0, 0)
    vL.BackgroundTransparency = 1
    vL.Text = tostring(default)
    vL.TextColor3 = Theme.AccentGlow
    vL.Font = Enum.Font.GothamBold
    vL.TextSize = 12
    vL.TextXAlignment = Enum.TextXAlignment.Right
    
    local bg = Instance.new("Frame", f)
    bg.Size = UDim2.new(1, 0, 0, 6)
    bg.Position = UDim2.new(0, 0, 0, 28)
    bg.BackgroundColor3 = Theme.ToggleOff
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)
    
    local fill = Instance.new("Frame", bg)
    local p = (default - min) / (max - min)
    fill.Size = UDim2.new(p, 0, 1, 0)
    fill.BackgroundColor3 = Theme.Accent
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
    
    local drag = false
    local function set(inpt)
        local pct = math.clamp((inpt.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
        local val = math.floor(min + (max - min) * pct)
        vL.Text = tostring(val)
        TweenService:Create(fill, TweenInfo.new(0.1), {Size = UDim2.new(pct, 0, 1, 0)}):Play()
        if cb then cb(val) end
    end
    
    bg.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = true; set(i) end end)
    UserInputService.InputChanged:Connect(function(i) if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then set(i) end end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = false end end)
end

--// INITIALIZATION
local tMain = CreateMainTab("Main", "🏠")
local pGen = tMain:AddSub("General Setup")
local sGen = AddCategory(pGen, "Frontend Options")
AddToggle(sGen, "Demo Toggle", false, function() end)

local tSet = CreateMainTab("Settings", "⚙️")
local pUI = tSet:AddSub("UI Scale")
local sUI = AddCategory(pUI, "Custom Adjuster")
AddSlider(sUI, "GUI Scale", 40, 120, math.floor(baseScale * 100), function(v)
    ScaleMod.Scale = v / 100
end)

-- First Tab Open
task.delay(0.1, function()
    Tabs[1].IsOpen = true
    Tabs[1].Btn.TextColor3 = Theme.Accent
    Tabs[1].Btn:FindFirstChildOfClass("UIStroke").Color = Theme.Accent
    Tabs[1].Wrapper.Size = UDim2.new(1, 0, 0, 40 + Tabs[1].SubBox:FindFirstChildOfClass("UIListLayout").AbsoluteContentSize.Y + 4)
    if #Tabs[1].SubTabs > 0 then Tabs[1].SubTabs[1]() end
end)
