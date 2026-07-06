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

-- แถบแสดงผล FPS & Ping สีดำตรงกลางขอบบนสุดของจอ
local trackerLabel = Instance.new("TextLabel")
trackerLabel.Size = UDim2.new(0.35, 0, 0.045, 0)
trackerLabel.Position = UDim2.new(0.325, 0, 0.01, 0) 
trackerLabel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
trackerLabel.BackgroundTransparency = 0.25 
trackerLabel.Text = "FPS: Calculating... | Ping: Calculating..."
trackerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
trackerLabel.TextScaled = true
trackerLabel.Font = Enum.Font.SourceSansBold
trackerLabel.Parent = screenGui

local trackerCorner = Instance.new("UICorner")
trackerCorner.CornerRadius = UDim.new(0.2, 0)
trackerCorner.Parent = trackerLabel

-- ปุ่มเปิด-ปิด ย่อขนาดเล็ก (ลากย้ายได้)
local toggleGuiButton = Instance.new("TextButton")
toggleGuiButton.Size = UDim2.new(0, 45, 0, 45) 
toggleGuiButton.Position = UDim2.new(0.02, 0, 0.15, 0)
toggleGuiButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
toggleGuiButton.Text = "+"
toggleGuiButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleGuiButton.TextScaled = true
toggleGuiButton.Font = Enum.Font.SourceSansBold
toggleGuiButton.Parent = screenGui

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(1, 0)
toggleCorner.Parent = toggleGuiButton

-- เมนูหลักตัวยาว
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
fpsButton.Parent = toggleGuiButton

local mainButtonCorner = Instance.new("UICorner")
mainButtonCorner.CornerRadius = UDim.new(0.3, 0) 
mainButtonCorner.Parent = fpsButton

-- ระบบลากปุ่มเล็ก
local dragToggle = nil
local dragSpeed = 0.15
local dragStart = nil
local startPos = nil

local function updateInput(input)
	local delta = input.Position - dragStart
	local position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	game:GetService("TweenService"):Create(toggleGuiButton, TweenInfo.new(dragSpeed), {Position = position}):Play()
end

toggleGuiButton.InputBegan:Connect(function(input)
	if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and not UserInputService:GetFocusedTextBox() then
		dragToggle = true
		dragStart = input.Position
		startPos = toggleGuiButton.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragToggle = false
			end
		end)
	end
end)

toggleGuiButton.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		if dragToggle then
			updateInput(input)
		end
	end
end)

toggleGuiButton.MouseButton1Click:Connect(function()
	fpsButton.Visible = not fpsButton.Visible
	if fpsButton.Visible then
		toggleGuiButton.Text = "-"
	else
		toggleGuiButton.Text = "+"
	end
end)

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
sliderText.Text = "Brightness: 0"
sliderText.TextColor3 = Color3.fromRGB(255, 255, 255)
sliderText.TextScaled = true
sliderText.Font = Enum.Font.SourceSansBold
sliderText.Parent = sliderFrame

local shadowButton = Instance.new("TextButton")
shadowButton.Size = UDim2.new(0.4, 0, 0.2, 0)
shadowButton.Position = UDim2.new(0.075, 0, 0.55, 0)
shadowButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
shadowButton.BorderSizePixel = 0
shadowButton.Text = "Shadow: ON"
shadowButton.TextColor3 = Color3.fromRGB(60, 255, 60)
shadowButton.TextScaled = true
shadowButton.Font = Enum.Font.SourceSansBold
shadowButton.Parent = sliderFrame

local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0.2, 0)
shadowCorner.Parent = shadowButton

local fpsCapButton = Instance.new("TextButton")
fpsCapButton.Size = UDim2.new(0.4, 0, 0.2, 0)
fpsCapButton.Position = UDim2.new(0.525, 0, 0.55, 0)
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
local currentSliderValue = 0
local screenColorCorrection = nil
local shadowActive = true
local fpsCapValues = {60, 90, 120, 144, 240, 999}
local currentFpsCapIndex = 1

local function updateScreenBrightness()
	if not fpsBoostActive then
		if screenColorCorrection then pcall(function() screenColorCorrection:Destroy() end) screenColorCorrection = nil end
		return
	end
	
	if not screenColorCorrection or screenColorCorrection.Parent ~= Lighting then
		if screenColorCorrection then pcall(function() screenColorCorrection:Destroy() end) end
		screenColorCorrection = Instance.new("ColorCorrectionEffect")
		screenColorCorrection.Name = "FPS_ScreenBrightness_Overlay"
		screenColorCorrection.Parent = Lighting
	end
	
	screenColorCorrection.Brightness = currentSliderValue * 0.25
	screenColorCorrection.Contrast = currentSliderValue * 0.1
end

local isChangingLight = false
local function applyLightingTweaks()
	if not fpsBoostActive or isChangingLight then return end
	isChangingLight = true
	Lighting.Brightness = 1
	Lighting.ExposureCompensation = 0
	Lighting.FogColor = Color3.fromRGB(0, 0, 0)
	Lighting.FogStart = 100
	Lighting.FogEnd = 500
	Lighting.GlobalShadows = shadowActive
	isChangingLight = false
end

Lighting.Changed:Connect(function(property)
	if fpsBoostActive and not isChangingLight then
		if property == "Brightness" or property == "ExposureCompensation" or property == "FogEnd" or property == "FogColor" or property == "GlobalShadows" then
			applyLightingTweaks()
		end
	end
end)

-- ลูปอัปเดตตัวเลข FPS & Ping (แก้ไขให้สูตร Ping ตรงกับระบบแท้ของเกม)
local timeBuffer = 0
local frameBuffer = 0
RunService.RenderStepped:Connect(function(deltaTime)
	frameBuffer = frameBuffer + 1
	timeBuffer = timeBuffer + deltaTime
	
	if timeBuffer >= 0.5 then
		local currentFps = math.floor(frameBuffer / timeBuffer)
		local currentPing = 0
		
		pcall(function()
			currentPing = math.floor(Stats.Network.ServerToClientPing:GetValue() * 1000)
		end)
		
		if currentPing == 0 then
			pcall(function()
				-- แก้ไขจาก * 2000 เป็น * 1000 ให้ตรงตามจริงไม่เบิ้ลสองเท่า
				currentPing = math.floor(localPlayer:GetNetworkPing() * 1000)
			end)
		end
		
		local pingString = ""
		if currentPing <= 0 then pingString = "Loading" else pingString = tostring(currentPing) .. " ms" end
		
		trackerLabel.Text = "FPS: " .. tostring(currentFps) .. " | Ping: " .. pingString
		frameBuffer = 0
		timeBuffer = 0
	end
end)

shadowButton.MouseButton1Click:Connect(function()
	shadowActive = not shadowActive
	if shadowActive then
		shadowButton.Text = "Shadow: ON"
		shadowButton.TextColor3 = Color3.fromRGB(60, 255, 60)
	else
		shadowButton.Text = "Shadow: OFF"
		shadowButton.TextColor3 = Color3.fromRGB(255, 60, 60)
	end
	Lighting.GlobalShadows = shadowActive
end)

fpsCapButton.MouseButton1Click:Connect(function()
	currentFpsCapIndex = currentFpsCapIndex + 1
	if currentFpsCapIndex > #fpsCapValues then
		currentFpsCapIndex = 1
	end
	local selectedCap = fpsCapValues[currentFpsCapIndex]
	if selectedCap == 999 then
		fpsCapButton.Text = "FPS Cap: INF"
	else
		fpsCapButton.Text = "FPS Cap: " .. tostring(selectedCap)
	end
	setfpscap(selectedCap)
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
	
	updateScreenBrightness()
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
		
		updateScreenBrightness()
		applyLightingTweaks()
	else
		fpsButton.Text = "boost fps: OFF"
		fpsButton.TextColor3 = Color3.fromRGB(255, 60, 60) 
		sliderFrame.Visible = false 
		
		if screenColorCorrection then pcall(function() screenColorCorrection:Destroy() end) screenColorCorrection = nil end
		pcall(function() if terrain then terrain.Decoration = originalGrass end end)
		
		isChangingLight = true
		Lighting.Brightness = originalBrightness
		Lighting.ExposureCompensation = originalExposure
		Lighting.FogColor = originalFogColor
		Lighting.FogStart = originalFogStart
		Lighting.FogEnd = originalFogEnd
		Lighting.GlobalShadows = true
		isChangingLight = false
		
		pcall(function() MaterialService.Use2022Materials = originalUse2022 end)
		pcall(function() SoundService.AmbientReverb = Enum.AmbientReverb.Default end)
	end
end)
