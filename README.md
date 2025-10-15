--// CONFIGURACIÓN
local Config = {
    {
        Name = "xDeMonzx_x", -- tu nick u otro jugador
        Level = "5672",
        Money = "$4,936,318",
        Title = "God"
    }
}

--// SERVICIOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Buscar configuración por nombre
local function getConfig(name)
    for _, cfg in ipairs(Config) do
        if cfg.Name == name then
            return cfg
        end
    end
end

-- Forzar stats visuales (leaderstats)
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

-- Colores predefinidos del sistema (puedes editarlos)
local Colors = {
    White = Color3.fromRGB(255, 255, 255),
    Celeste = Color3.fromRGB(100, 200, 255)
}

-- Spoof del texto sobre la cabeza (limpio y sin distorsión)
local function spoofOverheadPersistent(player, cfg)
    RunService.RenderStepped:Connect(function()
        if not player.Character then return end
        for _, gui in ipairs(player.Character:GetDescendants()) do
            if gui:IsA("TextLabel") and gui.Text ~= "" then
                -- Limpia el texto y formato original
                gui.RichText = true
                gui.TextScaled = true
                gui.TextWrapped = false
                gui.TextStrokeTransparency = 0.1

                -- Texto combinado: nivel blanco + rango celeste
                gui.Text = string.format(
                    "<font color='rgb(255,255,255)'>%s</font>  <font color='rgb(100,200,255)'>%s</font>",
                    cfg.Level,
                    cfg.Title
                )
            end
        end
    end)
end

-- Aplicar spoof completo
local function applySpoof(player, cfg)
    spoofStatsPersistent(player, cfg)
    spoofOverheadPersistent(player, cfg)

    player.CharacterAdded:Connect(function()
        task.wait(1)
        spoofOverheadPersistent(player, cfg)
    end)
end

-- Inicializar para jugadores configurados
for _, cfg in ipairs(Config) do
    local plr = Players:FindFirstChild(cfg.Name)
    if plr then
        applySpoof(plr, cfg)
    end
end

-- Aplicar si entra alguien nuevo de la lista
Players.PlayerAdded:Connect(function(plr)
    local cfg = getConfig(plr.Name)
    if cfg then
        applySpoof(plr, cfg)
    end
end)
