--// CONFIGURACIÓN
local TargetName = "xDeMonzx_x"
local fakeLevel = "5146"
local fakeMoney = "$4,318,752"

--// SERVICIOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--// CREAR INTERFAZ
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FakeLeaderboard"
ScreenGui.Parent = PlayerGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 100)
Frame.Position = UDim2.new(0.02, 0, 0.1, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 25)
Title.BackgroundTransparency = 1
Title.Text = "Leaderboard (Fake)"
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Parent = Frame

local NameLabel = Instance.new("TextLabel")
NameLabel.Size = UDim2.new(1, -10, 0, 25)
NameLabel.Position = UDim2.new(0, 5, 0, 35)
NameLabel.BackgroundTransparency = 1
NameLabel.Text = "👤 " .. TargetName
NameLabel.Font = Enum.Font.SourceSans
NameLabel.TextSize = 18
NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
NameLabel.TextXAlignment = Enum.TextXAlignment.Left
NameLabel.Parent = Frame

local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size = UDim2.new(1, -10, 0, 25)
InfoLabel.Position = UDim2.new(0, 5, 0, 65)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Text = "Level: " .. fakeLevel .. " | Money: " .. fakeMoney
InfoLabel.Font = Enum.Font.SourceSans
InfoLabel.TextSize = 18
InfoLabel.TextColor3 = Color3.fromRGB(0, 255, 100)
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left
InfoLabel.Parent = Frame
