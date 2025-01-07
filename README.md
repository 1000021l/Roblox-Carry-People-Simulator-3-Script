-- Services 
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- GUI Setup
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = playerGui
screenGui.Name = "GrabTazeGUI"
screenGui.ResetOnSpawn = false  -- This ensures the GUI persists after respawning

-- Draggable GUI setup
local dragging = false
local dragInput, dragStart, startPos

-- Function to make the GUI draggable
local function onDrag(input)
    if dragging then
        local delta = input.Position - dragStart
        screenGui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end

-- Toggle GUI visibility with "Insert" key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.Insert then
        screenGui.Enabled = not screenGui.Enabled
    end
end)

-- Create buttons for actions
local grabButton = Instance.new("TextButton")
grabButton.Size = UDim2.new(0, 200, 0, 50)
grabButton.Position = UDim2.new(0, 10, 0, 10)
grabButton.Text = "Grab Closest Player"
grabButton.Parent = screenGui

local tazeButton = Instance.new("TextButton")
tazeButton.Size = UDim2.new(0, 200, 0, 50)
tazeButton.Position = UDim2.new(0, 10, 0, 70)
tazeButton.Text = "Taze Closest Player"
tazeButton.Parent = screenGui

-- Function to find the closest player to the local player
local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local localCharacter = player.Character
    if not localCharacter then return nil end

    local localPosition = localCharacter.HumanoidRootPart.Position

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local otherPosition = otherPlayer.Character.HumanoidRootPart.Position
            local distance = (localPosition - otherPosition).Magnitude
            if distance < shortestDistance then
                closestPlayer = otherPlayer
                shortestDistance = distance
            end
        end
    end

    return closestPlayer
end

-- Function to grab the closest player when the "Grab" button is clicked
grabButton.MouseButton1Click:Connect(function()
    local closestPlayer = getClosestPlayer()
    if closestPlayer then
        local character = closestPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local args = {
                [1] = character.HumanoidRootPart,
                [2] = "Grab",
                [3] = Vector3.new(47, 70.63288879394531, 2677.952880859375),
                [4] = CFrame.new(-13.345794677734375, 82.92008209228516, 2680.66162109375, 
                -0.044845204800367355, 0.1991249918937683, 0.9789475202560425, 
                -0, 0.9799333214759827, -0.1993255317211151, 
                -0.9989939332008362, -0.008938794024288654, -0.0439453087747097)
            }
            ReplicatedStorage:WaitForChild("Events"):WaitForChild("Grab"):FireServer(unpack(args))
        end
    end
end)

-- Function to taze the closest player when the "Taze" button is clicked
tazeButton.MouseButton1Click:Connect(function()
    local closestPlayer = getClosestPlayer()
    if closestPlayer then
        local character = closestPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local args = {
                [1] = character.HumanoidRootPart,
                [2] = "Taser", 
                [3] = Vector3.new(47, 70.63288879394531, 2677.952880859375),
                [4] = CFrame.new(-13.345794677734375, 82.92008209228516, 2680.66162109375, 
                -0.044845204800367355, 0.1991249918937683, 0.9789475202560425, 
                -0, 0.9799333214759827, -0.1993255317211151, 
                -0.9989939332008362, -0.008938794024288654, -0.0439453087747097)
            }
            ReplicatedStorage:WaitForChild("Events"):WaitForChild("Taze"):FireServer(unpack(args))
        end
    end
end)

-- Dragging functionality
grabButton.InputBegan:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = screenGui.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        onDrag(input)
    end
end)
