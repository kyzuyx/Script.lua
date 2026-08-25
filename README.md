--[[
    ⚡ HUB ULTIMATE - Aimbot + Hitbox + ESP
    Compatível com Delta Executor (Mobile)
    Versão: 2.0
]]

-- ============================================
-- 1. SERVIÇOS E CONFIGURAÇÕES
-- ============================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local C = {
    Aimbot = {Ativo=false, FOV=150, Target="Head", Wall=false, Cor=Color3.new(1,0,0), Smooth=0.15},
    Hitbox = {Ativo=false, Tamanho=3},
    ESP = {
        Enemy = {Ativo=false, Dist=200, Cor=Color3.new(0,1,0)},
        Player = {Ativo=false, Dist=150, Cor=Color3.new(0,0.5,1)},
        Item = {Ativo=false, Dist=100, Cor=Color3.new(1,1,0)}
    }
}

local HitboxCache = {}
local Highlights = {}
local State = {Tab="Aimbot", Aberto=true}

-- ============================================
-- 2. GUI PRINCIPAL
-- ============================================
local sg = Instance.new("ScreenGui")
sg.Name = "Hub"
sg.ResetOnSpawn = false
sg.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Painel
local panel = Instance.new("Frame")
panel.Parent = sg
panel.Size = UDim2.new(0, 480, 0, 400)
panel.Position = UDim2.new(0.5, -240, 0.5, -200)
panel.BackgroundColor3 = Color3.new(0.07, 0.07, 0.09)
panel.BackgroundTransparency = 0.05
panel.BorderSizePixel = 0

local pCorner = Instance.new("UICorner")
pCorner.CornerRadius = UDim.new(0, 16)
pCorner.Parent = panel

local shadow = Instance.new("UIShadow")
shadow.Size = 15
shadow.Parent = panel

-- Sidebar
local sidebar = Instance.new("Frame")
sidebar.Parent = panel
sidebar.Size = UDim2.new(0, 140, 1, 0)
sidebar.BackgroundColor3 = Color3.new(0.09, 0.09, 0.11)
sidebar.BackgroundTransparency = 0.1
sidebar.BorderSizePixel = 0

local sCorner = Instance.new("UICorner")
sCorner.CornerRadius = UDim.new(0, 16)
sCorner.Parent = sidebar

local title = Instance.new("TextLabel")
title.Parent = sidebar
title.Size = UDim2.new(1, 0, 0, 45)
title.Position = UDim2.new(0, 0, 0, 8)
title.Text = "⚡ HUB"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 22
title.Font = Enum.Font.GothamBold
title.BackgroundTransparency = 1

-- Botões Sidebar
local navBtns = {}
local tabs = {"Aimbot", "Hitbox", "ESP", "Settings"}
local icons = {"🎯", "🛡️", "👁️", "⚙️"}
local navY = 60

for i, tab in ipairs(tabs) do
    local btn = Instance.new("TextButton")
    btn.Parent = sidebar
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0, navY)
    btn.Text = icons[i] .. "  " .. tab
    btn.TextColor3 = Color3.new(0.7, 0.7, 0.75)
    btn.TextSize = 14
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Font = Enum.Font.Gotham
    btn.BackgroundColor3 = Color3.new(0.12, 0.12, 0.15)
    btn.BackgroundTransparency = 0.5
    btn.BorderSizePixel = 0
    btn.Name = tab
    
    local bCorner = Instance.new("UICorner")
    bCorner.CornerRadius = UDim.new(0, 8)
    bCorner.Parent = btn
    
    navBtns[tab] = btn
    navY = navY + 48
end

-- Área de Conteúdo
local content = Instance.new("Frame")
content.Parent = panel
content.Size = UDim2.new(1, -155, 1, -16)
content.Position = UDim2.new(0, 150, 0, 8)
content.BackgroundTransparency = 1

-- ============================================
-- 3. FUNÇÕES UI
-- ============================================

local function makeToggle(parent, label, getVal, setVal, y)
    local f = Instance.new("Frame")
    f.Parent = parent
    f.Size = UDim2.new(1, 0, 0, 35)
    f.Position = UDim2.new(0, 0, 0, y)
    f.BackgroundTransparency = 1
    
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.Size = UDim2.new(0.6, 0, 1, 0)
    l.Text = label
    l.TextColor3 = Color3.new(0.78, 0.78, 0.82)
    l.TextSize = 13
    l.Font = Enum.Font.Gotham
    l.BackgroundTransparency = 1
    l.TextXAlignment = Enum.TextXAlignment.Left
    
    local btn = Instance.new("TextButton")
    btn.Parent = f
    btn.Size = UDim2.new(0, 44, 0, 24)
    btn.Position = UDim2.new(0.85, -22, 0.5, -12)
    btn.Text = ""
    btn.BackgroundColor3 = Color3.new(0.2, 0.2, 0.24)
    btn.BorderSizePixel = 0
    
    local bCorner = Instance.new("UICorner")
    bCorner.CornerRadius = UDim.new(1, 0)
    bCorner.Parent = btn
    
    local ind = Instance.new("Frame")
    ind.Parent = btn
    ind.Size = UDim2.new(0, 18, 0, 18)
    ind.Position = UDim2.new(0, 3, 0.5, -9)
    ind.BackgroundColor3 = Color3.new(0.8, 0.8, 0.8)
    ind.BorderSizePixel = 0
    
    local iCorner = Instance.new("UICorner")
    iCorner.CornerRadius = UDim.new(1, 0)
    iCorner.Parent = ind
    
    local function update(state)
        if state then
            btn.BackgroundColor3 = Color3.new(0, 0.7, 0.3)
            ind.Position = UDim2.new(0, 23, 0.5, -9)
            ind.BackgroundColor3 = Color3.new(1, 1, 1)
        else
            btn.BackgroundColor3 = Color3.new(0.2, 0.2, 0.24)
            ind.Position = UDim2.new(0, 3, 0.5, -9)
            ind.BackgroundColor3 = Color3.new(0.8, 0.8, 0.8)
        end
    end
    
    update(getVal())
    
    btn.MouseButton1Click:Connect(function()
        local new = not getVal()
        setVal(new)
        update(new)
    end)
    
    return {update=update}
end

local function makeSlider(parent, label, min, max, step, getVal, setVal, y, cb)
    local f = Instance.new("Frame")
    f.Parent = parent
    f.Size = UDim2.new(1, 0, 0, 45)
    f.Position = UDim2.new(0, 0, 0, y)
    f.BackgroundTransparency = 1
    
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.Size = UDim2.new(0.5, 0, 0, 20)
    l.Position = UDim2.new(0, 0, 0, 0)
    l.Text = label
    l.TextColor3 = Color3.new(0.78, 0.78, 0.82)
    l.TextSize = 13
    l.Font = Enum.Font.Gotham
    l.BackgroundTransparency = 1
    l.TextXAlignment = Enum.TextXAlignment.Left
    
    local v = Instance.new("TextLabel")
    v.Parent = f
    v.Size = UDim2.new(0.3, 0, 0, 20)
    v.Position = UDim2.new(0.7, 0, 0, 0)
    v.Text = tostring(getVal())
    v.TextColor3 = Color3.new(1, 1, 1)
    v.TextSize = 13
    v.Font = Enum.Font.GothamBold
    v.BackgroundTransparency = 1
    v.TextXAlignment = Enum.TextXAlignment.Right
    
    local track = Instance.new("Frame")
    track.Parent = f
    track.Size = UDim2.new(0.9, 0, 0, 3)
    track.Position = UDim2.new(0, 0, 0, 30)
    track.BackgroundColor3 = Color3.new(0.2, 0.2, 0.24)
    track.BorderSizePixel = 0
    
    local fill = Instance.new("Frame")
    fill.Parent = track
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = Color3.new(0, 0.7, 1)
    fill.BorderSizePixel = 0
    
    local handle = Instance.new("TextButton")
    handle.Parent = track
    handle.Size = UDim2.new(0, 14, 0, 14)
    handle.Position = UDim2.new(0, -7, 0.5, -7)
    handle.BackgroundColor3 = Color3.new(1, 1, 1)
    handle.BorderSizePixel = 0
    handle.Text = ""
    
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(1, 0)
    hCorner.Parent = handle
    
    local function update(val)
        val = math.max(min, math.min(max, math.round(val / step) * step))
        setVal(val)
        v.Text = tostring(val)
        local pct = (val - min) / (max - min)
        fill.Size = UDim2.new(pct, 0, 1, 0)
        handle.Position = UDim2.new(pct, -7, 0.5, -7)
        if cb then cb(val) end
    end
    
    update(getVal())
    
    local drag = false
    handle.MouseButton1Down:Connect(function() drag = true end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if not drag or i.UserInputType ~= Enum.UserInputType.Mouse then return end
        local pos = i.Position.X
        local abs = track.AbsolutePosition.X
        local w = track.AbsoluteSize.X
        local pct = math.clamp((pos - abs) / w, 0, 1)
        update(min + pct * (max - min))
    end)
    track.MouseButton1Click:Connect(function(i)
        local pos = i.Position.X
        local abs = track.AbsolutePosition.X
        local w = track.AbsoluteSize.X
        local pct = math.clamp((pos - abs) / w, 0, 1)
        update(min + pct * (max - min))
    end)
    
    return {update=update}
end

local function makeColorPicker(parent, label, getVal, setVal, y, cb)
    local f = Instance.new("Frame")
    f.Parent = parent
    f.Size = UDim2.new(1, 0, 0, 35)
    f.Position = UDim2.new(0, 0, 0, y)
    f.BackgroundTransparency = 1
    
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.Size = UDim2.new(0.6, 0, 1, 0)
    l.Text = label
    l.TextColor3 = Color3.new(0.78, 0.78, 0.82)
    l.TextSize = 13
    l.Font = Enum.Font.Gotham
    l.BackgroundTransparency = 1
    l.TextXAlignment = Enum.TextXAlignment.Left
    
    local btn = Instance.new("TextButton")
    btn.Parent = f
    btn.Size = UDim2.new(0, 28, 0, 28)
    btn.Position = UDim2.new(0.85, -14, 0.5, -14)
    btn.BackgroundColor3 = getVal()
    btn.BorderSizePixel = 0
    btn.Text = ""
    
    local bCorner = Instance.new("UICorner")
    bCorner.CornerRadius = UDim.new(0, 6)
    bCorner.Parent = btn
    
    local colors = {
        Color3.new(1,0,0), Color3.new(0,1,0), Color3.new(0,0.5,1),
        Color3.new(1,1,0), Color3.new(1,0,1), Color3.new(0,1,1),
        Color3.new(1,0.5,0), Color3.new(0.8,0.8,0.8)
    }
    
    local open = false
    local picker = nil
    
    btn.MouseButton1Click:Connect(function()
        if open then picker:Destroy() picker=nil open=false return end
        open = true
        picker = Instance.new("Frame")
        picker.Parent = f
        picker.Size = UDim2.new(0, 150, 0, 38)
        picker.Position = UDim2.new(0.5, -75, 1, 5)
        picker.BackgroundColor3 = Color3.new(0.12, 0.12, 0.15)
        picker.BorderSizePixel = 0
        
        local pCorner = Instance.new("UICorner")
        pCorner.CornerRadius = UDim.new(0, 8)
        pCorner.Parent = picker
        
        for i, color in ipairs(colors) do
            local b = Instance.new("TextButton")
            b.Parent = picker
            b.Size = UDim2.new(0, 22, 0, 22)
            b.Position = UDim2.new(0, 5 + (i-1) * 26, 0.5, -11)
            b.BackgroundColor3 = color
            b.BorderSizePixel = 0
            b.Text = ""
            
            local bc = Instance.new("UICorner")
            bc.CornerRadius = UDim.new(0, 4)
            bc.Parent = b
            
            b.MouseButton1Click:Connect(function()
                setVal(color)
                btn.BackgroundColor3 = color
                if cb then cb(color) end
                picker:Destroy() picker=nil open=false
            end)
        end
    end)
    
    return {update=function(c) btn.BackgroundColor3 = c end}
end

-- ============================================
-- 4. CONTEÚDO DAS ABAS
-- ============================================

local tabContents = {}

-- Aimbot
local aContent = Instance.new("Frame")
aContent.Parent = content
aContent.Size = UDim2.new(1, 0, 1, 0)
aContent.BackgroundTransparency = 1
aContent.Visible = true
tabContents["Aimbot"] = aContent

local y = 0
local aToggle = makeToggle(aContent, "Enable Aimbot", function() return C.Aimbot.Ativo end, function(v) 
    C.Aimbot.Ativo = v
    fovCircle.Visible = v
    crosshair.Visible = v
    if v then Camera.CameraType = Enum.CameraType.Scriptable else Camera.CameraType = Enum.CameraType.Custom end
end, y)
y = y + 45

local targetLbl = Instance.new("TextLabel")
targetLbl.Parent = aContent
targetLbl.Size = UDim2.new(0.6, 0, 0, 35)
targetLbl.Position = UDim2.new(0, 0, 0, y)
targetLbl.Text = "Target Part: Head"
targetLbl.TextColor3 = Color3.new(0.78, 0.78, 0.82)
targetLbl.TextSize = 13
targetLbl.Font = Enum.Font.Gotham
targetLbl.BackgroundTransparency = 1
targetLbl.TextXAlignment = Enum.TextXAlignment.Left
y = y + 40

local wallToggle = makeToggle(aContent, "Wall Check", function() return C.Aimbot.Wall end, function(v) C.Aimbot.Wall = v end, y)
y = y + 45

local fovSlider = makeSlider(aContent, "FOV", 30, 300, 1, function() return C.Aimbot.FOV end, function(v) 
    C.Aimbot.FOV = v
    fovCircle.Size = UDim2.new(0, v*2, 0, v*2)
    fovCircle.Position = UDim2.new(0.5, -v, 0.5, -v)
end, y)
y = y + 55

local colorPicker = makeColorPicker(aContent, "FOV Color", function() return C.Aimbot.Cor end, function(v) 
    C.Aimbot.Cor = v
    fovCircle.ImageColor3 = v
end, y)
y = y + 45

local statusLbl = Instance.new("TextLabel")
statusLbl.Parent = aContent
statusLbl.Size = UDim2.new(1, 0, 0, 25)
statusLbl.Position = UDim2.new(0, 0, 0, y)
statusLbl.Text = "Alvo: Nenhum"
statusLbl.TextColor3 = Color3.new(1, 0.8, 0.2)
statusLbl.TextSize = 13
statusLbl.Font = Enum.Font.Gotham
statusLbl.BackgroundTransparency = 1
statusLbl.TextXAlignment = Enum.TextXAlignment.Left

-- Hitbox
local hContent = Instance.new("Frame")
hContent.Parent = content
hContent.Size = UDim2.new(1, 0, 1, 0)
hContent.BackgroundTransparency = 1
hContent.Visible = false
tabContents["Hitbox"] = hContent

local hy = 0
local hToggle = makeToggle(hContent, "Enable Hitbox", function() return C.Hitbox.Ativo end, function(v) 
    C.Hitbox.Ativo = v
    if v then aplicarHitboxTodos() else restaurarHitboxTodos() end
end, hy)
hy = hy + 45

local hSlider = makeSlider(hContent, "Hitbox Size", 1, 10, 0.5, function() return C.Hitbox.Tamanho end, function(v) 
    C.Hitbox.Tamanho = v
    if C.Hitbox.Ativo then restaurarHitboxTodos() aplicarHitboxTodos() end
end, hy)
hy = hy + 55

local hStatus = Instance.new("TextLabel")
hStatus.Parent = hContent
hStatus.Size = UDim2.new(1, 0, 0, 25)
hStatus.Position = UDim2.new(0, 0, 0, hy)
hStatus.Text = "Status: Aguardando..."
hStatus.TextColor3 = Color3.new(1, 0.8, 0.2)
hStatus.TextSize = 13
hStatus.Font = Enum.Font.Gotham
hStatus.BackgroundTransparency = 1
hStatus.TextXAlignment = Enum.TextXAlignment.Left

-- ESP
local eContent = Instance.new("Frame")
eContent.Parent = content
eContent.Size = UDim2.new(1, 0, 1, 0)
eContent.BackgroundTransparency = 1
eContent.Visible = false
tabContents["ESP"] = eContent

local ey = 0
local eToggle = makeToggle(eContent, "Enemy ESP", function() return C.ESP.Enemy.Ativo end, function(v) 
    C.ESP.Enemy.Ativo = v
    atualizarESP()
end, ey)
ey = ey + 45
local eDist = makeSlider(eContent, "Enemy Distance", 50, 400, 5, function() return C.ESP.Enemy.Dist end, function(v) 
    C.ESP.Enemy.Dist = v
    atualizarESP()
end, ey)
ey = ey + 55

local sep1 = Instance.new("Frame")
sep1.Parent = eContent
sep1.Size = UDim2.new(0.9, 0, 0, 1)
sep1.Position = UDim2.new(0.05, 0, 0, ey)
sep1.BackgroundColor3 = Color3.new(0.24, 0.24, 0.28)
sep1.BackgroundTransparency = 0.5
ey = ey + 15

local pToggle = makeToggle(eContent, "Player ESP", function() return C.ESP.Player.Ativo end, function(v) 
    C.ESP.Player.Ativo = v
    atualizarESP()
end, ey)
ey = ey + 45
local pDist = makeSlider(eContent, "Player Distance", 50, 400, 5, function() return C.ESP.Player.Dist end, function(v) 
    C.ESP.Player.Dist = v
    atualizarESP()
end, ey)
ey = ey + 55

local sep2 = Instance.new("Frame")
sep2.Parent = eContent
sep2.Size = UDim2.new(0.9, 0, 0, 1)
sep2.Position = UDim2.new(0.05, 0, 0, ey)
sep2.BackgroundColor3 = Color3.new(0.24, 0.24, 0.28)
sep2.BackgroundTransparency = 0.5
ey = ey + 15

local iToggle = makeToggle(eContent, "Item ESP", function() return C.ESP.Item.Ativo end, function(v) 
    C.ESP.Item.Ativo = v
    atualizarESP()
end, ey)
ey = ey + 45
local iDist = makeSlider(eContent, "Item Distance", 50, 400, 5, function() return C.ESP.Item.Dist end, function(v) 
    C.ESP.Item.Dist = v
    atualizarESP()
end, ey)

-- Settings
local sContent = Instance.new("Frame")
sContent.Parent = content
sContent.Size = UDim2.new(1, 0, 1, 0)
sContent.BackgroundTransparency = 1
sContent.Visible = false
tabContents["Settings"] = sContent

local sy = 0
local sTitle = Instance.new("TextLabel")
sTitle.Parent = sContent
sTitle.Size = UDim2.new(1, 0, 0, 30)
sTitle.Position = UDim2.new(0, 0, 0, sy)
sTitle.Text = "⚙️ Configurações"
sTitle.TextColor3 = Color3.new(1, 1, 1)
sTitle.TextSize = 18
sTitle.Font = Enum.Font.GothamBold
sTitle.BackgroundTransparency = 1
sTitle.TextXAlignment = Enum.TextXAlignment.Left
sy = sy + 45

local hotkeyLbl = Instance.new("TextLabel")
hotkeyLbl.Parent = sContent
hotkeyLbl.Size = UDim2.new(1, 0, 0, 20)
hotkeyLbl.Position = UDim2.new(0, 0, 0, sy)
hotkeyLbl.Text = "🎮 Teclas: K=Aimbot | H=Hitbox | L=ESP"
hotkeyLbl.TextColor3 = Color3.new(0.7, 0.7, 0.75)
hotkeyLbl.TextSize = 13
hotkeyLbl.Font = Enum.Font.Gotham
hotkeyLbl.BackgroundTransparency = 1
hotkeyLbl.TextXAlignment = Enum.TextXAlignment.Left
sy = sy + 35

local closeBtn = Instance.new("TextButton")
closeBtn.Parent = sContent
closeBtn.Size = UDim2.new(0, 180, 0, 40)
closeBtn.Position = UDim2.new(0.5, -90, 0, sy + 20)
closeBtn.Text = "🔒 FECHAR"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.TextSize = 16
closeBtn.Font = Enum.Font.GothamBold
closeBtn.BackgroundColor3 = Color3.new(0.8, 0.2, 0.2)
closeBtn.BorderSizePixel = 0

local cCorner = Instance.new("UICorner")
cCorner.CornerRadius = UDim.new(0, 10)
cCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
    State.Aberto = false
    panel.Visible = false
    overlayBtn.Visible = true
end)

-- ============================================
-- 5. BOTÃO FLUTUANTE
-- ============================================
local overlayBtn = Instance.new("TextButton")
overlayBtn.Parent = sg
overlayBtn.Size = UDim2.new(0, 50, 0, 50)
overlayBtn.Position = UDim2.new(0.92, -25, 0.08, 0)
overlayBtn.Text = "⚡"
overlayBtn.TextColor3 = Color3.new(1, 1, 1)
overlayBtn.TextSize = 24
overlayBtn.Font = Enum.Font.GothamBold
overlayBtn.BackgroundColor3 = Color3.new(0.12, 0.12, 0.16)
overlayBtn.BackgroundTransparency = 0.2
overlayBtn.BorderSizePixel = 0
overlayBtn.Visible = false

local ovCorner = Instance.new("UICorner")
ovCorner.CornerRadius = UDim.new(1, 0)
ovCorner.Parent = overlayBtn

overlayBtn.MouseButton1Click:Connect(function()
    State.Aberto = true
    panel.Visible = true
    overlayBtn.Visible = false
end)

-- ============================================
-- 6. ELEMENTOS VISUAIS (FOV, Crosshair)
-- ============================================
local fovCircle = Instance.new("ImageLabel")
fovCircle.Parent = sg
fovCircle.Size = UDim2.new(0, C.Aimbot.FOV * 2, 0, C.Aimbot.FOV * 2)
fovCircle.Position = UDim2.new(0.5, -C.Aimbot.FOV, 0.5, -C.Aimbot.FOV)
fovCircle.BackgroundTransparency = 1
fovCircle.Image = "rbxassetid://14594639972"
fovCircle.ImageColor3 = C.Aimbot.Cor
fovCircle.ImageTransparency = 0.6
fovCircle.Visible = false
fovCircle.ZIndex = 10

local crosshair = Instance.new("ImageLabel")
crosshair.Parent = sg
crosshair.Size = UDim2.new(0, 20, 0, 20)
crosshair.Position = UDim2.new(0.5, -10, 0.5, -10)
crosshair.BackgroundTransparency = 1
crosshair.Image = "rbxassetid://14594640640"
crosshair.ImageColor3 = Color3.new(1, 0.2, 0.2)
crosshair.Visible = false
crosshair.ZIndex = 10

-- ============================================
-- 7. FUNÇÕES DO HITBOX
-- ============================================
function aplicarHitbox(char)
    if not char then return 0 end
    local count = 0
    for _, p in ipairs(char:GetDescendants()) do
        if p:IsA("BasePart") then
            if not HitboxCache[p] then HitboxCache[p] = p.Size end
            local o = HitboxCache[p]
            p.Size = Vector3.new(o.X * C.Hitbox.Tamanho, o.Y * C.Hitbox.Tamanho, o.Z * C.Hitbox.Tamanho)
            count = count + 1
        end
    end
    return count
end

function restaurarHitbox(char)
    if not char then return end
    for p, o in pairs(HitboxCache) do
        if p and p.Parent then p.Size = o end
    end
    HitboxCache = {}
end

function aplicarHitboxTodos()
    if not C.Hitbox.Ativo then return end
    local count = 0
    for _, o in pairs(workspace:GetDescendants()) do
        if o:IsA("Model") and not Players:GetPlayerFromCharacter(o) and o:FindFirstChildOfClass("Humanoid") then
            count = count + aplicarHitbox(o)
        end
    end
    if count > 0 then
        hStatus.Text = "✅ Hitbox aplicada (" .. count .. " partes)"
        hStatus.TextColor3 = Color3.new(0.4, 1, 0.4)
    else
        hStatus.Text = "⏳ Nenhum NPC encontrado"
        hStatus.TextColor3 = Color3.new(1, 0.8, 0.2)
    end
end

function restaurarHitboxTodos()
    for _, o in pairs(workspace:GetDescendants()) do
        if o:IsA("Model") and not Players:GetPlayerFromCharacter(o) then
            restaurarHitbox(o)
        end
    end
    HitboxCache = {}
    hStatus.Text = "🔴 Hitbox restaurada"
    hStatus.TextColor3 = Color3.new(1, 0.4, 0.4)
end

-- ============================================
-- 8. FUNÇÕES DO ESP
-- ============================================
function criarHighlight(obj, cor)
    if not obj then return end
    local key = tostring(obj)
    if Highlights[key] then Highlights[key]:Destroy() end
    local hl = Instance.new("Highlight")
    hl.Parent = obj
    hl.FillColor = cor
    hl.FillTransparency = 0.5
    hl.OutlineColor = cor
    hl.OutlineTransparency = 0.3
    hl.DepthMode = Enum.HighlightDepthMode.Occluded
    hl.Adornee = obj
    Highlights[key] = hl
end

function limparHighlights()
    for _, hl in pairs(Highlights) do hl:Destroy() end
    Highlights = {}
end

function atualizarESP()
    limparHighlights()
    if not C.ESP.Enemy.Ativo and not C.ESP.Player.Ativo and not C.ESP.Item.Ativo then return end
    
    local camPos = Camera.CFrame.Position
    
    if C.ESP.Enemy.Ativo then
        for _, o in pairs(workspace:GetDescendants()) do
            if o:IsA("Model") and o:FindFirstChildOfClass("Humanoid") and not Players:GetPlayerFromCharacter(o) then
                local dist = (o.PrimaryPart and (o.PrimaryPart.Position - camPos).Magnitude) or 0
                if dist <= C.ESP.Enemy.Dist then criarHighlight(o, C.ESP.Enemy.Cor) end
            end
        end
    end
    
    if C.ESP.Player.Ativo then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                local c = p.Character
                if c and c:FindFirstChildOfClass("Humanoid") then
                    local dist = (c.PrimaryPart and (c.PrimaryPart.Position - camPos).Magnitude) or 0
                    if dist <= C.ESP.Player.Dist then criarHighlight(c, C.ESP.Player.Cor) end
                end
            end
        end
    end
    
    if C.ESP.Item.Ativo then
        local names = {"Item","Pickup","Loot","Chest","Weapon","Ammo","Health","Shield"}
        for _, o in pairs(workspace:GetDescendants()) do
            if o:IsA("BasePart") or o:IsA("Model") then
                local n = o.Name:lower()
                for _, item in ipairs(names) do
                    if n:find(item:lower()) then
                        local dist = (o.Position - camPos).Magnitude or 0
                        if dist <= C.ESP.Item.Dist then criarHighlight(o, C.ESP.Item.Cor) end
                        break
                    end
                end
            end
        end
    end
end

-- ============================================
-- 9. LÓGICA DO AIMBOT
-- ============================================
function isNPC(model)
    if not model or not model:IsA("Model") then return false end
    local h = model:FindFirstChildOfClass# Script.lua
Brother's vow
