-- LocalScript  |  Coloque dentro de StarterPlayerScripts
-- NOME: Menu BH_LAB (aimbot, fov, wallhack, espeed, cor)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

-- Config
local cfg = {
    aimEnabled = false,
    aimFov = 90,
    wallhackOn = false,
    speedMulti = 16,
    menuColor = Color3.new(1,1,1)
}

-- Variáveis de UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BH_LAB_GUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Janela principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,300,0,220)
mainFrame.Position = UDim2.new(0.5,-150,0.5,-110)
mainFrame.AnchorPoint = Vector2.new(0.5,0.5)
mainFrame.BackgroundColor3 = Color3.new(0,0,0)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = false
mainFrame.Parent = screenGui

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "BH_LAB"
title.Font = Enum.Font.SourceSansBold
title.TextScaled = true
title.TextColor3 = cfg.menuColor
title.Parent = mainFrame

-- Botão Fechar (X)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0,20,0,20)
closeBtn.Position = UDim2.new(1,-25,0,5)
closeBtn.BackgroundColor3 = Color3.new(1,0,0)
closeBtn.Text = "X"
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextColor3 = Color3.new(1,1,1)
closeBtn.TextScaled = true
closeBtn.Parent = mainFrame

-- Abrir (miniatura)
local miniBtn = Instance.new("TextButton")
miniBtn.Size = UDim2.new(0,60,0,20)
miniBtn.Position = UDim2.new(0.5,-30,0,0)
miniBtn.BackgroundColor3 = Color3.new(0,0,0)
miniBtn.BorderSizePixel = 0
miniBtn.Text = "BH_LAB"
miniBtn.Font = Enum.Font.SourceSans
miniBtn.TextColor3 = cfg.menuColor
miniBtn.TextScaled = true
miniBtn.Parent = screenGui

-- UIListLayout para organizar toggles
local list = Instance.new("UIListLayout")
list.Padding = UDim.new(0,4)
list.Parent = mainFrame
list.SortOrder = Enum.SortOrder.LayoutOrder

-- Template para cada opção
local function novoToggle(nome, layout, callback)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1,0,0,24)
    container.BackgroundTransparency = 1
    container.LayoutOrder = layout
    container.Parent = mainFrame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.6,0,1,0)
    label.BackgroundTransparency = 1
    label.Text = nome
    label.Font = Enum.Font.SourceSans
    label.TextScaled = true
    label.TextColor3 = cfg.menuColor
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = container

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0.4,0,1,0)
    toggle.Position = UDim2.new(0.6,0,0,0)
    toggle.Text = "OFF"
    toggle.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    toggle.TextColor3 = cfg.menuColor
    toggle.Font = Enum.Font.SourceSans
    toggle.TextScaled = true
    toggle.Parent = container

    local isOn = false
    toggle.MouseButton1Click:Connect(function()
        isOn = not isOn
        toggle.Text = isOn and "ON" or "OFF"
        callback(isOn)
    end)

    function toggle:Set(val)
        isOn = val
        toggle.Text = val and "ON" or "OFF"
    end

    return toggle
end

local function novoSlider(nome, layout, min, max, default, callback)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1,0,0,24)
    container.BackgroundTransparency = 1
    container.LayoutOrder = layout
    container.Parent = mainFrame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.4,0,1,0)
    label.BackgroundTransparency = 1
    label.Text = nome.." ("..default..")"
    label.Font = Enum.Font.SourceSans
    label.TextScaled = true
    label.TextColor3 = cfg.menuColor
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = container

    local slider = Instance.new("TextBox")
    slider.Size = UDim2.new(0.55,0,1,0)
    slider.Position = UDim2.new(0.42,0,0,0)
    slider.Text = tostring(default)
    slider.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    slider.TextColor3 = cfg.menuColor
    slider.Font = Enum.Font.SourceSans
    slider.TextScaled = true
    slider.Parent = container

    slider.FocusLost:Connect(function()
        local num = tonumber(slider.Text)
        if num then
            num = math.clamp(num, min, max)
            label.Text = nome.." ("..num..")"
            slider.Text = tostring(num)
            callback(num)
        end
    end)
end

-- Criando toggles/sliders
novoToggle("Aimbot",0,function(s) cfg.aimEnabled=s end)
novoSlider("Aim FOV",1,5,360,cfg.aimFov,function(v) cfg.aimFov=v end)
novoToggle("Wallhack",2,function(s) cfg.wallhackOn=s; wallhack(s) end)
novoSlider("E-Speed",3,0,50,cfg.speedMulti,function(v) cfg.speedMulti=v; updateSpeed() end)

-- Funções internas
local charCache = nil
local speedConn = nil

function updateSpeed()
    if speedConn then speedConn:Disconnect() end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildWhichIsA("Humanoid")
    if hum then
        speedConn = hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
            hum.WalkSpeed = cfg.speedMulti
        end)
        hum.WalkSpeed = cfg.speedMulti
    end
end

player.CharacterAdded:Connect(updateSpeed)
if player.Character then updateSpeed() end

-- Wallhack: remove transparência de parts
local wallParts = {}
function wallhack(on)
    if on then
        for _,p in pairs(workspace:GetDescendants()) do
            if p:IsA("BasePart") and p.Transparency < 1 then
                table.insert(wallParts, {obj=p, old=p.Transparency})
                p.Transparency = 0.8
            end
        end
    else
        for _,data in pairs(wallParts) do
            data.obj.Transparency = data.old
        end
        wallParts = {}
    end
end

-- Aimbot: target lock mais próximo no FOV
local target = nil
local function dot(v1,v2) return v1:Dot(v2)/(v1.Magnitude*v2.Magnitude) end
RunService.RenderStepped:Connect(function(dt)
    if not cfg.aimEnabled then target = nil return end
    local char = player.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local camPos = camera.CFrame.Position
    local bestDot,cand
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            local hr = p.Character:FindFirstChild("HumanoidRootPart")
            if hr then
                local dir = (hr.Position - camPos).Unit
                local cf = camera.CFrame.LookVector
                local angle = math.acos(dot(cf,dir))
                if angle <= math.rad(cfg.aimFov/2) then
                    if not cand or angle < bestDot then
                        bestDot = angle
                        cand = hr
                    end
                end
            end
        end
    end
    target = cand
    if target then
        local pos = target.Position + Vector3.new(0,2,0)
        camera.CFrame = CFrame.new(camPos,pos)
    end
end)

-- Controle do menu
closeBtn.MouseButton1Click:Connect(function() 
    mainFrame.Visible = false 
    miniBtn.Visible = true 
end)
miniBtn.MouseButton1Click:Connect(function() 
    mainFrame.Visible = true 
    miniBtn.Visible = false 
end)

screenGui.Parent = player:WaitForChild("PlayerGui")
