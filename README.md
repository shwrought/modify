--// CONFIGURACIÓN
local Config = {
    {
        Name = "xDeMonzx_x", -- tu nombre o jugador
        Level = "5672",
        Money = "$4,936,318",
        Title = "God"
    }
}

--// SERVICIOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Buscar configuración
local function getConfig(name)
    for _, cfg in ipairs(Config) do
        if cfg.Name == name then
            return cfg
        end
    end
end

-- Mantener valores visuales falsos
local function spoofStatsPersistent(player, cfg)
    local ls = player:WaitForChild("leaderstats", 10)
    if not ls then return end

    RunService.RenderStepped:Connect(function()
        if ls:FindFirstChild("Level") then
            ls.Level.Value = cfg.Level
        end
        if ls:FindFirstChild("Money") then
            ls.Money.Value = cfg.Money
        end
    end)
end

-- Colores predefinidos
local COLORS = {
    White = Color3.fromRGB(255, 255, 255),
    Celeste = Color3.fromRGB(100, 200, 255)
}

-- Spoof del texto sobre la cabeza (separado correctamente)
local function spoofOverheadPersistent(player, cfg)
    local function applyToCharacter(char)
        -- Buscar BillboardGuis existentes
        for _, billboard in ipairs(char:GetChildren()) do
            if billboard:IsA("BillboardGui") then
                local levelLabel, titleLabel

                -- Buscar labels
                for _, lbl in ipairs(billboard:GetDescendants()) do
                    if lbl:IsA("TextLabel") then
                        local txt = lbl.Text:lower()
                        if txt:find("level") or txt:find(cfg.Level:lower()) then
                            levelLabel = lbl
                        elseif txt:find("god") or txt:find(cfg.Title:lower()) then
                            titleLabel = lbl
                        end
                    end
                end

                -- Crear o actualizar el label de nivel
                if levelLabel then
                    levelLabel.RichText = true
                    levelLabel.TextScaled = true
                    levelLabel.TextStrokeTransparency = 0.1
                    RunService.RenderStepped:Connect(function()
                        levelLabel.Text = string.format("<font color='rgb(255,255,255)'>%s</font>", cfg.Level)
                    end)
                end

                -- Crear o actualizar el label del título
                if titleLabel then
                    titleLabel.RichText = true
                    titleLabel.TextScaled = true
                    titleLabel.TextStrokeTransparency = 0.1
                    RunService.RenderStepped:Connect(function()
                        titleLabel.Text = string.format("<font color='rgb(100,200,255)'>%s</font>", cfg.Title)
                    end)
                end
            end
        end

        -- Si el Billboard aparece más tarde
        char.ChildAdded:Connect(function(child)
            if child:IsA("BillboardGui") then
                task.wait(0.5)
                spoofOverheadPersistent(player, cfg)
            end
        end)
    end

    if player.Character then
        applyToCharacter(player.Character)
    end
    player.CharacterAdded:Connect(applyToCharacter)
end

-- Aplicar spoof completo
local function applySpoof(player, cfg)
    spoofStatsPersistent(player, cfg)
    spoofOverheadPersistent(player, cfg)
end

-- Inicializar para jugadores configurados
for _, cfg in ipairs(Config) do
    local plr = Players:FindFirstChild(cfg.Name)
    if plr then
        applySpoof(plr, cfg)
    end
end

Players.PlayerAdded:Connect(function(plr)
    local cfg = getConfig(plr.Name)
    if cfg then
        applySpoof(plr, cfg)
    end
end)
