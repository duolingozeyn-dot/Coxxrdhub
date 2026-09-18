local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Общая конфигурация механик
local config = {
    LockOnEnabled = false,
    ESPEnabled = false,
    FlyEnabled = false,
    
    MaxDistance = 120,
    Smoothness = 0.2, -- Плавность наведения
    FlySpeed = 50     -- Скорость полета (стадс/сек)
}

local lockedTarget = nil
local activeHighlights = {}

-- Переменные для механики полета
local flyVelocity = nil
local flyGyro = nil

-- ==========================================
-- 1. ПОЛЬЗОВАТЕЛЬСКИЙ ИНТЕРФЕЙС (GUI)
-- ==========================================

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "GameControlPanel"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local menuFrame = Instance.new("Frame")
menuFrame.Name = "MenuWindow"
menuFrame.Size = UDim2.new(0, 250, 0, 235)
menuFrame.Position = UDim2.new(0, 25, 0.5, -117)
menuFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
menuFrame.BorderSizePixel = 0
menuFrame.Active = true
menuFrame.Draggable = true
menuFrame.Parent = screenGui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 12)
menuCorner.Parent = menuFrame

local menuStroke = Instance.new("UIStroke")
menuStroke.Color = Color3.fromRGB(45, 45, 55)
menuStroke.Thickness = 1.5
menuStroke.Parent = menuFrame

-- Заголовок
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 40)
header.BackgroundTransparency = 1
header.Parent = menuFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -20, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Control Hub"
titleLabel.TextColor3 = Color3.fromRGB(240, 240, 245)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 15
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = header

local divider = Instance.new("Frame")
divider.Size = UDim2.new(1, -20, 0, 1)
divider.Position = UDim2.new(0, 10, 0, 40)
divider.BackgroundColor3 = Color3.fromRGB(35, 35, 42)
divider.BorderSizePixel = 0
divider.Parent = menuFrame

-- Создание кнопки
local function createMenuButton(name, labelText, keyHint, yOffset)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(1, -24, 0, 44)
    button.Position = UDim2.new(0, 12, 0, yOffset)
    button.BackgroundColor3 = Color3.fromRGB(28, 28, 35)
    button.AutoButtonColor = false
    button.Text = ""
    button.Parent = menuFrame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button

    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(40, 40, 50)
    stroke.Thickness = 1
    stroke.Parent = button

    local text = Instance.new("TextLabel")
    text.Size = UDim2.new(1, -65, 1, 0)
    text.Position = UDim2.new(0, 12, 0, 0)
    text.BackgroundTransparency = 1
    text.Text = labelText
    text.TextColor3 = Color3.fromRGB(200, 200, 210)
    text.Font = Enum.Font.GothamMedium
    text.TextSize = 13
    text.TextXAlignment = Enum.TextXAlignment.Left
    text.Parent = button

    -- Подсказка клавиши
    if keyHint then
        local badge = Instance.new("TextLabel")
        badge.Size = UDim2.new(0, 24, 0, 20)
        badge.Position = UDim2.new(1, -55, 0.5, -10)
        badge.BackgroundColor3 = Color3.fromRGB(38, 38, 48)
        badge.Text = keyHint
        badge.TextColor3 = Color3.fromRGB(150, 150, 165)
        badge.Font = Enum.Font.GothamBold
        badge.TextSize = 11
        badge.Parent = button

        local bCorner = Instance.new("UICorner")
        bCorner.CornerRadius = UDim.new(0, 4)
        bCorner.Parent = badge
    end

    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.new(0, 10, 0, 10)
    indicator.Position = UDim2.new(1, -22, 0.5, -5)
    indicator.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
    indicator.BorderSizePixel = 0
    indicator.Parent = button

    local indCorner = Instance.new("UICorner")
    indCorner.CornerRadius = UDim.new(1, 0)
    indCorner.Parent = indicator

    return button, indicator, text
end

local lockBtn, lockIndicator, lockText = createMenuButton("LockBtn", "Lock-On Target", "Q", 52)
local flyBtn, flyIndicator, flyText = createMenuButton("FlyBtn", "Fly Mode", "E", 106)
local espBtn, espIndicator, espText = createMenuButton("EspBtn", "Player ESP", nil, 160)

local function updateButtonVisual(state, indicator, label)
    local targetColor = state and Color3.fromRGB(0, 210, 130) or Color3.fromRGB(80, 80, 90)
    local textColor = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(200, 200, 210)

    TweenService:Create(indicator, TweenInfo.new(0.2), {BackgroundColor3 = targetColor}):Play()
    label.TextColor3 = textColor
end

-- ==========================================
-- 2. ЛОГИКА ESP (HIGHLIGHTS)
-- ==========================================

local function syncPlayerESP(character)
    if not character or character == localPlayer.Character then return end

    local highlight = activeHighlights[character]
    if not highlight then
        highlight = Instance.new("Highlight")
        highlight.Adornee = character
        highlight.FillColor = Color3.fromRGB(0, 160, 255)
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.5
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = character
        activeHighlights[character] = highlight
    end

    highlight.Enabled = config.ESPEnabled
end

local function updateAllESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character then
            syncPlayerESP(player.Character)
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        syncPlayerESP(char)
    end)
end)

-- ==========================================
-- 3. МЕХАНИКА ПОЛЕТА (FLY)
-- ==========================================

local function stopFlight()
    config.FlyEnabled = false
    updateButtonVisual(false, flyIndicator, flyText)

    if flyVelocity then
        flyVelocity:Destroy()
        flyVelocity = nil
    end
    if flyGyro then
        flyGyro:Destroy()
        flyGyro = nil
    end

    local character = localPlayer.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.PlatformStand = false
        end
    end
end

local function startFlight()
    local character = localPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end

    local root = character.HumanoidRootPart
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return end

    config.FlyEnabled = true
    updateButtonVisual(true, flyIndicator, flyText)

    humanoid.PlatformStand = true

    -- Стабилизатор ориентации персонажа
    flyGyro = Instance.new("BodyGyro")
    flyGyro.P = 9e4
    flyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    flyGyro.CFrame = root.CFrame
    flyGyro.Parent = root

    -- Вектор перемещения
    flyVelocity = Instance.new("BodyVelocity")
    flyVelocity.Velocity = Vector3.zero
    flyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    flyVelocity.Parent = root
end

local function toggleFlight()
    if config.FlyEnabled then
        stopFlight()
    else
        startFlight()
    end
end

-- Обработка управления в полете
RunService.RenderStepped:Connect(function()
    if not config.FlyEnabled or not flyVelocity or not flyGyro then return end

    local character = localPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        stopFlight()
        return
    end

    local camCF = camera.CFrame
    local direction = Vector3.zero

    -- Считываем активные нажатия клавиш
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        direction = direction + camCF.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        direction = direction - camCF.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        direction = direction - camCF.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        direction = direction + camCF.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        direction = direction + Vector3.new(0, 1, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        direction = direction - Vector3.new(0, 1, 0)
    end

    if direction.Magnitude > 0 then
        direction = direction.Unit
    end

    flyVelocity.Velocity = direction * config.FlySpeed
    flyGyro.CFrame = camCF
end)

-- Сброс полета при гибели персонажа
local function onCharacterAdded(char)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.Died:Connect(stopFlight)
end

if localPlayer.Character then
    onCharacterAdded(localPlayer.Character)
end
localPlayer.CharacterAdded:Connect(onCharacterAdded)

-- ==========================================
-- 4. ФИКСАЦИЯ КАМЕРЫ (LOCK-ON)
-- ==========================================

local function getTarget()
    local myChar = localPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end

    local myPos = myChar.HumanoidRootPart.Position
    local bestTarget = nil
    local minDistance = config.MaxDistance

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")

            if root and humanoid and humanoid.Health > 0 then
                local dist = (root.Position - myPos).Magnitude
                if dist < minDistance then
                    minDistance = dist
                    bestTarget = player.Character
                end
            end
        end
    end

    return bestTarget
end

local function toggleLockOn()
    config.LockOnEnabled = not config.LockOnEnabled
    updateButtonVisual(config.LockOnEnabled, lockIndicator, lockText)

    if config.LockOnEnabled then
        lockedTarget = getTarget()
    else
        lockedTarget = nil
    end
end

RunService:BindToRenderStep("CombatCameraLock", Enum.RenderPriority.Camera.Value + 1, function()
    if not config.LockOnEnabled then return end

    if not lockedTarget 
        or not lockedTarget:FindFirstChild("HumanoidRootPart") 
        or not lockedTarget:FindFirstChildOfClass("Humanoid") 
        or lockedTarget:FindFirstChildOfClass("Humanoid").Health <= 0 then
        
        lockedTarget = getTarget()
        if not lockedTarget then return end
    end

    local myChar = localPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end

    local distance = (lockedTarget.HumanoidRootPart.Position - myChar.HumanoidRootPart.Position).Magnitude
    if distance > config.MaxDistance then
        lockedTarget = nil
        return
    end

    local aimPoint = lockedTarget:FindFirstChild("Head") and lockedTarget.Head.Position 
        or lockedTarget.HumanoidRootPart.Position + Vector3.new(0, 1.5, 0)

    local camPos = camera.CFrame.Position
    local desiredCFrame = CFrame.new(camPos, aimPoint)

    camera.CFrame = camera.CFrame:Lerp(desiredCFrame, config.Smoothness)
end)

-- ==========================================
-- 5. ПРИВЯЗКА СОБЫТИЙ И КНОПОК
-- ==========================================

lockBtn.MouseButton1Click:Connect(toggleLockOn)
flyBtn.MouseButton1Click:Connect(toggleFlight)

espBtn.MouseButton1Click:Connect(function()
    config.ESPEnabled = not config.ESPEnabled
    updateButtonVisual(config.ESPEnabled, espIndicator, espText)
    updateAllESP()
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.Q then
        toggleLockOn()
    elseif input.KeyCode == Enum.KeyCode.E then
        toggleFlight()
    end
end)
