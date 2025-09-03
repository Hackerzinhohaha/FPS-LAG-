--== Serviços ==--
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local LocalPlayer = Players.LocalPlayer

--== Multi-Idioma: EN/BR ==--
local Language = nil
local Translation = {
    BR = {
        TITLE = "🌟 Equipe a Tool",
        ACTIVATE = "Ativar",
        DEACTIVATE = "Desativar",
        SAFE = "Modo Seguro (Recomendado)",
        TURBO = "Modo Turbo (Rápido)",
        ULTRA = "Modo ULTRA (Máxima velocidade!)",
        ULTRALAG = "ULTRA LAG (TODAS AS SUAS TOOLS)",
        STOP_ULTRALAG = "PARAR ULTRA LAG",
        TOOLTIP = "Arraste a janela segurando o topo",
        COUNTER = "Equip: %d   |   Desequip: %d",
        CREDITS = "Mozart Scripts",
        PINGFPS = "Ping: %d ms | FPS: %d",
        NOTIF_NEED_TOOL = "Equipe a Tool para funcionar!",
        NOTIF_TURBO = "Modo turbo ativado!\nMuito rápido.",
        NOTIF_ULTRA = "Modo ULTRA ativado!\nO mais rápido possível!",
        NOTIF_SAFE = "Modo seguro ativado!",
        NOTIF_ULTRALAG_ON = "ULTRA LAG ATIVADO!\nEquipando/desquipando todas as suas Tools!",
        NOTIF_ULTRALAG_OFF = "ULTRA LAG DESATIVADO!",
        BTN_US = "🇺🇸 Estados Unidos",
        BTN_BR = "🇧🇷 Brasil",
        CLOSE = "Fechar",
        OPEN = "Abrir GUI"
    },
    US = {
        TITLE = "🌟 Equip the Tool",
        ACTIVATE = "Activate",
        DEACTIVATE = "Deactivate",
        SAFE = "Safe Mode (Recommended)",
        TURBO = "Turbo Mode (Fast)",
        ULTRA = "ULTRA Mode (Maximum speed!)",
        ULTRALAG = "ULTRA LAG (ALL YOUR TOOLS)",
        STOP_ULTRALAG = "STOP ULTRA LAG",
        TOOLTIP = "Drag the window by the top bar",
        COUNTER = "Equip: %d   |   Unequip: %d",
        CREDITS = "Mozart Scripts",
        PINGFPS = "Ping: %d ms | FPS: %d",
        NOTIF_NEED_TOOL = "Equip a Tool to make it work!",
        NOTIF_TURBO = "Turbo mode enabled!\nVery fast.",
        NOTIF_ULTRA = "ULTRA mode enabled!\nMaximum speed!",
        NOTIF_SAFE = "Safe mode enabled!",
        NOTIF_ULTRALAG_ON = "ULTRA LAG ENABLED!\nEquipping/unequipping all your Tools!",
        NOTIF_ULTRALAG_OFF = "ULTRA LAG DISABLED!",
        BTN_US = "🇺🇸 United States",
        BTN_BR = "🇧🇷 Brazil",
        CLOSE = "Close",
        OPEN = "Open GUI"
    }
}
local function T(msg) return Language and Translation[Language][msg] or msg end

--== Variáveis ==--
local EquipLoop = false
local UltraLagLoop = false
local Dragging = false
local DragOffset = Vector2.new(0,0)
local LoopMode = "safe"
local EquipCount = 0
local UnequipCount = 0

--== Notificação Customizada ==--
local function showNotification(msg, duration)
    duration = duration or 2
    local notifFrame = Instance.new("Frame")
    notifFrame.Size = UDim2.new(0, 320, 0, 74)
    notifFrame.Position = UDim2.new(1, -350, 0, 36)
    notifFrame.BackgroundColor3 = Color3.fromRGB(30, 34, 40)
    notifFrame.BackgroundTransparency = 0.1
    notifFrame.BorderSizePixel = 0
    notifFrame.AnchorPoint = Vector2.new(0,0)
    notifFrame.ZIndex = 999
    notifFrame.ClipsDescendants = true
    notifFrame.Parent = game:GetService("CoreGui"):FindFirstChildOfClass("ScreenGui") or game:GetService("CoreGui")

    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0,36,0,36)
    icon.Position = UDim2.new(0,15,0,19)
    icon.BackgroundTransparency = 1
    icon.Image = "rbxassetid://7733960981"
    icon.ZIndex = 1000
    icon.Parent = notifFrame

    local notifText = Instance.new("TextLabel")
    notifText.BackgroundTransparency = 1
    notifText.Position = UDim2.new(0, 65, 0, 11)
    notifText.Size = UDim2.new(1, -80, 1, -18)
    notifText.TextColor3 = Color3.fromRGB(255,255,255)
    notifText.Font = Enum.Font.GothamBold
    notifText.TextSize = 20
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
        notifFrame.BackgroundTransparency = 1 - 0.09*i
        notifText.TextTransparency = 1 - 0.09*i
        icon.ImageTransparency = 1 - 0.09*i
        notifFrame.Position = UDim2.new(1, -350 + 4*(10-i), 0, 36)
        wait(0.016)
    end
    notifFrame.BackgroundTransparency = 0.1
    notifText.TextTransparency = 0
    icon.ImageTransparency = 0
    notifFrame.Position = UDim2.new(1, -350, 0, 36)

    local notifSound = Instance.new("Sound")
    notifSound.SoundId = "rbxassetid://6026984224"
    notifSound.Volume = 0.5
    notifSound.Parent = notifFrame
    notifSound:Play()
    game:GetService("Debris"):AddItem(notifSound, 2.4)

    wait(duration)
    for i=1,10 do
        notifFrame.BackgroundTransparency = 0.1 + 0.09*i
        notifText.TextTransparency = 0 + 0.09*i
        icon.ImageTransparency = 0 + 0.09*i
        notifFrame.Position = UDim2.new(1, -350 + 2*i, 0, 36)
        wait(0.016)
    end
    notifFrame:Destroy()
end

--== Funções principais ==--
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
            if not EquipLoop then if connection then connection:Disconnect() end return end
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

local function ultraLagLoop()
    UltraLagLoop = true
    showNotification(T("NOTIF_ULTRALAG_ON"),2.5)
    local player = Players.LocalPlayer
    local connection
    connection = RunService.RenderStepped:Connect(function()
        if not UltraLagLoop then if connection then connection:Disconnect() end showNotification(T("NOTIF_ULTRALAG_OFF"),2) return end
        local tools = {}
        for _,tool in ipairs(player.Backpack:GetChildren()) do if tool:IsA("Tool") then table.insert(tools, tool) end end
        if player.Character then for _,tool in ipairs(player.Character:GetChildren()) do if tool:IsA("Tool") then table.insert(tools, tool) end end end
        for _,t in ipairs(workspace:GetChildren()) do if t:IsA("Tool") and t:FindFirstChild("Handle") then pcall(function() t.Parent = player.Backpack end) end end
        if player.Character then
            local hum = player.Character:FindFirstChildWhichIsA("Humanoid")
            if hum then
                for _, tool in ipairs(tools) do
                    if tool and tool.Parent == player.Backpack then hum:EquipTool(tool) end
                end
            end
        end
    end)
end

local function runAutoEquip()
    local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    stripCharacter()
    repeat task.wait() until character:FindFirstChild("HumanoidRootPart")
    EquipCount = 0
    UnequipCount = 0
    _G.UpdateCounter()
    local tool = character:FindFirstChildWhichIsA("Tool")
    if not tool then showNotification(T("NOTIF_NEED_TOOL"), 2.2) EquipLoop = false return end
    spawn(function() toggleToolLoop(tool) end)
end

--== GUI ==--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoToolEquipMozart"
ScreenGui.ResetOnSpawn = false
local success, hui = pcall(function() return gethui() end)
if success and hui then ScreenGui.Parent = hui else ScreenGui.Parent = game:GetService("CoreGui") end

--== Seletor de idioma inicial ==--
local LangSelectFrame = Instance.new("Frame")
LangSelectFrame.Size = UDim2.new(0, 340, 0, 170)
LangSelectFrame.Position = UDim2.new(0.5, -170, 0.32, 0)
LangSelectFrame.AnchorPoint = Vector2.new(0.5,0)
LangSelectFrame.BackgroundColor3 = Color3.fromRGB(33,36,43)
LangSelectFrame.BorderSizePixel = 0
LangSelectFrame.BackgroundTransparency = 0.05
LangSelectFrame.Parent = ScreenGui

local LangShadow = Instance.new("ImageLabel")
LangShadow.Size = UDim2.new(1, 32, 1, 32)
LangShadow.Position = UDim2.new(0, -16, 0, -16)
LangShadow.BackgroundTransparency = 1
LangShadow.Image = "rbxassetid://1316045217"
LangShadow.ImageTransparency = 0.42
LangShadow.ZIndex = LangSelectFrame.ZIndex - 1
LangShadow.Parent = LangSelectFrame

local LangTitle = Instance.new("TextLabel")
LangTitle.Size = UDim2.new(1,0,0,50)
LangTitle.Position = UDim2.new(0,0,0,10)
LangTitle.BackgroundTransparency = 1
LangTitle.Text = "🌎 Choose your language\n🌎 Escolha seu idioma"
LangTitle.Font = Enum.Font.GothamBold
LangTitle.TextSize = 20
LangTitle.TextColor3 = Color3.fromRGB(240,240,240)
LangTitle.TextStrokeTransparency = 0.77
LangTitle.TextWrapped = true
LangTitle.Parent = LangSelectFrame

local BtnBR = Instance.new("TextButton")
BtnBR.Size = UDim2.new(0.44, 0, 0, 48)
BtnBR.Position = UDim2.new(0.06, 0, 0.55, 0)
BtnBR.Text = Translation.BR.BTN_BR
BtnBR.Font = Enum.Font.GothamBlack
BtnBR.TextSize = 18
BtnBR.TextColor3 = Color3.fromRGB(255,255,255)
BtnBR.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
BtnBR.BorderSizePixel = 0
BtnBR.AutoButtonColor = true
BtnBR.Parent = LangSelectFrame

local BtnUS = Instance.new("TextButton")
BtnUS.Size = UDim2.new(0.44, 0, 0, 48)
BtnUS.Position = UDim2.new(0.50, 0, 0.55, 0)
BtnUS.Text = Translation.US.BTN_US
BtnUS.Font = Enum.Font.GothamBlack
BtnUS.TextSize = 18
BtnUS.TextColor3 = Color3.fromRGB(255,255,255)
BtnUS.BackgroundColor3 = Color3.fromRGB(41, 128, 185)
BtnUS.BorderSizePixel = 0
BtnUS.AutoButtonColor = true
BtnUS.Parent = LangSelectFrame

--== Função para mostrar/ocultar a GUI principal ==--
local MainFrame = nil
local OpenButton = nil

local function showMainGUI(lang)
    Language = lang
    LangSelectFrame:Destroy()

    -- Frame principal (MENOR)
    MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 340, 0, 370)
    MainFrame.Position = UDim2.new(0.5, -170, 0.26, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(28, 29, 36)
    MainFrame.BorderSizePixel = 0
    MainFrame.AnchorPoint = Vector2.new(0.5,0)
    MainFrame.Parent = ScreenGui
    MainFrame.Active = true
    MainFrame.Draggable = false
    MainFrame.Visible = true

    local Shadow = Instance.new("ImageLabel")
    Shadow.Size = UDim2.new(1, 24, 1, 24)
    Shadow.Position = UDim2.new(0, -12, 0, -12)
    Shadow.BackgroundTransparency = 1
    Shadow.Image = "rbxassetid://1316045217"
    Shadow.ImageTransparency = 0.48
    Shadow.ZIndex = MainFrame.ZIndex - 1
    Shadow.Parent = MainFrame

    -- Botão de Fechar
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 32, 0, 32)
    CloseBtn.Position = UDim2.new(1, -38, 0, 8)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
    CloseBtn.Text = "✕"
    CloseBtn.TextColor3 = Color3.new(1,1,1)
    CloseBtn.TextSize = 22
    CloseBtn.Font = Enum.Font.GothamBlack
    CloseBtn.BorderSizePixel = 0
    CloseBtn.ZIndex = 2
    CloseBtn.AutoButtonColor = true
    CloseBtn.Parent = MainFrame

    CloseBtn.MouseButton1Click:Connect(function()
        MainFrame.Visible = false
        if OpenButton then
            OpenButton.Visible = true
        end
    end)

    -- Botão de abrir (fica no canto inferior esquerdo)
    if not OpenButton then
        OpenButton = Instance.new("TextButton")
        OpenButton.Size = UDim2.new(0, 135, 0, 38)
        OpenButton.Position = UDim2.new(0, 16, 1, -56)
        OpenButton.AnchorPoint = Vector2.new(0,0)
        OpenButton.BackgroundColor3 = Color3.fromRGB(41, 128, 185)
        OpenButton.Text = "🌟 " .. T("OPEN")
        OpenButton.TextColor3 = Color3.new(1,1,1)
        OpenButton.TextSize = 16
        OpenButton.Font = Enum.Font.GothamBold
        OpenButton.BorderSizePixel = 0
        OpenButton.Visible = false
        OpenButton.Parent = ScreenGui
        OpenButton.MouseButton1Click:Connect(function()
            MainFrame.Visible = true
            OpenButton.Visible = false
        end)
    end

    -- Topo
    local TopBar = Instance.new("Frame")
    TopBar.Size = UDim2.new(1,0,0,54)
    TopBar.Position = UDim2.new(0,0,0,0)
    TopBar.BackgroundTransparency = 1
    TopBar.Parent = MainFrame

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1,0,0,36)
    Title.Position = UDim2.new(0,0,0,4)
    Title.BackgroundTransparency = 1
    Title.Text = T("TITLE")
    Title.Font = Enum.Font.GothamBlack
    Title.TextSize = 20
    Title.TextColor3 = Color3.fromRGB(225, 255, 180)
    Title.TextStrokeTransparency = 0.76
    Title.TextWrapped = true
    Title.Parent = TopBar

    local Line = Instance.new("Frame")
    Line.Size = UDim2.new(0.9,0,0,2)
    Line.Position = UDim2.new(0.05,0,0,47)
    Line.BackgroundColor3 = Color3.fromRGB(57,200,110)
    Line.BorderSizePixel = 0
    Line.Parent = TopBar

    -- Botões principais
    local y = 0.19
    local y_space = 0.15

    local ToggleButton = Instance.new("TextButton")
    ToggleButton.Size = UDim2.new(0.82,0,0,38)
    ToggleButton.Position = UDim2.new(0.09,0,y,0)
    ToggleButton.Font = Enum.Font.GothamBlack
    ToggleButton.TextSize = 16
    ToggleButton.TextColor3 = Color3.fromRGB(255,255,255)
    ToggleButton.BorderSizePixel = 0
    ToggleButton.AutoButtonColor = false
    ToggleButton.Text = T("ACTIVATE")
    ToggleButton.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
    ToggleButton.Parent = MainFrame

    local function updateToggleButton()
        ToggleButton.Text = EquipLoop and T("DEACTIVATE") or T("ACTIVATE")
        ToggleButton.BackgroundColor3 = EquipLoop and Color3.fromRGB(192,57,43) or Color3.fromRGB(39, 174, 96)
    end
    updateToggleButton()

    ToggleButton.MouseEnter:Connect(function()
        if EquipLoop then
            ToggleButton.BackgroundColor3 = Color3.fromRGB(231,76,60)
        else
            ToggleButton.BackgroundColor3 = Color3.fromRGB(46,204,113)
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

    local LoopModeButton = Instance.new("TextButton")
    LoopModeButton.Size = UDim2.new(0.82,0,0,32)
    LoopModeButton.Position = UDim2.new(0.09,0,y + y_space,0)
    LoopModeButton.Font = Enum.Font.GothamBold
    LoopModeButton.TextSize = 14
    LoopModeButton.TextColor3 = Color3.fromRGB(255,255,255)
    LoopModeButton.BorderSizePixel = 0
    LoopModeButton.AutoButtonColor = false
    LoopModeButton.Parent = MainFrame

    local function updateLoopModeButton()
        if LoopMode == "safe" then
            LoopModeButton.Text = T("SAFE")
            LoopModeButton.BackgroundColor3 = Color3.fromRGB(52, 152, 219)
            LoopModeButton.TextColor3 = Color3.fromRGB(255,255,255)
        elseif LoopMode == "turbo" then
            LoopModeButton.Text = T("TURBO")
            LoopModeButton.BackgroundColor3 = Color3.fromRGB(241,196,15)
            LoopModeButton.TextColor3 = Color3.fromRGB(60,60,60)
        else
            LoopModeButton.Text = T("ULTRA")
            LoopModeButton.BackgroundColor3 = Color3.fromRGB(155,89,182)
            LoopModeButton.TextColor3 = Color3.fromRGB(255,255,255)
        end
    end
    updateLoopModeButton()

    LoopModeButton.MouseButton1Click:Connect(function()
        if LoopMode == "safe" then
            LoopMode = "turbo"
            showNotification(T("NOTIF_TURBO"),2.2)
        elseif LoopMode == "turbo" then
            LoopMode = "ultra"
            showNotification(T("NOTIF_ULTRA"),2.3)
        else
            LoopMode = "safe"
            showNotification(T("NOTIF_SAFE"),2)
        end
        updateLoopModeButton()
    end)
    LoopModeButton.MouseEnter:Connect(function()
        if LoopMode == "safe" then
            LoopModeButton.BackgroundColor3 = Color3.fromRGB(41,128,185)
        elseif LoopMode == "turbo" then
            LoopModeButton.BackgroundColor3 = Color3.fromRGB(243,220,75)
        else
            LoopModeButton.BackgroundColor3 = Color3.fromRGB(187, 102, 255)
        end
    end)
    LoopModeButton.MouseLeave:Connect(updateLoopModeButton)

    local UltraLagButton = Instance.new("TextButton")
    UltraLagButton.Size = UDim2.new(0.82,0,0,32)
    UltraLagButton.Position = UDim2.new(0.09,0,y + y_space*2 + 0.01,0)
    UltraLagButton.Font = Enum.Font.GothamBlack
    UltraLagButton.TextSize = 14
    UltraLagButton.Text = T("ULTRALAG")
    UltraLagButton.TextColor3 = Color3.fromRGB(255,255,255)
    UltraLagButton.BorderSizePixel = 0
    UltraLagButton.AutoButtonColor = false
    UltraLagButton.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
    UltraLagButton.Parent = MainFrame

    local function updateUltraLagButton()
        if UltraLagLoop then
            UltraLagButton.Text = T("STOP_ULTRALAG")
            UltraLagButton.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
        else
            UltraLagButton.Text = T("ULTRALAG")
            UltraLagButton.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
        end
    end
    updateUltraLagButton()

    UltraLagButton.MouseEnter:Connect(function()
        if UltraLagLoop then
            UltraLagButton.BackgroundColor3 = Color3.fromRGB(46,204,113)
        else
            UltraLagButton.BackgroundColor3 = Color3.fromRGB(192,57,43)
        end
    end)
    UltraLagButton.MouseLeave:Connect(updateUltraLagButton)

    UltraLagButton.MouseButton1Click:Connect(function()
        if UltraLagLoop then
            UltraLagLoop = false
            updateUltraLagButton()
        else
            UltraLagLoop = true
            updateUltraLagButton()
            spawn(function()
                ultraLagLoop()
            end)
        end
    end)

    -- COUNTER GUI
    local CounterFrame = Instance.new("Frame")
    CounterFrame.Size = UDim2.new(0.82,0,0,32)
    CounterFrame.Position = UDim2.new(0.09,0,0.62,0)
    CounterFrame.BackgroundTransparency = 0.35
    CounterFrame.BackgroundColor3 = Color3.fromRGB(40, 44, 54)
    CounterFrame.BorderSizePixel = 0
    CounterFrame.Parent = MainFrame

    local CounterLabel = Instance.new("TextLabel")
    CounterLabel.Size = UDim2.new(1,0,1,0)
    CounterLabel.BackgroundTransparency = 1
    CounterLabel.Font = Enum.Font.GothamSemibold
    CounterLabel.TextSize = 14
    CounterLabel.TextColor3 = Color3.fromRGB(180, 230, 170)
    CounterLabel.TextStrokeTransparency = 0.85
    CounterLabel.Text = T("COUNTER"):format(0, 0)
    CounterLabel.Parent = CounterFrame

    function _G.UpdateCounter()
        CounterLabel.Text = T("COUNTER"):format(EquipCount, UnequipCount)
    end

    local PerfFrame = Instance.new("Frame")
    PerfFrame.Size = UDim2.new(0.82,0,0,26)
    PerfFrame.Position = UDim2.new(0.09,0,0.72,0)
    PerfFrame.BackgroundTransparency = 0.5
    PerfFrame.BackgroundColor3 = Color3.fromRGB(33,39,48)
    PerfFrame.BorderSizePixel = 0
    PerfFrame.Parent = MainFrame

    local PerfLabel = Instance.new("TextLabel")
    PerfLabel.Size = UDim2.new(1,0,1,0)
    PerfLabel.BackgroundTransparency = 1
    PerfLabel.Font = Enum.Font.GothamSemibold
    PerfLabel.TextSize = 13
    PerfLabel.TextColor3 = Color3.fromRGB(155, 200, 255)
    PerfLabel.TextStrokeTransparency = 0.85
    PerfLabel.Text = T("PINGFPS"):format(0, 0)
    PerfLabel.Parent = PerfFrame

    local frameCounter, lastTime, fps = 0, tick(), 60
    spawn(function()
        while PerfLabel and PerfLabel.Parent do
            frameCounter = frameCounter + 1
            if tick()-lastTime >= 1 then
                fps = frameCounter
                frameCounter = 0
                lastTime = tick()
            end
            local ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            PerfLabel.Text = T("PINGFPS"):format(ping, fps)
            task.wait(0.2)
        end
    end)
    RunService.RenderStepped:Connect(function() frameCounter = frameCounter + 1 end)

    -- Tooltip
    local Tooltip = Instance.new("TextLabel")
    Tooltip.Size = UDim2.new(1,0,0,18)
    Tooltip.Position = UDim2.new(0,0,0.91,0)
    Tooltip.BackgroundTransparency = 1
    Tooltip.Text = T("TOOLTIP")
    Tooltip.Font = Enum.Font.Gotham
    Tooltip.TextSize = 13
    Tooltip.TextColor3 = Color3.fromRGB(180,255,180)
    Tooltip.TextStrokeTransparency = 0.92
    Tooltip.Parent = MainFrame

    -- Créditos
    local Credits = Instance.new("TextLabel")
    Credits.Size = UDim2.new(1,0,0,15)
    Credits.Position = UDim2.new(0,0,0.96,0)
    Credits.BackgroundTransparency = 1
    Credits.Text = T("CREDITS")
    Credits.Font = Enum.Font.GothamBold
    Credits.TextSize = 13
    Credits.TextColor3 = Color3.fromRGB(255,0,0)
    Credits.TextStrokeTransparency = 0.8
    Credits.Parent = MainFrame
    spawn(function()
        local hue = 0
        while Credits and Credits.Parent do
            Credits.TextColor3 = Color3.fromHSV(hue, 0.75, 1)
            hue = (hue + 0.01) % 1
            task.wait(0.05)
        end
    end)

    -- Drag pelo topo
    Title.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Dragging = true
            DragOffset = Vector2.new(input.Position.X - MainFrame.AbsolutePosition.X, input.Position.Y - MainFrame.AbsolutePosition.Y)
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
            MainFrame.Position = UDim2.new(0, math.clamp(newX, 0, workspace.CurrentCamera.ViewportSize.X-MainFrame.Size.X.Offset), 0, math.clamp(newY, 0, workspace.CurrentCamera.ViewportSize.Y-MainFrame.Size.Y.Offset))
        end
    end)
end

BtnBR.MouseButton1Click:Connect(function() showMainGUI("BR") end)
BtnUS.MouseButton1Click:Connect(function() showMainGUI("US") end)
