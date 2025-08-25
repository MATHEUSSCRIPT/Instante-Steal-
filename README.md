-- Carregar Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HRP = Character:WaitForChild("HumanoidRootPart")

-- Janela principal
local Window = Rayfield:CreateWindow({
   Name = "Xtzin Hub",
   Icon = 0,
   LoadingTitle = "Carregando Hub...",
   LoadingSubtitle = "by Xtzin",
})

-- Aba
local MainTab = Window:CreateTab("Brainrot", 4483362458)

-- Variáveis
local SavedPosition = nil
local FlyingConnection = nil
local Speed = 80 -- velocidade padrão (studs/s)

-- Atualiza Character caso morra/respawne
LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    HRP = char:WaitForChild("HumanoidRootPart")
end)

-- Função para desativar colisão
local function noclip()
    for _, part in pairs(Character:GetDescendants()) do
        if part:IsA("BasePart") and part.CanCollide then
            part.CanCollide = false
        end
    end
end

-- Função para detectar obstáculo
local function hasObstacleAhead()
    local ray = RaycastParams.new()
    ray.FilterDescendantsInstances = {Character}
    ray.FilterType = Enum.RaycastFilterType.Blacklist

    local result = Workspace:Raycast(HRP.Position, HRP.CFrame.LookVector * 5, ray)
    return result ~= nil
end

-- Botão para salvar posição
MainTab:CreateButton({
   Name = "Salvar Posição Atual",
   Callback = function()
       if HRP then
           SavedPosition = HRP.Position
           Rayfield:Notify({
               Title = "Posição Salva!",
               Content = "Local salvo com sucesso.",
               Duration = 3
           })
       end
   end,
})

-- Slider para mudar velocidade
MainTab:CreateSlider({
    Name = "Velocidade do Voo",
    Range = {20, 300},
    Increment = 10,
    Suffix = "studs/s",
    CurrentValue = 80,
    Flag = "VelocidadeVoo",
    Callback = function(Value)
        Speed = Value
    end,
})

-- Toggle do Teleguiado
MainTab:CreateToggle({
   Name = "Ativar Teleguiado",
   CurrentValue = false,
   Flag = "Teleguiado",
   Callback = function(Value)
       if Value then
           if not SavedPosition then
               Rayfield:Notify({
                   Title = "Erro",
                   Content = "Você precisa salvar uma posição primeiro!",
                   Duration = 3
               })
               return
           end

           local BodyVel = Instance.new("BodyVelocity")
           BodyVel.MaxForce = Vector3.new(1e5, 1e5, 1e5)
           BodyVel.Velocity = Vector3.zero
           BodyVel.Parent = HRP

           FlyingConnection = RunService.Heartbeat:Connect(function()
               if HRP and SavedPosition then
                   noclip()
                   local dir = (SavedPosition - HRP.Position)

                   if dir.Magnitude > 5 then
                       -- Desvio automático se tiver parede
                       if hasObstacleAhead() then
                           BodyVel.Velocity = (HRP.CFrame.RightVector * Speed) + Vector3.new(0,2,0)
                       else
                           BodyVel.Velocity = dir.Unit * Speed
                       end
                   else
                       BodyVel:Destroy()
                       FlyingConnection:Disconnect()
                       FlyingConnection = nil
                       Rayfield:Notify({
                           Title = "Chegou!",
                           Content = "Você chegou ao local salvo.",
                           Duration = 3
                       })
                   end
               end
           end)
       else
           if FlyingConnection then
               FlyingConnection:Disconnect()
               FlyingConnection = nil
           end
           if HRP:FindFirstChildOfClass("BodyVelocity") then
               HRP:FindFirstChildOfClass("BodyVelocity"):Destroy()
           end
       end
   end,
})

-- Função nova: Teleporte instantâneo
MainTab:CreateButton({
   Name = "Teleporte Instantâneo",
   Callback = function()
       if HRP and SavedPosition then
           HRP.CFrame = CFrame.new(SavedPosition + Vector3.new(0,3,0))
           Rayfield:Notify({
               Title = "Teleporte",
               Content = "Você foi teleportado instantaneamente.",
               Duration = 3
           })
       else
           Rayfield:Notify({
               Title = "Erro",
               Content = "Salve uma posição antes de teleportar.",
               Duration = 3
           })
       end
   end,
})
