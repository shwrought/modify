-- LocalScript (MUST be in StarterPlayer > StarterPlayerScripts)
-- Versión reforzada: debug + UI visible inicialmente para comprobar que funciona

--// CONFIGURACIÓN
local TargetName = "xDeMonzx_x"       -- jugador objetivo cuyo nivel/dinero quieres "falsear"
local fakeLevel = 5146                -- nivel falso (number)
local fakeMoney = "$4,318,752"        -- dinero falso (string para mostrar)

local refreshInterval = 1.2           -- cada cuantos segundos se actualiza la lista
local maxRowsVisible = 12             -- filas visibles antes de scroll
local showInitiallyFor = 5            -- mostrar la UI los primeros N segundos para debug

--// SERVICIOS
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")

-- Esperar LocalPlayer (robusto)
local LocalPlayer = Players.LocalPlayer
local waitCount = 0
while not LocalPlayer and waitCount < 10 do
    wait(0.2)
    LocalPlayer = Players.LocalPlayer
    waitCount = waitCount + 1
end

if not LocalPlayer then
    warn("[FakeLeaderboard] ERROR: LocalPlayer not found. Asegúrate de ejecutar esto como LocalScript en StarterPlayerScripts.")
    return
end

local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 5)
if not PlayerGui then
    warn("[FakeLeaderboard] ERROR: PlayerGui no encontrado.")
    return
end

print("[FakeLeaderboard] Iniciando script. LocalPlayer:", LocalPlayer.Name)

--// UI (creación)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FakeLeaderboardGui"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "Main"
mainFrame.Size = UDim2.new(0, 320, 0, 36 + (maxRowsVisible * 30))
mainFrame.Position = UDim2.new(0.02, 0, 0.08, 0)
mainFrame.AnchorPoint = Vector2.new(0, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(18,18,18)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0
mainFrame.ZIndex = 10
mainFrame.Parent = screenGui
mainFrame.Visible = false -- empieza oculto, luego se muestra temporalmente para debug

local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 36)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "PLAYERS"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Parent = mainFrame

local container = Instance.new("ScrollingFrame")
container.Name = "Container"
container.Size = UDim2.new(1, -8, 1, -44)
container.Position = UDim2.new(0, 4, 0, 40)
container.CanvasSize = UDim2.new(0, 0, 0, 0)
container.BackgroundTransparency = 1
container.BorderSizePixel = 0
container.ScrollBarThickness = 6
container.Parent = mainFrame

local uiList = Instance.new("UIListLayout")
uiList.SortOrder = Enum.SortOrder.LayoutOrder
uiList.Padding = UDim.new(0, 4)
uiList.Parent = container

-- estilo para filas (función de ayuda)
local function makeRow(playerName, levelText, moneyText, isTarget)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 28)
    row.BackgroundTransparency = 1
    row.LayoutOrder = 0

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0.52, 0, 1, 0)
    nameLabel.Position = UDim2.new(0, 6, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.TextSize = 16
    nameLabel.Text = playerName
    nameLabel.TextColor3 = (isTarget and Color3.fromRGB(255,200,60)) or Color3.fromRGB(230,230,230)
    nameLabel.Parent = row

    local levelLabel = Instance.new("TextLabel")
    levelLabel.Size = UDim2.new(0.2, 0, 1, 0)
    levelLabel.AnchorPoint = Vector2.new(0, 0)
    levelLabel.Position = UDim2.new(0.55, 0, 0, 0)
    levelLabel.BackgroundTransparency = 1
    levelLabel.Font = Enum.Font.Gotham
    levelLabel.TextSize = 15
    levelLabel.Text = tostring(levelText)
    levelLabel.TextColor3 = Color3.fromRGB(180,180,180)
    levelLabel.Parent = row

    local moneyLabel = Instance.new("TextLabel")
    moneyLabel.Size = UDim2.new(0.28, -8, 1, 0)
    moneyLabel.AnchorPoint = Vector2.new(1, 0)
    moneyLabel.Position = UDim2.new(1, -6, 0, 0)
    moneyLabel.BackgroundTransparency = 1
    moneyLabel.Font = Enum.Font.Gotham
    moneyLabel.TextSize = 15
    moneyLabel.TextXAlignment = Enum.TextXAlignment.Right
    moneyLabel.Text = tostring(moneyText)
    moneyLabel.TextColor3 = (isTarget and Color3.fromRGB(120,230,120)) or Color3.fromRGB(160,160,160)
    moneyLabel.Parent = row

    return row
end

--// Funciones para obtener level/money de cada jugador (intenta leer leaderstats; si no existe usa valores por defecto)
local function getPlayerStats(p)
    local level = "-" local money = "-"

    local leader = p:FindFirstChild("leaderstats")
    if leader then
        for _, stat in pairs(leader:GetChildren()) do
            local n = tostring(stat.Name):lower()
            local v = stat.Value
            if type(v) == "number" or type(v) == "string" then
                if string.find(n, "level") or string.find(n, "lvl") then
                    level = v
                elseif string.find(n, "money") or string.find(n, "cash") or string.find(n, "coins") or string.find(n, "gold") or string.find(n, "bank") then
                    money = v
                end
            end
        end
    end

    -- fallback: propiedades o Data folder
    if level == "-" and p:FindFirstChild("Data") and p.Data:FindFirstChild("Level") then
        level = p.Data.Level.Value
    end

    return level, money
end

-- Construye la lista en la UI; reemplaza valores del target con los falsos
local function rebuildList()
    -- limpiar solo frames (preservar UIListLayout)
    for _, child in pairs(container:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end

    -- obtener lista de jugadores
    local players = Players:GetPlayers()

    -- ordenar por level si posible (usar numeric fallback)
    table.sort(players, function(a,b)
        local la,_ = getPlayerStats(a)
        local lb,_ = getPlayerStats(b)
        local na = tonumber(la) or -math.huge
        local nb = tonumber(lb) or -math.huge
        return na > nb
    end)

    for i, p in ipairs(players) do
        local isTarget = (p.Name == TargetName)
        local level, money = getPlayerStats(p)

        if isTarget then
            level = fakeLevel
            money = fakeMoney
        end

        local row = makeRow(p.Name, level, money, isTarget)
        row.LayoutOrder = i
        row.Parent = container
    end

    -- ajustar CanvasSize (contar solo frames)
    local rows = 0
    for _,c in pairs(container:GetChildren()) do
        if c:IsA("Frame") then rows = rows + 1 end
    end
    local rowHeight = 28
    local padding = 4
    local contentHeight = rows * (rowHeight + padding)
    container.CanvasSize = UDim2.new(0, 0, 0, contentHeight)
end

-- Mostrar UI inicialmente por unos segundos para verificar visibilidad (debug)
mainFrame.Visible = true
print("[FakeLeaderboard] UI visible temporalmente para debug ("..tostring(showInitiallyFor).."s). Si NO ves esto, revisa la consola.")
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "FakeLeaderboard",
        Text = "UI visible temporalmente. Revisa la consola si no la ves.",
        Duration = 3
    })
end)
delay(showInitiallyFor, function()
    mainFrame.Visible = false
end)

-- Actualizaciones periódicas + hooks para cambios
rebuildList()
local elapsed = 0
RunService.Heartbeat:Connect(function(dt)
    elapsed = elapsed + dt
    if elapsed >= refreshInterval then
        rebuildList()
        elapsed = 0
    end
end)

-- Actualizar cuando jugadores entren/salgan
Players.PlayerAdded:Connect(function() rebuildList() end)
Players.PlayerRemoving:Connect(function() rebuildList() end)

-- Escuchar cambios en leaderstats de jugadores (intentar)
for _,pl in pairs(Players:GetPlayers()) do
    pl.ChildAdded:Connect(function() rebuildList() end)
    pl.ChildRemoved:Connect(function() rebuildList() end)
end
Players.PlayerAdded:Connect(function(pl)
    pl.ChildAdded:Connect(function() rebuildList() end)
    pl.ChildRemoved:Connect(function() rebuildList() end)
end)

-- Abrir/Cerrar con Tab (mismo comportamiento visual que leaderboard nativo)
local visible = false
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Tab then
        visible = not visible
        mainFrame.Visible = visible
    end
end)

-- Ocultar si el jugador muere o respawnea (estética)
LocalPlayer.CharacterAdded:Connect(function() mainFrame.Visible = false visible = false end)

print("[FakeLeaderboard] Script cargado correctamente. Put this LocalScript in StarterPlayerScripts. Target:", TargetName)
