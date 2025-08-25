--// CONFIGURACIÓN
local TargetName = "InmortalS4vage" -- Nombre del jugador
local fakeLevel = "19283"
local fakeMoney = "$41,378,000"

--// SERVICIOS
local Players = game:GetService("Players")

-- Función para modificar Leaderboard
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
            leaderstats.Money.Changed:Connect(function()
                leaderstats.Money.Value = fakeMoney
            end)
        end
    end
end

-- Función para modificar el texto sobre la cabeza
local function spoofOverhead(char)
    task.wait(1) -- pequeño delay para que cargue el BillboardGui
    for _, gui in pairs(char:GetDescendants()) do
        if gui:IsA("TextLabel") and string.find(gui.Text, "Antihero") or string.find(gui.Text, "%d") then
            gui.Text = fakeLevel .. " Antihero" -- 👈 aquí editas el texto que aparece
        end
    end
end

-- Cuando entra un jugador
Players.PlayerAdded:Connect(function(player)
    if player.Name == TargetName then
        player.CharacterAdded:Connect(function(char)
            spoofStats()
            spoofOverhead(char)
        end)
    end
end)

-- Si ya está dentro al inicio
local targetPlayer = Players:FindFirstChild(TargetName)
if targetPlayer and targetPlayer.Character then
    spoofStats()
    spoofOverhead(targetPlayer.Character)
end
