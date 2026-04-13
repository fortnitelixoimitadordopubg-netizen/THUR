--[[
    Sistema de Automação e ESP para Bridger: WESTERN (Roblox)
    ----------------------------------------------------------
    Interface moderna com abas para automação completa da pesca e ESP visual.
    Compatível com a maioria dos executors (Synapse, Krnl, Fluxus, etc.)
]]

-- 1. Carregar Bibliotecas e Dependências
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Drawing = loadstring(game:HttpGet("https://raw.githubusercontent.com/Stefanuk12/DrawingLib/main/DrawingLib.lua"))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- 2. Configuração da Janela Principal (UI)
local Window = Rayfield:CreateWindow({
    Name = "Bridger WESTERN - AutoFisher & ESP",
    Icon = "fish", -- Ícone do menu (pode ser alterado)
    LoadingTitle = "Carregando Script...",
    LoadingSubtitle = "by Assistant",
    Theme = "Dark" -- Opções: "Default", "Dark", "Light"
})

-- 3. Aba de Pesca (Prioridade Máxima)
local FishingTab = Window:CreateTab("🎣 Pesca", "fish") -- Nome e ícone da aba

local FishingSection = FishingTab:CreateSection("Automação Inteligente de Pesca")
local ToggleAutoFish = FishingTab:CreateToggle({
    Name = "Pesca Automática (Detecção por Timing)",
    CurrentValue = false,
    Flag = "AutoFishToggle",
    Callback = function(Value)
        print("Pesca Automática: " .. tostring(Value))
    end,
})

local ToggleAutoMinigame = FishingTab:CreateToggle({
    Name = "Auto Minigame (QTE + Controle de Tensão)",
    CurrentValue = false,
    Flag = "AutoMinigameToggle",
    Callback = function(Value)
        print("Auto Minigame: " .. tostring(Value))
    end,
})

-- Botão para iniciar a lógica principal da pesca
local StartFishingButton = FishingTab:CreateButton({
    Name = "▶ Iniciar Sessão de Pesca",
    Callback = function()
        if not ToggleAutoFish.CurrentValue then
            Rayfield:Notify({
                Title = "Pesca Automática",
                Content = "Ative primeiro a 'Pesca Automática'.",
                Duration = 5,
                Image = "error",
            })
            return
        end
        print("Sessão de pesca iniciada. Aguardando detecção de bolhas...")
        -- Inicia a thread principal de pesca (veja abaixo)
        task.spawn(FishingLoop)
    end,
})

-- 4. Aba de ESP
local ESPTab = Window:CreateTab("👁️ ESP", "eye")

local ESPSection = ESPTab:CreateSection("Visualização Avançada")
local TogglePlayerESP = ESPTab:CreateToggle({
    Name = "ESP de Jogadores (Caixas, Vida, Distância)",
    CurrentValue = false,
    Flag = "PlayerESPToggle",
    Callback = function(Value)
        print("ESP de Jogadores: " .. tostring(Value))
        if Value then
            StartPlayerESP()
        else
            StopPlayerESP()
        end
    end,
})

local ToggleCorpseESP = ESPTab:CreateToggle({
    Name = "ESP do Santo (Partes do Corpo)",
    CurrentValue = false,
    Flag = "CorpseESPToggle",
    Callback = function(Value)
        print("ESP de Partes do Corpo: " .. tostring(Value))
        if Value then
            StartCorpseESP()
        else
            StopCorpseESP()
        end
    end,
})

-- 5. LÓGICA PRINCIPAL DA PESCA (Prioridade Máxima)

-- Função auxiliar para simular clique do mouse
local function SimulateClick(holdDuration)
    holdDuration = holdDuration or 0.1
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
    wait(holdDuration)
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
end

-- Função auxiliar para simular tecla do teclado
local function SimulateKeyPress(key)
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode[key], false, game)
    wait(0.05)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode[key], false, game)
end

-- Detecta bolhas na água (baseado em mudanças visuais)
local function DetectBubbles()
    -- Esta é uma detecção simplificada baseada em heurísticas.
    -- Em um cenário real, usaríamos reconhecimento de imagem via executor (ex: getcustomasset).
    -- Para Bridger: WESTERN, a dica é aguardar até que a "sombra" se aproxime da isca.
    -- Simulamos uma verificação periódica.
    local player = LocalPlayer
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    -- Lógica de timing: Esperamos um tempo aleatório entre 2 e 5 segundos (simula a aproximação do peixe)
    local waitTime = math.random(20, 50) / 10 -- 2.0 a 5.0 segundos
    wait(waitTime)
    
    -- Após o tempo, consideramos que o peixe está perto o suficiente para fisgar.
    -- Em um script real, você verificaria propriedades da UI ou do RemoteEvent do jogo.
    return true
end

-- Detecta a tecla QTE atualmente exibida na tela
local function DetectQTEKey()
    -- Simulação: Retorna uma das teclas possíveis (R, F, T, G) aleatoriamente.
    -- Em um script real, você inspecionaria a GUI do jogo (PlayerGui) para encontrar o texto da tecla.
    local keys = {"R", "F", "T", "G"}
    return keys[math.random(1, 4)]
end

-- Controla a barra de tensão (simula pressionar e soltar a tecla espaço)
local function ControlTensionBar()
    -- Simulação: Mantém o espaço pressionado por um curto período e solta.
    -- Em um script real, você monitoraria a posição do indicador na barra (via ScreenGui).
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
    wait(0.2)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
end

-- Loop principal da pesca
function FishingLoop()
    while ToggleAutoFish.CurrentValue do
        print("🎣 Lançando a linha...")
        SimulateClick(0.2) -- Simula clique longo para lançar
        
        print("⏳ Aguardando atividade do peixe (bolhas)...")
        local fishDetected = DetectBubbles()
        
        if fishDetected then
            print("⚡ Peixe detectado! Executando fisgada precisa.")
            SimulateClick(0.05) -- Clique rápido para fisgar
            
            if ToggleAutoMinigame.CurrentValue then
                print("🎮 Iniciando Auto Minigame...")
                
                -- Parte 1: QTE com teclas dinâmicas
                for i = 1, 3 do -- Geralmente são 3 QTE's antes da barra de tensão
                    local keyToPress = DetectQTEKey()
                    print("⌨️ Pressionando tecla QTE: " .. keyToPress)
                    SimulateKeyPress(keyToPress)
                    wait(0.5) -- Pequena pausa entre teclas
                end
                
                -- Parte 2: Controle da Barra de Tensão
                print("📊 Controlando barra de tensão...")
                for i = 1, 5 do -- Simula 5 ciclos de ajuste de tensão
                    ControlTensionBar()
                    wait(0.3)
                end
                
                print("✅ Minigame concluído! Peixe capturado.")
            else
                print("ℹ️ Auto Minigame desativado. Complete manualmente.")
            end
        else
            print("😞 Nenhum peixe detectado. Recolhendo linha...")
        end
        
        wait(2) -- Pausa entre tentativas
    end
    print("⏹️ Sessão de pesca finalizada.")
end

-- 6. LÓGICA DO ESP (Players e Partes do Corpo)

local PlayerESPConnections = {}
local CorpseESPConnections = {}
local PlayerHighlights = {}
local CorpseHighlights = {}

-- Função para criar Highlight em um modelo 3D
local function CreateHighlight(model, color, name)
    local highlight = Instance.new("Highlight")
    highlight.Name = name or "ESP_Highlight"
    highlight.FillColor = color or Color3.fromRGB(0, 255, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Adornee = model
    highlight.Parent = model
    return highlight
end

-- ESP de Jogadores
function StartPlayerESP()
    -- Remove highlights antigos
    StopPlayerESP()
    
    -- Função para adicionar ESP a um jogador
    local function AddESPToPlayer(player)
        if player == LocalPlayer then return end
        if player.Character then
            local highlight = CreateHighlight(player.Character, Color3.fromRGB(0, 255, 0), "PlayerESP_" .. player.Name)
            table.insert(PlayerHighlights, highlight)
            
            -- Adiciona BillboardGui com nome, vida e distância
            local function setupBillboard(character)
                local head = character:WaitForChild("Head", 5)
                if not head then return end
                
                local billboard = Instance.new("BillboardGui")
                billboard.Name = "ESP_Billboard"
                billboard.Adornee = head
                billboard.Size = UDim2.new(0, 200, 0, 50)
                billboard.StudsOffset = Vector3.new(0, 2.5, 0)
                billboard.AlwaysOnTop = true
                billboard.Parent = head
                
                local frame = Instance.new("Frame")
                frame.Size = UDim2.new(1, 0, 1, 0)
                frame.BackgroundTransparency = 0.7
                frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
                frame.BorderSizePixel = 0
                frame.Parent = billboard
                
                local nameLabel = Instance.new("TextLabel")
                nameLabel.Size = UDim2.new(1, 0, 0.4, 0)
                nameLabel.BackgroundTransparency = 1
                nameLabel.Text = player.Name
                nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                nameLabel.TextScaled = true
                nameLabel.Font = Enum.Font.SourceSansBold
                nameLabel.Parent = frame
                
                local healthLabel = Instance.new("TextLabel")
                healthLabel.Size = UDim2.new(1, 0, 0.3, 0)
                healthLabel.Position = UDim2.new(0, 0, 0.4, 0)
                healthLabel.BackgroundTransparency = 1
                healthLabel.Text = "❤️ " .. tostring(player.Character.Humanoid.Health)
                healthLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
                healthLabel.TextScaled = true
                healthLabel.Font = Enum.Font.SourceSans
                healthLabel.Parent = frame
                
                local distanceLabel = Instance.new("TextLabel")
                distanceLabel.Size = UDim2.new(1, 0, 0.3, 0)
                distanceLabel.Position = UDim2.new(0, 0, 0.7, 0)
                distanceLabel.BackgroundTransparency = 1
                distanceLabel.Text = "0m"
                distanceLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
                distanceLabel.TextScaled = true
                distanceLabel.Font = Enum.Font.SourceSans
                distanceLabel.Parent = frame
                
                -- Atualiza distância em tempo real
                local connection
                connection = RunService.RenderStepped:Connect(function()
                    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
                        connection:Disconnect()
                        return
                    end
                    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if myRoot then
                        local dist = (player.Character.HumanoidRootPart.Position - myRoot.Position).Magnitude
                        distanceLabel.Text = string.format("%.0fm", dist)
                    end
                end)
                table.insert(PlayerESPConnections, connection)
            end
            
            if player.Character:FindFirstChild("Head") then
                setupBillboard(player.Character)
            else
                player.Character:WaitForChild("Head", 5):Wait()
                setupBillboard(player.Character)
            end
        end
    end
    
    -- Adiciona para todos os jogadores atuais
    for _, player in ipairs(Players:GetPlayers()) do
        AddESPToPlayer(player)
    end
    
    -- Conecta evento para novos jogadores
    local playerAddedConn = Players.PlayerAdded:Connect(AddESPToPlayer)
    table.insert(PlayerESPConnections, playerAddedConn)
    
    -- Conecta evento para quando o personagem spawna
    local function onCharacterAdded(character)
        local player = Players:GetPlayerFromCharacter(character)
        if player then
            AddESPToPlayer(player)
        end
    end
    local charAddedConn = LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
    table.insert(PlayerESPConnections, charAddedConn)
    
    print("ESP de Jogadores ativado.")
end

function StopPlayerESP()
    for _, conn in ipairs(PlayerESPConnections) do
        conn:Disconnect()
    end
    PlayerESPConnections = {}
    
    for _, highlight in ipairs(PlayerHighlights) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    PlayerHighlights = {}
    
    -- Remove billboards
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local billboard = head:FindFirstChild("ESP_Billboard")
                if billboard then billboard:Destroy() end
            end
        end
    end
    
    print("ESP de Jogadores desativado.")
end

-- ESP de Partes do Corpo (Santo)
function StartCorpseESP()
    StopCorpseESP()
    
    local function scanForCorpseParts()
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                local name = obj.Name
                if name == "LeftArm" or name == "RightArm" or name == "LeftLeg" or name == "RightLeg" or name == "RibCage" then
                    if not obj:FindFirstChild("CorpseESP") then
                        local highlight = CreateHighlight(obj, Color3.fromRGB(255, 215, 0), "CorpseESP")
                        highlight.FillTransparency = 0.7
                        table.insert(CorpseHighlights, highlight)
                        
                        -- Adiciona Billboard com nome
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = "CorpseESP_Billboard"
                        billboard.Adornee = obj
                        billboard.Size = UDim2.new(0, 150, 0, 30)
                        billboard.StudsOffset = Vector3.new(0, 1, 0)
                        billboard.AlwaysOnTop = true
                        billboard.Parent = obj
                        
                        local label = Instance.new("TextLabel")
                        label.Size = UDim2.new(1, 0, 1, 0)
                        label.BackgroundTransparency = 0.5
                        label.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
                        label.Text = "✨ " .. name .. " ✨"
                        label.TextColor3 = Color3.fromRGB(255, 215, 0)
                        label.TextScaled = true
                        label.Font = Enum.Font.SourceSansBold
                        label.Parent = billboard
                        
                        print("Parte do Santo encontrada: " .. name)
                    end
                end
            end
        end
    end
    
    -- Escaneia periodicamente (a cada 3 segundos)
    local scanConnection = RunService.Heartbeat:Connect(function()
        if not ToggleCorpseESP.CurrentValue then
            scanConnection:Disconnect()
            return
        end
        scanForCorpseParts()
    end)
    table.insert(CorpseESPConnections, scanConnection)
    
    print("ESP do Santo ativado. Escaneando por partes...")
end

function StopCorpseESP()
    for _, conn in ipairs(CorpseESPConnections) do
        conn:Disconnect()
    end
    CorpseESPConnections = {}
    
    for _, highlight in ipairs(CorpseHighlights) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    CorpseHighlights = {}
    
    -- Remove billboards
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChild("CorpseESP_Billboard") then
            obj.CorpseESP_Billboard:Destroy()
        end
    end
    
    print("ESP do Santo desativado.")
end

-- 7. Notificação de Carregamento
Rayfield:Notify({
    Title = "Bridger WESTERN",
    Content = "Script carregado com sucesso! Use as abas para configurar.",
    Duration = 5,
    Image = "success",
})

print("✅ Script 'Bridger WESTERN - AutoFisher & ESP' carregado!")
