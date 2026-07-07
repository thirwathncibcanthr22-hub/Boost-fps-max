local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

local originalBrightness, originalAmbient = Lighting.Brightness, Lighting.Ambient
local originalOutdoorAmbient = Lighting.OutdoorAmbient
local originalFogStart, originalFogEnd = Lighting.FogStart, Lighting.FogEnd

local originalColors, originalPlayerParts = {}, {}
local fpsBoostActive, percentageValue, grayModeActive, trackerActive, simplifyPlayersActive = false, 0.5, false, false, false

-- สร้าง UI เมนูหลัก
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PureFPS_AllInOne_v11"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local trackerLabel = Instance.new("TextLabel")
trackerLabel.Size, trackerLabel.Position = UDim2.new(0.35, 0, 0.045, 0), UDim2.new(0.325, 0, 0.01, 0)
trackerLabel.BackgroundTransparency, trackerLabel.Text = 1, "FPS: Calculating..."
trackerLabel.TextColor3, trackerLabel.TextScaled, trackerLabel.Font = Color3.fromRGB(255, 255, 255), true, Enum.Font.SourceSansBold
trackerLabel.Visible, trackerLabel.Parent = false, screenGui

local masterContainer = Instance.new("Frame")
masterContainer.Size, masterContainer.Position = UDim2.new(0, 280, 0, 45), UDim2.new(0.02, 0, 0.15, 0)
masterContainer.BackgroundTransparency, masterContainer.ZIndex, masterContainer.Parent = 1, 10, screenGui

local toggleGuiButton = Instance.new("TextButton")
toggleGuiButton.Size, toggleGuiButton.BackgroundColor3 = UDim2.new(0, 45, 0, 45), Color3.fromRGB(20, 20, 20)
toggleGuiButton.Text, toggleGuiButton.TextColor3, toggleGuiButton.TextScaled = "+", Color3.fromRGB(255, 255, 255), true
toggleGuiButton.Font, toggleGuiButton.ZIndex, toggleGuiButton.Parent = Enum.Font.SourceSansBold, 11, masterContainer
Instance.new("UICorner", toggleGuiButton).CornerRadius = UDim.new(1, 0)

local fpsButton = Instance.new("TextButton")
fpsButton.Size, fpsButton.Position = UDim2.new(0, 220, 0, 45), UDim2.new(0, 55, 0, 0)
fpsButton.BackgroundColor3, fpsButton.Text = Color3.fromRGB(15, 15, 15), "boost fps: OFF"
fpsButton.TextColor3, fpsButton.TextScaled, fpsButton.Font = Color3.fromRGB(255, 60, 60), true, Enum.Font.SourceSansBold
fpsButton.ZIndex, fpsButton.Visible, fpsButton.Parent = 5, false, masterContainer
Instance.new("UICorner", fpsButton).CornerRadius = UDim.new(0.3, 0)

-- ปรับขนาดกรอบให้พอดีกับปุ่มที่เหลือ
local sliderFrame = Instance.new("Frame")
sliderFrame.Size, sliderFrame.Position = UDim2.new(1.25, 0, 5.5, 0), UDim2.new(-0.125, 0, 1.15, 0)
sliderFrame.BackgroundColor3, sliderFrame.Visible, sliderFrame.ZIndex, sliderFrame.Parent = Color3.fromRGB(15, 15, 15), false, 4, fpsButton
Instance.new("UICorner", sliderFrame).CornerRadius = UDim.new(0.05, 0)

local uiListLayout = Instance.new("UIListLayout")
uiListLayout.FillDirection, uiListLayout.SortOrder = Enum.FillDirection.Vertical, Enum.SortOrder.LayoutOrder
uiListLayout.HorizontalAlignment, uiListLayout.VerticalAlignment = Enum.HorizontalAlignment.Center, Enum.VerticalAlignment.Center
uiListLayout.Padding, uiListLayout.Parent = UDim.new(0, 8), sliderFrame

-- 1. แถบสไลเดอร์ปรับแสงสว่าง
local sliderText = Instance.new("TextLabel")
sliderText.Size, sliderText.BackgroundTransparency, sliderText.Text = UDim2.new(0.9, 0, 0.1, 0), 1, "Light: Normal"
sliderText.TextColor3, sliderText.TextScaled, sliderText.Font, sliderText.ZIndex, sliderText.LayoutOrder = Color3.fromRGB(255, 255, 255), true, Enum.Font.SourceSansBold, 6, 1
sliderText.Parent = sliderFrame

local sliderBarContainer = Instance.new("Frame")
sliderBarContainer.Size, sliderBarContainer.BackgroundTransparency, sliderBarContainer.LayoutOrder = UDim2.new(0.9, 0, 0.08, 0), 1, 2
sliderBarContainer.Parent = sliderFrame

local sliderBar = Instance.new("TextButton")
sliderBar.Size, sliderBar.Position = UDim2.new(1, 0, 0.35, 0), UDim2.new(0, 0, 0.3, 0)
sliderBar.BackgroundColor3, sliderBar.Text, sliderBar.AutoButtonColor, sliderBar.ZIndex = Color3.fromRGB(45, 45, 45), "", false, 6
sliderBar.Parent = sliderBarContainer
Instance.new("UICorner", sliderBar).CornerRadius = UDim.new(1, 0)

local sliderButton = Instance.new("Frame")
sliderButton.Size, sliderButton.Position = UDim2.new(0, 18, 0, 18), UDim2.new(0.5, -9, -0.2, 0)
sliderButton.BackgroundColor3, sliderButton.ZIndex, sliderButton.Parent = Color3.fromRGB(240, 240, 240), 7, sliderBar
Instance.new("UICorner", sliderButton).CornerRadius = UDim.new(1, 0)

-- ปุ่ม Simplify Players
local simplifyButton = Instance.new("TextButton")
simplifyButton.Size, simplifyButton.BackgroundColor3 = UDim2.new(0.92, 0, 0.16, 0), Color3.fromRGB(30, 30, 30)
simplifyButton.Text, simplifyButton.TextColor3, simplifyButton.TextScaled = "Simplify Players: OFF", Color3.fromRGB(255, 60, 60), true
simplifyButton.Font, simplifyButton.ZIndex, simplifyButton.LayoutOrder = Enum.Font.SourceSansBold, 6, 3
simplifyButton.Parent = sliderFrame
Instance.new("UICorner", simplifyButton).CornerRadius = UDim.new(0.15, 0)

-- ปุ่ม Gray Mode
local grayModeButton = Instance.new("TextButton")
grayModeButton.Size, grayModeButton.BackgroundColor3 = UDim2.new(0.92, 0, 0.16, 0), Color3.fromRGB(30, 30, 30)
grayModeButton.Text, grayModeButton.TextColor3, grayModeButton.TextScaled = "Gray Mode: OFF", Color3.fromRGB(255, 60, 60), true
grayModeButton.Font, grayModeButton.ZIndex, grayModeButton.LayoutOrder = Enum.Font.SourceSansBold, 6, 4
grayModeButton.Parent = sliderFrame
Instance.new("UICorner", grayModeButton).CornerRadius = UDim.new(0.15, 0)

-- ปุ่มแสดง FPS/Ping
local trackerButton = Instance.new("TextButton")
trackerButton.Size, trackerButton.BackgroundColor3 = UDim2.new(0.92, 0, 0.16, 0), Color3.fromRGB(30, 30, 30)
trackerButton.Text, trackerButton.TextColor3, trackerButton.TextScaled = "show fps/ping: OFF", Color3.fromRGB(255, 60, 60), true
trackerButton.Font, trackerButton.ZIndex, trackerButton.LayoutOrder = Enum.Font.SourceSansBold, 6, 5
trackerButton.Parent = sliderFrame
Instance.new("UICorner", trackerButton).CornerRadius = UDim.new(0.15, 0)

-- ระบบลาก GUI
local draggingGui, dragInput, dragStart, startPos = false
toggleGuiButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		draggingGui, dragStart, startPos = true, input.Position, masterContainer.Position
		input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then draggingGui = false end end)
	end
end)
toggleGuiButton.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end end)
UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and draggingGui then
		local delta = input.Position - dragStart
		masterContainer.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

toggleGuiButton.MouseButton1Click:Connect(function()
	fpsButton.Visible = not fpsButton.Visible
	toggleGuiButton.Text = fpsButton.Visible and "-" or "+"
	sliderFrame.Visible = fpsButton.Visible
end)

local function applyLightingTweaks()
	if not fpsBoostActive then return end
	local targetVal = percentageValue * 255
	local ambientColor = Color3.fromRGB(targetVal, targetVal, targetVal)
	pcall(function()
		Lighting.Brightness = percentageValue * 3
		if not grayModeActive then Lighting.Ambient, Lighting.OutdoorAmbient = ambientColor, ambientColor end
	end)
end

-- Camera Optimization
RunService.RenderStepped:Connect(function()
	local camera = Workspace.CurrentCamera
	if not camera then return end
	pcall(function()
		if camera.CameraType ~= Enum.CameraType.Custom then camera.CameraType = Enum.CameraType.Custom end
		if fpsBoostActive then
			camera.CameraMinZoomDistance = 0.5
			localPlayer.DevCameraOcclusionMode = Enum.DevCameraOcclusionMode.Invisicam
		else
			localPlayer.DevCameraOcclusionMode = Enum.DevCameraOcclusionMode.Zoom
		end
	end)
end)

-- ลบขยะเอฟเฟกต์รกๆ ทุก 5 วินาที
task.spawn(function()
	while task.wait(5) do
		if fpsBoostActive then
			pcall(function()
				for _, v in pairs(Workspace:GetDescendants()) do
					if v:IsA("Explosion") or v.Name == "Effect" or v.Name == "Particle" or v:IsA("Debris") then v:Destroy() end
				end
			end)
		end
	end
end)

-- ระบบนับ FPS / Ping
local timeBuffer, frameBuffer = 0, 0
RunService.RenderStepped:Connect(function(deltaTime)
	if not trackerActive then return end
	frameBuffer, timeBuffer = frameBuffer + 1, timeBuffer + deltaTime
	if timeBuffer >= 0.5 then
		local currentFps, currentPing = math.floor(frameBuffer / timeBuffer), 0
		pcall(function() currentPing = math.floor(localPlayer:GetNetworkPing() * 1000) end)
		if currentPing <= 0 then pcall(function() currentPing = math.floor(Stats.Network.ServerToClientPing:GetValue() * 1000) end) end
		trackerLabel.Text = "FPS: " .. tostring(currentFps) .. " | Ping: " .. tostring(currentPing) .. " ms"
		frameBuffer, timeBuffer = 0, 0
	end
end)

-- ระบบลากแถบปรับแสง
local draggingSlider = false
local function updateSlider(input)
	percentageValue = math.clamp((input.Position.X - sliderBar.AbsolutePosition.X) / sliderBar.AbsoluteSize.X, 0, 1)
	sliderButton.Position = UDim2.new(percentageValue, percentageValue == 1 and -18 or -9, -0.2, 0)
	sliderText.Text = percentageValue < 0.45 and "Light: Dark" or (percentageValue > 0.55 and "Light: Bright" or "Light: Normal")
	applyLightingTweaks()
end
sliderBar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then draggingSlider = true updateSlider(input) end end)
UserInputService.InputChanged:Connect(function(input) if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and draggingSlider then updateSlider(input) end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then draggingSlider = false end end)

local function isEssentialObject(object)
	local name = object.Name:lower()
	return name:find("ball") or name:find("sphere") or name:find("volley")
end

-- ระบบลดดีเทลตัวผู้เล่นอื่น (Simplify Players)
local function cleanSinglePlayer(character)
	pcall(function()
		local pl = Players:GetPlayerFromCharacter(character)
		if pl == localPlayer then return end
		for _, v in pairs(character:GetDescendants()) do
			if v:IsA("Shirt") or v:IsA("Pants") or v:IsA("GraphicShirt") or v:IsA("ShirtGraphic") or v:IsA("Accessory") or v:IsA("Hat") then
				if not originalPlayerParts[v] then originalPlayerParts[v] = v.Parent end
				v.Parent = nil
			elseif v:IsA("BasePart") then
				if not originalPlayerParts[v] then originalPlayerParts[v] = {Material = v.Material} end
				v.Material = Enum.Material.SmoothPlastic
			end
		end
	end)
end

-- ฟังก์ชันล้างแมพให้เป็นดินน้ำมันเลโก้เกลี้ยงเกลา
local function safeEverythingClean(object)
	pcall(function()
		if isEssentialObject(object) then return end
		
		if fpsBoostActive and (object:IsA("Decal") or object:IsA("Texture")) then
			object.Transparency = 1
			return
		end
		
		if fpsBoostActive and (object:IsA("MeshPart") or object:IsA("SpecialMesh")) then
			if not originalColors[object] then originalColors[object] = {Material = object.Material, BrickColor = object.BrickColor, TextureID = object.TextureID} end
			object.TextureID = ""
		end

		if fpsBoostActive and (object:IsA("Terrain") or object.Name == "Foliage" or object.Name == "Leaves" or object.Material == Enum.Material.Grass) then
			if object:IsA("Terrain") then object.Decoration = false else
				if not originalColors[object] then originalColors[object] = {Material = object.Material, BrickColor = object.BrickColor, TextureID = ""} end
				object.Transparency, object.CanCollide = 1, false return
			end
		end
		
		if fpsBoostActive and (object:IsA("Smoke") or object:IsA("Fire") or object:IsA("Sparkles") or object:IsA("ForceField") or object:IsA("ParticleEmitter") or object:IsA("Trail") or object:IsA("Highlight")) then 
			object:Destroy() return 
		end
		
		if object:IsA("BasePart") then
			if not originalColors[object] then originalColors[object] = {Material = object.Material, BrickColor = object.BrickColor, TextureID = ""} end
			object.Material = fpsBoostActive and Enum.Material.SmoothPlastic or originalColors[object].Material
			if fpsBoostActive then object.CastShadow = false end
			object.BrickColor = grayModeActive and BrickColor.new("Medium stone gray") or originalColors[object].BrickColor
		end
	end)
end

Workspace.DescendantAdded:Connect(function(object)
	safeEverythingClean(object)
	if simplifyPlayersActive and object.Parent and object.Parent.Parent == Workspace and Players:GetPlayerFromCharacter(object.Parent) then
		cleanSinglePlayer(object.Parent)
	end
end)

-- ปุ่มเปิดปิดการลดดีเทลตัวผู้เล่นอื่น
simplifyButton.MouseButton1Click:Connect(function()
	simplifyPlayersActive = not simplifyPlayersActive
	simplifyButton.Text = simplifyPlayersActive and "Simplify Players: ON" or "Simplify Players: OFF"
	simplifyButton.TextColor3 = simplifyPlayersActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	if simplifyPlayersActive then
		task.spawn(function()
			local allPlayers = Players:GetPlayers()
			for i, p in pairs(allPlayers) do
				if p.Character then cleanSinglePlayer(p.Character) end
				if i % 5 == 0 then task.wait() end 
			end
		end)
	else
		for v, parent in pairs(originalPlayerParts) do
			pcall(function()
				if typeof(parent) == "Instance" then v.Parent = parent else v.Material = parent.Material end
			end)
		end
		table.clear(originalPlayerParts)
	end
end)

grayModeButton.MouseButton1Click:Connect(function()
	grayModeActive = not grayModeActive
	grayModeButton.Text = grayModeActive and "Gray Mode: ON" or "Gray Mode: OFF"
	grayModeButton.TextColor3 = grayModeActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	if grayModeActive then
		for _, asset in pairs(Lighting:GetChildren()) do if asset:IsA("Sky") or asset:IsA("Atmosphere") or asset:IsA("Clouds") or asset:IsA("SunRaysEffect") then asset:Destroy() end end
		Lighting.Ambient, Lighting.OutdoorAmbient = Color3.fromRGB(120, 120, 120), Color3.fromRGB(120, 120, 120)
	else
		Lighting.Ambient, Lighting.OutdoorAmbient = originalAmbient, originalOutdoorAmbient
	end
	
	task.spawn(function()
		local allDescendants = Workspace:GetDescendants()
		for i, object in pairs(allDescendants) do
			safeEverythingClean(object)
			if i % 100 == 0 then task.wait() end 
		end
	end)
end)

trackerButton.MouseButton1Click:Connect(function()
	trackerActive = not trackerActive
	trackerButton.Text = trackerActive and "show fps/ping: ON" or "show fps/ping: OFF"
	trackerButton.TextColor3 = trackerActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	trackerLabel.Visible = trackerActive
end)

-- ปุ่มหลัก Boost FPS (ดินน้ำมันเลโก้ + No Fog ลบหมอกในปุ่มเดียว)
fpsButton.MouseButton1Click:Connect(function()
	fpsBoostActive = not fpsBoostActive
	fpsButton.Text = fpsBoostActive and "boost fps: ON" or "boost fps: OFF"
	fpsButton.TextColor3 = fpsBoostActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	if fpsBoostActive then
		pcall(function() 
			setfpscap(999) 
			settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
			Lighting.GlobalShadows = false
			-- ลบหมอกทันที
			Lighting.FogStart = 999999
			Lighting.FogEnd = 999999
		end)
		for _, asset in pairs(Lighting:GetChildren()) do if asset:IsA("Sky") or asset:IsA("Atmosphere") or asset:IsA("Clouds") or asset:IsA("SunRaysEffect") then asset:Destroy() end end
		
		task.spawn(function()
			local allDescendants = Workspace:GetDescendants()
			for i, object in pairs(allDescendants) do
				safeEverythingClean(object)
				if i % 100 == 0 then task.wait() end 
			end
		end)
		applyLightingTweaks()
	else
		pcall(function() 
			setfpscap(60) 
			settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
			Lighting.GlobalShadows = true
			-- คืนค่าหมอกดั้งเดิมของแมพ
			Lighting.FogStart = originalFogStart
			Lighting.FogEnd = originalFogEnd
		end)
		for object, data in pairs(originalColors) do
			pcall(function()
				if object:IsA("Decal") or object:IsA("Texture") then
					object.Transparency = 0
				elseif object:IsA("MeshPart") or object:IsA("SpecialMesh") then
					object.TextureID = data.TextureID
					object.Material = data.Material
				elseif object:IsA("BasePart") then
					object.Material = data.Material
					if not grayModeActive then object.BrickColor = data.BrickColor end
					object.CastShadow = true
				end
			end)
		end
		pcall(function() Workspace.Terrain.Decoration = true end)
		Lighting.Brightness = originalBrightness
		if not grayModeActive then Lighting.Ambient, Lighting.OutdoorAmbient = originalAmbient, originalOutdoorAmbient end
	end
end)
