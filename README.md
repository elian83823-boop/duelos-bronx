-- =========================================================================
-- ELIAN HUB - MOBILE OPTIMIZED AIMBOT (SIN TEAM CHECK / SIN FILTROS)
-- =========================================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local function getValidParent()
    local success, core = pcall(function() return CoreGui end)
    if success and core then return core end
    return player:WaitForChild("PlayerGui", 5) or player.PlayerGui
end

local parentGui = getValidParent()

-- Limpieza de ejecuciones previas
local function destruirTodo()
    pcall(function()
        for _, guiName in ipairs({"ElianHubMain", "ElianMobileAimGui", "ElianHubOpenBtn", "ElianPlatformSelector"}) do
            local old = parentGui:FindFirstChild(guiName)
            if old then old:Destroy() end
        end
    end)
end
destruirTodo()

-- =========================================================================
-- INICIALIZACIÓN SEGÚN PLATAFORMA
-- =========================================================================
local function initHub(isMobile)
    local aimbotEnabled = false
    local smoothness = isMobile and 0.25 or 1.0 -- Suavizado especial para táctil
    local aimDistance = 2000

    local isRightClicking = false
    local mobileAimingToggle = false

    local espNameEnabled = false
    local espBodyEnabled = false

    local connections = {}

    -- Controles PC (Mouse)
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

    -- Validación Ultra Rápida (Cualquier jugador vivo que no seas tú)
    local function isPlayerValid(targetPlayer)
        if not targetPlayer or targetPlayer == player then return false end
        local char = targetPlayer.Character
        if not char then return false end

        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if not humanoid or humanoid.Health <= 0 then return false end

        local targetPart = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
        if not targetPart then return false end

        return true, targetPart
    end

    local function fullCleanup()
        aimbotEnabled = false
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer.Character then
                local hl = otherPlayer.Character:FindFirstChild("ElianHighlight")
                if hl then hl:Destroy() end
                local bb = otherPlayer.Character:FindFirstChild("ElianNameTag")
                if bb then bb:Destroy() end
            end
        end
        for _, conn in pairs(connections) do
            pcall(function() if conn then conn:Disconnect() end end)
        end
        destruirTodo()
    end

    -- Interfaz Principal
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ElianHubMain"
    ScreenGui.Parent = parentGui
    ScreenGui.ResetOnSpawn = false
    ScreenGui.IgnoreGuiInset = true

    local menuWidth = isMobile and 340 or 420
    local menuHeight = isMobile and 260 or 300

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

    -- TopBar
    local TopBar = Instance.new("Frame", MainFrame)
    TopBar.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
    TopBar.Size = UDim2.new(1, 0, 0, 40)
    TopBar.BorderSizePixel = 0

    local Title = Instance.new("TextLabel", TopBar)
    Title.BackgroundTransparency = 1
    Title.Position = UDim2.new(0.04, 0, 0, 0)
    Title.Size = UDim2.new(0.7, 0, 1, 0)
    Title.Font = Enum.Font.GothamBold
    Title.Text = "💎 ELIAN HUB (" .. (isMobile and "MÓVIL 📱" or "PC 💻") .. ")"
    Title.TextColor3 = Color3.fromRGB(240, 243, 255)
    Title.TextSize = 11
    Title.TextXAlignment = Enum.TextXAlignment.Left

    local CloseBtn = Instance.new("TextButton", TopBar)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(235, 45, 80)
    CloseBtn.Position = UDim2.new(0.88, 0, 0.18, 0)
    CloseBtn.Size = UDim2.new(0, 28, 0, 25)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.Text = "✕"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.TextSize = 11
    CloseBtn.BorderSizePixel = 0

    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
    CloseBtn.MouseButton1Click:Connect(fullCleanup)
    CloseBtn.TouchTap:Connect(fullCleanup)

    -- Controles Táctiles para Móvil
    if isMobile then
        -- Botón Abrir / Cerrar Menú
        local OpenGui = Instance.new("ScreenGui", parentGui)
        OpenGui.Name = "ElianHubOpenBtn"
        OpenGui.ResetOnSpawn = false

        local ToggleMenuBtn = Instance.new("TextButton", OpenGui)
        ToggleMenuBtn.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
        ToggleMenuBtn.Position = UDim2.new(0.02, 0, 0.20, 0)
        ToggleMenuBtn.Size = UDim2.new(0, 110, 0, 36)
        ToggleMenuBtn.Font = Enum.Font.GothamBold
        ToggleMenuBtn.Text = "⚡ ELIAN MENU"
        ToggleMenuBtn.TextColor3 = Color3.fromRGB(0, 230, 255)
        ToggleMenuBtn.TextSize = 10
        ToggleMenuBtn.Active = true
        ToggleMenuBtn.Draggable = true

        Instance.new("UICorner", ToggleMenuBtn).CornerRadius = UDim.new(0, 8)
        local TS = Instance.new("UIStroke", ToggleMenuBtn)
        TS.Color = Color3.fromRGB(0, 230, 255)
        TS.Thickness = 1.5

        local function toggleVis() MainFrame.Visible = not MainFrame.Visible end
        ToggleMenuBtn.MouseButton1Click:Connect(toggleVis)
        ToggleMenuBtn.TouchTap:Connect(toggleVis)

        -- Botón Flotante para Fijar Apuntado (AIM)
        local AimGui = Instance.new("ScreenGui", parentGui)
        AimGui.Name = "ElianMobileAimGui"
        AimGui.ResetOnSpawn = false

        local MobileAimBtn = Instance.new("TextButton", AimGui)
        MobileAimBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
        MobileAimBtn.Position = UDim2.new(0.78, 0, 0.50, 0)
        MobileAimBtn.Size = UDim2.new(0, 65, 0, 65)
        MobileAimBtn.Font = Enum.Font.GothamBold
        MobileAimBtn.Text = "AIM: OFF"
        MobileAimBtn.TextColor3 = Color3.fromRGB(140, 150, 180)
        MobileAimBtn.TextSize = 10
        MobileAimBtn.Active = true
        MobileAimBtn.Draggable = true

        Instance.new("UICorner", MobileAimBtn).CornerRadius = UDim.new(1, 0)
        local MS = Instance.new("UIStroke", MobileAimBtn)
        MS.Color = Color3.fromRGB(0, 230, 255)
        MS.Thickness = 2

        local function toggleAimMobile()
            mobileAimingToggle = not mobileAimingToggle
            MobileAimBtn.Text = mobileAimingToggle and "AIM: ON" or "AIM: OFF"
            MobileAimBtn.TextColor3 = mobileAimingToggle and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(140, 150, 180)
            MobileAimBtn.BackgroundColor3 = mobileAimingToggle and Color3.fromRGB(0, 60, 80) or Color3.fromRGB(12, 14, 20)
        end
        MobileAimBtn.MouseButton1Click:Connect(toggleAimMobile)
        MobileAimBtn.TouchTap:Connect(toggleAimMobile)
    end

    -- Pestañas
    local Body = Instance.new("Frame", MainFrame)
    Body.BackgroundTransparency = 1
    Body.Position = UDim2.new(0, 0, 0, 40)
    Body.Size = UDim2.new(1, 0, 1, -40)

    local TabContainer = Instance.new("Frame", Body)
    TabContainer.BackgroundColor3 = Color3.fromRGB(10, 12, 17)
    TabContainer.Position = UDim2.new(0.02, 0, 0.04, 0)
    TabContainer.Size = UDim2.new(0, 95, 0, menuHeight - 55)
    Instance.new("UICorner", TabContainer).CornerRadius = UDim.new(0, 8)

    local PagesContainer = Instance.new("Frame", Body)
    PagesContainer.Position = UDim2.new(0, 110, 0.04, 0)
    PagesContainer.Size = UDim2.new(0, menuWidth - 120, 0, menuHeight - 55)
    PagesContainer.BackgroundTransparency = 1

    local function createPage()
        local page = Instance.new("ScrollingFrame", PagesContainer)
        page.BackgroundTransparency = 1
        page.Size = UDim2.new(1, 0, 1, 0)
        page.CanvasSize = UDim2.new(0, 0, 1.1, 0)
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
        btn.Position = UDim2.new(0.06, 0, 0, posY)
        btn.Size = UDim2.new(0.88, 0, 0, 32)
        btn.Font = Enum.Font.GothamBold
        btn.Text = text
        btn.TextColor3 = active and Color3.fromRGB(8, 9, 13) or Color3.fromRGB(140, 150, 180)
        btn.TextSize = 10
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

        btn.MouseButton1Click:Connect(function()
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
        end)
    end

    createTabBtn("⚔ Combat", 8, true, CombatPage)
    createTabBtn("👁 Visuals", 45, false, VisualPage)

    -- Creador de Toggles
    local function createToggle(parent, name, posY, initialVal, callback)
        local btn = Instance.new("TextButton", parent)
        btn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
        btn.Position = UDim2.new(0.01, 0, 0, posY)
        btn.Size = UDim2.new(0.97, 0, 0, 34)
        btn.Font = Enum.Font.GothamMedium
        btn.Text = "   " .. name
        btn.TextColor3 = Color3.fromRGB(210, 220, 240)
        btn.TextSize = 10
        btn.TextXAlignment = Enum.TextXAlignment.Left
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

        local indicator = Instance.new("Frame", btn)
        indicator.BackgroundColor3 = initialVal and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(30, 34, 48)
        indicator.Position = UDim2.new(0.80, 0, 0.22, 0)
        indicator.Size = UDim2.new(0, 28, 0, 18)
        Instance.new("UICorner", indicator).CornerRadius = UDim.new(0, 9)

        local dot = Instance.new("Frame", indicator)
        dot.BackgroundColor3 = initialVal and Color3.fromRGB(8, 9, 13) or Color3.fromRGB(140, 150, 180)
        dot.Position = initialVal and UDim2.new(0.5, 0, 0.12, 0) or UDim2.new(0.08, 0, 0.12, 0)
        dot.Size = UDim2.new(0, 13, 0, 13)
        Instance.new("UICorner", dot).CornerRadius = UDim.new(0, 7)

        local enabled = initialVal
        local function toggle()
            enabled = not enabled
            indicator.BackgroundColor3 = enabled and Color3.fromRGB(0, 230, 255) or Color3.fromRGB(30, 34, 48)
            dot.BackgroundColor3 = enabled and Color3.fromRGB(8, 9, 13) or Color3.fromRGB(140, 150, 180)
            dot.Position = enabled and UDim2.new(0.5, 0, 0.12, 0) or UDim2.new(0.08, 0, 0.12, 0)
            callback(enabled)
        end

        btn.MouseButton1Click:Connect(toggle)
        btn.TouchTap:Connect(toggle)
    end

    -- Opciones COMBAT
    createToggle(CombatPage, isMobile and "Activar Aimbot Móvil" or "Head Aimbot (Click Dcho)", 5, aimbotEnabled, function(st) aimbotEnabled = st end)

    -- Opciones VISUALS
    createToggle(VisualPage, "ESP Name (Nombre + Distancia)", 5, espNameEnabled, function(st) espNameEnabled = st end)
    createToggle(VisualPage, "ESP Body (Resaltado Total)", 45, espBodyEnabled, function(st) espBodyEnabled = st end)

    -- RenderStepped Principal
    table.insert(connections, RunService.RenderStepped:Connect(function()
        -- Lógica de Apuntado
        local shouldAim = aimbotEnabled and ((not isMobile and isRightClicking) or (isMobile and mobileAimingToggle))
        if shouldAim then
            local closestPart = nil
            local shortestDist = aimDistance
            local centerPos = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

            for _, otherPlayer in ipairs(Players:GetPlayers()) do
                local isValid, targetPart = isPlayerValid(otherPlayer)
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
                local targetCFrame = CFrame.new(camera.CFrame.Position, closestPart.Position)
                if isMobile then
                    -- Transición suave en móvil para no perder el control táctil
                    camera.CFrame = camera.CFrame:Lerp(targetCFrame, smoothness)
                else
                    camera.CFrame = targetCFrame
                end
            end
        end

        -- Lógica de Visuales (ESP)
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            local isValid, targetPart = isPlayerValid(otherPlayer)
            if otherPlayer ~= player and otherPlayer.Character then
                local char = otherPlayer.Character
                local hl = char:FindFirstChild("ElianHighlight")
                local tag = char:FindFirstChild("ElianNameTag")

                if isValid and targetPart then
                    -- Highlight Body
                    if espBodyEnabled then
                        if not hl then
                            hl = Instance.new("Highlight")
                            hl.Name = "ElianHighlight"
                            hl.FillColor = Color3.fromRGB(0, 230, 255)
                            hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                            hl.FillTransparency = 0.4
                            hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                            hl.Parent = char
                        end
                        hl.Enabled = true
                    elseif hl then
                        hl.Enabled = false
                    end

                    -- Name Tag
                    if espNameEnabled then
                        if not tag then
                            tag = Instance.new("BillboardGui")
                            tag.Name = "ElianNameTag"
                            tag.AlwaysOnTop = true
                            tag.Size = UDim2.new(0, 160, 0, 30)
                            tag.StudsOffset = Vector3.new(0, 3, 0)
                            tag.Adornee = targetPart

                            local txt = Instance.new("TextLabel", tag)
                            txt.Name = "Text"
                            txt.BackgroundTransparency = 1
                            txt.Size = UDim2.new(1, 0, 1, 0)
                            txt.Font = Enum.Font.GothamBold
                            txt.TextSize = 12
                            txt.TextColor3 = Color3.fromRGB(0, 230, 255)
                            txt.TextStrokeTransparency = 0
                            tag.Parent = char
                        end

                        local myHead = player.Character and player.Character:FindFirstChild("Head")
                        local dist = myHead and math.floor((targetPart.Position - myHead.Position).Magnitude) or 0
                        tag.Text.Text = otherPlayer.Name .. " [" .. dist .. "m]"
                        tag.Enabled = true
                    elseif tag then
                        tag.Enabled = false
                    end
                else
                    if hl then hl:Destroy() end
                    if tag then tag:Destroy() end
                end
            end
        end
    end))
end

-- =========================================================================
-- POPUP DE SELECCIÓN DE PLATAFORMA
-- =========================================================================
local PlatformGui = Instance.new("ScreenGui")
PlatformGui.Name = "ElianPlatformSelector"
PlatformGui.Parent = parentGui
PlatformGui.ResetOnSpawn = false

local SelectorFrame = Instance.new("Frame", PlatformGui)
SelectorFrame.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
SelectorFrame.Position = UDim2.new(0.5, -140, 0.4, -75)
SelectorFrame.Size = UDim2.new(0, 280, 0, 150)
SelectorFrame.Active = true
SelectorFrame.Draggable = true

Instance.new("UICorner", SelectorFrame).CornerRadius = UDim.new(0, 10)
local SStroke = Instance.new("UIStroke", SelectorFrame)
SStroke.Color = Color3.fromRGB(0, 230, 255)
SStroke.Thickness = 2

local STitle = Instance.new("TextLabel", SelectorFrame)
STitle.BackgroundTransparency = 1
STitle.Position = UDim2.new(0, 0, 0.1, 0)
STitle.Size = UDim2.new(1, 0, 0, 25)
STitle.Font = Enum.Font.GothamBold
STitle.Text = "💎 ¿EN QUÉ JUEGAS ELIAN?"
STitle.TextColor3 = Color3.fromRGB(240, 245, 255)
STitle.TextSize = 11

local PCBtn = Instance.new("TextButton", SelectorFrame)
PCBtn.BackgroundColor3 = Color3.fromRGB(0, 230, 255)
PCBtn.Position = UDim2.new(0.08, 0, 0.45, 0)
PCBtn.Size = UDim2.new(0.4, 0, 0, 40)
PCBtn.Font = Enum.Font.GothamBold
PCBtn.Text = "💻 PC"
PCBtn.TextColor3 = Color3.fromRGB(8, 9, 13)
PCBtn.TextSize = 12
Instance.new("UICorner", PCBtn).CornerRadius = UDim.new(0, 8)

local MobileBtn = Instance.new("TextButton", SelectorFrame)
MobileBtn.BackgroundColor3 = Color3.fromRGB(20, 24, 36)
MobileBtn.Position = UDim2.new(0.52, 0, 0.45, 0)
MobileBtn.Size = UDim2.new(0.4, 0, 0, 40)
MobileBtn.Font = Enum.Font.GothamBold
MobileBtn.Text = "📱 MÓVIL"
MobileBtn.TextColor3 = Color3.fromRGB(0, 230, 255)
MobileBtn.TextSize = 12
Instance.new("UICorner", MobileBtn).CornerRadius = UDim.new(0, 8)

local MSSelector = Instance.new("UIStroke", MobileBtn)
MSSelector.Color = Color3.fromRGB(0, 230, 255)
MSSelector.Thickness = 1.5

PCBtn.MouseButton1Click:Connect(function()
    PlatformGui:Destroy()
    initHub(false)
end)

MobileBtn.MouseButton1Click:Connect(function()
    PlatformGui:Destroy()
    initHub(true)
end)
MobileBtn.TouchTap:Connect(function()
    PlatformGui:Destroy()
    initHub(true)
end)
