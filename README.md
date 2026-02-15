-- DARK TEXTURE FULL + REMOVE TEXTURES + SKYBOX ORIGINAL + MIDDAY FIXO
-- pega tudo (chão, paredes, terrain, decals, textures)
-- não mexe em players/npcs
-- sem piscar e sem precisar executar 2x

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Terrain = workspace:FindFirstChildOfClass("Terrain")

---------------------------------------------------
-- CONFIG
---------------------------------------------------
local DARKNESS = 0.18
local FIXED_TIME = 13.0

---------------------------------------------------
-- FUNÇÕES
---------------------------------------------------
local function darkenColor(color)
	return Color3.new(
		math.clamp(color.R * DARKNESS, 0, 1),
		math.clamp(color.G * DARKNESS, 0, 1),
		math.clamp(color.B * DARKNESS, 0, 1)
	)
end

local function isCharacter(obj)
	local model = obj:FindFirstAncestorOfClass("Model")
	if model and model:FindFirstChildOfClass("Humanoid") then
		return true
	end
	return false
end

---------------------------------------------------
-- FIXAR HORÁRIO EM 13:00 (SEM LOOP)
---------------------------------------------------
Lighting.ClockTime = FIXED_TIME

Lighting:GetPropertyChangedSignal("ClockTime"):Connect(function()
	if Lighting.ClockTime ~= FIXED_TIME then
		Lighting.ClockTime = FIXED_TIME
	end
end)

---------------------------------------------------
-- RESET SKYBOX PRA ORIGINAL DO ROBLOX
---------------------------------------------------
for _, v in ipairs(Lighting:GetChildren()) do
	if v:IsA("Sky") then
		v:Destroy()
	end
end

local sky = Instance.new("Sky")
sky.Name = "OriginalSky"

sky.SkyboxBk = "rbxasset://textures/sky/sky512_bk.tex"
sky.SkyboxDn = "rbxasset://textures/sky/sky512_dn.tex"
sky.SkyboxFt = "rbxasset://textures/sky/sky512_ft.tex"
sky.SkyboxLf = "rbxasset://textures/sky/sky512_lf.tex"
sky.SkyboxRt = "rbxasset://textures/sky/sky512_rt.tex"
sky.SkyboxUp = "rbxasset://textures/sky/sky512_up.tex"

sky.Parent = Lighting

---------------------------------------------------
-- DARK + REMOVE TEXTURES
---------------------------------------------------
local function applyDark(obj)
	if isCharacter(obj) then return end

	-- remove texturas (deixa mais "dark texture")
	if obj:IsA("SurfaceAppearance") or obj:IsA("Texture") or obj:IsA("Decal") then
		obj:Destroy()
		return
	end

	-- escurece parts
	if obj:IsA("BasePart") then
		if obj.Material == Enum.Material.Neon then return end
		if obj.Material == Enum.Material.ForceField then return end

		obj.Color = darkenColor(obj.Color)

		pcall(function()
			obj.Reflectance = 0
		end)
	end

	-- escurece partículas / beams / trails
	if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then
		pcall(function()
			local kp = obj.Color.Keypoints
			if kp and kp[1] then
				obj.Color = ColorSequence.new(darkenColor(kp[1].Value))
			end
		end)
	end
end

---------------------------------------------------
-- TERRAIN (CHÃO DO MAPA MESMO)
---------------------------------------------------
if Terrain then
	for _, mat in ipairs(Enum.Material:GetEnumItems()) do
		pcall(function()
			local original = Terrain:GetMaterialColor(mat)
			Terrain:SetMaterialColor(mat, darkenColor(original))
		end)
	end
end

---------------------------------------------------
-- APLICAR EM TUDO
---------------------------------------------------
for _, v in ipairs(workspace:GetDescendants()) do
	applyDark(v)
end

workspace.DescendantAdded:Connect(function(v)
	task.wait(0.05)
	applyDark(v)
end)

print("Dark Texture FULL + Remove Textures + Skybox Original + 13:00 fixo aplicado.")
