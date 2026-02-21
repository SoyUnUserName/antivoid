local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local connection = nil
local enabled = true

local VOID_THRESHOLD = -250
local TELEPORT_UP = 180

local function applyImmortality()
    local character = LocalPlayer.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.MaxHealth = 9e9
        humanoid.Health = 9e9
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
    end
end

local function enableAntiVoid()
    if connection then return end
    pcall(function()
        Workspace.FallenPartsDestroyHeight = 0/0
    end)
    applyImmortality()
    
    connection = RunService.Heartbeat:Connect(function()
        pcall(function()
            local character = LocalPlayer.Character
            if not character then return end
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            if hrp.Position.Y < VOID_THRESHOLD then
                hrp.CFrame += Vector3.new(0, TELEPORT_UP, 0)
                hrp.Velocity = Vector3.new(hrp.Velocity.X, 30, hrp.Velocity.Z)
                hrp.AssemblyLinearVelocity = Vector3.new(0,0,0)
                hrp.AssemblyAngularVelocity = Vector3.new(0,0,0)
            end
        end)
    end)
end

local function disableAntiVoid()
    if connection then
        connection:Disconnect()
        connection = nil
    end
end

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.7)
    if enabled then
        applyImmortality()
    end
end)

enableAntiVoid()

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        enabled = not enabled
        if enabled then
            enableAntiVoid()
        else
            disableAntiVoid()
        end
    end
end)
