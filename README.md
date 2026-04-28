--[[
    FEACHT TOOLS – MUSCLE LEGENDS (FLUENT UI) v3.0
    Extremamente melhorado, Auto Strong a 2000 studs de altura.
--]]

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Feacht Tools – Muscle Legends",
    SubTitle = "v3.0 Extremo",
    TabWidth = 140,
    Size = UDim2.fromOffset(580, 460),
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
local TweenService = game:GetService("TweenService")

-- ========================== BOTÃO FLUTUANTE ==========================
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "FeachtToggleGui"
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
OpenButton.Position = UDim2.new(0.1, 0, 0.2, 0)
OpenButton.Size = UDim2.new(0, 50, 0, 50)
OpenButton.Font = Enum.Font.GothamBold
OpenButton.Text = "F"
OpenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenButton.TextSize = 24
OpenButton.Active = true
OpenButton.Draggable = true
Instance.new("UICorner", OpenButton).CornerRadius = UDim.new(0, 10)

OpenButton.MouseButton1Click:Connect(function()
    Window:Minimize()
end)

task.spawn(function()
    local FluentUI = CoreGui:WaitForChild("Fluent", 5)
    if FluentUI then
        FluentUI.AncestryChanged:Connect(function(_, parent)
            if not parent then ScreenGui:Destroy() end
        end)
        FluentUI.Destroying:Connect(function() ScreenGui:Destroy() end)
    end
end)

-- ========================== VARIÁVEIS GLOBAIS ==========================
local autoClickEnabled = false
local autoClickDelay = 0.01
local autoStrongEnabled = false
local killAutoEnabled = false
local autoRebirthEnabled = false
local autoSellEnabled = false
local autoEquipBest = false
local autoStatsEnabled = false
local flyEnabled = false
local flySpeed = 50
local noclipEnabled = false
local invisibleEnabled = false
local antiAfkEnabled = false
local infiniteJumpEnabled = false

-- ========================== FUNÇÕES AUXILIARES OTIMIZADAS ==========================
local function rapidClick(times)
    times = times or 1
    for _ = 1, times do
        VirtualUser:Button1Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(autoClickDelay)
        VirtualUser:Button1Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(autoClickDelay)
    end
end

local function equipTool(index)
    local char = Player.Character
    if not char then return end
    local tools = {}
    for _, item in ipairs(Player.Backpack:GetChildren()) do
        if item:IsA("Tool") then table.insert(tools, item) end
    end
    if index <= #tools then
        tools[index].Parent = char
    end
end

local function equipBestTool()
    local char = Player.Character
    if not char then return end
    local bestTool = nil
    for _, item in ipairs(Player.Backpack:GetChildren()) do
        if item:IsA("Tool") then
            bestTool = item
            break
        end
    end
    if bestTool then bestTool.Parent = char end
end

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

local function teleportTo(targetPlayer)
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    if targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
    end
end

-- ========================== FLY ==========================
local flyConnection
local keys = {W = false, A = false, S = false, D = false, E = false, Q = false}
local function startFly()
    local char = Player.Character or Player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.PlatformStand = true
    for key in pairs(keys) do
        UserInputService.InputBegan:Connect(function(input, processed)
            if processed then return end
            if input.KeyCode == Enum.KeyCode[key] then keys[key] = true end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if input.KeyCode == Enum.KeyCode[key] then keys[key] = false end
        end)
    end
    flyConnection = RunService.RenderStepped:Connect(function()
        if not flyEnabled then return end
        local rootPart = char:FindFirstChild("HumanoidRootPart")
        if not rootPart then return end
        local speed = flySpeed
        local moveDir = Vector3.new()
        if keys.W then moveDir += workspace.CurrentCamera.CFrame.LookVector end
        if keys.S then moveDir -= workspace.CurrentCamera.CFrame.LookVector end
        if keys.A then moveDir -= workspace.CurrentCamera.CFrame.RightVector end
        if keys.D then moveDir += workspace.CurrentCamera.CFrame.RightVector end
        if keys.E then moveDir += Vector3.new(0, 1, 0) end
        if keys.Q then moveDir -= Vector3.new(0, 1, 0) end
        moveDir = moveDir.Unit * speed
        rootPart.Velocity = moveDir
    end)
end
local function stopFly()
    if flyConnection then flyConnection:Disconnect(); flyConnection = nil end
    local char = Player.Character
    if char then
        if char:FindFirstChild("Humanoid") then char.Humanoid.PlatformStand = false end
        if char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.Velocity = Vector3.zero end
    end
end

-- ========================== NOCLIP ==========================
local noclipConnection
local function startNoclip()
    noclipConnection = RunService.Stepped:Connect(function()
        if noclipEnabled then
            local char = Player.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then part.CanCollide = false end
                end
            end
        end
    end)
end
local function stopNoclip()
    if noclipConnection then noclipConnection:Disconnect(); noclipConnection = nil end
    local char = Player.Character
    if char then
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = true end
        end
    end
end

-- ========================== INVISIBLE (BETA) ==========================
local savedParts = {}
local function makeInvisible(active)
    invisibleEnabled = active
    local char = Player.Character
    if not char then return end
    if active then
        savedParts = {}
        for _, item in ipairs(char:GetDescendants()) do
            if item:IsA("BasePart") then
                table.insert(savedParts, {item, item.Transparency, item.CanCollide})
                item.Transparency = 1
                item.CanCollide = false
            elseif item:IsA("Accessory") and item:FindFirstChild("Handle") then
                local h = item.Handle
                table.insert(savedParts, {h, h.Transparency, h.CanCollide})
                h.Transparency = 1
                h.CanCollide = false
            end
        end
        local hum = char:FindFirstChild("Humanoid")
        if hum then hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None end
        char.HumanoidRootPart.CanCollide = false
    else
        for _, data in ipairs(savedParts) do
            local part = data[1]
            if part and part.Parent then
                part.Transparency = data[2]
                part.CanCollide = data[3]
            end
        end
        savedParts = {}
        local hum = char:FindFirstChild("Humanoid")
        if hum then hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.Viewer end
        if char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CanCollide = true end
    end
end

-- ========================== ABAS E CONTEÚDOS ==========================
-- Universal
local UniversalTab = Window:AddTab({ Title = "Universal", Icon = "globe" })
UniversalTab:AddButton({ Title = "Infinite Yield", Callback = function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
end})

-- Auto Click
local AutoClickTab = Window:AddTab({ Title = "Auto Click", Icon = "mouse-pointer" })
AutoClickTab:AddToggle("AutoClickToggle", { Title = "Ativar Auto Click", Default = false, Callback = function(v)
    autoClickEnabled = v
    if v then task.spawn(function() while autoClickEnabled do rapidClick(1) end end) end
end})
AutoClickTab:AddSlider("ClickSpeed", { Title = "Velocidade (ms)", Default = 10, Min = 1, Max = 100, Rounding = 0, Callback = function(v) autoClickDelay = v/1000 end})

-- Auto Strong (MUITO MAIS ALTO)
local AutoStrongTab = Window:AddTab({ Title = "Auto Strong", Icon = "dumbbell" })
AutoStrongTab:AddToggle("AutoStrongToggle", { Title = "Ativar Auto Strong", Default = false, Callback = function(v)
    autoStrongEnabled = v
    if v then
        task.spawn(function()
            local platform = Instance.new("Part")
            platform.Name = "AutoStrongPlatform"
            platform.Size = Vector3.new(30, 0.4, 30)
            platform.Anchored = true
            platform.CanCollide = true
            platform.BrickColor = BrickColor.new("Dark stone grey")
            platform.Transparency = 0.5
            platform.Parent = workspace

            local char = Player.Character
            if not char or not char:FindFirstChild("HumanoidRootPart") then
                Fluent:Notify({Title = "Erro", Content = "Personagem não encontrado."})
                autoStrongEnabled = false
                if platform then platform:Destroy() end
                return
            end
            -- Coloca a plataforma a 2000 studs acima da posição atual
            platform.CFrame = char.HumanoidRootPart.CFrame * CFrame.new(0, 2000, 0)

            equipTool(2)
            -- Teleporta o personagem para cima da plataforma
            char.HumanoidRootPart.CFrame = platform.CFrame * CFrame.new(0, 3, 0)

            -- Loop de cliques
            while autoStrongEnabled do
                rapidClick(1)
            end

            if platform and platform.Parent then platform:Destroy() end
        end)
    end
end})

-- Kill Auto
local KillAutoTab = Window:AddTab({ Title = "Kill Auto", Icon = "crosshair" })
KillAutoTab:AddToggle("KillAutoToggle", { Title = "Ativar Kill Auto", Default = false, Callback = function(v)
    killAutoEnabled = v
    if v then
        task.spawn(function()
            while killAutoEnabled do
                local target = getNearestPlayer()
                if target then
                    equipTool(1)
                    teleportTo(target)
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
end})

-- Auto Farm
local AutoFarmTab = Window:AddTab({ Title = "Auto Farm", Icon = "bar-chart-2" })
AutoFarmTab:AddToggle("AutoRebirthToggle", { Title = "Auto Rebirth (experimental)", Default = false, Callback = function(v)
    autoRebirthEnabled = v
    if v then
        task.spawn(function()
            while autoRebirthEnabled do
                pcall(function()
                    local gui = Player.PlayerGui:FindFirstChild("RebirthGui")
                    if gui then
                        local btn = gui:FindFirstChildWhichIsA("TextButton") or gui:FindFirstChild("ImageButton")
                        if btn then
                            local click = btn:FindFirstChildWhichIsA("ClickDetector")
                            if click then fireclickdetector(click) end
                        end
                    end
                end)
                task.wait(10)
            end
        end)
    end
end})
AutoFarmTab:AddToggle("AutoSellToggle", { Title = "Auto Sell Weights", Default = false, Callback = function(v)
    autoSellEnabled = v
    if v then
        task.spawn(function()
            while autoSellEnabled do
                pcall(function()
                    workspace.Events.Sell:FireServer("SellAll")
                end)
                task.wait(5)
            end
        end)
    end
end})
AutoFarmTab:AddToggle("AutoEquipBestToggle", { Title = "Auto Equip Best Tool", Default = false, Callback = function(v)
    autoEquipBest = v
    if v then task.spawn(function() while autoEquipBest do equipBestTool(); task.wait(1) end end) end
end})
AutoFarmTab:AddToggle("AutoStatsToggle", { Title = "Auto Stats (Força)", Default = false, Callback = function(v)
    autoStatsEnabled = v
    if v then task.spawn(function() while autoStatsEnabled do pcall(function() workspace.Events.Stats:FireServer("Strength") end) task.wait(1) end end) end
end})

-- Local Player
local LocalPlayerTab = Window:AddTab({ Title = "Local Player", Icon = "user" })
LocalPlayerTab:AddSlider("WalkSpeed", { Title = "Velocidade (WalkSpeed)", Default = 16, Min = 1, Max = 200, Rounding = 0, Callback = function(v)
    local char = Player.Character
    if char and char:FindFirstChild("Humanoid") then char.Humanoid.WalkSpeed = v end
end})
LocalPlayerTab:AddToggle("FlyToggle", { Title = "Fly (E = Subir, Q = Descer)", Default = false, Callback = function(v)
    flyEnabled = v
    if v then startFly() else stopFly() end
end})
LocalPlayerTab:AddSlider("FlySpeed", { Title = "Velocidade do Fly", Default = 50, Min = 10, Max = 200, Rounding = 0, Callback = function(v) flySpeed = v end})
LocalPlayerTab:AddToggle("NoclipToggle", { Title = "Noclip", Default = false, Callback = function(v)
    noclipEnabled = v
    if v then startNoclip() else stopNoclip() end
end})

-- Misc
local MiscTab = Window:AddTab({ Title = "Misc", Icon = "star" })
MiscTab:AddToggle("InvisibleToggle", { Title = "Invisible (Beta)", Default = false, Callback = function(v)
    makeInvisible(v)
end})
MiscTab:AddToggle("AntiAFKToggle", { Title = "Anti-AFK", Default = false, Callback = function(v)
    antiAfkEnabled = v
    if v then
        task.spawn(function()
            while antiAfkEnabled do
                VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                wait(1)
                VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                wait(60)
            end
        end)
    end
end})
MiscTab:AddToggle("InfJumpToggle", { Title = "Infinite Jump", Default = false, Callback = function(v)
    infiniteJumpEnabled = v
    task.spawn(function()
        while infiniteJumpEnabled do
            local char = Player.Character
            if char and char:FindFirstChild("Humanoid") and char.Humanoid.FloorMaterial ~= Enum.Material.Air then
                char.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
            task.wait(0.1)
        end
    end)
end})

-- ========================== INÍCIO ==========================
Fluent:Notify({Title = "Feacht Tools v3.0", Content = "Script extremamente melhorado! Auto Strong a 2000 studs.", Duration = 5})
Window:SelectTab(1)
