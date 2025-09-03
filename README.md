--// CONFIGURACIÓN
local TargetName = "KAMILYRR14"
local fakeLevel = "19283"
local fakeMoney = "$9,378,000"
local fakeTitle = "Antihero"

--// SERVICIOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Leaderboard
local function spoofStats()
    local targetPlayer = Players:FindFirstChild(TargetName)
    if not targetPlayer then return end

    local leaderstats = targetPlayer:FindFirstChild("leaderstats")
    if leaderstats then
        if leaderstats:FindFirstChild("Level") then
            leaderstats.Level.Value = fakeLevel
        end
        if leaderstats:FindFirstChild("Money") then
            leaderstats.Money.Value = fakeMoney
            leaderstats.Money.Changed:Connect(function()
                leaderstats.Money.Value = fakeMoney
            end)
        end
    end
end

-- Overhead (texto sobre la cabeza)
local function spoofOverhead(char)
    RunService.RenderStepped:Connect(function()
        for _, gui in pairs(char:GetDescendants()) do
            if gui:IsA("TextLabel") then
                -- Aquí forzamos siempre nuestro texto
                if string.find(gui.Text, tostring(targetPlayer.Level)) or string.find(gui.Text, fakeTitle) then
                    gui.Text = fakeLevel .. " " .. fakeTitle
                end
            end
        end
    end)
end

-- Cuando el jugador aparece
Players.PlayerAdded:Connect(function(player)
    if player.Name == TargetName then
        player.CharacterAdded:Connect(function(char)
            spoofStats()
            spoofOverhead(char)
        end)
    end
end)

-- Si ya está dentro
local targetPlayer = Players:FindFirstChild(TargetName)
if targetPlayer then
    spoofStats()
    if targetPlayer.Character then
        spoofOverhead(targetPlayer.Character)
    end
end
