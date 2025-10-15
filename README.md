--// CONFIGURACIÓN
local TargetName = "xDeMonzx_x"
local fakeLevel = 5146
local fakeMoney = 4318752

--// SERVICIOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Función para modificar los leaderstats del jugador objetivo (solo visual)
local function spoofStats()
    local target = Players:FindFirstChild(TargetName)
    if not target then return end

    local leaderstats = target:FindFirstChild("leaderstats")
    if not leaderstats then return end

    -- Recorremos las stats
    for _, stat in pairs(leaderstats:GetChildren()) do
        if stat:IsA("IntValue") or stat:IsA("NumberValue") or stat:IsA("StringValue") then
            if string.find(stat.Name:lower(), "level") then
                stat.Value = fakeLevel
            elseif string.find(stat.Name:lower(), "money") or string.find(stat.Name:lower(), "cash") then
                stat.Value = fakeMoney
            end
        end
    end
end

-- Reaplicar cada pocos segundos
while task.wait(2) do
    spoofStats()
end
