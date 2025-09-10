-- Roblox Lua Script: Teleport GUI Lateral Atualizado
-- Painel arrastável + Auto Teleport com delay de 0.10s + Botão hambúrguer

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "TeleportGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

-- Lista de teleportes (11 locais)
local teleports = {
    {name = "Local 1", pos = Vector3.new(4009.78, 21.37, -6705.96)},
    {name = "Local 2", pos = Vector3.new(4125.84, 37.00, -6733.84)},
    {name = "Local 3", pos = Vector3.new(4124.55, 21.37, -6744.80)},
    {name = "Local 4", pos = Vector3.new(3953.47, 21.00, -6689.76)},
    {name = "Local 5", pos = Vector3.new(4100.20, 22.10, -6710.33)},
    {name = "Local 6", pos = Vector3.new(4077.15, 21.50, -6695.70)},
    {name = "Local 7", pos = Vector3.new(3999.42, 21.00, -6722.18)},
    {name = "Local 8", pos = Vector3.new(4022.11, 21.80, -6748.55)},
    {name = "Local 9", pos = Vector3.new(4140.66, 38.20, -6702.74)},
    {name = "Local 10", pos = Vector3.new(3968.37, 20.90, -6666.12)},
    {name = "Local 11", pos = Vector3.new(3935.25, 21.30, -6699.47)}
}

-- Painel lateral
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 320)
frame.Position = UDim2.new(0, 10, 0.3, 0) -- esquerda da tela
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.BackgroundTransparency = 0.05
frame.BorderSizePixel = 0
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,10)

-- Botão hambúrguer
local menuBtn = Instance.new("TextButton")
menuBtn.Size = UDim2.new(0, 40, 0, 40)
menuBtn.Position = UDim2.new(0, 10, 0, 10)
menuBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
menuBtn.Text = "☰"
menuBtn.TextScaled = true
menuBtn.TextColor3 = Color3.new(1,1,1)
menuBtn.Font = Enum.Font.SourceSansBold
menuBtn.Parent = gui
Instance.new("UICorner", menuBtn).CornerRadius = UDim.new(0,8)

local frameVisible = true
menuBtn.MouseButton1Click:Connect(function()
    frameVisible = not frameVisible
    frame.Visible = frameVisible
end)

-- Função para tornar frame arrastável
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    frame.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end

frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "🙂‍↕️boa noite cinderela"
title.TextScaled = true
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.Parent = frame

-- Lista (ScrollingFrame)
local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1, -10, 1, -50)
listFrame.Position = UDim2.new(0, 5, 0, 45)
listFrame.BackgroundTransparency = 1
listFrame.BorderSizePixel = 0
listFrame.ScrollBarThickness = 6
listFrame.CanvasSize = UDim2.new(0,0,0,#teleports*45 + 60)
listFrame.Parent = frame

local layout = Instance.new("UIListLayout", listFrame)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0,6)

-- Função de teleporte (suporte cadeira)
local tweenInfo = TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local function teleportTo(pos)
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid.SeatPart then
        TweenService:Create(humanoid.SeatPart, tweenInfo, {CFrame = CFrame.new(pos + Vector3.new(0,3,0))}):Play()
    else
        TweenService:Create(hrp, tweenInfo, {CFrame = CFrame.new(pos + Vector3.new(0,3,0))}):Play()
    end
end

-- Criar botões dos teleportes
for _, info in ipairs(teleports) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -8, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    btn.BackgroundTransparency = 0.1
    btn.BorderSizePixel = 0
    btn.Text = info.name
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Parent = listFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)

    btn.MouseButton1Click:Connect(function()
        teleportTo(info.pos)
    end)
end

-- Botão Auto Teleport
local autoTeleporting = false
local autoDelay = 0.10 -- 0.10s de delay

local autoBtn = Instance.new("TextButton")
autoBtn.Size = UDim2.new(1, -8, 0, 40)
autoBtn.BackgroundColor3 = Color3.fromRGB(80,50,50)
autoBtn.BackgroundTransparency = 0.08
autoBtn.BorderSizePixel = 0
autoBtn.Text = "Auto Teleport: OFF"
autoBtn.Font = Enum.Font
