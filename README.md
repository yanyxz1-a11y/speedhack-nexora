--!strict
-- Speed Controller GUI with Drag & Toggle Support
-- Target: Modern Luau / QuorumHub UNC Compatible

local Players = cloneref and cloneref(game:GetService("Players")) or game:GetService("Players")
local UserInputService = cloneref and cloneref(game:GetService("UserInputService")) or game:GetService("UserInputService")
local RunService = cloneref and cloneref(game:GetService("RunService")) or game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
assert(LocalPlayer, "LocalPlayer not found")

-- Determine safe GUI parent
local function getSafeGuiParent(): Instance
    if gethui then
        return gethui()
    end
    local coreGui = game:GetService("CoreGui")
    if coreGui then
        return coreGui
    end
    return LocalPlayer:WaitForChild("PlayerGui")
end

-- Cleanup existing instance if re-executed
local existingGui = getSafeGuiParent():FindFirstChild("QuorumSpeedHub")
if existingGui then
    existingGui:Destroy()
end

-- State variables
local speedEnabled = false
local customSpeed = 50
local defaultSpeed = 16

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "QuorumSpeedHub"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = getSafeGuiParent()

-- Main Window
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 240, 0, 160)
mainFrame.Position = UDim2.new(0.5, -120, 0.4, -80)
mainFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 8)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(50, 50, 60)
mainStroke.Thickness = 1
mainStroke.Parent = mainFrame

-- Top Bar (Draggable Area)
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 32)
titleBar.BackgroundColor3 = Color3.fromRGB(32, 32, 38)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Name = "Title"
titleText.Size = UDim2.new(1, -12, 1, 0)
titleText.Position = UDim2.new(0, 12, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Font = Enum.Font.GothamBold
titleText.Text = "Speed Controller"
titleText.TextColor3 = Color3.fromRGB(235, 235, 240)
titleText.TextSize = 13
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Parent = titleBar

-- Dragging Logic
local dragging = false
local dragStart: Vector3 = Vector3.zero
local startPos: UDim2 = UDim2.new()

titleBar.InputBegan:Connect(function(input: InputObject)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position

        local connection: RBXScriptConnection?
        connection = input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
                if connection then
                    connection:Disconnect()
                end
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input: InputObject)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- Toggle Button
local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleButton"
toggleBtn.Size = UDim2.new(1, -24, 0, 36)
toggleBtn.Position = UDim2.new(0, 12, 0, 44)
toggleBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
toggleBtn.BorderSizePixel = 0
toggleBtn.Font = Enum.Font.GothamSemibold
toggleBtn.Text = "Status: DISABLED"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.TextSize = 13
toggleBtn.AutoButtonColor = true
toggleBtn.Parent = mainFrame

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 6)
toggleCorner.Parent = toggleBtn

-- Speed Value Container
local inputFrame = Instance.new("Frame")
inputFrame.Name = "InputFrame"
inputFrame.Size = UDim2.new(1, -24, 0, 32)
inputFrame.Position = UDim2.new(0, 12, 0, 88)
inputFrame.BackgroundColor3 = Color3.fromRGB(32, 32, 38)
inputFrame.BorderSizePixel = 0
inputFrame.Parent = mainFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 6)
inputCorner.Parent = inputFrame

local inputLabel = Instance.new("TextLabel")
inputLabel.Name = "Label"
inputLabel.Size = UDim2.new(0.5, 0, 1, 0)
inputLabel.Position = UDim2.new(0, 8, 0, 0)
inputLabel.BackgroundTransparency = 1
inputLabel.Font = Enum.Font.Gotham
inputLabel.Text = "Target Speed:"
inputLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
inputLabel.TextSize = 12
inputLabel.TextXAlignment = Enum.TextXAlignment.Left
inputLabel.Parent = inputFrame

local speedInput = Instance.new("TextBox")
speedInput.Name = "SpeedBox"
speedInput.Size = UDim2.new(0.4, 0, 0.75, 0)
speedInput.Position = UDim2.new(0.55, 0, 0.125, 0)
speedInput.BackgroundColor3 = Color3.fromRGB(44, 44, 52)
speedInput.BorderSizePixel = 0
speedInput.Font = Enum.Font.GothamBold
speedInput.Text = tostring(customSpeed)
speedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
speedInput.TextSize = 12
speedInput.ClearTextOnFocus = false
speedInput.Parent = inputFrame

local boxCorner = Instance.new("UICorner")
boxCorner.CornerRadius = UDim.new(0, 4)
boxCorner.Parent = speedInput

-- Footer note
local footerLabel = Instance.new("TextLabel")
footerLabel.Name = "Footer"
footerLabel.Size = UDim2.new(1, 0, 0, 20)
footerLabel.Position = UDim2.new(0, 0, 1, -24)
footerLabel.BackgroundTransparency = 1
footerLabel.Font = Enum.Font.Gotham
footerLabel.Text = "Drag title bar to move"
footerLabel.TextColor3 = Color3.fromRGB(120, 120, 130)
footerLabel.TextSize = 10
footerLabel.Parent = mainFrame

-- Speed Update Routine
local function getHumanoid(): Humanoid?
    local char = LocalPlayer.Character
    if char then
        return char:FindFirstChildOfClass("Humanoid")
    end
    return nil
end

local function applySpeed()
    local humanoid = getHumanoid()
    if humanoid then
        if speedEnabled then
            humanoid.WalkSpeed = customSpeed
        else
            humanoid.WalkSpeed = defaultSpeed
        end
    end
end

-- Toggle Handler
toggleBtn.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    if speedEnabled then
        toggleBtn.Text = "Status: ENABLED"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 160, 80)
    else
        toggleBtn.Text = "Status: DISABLED"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    end
    applySpeed()
end)

-- Input Focus Lost Handler
speedInput.FocusLost:Connect(function()
    local num = tonumber(speedInput.Text)
    if num and num > 0 then
        customSpeed = num
    else
        speedInput.Text = tostring(customSpeed)
    end
    if speedEnabled then
        applySpeed()
    end
end)

-- Keep speed active on respawn & enforce on loop if enabled
LocalPlayer.CharacterAdded:Connect(function(character)
    character:WaitForChild("Humanoid")
    task.wait(0.2)
    applySpeed()
end)

RunService.Heartbeat:Connect(function()
    if speedEnabled then
        local humanoid = getHumanoid()
        if humanoid and humanoid.WalkSpeed ~= customSpeed then
            humanoid.WalkSpeed = customSpeed
        end
    end
end)
