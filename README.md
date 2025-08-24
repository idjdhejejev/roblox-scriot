local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")

-- Configurações do menu
local showKeys = false
local showEntities = false
local entityWarning = false

-- Controle de brilho para feedback visual
local maxBrightness = false
local function setBrightness(value, time)
    local tween = TweenService:Create(
        Lighting,
        TweenInfo.new(time or 1, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
        {Brightness = value}
    )
    tween:Play()
end

-- Função ESP (highlight + círculo)
local function highlight(obj, color)
    if not obj then return end

    if not obj:FindFirstChild("Highlight") then
        local hl = Instance.new("Highlight", obj)
        hl.FillColor = color
        hl.OutlineColor = Color3.new(1,1,1)
        hl.FillTransparency = 0.5
        hl.OutlineTransparency = 0
    end

    if not obj:FindFirstChild("CircleESP") then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "CircleESP"
        billboard.AlwaysOnTop = true
        billboard.Size = UDim2.new(3,0,3,0)
        billboard.StudsOffset = Vector3.new(0,2,0)

        if obj:IsA("Model") and obj:FindFirstChild("HumanoidRootPart") then
            billboard.Adornee = obj.HumanoidRootPart
            billboard.Parent = obj.HumanoidRootPart
        elseif obj:IsA("BasePart") then
            billboard.Adornee = obj
            billboard.Parent = obj
        else
            billboard:Destroy()
            return
        end

        local circle = Instance.new("ImageLabel", billboard)
        circle.BackgroundTransparency = 1
        circle.Size = UDim2.new(1,0,1,0)
        circle.Image = "rbxassetid://4695575676"
        circle.ImageColor3 = color
        circle.ImageTransparency = 0.5
    end
end

-- Função para mostrar notificação personalizada
local function notify(message, duration)
    StarterGui:SetCore("SendNotification", {
        Title = "DOORS ESP";
        Text = message;
        Duration = duration or 3;
    })
end

-- Detecta objetos de interesse
local function scanForInterest()
    for _, obj in ipairs(workspace:GetDescendants()) do
        local name = obj.Name:lower()
        if showKeys and name:find("key") then
            highlight(obj, Color3.new(1, 1, 0)) -- Amarelo para chaves
        end

        if showEntities and (name:find("entity") or name:find("mob") or name:find("rush") or name:find("ambush")) then
            highlight(obj, Color3.new(1, 0, 0)) -- Vermelho para entidades

            if entityWarning then
                task.spawn(function()
                    while obj and obj:IsDescendantOf(workspace) and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") do
                        local hrp = LocalPlayer.Character.HumanoidRootPart
                        local dist = (hrp.Position - obj.Position).Magnitude
                        if dist <= 20 then
                            notify("⚠ Entidade próxima! Se esconda!", 3)

                            if not maxBrightness then
                                maxBrightness = true
                                setBrightness(7, 0.5) 
                                task.delay(3, function()
                                    setBrightness(2, 1.5) 
                                    maxBrightness = false
                                end)
                            end
                            break
                        end
                        task.wait(1)
                    end
                end)
            end
        end
    end
end

-- Loop de varredura
task.spawn(function()
    while true do
        scanForInterest()
        task.wait(5)
    end
end)

-- =========================
-- 📌 Criação do Menu (ScreenGui)
-- =========================
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "DoorsESPMenu"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 150)
Frame.Position = UDim2.new(0, 20, 0.3, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BackgroundTransparency = 0.2

local UIListLayout = Instance.new("UIListLayout", Frame)
UIListLayout.Padding = UDim.new(0, 5)

-- Botão genérico
local function createButton(text, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Text = text
    btn.MouseButton1Click:Connect(callback)
end

-- Botões do menu
createButton("🔑 Toggle Keys ESP", function()
    showKeys = not showKeys
    notify("Keys ESP: " .. tostring(showKeys), 2)
end)

createButton("👻 Toggle Entities ESP", function()
    showEntities = not showEntities
    notify("Entities ESP: " .. tostring(showEntities), 2)
end)

createButton("⚠ Toggle Entity Warning", function()
    entityWarning = not entityWarning
    notify("Entity Warning: " .. tostring(entityWarning), 2)
end)
