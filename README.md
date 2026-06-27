-- MEGA KILLER: DETECCIÓN POR CERCANÍA (ANTI-DUPLICADOS)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local autoMode = false
local targetSaved = nil
local character = player.Character or player.CharacterAdded:Wait()

player.CharacterAdded:Connect(function(c) character = c end)

-- 🔍 Función mágica: Encuentra al jugador más cercano a ti para no fallar por nombre
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local myRoot = character:FindFirstChild("HumanoidRootPart")
    
    if not myRoot then return nil end

    for _, v in ipairs(Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local targetRoot = v.Character.HumanoidRootPart
            local distance = (myRoot.Position - targetRoot.Position).Magnitude
            
            -- Guarda al que esté más cerca
            if distance < shortestDistance then
                shortestDistance = distance
                closestPlayer = v
            end
        end
    end
    return closestPlayer
end

-- 🎯 FUNCIÓN MUERTE DIRECTA + FUERZA
local function KillTarget(target)
    if not target or not target.Character then return end
    local tchar = target.Character
    local troot = tchar:FindFirstChild("HumanoidRootPart")
    local thum = tchar:FindFirstChildOfClass("Humanoid")
    local root = character:FindFirstChild("HumanoidRootPart")
    if not troot or not thum or not root then return end

    -- ✅ DAÑO DIRECTO INSTANTE
    thum.Health = 0
    thum:TakeDamage(999999)
    thum:BreakJoints()
    thum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None

    -- ⚡ FLING SÚPER FUERTE
    local noclip = RunService.Stepped:Connect(function()
        for _,p in pairs(character:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide=false end
        end
    end)

    -- Fuerza máxima combinada
    local bv = Instance.new("BodyVelocity", root)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0, -25000, 0)

    local ba = Instance.new("BodyAngularVelocity", root)
    ba.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    ba.AngularVelocity = Vector3.new(15000, 15000, 15000)

    -- Pegarse y lanzar
    local dur = 1.5
    local ini = tick()
    while autoMode and tick()-ini < dur and troot.Parent do
        root.CFrame = troot.CFrame * CFrame.new(0, -2, 0)
        root.AssemblyLinearVelocity = Vector3.new(0, -30000, 0)
        task.wait()
    end

    -- Limpiar
    bv:Destroy() ba:Destroy() noclip:Disconnect()
end

-- 🖥️ GUI NUEVA Y MODIFICADA
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DirectKillPro"
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0.18,0,0.22,0)
frame.Position = UDim2.new(0.7,0,0.25,0)
frame.BackgroundColor3 = Color3.new(0.1,0.1,0.1)
frame.Active = true frame.Draggable = true frame.BorderSizePixel = 1

local infoText = Instance.new("TextLabel", frame)
infoText.Size = UDim2.new(0.9,0,0.3,0)
infoText.Position = UDim2.new(0.05,0,0.1,0)
infoText.Text = "Acércate al objetivo"
infoText.TextColor3 = Color3.new(1,1,0)
infoText.BackgroundTransparency = 1
infoText.TextScaled = true

local btn = Instance.new("TextButton", frame)
btn.Size = UDim2.new(0.8,0,0.45,0)
btn.Position = UDim2.new(0.1,0,0.45,0)
btn.Text = "🎯 FIJAR MÁS CERCANO"
btn.TextColor3 = Color3.new(1,1,1)
btn.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
btn.TextScaled = true

btn.MouseButton1Click:Connect(function()
    if not autoMode then
        -- Buscar al clon más cercano en el espacio físico
        local target = GetClosestPlayer()
        if not target then 
            infoText.Text = "❌ Nadie cerca" 
            task.wait(1) 
            infoText.Text = "Acércate al objetivo" 
            return 
        end

        autoMode = true
        targetSaved = target
        infoText.Text = "Fijado: " .. target.Name
        btn.Text = "🛑 DETENER AUTO"
        btn.BackgroundColor3 = Color3.new(0.6,0,0)

        -- BUCLE KILL
        task.spawn(function()
            while autoMode and task.wait(0.7) do
                if targetSaved and targetSaved.Character then
                    KillTarget(targetSaved)
                else
                    autoMode = false
                    break
                end
            end
            infoText.Text = "Acércate al objetivo"
            btn.Text = "🎯 FIJAR MÁS CERCANO"
            btn.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
        end)
    else
        autoMode = false
    end
end)
