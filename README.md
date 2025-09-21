--== Serviços ==--
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local LocalPlayer = Players.LocalPlayer

--== Variáveis ==--
local EquipLoop = false
local Dragging = false
local DragOffset = Vector2.new(0,0)
local LoopMode = "safe" -- "safe", "turbo", "ultra"
local Minimized = false
local LastPosition = UDim2.new(0.5, -100, 0.6, -150)

--== Counters ==--
local EquipCount = 0
local UnequipCount = 0

--== Variáveis do Modo Proteção ==--
local ProtectionModeActive = false
local PlayerAppearanceBackup = {}
local playerAddedConnection = nil

--== CORES DOS BOTÕES ==--
local redColor = Color3.fromRGB(140, 20, 20)
local greenColor = Color3.fromRGB(25, 170, 25)

--[[
  É permitido fazer alterações neste script com o auxílio de inteligência artificial ou por outros meios,
  desde que haja a devida autorização de Mozart, o criador original.

  Ao realizar qualquer modificação, é obrigatório manter os créditos de autoria originais de Mozart.
  A remoção ou adulteração de qualquer assinatura, marca ou crédito, inclusive do TIKTOK: MOZARTMZ01,
  será considerada uma violação dos direitos autorais.
]]

--== Notificação Customizada ==--
local function showNotification(msg, duration)
    duration = duration or 2
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Mozart LAG",
            Text = msg,
            Duration = duration
        })
    end)
end

--== Função: Strip Character ==--
local function stripCharacter()
    local character = LocalPlayer.Character
    if not character then return end
    for _, child in pairs(character:GetChildren()) do
        if child:IsA("Accessory") or child:IsA("Shirt") or child:IsA("Pants") then
            child:Destroy()
        end
    end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        if humanoid:FindFirstChild("BodyHeightScale") then humanoid.BodyHeightScale.Value = 0.7 end
        if humanoid:FindFirstChild("BodyWidthScale") then humanoid.BodyWidthScale.Value = 0.7 end
        if humanoid:FindFirstChild("BodyDepthScale") then humanoid.BodyDepthScale.Value = 0.7 end
        if humanoid:FindFirstChild("HeadScale") then humanoid.HeadScale.Value = 0.7 end
    end
end

--== Função: AutoEquip (FPS DEVOURER) ==--
local function toggleToolLoop(tool)
    EquipLoop = true
    local character = LocalPlayer.Character
    if not character or not tool then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if LoopMode == "ultra" then
        local connection
        connection = RunService.RenderStepped:Connect(function()
            if not EquipLoop then
                if connection then connection:Disconnect() end
                return
            end
            if tool.Parent ~= character then
                tool.Parent = LocalPlayer.Backpack
                UnequipCount += 1
            end
            humanoid:EquipTool(tool)
            EquipCount += 1
            if _G.UpdateCounter then _G.UpdateCounter() end
        end)
        return
    end

    while EquipLoop do
        if tool.Parent ~= character then
            tool.Parent = LocalPlayer.Backpack
            UnequipCount += 1
        end
        humanoid:EquipTool(tool)
        EquipCount += 1
        if _G.UpdateCounter then _G.UpdateCounter() end
        if LoopMode == "safe" then
            task.wait(0.1)
        elseif LoopMode == "turbo" then
            task.wait()
        end
    end
end

local function runAutoEquip()
    local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    stripCharacter()
    repeat task.wait() until character:FindFirstChild("HumanoidRootPart")
    EquipCount, UnequipCount = 0, 0
    if _G.UpdateCounter then _G.UpdateCounter() end
    local tool = character:FindFirstChildWhichIsA("Tool")
    
    if not tool then
        showNotification("PEGUE O ITEM NA MÃO PARA FUNCIONAR", 2.2)
        return
    end

    task.spawn(function()
        toggleToolLoop(tool)
    end)
end

--== Funções do Modo Proteção ==--
local function saveAndStripPlayer(player)
    local character = player.Character
    if not character or PlayerAppearanceBackup[player.UserId] then return end
    PlayerAppearanceBackup[player.UserId] = {}
    for _, item in ipairs(character:GetChildren()) do
        if item:IsA("Accessory") or item:IsA("Shirt") or item:IsA("Pants") then
            local itemClone = item:Clone()
            itemClone.Parent = nil
            table.insert(PlayerAppearanceBackup[player.UserId], itemClone)
            item:Destroy()
        end
    end
end

local function restorePlayerAppearance(player)
    local character = player.Character
    local backup = PlayerAppearanceBackup[player.UserId]
    if not character or not backup then return end
    for _, itemClone in ipairs(backup) do
        itemClone.Parent = character
    end
    PlayerAppearanceBackup[player.UserId] = nil
end

-- ========================================================= --
-- [[ LÓGICA DA FUNÇÃO FLY (Exatamente como você pediu) ]] --
-- ========================================================= --
local isFlying = false
local floor = nil
local connection = nil
local flyRaycastParams = nil
local flySpeed = 1.5
local slowWalkSpeed = 10
local flyOriginalWalkSpeed = 16

local function toggleFly()
    isFlying = not isFlying
    local character = LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local rootPart = character and character:FindFirstChild("HumanoidRootPart")

    if isFlying then
        if not humanoid or not rootPart then isFlying = false; return end
        showNotification("Fly Ativado!", 1.5)
        
        flyOriginalWalkSpeed = humanoid.WalkSpeed
        humanoid.WalkSpeed = slowWalkSpeed

        local platformThickness = 0.8

        floor = Instance.new("Part")
        floor.Name = "ChaoVoador"
        floor.Size = Vector3.new(6, platformThickness, 6)
        floor.Material = Enum.Material.Neon
        floor.Anchored = true
        floor.CanCollide = true
        
        local feetOffset = 3 + (platformThickness / 2)
        floor.CFrame = rootPart.CFrame - Vector3.new(0, feetOffset, 0)
        floor.Parent = workspace

        flyRaycastParams = RaycastParams.new()
        flyRaycastParams.FilterType = Enum.RaycastFilterType.Exclude
        flyRaycastParams.FilterDescendantsInstances = {character, floor}

        connection = RunService.RenderStepped:Connect(function()
            local char = LocalPlayer.Character
            local currentRootPart = char and char:FindFirstChild("HumanoidRootPart")
            if not isFlying or not currentRootPart or not floor or not floor.Parent then
                toggleFly()
                return
            end

            local canMoveUp = true
            local rayOrigin = floor.Position + Vector3.new(0, floor.Size.Y / 2, 0)
            local rayDirection = Vector3.new(0, 1.5, 0)
            local result = workspace:Raycast(rayOrigin, rayDirection, flyRaycastParams)
            if result then canMoveUp = false end

            local hue = tick() % 6 / 6
            floor.Color = Color3.fromHSV(hue, 1, 1)

            local playerPos = currentRootPart.Position
            local newFloorY = floor.Position.Y
            if canMoveUp then newFloorY = newFloorY + (flySpeed * 0.1) end
            floor.CFrame = CFrame.new(playerPos.X, newFloorY, playerPos.Z)

            if playerPos.Y < floor.Position.Y then
                currentRootPart.CFrame = CFrame.new(playerPos.X, floor.Position.Y + 4, playerPos.Z)
            end
        end)
    else
        showNotification("Fly Desativado!", 1.5)
        if humanoid then humanoid.WalkSpeed = flyOriginalWalkSpeed end

        if connection then connection:Disconnect(); connection = nil end
        if floor then floor:Destroy(); floor = nil end
    end
end

--== GUI ==--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoToolEquipMozartMobile"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

--== Main Frame ==--
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 200, 0, 250) -- << GUI MENOR
Frame.Position = LastPosition
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = false
Frame.Visible = true
Frame.Parent = ScreenGui

local UICorner_Frame = Instance.new("UICorner")
UICorner_Frame.CornerRadius = UDim.new(0, 8)
UICorner_Frame.Parent = Frame

local UIStroke_Frame = Instance.new("UIStroke")
UIStroke_Frame.Color = Color3.fromRGB(80, 80, 85)
UIStroke_Frame.Thickness = 1
UIStroke_Frame.Parent = Frame

--== Minimize Button ==--
local MiniBtn = Instance.new("TextButton")
MiniBtn.Size = UDim2.new(0, 24, 0, 24)
MiniBtn.Position = UDim2.new(1, -28, 0, 3)
MiniBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
MiniBtn.Text = "_"
MiniBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
MiniBtn.TextSize = 16
MiniBtn.Font = Enum.Font.SourceSansBold
MiniBtn.AutoButtonColor = false
MiniBtn.Parent = Frame

local UICorner_MiniBtn = Instance.new("UICorner")
UICorner_MiniBtn.CornerRadius = UDim.new(0, 5)
UICorner_MiniBtn.Parent = MiniBtn

--== Title ==--
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,30)
Title.Position = UDim2.new(0,0,0,2)
Title.BackgroundTransparency = 1
Title.Text = "MOZART LAG"
Title.Font = Enum.Font.Arcade
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Parent = Frame

local TitleStroke = Instance.new("UIStroke")
TitleStroke.Thickness = 0.5
TitleStroke.Color = Color3.fromRGB(25,25,30)
TitleStroke.Parent = Title

--== TikTok Label ==--
local TiktokLabel = Instance.new("TextLabel")
TiktokLabel.Size = UDim2.new(1,0,0,15)
TiktokLabel.Position = UDim2.new(0,0,0.09,0)
TiktokLabel.BackgroundTransparency = 1
TiktokLabel.Text = "TIKTOK: MOZARTMZ01"
TiktokLabel.Font = Enum.Font.Arcade
TiktokLabel.TextSize = 12
TiktokLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
TiktokLabel.Parent = Frame

--== Função para aplicar estilo aos botões ==--
local function styleAsPixelButton(button, text)
    button.Size = UDim2.new(0.85, 0, 0, 34)
    button.Font = Enum.Font.Arcade
    button.Text = text
    button.TextSize = 16
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.BackgroundColor3 = redColor
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = button
    
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 0.4
    stroke.Color = Color3.fromRGB(25,25,30)
    stroke.Parent = button
end

--== FPS DEVOURER Button ==--
local FPSDevourerButton = Instance.new("TextButton")
FPSDevourerButton.Position = UDim2.new(0.075, 0, 0.18, 0) -- << Posição ajustada
FPSDevourerButton.Parent = Frame
styleAsPixelButton(FPSDevourerButton, "FPS DEVOURER")

local function updateFPSDevourerButton()
    if EquipLoop then
        FPSDevourerButton.BackgroundColor3 = greenColor
    else
        FPSDevourerButton.BackgroundColor3 = redColor
    end
end
updateFPSDevourerButton()

FPSDevourerButton.MouseButton1Click:Connect(function()
    if not EquipLoop then
        local character = LocalPlayer.Character
        local tool = character and character:FindFirstChildWhichIsA("Tool")
        if tool then
            EquipLoop = true
            task.spawn(runAutoEquip)
        else
            showNotification("PEGUE O ITEM NA MÃO PARA FUNCIONAR", 2.2)
        end
    else
        EquipLoop = false
    end
    updateFPSDevourerButton()
end)

--== LoopMode Button ==--
local LoopModeButton = Instance.new("TextButton")
LoopModeButton.Position = UDim2.new(0.075, 0, 0.33, 0) -- << Posição ajustada
LoopModeButton.Parent = Frame
styleAsPixelButton(LoopModeButton, "MODO: SAFE")

local function updateLoopModeButton()
    if LoopMode == "safe" then
        LoopModeButton.Text = "MODO: SAFE"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(25, 170, 25)
    elseif LoopMode == "turbo" then
        LoopModeButton.Text = "MODO: TURBO"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
    else -- ultra
        LoopModeButton.Text = "MODO: ULTRA"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(220, 20, 20)
    end
end
updateLoopModeButton()

LoopModeButton.MouseButton1Click:Connect(function()
    if LoopMode == "safe" then LoopMode = "turbo"
    elseif LoopMode == "turbo" then LoopMode = "ultra"
    else LoopMode = "safe" end
    showNotification("Modo: " .. LoopMode:upper() .. "!", 2)
    updateLoopModeButton()
end)

--== Protection Mode Button ==--
local ProtectionModeButton = Instance.new("TextButton")
ProtectionModeButton.Position = UDim2.new(0.075, 0, 0.48, 0) -- << Posição ajustada
ProtectionModeButton.Parent = Frame
styleAsPixelButton(ProtectionModeButton, "PROTEÇÃO")

local function updateProtectionButton()
    if ProtectionModeActive then
        ProtectionModeButton.BackgroundColor3 = greenColor
    else
        ProtectionModeButton.BackgroundColor3 = redColor
    end
end
updateProtectionButton()

ProtectionModeButton.MouseButton1Click:Connect(function()
    ProtectionModeActive = not ProtectionModeActive
    updateProtectionButton()
    if ProtectionModeActive then
        showNotification("Modo Proteção Ativado!", 2.5)
        for _, player in ipairs(Players:GetPlayers()) do saveAndStripPlayer(player) end
        playerAddedConnection = Players.PlayerAdded:Connect(function(newPlayer)
            newPlayer.CharacterAdded:Wait()
            task.wait(0.5)
            saveAndStripPlayer(newPlayer)
        end)
    else
        showNotification("Modo Proteção Desativado!", 2.5)
        if playerAddedConnection then playerAddedConnection:Disconnect(); playerAddedConnection = nil; end
        for _, player in ipairs(Players:GetPlayers()) do restorePlayerAppearance(player) end
    end
end)

--== Safe Mode Button ==--
local SafeModeButton = Instance.new("TextButton")
SafeModeButton.Position = UDim2.new(0.075, 0, 0.63, 0) -- << Posição ajustada
SafeModeButton.Parent = Frame
styleAsPixelButton(SafeModeButton, "SAFE MODE")

local SafeModeActive = false
local SafeModePart = nil

local function updateSafeModeButton()
    if SafeModeActive then
        SafeModeButton.BackgroundColor3 = greenColor
    else
        SafeModeButton.BackgroundColor3 = redColor
    end
end
updateSafeModeButton()

local function activateSafeMode()
    local character = LocalPlayer.Character
    if not character then return end
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    SafeModeActive = not SafeModeActive
    if SafeModeActive then
        root.CFrame = CFrame.new(0,600,0)
        SafeModePart = Instance.new("Part")
        SafeModePart.Size = Vector3.new(700,10,700)
        SafeModePart.Anchored = true
        SafeModePart.Position = Vector3.new(0,595,0)
        SafeModePart.CanCollide = true
        SafeModePart.Transparency = 1
        SafeModePart.Parent = workspace
        showNotification("Safe Mode ativado!",2)
    else
        if SafeModePart then SafeModePart:Destroy(); SafeModePart = nil; end
        root.Anchored = false
        showNotification("Safe Mode desativado!",2)
    end
    updateSafeModeButton()
end
SafeModeButton.MouseButton1Click:Connect(activateSafeMode)

--== BOTÃO FLY ==--
local FlyButton = Instance.new("TextButton")
FlyButton.Position = UDim2.new(0.075, 0, 0.78, 0) -- << Posição ajustada
FlyButton.Parent = Frame
styleAsPixelButton(FlyButton, "FLY")

local function updateFlyButton()
    if isFlying then
        FlyButton.BackgroundColor3 = greenColor
    else
        FlyButton.BackgroundColor3 = redColor
    end
end
updateFlyButton()

FlyButton.MouseButton1Click:Connect(function()
    toggleFly()
    updateFlyButton()
end)

--== Drag Logic ==--
Frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragOffset = Vector2.new(input.Position.X - Frame.AbsolutePosition.X, input.Position.Y - Frame.AbsolutePosition.Y)
    end
end)
Frame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
        LastPosition = Frame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if Dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local newX = input.Position.X - DragOffset.X
        local newY = input.Position.Y - DragOffset.Y
        Frame.Position = UDim2.new(0, newX, 0, newY)
    end
end)

--== Minimize/Maximize Logic ==--
local BallBtn = Instance.new("TextButton")
BallBtn.Size = UDim2.new(0, 42, 0, 42)
BallBtn.Position = UDim2.new(0, 40, 0, 100)
BallBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
BallBtn.BackgroundTransparency = 0.1
BallBtn.Text = ""
BallBtn.Visible = false
BallBtn.Parent = ScreenGui
BallBtn.AutoButtonColor = true
BallBtn.AnchorPoint = Vector2.new(0.5, 0.5)
BallBtn.ZIndex = 10

local BallUICorner = Instance.new("UICorner")
BallUICorner.CornerRadius = UDim.new(1, 0)
BallUICorner.Parent = BallBtn

local BallIcon = Instance.new("TextLabel")
BallIcon.Size = UDim2.new(1,0,1,0)
BallIcon.BackgroundTransparency = 1
BallIcon.Text = "♫"
BallIcon.Font = Enum.Font.SourceSansBold
BallIcon.TextSize = 30
BallIcon.TextColor3 = Color3.fromRGB(255,255,255)
BallIcon.Parent = BallBtn

local BallStroke = Instance.new("UIStroke")
BallStroke.Color = Color3.fromRGB(80,80,85)
BallStroke.Thickness = 1
BallStroke.Parent = BallBtn

local BallDragging = false
local BallOffset = Vector2.new(0,0)
BallBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        BallDragging = true
        BallOffset = Vector2.new(input.Position.X - BallBtn.AbsolutePosition.X, input.Position.Y - BallBtn.AbsolutePosition.Y)
    end
end)
BallBtn.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        BallDragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if BallDragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local newX = input.Position.X - BallOffset.X
        local newY = input.Position.Y - BallOffset.Y
        BallBtn.Position = UDim2.new(0, newX, 0, newY)
    end
end)

MiniBtn.MouseButton1Click:Connect(function()
    LastPosition = Frame.Position
    Frame.Visible = false
    BallBtn.Position = UDim2.new(LastPosition.X.Scale, LastPosition.X.Offset + (Frame.Size.X.Offset / 2), LastPosition.Y.Scale, LastPosition.Y.Offset + (Frame.Size.Y.Offset / 2))
    BallBtn.Visible = true
    Minimized = true
end)
BallBtn.MouseButton1Click:Connect(function()
    Frame.Visible = true
    BallBtn.Visible = false
    Frame.Position = LastPosition
    Minimized = false
end)
