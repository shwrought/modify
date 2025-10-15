--// CONFIGURACIÓN
local TargetName = "habbathehut"
local fakeLevel = "5146"
local fakeMoney = "$4,318,752"

--// SERVICIOS
local Players = game:GetService("Players")

-- Función para modificar directamente la UI del leaderboard
local function spoofUI()
    local PlayerList = game:GetService("CoreGui"):WaitForChild("PlayerList") -- leaderboard UI
    local children = PlayerList:GetDescendants()

    for _, obj in pairs(children) do
        if obj:IsA("TextLabel") and string.find(obj.Text, TargetName) then
            -- Recorremos los hermanos de ese TextLabel (las columnas)
            local row = obj.Parent.Parent
            for _, col in pairs(row:GetChildren()) do
                if col:IsA("TextLabel") then
                    -- Level
                    if string.find(col.Text, "%d") and string.len(col.Text) < 6 then
                        col.Text = fakeLevel
                    end
                    -- Money
                    if string.find(col.Text, "%$") or string.find(col.Text, "%d") then
                        col.Text = fakeMoney
                    end
                end
            end
        end
    end
end

-- Reaplicar cada pocos segundos para que no se resetee
while task.wait(2) do
    spoofUI()
end
