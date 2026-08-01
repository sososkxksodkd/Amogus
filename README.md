--// ============================================================================
--// RYU HUB PREMIUM + : ENGINE HOOK & BYPASS CORE
--// Basierend auf GPO Source Decompiles & Dex Logs
--// ============================================================================

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer
local Events = ReplicatedStorage:WaitForChild("Events")

--// 1. METAMETHOD HOOKING (Anti-Stamina Drain & TP-Bypass)
local oldNamecall
oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    local args = {...}

    if not checkcaller() then
        -- Blockiert den Ausdauer-Abzug beim Dashen/Geppo exakt nach deinem Log
        if method == "FireServer" and self.Name == "takestam" then
            return -- Spoof: Server erfährt nie von unserem Stamina-Verbrauch
        end
        
        -- Blockiert unerwünschte Kamera-Shakes (aus MovementPhysics.lua)
        if method == "FireServer" and self.Name == "CameraShaker" then
            return 
        end
    end

    return oldNamecall(self, ...)
end)

--// 2. GOD-MODE & ANTI-STUN LOOP (Aus Movements.lua & MeleeScript.lua)
RunService.Stepped:Connect(function()
    -- Überschreibe die GPO Globals, damit wir immer angreifen können
    if _G.blocking ~= nil then _G.blocking = false end
    if _G.grounded ~= nil then _G.grounded = false end
    if _G.canuse ~= nil then _G.canuse = true end
    if _G.canM1 ~= nil then _G.canM1 = true end
    if _G.midM1 ~= nil then _G.midM1 = false end
    if _G.isRagdolled ~= nil then _G.isRagdolled = false end
    
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            -- Verhindert serverseitiges Hinfallen (Ragdoll-State-Lock)
            hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            
            -- Verhindert den WalkSpeed-Drop bei Low HP (aus MovementPhysics)
            if hum.WalkSpeed < 16 and hum.WalkSpeed > 0 then
                hum.WalkSpeed = 16 
            end
        end
        
        -- Zerstört alle Stun-Tags, die das Script "Movements" sucht
        local badTags = {"Dizzed", "Stun", "frozen", "PB", "Rag", "Cuffed", "inCombat"}
        for _, tag in pairs(badTags) do
            local found = char:FindFirstChild(tag)
            if found then found:Destroy() end
        end
    end
end)

--// 3. FAST ATTACK & ANIMATION CANCELLING (Aus InputCallbacks & CombatRegister)
local RyuCombat = {}
function RyuCombat:ExecuteM1()
    pcall(function()
        _G.HoldingM1 = true 
        
        -- Klick-Simulation
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
        task.wait(0.01)
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)

        -- Feuert das exakte CombatRegister, das du geloggt hast, um Hits zu erzwingen
        local dashAnim = ReplicatedStorage:WaitForChild("CombatAnimations"):WaitForChild("Melee"):WaitForChild("Dash")
        Events.CombatRegister:InvokeServer({
            "swingsfx", "Melee", 1, "Ground", true, dashAnim, 2, 1.5
        })
        
        -- Endlag-Cancel (Bricht die Angriffs-Verzögerung ab, gefunden in InputCallbacks)
        Events.CombatRegister:InvokeServer({Type = "M1Endlag"})
    end)
end

--// 4. LEGITIMATE MOVEMENT SPOOFER (Dein Sprint Log)
local RyuMotion = {}
function RyuMotion:SyncServer()
    pcall(function()
        Events.sprint:FireServer("rbxassetid://15382065457")
        Events.footstep:FireServer()
    end)
end

--// 5. SAFE QUEST SYSTEM (Dein Quest Log)
local RyuQuest = {}
function RyuQuest:Take(npcName)
    pcall(function()
        Events.Quest:InvokeServer({"npcChat", true})
        Events.Quest:InvokeServer({"takequest", "Help " .. npcName})
        Events.Quest:InvokeServer({"takequest", npcName})
        Events.Quest:InvokeServer({"takequest"})
        Events.Quest:InvokeServer({"acceptquest"})
    end)
end

--// BEISPIEL FÜR DIE NEUE FARM LOGIK:
-- Anstatt riesige Tweens zu nutzen, teleportierst du dich in kleinen, schnellen Schritten
-- und rufst nach jedem Schritt RyuMotion:SyncServer() auf, damit das Anti-Cheat denkt,
-- du sprintest ganz normal mit vollem Stamina dorthin.
