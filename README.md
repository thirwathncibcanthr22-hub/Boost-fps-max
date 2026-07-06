local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local SoundService = game:GetService("SoundService")
local MaterialService = game:GetService("MaterialService")
local UserInputService = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

local targetParent = playerGui

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

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DeltaOptimizer_V22_FixTrackerHide"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = targetParent

local trackerLabel = Instance.new("TextLabel")
trackerLabel.Size = UDim2.new(0.35, 0, 0.045, 0)
trackerLabel.Position = UDim2.new(0.325, 0, 0.01, 0) 
trackerLabel.BackgroundTransparency = 1 
trackerLabel.Text = "FPS: Calculating... | Ping: Calculating..."
trackerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
trackerLabel.TextScaled = true
trackerLabel.Font = Enum.Font.SourceSansBold
trackerLabel.Visible = false 
trackerLabel.Parent = screenGui

local masterContainer = Instance.new("Frame")
masterContainer.Size = UDim2.new(0, 280, 0, 45) 
masterContainer.Position = UDim2.new(0.02, 0, 0.15, 0)
masterContainer.BackgroundTransparency = 1
masterContainer.Parent = screenGui

local toggleGuiButton = Instance.new("TextButton")
toggleGuiButton.Size = UDim2.new(0, 45, 0, 45) 
toggleGuiButton.Position = UDim2.new(0, 0, 0, 0)
toggleGuiButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
toggleGuiButton.Text = "+"
toggleGuiButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleGuiButton.TextScaled = true
toggleGuiButton.Font = Enum.Font.SourceSansBold
toggleGuiButton.ZIndex = 10
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
fpsButton.ZIndex = 5
fpsButton.Visible = false 
fpsButton.Parent = masterContainer

local mainButtonCorner = Instance.new("UICorner")
mainButtonCorner.CornerRadius = UDim.new(0.3, 0) 
mainButtonCorner.Parent = fpsButton

local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(1.2, 0, 3.3, 0) 
sliderFrame.Position = UDim2.new(-0.1, 0, 1.15, 0) 
sliderFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15) 
sliderFrame.BorderSizePixel = 0
sliderFrame.Visible = false 
sliderFrame.ZIndex = 4
sliderFrame.Parent = fpsButton

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0.08, 0) 
frameCorner.Parent = sliderFrame

local uiListLayout = Instance.new("UIListLayout")
uiListLayout.FillDirection = Enum.FillDirection.Vertical
uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
uiListLayout.VerticalAlignment = Enum.VerticalAlignment.Center
uiListLayout.Padding = UDim.new(0, 8) 
uiListLayout.Parent = sliderFrame

local sliderText = Instance.new("TextLabel")
sliderText.Size = UDim2.new(0.9, 0, 0.15, 0)
sliderText.BackgroundTransparency = 1
sliderText.Text = "Light: Normal"
sliderText.TextColor3 = Color3.fromRGB(255, 255, 255)
sliderText.TextScaled = true
sliderText.Font = Enum.Font.SourceSansBold
sliderText.ZIndex = 6
sliderText.LayoutOrder = 1
sliderText.Parent = sliderFrame

local sliderBarContainer = Instance.new("Frame")
sliderBarContainer.Size = UDim2.new(0.9, 0, 0.2, 0)
sliderBarContainer.BackgroundTransparency = 1
sliderBarContainer.LayoutOrder = 2
sliderBarContainer.Parent = sliderFrame

local sliderBar = Instance.new("TextButton")
sliderBar.Size = UDim2.new(1, 0, 0.4, 0) 
sliderBar.Position = UDim2.new(0, 0, 0.3, 0)
sliderBar.BackgroundColor3 = Color3.fromRGB(45, 45, 45) 
sliderBar.BorderSizePixel = 0
sliderBar.Text = ""
sliderBar.AutoButtonColor = false
sliderBar.ZIndex = 6
sliderBar.Parent = sliderBarContainer

local barCorner = Instance.new("UICorner")
barCorner.CornerRadius = UDim.new(1, 0)
barCorner.Parent = sliderBar

local sliderButton = Instance.new("Frame")
sliderButton.Size = UDim2.new(0, 24, 0, 24) 
sliderButton.Position = UDim2.new(0.5, -12, -0.3, 0) 
sliderButton.BackgroundColor3 = Color3.fromRGB(240, 240, 240) 
sliderButton.BorderSizePixel = 0
sliderButton.ZIndex = 7
sliderButton.Parent = sliderBar

local aspectConstraint = Instance.new("UIAspectRatioConstraint")
aspectConstraint.AspectRatio = 1
aspectConstraint.Parent = sliderButton

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(1, 0) 
circleCorner.Parent = sliderButton

local grayModeButton = Instance.new("TextButton")
grayModeButton.Size = UDim2.new(0.9, 0, 0.22, 0)
grayModeButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
grayModeButton.BorderSizePixel = 0
grayModeButton.Text = "Gray Mode: OFF"
grayModeButton.TextColor3 = Color3.fromRGB(255, 60, 60)
grayModeButton.TextScaled = true
grayModeButton.Font = Enum.Font.SourceSansBold
grayModeButton.ZIndex = 6
grayModeButton.LayoutOrder = 3
grayModeButton.Parent = sliderFrame

local grayModeCorner = Instance.new("UICorner")
grayModeCorner.CornerRadius = UDim.new(0.15, 0)
grayModeCorner.Parent = grayModeButton

local trackerButton = Instance.new("TextButton")
trackerButton.Size = UDim2.new(0.9, 0, 0.22, 0)
trackerButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
trackerButton.BorderSizePixel = 0
trackerButton.Text = "snow fps/ping: OFF"
trackerButton.TextColor3 = Color3.fromRGB(255, 60, 60)
trackerButton.TextScaled = true
trackerButton.Font = Enum.Font.SourceSansBold
trackerButton.ZIndex = 6
trackerButton.LayoutOrder = 4 
trackerButton.Parent = sliderFrame

local trackerButtonCorner = Instance.new("UICorner")
trackerButtonCorner.CornerRadius = UDim.new(0.15, 0)
trackerButtonCorner.Parent = trackerButton

local fpsBoostActive = false
local percentageValue = 0.5 
local grayModeActive = false 
local trackerActive = false 

-- 🛠️ ปรับปรุงระบบลากใหม่: ล็อกให้ปุ่มเครื่องหมายบวก (+) เป็นตัวรับหน้าที่ลากแทนการใช้ตัวปุ่มหลัก เพื่อไม่ให้ทับซ้อนปุ่มคลิกภายใน
local draggingGui = false
local dragInput = nil
local dragStart = nil
local startPos = nil

toggleGuiButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		draggingGui = true
		dragStart = input.Position
		startPos = masterContainer.Position
		
		local connection
		connection = input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				draggingGui = false
				connection:Disconnect()
			end
		end)
	end
end)

toggleGuiButton.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and draggingGui then
		local delta = input.Position - dragStart
		masterContainer.Position = UDim2.new(
			startPos.X.Scale, 
			startPos.X.Offset + delta.X, 
			startPos.Y.Scale, 
			startPos.Y.Offset + delta.Y
		)
	end
end)

-- ปุ่มคลิกหด-ขยายเมนูหลัก
toggleGuiButton.MouseButton1Click:Connect(function()
	fpsButton.Visible = not fpsButton.Visible
	if fpsButton.Visible then
		toggleGuiButton.Text = "-"
		sliderFrame.Visible = true 
	else
		toggleGuiButton.Text = "+"
		sliderFrame.Visible = false
	end
end)

local isChangingLight = false
local function applyLightingTweaks()
	if not fpsBoostActive or isChangingLight then return end
	isChangingLight = true
	local targetVal = percentageValue * 255
	local ambientColor = Color3.fromRGB(targetVal, targetVal, targetVal)
	pcall(function()
		Lighting.Brightness = percentageValue * 3
		Lighting.ExposureCompensation = (percentageValue - 0.5) * 4
		Lighting.Ambient = ambientColor
		Lighting.OutdoorAmbient = ambientColor
		Lighting.FogColor = Color3.fromRGB(0, 0, 0)
		Lighting.FogStart = 100
		Lighting.FogEnd = 500
		Lighting.GlobalShadows = false
	end)
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
	if not (trackerActive or fpsBoostActive) then return end
	
	frameBuffer = frameBuffer + 1
	timeBuffer = timeBuffer + deltaTime
	if timeBuffer >= 0.5 then
		local currentFps = math.floor(frameBuffer / timeBuffer)
		local currentPing = 0
		pcall(function() currentPing = math.floor(Stats.Network.ServerToClientPing:GetValue() * 1000) end)
		if currentPing == 0 then pcall(function() currentPing = math.floor(localPlayer:GetNetworkPing() * 1000) end) end
		trackerLabel.Text = "FPS: " .. tostring(currentFps) .. " | Ping: " .. ((currentPing <= 0) and "Loading" or tostring(currentPing) .. " ms")
		frameBuffer = 0
		timeBuffer = 0
	end
end)

local draggingSlider = false
local function updateSlider(input)
	local barPosition = sliderBar.AbsolutePosition.X
	local barSize = sliderBar.AbsoluteSize.X
	local inputPosition = input.Position.X
	percentageValue = math.clamp((inputPosition - barPosition) / barSize, 0, 1)
	sliderButton.Position = UDim2.new(percentageValue, - (sliderButton.AbsoluteSize.X / 2), -0.3, 0)
	if percentageValue < 0.45 then sliderText.Text = "Light: Dark"
	elseif percentageValue > 0.55 then sliderText.Text = "Light: Bright"
	else sliderText.Text = "Light: Normal" end
	applyLightingTweaks()
end

sliderBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		draggingSlider = true
		updateSlider(input)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		updateSlider(input)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		draggingSlider = false
	end
end)

local function isEssentialObject(object)
	local name = object.Name:lower()
	if object:IsA("BasePart") then
		if name == "head" or name == "torso" or name == "left arm" or name == "right arm" or name == "left leg" or name == "right leg" or name:find("upper") or name:find("lower") or name:find("hand") or name:find("foot") or name == "humanoidrootpart" then
			return true
		end
	end
	if name:find("ball") or name:find("sphere") or name:find("volley") or name:find("tool") or name:find("weapon") then
		return true
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
	pcall(function()
		if isEssentialObject(object) then return end
		local nameLower = object.Name:lower()
		
		if fpsBoostActive then
			if object:IsA("ParticleEmitter") or object:IsA("Sparkles") or object:IsA("Smoke") or 
			   object:IsA("Fire") or object:IsA("Beam") or object:IsA("Trail") or 
			   object:IsA("Explosion") or object:IsA("Highlight") or object:IsA("SelectionBox") or
			   object:IsA("PointLight") or object:IsA("SurfaceLight") or object:IsA("SpotLight") then
				if not (object.Parent and isEssentialObject(object.Parent)) then
					object:Destroy()
					return
				end
			end
			
			if object:IsA("Sound") then
				object.Volume = 0
				object:Stop()
				return
			end
			
			if object:IsA("WrapLayer") or object:IsA("WrapTarget") or object:IsA("FaceControls") then
				object:Destroy()
				return
			end
			
			if object:IsA("BasePart") then
				if object:IsA("Decal") or object:IsA("Texture") then
					if nameLower ~= "face" and not getCharacterOwner(object) then object:Destroy() end
					return
				end
				
				if nameLower:find("leaf") or nameLower:find("leaves") or nameLower:find("foliage") or 
				   nameLower:find("bush") or nameLower:find("flower") or nameLower:find("plant") or 
				   nameLower:find("grass") or nameLower:find("effect") or nameLower:find("vfx") or
				   nameLower:find("chair") or nameLower:find("bench") or nameLower:find("audience") then
					object:Destroy()
					return
				end
			end
		end
		
		if object:IsA("BasePart") then
			if not originalColors[object] then
				originalColors[object] = {
					Color = object.Color, 
					Material = object.Material, 
					BrickColor = object.BrickColor
				}
			end
			
			if fpsBoostActive then
				object.Material = Enum.Material.SmoothPlastic
				object.CastShadow = false
			else
				object.Material = originalColors[object].Material
				object.CastShadow = true
			end
			
			if grayModeActive then 
				object.BrickColor = BrickColor.new("Medium stone gray")
			else 
				object.Color = originalColors[object].Color 
				object.BrickColor = originalColors[object].BrickColor
			end
		end
	end)
end

Workspace.DescendantAdded:Connect(safeEverythingClean)

local currentCamera = Workspace.CurrentCamera or Workspace:WaitForChild("Camera")
currentCamera.DescendantAdded:Connect(safeEverythingClean)

task.spawn(function()
	while true do
		task.wait(3)
		if fpsBoostActive then
			pcall(function()
				for _, effect in pairs(currentCamera:GetDescendants()) do
					if effect:IsA("ParticleEmitter") or effect:IsA("Beam") or effect:IsA("Trail") or effect:IsA("Highlight") then
						effect:Destroy()
					elseif effect:IsA("Sound") then
						effect.Volume = 0
						effect:Stop()
					end
				end
				
				for _, player in pairs(Players:GetPlayers()) do
					local char = player.Character
					if char then
						for _, part in pairs(char:GetChildren()) do
							if part:IsA("Accessory") or part:IsA("Clothing") then
								for _, sub in pairs(part:GetDescendants()) do
									if sub:IsA("WrapLayer") or sub:IsA("ParticleEmitter") or sub:IsA("Trail") then
										sub:Destroy()
									end
								end
							elseif part:IsA("BasePart") then
								for _, sub in pairs(part:GetChildren()) do
									if sub:IsA("FaceControls") or sub:IsA("ParticleEmitter") then
										sub:Destroy()
									end
								end
							end
						end
					end
				end
			end)
		end
	end
end)

grayModeButton.MouseButton1Click:Connect(function()
	grayModeActive = not grayModeActive
	
	grayModeButton.Text = grayModeActive and "Gray Mode: ON" or "Gray Mode: OFF"
	grayModeButton.TextColor3 = grayModeActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	task.spawn(function()
		local count = 0
		for _, object in pairs(Workspace:GetDescendants()) do
			if object:IsA("BasePart") and not isEssentialObject(object) then
				safeEverythingClean(object)
				count = count + 1
				if count % 150 == 0 then task.wait() end
			end
		end
	end)
end)

-- 🛠️ ตรวจสอบจุดนี้: แก้ไข Event ดึงการปิด UI ให้แสดงผลได้อย่างถูกต้อง ไร้บัคขัดขา
trackerButton.MouseButton1Click:Connect(function()
	trackerActive = not trackerActive
	
	trackerButton.Text = trackerActive and "snow fps/ping: ON" or "snow fps/ping: OFF"
	trackerButton.TextColor3 = trackerActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	if trackerActive or fpsBoostActive then
		trackerLabel.Visible = true
	else
		trackerLabel.Visible = false
	end
end)

pcall(function()
	if setfpscap then
		setfpscap(999)
	end
end)

fpsButton.MouseButton1Click:Connect(function()
	fpsBoostActive = not fpsBoostActive
	
	if fpsBoostActive then
		fpsButton.Text = "boost fps: ON"
		fpsButton.TextColor3 = Color3.fromRGB(60, 255, 60) 
		trackerLabel.Visible = true 
		
		pcall(function()
			if terrain then terrain.Decoration = false end
			MaterialService.Use2022Materials = false
			SoundService.AmbientReverb = Enum.AmbientReverb.NoReverb
			
			for _, effect in pairs(Lighting:GetChildren()) do
				if effect:IsA("PostEffect") or effect:IsA("BlurEffect") or effect:IsA("BloomEffect") or effect:IsA("SunRaysEffect") then
					effect:Destroy()
				end
			end
			for _, effect in pairs(currentCamera:GetDescendants()) do
				safeEverythingClean(effect)
			end
		end)
		
		task.spawn(function()
			local items = Workspace:GetDescendants()
			for idx, object in pairs(items) do
				if not fpsBoostActive then break end
				safeEverythingClean(object)
				if idx % 120 == 0 then task.wait() end
			end
		end)
		
		applyLightingTweaks()
	else
		fpsButton.Text = "boost fps: OFF"
		fpsButton.TextColor3 = Color3.fromRGB(255, 60, 60) 
		
		if not trackerActive then
			trackerLabel.Visible = false 
		end
		
		pcall(function() if terrain then terrain.Decoration = originalGrass end end)
		
		task.spawn(function()
			local count = 0
			for object, data in pairs(originalColors) do
				if object and object.Parent then
					pcall(function()
						object.Material = data.Material
						object.CastShadow = true
						if not grayModeActive then
							object.Color = data.Color
							object.BrickColor = data.BrickColor
						end
					end)
					count = count + 1
					if count % 150 == 0 then task.wait() end
				end
			end
		end)
		
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
