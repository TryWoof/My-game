local Players = game:GetService("Players")
local player = Players.LocalPlayer

local FARM_POSITION = Vector3.new(10.9, 11.6, 310.9)
local WIN_POSITION = Vector3.new(43.2, 858.3, 325.2)

local autoWin = false

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "NeonControlPanel"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(300, 255)
frame.Position = UDim2.new(0.5, -150, 0.5, -127)
frame.BackgroundColor3 = Color3.fromRGB(18, 18, 29)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 14)
corner.Parent = frame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(135, 75, 255)
stroke.Thickness = 1.5
stroke.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -20, 0, 40)
title.Position = UDim2.fromOffset(10, 8)
title.BackgroundTransparency = 1
title.Text = "✦  NEON CONTROL"
title.TextColor3 = Color3.fromRGB(190, 145, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 19
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = frame

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -20, 0, 20)
subtitle.Position = UDim2.fromOffset(10, 42)
subtitle.BackgroundTransparency = 1
subtitle.Text = "TELEPORT • MOVEMENT"
subtitle.TextColor3 = Color3.fromRGB(135, 135, 160)
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 10
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = frame

local function makeButton(text, y, color)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -24, 0, 48)
    button.Position = UDim2.fromOffset(12, y)
    button.BackgroundColor3 = color
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Font = Enum.Font.GothamBold
    button.TextSize = 14
    button.AutoButtonColor = true
    button.Parent = frame

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 9)
    c.Parent = button

    return button
end

local farmButton = makeButton(
    "⚡   FARM JUMP",
    76,
    Color3.fromRGB(105, 65, 210)
)

local winButton = makeButton(
    "🏆   AUTO WIN : OFF",
    132,
    Color3.fromRGB(35, 125, 100)
)

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -24, 0, 28)
status.Position = UDim2.fromOffset(12, 193)
status.BackgroundTransparency = 1
status.Text = "●  Système prêt"
status.TextColor3 = Color3.fromRGB(100, 225, 165)
status.Font = Enum.Font.Gotham
status.TextSize = 12
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = frame

local function getCharacter()
    local character = player.Character
    if not character then
        character = player.CharacterAdded:Wait()
    end

    local root = character:WaitForChild("HumanoidRootPart", 5)
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    if root and humanoid then
        return character, root, humanoid
    end
end

-- FARM JUMP

farmButton.MouseButton1Click:Connect(function()
    local character, root, humanoid = getCharacter()

    if not root or not humanoid then
        status.Text = "Personnage introuvable"
        return
    end

    -- Téléportation
    root.CFrame = CFrame.new(FARM_POSITION)

    -- Forcer le mode JumpPower
    humanoid.UseJumpPower = true
    humanoid.JumpPower = 1

    -- Vérifier la valeur appliquée
    task.wait(0.1)

    print("UseJumpPower :", humanoid.UseJumpPower)
    print("JumpPower :", humanoid.JumpPower)

    status.Text = "JumpPower : " .. humanoid.JumpPower
end)

-- AUTO WIN : TP toutes les 2 secondes
winButton.MouseButton1Click:Connect(function()
    autoWin = not autoWin

    if autoWin then
        winButton.Text = "■   AUTO WIN : ON"
        winButton.BackgroundColor3 = Color3.fromRGB(190, 65, 80)
        status.Text = "●  Auto Win activé"
        status.TextColor3 = Color3.fromRGB(100, 225, 165)

        task.spawn(function()
            while autoWin and gui.Parent do
                local success, err = pcall(function()
                    local character, root = getCharacter()

                    if root then
                        root.CFrame = CFrame.new(WIN_POSITION)
                    end
                end)

                if not success then
                    warn("Auto Win :", err)
                end

                task.wait(2)
            end
        end)
    else
        winButton.Text = "🏆   AUTO WIN : OFF"
        winButton.BackgroundColor3 = Color3.fromRGB(35, 125, 100)
        status.Text = "●  Auto Win arrêté"
        status.TextColor3 = Color3.fromRGB(220, 220, 230)
    end
end)

gui.Destroying:Connect(function()
    autoWin = false
end)
