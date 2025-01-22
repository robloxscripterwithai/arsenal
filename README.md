local
-- Arsenal GUI
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local ScrollingFrame = Instance.new("ScrollingFrame")
local UIListLayout = Instance.new("UIListLayout")
local AimbotButton = Instance.new("TextButton")
local ESPButton = Instance.new("TextButton")
local SpeedButton = Instance.new("TextButton")
local TriggerButton = Instance.new("TextButton")

-- GUI Setup
ScreenGui.Parent = game.CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

MainFrame.Name = "Simple cheat"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.Position = UDim2.new(0.1, 0, 0.3, 0)
MainFrame.Size = UDim2.new(0, 220, 0, 300)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true

-- Add corner radius
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

Title.Name = "Title"
Title.Parent = MainFrame
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Font = Enum.Font.GothamBold
Title.Text = "Simple cheat"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 22

-- Add corner radius to title
local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = Title

ScrollingFrame.Parent = MainFrame
ScrollingFrame.BackgroundTransparency = 1
ScrollingFrame.Position = UDim2.new(0, 0, 0, 50)
ScrollingFrame.Size = UDim2.new(1, 0, 1, -50)
ScrollingFrame.ScrollBarThickness = 4
ScrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(255, 255, 255)

UIListLayout.Parent = ScrollingFrame
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Button Template Function
local function CreateButton(name, position)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Parent = ScrollingFrame
    button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    button.Size = UDim2.new(0.9, 0, 0, 45)
    button.Font = Enum.Font.GothamSemibold
    button.Text = name .. ": OFF"
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 16
    button.AutoButtonColor = false
    button.LayoutOrder = position
    
    -- Add hover effect
    button.MouseEnter:Connect(function()
        if button.Text:match("OFF") then
            game:GetService("TweenService"):Create(button, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}):Play()
        end
    end)
    
    button.MouseLeave:Connect(function()
        if button.Text:match("OFF") then
            game:GetService("TweenService"):Create(button, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(45, 45, 45)}):Play()
        end
    end)
    
    -- Add corner radius to button
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 8)
    ButtonCorner.Parent = button
    
    return button
end

-- Create Buttons
local AimbotButton = CreateButton("Aimbot [MB2]", 1)
local ESPButton = CreateButton("ESP", 2)
local SpeedButton = CreateButton("Speed", 3)
local TriggerButton = CreateButton("Triggerbot", 4)

-- Button Functions
local function updateButton(button, enabled)
    local baseText = button.Name:gsub(" %[.*%]", "")
    button.Text = baseText .. (button == AimbotButton and " [MB2]" or "") .. ": " .. (enabled and "ON" or "OFF")
    local goal = {
        BackgroundColor3 = enabled and Color3.fromRGB(0, 170, 255) or Color3.fromRGB(45, 45, 45)
    }
    game:GetService("TweenService"):Create(button, TweenInfo.new(0.3), goal):Play()
end

local function isTeammate(player)
    local LocalPlayer = game:GetService("Players").LocalPlayer
    return player.Team == LocalPlayer.Team
end

local function getClosestPlayer()
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local Camera = workspace.CurrentCamera
    
    local closestPlayer = nil
    local shortestDistance = math.huge
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and 
           player.Character and 
           player.Character:FindFirstChild("Head") and
           not isTeammate(player) then
            
            local screenPoint = Camera:WorldToScreenPoint(player.Character.Head.Position)
            local vectorDistance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
            
            if vectorDistance < shortestDistance then
                closestPlayer = player
                shortestDistance = vectorDistance
            end
        end
    end
    
    return closestPlayer
end

local aimbotEnabled = false
local espEnabled = false
local speedEnabled = false
local triggerbotEnabled = false
local triggerConnection = nil

-- Aimbot Implementation
local UserInputService = game:GetService("UserInputService")

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        aimbotEnabled = true
        updateButton(AimbotButton, aimbotEnabled)
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        aimbotEnabled = false
        updateButton(AimbotButton, aimbotEnabled)
    end
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if aimbotEnabled then
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("Head") then
            local Camera = workspace.CurrentCamera
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, closestPlayer.Character.Head.Position)
        end
    end
end)

AimbotButton.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    updateButton(AimbotButton, aimbotEnabled)
end)

TriggerButton.MouseButton1Click:Connect(function()
    triggerbotEnabled = not triggerbotEnabled
    updateButton(TriggerButton, triggerbotEnabled)
    
    if triggerbotEnabled then
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer
        local Mouse = LocalPlayer:GetMouse()
        
        if triggerConnection then
            triggerConnection:Disconnect()
        end
        
        triggerConnection = game:GetService("RunService").RenderStepped:Connect(function()
            if triggerbotEnabled then
                local target = Mouse.Target
                if target and target.Parent then
                    local character = target.Parent
                    local player = Players:GetPlayerFromCharacter(character)
                    
                    if player and not isTeammate(player) then
                        mouse1press()
                        task.wait(0.05)
                        mouse1release()
                        task.wait(0.1)
                    end
                end
            end
        end)
    else
        if triggerConnection then
            triggerConnection:Disconnect()
            triggerConnection = nil
        end
    end
end)

ESPButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    updateButton(ESPButton, espEnabled)
    
    local function createESPBox(player)
        if not player.Character then return end
        
        local box = Instance.new("BoxHandleAdornment")
        box.Name = "ESPBox"
        box.Adornee = player.Character
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = player.Character:GetExtentsSize()
        box.Transparency = 0.5
        box.Color3 = isTeammate(player) and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        
        local billboardGui = Instance.new("BillboardGui")
        billboardGui.Name = "ESPTag"
        billboardGui.Adornee = player.Character:WaitForChild("Head")
        billboardGui.Size = UDim2.new(0, 100, 0, 30)
        billboardGui.StudsOffset = Vector3.new(0, 2, 0)
        billboardGui.AlwaysOnTop = true
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.BackgroundTransparency = 1
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextScaled = true
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.Parent = billboardGui
        
        box.Parent = player.Character
        billboardGui.Parent = player.Character
    end
    
    local function removeESP(player)
        if player.Character then
            local box = player.Character:FindFirstChild("ESPBox")
            local tag = player.Character:FindFirstChild("ESPTag")
            if box then box:Destroy() end
            if tag then tag:Destroy() end
        end
    end
    
    local function onCharacterAdded(player)
        if espEnabled then
            createESPBox(player)
        end
    end
    
    local function onCharacterRemoving(player)
        removeESP(player)
    end
    
    local Players = game:GetService("Players")
    
    if espEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= Players.LocalPlayer then
                createESPBox(player)
                player.CharacterAdded:Connect(function()
                    onCharacterAdded(player)
                end)
                player.CharacterRemoving:Connect(function()
                    onCharacterRemoving(player)
                end)
            end
        end
        
        Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function()
                onCharacterAdded(player)
            end)
            player.CharacterRemoving:Connect(function()
                onCharacterRemoving(player)
            end)
        end)
    else
        for _, player in pairs(Players:GetPlayers()) do
            removeESP(player)
        end
    end
end)

SpeedButton.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    updateButton(SpeedButton, speedEnabled)
    
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local RunService = game:GetService("RunService")
    
    if speedEnabled then
        RunService:BindToRenderStep("SpeedHack", 100, function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid.WalkSpeed = 50
            end
        end)
    else
        RunService:UnbindFromRenderStep("SpeedHack")
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = 16
        end
    end
end)

-- Update ScrollingFrame canvas size
UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 20)
end)
