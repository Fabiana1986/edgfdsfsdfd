-- tp_noclip_gui_img.lua
-- Script local Lua para Roblox
-- GUI com 2 botões, visual bonito com imagem de fundo:
-- Botão 1: Marcar local com T e teleporta para o local marcado ao clicar no botão
-- Botão 2: Liga/desliga Noclip
-- Pressionar P abre/fecha a tabela

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local marker = nil
local markerPart = nil
local noclipActive = false
local noclipConnection = nil

-- ID de exemplo de imagem de fundo (Roblox decal asset ID)
local BACKGROUND_IMAGE_ID = "rbxassetid://6071575925" -- Você pode trocar por outro ID se quiser

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TpNoclipGui"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.Enabled = false -- começa oculta

-- Frame principal com cantos arredondados
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 320, 0, 200)
Frame.Position = UDim2.new(0, 35, 0, 120)
Frame.BackgroundTransparency = 1
Frame.Parent = ScreenGui

-- Imagem de fundo
local BgImage = Instance.new("ImageLabel")
BgImage.Size = UDim2.new(1, 0, 1, 0)
BgImage.Position = UDim2.new(0,0,0,0)
BgImage.BackgroundTransparency = 1
BgImage.Image = BACKGROUND_IMAGE_ID
BgImage.ScaleType = Enum.ScaleType.Crop
BgImage.Parent = Frame

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 20)
UICorner.Parent = BgImage

-- Sobreposição semitransparente para melhor leitura dos botões
local Overlay = Instance.new("Frame")
Overlay.Size = UDim2.new(1, -24, 1, -24)
Overlay.Position = UDim2.new(0, 12, 0, 12)
Overlay.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Overlay.BackgroundTransparency = 0.25
Overlay.Parent = Frame

local UICorner2 = Instance.new("UICorner")
UICorner2.CornerRadius = UDim.new(0, 18)
UICorner2.Parent = Overlay

-- Botão Teleporte
local TpButton = Instance.new("TextButton")
TpButton.Size = UDim2.new(0, 260, 0, 45)
TpButton.Position = UDim2.new(0, 30, 0, 48)
TpButton.BackgroundColor3 = Color3.fromRGB(51, 153, 255)
TpButton.Text = "📍 Teleporte: Marque com T"
TpButton.TextColor3 = Color3.fromRGB(255,255,255)
TpButton.Font = Enum.Font.GothamBold
TpButton.TextSize = 20
TpButton.AutoButtonColor = true
TpButton.Parent = Overlay

local TpCorner = Instance.new("UICorner")
TpCorner.CornerRadius = UDim.new(0, 12)
TpCorner.Parent = TpButton

-- Botão Noclip
local NoclipButton = Instance.new("TextButton")
NoclipButton.Size = UDim2.new(0, 260, 0, 45)
NoclipButton.Position = UDim2.new(0, 30, 0, 104)
NoclipButton.BackgroundColor3 = Color3.fromRGB(255, 80, 120)
NoclipButton.Text = "🚪 Noclip: OFF"
NoclipButton.TextColor3 = Color3.fromRGB(255,255,255)
NoclipButton.Font = Enum.Font.GothamBold
NoclipButton.TextSize = 20
NoclipButton.AutoButtonColor = true
NoclipButton.Parent = Overlay

local NcCorner = Instance.new("UICorner")
NcCorner.CornerRadius = UDim.new(0, 12)
NcCorner.Parent = NoclipButton

-- Label de instruções
local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size = UDim2.new(1, -10, 0, 28)
InfoLabel.Position = UDim2.new(0, 5, 0, 5)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Text = "🟦 P: Abrir/fechar | 📍 T: Marcar | 🚪 Noclip"
InfoLabel.TextColor3 = Color3.fromRGB(240,240,240)
InfoLabel.TextStrokeTransparency = 0.7
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.TextSize = 16
InfoLabel.Parent = Overlay

-- Efeito glow sutil nos botões
local function addButtonGlow(btn, color)
    local glow = Instance.new("UIStroke")
    glow.Thickness = 2
    glow.Color = color
    glow.Transparency = 0.5
    glow.Parent = btn
end
addButtonGlow(TpButton, Color3.fromRGB(100,170,255))
addButtonGlow(NoclipButton, Color3.fromRGB(255,110,150))

-- Cria marcador visual
local function createMarker(position)
    if markerPart then markerPart:Destroy() end
    markerPart = Instance.new("Part")
    markerPart.Shape = Enum.PartType.Ball
    markerPart.Size = Vector3.new(1.2,1.2,1.2)
    markerPart.Color = Color3.fromRGB(250, 50, 50)
    markerPart.Anchored = true
    markerPart.CanCollide = false
    markerPart.Position = position
    markerPart.Material = Enum.Material.Neon
    markerPart.Transparency = 0.25
    markerPart.Parent = workspace
end

-- Marcar o local com T
local mouse = LocalPlayer:GetMouse()
UserInputService.InputBegan:Connect(function(input, processed)
    if not processed then
        if input.KeyCode == Enum.KeyCode.T and ScreenGui.Enabled then
            local hit = mouse.Hit
            if hit then
                marker = hit.Position
                createMarker(marker)
            end
        elseif input.KeyCode == Enum.KeyCode.P then
            ScreenGui.Enabled = not ScreenGui.Enabled
        end
    end
end)

-- Teleporta para o local marcado ao clicar no botão 1
TpButton.MouseButton1Click:Connect(function()
    if marker then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(marker)
            if markerPart then
                markerPart:Destroy()
                markerPart = nil
            end
        end
    end
end)

-- Noclip visual e lógica
local function setNoclip(state)
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    noclipActive = state
    NoclipButton.Text = "🚪 Noclip: " .. (noclipActive and "ON" or "OFF")
    NoclipButton.BackgroundColor3 = noclipActive and Color3.fromRGB(115, 240, 120) or Color3.fromRGB(255, 80, 120)
    if noclipActive then
        noclipConnection = game:GetService("RunService").Stepped:Connect(function()
            local char = LocalPlayer.Character
            if char then
                for _, part in ipairs(char:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end
end

NoclipButton.MouseButton1Click:Connect(function()
    setNoclip(not noclipActive)
end)

-- Limpa marcador e desativa noclip se personagem morrer
LocalPlayer.CharacterAdded:Connect(function()
    if markerPart then
        markerPart:Destroy()
        markerPart = nil
    end
    marker = nil
    setNoclip(false)
end)

print("[TpNoclipGui] Use P para abrir/fechar, T para marcar, botão 1 para teleportar, botão 2 para noclip.")
