-- Enhanced Sniper Arena Menu System
-- Features: Character selection, weapon customization, game settings, aimbot toggle
-- Author: DeepHat
-- Date: 2023-09-15

local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Aimbot functionality
local aimbotEnabled = false
local aimbotPart = nil

-- Create main menu frame
local mainMenu = Instance.new("ScreenGui")
mainMenu.Name = "MainMenu"
mainMenu.Parent = StarterGui

-- Main menu background
local background = Instance.new("Frame")
background.Size = UDim2.new(0, 800, 0, 600)
background.Position = UDim2.new(0.5, -400, 0.5, -300)
background.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
background.BorderSizePixel = 0
background.Parent = mainMenu

-- Title label
local title = Instance.new("TextLabel")
title.Text = "SNIPER ARENA"
title.Size = UDim2.new(0, 600, 0, 50)
title.Position = UDim2.new(0.5, -300, 0.1, 0)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 36
title.TextColor3 = Color3.fromRGB(255, 215, 0)
title.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
title.BorderSizePixel = 0
title.Parent = background

-- Character selection frame
local charSelectFrame = Instance.new("Frame")
charSelectFrame.Visible = false
charSelectFrame.Size = UDim2.new(0, 700, 0, 400)
charSelectFrame.Position = UDim2.new(0.5, -350, 0.5, -200)
charSelectFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
charSelectFrame.BorderSizePixel = 0
charSelectFrame.Parent = background

-- Weapon selection frame
local weaponSelectFrame = Instance.new("Frame")
weaponSelectFrame.Visible = false
weaponSelectFrame.Size = UDim2.new(0, 700, 0, 400)
weaponSelectFrame.Position = UDim2.new(0.5, -350, 0.5, -200)
weaponSelectFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
weaponSelectFrame.BorderSizePixel = 0
weaponSelectFrame.Parent = background

-- Game settings frame
local settingsFrame = Instance.new("Frame")
settingsFrame.Visible = false
settingsFrame.Size = UDim2.new(0, 700, 0, 400)
settingsFrame.Position = UDim2.new(0.5, -350, 0.5, -200)
settingsFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
settingsFrame.BorderSizePixel = 0
settingsFrame.Parent = background

-- Aimbot toggle frame
local aimbotFrame = Instance.new("Frame")
aimbotFrame.Visible = false
aimbotFrame.Size = UDim2.new(0, 700, 0, 400)
aimbotFrame.Position = UDim2.new(0.5, -350, 0.5, -200)
aimbotFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
aimbotFrame.BorderSizePixel = 0
aimbotFrame.Parent = background

-- Character buttons (example)
local charButtons = {}
local weapons = {"AK-47", "M4A1", "AWP", "Glock"}
local characters = {"Elite", "Assassin", "Marksman", "Recon"}

for i, char in ipairs(characters) do
    local button = Instance.new("TextButton")
    button.Text = char
    button.Size = UDim2.new(0, 120, 0, 50)
    button.Position = UDim2.new(0, (i-1)*130, 0.3, 0)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 18
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.MouseButton1Click:Connect(function()
        print("Selected character:", char)
        -- Apply character selection
        charSelectFrame.Visible = false
        weaponSelectFrame.Visible = true
    end)
    button.Parent = charSelectFrame
    table.insert(charButtons, button)
end

-- Weapon buttons
for i, weapon in ipairs(weapons) do
    local button = Instance.new("TextButton")
    button.Text = weapon
    button.Size = UDim2.new(0, 120, 0, 50)
    button.Position = UDim2.new(0, (i-1)*130, 0.3, 0)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 18
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.MouseButton1Click:Connect(function()
        print("Selected weapon:", weapon)
        -- Apply weapon selection
        weaponSelectFrame.Visible = false
        settingsFrame.Visible = true
    end)
    button.Parent = weaponSelectFrame
end

-- Settings controls
local difficulty = Instance.new("TextLabel")
difficulty.Text = "Difficulty:"
difficulty.Size = UDim2.new(0, 150, 0, 30)
difficulty.Position = UDim2.new(0.2, 0, 0.2, 0)
difficulty.Font = Enum.Font.SourceSansBold
difficulty.TextSize = 20
difficulty.TextColor3 = Color3.fromRGB(255, 255, 255)
difficulty.Parent = settingsFrame

local diffSlider = Instance.new("Slider")
diffSlider.Size = UDim2.new(0, 200, 0, 20)
diffSlider.Position = UDim2.new(0.5, 0, 0.2, 0)
diffSlider.Value = 50
diffSlider.Parent = settingsFrame

-- Crosshair settings
local crosshairLabel = Instance.new("TextLabel")
crosshairLabel.Text = "Crosshair:"
crosshairLabel.Size = UDim2.new(0, 150, 0, 30)
crosshairLabel.Position = UDim2.new(0.2, 0, 0.4, 0)
crosshairLabel.Font = Enum.Font.SourceSansBold
crosshairLabel.TextSize = 20
crosshairLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
crosshairLabel.Parent = settingsFrame

local crosshairColors = {
    {Color3.fromRGB(255, 255, 255), "White"},
    {Color3.fromRGB(255, 0, 0), "Red"},
    {Color3.fromRGB(0, 255, 0), "Green"},
    {Color3.fromRGB(0, 0, 255), "Blue"}
}

local crosshairButtons = {}
for i, color in ipairs(crosshairColors) do
    local button = Instance.new("TextButton")
    button.Text = color[2]
    button.Size = UDim2.new(0, 100, 0, 30)
    button.Position = UDim2.new(0, (i-1)*110, 0.5, 0)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 16
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.BackgroundColor3 = color[1]
    button.MouseButton1Click:Connect(function()
        print("Selected crosshair color:", color[2])
        -- Apply crosshair color
    end)
    button.Parent = settingsFrame
    table.insert(crosshairButtons, button)
end

-- Aimbot toggle
local aimbotToggle = Instance.new("TextButton")
aimbotToggle.Text = "AIMBOT: OFF"
aimbotToggle.Size = UDim2.new(0, 150, 0, 50)
aimbotToggle.Position = UDim2.new(0.5, -75, 0.6, 0)
aimbotToggle.Font = Enum.Font.SourceSansBold
aimbotToggle.TextSize = 24
aimbotToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotToggle.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
aimbotToggle.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    aimbotToggle.Text = "AIMBOT: " .. (aimbotEnabled and "ON" or "OFF")
    aimbotToggle.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
    
    -- Initialize aimbot if enabled
    if aimbotEnabled then
        initAimbot()
    else
        destroyAimbot()
    end
end)
aimbotToggle.Parent = settingsFrame

-- Start game button
local startBtn = Instance.new("TextButton")
startBtn.Text = "START GAME"
startBtn.Size = UDim2.new(0, 200, 0, 50)
startBtn.Position = UDim2.new(0.5, -100, 0.8, 0)
startBtn.Font = Enum.Font.SourceSansBold
startBtn.TextSize = 24
startBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
startBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
startBtn.MouseButton1Click:Connect(function()
    print("Starting game with difficulty:", math.floor(diffSlider.Value))
    -- Start the actual game
    settingsFrame.Visible = false
    -- Hide main menu
    mainMenu.Visible = false
end)
startBtn.Parent = settingsFrame

-- Back buttons
local backBtns = {}
for _, frame in ipairs({charSelectFrame, weaponSelectFrame, settingsFrame, aimbotFrame}) do
    local btn = Instance.new("TextButton")
    btn.Text = "BACK"
    btn.Size = UDim2.new(0, 100, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, 10)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    btn.MouseButton1Click:Connect(function()
        if frame == charSelectFrame then
            background.Visible = true
            charSelectFrame.Visible = false
        elseif frame == weaponSelectFrame then
            charSelectFrame.Visible = true
            weaponSelectFrame.Visible = false
        elseif frame == settingsFrame then
            weaponSelectFrame.Visible = true
            settingsFrame.Visible = false
        elseif frame == aimbotFrame then
            settingsFrame.Visible = true
            aimbotFrame.Visible = false
        end
    end)
    btn.Parent = frame
    table.insert(backBtns, btn)
end

-- Aimbot initialization function
function initAimbot()
    -- Find player's camera
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChild("Humanoid") or character:WaitForChild("Humanoid")
    local root = character:FindFirstChild("HumanoidRootPart") or character:WaitForChild("HumanoidRootPart")
    local camera = workspace.CurrentCamera
    
    -- Find closest enemy
    local closestEnemy = nil
    local minDistance = math.huge
    
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local distance = (player.Character.HumanoidRootPart.Position - root.Position).magnitude
            if distance < minDistance then
                minDistance = distance
                closestEnemy = player
            end
        end
    end
    
    -- Create aimbot part
    if closestEnemy and closestEnemy.Character then
        aimbotPart = Instance.new("Part")
        aimbotPart.Size = Vector3.new(0.1, 0.1, 0.1)
        aimbotPart.Anchored = true
        aimbotPart.CanCollide = false
        aimbotPart.Material = Enum.Material.Neon
        aimbotPart.BrickColor = BrickColor.new("Bright red")
        aimbotPart.Name = "AimbotTarget"
        
        -- Connect to humanoid health changes to track enemy health
        local enemyHumanoid = closestEnemy.Character:FindFirstChild("Humanoid")
        if enemyHumanoid then
            enemyHumanoid.HealthChanged:Connect(function(newHealth)
                if newHealth <= 0 then
                    destroyAimbot()
                end
            end)
        end
    end
end

-- Destroy aimbot function
function destroyAimbot()
    if aimbotPart then
        aimbotPart:Destroy()
        aimbotPart = nil
    end
end

-- Aimbot update loop
game:GetService("RunService").Heartbeat:Connect(function()
    if aimbotEnabled and aimbotPart then
        -- Find player's camera
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        local root = character:FindFirstChild("HumanoidRootPart") or character:WaitForChild("HumanoidRootPart")
        local camera = workspace.CurrentCamera
        
        -- Update aimbot target position
        local closestEnemy = nil
        local minDistance = math.huge
        
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (player.Character.HumanoidRootPart.Position - root.Position).magnitude
                if distance < minDistance then
                    minDistance = distance
                    closestEnemy = player
                end
            end
        end
        
        -- Move aimbot part to enemy head
        if closestEnemy and closestEnemy.Character then
            local enemyHead = closestEnemy.Character:FindFirstChild("Head")
            if enemyHead then
                aimbotPart.CFrame = CFrame.lookAt(root.Position, enemyHead.Position + Vector3.new(0, 1, 0))
            end
        end
    end
end)

-- Initial visibility
background.Visible = true
charSelectFrame.Visible = false
weaponSelectFrame.Visible = false
settingsFrame.Visible = false
aimbotFrame.Visible = false

-- Add a way to open the menu from the game world
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        local playerGui = player:WaitForChild("PlayerGui")
        local menu = mainMenu:Clone()
        menu.Parent = playerGui
        menu.Visible = true
        
        -- Handle menu closing
        local closeBtn = Instance.new("TextButton")
        closeBtn.Text = "CLOSE MENU"
        closeBtn.Size = UDim2.new(0, 150, 0, 40)
        closeBtn.Position = UDim2.new(0.5, -75, 0.9, 0)
        closeBtn.Font = Enum.Font.SourceSansBold
        closeBtn.TextSize = 18
        closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        closeBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        closeBtn.MouseButton1Click:Connect(function()
            menu.Visible = false
        end)
        closeBtn.Parent = menu
    end)
end)

print("Enhanced Sniper Arena menu loaded successfully!")
