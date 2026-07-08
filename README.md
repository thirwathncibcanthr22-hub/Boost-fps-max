local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

local originalBrightness = Lighting.Brightness
local originalAmbient = Lighting.Ambient
local originalOutdoorAmbient = Lighting.OutdoorAmbient
local originalFogStart = Lighting.FogStart
local originalFogEnd = Lighting.FogEnd
local originalSpecular = 0
local originalDiffuse = 0

pcall(function()
	originalSpecular = Lighting.EnvironmentSpecularScale or 0
	originalDiffuse = Lighting.EnvironmentDiffuseScale or 0
end)

local originalColors, originalPlayerParts, originalMapStates = {}, {}, {}
local originalSkyObjects = {}
local fpsBoostActive, percentageValue, grayModeActive, trackerActive, simplifyPlayersActive = false, 0.5, false, false, false
local timeBuffer, frameBuffer = 0, 0

pcall(function()
	if playerGui:FindFirstChild("TSB_InstantAll_v18_5") then playerGui["TSB_InstantAll_v18_5"]:Destroy() end
	if playerGui:FindFirstChild("TSB_Tracker_LayerTop") then playerGui["TSB_Tracker_LayerTop"]:Destroy() end
end)

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TSB_InstantAll_v18_5"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local trackerGui = Instance.new("ScreenGui")
trackerGui.Name = "TSB_Tracker_LayerTop"
trackerGui.ResetOnSpawn = false
trackerGui.DisplayOrder = 999999
trackerGui.Parent = playerGui

local trackerLabel = Instance.new("TextLabel")
trackerLabel.Size, trackerLabel.Position = UDim2.new(0.35, 0, 0.045, 0), UDim2.new(0.325, 0, 0.01, 0)
trackerLabel.BackgroundTransparency = 1
trackerLabel.TextColor3, trackerLabel.TextScaled, trackerLabel.Font = Color3.fromRGB(255, 255, 255), true, Enum.Font.SourceSansBold
trackerLabel.ZIndex = 999999
trackerLabel.Visible, trackerLabel.Parent = false, trackerGui

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

local sliderFrame = Instance.new("Frame")
sliderFrame.Size, sliderFrame.Position = UDim2.new(1.25, 0, 0, 0), UDim2.new(-0.125, 0, 1.15, 0)
sliderFrame.AutomaticSize = Enum.AutomaticSize.Y
sliderFrame.BackgroundColor3, sliderFrame.Visible, sliderFrame.ZIndex, sliderFrame.Parent = Color3.fromRGB(15, 15, 15), false, 4, fpsButton
Instance.new("UICorner", sliderFrame).CornerRadius = UDim.new(0.05, 0)

local uiListLayout = Instance.new("UIListLayout")
uiListLayout.FillDirection, uiListLayout.SortOrder = Enum.FillDirection.Vertical, Enum.SortOrder.LayoutOrder
uiListLayout.HorizontalAlignment, uiListLayout.VerticalAlignment = Enum.HorizontalAlignment.Center, Enum.VerticalAlignment.Top
uiListLayout.Padding, uiListLayout.Parent = UDim.new(0, 8), sliderFrame

local uiPadding = Instance.new("UIPadding")
uiPadding.PaddingTop, uiPadding.PaddingBottom = UDim.new(0, 10), UDim.new(0, 10)
uiPadding.Parent = sliderFrame

local sliderText = Instance.new("TextLabel")
sliderText.Size, sliderText.BackgroundTransparency, sliderText.Text = UDim2.new(0.9, 0, 0, 25), 1, "Light: Normal"
sliderText.TextColor3, sliderText.TextScaled, sliderText.Font, sliderText.ZIndex, sliderText.LayoutOrder = Color3.fromRGB(255, 255, 255), true, Enum.Font.SourceSansBold, 6, 1
sliderText.Parent = sliderFrame

local sliderBarContainer = Instance.new("Frame")
sliderBarContainer.Size, sliderBarContainer.BackgroundTransparency, sliderBarContainer.LayoutOrder = UDim2.new(0.9, 0, 0, 20), 1, 2
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

local simplifyButton = Instance.new("TextButton")
simplifyButton.Size, simplifyButton.BackgroundColor3 = UDim2.new(0.92, 0, 0, 40), Color3.fromRGB(30, 30, 30)
simplifyButton.Text, simplifyButton.TextColor3, simplifyButton.TextScaled = "Simplify Players: OFF", Color3.fromRGB(255, 60, 60), true
simplifyButton.Font, simplifyButton.ZIndex, simplifyButton.LayoutOrder = Enum.Font.SourceSansBold, 6, 3
simplifyButton.Parent = sliderFrame
Instance.new("UICorner", simplifyButton).CornerRadius = UDim.new(0.15, 0)

local grayModeButton = Instance.new("TextButton")
grayModeButton.Size, grayModeButton.BackgroundColor3 = UDim2.new(0.92, 0, 0, 40), Color3.fromRGB(30, 30, 30)
grayModeButton.Text, grayModeButton.TextColor3, grayModeButton.TextScaled = "Gray Mode: OFF", Color3.fromRGB(255, 60, 60), true
grayModeButton.Font, grayModeButton.ZIndex, grayModeButton.LayoutOrder = Enum.Font.SourceSansBold, 6, 4
grayModeButton.Parent = sliderFrame
Instance.new("UICorner", grayModeButton).CornerRadius = UDim.new(0.15, 0)

local trackerButton = Instance.new("TextButton")
trackerButton.Size, trackerButton.BackgroundColor3 = UDim2.new(0.92, 0, 0, 40), Color3.fromRGB(30, 30, 30)
trackerButton.Text, trackerButton.TextColor3, trackerButton.TextScaled = "show fps/ping: OFF", Color3.fromRGB(255, 60, 60), true
trackerButton.Font, trackerButton.ZIndex, trackerButton.LayoutOrder = Enum.Font.SourceSansBold, 6, 5
trackerButton.Parent = sliderFrame
Instance.new("UICorner", trackerButton).CornerRadius = UDim.new(0.15, 0)

-- ลาก GUI
local draggingGui, dragInput, dragStart, startPos = false
toggleGuiButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		draggingGui, dragStart, startPos = true, input.Position, masterContainer.Position
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if draggingGui and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		masterContainer.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then draggingGui = false end
end)

toggleGuiButton.MouseButton1Click:Connect(function()
	fpsButton.Visible = not fpsButton.Visible
	toggleGuiButton.Text = fpsButton.Visible and "-" or "+"
	sliderFrame.Visible = fpsButton.Visible
end)

local function applyLightingTweaks()
	if not fpsBoostActive then return end
	pcall(function()
		local targetVal = percentageValue * 255
		Lighting.Brightness = percentageValue * 3
		if not grayModeActive then 
			Lighting.Ambient = Color3.fromRGB(targetVal, targetVal, targetVal)
			Lighting.OutdoorAmbient = Color3.fromRGB(targetVal, targetVal, targetVal)
		end
	end)
end

local draggingSlider = false
local function updateSlider(input)
	pcall(function()
		percentageValue = math.clamp((input.Position.X - sliderBar.AbsolutePosition.X) / sliderBar.AbsoluteSize.X, 0, 1)
		sliderButton.Position = UDim2.new(percentageValue, percentageValue == 1 and -18 or -9, -0.2, 0)
		sliderText.Text = percentageValue < 0.45 and "Light: Dark" or (percentageValue > 0.55 and "Light: Bright" or "Light: Normal")
		applyLightingTweaks()
	end)
end
sliderBar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then draggingSlider = true updateSlider(input) end end)
UserInputService.InputChanged:Connect(function(input) if draggingSlider then updateSlider(input) end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then draggingSlider = false end end)

-- FPS/Ping Counter
RunService.RenderStepped:Connect(function(deltaTime)
	if not trackerActive then return end
	timeBuffer = timeBuffer + deltaTime
	frameBuffer = frameBuffer + 1
	if timeBuffer >= 0.5 then
		pcall(function()
			local currentFps = math.floor(frameBuffer / timeBuffer)
			local currentPing = math.floor(localPlayer:GetNetworkPing() * 1000)
			if currentPing <= 0 then currentPing = math.floor(Stats.Network.ServerToClientPing:GetValue() * 1000) end
			trackerLabel.Text = "FPS: " .. tostring(currentFps) .. " | Ping: " .. tostring(currentPing) .. " ms"
		end)
		frameBuffer, timeBuffer = 0, 0
	end
end)

local function isMapStructure(object)
	if not object:IsA("BasePart") then return false end
	if object.Parent and Players:GetPlayerFromCharacter(object.Parent) then return false end
	local name = object.Name:lower()
	local pName = object.Parent and object.Parent.Name:lower() or ""
	if name:find("building") or name:find("wall") or name:find("mountain") or name:find("rock") or name:find("tree") or name:find("pillar") or name:find("house") or name:find("destruct") or name:find("map") or name:find("statue") then return true end
	if pName:find("building") or pName:find("wall") or pName:find("map") or pName:find("scenery") or pName:find("stage") or pName:find("prop") then return true end
	if object.Size.X > 25 or object.Size.Y > 25 or object.Size.Z > 25 then
		if not name:find("floor") and not name:find("ground") and not name:find("baseplate") then return true end
	end
	return false
end

-- Instant Execution (จัดการแมพและตัวละครอื่นทันทีรอบเดียว)
local function runMassClean()
	for _, object in pairs(Workspace:GetDescendants()) do
		pcall(function()
			if object.Parent and Players:GetPlayerFromCharacter(object.Parent) then
				local charPlayer = Players:GetPlayerFromCharacter(object.Parent)
				if charPlayer ~= localPlayer then
					if fpsBoostActive and object:IsA("BasePart") and object.Transparency == 1 then
						object.Transparency = 0.5
					end
					if object:IsA("Humanoid") then
						if fpsBoostActive then
							object:SetStateEnabled(Enum.HumanoidStateType.Climbing, false)
							object:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
							object:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
							object:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
						else
							object:SetStateEnabled(Enum.HumanoidStateType.Climbing, true)
							object:SetStateEnabled(Enum.HumanoidStateType.Swimming, true)
							object:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
							object:SetStateEnabled(Enum.HumanoidStateType.GettingUp, true)
						end
					end
				end
				return
			end
			
			local objName = object.Name:lower()
			if fpsBoostActive then
				if objName:find("dash") or objName:find("wind") or objName:find("trail") or objName:find("sweep") or objName:find("burst") or objName:find("rubble") or objName:find("break") or objName:find("shattered") or objName:find("destruction") or objName:find("effect") or objName:find("particle") or objName:find("rocks") or objName:find("debris") or objName:find("groundcrack") then
					object:Destroy() return
				end
				if object:IsA("Smoke") or object:IsA("Fire") or object:IsA("Sparkles") or object:IsA("ParticleEmitter") or object:IsA("Trail") or object:IsA("Highlight") then
					object:Destroy() return
				end
				if object:IsA("Decal") or object:IsA("Texture") then
					object.Transparency = 1 return
				end
				if object:IsA("MeshPart") or object:IsA("SpecialMesh") then
					if not originalColors[object] then originalColors[object] = {Material = object.Material, BrickColor = object.BrickColor, TextureID = object.TextureID} end
					object.TextureID = ""
				end
				if object:IsA("Terrain") then object.Decoration = false end
			end

			if object:IsA("BasePart") then
				if not originalColors[object] then originalColors[object] = {Material = object.Material, BrickColor = object.BrickColor, TextureID = ""} end
				
				if fpsBoostActive and isMapStructure(object) then
					if not originalMapStates[object] then originalMapStates[object] = {Transparency = object.Transparency, CanCollide = object.CanCollide} end
					object.Transparency = 1
					object.CanCollide = false
					return
				end

				object.Material = fpsBoostActive and Enum.Material.SmoothPlastic or originalColors[object].Material
				if fpsBoostActive then object.CastShadow = false end
				object.BrickColor = grayModeActive and BrickColor.new("Dark stone gray") or originalColors[object].BrickColor
			end
		end)
	end
end

-- Instant Simplify Players
simplifyButton.MouseButton1Click:Connect(function()
	simplifyPlayersActive = not simplifyPlayersActive
	simplifyButton.Text = simplifyPlayersActive and "Simplify Players: ON" or "Simplify Players: OFF"
	simplifyButton.TextColor3 = simplifyPlayersActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	if simplifyPlayersActive then
		for _, p in pairs(Players:GetPlayers()) do
			pcall(function()
				if p ~= localPlayer and p.Character then
					for _, v in pairs(p.Character:GetDescendants()) do
						if v:IsA("Shirt") or v:IsA("Pants") or v:IsA("GraphicShirt") or v:IsA("ShirtGraphic") or v:IsA("Accessory") or v:IsA("Hat") then
							originalPlayerParts[v] = v.Parent
							v.Parent = nil
						elseif v:IsA("BasePart") then
							originalPlayerParts[v] = {Material = v.Material}
							v.Material = Enum.Material.SmoothPlastic
						end
					end
				end
			end)
		end
	else
		for v, parent in pairs(originalPlayerParts) do
			pcall(function() if typeof(parent) == "Instance" then v.Parent = parent else v.Material = parent.Material end end)
		end
		table.clear(originalPlayerParts)
	end
end)

-- Instant Gray Mode
grayModeButton.MouseButton1Click:Connect(function()
	grayModeActive = not grayModeActive
	grayModeButton.Text = grayModeActive and "Gray Mode: ON" or "Gray Mode: OFF"
	grayModeButton.TextColor3 = grayModeActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	pcall(function()
		if grayModeActive then
			Lighting.Ambient = Color3.fromRGB(55, 55, 55)
			Lighting.OutdoorAmbient = Color3.fromRGB(55, 55, 55)
		else
			Lighting.Ambient, Lighting.OutdoorAmbient = originalAmbient, originalOutdoorAmbient
		end
	end)
	runMassClean()
end)

trackerButton.MouseButton1Click:Connect(function()
	trackerActive = not trackerActive
	trackerButton.Text = trackerActive and "show fps/ping: ON" or "show fps/ping: OFF"
	trackerButton.TextColor3 = trackerActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	trackerLabel.Visible = trackerActive
end)

-- Instant Boost FPS
fpsButton.MouseButton1Click:Connect(function()
	fpsBoostActive = not fpsBoostActive
	fpsButton.Text = fpsBoostActive and "boost fps: ON" or "boost fps: OFF"
	fpsButton.TextColor3 = fpsBoostActive and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
	
	pcall(function() 
		if fpsBoostActive then
			pcall(function() setfpscap(999) end)
			pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
			Lighting.GlobalShadows = false
			Lighting.FogStart, Lighting.FogEnd = 999999, 999999
			
			pcall(function() Lighting.EnvironmentSpecularScale = 0 end)
			pcall(function() Lighting.EnvironmentDiffuseScale = 0 end)
			
			for _, child in pairs(Lighting:GetChildren()) do
				if child:IsA("Sky") or child:IsA("Atmosphere") or child:IsA("Clouds") or child:IsA("SunRays") or child:IsA("Bloom") or child:IsA("Blur") or child:IsA("ColorCorrection") then
					originalSkyObjects[child] = child.Parent
					child.Parent = nil
				end
			end
			
			runMassClean()
			applyLightingTweaks()
		else
			pcall(function() setfpscap(60) end)
			pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic end)
			Lighting.GlobalShadows = true
			Lighting.FogStart, Lighting.FogEnd = originalFogStart, originalFogEnd
			
			pcall(function() Lighting.EnvironmentSpecularScale = originalSpecular end)
			pcall(function() Lighting.EnvironmentDiffuseScale = originalDiffuse end)
			
			for obj, parent in pairs(originalSkyObjects) do
				pcall(function() obj.Parent = parent end)
			end
			table.clear(originalSkyObjects)
			
			for object, data in pairs(originalMapStates) do
				pcall(function()
					object.Transparency = data.Transparency
					object.CanCollide = data.CanCollide
				end)
			end
			table.clear(originalMapStates)

			for object, data in pairs(originalColors) do
				pcall(function()
					if object:IsA("Decal") or object:IsA("Texture") then object.Transparency = 0
					elseif object:IsA("MeshPart") or object:IsA("SpecialMesh") then object.TextureID = data.TextureID
					elseif object:IsA("BasePart") then
						object.Material = data.Material
						object.BrickColor = grayModeActive and BrickColor.new("Dark stone gray") or data.BrickColor
						object.CastShadow = true
					end
				end)
			end
			
			runMassClean()
			Workspace.Terrain.Decoration = true
			Lighting.Brightness = originalBrightness
			if not grayModeActive then Lighting.Ambient, Lighting.OutdoorAmbient = originalAmbient, originalOutdoorAmbient end
		end
	end)
end)
