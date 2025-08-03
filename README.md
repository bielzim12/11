-- LD MODZ MOBILE MENU - ESP + Aimbot Forte
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local drawings = {}
local espEnabled = false
local aimbotEnabled = false
local showFOV = true
local fovRadius = 100

-- ESP
function createESP(player)
drawings[player] = {
Box = Drawing.new("Square"),
Line = Drawing.new("Line"),
Name = Drawing.new("Text")
}
for _, d in pairs(drawings[player]) do d.Visible = false end
drawings[player].Box.Thickness = 2
drawings[player].Box.Filled = false
drawings[player].Box.Color = Color3.fromRGB(0, 255, 0)
drawings[player].Line.Thickness = 1.5
drawings[player].Name.Size = 13
drawings[player].Name.Center = true
drawings[player].Name.Outline = true
drawings[player].Name.Color = Color3.new(1, 1, 1)
end

function updateESP()
for _, player in pairs(Players:GetPlayers()) do
if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Head") then
if not drawings[player] then createESP(player) end
local root = player.Character.HumanoidRootPart
local head = player.Character.Head
local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 1, 0))
local footPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 2.5, 0))

if espEnabled and onScreen then  
            local height = math.abs(headPos.Y - footPos.Y)  
            local width = height / 2  
            local x = pos.X - width / 2  
            local y = headPos.Y  

            drawings[player].Box.Size = Vector2.new(width, height)  
            drawings[player].Box.Position = Vector2.new(x, y)  
            drawings[player].Box.Visible = true  

            local t = tick() % 6  
            local r = math.sin(t) * 127 + 128  
            local g = math.sin(t + 2) * 127 + 128  
            local b = math.sin(t + 4) * 127 + 128  
            drawings[player].Line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)  
            drawings[player].Line.To = Vector2.new(pos.X, pos.Y)  
            drawings[player].Line.Color = Color3.fromRGB(r, g, b)  
            drawings[player].Line.Visible = true  

            drawings[player].Name.Text = player.Name  
            drawings[player].Name.Position = Vector2.new(pos.X, y - 15)  
            drawings[player].Name.Visible = true  
        else  
            for _, d in pairs(drawings[player]) do d.Visible = false end  
        end  
    elseif drawings[player] then  
        for _, d in pairs(drawings[player]) do d.Visible = false end  
    end  
end

end

-- FOV Circle (fino)
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = fovRadius
fovCircle.Thickness = 0.5
fovCircle.NumSides = 100
fovCircle.Transparency = 1
fovCircle.Filled = false
fovCircle.Color = Color3.fromRGB(255, 255, 0)
fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
fovCircle.Visible = true

-- Aimbot: mira na cabeça do jogador mais próximo (ignora FOV)
function getClosestPlayer()
local closest = nil
local shortestDist = math.huge
local myPos = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.Position

if not myPos then return nil end  

for _, player in pairs(Players:GetPlayers()) do  
    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Head") then  
        local dist = (player.Character.HumanoidRootPart.Position - myPos).Magnitude  
        if dist < shortestDist then  
            shortestDist = dist  
            closest = player  
        end  
    end  
end  
return closest

end

-- Atualização contínua
RunService.RenderStepped:Connect(function()
updateESP()
fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
fovCircle.Visible = showFOV

if aimbotEnabled then  
    local target = getClosestPlayer()  
    if target and target.Character and target.Character:FindFirstChild("Head") then  
        local head = target.Character.Head  
        local dir = (head.Position - Camera.CFrame.Position).Unit  
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)  
    end  
end

end)

-- MENU FLUTUANTE MOBILE
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.ResetOnSpawn = false

-- Botão flutuante ≡
local menuButton = Instance.new("TextButton", ScreenGui)
menuButton.Size = UDim2.new(0, 45, 0, 45)
menuButton.Position = UDim2.new(0, 10, 0, 150)
menuButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
menuButton.Text = "≡"
menuButton.TextColor3 = Color3.new(1, 1, 1)
menuButton.TextScaled = true

-- Menu
local menuFrame = Instance.new("Frame", ScreenGui)
menuFrame.Size = UDim2.new(0, 200, 0, 140)
menuFrame.Position = UDim2.new(0, 60, 0, 150)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.Visible = false

-- Título
local title = Instance.new("TextLabel", menuFrame)
title.Size = UDim2.new(1, 0, 0, 25)
title.Text = "LD MODZ"
title.TextColor3 = Color3.fromRGB(255, 255, 0)
title.BackgroundTransparency = 1
title.TextScaled = true

-- Botão ESP
local espBtn = Instance.new("TextButton", menuFrame)
espBtn.Size = UDim2.new(1, -20, 0, 30)
espBtn.Position = UDim2.new(0, 10, 0, 35)
espBtn.Text = "ESP [OFF]"
espBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
espBtn.TextColor3 = Color3.new(1, 1, 1)
espBtn.TextScaled = true
espBtn.MouseButton1Click:Connect(function()
espEnabled = not espEnabled
espBtn.Text = espEnabled and "ESP [ON]" or "ESP [OFF]"
end)

-- Botão Aimbot
local aimBtn = Instance.new("TextButton", menuFrame)
aimBtn.Size = UDim2.new(1, -20, 0, 30)
aimBtn.Position = UDim2.new(0, 10, 0, 70)
aimBtn.Text = "Aimbot [OFF]"
aimBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
aimBtn.TextColor3 = Color3.new(1, 1, 1)
aimBtn.TextScaled = true
aimBtn.MouseButton1Click:Connect(function()
aimbotEnabled = not aimbotEnabled
aimBtn.Text = aimbotEnabled and "Aimbot [ON]" or "Aimbot [OFF]"
end)

-- Botão FOV
local fovBtn = Instance.new("TextButton", menuFrame)
fovBtn.Size = UDim2.new(1, -20, 0, 30)
fovBtn.Position = UDim2.new(0, 10, 0, 105)
fovBtn.Text = "FOV [ON]"
fovBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
fovBtn.TextColor3 = Color3.new(1, 1, 1)
fovBtn.TextScaled = true
fovBtn.MouseButton1Click:Connect(function()
showFOV = not showFOV
fovCircle.Visible = showFOV
fovBtn.Text = showFOV and "FOV [ON]" or "FOV [OFF]"
end)

-- Abre / Fecha o menu
menuButton.MouseButton1Click:Connect(function()
menuFrame.Visible = not menuFrame.Visible
end)
