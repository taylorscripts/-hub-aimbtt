-- Carregar OrionLib (lib de interface)
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

-- Janela principal
local Window = OrionLib:MakeWindow({
    Name = "Aimbot by ChatGPT",
    HidePremium = false,
    SaveConfig = false,
    ConfigFolder = "AimbotConfig",
    IntroEnabled = false
})

-- Variáveis do Aimbot
local players = game:GetService("Players")
local camera = workspace.CurrentCamera
local localPlayer = players.LocalPlayer
local runService = game:GetService("RunService")
local fovCircle = Instance.new("Frame") -- Frame para desenhar o círculo de FOV
local screenSize = camera.ViewportSize

local aimbotActive = false
local fov = 150
local smoothing = 0.2
local aimlockActive = false

-- Função do Aimbot
local function getClosestEnemy()
    local closestEnemy = nil
    local shortestDistance = fov

    for i, v in pairs(players:GetPlayers()) do
        if v ~= localPlayer and v.Team ~= localPlayer.Team and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local pos, onScreen = camera:WorldToViewportPoint(v.Character.HumanoidRootPart.Position)
            if onScreen then
                local distance = (Vector2.new(pos.X, pos.Y) - Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)).magnitude
                if distance < shortestDistance then
                    closestEnemy = v
                    shortestDistance = distance
                end
            end
        end
    end

    return closestEnemy
end

-- Loop do Aimbot
runService.RenderStepped:Connect(function()
    if aimbotActive then
        local target = getClosestEnemy()

        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = camera:WorldToViewportPoint(target.Character.HumanoidRootPart.Position)
            local mousePos = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
            local aimPos = Vector2.new(targetPos.X, targetPos.Y)
            local move = (aimPos - mousePos) * smoothing

            if aimlockActive then
                -- Se o Aimlock estiver ativado, "gruda" no inimigo e o segue
                mousemoverel(move.X, move.Y)
            end
        end
    end
end)

-- Função para desenhar o círculo de FOV
local function drawFOVCircle()
    if fovCircle.Parent == nil then
        fovCircle.Parent = game.CoreGui
        fovCircle.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        fovCircle.BackgroundTransparency = 0.5
        fovCircle.BorderSizePixel = 0
        fovCircle.ZIndex = 10
    end
    -- Calcular o tamanho do círculo de FOV com base no FOV
    local radius = fov
    fovCircle.Size = UDim2.new(0, radius * 2, 0, radius * 2)
    fovCircle.Position = UDim2.new(0, (screenSize.X / 2) - radius, 0, (screenSize.Y / 2) - radius)
end

-- Atualiza o círculo de FOV sempre
runService.RenderStepped:Connect(function()
    if aimbotActive then
        drawFOVCircle()
    end
end)

-- Aba Principal
local MainTab = Window:MakeTab({
    Name = "Aimbot",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Botão ON/OFF
MainTab:AddToggle({
    Name = "Ativar Aimbot",
    Default = false,
    Callback = function(Value)
        aimbotActive = Value
    end    
})

-- Slider FOV
MainTab:AddSlider({
    Name = "Aimbot FOV",
    Min = 50,
    Max = 500,
    Default = 150,
    Color = Color3.fromRGB(0,0,0), -- texto preto
    Increment = 10,
    ValueName = "px",
    Callback = function(Value)
        fov = Value
    end    
})

-- Slider Smoothing
MainTab:AddSlider({
    Name = "Aimbot Suavização",
    Min = 0.05,
    Max = 1,
    Default = 0.2,
    Color = Color3.fromRGB(0,0,0), -- texto preto
    Increment = 0.05,
    ValueName = "x",
    Callback = function(Value)
        smoothing = Value
    end    
})

-- Toggle Aimlock (grudar na mira com LButton)
MainTab:AddToggle({
    Name = "Ativar Aimlock (LButton)",
    Default = false,
    Callback = function(Value)
        aimlockActive = Value
    end    
})

-- Créditos
MainTab:AddLabel("Feito por ChatGPT - Cor branca e texto preto!")

-- Tema Branco (padrão do Orion)
OrionLib:MakeNotification({
    Name = "Script Iniciado",
    Content = "Aimbot pronto para usar!",
    Image = "rbxassetid://4483345998",
    Time = 5
})

-- Lidar com o evento de pressionar o botão esquerdo do mouse (LButton)
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        -- LButton pressionado (starta o Aimlock)
        aimlockActive = true
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        -- LButton solto (desliga o Aimlock)
        aimlockActive = false
    end
end)
