local Config = {
    {
        Name = "ooityu", 
        Level = 16027,   
        Money = "$16,380,744"
    }
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local function getConfig(name)
    for _, cfg in ipairs(Config) do
        if cfg.Name == name then
            return cfg
        end
    end
end

local function spoofStatsPersistent(player, cfg)
    local ls = player:WaitForChild("leaderstats", 10)
    if not ls then return end

    RunService.RenderStepped:Connect(function()
        
        if ls:FindFirstChild("Level") then
            ls.Level.Value = tonumber(cfg.Level)  
        end

        
        if ls:FindFirstChild("Money") then
            ls.Money.Value = tostring(cfg.Money)
        end
    end)
end

local function spoofLevelOverhead(player, cfg)
    local function applyToCharacter(char)
        for _, billboard in ipairs(char:GetChildren()) do
            if billboard:IsA("BillboardGui") then
                for _, label in ipairs(billboard:GetDescendants()) do
                    if label:IsA("TextLabel") then
                        local txt = label.Text:lower()

                        if txt:find("level") or txt:find("lvl") or txt:match("%d+") then
                            label.RichText = true
                            label.TextScaled = true
                            label.TextStrokeTransparency = 0.1

                            RunService.RenderStepped:Connect(function()
                                label.Text = "<font color='rgb(255,255,255)'>"..cfg.Level.."</font>"
                            end)
                        end
                    end
                end
            end
        end

        char.ChildAdded:Connect(function(child)
            if child:IsA("BillboardGui") then
                task.wait(0.5)
                spoofLevelOverhead(player, cfg)
            end
        end)
    end

    if player.Character then
        applyToCharacter(player.Character)
    end
    player.CharacterAdded:Connect(applyToCharacter)
end

local function applySpoof(player, cfg)
    spoofStatsPersistent(player, cfg)
    spoofLevelOverhead(player, cfg)
end

for _, cfg in ipairs(Config) do
    local plr = Players:FindFirstChild(cfg.Name)
    if plr then
        applySpoof(plr, cfg)
    end
end

Players.PlayerAdded:Connect(function(plr)
    local cfg = getConfig(plr.Name)
    if cfg then
        applySpoof(plr, cfg)
    end
end)
