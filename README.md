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

-- Colores por título
local TitleColors = {
    ["Antihero"] = Color3.fromRGB(255, 85, 85),      -- Rojo fuerte
    ["God"] = Color3.fromRGB(255, 215, 0),           -- Dorado
    ["Supervillain"] = Color3.fromRGB(170, 85, 255), -- Violeta
    ["Hero"] = Color3.fromRGB(85, 255, 85),          -- Verde
    ["Villain"] = Color3.fromRGB(255, 0, 255),       -- Magenta
}

-- Buscar configuración del jugador
local function getConfig(name)
    for _, cfg in ipairs(Config) do
        if cfg.Name == name then
            return cfg
        end
    end
end

-- Fake Overhead (solo visual, no altera stats reales)
local function spoofOverhead(char, cfg)
    local color = TitleColors[cfg.Title] or Color3.fromRGB(255, 255, 255)
    RunService.RenderStepped:Connect(function()
        for _, gui in ipairs(char:GetDescendants()) do
            if gui:IsA("TextLabel") then
                if gui.Text and string.find(gui.Text, "%d") then
                    gui.Text = tostring(cfg.Level) .. " " .. tostring(cfg.Title)
                    gui.TextColor3 = color
                    gui.TextStrokeTransparency = 0.3
                end
            end
        end
    end)
end

-- Aplicar spoof solo visual
local function applySpoof(player, cfg)
    if player.Character then
        spoofOverhead(player.Character, cfg)
    end
    player.CharacterAdded:Connect(function(char)
        task.wait(1)
        spoofOverhead(char, cfg)
    end)
end

-- Aplicar spoof a los jugadores existentes
for _, cfg in ipairs(Config) do
    local plr = Players:FindFirstChild(cfg.Name)
    if plr then
        applySpoof(plr, cfg)
    end
end

-- Cuando entre alguien nuevo
Players.PlayerAdded:Connect(function(plr)
    local cfg = getConfig(plr.Name)
    if cfg then
        applySpoof(plr, cfg)
    end
end)
