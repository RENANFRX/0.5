--[[
    RENAN FRX | Scripts - v4.0
    Auto Farm Local (Client-Side Only)
    Para uso em modo Play Solo no Roblox Studio
    SEM necessidade de scripts no servidor
--]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- ============================================
-- CONFIGURAÇÕES
-- ============================================
local CONFIG = {
    -- Palavras-chave para detectar clientes/rotas no jogo
    CLIENTES_KEYWORDS = {
        "cliente", "client", "customer",
        "rota", "route",
        "ober",
        "entrega", "delivery",
        "destino", "destination",
        "waypoint", "marcador", "marker",
        "point", "ponto",
        "target", "alvo",
        "coleta", "pickup",
        "drop", "dropoff"
    },
    
    -- Cores de objetos que podem indicar rota (partes específicas)
    CORES_INDICADORAS = {
        Color3.fromRGB(255, 255, 0), -- Amarelo
        Color3.fromRGB(0, 255, 0),   -- Verde
        Color3.fromRGB(255, 0, 0),   -- Vermelho
        Color3.fromRGB(0, 255, 255), -- Ciano
        Color3.fromRGB(255, 100, 0), -- Laranja
    },
    
    ALTURA_TELEPORTE = 5,
    TEMPO_ESPERA = 2,
    RAIO_BUSCA = 500
}

-- ============================================
-- VARIÁVEIS GLOBAIS
-- ============================================
local autoFarmAtivo = false
local rotaAtual = nil
local clienteAtual = nil
local destinoAtual = nil

-- ============================================
-- FUNÇÕES DE DETECÇÃO AVANÇADA
-- ============================================

-- Função para verificar se uma parte tem cor indicadora
local function temCorIndicadora(part)
    if not part:IsA("BasePart") then return false end
    
    local cor = part.Color
    for _, corIndicadora in ipairs(CONFIG.CORES_INDICADORAS) do
        if cor == corIndicadora then
            return true
        end
    end
    return false
end

-- Função para verificar se é um cliente/rota
local function ehClienteOuRota(obj)
    local nome = obj.Name:lower()
    
    -- Verifica nome
    for _, palavra in ipairs(CONFIG.CLIENTES_KEYWORDS) do
        if nome:find(palavra) then
            return true
        end
    end
    
    -- Verifica cor (se for BasePart)
    if obj:IsA("BasePart") then
        if temCorIndicadora(obj) then
            return true
        end
    end
    
    -- Verifica se tem Highlight ou Outline (indicadores visuais)
    if obj:FindFirstChild("Highlight") or obj:FindFirstChild("Outline") then
        return true
    end
    
    return false
end

-- Função para encontrar todos os clientes/rotas
local function encontrarTodosClientes()
    local char = Player.Character
    if not char then return {} end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return {} end
    
    local clientes = {}
    
    -- Procura em Models
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and ehClienteOuRota(obj) then
            local posicao = obj:GetPivot().Position
            local distancia = (hrp.Position - posicao).Magnitude
            
            if distancia <= CONFIG.RAIO_BUSCA then
                table.insert(clientes, {
                    objeto = obj,
                    posicao = posicao,
                    distancia = distancia,
                    nome = obj.Name,
                    tipo = "Model"
                })
            end
        end
    end
    
    -- Procura em BaseParts (marcadores individuais)
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and ehClienteOuRota(obj) then
            local posicao = obj.Position
            local distancia = (hrp.Position - posicao).Magnitude
            
            if distancia <= CONFIG.RAIO_BUSCA then
                -- Evita duplicatas
                local duplicado = false
                for _, c in ipairs(clientes) do
                    if (c.posicao - posicao).Magnitude < 1 then
                        duplicado = true
                        break
                    end
                end
                
                if not duplicado then
                    table.insert(clientes, {
                        objeto = obj,
                        posicao = posicao,
                        distancia = distancia,
                        nome = obj.Name,
                        tipo = "Part"
                    })
                end
            end
        end
    end
    
    -- Também procura por BillboardGuis com texto indicativo
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BillboardGui") or obj:IsA("SurfaceGui") then
            for _, child in ipairs(obj:GetDescendants()) do
                if child:IsA("TextLabel") then
                    local texto = child.Text:lower()
                    for _, palavra in ipairs(CONFIG.CLIENTES_KEYWORDS) do
                        if texto:find(palavra) then
                            local parent = obj.Parent
                            if parent then
                                local posicao = parent:IsA("Model") and parent:GetPivot().Position or (parent:IsA("BasePart") and parent.Position)
                                if posicao then
                                    local distancia = (hrp.Position - posicao).Magnitude
                                    if distancia <= CONFIG.RAIO_BUSCA then
                                        table.insert(clientes, {
                                            objeto = parent,
                                            posicao = posicao,
                                            distancia = distancia,
                                            nome = parent.Name .. " - " .. child.Text,
                                            tipo = "GUI"
                                        })
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    
    -- Ordena por distância
    table.sort(clientes, function(a, b)
        return a.distancia < b.distancia
    end)
    
    return clientes
end

-- Função para separar cliente e destino
local function identificarClienteEDestino(clientes)
    if #clientes < 1 then return nil, nil end
    
    -- O primeiro (mais próximo) geralmente é o ponto de coleta
    local pickup = clientes[1]
    
    -- O segundo (se existir) é o destino
    local delivery = nil
    if #clientes >= 2 then
        -- Pega o mais distante como destino de entrega
        delivery = clientes[#clientes]
    end
    
    return pickup, delivery
end

-- Função para encontrar veículo próximo
local function encontrarVeiculoProximo(raio)
    local char = Player.Character
    if not char then return nil end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    
    raio = raio or 50
    
    local veiculosProximos = {}
    
    for _, obj in ipairs(Workspace:GetDescendants()) do
        -- Procura por assentos de veículo
        if obj:IsA("VehicleSeat") then
            local distancia = (hrp.Position - obj.Position).Magnitude
            if distancia <= raio then
                table.insert(veiculosProximos, {
                    seat = obj,
                    vehicle = obj.Parent,
                    distancia = distancia,
                    posicao = obj.Position
                })
            end
        end
        
        -- Procura por outros tipos de assento
        if obj:IsA("Seat") and obj.Parent:IsA("Model") and not obj:IsA("VehicleSeat") then
            local distancia = (hrp.Position - obj.Position).Magnitude
            if distancia <= raio then
                table.insert(veiculosProximos, {
                    seat = obj,
                    vehicle = obj.Parent,
                    distancia = distancia,
                    posicao = obj.Position
                })
            end
        end
    end
    
    -- Ordena por distância
    table.sort(veiculosProximos, function(a, b)
        return a.distancia < b.distancia
    end)
    
    return veiculosProximos
end

-- ============================================
-- FUNÇÕES DE MOVIMENTAÇÃO
-- ============================================

-- Teleportar personagem para posição
local function teleportarPersonagem(posicao)
    local char = Player.Character
    if not char then return false end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    
    -- Método de teleporte compatível com vários executores
    local targetCFrame = CFrame.new(posicao + Vector3.new(0, CONFIG.ALTURA_TELEPORTE, 0))
    
    -- Tenta múltiplos métodos
    pcall(function()
        char:PivotTo(targetCFrame)
    end)
    
    if (hrp.Position - targetCFrame.Position).Magnitude > 10 then
        pcall(function()
            hrp.CFrame = targetCFrame
        end)
    end
    
    return (hrp.Position - targetCFrame.Position).Magnitude < 10
end

-- Teleportar veículo
local function teleportarVeiculo(vehicleSeat, posicao)
    if not vehicleSeat or not posicao then return false end
    
    local vehicleModel = vehicleSeat.Parent
    if not vehicleModel then return false end
    
    local targetPos = posicao + Vector3.new(0, CONFIG.ALTURA_TELEPORTE, 0)
    
    -- Método 1: PivotTo do modelo
    pcall(function()
        vehicleModel:PivotTo(CFrame.new(targetPos))
    end)
    
    -- Método 2: SetPrimaryPartCFrame
    local primaryPart = vehicleModel.PrimaryPart
    if primaryPart then
        pcall(function()
            vehicleModel:SetPrimaryPartCFrame(CFrame.new(targetPos))
        end)
    end
    
    -- Método 3: Mover a parte principal manualmente
    local mainPart = vehicleModel.PrimaryPart or vehicleModel:FindFirstChild("HumanoidRootPart")
    if mainPart then
        pcall(function()
            mainPart.CFrame = CFrame.new(targetPos)
        end)
    end
    
    return true
end

-- Sentar no veículo
local function sentarNoVeiculo(seat)
    local char = Player.Character
    if not char then return false end
    
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return false end
    
    -- Verifica se o assento já está ocupado
    if seat.Occupant and seat.Occupant ~= humanoid then
        print("⚠️ Assento ocupado por outro jogador")
        return false
    end
    
    -- Teleporta o personagem para perto do assento
    local seatPos = seat.Position
    teleportarPersonagem(seatPos)
    
    -- Aguarda e senta
    task.wait(0.3)
    
    pcall(function()
        seat:Sit(humanoid)
    end)
    
    -- Verifica se sentou
    task.wait(0.5)
    return seat.Occupant == humanoid
end

-- ============================================
-- SCANNER DO JOGO (PARA DEBUG)
-- ============================================

local function scanGame()
    print("=" .. string.rep("=", 50))
    print("🔍 SCANEANDO O JOGO...")
    print("=" .. string.rep("=", 50))
    
    local clientes = encontrarTodosClientes()
    print("📦 Total de pontos encontrados: " .. #clientes)
    
    for i, cliente in ipairs(clientes) do
        print(string.format("[%d] %s", i, cliente.nome))
        print("    Tipo: " .. cliente.tipo)
        print("    Distância: " .. math.floor(cliente.distancia) .. "m")
        print("    Posição: " .. tostring(cliente.posicao))
        print("    ---")
    end
    
    print("=" .. string.rep("=", 50))
end

-- ============================================
-- CICLO DO AUTO FARM
-- ============================================

local function cicloAutoFarm()
    while autoFarmAtivo do
        print("🔄 Buscando pontos no mapa...")
        
        -- Encontra todos os pontos
        local todosPontos = encontrarTodosClientes()
        
        if #todosPontos > 0 then
            -- Identifica pickup e delivery
            local pickup, delivery = identificarClienteEDestino(todosPontos)
            
            if pickup then
                print("📍 Ponto de coleta: " .. pickup.nome .. " (" .. math.floor(pickup.distancia) .. "m)")
                
                -- PASSO 1: Teleportar para o ponto de coleta
                print("✈️ Indo para coleta...")
                local tpOk = teleportarPersonagem(pickup.posicao)
                
                if tpOk then
                    task.wait(0.5)
                    
                    -- PASSO 2: Procurar veículo
                    print("🚗 Procurando veículo próximo...")
                    local veiculos = encontrarVeiculoProximo(50)
                    
                    if #veiculos > 0 then
                        local veiculo = veiculos[1]
                        print("✅ Veículo encontrado: " .. veiculo.vehicle.Name)
                        
                        -- PASSO 3: Sentar no veículo
                        print("🪑 Entrando no veículo...")
                        local sentado = sentarNoVeiculo(veiculo.seat)
                        
                        if sentado then
                            print("✅ Motorista a bordo!")
                            task.wait(1)
                            
                            -- PASSO 4: Definir destino
                            if delivery then
                                print("🎯 Destino definido: " .. delivery.nome)
                                destinoAtual = delivery
                            elseif #todosPontos >= 2 then
                                -- Usa o segundo ponto mais distante como destino
                                destinoAtual = todosPontos[#todosPontos]
                                print("🎯 Usando ponto distante como destino: " .. destinoAtual.nome)
                            else
                                -- Se só tem um ponto, procura outro diferente
                                for i = #todosPontos, 1, -1 do
                                    if todosPontos[i].objeto ~= pickup.objeto then
                                        destinoAtual = todosPontos[i]
                                        print("🎯 Destino alternativo: " .. destinoAtual.nome)
                                        break
                                    end
                                end
                            end
                            
                            -- PASSO 5: Teleportar veículo para o destino
                            if destinoAtual then
                                print("✈️ Teleportando veículo para destino...")
                                teleportarVeiculo(veiculo.seat, destinoAtual.posicao)
                                task.wait(2)
                                
                                -- PASSO 6: Descer do veículo (opcional)
                                local char = Player.Character
                                if char and char.Humanoid then
                                    pcall(function()
                                        char.Humanoid.Sit = false
                                    end)
                                end
                                
                                print("✅ Ciclo completo!")
                            else
                                print("⚠️ Nenhum destino encontrado")
                            end
                        else
                            print("❌ Falha ao entrar no veículo")
                        end
                    else
                        print("⚠️ Nenhum veículo encontrado próximo")
                    end
                end
            end
        else
            print("⚠️ Nenhum ponto encontrado no mapa")
        end
        
        print("⏳ Aguardando " .. CONFIG.TEMPO_ESPERA .. " segundos...")
        task.wait(CONFIG.TEMPO_ESPERA)
    end
end

-- ============================================
-- INTERFACE DO USUÁRIO
-- ============================================

local function criarUI()
    -- ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "RENAN FRX | Scripts"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = PlayerGui
    
    -- Frame Principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 450, 0, 400)
    MainFrame.Position = UDim2.new(0.5, -225, 0.5, -200)
    MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui
    
    -- Borda decorativa
    local Border = Instance.new("Frame")
    Border.Size = UDim2.new(1, 4, 1, 4)
    Border.Position = UDim2.new(0, -2, 0, -2)
    Border.BackgroundColor3 = Color3.fromRGB(255, 100, 50)
    Border.BorderSizePixel = 0
    Border.ZIndex = -1
    Border.Parent = MainFrame
    
    -- Título
    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 45)
    TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame
    
    local Title = Instance.new("TextLabel")
    Title.Text = "🚛 RENAN FRX | SCRIPTS"
    Title.Size = UDim2.new(0, 350, 1, 0)
    Title.Position = UDim2.new(0, 15, 0, 0)
    Title.BackgroundTransparency = 1
    Title.TextColor3 = Color3.fromRGB(255, 120, 30)
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Font = Enum.Font.GothamBlack
    Title.TextSize = 18
    Title.Parent = TitleBar
    
    -- Botão Minimizar
    local MinimizeBtn = Instance.new("TextButton")
    MinimizeBtn.Text = "─"
    MinimizeBtn.Size = UDim2.new(0, 40, 0, 35)
    MinimizeBtn.Position = UDim2.new(1, -85, 0, 5)
    MinimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
    MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinimizeBtn.Font = Enum.Font.GothamBold
    MinimizeBtn.TextSize = 22
    MinimizeBtn.BorderSizePixel = 0
    MinimizeBtn.Parent = TitleBar
    
    -- Container do conteúdo
    local Content = Instance.new("Frame")
    Content.Size = UDim2.new(1, 0, 1, -45)
    Content.Position = UDim2.new(0, 0, 0, 45)
    Content.BackgroundTransparency = 1
    Content.Parent = MainFrame
    
    -- Variável de minimizado
    local minimized = false
    MinimizeBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            MainFrame.Size = UDim2.new(0, 450, 0, 45)
            Content.Visible = false
        else
            MainFrame.Size = UDim2.new(0, 450, 0, 400)
            Content.Visible = true
        end
    end)
    
    -- Botão Fechar/Abrir
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Text = "✕"
    CloseBtn.Size = UDim2.new(0, 40, 0, 35)
    CloseBtn.Position = UDim2.new(1, -40, 0, 5)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 20
    CloseBtn.BorderSizePixel = 0
    CloseBtn.Parent = TitleBar
    
    local fechado = false
    CloseBtn.MouseButton1Click:Connect(function()
        fechado = true
        ScreenGui.Enabled = false
    end)
    
    -- Reabrir com INSERT
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.KeyCode == Enum.KeyCode.Insert and fechado then
            fechado = false
            ScreenGui.Enabled = true
        end
    end)
    
    -- ============================================
    -- BOTÕES DO MENU
    -- ============================================
    local yPos = 10
    
    -- Scanner
    local ScanBtn = Instance.new("TextButton")
    ScanBtn.Text = "🔍 ESCANEAR MAPA"
    ScanBtn.Size = UDim2.new(1, -20, 0, 45)
    ScanBtn.Position = UDim2.new(0, 10, 0, yPos)
    ScanBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    ScanBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ScanBtn.Font = Enum.Font.GothamBold
    ScanBtn.TextSize = 14
    ScanBtn.BorderSizePixel = 0
    ScanBtn.Parent = Content
    ScanBtn.MouseButton1Click:Connect(scanGame)
    
    yPos = yPos + 55
    
    -- Auto Farm Toggle
    local FarmBtn = Instance.new("TextButton")
    FarmBtn.Text = "🤖 AUTO FARM: OFF"
    FarmBtn.Size = UDim2.new(1, -20, 0, 50)
    FarmBtn.Position = UDim2.new(0, 10, 0, yPos)
    FarmBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    FarmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    FarmBtn.Font = Enum.Font.GothamBold
    FarmBtn.TextSize = 16
    FarmBtn.BorderSizePixel = 0
    FarmBtn.Parent = Content
    
    FarmBtn.MouseButton1Click:Connect(function()
        autoFarmAtivo = not autoFarmAtivo
        
        if autoFarmAtivo then
            FarmBtn.Text = "🤖 AUTO FARM: ON"
            FarmBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
            print("🚀 Auto Farm ATIVADO!")
            coroutine.wrap(cicloAutoFarm)()
        else
            FarmBtn.Text = "🤖 AUTO FARM: OFF"
            FarmBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            print("🛑 Auto Farm DESATIVADO!")
        end
    end)
    
    yPos = yPos + 60
    
    -- Botão Ir ao Cliente
    local IrPickupBtn = Instance.new("TextButton")
    IrPickupBtn.Text = "📍 IR AO PONTO DE COLETA"
    IrPickupBtn.Size = UDim2.new(1, -20, 0, 40)
    IrPickupBtn.Position = UDim2.new(0, 10, 0, yPos)
    IrPickupBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    IrPickupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    IrPickupBtn.Font = Enum.Font.GothamBold
    IrPickupBtn.TextSize = 13
    IrPickupBtn.BorderSizePixel = 0
    IrPickupBtn.Parent = Content
    
    IrPickupBtn.MouseButton1Click:Connect(function()
        local pontos = encontrarTodosClientes()
        if #pontos > 0 then
            local pickup = pontos[1]
            teleportarPersonagem(pickup.posicao)
            print("✈️ Teleportado para: " .. pickup.nome)
        else
            print("❌ Nenhum ponto encontrado!")
        end
    end)
    
    yPos = yPos + 50
    
    -- Botão Entrar no Veículo
    local VeiculoBtn = Instance.new("TextButton")
    VeiculoBtn.Text = "🚗 ENTRAR NO VEÍCULO"
    VeiculoBtn.Size = UDim2.new(1, -20, 0, 40)
    VeiculoBtn.Position = UDim2.new(0, 10, 0, yPos)
    VeiculoBtn.BackgroundColor3 = Color3.fromRGB(120, 50, 200)
    VeiculoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    VeiculoBtn.Font = Enum.Font.GothamBold
    VeiculoBtn.TextSize = 13
    VeiculoBtn.BorderSizePixel = 0
    VeiculoBtn.Parent = Content
    
    VeiculoBtn.MouseButton1Click:Connect(function()
        local veiculos = encontrarVeiculoProximo(50)
        if #veiculos > 0 then
            sentarNoVeiculo(veiculos[1].seat)
            print("🪑 Entrando no veículo: " .. veiculos[1].vehicle.Name)
        else
            print("❌ Nenhum veículo próximo!")
        end
    end)
    
    yPos = yPos + 50
    
    -- Botão Ir ao Destino
    local IrDestinoBtn = Instance.new("TextButton")
    IrDestinoBtn.Text = "🎯 IR AO DESTINO"
    IrDestinoBtn.Size = UDim2.new(1, -20, 0, 40)
    IrDestinoBtn.Position = UDim2.new(0, 10, 0, yPos)
    IrDestinoBtn.BackgroundColor3 = Color3.fromRGB(255, 100, 0)
    IrDestinoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    IrDestinoBtn.Font = Enum.Font.GothamBold
    IrDestinoBtn.TextSize = 13
    IrDestinoBtn.BorderSizePixel = 0
    IrDestinoBtn.Parent = Content
    
    IrDestinoBtn.MouseButton1Click:Connect(function()
        local char = Player.Character
        if not char or not char.Humanoid.SeatPart then
            print("❌ Você precisa estar em um veículo!")
            return
        end
        
        local pontos = encontrarTodosClientes()
        if #pontos >= 2 then
            local destino = pontos[#pontos] -- Ponto mais distante
            local seat = char.Humanoid.SeatPart
            teleportarVeiculo(seat, destino.posicao)
            print("✈️ Veículo teleportado para: " .. destino.nome)
        else
            print("❌ Destino não encontrado!")
        end
    end)
    
    yPos = yPos + 50
    
    -- Status
    local Status = Instance.new("TextLabel")
    Status.Text = "INSERT = Abrir/Fechar | F9 = Console"
    Status.Size = UDim2.new(1, -20, 0, 25)
    Status.Position = UDim2.new(0, 10, 0, yPos + 20)
    Status.BackgroundTransparency = 1
    Status.TextColor3 = Color3.fromRGB(150, 150, 150)
    Status.Font = Enum.Font.Gotham
    Status.TextSize = 11
    Status.TextXAlignment = Enum.TextXAlignment.Center
    Status.Parent = Content
    
    return ScreenGui
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================

print("╔══════════════════════════════════╗")
print("║  RENAN FRX | Scripts v4.0      ║")
print("║  Modo Local - Client Side      ║")
print("║  Pressione INSERT para abrir   ║")
print("╚══════════════════════════════════╝")

-- Cria a interface
local ui = criarUI()

-- Scan inicial
task.wait(1)
print("🔍 Execute o scan para ver os pontos disponíveis!")
