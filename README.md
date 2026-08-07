--// ==========================================
--// GPO SOLO TEST GUI (NOCLIP & ANTI-CHEAT BYPASS)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "GPOSoloTest" then v:Destroy() end end

--// GLOBALS FÜR TOGGLES
_G.NoclipEnabled = false
_G.AntiCheatBypassEnabled = false

--// =======================
--// 1. NOCLIP LOGIC
--// =======================
local noclipConnection = nil
local function ToggleNoclip(state)
    _G.NoclipEnabled = state
    if state then
        if noclipConnection then noclipConnection:Disconnect() end
        noclipConnection = RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        -- Reset Collisions
        local char = LocalPlayer.Character
        if char then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") and (part.Name == "HumanoidRootPart" or part.Name == "Torso" or part.Name == "UpperTorso" or part.Name == "Head") then
                    part.CanCollide = true
                end
            end
        end
    end
end

--// =======================
--// 2. ANTI-CHEAT BYPASS LOGIC (Namecall Hook)
--// =======================
-- Dieser Hook fängt die Reports des ClientMovers ab, bevor sie den Server erreichen.
if not _G.HookInit then
    _G.HookInit = true
    local mt = getrawmetatable(game)
    local oldNamecall = mt.__namecall
    setreadonly(mt, false)

    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        local args = {...}

        if _G.AntiCheatBypassEnabled and method == "FireServer" then
            -- Blockiere das spezifische Remote Event
            if tostring(self) == "3c014433-4ff2-47e9-b83e-ec7c0bdd8f64" then
                return -- Blockiert!
            end
            
            -- Generischer Check, falls sich die UUID ändert, wir filtern nach "ClientMover"
            if type(args[1]) == "table" and args[1].Loader then
                if tostring(args[1].Loader) == "ClientMover" then
                    return -- Blockiert!
                end
            end
        end

        return oldNamecall(self, ...)
    end)
    setreadonly(mt, true)
end

--// =======================
--// 3. SOLO GUI BUILDER
--// =======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GPOSoloTest"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = guiParent

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 160)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -80)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true -- Einfaches Dragging für das Test-GUI
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner", MainFrame)
UICorner.CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", MainFrame).Color = Color3.fromRGB(60, 60, 65)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = " GPO TEST: Noclip & Bypass"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", MainFrame)
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -30, 0, 0)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    ToggleNoclip(false)
    _G.AntiCheatBypassEnabled = false
end)

local function CreateTestToggle(yPos, text, callback)
    local frame = Instance.new("Frame", MainFrame)
    frame.Size = UDim2.new(1, -20, 0, 40)
    frame.Position = UDim2.new(0, 10, 0, yPos)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 33)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left

    local toggleBtn = Instance.new("TextButton", frame)
    toggleBtn.Size = UDim2.new(0, 40, 0, 20)
    toggleBtn.Position = UDim2.new(1, -50, 0.5, -10)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
    toggleBtn.Text = ""
    Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(1, 0)

    local circle = Instance.new("Frame", toggleBtn)
    circle.Size = UDim2.new(0, 16, 0, 16)
    circle.Position = UDim2.new(0, 2, 0.5, -8)
    circle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)

    local state = false
    toggleBtn.MouseButton1Click:Connect(function()
        state = not state
        if state then
            toggleBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113) -- Grün
            circle.Position = UDim2.new(1, -18, 0.5, -8)
            circle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            label.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 55) -- Grau
            circle.Position = UDim2.new(0, 2, 0.5, -8)
            circle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
            label.TextColor3 = Color3.fromRGB(200, 200, 200)
        end
        callback(state)
    end)
end

-- Toggles erstellen
CreateTestToggle(40, "Enable AC Bypass", function(state)
    _G.AntiCheatBypassEnabled = state
end)

CreateTestToggle(90, "Enable Noclip", function(state)
    ToggleNoclip(state)
end)

local Warning = Instance.new("TextLabel", MainFrame)
Warning.Size = UDim2.new(1, 0, 0, 20)
Warning.Position = UDim2.new(0, 0, 1, -25)
Warning.BackgroundTransparency = 1
Warning.Text = "Turn on AC Bypass BEFORE Noclip!"
Warning.TextColor3 = Color3.fromRGB(255, 180, 50)
Warning.Font = Enum.Font.Gotham
Warning.TextSize = 11
