--// ==========================================
--// RYUHUB MOBILE REMOTE SPY (SOLO TEST)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "MobileRspyTest" then v:Destroy() end end

--// GLOBALS
_G.SpyEnabled = false
local LoggedRemotes = {}

--// =======================
--// 1. UI BUILDER
--// =======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MobileRspyTest"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = guiParent

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 250)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true 
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", MainFrame).Color = Color3.fromRGB(60, 60, 65)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, -80, 0, 30)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "MOBILE REMOTE SPY"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left

local ToggleBtn = Instance.new("TextButton", MainFrame)
ToggleBtn.Size = UDim2.new(0, 60, 0, 20)
ToggleBtn.Position = UDim2.new(1, -95, 0, 5)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
ToggleBtn.Text = "OFF"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 12
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 4)

local CloseBtn = Instance.new("TextButton", MainFrame)
CloseBtn.Size = UDim2.new(0, 20, 0, 20)
CloseBtn.Position = UDim2.new(1, -25, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)

local ClearBtn = Instance.new("TextButton", MainFrame)
ClearBtn.Size = UDim2.new(1, -20, 0, 25)
ClearBtn.Position = UDim2.new(0, 10, 1, -35)
ClearBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
ClearBtn.Text = "CLEAR LOGS"
ClearBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
ClearBtn.Font = Enum.Font.GothamBold
ClearBtn.TextSize = 12
Instance.new("UICorner", ClearBtn).CornerRadius = UDim.new(0, 4)

local Scroll = Instance.new("ScrollingFrame", MainFrame)
Scroll.Size = UDim2.new(1, -20, 1, -75)
Scroll.Position = UDim2.new(0, 10, 0, 35)
Scroll.BackgroundTransparency = 1
Scroll.ScrollBarThickness = 4
Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)

local ListLayout = Instance.new("UIListLayout", Scroll)
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
ListLayout.Padding = UDim.new(0, 5)

--// =======================
--// 2. LOGGING LOGIC
--// =======================
local function FormatArgs(args)
    local str = "{"
    for i, v in pairs(args) do
        if type(v) == "string" then
            str = str .. '"' .. tostring(v) .. '"'
        elseif type(v) == "userdata" then
            str = str .. typeof(v) .. ".new(" .. tostring(v) .. ")"
        else
            str = str .. tostring(v)
        end
        if i < #args then str = str .. ", " end
    end
    return str .. "}"
end

local function GetHierarchy(obj)
    local path = ""
    local current = obj
    while current and current.Parent and current ~= game do
        local name = current.Name
        if string.find(name, " ") or not string.match(name, "^[%w_]+$") then
            path = '["' .. name .. '"]' .. (path ~= "" and "." .. path or "")
        else
            path = name .. (path ~= "" and "." .. path or "")
        end
        current = current.Parent
    end
    return "game." .. path
end

local function AddLogToUI(method, remote, args)
    local argString = FormatArgs(args)
    local pathString = GetHierarchy(remote)
    
    local FinalScript = pathString .. ":" .. method .. "(unpack(" .. argString .. "))"
    
    local LogFrame = Instance.new("Frame", Scroll)
    LogFrame.Size = UDim2.new(1, 0, 0, 40)
    LogFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    Instance.new("UICorner", LogFrame).CornerRadius = UDim.new(0, 4)
    
    local NameLabel = Instance.new("TextLabel", LogFrame)
    NameLabel.Size = UDim2.new(1, -60, 1, 0)
    NameLabel.Position = UDim2.new(0, 5, 0, 0)
    NameLabel.BackgroundTransparency = 1
    NameLabel.Text = remote.Name .. " (" .. method .. ")"
    NameLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    NameLabel.Font = Enum.Font.GothamMedium
    NameLabel.TextSize = 12
    NameLabel.TextXAlignment = Enum.TextXAlignment.Left
    NameLabel.TextWrapped = true
    
    local CopyBtn = Instance.new("TextButton", LogFrame)
    CopyBtn.Size = UDim2.new(0, 50, 0, 24)
    CopyBtn.Position = UDim2.new(1, -55, 0.5, -12)
    CopyBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
    CopyBtn.Text = "COPY"
    CopyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CopyBtn.Font = Enum.Font.GothamBold
    CopyBtn.TextSize = 11
    Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 4)
    
    CopyBtn.MouseButton1Click:Connect(function()
        if setclipboard then
            pcall(function() setclipboard(FinalScript) end)
            CopyBtn.Text = "COPIED"
            CopyBtn.BackgroundColor3 = Color3.fromRGB(155, 89, 182)
            task.wait(1)
            CopyBtn.Text = "COPY"
            CopyBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        end
    end)
    
    Scroll.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y)
end

--// =======================
--// 3. HOOKING
--// =======================
if not _G.SpyInit then
    _G.SpyInit = true
    local mt = getrawmetatable(game)
    local oldNamecall = mt.__namecall
    setreadonly(mt, false)

    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        local args = {...}

        if _G.SpyEnabled and not checkcaller() then
            if method == "FireServer" or method == "InvokeServer" then
                -- Ignoriere Spam-Remotes (Movement etc.)
                if self.Name ~= "CharacterSoundEvent" and self.Name ~= "MousePos" then
                    task.spawn(AddLogToUI, method, self, args)
                end
            end
        end

        return oldNamecall(self, ...)
    end)
    setreadonly(mt, true)
end

--// =======================
--// 4. BUTTON EVENTS
--// =======================
ToggleBtn.MouseButton1Click:Connect(function()
    _G.SpyEnabled = not _G.SpyEnabled
    if _G.SpyEnabled then
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        ToggleBtn.Text = "ON"
    else
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
        ToggleBtn.Text = "OFF"
    end
end)

ClearBtn.MouseButton1Click:Connect(function()
    for _, v in pairs(Scroll:GetChildren()) do
        if v:IsA("Frame") then v:Destroy() end
    end
    Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
end)

CloseBtn.MouseButton1Click:Connect(function()
    _G.SpyEnabled = false
    ScreenGui:Destroy()
end)
