local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

-- Configurações Gerais
local settings = {
    autoReload = true,      -- Ativar/desativar Auto-Reload
    minAmmoBeforeReload = 2,-- Munição mínima antes de recarregar
    checkDelay = 0.2,       -- Delay de verificação

    aimbotEnabled = false,  -- Ativar/desativar Aimbot
    aimlockKey = Enum.KeyCode.E, -- Tecla para Aimlock

    espEnabled = true,      -- Ativar ESP
    teamColor = Color3.fromRGB(0, 0, 255),    -- Azul para aliados
    enemyColor = Color3.fromRGB(255, 0, 0)    -- Vermelho para inimigos
}

-- Auto Reload
local function autoReload()
    while settings.autoReload do
        task.wait(settings.checkDelay)
        
        local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
        if tool and tool:FindFirstChild("Ammo") then
            local ammo = tool.Ammo.Value
            if ammo <= settings.minAmmoBeforeReload then
                tool:FindFirstChild("ReloadEvent"):FireServer()
            end
        end
    end
end

-- Aimbot / Aimlock
local function getClosestEnemy()
    local closest, shortestDist = nil, math.huge

    for _, enemy in pairs(game.Players:GetPlayers()) do
        if enemy ~= player and enemy.Team ~= player.Team and enemy.Character then
            local head = enemy.Character:FindFirstChild("Head")
            if head then
                local pos, visible = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
                if visible then
                    local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                    if dist < shortestDist then
                        closest, shortestDist = head, dist
                    end
                end
            end
        end
    end
    return closest
end

local aimlockTarget = nil
mouse.KeyDown:Connect(function(key)
    if key:upper() == settings.aimlockKey.Name then
        aimlockTarget = getClosestEnemy()
    end
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if settings.aimbotEnabled and aimlockTarget then
        workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, aimlockTarget.Position)
    end
end)

-- ESP (Destaca inimigos e aliados)
local function applyESP(player)
    if player == game.Players.LocalPlayer then return end

    player.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        local head = char:FindFirstChild("Head")
        if head then
            local highlight = Instance.new("Highlight", head)
            highlight.FillColor = player.Team == game.Players.LocalPlayer.Team and settings.teamColor or settings.enemyColor
            highlight.OutlineColor = Color3.new(1,1,1)
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        end
    end)
end

if settings.espEnabled then
    for _, v in pairs(game.Players:GetPlayers()) do
        applyESP(v)
    end
    game.Players.PlayerAdded:Connect(applyESP)
end

-- Iniciar Auto-Reload
if settings.autoReload then
    autoReload()
end


