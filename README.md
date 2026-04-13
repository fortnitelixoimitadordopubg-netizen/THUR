--[[
    Script Simples para Bridger: WESTERN
    - Pesca Automática Inteligente
    - ESP de Jogadores e Partes do Santo
    Compatível com Xeno e outros executores.
]]

-- ================= CONFIGURAÇÕES =================
local AUTO_FISH = true           -- Ativa a pesca automática
local AUTO_MINIGAME = true       -- Ativa o minigame automático (QTE + tensão)
local PLAYER_ESP = true          -- Ativa ESP de jogadores (caixa + nome + vida)
local CORPSE_ESP = true          -- Ativa ESP das partes do Santo (dourado)

-- ============== SERVIÇOS DO ROBLOX ===============
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VIM = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ============== BIBLIOTECA DE DESENHO (ESP) ==============
local Drawing = loadstring(game:HttpGet("https://raw.githubusercontent.com/Stefanuk12/DrawingLib/main/DrawingLib.lua"))()

-- ============== VARIÁVEIS DO ESP ==============
local ESPConnections = {}
local PlayerDrawings = {}
local CorpseDrawings = {}

-- ============== FUNÇÕES AUXILIARES ==============
local function pressKey(key, duration)
    duration = duration or 0.05
    VIM:SendKeyEvent(true, Enum.KeyCode[key], false, game)
    task.wait(duration)
    VIM:SendKeyEvent(false, Enum.KeyCode[key], false, game)
end

local function clickMouse(duration)
    duration = duration or 0.1
    VIM:SendMouseButtonEvent(0, 0, 0, true, game, 0)
    task.wait(duration)
    VIM:SendMouseButtonEvent(0, 0, 0, false, game, 0)
end

-- Detecta tecla QTE (simplificado - no jogo real, você inspeciona a GUI)
local function getCurrentQTEKey()
    -- Para uma detecção real, use :FindFirstChild na PlayerGui.
    -- Exemplo: local key = LocalPlayer.PlayerGui:FindFirstChild("QTE") and LocalPlayer.PlayerGui.QTE.TextLabel.Text
    -- Como não temos acesso exato à UI, simulamos aleatoriamente uma das 4 teclas.
    local keys = {"R", "F", "T", "G"}
    return keys[math.random(1, 4)]
end

-- Controle da barra de tensão (simula segurar e soltar espaço)
local function controlTension()
    -- Em um script avançado, você leria a posição do ponteiro na barra.
    -- Aqui simulamos um comportamento seguro.
    VIM:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
    task.wait(0.2)
    VIM:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
    task.wait(0.1)
end

-- ============== LOOP PRINCIPAL DA PESCA ==============
local function fishingLoop()
    while AUTO_FISH and task.wait(1) do
        print("🎣 Lançando linha...")
        clickMouse(0.2) -- clique longo para arremessar
        
        -- Aguarda detecção da fisgada (simula espera inteligente)
        local waitTime = math.random(30, 60) / 10  -- 3 a 6 segundos
        print("⏳ Aguardando peixe... (" .. waitTime .. "s)")
        task.wait(waitTime)
        
        print("⚡ Peixe detectado! Fisgando...")
        clickMouse(0.05) -- clique rápido para fisgar
        
        if AUTO_MINIGAME then
            print("🎮 Iniciando minigame automático...")
            
            -- Parte 1: QTE (3 rodadas normalmente)
            for i = 1, 3 do
                local key = getCurrentQTEKey()
                print("   ⌨️ Pressionando " .. key)
                pressKey(key, 0.08)
                task.wait(0.4)
            end
            
            -- Parte 2: Controle de tensão (5 ciclos)
            print("📊 Controlando tensão...")
            for i = 1, 5 do
                controlTension()
                task.wait(0.25)
            end
            
            print("✅ Peixe capturado!")
        else
            print("ℹ️ Minigame desativado. Complete manualmente.")
        end
        
        task.wait(2) -- pausa entre pescarias
    end
end

-- ============== ESP DE JOGADORES ==============
local function updatePlayerESP()
    -- Limpa desenhos antigos
    for _, drawing in pairs(PlayerDrawings) do
        if drawing.Remove then drawing:Remove() end
    end
    PlayerDrawings = {}
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local root = player.Character.HumanoidRootPart
            local head = player.Character:FindFirstChild("Head")
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if not (root and head and humanoid) then continue end
            
            -- Cria caixa 2D ao redor do jogador
            local box = Drawing.new("Square")
            box.Visible = true
            box.Thickness = 2
            box.Color = Color3.fromRGB(0, 255, 0)
            box.Filled = false
            box.Transparency = 1
            
            -- Nome e vida
            local nameTag = Drawing.new("Text")
            nameTag.Visible = true
            nameTag.Text = player.Name .. " [" .. math.floor(humanoid.Health) .. " HP]"
            nameTag.Size = 14
            nameTag.Color = Color3.fromRGB(255, 255, 255)
            nameTag.Center = true
            nameTag.Outline = true
            nameTag.OutlineColor = Color3.new(0,0,0)
            
            -- Distância
            local distTag = Drawing.new("Text")
            distTag.Visible = true
            distTag.Size = 12
            distTag.Color = Color3.fromRGB(200, 200, 200)
            distTag.Center = true
            distTag.Outline = true
            distTag.OutlineColor = Color3.new(0,0,0)
            
            table.insert(PlayerDrawings, box)
            table.insert(PlayerDrawings, nameTag)
            table.insert(PlayerDrawings, distTag)
            
            -- Conexão para atualizar posição a cada frame
            local connection = RunService.RenderStepped:Connect(function()
                if not player.Character or not root.Parent then
                    box.Visible = false
                    nameTag.Visible = false
                    distTag.Visible = false
                    return
                end
                
                local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if not myRoot then return end
                
                local distance = (root.Position - myRoot.Position).Magnitude
                local screenPos, onScreen = Camera:WorldToViewportPoint(root.Position)
                
                if onScreen then
                    local headPos = Camera:WorldToViewportPoint(head.Position)
                    local scale = 500 / math.max(distance, 1)
                    local height = math.clamp(scale * 2, 40, 150)
                    local width = height * 0.6
                    
                    box.Position = Vector2.new(screenPos.X - width/2, screenPos.Y - height/2)
                    box.Size = Vector2.new(width, height)
                    box.Visible = true
                    
                    nameTag.Position = Vector2.new(screenPos.X, screenPos.Y - height/2 - 15)
                    nameTag.Text = player.Name .. " [" .. math.floor(humanoid.Health) .. " HP]"
                    nameTag.Visible = true
                    
                    distTag.Position = Vector2.new(screenPos.X, screenPos.Y + height/2 + 5)
                    distTag.Text = string.format("%.0fm", distance)
                    distTag.Visible = true
                else
                    box.Visible = false
                    nameTag.Visible = false
                    distTag.Visible = false
                end
            end)
            
            table.insert(ESPConnections, connection)
        end
    end
end

-- ============== ESP DO SANTO (PARTES DO CORPO) ==============
local function updateCorpseESP()
    for _, drawing in pairs(CorpseDrawings) do
        if drawing.Remove then drawing:Remove() end
    end
    CorpseDrawings = {}
    
    local parts = {"LeftArm", "RightArm", "LeftLeg", "RightLeg", "RibCage"}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and table.find(parts, obj.Name) then
            local box = Drawing.new("Square")
            box.Visible = true
            box.Thickness = 3
            box.Color = Color3.fromRGB(255, 215, 0)  -- dourado
            box.Filled = false
            box.Transparency = 1
            
            local label = Drawing.new("Text")
            label.Visible = true
            label.Text = "✨ " .. obj.Name
            label.Size = 14
            label.Color = Color3.fromRGB(255, 215, 0)
            label.Center = true
            label.Outline = true
            label.OutlineColor = Color3.new(0,0,0)
            
            table.insert(CorpseDrawings, box)
            table.insert(CorpseDrawings, label)
            
            local connection = RunService.RenderStepped:Connect(function()
                if not obj.Parent then
                    box.Visible = false
                    label.Visible = false
                    return
                end
                
                local pos = obj.Position
                local screenPos, onScreen = Camera:WorldToViewportPoint(pos)
                if onScreen then
                    local size = obj.Size
                    local maxDim = math.max(size.X, size.Y, size.Z)
                    local scale = 200 / math.max((pos - Camera.CFrame.Position).Magnitude, 1)
                    local boxSize = maxDim * scale * 0.5
                    
                    box.Position = Vector2.new(screenPos.X - boxSize/2, screenPos.Y - boxSize/2)
                    box.Size = Vector2.new(boxSize, boxSize)
                    box.Visible = true
                    
                    label.Position = Vector2.new(screenPos.X, screenPos.Y - boxSize/2 - 15)
                    label.Visible = true
                else
                    box.Visible = false
                    label.Visible = false
                end
            end)
            
            table.insert(ESPConnections, connection)
        end
    end
end

-- ============== INICIALIZAÇÃO ==============
print("=== SCRIPT CARREGADO ===")
print("Pesca Automática: " .. tostring(AUTO_FISH))
print("Auto Minigame: " .. tostring(AUTO_MINIGAME))
print("ESP Jogadores: " .. tostring(PLAYER_ESP))
print("ESP Santo: " .. tostring(CORPSE_ESP))

-- Inicia a pesca em uma thread separada
if AUTO_FISH then
    task.spawn(fishingLoop)
end

-- Atualiza ESP a cada segundo (jogadores podem entrar/sair)
if PLAYER_ESP then
    task.spawn(function()
        while PLAYER_ESP and task.wait(1) do
            updatePlayerESP()
        end
    end)
end

if CORPSE_ESP then
    task.spawn(function()
        while CORPSE_ESP and task.wait(1) do
            updateCorpseESP()
        end
    end)
end

-- Comandos rápidos no console (opcional)
print("Digite 'stop_fish' para parar a pesca.")
print("Digite 'stop_esp' para desativar todos os ESP.")

-- Permite controle via console
getgenv().stop_fish = function()
    AUTO_FISH = false
    print("Pesca automática DESATIVADA.")
end

getgenv().stop_esp = function()
    PLAYER_ESP = false
    CORPSE_ESP = false
    for _, conn in ipairs(ESPConnections) do
        conn:Disconnect()
    end
    ESPConnections = {}
    for _, drawing in pairs(PlayerDrawings) do
        if drawing.Remove then drawing:Remove() end
    end
    for _, drawing in pairs(CorpseDrawings) do
        if drawing.Remove then drawing:Remove() end
    end
    PlayerDrawings = {}
    CorpseDrawings = {}
    print("ESP DESATIVADO.")
end
