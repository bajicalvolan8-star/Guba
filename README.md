-- ==========================================
-- SCRIPT DE HITBOX ANTI-TRAVAMENTO (CORRIGIDO)
-- ==========================================
local HeadSize = 20
local HitboxTransparency = 0.7
local IsEnabled = true

local UIService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Tabela para guardar os dados originais dos players e não bugar o jogo
local originalProperties = {}

-- Garantindo a criação da interface
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HitboxPremiumMenu"
ScreenGui.ResetOnSpawn = false

local success, err = pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)
if not success then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Janela Principal (MainFrame)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 240)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -120)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.Active = true
MainFrame.Draggable = true

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Barra de Título
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Title.Text = "Hitbox Mod v3"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = Title
Title.Parent = MainFrame

-- Botão Fechar (X)
local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextSize = 16

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 8)
CloseCorner.Parent = CloseBtn
CloseBtn.Parent = MainFrame

-- Bolinha para Abrir quando Fechado
local OpenBtn = Instance.new("TextButton")
OpenBtn.Name = "OpenBtn"
OpenBtn.Size = UDim2.new(0, 50, 0, 50)
OpenBtn.Position = UDim2.new(0, 15, 0.2, 0) -- Ajustado para não tampar botões do jogo
OpenBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
OpenBtn.Text = "MENU"
OpenBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenBtn.Font = Enum.Font.SourceSansBold
OpenBtn.TextSize = 12
OpenBtn.Visible = false
OpenBtn.Active = true
OpenBtn.Draggable = true

local OpenCorner = Instance.new("UICorner")
OpenCorner.CornerRadius = UDim.new(1, 0)
OpenCorner.Parent = OpenBtn
OpenBtn.Parent = ScreenGui

-- Função Criadora de Sliders Mobile / PC
local function CreateSlider(name, yPos, min, max, default, callback)
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.8, 0, 0, 20)
    Label.Position = UDim2.new(0.1, 0, 0, yPos)
    Label.BackgroundTransparency = 1
    Label.Text = name .. ": " .. string.format("%.1f", default)
    Label.TextColor3 = Color3.fromRGB(230, 230, 230)
    Label.Font = Enum.Font.SourceSansSemibold
    Label.TextSize = 14
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = MainFrame
    
    local Track = Instance.new("Frame")
    Track.Size = UDim2.new(0.8, 0, 0, 8)
    Track.Position = UDim2.new(0.1, 0, 0, yPos + 22)
    Track.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    Track.BorderSizePixel = 0
    Instance.new("UICorner", Track).CornerRadius = UDim.new(0, 4)
    Track.Parent = MainFrame
    
    local Knob = Instance.new("TextButton")
    Knob.Size = UDim2.new(0, 18, 0, 18)
    local startPercent = (default - min) / (max - min)
    Knob.Position = UDim2.new(startPercent, -9, 0.5, -9)
    Knob.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    Knob.Text = ""
    Instance.new("UICorner", Knob).CornerRadius = UDim.new(1, 0)
    Knob.Parent = Track
    
    local dragging = false
    local inputConnection
    
    Knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            if inputConnection then inputConnection:Disconnect() end
            
            inputConnection = UIService.InputChanged:Connect(function(changedInput)
                if dragging and (changedInput.UserInputType == Enum.UserInputType.MouseMovement or changedInput.UserInputType == Enum.UserInputType.Touch) then
                    local mouseX = changedInput.Position.X
                    local trackLeft = Track.AbsolutePosition.X
                    local trackWidth = Track.AbsoluteSize.X
                    local relativeX = math.clamp(mouseX - trackLeft, 0, trackWidth)
                    local percentage = relativeX / trackWidth
                    
                    Knob.Position = UDim2.new(percentage, -9, 0.5, -9)
                    local value = min + (max - min) * percentage
                    Label.Text = name .. ": " .. string.format("%.1f", value)
                    callback(value)
                end
            end)
        end
    end)
    
    UIService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            if inputConnection then
                inputConnection:Disconnect()
                inputConnection = nil
            end
        end
    end)
end

-- Criando Sliders
CreateSlider("Tamanho da Cabeça", 55, 1, 30, HeadSize, function(val) HeadSize = val end)
CreateSlider("Transparência", 115, 0, 1, HitboxTransparency, function(val) HitboxTransparency = val end)

-- Botão de Ativar/Desativar Hitbox
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "ToggleBtn"
ToggleBtn.Size = UDim2.new(0.8, 0, 0, 35)
ToggleBtn.Position = UDim2.new(0.1, 0, 0, 185)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
ToggleBtn.Text = "Status: ATIVADO"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.TextSize = 15
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 8)
ToggleBtn.Parent = MainFrame

-- Função para restaurar o estado original de fábrica do jogo
local function resetHeads()
    for head, props in pairs(originalProperties) do
        if head and head.Parent then
            head.Size = props.Size
            head.Massless = props.Massless
            head.CanCollide = props.CanCollide
            head.Transparency = props.Transparency
        end
    end
    table.clear(originalProperties)
end

ToggleBtn.MouseButton1Click:Connect(function()
    IsEnabled = not IsEnabled
    if IsEnabled then
        ToggleBtn.Text = "Status: ATIVADO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
    else
        ToggleBtn.Text = "Status: DESATIVADO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        resetHeads()
    end
end)

-- Fechar / Mostrar
CloseBtn.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenBtn.Visible = true end)
OpenBtn.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenBtn.Visible = false end)

MainFrame.Parent = ScreenGui

-- ==========================================
-- LOOP PRINCIPAL COM ANTI-FREEZE
-- ==========================================
RunService.RenderStepped:Connect(function()
    if not IsEnabled then return end
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local head = player.Character:FindFirstChild("Head")
            if head and head:IsA("BasePart") then
                -- Salva as propriedades originais antes de alterar pela primeira vez
                if not originalProperties[head] then
                    originalProperties[head] = {
                        Size = head.Size,
                        Massless = head.Massless,
                        CanCollide = head.CanCollide,
                        Transparency = head.Transparency
                    }
                end
                
                -- Aplica as modificações sem quebrar a física
                head.Size = Vector3.new(HeadSize, HeadSize, HeadSize)
                head.Transparency = HitboxTransparency
                head.CanCollide = false
                head.Massless = true -- ISSO AQUI IMPEDE ELES DE TRAVAREM!
            end
        end
    end
end)
