local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local SoundService = game:GetService("SoundService")
local MaterialService = game:GetService("MaterialService")
local UserInputService = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local RunService = game:GetService("RunService")
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
local originalAmbient = Lighting.Ambient
local originalOutdoorAmbient = Lighting.OutdoorAmbient

local originalUse2022 = true
pcall(function() originalUse2022 = MaterialService.Use2022Materials end)

local terrain = Workspace:FindFirstChildOfClass("Terrain")
local originalGrass = false
if terrain then pcall(function() originalGrass = terrain.Decoration end) end

local originalColors = {}
local originalMeshes = {}
local cameraConnection = nil

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DeltaGodMode_LatestSupported"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = targetParent

-- ปรับให้ไม่มีพื้นหลังสีดำ (ลอยคลีนๆ)
local trackerLabel = Instance.new("TextLabel")
trackerLabel.Size = UDim2.new(0.35, 0, 0.045, 0)
trackerLabel.Position = UDim2.new(0.325, 0, 0.01, 0) 
trackerLabel.BackgroundTransparency = 1 -- เอาพื้นหลังออก 100%
trackerLabel.Text = "FPS: Calculating... | Ping: Calculating..."
trackerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
trackerLabel.TextScaled = true
trackerLabel.Font = Enum.Font.SourceSansBold
trackerLabel.Parent = screenGui

local masterContainer = Instance.new("Frame")
masterContainer.Size = UDim2.new(0, 45, 0, 45)
masterContainer.Position = UDim2.new(0.02, 0, 0.15, 0)
masterContainer.BackgroundTransparency = 1
masterContainer.Parent = screenGui

local toggleGuiButton = Instance.new("TextButton")
toggleGuiButton.Size = UDim2.new(1, 0, 1, 0) 
toggleGuiButton.Position = UDim2.new(0, 0, 0, 0)
toggleGuiButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
toggleGuiButton.Text = "+"
toggleGuiButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleGuiButton.TextScaled = true
toggleGuiButton.Font = Enum.Font.SourceSansBold
toggleGuiButton.Parent = masterContainer

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(1, 0)
toggleCorner.Parent = toggleGuiButton

local fpsButton = Instance.new("TextButton")
fpsButton.Size = UDim2.new(0, 220, 0, 45) 
fpsButton.Position = UDim2.new(0, 55, 0, 0) 
fpsButton.BackgroundColor3 = Color3.fromRGB(15, 15, 15) 
fpsButton.BorderSizePixel = 0 
fpsButton.Text = "boost fps: OFF"
fpsButton.TextColor3 = Color3.fromRGB(255, 60, 60) 
fpsButton.TextScaled = true 
fpsButton.Font = Enum.Font.SourceSansBold
fpsButton.Visible = false 
fpsButton.Parent = masterContainer

local mainButtonCorner = Instance.new("UICorner")
mainButtonCorner.CornerRadius = UDim.new(0.3, 0) 
mainButtonCorner.Parent = fpsButton

local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(1.2, 0, 1.8, 0) 
sliderFrame.Position = UDim2.new(-0.1, 0, 1.2, 0) 
sliderFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15) 
sliderFrame.BorderSizePixel = 0
sliderFrame.Visible = false 
sliderFrame.Parent = fpsButton

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0.1, 0) 
frameCorner.Parent = sliderFrame

local sliderBar = Instance.new("TextButton")
sliderBar.Size = UDim2.new(0.85, 0, 0.12, 0) 
sliderBar.Position = UDim2.new(0.075, 0, 0.35, 0)
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
sliderText.Size = UDim2.new(1, 0, 0.2, 0)
sliderText.Position = UDim2.new(0, 0, 0.1, 0) 
sliderText.BackgroundTransparency = 1
sliderText.Text = "Light: Normal"
sliderText.TextColor3 = Color3.fromRGB(255, 255, 255)
sliderText.TextScaled = true
sliderText.Font = Enum.Font.SourceSansBold
sliderText.Parent = sliderFrame

local grayModeButton = Instance.new("TextButton")
grayModeButton.Size = UDim2.new(0.42, 0, 0.2, 0)
grayModeButton.Position = UDim2.new(0.065, 0, 0.55, 0)
grayModeButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
grayModeButton.BorderSizePixel = 0
grayModeButton.Text = "Gray Mode: OFF"
grayModeButton.TextColor3 = Color3.fromRGB(255, 60, 60)
grayModeButton.TextScaled = true
grayModeButton.Font = Enum.Font.SourceSansBold
grayModeButton.Parent = sliderFrame

local grayModeCorner = Instance.new("UICorner")
grayModeCorner.CornerRadius = UDim.new(0.2, 0)
grayModeCorner.Parent = grayModeButton

local fpsCapButton = Instance.new("TextButton")
fpsCapButton.Size = UDim2.new(0.42, 0, 0.2, 0)
fpsCapButton.Position = UDim2.new(0.515, 0, 0.55, 0)
fpsCapButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
fpsCapButton.BorderSizePixel = 0
fpsCapButton.Text = "FPS Cap: 60"
fpsCapButton.TextColor3 = Color3.fromRGB(255, 255, 255)
fpsCapButton.TextScaled = true
fpsCapButton.Font = Enum.Font.SourceSansBold
fpsCapButton.Parent = sliderFrame

local fpsCapCorner = Instance.new("UICorner")
fpsCapCorner.CornerRadius = UDim.new(0.2, 0)
fpsCapCorner.Parent = fpsCapButton

local fpsBoostActive = false
local percentageValue = 0.5 
local grayModeActive = false 
local fpsCapValues = {60, 90, 120, 144, 240, 999}
local currentFpsCapIndex = 1

local dragToggle = nil
local dragSpeed = 0.1
local dragStart = nil
local startPos = nil

local function updateInput(input)
	local delta = input.Position - dragStart
	local position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	game:GetService("TweenService"):Create(masterContainer, TweenInfo.new(dragSpeed), {Position = position}):Play()
end

local function enableDragging(guiObject)
	guiObject.InputBegan:Connect(function(input)
		if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and not UserInputService:GetFocusedTextBox() then
			if guiObject == sliderBar or guiObject == sliderButton then return end
			dragToggle = true
			dragStart = input.Position
			startPos = masterContainer.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragToggle = false
				end
			end)
		end
	end)

	guiObject.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			if dragToggle then
				updateInput(input)
			end
		end
	end)
end

enableDragging(toggleGuiButton)
enableDragging(fpsButton)
enableDragging(sliderFrame)

toggleGuiButton.MouseButton1Click:Connect(function()
	fpsButton.Visible = not fpsButton.Visible
	if fpsButton.Visible then
		toggleGuiButton.Text = "-"
	else
		toggleGuiButton.Text = "+"
	end
end)

local isChangingLight = false
local function applyLightingTweaks()
	if not fpsBoostActive or isChangingLight then return end
	isChangingLight = true
	
	local targetVal = percentageValue * 255
	local ambientColor = Color3.fromRGB(targetVal, targetVal, targetVal)
	
	Lighting.Brightness = percentageValue * 3
	Lighting.ExposureCompensation = (percentageValue - 0.5) * 4
	Lighting.Ambient = ambientColor
	Lighting.OutdoorAmbient = ambientColor
	Lighting.FogColor = Color3.fromRGB(0, 0, 0)
	Lighting.FogStart = 100
	Lighting.FogEnd = 500
	Lighting.GlobalShadows = false
	
	isChangingLight = false
end

Lighting.Changed:Connect(function(property)
	if fpsBoostActive and not isChangingLight then
		if property == "Brightness" or property == "ExposureCompensation" or property == "Ambient" or property == "OutdoorAmbient" or property == "GlobalShadows" then
			applyLightingTweaks()
		end
	end
end)

local timeBuffer = 0
local frameBuffer = 0
RunService.RenderStepped:Connect(function(deltaTime)
	frameBuffer = frameBuffer + 1
	timeBuffer = timeBuffer + deltaTime
	
	if timeBuffer >= 0.5 then
		local currentFps = math.floor(frameBuffer / timeBuffer)
		local currentPing = 0
		
		pcall(function() currentPing = math.floor(Stats.Network.ServerToClientPing:GetValue() * 1000) end)
		if currentPing == 0 then pcall(function() currentPing = math.floor(localPlayer:GetNetworkPing() * 1000) end) end
		
		local pingString = (currentPing <= 0) and "Loading" or tostring(currentPing) .. " ms"
		trackerLabel.Text = "FPS: " .. tostring(currentFps) .. " | Ping: " .. pingString
		frameBuffer = 0
		timeBuffer = 0
	end
end)

fpsCapButton.MouseButton1Click:Connect(function()
	currentFpsCapIndex = currentFpsCapIndex + 1
	if currentFpsCapIndex > #fpsCapValues then currentFpsCapIndex = 1 end
	local selectedCap = fpsCapValues[currentFpsCapIndex]
	fpsCapButton.Text = (selectedCap == 999) and "FPS Cap: INF" or "FPS Cap: " .. tostring(selectedCap)
	setfpscap(selectedCap)
end)

local dragging = false
local function updateSlider(input)
	local barPosition = sliderBar.AbsolutePosition.X
	local barSize = sliderBar.AbsoluteSize.X
	local inputPosition = input.Position.X
	
	percentageValue = math.clamp((inputPosition - barPosition) / barSize, 0, 1)
	sliderButton.Position = UDim2.new(percentageValue, - (sliderButton.AbsoluteSize.X / 2), -0.6, 0)
	
	if percentageValue < 0.45 then
		sliderText.Text = "Light: Dark"
	elseif percentageValue > 0.55 then
		sliderText.Text = "Light: Bright"
	else
		sliderText.Text = "Light: Normal"
	end
	
	applyLightingTweaks()
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
		if player.Character and object:IsDescendantOf(player.Character) then return player.Character end
	end
	return nil
end

local function safeEverythingClean(object)
	if not fpsBoostActive then return end
	pcall(function()
		if object:IsA("ParticleEmitter") or object:IsA("Sparkles") or object:IsA("Smoke") or 
		   object:IsA("Fire") or object:IsA("Beam") or object:IsA("Trail") or 
		   object:IsA("Explosion") or object:IsA("Highlight") or object:IsA("SelectionBox") or
		   object:IsA("PostEffect") or object:IsA("SunRaysEffect") or object:IsA("BlurEffect") or 
		   object:IsA("BloomEffect") or object:IsA("DepthOfFieldEffect") then
			object:Destroy()
			return
		end
		
		if object:IsA("TweenScript") or object:IsA("AnimationController") then
			if not getCharacterOwner(object) then object:Destroy() return end
		end
		
		local chrOwner = getCharacterOwner(object)
		if chrOwner then
			if (object:IsA("Decal") or object:IsA("Texture")) and object.Name ~= "Face" and object.Name ~= "face" then
				object:Destroy()
			end
			return
		end
		
		if object:IsA("BasePart") then
			local name = object.Name:lower()
			
			if object:IsA("Decal") or object:IsA("Texture") then
				object:Destroy()
				return
			end
			
			if name:find("leaf") or name:find("leaves") or name:find("foliage") or name:find("bush") or name:find("tree") or name:find("flower") or name:find("plant") or name:find("grass") then
				object:Destroy()
				return
			end
			
			if not isEssentialBodyPart(object) then
				if not originalColors[object] then
					originalColors[object] = {
						Color = object.Color,
						Material = object.Material
					}
				end
				
				object.Material = Enum.Material.SmoothPlastic
				object.CastShadow = false
				
				if grayModeActive then
					object.BrickColor = BrickColor.new("Medium stone gray")
				else
					object.Color = originalColors[object].Color
				end
				
				if object:IsA("MeshPart") or object:IsA("SpecialMesh") then
					if not originalMeshes[object] then
						originalMeshes[object] = {
							MeshId = object:IsA("MeshPart") and object.MeshId or object.MeshTemplateId,
							TextureId = object:IsA("MeshPart") and object.TextureID or object.TextureId,
							MeshType = object:IsA("SpecialMesh") and object.MeshType or nil
						}
					end
					
					if not grayModeActive then
						if object:IsA("MeshPart") then
							object.MeshId = originalMeshes[object].MeshId
							object.TextureID = originalMeshes[object].TextureId
						elseif object:IsA("SpecialMesh") then
							object.MeshType = originalMeshes[object].MeshType
							object.TextureId = originalMeshes[object].TextureId
						end
					else
						if object:IsA("MeshPart") then
							object.MeshId = ""
							object.TextureID = ""
						elseif object:IsA("SpecialMesh") then
							object.MeshType = Enum.MeshType.Box
							object.TextureId = ""
						end
					end
				end
				
				if object:IsA("MeshPart") then
					object.RenderFidelity = Enum.RenderFidelity.Performance
				end
			end
		end
	end)
end

Workspace.DescendantAdded:Connect(safeEverythingClean)
Lighting.DescendantAdded:Connect(safeEverythingClean)

grayModeButton.MouseButton1Click:Connect(function()
	if not fpsBoostActive then return end
	grayModeActive = not grayModeActive
	
	if grayModeActive then
		grayModeButton.Text = "Gray Mode: ON"
		grayModeButton.TextColor3 = Color3.fromRGB(60, 255, 60)
	else
		grayModeButton.Text = "Gray Mode: OFF"
		grayModeButton.TextColor3 = Color3.fromRGB(255, 60, 60)
	end
	
	for _, object in pairs(Workspace:GetDescendants()) do
		if object:IsA("BasePart") and not isEssentialBodyPart(object) then
			safeEverythingClean(object)
		end
	end
end)

local gcRunning = false
local function startGarbageCollector()
	gcRunning = true
	task.spawn(function()
		while fpsBoostActive and gcRunning do
			task.wait(60)
			pcall(function()
				gcinfo()
				collectgarbage("collect")
			end)
		end
	end)
end

local function cleanExistingSkillEffects()
	for _, object in pairs(Workspace:GetDescendants()) do
		if object:IsA("BasePart") and not isEssentialBodyPart(object) and not originalColors[object] then
			local name = object.Name:lower()
			if name:find("effect") or name:find("skill") or name:find("hit") or name:find("slash") or name:find("magic") or name:find("projectile") then
				pcall(function() object:Destroy() end)
			end
		end
	end
end

local function startCameraStabilizer()
	local camera = Workspace.CurrentCamera
	if not camera then return end
	cameraConnection = camera:GetPropertyChangedSignal("CFrame"):Connect(function()
		if not fpsBoostActive then return end
		local character = localPlayer.Character
		if character and character:FindFirstChild("HumanoidRootPart") then
			local humanoid = character:FindFirstChildOfClass("Humanoid")
			if humanoid and humanoid.Health > 0 then
				pcall(function()
					camera.CameraSubject = character:FindFirstChild("Head") or character.HumanoidRootPart
				end)
			end
		end
	end)
end

local function forceLowGraphics(state)
	pcall(function()
		local settings = settings()
		if settings and settings.Rendering then
			if state then
				settings.Rendering.QualityLevel = Enum.QualityLevel.Level01
				settings.Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.DistanceBased
			else
				settings.Rendering.QualityLevel = Enum.QualityLevel.Automatic
				settings.Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Automatic
			end
		end
	end)
end

fpsButton.MouseButton1Click:Connect(function()
	fpsBoostActive = not fpsBoostActive
	
	if fpsBoostActive then
		fpsButton.Text = "boost fps: ON"
		fpsButton.TextColor3 = Color3.fromRGB(60, 255, 60) 
		sliderFrame.Visible = true 
		
		pcall(function()
			if terrain then terrain.Decoration = false end
			MaterialService.Use2022Materials = false
			SoundService.AmbientReverb = Enum.AmbientReverb.NoReverb
			for _, effect in pairs(Lighting:GetChildren()) do
				if effect:IsA("PostEffect") or effect:IsA("BlurEffect") or effect:IsA("BloomEffect") or effect:IsA("SunRaysEffect") then
					effect:Destroy()
				end
			end
		end)
		
		cleanExistingSkillEffects()
		forceLowGraphics(true)
		
		for _, object in pairs(Workspace:GetDescendants()) do
			safeEverythingClean(object)
		end
		
		applyLightingTweaks()
		startGarbageCollector()
		startCameraStabilizer()
	else
		fpsButton.Text = "boost fps: OFF"
		fpsButton.TextColor3 = Color3.fromRGB(255, 60, 60) 
		sliderFrame.Visible = false 
		gcRunning = false
		
		if cameraConnection then
			cameraConnection:Disconnect()
			cameraConnection = nil
		end
		
		pcall(function() if terrain then terrain.Decoration = originalGrass end end)
		forceLowGraphics(false)
		
		for object, data in pairs(originalColors) do
			if object and object.Parent then
				pcall(function()
					object.Color = data.Color
					object.Material = data.Material
				end)
			end
		end
		
		for object, backup in pairs(originalMeshes) do
			if object and object.Parent and backup then
				pcall(function()
					if object:IsA("MeshPart") then
						object.MeshId = backup.MeshId
						object.TextureID = backup.TextureId
					elseif object:IsA("SpecialMesh") then
						object.MeshType = backup.MeshType
						object.TextureId = backup.TextureId
					end
				end)
			end
		end
		
		originalColors = {}
		originalMeshes = {}
		
		isChangingLight = true
		Lighting.Brightness = originalBrightness
		Lighting.ExposureCompensation = originalExposure
		Lighting.FogColor = originalFogColor
		Lighting.FogStart = originalFogStart
		Lighting.FogEnd = originalFogEnd
		Lighting.Ambient = originalAmbient
		Lighting.OutdoorAmbient = originalOutdoorAmbient
		Lighting.GlobalShadows = true
		isChangingLight = false
		
		pcall(function() MaterialService.Use2022Materials = originalUse2022 end)
		pcall(function() SoundService.AmbientReverb = Enum.AmbientReverb.Default end)
	end
end)
