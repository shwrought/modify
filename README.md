--// CONFIGURACIÓN
local TargetName = "InmortalS4vage" -- 👈 Nombre del jugador en la tabla
local fakeLevel = "11700"
local fakeMoney = "$15,378,000"

--// SERVICIOS
local Players = game:GetService("Players")

-- Función para modificar los valores
local function spoofStats()
    local targetPlayer = Players:FindFirstChild(TargetName)
    if not targetPlayer then return end

    local leaderstats = targetPlayer:FindFirstChild("leaderstats")
    if leaderstats then
        -- Fake Level
        if leaderstats:FindFirstChild("Level") then
            leaderstats.Level.Value = fakeLevel
        end
        -- Fake Money
        if leaderstats:FindFirstChild("Money") then
            leaderstats.Money.Value = fakeMoney
        end
    end
end

-- Cuando entra un jugador, si es el objetivo lo editamos
Players.PlayerAdded:Connect(function(player)
    if player.Name == TargetName then
        player.CharacterAdded:Connect(spoofStats)
    end
end)

-- Ejecutar al inicio si ya está en el server
spoofStats()
