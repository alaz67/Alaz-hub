-- ╔══════════════════════════════════════════════════════════════╗
-- ║              ⚔  ALAZ HUB  ⚔                                 ║
-- ║              Steal a Brainrot Edition                        ║
-- ║              discord.gg/U4XXCxKUm                            ║
-- ╚══════════════════════════════════════════════════════════════╝

repeat task.wait() until game:IsLoaded()

local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local Lighting         = game:GetService("Lighting")
local TeleportService  = game:GetService("TeleportService")
local Player           = Players.LocalPlayer

-- ──────────────────────────────────────────────────────────────
-- STATE
-- ──────────────────────────────────────────────────────────────
local Enabled = {
    SpeedBoost       = false,
    AntiRagdoll      = false,
    AutoSteal        = false,
    AutoStealNearest = false,
    AutoStealPriority= false,
    ESP              = false,
    PlayerESP        = false,
    Fullbright       = false,
    FloorSteal       = false,
    AntiLag          = false,
    AntiBee          = false,
    Desync           = false,
    Invisible        = false,
    StealSpeed       = false,
    XrayBase         = false,
    AutoTP           = false,
    DisableAnim      = false,
}
local Values = {
    BoostSpeed  = 30,
    StealRadius = 20,
    StealSpeed  = 25,
}
local CONFIG = {
    SAFE_TP       = true,
    PREDICTIVE    = true,
    CARPET_MIDDLE = false,
}
local PRIORITY_LIST = {
    "Strawberry Elephant","Meowl","Dragon Cannelloni","Capitano Moby",
    "Cooki and Milki","La Supreme Combinasion","Burguro and Fryuro",
    "Nuclearo Dinossauro","Money Money Puggy","Garama and Madundung",
}

local guiVisible    = true
local isStealing    = false
local StealData     = {}
local espInstances  = {}
local playerESP     = {}
local Connections   = {}
local VisualSetters = {}
local floatPlatform = nil
local origTransp    = {}

-- ──────────────────────────────────────────────────────────────
-- HELPERS
-- ──────────────────────────────────────────────────────────────
local function getHRP()
    local c = Player.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end
local function getMoveDir()
    local c = Player.Character; if not c then return Vector3.zero end
    local h = c:FindFirstChildOfClass("Humanoid")
    return h and h.MoveDirection or Vector3.zero
end
local function isMyPlot(name)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(name); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then local yb=sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then return yb.Enabled end end
    return false
end
local function getGameServices()
    local rs = game:GetService("ReplicatedStorage")
    local ok, S = pcall(function()
        return {
            Sync    = require(rs:WaitForChild("Packages"):WaitForChild("Synchronizer")),
            Animals = require(rs:WaitForChild("Datas"):WaitForChild("Animals")),
            Shared  = require(rs:WaitForChild("Shared"):WaitForChild("Animals")),
            Numbers = require(rs:WaitForChild("Utils"):WaitForChild("NumberUtils")),
        }
    end)
    return ok and S or nil
end

-- ──────────────────────────────────────────────────────────────
-- SPEED BOOST
-- ──────────────────────────────────────────────────────────────
local function startSpeedBoost()
    if Connections.speed then return end
    Connections.speed = RunService.Heartbeat:Connect(function()
        if not Enabled.SpeedBoost then return end
        local h = getHRP(); if not h then return end
        local md = getMoveDir()
        if md.Magnitude > 0.1 then
            h.AssemblyLinearVelocity = Vector3.new(md.X*Values.BoostSpeed, h.AssemblyLinearVelocity.Y, md.Z*Values.BoostSpeed)
        end
    end)
end
local function stopSpeedBoost()
    if Connections.speed then Connections.speed:Disconnect(); Connections.speed = nil end
end

-- ──────────────────────────────────────────────────────────────
-- ANTI RAGDOLL
-- ──────────────────────────────────────────────────────────────
local function startAntiRagdoll()
    if Connections.antiRag then return end
    Connections.antiRag = RunService.Heartbeat:Connect(function()
        if not Enabled.AntiRagdoll then return end
        local char = Player.Character; if not char then return end
        local hum  = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if hum then
            local s = hum:GetState()
            if s==Enum.HumanoidStateType.Physics or s==Enum.HumanoidStateType.Ragdoll or s==Enum.HumanoidStateType.FallingDown then
                hum:ChangeState(Enum.HumanoidStateType.Running)
                workspace.CurrentCamera.CameraSubject = hum
                if root then root.AssemblyLinearVelocity=Vector3.zero; root.AssemblyAngularVelocity=Vector3.zero end
            end
        end
        for _, obj in ipairs(char:GetDescendants()) do
            if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled = true end
        end
    end)
end
local function stopAntiRagdoll()
    if Connections.antiRag then Connections.antiRag:Disconnect(); Connections.antiRag = nil end
end

-- ──────────────────────────────────────────────────────────────
-- INVISIBLE
-- ──────────────────────────────────────────────────────────────
local function enableInvisible()
    local char = Player.Character; if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") or part:IsA("Decal") then
            pcall(function() part.LocalTransparencyModifier = 1 end)
        end
    end
end
local function disableInvisible()
    local char = Player.Character; if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") or part:IsA("Decal") then
            pcall(function() part.LocalTransparencyModifier = 0 end)
        end
    end
end

-- ──────────────────────────────────────────────────────────────
-- STEAL SPEED DURING STEALING
-- ──────────────────────────────────────────────────────────────
local function startStealSpeed()
    if Connections.stealSpeed then return end
    Connections.stealSpeed = RunService.Heartbeat:Connect(function()
        if not Enabled.StealSpeed then return end
        if not Player:GetAttribute("Stealing") then return end
        local h = getHRP(); if not h then return end
        local md = getMoveDir()
        if md.Magnitude > 0.1 then
            h.AssemblyLinearVelocity = Vector3.new(md.X*Values.StealSpeed, h.AssemblyLinearVelocity.Y, md.Z*Values.StealSpeed)
        end
    end)
end
local function stopStealSpeed()
    if Connections.stealSpeed then Connections.stealSpeed:Disconnect(); Connections.stealSpeed = nil end
end

-- ──────────────────────────────────────────────────────────────
-- DISABLE ANIMATION DURING STEAL
-- ──────────────────────────────────────────────────────────────
local savedAnims = {}
local function startDisableAnim()
    if Connections.disableAnim then return end
    Connections.disableAnim = RunService.Heartbeat:Connect(function()
        if not Enabled.DisableAnim then return end
        if not Player:GetAttribute("Stealing") then return end
        local char = Player.Character; if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid"); if not hum then return end
        for _, t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop(0) end
    end)
end
local function stopDisableAnim()
    if Connections.disableAnim then Connections.disableAnim:Disconnect(); Connections.disableAnim = nil end
end

-- ──────────────────────────────────────────────────────────────
-- AUTO STEAL
-- ──────────────────────────────────────────────────────────────
local function getAllAnimals()
    local plots = workspace:FindFirstChild("Plots"); if not plots then return {} end
    local S = getGameServices(); if not S then return {} end
    local animals = {}
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlot(plot.Name) then continue end
        pcall(function()
            local ch = S.Sync:Get(plot.Name); if not ch then return end
            local list = ch:Get("AnimalList"); if not list then return end
            local pods = plot:FindFirstChild("AnimalPodiums"); if not pods then return end
            for slot, data in pairs(list) do
                if type(data)~="table" then continue end
                local val = S.Shared:GetGeneration(data.Index, data.Mutation, data.Traits, nil) or 0
                local ai  = S.Animals[data.Index]
                table.insert(animals, {
                    name  = ai and (ai.DisplayName or data.Index) or data.Index,
                    val   = val,
                    plot  = plot.Name,
                    slot  = tostring(slot),
                    data  = data,
                })
            end
        end)
    end
    table.sort(animals, function(a,b) return a.val > b.val end)
    return animals
end

local function getPodiumPart(plotName, slot)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil end
    local plot  = plots:FindFirstChild(plotName); if not plot then return nil end
    local pods  = plot:FindFirstChild("AnimalPodiums"); if not pods then return nil end
    local pod   = pods:FindFirstChild(slot); if not pod then return nil end
    return pod
end

local function findPromptForAnimal(plotName, slot)
    local pod = getPodiumPart(plotName, slot); if not pod then return nil end
    local base  = pod:FindFirstChild("Base"); if not base then return nil end
    local spawn = base:FindFirstChild("Spawn"); if not spawn then return nil end
    local att   = spawn:FindFirstChild("PromptAttachment"); if not att then return nil end
    for _, ch in ipairs(att:GetChildren()) do
        if ch:IsA("ProximityPrompt") then return ch end
    end
    return nil
end

local function findNearestPrompt()
    local h = getHRP(); if not h then return nil, nil end
    local animals = getAllAnimals()
    local np, na = nil, nil
    local nd = math.huge
    for _, animal in ipairs(animals) do
        local pod = getPodiumPart(animal.plot, animal.slot)
        if pod then
            local dist = (pod:GetPivot().Position - h.Position).Magnitude
            if dist < nd and dist <= Values.StealRadius then
                local prompt = findPromptForAnimal(animal.plot, animal.slot)
                if prompt then np=prompt; nd=dist; na=animal; break end
            end
        end
    end
    return np, na
end

local function findNearestAnimalPrompt()
    local h = getHRP(); if not h then return nil, nil end
    local animals = getAllAnimals()
    local np, na = nil, nil
    local nd = math.huge
    for _, animal in ipairs(animals) do
        local pod = getPodiumPart(animal.plot, animal.slot)
        if pod then
            local dist = (pod:GetPivot().Position - h.Position).Magnitude
            if dist < nd then
                local prompt = findPromptForAnimal(animal.plot, animal.slot)
                if prompt then np=prompt; nd=dist; na=animal; break end
            end
        end
    end
    return np, na
end

local function getPriorityPrompt()
    local animals = getAllAnimals()
    for _, pname in ipairs(PRIORITY_LIST) do
        for _, animal in ipairs(animals) do
            if animal.name:lower() == pname:lower() then
                local prompt = findPromptForAnimal(animal.plot, animal.slot)
                if prompt then return prompt, animal end
            end
        end
    end
    return nil, nil
end

local function buildSteal(prompt)
    if StealData[prompt] then return end
    local d = {hold={}, trigger={}, ready=true}
    pcall(function()
        if getconnections then
            for _,c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(d.hold, c.Function) end end
            for _,c in ipairs(getconnections(prompt.Triggered))             do if c.Function then table.insert(d.trigger, c.Function) end end
        end
    end)
    if #d.hold > 0 or #d.trigger > 0 then StealData[prompt] = d end
end

local function executeSteal(prompt)
    if not prompt or not prompt.Parent then return end
    buildSteal(prompt)
    local d = StealData[prompt]; if not d or not d.ready then return end
    d.ready = false; isStealing = true
    if Enabled.Invisible then enableInvisible() end
    task.spawn(function()
        for _,f in ipairs(d.hold) do task.spawn(f) end
        task.wait(1.3)
        for _,f in ipairs(d.trigger) do task.spawn(f) end
        if Enabled.Invisible then disableInvisible() end
        d.ready = true; isStealing = false
    end)
end

local function startAutoSteal()
    if Connections.steal then return end
    Connections.steal = RunService.Heartbeat:Connect(function()
        if not (Enabled.AutoSteal or Enabled.AutoStealNearest or Enabled.AutoStealPriority) then return end
        if isStealing then return end
        local p = nil
        if Enabled.AutoStealPriority then
            p = getPriorityPrompt()
            if not p and Enabled.AutoStealNearest then p = findNearestAnimalPrompt() end
            if not p and Enabled.AutoSteal       then p = findNearestPrompt()       end
        elseif Enabled.AutoStealNearest then
            p = findNearestAnimalPrompt()
        elseif Enabled.AutoSteal then
            p = findNearestPrompt()
        end
        if p then executeSteal(p) end
    end)
end
local function stopAutoSteal()
    if Connections.steal then Connections.steal:Disconnect(); Connections.steal = nil end
    isStealing = false
end

-- ──────────────────────────────────────────────────────────────
-- TP SYSTEM
-- ──────────────────────────────────────────────────────────────
local function tpToAnimal(animal)
    if not animal then return end
    local h = getHRP(); if not h then return end
    local pod = getPodiumPart(animal.plot, animal.slot); if not pod then return end
    local pos = pod:GetPivot().Position
    h.CFrame = CFrame.new(pos + Vector3.new(0,5,0))
end

local function tpToHighest()
    local animals = getAllAnimals()
    if #animals > 0 then tpToAnimal(animals[1]) end
end

local function tpToPriority()
    local animals = getAllAnimals()
    for _, pname in ipairs(PRIORITY_LIST) do
        for _, animal in ipairs(animals) do
            if animal.name:lower() == pname:lower() then
                tpToAnimal(animal); return
            end
        end
    end
    tpToHighest()
end

-- ──────────────────────────────────────────────────────────────
-- FLOOR STEAL
-- ──────────────────────────────────────────────────────────────
local function startFloorSteal()
    if floatPlatform then floatPlatform:Destroy() end
    floatPlatform = Instance.new("Part")
    floatPlatform.Size = Vector3.new(6,1,6)
    floatPlatform.Anchored = true; floatPlatform.CanCollide = true
    floatPlatform.Transparency = 1; floatPlatform.Parent = workspace
    task.spawn(function()
        while floatPlatform and Enabled.FloorSteal do
            local hrp = getHRP()
            if hrp then floatPlatform.Position = hrp.Position - Vector3.new(0,3,0) end
            task.wait(0.05)
        end
    end)
end
local function stopFloorSteal()
    if floatPlatform then floatPlatform:Destroy(); floatPlatform = nil end
end

-- ──────────────────────────────────────────────────────────────
-- XRAY BASE
-- ──────────────────────────────────────────────────────────────
local function enableXray()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Anchored then
            pcall(function()
                local n = obj.Name:lower()
                if n:find("wall") or n:find("base") or n:find("fence") then
                    origTransp[obj] = obj.LocalTransparencyModifier
                    obj.LocalTransparencyModifier = 0.85
                end
            end)
        end
    end
end
local function disableXray()
    for obj, val in pairs(origTransp) do
        pcall(function() obj.LocalTransparencyModifier = val end)
    end
    origTransp = {}
end

-- ──────────────────────────────────────────────────────────────
-- ANTI LAG
-- ──────────────────────────────────────────────────────────────
local function enableAntiLag()
    pcall(function()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player and p.Character then
                for _, obj in ipairs(p.Character:GetDescendants()) do
                    pcall(function()
                        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then obj.Enabled=false end
                        if obj:IsA("BasePart") then obj.CastShadow=false end
                        if obj:IsA("Accessory") then obj:Destroy() end
                    end)
                end
            end
        end
        Lighting.GlobalShadows = false
        pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
    end)
end
local function disableAntiLag()
    Lighting.GlobalShadows = true
    pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic end)
end

-- ──────────────────────────────────────────────────────────────
-- ANTI BEE / DISCO
-- ──────────────────────────────────────────────────────────────
local function enableAntiBee()
    if Connections.antiBee then return end
    Connections.antiBee = Lighting.DescendantAdded:Connect(function(obj)
        if not Enabled.AntiBee then return end
        local bad = {"Blue","DiscoEffect","BeeBlur","ColorCorrection"}
        for _, name in ipairs(bad) do
            if obj.Name == name then pcall(function() obj:Destroy() end) end
        end
    end)
    for _, obj in ipairs(Lighting:GetDescendants()) do
        local bad = {"Blue","DiscoEffect","BeeBlur"}
        for _, name in ipairs(bad) do
            if obj.Name == name then pcall(function() obj:Destroy() end) end
        end
    end
end
local function disableAntiBee()
    if Connections.antiBee then Connections.antiBee:Disconnect(); Connections.antiBee = nil end
end

-- ──────────────────────────────────────────────────────────────
-- DESYNC
-- ──────────────────────────────────────────────────────────────
local DESYNC_FLAGS = {
    LargeReplicatorEnabled9 = true,
    MaxTimestepMultiplierAcceleration = 2147483647,
    GameNetDontSendRedundantNumTimes = 1,
    PhysicsSenderMaxBandwidthBps = 20000,
    ServerMaxBandwith = 52,
    WorldStepMax = 30,
    MaxAcceptableUpdateDelay = 1,
    AngularVelociryLimit = 360,
}
local function enableDesync()
    for k, v in pairs(DESYNC_FLAGS) do
        pcall(function() setfflag(tostring(k), tostring(v)) end)
    end
end

-- ──────────────────────────────────────────────────────────────
-- FULLBRIGHT
-- ──────────────────────────────────────────────────────────────
local function enableFullbright()
    Lighting.Ambient=Color3.fromRGB(255,255,255); Lighting.Brightness=5
    Lighting.OutdoorAmbient=Color3.fromRGB(255,255,255); Lighting.GlobalShadows=false
end
local function disableFullbright()
    Lighting.Ambient=Color3.fromRGB(127,127,127); Lighting.Brightness=2
    Lighting.OutdoorAmbient=Color3.fromRGB(127,127,127); Lighting.GlobalShadows=true
end

-- ──────────────────────────────────────────────────────────────
-- ESP
-- ──────────────────────────────────────────────────────────────
local espSg = nil

local function clearESP()
    for _, r in pairs(espInstances) do
        if r.hl then pcall(function() r.hl:Destroy() end) end
        if r.bb then pcall(function() r.bb:Destroy() end) end
    end
    espInstances = {}
end

local function refreshESP(sg)
    clearESP()
    if not Enabled.ESP then return end
    local animals = getAllAnimals()
    for _, animal in ipairs(animals) do
        pcall(function()
            local pod = getPodiumPart(animal.plot, animal.slot); if not pod then return end
            local S = getGameServices(); if not S then return end
            local txt = "$" .. S.Numbers:ToString(animal.val) .. "/s"

            local hl = Instance.new("Highlight")
            hl.Adornee = pod; hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            hl.FillColor = Color3.fromRGB(0,120,255); hl.FillTransparency = 0.5
            hl.OutlineColor = Color3.fromRGB(100,200,255); hl.OutlineTransparency = 0
            hl.Parent = sg

            local bb = Instance.new("BillboardGui")
            bb.Adornee = pod; bb.AlwaysOnTop = true
            bb.Size = UDim2.new(0,220,0,60); bb.StudsOffset = Vector3.new(0,4,0); bb.Parent = sg

            local bg = Instance.new("Frame",bb)
            bg.Size=UDim2.new(1,0,1,0); bg.BackgroundColor3=Color3.fromRGB(0,0,0)
            bg.BackgroundTransparency=0.4; bg.BorderSizePixel=0
            Instance.new("UICorner",bg).CornerRadius=UDim.new(0,8)

            local nl=Instance.new("TextLabel",bg)
            nl.Size=UDim2.new(1,0,0.55,0); nl.BackgroundTransparency=1
            nl.Text=animal.name; nl.TextColor3=Color3.fromRGB(100,200,255)
            nl.Font=Enum.Font.GothamBold; nl.TextSize=14; nl.TextXAlignment=Enum.TextXAlignment.Center

            local vl=Instance.new("TextLabel",bg)
            vl.Size=UDim2.new(1,0,0.45,0); vl.Position=UDim2.new(0,0,0.55,0)
            vl.BackgroundTransparency=1; vl.Text=txt
            vl.TextColor3=Color3.fromRGB(255,255,255); vl.Font=Enum.Font.GothamBold
            vl.TextSize=12; vl.TextXAlignment=Enum.TextXAlignment.Center

            espInstances[animal.plot.."_"..animal.slot] = {hl=hl, bb=bb}
        end)
    end
end

-- PLAYER ESP
local function clearPlayerESP()
    for _, r in pairs(playerESP) do
        if r.hl then pcall(function() r.hl:Destroy() end) end
        if r.bb then pcall(function() r.bb:Destroy() end) end
    end
    playerESP = {}
end

local function refreshPlayerESP(sg)
    clearPlayerESP()
    if not Enabled.PlayerESP then return end
    for _, p in ipairs(Players:GetPlayers()) do
        if p == Player then continue end
        if not p.Character then continue end
        local hrp = p.Character:FindFirstChild("HumanoidRootPart"); if not hrp then continue end

        local hl = Instance.new("Highlight")
        hl.Adornee = p.Character; hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        hl.FillTransparency = 1; hl.OutlineColor = Color3.fromRGB(0,180,255)
        hl.OutlineTransparency = 0; hl.Parent = sg

        local bb = Instance.new("BillboardGui")
        bb.Adornee = hrp; bb.AlwaysOnTop = true
        bb.Size = UDim2.new(0,150,0,30); bb.StudsOffset = Vector3.new(0,3,0); bb.Parent = sg

        local nl = Instance.new("TextLabel",bb)
        nl.Size=UDim2.new(1,0,1,0); nl.BackgroundTransparency=1
        nl.Text=p.Name; nl.TextColor3=Color3.fromRGB(100,200,255)
        nl.Font=Enum.Font.GothamBold; nl.TextSize=14; nl.TextStrokeTransparency=0

        playerESP[p] = {hl=hl, bb=bb}
    end
end

-- ──────────────────────────────────────────────────────────────
-- NOTIFICATIONS
-- ──────────────────────────────────────────────────────────────
local notifSg = nil
local notifCount = 0

local function showNotif(msg, color)
    if not notifSg then return end
    color = color or Color3.fromRGB(30,120,255)
    notifCount = notifCount + 1
    local yOff = -70 - (notifCount-1)*75

    local frame = Instance.new("Frame", notifSg)
    frame.Size = UDim2.new(0,300,0,60)
    frame.Position = UDim2.new(1,320,1,yOff)
    frame.BackgroundColor3 = Color3.fromRGB(255,255,255)
    frame.BackgroundTransparency = 0; frame.BorderSizePixel = 0; frame.ZIndex = 999
    Instance.new("UICorner",frame).CornerRadius = UDim.new(0,12)
    local fs = Instance.new("UIStroke",frame); fs.Color = color; fs.Thickness = 2

    local bar = Instance.new("Frame",frame)
    bar.Size = UDim2.new(0,5,1,0); bar.BackgroundColor3 = color; bar.BorderSizePixel = 0
    Instance.new("UICorner",bar).CornerRadius = UDim.new(1,0)

    local lbl = Instance.new("TextLabel",frame)
    lbl.Size = UDim2.new(1,-20,1,0); lbl.Position = UDim2.new(0,14,0,0)
    lbl.BackgroundTransparency = 1; lbl.Text = msg
    lbl.TextColor3 = Color3.fromRGB(15,15,15); lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 13; lbl.TextWrapped = true; lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.ZIndex = 999

    TweenService:Create(frame, TweenInfo.new(0.3), {Position=UDim2.new(1,-310,1,yOff)}):Play()
    task.delay(3, function()
        TweenService:Create(frame, TweenInfo.new(0.3), {Position=UDim2.new(1,320,1,yOff), BackgroundTransparency=1}):Play()
        task.wait(0.35); frame:Destroy(); notifCount = math.max(0, notifCount-1)
    end)
end

-- ──────────────────────────────────────────────────────────────
-- GUI
-- ──────────────────────────────────────────────────────────────
local sg = Instance.new("ScreenGui")
sg.Name = "AlazHub"; sg.ResetOnSpawn = false
sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
sg.Parent = Player:FindFirstChildOfClass("PlayerGui") or Player.PlayerGui
notifSg = sg
espSg   = sg

local W = {
    bg        = Color3.fromRGB(255,255,255),
    card      = Color3.fromRGB(245,245,245),
    cardAlt   = Color3.fromRGB(235,235,235),
    border    = Color3.fromRGB(210,210,210),
    ink       = Color3.fromRGB(15,15,15),
    inkMid    = Color3.fromRGB(80,80,80),
    inkLight  = Color3.fromRGB(150,150,150),
    white     = Color3.fromRGB(255,255,255),
    blue      = Color3.fromRGB(30,120,255),
    blueDark  = Color3.fromRGB(10,80,200),
    blueLight = Color3.fromRGB(100,180,255),
    toggleOn  = Color3.fromRGB(30,120,255),
    toggleOff = Color3.fromRGB(200,200,200),
    success   = Color3.fromRGB(25,165,80),
    danger    = Color3.fromRGB(195,50,50),
    yellow    = Color3.fromRGB(220,160,0),
}

-- BLUE FIRE BG
local fireBg = Instance.new("Frame",sg)
fireBg.Size=UDim2.new(1,0,1,0); fireBg.BackgroundTransparency=1; fireBg.ZIndex=0

local FCOUNT=35; local fParticles={}
local function spawnFire(yO)
    local p=Instance.new("Frame",fireBg); local sz=math.random(4,18)
    p.Size=UDim2.new(0,sz,0,sz); p.BorderSizePixel=0; p.ZIndex=1
    Instance.new("UICorner",p).CornerRadius=UDim.new(1,0)
    local r=math.random(0,60); local g=math.random(80,180); local b=math.random(200,255)
    p.BackgroundColor3=Color3.fromRGB(r,g,b); p.BackgroundTransparency=math.random(30,65)/100
    local sx=math.random(0,1280); local sy=yO or math.random(600,1000)
    p.Position=UDim2.new(0,sx,0,sy)
    local d={f=p,x=sx,y=sy,vx=math.random(-12,12)*0.3,vy=-math.random(25,80),sz=sz,alpha=math.random(20,55)/100,life=0,max=math.random(20,55)/10,wig=math.random()*math.pi*2,r=r,g=g,b=b}
    table.insert(fParticles,d); return d
end
for i=1,FCOUNT do local d=spawnFire(); d.y=math.random(50,1000); d.life=math.random()*d.max end

local lastFT=tick()
RunService.RenderStepped:Connect(function()
    local now=tick(); local dt=now-lastFT; lastFT=now
    for i=#fParticles,1,-1 do
        local d=fParticles[i]
        if not d.f or not d.f.Parent then table.remove(fParticles,i); continue end
        d.life=d.life+dt; local t=d.life/d.max
        if t>=1 then d.f:Destroy(); table.remove(fParticles,i); if #fParticles<FCOUNT then spawnFire() end; continue end
        d.wig=d.wig+dt*2.8; d.x=d.x+(d.vx+math.sin(d.wig)*7)*dt; d.y=d.y+d.vy*dt
        local sT=math.max(0,(t-0.4)*1.8); local cSz=d.sz*(1-sT*0.8); local al=d.alpha*(1-t)
        d.f.Size=UDim2.new(0,cSz,0,cSz); d.f.Position=UDim2.new(0,d.x,0,d.y); d.f.BackgroundTransparency=1-al
        d.f.BackgroundColor3=Color3.fromRGB(math.min(math.floor(d.r*(1+sT*0.5)),255),math.min(math.floor(d.g*(1+sT*0.3)),255),math.min(d.b,255))
    end
end)

-- MINI TOGGLE BUTTON
local miniBtn=Instance.new("TextButton",sg)
miniBtn.Size=UDim2.new(0,48,0,48); miniBtn.Position=UDim2.new(0,12,0.5,-24)
miniBtn.BackgroundColor3=W.blue; miniBtn.BackgroundTransparency=0.1
miniBtn.Text="⚔"; miniBtn.TextColor3=W.white; miniBtn.Font=Enum.Font.GothamBlack
miniBtn.TextSize=22; miniBtn.ZIndex=999; miniBtn.BorderSizePixel=0
Instance.new("UICorner",miniBtn).CornerRadius=UDim.new(1,0)
local mS=Instance.new("UIStroke",miniBtn); mS.Color=W.blueLight; mS.Thickness=2

-- MAIN WINDOW
local main=Instance.new("Frame",sg); main.Name="Main"
main.Size=UDim2.new(0,440,0,640); main.Position=UDim2.new(1,-465,0,20)
main.BackgroundColor3=W.bg; main.BackgroundTransparency=0; main.BorderSizePixel=0
main.Active=true; main.Draggable=true; main.ClipsDescendants=true; main.ZIndex=10
Instance.new("UICorner",main).CornerRadius=UDim.new(0,16)
local mainS=Instance.new("UIStroke",main); mainS.Color=W.border; mainS.Thickness=1.5

-- HEADER
local hdr=Instance.new("Frame",main); hdr.Size=UDim2.new(1,0,0,85)
hdr.BackgroundColor3=W.blue; hdr.BorderSizePixel=0; hdr.ZIndex=11
Instance.new("UICorner",hdr).CornerRadius=UDim.new(0,16)
local hFill=Instance.new("Frame",hdr); hFill.Size=UDim2.new(1,0,0.5,0); hFill.Position=UDim2.new(0,0,0.5,0)
hFill.BackgroundColor3=W.blue; hFill.BorderSizePixel=0; hFill.ZIndex=11

-- Blue fire in header
local hFC=Instance.new("Frame",hdr); hFC.Size=UDim2.new(1,0,1,0)
hFC.BackgroundTransparency=1; hFC.ZIndex=12; hFC.ClipsDescendants=true
local hFP={}
local function spawnHFire()
    local p=Instance.new("Frame",hFC); local sz=math.random(2,10)
    p.Size=UDim2.new(0,sz,0,sz); p.BorderSizePixel=0; p.ZIndex=12
    Instance.new("UICorner",p).CornerRadius=UDim.new(1,0)
    local r=math.random(0,50); local g=math.random(100,200); local b=math.random(220,255)
    p.BackgroundColor3=Color3.fromRGB(r,g,b); p.BackgroundTransparency=math.random(25,55)/100
    local d={f=p,x=math.random(0,440),y=math.random(60,85),vx=math.random(-5,5)*0.4,vy=-math.random(14,40),sz=sz,life=math.random()*2.5,max=math.random(10,28)/10,wig=math.random()*math.pi*2,r=r,g=g,b=b}
    p.Position=UDim2.new(0,d.x,0,d.y); table.insert(hFP,d)
end
for i=1,22 do spawnHFire() end
local lastHFT=tick()
RunService.RenderStepped:Connect(function()
    local now=tick(); local dt=now-lastHFT; lastHFT=now
    for i=#hFP,1,-1 do
        local d=hFP[i]; if not d.f or not d.f.Parent then table.remove(hFP,i); continue end
        d.life=d.life+dt; local t=d.life/d.max
        if t>=1 then d.f:Destroy(); table.remove(hFP,i); spawnHFire(); continue end
        d.wig=d.wig+dt*3; d.x=d.x+(d.vx+math.sin(d.wig)*3)*dt; d.y=d.y+d.vy*dt
        local sT=math.max(0,(t-0.35)*1.6); local cSz=d.sz*(1-sT*0.85); local al=(1-t)*0.7
        d.f.Size=UDim2.new(0,cSz,0,cSz); d.f.Position=UDim2.new(0,d.x,0,d.y); d.f.BackgroundTransparency=1-al
        d.f.BackgroundColor3=Color3.fromRGB(math.min(d.r+math.floor(sT*60),255),math.min(d.g+math.floor(sT*40),255),math.min(d.b,255))
    end
end)

local titleLbl=Instance.new("TextLabel",hdr)
titleLbl.Size=UDim2.new(1,-50,0,42); titleLbl.Position=UDim2.new(0,0,0.06,0)
titleLbl.BackgroundTransparency=1; titleLbl.Text="⚔  ALAZ HUB  ⚔"
titleLbl.TextColor3=W.white; titleLbl.Font=Enum.Font.GothamBlack; titleLbl.TextSize=26
titleLbl.TextXAlignment=Enum.TextXAlignment.Center; titleLbl.ZIndex=14

local subLbl=Instance.new("TextLabel",hdr)
subLbl.Size=UDim2.new(1,0,0,18); subLbl.Position=UDim2.new(0,0,0.65,0)
subLbl.BackgroundTransparency=1; subLbl.Text="Steal a Brainrot  •  discord.gg/U4XXCxKUm"
subLbl.TextColor3=Color3.fromRGB(180,220,255); subLbl.Font=Enum.Font.GothamBold
subLbl.TextSize=11; subLbl.TextXAlignment=Enum.TextXAlignment.Center; subLbl.ZIndex=14

local closeBtn=Instance.new("TextButton",hdr)
closeBtn.Size=UDim2.new(0,32,0,32); closeBtn.Position=UDim2.new(1,-42,0.5,-16)
closeBtn.BackgroundColor3=Color3.fromRGB(30,60,150); closeBtn.Text="✕"
closeBtn.TextColor3=W.white; closeBtn.Font=Enum.Font.GothamBold; closeBtn.TextSize=14; closeBtn.ZIndex=15
Instance.new("UICorner",closeBtn).CornerRadius=UDim.new(1,0)
closeBtn.MouseButton1Click:Connect(function() sg:Destroy() end)
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.18),{BackgroundColor3=W.danger}):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.18),{BackgroundColor3=Color3.fromRGB(30,60,150)}):Play() end)

-- TABS
local tabCont=Instance.new("Frame",main)
tabCont.Size=UDim2.new(1,-22,0,38); tabCont.Position=UDim2.new(0,11,0,90)
tabCont.BackgroundTransparency=1; tabCont.ZIndex=11

local tabs={"STEAL","MOVEMENT","VISUALS","MISC"}
local tabButtons={}

local function mkPanel(cv)
    local p=Instance.new("ScrollingFrame",main)
    p.Size=UDim2.new(1,-20,0,475); p.Position=UDim2.new(0,10,0,136)
    p.BackgroundTransparency=1; p.BorderSizePixel=0
    p.CanvasSize=UDim2.new(0,0,0,cv); p.ScrollBarThickness=4
    p.ScrollBarImageColor3=W.border; p.ClipsDescendants=true
    p.ZIndex=11; p.Visible=false; return p
end

local stealPanel    = mkPanel(700); stealPanel.Visible=true
local movementPanel = mkPanel(500)
local visualsPanel  = mkPanel(500)
local miscPanel     = mkPanel(500)

local function createTab(name, xp)
    local btn=Instance.new("TextButton",tabCont)
    btn.Size=UDim2.new(0,98,0,32); btn.Position=UDim2.new(0,xp,0.5,-16)
    btn.BackgroundColor3=W.cardAlt; btn.Text=name
    btn.TextColor3=W.inkMid; btn.Font=Enum.Font.GothamBold; btn.TextSize=11; btn.ZIndex=12
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,10)
    local bs=Instance.new("UIStroke",btn); bs.Color=W.border; bs.Thickness=1
    btn.MouseButton1Click:Connect(function()
        for _,b in pairs(tabButtons) do
            TweenService:Create(b,TweenInfo.new(0.14),{BackgroundColor3=W.cardAlt,TextColor3=W.inkMid}):Play()
            local s=b:FindFirstChildOfClass("UIStroke"); if s then s.Color=W.border end
        end
        TweenService:Create(btn,TweenInfo.new(0.14),{BackgroundColor3=W.blue,TextColor3=W.white}):Play()
        local s=btn:FindFirstChildOfClass("UIStroke"); if s then s.Color=W.blue end
        stealPanel.Visible=(name=="STEAL"); movementPanel.Visible=(name=="MOVEMENT")
        visualsPanel.Visible=(name=="VISUALS"); miscPanel.Visible=(name=="MISC")
    end)
    table.insert(tabButtons,btn); return btn
end
for i,name in ipairs(tabs) do
    local btn=createTab(name,(i-1)*102)
    if name=="STEAL" then btn.BackgroundColor3=W.blue; btn.TextColor3=W.white; local s=btn:FindFirstChildOfClass("UIStroke"); if s then s.Color=W.blue end end
end

-- TOGGLE FACTORY
local function mkToggle(parent, yp, title, desc, key, cb)
    local cont=Instance.new("Frame",parent)
    cont.Size=UDim2.new(1,-6,0,58); cont.Position=UDim2.new(0,3,0,yp)
    cont.BackgroundColor3=W.card; cont.BackgroundTransparency=0; cont.BorderSizePixel=0
    cont.ClipsDescendants=true; cont.ZIndex=12
    Instance.new("UICorner",cont).CornerRadius=UDim.new(0,11)
    local cs=Instance.new("UIStroke",cont); cs.Color=W.border; cs.Thickness=1

    local tl=Instance.new("TextLabel",cont)
    tl.Size=UDim2.new(0.65,0,0,20); tl.Position=UDim2.new(0,12,0,8)
    tl.BackgroundTransparency=1; tl.Text=title; tl.TextColor3=W.ink
    tl.Font=Enum.Font.GothamBold; tl.TextSize=13; tl.TextXAlignment=Enum.TextXAlignment.Left; tl.ZIndex=13

    local dl=Instance.new("TextLabel",cont)
    dl.Size=UDim2.new(0.65,0,0,14); dl.Position=UDim2.new(0,12,0,30)
    dl.BackgroundTransparency=1; dl.Text=desc; dl.TextColor3=W.inkLight
    dl.Font=Enum.Font.Gotham; dl.TextSize=10; dl.TextXAlignment=Enum.TextXAlignment.Left; dl.ZIndex=13

    local defOn=Enabled[key] or false
    local tb=Instance.new("Frame",cont)
    tb.Size=UDim2.new(0,50,0,24); tb.Position=UDim2.new(1,-62,0.5,-12)
    tb.BackgroundColor3=defOn and W.toggleOn or W.toggleOff; tb.BorderSizePixel=0; tb.ZIndex=13
    Instance.new("UICorner",tb).CornerRadius=UDim.new(1,0)

    local knob=Instance.new("Frame",tb)
    knob.Size=UDim2.new(0,18,0,18)
    knob.Position=defOn and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)
    knob.BackgroundColor3=W.white; knob.BorderSizePixel=0; knob.ZIndex=14
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)

    local clk=Instance.new("TextButton",cont)
    clk.Size=UDim2.new(1,0,1,0); clk.BackgroundTransparency=1; clk.Text=""; clk.ZIndex=15

    local isOn=defOn
    local function sv(state, skip)
        isOn=state
        TweenService:Create(tb,TweenInfo.new(0.22),{BackgroundColor3=isOn and W.toggleOn or W.toggleOff}):Play()
        TweenService:Create(knob,TweenInfo.new(0.22,Enum.EasingStyle.Back),{Position=isOn and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)}):Play()
        if not skip then cb(isOn) end
    end
    VisualSetters[key]=sv
    clk.MouseButton1Click:Connect(function() isOn=not isOn; Enabled[key]=isOn; sv(isOn) end)
    return cont
end

-- SLIDER FACTORY
local function mkSlider(parent, yp, title, desc, mn, mx, vk, unit, cb)
    local cont=Instance.new("Frame",parent)
    cont.Size=UDim2.new(1,-6,0,72); cont.Position=UDim2.new(0,3,0,yp)
    cont.BackgroundColor3=W.card; cont.BackgroundTransparency=0; cont.BorderSizePixel=0; cont.ZIndex=12
    Instance.new("UICorner",cont).CornerRadius=UDim.new(0,11)
    Instance.new("UIStroke",cont).Color=W.border

    local tl=Instance.new("TextLabel",cont)
    tl.Size=UDim2.new(0.6,0,0,18); tl.Position=UDim2.new(0,12,0,7)
    tl.BackgroundTransparency=1; tl.Text=title; tl.TextColor3=W.ink
    tl.Font=Enum.Font.GothamBold; tl.TextSize=13; tl.TextXAlignment=Enum.TextXAlignment.Left; tl.ZIndex=13

    local dl=Instance.new("TextLabel",cont)
    dl.Size=UDim2.new(0.6,0,0,13); dl.Position=UDim2.new(0,12,0,26)
    dl.BackgroundTransparency=1; dl.Text=desc; dl.TextColor3=W.inkLight
    dl.Font=Enum.Font.Gotham; dl.TextSize=10; dl.TextXAlignment=Enum.TextXAlignment.Left; dl.ZIndex=13

    local defV=Values[vk]
    local vBox=Instance.new("Frame",cont)
    vBox.Size=UDim2.new(0,55,0,30); vBox.Position=UDim2.new(1,-64,0,7)
    vBox.BackgroundColor3=W.cardAlt; vBox.BorderSizePixel=0; vBox.ZIndex=13
    Instance.new("UICorner",vBox).CornerRadius=UDim.new(0,8)
    Instance.new("UIStroke",vBox).Color=W.border

    local vLbl=Instance.new("TextLabel",vBox)
    vLbl.Size=UDim2.new(1,0,0.6,0); vLbl.Position=UDim2.new(0,0,0,1)
    vLbl.BackgroundTransparency=1; vLbl.Text=tostring(defV)
    vLbl.TextColor3=W.blue; vLbl.Font=Enum.Font.GothamBlack; vLbl.TextSize=14; vLbl.ZIndex=14

    local uLbl=Instance.new("TextLabel",vBox)
    uLbl.Size=UDim2.new(1,0,0.4,0); uLbl.Position=UDim2.new(0,0,0.6,0)
    uLbl.BackgroundTransparency=1; uLbl.Text=unit; uLbl.TextColor3=W.inkLight
    uLbl.Font=Enum.Font.Gotham; uLbl.TextSize=9; uLbl.ZIndex=14

    local sTrack=Instance.new("Frame",cont)
    sTrack.Size=UDim2.new(0.88,0,0,7); sTrack.Position=UDim2.new(0.06,0,0,54)
    sTrack.BackgroundColor3=W.cardAlt; sTrack.BorderSizePixel=0; sTrack.ZIndex=13
    Instance.new("UICorner",sTrack).CornerRadius=UDim.new(1,0)

    local pct=(defV-mn)/(mx-mn)
    local sFill=Instance.new("Frame",sTrack)
    sFill.Size=UDim2.new(pct,0,1,0); sFill.BackgroundColor3=W.blue; sFill.BorderSizePixel=0; sFill.ZIndex=13
    Instance.new("UICorner",sFill).CornerRadius=UDim.new(1,0)

    local thumb=Instance.new("Frame",sTrack)
    thumb.Size=UDim2.new(0,13,0,13); thumb.Position=UDim2.new(pct,-6.5,0.5,-6.5)
    thumb.BackgroundColor3=W.white; thumb.BorderSizePixel=0; thumb.ZIndex=14
    Instance.new("UICorner",thumb).CornerRadius=UDim.new(1,0)
    local ts=Instance.new("UIStroke",thumb); ts.Color=W.blue; ts.Thickness=2

    local sBtn=Instance.new("TextButton",sTrack)
    sBtn.Size=UDim2.new(1,0,4,0); sBtn.Position=UDim2.new(0,0,-1.5,0)
    sBtn.BackgroundTransparency=1; sBtn.Text=""; sBtn.ZIndex=15

    local dragging=false
    local function upd(rel)
        rel=math.clamp(rel,0,1); sFill.Size=UDim2.new(rel,0,1,0); thumb.Position=UDim2.new(rel,-6.5,0.5,-6.5)
        local val=math.floor(mn+(mx-mn)*rel); vLbl.Text=tostring(val); Values[vk]=val; cb(val)
    end
    sBtn.MouseButton1Down:Connect(function() dragging=true end)
    UserInputService.InputEnded:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 then dragging=false end end)
    UserInputService.InputChanged:Connect(function(inp) if dragging and inp.UserInputType==Enum.UserInputType.MouseMovement then upd((inp.Position.X-sTrack.AbsolutePosition.X)/sTrack.AbsoluteSize.X) end end)
    return cont
end

-- ACTION BUTTON FACTORY
local function mkBtn(parent, yp, label, btnTxt, color, cb)
    color = color or W.blue
    local cont=Instance.new("Frame",parent)
    cont.Size=UDim2.new(1,-6,0,52); cont.Position=UDim2.new(0,3,0,yp)
    cont.BackgroundColor3=W.card; cont.BackgroundTransparency=0; cont.BorderSizePixel=0; cont.ZIndex=12
    Instance.new("UICorner",cont).CornerRadius=UDim.new(0,11)
    Instance.new("UIStroke",cont).Color=W.border

    local lbl=Instance.new("TextLabel",cont)
    lbl.Size=UDim2.new(0.6,0,1,0); lbl.Position=UDim2.new(0,12,0,0)
    lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=W.ink
    lbl.Font=Enum.Font.GothamBold; lbl.TextSize=13; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=13

    local btn=Instance.new("TextButton",cont)
    btn.Size=UDim2.new(0,90,0,30); btn.Position=UDim2.new(1,-100,0.5,-15)
    btn.BackgroundColor3=color; btn.Text=btnTxt
    btn.TextColor3=W.white; btn.Font=Enum.Font.GothamBold; btn.TextSize=12; btn.ZIndex=13
    btn.BorderSizePixel=0
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8)
    btn.MouseButton1Click:Connect(cb)
    btn.MouseEnter:Connect(function() TweenService:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=W.blueDark}):Play() end)
    btn.MouseLeave:Connect(function() TweenService:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=color}):Play() end)
    return cont
end

-- HEADER LABEL
local function mkHeader(parent, yp, text)
    local lbl=Instance.new("TextLabel",parent)
    lbl.Size=UDim2.new(1,-6,0,24); lbl.Position=UDim2.new(0,3,0,yp)
    lbl.BackgroundTransparency=1; lbl.Text=text
    lbl.TextColor3=W.blue; lbl.Font=Enum.Font.GothamBold; lbl.TextSize=13
    lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.ZIndex=12
    return lbl
end

-- ──────────────────────────────────────────────────────────────
-- POPULATE STEAL TAB
-- ──────────────────────────────────────────────────────────────
local y=8
mkHeader(stealPanel,y,"— AUTO STEAL")
y=y+28
mkToggle(stealPanel,y,"Auto Steal (Best Gen)","Steals highest gen brainrot in radius","AutoSteal",
    function(s) Enabled.AutoSteal=s; if s then startAutoSteal() else stopAutoSteal() end end)
y=y+62
mkToggle(stealPanel,y,"Auto Steal (Nearest)","Steals the closest brainrot","AutoStealNearest",
    function(s) Enabled.AutoStealNearest=s; if s then startAutoSteal() else stopAutoSteal() end end)
y=y+62
mkToggle(stealPanel,y,"Auto Steal (Priority)","Steals priority list animals first","AutoStealPriority",
    function(s) Enabled.AutoStealPriority=s; if s then startAutoSteal() else stopAutoSteal() end end)
y=y+62
mkToggle(stealPanel,y,"Speed During Steal","Move faster while stealing","StealSpeed",
    function(s) Enabled.StealSpeed=s; if s then startStealSpeed() else stopStealSpeed() end end)
y=y+62
mkToggle(stealPanel,y,"Disable Anim During Steal","Stops animations while stealing","DisableAnim",
    function(s) Enabled.DisableAnim=s; if s then startDisableAnim() else stopDisableAnim() end end)
y=y+62
mkToggle(stealPanel,y,"Invisible During Steal","Makes you invisible while stealing","Invisible",
    function(s) Enabled.Invisible=s end)
y=y+62
mkSlider(stealPanel,y,"Steal Radius","Max distance to steal from",5,100,"StealRadius","st",
    function(v) Values.StealRadius=v end)
y=y+76
mkSlider(stealPanel,y,"Steal Speed","Speed boost during steal",10,60,"StealSpeed","x",
    function(v) Values.StealSpeed=v end)
y=y+76
mkHeader(stealPanel,y,"— TELEPORT")
y=y+28
mkBtn(stealPanel,y,"TP to Highest Gen","TELEPORT",W.blue,function()
    tpToHighest(); showNotif("Teleporting to highest!", W.blue)
end)
y=y+56
mkBtn(stealPanel,y,"TP to Priority","PRIORITY",W.yellow,function()
    tpToPriority(); showNotif("Teleporting to priority!", W.yellow)
end)
y=y+56
mkToggle(stealPanel,y,"Floor Steal","Float platform below you","FloorSteal",
    function(s) Enabled.FloorSteal=s; if s then startFloorSteal() else stopFloorSteal() end end)
stealPanel.CanvasSize=UDim2.new(0,0,0,y+65)

-- ──────────────────────────────────────────────────────────────
-- POPULATE MOVEMENT TAB
-- ──────────────────────────────────────────────────────────────
y=8
mkToggle(movementPanel,y,"Speed Boost","Move faster around the map","SpeedBoost",
    function(s) Enabled.SpeedBoost=s; if s then startSpeedBoost() else stopSpeedBoost() end end)
y=y+62
mkSlider(movementPanel,y,"Speed Multiplier","How fast you move",5,100,"BoostSpeed","x",
    function(v) Values.BoostSpeed=v end)
y=y+76
mkToggle(movementPanel,y,"Anti Ragdoll","Prevents getting stunned","AntiRagdoll",
    function(s) Enabled.AntiRagdoll=s; if s then startAntiRagdoll() else stopAntiRagdoll() end end)
y=y+62
mkToggle(movementPanel,y,"Anti Bee","Prevents bee effects","AntiBee",
    function(s) Enabled.AntiBee=s; if s then enableAntiBee() else disableAntiBee() end end)
movementPanel.CanvasSize=UDim2.new(0,0,0,y+65)

-- ──────────────────────────────────────────────────────────────
-- POPULATE VISUALS TAB
-- ──────────────────────────────────────────────────────────────
y=8
mkToggle(visualsPanel,y,"Brainrot ESP","Highlight brainrots through walls","ESP",
    function(s) Enabled.ESP=s; if s then refreshESP(sg) else clearESP() end end)
y=y+62
mkToggle(visualsPanel,y,"Player ESP","Show players through walls","PlayerESP",
    function(s) Enabled.PlayerESP=s; if s then refreshPlayerESP(sg) else clearPlayerESP() end end)
y=y+62
mkToggle(visualsPanel,y,"Fullbright","Makes everything bright","Fullbright",
    function(s) Enabled.Fullbright=s; if s then enableFullbright() else disableFullbright() end end)
y=y+62
mkToggle(visualsPanel,y,"Xray Base Walls","See through walls","XrayBase",
    function(s) Enabled.XrayBase=s; if s then enableXray() else disableXray() end end)
visualsPanel.CanvasSize=UDim2.new(0,0,0,y+65)

-- ──────────────────────────────────────────────────────────────
-- POPULATE MISC TAB
-- ──────────────────────────────────────────────────────────────
y=8
mkToggle(miscPanel,y,"Desync V3","Reduces lag and prevents lagback","Desync",
    function(s) Enabled.Desync=s; if s then enableDesync(); showNotif("Desync enabled!",W.blue) end end)
y=y+62
mkToggle(miscPanel,y,"Anti Lag","Removes FPS killers","AntiLag",
    function(s) Enabled.AntiLag=s; if s then enableAntiLag() else disableAntiLag() end end)
y=y+62
mkBtn(miscPanel,y,"Rejoin Server","REJOIN",W.blue,function()
    pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, Player) end)
end)
y=y+56
mkBtn(miscPanel,y,"Kick Self","KICK",W.danger,function()
    Player:Kick("Kicked by Alaz Hub")
end)
y=y+56
mkBtn(miscPanel,y,"Rescan Plots","SCAN",W.blue,function()
    if Enabled.ESP then refreshESP(sg) end
    if Enabled.PlayerESP then refreshPlayerESP(sg) end
    showNotif("Rescanning plots!", W.blue)
end)
miscPanel.CanvasSize=UDim2.new(0,0,0,y+65)

-- BOTTOM BAR
local bottomBar=Instance.new("Frame",main)
bottomBar.Size=UDim2.new(1,-20,0,38); bottomBar.Position=UDim2.new(0,10,1,-44)
bottomBar.BackgroundColor3=W.blue; bottomBar.BorderSizePixel=0; bottomBar.ZIndex=11
Instance.new("UICorner",bottomBar).CornerRadius=UDim.new(0,10)

local keyInfo=Instance.new("TextLabel",bottomBar)
keyInfo.Size=UDim2.new(1,0,1,0); keyInfo.BackgroundTransparency=1
keyInfo.Text="⚔ Alaz Hub  •  discord.gg/U4XXCxKUm  •  U = Toggle"
keyInfo.TextColor3=Color3.fromRGB(200,230,255); keyInfo.Font=Enum.Font.GothamBold
keyInfo.TextSize=10; keyInfo.ZIndex=12

-- MINI BUTTON + KEYBIND
miniBtn.MouseButton1Click:Connect(function()
    guiVisible=not guiVisible; main.Visible=guiVisible
    TweenService:Create(miniBtn,TweenInfo.new(0.2),{BackgroundColor3=guiVisible and W.blue or W.blueDark}):Play()
end)

UserInputService.InputBegan:Connect(function(inp,gpe)
    if gpe then return end
    if inp.KeyCode==Enum.KeyCode.U then guiVisible=not guiVisible; main.Visible=guiVisible end
end)

-- RESPAWN
Player.CharacterAdded:Connect(function()
    task.wait(1)
    if Enabled.SpeedBoost  then stopSpeedBoost();  task.wait(0.1); startSpeedBoost()  end
    if Enabled.AntiRagdoll then stopAntiRagdoll(); task.wait(0.1); startAntiRagdoll() end
    if Enabled.AutoSteal or Enabled.AutoStealNearest or Enabled.AutoStealPriority then
        stopAutoSteal(); task.wait(0.1); startAutoSteal()
    end
    if Enabled.FloorSteal  then startFloorSteal() end
    if Enabled.StealSpeed  then startStealSpeed() end
end)

-- ESP REFRESH LOOP
task.spawn(function()
    while sg and sg.Parent do
        if Enabled.ESP         then refreshESP(sg)       end
        if Enabled.PlayerESP   then refreshPlayerESP(sg) end
        task.wait(3)
    end
end)

showNotif("⚔ Alaz Hub Loaded!  U = Toggle", W.blue)
print("⚔ ALAZ HUB Loaded!  U=Toggle  discord.gg/U4XXCxKUm")
