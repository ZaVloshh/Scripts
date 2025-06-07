local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Teams = game:GetService("Teams")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

-- Настройки
local DEFAULT_RADIUS = 15
local ATTACK_RANGE = 3.0
local ATTACK_COOLDOWN = 0.3
local TOGGLE_KEY = Enum.KeyCode.H
local TARGET_KEY = Enum.KeyCode.T
local active = false
local selectedTarget = nil
local highlightColor = Color3.fromRGB(255, 50, 50)
local lastTargetUpdate = 0
local baseWalkSpeed = 16
local lastAttackTime = 0
local equippedTool = nil
local minFollowDistance = 1.5
local maxFollowDistance = 5.0
local puddleTransparency = 0.9
local isPuddlesIgnored = true
local targetAcquiredSpeed = 17

-- Команды
local humanTeam = Teams.Humans
local infectedTeam = Teams.Infected
local currentTeam = player.Team

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StealthAura"
ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 240, 0, 250)
MainFrame.Position = UDim2.new(0.01, 0, 0.01, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
MainFrame.BackgroundTransparency = 0.3
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 80, 20)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Text = "STEALTH AURA"
Title.Size = UDim2.new(1, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.Parent = TitleBar

local SettingsFrame = Instance.new("Frame")
SettingsFrame.Size = UDim2.new(1, 0, 1, -30)
SettingsFrame.Position = UDim2.new(0, 0, 0, 30)
SettingsFrame.BackgroundTransparency = 1
SettingsFrame.Parent = MainFrame

-- Создание элементов GUI
local function createSettingLabel(text, yPosition)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.4, 0, 0, 20)
    label.Position = UDim2.new(0.05, 0, yPosition, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(0.8, 0.8, 0.8)
    label.Text = text
    label.Font = Enum.Font.SourceSans
    label.TextSize = 16
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = SettingsFrame
    return label
end

local function createValueLabel(yPosition)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.5, 0, 0, 20)
    label.Position = UDim2.new(0.45, 0, yPosition, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 16
    label.TextXAlignment = Enum.TextXAlignment.Right
    label.Parent = SettingsFrame
    return label
end

createSettingLabel("Status:", 0.05)
local StatusLabel = createValueLabel(0.05)
StatusLabel.Text = "OFF"

createSettingLabel("Team:", 0.15)
local TeamLabel = createValueLabel(0.15)
TeamLabel.Text = currentTeam and currentTeam.Name or "NONE"

createSettingLabel("Target:", 0.25)
local TargetLabel = createValueLabel(0.25)
TargetLabel.Text = "NONE"

createSettingLabel("Speed:", 0.35)
local SpeedLabel = createValueLabel(0.35)
SpeedLabel.Text = "16"

createSettingLabel("Puddles:", 0.45)
local PuddleLabel = createValueLabel(0.45)
PuddleLabel.Text = isPuddlesIgnored and "IGNORED" or "VISIBLE"

-- Создание кнопок
local function createButton(text, yPosition, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.9, 0, 0, 30)
    button.Position = UDim2.new(0.05, 0, yPosition, 0)
    button.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Text = text
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 16
    button.Parent = SettingsFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 6)
    buttonCorner.Parent = button
    
    button.MouseButton1Click:Connect(callback)
    return button
end

local ToggleBtn = createButton("TOGGLE BOT (H)", 0.6, function()
    active = not active
    StatusLabel.Text = active and "ON" or "OFF"
    StatusLabel.TextColor3 = active and Color3.new(0, 1, 0.5) or Color3.new(1, 1, 1)
end)

local TogglePuddlesBtn = createButton("TOGGLE PUDDLES", 0.75, function()
    isPuddlesIgnored = not isPuddlesIgnored
    updatePuddles()
    PuddleLabel.Text = isPuddlesIgnored and "IGNORED" or "VISIBLE"
end)

-- Система подсветки
local highlights = {}

local function createHighlight(targetChar)
    if not targetChar then return end
    
    if highlights[targetChar] then
        highlights[targetChar]:Destroy()
        highlights[targetChar] = nil
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "TargetHighlight"
    highlight.FillColor = highlightColor
    highlight.OutlineColor = highlightColor
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = targetChar
    
    highlights[targetChar] = highlight
end

local function removeHighlight(targetChar)
    if targetChar and highlights[targetChar] then
        highlights[targetChar]:Destroy()
        highlights[targetChar] = nil
    end
end

-- Улучшенная обработка луж с визуальными эффектами
local function updatePuddles()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChild("TouchInterest") and obj.Parent.Name == "Model" then
            if isPuddlesIgnored then
                -- Режим игнорирования: лужи прозрачные и сквозные
                obj.Transparency = puddleTransparency
                obj.CanCollide = false
                obj.CanTouch = false
            else
                -- Режим видимости: лужи видны и взаимодействуют
                obj.Transparency = 0
                obj.CanCollide = true
                obj.CanTouch = true
            end
        end
    end
    PuddleLabel.Text = isPuddlesIgnored and "IGNORED" or "VISIBLE"
end

-- Отслеживание новых луж
workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("BasePart") and obj:FindFirstChild("TouchInterest") and obj.Parent.Name == "Model" then
        task.wait()
        if isPuddlesIgnored then
            obj.Transparency = puddleTransparency
            obj.CanCollide = false
            obj.CanTouch = false
        else
            obj.Transparency = 0
            obj.CanCollide = true
            obj.CanTouch = true
        end
    end
end)

-- Определение противоположной команды
local function getOppositeTeam()
    if not currentTeam then return nil end
    return currentTeam == humanTeam and infectedTeam or humanTeam
end

-- Проверка, может ли игрок атаковать цель
local function isValidTarget(target)
    if not target or not target.Character then return false end
    return target.Team == getOppositeTeam()
end

-- Оптимальная дистанция притягивания
local function getOptimalPosition(targetRoot)
    if not targetRoot then return rootPart.Position end
    
    local targetCF = targetRoot.CFrame
    local behindDistance = 2.0
    
    local toTarget = (targetRoot.Position - rootPart.Position).Unit
    local dotProduct = targetCF.LookVector:Dot(toTarget)
    
    if dotProduct > 0.7 then
        behindDistance = 1.5
    end
    
    local behindPosition = targetCF.Position - targetCF.LookVector * behindDistance
    return Vector3.new(behindPosition.X, rootPart.Position.Y, behindPosition.Z)
end

-- Ускорение при обнаружении цели
local function setTargetSpeed()
    if selectedTarget and humanoid and humanoid.Health > 0 then
        humanoid.WalkSpeed = targetAcquiredSpeed
        SpeedLabel.Text = "17"
    else
        humanoid.WalkSpeed = baseWalkSpeed
        SpeedLabel.Text = "16"
    end
end

-- Плавное притягивание
local function smoothMoveToTarget(targetRoot)
    if not targetRoot or not humanoid or humanoid:GetState() == Enum.HumanoidStateType.Dead then 
        return 
    end
    
    local desiredPosition = getOptimalPosition(targetRoot)
    local distance = (desiredPosition - rootPart.Position).Magnitude
    
    setTargetSpeed()
    
    if distance > minFollowDistance then
        humanoid:MoveTo(desiredPosition)
    end
    
    local targetDirection = (targetRoot.Position - rootPart.Position).Unit
    local currentLook = rootPart.CFrame.LookVector
    local smoothDirection = currentLook:Lerp(targetDirection, 0.08)
    
    rootPart.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + smoothDirection)
end

-- Экипировка биты для человека
local function equipBat()
    if currentTeam ~= humanTeam then return false end
    
    for _, tool in ipairs(character:GetChildren()) do
        if tool.Name == "Bat" then
            equippedTool = tool
            return true
        end
    end
    
    if player.Backpack then
        local bat = player.Backpack:FindFirstChild("Bat")
        if bat then
            humanoid:EquipTool(bat)
            equippedTool = bat
            return true
        end
    end
    
    equippedTool = nil
    return false
end

-- Надежная система атаки
local function performAttack()
    if currentTeam == humanTeam then
        if equipBat() and equippedTool then
            equippedTool:Activate()
            
            local input = Instance.new("InputObject")
            input.UserInputType = Enum.UserInputType.MouseButton1
            input.UserInputState = Enum.UserInputState.Begin
            UserInputService:ProcessInput(input)
            
            wait(0.05)
            
            input.UserInputState = Enum.UserInputState.End
            UserInputService:ProcessInput(input)
        end
    else
        local input = Instance.new("InputObject")
        input.UserInputType = Enum.UserInputType.MouseButton1
        input.UserInputState = Enum.UserInputState.Begin
        UserInputService:ProcessInput(input)
        
        wait(0.05)
        
        input.UserInputState = Enum.UserInputState.End
        UserInputService:ProcessInput(input)
    end
end

-- Поиск ближайшей цели
local function findClosestTarget()
    local closestDist = math.huge
    local closestTarget = nil
    local oppositeTeam = getOppositeTeam()
    
    if not oppositeTeam then return nil end
    
    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and target.Team == oppositeTeam and target.Character 
            and target.Character:FindFirstChild("Humanoid") 
            and target.Character.Humanoid.Health > 0 
            and target.Character:FindFirstChild("HumanoidRootPart") then
            
            local targetRoot = target.Character.HumanoidRootPart
            local distance = (rootPart.Position - targetRoot.Position).Magnitude
            
            if distance < closestDist and distance <= DEFAULT_RADIUS then
                closestDist = distance
                closestTarget = target
            end
        end
    end
    
    return closestTarget
end

-- Горячие клавиши
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    
    if input.KeyCode == Enum.KeyCode.F8 then
        MainFrame.Visible = not MainFrame.Visible
    elseif input.KeyCode == TOGGLE_KEY then
        active = not active
        StatusLabel.Text = active and "ON" or "OFF"
        StatusLabel.TextColor3 = active and Color3.new(0, 1, 0.5) or Color3.new(1, 1, 1)
    elseif input.KeyCode == TARGET_KEY then
        if selectedTarget then
            removeHighlight(selectedTarget.Character)
            selectedTarget = nil
            TargetLabel.Text = "NONE"
            setTargetSpeed()
        else
            selectedTarget = findClosestTarget()
            if selectedTarget and selectedTarget.Character then
                createHighlight(selectedTarget.Character)
                TargetLabel.Text = selectedTarget.Name
                setTargetSpeed()
            end
        end
    end
end)

-- Обновление команды игрока
local function updatePlayerTeam()
    currentTeam = player.Team
    TeamLabel.Text = currentTeam and currentTeam.Name or "NONE"
end

player:GetPropertyChangedSignal("Team"):Connect(updatePlayerTeam)

-- Обновление персонажа
player.CharacterAdded:Connect(function(char)
    character = char
    rootPart = char:WaitForChild("HumanoidRootPart")
    humanoid = char:WaitForChild("Humanoid")
    humanoid.WalkSpeed = baseWalkSpeed
    equippedTool = nil
    updatePlayerTeam()
    
    if selectedTarget and selectedTarget.Character then
        createHighlight(selectedTarget.Character)
    end
end)

-- Основной цикл
RunService.Heartbeat:Connect(function()
    local currentTime = tick()
    updatePlayerTeam()
    
    if not active then 
        if humanoid.WalkSpeed ~= baseWalkSpeed then
            humanoid.WalkSpeed = baseWalkSpeed
            SpeedLabel.Text = "16"
        end
        return 
    end
    
    if currentTime - lastTargetUpdate > 0.1 then
        lastTargetUpdate = currentTime
        local closest = findClosestTarget()
        
        if not selectedTarget and closest then
            selectedTarget = closest
            if selectedTarget and selectedTarget.Character then
                createHighlight(selectedTarget.Character)
                TargetLabel.Text = selectedTarget.Name
                setTargetSpeed()
            end
        end
    end
    
    if selectedTarget and selectedTarget.Character and selectedTarget.Character:FindFirstChild("HumanoidRootPart") then
        local targetRoot = selectedTarget.Character.HumanoidRootPart
        local targetHum = selectedTarget.Character:FindFirstChild("Humanoid")
        
        if not targetHum or targetHum.Health <= 0 or not isValidTarget(selectedTarget) then
            removeHighlight(selectedTarget.Character)
            selectedTarget = nil
            TargetLabel.Text = "NONE"
            setTargetSpeed()
            return
        end
        
        local distance = (rootPart.Position - targetRoot.Position).Magnitude
        
        if distance <= DEFAULT_RADIUS then
            smoothMoveToTarget(targetRoot)
            
            if distance < ATTACK_RANGE then
                if currentTime - lastAttackTime > ATTACK_COOLDOWN then
                    performAttack()
                    lastAttackTime = currentTime
                end
            end
        else
            removeHighlight(selectedTarget.Character)
            selectedTarget = nil
            TargetLabel.Text = "NONE"
            setTargetSpeed()
        end
    elseif selectedTarget then
        removeHighlight(selectedTarget.Character)
        selectedTarget = nil
        TargetLabel.Text = "NONE"
        setTargetSpeed()
    end
end)

-- Очистка при выходе игрока
Players.PlayerRemoving:Connect(function(playerLeft)
    if playerLeft == selectedTarget then
        removeHighlight(selectedTarget.Character)
        selectedTarget = nil
        TargetLabel.Text = "NONE"
        setTargetSpeed()
    end
end)

-- Инициализация
updatePlayerTeam()
humanoid.WalkSpeed = baseWalkSpeed
SpeedLabel.Text = "16"
updatePuddles()

print("Stealth Aura FINAL VERSION loaded! Press F8 to toggle GUI, H to toggle bot, T to clear target")
