-- LocalScript (poner en StarterPlayerScripts)
-- Fake leaderboard overlay que se abre con Tab y reemplaza valores del TargetName

--// CONFIGURACIÓN
local TargetName = "xDeMonzx_x"       -- jugador objetivo cuyo nivel/dinero quieres "falsear"
local fakeLevel = 5146                -- nivel falso (number)
local fakeMoney = "$4,318,752"        -- dinero falso (string para mostrar)

local refreshInterval = 1.5           -- cada cuantos segundos se actualiza la lista
local maxRowsVisible = 12             -- filas visibles antes de scroll

--// SERVICIOS
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--// UI (creación)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FakeLeaderboardGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "Main"
mainFrame.Size = UDim2.new(0, 300, 0, 32 + (maxRowsVisible * 28)) -- altura base + filas
mainFrame.Position = UDim2.new(0.02, 0, 0.08, 0)
mainFrame.AnchorPoint = Vector2.new(0, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0
mainFrame.Parent = screenGui
mainFrame.Visible = false

local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 32)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "PLAYERS"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Parent = mainFrame

local container = Instance.new("ScrollingFrame")
container.Name = "Container"
container.Size = UDim2.new(1, -8, 1, -40)
container.Position = UDim2.new(0, 4, 0, 36)
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
    local level = "-"
    local money = "-"

    local leader = p:FindFirstChild("leaderstats")
    if leader then
        -- buscar nombres comunes
        for _, stat in pairs(leader:GetChildren()) do
            local n = stat.Name:lower()
            if typeof(stat.Value) == "number" or typeof(stat.Value) == "string" then
                if string.find(n, "level") or string.find(n, "lvl") then
                    level = stat.Value
                elseif string.find(n, "money") or string.find(n, "cash") or string.find(n, "coins") or string.find(n, "gold") then
                    money = stat.Value
                end
            end
        end
    end

    -- si siguen sin valor, intentar valores por defecto de propiedades conocidas
    if level == "-" and p:FindFirstChild("Data") and p.Data:FindFirstChild("Level") then
        level = p.Data.Level.Value
    end

    return level, money
end

-- Construye la lista en la UI; reemplaza valores del target con los falsos
local function rebuildList()
    -- limpiar
    for _, child in pairs(container:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end

    -- obtener lista de jugadores y ordenarlos (opcional: por level si existe)
    local players = Players:GetPlayers()
    table.sort(players, function(a,b)
        local la = tonumber(getPlayerStats(a)) or 0
        local lb = tonumber(getPlayerStats(b)) or 0
        return la > lb
    end)

    for i, p in ipairs(players) do
        local isTarget = (p.Name == TargetName)
        local level, money = getPlayerStats(p)

        -- si es el objetivo, usamos los valores falsos para mostrar
        if isTarget then
            level = fakeLevel
            money = fakeMoney
        end

        local row = makeRow(p.Name, level, money, isTarget)
        row.LayoutOrder = i
        row.Parent = container
    end

    -- ajustar CanvasSize
    local total = #container:GetChildren()
    local rows = 0
    for _,c in pairs(container:GetChildren()) do
        if c:IsA("Frame") then rows = rows + 1 end
    end
    local contentHeight = rows * 32
    container.CanvasSize = UDim2.new(0, 0, 0, contentHeight)
end

-- Actualizaciones periódicas + hooks para cambios
rebuildList()
local lastUpdate = 0
RunService.Heartbeat:Connect(function(dt)
    lastUpdate = lastUpdate + dt
    if lastUpdate >= refreshInterval then
        rebuildList()
        lastUpdate = 0
    end
end)

-- Actualizar cuando jugadores entren/salgan o cambien leaderstats
Players.PlayerAdded:Connect(function() rebuildList() end)
Players.PlayerRemoving:Connect(function() rebuildList() end)

-- También escuchar cambios en leaderstats de cualquier jugador (si existen)
Players.PlayerAdded:Connect(function(pl)
    pl.ChildAdded:Connect(function() rebuildList() end)
    pl.ChildRemoved:Connect(function() rebuildList() end)
end)
for _,pl in pairs(Players:GetPlayers()) do
    pl.ChildAdded:Connect(function() rebuildList() end)
    pl.ChildRemoved:Connect(function() rebuildList() end)
end

-- Abrir/Cerrar con Tab (mismo comportamiento visual que leaderboard nativo)
local visible = false
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Tab then
        visible = not visible
        mainFrame.Visible = visible
    end
end)

-- Atajo para ocultar si el jugador muere o pierde foco (mejora estética)
LocalPlayer.CharacterAdded:Connect(function() mainFrame.Visible = false visible = false end)
