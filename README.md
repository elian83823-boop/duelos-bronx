-- =========================================================================
-- ELIAN HUB v3.8 - BIG HEALTH BAR FIX + FULL MOBILE & PC DETECT
-- =========================================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local SoundService = game:GetService("SoundService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Detección ultra precisa de dispositivo
local isMobile = UserInputService.TouchEnabled

local function getValidParent()
    local success, core = pcall(function() return CoreGui end)
    if success and core then return core end
    return player:WaitForChild("PlayerGui", 5) or player.PlayerGui
end

local parentGui = getValidParent()

-- Limpieza de versiones anteriores
local function destruirTodo()
    pcall(function()
        for _, guiName in ipairs({"ElianHubMain", "ElianMobileAimGui", "ElianHubOpenBtn", "ElianMinimapGui", "ElianWelcomeGui", "ElianNotifyGui", "ElianEspContainer"}) do
            local old = parentGui:FindFirstChild(guiName)
            if old then old:Destroy() end
        end
    end)
end
destruirTodo()

-- Contenedor global seguro para los elementos del ESP
local espContainer = Instance.new("Folder")
espContainer.Name = "ElianEspContainer"
espContainer.Parent = parentGui

-- =========================================================================
-- DISCORD & NOTIFICACIONES
-- =========================================================================
local DISCORD_LINK = "https://discord.gg/JfB7geqbR"

local function setClipboardLink(text)
    if setclipboard then
        setclipboard(text)
    elseif toclipboard then
        toclipboard(text)
    elseif Clipboard and Clipboard.set then
        Clipboard.set(text)
    end
end

local function showNotification(msg)
    pcall(function()
        local old = parentGui:FindFirstChild("ElianNotifyGui")
        if old then old:Destroy() end
    end)

    local notifySound = Instance.new("Sound")
    notifySound.SoundId = "rbxassetid://4590662766"
    notifySound.Volume = 1
    notifySound.Parent = SoundService
    notifySound:Play()
    game:GetService("Debris"):AddItem(notifySound, 2)

    local NotifyGui = Instance.new("ScreenGui")
    NotifyGui.Name = "ElianNotifyGui"
    NotifyGui.Parent = parentGui
    NotifyGui.ResetOnSpawn = false

    local Frame = Instance.new("Frame", NotifyGui)
    Frame.BackgroundColor3 = Color3.fromRGB(15, 18, 26)
    Frame.Position = UDim2.new(0.5, -120, 0.82, 0)
    Frame.Size = UDim2.new(0, 240, 0, 40)
    Frame.BackgroundTransparency = 0.1
    Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 8)

    local Stroke = Instance.new("UIStroke", Frame)
    Stroke.Color = Color3.fromRGB(88, 101, 242)
    Stroke.Thickness = 2

    local Text = Instance.new("TextLabel", Frame)
    Text.Size = UDim2.new(1, 0, 1, 0)
    Text.BackgroundTransparency = 1
    Text.Font = Enum.Font.GothamBold
    Text.Text = msg
    Text.TextColor3 = Color3.fromRGB(240, 243, 255)
    Text.TextSize = 11

    task.delay(2, function()
        local tweenInfo = TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        local tweenFrame = TweenService:Create(Frame, tweenInfo, {BackgroundTransparency = 1, Position = UDim2.new(0.5, -120, 0.85, 0)})
        local tweenText = TweenService:Create(Text, tweenInfo, {TextTransparency = 1})
        local tweenStroke = TweenService:Create(Stroke, tweenInfo, {Transparency = 1})

        tweenFrame:Play()
        tweenText:Play()
        tweenStroke:Play()

        tweenFrame.Completed:Connect(function()
            NotifyGui:Destroy()
        end)
    end)
end

-- =========================================================================
-- BIENVENIDA Y SONIDO
-- =========================================================================
local function playWelcomeSoundAndNotify()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6895079853"
    sound.Volume = 1.5
    sound.Parent = SoundService
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 3)

    local WelcomeGui = Instance.new("ScreenGui")
    WelcomeGui.Name = "ElianWelcomeGui"
    WelcomeGui.Parent = parentGui
    WelcomeGui.ResetOnSpawn = false

    local NotifyFrame = Instance.new("Frame", WelcomeGui)
    NotifyFrame.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
    NotifyFrame.Position = UDim2.new(0.5, -130, 0.05, 0)
    NotifyFrame.Size = UDim2.new(0, 260, 0, 45)
    NotifyFrame.BackgroundTransparency = 0.1
    Instance.new("UICorner", NotifyFrame).CornerRadius = UDim.new(0, 8)

    local Stroke = Instance.new("UIStroke", NotifyFrame)
    Stroke.Color = Color3.fromRGB(0, 230, 255)
    Stroke.Thickness = 2

    local Text = Instance.new("TextLabel", NotifyFrame)
    Text.Size = UDim2.new(1, 0, 1, 0)
    Text.BackgroundTransparency = 1
    Text.Font = Enum.Font.GothamBold
    Text.Text = "⚡ ELIAN HUB (" .. (isMobile and "📱 MÓVIL" or "🪟 WINDOWS") .. ") ⚡"
    Text.TextColor3 = Color3.fromRGB(0, 230, 255)
    Text.TextSize = 11

    task.delay(2.5, function()
        local tweenInfo = TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        local tweenFrame = TweenService:Create(NotifyFrame, tweenInfo, {BackgroundTransparency = 1, Position = UDim2.new(0.5, -130, 0.02, 0)})
        local tweenText = TweenService:Create(Text, tweenInfo, {TextTransparency = 1})
        local tweenStroke = TweenService:Create(Stroke, tweenInfo, {Transparency = 1})
        
        tweenFrame:Play()
        tweenText:Play()
        tweenStroke:Play()
        
        tweenFrame.Completed:Connect(function()
            WelcomeGui:Destroy()
        end)
    end)
end

playWelcomeSoundAndNotify()

-- =========================================================================
-- VARIABLES GLOBAL Y CONFIGURACIÓN
-- =========================================================================
local aimbotEnabled = false
local targetPartName = "Head"
local isRightClicking = false
local mobileAimingToggle = false

local espNameEnabled = false
local espDistEnabled = false
local espBodyEnabled = false
local espHealthBarEnabled = false
local minimapEnabled = false

local maxEspDistance = 150
local MAX_SLIDER_DIST = 460

local connections = {}

-- Eventos Mouse PC
table.insert(connections, UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.UserInputType == Enum.UserInputType.MouseButton2 then
        isRightClicking = true
    end
end))

table.insert(connections, UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        isRightClicking = false
    end
end))

local function getMyRoot()
    local char = player.Character
    if char then
        return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Head")
    end
    return nil
end

local function isPlayerValid(targetPlayer, customMaxDist)
    if not targetPlayer or targetPlayer == player then return false end
    local char = targetPlayer.Character
    if not char then return false end

    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return false end

    local targetPart = char:FindFirstChild(targetPartName) or char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
    if not targetPart then return false end

    local myRoot = getMyRoot()
    local myPos = myRoot and myRoot.Position or camera.CFrame.Position
    local dist = (targetPart.Position - myPos).Magnitude

    local limit = customMaxDist or 150
    if dist > limit then return false end

    return true, targetPart, humanoid, dist
end

local function fullCleanup()
    aimbotEnabled = false
    espContainer:ClearAllChildren()
    for _, conn in pairs(connections) do
        pcall(function() if conn then conn:Disconnect() end end)
    end
    destruirTodo()
end

-- =========================================================================
-- INTERFAZ GRÁFICA PRINCIPAL
-- =========================================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ElianHubMain"
ScreenGui.Parent = parentGui
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true

local menuWidth = isMobile and 340 or 420
local menuHeight = isMobile and 280 or 320

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name = "MainFrame"
MainFrame.BackgroundColor3 = Color3.fromRGB(8, 9, 13)
MainFrame.Position = UDim2.new(0.5, -menuWidth/2, 0.4, -menuHeight/2)
MainFrame.Size = UDim2.new(0, menuWidth, 0, menuHeight)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 2
MainStroke.Color = Color3.fromRGB(0, 230, 255)

-- TOPBAR
local TopBar = Instance.new("Frame", MainFrame)
TopBar.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
TopBar.Size = UDim2.new(1, 0, 0, 42)
TopBar.BorderSizePixel = 0

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0.04, 0, 0, 0)
Title.Size = UDim2.new(0.52, 0, 1, 0)
Title.Font = Enum.Font.GothamBold
Title.Text = "💎 ELIAN HUB (" .. (isMobile and "📱 Móvil" or "🪟 PC") .. ")"
Title.TextColor3 = Color3.fromRGB(240, 243, 255)
Title.TextSize = isMobile and 11 or 12
Title.TextXAlignment = Enum.TextXAlignment.Left

-- BOTÓN DISCORD
local DiscordBtn = Instance.new("ImageButton", TopBar)
DiscordBtn.Name = "DiscordBtn"
DiscordBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
DiscordBtn.Position = UDim2.new(0.60, 0, 0.16, 0)
DiscordBtn.Size = UDim2.new(0, 32, 0, 28)
DiscordBtn.Image = "rbxassetid://6031075938"
DiscordBtn.ImageColor3 = Color3.fromRGB(255, 255, 255)
DiscordBtn.BorderSizePixel = 0
Instance.new("UICorner", DiscordBtn).CornerRadius = UDim.new(0, 6)

DiscordBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        setClipboardLink(DISCORD_LINK)
        showNotification("📋 ¡Discord copiado!")
    end
end)

-- Botón Minimizar (−)
local MinimizeBtn = Instance.new("TextButton", TopBar)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(30, 36, 50)
MinimizeBtn.Position = UDim2.new(0.74, 0, 0.16, 0)
MinimizeBtn.Size = UDim2.new(0, 32, 0, 28)
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Text = "−"
MinimizeBtn.TextColor3 = Color3.fromRGB(0, 230, 255)
MinimizeBtn.TextSize = 16
MinimizeBtn.BorderSizePixel = 0
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(0, 6)

-- Botón Cerrar (✕)
local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.BackgroundColor3 = Color3.fromRGB(235, 45, 80)
CloseBtn.Position = UDim2.new(0.87, 0, 0.16, 0)
CloseBtn.Size = UDim2.new(0, 32, 0, 28)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

CloseBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        fullCleanup()
    end
end)

-- =========================================================================
-- BOTÓN FLOTANTE (ABRIR / CERRAR MENÚ)
-- =========================================================================
local OpenGui = Instance.new("ScreenGui", parentGui)
OpenGui.Name = "ElianHubOpenBtn"
OpenGui.ResetOnSpawn = false

local ToggleMenuBtn = Instance.new("TextButton", OpenGui)
ToggleMenuBtn.Name = "ToggleMenuBtn"
ToggleMenuBtn.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
ToggleMenuBtn.Position = UDim2.new(0.02, 0, 0.15, 0)
ToggleMenuBtn.Size = UDim2.new(0, isMobile and 120 or 110, 0, isMobile and 42 or 36)
ToggleMenuBtn.Font = Enum.Font.GothamBold
ToggleMenuBtn.Text = "⚡ ELIAN MENU"
ToggleMenuBtn.TextColor3 = Color3.fromRGB(0, 230, 255)
ToggleMenuBtn.TextSize = isMobile and 11 or 10
ToggleMenuBtn.Active = true
ToggleMenuBtn.Draggable = true

Instance.new("UICorner", ToggleMenuBtn).CornerRadius = UDim.new(0, 8)
local TS = Instance.new("UIStroke", ToggleMenuBtn)
TS.Color = Color3.fromRGB(0, 230, 255)
TS.Thickness = 2

local function toggleMainFrame()
    MainFrame.Visible = not MainFrame.Visible
end

ToggleMenuBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        toggleMainFrame()
    end
end)

MinimizeBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        toggleMainFrame()
    end
end)

-- BOTÓN MÓVIL AIM
if isMobile then
    local AimGui = Instance.new("ScreenGui", parentGui)
    AimGui.Name = "ElianMobileAimGui"
    AimGui.ResetOnSpawn = false

    local MobileAimBtn = Instance.new("TextButton", AimGui)
    MobileAimBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
    MobileAimBtn.Position = UDim2.new(0.78, 0, 0.45, 0)
    MobileAimBtn.Size = UDim2.new(0, 65, 0, 65)
    MobileAimBtn.Font = Enum.Font.GothamBold
    MobileAimBtn.Text = "AIM: OFF"
    MobileAimBtn.TextColor3 = Color3.fromRGB(140, 150, 180)
    MobileAimBtn.TextSize = 10
    MobileAimBtn.Active = true
    MobileAimBtn.Draggable = true
    MobileAimBtn.ZIndex = 10

    Instance.new("UICorner", MobileAimBtn).CornerRadius = UDim.new(1, 0)
    local MS = Instance.new("UIStroke", MobileAimBtn)
    MS.Color = Color3.fromRGB(0, 230, 255)
    MS.Thickness = 2

    MobileAimBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            mobileAimingToggle = not mobileAimingToggle
            MobileAimBtn.Text = mobileAimingToggle and "AIM: ON" or "AIM: OFF"
            MobileAimBtn.TextColor3 = mobileAimingToggle and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(140, 150, 180)
            MobileAimBtn.BackgroundColor3 = mobileAimingToggle and Color3.fromRGB(0, 60, 80) or Color3.fromRGB(12, 14, 20)
        end
    end)
end

-- RADAR / MINIMAPA (SOLO PC)
local MinimapFrame, RadarArea
local blips = {}

if not isMobile then
    local MinimapGui = Instance.new("ScreenGui")
    MinimapGui.Name = "ElianMinimapGui"
    MinimapGui.Parent = parentGui
    MinimapGui.ResetOnSpawn = false

    MinimapFrame = Instance.new("Frame", MinimapGui)
    MinimapFrame.Name = "MinimapFrame"
    MinimapFrame.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
    MinimapFrame.Position = UDim2.new(0.82, 0, 0.05, 0)
    MinimapFrame.Size = UDim2.new(0, 140, 0, 140)
    MinimapFrame.Active = true
    MinimapFrame.Draggable = true
    MinimapFrame.Visible = false

    Instance.new("UICorner", MinimapFrame).CornerRadius = UDim.new(1, 0)
    local MStroke = Instance.new("UIStroke", MinimapFrame)
    MStroke.Color = Color3.fromRGB(0, 230, 255)
    MStroke.Thickness = 2

    RadarArea = Instance.new("Frame", MinimapFrame)
    RadarArea.BackgroundTransparency = 1
    RadarArea.Size = UDim2.new(1, 0, 1, 0)

    local CenterDot = Instance.new("Frame", MinimapFrame)
    CenterDot.BackgroundColor3 = Color3.fromRGB(0, 230, 255)
    CenterDot.Position = UDim2.new(0.5, -4, 0.5, -4)
    CenterDot.Size = UDim2.new(0, 8, 0, 8)
    CenterDot.ZIndex = 5
    Instance.new("UICorner", CenterDot).CornerRadius = UDim.new(1, 0)
end

-- PESTAÑAS
local Body = Instance.new("Frame", MainFrame)
Body.BackgroundTransparency = 1
Body.Position = UDim2.new(0, 0, 0, 42)
Body.Size = UDim2.new(1, 0, 1, -42)

local TabContainer = Instance.new("Frame", Body)
TabContainer.BackgroundColor3 = Color3.fromRGB(10, 12, 17)
TabContainer.Position = UDim2.new(0.02, 0, 0.04, 0)
TabContainer.Size = UDim2.new(0, isMobile and 95 or 90, 0, menuHeight - 55)
Instance.new("UICorner", TabContainer).CornerRadius = UDim.new(0, 8)

local PagesContainer = Instance.new("Frame", Body)
PagesContainer.Position = UDim2.new(0, isMobile and 105 or 100, 0.04, 0)
PagesContainer.Size = UDim2.new(0, menuWidth - (isMobile and 112 or 110), 0, menuHeight - 55)
PagesContainer.BackgroundTransparency = 1

local function createPage()
    local page = Instance.new("ScrollingFrame", PagesContainer)
    page.BackgroundTransparency = 1
    page.Size = UDim2.new(1, 0, 1, 0)
    page.CanvasSize = UDim2.new(0, 0, 1.5, 0)
    page.ScrollBarThickness = 2
    page.Active = true
    return page
end

local CombatPage = createPage()
local VisualPage = createPage()
VisualPage.Visible = false

local function createTabBtn(text, posY, active, page)
    local btn = Instance.new("TextButton", TabContainer)
    btn.BackgroundColor3 = active and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(16, 18, 26)
    btn.Position = UDim2.new(0.05, 0, 0, posY)
    btn.Size = UDim2.new(0.90, 0, 0, 35)
    btn.Font = Enum.Font.GothamBold
    btn.Text = text
    btn.TextColor3 = active and Color3.fromRGB(8, 9, 13) or Color3.fromRGB(140, 150, 180)
    btn.TextSize = 10
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    btn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            CombatPage.Visible = false
            VisualPage.Visible = false
            page.Visible = true

            for _, child in ipairs(TabContainer:GetChildren()) do
                if child:IsA("TextButton") then
                    child.BackgroundColor3 = Color3.fromRGB(16, 18, 26)
                    child.TextColor3 = Color3.fromRGB(140, 150, 180)
                end
            end
            btn.BackgroundColor3 = Color3.fromRGB(0, 230, 255)
            btn.TextColor3 = Color3.fromRGB(8, 9, 13)
        end
    end)
end

createTabBtn("⚔ Combat", 8, true, CombatPage)
createTabBtn("👁 Visuals", 48, false, VisualPage)

-- CREADOR DE INTERRUPTORES
local function createToggle(parent, name, posY, initialVal, callback)
    local btn = Instance.new("TextButton", parent)
    btn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
    btn.Position = UDim2.new(0.01, 0, 0, posY)
    btn.Size = UDim2.new(0.97, 0, 0, 35)
    btn.Font = Enum.Font.GothamMedium
    btn.Text = "   " .. name
    btn.TextColor3 = Color3.fromRGB(210, 220, 240)
    btn.TextSize = 10
    btn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    local indicator = Instance.new("Frame", btn)
    indicator.BackgroundColor3 = initialVal and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(30, 34, 48)
    indicator.Position = UDim2.new(0.78, 0, 0.20, 0)
    indicator.Size = UDim2.new(0, 32, 0, 20)
    Instance.new("UICorner", indicator).CornerRadius = UDim.new(0, 10)

    local dot = Instance.new("Frame", indicator)
    dot.BackgroundColor3 = initialVal and Color3.fromRGB(8, 9, 13) or Color3.fromRGB(140, 150, 180)
    dot.Position = initialVal and UDim2.new(0.5, 0, 0.12, 0) or UDim2.new(0.08, 0, 0.12, 0)
    dot.Size = UDim2.new(0, 15, 0, 15)
    Instance.new("UICorner", dot).CornerRadius = UDim.new(0, 8)

    local enabled = initialVal
    btn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            enabled = not enabled
            indicator.BackgroundColor3 = enabled and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(30, 34, 48)
            dot.BackgroundColor3 = enabled and Color3.fromRGB(8, 9, 13) or Color3.fromRGB(140, 150, 180)
            dot.Position = enabled and UDim2.new(0.5, 0, 0.12, 0) or UDim2.new(0.08, 0, 0.12, 0)
            callback(enabled)
        end
    end)
end

-- CREADOR DE SLIDER DE DISTANCIA
local function createSlider(parent, name, posY, minVal, maxVal, defaultVal, callback)
    local frame = Instance.new("Frame", parent)
    frame.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
    frame.Position = UDim2.new(0.01, 0, 0, posY)
    frame.Size = UDim2.new(0.97, 0, 0, 48)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local label = Instance.new("TextLabel", frame)
    label.BackgroundTransparency = 1
    label.Position = UDim2.new(0.04, 0, 0.1, 0)
    label.Size = UDim2.new(0.92, 0, 0, 18)
    label.Font = Enum.Font.GothamMedium
    label.Text = name .. ": " .. defaultVal .. "m"
    label.TextColor3 = Color3.fromRGB(0, 230, 255)
    label.TextSize = 10
    label.TextXAlignment = Enum.TextXAlignment.Left

    local track = Instance.new("Frame", frame)
    track.BackgroundColor3 = Color3.fromRGB(25, 30, 42)
    track.Position = UDim2.new(0.04, 0, 0.62, 0)
    track.Size = UDim2.new(0.92, 0, 0, 10)
    Instance.new("UICorner", track).CornerRadius = UDim.new(0, 5)

    local fill = Instance.new("Frame", track)
    fill.BackgroundColor3 = Color3.fromRGB(0, 230, 255)
    local pctInit = (defaultVal - minVal) / (maxVal - minVal)
    fill.Size = UDim2.new(pctInit, 0, 1, 0)
    Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 5)

    local dragging = false
    local function update(input)
        local pos = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local val = math.floor(minVal + (maxVal - minVal) * pos)
        label.Text = name .. ": " .. val .. "m"
        callback(val)
    end

    track.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            update(input)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
            update(input)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
end

-- CONTROLES COMBAT
createToggle(CombatPage, isMobile and "Activar Aimbot Móvil" or "Aimbot (Click Dcho)", 5, aimbotEnabled, function(st) aimbotEnabled = st end)

local TargetBtn = Instance.new("TextButton", CombatPage)
TargetBtn.BackgroundColor3 = Color3.fromRGB(16, 20, 28)
TargetBtn.Position = UDim2.new(0.01, 0, 0, 45)
TargetBtn.Size = UDim2.new(0.97, 0, 0, 35)
TargetBtn.Font = Enum.Font.GothamBold
TargetBtn.Text = "🎯 Apuntar a: CABEZA"
TargetBtn.TextColor3 = Color3.fromRGB(0, 230, 255)
TargetBtn.TextSize = 10
Instance.new("UICorner", TargetBtn).CornerRadius = UDim.new(0, 6)

TargetBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        if targetPartName == "Head" then
            targetPartName = "HumanoidRootPart"
            TargetBtn.Text = "🎯 Apuntar a: PECHO"
            TargetBtn.TextColor3 = Color3.fromRGB(255, 180, 0)
        else
            targetPartName = "Head"
            TargetBtn.Text = "🎯 Apuntar a: CABEZA"
            TargetBtn.TextColor3 = Color3.fromRGB(0, 230, 255)
        end
    end
end)

-- CONTROLES VISUALS
createToggle(VisualPage, "ESP Nombre Jugador", 5, espNameEnabled, function(st) espNameEnabled = st end)
createToggle(VisualPage, "ESP Ver Distancia", 45, espDistEnabled, function(st) espDistEnabled = st end)
createToggle(VisualPage, "ESP Barra de Vida (Grande)", 85, espHealthBarEnabled, function(st) espHealthBarEnabled = st end)
createToggle(VisualPage, "ESP Cuerpo (Chams)", 125, espBodyEnabled, function(st) espBodyEnabled = st end)

-- Slider de Distancia ESP
createSlider(VisualPage, "Rango Distancia ESP", 165, 20, MAX_SLIDER_DIST, maxEspDistance, function(val)
    maxEspDistance = val
end)

if not isMobile then
    createToggle(VisualPage, "Minimapa Radar (PC)", 220, minimapEnabled, function(st)
        minimapEnabled = st
        if MinimapFrame then MinimapFrame.Visible = st end
    end)
end

-- =========================================================================
-- BUCLE PRINCIPAL (RENDERSTEPPED) - ESP & AIMBOT
-- =========================================================================
table.insert(connections, RunService.RenderStepped:Connect(function()
    camera = workspace.CurrentCamera
    local myRoot = getMyRoot()

    -- AIMBOT (SIN MODIFICACIONES)
    local shouldAim = aimbotEnabled and ((not isMobile and isRightClicking) or (isMobile and mobileAimingToggle))
    if shouldAim then
        local closestPart = nil
        local shortestDist = 99999
        local centerPos = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            local isValid, targetPart = isPlayerValid(otherPlayer, maxEspDistance)
            if isValid and targetPart then
                local screenPos, onScreen = camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - centerPos).Magnitude
                    if dist < shortestDist then
                        shortestDist = dist
                        closestPart = targetPart
                    end
                end
            end
        end

        if closestPart then
            camera.CFrame = CFrame.new(camera.CFrame.Position, closestPart.Position)
        end
    end

    -- MINIMAPA (PC)
    if not isMobile and minimapEnabled and myRoot then
        local myPos = myRoot.Position

        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            local isValid, targetPart = isPlayerValid(otherPlayer, maxEspDistance)
            local blip = blips[otherPlayer]

            if isValid and targetPart then
                if not blip then
                    blip = Instance.new("Frame", RadarArea)
                    blip.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
                    blip.Size = UDim2.new(0, 6, 0, 6)
                    Instance.new("UICorner", blip).CornerRadius = UDim.new(1, 0)
                    blips[otherPlayer] = blip
                end

                local relPos = targetPart.Position - myPos
                local scale = 65 / maxEspDistance
                local mapX = math.clamp(relPos.X * scale, -60, 60)
                local mapZ = math.clamp(relPos.Z * scale, -60, 60)

                blip.Position = UDim2.new(0.5, mapX - 3, 0.5, mapZ - 3)
                blip.Visible = true
            else
                if blip then blip.Visible = false end
            end
        end
    end

    -- ESP VISUALES (CON BARRA DE VIDA AGRANDADA)
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player then
            local char = otherPlayer.Character
            local isValid, targetPart, humanoid, dist = isPlayerValid(otherPlayer, maxEspDistance)

            local pName = otherPlayer.Name
            local hl = espContainer:FindFirstChild("HL_" .. pName)
            local tag = espContainer:FindFirstChild("TAG_" .. pName)
            local sideBar = espContainer:FindFirstChild("HP_" .. pName)

            if char and isValid and targetPart and humanoid then
                local healthPct = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                local healthColor = healthPct > 0.6 and Color3.fromRGB(0, 255, 120) or (healthPct > 0.3 and Color3.fromRGB(255, 200, 0) or Color3.fromRGB(255, 50, 50))

                -- 1. CHAMS (CUERPO)
                if espBodyEnabled then
                    if not hl then
                        hl = Instance.new("Highlight")
                        hl.Name = "HL_" .. pName
                        hl.FillColor = Color3.fromRGB(0, 230, 255)
                        hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                        hl.FillTransparency = 0.4
                        hl.OutlineTransparency = 0
                        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                        hl.Parent = espContainer
                    end
                    hl.Adornee = char
                    hl.Enabled = true
                elseif hl then
                    hl.Enabled = false
                end

                -- 2. NOMBRE Y DISTANCIA
                if espNameEnabled or espDistEnabled then
                    if not tag then
                        tag = Instance.new("BillboardGui")
                        tag.Name = "TAG_" .. pName
                        tag.AlwaysOnTop = true
                        tag.Size = UDim2.new(0, 180, 0, 25)
                        tag.StudsOffset = Vector3.new(0, 3.2, 0)
                        
                        local txt = Instance.new("TextLabel", tag)
                        txt.Name = "Text"
                        txt.BackgroundTransparency = 1
                        txt.Size = UDim2.new(1, 0, 1, 0)
                        txt.Font = Enum.Font.GothamBold
                        txt.TextSize = 12
                        txt.TextColor3 = Color3.fromRGB(0, 230, 255)
                        txt.TextStrokeTransparency = 0

                        tag.Parent = espContainer
                    end
                    tag.Adornee = targetPart

                    local txtLabel = tag:FindFirstChild("Text")
                    if txtLabel then
                        local str = ""
                        if espNameEnabled then str = otherPlayer.Name end
                        if espDistEnabled then 
                            str = (str ~= "" and str .. " " or "") .. "[" .. math.floor(dist) .. "m]"
                        end
                        txtLabel.Text = str
                    end
                    tag.Enabled = true
                elseif tag then
                    tag.Enabled = false
                end

                -- 3. BARRA DE VIDA VERTICAL AGRANDADA (ALTO Y ANCHO MULTIPLICADOS)
                if espHealthBarEnabled then
                    if not sideBar then
                        sideBar = Instance.new("BillboardGui")
                        sideBar.Name = "HP_" .. pName
                        sideBar.AlwaysOnTop = true
                        sideBar.Size = UDim2.new(0, 10, 0, 50) -- Ancho subió de 5 a 10, alto de 35 a 50
                        sideBar.StudsOffset = Vector3.new(-2.6, 0, 0)

                        local bgBar = Instance.new("Frame", sideBar)
                        bgBar.Name = "BG"
                        bgBar.BackgroundColor3 = Color3.fromRGB(12, 14, 18)
                        bgBar.Size = UDim2.new(1, 0, 1, 0)
                        bgBar.BorderSizePixel = 0
                        Instance.new("UICorner", bgBar).CornerRadius = UDim.new(0, 4)

                        local stroke = Instance.new("UIStroke", bgBar)
                        stroke.Color = Color3.fromRGB(0, 0, 0)
                        stroke.Thickness = 1.5

                        local fillBar = Instance.new("Frame", bgBar)
                        fillBar.Name = "Fill"
                        fillBar.BackgroundColor3 = Color3.fromRGB(0, 255, 120)
                        fillBar.AnchorPoint = Vector2.new(0, 1)
                        fillBar.Position = UDim2.new(0, 0, 1, 0)
                        fillBar.Size = UDim2.new(1, 0, 1, 0)
                        fillBar.BorderSizePixel = 0
                        Instance.new("UICorner", fillBar).CornerRadius = UDim.new(0, 4)

                        -- Texto Porcentaje de vida arriba de la barra
                        local hpTxt = Instance.new("TextLabel", sideBar)
                        hpTxt.Name = "HpText"
                        hpTxt.BackgroundTransparency = 1
                        hpTxt.Position = UDim2.new(0.5, -20, 0, -14)
                        hpTxt.Size = UDim2.new(0, 40, 0, 12)
                        hpTxt.Font = Enum.Font.GothamBold
                        hpTxt.TextSize = 9
                        hpTxt.TextColor3 = Color3.fromRGB(255, 255, 255)
                        hpTxt.TextStrokeTransparency = 0

                        sideBar.Parent = espContainer
                    end
                    sideBar.Adornee = targetPart

                    local bgBar = sideBar:FindFirstChild("BG")
                    local hpTxt = sideBar:FindFirstChild("HpText")

                    if bgBar then
                        local fillBar = bgBar:FindFirstChild("Fill")
                        if fillBar then
                            fillBar.Size = UDim2.new(1, 0, healthPct, 0)
                            fillBar.BackgroundColor3 = healthColor
                        end
                    end

                    if hpTxt then
                        hpTxt.Text = math.floor(healthPct * 100) .. "%"
                        hpTxt.TextColor3 = healthColor
                    end

                    sideBar.Enabled = true
                elseif sideBar then
                    sideBar.Enabled = false
                end

            else
                if hl then hl.Enabled = false end
                if tag then tag.Enabled = false end
                if sideBar then sideBar.Enabled = false end
            end
        end
    end
end))
