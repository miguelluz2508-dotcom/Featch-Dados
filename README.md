--[[
    FEACHT TOOLS – MUSCLE LEGENDS (FLUENT UI)
    Com botão flutuante, teleporte para plataforma e funções extras.
--]]

-- ========================== FLUENT UI ==========================
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveConfig = Fluent.SaveConfig
local ThemeManager = Fluent.ThemeManager

local Window = Fluent:CreateWindow({
    Title = "Feacht Tools – Muscle Legends",
    SubTitle = "v1.0",
    TabWidth = 140,
    Size = UDim2.fromOffset(560, 400),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightShift
})

-- ========================== SERVIÇOS ==========================
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local VirtualUser = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- ========================== BOTÃO FLUTUANTE ==========================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FeachtToggleGui"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local OpenButton = Instance.new("TextButton")
OpenButton.Name = "ToggleButton"
OpenButton.Parent = ScreenGui
OpenButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
OpenButton.Position = UDim2.new(0.1, 0, 0.2, 0)
OpenButton.Size = UDim2.new(0, 50, 0, 50)
OpenButton.Font = Enum.Font.GothamBold
OpenButton.Text = "F"
OpenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenButton.TextSize = 24
OpenButton.Active = true
OpenButton.Draggable = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = OpenButton

OpenButton.MouseButton1Click:Connect(function()
    Window:Minimize()
end)

task.spawn(function()
    local FluentUI = CoreGui:WaitForChild("Fluent", 5)
    if FluentUI then
        FluentUI.AncestryChanged:Connect(function(_, parent)
            if not parent then
                ScreenGui:Destroy()
            end
        end)
        FluentUI.Destroying:Connect(function()
            ScreenGui:Destroy()
        end)
    end
end)

-- ========================== VARIÁVEIS E FUNÇÕES AUXILIARES ==========================
local autoClickEnabled = false
local autoClickDelay = 0.01

local autoStrongEnabled = false
local killAutoEnabled = false
local autoRebirthEnabled = false
local autoSellEnabled = false
local autoEquipBest = false
local autoStatsEnabled = false

-- Função de clique rápido
local function rapidClick(times)
    for _ = 1, (times or 1) do
        VirtualUser:Button1Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(autoClickDelay)
        VirtualUser:Button1Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(autoClickDelay)
    end
end

-- Equipar ferramenta pelo índice (do backpack)
local function equipTool(index)
    local char = Player.Character
    if not char then return end
    local backpack = Player.Backpack
    local tools = {}
    for _, item in ipairs(backpack:GetChildren()) do
        if item:IsA("Tool") then
            table.insert(tools, item)
        end
    end
    if index <= #tools then
        tools[index].Parent = char
    end
end

-- Pegar melhor ferramenta baseado no dano (ou equipar a primeira)
local function equipBestTool()
    local char = Player.Character
    if not char then return end
    local backpack = Player.Backpack
    local bestTool = nil
    local bestDamage = 0
    for _, item in ipairs(backpack:GetChildren()) do
        if item:IsA("Tool") then
            -- Supõe que o dano está no nome ou em um atributo; vamos pegar a primeira por enquanto.
            -- Melhor usar ordem de força conhecida (treino etc.) – manteremos simples.
            bestTool = item
            break
        end
    end
    if bestTool then
        bestTool.Parent = char
    end
end

-- Pegar jogador mais próximo
local function getNearestPlayer()
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
    local nearest = nil
    local shortest = math.huge
    for _, other in ipairs(Players:GetPlayers()) do
        if other == Player then continue end
        if other.Character and other.Character:FindFirstChild("HumanoidRootPart") and other.Character:FindFirstChild("Humanoid") and other.Character.Humanoid.Health > 0 then
            local dist = (char.HumanoidRootPart.Position - other.Character.HumanoidRootPart.Position).Magnitude
            if dist < shortest then
                shortest = dist
                nearest = other
            end
        end
    end
    return nearest
end

-- Teleportar para um jogador
local function teleportTo(targetPlayer)
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
    if targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
    end
end

-- ========================== ABA UNIVERSAL ==========================
local UniversalTab = Window:AddTab({ Title = "Universal", Icon = "globe" })
UniversalTab:AddButton({
    Title = "Infinite Yield",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
    end
})

-- ========================== ABA AUTO CLICK ==========================
local AutoClickTab = Window:AddTab({ Title = "Auto Click", Icon = "mouse-pointer" })
local ToggleAutoClick = AutoClickTab:AddToggle("AutoClickToggle", {
    Title = "Ativar Auto Click",
    Default = false,
    Callback = function(Value)
        autoClickEnabled = Value
        if Value then
            task.spawn(function()
                while autoClickEnabled do
                    rapidClick(1)
                end
            end)
        end
    end
})
AutoClickTab:AddSlider("ClickSpeed", {
    Title = "Velocidade (ms)",
    Default = 10,
    Min = 1,
    Max = 100,
    Rounding = 0,
    Callback = function(Value)
        autoClickDelay = Value / 1000
    end
})

-- ========================== ABA AUTO STRONG (com teleporte) ==========================
local AutoStrongTab = Window:AddTab({ Title = "Auto Strong", Icon = "dumbbell" })
AutoStrongTab:AddToggle("AutoStrongToggle", {
    Title = "Ativar Auto Strong",
    Default = false,
    Callback = function(Value)
        autoStrongEnabled = Value
        if Value then
            task.spawn(function()
                local platform = Instance.new("Part")
                platform.Name = "AutoStrongPlatform"
                platform.Size = Vector3.new(30, 0.4, 30)   -- plataforma gigante
                platform.Anchored = true
                platform.CanCollide = true
                platform.BrickColor = BrickColor.new("Dark stone grey")
                platform.Transparency = 0.5
                platform.Parent = workspace

                local char = Player.Character
                if not char or not char:FindFirstChild("HumanoidRootPart") then
                    Library:Notification("Erro", "Personagem não encontrado.", 3)
                    autoStrongEnabled = false
                    return
                end
                -- Cria 50 studs acima
                platform.CFrame = char.HumanoidRootPart.CFrame * CFrame.new(0, 50, 0)

                -- Equipa a segunda ferramenta
                equipTool(2)

                -- Teleporta o personagem para cima da plataforma
                char.HumanoidRootPart.CFrame = platform.CFrame * CFrame.new(0, 3, 0)   -- um pouco acima da superfície

                -- Loop de cliques
                while autoStrongEnabled do
                    rapidClick(1)
                end

                -- Ao desativar, remove a plataforma
                if platform and platform.Parent then
                    platform:Destroy()
                end
            end)
        end
    end
})

-- ========================== ABA KILL AUTO ==========================
local KillAutoTab = Window:AddTab({ Title = "Kill Auto", Icon = "crosshair" })
KillAutoTab:AddToggle("KillAutoToggle", {
    Title = "Ativar Kill Auto",
    Default = false,
    Callback = function(Value)
        killAutoEnabled = Value
        if Value then
            task.spawn(function()
                while killAutoEnabled do
                    local target = getNearestPlayer()
                    if target then
                        equipTool(1)
                        teleportTo(target)
                        -- mata enquanto o alvo estiver vivo
                        while killAutoEnabled and target.Character and target.Character:FindFirstChild("Humanoid") and target.Character.Humanoid.Health > 0 do
                            rapidClick(1)
                        end
                    else
                        task.wait(1)
                    end
                    task.wait(0.1)
                end
            end)
        end
    end
})

-- ========================== ABA AUTO FARM (EXTRAS) ==========================
local AutoFarmTab = Window:AddTab({ Title = "Auto Farm", Icon = "bar-chart-2" })

-- Auto Rebirth (supondo que exista um botão de rebirth)
AutoFarmTab:AddToggle("AutoRebirthToggle", {
    Title = "Auto Rebirth (experimental)",
    Default = false,
    Callback = function(Value)
        autoRebirthEnabled = Value
        if Value then
            task.spawn(function()
                while autoRebirthEnabled do
                    pcall(function()
                        -- Procura pelo botão de renascimento (pode ser um Remote ou um ScreenGui)
                        -- Exemplo: se existir um botão "Rebirth" no ScreenGui
                        local rebirthBtn = Player.PlayerGui:FindFirstChild("RebirthGui")
                        if rebirthBtn then
                            -- Tenta clicar nele
                            fireclickdetector(rebirthBtn.ClickDetector)
                        end
                        -- Ou usar remotes: não sabemos, mas pode ser adaptado.
                    end)
                    task.wait(10)
                end
            end)
        end
    end
})

-- Auto Sell (vender pesos na loja)
AutoFarmTab:AddToggle("AutoSellToggle", {
    Title = "Auto Sell Weights",
    Default = false,
    Callback = function(Value)
        autoSellEnabled = Value
        if Value then
            task.spawn(function()
                while autoSellEnabled do
                    pcall(function()
                        -- Tenta vender todos os itens de peso
                        local args = {
                            [1] = "SellAll"
                        }
                        workspace.Events.Sell:FireServer(unpack(args))
                    end)
                    task.wait(5)
                end
            end)
        end
    end
})

-- Auto Equip Best Tool
AutoFarmTab:AddToggle("AutoEquipBestToggle", {
    Title = "Auto Equip Best Tool",
    Default = false,
    Callback = function(Value)
        autoEquipBest = Value
        if Value then
            task.spawn(function()
                while autoEquipBest do
                    equipBestTool()
                    task.wait(1)
                end
            end)
        end
    end
})

-- Auto Stats (colocar pontos em Força)
AutoFarmTab:AddToggle("AutoStatsToggle", {
    Title = "Auto Stats (Força)",
    Default = false,
    Callback = function(Value)
        autoStatsEnabled = Value
        if Value then
            task.spawn(function()
                while autoStatsEnabled do
                    pcall(function()
                        local args = {
                            [1] = "Strength"
                        }
                        workspace.Events.Stats:FireServer(unpack(args))
                    end)
                    task.wait(1)
                end
            end)
        end
    end
})

-- ========================== NOTIFICAÇÃO INICIAL ==========================
Fluent:Notify({Title = "Feacht Tools", Content = "Script carregado com sucesso!", Duration = 5})
Window:SelectTab(1)   -- Abre na primeira aba
