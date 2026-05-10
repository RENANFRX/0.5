-- Teleportação complexa com menu interativo no Roblox
local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local runService = game:GetService("RunService")
local replicatedStorage = game:GetService("ReplicatedStorage")
local playerGui = player:WaitForChild("PlayerGui")

-- Configurações do menu
local menuVisible = false
local menuFrame = nil
local maxDistance = 1000 -- Distância máxima do mapa
local teleportCooldown = 1 -- Segundos entre teletransportações
local lastTeleportTime = 0

-- Sistema de cache de locais
local locationCache = {}
local locationHistory = {}

-- Função para criar o menu
local function createMenu()
    if menuFrame then return end
    
    menuFrame = Instance.new("ScreenGui")
    menuFrame.Name = "TeleportMenu"
    menuFrame.ResetOnSpawn = false
    menuFrame.Parent = playerGui
    
    -- Painel principal
    local panel = Instance.new("Frame")
    panel.Name = "Panel"
    panel.Size = UDim2.new(0, 250, 0, 250)
    panel.Position = UDim2.new(0.5, -125, 0.5, -125)
    panel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    panel.BorderSizePixel = 2
    panel.BorderColor3 = Color3.fromRGB(80, 80, 80)
    panel.Visible = false
    panel.Parent = menuFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Text = "Teleportação Avançada"
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(200, 200, 200)
    title.FontSize = Enum.FontSize.Size18
    title.TextScaled = true
    title.Parent = panel
    
    -- Botões de localização
    local locations = {
        {"Base A", Vector3.new(100, 50, 200)},
        {"Base B", Vector3.new(-100, 75, -150)},
        {"Base C", Vector3.new(300, 100, 400)},
        {"Safe Zone", Vector3.new(0, 100, 0)},
        {"Tower Top", Vector3.new(500, 200, 500)}
    }
    
    for i, loc in ipairs(locations) do
        local btn = Instance.new("TextButton")
        btn.Text = loc[1]
        btn.Size = UDim2.new(0, 150, 0, 30)
        btn.Position = UDim2.new(0, 50, 0, 50 + (i*35))
        btn.BackgroundTransparency = 0.3
        btn.TextColor3 = Color3.fromRGB(150, 255, 150)
        btn.MouseButton1Click:Connect(function()
            teleportToLocation(loc[2], loc[1])
        end)
        btn.Parent = panel
    end
    
    -- Botões de controle
    local historyBtn = Instance.new("TextButton")
    historyBtn.Text = "Histórico"
    historyBtn.Size = UDim2.new(0, 80, 0, 25)
    historyBtn.Position = UDim2.new(0, 10, 0, 210)
    historyBtn.BackgroundTransparency = 0.4
    historyBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    historyBtn.MouseButton1Click:Connect(function()
        showHistory()
    end)
    historyBtn.Parent = panel
    
    local clearBtn = Instance.new("TextButton")
    clearBtn.Text = "Limpar"
    clearBtn.Size = UDim2.new(0, 80, 0, 25)
    clearBtn.Position = UDim2.new(0, 95, 0, 210)
    clearBtn.BackgroundTransparency = 0.4
    clearBtn.TextColor3 = Color3.fromRGB(200, 150, 100)
    clearBtn.MouseButton1Click:Connect(function()
        clearHistory()
    end)
    clearBtn.Parent = panel
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Text = "Fechar"
    closeBtn.Size = UDim2.new(0, 80, 0, 25)
    closeBtn.Position = UDim2.new(0, 180, 0, 210)
    closeBtn.BackgroundTransparency = 0.4
    closeBtn.TextColor3 = Color3.fromRGB(200, 100, 100)
    closeBtn.MouseButton1Click:Connect(function()
        menuFrame:Destroy()
        menuFrame = nil
    end)
    closeBtn.Parent = panel
    
    return panel
end

-- Função para teleportar para local
local function teleportToLocation(position, name)
    local currentTime = tick()
    if currentTime - lastTeleportTime < teleportCooldown then
        print("Aguarde " .. math.ceil(teleportCooldown - (currentTime - lastTeleportTime)) .. " segundos")
        return
    end
    
    if not player or not player.Character then return end
    
    local root = player.Character:FindFirstChild("HumanoidRootPart") or player.Character:FindFirstChild("Head")
    if not root then return end
    
    -- Verifica se a posição está dentro do mapa
    local center = Vector3.new(0, 0, 0)
    local distance = (position - center).Magnitude
    
    if distance > maxDistance then
        print("Posição fora do mapa! Teleportando para posição segura.")
        position = Vector3.new(0, 100, 0)
    end
    
    -- Raycast para encontrar o chão
    local ray = Ray.new(position, Vector3.new(0, -100, 0))
    local hit, pos = workspace:FindPartOnRay(ray, player.Character)
    
    if hit then
        -- Verifica se o ponto está acima do chão
        if pos.Y > position.Y then
            pos = Vector3.new(pos.X, position.Y + 10, pos.Z)
        end
        
        root.CFrame = CFrame.new(pos)
        lastTeleportTime = currentTime
        addToHistory(name, pos)
        print("Teleportado para:", name)
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
                    lastTeleportTime = currentTime
                    addToHistory(name, pos)
                    print("Teleportado para:", name)
                    return
                end
            end
        end
        
        -- Se não encontrou nada, usa posição original com Y+10
        pos = Vector3.new(position.X, position.Y + 10, position.Z)
        root.CFrame = CFrame.new(pos)
        lastTeleportTime = currentTime
        addToHistory(name, pos)
        print("Teleportado para posição original com ajuste:", name)
    end
end

-- Função para adicionar ao histórico
local function addToHistory(name, position)
    table.insert(locationHistory, 1, {name = name, position = position})
    if #locationHistory > 10 then
        table.remove(locationHistory, #locationHistory)
    end
end

-- Função para mostrar histórico
local function showHistory()
    if #locationHistory == 0 then
        print("Histórico vazio")
        return
    end
    
    print("\n=== Histórico de Teleportações ===")
    for i, entry in ipairs(locationHistory) do
        print(i .. ". " .. entry.name .. " (" .. tostring(entry.position) .. ")")
    end
    print("=============================\n")
end

-- Função para limpar histórico
local function clearHistory()
    locationHistory = {}
    print("Histórico limpo")
end

-- Função para alternar o menu
local function toggleMenu()
    if not menuFrame then
        createMenu()
    end
    
    menuVisible = not menuVisible
    menuFrame:FindFirstChild("Panel").Visible = menuVisible
end

-- Evento de tecla para alternar menu
mouse.KeyDown:Connect(function(key)
    if key == "m" then
        toggleMenu()
    end
end)

print("Menu de teletransporte carregado - Pressione 'M' para abrir menu")
