--// CONFIGURACIÓN
local Config = {
    {
        Name = "xDeMonzx_x", -- tu nombre o jugador
        Level = "5672",
        Money = "$4,936,318"
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

-- Mantener los valores falsos en el leaderboard
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

-- Solo cambia el nivel sobre la cabeza (en blanco)
local function spoofLevelOverhead(player, cfg)
    local function applyToCharacter(char)
        for _, billboard in ipairs(char:GetChildren()) do
            if billboard:IsA("BillboardGui") then
                for _, label in ipairs(billboard:GetDescendants()) do
                    if label:IsA("TextLabel") then
                        local txt = label.Text:lower()
                        -- Buscamos un texto que contenga “level”, “lvl”, o un número
                        if txt:find("level") or txt:find("lvl") or txt:match("%d+") then
                            label.RichText = true
                            label.TextScaled = true
                            label.TextStrokeTransparency = 0.1

                            RunService.RenderStepped:Connect(function()
                                label.Text = string.format("<font color='rgb(255,255,255)'>%s</font>", cfg.Level)
                            end)
                        end
                    end
                end
            end
        end

        -- Si aparece más tarde el BillboardGui
        char.ChildAdded:Connect(function(child)
            if child:IsA("BillboardGui") then
                task.wait(0.5)
                spoofLevelOverhead(player, cfg)
            end
        end)
    end

    if player.Character then
        applyToCharacter(player.Character)
    end
    player.CharacterAdded:Connect(applyToCharacter)
end

-- Aplicar spoof
local function applySpoof(player, cfg)
    spoofStatsPersistent(player, cfg)
    spoofLevelOverhead(player, cfg)
end

-- Inicializar
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
