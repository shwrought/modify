--// CONFIGURACIÓN (agrega los jugadores que quieras)
local Config = {
    {
        Name = "xDeMonzx_x",
        Level = "5672",
        Money = "$4,936,318",
        Title = "Antihero"
    },
    {
        Name = "xDeMonzx_x", -- tu nick por ejemplo
        Level = "5672",
        Money = "$4,936,318",
        Title = "God"
    }
}

--// SERVICIOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Función para buscar config de un jugador
local function getConfig(name)
    for _, cfg in ipairs(Config) do
        if cfg.Name == name then
            return cfg
        end
    end
end

-- Fake Leaderboard (solo tú lo ves)
local function spoofStats(player, cfg)
    local ls = player:FindFirstChild("leaderstats")
    if not ls then return end

    if ls:FindFirstChild("Level") then
        ls.Level.Value = cfg.Level
        ls.Level.Changed:Connect(function()
            ls.Level.Value = cfg.Level
        end)
    end
    if ls:FindFirstChild("Money") then
        ls.Money.Value = cfg.Money
        ls.Money.Changed:Connect(function()
            ls.Money.Value = cfg.Money
        end)
    end
end

-- Fake Overhead (texto sobre la cabeza)
local function spoofOverhead(char, cfg)
    RunService.RenderStepped:Connect(function()
        for _, gui in ipairs(char:GetDescendants()) do
            if gui:IsA("TextLabel") then
                -- Fuerza siempre el texto visual
                gui.Text = cfg.Level .. " " .. cfg.Title
            end
        end
    end)
end

-- Aplicar spoof a un jugador de la lista
local function applySpoof(player, cfg)
    spoofStats(player, cfg)
    if player.Character then
        spoofOverhead(player.Character, cfg)
    end
    player.CharacterAdded:Connect(function(char)
        task.wait(1)
        spoofOverhead(char, cfg)
    end)
end

-- Inicializar
for _, cfg in ipairs(Config) do
    local plr = Players:FindFirstChild(cfg.Name)
    if plr then
        applySpoof(plr, cfg)
    end
end

-- Para cuando entre alguien de la lista
Players.PlayerAdded:Connect(function(plr)
    local cfg = getConfig(plr.Name)
    if cfg then
        applySpoof(plr, cfg)
    end
end)
