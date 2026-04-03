local Players = cloneref(game:GetService("Players")) or game:GetService("Players")
local player = Players.LocalPlayer

local animation = Instance.new("Animation")
animation.AnimationId = "rbxassetid://68433924"

local activeTrack = nil
local keepRunning = true

-- Função que aplica a animação com prioridade máxima
local function aplicarAnimacao(humanoid)
    if activeTrack then
        activeTrack:Stop()
        activeTrack = nil
    end
    
    if humanoid and humanoid.Parent then
        local track = humanoid:LoadAnimation(animation)
        
        -- PRIORIDADE MÁXIMA (substitui qualquer animação)
        track.Priority = Enum.AnimationPriority.Action4
        
        track:Play()
        track:AdjustSpeed(0)
        track:AdjustWeight(10) -- Peso máximo
        
        -- Força a animação a se manter ativa
        track.TimePosition = 0
        
        activeTrack = track
    end
end

-- Loop que mantém a cabeça pra baixo
local function manterCabecaBaixa()
    while keepRunning and player.Character do
        local character = player.Character
        local humanoid = character and character:FindFirstChildWhichIsA("Humanoid")
        
        if humanoid and humanoid.RigType == Enum.HumanoidRigType.R6 then
            if not activeTrack or not activeTrack.IsPlaying then
                aplicarAnimacao(humanoid)
            else
                -- Força a posição 0 da animação e reforça peso
                activeTrack:AdjustWeight(10)
                activeTrack.TimePosition = 0
            end
        end
        
        task.wait(0.05) -- Verifica mais rápido
    end
end

local function esconderCabeca()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    
    if humanoid.RigType == Enum.HumanoidRigType.R6 then
        aplicarAnimacao(humanoid)
    else
        local AvatarEditorService = game:GetService("AvatarEditorService")
        local humanoidDesc = humanoid.HumanoidDescription
        
        if humanoidDesc then
            AvatarEditorService:PromptSaveAvatar(humanoidDesc, Enum.HumanoidRigType.R6)
            task.wait(2)
            humanoid.Health = 0
        end
    end
end

local function onCharacterAdded(character)
    task.wait(0.3)
    keepRunning = true
    esconderCabeca()
    task.spawn(manterCabecaBaixa)
end

player.CharacterRemoving:Connect(function()
    keepRunning = false
    if activeTrack then
        activeTrack:Stop()
        activeTrack = nil
    end
end)

player.CharacterAdded:Connect(onCharacterAdded)

if player.Character then
    onCharacterAdded(player.Character)
end
