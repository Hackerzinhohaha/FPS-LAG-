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

--== Counters ==--
local EquipCount = 0
local UnequipCount = 0

--== Notificação Customizada ==--
local function showNotification(msg, duration)
    duration = duration or 2
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Mozart Scripts",
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

--== Função: AutoEquip ==--
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
            _G.UpdateCounter()
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
        _G.UpdateCounter()

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
    _G.UpdateCounter()

    local tool = character:FindFirstChildWhichIsA("Tool")
    if not tool then
        showNotification("Equipe a Tool para funcionar!", 2.2)
        EquipLoop = false
        return
    end
    task.spawn(function()
        toggleToolLoop(tool)
    end)
end

--== GUI ==--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoToolEquipMozartMobile"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

--== Main Frame ==--
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 230, 0, 240)
Frame.Position = UDim2.new(0.5, -115, 0.6, -120)
Frame.BackgroundColor3 = Color3.fromRGB(32, 34, 37)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = false -- Usamos nosso próprio sistema de drag
Frame.Visible = true
Frame.Parent = ScreenGui

--== Minimize Button ==--
local MiniBtn = Instance.new("TextButton")
MiniBtn.Size = UDim2.new(0, 26, 0, 26)
MiniBtn.Position = UDim2.new(1, -32, 0, 2)
MiniBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
MiniBtn.Text = "✕"
MiniBtn.TextColor3 = Color3.new(1,1,1)
MiniBtn.TextSize = 17
MiniBtn.Font = Enum.Font.GothamBlack
MiniBtn.AutoButtonColor = true
MiniBtn.Parent = Frame

--== Title ==--
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,28)
Title.BackgroundTransparency = 1
Title.Text = "Mozart Tool"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(230,230,230)
Title.Parent = Frame

--== Toggle Button ==--
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.85,0,0,36)
ToggleButton.Position = UDim2.new(0.075,0,0.19,0)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 17
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.Parent = Frame

local function updateToggleButton()
    if EquipLoop then
        ToggleButton.Text = "Desativar"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(200,50,50)
    else
        ToggleButton.Text = "Ativar"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(50,200,100)
    end
end
updateToggleButton()

ToggleButton.MouseButton1Click:Connect(function()
    if EquipLoop then
        EquipLoop = false
        updateToggleButton()
    else
        EquipLoop = true
        updateToggleButton()
        task.spawn(function()
            runAutoEquip()
            updateToggleButton()
        end)
    end
end)

--== LoopMode Button ==--
local LoopModeButton = Instance.new("TextButton")
LoopModeButton.Size = UDim2.new(0.85,0,0,28)
LoopModeButton.Position = UDim2.new(0.075,0,0.36,0)
LoopModeButton.Font = Enum.Font.Gotham
LoopModeButton.TextSize = 14
LoopModeButton.TextColor3 = Color3.new(1,1,1)
LoopModeButton.Parent = Frame

local function updateLoopModeButton()
    if LoopMode == "safe" then
        LoopModeButton.Text = "Modo Seguro"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(40, 120, 220)
    elseif LoopMode == "turbo" then
        LoopModeButton.Text = "Modo Turbo"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(255,200,60)
    else
        LoopModeButton.Text = "Modo ULTRA"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(250,40,240)
    end
end
updateLoopModeButton()

LoopModeButton.MouseButton1Click:Connect(function()
    if LoopMode == "safe" then
        LoopMode = "turbo"
        showNotification("Modo turbo ativado!",2.2)
    elseif LoopMode == "turbo" then
        LoopMode = "ultra"
        showNotification("Modo ULTRA ativado!",2.3)
    else
        LoopMode = "safe"
        showNotification("Modo seguro ativado!",2)
    end
    updateLoopModeButton()
end)

--== COUNTERS ==--
local CounterLabel = Instance.new("TextLabel")
CounterLabel.Size = UDim2.new(0.85,0,0,26)
CounterLabel.Position = UDim2.new(0.075,0,0.51,0)
CounterLabel.BackgroundTransparency = 1
CounterLabel.Font = Enum.Font.GothamSemibold
CounterLabel.TextSize = 15
CounterLabel.TextColor3 = Color3.fromRGB(180, 230, 170)
CounterLabel.Text = "Equip: 0   |   Desequip: 0"
CounterLabel.Parent = Frame

function _G.UpdateCounter()
    CounterLabel.Text = "Equip: "..EquipCount.."   |   Desequip: "..UnequipCount
end

--== FPS / Ping ==--
local PerfLabel = Instance.new("TextLabel")
PerfLabel.Size = UDim2.new(0.85,0,0,21)
PerfLabel.Position = UDim2.new(0.075,0,0.62,0)
PerfLabel.BackgroundTransparency = 1
PerfLabel.Font = Enum.Font.GothamSemibold
PerfLabel.TextSize = 13
PerfLabel.TextColor3 = Color3.fromRGB(180, 200, 255)
PerfLabel.Text = "Ping: ... | FPS: ..."
PerfLabel.Parent = Frame

local fps, frames, lastTime = 0, 0, tick()
RunService.RenderStepped:Connect(function()
    frames += 1
    if tick()-lastTime >= 1 then
        fps = frames
        frames, lastTime = 0, tick()
    end
    local ping = 0
    pcall(function()
        ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
    end)
    PerfLabel.Text = ("Ping: %d ms | FPS: %d"):format(ping, fps)
end)

--== Player Counter ==--
local PlayerLabel = Instance.new("TextLabel")
PlayerLabel.Size = UDim2.new(0.85,0,0,21)
PlayerLabel.Position = UDim2.new(0.075,0,0.71,0)
PlayerLabel.BackgroundTransparency = 1
PlayerLabel.Font = Enum.Font.GothamSemibold
PlayerLabel.TextSize = 13
PlayerLabel.TextColor3 = Color3.fromRGB(200,255,200)
PlayerLabel.Text = "Players: ..."
PlayerLabel.Parent = Frame

local function updatePlayerCount()
    PlayerLabel.Text = ("Players: %d"):format(#Players:GetPlayers())
end
updatePlayerCount()
Players.PlayerAdded:Connect(updatePlayerCount)
Players.PlayerRemoving:Connect(updatePlayerCount)

--== Drag (arrasta pelo Frame inteiro, sem limites de tela) ==--
Frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragOffset = Vector2.new(input.Position.X - Frame.AbsolutePosition.X, input.Position.Y - Frame.AbsolutePosition.Y)
    end
end)
Frame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if Dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local newX = input.Position.X - DragOffset.X
        local newY = input.Position.Y - DragOffset.Y
        -- SEM CLAMP: permite arrastar para fora da tela
        Frame.Position = UDim2.new(0, newX, 0, newY)
    end
end)

--== Créditos ==--
local Credits = Instance.new("TextLabel")
Credits.Size = UDim2.new(1,0,0,15)
Credits.Position = UDim2.new(0,0,0.92,0)
Credits.BackgroundTransparency = 1
Credits.Text = "Mozart Scripts"
Credits.Font = Enum.Font.GothamBold
Credits.TextSize = 13
Credits.TextColor3 = Color3.fromRGB(255,0,0)
Credits.Parent = Frame

task.spawn(function()
    local hue = 0
    while Credits.Parent do
        Credits.TextColor3 = Color3.fromHSV(hue, 0.75, 1)
        hue = (hue + 0.01) % 1
        task.wait(0.05)
    end
end)

--== Bolinha de Minimizar/Maximizar ==--
local BallBtn = Instance.new("TextButton")
BallBtn.Size = UDim2.new(0, 38, 0, 38)
BallBtn.Position = UDim2.new(0, 40, 0, 100)
BallBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 255)
BallBtn.BackgroundTransparency = 0.15
BallBtn.Text = ""
BallBtn.Visible = false
BallBtn.Parent = ScreenGui
BallBtn.AutoButtonColor = true
BallBtn.ClipsDescendants = false
BallBtn.AnchorPoint = Vector2.new(0.5, 0.5)
BallBtn.ZIndex = 10

local BallUICorner = Instance.new("UICorner")
BallUICorner.CornerRadius = UDim.new(1, 0)
BallUICorner.Parent = BallBtn

local BallIcon = Instance.new("TextLabel")
BallIcon.Size = UDim2.new(1,0,1,0)
BallIcon.BackgroundTransparency = 1
BallIcon.Text = "+"
BallIcon.Font = Enum.Font.GothamBlack
BallIcon.TextSize = 30
BallIcon.TextColor3 = Color3.fromRGB(255,255,255)
BallIcon.Parent = BallBtn

-- Arrastar a bolinha (também sem limites)
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

-- Minimizar e restaurar
MiniBtn.MouseButton1Click:Connect(function()
    Frame.Visible = false
    BallBtn.Visible = true
    Minimized = true
end)
BallBtn.MouseButton1Click:Connect(function()
    Frame.Visible = true
    BallBtn.Visible = false
    Minimized = false
end)
