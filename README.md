-- Teleportação com menu interativo no Roblox
local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local guiService = game:GetService("GuiService")

-- Configurações do menu
local menuConfig = {
    position = Vector2.new(50, 50),
    size = Vector2.new(200, 150),
    visible = false
}

-- Menu GUI
local menuFrame = Instance.new("ScreenGui")
menuFrame.Name = "TeleportMenu"
menuFrame.Parent = player:WaitForChild("PlayerGui")

-- Painel principal
local panel = Instance.new("Frame")
panel.Name = "Panel"
panel.Size = UDim2.new(0, menuConfig.size.X, 0, menuConfig.size.Y)
panel.Position = UDim2.new(0, menuConfig.position.X, 0, menuConfig.position.Y)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
panel.BorderSizePixel = 2
panel.BorderColor3 = Color3.fromRGB(80, 80, 80)
panel.Parent = menuFrame

-- Botões
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "Minimizar"
minimizeBtn.Size = UDim2.new(0, 100, 0, 30)
minimizeBtn.Position = UDim2.new(0, 10, 0, 40)
minimizeBtn.BackgroundTransparency = 0.3
minimizeBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
minimizeBtn.MouseButton1Click:Connect(function()
    menuConfig.visible = false
    panel.Visible = false
end)
minimizeBtn.Parent = panel

local closeBtn = Instance.new("TextButton")
closeBtn.Text = "Fechar"
closeBtn.Size = UDim2.new(0, 100, 0, 30)
closeBtn.Position = UDim2.new(0, 10, 0, 80)
closeBtn.BackgroundTransparency = 0.3
closeBtn.TextColor3 = Color3.fromRGB(200, 100, 100)
closeBtn.MouseButton1Click:Connect(function()
    menuFrame:Destroy()
end)
closeBtn.Parent = panel

-- Teleporte para coordenadas
local function teleportToTarget(position)
    if player and player.Character then
        local rootPart = player.Character:FindFirstChild("HumanoidRootPart") 
                      or player.Character:FindFirstChild("Head")
        
        if rootPart then
            rootPart.CFrame = CFrame.new(position)
            print("Teleportado para:", position)
        end
    end
end

-- Coordenadas predefinidas
local locations = {
    {"Local A", Vector3.new(100, 50, 200)},
    {"Local B", Vector3.new(-100, 75, -150)},
    {"Local C", Vector3.new(300, 100, 400)}
}

-- Criar botões para locais
for i, location in ipairs(locations) do
    local btn = Instance.new("TextButton")
    btn.Text = location[1]
    btn.Size = UDim2.new(0, 150, 0, 30)
    btn.Position = UDim2.new(0, 25, 0, 120 + (i * 40))
    btn.BackgroundTransparency = 0.4
    btn.TextColor3 = Color3.fromRGB(150, 255, 150)
    btn.MouseButton1Click:Connect(function()
        teleportToTarget(location[2])
    end)
    btn.Parent = panel
end

-- Ativar/Desativar menu com tecla
mouse.KeyDown:Connect(function(key)
    if key == "m" then
        menuConfig.visible = not menuConfig.visible
        panel.Visible = menuConfig.visible
    end
end)

-- Inicializar
panel.Visible = false
print("Menu de teletransporte carregado - Pressione 'M' para abrir")
