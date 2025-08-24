--// CONFIGURACIÓN
local TargetName = "NombreDelJugador" -- 👈 Aquí pones el nombre de la persona
local fakeNivel = 3450
local fakeDinero = "10,500,000"

--// SERVICIOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

--// CREAR UI
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local PlayerLabel = Instance.new("TextLabel")
local NivelLabel = Instance.new("TextLabel")
local DineroLabel = Instance.new("TextLabel")

-- GUI Setup
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.Size = UDim2.new(0, 280, 0, 150)
Frame.Position = UDim2.new(0.05, 0, 0.25, 0)
Frame.BackgroundTransparency = 0.2
Frame.Active = true
Frame.Draggable = true

-- Nombre del jugador
PlayerLabel.Parent = Frame
PlayerLabel.Size = UDim2.new(1, 0, 0.3, 0)
PlayerLabel.BackgroundTransparency = 1
PlayerLabel.TextColor3 = Color3.fromRGB(0, 180, 255)
PlayerLabel.Font = Enum.Font.SourceSansBold
PlayerLabel.TextScaled = true
PlayerLabel.Text = "Jugador: " .. TargetName

-- Nivel
NivelLabel.Parent = Frame
NivelLabel.Size = UDim2.new(1, 0, 0.3, 0)
NivelLabel.Position = UDim2.new(0, 0, 0.33, 0)
NivelLabel.BackgroundTransparency = 1
NivelLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
NivelLabel.Font = Enum.Font.SourceSansBold
NivelLabel.TextScaled = true
NivelLabel.Text = "Nivel: " .. fakeNivel

-- Dinero
DineroLabel.Parent = Frame
DineroLabel.Size = UDim2.new(1, 0, 0.3, 0)
DineroLabel.Position = UDim2.new(0, 0, 0.66, 0)
DineroLabel.BackgroundTransparency = 1
DineroLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
DineroLabel.Font = Enum.Font.SourceSansBold
DineroLabel.TextScaled = true
DineroLabel.Text = "Dinero: $" .. fakeDinero
