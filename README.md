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
local AutoLoad = false
local LoopMode = "safe" -- "safe", "turbo", "ultra"

--== Counters ==--
local EquipCount = 0
local UnequipCount = 0

--== Notificação Customizada ==--
local function showNotification(msg, duration)
    duration = duration or 2
    local notifFrame = Instance.new("Frame")
    notifFrame.Size = UDim2.new(0, 260, 0, 60)
    notifFrame.Position = UDim2.new(1, -280, 0, 30)
    notifFrame.BackgroundColor3 = Color3.fromRGB(38, 41, 55)
    notifFrame.BackgroundTransparency = 0.07
    notifFrame.BorderSizePixel = 0
    notifFrame.AnchorPoint = Vector2.new(0,0)
    notifFrame.ZIndex = 999
    notifFrame.ClipsDescendants = true
    notifFrame.Parent = game:GetService("CoreGui"):FindFirstChildOfClass("ScreenGui") or game:GetService("CoreGui")

    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0,32,0,32)
    icon.Position = UDim2.new(0,10,0,14)
    icon.BackgroundTransparency = 1
    icon.Image = "rbxassetid://7733960981"
    icon.ZIndex = 1000
    icon.Parent = notifFrame

    local notifText = Instance.new("TextLabel")
    notifText.BackgroundTransparency = 1
    notifText.Position = UDim2.new(0, 54, 0, 7)
    notifText.Size = UDim2.new(1, -60, 1, -14)
    notifText.TextColor3 = Color3.fromRGB(255,255,255)
    notifText.Font = Enum.Font.GothamBold
    notifText.TextSize = 18
    notifText.Text = msg
    notifText.TextWrapped = true
    notifText.TextXAlignment = Enum.TextXAlignment.Left
    notifText.ZIndex = 1000
    notifText.Parent = notifFrame

    notifFrame.BackgroundTransparency = 1
    notifText.TextTransparency = 1
    icon.ImageTransparency = 1
    notifFrame.Position = notifFrame.Position + UDim2.new(0, 40, 0, 0)

    for i=1,10 do
        notifFrame.BackgroundTransparency = 1 - 0.093*i
        notifText.TextTransparency = 1 - 0.1*i
        icon.ImageTransparency = 1 - 0.1*i
        notifFrame.Position = UDim2.new(1, -280 + 4*(10-i), 0, 30)
        wait(0.02)
    end
    notifFrame.BackgroundTransparency = 0.07
    notifText.TextTransparency = 0
    icon.ImageTransparency = 0
    notifFrame.Position = UDim2.new(1, -280, 0, 30)

    local notifSound = Instance.new("Sound")
    notifSound.SoundId = "rbxassetid://6026984224"
    notifSound.Volume = 0.45
    notifSound.Parent = notifFrame
    notifSound:Play()
    game:GetService("Debris"):AddItem(notifSound, 3)

    wait(duration)
    for i=1,10 do
        notifFrame.BackgroundTransparency = 0.07 + 0.093*i
        notifText.TextTransparency = 0 + 0.1*i
        icon.ImageTransparency = 0 + 0.1*i
        notifFrame.Position = UDim2.new(1, -280 + 2*i, 0, 30)
        wait(0.02)
    end
    notifFrame:Destroy()
end

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
        humanoid.BodyHeightScale.Value = 0.7
        humanoid.BodyWidthScale.Value = 0.7
        humanoid.BodyDepthScale.Value = 0.7
        humanoid.HeadScale.Value = 0.7
    end
end

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
                UnequipCount = UnequipCount + 1
            end
            humanoid:EquipTool(tool)
            EquipCount = EquipCount + 1
            _G.UpdateCounter()
        end)
        return
    end

    while EquipLoop do
        if tool.Parent ~= character then
            tool.Parent = LocalPlayer.Backpack
            UnequipCount = UnequipCount + 1
        end
        humanoid:EquipTool(tool)
        EquipCount = EquipCount + 1
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

    EquipCount = 0
    UnequipCount = 0
    _G.UpdateCounter()

    local tool = character:FindFirstChildWhichIsA("Tool")
    if not tool then
        showNotification("Equipe a Tool para funcionar!", 2.2)
        EquipLoop = false
        return
    end
    spawn(function()
        toggleToolLoop(tool)
    end)
end

--== GUI ==--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoToolEquipMozart"
ScreenGui.ResetOnSpawn = false

local success, hui = pcall(function() return gethui() end)
if success and hui then
    ScreenGui.Parent = hui
else
    ScreenGui.Parent = game:GetService("CoreGui")
end

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 380, 0, 370)
Frame.Position = UDim2.new(0, 50, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(32, 34, 37)
Frame.BorderSizePixel = 0
Frame.AnchorPoint = Vector2.new(0,0)
Frame.Parent = ScreenGui

Frame.Active = true
Frame.Draggable = false

local Shadow = Instance.new("ImageLabel")
Shadow.Size = UDim2.new(1, 20, 1, 20)
Shadow.Position = UDim2.new(0, -10, 0, -10)
Shadow.BackgroundTransparency = 1
Shadow.Image = "rbxassetid://1316045217"
Shadow.ImageTransparency = 0.4
Shadow.ZIndex = Frame.ZIndex - 1
Shadow.Parent = Frame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,40)
Title.BackgroundTransparency = 1
Title.Text = "🌟 Equipe a Tool"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Title.TextColor3 = Color3.fromRGB(230,230,230)
Title.TextStrokeTransparency = 0.7
Title.Parent = Frame

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 32, 0, 32)
CloseBtn.Position = UDim2.new(1, -38, 0, 6)
CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.TextSize = 18
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 2
CloseBtn.AutoButtonColor = true
CloseBtn.Parent = Frame

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

local Line = Instance.new("Frame")
Line.Size = UDim2.new(0.9,0,0,2)
Line.Position = UDim2.new(0.05,0,0,42)
Line.BackgroundColor3 = Color3.fromRGB(50,200,100)
Line.BorderSizePixel = 0
Line.Parent = Frame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.8,0,0,44)
ToggleButton.Position = UDim2.new(0.1,0,0.24,0)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 18
ToggleButton.TextColor3 = Color3.fromRGB(255,255,255)
ToggleButton.BorderSizePixel = 0
ToggleButton.AutoButtonColor = false
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

ToggleButton.MouseEnter:Connect(function()
    if EquipLoop then
        ToggleButton.BackgroundColor3 = Color3.fromRGB(230,70,70)
    else
        ToggleButton.BackgroundColor3 = Color3.fromRGB(60,230,120)
    end
end)
ToggleButton.MouseLeave:Connect(updateToggleButton)

ToggleButton.MouseButton1Click:Connect(function()
    if EquipLoop then
        EquipLoop = false
        updateToggleButton()
    else
        EquipLoop = true
        updateToggleButton()
        spawn(function()
            runAutoEquip()
            updateToggleButton()
        end)
    end
end)

-- Botão para alternar o modo de loop
local LoopModeButton = Instance.new("TextButton")
LoopModeButton.Size = UDim2.new(0.8,0,0,36)
LoopModeButton.Position = UDim2.new(0.1,0,0.38,0)
LoopModeButton.Font = Enum.Font.Gotham
LoopModeButton.TextSize = 16
LoopModeButton.TextColor3 = Color3.fromRGB(255,255,255)
LoopModeButton.BorderSizePixel = 0
LoopModeButton.AutoButtonColor = false
LoopModeButton.BackgroundColor3 = Color3.fromRGB(40, 120, 220)
LoopModeButton.Parent = Frame

local function updateLoopModeButton()
    if LoopMode == "safe" then
        LoopModeButton.Text = "Modo Seguro (Recomendado)"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(40, 120, 220)
        LoopModeButton.TextColor3 = Color3.fromRGB(255,255,255)
    elseif LoopMode == "turbo" then
        LoopModeButton.Text = "Modo Turbo (Rápido)"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(255,200,60)
        LoopModeButton.TextColor3 = Color3.fromRGB(60,60,60)
    else
        LoopModeButton.Text = "Modo ULTRA (Máxima velocidade!)"
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(250,40,240)
        LoopModeButton.TextColor3 = Color3.fromRGB(255,255,255)
    end
end
updateLoopModeButton()

LoopModeButton.MouseButton1Click:Connect(function()
    if LoopMode == "safe" then
        LoopMode = "turbo"
        showNotification("Modo turbo ativado!\nMuito rápido.",2.2)
    elseif LoopMode == "turbo" then
        LoopMode = "ultra"
        showNotification("Modo ULTRA ativado!\nO mais rápido possível!",2.3)
    else
        LoopMode = "safe"
        showNotification("Modo seguro ativado!",2)
    end
    updateLoopModeButton()
end)

LoopModeButton.MouseEnter:Connect(function()
    if LoopMode == "safe" then
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(60, 140, 255)
    elseif LoopMode == "turbo" then
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(255,225,100)
    else
        LoopModeButton.BackgroundColor3 = Color3.fromRGB(255,60,250)
    end
end)
LoopModeButton.MouseLeave:Connect(updateLoopModeButton)

-- COUNTER GUI
local CounterFrame = Instance.new("Frame")
CounterFrame.Size = UDim2.new(0.8,0,0,38)
CounterFrame.Position = UDim2.new(0.1,0,0.54,0)
CounterFrame.BackgroundTransparency = 0.4
CounterFrame.BackgroundColor3 = Color3.fromRGB(44, 48, 54)
CounterFrame.BorderSizePixel = 0
CounterFrame.Parent = Frame

local CounterLabel = Instance.new("TextLabel")
CounterLabel.Size = UDim2.new(1,0,1,0)
CounterLabel.BackgroundTransparency = 1
CounterLabel.Font = Enum.Font.GothamSemibold
CounterLabel.TextSize = 17
CounterLabel.TextColor3 = Color3.fromRGB(180, 230, 170)
CounterLabel.TextStrokeTransparency = 0.85
CounterLabel.Text = "Equip: 0   |   Desequip: 0"
CounterLabel.Parent = CounterFrame

function _G.UpdateCounter()
    CounterLabel.Text = "Equip: "..EquipCount.."   |   Desequip: "..UnequipCount
end

-- FPS & PING GUI
local PerfFrame = Instance.new("Frame")
PerfFrame.Size = UDim2.new(0.8,0,0,34)
PerfFrame.Position = UDim2.new(0.1,0,0.68,0)
PerfFrame.BackgroundTransparency = 0.6
PerfFrame.BackgroundColor3 = Color3.fromRGB(24,24,28)
PerfFrame.BorderSizePixel = 0
PerfFrame.Parent = Frame

local PerfLabel = Instance.new("TextLabel")
PerfLabel.Size = UDim2.new(1,0,1,0)
PerfLabel.BackgroundTransparency = 1
PerfLabel.Font = Enum.Font.GothamSemibold
PerfLabel.TextSize = 16
PerfLabel.TextColor3 = Color3.fromRGB(180, 200, 255)
PerfLabel.TextStrokeTransparency = 0.85
PerfLabel.Text = "Ping: ... | FPS: ..."
PerfLabel.Parent = PerfFrame

-- Atualização de Ping e FPS
local frameCounter, lastTime, fps = 0, tick(), 60
spawn(function()
    while true do
        -- FPS
        frameCounter = frameCounter + 1
        if tick()-lastTime >= 1 then
            fps = frameCounter
            frameCounter = 0
            lastTime = tick()
        end
        -- Ping
        local ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
        PerfLabel.Text = ("Ping: %d ms | FPS: %d"):format(ping, fps)
        task.wait(0.2)
    end
end)
RunService.RenderStepped:Connect(function() frameCounter = frameCounter + 1 end)

-- PLAYER COUNTER GUI
local PlayerFrame = Instance.new("Frame")
PlayerFrame.Size = UDim2.new(0.8, 0, 0, 34)
PlayerFrame.Position = UDim2.new(0.1, 0, 0.80, 0) -- abaixo do PerfFrame
PlayerFrame.BackgroundTransparency = 0.6
PlayerFrame.BackgroundColor3 = Color3.fromRGB(24,24,28)
PlayerFrame.BorderSizePixel = 0
PlayerFrame.Parent = Frame

local PlayerLabel = Instance.new("TextLabel")
PlayerLabel.Size = UDim2.new(1,0,1,0)
PlayerLabel.BackgroundTransparency = 1
PlayerLabel.Font = Enum.Font.GothamSemibold
PlayerLabel.TextSize = 16
PlayerLabel.TextColor3 = Color3.fromRGB(200,255,200)
PlayerLabel.TextStrokeTransparency = 0.85
PlayerLabel.Text = "Players: ..."
PlayerLabel.Parent = PlayerFrame

local function updatePlayerCount()
    PlayerLabel.Text = ("Players: %d"):format(#Players:GetPlayers())
end
-- atualiza agora e quando alguém entrar/sair
updatePlayerCount()
Players.PlayerAdded:Connect(updatePlayerCount)
Players.PlayerRemoving:Connect(updatePlayerCount)

-- Tooltip para instruções
local Tooltip = Instance.new("TextLabel")
Tooltip.Size = UDim2.new(0.96,0,0,18)
Tooltip.Position = UDim2.new(0.02,0,0.90,0)
Tooltip.BackgroundTransparency = 1
Tooltip.Text = "Arraste a janela segurando o topo"
Tooltip.Font = Enum.Font.Gotham
Tooltip.TextSize = 14
Tooltip.TextColor3 = Color3.fromRGB(180,180,180)
Tooltip.TextStrokeTransparency = 0.9
Tooltip.Parent = Frame

-- Créditos
local Credits = Instance.new("TextLabel")
Credits.Size = UDim2.new(1,0,0,18)
Credits.Position = UDim2.new(0,0,0.95,0)
Credits.BackgroundTransparency = 1
Credits.Text = "Mozart Scripts"
Credits.Font = Enum.Font.GothamBold
Credits.TextSize = 15
Credits.TextColor3 = Color3.fromRGB(255,0,0)
Credits.TextStrokeTransparency = 0.8
Credits.Parent = Frame

spawn(function()
    local hue = 0
    while Credits and Credits.Parent do
        Credits.TextColor3 = Color3.fromHSV(hue, 0.75, 1)
        hue = (hue + 0.01) % 1
        task.wait(0.05)
    end
end)

-- MELHOR SISTEMA DE ARRASTAR: Só arrasta segurando o topo (Title)
Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragOffset = Vector2.new(input.Position.X - Frame.AbsolutePosition.X, input.Position.Y - Frame.AbsolutePosition.Y)
    end
end)

Title.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local newX = input.Position.X - DragOffset.X
        local newY = input.Position.Y - DragOffset.Y
        Frame.Position = UDim2.new(0, math.clamp(newX, 0, workspace.CurrentCamera.ViewportSize.X-Frame.Size.X.Offset), 0, math.clamp(newY, 0, workspace.CurrentCamera.ViewportSize.Y-Frame.Size.Y.Offset))
    end
end)
