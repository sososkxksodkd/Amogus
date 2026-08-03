--// ============================================================================
--// RYU HUB - MODERN GLASS EDITION (SERVER-SAFE TP & RESPONSIVE UI)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

--// SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function()
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end
end)

for _, v in pairs(guiParent:GetChildren()) do
    if v.Name == "RyuHubModernGlass" then v:Destroy() end
end

--// SAFE ANTICHEAT HOOK (CLIENT SIDE)
pcall(function()
    if getrawmetatable and setreadonly and newcclosure and getnamecallmethod then
        local mt = getrawmetatable(game)
        local rawNamecall = mt.__namecall
        setreadonly(mt, false)
        
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if not checkcaller() and (method == "FireServer" or method == "InvokeServer") then
                local remoteName = tostring(self.Name):lower()
                if remoteName:find("ban") or remoteName:find("kick") or remoteName:find("detection") or remoteName:find("check") or remoteName:find("strike") then
                    return nil
                end
            end
            return rawNamecall(self, ...)
        end)
        setreadonly(mt, true)
    end
end)

--// ANTI-ANNOYING MESSAGE HIDER
task.spawn(function()
    local pg = LocalPlayer:WaitForChild("PlayerGui", 10)
    if pg then
        pg.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
                task.delay(0.01, function()
                    if descendant.Parent and descendant.Text then
                        local txt = descendant.Text:lower()
                        if txt:match("cd") or txt:match("cooldown") or txt:match("climb") or txt:match("error") or txt:match("strike") or txt:match("threshold") then
                            descendant.Visible = false
                            descendant:Destroy()
                        end
                    end
                end)
            end
        end)
    end
end)

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

--// RYU CONFIGURATION
local RyuConfig = {
    AutoFarm = false,
    FarmMode = "Single",
    AutoQuest = false,
    QuestInterval = 45, 
    
    TargetMob = "Bandit", 
    TargetNPC = "Daph",               
    TargetWeapon = "Combat",           
    
    TweenSpeed = 35, -- DEUTLICH SICHERER GEGEN SERVER BANS
    KillHeight = 5, 
    IslandSpeed = 35, -- SICHERE GESCHWINDIGKEIT
    TargetIsland = "Town of Beginnings"
}

--// SMART MOBILE SCALING
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local screenY = Camera.ViewportSize.Y
-- Wir nutzen eine größere BaseScale für Mobile, damit es nicht mehr "einfach nur klein" ist
local baseScale = isMobile and math.clamp(screenY / 500, 0.75, 1.2) or 1.0

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

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 1.5
MainStroke.Color = Theme.Accent
MainStroke.Transparency = 0.3

--// TOPBAR
local Topbar = Instance.new("Frame", MainFrame)
Topbar.Name = "Topbar"
Topbar.Size = UDim2.new(1, 0, 0, 50)
Topbar.BackgroundTransparency = 1

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
            if #tabObj.SubTabs > 0 then tabObj.SubTabs[1].Activate() end
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
--// GUI INITIALIZATION
--// ============================================================================

-- Mobility / Transport
local TabMobility = CreateMainTab("Mobility", "🗺️")
local SubTransport = TabMobility:CreateSubTab("Spider TP")
local SecIslandTP = CreateSection(SubTransport, "Spider Teleportation (Safe Mode)")

CreateSlider(SecIslandTP, "Travel Speed", 10, 40, RyuConfig.IslandSpeed, function(val)
    RyuConfig.IslandSpeed = val
end)

--// SERVER-SAFE SPIDER TELEPORT (SLOWED DOWN & REDUCED REMOTE SPAM)
CreateButton(SecIslandTP, "Start Safe TP", function()
    if _G.RyuIsTweening then return end
    _G.RyuIsTweening = true
    
    task.spawn(function()
        local success, err = pcall(function()
            local targetIslandName = RyuConfig.TargetIsland
            local island = Workspace:FindFirstChild("Islands") and Workspace.Islands:FindFirstChild(targetIslandName)
            if not island then return end
            
            local rawPos = island:IsA("Model") and island:GetPivot().Position or island.Position
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if not root or not hum then return end

            local targetPos = Vector3.new(rawPos.X, root.Position.Y, rawPos.Z)
            local floorOffset = (hum.HipHeight or 2) + (root.Size.Y / 2)
            
            local bp = Instance.new("BodyPosition")
            bp.Name = "RyuHover"
            bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.D = 500; bp.P = 50000; bp.Parent = root
            bp.Position = root.Position
            
            root.CFrame = root.CFrame + Vector3.new(0, 1, 0)
            
            local sprintEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("sprint")
            local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
            
            -- WICHTIG: Remotes nicht mehr 10x pro Sekunde spammen, sondern normaler feuern!
            local isFlyingActive = true
            task.spawn(function()
                if sprintEvent and type(sprintEvent.FireServer) == "function" then 
                    pcall(function() sprintEvent:FireServer("rbxassetid://15382065457") end) 
                end
                while isFlyingActive and _G.RyuIsTweening do
                    if footstepEvent and type(footstepEvent.FireServer) == "function" then 
                        pcall(function() footstepEvent:FireServer() end) 
                    end
                    task.wait(0.4) -- Weniger Spam für Server-Sicherheit!
                end
            end)

            local function SpiderLerp(tPos, currentSpeed)
                local startPos = root.Position
                local flatStart = Vector3.new(startPos.X, 0, startPos.Z)
                local flatTarget = Vector3.new(tPos.X, 0, tPos.Z)
                local totalDist = (flatStart - flatTarget).Magnitude
                
                if totalDist < 5 then return true end 
                
                local elapsedTime = 0
                local currentY = root.Position.Y
                local isClimbingState = false
                
                char:SetAttribute("evading", true)
                _G.soruDashing = true
                _G.canuse = true

                local rayParams = RaycastParams.new()
                rayParams.FilterDescendantsInstances = {char, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles")}
                rayParams.FilterType = Enum.RaycastFilterType.Exclude
                rayParams.IgnoreWater = true

                local baseMinY = 1
                pcall(function()
                    if Workspace:FindFirstChild("Env") and Workspace.Env:FindFirstChild("WaterStuff") and Workspace.Env.WaterStuff:FindFirstChild("Water") then
                        baseMinY = math.max(Workspace.Env.WaterStuff.Water.Position.Y + floorOffset, 1)
                    end
                end)

                local function GetTrueTopY(x, z)
                    local origin = Vector3.new(x, 3000, z)
                    local hit = Workspace:Raycast(origin, Vector3.new(0, -4000, 0), rayParams)
                    if hit and hit.Instance.Transparency < 1 then
                        return math.max(hit.Position.Y + floorOffset, baseMinY)
                    end
                    return baseMinY
                end

                hum.PlatformStand = false

                while elapsedTime < 200 do
                    local dt = RunService.Heartbeat:Wait()
                    dt = math.clamp(dt, 0.001, 0.05)
                    
                    for _, part in pairs(char:GetChildren()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                    
                    local currentPos = root.Position
                    local flatCurrent = Vector3.new(currentPos.X, 0, currentPos.Z)
                    local flatTargetPos = Vector3.new(tPos.X, 0, tPos.Z)
                    local remainingDist = (flatCurrent - flatTargetPos).Magnitude
                    
                    if remainingDist <= 5 then break end
                    
                    -- Server Safe Speed: Maximal 40!
                    local stepDist = math.min(currentSpeed * dt, remainingDist)
                    local flatMoveDir = (flatTargetPos - flatCurrent).Unit 
                    
                    local nextX = currentPos.X + (flatMoveDir.X * stepDist)
                    local nextZ = currentPos.Z + (flatMoveDir.Z * stepDist)
                    local calcPos = Vector3.new(nextX, currentY, nextZ)
                    
                    local wallAhead = Workspace:Raycast(calcPos, flatMoveDir * 4, rayParams) 
                    local roofAbove = Workspace:Raycast(calcPos, Vector3.new(0, 6, 0), rayParams)
                    
                    local targetY = GetTrueTopY(nextX, nextZ)

                    if wallAhead and wallAhead.Instance and wallAhead.Instance.Transparency < 1 then
                        targetY = math.max(targetY, GetTrueTopY(wallAhead.Position.X + (flatMoveDir.X * 0.8), wallAhead.Position.Z + (flatMoveDir.Z * 0.8)))
                        isClimbingState = true
                    end
                    
                    local advanceSpeed = 1
                    if (isClimbingState and currentY < targetY - 0.5) or (roofAbove and roofAbove.Instance.Transparency < 1) then
                        advanceSpeed = 0
                    end

                    local maxYStep = isClimbingState and (50 * dt * 25) or (30 * dt * 25)
                    local yDiff = targetY - currentY
                    
                    if yDiff > 0 then
                        currentY = math.min(currentY + maxYStep, targetY)
                    elseif yDiff < 0 then
                        currentY = math.max(currentY - maxYStep, targetY)
                    end
                    
                    if isClimbingState and math.abs(currentY - targetY) <= 0.5 then
                        isClimbingState = false
                    end
                    
                    currentY = math.max(currentY, baseMinY)
                    elapsedTime = elapsedTime + (dt * advanceSpeed)
                    
                    local finalPos = Vector3.new(nextX, currentY, nextZ)
                    root.CFrame = CFrame.lookAt(finalPos, Vector3.new(tPos.X, currentY, tPos.Z))
                    root.Velocity = Vector3.new(flatMoveDir.X * currentSpeed * advanceSpeed, 0, flatMoveDir.Z * currentSpeed * advanceSpeed)
                    bp.Position = finalPos
                end
                
                isFlyingActive = false
                if footstepEvent and type(footstepEvent.FireServer) == "function" then 
                    pcall(function() footstepEvent:FireServer("land") end) 
                end
                
                hum:Move(Vector3.new(0,0,0), false)
                char:SetAttribute("evading", nil)
                _G.soruDashing = nil
                root.Velocity = Vector3.new(0, 0, 0)
                return true
            end
            
            SpiderLerp(targetPos, RyuConfig.IslandSpeed)
            hum.Jump = true
            root.Velocity = Vector3.new(0, 0, 0)
            if bp then bp:Destroy() end
        end)
        _G.RyuIsTweening = false
    end)
end)

-- Settings
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
