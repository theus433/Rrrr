-- Script put in ServerScriptService 📝
-- Make sure add RemoteEvent and name PunchEvent ⚠️
-- Made by potato dev 👍😎
-- Like and Subscribe 🤯🥳🥳🥳

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")


local punchEvent = ReplicatedStorage:FindFirstChild("PunchEvent") or Instance.new("RemoteEvent", ReplicatedStorage)
punchEvent.Name = "PunchEvent"

local punchCooldown = 0.6 -- Each punch CD 📝
local punchRadius = 5 -- This is punch range 📝
local maxDamage = 10 -- Damage 📝
local damageAnimationId = "rbxassetid://282574440" 
local playerCooldowns = {}


local function canPlayerPunch(player)
    local lastPunchTime = playerCooldowns[player.UserId] or 0
    local currentTime = tick()
    if currentTime - lastPunchTime < punchCooldown then
        return false
    end
    playerCooldowns[player.UserId] = currentTime
    return true
end


local function disableMovement(target)
    local humanoid = target:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = 0
        humanoid.JumpPower = 0
    end
end


local function enableMovement(target)
    local humanoid = target:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = 16 
        humanoid.JumpPower = 50 
    end
end


local function playDamageAnimation(target)
    local humanoid = target:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    local animator = humanoid:FindFirstChildOfClass("Animator") or humanoid:WaitForChild("Animator")
    if not animator then return end

    local animation = Instance.new("Animation")
    animation.AnimationId = damageAnimationId

    local animationTrack = animator:LoadAnimation(animation)

    
    animationTrack:GetMarkerReachedSignal("AnimationStart"):Connect(function()
        disableMovement(target)
    end)

    animationTrack.Stopped:Connect(function()
        
        enableMovement(target)
    end)

    animationTrack:Play()

    
    task.delay(2, function()
        animationTrack:Stop()
        animation:Destroy()
    end)
end


local function flingCharacter(target, isBackwardPush)
    local humanoidRootPart = target:FindFirstChild("HumanoidRootPart")
    if humanoidRootPart then
        local direction = target.HumanoidRootPart.CFrame.LookVector
        local flingDirection

        if isBackwardPush then
            
            flingDirection = -direction * 30
        else
            
            flingDirection = Vector3.new(math.random(-10, 20), 70, math.random(-10, 20))
        end

        
        local success, _ = pcall(function()
            humanoidRootPart:SetNetworkOwner(nil)
        end)

        
        humanoidRootPart.Velocity = humanoidRootPart.Velocity + flingDirection

        
        if success then
            task.delay(0.1, function()
                local player = Players:GetPlayerFromCharacter(target)
                if player then
                    humanoidRootPart:SetNetworkOwner(player)
                end
            end)
        end
    end
end


punchEvent.OnServerEvent:Connect(function(player, damage, playAnimation, playFling, isBackwardPush)
    if not canPlayerPunch(player) then
        return
    end

    local character = player.Character
    if character then
        local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
        if not humanoidRootPart then return end

        local origin = humanoidRootPart.Position

        
        for _, obj in pairs(Workspace:GetDescendants()) do
            local humanoidRootPart = obj:FindFirstChild("HumanoidRootPart")
            local humanoid = obj:FindFirstChildOfClass("Humanoid")
            if humanoidRootPart and humanoid then
                local distance = (origin - humanoidRootPart.Position).Magnitude
                if distance <= punchRadius and obj ~= character then
                    humanoid:TakeDamage(damage)

                    
                    if playAnimation then
                        playDamageAnimation(obj)
                    end

                    
                    if playFling then
                        flingCharacter(obj, isBackwardPush)
                    end
                end
            end
        end
    end
end)
