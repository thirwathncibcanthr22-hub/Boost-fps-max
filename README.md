local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local SoundService = game:GetService("SoundService")
local MaterialService = game:GetService("MaterialService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

local targetParent = nil
local success, err = pcall(function()
	if CoreGui and not game:GetService("RunService"):IsStudio() then
		targetParent = CoreGui
	else
		targetParent = playerGui
	end
end)

if not success or not targetParent then
	targetParent = playerGui
end

local originalBrightness = Lighting.Brightness
local originalExposure = Lighting.ExposureCompensation
local originalFogColor = Lighting.FogColor
local originalFogEnd = Lighting.FogEnd
local originalFogStart = Lighting.FogStart

local originalUse2022 = true
pcall(function() originalUse2022 = MaterialService.Use2022Materials end)

local terrain = Workspace:FindFirstChildOfClass("Terrain")
local originalGrass = false
if terrain then pcall(function() originalGrass = terrain.Decoration end) end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DeltaGodMode_LatestSupported"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = targetParent

local fpsButton = Instance.new("TextButton")
fpsButton.Size = UDim2.new(0.25, 0, 0.09, 0) 
fpsButton.Position = UDim2.new(0.02, 0, 0.05, 0) 
fpsButton.BackgroundColor3 = Color3.fromRGB(15, 15, 15) 
fpsButton.BorderSizePixel = 0 
fpsButton.Text = "FPS Boost: OFF"
fpsButton.TextColor3 = Color3.fromRGB(255, 60, 60) 
fpsButton.TextScaled = true 
fpsButton.Font = Enum.Font.SourceSansBold
fpsButton.Active = true
fpsButton.Parent = screenGui

local mainButtonCorner = Instance.new("UICorner")
mainButtonCorner.CornerRadius = UDim.new(0.5, 0) 
mainButtonCorner.Parent = fpsButton

local dragToggle = nil
local dragSpeed = 0.15
local dragStart = nil
local startPos = nil

local function updateInput(input)
	local delta = input.Position - dragStart
	local position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	game:GetService("TweenService"):Create(fpsButton, TweenInfo.new(dragSpeed), {Position = position}):Play()
end

fpsButton.InputBegan:Connect(function(input)
	if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and not UserInputService:GetFocusedTextBox() then
		dragToggle = true
		dragStart = input.Position
		startPos = fpsButton.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragToggle = false
			end
		end)
	end
end)

fpsButton.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		if dragToggle then
			updateInput(input)
		end
	end
end)

local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(1, 0, 0.85, 0) 
sliderFrame.Position = UDim2.new(0, 0, 1.2, 0) 
sliderFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15) 
sliderFrame.BorderSizePixel = 0
sliderFrame.Visible = false 
sliderFrame.Parent = fpsButton

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0.25, 0) 
frameCorner.Parent = sliderFrame

local sliderBar = Instance.new("TextButton")
sliderBar.Size = UDim2.new(0.85, 0, 0.22, 0) 
sliderBar.Position = UDim2.new(0.075, 0, 0.62, 0)
sliderBar.BackgroundColor3 = Color3.fromRGB(45, 45, 45) 
sliderBar.BorderSizePixel = 0
sliderBar.Text = ""
sliderBar.AutoButtonColor = false
sliderBar.Parent = sliderFrame

local barCorner = Instance.new("UICorner")
barCorner.CornerRadius = UDim.new(1, 0)
barCorner.Parent = sliderBar

local sliderButton = Instance.new("Frame")
sliderButton.Size = UDim2.new(0, 32, 0, 32) 
sliderButton.Position = UDim2.new(0.5, -16, -0.6, 0) 
sliderButton.BackgroundColor3 = Color3.fromRGB(240, 240, 240) 
sliderButton.BorderSizePixel = 0
sliderButton.Parent = sliderBar

local aspectConstraint = Instance.new("UIAspectRatioConstraint")
aspectConstraint.AspectRatio = 1
aspectConstraint.AspectType = Enum.AspectType.ScaleWithParentSize
aspectConstraint.DominantAxis = Enum.DominantAxis.Height
aspectConstraint.Parent = sliderButton

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(1, 0) 
circleCorner.Parent = sliderButton

local sliderText = Instance.new("TextLabel")
sliderText.Size = UDim2.new(1, 0, 0.35, 0)
sliderText.Position = UDim2.new(0, 0, 0.15, 0) 
sliderText.BackgroundTransparency = 1
sliderText.Text = "Brightness: 0"
sliderText.TextColor3 = Color3.fromRGB(255, 255, 255)
sliderText.TextScaled = true
sliderText.Font = Enum.Font.SourceSansBold
sliderText.Parent = sliderFrame

local fpsBoostActive = false
local currentSliderValue = 0
local localLight = nil

local function isEssentialBodyPart(object)
	if object:IsA("BasePart") then
		local name = object.Name
		if name == "Head" or name == "Torso" or name == "Left Arm" or name == "Right Arm" or name == "Left Leg" or name == "Right Leg" or name:find("Upper") or name:find("Lower") or name:find("Hand") or name:find("Foot") or name == "HumanoidRootPart" then
			return true
		end
	end
	return false
end

local function getCharacterOwner(object)
	for _, player in pairs(Players:GetPlayers()) do
		if player.Character and object:IsDescendantOf(player.Character) then
			return player.Character
		end
	end
	return nil
end

local function safeEverythingClean(object)
	if not fpsBoostActive then return end
	pcall(function()
		if object:IsA("ParticleEmitter") or object:IsA("Sparkles") or object:IsA("Smoke") or object:IsA("Fire") or object:IsA("Beam") or object:IsA("Trail") or object:IsA("Explosion") then
			object:Destroy()
			return
		end
		
		local chrOwner = getCharacterOwner(object)
		if chrOwner then
			if (object:IsA("Decal") or object:IsA("Texture")) and object.Name ~= "Face" and object.Name ~= "face" then
				object:Destroy()
				return
			end
			if object:IsA("BasePart") then
				local name = object.Name:lower()
				if name:find("effect") or name:find("fx") or name:find("aura") or object.Material == Enum.Material.Neon or not object.CanCollide then
					if not isEssentialBodyPart(object) then
						object.Transparency = 1
						object.CastShadow = false
					end
				end
			end
			return
		end
		
		if object:IsA("BasePart") then
			local name = object.Name:lower()
			if object:IsA("Decal") or object:IsA("Texture") then
				object:Destroy()
				return
			end
			if name:find("effect") or name:find("fx") or name:find("aura") or name:find("crack") or name:find("debris") or object.Material == Enum.Material.Neon then
				object.Transparency = 1
				object.CanCollide = false
				object.CastShadow = false
			else
				object.Material = Enum.Material.SmoothPlastic
				object.Reflectance = 0
				object.CastShadow = false
				if object:IsA("MeshPart") then
					object.RenderFidelity = Enum.RenderFidelity.Performance
				end
			end
		end
	end)
end

Workspace.DescendantAdded:Connect(safeEverythingClean)

-- ==========================================
-- ระบบไฟจำลองรอบตัวผู้เล่น (ไม่สนใจค่าเซิร์ฟเวอร์)
-- ==========================================
local function updateLocalLight()
	if not fpsBoostActive then
		if localLight then pcall(function() localLight:Destroy() end) localLight = nil end
		return
	end
	
	local character = localPlayer.Character
	local rootPart = character and character:FindFirstChild("HumanoidRootPart")
	
	if rootPart then
		if not localLight or localLight.Parent ~= rootPart then
			if localLight then pcall(function() localLight:Destroy() end) end
			localLight = Instance.new("PointLight")
			localLight.Name = "FPS_Boost_LocalLight"
			localLight.Shadows = false
			localLight.Parent = rootPart
		end
		
		-- แปลงค่าสไลเดอร์ (-3 ถึง 3) ให้เป็นความสว่างและระยะส่องของแสงไฟรอบตัว
		local sliderRatio = (currentSliderValue + 3) / 6 -- สเกล 0 ถึง 1
		localLight.Brightness = sliderRatio * 15 -- ความเข้มของดวงไฟ
		localLight.Range = 60 + (sliderRatio * 500) -- ระยะส่องแสง ยิ่งเลื่อนเยอะยิ่งสว่างไกลสุดแมพ
	end
end

-- อัปเดตตำแหน่งดวงไฟตามตัวละครตลอดเวลา
game:GetService("RunService").RenderStepped:Connect(function()
	if fpsBoostActive then
		updateLocalLight()
	end
end)

local isChangingLight = false
Lighting.Changed:Connect(function(property)
	if fpsBoostActive and not isChangingLight then
		if property == "Brightness" or property == "ExposureCompensation" or property == "FogEnd" or property == "FogColor" then
			isChangingLight = true
			Lighting.Brightness = 1.5
			Lighting.ExposureCompensation = 0.5
			Lighting.FogColor = Color3.fromRGB(0, 0, 0)
			Lighting.FogStart = 50
			Lighting.FogEnd = 200
			isChangingLight = false
		end
	end
end)

local dragging = false
local function updateSlider(input)
	local barPosition = sliderBar.AbsolutePosition.X
	local barSize = sliderBar.AbsoluteSize.X
	local inputPosition = input.Position.X
	
	local percentage = math.clamp((inputPosition - barPosition) / barSize, 0, 1)
	sliderButton.Position = UDim2.new(percentage, - (sliderButton.AbsoluteSize.X / 2), -0.6, 0)
	
	currentSliderValue = math.floor(((percentage * 6) - 3) * 10) / 10
	sliderText.Text = "Brightness: " .. tostring(currentSliderValue)
	
	updateLocalLight()
end

sliderBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		updateSlider(input)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		updateSlider(input)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

fpsButton.MouseButton1Click:Connect(function()
	fpsBoostActive = not fpsBoostActive
	
	if fpsBoostActive then
		fpsButton.Text = "FPS Boost: ON"
		fpsButton.TextColor3 = Color3.fromRGB(60, 255, 60) 
		sliderFrame.Visible = true 
		
		pcall(function()
			if terrain then terrain.Decoration = false end
			MaterialService.Use2022Materials = false
			SoundService.AmbientReverb = Enum.AmbientReverb.NoReverb
		end)
		
		for _, object in pairs(Workspace:GetDescendants()) do
			safeEverythingClean(object)
			pcall(function()
				if object:IsA("BasePart") or object:IsA("Model") then
					local name = object.Name:lower()
					if name:find("tree") or name:find("leaf") or name:find("leaves") or name:find("foliage") or name:find("bush") or name:find("pine") then
						if not getCharacterOwner(object) and not object:FindFirstChildOfClass("Humanoid") then
							object:Destroy()
						end
					end
				end
			end)
		end
		
		updateLocalLight()
		
		isChangingLight = true
		Lighting.Brightness = 1.5
		Lighting.ExposureCompensation = 0.5
		Lighting.FogColor = Color3.fromRGB(0, 0, 0)
		Lighting.FogStart = 50
		Lighting.FogEnd = 200
		isChangingLight = false
	else
		fpsButton.Text = "FPS Boost: OFF (Restart Game)"
		fpsButton.TextColor3 = Color3.fromRGB(255, 60, 60) 
		sliderFrame.Visible = false 
		
		if localLight then pcall(function() localLight:Destroy() end) localLight = nil end
		pcall(function() if terrain then terrain.Decoration = originalGrass end end)
		
		isChangingLight = true
		Lighting.Brightness = originalBrightness
		Lighting.ExposureCompensation = originalExposure
		Lighting.FogColor = originalFogColor
		Lighting.FogStart = originalFogStart
		Lighting.FogEnd = originalFogEnd
		isChangingLight = false
		
		pcall(function() MaterialService.Use2022Materials = originalUse2022 end)
		pcall(function() SoundService.AmbientReverb = Enum.AmbientReverb.Default end)
	end
end)
