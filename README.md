--// CONFIGURACIÓN
local TargetName = "InmortalS4vage" -- Nombre del jugador
local fakeLevel = "19283"
local fakeMoney = "$41,378,000"
local fakeTitle = "Antihero" -- 👈 pon el título que quieras que aparezca

--// SERVICIOS
local Players = game:GetService("Players")

-- Modificar leaderboard
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

-- Modificar el BillboardGui (texto sobre la cabeza)
local function spoofOverhead(char)
    task.wait(1) -- esperar a que cargue todo
    for _, gui in pairs(char:GetDescendants()) do
        if gui:IsA("TextLabel") and string.find(gui.Text, "%d") then
            -- Forzar texto falso
            gui.Text = fakeLevel .. " " .. fakeTitle
            -- Reaplicar cada vez que el servidor lo cambie
            gui:GetPropertyChangedSignal("Text"):Connect(function()
                gui.Text = fakeLevel .. " " .. fakeTitle
            end)
        end
    end
end

-- Cuando el jugador entra
Players.PlayerAdded:Connect(function(player)
    if player.Name == TargetName then
        player.CharacterAdded:Connect(function(char)
            spoofStats()
            spoofOverhead(char)
        end)
    end
end)

-- Si ya está en el server
local targetPlayer = Players:FindFirstChild(TargetName)
if targetPlayer then
    spoofStats()
    if targetPlayer.Character then
        spoofOverhead(targetPlayer.Character)
    end
end
