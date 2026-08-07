--// ==========================================
--// GPO SOLO TEST: SPIDER-SKEDADDLE TRANSPORT
--// ==========================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

--// GUI CLEANUP
local guiParent = LocalPlayer:WaitForChild("PlayerGui")
pcall(function() if gethui then guiParent = gethui() elseif syn and syn.protect_gui then guiParent = CoreGui end end)
for _, v in pairs(guiParent:GetChildren()) do if v.Name == "GPOSpiderTest" then v:Destroy() end end

--// GLOBALS
_G.IsSpiderTransporting = false

--// =======================
--// SPIDER TRANSPORT LOGIC
--// =======================
local function SpiderScedaddleTransport(distance)
    if _G.IsSpiderTransporting then return end
    _G.IsSpiderTransporting = true

    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then _G.IsSpiderTransporting = false; return end

    local events = ReplicatedStorage:FindFirstChild("Events")
    local targetPos = root.Position + (root.CFrame.LookVector * distance)
    
    -- 1. REMOTE SPAMMER LOOP (Asynchron, damit das Game nicht crasht)
    task.spawn(function()
        while _G.IsSpiderTransporting do
            task.wait() -- So schnell es geht ohne Crash
            if events then
                -- Climb Remote (Permanent an)
                task.spawn(function() pcall(function() events.climb:InvokeServer(true) end) end)
                
                -- Skedaddle Skill Remote
                task.spawn(function() pcall(function() events.Skill:InvokeServer("Skedaddle") end) end)
                
                -- Run / Sprint Remote (GPO nutzt oft Events.Sprint oder ahnliches)
                task.spawn(function() pcall(function() 
                    if events:FindFirstChild("Sprint") then events.Sprint:FireServer(true) end 
                end) end)
                
                -- Fall Emote / Animation erzwingen
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.Freefall) end)
            end
        end
        
        -- Cleanup nach dem Transport
        if events then pcall(function() events.climb:InvokeServer(false) end) end
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
    end)

    -- 2. SPIDER LERP MOVEMENT
    task.spawn(function()
        -- BodyPosition für Stabilität in der Luft
        local bp = root:FindFirstChild("SpiderHover") or Instance.new("BodyPosition")
        bp.Name = "SpiderHover"
        bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bp.P = 50000
        bp.D = 500
        bp.Parent = root
        
        while _G.IsSpiderTransporting do
            local dist = (root.Position - targetPos).Magnitude
            if dist < 5 then break end

            local dt = RunService.Heartbeat:Wait()
            local dir = (targetPos - root.Position).Unit
            
            -- Spider Lerp: Nutzt Lerp statt simpler Addition für "krabbelndes" Movement
            local nextPos = root.Position:Lerp(root.Position + dir * 50, dt * 1.5)
            
            -- Charakter wild rotieren für den Spider-Look (optional, verwirrt Hitbox zusätzlich)
            -- root.CFrame = CFrame.lookAt(nextPos, nextPos + dir) * CFrame.Angles(math.rad(math.random(-45, 45)), 0, 0)
            
            root.CFrame = CFrame.lookAt(nextPos, nextPos + dir)
            bp.Position = nextPos
            root.Velocity = Vector3.new(0,0,0)
            root.RotVelocity = Vector3.new(0,0,0)
        end
        
        _G.IsSpiderTransporting = false
        bp:Destroy()
    end)
end

--// =======================
--// SOLO GUI BUILDER
--// =======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GPOSpiderTest"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = guiParent

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 110)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -55)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", MainFrame).Color = Color3.fromRGB(150, 50, 255)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = " GPO: SPIDER TRANSPORT"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 13
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
    _G.IsSpiderTransporting = false
    ScreenGui:Destroy()
end)

local RunBtn = Instance.new("TextButton", MainFrame)
RunBtn.Size = UDim2.new(0.9, 0, 0, 40)
RunBtn.Position = UDim2.new(0.05, 0, 0, 45)
RunBtn.BackgroundColor3 = Color3.fromRGB(120, 50, 200)
RunBtn.Text = "TEST (FLY 150 STUDS FWD)"
RunBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RunBtn.Font = Enum.Font.GothamBold
RunBtn.TextSize = 12
Instance.new("UICorner", RunBtn).CornerRadius = UDim.new(0, 6)

RunBtn.MouseButton1Click:Connect(function()
    if _G.IsSpiderTransporting then
        _G.IsSpiderTransporting = false
        RunBtn.Text = "TEST (FLY 150 STUDS FWD)"
        RunBtn.BackgroundColor3 = Color3.fromRGB(120, 50, 200)
    else
        RunBtn.Text = "STOP TRANSPORT"
        RunBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        SpiderScedaddleTransport(150)
        
        -- Reset Button Text when done
        task.spawn(function()
            while _G.IsSpiderTransporting do task.wait(0.1) end
            RunBtn.Text = "TEST (FLY 150 STUDS FWD)"
            RunBtn.BackgroundColor3 = Color3.fromRGB(120, 50, 200)
        end)
    end
end)
