--// ============================================================================
--// RYU HUB - BATTLE ROYALE & GPO EDITION (NO DISTANCE LIMIT / NO ERRORS)
--// ============================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI SECURITY & CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui", 10) or LocalPlayer:FindFirstChild("PlayerGui")
pcall(function() 
    if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end 
end)

for _, v in pairs(guiParent:GetChildren()) do 
    if v.Name == "RyuHubPremium" or v.Name == "RyuNotifications" then v:Destroy() end 
end

--// DYNAMISCHER WORKSPACE SCANNER
local function GetDynamicLists()
    local islands = {}
    local islandsFolder = Workspace:FindFirstChild("Islands")
    if islandsFolder then
        for _, v in pairs(islandsFolder:GetChildren()) do
            table.insert(islands, v.Name)
        end
    else
        islands = {"Town of Beginnings", "Sandora", "Shell's Town", "Coco Island"} -- Fallbacks
    end
    table.sort(islands)
    return islands
end

local InitIslands = GetDynamicLists()

--// RYU CONFIGURATION
local RyuConfig = {
    TargetIsland = InitIslands[1] or "Town of Beginnings",
    IslandSpeed = 65, -- Max 65 limit
    GuiColor = Color3.fromRGB(255, 255, 255)
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

--// UI SETUP
local currentMainSize = UDim2.new(0, 550, 0, 380) 
local SidebarWidth = 150

local RyuHub = Instance.new("ScreenGui"); RyuHub.Name = "RyuHubPremium"; RyuHub.ResetOnSpawn = false; RyuHub.IgnoreGuiInset = true; RyuHub.Parent = guiParent
local MainFrame = Instance.new("Frame"); MainFrame.Size = currentMainSize; MainFrame.Position = UDim2.new(0.5, -currentMainSize.X.Offset/2, 0.5, -currentMainSize.Y.Offset/2); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.Active = true; MainFrame.Visible = true; MainFrame.ClipsDescendants = true; MainFrame.Parent = RyuHub; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local Topbar = Instance.new("Frame", MainFrame); Topbar.Size = UDim2.new(1, 0, 0, 60); Topbar.BackgroundTransparency = 1
local Title = Instance.new("TextLabel", Topbar); Title.Size = UDim2.new(0, 300, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1; Title.Text = "RYU HUB"; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 22; Title.TextColor3 = Theme.Text; Title.TextXAlignment = Enum.TextXAlignment.Left

local ResizeGrip = Instance.new("TextButton", MainFrame); ResizeGrip.Size = UDim2.new(0, 20, 0, 20); ResizeGrip.Position = UDim2.new(1, -20, 1, -20); ResizeGrip.BackgroundTransparency = 1; ResizeGrip.Text = "◢"; ResizeGrip.TextColor3 = Theme.SubText; ResizeGrip.TextSize = 16; ResizeGrip.Font = Enum.Font.GothamBold

local mDragging, mDragStart, mStartPos
Topbar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = true; mDragStart = input.Position; mStartPos = MainFrame.Position end end)
Topbar.InputChanged:Connect(function(input) if mDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - mDragStart; MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + delta.X, mStartPos.Y.Scale, mStartPos.Y.Offset + delta.Y) end end)
Topbar.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then mDragging = false end end)

local ContentContainer = Instance.new("Frame", MainFrame); ContentContainer.Size = UDim2.new(1, -(SidebarWidth + 25), 1, -85); ContentContainer.Position = UDim2.new(0, SidebarWidth + 15, 0, 75); ContentContainer.BackgroundTransparency = 1
local Sidebar = Instance.new("ScrollingFrame", MainFrame); Sidebar.Size = UDim2.new(0, SidebarWidth, 1, -85); Sidebar.Position = UDim2.new(0, 10, 0, 75); Sidebar.BackgroundTransparency = 1; Sidebar.ScrollBarThickness = 0
local SideLayout = Instance.new("UIListLayout", Sidebar); SideLayout.Padding = UDim.new(0, 6); SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; SideLayout.SortOrder = Enum.SortOrder.LayoutOrder

local Tabs = {}
local function UpdateSidebarCanvas()
    local totalH = 10
    for _, t in pairs(Tabs) do totalH = totalH + 36 + 6; if t.IsOpen then totalH = totalH + t.SubLayout.AbsoluteContentSize.Y + 6 end end
    Sidebar.CanvasSize = UDim2.new(0, 0, 0, totalH)
end

local function CreateMainTab(name)
    local tabObj = { Btn = nil, SubContainer = nil, SubLayout = nil, IsOpen = false, SubTabs = {} }
    local tabBtn = Instance.new("TextButton", Sidebar); tabBtn.Size = UDim2.new(1, 0, 0, 36); tabBtn.BackgroundColor3 = Theme.Sidebar; tabBtn.Text = "  " .. string.upper(name); tabBtn.TextColor3 = Theme.SubText; tabBtn.Font = Enum.Font.GothamBlack; tabBtn.TextSize = 13; tabBtn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8); tabObj.Btn = tabBtn
    local subContainer = Instance.new("Frame", Sidebar); subContainer.Size = UDim2.new(1, 0, 0, 0); subContainer.BackgroundTransparency = 1; subContainer.ClipsDescendants = true; tabObj.SubContainer = subContainer
    local subLayout = Instance.new("UIListLayout", subContainer); subLayout.Padding = UDim.new(0, 2); tabObj.SubLayout = subLayout
    tabBtn.Activated:Connect(function() tabObj.IsOpen = not tabObj.IsOpen; TweenService:Create(subContainer, TweenInfo.new(0.25), {Size = tabObj.IsOpen and UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) or UDim2.new(1, 0, 0, 0)}):Play(); tabBtn.TextColor3 = tabObj.IsOpen and Theme.Text or Theme.SubText; task.delay(0.26, UpdateSidebarCanvas); UpdateSidebarCanvas() end)
    subLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() if tabObj.IsOpen then subContainer.Size = UDim2.new(1, 0, 0, subLayout.AbsoluteContentSize.Y) end; UpdateSidebarCanvas() end)
    table.insert(Tabs, tabObj); return tabObj
end

local function CreateSubTab(tabObj, subName)
    local subBtn = Instance.new("TextButton", tabObj.SubContainer); subBtn.Size = UDim2.new(1, 0, 0, 28); subBtn.BackgroundTransparency = 1; subBtn.Text = "     " .. subName; subBtn.TextColor3 = Theme.SubText; subBtn.Font = Enum.Font.GothamMedium; subBtn.TextSize = 12; subBtn.TextXAlignment = Enum.TextXAlignment.Left
    local page = Instance.new("ScrollingFrame", ContentContainer); page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.ScrollBarThickness = 2; page.Visible = false
    local pageLayout = Instance.new("UIListLayout", page); pageLayout.Padding = UDim.new(0, 12); pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 20) end)
    subBtn.Activated:Connect(function() for _, v in pairs(ContentContainer:GetChildren()) do v.Visible = false end; page.Visible = true end)
    return page
end

local function CreateSection(page, titleText)
    local section = Instance.new("Frame", page); section.Size = UDim2.new(0.98, 0, 0, 50); section.BackgroundColor3 = Theme.SectionBG; Instance.new("UICorner", section).CornerRadius = UDim.new(0, 10)
    local secLayout = Instance.new("UIListLayout", section); secLayout.Padding = UDim.new(0, 10); secLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; secLayout.SortOrder = Enum.SortOrder.LayoutOrder
    Instance.new("UIPadding", section).PaddingTop = UDim.new(0, 12); Instance.new("UIPadding", section).PaddingBottom = UDim.new(0, 12)
    local title = Instance.new("TextLabel", section); title.LayoutOrder = -1; title.Size = UDim2.new(0.92, 0, 0, 24); title.BackgroundTransparency = 1; title.Text = titleText; title.TextColor3 = Theme.Text; title.Font = Enum.Font.GothamBold; title.TextSize = 14; title.TextXAlignment = Enum.TextXAlignment.Left
    secLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() section.Size = UDim2.new(1, 0, 0, secLayout.AbsoluteContentSize.Y + 24) end)
    return section
end

local function CreateButton(section, text, callback)
    local btn = Instance.new("TextButton", section); btn.Size = UDim2.new(0.92, 0, 0, 34); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = text; btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 13; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    local stroke = Instance.new("UIStroke", btn); stroke.Color = Theme.Stroke; stroke.Thickness = 1
    btn.Activated:Connect(function() if callback then callback() end end)
end

local function CreateSlider(section, text, min, max, default, callback)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 50); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(1, 0, 0, 20); label.BackgroundTransparency = 1; label.Text = text; label.TextColor3 = Theme.SubText; label.Font = Enum.Font.GothamMedium; label.TextSize = 13; label.TextXAlignment = Enum.TextXAlignment.Left
    local valLabel = Instance.new("TextLabel", frame); valLabel.Size = UDim2.new(0, 40, 0, 20); valLabel.Position = UDim2.new(1, -40, 0, 0); valLabel.BackgroundTransparency = 1; valLabel.Text = tostring(default); valLabel.TextColor3 = Theme.Accent; valLabel.Font = Enum.Font.GothamBold; valLabel.TextSize = 13; valLabel.TextXAlignment = Enum.TextXAlignment.Right
    local sliderBg = Instance.new("Frame", frame); sliderBg.Size = UDim2.new(1, 0, 0, 4); sliderBg.Position = UDim2.new(0, 0, 0, 32); sliderBg.BackgroundColor3 = Theme.ToggleOff; Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)
    local sliderFill = Instance.new("Frame", sliderBg); local percentage = (default - min) / (max - min); sliderFill.Size = UDim2.new(percentage, 0, 1, 0); sliderFill.BackgroundColor3 = Theme.Accent; Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
    local knob = Instance.new("TextButton", sliderFill); knob.Size = UDim2.new(0, 14, 0, 14); knob.Position = UDim2.new(1, -7, 0.5, -7); knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255); knob.Text = ""; Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)
    local dragging = false
    local function setSlider(value) local relative = math.clamp((value - min) / (max - min), 0, 1); valLabel.Text = tostring(value); TweenService:Create(sliderFill, TweenInfo.new(0.08, Enum.EasingStyle.Quad), {Size = UDim2.new(relative, 0, 1, 0)}):Play(); if callback then callback(value) end end
    knob.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(1, -9, 0.5, -9)}):Play() end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false; TweenService:Create(knob, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {Size = UDim2.new(0, 14, 0, 14), Position = UDim2.new(1, -7, 0.5, -7)}):Play() end end)
    UserInputService.InputChanged:Connect(function(input) if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local relative = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1); setSlider(math.floor(min + (max - min) * relative)) end end)
end

--// SEARCHABLE DROPDOWN
local function CreateSearchableDropdown(section, headerText, itemsList, targetConfigKey)
    local frame = Instance.new("Frame", section); frame.Size = UDim2.new(0.92, 0, 0, 200); frame.BackgroundTransparency = 1
    local header = Instance.new("TextLabel", frame); header.Size = UDim2.new(1, 0, 0, 20); header.BackgroundTransparency = 1; header.Text = headerText .. ": " .. tostring(RyuConfig[targetConfigKey] or "None"); header.TextColor3 = Theme.SubText; header.Font = Enum.Font.GothamMedium; header.TextSize = 12; header.TextXAlignment = Enum.TextXAlignment.Left
    
    local searchBox = Instance.new("TextBox", frame)
    searchBox.Size = UDim2.new(1, 0, 0, 26); searchBox.Position = UDim2.new(0, 0, 0, 25); searchBox.BackgroundColor3 = Theme.ToggleOff; searchBox.TextColor3 = Theme.Text; searchBox.PlaceholderText = "Search..."; searchBox.Font = Enum.Font.Gotham; searchBox.TextSize = 12; Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 4)
    
    local scroll = Instance.new("ScrollingFrame", frame); scroll.Size = UDim2.new(1, 0, 0, 135); scroll.Position = UDim2.new(0, 0, 0, 60); scroll.BackgroundColor3 = Theme.Background; scroll.ScrollBarThickness = 4; Instance.new("UICorner", scroll).CornerRadius = UDim.new(0, 6)
    local listLayout = Instance.new("UIListLayout", scroll); listLayout.Padding = UDim.new(0, 4); listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    
    local function populate(filter)
        for _, child in pairs(scroll:GetChildren()) do if child:IsA("TextButton") then child:Destroy() end end
        for _, itemName in ipairs(itemsList) do
            if filter == "" or string.lower(itemName):find(string.lower(filter)) then
                local btn = Instance.new("TextButton", scroll); btn.Size = UDim2.new(0.94, 0, 0, 26); btn.BackgroundColor3 = Theme.SectionBG; btn.Text = "  " .. tostring(itemName); btn.TextColor3 = Theme.Text; btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
                btn.Activated:Connect(function() RyuConfig[targetConfigKey] = itemName; header.Text = headerText .. ": " .. tostring(itemName) end)
            end
        end
        task.defer(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
    end
    
    searchBox:GetPropertyChangedSignal("Text"):Connect(function() populate(searchBox.Text) end)
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10) end)
    populate("")
end

--// =======================
--// TABS & SECTIONS
--// =======================

-- TAB 1: FARM
local TabFarm = CreateMainTab("FARM")
local SubAutoLevel = CreateSubTab(TabFarm, "Auto Level")
local SecAutoLevel = CreateSection(SubAutoLevel, "Auto Leveling (Placeholder)")
local phLevel = Instance.new("TextLabel", SecAutoLevel); phLevel.Size = UDim2.new(1, 0, 0, 30); phLevel.BackgroundTransparency = 1; phLevel.Text = "Coming Soon..."; phLevel.TextColor3 = Theme.SubText; phLevel.Font = Enum.Font.Gotham; phLevel.TextSize = 12

-- TAB 2: PLAYER
local TabPlayer = CreateMainTab("PLAYER")
local SubBR = CreateSubTab(TabPlayer, "Battle Royale")
local SecBR = CreateSection(SubBR, "Battle Royale Settings")
local phBR = Instance.new("TextLabel", SecBR); phBR.Size = UDim2.new(1, 0, 0, 30); phBR.BackgroundTransparency = 1; phBR.Text = "Coming Soon..."; phBR.TextColor3 = Theme.SubText; phBR.Font = Enum.Font.Gotham; phBR.TextSize = 12

-- TAB 3: MOBILITY
local TabMobility = CreateMainTab("MOBILITY")
local SubTransport = CreateSubTab(TabMobility, "Transport")
local SubTeleport = CreateSubTab(TabMobility, "Teleport")
local SubAutoBuy = CreateSubTab(TabMobility, "Auto Buy")

local SecIslandTP = CreateSection(SubTransport, "Spider Tween (Islands)")
CreateSearchableDropdown(SecIslandTP, "Selected Island", InitIslands, "TargetIsland")
CreateSlider(SecIslandTP, "Tween Speed (Max 65)", 10, 65, 65, function(val) RyuConfig.IslandSpeed = val end)

-- TAB 4: SETTINGS
local TabSettings = CreateMainTab("SETTINGS")
local SubConfig = CreateSubTab(TabSettings, "Configs")
local SecGui = CreateSection(SubConfig, "GUI Recolour")
local phGui = Instance.new("TextLabel", SecGui); phGui.Size = UDim2.new(1, 0, 0, 30); phGui.BackgroundTransparency = 1; phGui.Text = "Color Picker Coming Soon..."; phGui.TextColor3 = Theme.SubText; phGui.Font = Enum.Font.Gotham; phGui.TextSize = 12

--// =======================
--// SPIDER TWEEN LOGIC
--// =======================

CreateButton(SecIslandTP, "Start Spider Tween", function()
    if _G.RyuIsTweening then return end
    _G.RyuIsTweening = true
    
    task.spawn(function()
        local targetIslandName = RyuConfig.TargetIsland
        local island = nil
        
        -- Suche Island in Workspace.Islands
        local islandsFolder = Workspace:FindFirstChild("Islands")
        if islandsFolder then
            for _, v in pairs(islandsFolder:GetChildren()) do
                if string.lower(v.Name) == string.lower(targetIslandName) then
                    island = v; break
                end
            end
        end
        
        if not island then _G.RyuIsTweening = false; return end
        
        local targetPos
        if island:IsA("Model") then targetPos = island:GetPivot().Position
        elseif island:IsA("BasePart") then targetPos = island.Position
        else
            local part = island:FindFirstChildWhichIsA("BasePart", true)
            if part then targetPos = part.Position else _G.RyuIsTweening = false; return end
        end
        
        local char = LocalPlayer.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then _G.RyuIsTweening = false; return end

        -- Setup Hover BodyPosition
        local bp = root:FindFirstChild("RyuHover")
        if not bp then
            bp = Instance.new("BodyPosition")
            bp.Name = "RyuHover"; bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.D = 500; bp.P = 50000; bp.Parent = root
        end
        
        local climbEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("climb")
        local sprintEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("sprint")
        local footstepEvent = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("footstep")
        
        -- Sprint Remote einmalig triggern, damit Server denkt wir rennen
        if sprintEvent then pcall(function() sprintEvent:FireServer("rbxassetid://15382065457") end) end
        
        -- Helper für echten Y-Top (Von oben nach unten scannen)
        local function GetTrueTopY(x, z)
            local rParams = RaycastParams.new()
            rParams.FilterDescendantsInstances = {char, Workspace:FindFirstChild("Effects"), Workspace:FindFirstChild("Projectiles")}
            rParams.FilterType = Enum.RaycastFilterType.Exclude
            rParams.IgnoreWater = true
            
            local origin = Vector3.new(x, 3000, z)
            local hit = Workspace:Raycast(origin, Vector3.new(0, -4000, 0), rParams)
            if hit and hit.Instance.Transparency < 1 then return hit.Position.Y end
            return 0
        end

        local currentSpeed = RyuConfig.IslandSpeed
        local floorOffset = 5 -- PERMANENT 5 STUDS ÜBER ALLEM
        local lastFootstep = tick()
        
        char:SetAttribute("evading", true)
        
        while true do
            local dt = RunService.Heartbeat:Wait()
            local flatStart = Vector3.new(root.Position.X, 0, root.Position.Z)
            local flatTarget = Vector3.new(targetPos.X, 0, targetPos.Z)
            local dist = (flatTarget - flatStart).Magnitude
            
            if dist < 5 then break end -- Angekommen
            
            local moveDir = (flatTarget - flatStart).Unit
            local currentX = root.Position.X + (moveDir.X * currentSpeed * dt)
            local currentZ = root.Position.Z + (moveDir.Z * currentSpeed * dt)
            
            -- Berechne Top Y an der aktuellen und nächsten Position
            local currentGroundY = GetTrueTopY(currentX, currentZ)
            local nextGroundY = GetTrueTopY(currentX + (moveDir.X * 3), currentZ + (moveDir.Z * 3))
            
            -- Ziel Y ist der höchste Punkt unter uns + 5 Studs (SOFORT hoch, wenn wir unter was fliegen)
            local targetY = math.max(currentGroundY, nextGroundY) + floorOffset
            
            -- 1 Stud Wand Erkennung für Climb
            local rayParamsDown = RaycastParams.new()
            rayParamsDown.FilterDescendantsInstances = {char, Workspace:FindFirstChild("Effects")}
            rayParamsDown.FilterType = Enum.RaycastFilterType.Exclude
            local wallHit = Workspace:Raycast(root.Position, moveDir * 1.5, rayParamsDown)
            
            if climbEvent then
                pcall(function() climbEvent:InvokeServer(wallHit ~= nil) end)
            end
            
            -- Softes Y Lerping (Zack-Rauf wenn starke Kante)
            local newY = root.Position.Y
            if newY < targetY then
                newY = math.min(newY + (200 * dt), targetY) -- Zieht extrem schnell nach oben
            elseif newY > targetY then
                newY = math.max(newY - (200 * dt), targetY)
            end
            
            local finalPos = Vector3.new(currentX, newY, currentZ)
            bp.Position = finalPos
            root.CFrame = CFrame.lookAt(root.Position, Vector3.new(targetPos.X, root.Position.Y, targetPos.Z))
            
            -- Lauf Animation triggern (Optisch)
            if hum then hum:Move(moveDir, false) end
            
            -- Footstep Remote spamen
            if tick() - lastFootstep > 0.35 then
                lastFootstep = tick()
                if footstepEvent then pcall(function() footstepEvent:FireServer() end) end
            end
        end
        
        if hum then hum:Move(Vector3.new(0,0,0), false) end
        if climbEvent then pcall(function() climbEvent:InvokeServer(false) end) end
        if bp then bp:Destroy() end
        char:SetAttribute("evading", nil)
        
        _G.RyuIsTweening = false
    end)
end)

-- Öffne ersten Tab standardmäßig
task.spawn(function()
    Tabs[3].Btn.MouseButton1Click:Fire() -- Öffnet Mobility
    Tabs[3].SubTabs[1].Btn.MouseButton1Click:Fire() -- Öffnet Transport
end)
