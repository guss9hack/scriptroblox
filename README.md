-- ESP Script for Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Settings
local ESP_ENABLED = true
local TEAM_CHECK = true
local BOX_COLOR = Color3.fromRGB(255, 0, 0)
local TEXT_COLOR = Color3.fromRGB(255, 255, 255)
local TEXT_SIZE = 15

-- Store ESP objects
local ESPObjects = {}

-- Function to create ESP for a player
local function CreateESP(player)
    if player == LocalPlayer then return end
    
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    local humanoid = character:WaitForChild("Humanoid")
    
    -- Create ESP container
    local espHolder = Instance.new("Folder")
    espHolder.Name = player.Name .. "_ESP"
    espHolder.Parent = Camera
    
    -- Create box outline
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "Box"
    box.Adornee = humanoidRootPart
    box.AlwaysOnTop = true
    box.ZIndex = 1
    box.Size = Vector3.new(3, 5, 3)
    box.Transparency = 0.7
    box.Color3 = BOX_COLOR
    box.Parent = espHolder
    
    -- Create name label
    local nameLabel = Instance.new("BillboardGui")
    nameLabel.Name = "NameLabel"
    nameLabel.Adornee = humanoidRootPart
    nameLabel.Size = UDim2.new(0, 200, 0, 50)
    nameLabel.StudsOffset = Vector3.new(0, 3, 0)
    nameLabel.AlwaysOnTop = true
    
    local nameText = Instance.new("TextLabel")
    nameText.Name = "Text"
    nameText.Size = UDim2.new(1, 0, 1, 0)
    nameText.BackgroundTransparency = 1
    nameText.Text = player.Name
    nameText.TextColor3 = TEXT_COLOR
    nameText.TextSize = TEXT_SIZE
    nameText.Font = Enum.Font.SourceSansBold
    nameText.TextStrokeTransparency = 0
    nameText.Parent = nameLabel
    
    nameLabel.Parent = espHolder
    
    -- Create health label
    local healthLabel = Instance.new("BillboardGui")
    healthLabel.Name = "HealthLabel"
    healthLabel.Adornee = humanoidRootPart
    healthLabel.Size = UDim2.new(0, 200, 0, 50)
    healthLabel.StudsOffset = Vector3.new(0, 1.5, 0)
    healthLabel.AlwaysOnTop = true
    
    local healthText = Instance.new("TextLabel")
    healthText.Name = "Text"
    healthText.Size = UDim2.new(1, 0, 1, 0)
    healthText.BackgroundTransparency = 1
    healthText.TextColor3 = TEXT_COLOR
    healthText.TextSize = TEXT_SIZE
    healthText.Font = Enum.Font.SourceSansBold
    healthText.TextStrokeTransparency = 0
    healthText.Parent = healthLabel
    
    healthLabel.Parent = espHolder
    
    -- Store ESP objects
    ESPObjects[player] = {
        Holder = espHolder,
        Box = box,
        NameLabel = nameLabel,
        HealthLabel = healthLabel,
        Humanoid = humanoid
    }
    
    -- Update health function
    local function UpdateHealth()
        if ESPObjects[player] then
            local healthText = ESPObjects[player].HealthLabel:FindFirstChild("Text")
            if healthText then
                healthText.Text = "Health: " .. math.floor(humanoid.Health) .. "/" .. humanoid.MaxHealth
            end
        end
    end
    
    -- Initial health update
    UpdateHealth()
    
    -- Connect health changed event
    humanoid.HealthChanged:Connect(UpdateHealth)
    
    -- Team check update
    if TEAM_CHECK then
        player:GetPropertyChangedSignal("Team"):Connect(function()
            if player.Team == LocalPlayer.Team then
                box.Color3 = Color3.fromRGB(0, 255, 0) -- Green for teammates
            else
                box.Color3 = BOX_COLOR -- Red for enemies
            end
        end)
        
        -- Initial team check
        if player.Team == LocalPlayer.Team then
            box.Color3 = Color3.fromRGB(0, 255, 0) -- Green for teammates
        end
    end
end

-- Function to remove ESP for a player
local function RemoveESP(player)
    if ESPObjects[player] then
        ESPObjects[player].Holder:Destroy()
        ESPObjects[player] = nil
    end
end

-- Main ESP loop
local function UpdateESP()
    if not ESP_ENABLED then return end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if not ESPObjects[player] then
                CreateESP(player)
            end
        elseif ESPObjects[player] then
            RemoveESP(player)
        end
    end
end

-- Connect player added/removed events
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if ESP_ENABLED then
            CreateESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
end)

-- Initialize ESP for existing players
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer and player.Character then
        CreateESP(player)
    end
end

-- Main loop
RunService.RenderStepped:Connect(UpdateESP)

-- Toggle function (for testing)
local function ToggleESP()
    ESP_ENABLED = not ESP_ENABLED
    if not ESP_ENABLED then
        for player, _ in pairs(ESPObjects) do
            RemoveESP(player)
        end
    end
    print("ESP is now " .. (ESP_ENABLED and "enabled" or "disabled"))
end

-- Uncomment to add a toggle key (for testing)
-- game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
--     if input.KeyCode == Enum.KeyCode.F1 then
--         ToggleESP()
--     end
-- end)

print("ESP script loaded. Use at your own risk Coded By Guss.")
