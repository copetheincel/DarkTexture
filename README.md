-- DARK TEXTURE + REMOVE TEXTURES + SKY ORIGINAL + FULLBRIGHT LEGIT
-- sem hitbox, leve e permanente

local Lighting = game:GetService("Lighting")
local Terrain = workspace:FindFirstChildOfClass("Terrain")

---------------------------------------------------
-- CONFIG DARK TEXTURE
---------------------------------------------------
local DARKNESS = 0.18

---------------------------------------------------
-- FULL BRIGHT CONFIG
---------------------------------------------------
local function setFullBright()
	Lighting.Brightness = 3
	Lighting.ClockTime = 14
	Lighting.FogEnd = 100000
	Lighting.GlobalShadows = false
	Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
	Lighting.Ambient = Color3.fromRGB(255, 255, 255)

	-- remove efeitos que deixam escuro
	for _, v in ipairs(Lighting:GetChildren()) do
		if v:IsA("ColorCorrectionEffect")
		or v:IsA("BloomEffect")
		or v:IsA("DepthOfFieldEffect")
		or v:IsA("SunRaysEffect")
		or v:IsA("Atmosphere") then
			v:Destroy()
		end
	end
end

-- aplica fullbright inicial
setFullBright()

-- reaplica caso o jogo tente mudar
for _, prop in ipairs({"Brightness","ClockTime","FogEnd","Ambient","OutdoorAmbient","GlobalShadows"}) do
	Lighting:GetPropertyChangedSignal(prop):Connect(setFullBright)
end

---------------------------------------------------
-- FUNÇÕES DARK
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
-- APPLY DARK + REMOVE TEXTURES
---------------------------------------------------
local function applyDark(obj)
	if isCharacter(obj) then return end

	-- remove texturas
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
-- DARKEN TERRAIN
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
-- APPLY DARK EM TUDO
---------------------------------------------------
for _, v in ipairs(workspace:GetDescendants()) do
	applyDark(v)
end

workspace.DescendantAdded:Connect(function(v)
	task.wait(0.05)
	applyDark(v)
end)

print("FULLBRIGHT LEGIT + Dark Texture + Remove Textures + Sky Original aplicado.")
