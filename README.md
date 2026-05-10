-- =========================
-- LÓGICA DAS FUNCIONALIDADES
-- =========================

local function getCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

-- ── TELEPORTA ──────────────────────────────────────────────
local Mouse = LocalPlayer:GetMouse()

btnTeleport.MouseButton1Click:Connect(function()
    local char = getCharacter()
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then
        root.CFrame = CFrame.new(Mouse.Hit.Position + Vector3.new(0, 3, 0))
    end
end)

-- ── PEGA CAIXA ─────────────────────────────────────────────
btnCaixa.MouseButton1Click:Connect(function()
    local char = getCharacter()
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and string.lower(obj.Name):find("caixa") then
            local dist = (obj.Position - root.Position).Magnitude
            if dist < 50 then
                obj.CFrame = root.CFrame * CFrame.new(0, 2, -3)
                -- Torna a parte não colidível temporariamente para "segurar"
                obj.Anchored = false
                local weld = Instance.new("WeldConstraint")
                weld.Part0 = root
                weld.Part1 = obj
                weld.Parent = obj
            end
        end
    end
end)

-- ── NOCLIP ─────────────────────────────────────────────────
local noclipActive = false
local noclipConn

local _, getNoclipState = toggleNoclip  -- pega getter do estado

RunService.Stepped:Connect(function()
    -- Relê o estado do toggle a cada frame
    if getNoclipState() then
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
end)

-- ── FLY ────────────────────────────────────────────────────
local _, getFlyState = toggleFly

local flyConn
local isFlyActive = false
local bodyVelocity, bodyGyro

local function enableFly(char)
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum   = char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end

    hum.PlatformStand = true

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Parent = root

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bodyGyro.P = 1e4
    bodyGyro.CFrame = root.CFrame
    bodyGyro.Parent = root

    flyConn = RunService.Heartbeat:Connect(function()
        if not getFlyState() then return end

        local cam = workspace.CurrentCamera
        local moveDir = Vector3.zero
        local spd = flySpeed

        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDir = moveDir + cam.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDir = moveDir - cam.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDir = moveDir - cam.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDir = moveDir + cam.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDir = moveDir + Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            moveDir = moveDir - Vector3.new(0, 1, 0)
        end

        if moveDir.Magnitude > 0 then
            moveDir = moveDir.Unit
        end

        bodyVelocity.Velocity = moveDir * spd
        bodyGyro.CFrame = cam.CFrame
    end)
end

local function disableFly(char)
    if flyConn then flyConn:Disconnect() end
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then hum.PlatformStand = false end
end

-- Monitora mudança de estado do toggle Fly
RunService.Heartbeat:Connect(function()
    local active = getFlyState()
    if active and not isFlyActive then
        isFlyActive = true
        local char = LocalPlayer.Character
        if char then enableFly(char) end
    elseif not active and isFlyActive then
        isFlyActive = false
        disableFly(LocalPlayer.Character)
    end
end)

-- Reaplica fly ao respawnar
LocalPlayer.CharacterAdded:Connect(function(char)
    isFlyActive = false
    -- Noclip: reativa automaticamente via Stepped se toggle estiver on
end)

print("[RENAN FRX] Lógica carregada.")
