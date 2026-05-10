-- Teleportação com menu interativo no Roblox
local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

-- Configurações do menu
local menuVisible = false
local menuFrame = nil

-- Função para criar o menu
local function createMenu()
    if menuFrame then return end
    
    menuFrame = Instance.new("ScreenGui")
    menuFrame.Name = "TeleportMenu"
    menuFrame.ResetOnSpawn = false
    menuFrame.Parent = player:WaitForChild("PlayerGui")
    
    -- Painel
    local panel = Instance.new("Frame")
    panel.Name = "Panel"
    panel.Size = UDim2.new(0, 200, 0, 180)
    panel.Position = UDim2.new(0.5, -100, 0.5, -90)
    panel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    panel.BorderSizePixel = 2
    panel.BorderColor3 = Color3.fromRGB(80, 80, 80)
    panel.Visible = false
    panel.Parent = menuFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Text = "Teleportação"
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(200, 200, 200)
    title.FontSize = Enum.FontSize.Size18
    title.TextScaled = true
    title.Parent = panel
    
    -- Botões de localização
    local locations = {
        {"Local A", Vector3.new(100, 50, 200)},
        {"Local B", Vector3.new(-100, 75, -150)},
        {"Local C", Vector3.new(300, 100, 400)}
    }
    
    for i, loc in ipairs(locations) do
        local btn = Instance.new("TextButton")
        btn.Text = loc[1]
        btn.Size = UDim2.new(0, 150, 0, 30)
        btn.Position = UDim2.new(0, 25, 0, 50 + (i*35))
        btn.BackgroundTransparency = 0.3
        btn.TextColor3 = Color3.fromRGB(150, 255, 150)
        btn.MouseButton1Click:Connect(function()
            if player and player.Character then
                local root = player.Character:FindFirstChild("HumanoidRootPart") or player.Character:FindFirstChild("Head")
                if root then
                    root.CFrame = CFrame.new(loc[2])
                    print("Teleportado para:", loc[1])
                end
            end
        end)
        btn.Parent = panel
    end
    
    -- Botões de controle
    local minBtn = Instance.new("TextButton")
    minBtn.Text = "Minimizar"
    minBtn.Size = UDim2.new(0, 80, 0, 25)
    minBtn.Position = UDim2.new(0, 10, 0, 145)
    minBtn.BackgroundTransparency = 0.4
    minBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    minBtn.MouseButton1Click:Connect(function()
        menuVisible = false
        panel.Visible = false
    end)
    minBtn.Parent = panel
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Text = "Fechar"
    closeBtn.Size = UDim2.new(0, 80, 0, 25)
    closeBtn.Position = UDim2.new(0, 110, 0, 145)
    closeBtn.BackgroundTransparency = 0.4
    closeBtn.TextColor3 = Color3.fromRGB(200, 100, 100)
    closeBtn.MouseButton1Click:Connect(function()
        menuFrame:Destroy()
        menuFrame = nil
    end)
    closeBtn.Parent = panel
    
    return panel
end

-- Função para alternar o menu
local function toggleMenu()
    if not menuFrame then
        createMenu()
    end
    
    menuVisible = not menuVisible
    menuFrame:FindFirstChild("Panel").Visible = menuVisible
end

-- Função para teleportar para posição válida
local function teleportToValidPosition(position)
    if not player or not player.Character then return end
    
    local root = player.Character:FindFirstChild("HumanoidRootPart") or player.Character:FindFirstChild("Head")
    if not root then return end
    
    -- Raycast para encontrar o chão
    local ray = Ray.new(position, Vector3.new(0, -100, 0))
    local hit, pos = workspace:FindPartOnRay(ray, player.Character)
    
    if hit then
        -- Verifica se o ponto está acima do chão
        if pos.Y > position.Y then
            pos = Vector3.new(pos.X, position.Y + 10, pos.Z)
        end
        
        root.CFrame = CFrame.new(pos)
        print("Teleportado para posição válida:", pos)
    else
        -- Se não encontrou chão, tenta encontrar um bloco próximo
        local searchRadius = 50
        for x = -searchRadius, searchRadius, 10 do
            for z = -searchRadius, searchRadius, 10 do
                local testPos = Vector3.new(position.X + x, position.Y, position.Z + z)
                local rayTest = Ray.new(testPos, Vector3.new(0, -100, 0))
                local hitTest, posTest = workspace:FindPartOnRay(rayTest, player.Character)
                
                if hitTest then
                    pos = Vector3.new(posTest.X, position.Y + 10, posTest.Z)
                    root.CFrame = CFrame.new(pos)
                    print("Teleportado para posição válida:", pos)
                    return
                end
            end
        end
        
        -- Se não encontrou nada, usa posição original com Y+10
        pos = Vector3.new(position.X, position.Y + 10, position.Z)
        root.CFrame = CFrame.new(pos)
        print("Teleportado para posição original com ajuste:", pos)
    end
end

-- Evento de clique do mouse para teleportação
mouse.Button1Down:Connect(function()
    if menuVisible then
        -- Teleporta para o local mais próximo do cursor
        teleportToValidPosition(mouse.Hit.Position)
    end
end)

-- Evento de tecla para alternar menu
mouse.KeyDown:Connect(function(key)
    if key == "m" then
        toggleMenu()
    end
end)

print("Menu de teletransporte carregado - Clique com o botão esquerdo para teleportar ou pressione 'M' para abrir menu")
