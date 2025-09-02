--== Serviços ==--
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

--== Variáveis ==--
local EquipLoop = false
local Dragging = false
local DragOffset = Vector2.new(0,0)
local AutoLoad = true -- executa automaticamente ao entrar no mapa

--== Função para remover roupas, acessórios e reduzir o tamanho ==--
local function stripCharacter()
    local character = LocalPlayer.Character
    if not character then return end

    -- Remove acessórios
    for _, child in pairs(character:GetChildren()) do
        if child:IsA("Accessory") or child:IsA("Shirt") or child:IsA("Pants") then
            child:Destroy()
        end
    end

    -- Diminuir tamanho do personagem
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.BodyHeightScale.Value = 0.7
        humanoid.BodyWidthScale.Value = 0.7
        humanoid.BodyDepthScale.Value = 0.7
        humanoid.HeadScale.Value = 0.7
    end
end

--== Loop para manter uma Tool equipada sem sumir ==--
local function toggleToolLoop(tool)
    EquipLoop = true
    if not tool then return end

    while EquipLoop do
        if tool.Parent ~= LocalPlayer.Character then
            tool.Parent = LocalPlayer.Character
        end
        task.wait(0.05) -- rápido, mas estável visualmente
    end
end

--== Função principal ==--
local function runAutoEquip()
    local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    stripCharacter() -- remove roupas, acessórios e diminui
    repeat task.wait() until character:FindFirstChild("HumanoidRootPart")

    -- Pega a Tool ativa ou a primeira Tool do Backpack
    local tool = character:FindFirstChildWhichIsA("Tool")
    if not tool then
        tool = LocalPlayer.Backpack:FindFirstChildWhichIsA("Tool")
    end
    if tool then
        toggleToolLoop(tool)
    else
        warn("Nenhuma Tool encontrada!")
    end
end

--== GUI ==--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false

local success, hui = pcall(function() return gethui() end)
if success and hui then
    ScreenGui.Parent = hui
else
    ScreenGui.Parent = game:GetService("CoreGui")
end

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 260, 0, 180)
Frame.Position = UDim2.new(0, 50, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,30)
Title.BackgroundTransparency = 1
Title.Text = "AutoEquip Tool"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.Parent = Frame

-- Botões
local ActivateButton = Instance.new("TextButton")
ActivateButton.Size = UDim2.new(0.8,0,0,40)
ActivateButton.Position = UDim2.new(0.1,0,0.3,0)
ActivateButton.BackgroundColor3 = Color3.fromRGB(50,200,100)
ActivateButton.Font = Enum.Font.GothamBold
ActivateButton.TextSize = 16
ActivateButton.TextColor3 = Color3.fromRGB(255,255,255)
ActivateButton.Text = "Ativar"
ActivateButton.Parent = Frame
ActivateButton.MouseButton1Click:Connect(runAutoEquip)

local DeactivateButton = Instance.new("TextButton")
DeactivateButton.Size = UDim2.new(0.8,0,0,40)
DeactivateButton.Position = UDim2.new(0.1,0,0.65,0)
DeactivateButton.BackgroundColor3 = Color3.fromRGB(200,50,50)
DeactivateButton.Font = Enum.Font.GothamBold
DeactivateButton.TextSize = 16
DeactivateButton.TextColor3 = Color3.fromRGB(255,255,255)
DeactivateButton.Text = "Desativar"
DeactivateButton.Parent = Frame
DeactivateButton.MouseButton1Click:Connect(function() EquipLoop = false end)

-- Arrastar GUI segurando qualquer lugar
Frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragOffset = input.Position - Frame.AbsolutePosition
    end
end)

Frame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        Frame.Position = UDim2.new(0, input.Position.X - DragOffset.X, 0, input.Position.Y - DragOffset.Y)
    end
end)

-- Auto Load
if AutoLoad then
    task.defer(function()
        if LocalPlayer.Character then
            runAutoEquip()
        end
    end)
    LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1)
        runAutoEquip()
    end)
end

