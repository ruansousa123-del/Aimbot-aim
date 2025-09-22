-- LocalScript (colocar em StarterGui > ScreenGui)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

-- UI setup (se preferir, crie os elementos no Studio e referencie-os)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimAssistGui"
screenGui.Parent = PlayerGui

local toggleButton = Instance.new("TextButton")
toggleButton.Name = "GreenButton"
toggleButton.Size = UDim2.new(0,120,0,40)
toggleButton.Position = UDim2.new(0,20,0,20)
toggleButton.BackgroundColor3 = Color3.fromRGB(50,205,50) -- verde
toggleButton.Text = "Ativar ajuda"
toggleButton.Parent = screenGui

local reticle = Instance.new("ImageLabel")
reticle.Name = "Reticle"
reticle.Size = UDim2.new(0,24,0,24)
reticle.BackgroundTransparency = 1
reticle.Image = "rbxassetid://25221132" -- pequeno círculo (pode trocar)
reticle.Visible = false
reticle.AnchorPoint = Vector2.new(0.5,0.5)
reticle.Position = UDim2.new(0.5,0.5)
reticle.Parent = screenGui

-- Configuração
local enabled = false
local maxTargetDistance = 200 -- ajuste conforme precisa
local teamsIgnore = true -- se quiser ignorar companheiros de equipe (para jogos com Teams)

-- Função para achar o alvo mais próximo do crosshair/center do ecrã
local function getNearestTarget()
    local bestTarget = nil
    local bestScore = math.huge

    -- procura jogadores (pode adaptar para NPCs: iterar por workspace.Enemies por exemplo)
    for _, pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character.Parent then
            -- opcional: ignorar membros da mesma team
            if teamsIgnore and LocalPlayer.Team and pl.Team and pl.Team == LocalPlayer.Team then
                -- pula
            else
                local head = pl.Character:FindFirstChild("Head") or pl.Character:FindFirstChild("HumanoidRootPart")
                local humanoid = pl.Character:FindFirstChild("Humanoid")
                if head and humanoid and humanoid.Health > 0 then
                    local worldPos = head.Position
                    local screenPos, onScreen = Camera:WorldToViewportPoint(worldPos)
                    if onScreen then
                        -- score baseado em distância do centro (pode usar também distância real)
                        local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                        local delta = Vector2.new(screenPos.X, screenPos.Y) - screenCenter
                        local score = delta.Magnitude
                        -- opcionalmente penalizar muito distantes no mundo
                        local worldDist = (Camera.CFrame.Position - worldPos).Magnitude
                        if worldDist <= maxTargetDistance and score < bestScore then
                            bestScore = score
                            bestTarget = {player = pl, part = head, screenPos = Vector2.new(screenPos.X, screenPos.Y), worldDist = worldDist}
                        end
                    end
                end
            end
        end
    end

    return bestTarget
end

-- alterna
toggleButton.MouseButton1Click:Connect(function()
    enabled = not enabled
    reticle.Visible = enabled
    toggleButton.Text = enabled and "Desativar ajuda" or "Ativar ajuda"
end)

-- Atualiza cada frame (apenas visual)
local currentTarget = nil
RunService.RenderStepped:Connect(function(dt)
    if not enabled then return end

    local target = getNearestTarget()
    if target then
        -- move o retículo suavemente para a posição do alvo
        local targetPos = target.screenPos
        local currentPos = Vector2.new(reticle.AbsolutePosition.X + reticle.AbsoluteSize.X/2,
                                       reticle.AbsolutePosition.Y + reticle.AbsoluteSize.Y/2)
        local newPos = currentPos:Lerp(targetPos, math.clamp(10 * dt, 0, 1))
        reticle.Position = UDim2.new(0, newPos.X, 0, newPos.Y)
    else
        -- voltar para o centro (ou esconder)
        local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        local currentPos = Vector2.new(reticle.AbsolutePosition.X + reticle.AbsoluteSize.X/2,
                                       reticle.AbsolutePosition.Y + reticle.AbsoluteSize.Y/2)
        local newPos = currentPos:Lerp(center, math.clamp(5 * dt, 0, 1))
        reticle.Position = UDim2.new(0, newPos.X, 0, newPos.Y)
    end
end)
