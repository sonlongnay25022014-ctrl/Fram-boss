-- ==============================================================================
-- TÍNH NĂNG: CIRCLE FARM BOSS & TỰ ĐỘNG BAY TỚI BOSS
-- ==============================================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local TARGET_POS = Vector3.new(-2789.7085, 238.5443, -1758.7052)
local HEIGHT_OFFSET = 3
local RADIUS = 15
local SPEED = 3 -- Tốc độ bay vòng tròn
local NPC_NAME = "npc2"

local isFarming = false
local angle = 0
local farmConnection = nil
local nextTeleportTime = 0
local cachedNPC = nil
local lastCacheCheck = 0
local originalMaterials = {}

local function CleanGraphics()
    Lighting.GlobalShadows = false
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and not obj:IsA("MeshPart") then
            if not originalMaterials[obj] then originalMaterials[obj] = obj.Material end
            obj.Material = Enum.Material.SmoothPlastic
        end
    end
end

local function RestoreGraphics()
    Lighting.GlobalShadows = true
    for obj, mat in pairs(originalMaterials) do if obj and obj.Parent then obj.Material = mat end end
    table.clear(originalMaterials)
end

local function findTargetNPC()
    local now = tick()
    if now - lastCacheCheck < 0.3 and cachedNPC and cachedNPC.Parent and cachedNPC:FindFirstChild("HumanoidRootPart") then 
        local hum = cachedNPC:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health > 0 then return cachedNPC end
    end
    lastCacheCheck = now

    local lowerTargetName = string.lower(NPC_NAME)
    local bestTarget, maxHealthFound = nil, -1

    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") then
            if string.find(string.lower(obj.Name), lowerTargetName) then
                local hrp = obj:FindFirstChild("HumanoidRootPart")
                local hum = obj:FindFirstChildOfClass("Humanoid")
                if hrp and hum and hum.Health > 0 then
                    if hum.MaxHealth > maxHealthFound then
                        maxHealthFound = hum.MaxHealth
                        bestTarget = obj
                    end
                end
            end
        end
    end
    cachedNPC = bestTarget
    return bestTarget
end

local function flyToPosition(targetPos)
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local distance = (targetPos - hrp.Position).Magnitude
    while distance > 8 and isFarming do
        local currentChar = player.Character
        if not currentChar then break end
        local currentHRP = currentChar:FindFirstChild("HumanoidRootPart")
        if not currentHRP then break end

        local dir = (targetPos - currentHRP.Position).Unit
        currentHRP.CFrame = currentHRP.CFrame + dir * math.min(25, distance)
        task.wait(0.01)
        distance = (targetPos - currentHRP.Position).Magnitude
    end
end

local function startFarm()
    isFarming = true
    CleanGraphics()

    -- Bay tới boss ngay khi bật
    task.spawn(function()
        while isFarming do
            local npcModel = findTargetNPC()
            if npcModel and npcModel:FindFirstChild("HumanoidRootPart") then
                local bossPos = npcModel.HumanoidRootPart.Position
                flyToPosition(Vector3.new(bossPos.X, bossPos.Y + HEIGHT_OFFSET, bossPos.Z))
                break
            else
                flyToPosition(TARGET_POS)
            end
            task.wait(0.5)
        end
    end)

    farmConnection = RunService.Heartbeat:Connect(function(deltaTime)
        if not isFarming then return end
        local character = player.Character
        local rootPart = character and character:FindFirstChild("HumanoidRootPart")
        if not rootPart then return end

        if tick() >= nextTeleportTime then
            angle = math.random() * (2 * math.pi)
            nextTeleportTime = tick() + 5
        else
            angle = angle + (SPEED * deltaTime)
        end

        local npcModel = findTargetNPC()
        local currentCenterPos = TARGET_POS
        local lookAtTarget = TARGET_POS

        if npcModel and npcModel.Parent then
            local npcRoot = npcModel:FindFirstChild("HumanoidRootPart")
            if npcRoot then
                currentCenterPos = npcRoot.Position
                lookAtTarget = npcRoot.Position
            end
        end

        local currentFarmPos = Vector3.new(currentCenterPos.X, currentCenterPos.Y + HEIGHT_OFFSET, currentCenterPos.Z)
        local offsetX = math.cos(angle) * RADIUS
        local offsetZ = math.sin(angle) * RADIUS
        local newPos = Vector3.new(currentFarmPos.X + offsetX, currentFarmPos.Y, currentFarmPos.Z + offsetZ)

        rootPart.CFrame = CFrame.new(newPos, Vector3.new(lookAtTarget.X, newPos.Y, lookAtTarget.Z))
    end)
end

local function stopFarm()
    isFarming = false
    if farmConnection then farmConnection:Disconnect(); farmConnection = nil end
    RestoreGraphics()
end
