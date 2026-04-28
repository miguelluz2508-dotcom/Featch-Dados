--[[
    FEACHT TOOLS - MUSCLE LEGENDS
    Kavo UI Original
    Abas: Universal | Auto Click | Auto Strong | Kill Auto
--]]

-- ========================== KAVO UI ==========================
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("FEACHT TOOLS", "Serpent")

-- ========================== SERVIÇOS ==========================
local Players = game:GetService("Players")
local VirtualUser = game:GetService("VirtualUser")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- ========================== ABA UNIVERSAL ==========================
local UniversalTab = Window:NewTab("Universal")
local UniversalSection = UniversalTab:NewSection("Scripts")

UniversalSection:NewButton("Infinite Yield", "Carrega o Infinite Yield", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
end)

-- ========================== ABA AUTO CLICK ==========================
local AutoClickTab = Window:NewTab("Auto Click")
local AutoClickSection = AutoClickTab:NewSection("Configuração")

local autoClickEnabled = false
local autoClickDelay = 0.01 -- intervalo entre cliques (segundos)

AutoClickSection:NewToggle("Ativar Auto Click", false, function(state)
    autoClickEnabled = state
    if state then
        task.spawn(function()
            while autoClickEnabled do
                VirtualUser:Button1Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(autoClickDelay)
                VirtualUser:Button1Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(autoClickDelay)
            end
        end)
    end
end)

AutoClickSection:NewSlider("Velocidade (ms)", 10, 1, 100, false, function(value)
    autoClickDelay = value / 1000 -- converte ms para segundos
end)

-- ========================== ABA AUTO STRONG ==========================
local AutoStrongTab = Window:NewTab("Auto Strong")
local AutoStrongSection = AutoStrongTab:NewSection("Treino Automático")

local autoStrongEnabled = false

local function equipTool(toolIndex)
    local character = LocalPlayer.Character
    if not character then return end
    local backpack = LocalPlayer.Backpack
    local tools = {}
    -- pega tools do backpack
    for _, item in ipairs(backpack:GetChildren()) do
        if item:IsA("Tool") then
            table.insert(tools, item)
        end
    end
    if toolIndex <= #tools then
        -- equipa a ferramenta
        local tool = tools[toolIndex]
        tool.Parent = character
    end
end

local function createPlatform()
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local platform = Instance.new("Part")
    platform.Name = "AutoStrongPlatform"
    platform.Size = Vector3.new(10, 0.4, 10)
    platform.Anchored = true
    platform.CanCollide = true
    platform.BrickColor = BrickColor.new("Dark stone grey")
    platform.Transparency = 0.5
    platform.CFrame = character.HumanoidRootPart.CFrame + Vector3.new(0, 50, 0) -- 50 studs acima
    platform.Parent = workspace
    return platform
end

AutoStrongSection:NewToggle("Ativar Auto Strong", false, function(state)
    autoStrongEnabled = state
    if state then
        task.spawn(function()
            local platform = createPlatform()
            if not platform then
                Library:Notification("Erro", "Não foi possível criar a plataforma.", 3)
                autoStrongEnabled = false
                return
            end
            -- equipa a segunda ferramenta do inventário
            equipTool(2)
            -- loop de cliques
            while autoStrongEnabled do
                VirtualUser:Button1Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(0.01)
                VirtualUser:Button1Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(0.01)
            end
            -- ao desativar, remove a plataforma
            if platform and platform.Parent then
                platform:Destroy()
            end
        end)
    end
end)

-- ========================== ABA KILL AUTO ==========================
local KillAutoTab = Window:NewTab("Kill Auto")
local KillAutoSection = KillAutoTab:NewSection("Assassino Automático")

local killAutoEnabled = false

local function getNearestPlayer()
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return nil end
    local nearest = nil
    local shortestDist = math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local dist = (character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
            if dist < shortestDist then
                shortestDist = dist
                nearest = player
            end
        end
    end
    return nearest
end

local function teleportTo(targetPlayer)
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return false end
    if targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetPos = targetPlayer.Character.HumanoidRootPart.Position
        character.HumanoidRootPart.CFrame = CFrame.new(targetPos + Vector3.new(0, 2, 0)) -- um pouco acima
        return true
    end
    return false
end

KillAutoSection:NewToggle("Ativar Kill Auto", false, function(state)
    killAutoEnabled = state
    if state then
        task.spawn(function()
            while killAutoEnabled do
                local target = getNearestPlayer()
                if target then
                    -- equipa a primeira ferramenta
                    equipTool(1)
                    -- teleporta até o alvo
                    local success = teleportTo(target)
                    if not success then
                        task.wait(0.5)
                        continue
                    end
                    -- loop de cliques até o alvo morrer
                    local targetAlive = true
                    while targetAlive and killAutoEnabled do
                        if not target.Character or not target.Character:FindFirstChild("Humanoid") or target.Character.Humanoid.Health <= 0 then
                            targetAlive = false
                            break
                        end
                        VirtualUser:Button1Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                        task.wait(0.01)
                        VirtualUser:Button1Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                        task.wait(0.01)
                    end
                else
                    task.wait(1) -- espera se não houver alvo
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- ========================== INICIAR ==========================
Library:Notification("FEACHT TOOLS", "Script carregado com sucesso!", 3)
