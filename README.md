--[[
    ═══════════════════════════════════════════════════════════════════════════
              GOD MODE SYSTEM - IMORTALIDADE PROFISSIONAL
    ═══════════════════════════════════════════════════════════════════════════
    
    FUNCIONALIDADES:
    ✓ God Mode (Imortalidade) - Proteção contra qualquer dano
    ✓ UI moderna com efeitos visuais profissionais
    ✓ Sistema de logs em tempo real
    ✓ Animações suaves e feedback visual
    ✓ Proteção contra quedas e afogamento
    ✓ Regeneração automática de vida
    ✓ Indicadores visuais de status
    
    CONTROLES:
    • Botão GOD MODE → Ativa/desativa imortalidade
    • Tecla G → Atalho para ativar/desativar
--]]

-- Carregar serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")

-- Configurações do jogador
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- Variáveis de controle
local GodModeActive = false
local originalHealth = nil
local originalMaxHealth = nil
local connection = nil
local humanoid = nil
local character = nil
local regenThread = nil
local antiDrownThread = nil

-- Configurações do God Mode
local config = {
    AutoRegen = true,
    RegenAmount = 50,
    RegenDelay = 2,
    ImmuneToFalling = true,
    ImmuneToDrowning = true,
    ShowEffects = true
}

-- Criar ScreenGui principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GodModeSystem"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true

-- Efeitos visuais
local Blur = Instance.new("BlurEffect")
Blur.Size = 0
Blur.Parent = Lighting

local Bloom = Instance.new("BloomEffect")
Bloom.Intensity = 0
Bloom.Parent = Lighting

-- ========== UI PRINCIPAL ==========
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 420, 0, 520)
MainFrame.Position = UDim2.new(0.5, -210, 0.5, -260)
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Arredondamento
local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 24)
Corner.Parent = MainFrame

-- Sombra
local Shadow = Instance.new("ImageLabel")
Shadow.Size = UDim2.new(1, 40, 1, 40)
Shadow.Position = UDim2.new(0, -20, 0, -20)
Shadow.BackgroundTransparency = 1
Shadow.Image = "rbxassetid://1316045217"
Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
Shadow.ImageTransparency = 0.7
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(10, 10, 10, 10)
Shadow.Parent = MainFrame

-- Gradiente de fundo animado
local BackgroundGradient = Instance.new("Frame")
BackgroundGradient.Size = UDim2.new(1, 0, 1, 0)
BackgroundGradient.BackgroundTransparency = 1
BackgroundGradient.Parent = MainFrame

local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(15, 15, 35)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(25, 20, 45)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 30))
})
Gradient.Rotation = 45
Gradient.Parent = BackgroundGradient

-- Borda brilhante animada
local BorderGlow = Instance.new("Frame")
BorderGlow.Size = UDim2.new(1, 0, 1, 0)
BorderGlow.BackgroundTransparency = 1
BorderGlow.BorderSizePixel = 3
BorderGlow.BorderColor3 = Color3.fromRGB(255, 215, 0)
BorderGlow.BorderTransparency = 0.5
BorderGlow.Parent = MainFrame

local BorderCorner = Instance.new("UICorner")
BorderCorner.CornerRadius = UDim.new(0, 24)
BorderCorner.Parent = BorderGlow

-- ========== CABEÇALHO ==========
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 110)
Header.BackgroundColor3 = Color3.fromRGB(20, 20, 40)
Header.BackgroundTransparency = 0.3
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 24)
HeaderCorner.Parent = Header

local IconContainer = Instance.new("Frame")
IconContainer.Size = UDim2.new(0, 80, 0, 80)
IconContainer.Position = UDim2.new(0, 20, 0.5, -40)
IconContainer.BackgroundTransparency = 1
IconContainer.Parent = Header

local IconGlow = Instance.new("Frame")
IconGlow.Size = UDim2.new(1, 0, 1, 0)
IconGlow.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
IconGlow.BackgroundTransparency = 0.5
IconGlow.BorderSizePixel = 0
IconGlow.Parent = IconContainer

local IconGlowCorner = Instance.new("UICorner")
IconGlowCorner.CornerRadius = UDim.new(1, 0)
IconGlowCorner.Parent = IconGlow

local Icon = Instance.new("TextLabel")
Icon.Size = UDim2.new(1, 0, 1, 0)
Icon.BackgroundTransparency = 1
Icon.Text = "👑"
Icon.TextColor3 = Color3.fromRGB(255, 215, 0)
Icon.TextSize = 55
Icon.Font = Enum.Font.GothamBlack
Icon.Parent = IconContainer

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -120, 0, 45)
Title.Position = UDim2.new(0, 110, 0, 20)
Title.BackgroundTransparency = 1
Title.Text = "GOD MODE"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 32
Title.Font = Enum.Font.GothamBlack
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Header

local SubTitle = Instance.new("TextLabel")
SubTitle.Size = UDim2.new(1, -120, 0, 30)
SubTitle.Position = UDim2.new(0, 110, 0, 65)
SubTitle.BackgroundTransparency = 1
SubTitle.Text = "Sistema de Imortalidade"
SubTitle.TextColor3 = Color3.fromRGB(180, 180, 220)
SubTitle.TextSize = 13
SubTitle.Font = Enum.Font.Gotham
SubTitle.TextXAlignment = Enum.TextXAlignment.Left
SubTitle.Parent = Header

-- ========== BOTÃO PRINCIPAL GOD MODE ==========
local GodButton = Instance.new("TextButton")
GodButton.Size = UDim2.new(0, 340, 0, 80)
GodButton.Position = UDim2.new(0.5, -170, 0, 135)
GodButton.Text = "👑 ATIVAR GOD MODE"
GodButton.TextColor3 = Color3.fromRGB(255, 255, 255)
GodButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
GodButton.Font = Enum.Font.GothamBold
GodButton.TextSize = 18
GodButton.BorderSizePixel = 0
GodButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 16)
ButtonCorner.Parent = GodButton

-- Efeito de brilho no botão
local ButtonGlow = Instance.new("Frame")
ButtonGlow.Size = UDim2.new(1, 0, 1, 0)
ButtonGlow.BackgroundTransparency = 1
ButtonGlow.BorderSizePixel = 2
ButtonGlow.BorderColor3 = Color3.fromRGB(255, 215, 0)
ButtonGlow.BorderTransparency = 1
ButtonGlow.Parent = GodButton

local ButtonGlowCorner = Instance.new("UICorner")
ButtonGlowCorner.CornerRadius = UDim.new(0, 16)
ButtonGlowCorner.Parent = ButtonGlow

-- Efeitos hover
GodButton.MouseEnter:Connect(function()
    TweenService:Create(GodButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(55, 55, 80)}):Play()
    TweenService:Create(ButtonGlow, TweenInfo.new(0.2), {BorderTransparency = 0.6}):Play()
end)
GodButton.MouseLeave:Connect(function()
    if not GodModeActive then
        TweenService:Create(GodButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40, 40, 60)}):Play()
    end
    TweenService:Create(ButtonGlow, TweenInfo.new(0.2), {BorderTransparency = 1}):Play()
end)

-- ========== PAINEL DE STATUS ==========
local StatusPanel = Instance.new("Frame")
StatusPanel.Size = UDim2.new(0.9, 0, 0, 150)
StatusPanel.Position = UDim2.new(0.05, 0, 0, 235)
StatusPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 45)
StatusPanel.BackgroundTransparency = 0.4
StatusPanel.BorderSizePixel = 0
StatusPanel.Parent = MainFrame

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0, 16)
StatusCorner.Parent = StatusPanel

-- Status God Mode
local GodStatus = Instance.new("TextLabel")
GodStatus.Size = UDim2.new(1, -20, 0, 40)
GodStatus.Position = UDim2.new(0, 10, 0, 12)
GodStatus.BackgroundTransparency = 1
GodStatus.Text = "👑 STATUS: 🔴 DESATIVADO"
GodStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
GodStatus.TextSize = 16
GodStatus.Font = Enum.Font.GothamBold
GodStatus.TextXAlignment = Enum.TextXAlignment.Left
GodStatus.Parent = StatusPanel

-- Status de Vida
local HealthStatus = Instance.new("TextLabel")
HealthStatus.Size = UDim2.new(1, -20, 0, 35)
HealthStatus.Position = UDim2.new(0, 10, 0, 55)
HealthStatus.BackgroundTransparency = 1
HealthStatus.Text = "❤️ VIDA: --/--"
HealthStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
HealthStatus.TextSize = 14
HealthStatus.Font = Enum.Font.GothamSemibold
HealthStatus.TextXAlignment = Enum.TextXAlignment.Left
HealthStatus.Parent = StatusPanel

-- Barra de Vida Visual
local HealthBarBg = Instance.new("Frame")
HealthBarBg.Size = UDim2.new(0.8, 0, 0, 12)
HealthBarBg.Position = UDim2.new(0, 10, 0, 95)
HealthBarBg.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
HealthBarBg.BorderSizePixel = 0
HealthBarBg.Parent = StatusPanel

local HealthBarBgCorner = Instance.new("UICorner")
HealthBarBgCorner.CornerRadius = UDim.new(0, 6)
HealthBarBgCorner.Parent = HealthBarBg

local HealthBar = Instance.new("Frame")
HealthBar.Size = UDim2.new(1, 0, 1, 0)
HealthBar.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
HealthBar.BorderSizePixel = 0
HealthBar.Parent = HealthBarBg

local HealthBarCorner = Instance.new("UICorner")
HealthBarCorner.CornerRadius = UDim.new(0, 6)
HealthBarCorner.Parent = HealthBar

-- Status de Proteção
local ProtectionStatus = Instance.new("TextLabel")
ProtectionStatus.Size = UDim2.new(1, -20, 0, 30)
ProtectionStatus.Position = UDim2.new(0, 10, 0, 112)
ProtectionStatus.BackgroundTransparency = 1
ProtectionStatus.Text = "🛡️ PROTEÇÕES: NENHUMA"
ProtectionStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
ProtectionStatus.TextSize = 12
ProtectionStatus.Font = Enum.Font.Gotham
ProtectionStatus.TextXAlignment = Enum.TextXAlignment.Left
ProtectionStatus.Parent = StatusPanel

-- ========== PAINEL DE INFORMAÇÕES ==========
local InfoPanel = Instance.new("Frame")
InfoPanel.Size = UDim2.new(0.9, 0, 0, 100)
InfoPanel.Position = UDim2.new(0.05, 0, 0, 405)
InfoPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 45)
InfoPanel.BackgroundTransparency = 0.4
InfoPanel.BorderSizePixel = 0
InfoPanel.Parent = MainFrame

local InfoCorner = Instance.new("UICorner")
InfoCorner.CornerRadius = UDim.new(0, 16)
InfoCorner.Parent = InfoPanel

local InfoTitle = Instance.new("TextLabel")
InfoTitle.Size = UDim2.new(1, -20, 0, 25)
InfoTitle.Position = UDim2.new(0, 10, 0, 8)
InfoTitle.BackgroundTransparency = 1
InfoTitle.Text = "⚡ BENEFÍCIOS DO GOD MODE"
InfoTitle.TextColor3 = Color3.fromRGB(200, 200, 255)
InfoTitle.TextSize = 12
InfoTitle.Font = Enum.Font.GothamBold
InfoTitle.TextXAlignment = Enum.TextXAlignment.Left
InfoTitle.Parent = InfoPanel

local Benefits = Instance.new("TextLabel")
Benefits.Size = UDim2.new(1, -20, 0, 60)
Benefits.Position = UDim2.new(0, 10, 0, 35)
Benefits.BackgroundTransparency = 1
Benefits.Text = "✓ Imune a qualquer tipo de dano\n✓ Regeneração automática de vida\n✓ Proteção contra quedas e afogamento\n✓ Efeitos visuais especiais"
Benefits.TextColor3 = Color3.fromRGB(170, 170, 200)
Benefits.TextSize = 11
Benefits.Font = Enum.Font.Gotham
Benefits.TextXAlignment = Enum.TextXAlignment.Left
Benefits.TextYAlignment = Enum.TextYAlignment.Top
Benefits.Parent = InfoPanel

-- ========== PAINEL DE LOGS ==========
local LogPanel = Instance.new("Frame")
LogPanel.Size = UDim2.new(0.9, 0, 0, 85)
LogPanel.Position = UDim2.new(0.05, 0, 0, 520)
LogPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 45)
LogPanel.BackgroundTransparency = 0.4
LogPanel.BorderSizePixel = 0
LogPanel.Parent = MainFrame

local LogCorner = Instance.new("UICorner")
LogCorner.CornerRadius = UDim.new(0, 16)
LogCorner.Parent = LogPanel

local LogTitle = Instance.new("TextLabel")
LogTitle.Size = UDim2.new(1, -20, 0, 20)
LogTitle.Position = UDim2.new(0, 10, 0, 5)
LogTitle.BackgroundTransparency = 1
LogTitle.Text = "📋 SISTEMA DE LOGS"
LogTitle.TextColor3 = Color3.fromRGB(200, 200, 255)
LogTitle.TextSize = 10
LogTitle.Font = Enum.Font.GothamBold
LogTitle.TextXAlignment = Enum.TextXAlignment.Left
LogTitle.Parent = LogPanel

local LogTextBox = Instance.new("TextBox")
LogTextBox.Size = UDim2.new(1, -20, 1, -30)
LogTextBox.Position = UDim2.new(0, 10, 0, 25)
LogTextBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
LogTextBox.BackgroundTransparency = 0.3
LogTextBox.TextColor3 = Color3.fromRGB(180, 180, 200)
LogTextBox.Text = "> Sistema God Mode iniciado\n> Pressione G ou clique no botão para ativar"
LogTextBox.TextWrapped = true
LogTextBox.TextXAlignment = Enum.TextXAlignment.Left
LogTextBox.TextYAlignment = Enum.TextYAlignment.Top
LogTextBox.ClearTextOnFocus = false
LogTextBox.Font = Enum.Font.Code
LogTextBox.TextSize = 9
LogTextBox.Parent = LogPanel

local LogCornerInner = Instance.new("UICorner")
LogCornerInner.CornerRadius = UDim.new(0, 8)
LogCornerInner.Parent = LogTextBox

-- ========== FUNÇÕES DE LOG ==========
local function AddLog(message)
    local timestamp = os.date("%H:%M:%S")
    local lines = {}
    for line in LogTextBox.Text:gmatch("[^\r\n]+") do
        table.insert(lines, line)
    end
    table.insert(lines, "> [" .. timestamp .. "] " .. message)
    if #lines > 6 then table.remove(lines, 1) end
    LogTextBox.Text = table.concat(lines, "\n")
end

-- ========== ATUALIZAR STATUS NA UI ==========
local function UpdateUIStatus()
    if GodModeActive then
        GodStatus.Text = "👑 STATUS: 🟢 ATIVADO"
        GodStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
        GodButton.Text = "👑 GOD MODE ATIVADO"
        GodButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        
        local protections = {}
        if config.ImmuneToFalling then table.insert(protections, "Queda") end
        if config.ImmuneToDrowning then table.insert(protections, "Afogamento") end
        if config.AutoRegen then table.insert(protections, "Regeneração") end
        ProtectionStatus.Text = "🛡️ PROTEÇÕES: " .. table.concat(protections, " • ")
        ProtectionStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
    else
        GodStatus.Text = "👑 STATUS: 🔴 DESATIVADO"
        GodStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
        GodButton.Text = "👑 ATIVAR GOD MODE"
        GodButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
        ProtectionStatus.Text = "🛡️ PROTEÇÕES: NENHUMA"
        ProtectionStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
    end
end

-- ========== SISTEMA GOD MODE ==========
local function ApplyGodMode()
    character = LocalPlayer.Character
    if not character then return end
    
    humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    if GodModeActive then
        -- Salvar vida original
        originalHealth = humanoid.Health
        originalMaxHealth = humanoid.MaxHealth
        
        -- Tornar imortal
        humanoid.BreakJointsOnDeath = false
        
        -- Remover conexão de dano anterior se existir
        if connection then
            connection:Disconnect()
        end
        
        -- Conectar ao evento de dano
        connection = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if GodModeActive and humanoid.Health < originalHealth then
                humanoid.Health = originalHealth
                
                -- Efeito visual ao receber dano
                if config.ShowEffects then
                    local highlight = Instance.new("Highlight")
                    highlight.Parent = character
                    highlight.FillColor = Color3.fromRGB(255, 215, 0)
                    highlight.FillTransparency = 0.6
                    highlight.OutlineColor = Color3.fromRGB(255, 215, 0)
                    highlight.OutlineTransparency = 0.3
                    game:GetService("Debris"):AddItem(highlight, 0.2)
                    
                    -- Efeito de partícula dourada
                    local particles = Instance.new("ParticleEmitter")
                    particles.Parent = character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
                    particles.Texture = "rbxasset://textures/particles/sparkles_main.dds"
                    particles.Color = ColorSequence.new(Color3.fromRGB(255, 215, 0))
                    particles.Rate = 50
                    particles.Lifetime = NumberRange.new(0.3)
                    particles.SpreadAngle = Vector2.new(360, 360)
                    particles.VelocityInherit = 0
                    particles.Enabled = true
                    game:GetService("Debris"):AddItem(particles, 0.3)
                end
            end
        end)
        
        -- Proteção contra quedas
        if config.ImmuneToFalling then
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        end
        
        -- Sistema de regeneração automática
        if config.AutoRegen then
            if regenThread then coroutine.close(regenThread) end
            regenThread = coroutine.create(function()
                while GodModeActive do
                    wait(config.RegenDelay)
                    if humanoid and humanoid.Health < humanoid.MaxHealth then
                        local newHealth = math.min(humanoid.Health + config.RegenAmount, humanoid.MaxHealth)
                        humanoid.Health = newHealth
                        AddLog("⚡ Regenerando: " .. math.floor(newHealth) .. "/" .. humanoid.MaxHealth)
                        
                        -- Efeito de regeneração
                        if config.ShowEffects then
                            local regenEffect = Instance.new("SelectionBox")
                            regenEffect.Adornee = character
                            regenEffect.Color3 = Color3.fromRGB(100, 255, 100)
                            regenEffect.LineThickness = 0.05
                            regenEffect.Transparency = 0.5
                            game:GetService("Debris"):AddItem(regenEffect, 0.5)
                        end
                    end
                end
            end)
            coroutine.resume(regenThread)
        end
        
        -- Efeito visual de God Mode (aura)
        if config.ShowEffects then
            local godAura = Instance.new("SelectionBox")
            godAura.Adornee = character
            godAura.Color3 = Color3.fromRGB(255, 215, 0)
            godAura.LineThickness = 0.1
            godAura.Transparency = 0.4
            godAura.Parent = character
            
            -- Animação pulsante da aura
            spawn(function()
                local pulse = 0
                while godAura and godAura.Parent and GodModeActive do
                    pulse = pulse + 0.05
                    local alpha = 0.3 + math.sin(pulse) * 0.2
                    godAura.Transparency = alpha
                    wait(0.05)
                end
                if godAura then godAura:Destroy() end
            end)
        end
        
        AddLog("✨ GOD MODE ATIVADO! Você está imortal!")
        
    else
        -- Desativar God Mode
        if humanoid then
            humanoid.BreakJointsOnDeath = true
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
            
            if connection then
                connection:Disconnect()
                connection = nil
            end
        end
        
        -- Parar regeneração
        if regenThread then
            regenThread = nil
        end
        
        -- Remover efeitos visuais
        for _, effect in pairs(character:GetChildren()) do
            if effect:IsA("SelectionBox") or effect:IsA("Highlight") then
                effect:Destroy()
            end
        end
        
        AddLog("💀 GOD MODE DESATIVADO! Você voltou ao normal")
    end
    
    UpdateUIStatus()
end

-- ========== ATUALIZAR VIDA NA UI ==========
local function UpdateHealthDisplay()
    if character and humanoid then
        local currentHealth = humanoid.Health
        local maxHealth = humanoid.MaxHealth
        HealthStatus.Text = string.format("❤️ VIDA: %.0f / %.0f", currentHealth, maxHealth)
        
        local healthPercent = currentHealth / maxHealth
        HealthBar.Size = UDim2.new(healthPercent, 0, 1, 0)
        
        if healthPercent < 0.3 then
            HealthStatus.TextColor3 = Color3.fromRGB(255, 50, 50)
            HealthBar.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        elseif healthPercent < 0.6 then
            HealthStatus.TextColor3 = Color3.fromRGB(255, 200, 50)
            HealthBar.BackgroundColor3 = Color3.fromRGB(255, 200, 50)
        else
            HealthStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
            HealthBar.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        end
    end
end

-- ========== PROTEÇÃO CONTRA AFOGAMENTO ==========
local function AntiDrown()
    while true do
        wait(2)
        if GodModeActive and config.ImmuneToDrowning and character then
            local humanoidRoot = character:FindFirstChild("HumanoidRootPart")
            if humanoidRoot then
                local waterParts = workspace:FindPartsInRegion3(
                    Region3.new(humanoidRoot.Position - Vector3.new(3, 2, 3), humanoidRoot.Position + Vector3.new(3, 2, 3)),
                    nil,
                    100
                )
                
                local inWater = false
                for _, part in pairs(waterParts) do
                    if part.Material == Enum.Material.Water then
                        inWater = true
                        break
                    end
                end
                
                if inWater then
                    humanoidRoot.Velocity = Vector3.new(humanoidRoot.Velocity.X, 15, humanoidRoot.Velocity.Z)
                    AddLog("💧 Proteção contra afogamento ativada!")
                    
                    -- Efeito visual
                    if config.ShowEffects then
                        local splash = Instance.new("Part")
                        splash.Size = Vector3.new(2, 0.5, 2)
                        splash.Position = humanoidRoot.Position
                        splash.Anchored = true
                        splash.CanCollide = false
                        splash.Material = Enum.Material.Neon
                        splash.BrickColor = BrickColor.new("Bright blue")
                        splash.Parent = workspace
                        game:GetService("Debris"):AddItem(splash, 0.5)
                    end
                end
            end
        end
    end
end

-- ========== FUNÇÃO TOGGLE GOD MODE ==========
local function ToggleGodMode()
    GodModeActive = not GodModeActive
    
    if GodModeActive then
        ApplyGodMode()
        
        -- Efeitos visuais na UI
        TweenService:Create(Bloom, TweenInfo.new(0.5), {Intensity = 0.3}):Play()
        TweenService:Create(Blur, TweenInfo.new(0.5), {Size = 2}):Play()
        TweenService:Create(BorderGlow, TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), 
            {BorderTransparency = 0.2}):Play()
        TweenService:Create(IconGlow, TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), 
            {BackgroundTransparency = 0.3}):Play()
        
        -- Efeito de partícula no botão
        local particles = Instance.new("ParticleEmitter")
        particles.Parent = GodButton
        particles.Texture = "rbxasset://textures/particles/sparkles_main.dds"
        particles.Color = ColorSequence.new(Color3.fromRGB(255, 215, 0))
        particles.Rate = 30
        particles.Lifetime = NumberRange.new(0.5)
        particles.SpreadAngle = Vector2.new(360, 360)
        game:GetService("Debris"):AddItem(particles, 1)
        
    else
        ApplyGodMode()
        
        -- Remover efeitos visuais
        TweenService:Create(Bloom, TweenInfo.new(0.5), {Intensity = 0}):Play()
        TweenService:Create(Blur, TweenInfo.new(0.5), {Size = 0}):Play()
        TweenService:Create(BorderGlow, TweenInfo.new(0.3), {BorderTransparency = 0.5}):Play()
        TweenService:Create(IconGlow, TweenInfo.new(0.3), {BackgroundTransparency = 0.5}):Play()
    end
    
    UpdateUIStatus()
end

-- ========== MONITORAR PERSONAGEM ==========
local function onCharacterAdded(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    
    if GodModeActive then
        ApplyGodMode()
    end
    
    -- Atualizar vida periodicamente
    spawn(function()
        while character and humanoid do
            UpdateHealthDisplay()
            wait(0.3)
        end
    end)
end

-- Conectar eventos
if LocalPlayer.Character then
    onCharacterAdded(LocalPlayer.Character)
end
LocalPlayer.CharacterAdded:Connect(onCharacterAdded)

-- Conectar botão
GodButton.MouseButton1Click:Connect(ToggleGodMode)

-- ========== HOTKEY ==========
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.G then
        ToggleGodMode()
    end
end)

-- ========== INICIAR PROTEÇÃO CONTRA AFOGAMENTO ==========
spawn(AntiDrown)

-- ========== ANIMAÇÃO DE ENTRADA ==========
MainFrame.BackgroundTransparency = 1
TweenService:Create(MainFrame, TweenInfo.new(0.6, Enum.EasingStyle.Quad), {BackgroundTransparency = 0.05}):Play()
TweenService:Create(BorderGlow, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), 
    {BorderTransparency = 0.5}):Play()

-- Mensagens de inicialização
AddLog("🎮 God Mode System carregado com sucesso!")
AddLog("💪 Clique no botão ou pressione G para se tornar imortal!")
AddLog("✨ Proteção contra quedas, afogamento e qualquer dano!")

print("✅ God Mode System carregado com sucesso!")
print("💪 Pressione G para ativar/desativar a imortalidade!")
