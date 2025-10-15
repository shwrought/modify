--// CONFIGURACIÓN (jugadores falsos o modificados)
local Config = {
    {
        Name = "xDeMonzx_x",
        Level = "5672",
        Money = "$4,936,318",
        Title = "God"
    }
}

--// SERVICIOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Buscar configuración por nombre
local function getConfig(name)
    for _, cfg in ipairs(Config) do
        if cfg.Name == name then
            return cfg
        end
    end
end

-- Forzar valores visuales constantemente
local function spoofStatsPersistent(player, cfg)
    local ls = player:WaitForChild("leaderstats", 10)
    if not ls then return end

    -- Reforzar los valores visuales constantemente
    RunService.RenderStepped:Connect(function()
        if ls:FindFirstChild("Level") then
            ls.Level.Value = cfg.Level
        end
        if ls:FindFirstChild("Money") then
            ls.Money.Value = cfg.Money
        end
    end)
end

-- Spoof del texto sobre la cabeza (Overhead)
local function spoofOverheadPersistent(player, cfg)
    RunService.RenderStepped:Connect(function()
        if player.Character then
            for _, gui in ipairs(player.Character:GetDescendants()) do
                if gui:IsA("TextLabel") then
                    gui.Text = cfg.Level .. " " .. cfg.Title
                end
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

-- Inicializar para los jugadores configurados
for _, cfg in ipairs(Config) do
    local plr = Players:FindFirstChild(cfg.Name)
    if plr then
        applySpoof(plr, cfg)
    end
end

-- Si entra alguien con nombre en la lista
Players.PlayerAdded:Connect(function(plr)
    local cfg = getConfig(plr.Name)
    if cfg then
        applySpoof(plr, cfg)
    end
end)
