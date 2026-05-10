--[[
    RENAN FRX | Script
    Cores: Vermelho e Preto
    Opções: Teleporta, Pega Caixa, Noclip, Fly (com velocidade ajustável)
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- Limpeza
local function limpeza()
    pcall(function()
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg and pg:FindFirstChild("RenanFrxScript") then
            pg.RenanFrxScript:Destroy()
        end
    end)
    pcall(function()
        local cg = game:GetService("CoreGui")
        if cg:FindFirstChild("RenanFrxScript") then
            cg.RenanFrxScript:Destroy()
        end
    end)
end
limpeza()

-- ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RenanFrxScript"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local ok = pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)
if not ok then
    ScreenGui.Parent = LocalPlayer.PlayerGui
end

-- =========================
-- PALETA DE CORES
-- =========================
local COLOR_BG        = Color3.fromRGB(15, 15, 15)
local COLOR_BG_DARK   = Color3.fromRGB(8, 8, 8)
local COLOR_PANEL     = Color3.fromRGB(22, 22, 22)
local COLOR_RED       = Color3.fromRGB(200, 20, 20)
local COLOR_RED_DARK  = Color3.fromRGB(140, 10, 10)
local COLOR_RED_GLOW  = Color3.fromRGB(255, 40, 40)
local COLOR_TEXT      = Color3.fromRGB(240, 240, 240)
local COLOR_MUTED     = Color3.fromRGB(160, 160, 160)

-- =========================
-- HELPERS
-- =========================
local function corner(parent, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 8)
    c.Parent = parent
    return c
end

local function stroke(parent, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color or COLOR_RED
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = parent
    return s
end

-- =========================
-- FRAME PRINCIPAL
-- =========================
local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.new(0, 420, 0, 360)
Main.Position = UDim2.new(0.5, -210, 0.5, -180)
Main.BackgroundColor3 = COLOR_BG
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = false
Main.Parent = ScreenGui
corner(Main, 12)
stroke(Main, COLOR_RED, 1.5, 0.2)

local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, COLOR_BG),
    ColorSequenceKeypoint.new(1, COLOR_BG_DARK),
}
Gradient.Rotation = 90
Gradient.Parent = Main

-- =========================
-- BARRA DE TÍTULO
-- =========================
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 38)
TitleBar.BackgroundColor3 = COLOR_BG_DARK
TitleBar.BorderSizePixel = 0
TitleBar.Parent = Main
corner(TitleBar, 12)

local RedStrip = Instance.new("Frame")
RedStrip.Size = UDim2.new(1, 0, 0, 2)
RedStrip.Position = UDim2.new(0, 0, 1, -2)
RedStrip.BackgroundColor3 = COLOR_RED
RedStrip.BorderSizePixel = 0
RedStrip.Parent = TitleBar

local Dot = Instance.new("Frame")
Dot.Size = UDim2.new(0, 10, 0, 10)
Dot.Position = UDim2.new(0, 12, 0.5, -5)
Dot.BackgroundColor3 = COLOR_RED_GLOW
Dot.BorderSizePixel = 0
Dot.Parent = TitleBar
corner(Dot, 5)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -120, 1, 0)
Title.Position = UDim2.new(0, 32, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "RENAN FRX  |  Script"
Title.TextColor3 = COLOR_TEXT
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Font = Enum.Font.GothamBold
Title.TextSize = 15
Title.Parent = TitleBar

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 28, 0, 24)
MinBtn.Position = UDim2.new(1, -68, 0.5, -12)
MinBtn.BackgroundColor3 = COLOR_PANEL
MinBtn.Text = "—"
MinBtn.TextColor3 = COLOR_TEXT
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 14
MinBtn.AutoButtonColor = false
MinBtn.Parent = TitleBar
corner(MinBtn, 6)
stroke(MinBtn, COLOR_RED_DARK, 1, 0.4)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 28, 0, 24)
CloseBtn.Position = UDim2.new(1, -34, 0.5, -12)
CloseBtn.BackgroundColor3 = COLOR_RED_DARK
CloseBtn.Text = "X"
CloseBtn.TextColor3 = COLOR_TEXT
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 13
CloseBtn.AutoButtonColor = false
CloseBtn.Parent = TitleBar
corner(CloseBtn, 6)
stroke(CloseBtn, COLOR_RED, 1, 0.2)

-- =========================
-- CONTAINER DO CONTEÚDO
-- =========================
local Content = Instance.new("Frame")
Content.Name = "Content"
Content.Size = UDim2.new(1, -20, 1, -50)
Content.Position = UDim2.new(0, 10, 0, 44)
Content.BackgroundTransparency = 1
Content.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 8)
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Parent = Content

-- =========================
-- FUNÇÃO: BOTÃO DE AÇÃO
-- =========================
local function createActionButton(text, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 38)
    btn.BackgroundColor3 = COLOR_PANEL
    btn.Text = text
    btn.TextColor3 = COLOR_TEXT
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 14
    btn.AutoButtonColor = false
    btn.LayoutOrder = order
    btn.Parent = Content
    corner(btn, 8)
    stroke(btn, COLOR_RED_DARK, 1, 0.5)

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(0, 3, 0.6, 0)
    bar.Position = UDim2.new(0, 8, 0.2, 0)
    bar.BackgroundColor3 = COLOR_RED
    bar.BorderSizePixel = 0
    bar.Parent = btn
    corner(bar, 2)

    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = COLOR_RED_DARK}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = COLOR_PANEL}):Play()
    end)

    return btn
end

-- =========================
-- FUNÇÃO: TOGGLE (ON/OFF)
-- =========================
local function createToggleButton(text, order)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 38)
    frame.BackgroundColor3 = COLOR_PANEL
    frame.BorderSizePixel = 0
    frame.LayoutOrder = order
    frame.Parent = Content
    corner(frame, 8)
    stroke(frame, COLOR_RED_DARK, 1, 0.5)

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(0, 3, 0.6, 0)
    bar.Position = UDim2.new(0, 8, 0.2, 0)
    bar.BackgroundColor3 = COLOR_RED
    bar.BorderSizePixel = 0
    bar.Parent = frame
    corner(bar, 2)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -80, 1, 0)
    label.Position = UDim2.new(0, 18, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = COLOR_TEXT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 14
    label.Parent = frame

    local switch = Instance.new("TextButton")
    switch.Size = UDim2.new(0, 50, 0, 22)
    switch.Position = UDim2.new(1, -60, 0.5, -11)
    switch.BackgroundColor3 = COLOR_BG_DARK
    switch.Text = ""
    switch.AutoButtonColor = false
    switch.Parent = frame
    corner(switch, 11)
    stroke(switch, COLOR_RED_DARK, 1, 0.3)

    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 18, 0, 18)
    knob.Position = UDim2.new(0, 2, 0.5, -9)
    knob.BackgroundColor3 = COLOR_MUTED
    knob.BorderSizePixel = 0
    knob.Parent = switch
    corner(knob, 9)

    local state = false
    switch.MouseButton1Click:Connect(function()
        state = not state
        if state then
            TweenService:Create(knob, TweenInfo.new(0.18), {
                Position = UDim2.new(1, -20, 0.5, -9),
                BackgroundColor3 = COLOR_RED_GLOW,
            }):Play()
            TweenService:Create(switch, TweenInfo.new(0.18), {BackgroundColor3 = COLOR_RED_DARK}):Play()
        else
            TweenService:Create(knob, TweenInfo.new(0.18), {
                Position = UDim2.new(0, 2, 0.5, -9),
                BackgroundColor3 = COLOR_MUTED,
            }):Play()
            TweenService:Create(switch, TweenInfo.new(0.18), {BackgroundColor3 = COLOR_BG_DARK}):Play()
        end
    end)

    return frame, function() return state end
end

-- =========================
-- BOTÕES / OPÇÕES
-- =========================
local btnTeleport                    = createActionButton("⚡  Teleporta", 1)
local btnCaixa                       = createActionButton("📦  Pega Caixa", 2)
local toggleNoclipFrame, getNoclip   = createToggleButton("🚶  Noclip", 3)
local toggleFlyFrame,    getFly      = createToggleButton("🕊️  Fly", 4)

-- =========================
-- SLIDER DE VELOCIDADE (FLY)
-- =========================
local SliderFrame = Instance.new("Frame")
SliderFrame.Size = UDim2.new(1, 0, 0, 54)
SliderFrame.BackgroundColor3 = COLOR_PANEL
SliderFrame.BorderSizePixel = 0
SliderFrame.LayoutOrder = 5
SliderFrame.Parent = Content
corner(SliderFrame, 8)
stroke(SliderFrame, COLOR_RED_DARK, 1, 0.5)

local sliderLabel = Instance.new("TextLabel")
sliderLabel.Size = UDim2.new(1, -20, 0, 20)
sliderLabel.Position = UDim2.new(0, 12, 0, 6)
sliderLabel.BackgroundTransparency = 1
sliderLabel.Text = "Velocidade do Fly"
sliderLabel.TextColor3 = COLOR_TEXT
sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
sliderLabel.Font = Enum.Font.GothamSemibold
sliderLabel.TextSize = 13
sliderLabel.Parent = SliderFrame

local sliderValue = Instance.new("TextLabel")
sliderValue.Size = UDim2.new(0, 50, 0, 20)
sliderValue.Position = UDim2.new(1, -62, 0, 6)
sliderValue.BackgroundTransparency = 1
sliderValue.Text = "50"
sliderValue.TextColor3 = COLOR_RED_GLOW
sliderValue.TextXAlignment = Enum.TextXAlignment.Right
sliderValue.Font = Enum.Font.GothamBold
sliderValue.TextSize = 13
sliderValue.Parent = SliderFrame

local Track = Instance.new("Frame")
Track.Size = UDim2.new(1, -24, 0, 6)
Track.Position = UDim2.new(0, 12, 1, -16)
Track.BackgroundColor3 = COLOR_BG_DARK
Track.BorderSizePixel = 0
Track.Parent = SliderFrame
corner(Track, 3)

local Fill = Instance.new("Frame")
Fill.Size = UDim2.new(0.5, 0, 1, 0)
Fill.BackgroundColor3 = COLOR_RED
Fill.BorderSizePixel = 0
Fill.Parent = Track
corner(Fill, 3)

local Knob = Instance.new("Frame")
Knob.Size = UDim2.new(0, 14, 0, 14)
Knob.Position = UDim2.new(0.5, -7, 0.5, -7)
Knob.BackgroundColor3 = COLOR_RED_GLOW
Knob.BorderSizePixel = 0
Knob.Parent = Track
corner(Knob, 7)
stroke(Knob, COLOR_TEXT, 1, 0.4)

local MIN_SPEED, MAX_SPEED = 10, 200
local flySpeed = 50
local dragging = false

local function updateSlider(input)
    local rel = math.clamp((input.Position.X - Track.AbsolutePosition.X) / Track.AbsoluteSize.X, 0, 1)
    Fill.Size = UDim2.new(rel, 0, 1, 0)
    Knob.Position = UDim2.new(rel, -7, 0.5, -7)
    flySpeed = math.floor(MIN_SPEED + (MAX_SPEED - MIN_SPEED) * rel)
    sliderValue.Text = tostring(flySpeed)
end

Track.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        updateSlider(input)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSlider(input)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- =========================
-- RODAPÉ
-- =========================
local Footer = Instance.new("TextLabel")
Footer.Size = UDim2.new(1, 0, 0, 16)
Footer.BackgroundTransparency = 1
Footer.Text = "RENAN FRX • v1.0"
Footer.TextColor3 = COLOR_MUTED
Footer.Font = Enum.Font.Gotham
Footer.TextSize = 11
Footer.LayoutOrder = 6
Footer.Parent = Content

-- =========================
-- ÍCONE FLUTUANTE
-- =========================
local FloatBtn = Instance.new("TextButton")
FloatBtn.Size = UDim2.new(0, 48, 0, 48)
FloatBtn.Position = UDim2.new(0, 20, 0.5, -24)
FloatBtn.BackgroundColor3 = COLOR_BG
FloatBtn.Text = "R"
FloatBtn.TextColor3 = COLOR_RED_GLOW
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 22
FloatBtn.Visible = false
FloatBtn.AutoButtonColor = false
FloatBtn.Active = true
FloatBtn.Draggable = true
FloatBtn.Parent = ScreenGui
corner(FloatBtn, 24)
stroke(FloatBtn, COLOR_RED, 1.5, 0.1)

-- =========================
-- MINIMIZAR / FECHAR
-- =========================
MinBtn.MouseButton1Click:Connect(function()
    Main.Visible = false
    FloatBtn.Visible = true
end)

FloatBtn.MouseButton1Click:Connect(function()
    Main.Visible = true
    FloatBtn.Visible = false
end)

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- =========================
-- ARRASTAR JANELA
-- =========================
do
    local dragInput, dragStart, startPos, draggingWin
    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            draggingWin = true
            dragStart = input.Position
            startPos = Main.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    draggingWin = false
                end
            end)
        end
    end)
    TitleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and draggingWin then
            local delta = input.Position - dragStart
            Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- =========================
-- LÓGICA: TELEPORTA
-- =========================
local Mouse = LocalPlayer:GetMouse()

btnTeleport.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then
        root.CFrame = CFrame.new(Mouse.Hit.Position + Vector3.new(0, 3, 0))
    end
end)

-- =========================
-- LÓGICA: PEGA CAIXA
-- =========================
btnCaixa.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and string.lower(obj.Name):find("caixa") then
            local dist = (obj.Position - root.Position).Magnitude
            if dist < 50 then
                obj.Anchored = false
                -- Remove weld antigo se existir
                for _, w in ipairs(obj:GetChildren()) do
                    if w:IsA("WeldConstraint") then w:Destroy() end
                end
                obj.CFrame = root.CFrame * CFrame.new(0, 2, -3)
                local weld = Instance.new("WeldConstraint")
                weld.Part0 = root
                weld.Part1 = obj
                weld.Parent = obj
            end
        end
    end
end)

-- =========================
-- LÓGICA: NOCLIP
-- =========================
RunService.Stepped:Connect(function()
    if getNoclip() then
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

-- =========================
-- LÓGICA: FLY
-- =========================
local isFlyActive = false
local bodyVelocity, bodyGyro
local flyConn

local function enableFly(char)
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum  = char:FindFirstChildOfClass("Humanoid")
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
        if not getFly() then return end
        local cam = workspace.CurrentCamera
        local moveDir = Vector3.zero

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

        bodyVelocity.Velocity = moveDir * flySpeed
        bodyGyro.CFrame = cam.CFrame
    end)
end

local function disableFly(char)
    if flyConn then
        flyConn:Disconnect()
        flyConn = nil
    end
    if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
    if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = false end
    end
end

RunService.Heartbeat:Connect(function()
    local active = getFly()
    if active and not isFlyActive then
        isFlyActive = true
        local char = LocalPlayer.Character
        if char then enableFly(char) end
    elseif not active and isFlyActive then
        isFlyActive = false
        disableFly(LocalPlayer.Character)
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    isFlyActive = false
    if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
    if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
    if flyConn then flyConn:Disconnect() flyConn = nil end
end)

print("[RENAN FRX] Script carregado com sucesso!")
