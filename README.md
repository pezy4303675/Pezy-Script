--[[
    Roube um Brainrot - Ultimate Script Edition + DAMAGE BOOST
    Sistema Completo: Bypass + Invisibilidade + Fly + GodMode + Size Changer + Iluminação + Auto Click + ESP + DAMAGE BOOST
    Compatível com: Bloxburg, Roube um Brainrot e outros jogos
    Features: Fly Mode, GodMode Ultra, Modo Muito Pequeno, Luz ao Redor, Auto Click para Armas, ESP com Highlight, DAMAGE MULTIPLIER
]]

repeat task.wait() until game:IsLoaded()
task.wait(3)

-- ==================== ANTI-DETECÇÃO ====================
local function RandomString(length)
    local chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    local result = ""
    for i = 1, length do
        local rand = math.random(1, #chars)
        result = result .. chars:sub(rand, rand)
    end
    return result
end

-- ==================== CORE ====================
local Core = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    TweenService = game:GetService("TweenService"),
    Workspace = game:GetService("Workspace"),
}

Core.Player = Core.Players.LocalPlayer
Core.Character = Core.Player.Character or Core.Player.CharacterAdded:Wait()
Core.Humanoid = Core.Character:WaitForChild("Humanoid", 10)
Core.RootPart = Core.Character:WaitForChild("HumanoidRootPart", 10)

-- ==================== CONFIG ====================
local Config = {
    SpeedBoost = 1.0,
    JumpBoostMultiplier = 5.0,
    MaxVelocity = 80,
    TeleportThreshold = 2,
    UseRandomization = true,
    MenuKey = Enum.KeyCode.RightBracket,
    ClickTeleportMaxDistance = 5000,
    FlySpeed = 100,
    FlyUpKey = Enum.KeyCode.Space,
    FlyDownKey = Enum.KeyCode.LeftShift,
    TinySize = 0.02,
    LightBrightness = 5,
    LightRange = 60,
    AutoClickSpeed = 0.01,
    ESPColor = Color3.fromRGB(255, 0, 0),
    ESPOutlineColor = Color3.fromRGB(255, 255, 255),
    DamageMultiplier = 10, -- Multiplica o dano por 10x
}

-- ==================== STATE ====================
local State = {
    Speed = false,
    Jump = false,
    Noclip = false,
    PropertyBypass = false,
    Invisibility = false,
    ClickTeleport = false,
    GodMode = false,
    Fly = false,
    TinyMode = false,
    Light = false,
    AutoClick = false,
    ESP = false,
    DamageBoost = false,
    OriginalSpeed = 16,
    OriginalJump = 50,
    SavedPosition = nil,
    LastSaveTime = 0,
    InEnemyProperty = false,
}

if Core.Humanoid then
    State.OriginalSpeed = Core.Humanoid.WalkSpeed
    if Core.Humanoid.UseJumpPower then
        State.OriginalJump = Core.Humanoid.JumpPower
    else
        State.OriginalJump = Core.Humanoid.JumpHeight
    end
end

-- ==================== CONNECTIONS ====================
local Connections = {}

-- ==================== FUNÇÕES DE PROTEÇÃO ====================

local function RefreshCharacter()
    Core.Character = Core.Player.Character
    if Core.Character then
        Core.Humanoid = Core.Character:FindFirstChildOfClass("Humanoid")
        Core.RootPart = Core.Character:FindFirstChild("HumanoidRootPart")
        
        if Core.Humanoid then
            State.OriginalSpeed = Core.Humanoid.WalkSpeed
            if Core.Humanoid.UseJumpPower then
                State.OriginalJump = Core.Humanoid.JumpPower
            else
                State.OriginalJump = Core.Humanoid.JumpHeight
            end
        end
    end
end

local function GetRandomVariation()
    if Config.UseRandomization then
        return math.random(85, 115) / 100
    end
    return 1
end

local function CheckEnemyProperty()
    if not Core.RootPart then return false end
    
    local region = workspace:FindFirstChild("Properties")
    if not region then return false end
    
    for _, property in pairs(region:GetChildren()) do
        if property:IsA("Model") and property:FindFirstChild("Owner") then
            local owner = property.Owner.Value
            if owner and owner ~= Core.Player then
                local originPos = property:FindFirstChild("OriginSquare")
                if originPos then
                    local distance = (Core.RootPart.Position - originPos.Position).Magnitude
                    if distance < 100 then
                        return true
                    end
                end
            end
        end
    end
    return false
end

-- ==================== DAMAGE BOOST ====================
local originalDamageScripts = {}

local function ToggleDamageBoost()
    State.DamageBoost = not State.DamageBoost
    
    if State.DamageBoost then
        -- Método 1: Modificar ferramentas (armas)
        local function boostToolDamage(tool)
            if not tool:IsA("Tool") then return end
            
            -- Procura por scripts de dano
            for _, child in pairs(tool:GetDescendants()) do
                if child:IsA("Script") or child:IsA("LocalScript") then
                    -- Procura valores de dano
                    for _, value in pairs(child:GetDescendants()) do
                        if value:IsA("NumberValue") or value:IsA("IntValue") then
                            if value.Name:lower():find("damage") or value.Name:lower():find("dmg") then
                                if not originalDamageScripts[value] then
                                    originalDamageScripts[value] = value.Value
                                end
                                value.Value = value.Value * Config.DamageMultiplier
                            end
                        end
                    end
                end
            end
            
            -- Procura valores de dano diretamente na tool
            for _, value in pairs(tool:GetDescendants()) do
                if value:IsA("NumberValue") or value:IsA("IntValue") then
                    if value.Name:lower():find("damage") or value.Name:lower():find("dmg") then
                        if not originalDamageScripts[value] then
                            originalDamageScripts[value] = value.Value
                        end
                        value.Value = value.Value * Config.DamageMultiplier
                    end
                end
            end
            
            -- Procura configurações da arma
            local config = tool:FindFirstChild("Configuration") or tool:FindFirstChild("Config") or tool:FindFirstChild("Settings")
            if config then
                for _, value in pairs(config:GetChildren()) do
                    if value:IsA("NumberValue") or value:IsA("IntValue") then
                        if value.Name:lower():find("damage") or value.Name:lower():find("dmg") then
                            if not originalDamageScripts[value] then
                                originalDamageScripts[value] = value.Value
                            end
                            value.Value = value.Value * Config.DamageMultiplier
                        end
                    end
                end
            end
        end
        
        -- Aplica boost em todas as ferramentas atuais
        if Core.Character then
            for _, tool in pairs(Core.Character:GetChildren()) do
                boostToolDamage(tool)
            end
        end
        
        if Core.Player.Backpack then
            for _, tool in pairs(Core.Player.Backpack:GetChildren()) do
                boostToolDamage(tool)
            end
        end
        
        -- Monitora novas ferramentas
        Connections.DamageBoostChar = Core.Character.ChildAdded:Connect(function(child)
            if State.DamageBoost then
                task.wait(0.1)
                boostToolDamage(child)
            end
        end)
        
        if Core.Player.Backpack then
            Connections.DamageBoostBackpack = Core.Player.Backpack.ChildAdded:Connect(function(child)
                if State.DamageBoost then
                    task.wait(0.1)
                    boostToolDamage(child)
                end
            end)
        end
        
        -- Método 2: Hook de dano direto (para jogos compatíveis)
        Connections.DamageBoostHook = Core.RunService.Heartbeat:Connect(function()
            if not Core.Character then return end
            
            -- Procura e modifica valores de dano em tempo real
            for _, tool in pairs(Core.Character:GetChildren()) do
                if tool:IsA("Tool") then
                    for _, value in pairs(tool:GetDescendants()) do
                        if (value:IsA("NumberValue") or value:IsA("IntValue")) and 
                           (value.Name:lower():find("damage") or value.Name:lower():find("dmg")) then
                            if value.Value > 0 and value.Value < 1000 then
                                local expected = (originalDamageScripts[value] or value.Value) * Config.DamageMultiplier
                                if math.abs(value.Value - expected) > 1 then
                                    value.Value = expected
                                end
                            end
                        end
                    end
                end
            end
        end)
        
        print("✅ Damage Boost Ativado (x" .. Config.DamageMultiplier .. " dano)")
    else
        -- Desconecta monitores
        if Connections.DamageBoostChar then
            Connections.DamageBoostChar:Disconnect()
            Connections.DamageBoostChar = nil
        end
        if Connections.DamageBoostBackpack then
            Connections.DamageBoostBackpack:Disconnect()
            Connections.DamageBoostBackpack = nil
        end
        if Connections.DamageBoostHook then
            Connections.DamageBoostHook:Disconnect()
            Connections.DamageBoostHook = nil
        end
        
        -- Restaura valores originais
        for value, originalDamage in pairs(originalDamageScripts) do
            if value and value.Parent then
                pcall(function()
                    value.Value = originalDamage
                end)
            end
        end
        originalDamageScripts = {}
        
        print("❌ Damage Boost Desativado")
    end
end

-- ==================== VELOCIDADE ====================
local function ToggleSpeed()
    State.Speed = not State.Speed
    
    if State.Speed then
        if Connections.Speed then Connections.Speed:Disconnect() end
        
        Connections.Speed = Core.RunService.Heartbeat:Connect(function()
            if not Core.Character or not Core.RootPart or not Core.Humanoid then return end
            
            if Core.Humanoid.MoveDirection.Magnitude > 0.3 then
                local variation = GetRandomVariation()
                local boost = Config.SpeedBoost * variation
                local moveVector = Core.Humanoid.MoveDirection * boost
                
                Core.RootPart.CFrame = Core.RootPart.CFrame + moveVector
            end
        end)
        
        print("✅ Velocidade Ativada")
    else
        if Connections.Speed then
            Connections.Speed:Disconnect()
            Connections.Speed = nil
        end
        print("❌ Velocidade Desativada")
    end
end

-- ==================== ESP COM HIGHLIGHT ====================
local ESPHighlights = {}

local function CreateESPForPlayer(player)
    if player == Core.Player then return end
    
    local function addHighlight(character)
        if not character then return end
        
        local existingHighlight = character:FindFirstChildOfClass("Highlight")
        if existingHighlight then return end
        
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESPHighlight"
        highlight.FillColor = Config.ESPColor
        highlight.OutlineColor = Config.ESPOutlineColor
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Adornee = character
        highlight.Parent = character
        
        ESPHighlights[player.UserId] = highlight
    end
    
    if player.Character then
        addHighlight(player.Character)
    end
    
    player.CharacterAdded:Connect(function(character)
        if State.ESP then
            task.wait(0.1)
            addHighlight(character)
        end
    end)
end

local function RemoveESPForPlayer(userId)
    if ESPHighlights[userId] then
        pcall(function()
            ESPHighlights[userId]:Destroy()
        end)
        ESPHighlights[userId] = nil
    end
end

local function ToggleESP()
    State.ESP = not State.ESP
    
    if State.ESP then
        for _, player in pairs(Core.Players:GetPlayers()) do
            CreateESPForPlayer(player)
        end
        
        Connections.ESPPlayerAdded = Core.Players.PlayerAdded:Connect(function(player)
            if State.ESP then
                CreateESPForPlayer(player)
            end
        end)
        
        Connections.ESPPlayerRemoving = Core.Players.PlayerRemoving:Connect(function(player)
            RemoveESPForPlayer(player.UserId)
        end)
        
        print("✅ ESP Ativado (Todos os jogadores visíveis)")
    else
        if Connections.ESPPlayerAdded then
            Connections.ESPPlayerAdded:Disconnect()
            Connections.ESPPlayerAdded = nil
        end
        if Connections.ESPPlayerRemoving then
            Connections.ESPPlayerRemoving:Disconnect()
            Connections.ESPPlayerRemoving = nil
        end
        
        for userId, highlight in pairs(ESPHighlights) do
            pcall(function()
                highlight:Destroy()
            end)
        end
        ESPHighlights = {}
        
        print("❌ ESP Desativado")
    end
end

-- ==================== PULO ALTO ====================
local function ToggleJump()
    State.Jump = not State.Jump
    
    if State.Jump then
        if Connections.Jump then Connections.Jump:Disconnect() end
        
        Connections.Jump = Core.Humanoid.StateChanged:Connect(function(oldState, newState)
            if newState == Enum.HumanoidStateType.Jumping then
                task.wait(0.04)
                
                if Core.RootPart and Core.RootPart.Parent then
                    local baseJump = State.OriginalJump
                    local variation = GetRandomVariation()
                    local jumpBoost = baseJump * Config.JumpBoostMultiplier * variation
                    
                    local bodyVel = Instance.new("BodyVelocity")
                    bodyVel.MaxForce = Vector3.new(0, math.huge, 0)
                    bodyVel.Velocity = Vector3.new(0, jumpBoost * 0.8, 0)
                    bodyVel.Parent = Core.RootPart
                    
                    task.delay(0.2, function()
                        if bodyVel and bodyVel.Parent then
                            bodyVel:Destroy()
                        end
                    end)
                    
                    task.wait(0.05)
                    if Core.RootPart and Core.RootPart.AssemblyLinearVelocity.Y > 0 then
                        Core.RootPart.AssemblyLinearVelocity = Core.RootPart.AssemblyLinearVelocity + Vector3.new(0, jumpBoost * 0.3, 0)
                    end
                end
            end
        end)
        
        print("✅ Pulo Alto Ativado")
    else
        if Connections.Jump then
            Connections.Jump:Disconnect()
            Connections.Jump = nil
        end
        print("❌ Pulo Alto Desativado")
    end
end

-- ==================== NOCLIP ====================
local function ToggleNoclip()
    State.Noclip = not State.Noclip
    
    if State.Noclip then
        if Connections.Noclip then Connections.Noclip:Disconnect() end
        
        Connections.Noclip = Core.RunService.Stepped:Connect(function()
            if not Core.Character then return end
            
            for _, part in pairs(Core.Character:GetDescendants()) do
                if part:IsA("BasePart") and part.CanCollide then
                    part.CanCollide = false
                end
            end
        end)
        
        if not State.PropertyBypass then
            TogglePropertyBypass()
        end
        
        print("✅ Noclip Ativado")
    else
        if Connections.Noclip then
            Connections.Noclip:Disconnect()
            Connections.Noclip = nil
        end
        
        if Core.Character then
            task.spawn(function()
                for _, part in pairs(Core.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        task.wait(0.02)
                        part.CanCollide = true
                    end
                end
            end)
        end
        print("❌ Noclip Desativado")
    end
end

-- ==================== INVISIBILIDADE ====================
local function ToggleInvisibility()
    State.Invisibility = not State.Invisibility
    
    if State.Invisibility then
        if Core.Character then
            for _, part in pairs(Core.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = 1
                elseif part:IsA("Decal") then
                    part.Transparency = 1
                elseif part:IsA("Accessory") then
                    local handle = part:FindFirstChild("Handle")
                    if handle and handle:IsA("BasePart") then
                        handle.Transparency = 1
                    end
                end
            end
            
            local head = Core.Character:FindFirstChild("Head")
            if head then
                local face = head:FindFirstChildOfClass("Decal")
                if face then
                    face.Transparency = 1
                end
            end
        end
        
        print("✅ Invisibilidade Ativada")
    else
        if Core.Character then
            for _, part in pairs(Core.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    if part.Name == "HumanoidRootPart" then
                        part.Transparency = 1
                    else
                        part.Transparency = 0
                    end
                elseif part:IsA("Decal") then
                    part.Transparency = 0
                elseif part:IsA("Accessory") then
                    local handle = part:FindFirstChild("Handle")
                    if handle and handle:IsA("BasePart") then
                        handle.Transparency = 0
                    end
                end
            end
            
            local head = Core.Character:FindFirstChild("Head")
            if head then
                local face = head:FindFirstChildOfClass("Decal")
                if face then
                    face.Transparency = 0
                end
            end
        end
        
        print("❌ Invisibilidade Desativada")
    end
end

-- ==================== CLICK TELEPORT ====================
local function ToggleClickTeleport()
    State.ClickTeleport = not State.ClickTeleport
    
    if State.ClickTeleport then
        if Connections.ClickTeleport then Connections.ClickTeleport:Disconnect() end
        
        Connections.ClickTeleport = Core.UserInputService.InputBegan:Connect(function(input, processed)
            if processed then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                local mouse = Core.Player:GetMouse()
                local target = mouse.Target
                local hitPos = mouse.Hit.p
                
                if target and hitPos then
                    local distance = (Core.RootPart.Position - hitPos).Magnitude
                    if distance <= Config.ClickTeleportMaxDistance then
                        local newCFrame = CFrame.new(hitPos + Vector3.new(0, Core.Humanoid.HipHeight + 2, 0))
                        Core.RootPart.CFrame = newCFrame
                        Core.RootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                        Core.Humanoid:ChangeState(Enum.HumanoidStateType.Running)
                    else
                        print("🚫 Teleport cancelado: Alvo muito distante.")
                    end
                end
            end
        end)
        print("✅ Click Teleport Ativado")
    else
        if Connections.ClickTeleport then
            Connections.ClickTeleport:Disconnect()
            Connections.ClickTeleport = nil
        end
        print("❌ Click Teleport Desativado")
    end
end

-- ==================== FLY MODE ====================
local FlyBV, FlyBG

local function ToggleFly()
    State.Fly = not State.Fly
    
    if State.Fly then
        if Connections.Fly then Connections.Fly:Disconnect() end
        if Connections.FlyInput then Connections.FlyInput:Disconnect() end
        
        FlyBV = Instance.new("BodyVelocity")
        FlyBV.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        FlyBV.Velocity = Vector3.new(0, 0, 0)
        FlyBV.Parent = Core.RootPart
        
        FlyBG = Instance.new("BodyGyro")
        FlyBG.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        FlyBG.P = 9e4
        FlyBG.CFrame = Core.RootPart.CFrame
        FlyBG.Parent = Core.RootPart
        
        Core.Humanoid.PlatformStand = true
        
        local keysPressed = {W = false, A = false, S = false, D = false, Space = false, Shift = false}
        
        Connections.FlyInput = Core.UserInputService.InputBegan:Connect(function(input, processed)
            if processed then return end
            if input.KeyCode == Enum.KeyCode.W then keysPressed.W = true end
            if input.KeyCode == Enum.KeyCode.A then keysPressed.A = true end
            if input.KeyCode == Enum.KeyCode.S then keysPressed.S = true end
            if input.KeyCode == Enum.KeyCode.D then keysPressed.D = true end
            if input.KeyCode == Config.FlyUpKey then keysPressed.Space = true end
            if input.KeyCode == Config.FlyDownKey then keysPressed.Shift = true end
        end)
        
        Connections.FlyInputEnded = Core.UserInputService.InputEnded:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.W then keysPressed.W = false end
            if input.KeyCode == Enum.KeyCode.A then keysPressed.A = false end
            if input.KeyCode == Enum.KeyCode.S then keysPressed.S = false end
            if input.KeyCode == Enum.KeyCode.D then keysPressed.D = false end
            if input.KeyCode == Config.FlyUpKey then keysPressed.Space = false end
            if input.KeyCode == Config.FlyDownKey then keysPressed.Shift = false end
        end)
        
        Connections.Fly = Core.RunService.Heartbeat:Connect(function()
            if not Core.RootPart or not FlyBV or not FlyBG then return end
            
            local camera = workspace.CurrentCamera
            local direction = Vector3.new(0, 0, 0)
            
            if keysPressed.W then direction = direction + (camera.CFrame.LookVector) end
            if keysPressed.S then direction = direction - (camera.CFrame.LookVector) end
            if keysPressed.A then direction = direction - (camera.CFrame.RightVector) end
            if keysPressed.D then direction = direction + (camera.CFrame.RightVector) end
            if keysPressed.Space then direction = direction + Vector3.new(0, 1, 0) end
            if keysPressed.Shift then direction = direction - Vector3.new(0, 1, 0) end
            
            if direction.Magnitude > 0 then
                direction = direction.Unit
            end
            
            FlyBV.Velocity = direction * Config.FlySpeed
            FlyBG.CFrame = camera.CFrame
        end)
        
        print("✅ Fly Mode Ativado (WASD + Space/Shift)")
    else
        if Connections.Fly then
            Connections.Fly:Disconnect()
            Connections.Fly = nil
        end
        if Connections.FlyInput then
            Connections.FlyInput:Disconnect()
            Connections.FlyInput = nil
        end
        if Connections.FlyInputEnded then
            Connections.FlyInputEnded:Disconnect()
            Connections.FlyInputEnded = nil
        end
        
        if FlyBV then FlyBV:Destroy() FlyBV = nil end
        if FlyBG then FlyBG:Destroy() FlyBG = nil end
        
        if Core.Humanoid then
            Core.Humanoid.PlatformStand = false
        end
        
        print("❌ Fly Mode Desativado")
    end
end

-- ==================== FICAR MUITO PEQUENO ====================
local originalHumanoidScales = {}

local function ToggleTinyMode()
    State.TinyMode = not State.TinyMode
    
    if State.TinyMode then
        if not Core.Character or not Core.Humanoid then return end
        
        originalHumanoidScales = {
            BodyDepthScale = Core.Humanoid.BodyDepthScale.Value,
            BodyHeightScale = Core.Humanoid.BodyHeightScale.Value,
            BodyWidthScale = Core.Humanoid.BodyWidthScale.Value,
            HeadScale = Core.Humanoid.HeadScale.Value,
            HipHeight = Core.Humanoid.HipHeight
        }
        
        Core.Humanoid.BodyDepthScale.Value = originalHumanoidScales.BodyDepthScale * Config.TinySize
        Core.Humanoid.BodyHeightScale.Value = originalHumanoidScales.BodyHeightScale * Config.TinySize
        Core.Humanoid.BodyWidthScale.Value = originalHumanoidScales.BodyWidthScale * Config.TinySize
        Core.Humanoid.HeadScale.Value = originalHumanoidScales.HeadScale * Config.TinySize
        
        Core.Humanoid.HipHeight = 0.5
        
        if Core.RootPart then
            task.wait(0.1)
            Core.RootPart.CFrame = Core.RootPart.CFrame + Vector3.new(0, 2, 0)
        end
        
        print("✅ Modo Pequeno Ativado (2% do tamanho)")
    else
        if Core.Humanoid and next(originalHumanoidScales) then
            Core.Humanoid.BodyDepthScale.Value = originalHumanoidScales.BodyDepthScale
            Core.Humanoid.BodyHeightScale.Value = originalHumanoidScales.BodyHeightScale
            Core.Humanoid.BodyWidthScale.Value = originalHumanoidScales.BodyWidthScale
            Core.Humanoid.HeadScale.Value = originalHumanoidScales.HeadScale
            Core.Humanoid.HipHeight = originalHumanoidScales.HipHeight
        end
        
        originalHumanoidScales = {}
        
        print("❌ Modo Pequeno Desativado")
    end
end

-- ==================== ILUMINAÇÃO AO REDOR ====================
local LightObject = nil

local function ToggleLight()
    State.Light = not State.Light
    
    if State.Light then
        if not Core.RootPart then return end
        
        LightObject = Instance.new("PointLight")
        LightObject.Brightness = Config.LightBrightness
        LightObject.Range = Config.LightRange
        LightObject.Color = Color3.fromRGB(255, 255, 255)
        LightObject.Shadows = true
        LightObject.Parent = Core.RootPart
        
        print("✅ Iluminação Ativada (Raio: " .. Config.LightRange .. " studs)")
    else
        if LightObject then
            LightObject:Destroy()
            LightObject = nil
        end
        
        print("❌ Iluminação Desativada")
    end
end

-- ==================== AUTO CLICK PARA ARMAS ====================
local AutoClickActive = false

local function ToggleAutoClick()
    State.AutoClick = not State.AutoClick
    
    if State.AutoClick then
        if Connections.AutoClick then Connections.AutoClick:Disconnect() end
        if Connections.AutoClickEnd then Connections.AutoClickEnd:Disconnect() end
        
        Connections.AutoClick = Core.UserInputService.InputBegan:Connect(function(input, processed)
            if input.UserInputType == Enum.UserInputType.MouseButton1 and not processed then
                AutoClickActive = true
                
                task.spawn(function()
                    while AutoClickActive and State.AutoClick do
                        mouse1press()
                        task.wait(Config.AutoClickSpeed)
                        mouse1release()
                        task.wait(Config.AutoClickSpeed)
                    end
                end)
            end
        end)
        
        Connections.AutoClickEnd = Core.UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                AutoClickActive = false
            end
        end)
        
        print("✅ Auto Click Ativado (Segure o botão do mouse)")
    else
        AutoClickActive = false
        
        if Connections.AutoClick then
            Connections.AutoClick:Disconnect()
            Connections.AutoClick = nil
        end
        if Connections.AutoClickEnd then
            Connections.AutoClickEnd:Disconnect()
            Connections.AutoClickEnd = nil
        end
        
        print("❌ Auto Click Desativado")
    end
end

-- ==================== VIDA INFINITA ULTRA ====================
local function ToggleGodMode()
    State.GodMode = not State.GodMode
    
    if State.GodMode then
        if Connections.GodMode then Connections.GodMode:Disconnect() end
        if Connections.GodModeFallDamage then Connections.GodModeFallDamage:Disconnect() end
        if Connections.GodModeAntiDeath then Connections.GodModeAntiDeath:Disconnect() end
        
        Connections.GodMode = Core.RunService.Heartbeat:Connect(function()
            if Core.Humanoid and Core.Humanoid.Health < Core.Humanoid.MaxHealth then
                Core.Humanoid.Health = Core.Humanoid.MaxHealth
            end
        end)
        
        Connections.GodModeFallDamage = Core.Humanoid.StateChanged:Connect(function(oldState, newState)
            if newState == Enum.HumanoidStateType.Freefall or 
               newState == Enum.HumanoidStateType.FallingDown or
               newState == Enum.HumanoidStateType.Ragdoll then
                task.spawn(function()
                    task.wait(0.05)
                    if Core.Humanoid then
                        Core.Humanoid.Health = Core.Humanoid.MaxHealth
                    end
                end)
            end
        end)
        
        Connections.GodModeAntiDeath = Core.Humanoid.Died:Connect(function()
            task.wait(0.1)
            if Core.Character and Core.Character.Parent then
                Core.Character:BreakJoints()
            end
        end)
        
        Connections.GodModeHealthProtection = Core.Humanoid.HealthChanged:Connect(function(health)
            if health < Core.Humanoid.MaxHealth and State.GodMode then
                Core.Humanoid.Health = Core.Humanoid.MaxHealth
            end
        end)
        
        Core.Humanoid.Health = Core.Humanoid.MaxHealth
        
        print("✅ GodMode ULTRA Ativado (Invencível Total)")
    else
        if Connections.GodMode then
            Connections.GodMode:Disconnect()
            Connections.GodMode = nil
        end
        if Connections.GodModeFallDamage then
            Connections.GodModeFallDamage:Disconnect()
            Connections.GodModeFallDamage = nil
        end
        if Connections.GodModeAntiDeath then
            Connections.GodModeAntiDeath:Disconnect()
            Connections.GodModeAntiDeath = nil
        end
        if Connections.GodModeHealthProtection then
            Connections.GodModeHealthProtection:Disconnect()
            Connections.GodModeHealthProtection = nil
        end
        print("❌ GodMode Desativado")
    end
end

-- ==================== BYPASS OTIMIZADO ====================
local function TogglePropertyBypass()
    State.PropertyBypass = not State.PropertyBypass
    
    if State.PropertyBypass then
        if Connections.PropertyBypass then Connections.PropertyBypass:Disconnect() end
        if Connections.PropertyBypass2 then Connections.PropertyBypass2:Disconnect() end
        if Connections.PropertyBypass3 then Connections.PropertyBypass3:Disconnect() end
        if Connections.PropertyBypass4 then Connections.PropertyBypass4:Disconnect() end
        
        State.SavedPosition = Core.RootPart.CFrame
        
        Connections.PropertyBypass = Core.RunService.RenderStepped:Connect(function()
            if not Core.RootPart or not Core.RootPart.Parent then return end
            
            local velocity = Core.RootPart.AssemblyLinearVelocity
            local speed = velocity.Magnitude
            
            if speed > Config.MaxVelocity then
                Core.RootPart.AssemblyLinearVelocity = Vector3.new(0, velocity.Y * 0.3, 0)
                Core.RootPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            end
        end)
        
        Connections.PropertyBypass2 = Core.RunService.Heartbeat:Connect(function()
            State.InEnemyProperty = CheckEnemyProperty()
            
            if State.InEnemyProperty and Core.Humanoid then
                if Core.Humanoid:GetState() ~= Enum.HumanoidStateType.Physics then
                    Core.Humanoid:ChangeState(Enum.HumanoidStateType.Physics)
                end
            end
        end)
        
        Connections.PropertyBypass3 = Core.RunService.Heartbeat:Connect(function()
            if not Core.RootPart or not Core.RootPart.Parent then return end
            
            local now = tick()
            
            if now - State.LastSaveTime > 0.02 then
                local currentPos = Core.RootPart.CFrame.Position
                local savedPos = State.SavedPosition.Position
                local distance = (currentPos - savedPos).Magnitude
                
                if distance < 20 then
                    State.SavedPosition = Core.RootPart.CFrame
                    State.LastSaveTime = now
                end
            end
            
            local currentPos = Core.RootPart.CFrame.Position
            local savedPos = State.SavedPosition.Position
            local distance = (currentPos - savedPos).Magnitude
            
            if distance > Config.TeleportThreshold and distance < 100 then
                Core.RootPart.CFrame = State.SavedPosition
                Core.RootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            end
        end)
        
        Connections.PropertyBypass4 = Core.RootPart.ChildAdded:Connect(function(child)
            if child:IsA("BodyMover") or child:IsA("BodyPosition") or child:IsA("BodyVelocity") then
                if not State.Fly then
                    child:Destroy()
                end
            end
        end)
        
        print("✅ Bypass ULTRA Ativado")
    else
        if Connections.PropertyBypass then Connections.PropertyBypass:Disconnect() end
        if Connections.PropertyBypass2 then Connections.PropertyBypass2:Disconnect() end
        if Connections.PropertyBypass3 then Connections.PropertyBypass3:Disconnect() end
        if Connections.PropertyBypass4 then Connections.PropertyBypass4:Disconnect() end
        
        Connections.PropertyBypass = nil
        Connections.PropertyBypass2 = nil
        Connections.PropertyBypass3 = nil
        Connections.PropertyBypass4 = nil
        
        print("❌ Bypass Desativado")
    end
end

-- ==================== RESPAWN HANDLER ====================
Core.Player.CharacterAdded:Connect(function(char)
    task.wait(2)
    RefreshCharacter()
    
    local wasSpeedActive = State.Speed
    local wasJumpActive = State.Jump
    local wasNoclipActive = State.Noclip
    local wasBypassActive = State.PropertyBypass
    local wasInvisibilityActive = State.Invisibility
    local wasClickTeleportActive = State.ClickTeleport
    local wasGodModeActive = State.GodMode
    local wasFlyActive = State.Fly
    local wasTinyActive = State.TinyMode
    local wasLightActive = State.Light
    local wasAutoClickActive = State.AutoClick
    local wasESPActive = State.ESP
    local wasDamageBoostActive = State.DamageBoost
    
    State.Speed = false
    State.Jump = false
    State.Noclip = false
    State.PropertyBypass = false
    State.Invisibility = false
    State.ClickTeleport = false
    State.GodMode = false
    State.Fly = false
    State.TinyMode = false
    State.Light = false
    State.AutoClick = false
    State.ESP = false
    State.DamageBoost = false
    
    if wasSpeedActive then task.wait(0.5); ToggleSpeed() end
    if wasJumpActive then task.wait(0.5); ToggleJump() end
    if wasNoclipActive then task.wait(0.5); ToggleNoclip() end
    if wasBypassActive then task.wait(0.5); TogglePropertyBypass() end
    if wasInvisibilityActive then task.wait(0.5); ToggleInvisibility() end
    if wasClickTeleportActive then task.wait(0.5); ToggleClickTeleport() end
    if wasGodModeActive then task.wait(0.5); ToggleGodMode() end
    if wasFlyActive then task.wait(0.5); ToggleFly() end
    if wasTinyActive then task.wait(0.5); ToggleTinyMode() end
    if wasLightActive then task.wait(0.5); ToggleLight() end
    if wasAutoClickActive then task.wait(0.5); ToggleAutoClick() end
    if wasESPActive then task.wait(0.5); ToggleESP() end
    if wasDamageBoostActive then task.wait(0.5); ToggleDamageBoost() end
end)

-- ==================== GUI ====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = RandomString(12)
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 100

pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)
if not ScreenGui.Parent then
    ScreenGui.Parent = Core.Player:WaitForChild("PlayerGui")
end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "Main"
MainFrame.Size = UDim2.new(0, 320, 0, 640)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -320)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(25, 25, 30)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 15, 20))
}
Gradient.Rotation = 45
Gradient.Parent = MainFrame

local Border = Instance.new("UIStroke")
Border.Color = Color3.fromRGB(100, 100, 255)
Border.Thickness = 2
Border.Transparency = 0.5
Border.Parent = MainFrame

local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = TitleBar

local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

Core.UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -100, 1, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.Text = "⚡ BRAINROT MENU"
Title.TextColor3 = Color3.fromRGB(150, 150, 255)
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Parent = TitleBar

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Size = UDim2.new(0, 35, 0, 35)
MinimizeBtn.Position = UDim2.new(1, -78, 0, 5)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(100, 120, 180)
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Text = "_"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.TextSize = 20
MinimizeBtn.Parent = TitleBar

local MinimizeBtnCorner = Instance.new("UICorner")
MinimizeBtnCorner.CornerRadius = UDim.new(0, 8)
MinimizeBtnCorner.Parent = MinimizeBtn

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 35, 0, 35)
CloseBtn.Position = UDim2.new(1, -38, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 80)
CloseBtn.BorderSizePixel = 0
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 18
CloseBtn.Parent = TitleBar

local CloseBtnCorner = Instance.new("UICorner")
CloseBtnCorner.CornerRadius = UDim.new(0, 8)
CloseBtnCorner.Parent = CloseBtn

MinimizeBtn.MouseEnter:Connect(function()
    MinimizeBtn.BackgroundColor3 = Color3.fromRGB(120, 140, 200)
end)
MinimizeBtn.MouseLeave:Connect(function()
    MinimizeBtn.BackgroundColor3 = Color3.fromRGB(100, 120, 180)
end)

CloseBtn.MouseEnter:Connect(function()
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 80)
end)
CloseBtn.MouseLeave:Connect(function()
    CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 80)
end)

local isMinimized = false
local originalSize = MainFrame.Size

MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    
    if isMinimized then
        MainFrame:TweenSize(UDim2.new(0, 320, 0, 45), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        MinimizeBtn.Text = "□"
    else
        MainFrame:TweenSize(originalSize, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        MinimizeBtn.Text = "_"
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    if State.Speed then ToggleSpeed() end
    if State.Jump then ToggleJump() end
    if State.Noclip then ToggleNoclip() end
    if State.PropertyBypass then TogglePropertyBypass() end
    if State.Invisibility then ToggleInvisibility() end
    if State.ClickTeleport then ToggleClickTeleport() end
    if State.GodMode then ToggleGodMode() end
    if State.Fly then ToggleFly() end
    if State.TinyMode then ToggleTinyMode() end
    if State.Light then ToggleLight() end
    if State.AutoClick then ToggleAutoClick() end
    if State.ESP then ToggleESP() end
    if State.DamageBoost then ToggleDamageBoost() end
    
    for _, conn in pairs(Connections) do
        if conn then pcall(function() conn:Disconnect() end) end
    end
    
    ScreenGui:Destroy()
    print("🔒 Script Fechado")
end)

local ButtonContainer = Instance.new("Frame")
ButtonContainer.Size = UDim2.new(1, -30, 1, -70)
ButtonContainer.Position = UDim2.new(0, 15, 0, 55)
ButtonContainer.BackgroundTransparency = 1
ButtonContainer.Parent = MainFrame

local function CreateButton(icon, name, yPos, callback, stateKey)
    local BtnFrame = Instance.new("Frame")
    BtnFrame.Size = UDim2.new(1, 0, 0, 38)
    BtnFrame.Position = UDim2.new(0, 0, 0, yPos)
    BtnFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    BtnFrame.BorderSizePixel = 0
    BtnFrame.Parent = ButtonContainer
    
    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 10)
    BtnCorner.Parent = BtnFrame
    
    local BtnStroke = Instance.new("UIStroke")
    BtnStroke.Color = Color3.fromRGB(60, 60, 80)
    BtnStroke.Thickness = 1
    BtnStroke.Transparency = 0.5
    BtnStroke.Parent = BtnFrame
    
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 1, 0)
    Btn.BackgroundTransparency = 1
    Btn.Text = ""
    Btn.Parent = BtnFrame
    
    local Icon = Instance.new("TextLabel")
    Icon.Size = UDim2.new(0, 28, 0, 28)
    Icon.Position = UDim2.new(0, 8, 0.5, -14)
    Icon.BackgroundTransparency = 1
    Icon.Font = Enum.Font.GothamBold
    Icon.Text = icon
    Icon.TextColor3 = Color3.fromRGB(150, 150, 200)
    Icon.TextSize = 18
    Icon.Parent = BtnFrame
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -95, 1, 0)
    Label.Position = UDim2.new(0, 45, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Font = Enum.Font.GothamBold
    Label.Text = name
    Label.TextColor3 = Color3.fromRGB(200, 200, 220)
    Label.TextSize = 13
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = BtnFrame
    
    local StatusLabel = Instance.new("TextLabel")
    StatusLabel.Size = UDim2.new(0, 45, 0, 22)
    StatusLabel.Position = UDim2.new(1, -52, 0.5, -11)
    StatusLabel.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
    StatusLabel.Font = Enum.Font.GothamBold
    StatusLabel.Text = "OFF"
    StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 170)
    StatusLabel.TextSize = 10
    StatusLabel.Parent = BtnFrame
    
    local StatusCorner = Instance.new("UICorner")
    StatusCorner.CornerRadius = UDim.new(0, 6)
    StatusCorner.Parent = StatusLabel
    
    local function UpdateVisual()
        local isActive = State[stateKey]
        
        if isActive then
            BtnFrame.BackgroundColor3 = Color3.fromRGB(50, 80, 120)
            BtnStroke.Color = Color3.fromRGB(100, 150, 255)
            Icon.TextColor3 = Color3.fromRGB(150, 200, 255)
            StatusLabel.BackgroundColor3 = Color3.fromRGB(80, 200, 120)
            StatusLabel.Text = "ON"
            StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            BtnFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
            BtnStroke.Color = Color3.fromRGB(60, 60, 80)
            Icon.TextColor3 = Color3.fromRGB(150, 150, 200)
            StatusLabel.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
            StatusLabel.Text = "OFF"
            StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 170)
        end
    end
    
    Btn.MouseEnter:Connect(function()
        if not State[stateKey] then
            BtnFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
        end
    end)
    
    Btn.MouseLeave:Connect(function()
        UpdateVisual()
    end)
    
    Btn.MouseButton1Click:Connect(function()
        callback()
        UpdateVisual()
    end)
    
    UpdateVisual()
end

CreateButton("🏃", "Velocidade Boost", 0, ToggleSpeed, "Speed")
CreateButton("⬆️", "Pulo Alto (5x)", 47, ToggleJump, "Jump")
CreateButton("👻", "Atravessar Paredes", 94, ToggleNoclip, "Noclip")
CreateButton("👁️", "Invisibilidade", 141, ToggleInvisibility, "Invisibility")
CreateButton("⚡", "Click Teleport", 188, ToggleClickTeleport, "ClickTeleport")
CreateButton("🚀", "Fly Mode (WASD)", 235, ToggleFly, "Fly")
CreateButton("🐜", "Ficar Muito Pequeno (2%)", 282, ToggleTinyMode, "TinyMode")
CreateButton("💡", "Luz ao Redor", 329, ToggleLight, "Light")
CreateButton("🔫", "Auto Click (Armas)", 376, ToggleAutoClick, "AutoClick")
CreateButton("👀", "ESP Players (Highlight)", 423, ToggleESP, "ESP")
CreateButton("❤️", "GodMode Ultra", 470, ToggleGodMode, "GodMode")
CreateButton("💥", "Damage Boost (x10)", 517, ToggleDamageBoost, "DamageBoost")

local Footer = Instance.new("TextLabel")
Footer.Size = UDim2.new(1, 0, 0, 25)
Footer.Position = UDim2.new(0, 0, 1, -30)
Footer.BackgroundTransparency = 1
Footer.Font = Enum.Font.Gotham
Footer.Text = "Pressione ] para mostrar/ocultar | 13 Funções Ativas"
Footer.TextColor3 = Color3.fromRGB(100, 100, 150)
Footer.TextSize = 9
Footer.TextTransparency = 0.5
Footer.Parent = MainFrame

Core.UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Config.MenuKey then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("═══════════════════════════════")
print("✅ ROUBE UM BRAINROT - SCRIPT CARREGADO + DAMAGE BOOST")
print("═══════════════════════════════")
print("🏃 Velocidade Boost (x2 MAIS RÁPIDO)")
print("⬆️ Pulo Alto (5x - DOBRADO)")
print("👻 Noclip + Bypass ULTRA")
print("👁️ Invisibilidade")
print("⚡ Click Teleport")
print("🚀 Fly Mode (100 Speed - x2 MAIS RÁPIDO)")
print("🐜 Ficar Muito Pequeno (2% - SUPER MINI)")
print("💡 Luz ao Redor (60 studs)")
print("🔫 Auto Click (Para armas)")
print("👀 ESP Players (Ver todos os jogadores)")
print("❤️ GodMode Ultra (Invencível)")
print("💥 DAMAGE BOOST (x10 DANO EM TODAS AS ARMAS) ⭐ NOVO!")
print("═══════════════════════════════")
print("🔑 Pressione ] para abrir menu")
print("🎮 Fly: WASD + Space (subir) + Shift (descer)")
print("💡 Luz: Ilumina 60 studs ao seu redor")
print("🐜 Modo Pequeno: Agora 2% - SUPER pequeno e acima do chão!")
print("🔫 Auto Click: Segure o botão do mouse para atirar automaticamente")
print("👀 ESP: Highlight vermelho em todos os jogadores")
print("💥 Damage Boost: Multiplica o dano de TODAS as suas armas por 10x!")
print("⚡ VELOCIDADES DOBRADAS: Tudo x2 mais rápido!")
print("═══════════════════════════════")
