--// ==========================================
--// RYU HUB - GPO EDITION (CHECKPOINT 1 TWEEN + AUTO FARM)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "RyuHubPremium" then v:Destroy() end end

--// DYNAMISCHER WORKSPACE SCANNER FÜR INSELN & MOBS
local function GetDynamicLists()
    local islands = {}
    local islandsFolder = Workspace:FindFirstChild("Islands")
    if islandsFolder then
        for _, v in pairs(islandsFolder:GetChildren()) do
            table.insert(islands, v.Name)
        end
    else
        islands = {
            "???? Shrine", "A rock", "Coco Island", "Colosseum", "Colosseum of Arc", 
            "Desert Kingdom", "Dokkan Island", "Fishman Cave", "Fishman Island", 
            "Foro Island", "Impel Base", "Marine Base G-1", "Marine Fort F-1", 
            "Mirror World", "Mysterious Cliff", "Mysterious Reef", "Orange Town", 
            "Restaurant Baratie", "Reverse Mountain", "Roca Island", "Rose Kingdom", 
            "Rovo Island", "Sakura Stronghold", "Sandora", "Sashi Island", 
            "Sett's Arena", "Shark Park", "Shell's Town", "Sphinx Island", 
            "Spirit Island", "Thriller Bark", "Town of Beginnings", 
            "Turtleback Cave", "Umi Island", "Whole Cake Island"
        }
    end
    table.sort(islands)
    return islands
end

local InitIslands = GetDynamicLists()

--// RYU CONFIGURATION
local RyuConfig = {
    TargetIsland = InitIslands[1] or "Town of Beginnings",
    IslandSpeed = 65, -- Fixer Speed
    GuiColor = Color3.fromRGB(255, 255, 255),
    
    -- Auto Farm
    AutoFarm = false,
    FarmMode = "Solo",
    TargetNPC = "",
    TargetMob = "",
    FarmHoverHeight = 5,
    NoclipDash = false
}

--// PREMIUM MONOCHROME THEME
local Theme = {
    Background = Color3.fromRGB(12, 12, 14),
    Sidebar = Color3.fromRGB(18, 18, 20),
    SectionBG = Color3.fromRGB(24, 24, 26),
    Text = Color3.fromRGB(250, 250, 250),
    SubText = Color3.fromRGB(130, 130, 135),
    Accent = RyuConfig.GuiColor,
    ToggleOff = Color3.fromRGB(35, 35, 38),
    ToggleOn = RyuConfig.GuiColor,
    Stroke = Color3.fromRGB(45, 45, 50)
}

local MainSize = UDim2.new(0, math.min(750, camera.ViewportSize.X - 40), 0, math.min(480, camera.ViewportSize.Y - 40))
local SidebarWidth = 160

local RyuHub = Instance.new("ScreenGui")
RyuHub.Name = "RyuHubPremium"
RyuHub.ResetOnSpawn = false
RyuHub.IgnoreGuiInset = true
RyuHub.Parent = guiParent

--// NOTIFICATIONS
local NotificationContainer = Instance.new("Frame", guiParent)
NotificationContainer.Name = "RyuNotifications"
NotificationContainer.Size = UDim2.new(0, 260, 1, -40)
NotificationContainer.Position = UDim2.new(1, -280, 0, 20)
NotificationContainer.BackgroundTransparency = 1
local NotifLayout = Instance.new("UIListLayout", NotificationContainer)
NotifLayout.SortOrder = Enum.SortOrder.LayoutOrder
NotifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
NotifLayout.Padding = UDim.new(0, 8)

local RyuNotify = {}
function RyuNotify:Send(title, text, duration)
    duration = duration or 3
    local NotifFrame = Instance.new("Frame", NotificationContainer)
    NotifFrame.Size = UDim2.new(1, 0, 0, 60); NotifFrame.BackgroundColor3 = Theme.Sidebar; NotifFrame.BorderSizePixel = 0
    Instance.new("UICorner", NotifFrame).CornerRadius = UDim.new(0, 8)
    local Stroke = Instance.new("UIStroke", NotifFrame); Stroke.Color = Theme.Accent; Stroke.Transparency = 0.5
    local TitleLabel = Instance.new("TextLabel", NotifFrame); TitleLabel.Size = UDim2.new(1, -20, 0, 20); TitleLabel.Position = UDim2.new(0, 15, 0, 8); TitleLabel.BackgroundTransparency = 1; TitleLabel.Text = title; TitleLabel.TextColor3 = Theme.Text; TitleLabel.Font = Enum.Font.GothamBold; TitleLabel.TextSize = 13; TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    local DescLabel = Instance.new("TextLabel", NotifFrame); DescLabel.Size = UDim2.new(1, -20, 0, 20); DescLabel.Position = UDim2.new(0, 15, 0, 28); DescLabel.BackgroundTransparency = 1; DescLabel.Text = text; DescLabel.TextColor3 = Theme.SubText; DescLabel.Font = Enum.Font.Gotham; DescLabel.TextSize = 11; DescLabel.TextXAlignment = Enum.TextXAlignment.Left
    task.spawn(function()
        task.wait(duration)
        NotifFrame:Destroy()
    end)
end

--// UI BUILDER HELPERS
local MainFrame = Instance.new("Frame", RyuHub)
MainFrame.Size = MainSize; MainFrame.Position = UDim2.new(0.5, -MainSize.X.Offset/2, 0.5, -MainSize.Y.Offset/2); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.Active = true; MainFrame.Visible = true; MainFrame.ClipsDescendants = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", MainFrame).Color = Theme.Stroke

local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextColor3 = Theme.Accent; Title.TextXAlignment = Enum.TextXAlignment.Left
local SubTitle = Instance.new("TextLabel", Topbar); SubTitle.Size = UDim2.new(0, 300, 0, 15); SubTitle.Position = UDim2.new(0, 20, 0, 38); SubTitle.BackgroundTransparency = 1; SubTitle.Text = "Grand Piece Online"; SubTitle.TextColor3 = Theme.SubText; SubTitle.Font = Enum.Font.Gotham; SubTitle.TextSize = 11; SubTitle.TextXAlignment = Enum.TextXAlignment.Left

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end end)
Topbar.InputChanged:Connect(function(input) if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - mDragStart; MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y) end end)
Topbar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end end)

local Sidebar = Instance.new("ScrollingFrame", MainFrame); Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar); SideLayout.Padding = UDim.new(0, 6); SideLayout.SortOrder = Enum.SortOrder.LayoutOrder
local ContentContainer = Instance.new("Frame", MainFrame); ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1

local Tabs = {}
local function CreateMainTab(name)
    local tabObj = { Btn = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {}, ToggleFunc = nil }
    local tabBtn = Instance.new("TextButton", Sidebar); tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name); tabBtn.TextColor3 = Theme.SubText; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 13; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8); tabObj.Btn = tabBtn
    local subContainer = Instance.new("Frame", Sidebar); subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer); subLayout.Padding = UDim.new(0, 2); tabObj.SubLayout = subLayout
    tabObj.ToggleFunc = function()
        tabObj.IsOpen = not tabObj.IsOpen
        subContainer.Size = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)
    end
    tabBtn.MouseButton1Click:Connect(tabObj.ToggleFunc)
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end end)
    table.insert(Tabs, tabObj); return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subBtn = Instance.new("TextButton", tabObj.SubContainer); subBtn.Size = UDim2.new(1, 0, 0, 28); subBtn.BackgroundTransparency = 1; subBtn.Text = "     " .. subName; subBtn.TextColor3 = Theme.SubText; subBtn.Font = Enum.Font.GothamMedium; subBtn.TextSize = 12; subBtn.TextXAlignment = Enum.TextXAlignment.Left
    local page = Instance.new("ScrollingFrame", ContentContainer); page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.ScrollBarThickness = 2; page.Visible = false
    local pageLayout = Instance.new("UIListLayout", page); pageLayout.Padding = UDim.new(0, 12); pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20) end)
    subBtn.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do for _, st in pairs(t.SubTabs) do st.Page.Visible = false end end
        page.Visible = true
    end)
    table.insert(tabObj.SubTabs, {Page = page})
    return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page); section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10); Instance.new("UIStroke", section).Color = Theme.Stroke
    local secLayout = Instance.new("UIListLayout", section); secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local secPadding = Instance.new("UIPadding", section); secPadding.PaddingTop = UDim.new(0, 12); secPadding.PaddingBottom = UDim.new(0, 12)
    local title = Instance.new("TextLabel", section); title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText; title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end)
    return section
end

local function CreateToggle(section, text, defaultState, callback)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 34); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(0.7, 0, 0, 34); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.Text; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    local tBtn = Instance.new("TextButton", frame); tBtn.Size = UDim2.new(0, 42, 0, 22); tBtn.Position = UDim2.new(1, -42, 0, 6); tBtn.BackgroundColor3 = defaultState and Theme.ToggleOn or Theme.ToggleOff; tBtn.Text = ""; Instance.new("UICorner", tBtn).CornerRadius = UDim.new(1, 0)
    local isOn = defaultState or false
    tBtn.MouseButton1Click:Connect(function()
        isOn = not isOn
        tBtn.BackgroundColor3 = isOn and Theme.ToggleOn or Theme.ToggleOff
        if callback then callback(isOn) end
    end)
end

local function CreateTextBox(section, placeholder, callback)
    local box = Instance.new("TextBox", section); box.Size = UDim2.new(0.92, 0, 0, 34); box.BackgroundColor3 = Theme.Background; box.Text = ""; box.PlaceholderText = placeholder; box.TextColor3 = Theme.Text; box.Font = Enum.Font.GothamMedium; box.TextSize = 12; Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", box).Color = Theme.Stroke
    if callback then box.FocusLost:Connect(function() callback(box.Text) end) end
    return box
end

local function CreateButton(section, text, color, callback)
    local btn = Instance.new("TextButton", section); btn.Size = UDim2.new(0.92, 0, 0, 34); btn.BackgroundColor3 = color; btn.Text = text; btn.TextColor3 = Color3.fromRGB(255,255,255); btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", btn).Color = Theme.Stroke
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function CreateDropdown(section, headerText, itemsList, targetConfigKey)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 130); frame.BackgroundTransparency = 1
    local header = Instance.new("TextLabel", frame); header.Size = UDim2.new(1, 0, 0, 20); header.BackgroundTransparency = 1; header.Text = headerText .. ": " .. tostring(RyuConfig[targetConfigKey] or "None"); header.TextColor3 = Theme.SubText; header.Font = Enum.Font.GothamMedium; header.TextSize = 12; header.TextXAlignment = Enum.TextXAlignment.Left
    local scroll = Instance.new("ScrollingFrame", frame); scroll.Size = UDim2.new(1, 0, 0, 100); scroll.Position = UDim2.new(0, 0, 0, 25); scroll.BackgroundColor3 = Theme.Background; scroll.ScrollBarThickness = 2; Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 6)
    local listLayout = Instance.new("UIListLayout", scroll); listLayout.Padding = UDim.new(0, 4); listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    for _, itemName in ipairs(itemsList) do
        local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. tostring(itemName); btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
        btn.MouseButton1Click:Connect(function() RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. tostring(itemName) end)
    end
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
end

--// =======================
--// CHECKPOINT 1 SMART TWEEN ENGINE
--// =======================
local function ToggleHover(state, root)
    if state then
        local bp = root:FindFirstChild("RyuHover")
        if not bp then
            bp = Instance.new("BodyPosition")
            bp.Name = "RyuHover"; bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.D = 500; bp.P = 50000; bp.Parent = root
        end
        bp.Position = root.Position
        return bp
    else
        local bp = root:FindFirstChild("RyuHover")
        if bp then bp:Destroy() end
        return nil
    end
end

-- Universelle Tween Funktion (wird für Transport & Auto Farm genutzt)
local function SmartTween(targetPos, speedLimit, floorOffset)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return false end

    local bp = ToggleHover(true, root)
    
    local climbEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("climb")
    local sprintEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("sprint")
    
    if sprintEvent then pcall(function() sprintEvent:FireServer("rbxassetid://15382065457") end) end
    
    local fakeFloor = Workspace:FindFirstChild("RyuFakeFloor")
    if not fakeFloor then
        fakeFloor = Instance.new("Part")
        fakeFloor.Name = "RyuFakeFloor"; fakeFloor.Size = Vector3.new(4, 1, 4); fakeFloor.Anchored = true; fakeFloor.CanCollide = true; fakeFloor.Transparency = 1; fakeFloor.Parent = Workspace
    end
    
    local function GetTrueTopY(x, z)
        local rParams = RaycastParams.new()
        local currentFilter = {char, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles"), fakeFloor}
        rParams.FilterType = Enum.RaycastFilterType.Exclude
        rParams.IgnoreWater = true
        local origin = Vector3.new(x, 4000, z)
        for i = 1, 10 do
            rParams.FilterDescendantsInstances = currentFilter
            local hit = Workspace:Raycast(origin, Vector3.new(0, -5000, 0), rParams)
            if hit then 
                if hit.Instance.Transparency < 1 then return hit.Position.Y 
                else table.insert(currentFilter, hit.Instance) end
            else break end
        end
        return 0 
    end

    local isClimbingState = false
    char:SetAttribute("evading", true)
    
    local elapsedTime = 0
    local flatStart = Vector3.new(root.Position.X, 0, root.Position.Z)
    local flatTarget = Vector3.new(targetPos.X, 0, targetPos.Z)
    local totalDist = (flatStart - flatTarget).Magnitude
    local t = totalDist / speedLimit
    local currentY = root.Position.Y
    
    while elapsedTime < t do
        -- Abbruch wenn Features ausgeschaltet werden
        if not RyuConfig.AutoFarm and not _G.RyuIsTweening then break end

        local dt = RunService.Heartbeat:Wait()
        dt = math.clamp(dt, 0.001, 0.05)
        
        local currentFlat = Vector3.new(root.Position.X, 0, root.Position.Z)
        local distToTarget = (Vector3.new(targetPos.X, 0, targetPos.Z) - currentFlat).Magnitude
        
        -- WICHTIG: Bricht NIEMALS vor dem Ziel ab. Erst bei < 3 Studs!
        if distToTarget < 3 then break end 
        
        local moveDir = (Vector3.new(targetPos.X, 0, targetPos.Z) - currentFlat).Unit
        if Vector3.new(targetPos.X, 0, targetPos.Z) == currentFlat or distToTarget < 0.1 then moveDir = root.CFrame.LookVector end
        
        local currentX = root.Position.X + (moveDir.X * speedLimit * dt)
        local currentZ = root.Position.Z + (moveDir.Z * speedLimit * dt)
        local calcPos = Vector3.new(currentX, currentY, currentZ)
        
        local roofY = GetTrueTopY(currentX, currentZ) + floorOffset
        local groundY = GetTrueTopY(currentX, currentZ)
        local targetY = math.max(groundY + floorOffset, roofY)
        targetY = math.max(targetY, 5) -- Niemals ins Wasser fallen
        
        local addTime = dt
        local isHittingWall1Stud = false
        
        -- 1 Stud Wall Check für das Klettern
        local rayParamsDown = RaycastParams.new()
        rayParamsDown.FilterDescendantsInstances = {char, Workspace:FindFirstChild("Effects"), fakeFloor}
        rayParamsDown.FilterType = Enum.RaycastFilterType.Exclude
        local wallHit = Workspace:Raycast(calcPos, moveDir * 1, rayParamsDown)
        
        if wallHit then
            if wallHit.Instance.Transparency < 1 and wallHit.Instance.Name ~= "Ocean" then
                isHittingWall1Stud = true
                local wallTopY = GetTrueTopY(wallHit.Position.X, wallHit.Position.Z) + floorOffset
                if wallTopY > currentY then targetY = math.max(targetY, wallTopY) end
            end
        end
        
        local isWallInFront = (targetY > currentY + 1) or isHittingWall1Stud
        
        -- Klettern Halten (Kein Spam)
        if isWallInFront and not isClimbingState then
            isClimbingState = true; if climbEvent then task.spawn(function() pcall(function() climbEvent:InvokeServer(true) end) end) end
        elseif not isWallInFront and isClimbingState then
            isClimbingState = false; if climbEvent then task.spawn(function() pcall(function() climbEvent:InvokeServer(false) end) end) end
        end

        if targetY > currentY + 1 then currentX = root.Position.X; currentZ = root.Position.Z end 

        if currentY < targetY - 0.5 then currentY = math.min(currentY + (600 * dt), targetY)
        elseif currentY > targetY + 0.5 then currentY = math.max(currentY - (60 * dt), targetY)
        else currentY = targetY end
        
        currentY = math.max(currentY, 0)
        
        local finalPos = Vector3.new(currentX, currentY, currentZ)
        bp.Position = finalPos
        root.CFrame = CFrame.lookAt(root.Position, Vector3.new(targetPos.X, root.Position.Y, targetPos.Z))
        
        if fakeFloor then fakeFloor.CFrame = root.CFrame * CFrame.new(0, -((hum.HipHeight or 2) + (root.Size.Y / 2) + 0.05), 0) end
        
        -- Lauf Animation
        if hum then hum:ChangeState(Enum.HumanoidStateType.Running); hum:Move(moveDir, false) end
        root.Velocity = Vector3.new(moveDir.X * speedLimit, 0, moveDir.Z * speedLimit)
        
        elapsedTime = elapsedTime + addTime
    end
    
    if fakeFloor then fakeFloor:Destroy() end
    if hum then hum:Move(Vector3.new(0,0,0), false) end
    if climbEvent and isClimbingState then pcall(function() climbEvent:InvokeServer(false) end) end
    ToggleHover(false, root)
    char:SetAttribute("evading", nil)
    root.Velocity = Vector3.new(0, 0, 0)
    return true
end

--// =======================
--// AUTO FARM LOGIC
--// =======================
local currentComboIndex = 1

local function PerformMeleeAttack(targets)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    -- Exakt 0.3s Delay, kein Spam!
    task.wait(0.3)
    
    local hitParts = {}
    for _, npc in ipairs(targets) do
        local mRoot = npc:FindFirstChild("HumanoidRootPart")
        local mHum = npc:FindFirstChildOfClass("Humanoid")
        if mRoot and mHum and mHum.Health > 0 then table.insert(hitParts, mRoot) end
    end
    
    local animName = "Punch" .. currentComboIndex
    if currentComboIndex == 4 then animName = "GroundPunch4" end
    
    local animObj = ReplicatedStorage:FindFirstChild("CombatAnimations") and ReplicatedStorage.CombatAnimations:FindFirstChild("Melee") and ReplicatedStorage.CombatAnimations.Melee:FindFirstChild(animName)
    if animObj then
        pcall(function() ReplicatedStorage.Events.CombatRegister:InvokeServer({"swingsfx", "Melee", currentComboIndex, "Ground", currentComboIndex == 1, animObj, 2, 1.5}) end)
    end
    
    if #hitParts > 0 then
        pcall(function() ReplicatedStorage.Events.CombatRegister:InvokeServer({"damage", hitParts, "Melee", {currentComboIndex, "Ground", "Melee"}, true, root.CFrame, ["aircombo"] = "Ground"}) end)
    end
    
    currentComboIndex = currentComboIndex + 1
    if currentComboIndex > 4 then currentComboIndex = 1 end
end

task.spawn(function()
    while true do
        task.wait(0.1)
        if not RyuConfig.AutoFarm then continue end
        
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then continue end
        
        if RyuConfig.TargetMob ~= "" then
            local npcs = Workspace:FindFirstChild("NPCs")
            if not npcs then continue end
            
            local targetMobs = {}
            for _, npc in pairs(npcs:GetChildren()) do
                if npc.Name == RyuConfig.TargetMob then
                    local mHum = npc:FindFirstChildOfClass("Humanoid")
                    local mRoot = npc:FindFirstChild("HumanoidRootPart")
                    if mHum and mRoot and mHum.Health > 0 and not npc:FindFirstChild("Rag") and not mHum:GetAttribute("isRagdolled") then
                        table.insert(targetMobs, npc)
                    end
                end
            end
            
            if #targetMobs > 0 then
                if RyuConfig.FarmMode == "Solo" then
                    for _, mob in ipairs(targetMobs) do
                        if not RyuConfig.AutoFarm then break end
                        local mHum = mob:FindFirstChildOfClass("Humanoid")
                        local mRoot = mob:FindFirstChild("HumanoidRootPart")
                        if mHum and mRoot and mHum.Health > 0 then
                            -- Nutzt SmartTween mit max 65 Speed und festgelegter Höhe
                            SmartTween(mRoot.Position, 65, RyuConfig.FarmHoverHeight)
                            
                            while RyuConfig.AutoFarm and mHum.Health > 0 do
                                PerformMeleeAttack({mob})
                                local bp = root:FindFirstChild("RyuHover")
                                if bp then bp.Position = mRoot.Position + Vector3.new(0, RyuConfig.FarmHoverHeight, 0) end
                            end
                        end
                    end
                elseif RyuConfig.FarmMode == "Group" then
                    -- Group Farming (Zieht NPCs zusammen, Tweened zum Zentrum)
                    local centerPos = Vector3.new(0,0,0)
                    for _, mob in ipairs(targetMobs) do
                        local mRoot = mob:FindFirstChild("HumanoidRootPart")
                        if mRoot then 
                            centerPos = centerPos + mRoot.Position
                            SmartTween(mRoot.Position, 65, RyuConfig.FarmHoverHeight)
                            PerformMeleeAttack({mob}) 
                        end
                    end
                    centerPos = centerPos / #targetMobs
                    SmartTween(centerPos, 65, RyuConfig.FarmHoverHeight)
                    
                    local anyAlive = true
                    while RyuConfig.AutoFarm and anyAlive do
                        anyAlive = false
                        for _, mob in ipairs(targetMobs) do
                            local mHum = mob:FindFirstChildOfClass("Humanoid")
                            local mRoot = mob:FindFirstChild("HumanoidRootPart")
                            if mHum and mRoot and mHum.Health > 0 then
                                anyAlive = true
                                mRoot.CFrame = CFrame.new(centerPos)
                            end
                        end
                        if anyAlive then PerformMeleeAttack(targetMobs) end
                    end
                end
            end
        end
    end
end)


--// =======================
--// TABS & SECTIONS
--// =======================

-- TAB 1: FARM
local TabFarm = CreateMainTab("FARM")
local SubNPCFarm = CreateSubTab(TabFarm, "NPC Farm")

local SecNPC = CreateSection(SubNPCFarm, "Auto Farm Settings")
CreateToggle(SecNPC, "Enable NPC Farm", false, function(state) RyuConfig.AutoFarm = state end)
CreateDropdown(SecNPC, "Farm Mode", {"Solo", "Group"}, "Solo")
CreateTextBox(SecNPC, "Quest NPC Name", function(val) RyuConfig.TargetNPC = val end)
CreateTextBox(SecNPC, "Target Mob Name", function(val) RyuConfig.TargetMob = val end)

-- TAB 2: PLAYER
local TabPlayer = CreateMainTab("PLAYER")
local SubUtility = CreateSubTab(TabPlayer, "Utility")

local SecUtil = CreateSection(SubUtility, "Player Utility")
CreateButton(SecUtil, "Unlock Geppo", Theme.SectionBG, function()
    pcall(function() ReplicatedStorage.Events.Skill:InvokeServer("Geppo") end)
    pcall(function() ReplicatedStorage.Events.Skill:InvokeServer("Sky Walk") end)
    pcall(function() ReplicatedStorage.Events.Skill:InvokeServer("Sky Walk2") end)
    -- Lokaler Bypass, falls Remote geblockt ist
    local stats = LocalPlayer:FindFirstChild("Stats") or LocalPlayer:FindFirstChild("Data")
    if stats and stats:FindFirstChild("Skills") then
        if stats.Skills:FindFirstChild("skyWalk") then stats.Skills.skyWalk.Value = true end
        if stats.Skills:FindFirstChild("Geppo") then stats.Skills.Geppo.Value = 1 end
    end
    RyuNotify:Send("Utility", "Geppo unlocked!", 3)
end)

local NoclipLoop
CreateToggle(SecUtil, "Noclip (Dash Bypass)", false, function(state)
    RyuConfig.NoclipDash = state
    if state then
        NoclipLoop = RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            if char and RyuConfig.NoclipDash then
                for _, v in pairs(char:GetChildren()) do
                    if v:IsA("BasePart") and v.CanCollide then v.CanCollide = false end
                end
                if char:FindFirstChild("Humanoid") and char.Humanoid.MoveDirection.Magnitude > 0 then
                    pcall(function() ReplicatedStorage.Events.Skill:InvokeServer("Dash") end)
                    pcall(function() ReplicatedStorage.Events.Skill:InvokeServer("Soru") end)
                end
            end
        end)
    else
        if NoclipLoop then NoclipLoop:Disconnect(); NoclipLoop = nil end
    end
end)

-- TAB 3: MOBILITY
local TabMobility = CreateMainTab("MOBILITY")
local SubTween = CreateSubTab(TabMobility, "Tween")

local SecIslandTP = CreateSection(SubTween, "Spider Tween (Islands)")
CreateDropdown(SecIslandTP, "Selected Island", InitIslands, "TargetIsland")
CreateButton(SecIslandTP, "Start Spider Tween", Theme.SectionBG, function()
    if _G.RyuIsTweening then return end
    _G.RyuIsTweening = true
    task.spawn(function()
        local targetIslandName = RyuConfig.TargetIsland
        local island = nil
        local islandsFolder = Workspace:FindFirstChild("Islands")
        if islandsFolder then
            for _, v in pairs(islandsFolder:GetChildren()) do
                if string.lower(v.Name) == string.lower(targetIslandName) then island = v; break end
            end
        end
        if not island then _G.RyuIsTweening = false; return end
        
        local targetPos = island:IsA("Model") and island:GetPivot().Position or island.Position
        
        SmartTween(targetPos, RyuConfig.IslandSpeed, 5)
        _G.RyuIsTweening = false
    end)
end)

-- INITIALISIERUNG
task.spawn(function()
    if Tabs[1] and Tabs[1].ToggleFunc then Tabs[1].ToggleFunc() end
    if Tabs[1].SubTabs[1] and Tabs[1].SubTabs[1].SelectFunc then Tabs[1].SubTabs[1].SelectFunc() end
end)
