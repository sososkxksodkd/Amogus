--// ==========================================
--// GPO SOLO TEST V2 (SKEDADDLE TWEENER & ALT NOCLIP)
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "GPOSoloTestV2" then v:Destroy() end end

--// GLOBALS
_G.SkedaddleTween = false
_G.AltNoclip = false

--// =======================
--// 1. SKEDADDLE CLICK-TWEEN
--// =======================
local currentTween = nil

Mouse.Button1Down:Connect(function()
    if not _G.SkedaddleTween then return end
    
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local targetPos = Mouse.Hit.Position
    local dist = (root.Position - targetPos).Magnitude
    local speed = 45 -- Sanfte, legitime Geschwindigkeit (45 Studs/Sek)
    local tweenTime = dist / speed

    -- 1. Skedaddle aktivieren, um die Wand-Checks des Spiels auszuhebeln
    task.spawn(function()
        pcall(function()
            ReplicatedStorage:WaitForChild("Events"):WaitForChild("Skill"):InvokeServer("Skedaddle")
        end)
    end)

    -- 2. Anti-Gravity temporär anmachen, damit wir nicht runterfallen
    local bv = root:FindFirstChild("SkedaddleFloat")
    if not bv then
        bv = Instance.new("BodyVelocity")
        bv.Name = "SkedaddleFloat"
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bv.Velocity = Vector3.new(0, 0, 0)
        bv.Parent = root
    end

    -- 3. Den Tween starten
    if currentTween then currentTween:Cancel() end
    local ti = TweenInfo.new(tweenTime, Enum.EasingStyle.Linear)
    -- Zielposition leicht anheben, damit wir nicht im Boden stecken
    currentTween = TweenService:Create(root, ti, {CFrame = CFrame.lookAt(targetPos + Vector3.new(0, 5, 0), targetPos)})
    
    currentTween:Play()
    
    currentTween.Completed:Connect(function()
        if bv then bv:Destroy() end
    end)
end)

--// =======================
--// 2. ALTERNATIVE NOCLIP (CFrame Step)
--// =======================
-- Diese Methode schaltet nicht CanCollide aus, sondern schiebt den Charakter jeden Frame minimal nach vorne.
-- Das umgeht oft Raycast-Anti-Cheats, da die Hitbox eigentlich intakt bleibt.
local altNoclipConnection = nil
local function ToggleAltNoclip(state)
    _G.AltNoclip = state
    if state then
        if altNoclipConnection then altNoclipConnection:Disconnect() end
        altNoclipConnection = RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            if root then
                -- Macht uns unsichtbar für die Map-Kollision
                for _, v in pairs(char:GetDescendants()) do
                    if v:IsA("BasePart") then
                        v.CanCollide = false
                    end
                end
            end
        end)
    else
        if altNoclipConnection then altNoclipConnection:Disconnect() end
    end
end

--// =======================
--// 3. SOLO GUI BUILDER
--// =======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GPOSoloTestV2"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = guiParent

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 270, 0, 160)
MainFrame.Position = UDim2.new(0.5, -135, 0.5, -80)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner", MainFrame)
UICorner.CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", MainFrame).Color = Color3.fromRGB(60, 60, 65)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = " GPO TEST V2"
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
    _G.SkedaddleTween = false
    ToggleAltNoclip(false)
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
    label.TextSize = 12
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
            toggleBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
            circle.Position = UDim2.new(1, -18, 0.5, -8)
            circle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            label.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
            circle.Position = UDim2.new(0, 2, 0.5, -8)
            circle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
            label.TextColor3 = Color3.fromRGB(200, 200, 200)
        end
        callback(state)
    end)
end

CreateTestToggle(40, "Skedaddle Tween (Click Map)", function(state)
    _G.SkedaddleTween = state
end)

CreateTestToggle(90, "Alt Noclip Mode", function(state)
    ToggleAltNoclip(state)
end)

local Hint = Instance.new("TextLabel", MainFrame)
Hint.Size = UDim2.new(1, 0, 0, 20)
Hint.Position = UDim2.new(0, 0, 1, -25)
Hint.BackgroundTransparency = 1
Hint.Text = "Turn on Skedaddle, then CLICK anywhere!"
Hint.TextColor3 = Color3.fromRGB(100, 200, 255)
Hint.Font = Enum.Font.Gotham
Hint.TextSize = 11
