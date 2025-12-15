-- Colocar em: StarterPlayer > StarterPlayerScripts (LocalScript)
-- UI + bindings de tecla G + minigame simples. Usa RemoteEvents do servidor.

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ContextActionService = game:GetService("ContextActionService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Espera eventos (assume que o servidor já criou os RemoteEvents)
local function waitEvent(name)
    local ev = ReplicatedStorage:WaitForChild(name, 10)
    if not ev then
        warn("[Client] RemoteEvent não encontrado:", name)
    end
    return ev
end

local ToggleAutoFarmEvent = waitEvent("ToggleAutoFarm")
local RequestOpenLockpickEvent = waitEvent("RequestOpenLockpick")
local OpenLockpickGuiEvent = waitEvent("OpenLockpickGui")
local AttemptLockpickEvent = waitEvent("AttemptLockpick")
local LockpickResultEvent = waitEvent("LockpickResult")
local ClientNotificationEvent = waitEvent("ClientNotification")

-- Cria UI básica
local function createUI()
    if playerGui:FindFirstChild("AutoFarmLockpickUI") then
        playerGui.AutoFarmLockpickUI:Destroy()
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AutoFarmLockpickUI"
    screenGui.Parent = playerGui
    screenGui.ResetOnSpawn = false

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 260, 0, 110)
    frame.Position = UDim2.new(0, 12, 0, 80)
    frame.BackgroundTransparency = 0.15
    frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
    frame.Parent = screenGui
    local corner = Instance.new("UICorner", frame); corner.CornerRadius = UDim.new(0,8)

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, -16, 0, 24)
    title.Position = UDim2.new(0,8,0,6)
    title.BackgroundTransparency = 1
    title.Text = "Ferramentas"
    title.TextColor3 = Color3.fromRGB(230,230,230)
    title.Font = Enum.Font.SourceSansSemibold
    title.TextSize = 18
    title.TextXAlignment = Enum.TextXAlignment.Left

    local toggleBtn = Instance.new("TextButton", frame)
    toggleBtn.Size = UDim2.new(0.92, 0, 0, 36)
    toggleBtn.Position = UDim2.new(0.04, 0, 0, 34)
    toggleBtn.Text = "Alternar AutoFarm (G)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(45,150,255)
    toggleBtn.TextColor3 = Color3.new(1,1,1)
    toggleBtn.Font = Enum.Font.SourceSansBold
    toggleBtn.TextSize = 14
    local tcorner = Instance.new("UICorner", toggleBtn); tcorner.CornerRadius = UDim.new(0,6)

    local lockBtn = Instance.new("TextButton", frame)
    lockBtn.Size = UDim2.new(0.92, 0, 0, 28)
    lockBtn.Position = UDim2.new(0.04, 0, 0, 74)
    lockBtn.Text = "Tentar Lockpick"
    lockBtn.BackgroundColor3 = Color3.fromRGB(90,90,90)
    lockBtn.TextColor3 = Color3.fromRGB(230,230,230)
    lockBtn.Font = Enum.Font.SourceSans
    lockBtn.TextSize = 14
    local lcorner = Instance.new("UICorner", lockBtn); lcorner.CornerRadius = UDim.new(0,6)

    local status = Instance.new("TextLabel", frame)
    status.Size = UDim2.new(1, -16, 0, 16)
    status.Position = UDim2.new(0,8,0,96)
    status.BackgroundTransparency = 1
    status.Text = ""
    status.TextColor3 = Color3.fromRGB(180,180,180)
    status.Font = Enum.Font.SourceSans
    status.TextSize = 14
    status.TextXAlignment = Enum.TextXAlignment.Left

    -- Minigame gui (oculto)
    local mg = Instance.new("Frame", screenGui)
    mg.Size = UDim2.new(0, 420, 0, 140)
    mg.Position = UDim2.new(0.5, -210, 0.66, -70)
    mg.AnchorPoint = Vector2.new(0.5,0)
    mg.BackgroundColor3 = Color3.fromRGB(18,18,18)
    mg.Visible = false
    local mgCorner = Instance.new("UICorner", mg); mgCorner.CornerRadius = UDim.new(0,10)

    local mgTitle = Instance.new("TextLabel", mg)
    mgTitle.Size = UDim2.new(1, -24, 0, 28)
    mgTitle.Position = UDim2.new(0,12,0,8)
    mgTitle.BackgroundTransparency = 1
    mgTitle.Text = "Lockpick"
    mgTitle.TextColor3 = Color3.fromRGB(230,230,230)
    mgTitle.Font = Enum.Font.SourceSansSemibold
    mgTitle.TextSize = 18

    local bar = Instance.new("Frame", mg)
    bar.Size = UDim2.new(0.92,0,0,36)
    bar.Position = UDim2.new(0.04,0,0,46)
    bar.BackgroundColor3 = Color3.fromRGB(200,200,200)
    local barCorner = Instance.new("UICorner", bar); barCorner.CornerRadius = UDim.new(0,6)

    local mover = Instance.new("Frame", bar)
    mover.Size = UDim2.new(0.06,0,1,0)
    mover.Position = UDim2.new(0,0,0,0)
    mover.BackgroundColor3 = Color3.fromRGB(255,80,80)
    local moverCorner = Instance.new("UICorner", mover); moverCorner.CornerRadius = UDim.new(0,4)

    local target = Instance.new("Frame", bar)
    target.Size = UDim2.new(0.14,0,1,0)
    target.Position = UDim2.new(0.43,0,0,0)
    target.BackgroundColor3 = Color3.fromRGB(100,220,120)
    target.Transparency = 0.15
    local targetCorner = Instance.new("UICorner", target); targetCorner.CornerRadius = UDim.new(0,6)

    local mgInfo = Instance.new("TextLabel", mg)
    mgInfo.Size = UDim2.new(1, -24, 0, 28)
    mgInfo.Position = UDim2.new(0,12,0,88)
    mgInfo.BackgroundTransparency = 1
    mgInfo.Text = "Clique para tentar"
    mgInfo.TextColor3 = Color3.fromRGB(230,230,230)
    mgInfo.Font = Enum.Font.SourceSans
    mgInfo.TextSize = 16

    -- Minigame state
    local running = false
    local dir = 1
    local speed = 0.9
    local currentCar = nil

    local function openMinigame(carName)
        currentCar = carName
        mg.Visible = true
        running = true
        mover.Position = UDim2.new(0,0,0,0)
        dir = 1
        mgInfo.Text = "Clique para tentar"
        status.Text = ""
    end
    local function closeMinigame()
        running = false
        currentCar = nil
        mg.Visible = false
    end

    -- Bindings
    ContextActionService:BindAction("ToggleAutoFarmKey", function(_, state)
        if state == Enum.UserInputState.Begin and ToggleAutoFarmEvent then
            ToggleAutoFarmEvent:FireServer()
            status.Text = "AutoFarm solicitado..."
            return Enum.ContextActionResult.Sink
        end
        return Enum.ContextActionResult.Pass
    end, false, Enum.KeyCode.G)

    toggleBtn.MouseButton1Click:Connect(function()
        if ToggleAutoFarmEvent then
            ToggleAutoFarmEvent:FireServer()
            status.Text = "AutoFarm solicitado..."
        end
    end)

    lockBtn.MouseButton1Click:Connect(function()
        if RequestOpenLockpickEvent then
            RequestOpenLockpickEvent:FireServer()
            status.Text = "Procurando carro travado..."
        end
    end)

    bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 and running and currentCar then
            running = false
            local moverCenter = mover.AbsolutePosition.X + mover.AbsoluteSize.X * 0.5
            local targetStart = target.AbsolutePosition.X
            local targetEnd = target.AbsolutePosition.X + target.AbsoluteSize.X
            local inside = moverCenter >= targetStart and moverCenter <= targetEnd
            if AttemptLockpickEvent then
                AttemptLockpickEvent:FireServer(currentCar, inside)
            end
            closeMinigame()
            status.Text = "Tentando lockpick..."
        end
    end)

    RunService.Heartbeat:Connect(function(dt)
        if running then
            local pos = mover.Position.X.Scale
            pos = pos + dir * speed * dt
            if pos <= 0 then
                pos = 0; dir = 1
            elseif pos + mover.Size.X.Scale >= 1 then
                pos = 1 - mover.Size.X.Scale; dir = -1
            end
            mover.Position = UDim2.new(pos,0,0,0)
        end
    end)

    -- Events from server
    OpenLockpickGuiEvent.OnClientEvent:Connect(function(carNameOrFalse, msg)
        if carNameOrFalse == false then
            status.Text = msg or "Nenhum carro perto."
            delay(2, function() if status then status.Text = "" end end)
            return
        end
        openMinigame(carNameOrFalse)
    end)

    LockpickResultEvent.OnClientEvent:Connect(function(success, message)
        status.Text = message or (success and "Desbloqueado!" or "Falhou.")
        delay(2, function() if status then status.Text = "" end end)
    end)

    if ClientNotificationEvent then
        ClientNotificationEvent.OnClientEvent:Connect(function(msg)
            status.Text = tostring(msg)
            delay(1.8, function() if status then status.Text = "" end end)
        end)
    end
end

createUI()
