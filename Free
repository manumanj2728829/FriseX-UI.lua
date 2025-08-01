-- Carrega Rayfield UI
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Lighting = game:GetService("Lighting")
local UserInputService = game:GetService("UserInputService")

-- Config geral
local Config = {
    -- Hitbox
    HitboxEnabled = false,
    HitboxMode = "Legit",
    HitboxSize = Vector3.new(3.5, 3.5, 3.5),
    HitboxTransparency = 0.5,

    -- ESP
    ESPEnabled = false,
    BoxESP = false,
    NameESP = false,
    TracerESP = false,
    HealthBarESP = false,
    ShowDistance = true,
    ShowWeapon = true,

    -- Chams
    ChamsEnabled = false,
    RGBMode = false,
    RGBSpeed = 0.6,
    ChamsColor = Color3.fromRGB(255, 0, 255),

    -- FOV
    FOVCircle = false,
    FOVRadius = 80,
    FOVColor = Color3.fromRGB(255, 0, 255),

    -- XRAY
    XRAYEnabled = false,
    XRAYTransparency = 0.2,

    -- Ambiente
    AmbientLighting = false,
    AmbientColor = Color3.fromRGB(255,255,255),
    AmbientBrightness = 2.5,
}

-- Armazenamentos
local Highlights = {}
local ESPObjects = {}
local OriginalHitboxes = {}
local XRayed = {}
local OriginalAmbient = Lighting.Ambient
local OriginalBrightness = Lighting.Brightness

-- Rainbow util
local function getRainbowColor(speed)
    local t = tick() * speed
    return Color3.fromHSV((t % 5) / 5, 1, 1)
end

-- Hitbox Modos
local HitboxModes = {
    Legit = {Size = Vector3.new(3.5,3.5,3.5), Transparency = 0.5},
    Rage = {Size = Vector3.new(10,10,10), Transparency = 0.2},
}

-- Aplica hitbox sem travar
local function applyHitbox(player)
    if player == LocalPlayer then return end
    local char = player.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head or not head:IsA("BasePart") then return end
    OriginalHitboxes[player] = OriginalHitboxes[player] or {}
    if not OriginalHitboxes[player].Size then
        OriginalHitboxes[player].Size = head.Size
        OriginalHitboxes[player].Transparency = head.Transparency
        OriginalHitboxes[player].CanCollide = head.CanCollide
        OriginalHitboxes[player].Massless = head.Massless
    end
    local size = Config.HitboxSize
    local transparency = Config.HitboxTransparency
    pcall(function()
        head.Size = size
        head.Transparency = transparency
        head.CanCollide = false
        head.Massless = true
    end)
end

local function resetHitbox(player)
    local char = player.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head or not head:IsA("BasePart") then return end
    if OriginalHitboxes[player] then
        pcall(function()
            head.Size = OriginalHitboxes[player].Size or Vector3.new(2,1,1)
            head.Transparency = OriginalHitboxes[player].Transparency or 0
            head.CanCollide = OriginalHitboxes[player].CanCollide or true
            head.Massless = OriginalHitboxes[player].Massless or false
        end)
        OriginalHitboxes[player] = nil
    end
end

-- Clear hitboxes ao remover
Players.PlayerRemoving:Connect(function(plr)
    resetHitbox(plr)
    if Highlights[plr] then
        Highlights[plr]:Destroy()
        Highlights[plr] = nil
    end
    if ESPObjects[plr] then
        for _, obj in pairs(ESPObjects[plr]) do
            if obj.Remove then obj:Remove() end
        end
        ESPObjects[plr] = nil
    end
end)

-- Chams (Highlight simples)
local function clearChams(player)
    if Highlights[player] then
        if Highlights[player].Parent then
            Highlights[player]:Destroy()
        end
        Highlights[player] = nil
    end
end

local function applyChams(player)
    if not Config.ChamsEnabled or not Config.ESPEnabled then
        clearChams(player)
        return
    end
    local char = player.Character
    if not char then clearChams(player); return end

    local highlight = Highlights[player]
    if not highlight or not highlight.Parent then
        highlight = Instance.new("Highlight")
        highlight.Name = "CustomHighlight"
        highlight.Adornee = char
        highlight.Parent = char
        Highlights[player] = highlight
    end

    local color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.ChamsColor
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = 1
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
end

local function updateChamsRGB()
    for player, highlight in pairs(Highlights) do
        if highlight and highlight.Parent then
            local color = getRainbowColor(Config.RGBSpeed)
            highlight.FillColor = color
            highlight.OutlineColor = color
        end
    end
end

-- ESP usando Drawing
local function newDrawing(class, props)
    local d = Drawing.new(class)
    for i,v in pairs(props) do d[i] = v end
    return d
end

local function createESP(player)
    local box = newDrawing("Square", {Visible=false, Thickness=2, Filled=false})
    local name = newDrawing("Text", {Visible=false, Center=true, Outline=true, Size=16, Font=2})
    local tracer = newDrawing("Line", {Visible=false, Thickness=1})
    local health = newDrawing("Line", {Visible=false, Thickness=4})
    local distance = newDrawing("Text", {Visible=false, Center=true, Outline=true, Size=14, Font=2})
    local weapon = newDrawing("Text", {Visible=false, Center=true, Outline=true, Size=14, Font=2})
    return {Box=box, Name=name, Tracer=tracer, Health=health, Distance=distance, Weapon=weapon}
end

local function hideESP(data)
    if not data then return end
    for _, obj in pairs(data) do
        obj.Visible = false
    end
end

-- XRAY
local function enableXRAY(transparency)
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and not v:IsDescendantOf(LocalPlayer.Character) then
            if not XRayed[v] then
                XRayed[v] = v.Transparency
            end
            v.LocalTransparencyModifier = transparency or Config.XRAYTransparency
        end
    end
end

local function disableXRAY()
    for v, orig in pairs(XRayed) do
        if v and v:IsA("BasePart") then
            v.LocalTransparencyModifier = orig or 0
        end
    end
    XRayed = {}
end

-- FOV Circle
local FOVCircle = newDrawing("Circle", {
    Thickness = 1.8,
    Color = Config.FOVColor,
    Filled = false,
    Radius = Config.FOVRadius,
    Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2),
    Visible = Config.FOVCircle
})

local FOVLine = newDrawing("Line", {
    Color = Config.FOVColor,
    Thickness = 1.5,
    Visible = false
})

-- Função para obter parte alvo
local function getTargetPart(char)
    local parts = {"Head","UpperTorso","Torso","HumanoidRootPart"}
    for _, partName in ipairs(parts) do
        local part = char:FindFirstChild(partName)
        if part and part:IsA("BasePart") then
            return part
        end
    end
    return nil
end

-- Obter jogador mais próximo ao centro da FOV
local function getClosestTarget()
    local closest
    local minDist = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local part = getTargetPart(player.Character)
            if part then
                local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - FOVCircle.Position).Magnitude
                    if dist < Config.FOVRadius and dist < minDist then
                        closest = player
                        minDist = dist
                    end
                end
            end
        end
    end
    return closest
end

-- Atualiza iluminação ambiente
local function updateLighting()
    if Config.AmbientLighting then
        Lighting.Ambient = Config.AmbientColor
        Lighting.Brightness = Config.AmbientBrightness
    else
        Lighting.Ambient = OriginalAmbient
        Lighting.Brightness = OriginalBrightness
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    updateLighting()

    -- Atualiza FOV Circle
    FOVCircle.Visible = Config.FOVCircle
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    FOVCircle.Color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.FOVColor
    FOVCircle.Radius = Config.FOVRadius

    -- XRAY
    if Config.XRAYEnabled then
        enableXRAY(Config.XRAYTransparency)
    else
        disableXRAY()
    end

    -- Loop hitbox para todos os players
    if Config.HitboxEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            applyHitbox(player)
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            resetHitbox(player)
        end
    end

    -- Aimbot (apenas mirando)
    local target = nil -- Desabilitado, mas pode ativar aqui se quiser

    if target and target.Character then
        local part = getTargetPart(target.Character)
        if part then
            local screenPos = Camera:WorldToViewportPoint(part.Position)
            local dir = (part.Position - Camera.CFrame.Position).Unit
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir), 0.15)

            if Config.FOVCircle then
                FOVLine.Visible = true
                FOVLine.From = FOVCircle.Position
                FOVLine.To = Vector2.new(screenPos.X, screenPos.Y)
                FOVLine.Color = FOVCircle.Color
            end
        end
    else
        FOVLine.Visible = false
    end

    -- ESP e Chams
    for _, player in pairs(Players:GetPlayers()) do
        local char = player.Character
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")
        if player ~= LocalPlayer and char and humanoid and humanoid.Health > 0 then
            if Config.ESPEnabled then
                -- Chams
                if Config.ChamsEnabled then
                    applyChams(player)
                else
                    clearChams(player)
                end

                -- ESP Box, Name, Tracer, Health Bar, Distance, Weapon
                if not ESPObjects[player] then ESPObjects[player] = createESP(player) end
                local boxData = ESPObjects[player]

                local success, cf, size = pcall(char.GetBoundingBox, char)
                if success then
                    local points = {}
                    local visible = true
                    for x = -1,1,2 do
                        for y = -1,1,2 do
                            for z = -1,1,2 do
                                local corner = cf * Vector3.new(size.X/2 * x, size.Y/2 * y, size.Z/2 * z)
                                local pos, onScreen = Camera:WorldToViewportPoint(corner)
                                if not onScreen then visible = false end
                                table.insert(points, Vector2.new(pos.X, pos.Y))
                            end
                        end
                    end
                    if visible then
                        local minX, minY = math.huge, math.huge
                        local maxX, maxY = -math.huge, -math.huge
                        for _, pt in pairs(points) do
                            minX = math.min(minX, pt.X)
                            minY = math.min(minY, pt.Y)
                            maxX = math.max(maxX, pt.X)
                            maxY = math.max(maxY, pt.Y)
                        end
                        local width, height = maxX - minX, maxY - minY
                        local midX = minX + width / 2

                        -- Box ESP
                        boxData.Box.Visible = Config.BoxESP
                        boxData.Box.Position = Vector2.new(minX, minY)
                        boxData.Box.Size = Vector2.new(width, height)
                        boxData.Box.Color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.FOVColor

                        -- Name ESP
                        boxData.Name.Visible = Config.NameESP
                        boxData.Name.Text = player.Name
                        boxData.Name.Position = Vector2.new(midX, minY - 18)
                        boxData.Name.Color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.FOVColor

                        -- Tracer ESP
                        boxData.Tracer.Visible = Config.TracerESP
                        boxData.Tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                        boxData.Tracer.To = Vector2.new(midX, maxY)
                        boxData.Tracer.Color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.FOVColor

                        -- Health Bar ESP
                        local ratio = humanoid.Health / humanoid.MaxHealth
                        boxData.Health.Visible = Config.HealthBarESP
                        if Config.RGBMode then
                            boxData.Health.Color = getRainbowColor(Config.RGBSpeed)
                        else
                            boxData.Health.Color = ratio > 0.7 and Color3.new(0,1,0) or ratio > 0.3 and Color3.new(1,1,0) or Color3.new(1,0,0)
                        end
                        boxData.Health.From = Vector2.new(minX - 5, maxY)
                        boxData.Health.To = Vector2.new(minX - 5, maxY - height * ratio)

                        -- Distance
                        local dist = (Camera.CFrame.Position - char.HumanoidRootPart.Position).Magnitude
                        boxData.Distance.Visible = Config.ShowDistance
                        boxData.Distance.Text = ("[%.1f m]"):format(dist)
                        boxData.Distance.Position = Vector2.new(midX, maxY + 16)
                        boxData.Distance.Color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.FOVColor

                        -- Weapon Name
                        local weaponName = ""
                        if Config.ShowWeapon then
                            for _, v in pairs(char:GetChildren()) do
                                if v:IsA("Tool") or (v:IsA("Model") and (v:FindFirstChild("GunScript") or v:FindFirstChild("Handle"))) then
                                    weaponName = v.Name
                                    break
                                end
                            end
                        end
                        boxData.Weapon.Visible = Config.ShowWeapon and weaponName ~= ""
                        boxData.Weapon.Text = weaponName
                        boxData.Weapon.Position = Vector2.new(midX, minY - 34)
                        boxData.Weapon.Color = Config.RGBMode and getRainbowColor(Config.RGBSpeed) or Config.FOVColor
                    else
                        hideESP(boxData)
                    end
                else
                    hideESP(boxData)
                end
            else
                -- Limpar ESP e Chams
                clearChams(player)
                if ESPObjects[player] then
                    hideESP(ESPObjects[player])
                end
            end
        else
            if ESPObjects[player] then
                hideESP(ESPObjects[player])
            end
            clearChams(player)
        end
    end

    -- Atualiza cor RGB dos chams
    if Config.ChamsEnabled and Config.RGBMode then
        updateChamsRGB()
    end
end)

-- Interface Rayfield

local Window = Rayfield:CreateWindow({
    Name = "ESP & Hitbox",
    LoadingTitle = "Carregando ESP & Hitbox",
    LoadingSubtitle = "por manumanj2728829"
})

local TabESP = Window:CreateTab("ESP & Hitbox", 4483362458)

-- Hitbox controls
TabESP:CreateToggle({
    Name = "Ativar Hitbox",
    CurrentValue = Config.HitboxEnabled,
    Callback = function(value)
        Config.HitboxEnabled = value
    end
})

TabESP:CreateDropdown({
    Name = "Modo Hitbox",
    Options = {"Legit", "Rage"},
    CurrentOption = Config.HitboxMode,
    Callback = function(value)
        Config.HitboxMode = value
        local modeData = HitboxModes[value]
        Config.HitboxSize = modeData.Size
        Config.HitboxTransparency = modeData.Transparency
    end
})

TabESP:CreateSlider({
    Name = "Tamanho Hitbox",
    Range = {1, 15},
    Increment = 0.5,
    CurrentValue = Config.HitboxSize.X,
    Callback = function(value)
        Config.HitboxSize = Vector3.new(value, value, value)
    end
})

TabESP:CreateSlider({
    Name = "Transparência Hitbox",
    Range = {0, 1},
    Increment = 0.05,
    CurrentValue = Config.HitboxTransparency,
    Callback = function(value)
        Config.HitboxTransparency = value
    end
})

-- ESP Toggles
TabESP:CreateToggle({
    Name = "Ativar ESP",
    CurrentValue = Config.ESPEnabled,
    Callback = function(value)
        Config.ESPEnabled = value
    end
})

TabESP:CreateToggle({
    Name = "Box ESP",
    CurrentValue = Config.BoxESP,
    Callback = function(value)
        Config.BoxESP = value
    end
})

TabESP:CreateToggle({
    Name = "Nome ESP",
    CurrentValue = Config.NameESP,
    Callback = function(value)
        Config.NameESP = value
    end
})

TabESP:CreateToggle({
    Name = "Tracer ESP",
    CurrentValue = Config.TracerESP,
    Callback = function(value)
        Config.TracerESP = value
    end
})

TabESP:CreateToggle({
    Name = "Barra de Vida ESP",
    CurrentValue = Config.HealthBarESP,
    Callback = function(value)
        Config.HealthBarESP = value
    end
})

TabESP:CreateToggle({
    Name = "Mostrar Distância",
    CurrentValue = Config.ShowDistance,
    Callback = function(value)
        Config.ShowDistance = value
    end
})

TabESP:CreateToggle({
    Name = "Mostrar Arma",
    CurrentValue = Config.ShowWeapon,
    Callback = function(value)
        Config.ShowWeapon = value
    end
})

-- Chams Controls
TabESP:CreateToggle({
    Name = "Ativar Chams",
    CurrentValue = Config.ChamsEnabled,
    Callback = function(value)
        Config.ChamsEnabled = value
    end
})

TabESP:CreateToggle({
    Name = "Modo RGB Chams",
    CurrentValue = Config.RGBMode,
    Callback = function(value)
        Config.RGBMode = value
    end
})

TabESP:CreateSlider({
    Name = "Velocidade RGB",
    Range = {0.1, 2},
    Increment = 0.05,
    CurrentValue = Config.RGBSpeed,
    Callback = function(value)
        Config.RGBSpeed = value
    end
})

TabESP:CreateColorPicker({
    Name = "Cor Chams",
    Color = Config.ChamsColor,
    Callback = function(color)
        Config.ChamsColor = color
    end
})

-- FOV Controls
TabESP:CreateToggle({
    Name = "Mostrar FOV",
    CurrentValue = Config.FOVCircle,
    Callback = function(value)
        Config.FOVCircle = value
    end
})

TabESP:CreateSlider({
    Name = "Raio do FOV",
    Range = {20, 200},
    Increment = 1,
    CurrentValue = Config.FOVRadius,
    Callback = function(value)
        Config.FOVRadius = value
    end
})

TabESP:CreateColorPicker({
    Name = "Cor do FOV",
    Color = Config.FOVColor,
    Callback = function(color)
        Config.FOVColor = color
    end
})

-- XRAY Controls
TabESP:CreateToggle({
    Name = "Ativar XRAY",
    CurrentValue = Config.XRAYEnabled,
    Callback = function(value)
        Config.XRAYEnabled = value
    end
})

TabESP:CreateSlider({
    Name = "Transparência XRAY",
    Range = {0, 1},
    Increment = 0.05,
    CurrentValue = Config.XRAYTransparency,
    Callback = function(value)
        Config.XRAYTransparency = value
    end
})

-- Ambiente Controls
TabESP:CreateToggle({
    Name = "Iluminação Ambiente",
    CurrentValue = Config.AmbientLighting,
    Callback = function(value)
        Config.AmbientLighting = value
    end
})

TabESP:CreateColorPicker({
    Name = "Cor Ambiente",
    Color = Config.AmbientColor,
    Callback = function(color)
        Config.AmbientColor = color
    end
})

TabESP:CreateSlider({
    Name = "Brilho Ambiente",
    Range = {0, 10},
    Increment = 0.1,
    CurrentValue = Config.AmbientBrightness,
    Callback = function(value)
        Config.AmbientBrightness = value
    end
})
