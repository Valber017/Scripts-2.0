-- Configurações de cor
local colorTable = {
    Green = Color3.fromRGB(0, 255, 0),
    Blue = Color3.fromRGB(0, 0, 255),
    Red = Color3.fromRGB(255, 0, 0),
    Yellow = Color3.fromRGB(255, 255, 0),
    Orange = Color3.fromRGB(255, 165, 0),
    Purple = Color3.fromRGB(128, 0, 128)
}
local colorChosen = colorTable.Red -- Escolha a cor desejada

-- Variáveis globais
_G.ESPToggle = false
_G.AimbotToggle = false
_G.HitKillToggle = false
_G.NoReloadToggle = false

-- Serviços e jogador local
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

-- Função para obter o personagem do jogador
local function getCharacter(player)
    return Workspace:FindFirstChild(player.Name)
end

-- Adicionar destaque aos jogadores
local function addHighlightToCharacter(player, character)
    if player == LocalPlayer then return end -- Ignorar o jogador local
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if humanoidRootPart and not humanoidRootPart:FindFirstChild("Highlight") then
        local highlightClone = Instance.new("Highlight")
        highlightClone.Name = "Highlight"
        highlightClone.Adornee = character
        highlightClone.Parent = humanoidRootPart
        highlightClone.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlightClone.FillColor = colorChosen
        highlightClone.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlightClone.FillTransparency = 0.5

        -- Adicionar seta vermelha acima do inimigo
        local arrow = Instance.new("BillboardGui", humanoidRootPart)
        arrow.Name = "Arrow"
        arrow.Size = UDim2.new(1, 0, 1, 0)
        arrow.StudsOffset = Vector3.new(0, 3, 0)
        local arrowImage = Instance.new("ImageLabel", arrow)
        arrowImage.Size = UDim2.new(1, 0, 1, 0)
        arrowImage.BackgroundTransparency = 1
        arrowImage.Image = "rbxassetid://6031094678" -- ID da imagem da seta
        arrowImage.ImageColor3 = Color3.fromRGB(255, 0, 0)
    end
end

-- Remover destaque dos jogadores
local function removeHighlightFromCharacter(character)
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if humanoidRootPart then
        local highlightInstance = humanoidRootPart:FindFirstChild("Highlight")
        if highlightInstance then
            highlightInstance:Destroy()
        end
        local arrowInstance = humanoidRootPart:FindFirstChild("Arrow")
        if arrowInstance then
            arrowInstance:Destroy()
        end
    end
end

-- Atualizar destaques com base no valor de _G.ESPToggle
local function updateHighlights()
    for _, player in pairs(Players:GetPlayers()) do
        local character = getCharacter(player)
        if character then
            if _G.ESPToggle then
                addHighlightToCharacter(player, character)
            else
                removeHighlightFromCharacter(character)
            end
        end
    end
end

-- Função para obter o alvo mais próximo
local function getClosestTarget()
    local closestTarget = nil
    local shortestDistance = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
            local character = getCharacter(player)
            if character and character:FindFirstChild("HumanoidRootPart") then
                local distance = (character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                if distance < shortestDistance then
                    closestTarget = character
                    shortestDistance = distance
                end
            end
        end
    end
    return closestTarget
end

-- Função para mirar no alvo
local function aimAtTarget(target)
    if target and target:FindFirstChild("HumanoidRootPart") then
        local camera = Workspace.CurrentCamera
        camera.CFrame = CFrame.new(camera.CFrame.Position, target.HumanoidRootPart.Position)
    end
end

-- Função para matar o alvo
local function hitKillTarget(target)
    if target and target:FindFirstChild("Humanoid") then
        target.Humanoid.Health = 0
    end
end

-- Função para eliminar a necessidade de recarregar a arma
local function noReload()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool") then
        local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if tool:FindFirstChild("Ammo") then
            tool.Ammo.Value = tool.Ammo.MaxValue
        end
    end
end

-- Criar botão de toque para alternar ESP, AIMBOT, Hit Kill e No Reload
local function createToggleButton()
    local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
    local toggleButtonESP = Instance.new("TextButton", screenGui)
    toggleButtonESP.Size = UDim2.new(0, 200, 0, 50)
    toggleButtonESP.Position = UDim2.new(0.5, -100, 0.6, -25)
    toggleButtonESP.Text = "Toggle ESP"
    toggleButtonESP.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    toggleButtonESP.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButtonESP.TextScaled = true

    local toggleButtonAimbot = Instance.new("TextButton", screenGui)
    toggleButtonAimbot.Size = UDim2.new(0, 200, 0, 50)
    toggleButtonAimbot.Position = UDim2.new(0.5, -100, 0.7, -25)
    toggleButtonAimbot.Text = "Toggle Aimbot"
    toggleButtonAimbot.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    toggleButtonAimbot.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButtonAimbot.TextScaled = true

    local toggleButtonHitKill = Instance.new("TextButton", screenGui)
    toggleButtonHitKill.Size = UDim2.new(0, 200, 0, 50)
    toggleButtonHitKill.Position = UDim2.new(0.5, -100, 0.8, -25)
    toggleButtonHitKill.Text = "Toggle Hit Kill"
    toggleButtonHitKill.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    toggleButtonHitKill.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButtonHitKill.TextScaled = true

    local toggleButtonNoReload = Instance.new("TextButton", screenGui)
    toggleButtonNoReload.Size = UDim2.new(0, 200, 0, 50)
    toggleButtonNoReload.Position = UDim2.new(0.5, -100, 0.9, -25)
    toggleButtonNoReload.Text = "Toggle No Reload"
    toggleButtonNoReload.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    toggleButtonNoReload.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButtonNoReload.TextScaled = true

    toggleButtonESP.MouseButton1Click:Connect(function()
        _G.ESPToggle = not _G.ESPToggle
        updateHighlights()
    end)

    toggleButtonAimbot.MouseButton1Click:Connect(function()
        _G.AimbotToggle = not _G.AimbotToggle
    end)

    toggleButtonHitKill.MouseButton1Click:Connect(function()
        _G.HitKillToggle = not _G.HitKillToggle
    end)

    toggleButtonNoReload.MouseButton1Click:Connect(function()
        _G.NoReloadToggle = not _G.NoReloadToggle
    end)
end

-- Inicializar botões de toque
createToggleButton()

-- Atualizar destaques continuamente
RunService.RenderStepped:Connect(function()
    updateHighlights()
    if _G.AimbotToggle then
        local target = getClosestTarget()
        aimAtTarget(target)
    end
    if _G.HitKillToggle then
        local target = getClosestTarget()
        hitKillTarget(target)
    end
    if _G.NoReloadToggle then
        noReload()
    end
end)
