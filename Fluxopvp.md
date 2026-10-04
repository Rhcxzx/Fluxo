-- Menu visual / UI somente (sem lógica de trapaça)
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local config = {
	menuSize = "Medium",
	combat = {
		aimSilent = false,
		aimTarget = "Head",
		aimbot = false,
		aimKill = false,
		fov = 90,
		ignoreDead = true,
		ignoreBehindWall = true,
		ignoreAllies = true,
	},
	movement = {
		speed = 32,
		spinbot = false,
		infinityJump = false,
	},
	visuals = {
		enable = true,
		boxColor = Color3.fromRGB(255, 0, 0),
		line = true,
		name = true,
		distance = true,
		healthBar = true,
		showFov = true,
		showFovColor = true,
		fovColor = Color3.fromRGB(255, 128, 0),
	},
	misc = {
		hidePanelGesture = true,
		streamMode = false,
	},
}

local font = Enum.Font.GothamBold

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DebugMenuUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local root = Instance.new("Frame")
root.Name = "Root"
root.Size = UDim2.new(0, 430, 0, 520)
root.Position = UDim2.new(0.5, -215, 0.5, -260)
root.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
root.BorderSizePixel = 0
root.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 14)
corner.Parent = root

local shadow = Instance.new("Frame")
shadow.Name = "Shadow"
shadow.Size = UDim2.new(1, 22, 1, 22)
shadow.Position = UDim2.new(0, -11, 0, -11)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BorderSizePixel = 0
shadow.BackgroundTransparency = 0.7
shadow.ZIndex = -1
shadow.Parent = root

local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0, 18)
shadowCorner.Parent = shadow

local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, -20, 0, 42)
title.Position = UDim2.new(0, 10, 0, 8)
title.BackgroundTransparency = 1
title.Text = "Menu"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = font
title.TextSize = 26
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = root

local closeBtn = Instance.new("TextButton")
closeBtn.Name = "Close"
closeBtn.Size = UDim2.new(0, 34, 0, 34)
closeBtn.Position = UDim2.new(1, -44, 0, 10)
closeBtn.BackgroundColor3 = Color3.fromRGB(255, 92, 92)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = font
closeBtn.TextSize = 18
closeBtn.BorderSizePixel = 0
closeBtn.Parent = root

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 10)
closeCorner.Parent = closeBtn

local tabs = {
	"⚔️ Combat",
	"🏃 Movement",
	"👁️ Visuals",
	"👤 Misc",
	"⚙️ Settings"
}

local tabHolder = Instance.new("Frame")
tabHolder.Name = "Tabs"
tabHolder.Size = UDim2.new(1, -20, 0, 38)
tabHolder.Position = UDim2.new(0, 10, 0, 52)
tabHolder.BackgroundColor3 = Color3.fromRGB(33, 33, 33)
tabHolder.BorderSizePixel = 0
tabHolder.Parent = root

local tabCorner = Instance.new("UICorner")
tabCorner.CornerRadius = UDim.new(0, 10)
tabCorner.Parent = tabHolder

local tabButtons = {}
local tabPages = {}
local activeTab = "⚔️ Combat"

local function createTabButton(label)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1 / #tabs, -4, 1, -6)
	btn.Position = UDim2.new((#tabButtons * (1 / #tabs)), 2, 0, 3)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.Text = label
	btn.TextColor3 = Color3.fromRGB(220, 220, 220)
	btn.Font = Enum.Font.GothamMedium
	btn.TextSize = 11
	btn.BorderSizePixel = 0
	btn.Parent = tabHolder

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = btn

	table.insert(tabButtons, btn)

	return btn
end

for _, label in ipairs(tabs) do
	local btn = createTabButton(label)
	btn.MouseButton1Click:Connect(function()
		activeTab = label
		for _, b in ipairs(tabButtons) do
			b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
			b.TextColor3 = Color3.fromRGB(220, 220, 220)
		end
		btn.BackgroundColor3 = Color3.fromRGB(66, 133, 244)
		btn.TextColor3 = Color3.fromRGB(255, 255, 255)
		for name, page in pairs(tabPages) do
			page.Visible = (name == label)
		end
	end)
end

local content = Instance.new("Frame")
content.Name = "Content"
content.Size = UDim2.new(1, -20, 1, -100)
content.Position = UDim2.new(0, 10, 0, 95)
content.BackgroundColor3 = Color3.fromRGB(26, 26, 26)
content.BorderSizePixel = 0
content.Parent = root

local contentCorner = Instance.new("UICorner")
contentCorner.CornerRadius = UDim.new(0, 12)
contentCorner.Parent = content

local pageLayout = Instance.new("UIListLayout")
pageLayout.Padding = UDim.new(0, 10)
pageLayout.FillDirection = Enum.FillDirection.Vertical
pageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
pageLayout.VerticalAlignment = Enum.VerticalAlignment.Top
pageLayout.Parent = content

local function addSection(parent, titleText)
	local sec = Instance.new("Frame")
	sec.Name = "Section"
	sec.Size = UDim2.new(1, -12, 0, 0)
	sec.AutomaticSize = Enum.AutomaticSize.Y
	sec.BackgroundColor3 = Color3.fromRGB(38, 38, 38)
	sec.BorderSizePixel = 0
	sec.Parent = parent

	local secCorner = Instance.new("UICorner")
	secCorner.CornerRadius = UDim.new(0, 10)
	secCorner.Parent = sec

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -14, 0, 24)
	label.Position = UDim2.new(0, 7, 0, 6)
	label.BackgroundTransparency = 1
	label.Text = titleText
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.Font = Enum.Font.GothamBold
	label.TextSize = 13
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = sec

	return sec
end

local function createToggle(parent, labelText, valueRef, onToggle)
	local row = Instance.new("Frame")
	row.Size = UDim2.new(1, -18, 0, 30)
	row.BackgroundTransparency = 1
	row.Parent = parent

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, -70, 1, 0)
	text.BackgroundTransparency = 1
	text.Text = labelText
	text.TextColor3 = Color3.fromRGB(232, 232, 232)
	text.Font = Enum.Font.GothamMedium
	text.TextSize = 12
	text.TextXAlignment = Enum.TextXAlignment.Left
	text.Parent = row

	local toggleBg = Instance.new("Frame")
	toggleBg.Size = UDim2.new(0, 44, 0, 20)
	toggleBg.Position = UDim2.new(1, -44, 0.5, -10)
	toggleBg.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
	toggleBg.BorderSizePixel = 0
	toggleBg.Parent = row

	local toggleCorner = Instance.new("UICorner")
	toggleCorner.CornerRadius = UDim.new(0, 10)
	toggleCorner.Parent = toggleBg

	local knob = Instance.new("Frame")
	knob.Size = UDim2.new(0, 16, 0, 16)
	knob.Position = UDim2.new(0, 2, 0.5, -8)
	knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	knob.BorderSizePixel = 0
	knob.Parent = toggleBg

	local knobCorner = Instance.new("UICorner")
	knobCorner.CornerRadius = UDim.new(1, 0)
	knobCorner.Parent = knob

	local function sync()
		if valueRef then
			if valueRef == true then
				toggleBg.BackgroundColor3 = Color3.fromRGB(82, 180, 92)
				knob.Position = UDim2.new(1, -18, 0.5, -8)
			else
				toggleBg.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
				knob.Position = UDim2.new(0, 2, 0.5, -8)
			end
		end
	end

	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 1, 0)
	btn.BackgroundTransparency = 1
	btn.Text = ""
	btn.Parent = row
	btn.MouseButton1Click:Connect(function()
		if type(valueRef) == "boolean" then
			valueRef = not valueRef
			if onToggle then
				onToggle(valueRef)
			end
			sync()
		end
	end)

	sync()
	return row
end

local function createSlider(parent, labelText, minValue, maxValue, currentValue, onChange)
	local row = Instance.new("Frame")
	row.Size = UDim2.new(1, -18, 0, 42)
	row.BackgroundTransparency = 1
	row.Parent = parent

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, -110, 0, 18)
	text.Position = UDim2.new(0, 0, 0, 0)
	text.BackgroundTransparency = 1
	text.Text = labelText
	text.TextColor3 = Color3.fromRGB(232, 232, 232)
	text.Font = Enum.Font.GothamMedium
	text.TextSize = 12
	text.TextXAlignment = Enum.TextXAlignment.Left
	text.Parent = row

	local valueLabel = Instance.new("TextLabel")
	valueLabel.Size = UDim2.new(0, 52, 0, 18)
	valueLabel.Position = UDim2.new(1, -52, 0, 0)
	valueLabel.BackgroundTransparency = 1
	valueLabel.Text = tostring(currentValue)
	valueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	valueLabel.Font = Enum.Font.GothamBold
	valueLabel.TextSize = 12
	valueLabel.TextXAlignment = Enum.TextXAlignment.Right
	valueLabel.Parent = row

	local barBg = Instance.new("Frame")
	barBg.Size = UDim2.new(1, 0, 0, 10)
	barBg.Position = UDim2.new(0, 0, 0, 22)
	barBg.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	barBg.BorderSizePixel = 0
	barBg.Parent = row

	local barCorner = Instance.new("UICorner")
	barCorner.CornerRadius = UDim.new(0, 6)
	barCorner.Parent = barBg

	local fill = Instance.new("Frame")
	fill.Size = UDim2.new(0, 0, 1, 0)
	fill.BackgroundColor3 = Color3.fromRGB(82, 180, 92)
	fill.BorderSizePixel = 0
	fill.Parent = barBg

	local fillCorner = Instance.new("UICorner")
	fillCorner.CornerRadius = UDim.new(0, 6)
	fillCorner.Parent = fill

	local knob = Instance.new("Frame")
	knob.Size = UDim2.new(0, 12, 0, 12)
	knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	knob.BorderSizePixel = 0
	knob.Parent = barBg

	local knobCorner = Instance.new("UICorner")
	knobCorner.CornerRadius = UDim.new(1, 0)
	knobCorner.Parent = knob
	
	local function updateVisual()
		local pct = (currentValue - minValue) / (maxValue - minValue)
		fill.Size = UDim2.new(pct, 0, 1, 0)
		knob.Position = UDim2.new(pct, -6, 0.5, -6)
		valueLabel.Text = tostring(math.floor(currentValue + 0.5))
	end

	updateVisual()

	local drag = false
	knob.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			drag = true
		end
	end)

	knob.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			drag = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if drag and input.UserInputType == Enum.UserInputType.MouseMovement then
			local mouseX = input.Position.X
			local absPos = barBg.AbsolutePosition.X
			local sizeX = barBg.AbsoluteSize.X
			local pct = math.clamp((mouseX - absPos) / sizeX, 0, 1)
			currentValue = minValue + (maxValue - minValue) * pct
			if onChange then
				onChange(currentValue)
			end
			updateVisual()
		end
	end)

	return row
end

local function createDropdown(parent, labelText, options, currentIndex, onChange)
	local row = Instance.new("Frame")
	row.Size = UDim2.new(1, -18, 0, 34)
	row.BackgroundTransparency = 1
	row.Parent = parent

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, -120, 1, 0)
	text.BackgroundTransparency = 1
	text.Text = labelText
	text.TextColor3 = Color3.fromRGB(232, 232, 232)
	text.Font = Enum.Font.GothamMedium
	text.TextSize = 12
	text.TextXAlignment = Enum.TextXAlignment.Left
	text.Parent = row

	local button = Instance.new("TextButton")
	button.Size = UDim2.new(0, 90, 0, 24)
	button.Position = UDim2.new(1, -90, 0.5, -12)
	button.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
	button.Text = options[currentIndex] or options[1]
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.GothamMedium
	button.TextSize = 11
	button.BorderSizePixel = 0
	button.Parent = row

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = button

	local idx = currentIndex or 1
	button.MouseButton1Click:Connect(function()
		idx = idx % #options + 1
		button.Text = options[idx]
		if onChange then
			onChange(idx, options[idx])
		end
	end)

	return row
end

local function makePage(name)
	local page = Instance.new("ScrollingFrame")
	page.Name = name
	page.Size = UDim2.new(1, 0, 1, 0)
	page.BackgroundTransparency = 1
	page.BorderSizePixel = 0
	page.CanvasSize = UDim2.new(0, 0, 0, 900)
	page.ScrollBarThickness = 6
	page.Visible = (name == "⚔️ Combat")
	page.Parent = content

	local layout = Instance.new("UIListLayout")
	layout.Padding = UDim.new(0, 10)
	layout.FillDirection = Enum.FillDirection.Vertical
	layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	layout.VerticalAlignment = Enum.VerticalAlignment.Top
	layout.Parent = page

	return page
end

tabPages["⚔️ Combat"] = makePage("⚔️ Combat")
tabPages["🏃 Movement"] = makePage("🏃 Movement")
tabPages["👁️ Visuals"] = makePage("👁️ Visuals")
tabPages["👤 Misc"] = makePage("👤 Misc")
tabPages["⚙️ Settings"] = makePage("⚙️ Settings")

local function fillCombat()
	local sec = addSection(tabPages["⚔️ Combat"], "Combat")
	local rowA = createToggle(sec, "Aim Silent", config.combat.aimSilent, function(v) config.combat.aimSilent = v end)
	local rowB = createToggle(sec, "Aimbot", config.combat.aimbot, function(v) config.combat.aimbot = v end)
	local rowC = createToggle(sec, "Aim Kill", config.combat.aimKill, function(v) config.combat.aimKill = v end)
	local rowD = createSlider(sec, "FOV", 0, 360, config.combat.fov, function(v) config.combat.fov = v end)
	local rowE = createDropdown(sec, "Target", {"Head", "Neck", "Torso"}, 1, function(idx, value) config.combat.aimTarget = value end)
	local rowF = createToggle(sec, "Mortos não", config.combat.ignoreDead, function(v) config.combat.ignoreDead = v end)
	local rowG = createToggle(sec, "Atrás da parede não", config.combat.ignoreBehindWall, function(v) config.combat.ignoreBehindWall = v end)
	local rowH = createToggle(sec, "Não colar em aliados", config.combat.ignoreAllies, function(v) config.combat.ignoreAllies = v end)
	sec.Size = UDim2.new(1, -10, 0, 0)
	sec.AutomaticSize = Enum.AutomaticSize.Y
end

local function fillMovement()
	local sec = addSection(tabPages["🏃 Movement"], "Movement")
	createToggle(sec, "Speed", false, function() end)
	createSlider(sec, "Speed", 32, 100, config.movement.speed, function(v) config.movement.speed = v end)
	createToggle(sec, "Spinbot", config.movement.spinbot, function(v) config.movement.spinbot = v end)
	createToggle(sec, "Infinity Jump", config.movement.infinityJump, function(v) config.movement.infinityJump = v end)
	sec.Size = UDim2.new(1, -10, 0, 0)
	sec.AutomaticSize = Enum.AutomaticSize.Y
end

local function fillVisuals()
	local sec = addSection(tabPages["👁️ Visuals"], "Visuals")
	createToggle(sec, "Ativar", config.visuals.enable, function(v) config.visuals.enable = v end)
	createToggle(sec, "Box", true, function() end)
	createToggle(sec, "Line", config.visuals.line, function(v) config.visuals.line = v end)
	createToggle(sec, "Nome", config.visuals.name, function(v) config.visuals.name = v end)
	createToggle(sec, "Distance", config.visuals.distance, function(v) config.visuals.distance = v end)
	createToggle(sec, "Vida", config.visuals.healthBar, function(v) config.visuals.healthBar = v end)
	createToggle(sec, "Mostrar FOV", config.visuals.showFov, function(v) config.visuals.showFov = v end)
	createToggle(sec, "Mostrar cor do FOV", config.visuals.showFovColor, function(v) config.visuals.showFovColor = v end)
	sec.Size = UDim2.new(1, -10, 0, 0)
	sec.AutomaticSize = Enum.AutomaticSize.Y
end

local function fillMisc()
	local sec = addSection(tabPages["👤 Misc"], "Misc")
	createToggle(sec, "Ocultar painel", config.misc.hidePanelGesture, function(v) config.misc.hidePanelGesture = v end)
	createToggle(sec, "Stream Mode", config.misc.streamMode, function(v) config.misc.streamMode = v end)
	sec.Size = UDim2.new(1, -10, 0, 0)
	sec.AutomaticSize = Enum.AutomaticSize.Y
end

local function fillSettings()
	local sec = addSection(tabPages["⚙️ Settings"], "Settings")
	createDropdown(sec, "Tamanho do menu", {"Pequeno", "Médio", "Grande"}, 2, function(idx, value)
		config.menuSize = value
	end)
	sec.Size = UDim2.new(1, -10, 0, 0)
	sec.AutomaticSize = Enum.AutomaticSize.Y
end

fillCombat()
fillMovement()
fillVisuals()
fillMisc()
fillSettings()

for _, btn in ipairs(tabButtons) do
	btn.MouseButton1Click:Connect(function()
		for _, b in ipairs(tabButtons) do
			b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
			b.TextColor3 = Color3.fromRGB(220, 220, 220)
		end
		btn.BackgroundColor3 = Color3.fromRGB(66, 133, 244)
		btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	end)
end

tabButtons[1].BackgroundColor3 = Color3.fromRGB(66, 133, 244)
tabButtons[1].TextColor3 = Color3.fromRGB(255, 255, 255)

closeBtn.MouseButton1Click:Connect(function()
	screenGui.Enabled = false
end)

local hidden = false
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.RightShift then
		hidden = not hidden
		root.Visible = not hidden
	end
end)
