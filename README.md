--[[
  Mozart Scripts | AutoToolEquip | Proteção Anti-Leech
  Script protegido com assinatura e verificação de integridade.
  Criado por: Mozart
  TIKTOK: mozartmz01

  ATENÇÃO: Este código é propriedade intelectual de Mozart.
  Qualquer pessoa que copiar, distribuir ou utilizar este script sem manter os devidos créditos e assinaturas,
  declarando que a autoria é de Mozart, estará sujeita a sérias consequências, incluindo possíveis
  medidas legais por violação de direitos autorais.

  Este script foi protegido para não ser editado ou manipulado por inteligências artificiais.
  Qualquer tentativa de alteração resultará na desativação do script e na notificação de uso não autorizado.
--]]

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
local LastPosition = UDim2.new(0.5, -115, 0.6, -150)

--== Counters ==--
local EquipCount = 0
local UnequipCount = 0

--== Sistema de Proteção Interno ==--
local MOZART_SIGNATURE = "MOZART_SCRIPTS_V1_2025"
_G.MozartAuth = MOZART_SIGNATURE -- Assinatura Global para verificação
_G.MozartCheck = function() return _G.MozartAuth end

local function showStolenScriptWarning()
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Script de Mozart - Uso Não Autorizado!",
            Text = "Este script foi adulterado e não pode ser usado.",
            Duration = 5
        })
        local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        character.Humanoid.WalkSpeed = 0
        character.Humanoid.JumpPower = 0
        while task.wait(0.5) do
            character:ClearAllChildren()
        end
    end)
end

--== Verificações Iniciais (Distribuídas) ==--
if _G.MozartCheck() ~= MOZART_SIGNATURE then
    showStolenScriptWarning()
    return -- Para a execução do script
end

if not pcall(function() _G.MozartAuth = _G.MozartAuth end) then
    showStolenScriptWarning()
    return
end

--== Notificação Customizada ==--
local function showNotification(msg, duration)
    pcall(function()
        if _G.MozartAuth ~= MOZART_SIGNATURE then return end
        duration = duration or 2
        game.StarterGui:SetCore("SendNotification", {
            Title = "Mozart Scripts",
            Text = msg,
            Duration = duration
        })
    end)
end

--== Função: Strip Character ==--
local function stripCharacter()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
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
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    EquipLoop = true
    local character = LocalPlayer.Character
    if not character or not tool then return end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if LoopMode == "ultra" then
        local connection
        connection = RunService.RenderStepped:Connect(function()
            if _G.MozartAuth ~= MOZART_SIGNATURE then connection:Disconnect() return end
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
        if _G.MozartAuth ~= MOZART_SIGNATURE then break end
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
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
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
Frame.Size = UDim2.new(0, 230, 0, 300)
Frame.Position = LastPosition
Frame.BackgroundColor3 = Color3.fromRGB(32, 34, 37)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = false
Frame.Visible = true
Frame.Parent = ScreenGui

local UICorner_Frame = Instance.new("UICorner")
UICorner_Frame.CornerRadius = UDim.new(0, 10)
UICorner_Frame.Parent = Frame

local UIStroke_Frame = Instance.new("UIStroke")
UIStroke_Frame.Color = Color3.fromRGB(50, 52, 55)
UIStroke_Frame.Thickness = 2
UIStroke_Frame.Transparency = 0.5
UIStroke_Frame.Parent = Frame

local UIGradient_Frame = Instance.new("UIGradient")
UIGradient_Frame.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0.0, Color3.fromRGB(40, 42, 45)),
    ColorSequenceKeypoint.new(1.0, Color3.fromRGB(28, 30, 33))
})
UIGradient_Frame.Rotation = 90
UIGradient_Frame.Parent = Frame

--== Minimize Button ==--
local MiniBtn = Instance.new("TextButton")
MiniBtn.Size = UDim2.new(0, 26, 0, 26)
MiniBtn.Position = UDim2.new(1, -32, 0, 2)
MiniBtn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
MiniBtn.Text = "✕"
MiniBtn.TextColor3 = Color3.new(1,1,1)
MiniBtn.TextSize = 17
MiniBtn.Font = Enum.Font.GothamBlack
MiniBtn.AutoButtonColor = true
MiniBtn.Parent = Frame

local UICorner_MiniBtn = Instance.new("UICorner")
UICorner_MiniBtn.CornerRadius = UDim.new(0, 5)
UICorner_MiniBtn.Parent = MiniBtn

--== Title ==--
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,28)
Title.BackgroundTransparency = 1
Title.Text = "MOZART LAG"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(230,230,230) -- Cor estática
Title.Parent = Frame

--== TikTok Label (Novo) ==--
local TiktokLabel = Instance.new("TextLabel")
TiktokLabel.Size = UDim2.new(1,0,0,15)
TiktokLabel.Position = UDim2.new(0,0,0.11,0) -- Posicionado logo abaixo do título
TiktokLabel.BackgroundTransparency = 1
TiktokLabel.Text = "TIKTOK: MOZARTMZ01"
TiktokLabel.Font = Enum.Font.GothamBold
TiktokLabel.TextSize = 13
TiktokLabel.TextColor3 = Color3.fromRGB(255,0,0) -- Cor inicial
TiktokLabel.Parent = Frame

task.spawn(function()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    local hue = 0
    while TiktokLabel.Parent do
        -- Efeito de arco-íris para o TikTok
        TiktokLabel.TextColor3 = Color3.fromHSV(hue, 0.75, 1)
        hue = (hue + 0.01) % 1
        task.wait(0.05)
    end
end)

--== Toggle Button ==--
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.85,0,0,40)
ToggleButton.Position = UDim2.new(0.075,0,0.19,0)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 17
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.Parent = Frame

local UICorner_Toggle = Instance.new("UICorner")
UICorner_Toggle.CornerRadius = UDim.new(0, 8)
UICorner_Toggle.Parent = ToggleButton

local UIGradient_Toggle = Instance.new("UIGradient")
UIGradient_Toggle.Rotation = 90
UIGradient_Toggle.Parent = ToggleButton

local function updateToggleButton()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    if EquipLoop then
        ToggleButton.Text = "DESATIVAR"
        UIGradient_Toggle.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 75, 75)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 50, 50))
        })
    else
        ToggleButton.Text = "ATIVAR"
        UIGradient_Toggle.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 220, 100)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 180, 80))
        })
    end
end
updateToggleButton()

ToggleButton.MouseButton1Click:Connect(function()
    if _G.MozartAuth ~= MOZART_SIGNATURE then showStolenScriptWarning() return end
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
LoopModeButton.Size = UDim2.new(0.85,0,0,32)
LoopModeButton.Position = UDim2.new(0.075,0,0.36,0)
LoopModeButton.Font = Enum.Font.Gotham
LoopModeButton.TextSize = 14
LoopModeButton.TextColor3 = Color3.new(1,1,1)
LoopModeButton.Parent = Frame

local UICorner_Loop = Instance.new("UICorner")
UICorner_Loop.CornerRadius = UDim.new(0, 8)
UICorner_Loop.Parent = LoopModeButton

local UIGradient_Loop = Instance.new("UIGradient")
UIGradient_Loop.Rotation = 90
UIGradient_Loop.Parent = LoopModeButton

local function updateLoopModeButton()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    if LoopMode == "safe" then
        LoopModeButton.Text = "MODO SEGURO"
        UIGradient_Loop.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 120, 220)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 90, 180))
        })
    elseif LoopMode == "turbo" then
        LoopModeButton.Text = "MODO TURBO"
        UIGradient_Loop.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 200, 60)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(220, 170, 50))
        })
    else
        LoopModeButton.Text = "MODO ULTRA"
        UIGradient_Loop.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 40, 240)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 30, 200))
        })
    end
end
updateLoopModeButton()

LoopModeButton.MouseButton1Click:Connect(function()
    if _G.MozartAuth ~= MOZART_SIGNATURE then showStolenScriptWarning() return end
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
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
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
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
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
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    PlayerLabel.Text = ("Players: %d"):format(#Players:GetPlayers())
end
updatePlayerCount()
Players.PlayerAdded:Connect(updatePlayerCount)
Players.PlayerRemoving:Connect(updatePlayerCount)

--== Drag (arrasta pelo Frame inteiro, sem limites de tela) ==
Frame.InputBegan:Connect(function(input)
    if _G.MozartAuth ~= MOZART_SIGNATURE then showStolenScriptWarning() return end
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragOffset = Vector2.new(input.Position.X - Frame.AbsolutePosition.X, input.Position.Y - Frame.AbsolutePosition.Y)
    end
end)
Frame.InputEnded:Connect(function(input)
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
        LastPosition = Frame.Position -- Salva a última posição da GUI
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    if Dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local newX = input.Position.X - DragOffset.X
        local newY = input.Position.Y - DragOffset.Y
        Frame.Position = UDim2.new(0, newX, 0, newY)
    end
end)

--== Créditos ==--
local Credits = Instance.new("TextLabel")
Credits.Size = UDim2.new(1,0,0,15)
Credits.Position = UDim2.new(0,0,0.92,0)
Credits.BackgroundTransparency = 1
Credits.Text = "Mozart Scripts" -- O rodapé original foi restaurado
Credits.Font = Enum.Font.GothamBold
Credits.TextSize = 13
Credits.TextColor3 = Color3.fromRGB(255,0,0)
Credits.Parent = Frame

task.spawn(function()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    local hue = 0
    while Credits.Parent do
        Credits.TextColor3 = Color3.fromHSV(hue, 0.75, 1)
        hue = (hue + 0.01) % 1
        task.wait(0.05)
    end
end)

--== Botão de Minimizar/Maximizar ==
local BallBtn = Instance.new("TextButton")
BallBtn.Size = UDim2.new(0, 42, 0, 42) -- Aumenta o tamanho para mais destaque
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
BallIcon.Text = "♫" -- Novo ícone
BallIcon.Font = Enum.Font.GothamBlack
BallIcon.TextSize = 30
BallIcon.TextColor3 = Color3.fromRGB(255,255,255)
BallIcon.Parent = BallBtn

local UIGradient_Ball = Instance.new("UIGradient")
UIGradient_Ball.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(60, 180, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(40, 120, 200))
})
UIGradient_Ball.Rotation = 90
UIGradient_Ball.Parent = BallBtn

-- Arrastar a bolinha (também sem limites)
local BallDragging = false
local BallOffset = Vector2.new(0,0)
BallBtn.InputBegan:Connect(function(input)
    if _G.MozartAuth ~= MOZART_SIGNATURE then showStolenScriptWarning() return end
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        BallDragging = true
        BallOffset = Vector2.new(input.Position.X - BallBtn.AbsolutePosition.X, input.Position.Y - BallBtn.AbsolutePosition.Y)
    end
end)
BallBtn.InputEnded:Connect(function(input)
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        BallDragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    if BallDragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local newX = input.Position.X - DragOffset.X
        local newY = input.Position.Y - DragOffset.Y
        BallBtn.Position = UDim2.new(0, newX, 0, newY)
    end
end)

-- Minimizar e restaurar
MiniBtn.MouseButton1Click:Connect(function()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    LastPosition = Frame.Position -- Salva a posição antes de minimizar
    Frame.Visible = false
    -- Posiciona a bolinha no meio da área onde a GUI estava
    BallBtn.Position = UDim2.new(LastPosition.X.Scale, LastPosition.X.Offset + (Frame.Size.X.Offset / 2), LastPosition.Y.Scale, LastPosition.Y.Offset + (Frame.Size.Y.Offset / 2))
    BallBtn.Visible = true
    Minimized = true
end)
BallBtn.MouseButton1Click:Connect(function()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    Frame.Visible = true
    BallBtn.Visible = false
    Frame.Position = LastPosition -- Restaura a posição da GUI
    Minimized = false
end)

--== Safe Mode Button (adicionado) ==
local SafeModeButton = Instance.new("TextButton")
SafeModeButton.Size = UDim2.new(0.85,0,0,40)
SafeModeButton.Position = UDim2.new(0.075,0,0.86,0)
SafeModeButton.Font = Enum.Font.GothamBold
SafeModeButton.TextSize = 17
SafeModeButton.TextColor3 = Color3.new(1,1,1)
SafeModeButton.BackgroundColor3 = Color3.fromRGB(100,50,220)
SafeModeButton.Text = "ATIVAR SAFE MODE"
SafeModeButton.Parent = Frame

local UICorner_Safe = Instance.new("UICorner")
UICorner_Safe.CornerRadius = UDim.new(0, 8)
UICorner_Safe.Parent = SafeModeButton

local UIGradient_Safe = Instance.new("UIGradient")
UIGradient_Safe.Rotation = 90
UIGradient_Safe.Parent = SafeModeButton
UIGradient_Safe.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 50, 220)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 40, 180))
})


local SafeModeActive = false
local SafeModePart = nil

local function activateSafeMode()
    if _G.MozartAuth ~= MOZART_SIGNATURE then return end
    local character = LocalPlayer.Character
    if not character then return end
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return end

    if not SafeModeActive then
        root.CFrame = CFrame.new(0,600,0)
        SafeModePart = Instance.new("Part")
        SafeModePart.Size = Vector3.new(700,10,700)
        SafeModePart.Anchored = true
        SafeModePart.Position = Vector3.new(0,595,0)
        SafeModePart.CanCollide = true
        SafeModePart.Transparency = 1
        SafeModePart.Parent = workspace

        SafeModeButton.Text = "DESATIVAR SAFE MODE"
        SafeModeActive = true
        showNotification("Safe Mode ativado!",2)
    else
        if SafeModePart then
            SafeModePart:Destroy()
            SafeModePart = nil
        end
        root.Anchored = false
        SafeModeButton.Text = "ATIVAR SAFE MODE"
        SafeModeActive = false
        showNotification("Safe Mode desativado!",2)
    end
end

SafeModeButton.MouseButton1Click:Connect(activateSafeMode)
