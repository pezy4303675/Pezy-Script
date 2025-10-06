--[[
    Brainrot - Ultimate Script Pro Edition
    Sistema Completo: Bypass + Invisibilidade + Fly + GodMode + Size Changer + Iluminação + Auto Click + Highlight + Kill Nearby
    Compatível com: Bloxburg, Roube um Brainrot e outros jogos
    Atalhos: J (Highlight Vermelho), P (Matar próximos)
]]

repeat task.wait() until game:IsLoaded()
task.wait(3)

local function RandomString(length)
    local chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    local result = ""
    for i = 1, length do
        local rand = math.random(1, #chars)
        result = result .. chars:sub(rand, rand)
    end
    return result
end

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
}

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

local Connections = {}

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

-- ==================== FUNÇÕES PRINCIPAIS ====================
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
    else
        if Connections.Speed then Connections.Speed:Disconnect() Connections.Speed = nil end
    end
end

local function ToggleJump()
    State.Jump = not State.Jump
    if State.Jump then
        if Connections.Jump then Connections.Jump:Disconnect() end
        Connections.Jump = Core.Humanoid.StateChanged:Connect(function(_, newState)
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
                    task.delay(0.2, function() if bodyVel and bodyVel.Parent then bodyVel:Destroy() end end)
                    task.wait(0.05)
                    if Core.RootPart and Core.RootPart.AssemblyLinearVelocity.Y > 0 then
                        Core.RootPart.AssemblyLinearVelocity = Core.RootPart.AssemblyLinearVelocity + Vector3.new(0, jumpBoost * 0.3, 0)
                    end
                end
            end
        end)
    else
        if Connections.Jump then Connections.Jump:Disconnect() Connections.Jump = nil end
    end
end

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
    else
        if Connections.Noclip then Connections.Noclip:Disconnect() Connections.Noclip = nil end
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
    end
end

local function ToggleInvisibility()
    State.Invisibility = not State.Invisibility
    if Core.Character then
        for _, part in pairs(Core.Character:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("Decal") then
                part.Transparency = State.Invisibility and 1 or 0
            elseif part:IsA("Accessory") then
                local handle = part:FindFirstChild("Handle")
                if handle and handle:IsA("BasePart") then
                    handle.Transparency = State.Invisibility and 1 or 0
                end
            end
        end
        local head = Core.Character:FindFirstChild("Head")
        if head then
            local face = head:FindFirstChildOfClass("Decal")
            if face then face.Transparency = State.Invisibility and 1 or 0 end
        end
    end
end

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
                    end
                end
            end
        end)
    else
        if Connections.ClickTeleport then Connections.ClickTeleport:Disconnect() Connections.ClickTeleport = nil end
    end
end

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
            if direction.Magnitude > 0 then direction = direction.Unit end
            FlyBV.Velocity = direction * Config.FlySpeed
            FlyBG.CFrame = camera.CFrame
        end)
    else
        if Connections.Fly then Connections.Fly:Disconnect() Connections.Fly = nil end
        if Connections.FlyInput then Connections.FlyInput:Disconnect() Connections.FlyInput = nil end
        if Connections.FlyInputEnded then Connections.FlyInputEnded:Disconnect() Connections.FlyInputEnded = nil end
        if FlyBV then FlyBV:Destroy() FlyBV = nil end
        if FlyBG then FlyBG:Destroy() FlyBG = nil end
        if Core.Humanoid then Core.Humanoid.PlatformStand = false end
    end
end

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
        if Core.RootPart then task.wait(0.1) Core.RootPart.CFrame = Core.RootPart.CFrame + Vector3.new(0, 2, 0) end
    else
        if Core.Humanoid and next(originalHumanoidScales) then
            Core.Humanoid.BodyDepthScale.Value = originalHumanoidScales.BodyDepthScale
            Core.Humanoid.BodyHeightScale.Value = originalHumanoidScales.BodyHeightScale
            Core.Humanoid.BodyWidthScale.Value = originalHumanoidScales.BodyWidthScale
            Core.Humanoid.HeadScale.Value = originalHumanoidScales.HeadScale
            Core.Humanoid.HipHeight = originalHumanoidScales.HipHeight
        end
        originalHumanoidScales = {}
    end
end

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
    else
        if LightObject then LightObject:Destroy() LightObject = nil end
    end
end

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
    else
        AutoClickActive = false
        if Connections.AutoClick then Connections.AutoClick:Disconnect() Connections.AutoClick = nil end
        if Connections.AutoClickEnd then Connections.AutoClickEnd:Disconnect() Connections.AutoClickEnd = nil end
    end
end

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
        Connections.GodModeFallDamage = Core.Humanoid.StateChanged:Connect(function(_, newState)
            if newState == Enum.HumanoidStateType.Freefall or newState == Enum.HumanoidStateType.FallingDown or newState == Enum.HumanoidStateType.Ragdoll then
                task.spawn(function() task.wait(0.05) if Core.Humanoid then Core.Humanoid.Health = Core.Humanoid.MaxHealth end end)
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
    else
        if Connections.GodMode then Connections.GodMode:Disconnect() Connections.GodMode = nil end
        if Connections.GodModeFallDamage then Connections.GodModeFallDamage:Disconnect() Connections.GodModeFallDamage = nil end
        if Connections.GodModeAntiDeath then Connections.GodModeAntiDeath:Disconnect() Connections.GodModeAntiDeath = nil end
        if Connections.GodModeHealthProtection then Connections.GodModeHealthProtection:Disconnect() Connections.GodModeHealthProtection = nil end
    end
end

-- ==================== FUNÇÕES EXTRAS: HIGHLIGHT E KILL NEARBY ====================
local function HighlightAllHumanoids()
    for _, obj in ipairs(Core.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") then
            if obj:FindFirstChild("CopilotRedHighlight") then
                obj.CopilotRedHighlight:Destroy()
            end
            local highlight = Instance.new("Highlight")
            highlight.Name = "CopilotRedHighlight"
            highlight.FillColor = Color3.fromRGB(255, 0, 0)
            highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
            highlight.Parent = obj
        end
    end
end

local function KillNearbyHumanoids()
    local myChar = Core.Player.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end
    local range = 15
    for _, obj in ipairs(Core.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= myChar then
            local humanoid = obj:FindFirstChildOfClass("Humanoid")
            local rootPart = obj:FindFirstChild("HumanoidRootPart")
            if humanoid and humanoid.Health > 0 and rootPart then
                local dist = (myRoot.Position - rootPart.Position).Magnitude
                if dist <= range then
                    humanoid.Health = 0
                end
            end
        end
    end
end

Core.UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.J then
        HighlightAllHumanoids()
    elseif not processed and input.KeyCode == Enum.KeyCode.P then
        KillNearbyHumanoids()
    end
end)

-- ==================== GUI PROFISSIONAL ====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = RandomString(12)
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 100
pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
if not ScreenGui.Parent then
    ScreenGui.Parent = Core.Player:WaitForChild("PlayerGui")
end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "Main"
MainFrame.Size = UDim2.new(0, 340, 0, 510)
MainFrame.Position = UDim2.new(0.5, -170, 0.5, -255)
MainFrame.BackgroundColor3 = Color3.fromRGB(28, 32, 45)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 13)
MainCorner.Parent = MainFrame
local Border = Instance.new("UIStroke")
Border.Color = Color3.fromRGB(60, 120, 230)
Border.Thickness = 2
Border.Transparency = 0.18
Border.Parent = MainFrame

local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 44)
TitleBar.BackgroundColor3 = Color3.fromRGB(36, 40, 55)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame
local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 13)
TitleCorner.Parent = TitleBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -90, 1, 0)
Title.AnchorPoint = Vector2.new(0, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamSemibold
Title.Text = "Brainrot Ultimate Script"
Title.TextColor3 = Color3.fromRGB(200, 210, 255)
Title.TextSize = 20
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Position = UDim2.new(0, 18, 0, 0)
Title.Parent = TitleBar

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Size = UDim2.new(0, 34, 0, 34)
MinimizeBtn.Position = UDim2.new(1, -75, 0, 5)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(66, 75, 105)
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Text = "-"
MinimizeBtn.TextColor3 = Color3.fromRGB(220, 220, 240)
MinimizeBtn.TextSize = 20
MinimizeBtn.Parent = TitleBar
local MinimizeBtnCorner = Instance.new("UICorner")
MinimizeBtnCorner.CornerRadius = UDim.new(0, 8)
MinimizeBtnCorner.Parent = MinimizeBtn

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 34, 0, 34)
CloseBtn.Position = UDim2.new(1, -37, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 60)
CloseBtn.BorderSizePixel = 0
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 17
CloseBtn.Parent = TitleBar
local CloseBtnCorner = Instance.new("UICorner")
CloseBtnCorner.CornerRadius = UDim.new(0, 8)
CloseBtnCorner.Parent = CloseBtn

local isMinimized = false
local originalSize = MainFrame.Size
MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        MainFrame:TweenSize(UDim2.new(0, 340, 0, 44), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        MinimizeBtn.Text = "+"
    else
        MainFrame:TweenSize(originalSize, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        MinimizeBtn.Text = "-"
    end
end)
CloseBtn.MouseButton1Click:Connect(function()
    for _, conn in pairs(Connections) do if conn then pcall(function() conn:Disconnect() end) end end
    ScreenGui:Destroy()
end)

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
        input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
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

local ButtonContainer = Instance.new("Frame")
ButtonContainer.Size = UDim2.new(1, -32, 1, -64)
ButtonContainer.Position = UDim2.new(0, 16, 0, 56)
ButtonContainer.BackgroundTransparency = 1
ButtonContainer.Parent = MainFrame

local BUTTONS = {
    { "Speed Boost",      ToggleSpeed,      "Speed" },
    { "Jump Boost",       ToggleJump,       "Jump" },
    { "Noclip",           ToggleNoclip,     "Noclip" },
    { "Invisibility",     ToggleInvisibility,"Invisibility" },
    { "Click Teleport",   ToggleClickTeleport, "ClickTeleport" },
    { "Fly Mode",         ToggleFly,        "Fly" },
    { "Tiny Size",        ToggleTinyMode,   "TinyMode" },
    { "Light Around",     ToggleLight,      "Light" },
    { "Auto Click",       ToggleAutoClick,  "AutoClick" },
    { "GodMode",          ToggleGodMode,    "GodMode" }
}

local function CreateButton(name, callback, stateKey, yPos)
    local BtnFrame = Instance.new("Frame")
    BtnFrame.Size = UDim2.new(1, 0, 0, 35)
    BtnFrame.Position = UDim2.new(0, 0, 0, yPos)
    BtnFrame.BackgroundColor3 = Color3.fromRGB(39, 45, 65)
    BtnFrame.BorderSizePixel = 0
    BtnFrame.Parent = ButtonContainer
    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 8)
    BtnCorner.Parent = BtnFrame
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 1, 0)
    Btn.BackgroundTransparency = 1
    Btn.Text = ""
    Btn.Parent = BtnFrame
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -80, 1, 0)
    Label.Position = UDim2.new(0, 12, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Font = Enum.Font.Gotham
    Label.Text = name
    Label.TextColor3 = Color3.fromRGB(220, 225, 245)
    Label.TextSize = 15
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = BtnFrame
    local StatusLabel = Instance.new("TextLabel")
    StatusLabel.Size = UDim2.new(0, 65, 0, 26)
    StatusLabel.Position = UDim2.new(1, -72, 0.5, -13)
    StatusLabel.BackgroundColor3 = Color3.fromRGB(70, 85, 125)
    StatusLabel.Font = Enum.Font.GothamBold
    StatusLabel.Text = "OFF"
    StatusLabel.TextColor3 = Color3.fromRGB(190, 200, 235)
    StatusLabel.TextSize = 11
    StatusLabel.Parent = BtnFrame
    local StatusCorner = Instance.new("UICorner")
    StatusCorner.CornerRadius = UDim.new(0, 7)
    StatusCorner.Parent = StatusLabel

    local function UpdateVisual()
        local isActive = State[stateKey]
        if isActive then
            BtnFrame.BackgroundColor3 = Color3.fromRGB(47, 82, 150)
            Label.TextColor3 = Color3.fromRGB(220, 230, 255)
            StatusLabel.BackgroundColor3 = Color3.fromRGB(80, 160, 110)
            StatusLabel.Text = "ON"
            StatusLabel.TextColor3 = Color3.fromRGB(245, 255, 255)
        else
            BtnFrame.BackgroundColor3 = Color3.fromRGB(39, 45, 65)
            Label.TextColor3 = Color3.fromRGB(220, 225, 245)
            StatusLabel.BackgroundColor3 = Color3.fromRGB(70, 85, 125)
            StatusLabel.Text = "OFF"
            StatusLabel.TextColor3 = Color3.fromRGB(190, 200, 235)
        end
    end

    Btn.MouseEnter:Connect(function()
        BtnFrame.BackgroundColor3 = Color3.fromRGB(53, 61, 82)
    end)
    Btn.MouseLeave:Connect(UpdateVisual)
    Btn.MouseButton1Click:Connect(function()
        callback()
        UpdateVisual()
    end)
    UpdateVisual()
end

for i, btnData in ipairs(BUTTONS) do
    CreateButton(btnData[1], btnData[2], btnData[3], (i-1)*41)
end

local Footer = Instance.new("TextLabel")
Footer.Size = UDim2.new(1, 0, 0, 22)
Footer.Position = UDim2.new(0, 0, 1, -24)
Footer.BackgroundTransparency = 1
Footer.Font = Enum.Font.Gotham
Footer.Text = "Atalhos: [J] Highlight | [P] Kill Nearby | [ ] Menu"
Footer.TextColor3 = Color3.fromRGB(145, 155, 200)
Footer.TextSize = 11
Footer.TextTransparency = 0.25
Footer.Parent = MainFrame

Core.UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Config.MenuKey then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("==== Brainrot Ultimate Script Pro ====")
print("Painel profissional, atalhos: [J] Highlight, [P] Kill Nearby, [ ]] Menu.")
print("=====================================")
