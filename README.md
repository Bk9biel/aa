local teamCheck = false

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local fov = 150
local smoothing = 1
local aimbotEnabled = true
local fovVisible = true
local aimPart = "Head" -- Padrão: Cabeça

local function getBestAimPart(character)
    if character:FindFirstChild("UpperTorso") then
        return "UpperTorso"
    elseif character:FindFirstChild("Torso") then
        return "Torso"
    elseif character:FindFirstChild("HumanoidRootPart") then
        return "HumanoidRootPart"
    end
    return nil
end

local FOVring = Drawing.new("Circle")
FOVring.Visible = fovVisible
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = workspace.CurrentCamera.ViewportSize / 2

local function getClosestTarget()
    local closestTarget = nil
    local closestDist = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") then
            local bestPart = getBestAimPart(player.Character)
            if bestPart and player.Character:FindFirstChild(bestPart) then
                if not teamCheck or (player.Team ~= LocalPlayer.Team) then
                    local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(player.Character[bestPart].Position)
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - workspace.CurrentCamera.ViewportSize / 2).Magnitude
                    
                    if onScreen and dist < closestDist and dist <= fov then
                        closestDist = dist
                        closestTarget = player
                    end
                end
            end
        end
    end
    return closestTarget
end

local function aimAt(target)
    if target and target.Character then
        local bestPart = getBestAimPart(target.Character)
        if bestPart and target.Character:FindFirstChild(bestPart) then
            local aimPosition = target.Character[bestPart].Position
            workspace.CurrentCamera.CFrame = workspace.CurrentCamera.CFrame:Lerp(CFrame.new(workspace.CurrentCamera.CFrame.Position, aimPosition), smoothing)
        end
    end
end

RunService.RenderStepped:Connect(function()
    FOVring.Radius = fov
    FOVring.Position = workspace.CurrentCamera.ViewportSize / 2
    FOVring.Visible = fovVisible

    if aimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestTarget()
        aimAt(target)
    end
end)

-- GUI
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local EclipseMenuLabel = Instance.new("TextLabel")
local ToggleAimbotButton = Instance.new("TextButton")
local ToggleFOVButton = Instance.new("TextButton")
local ChangeAimPartButton = Instance.new("TextButton")
local FOVIncreaseButton = Instance.new("TextButton")
local FOVDecreaseButton = Instance.new("TextButton")
local FOVLabel = Instance.new("TextLabel")
local CloseButton = Instance.new("TextButton")

ScreenGui.Parent = game.Players.LocalPlayer.PlayerGui

Frame.Size = UDim2.new(0, 250, 0, 360)
Frame.Position = UDim2.new(0, 50, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.Parent = ScreenGui

EclipseMenuLabel.Size = UDim2.new(0, 230, 0, 30)
EclipseMenuLabel.Position = UDim2.new(0, 10, 0, 10)
EclipseMenuLabel.Text = "🚀ECLIPSE MENU🚀"
EclipseMenuLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
EclipseMenuLabel.TextSize = 16
EclipseMenuLabel.Parent = Frame

ToggleAimbotButton.Size = UDim2.new(0, 230, 0, 40)
ToggleAimbotButton.Position = UDim2.new(0, 10, 0, 60)
ToggleAimbotButton.Text = "Aimbot: ON"
ToggleAimbotButton.Parent = Frame

ToggleFOVButton.Size = UDim2.new(0, 230, 0, 40)
ToggleFOVButton.Position = UDim2.new(0, 10, 0, 110)
ToggleFOVButton.Text = "Exibir FOV: ON"
ToggleFOVButton.Parent = Frame

ChangeAimPartButton.Size = UDim2.new(0, 230, 0, 40)
ChangeAimPartButton.Position = UDim2.new(0, 10, 0, 160)
ChangeAimPartButton.Text = "Mira em: Cabeça"
ChangeAimPartButton.Parent = Frame

FOVLabel.Size = UDim2.new(0, 230, 0, 20)
FOVLabel.Position = UDim2.new(0, 10, 0, 200)
FOVLabel.BackgroundTransparency = 1
FOVLabel.Text = "Tamanho do FOV: " .. fov
FOVLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
FOVLabel.Parent = Frame

FOVIncreaseButton.Size = UDim2.new(0, 100, 0, 40)
FOVIncreaseButton.Position = UDim2.new(0, 10, 0, 230)
FOVIncreaseButton.Text = "+"
FOVIncreaseButton.Parent = Frame

FOVDecreaseButton.Size = UDim2.new(0, 100, 0, 40)
FOVDecreaseButton.Position = UDim2.new(0, 130, 0, 230)
FOVDecreaseButton.Text = "-"
FOVDecreaseButton.Parent = Frame

CloseButton.Size = UDim2.new(0, 230, 0, 40)
CloseButton.Position = UDim2.new(0, 10, 0, 280)
CloseButton.Text = "Fechar Menu"
CloseButton.Parent = Frame

ToggleAimbotButton.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    ToggleAimbotButton.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)

ToggleFOVButton.MouseButton1Click:Connect(function()
    fovVisible = not fovVisible
    ToggleFOVButton.Text = "Exibir FOV: " .. (fovVisible and "ON" or "OFF")
end)

ChangeAimPartButton.MouseButton1Click:Connect(function()
    aimPart = (aimPart == "Head") and "Auto" or "Head"
    ChangeAimPartButton.Text = "Mira em: " .. ((aimPart == "Head") and "Cabeça" or "Peito")
end)

FOVIncreaseButton.MouseButton1Click:Connect(function()
    if fov < 250 then
        fov = fov + 10
        FOVring.Radius = fov
        FOVLabel.Text = "Tamanho do FOV: " .. fov
    end
end)

FOVDecreaseButton.MouseButton1Click:Connect(function()
    if fov > 50 then
        fov = fov - 10
        FOVring.Radius = fov
        FOVLabel.Text = "Tamanho do FOV: " .. fov
    end
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)
