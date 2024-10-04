-- Função para criar o Highlight e a box 3D no time inimigo
local function createEnemyInfo(player)
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        
        -- Cria um objeto Highlight para a hitbox
        local highlight = Instance.new("Highlight")
        highlight.Name = "HitboxHighlight"
        highlight.Adornee = character
        highlight.FillColor = Color3.new(1, 0, 0) -- Cor vermelha para o inimigo
        highlight.FillTransparency = 0.7 -- Transparência do "glow"
        highlight.OutlineColor = Color3.new(1, 1, 1) -- Cor da borda
        highlight.OutlineTransparency = 0 -- Borda completamente opaca
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- Para que o glow seja visível através das paredes
        highlight.Parent = character

        -- Cria uma box 3D ao redor do HumanoidRootPart
        local box = Instance.new("BoxHandleAdornment")
        box.Name = "EnemyBox"
        box.Adornee = character.HumanoidRootPart
        box.Size = character.HumanoidRootPart.Size + Vector3.new(1, 1, 1) -- Aumenta ligeiramente a box em torno da hitbox
        box.Color3 = Color3.new(1, 0, 0) -- Cor vermelha para a box
        box.Transparency = 0.5 -- Transparência da box
        box.AlwaysOnTop = true -- Para que a box seja visível através das paredes
        box.ZIndex = 10 -- Garante que a box fique no topo
        box.Parent = character.HumanoidRootPart

        -- Cria um BillboardGui para mostrar o nome e vida do inimigo
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "PlayerInfo"
        billboard.Adornee = character.Head
        billboard.Size = UDim2.new(4, 0, 2, 0)
        billboard.StudsOffset = Vector3.new(0, 2, 0) -- Levanta o texto acima da cabeça
        billboard.AlwaysOnTop = true
        
        -- Cria o rótulo para o nome do jogador
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextStrokeTransparency = 0.5
        nameLabel.TextScaled = true
        nameLabel.Parent = billboard

        -- Cria o rótulo para a vida do jogador
        local healthLabel = Instance.new("TextLabel")
        healthLabel.Name = "HealthLabel"
        healthLabel.Size = UDim2.new(1, 0, 0.5, 0)
        healthLabel.Position = UDim2.new(0, 0, 0.5, 0)
        healthLabel.BackgroundTransparency = 1
        healthLabel.TextColor3 = Color3.new(0, 1, 0)
        healthLabel.TextStrokeTransparency = 0.5
        healthLabel.TextScaled = true
        healthLabel.Parent = billboard

        -- Atualiza a vida do jogador constantemente
        game:GetService("RunService").RenderStepped:Connect(function()
            if character:FindFirstChild("Humanoid") then
                healthLabel.Text = "HP: " .. math.floor(character.Humanoid.Health)
            end

            -- Faz o glow e a box mais destacáveis quanto mais longe o inimigo estiver
            local distance = (character.Head.Position - game.Players.LocalPlayer.Character.Head.Position).magnitude
            highlight.FillTransparency = math.clamp(distance / 100, 0.1, 0.7)
            box.Transparency = math.clamp(distance / 100, 0.1, 0.5)
        end)

        -- Adiciona o BillboardGui na cabeça do jogador
        billboard.Parent = character.Head
    end
end

-- Verifica se o jogador pertence ao time inimigo
local function isEnemy(player)
    local localPlayerTeam = game.Players.LocalPlayer.Team
    return player.Team ~= localPlayerTeam
end

-- Aplica o Highlight, box 3D e informações aos jogadores inimigos
local function applyToAllEnemies()
    for _, player in pairs(game.Players:GetPlayers()) do
        if isEnemy(player) then
            createEnemyInfo(player)
        end
    end
end

-- Aplica o Highlight, box 3D e informações aos jogadores inimigos existentes e novos jogadores que entrarem no jogo
applyToAllEnemies()
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if isEnemy(player) then
            createEnemyInfo(player)
        end
    end)
end)
