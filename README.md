
--==================================================
-- MM2 INFERNO V4 | ULTIMATE EDITION
-- Para tu propia experiencia de Roblox Studio
-- StarterPlayer > StarterPlayerScripts > LocalScript
--==================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("MM2_INFERNO_V4")
if old then old:Destroy() end

local RED = Color3.fromRGB(255, 30, 50)
local BLUE = Color3.fromRGB(30, 145, 255)
local GREEN = Color3.fromRGB(40, 230, 120)
local PURPLE = Color3.fromRGB(175, 70, 255)
local BG = Color3.fromRGB(9, 9, 15)
local CARD = Color3.fromRGB(22, 22, 32)
local WHITE = Color3.fromRGB(245, 245, 255)
local GRAY = Color3.fromRGB(160, 160, 180)

local accent = RED
local espEnabled = false
local showBoxes = true
local showNames = true
local showDistance = true
local highlightNearest = true
local espRange = 800
local speedEnabled = false
local jumpEnabled = false
local speedValue = 24
local savedSpeed = 16

local espObjects = {}
local nearestPlayer = nil

local gui = Instance.new("ScreenGui")
gui.Name = "MM2_INFERNO_V4"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

local function create(class, props, parent)
	local obj = Instance.new(class)
	for k, v in pairs(props) do
		obj[k] = v
	end
	obj.Parent = parent
	return obj
end

local function corner(obj, radius)
	create("UICorner", {
		CornerRadius = UDim.new(0, radius or 8)
	}, obj)
end

local function outline(obj, color, thickness)
	return create("UIStroke", {
		Color = color or accent,
		Thickness = thickness or 1
	}, obj)
end

local function getHumanoid(plr)
	local char = plr.Character
	return char and char:FindFirstChildOfClass("Humanoid")
end

local function getRoot(plr)
	local char = plr.Character
	return char and char:FindFirstChild("HumanoidRootPart")
end

local function getRoleColor(plr)
	local role = plr:GetAttribute("Role")
	if not role and plr.Team then
		role = plr.Team.Name
	end

	role = string.lower(tostring(role or ""))

	if string.find(role, "murder") or string.find(role, "asesino") then
		return RED
	elseif string.find(role, "sheriff") then
		return BLUE
	end

	return GREEN
end

-- MAIN WINDOW
local main = create("Frame", {
	Name = "Main",
	Size = UDim2.fromOffset(620, 440),
	Position = UDim2.new(0.5, -310, 0.5, -220),
	BackgroundColor3 = BG,
	BorderSizePixel = 0,
	ClipsDescendants = true
}, gui)

corner(main, 12)
local mainStroke = outline(main, accent, 2)

local header = create("Frame", {
	Size = UDim2.new(1, 0, 0, 65),
	BackgroundColor3 = Color3.fromRGB(35, 7, 16),
	BorderSizePixel = 0
}, main)

create("TextLabel", {
	Position = UDim2.fromOffset(14, 4),
	Size = UDim2.new(1, -100, 0, 37),
	BackgroundTransparency = 1,
	Text = "MM2 INFERNO V4",
	TextColor3 = RED,
	Font = Enum.Font.GothamBlack,
	TextSize = 25,
	TextXAlignment = Enum.TextXAlignment.Left
}, header)

create("TextLabel", {
	Position = UDim2.fromOffset(16, 40),
	Size = UDim2.new(1, -110, 0, 16),
	BackgroundTransparency = 1,
	Text = "ULTIMATE EDITION  •  INFERNO SYSTEM",
	TextColor3 = GRAY,
	Font = Enum.Font.Gotham,
	TextSize = 10,
	TextXAlignment = Enum.TextXAlignment.Left
}, header)

local function headerButton(txt, x)
	local b = create("TextButton", {
		Position = UDim2.new(1, x, 0, 17),
		Size = UDim2.fromOffset(30, 30),
		BackgroundColor3 = CARD,
		Text = txt,
		TextColor3 = accent,
		Font = Enum.Font.GothamBold,
		TextSize = 18,
		BorderSizePixel = 0
	}, header)
	corner(b, 7)
	outline(b)
	return b
end

local close = headerButton("×", -36)
local minimize = headerButton("—", -72)

local nav = create("Frame", {
	Position = UDim2.fromOffset(10, 76),
	Size = UDim2.fromOffset(145, 350),
	BackgroundTransparency = 1
}, main)

local content = create("Frame", {
	Position = UDim2.fromOffset(165, 76),
	Size = UDim2.new(1, -175, 1, -86),
	BackgroundColor3 = CARD,
	BorderSizePixel = 0
}, main)

corner(content, 9)
outline(content, Color3.fromRGB(65, 65, 80))

local pages = {}

local function newPage(name)
	local page = create("ScrollingFrame", {
		Name = name,
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		ScrollBarThickness = 3,
		ScrollBarImageColor3 = accent,
		CanvasSize = UDim2.new(),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		Visible = false
	}, content)

	create("UIPadding", {
		PaddingTop = UDim.new(0, 12),
		PaddingBottom = UDim.new(0, 12),
		PaddingLeft = UDim.new(0, 12),
		PaddingRight = UDim.new(0, 12)
	}, page)

	create("UIListLayout", {
		Padding = UDim.new(0, 9),
		SortOrder = Enum.SortOrder.LayoutOrder
	}, page)

	pages[name] = page
	return page
end

local home = newPage("Inicio")
local espPage = newPage("ESP")
local movement = newPage("Movimiento")
local rolesPage = newPage("Roles")
local settings = newPage("Ajustes")

local function label(parent, txt, height, size)
	return create("TextLabel", {
		Size = UDim2.new(1, 0, 0, height or 32),
		BackgroundTransparency = 1,
		Text = txt,
		TextColor3 = WHITE,
		Font = Enum.Font.GothamBold,
		TextSize = size or 15,
		TextWrapped = true,
		TextXAlignment = Enum.TextXAlignment.Left
	}, parent)
end

local function card(parent, title, description)
	local f = create("Frame", {
		Size = UDim2.new(1, 0, 0, description and 75 or 48),
		BackgroundColor3 = BG,
		BorderSizePixel = 0
	}, parent)

	corner(f, 8)
	outline(f, Color3.fromRGB(65, 65, 80))

	create("TextLabel", {
		Position = UDim2.fromOffset(10, 4),
		Size = UDim2.new(1, -20, 0, 26),
		BackgroundTransparency = 1,
		Text = title,
		TextColor3 = accent,
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		TextXAlignment = Enum.TextXAlignment.Left
	}, f)

	if description then
		create("TextLabel", {
			Position = UDim2.fromOffset(10, 31),
			Size = UDim2.new(1, -20, 0, 38),
			BackgroundTransparency = 1,
			Text = description,
			TextColor3 = GRAY,
			Font = Enum.Font.Gotham,
			TextSize = 11,
			TextWrapped = true,
			TextXAlignment = Enum.TextXAlignment.Left
		}, f)
	end

	return f
end

local function action(parent, title, callback)
	local b = create("TextButton", {
		Size = UDim2.new(1, 0, 0, 38),
		BackgroundColor3 = Color3.fromRGB(35, 35, 48),
		Text = title,
		TextColor3 = WHITE,
		Font = Enum.Font.GothamBold,
		TextSize = 12,
		BorderSizePixel = 0
	}, parent)
	corner(b, 7)
	outline(b)
	b.Activated:Connect(callback)
	return b
end

local function toggle(parent, title, description, callback)
	local f = card(parent, title, description)
	local on = false

	local b = create("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, -9, 0, description and 19 or 24),
		Size = UDim2.fromOffset(52, 25),
		BackgroundColor3 = Color3.fromRGB(55, 55, 68),
		Text = "OFF",
		TextColor3 = WHITE,
		Font = Enum.Font.GothamBold,
		TextSize = 10,
		BorderSizePixel = 0
	}, f)

	corner(b, 14)

	b.Activated:Connect(function()
		on = not on
		b.Text = on and "ON" or "OFF"
		TweenService:Create(b, TweenInfo.new(0.15), {
			BackgroundColor3 = on and accent or Color3.fromRGB(55, 55, 68)
		}):Play()
		if callback then callback(on) end
	end)

	return b
end

-- HOME
label(home, "WELCOME TO", 25, 16)
label(home, "🔥 MM2 INFERNO V4", 36, 23)
card(home, "ULTIMATE EDITION", "Nuevo sistema de interfaz, ESP y movimiento.")
card(home, "DISPOSITIVO", "Diseñado para PC y móviles.")
card(home, "ESTADO", "INFERNO V4 iniciado correctamente.")

-- ESP
label(espPage, "PLAYER ESP", 28, 18)

toggle(espPage, "ESP GLOBAL", "Mostrar información visual de jugadores.", function(on)
	espEnabled = on
end)

toggle(espPage, "NOMBRES", "Mostrar nombre sobre cada jugador.", function(on)
	showNames = on
end)

toggle(espPage, "CAJAS", "Mostrar resaltado alrededor del personaje.", function(on)
	showBoxes = on
end)

toggle(espPage, "DISTANCIA", "Mostrar distancia en studs.", function(on)
	showDistance = on
end)

toggle(espPage, "JUGADOR MÁS CERCANO", "Resaltar al jugador más próximo.", function(on)
	highlightNearest = on
end)

local rangeCard = card(espPage, "ALCANCE DEL ESP", "Usa los botones para cambiar el alcance.")
local rangeLabel = create("TextLabel", {
	Position = UDim2.new(1, -100, 0, 4),
	Size = UDim2.fromOffset(90, 25),
	BackgroundTransparency = 1,
	Text = "800 studs",
	TextColor3 = accent,
	Font = Enum.Font.GothamBold,
	TextSize = 12
}, rangeCard)

local rangeButtons = create("Frame", {
	Position = UDim2.fromOffset(8, 36),
	Size = UDim2.new(1, -16, 0, 30),
	BackgroundTransparency = 1
}, rangeCard)

local rangeLayout = create("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal,
	Padding = UDim.new(0, 7),
	HorizontalAlignment = Enum.HorizontalAlignment.Left
}, rangeButtons)

for _, amount in ipairs({100, 250, 500, 800}) do
	local b = create("TextButton", {
		Size = UDim2.fromOffset(75, 27),
		BackgroundColor3 = CARD,
		Text = tostring(amount),
		TextColor3 = WHITE,
		Font = Enum.Font.GothamBold,
		TextSize = 11,
		BorderSizePixel = 0
	}, rangeButtons)
	corner(b, 6)
	b.Activated:Connect(function()
		espRange = amount
		rangeLabel.Text = amount .. " studs"
	end)
end

-- MOVIMIENTO
label(movement, "MOVIMIENTO", 28, 18)

local speedCard = card(movement, "SPEED", "Velocidad de movimiento local para pruebas.")
local speedButton = create("TextButton", {
	AnchorPoint = Vector2.new(1, 0.5),
	Position = UDim2.new(1, -9, 0.5, 0),
	Size = UDim2.fromOffset(85, 28),
	BackgroundColor3 = accent,
	Text = "ACTIVAR",
	TextColor3 = WHITE,
	Font = Enum.Font.GothamBold,
	TextSize = 11,
	BorderSizePixel = 0
}, speedCard)
corner(speedButton, 7)

speedButton.Activated:Connect(function()
	local hum = getHumanoid(player)
	if not hum then return end

	speedEnabled = not speedEnabled

	if speedEnabled then
		savedSpeed = hum.WalkSpeed
		hum.WalkSpeed = speedValue
		speedButton.Text = "DESACTIVAR"
	else
		hum.WalkSpeed = savedSpeed
		speedButton.Text = "ACTIVAR"
	end
end)

toggle(movement, "INFINITY JUMP", "Permitir saltar repetidamente en el aire.", function(on)
	jumpEnabled = on
end)

UIS.JumpRequest:Connect(function()
	if not jumpEnabled then return end
	local hum = getHumanoid(player)
	if hum then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

-- ROLES
label(rolesPage, "ROLE DISPLAY", 28, 18)
card(rolesPage, "🔴 MURDERER", "Color de identificación rojo.")
card(rolesPage, "🔵 SHERIFF", "Color de identificación azul.")
card(rolesPage, "🟢 INNOCENT", "Color de identificación verde.")
card(rolesPage, "SISTEMA DE ROLES", "Lee el atributo Role o el nombre del Team de tu propia experiencia.")

-- SETTINGS
label(settings, "PERSONALIZACIÓN", 28, 18)

local function setTheme(color)
	accent = color
	mainStroke.Color = color
end

action(settings, "Tema rojo", function()
	setTheme(RED)
end)

action(settings, "Tema azul", function()
	setTheme(BLUE)
end)

action(settings, "Tema violeta", function()
	setTheme(PURPLE)
end)

action(settings, "Tema verde", function()
	setTheme(GREEN)
end)

local transparency = 0
action(settings, "Cambiar transparencia", function()
	transparency = transparency >= 0.3 and 0 or transparency + 0.1
	main.BackgroundTransparency = transparency
end)

action(settings, "Tamaño compacto / normal", function()
	local compact = main:GetAttribute("Compact") or false
	compact = not compact
	main:SetAttribute("Compact", compact)
	main.Size = compact and UDim2.fromOffset(500, 360)
		or UDim2.fromOffset(620, 440)
end)

-- NAVIGATION
local navButtons = {}

local function showPage(name)
	for pageName, page in pairs(pages) do
		page.Visible = pageName == name
	end
	for pageName, button in pairs(navButtons) do
		button.BackgroundColor3 = pageName == name and accent or CARD
	end
end

local entries = {
	{"⌂  Inicio", "Inicio"},
	{"👁  ESP", "ESP"},
	{"⚡  Movimiento", "Movimiento"},
	{"☠  Roles", "Roles"},
	{"⚙  Ajustes", "Ajustes"}
}

for i, entry in ipairs(entries) do
	local b = create("TextButton", {
		Position = UDim2.fromOffset(0, (i - 1) * 45),
		Size = UDim2.new(1, 0, 0, 38),
		BackgroundColor3 = CARD,
		Text = entry[1],
		TextColor3 = WHITE,
		Font = Enum.Font.GothamBold,
		TextSize = 12,
		BorderSizePixel = 0
	}, nav)
	corner(b, 7)
	outline(b)
	navButtons[entry[2]] = b
	b.Activated:Connect(function()
		showPage(entry[2])
	end)
end

showPage("Inicio")

-- DRAG: ratón y táctil
local dragging = false
local dragStart
local startPos

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = main.Position
	end
end)

UIS.InputChanged:Connect(function(input)
	if dragging and (
		input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch
	) then
		local delta = input.Position - dragStart
		main.Position = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y
		)
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

local minimized = false
minimize.Activated:Connect(function()
	minimized = not minimized
	nav.Visible = not minimized
	content.Visible = not minimized
	main.Size = minimized and UDim2.fromOffset(300, 65)
		or UDim2.fromOffset(620, 440)
end)

close.Activated:Connect(function()
	gui:Destroy()
end)

-- ESP UPDATE
local function clearESP(plr)
	local data = espObjects[plr]
	if data then
		if data.highlight then data.highlight:Destroy() end
		if data.billboard then data.billboard:Destroy() end
		espObjects[plr] = nil
	end
end

local function createESP(plr)
	if plr == player or espObjects[plr] then return end

	local char = plr.Character
	if not char then return end

	local hum = getHumanoid(plr)
	local root = getRoot(plr)
	if not hum or not root then return end

	local highlight = Instance.new("Highlight")
	highlight.Name = "InfernoESP"
	highlight.Adornee = char
	highlight.FillTransparency = 0.85
	highlight.OutlineTransparency = 0
	highlight.Parent = char

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "InfernoInfo"
	billboard.Adornee = root
	billboard.Size = UDim2.fromOffset(180, 48)
	billboard.StudsOffset = Vector3.new(0, 3, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = playerGui

	local text = Instance.new("TextLabel")
	text.Size = UDim2.fromScale(1, 1)
	text.BackgroundTransparency = 1
	text.TextStrokeTransparency = 0.2
	text.Font = Enum.Font.GothamBold
	text.TextSize = 12
	text.TextColor3 = WHITE
	text.TextWrapped = true
	text.Parent = billboard

	espObjects[plr] = {
		highlight = highlight,
		billboard = billboard,
		text = text
	}
end

local function updateESP()
	local myRoot = getRoot(player)
	if not myRoot then return end

	local closestDistance = math.huge
	nearestPlayer = nil

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= player then
			local root = getRoot(plr)
			local hum = getHumanoid(plr)

			if root and hum and hum.Health > 0 then
				local distance = (root.Position - myRoot.Position).Magnitude

				if distance <= espRange and distance < closestDistance then
					closestDistance = distance
					nearestPlayer = plr
				end
			end
		end
	end

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= player then
			if espEnabled then
				local root = getRoot(plr)
				local hum = getHumanoid(plr)

				if root and hum and hum.Health > 0 then
					local distance = (root.Position - myRoot.Position).Magnitude

					if distance <= espRange then
						createESP(plr)

						local data = espObjects[plr]
						if data then
							local roleColor = getRoleColor(plr)
							local isNearest = plr == nearestPlayer

							data.highlight.Enabled = showBoxes
							data.highlight.OutlineColor =
								highlightNearest and isNearest and WHITE or roleColor
							data.highlight.FillColor = roleColor

							data.billboard.Enabled = showNames or showDistance

							local nameText = showNames and plr.DisplayName or ""
							local distanceText = showDistance
								and string.format("  [%.0f studs]", distance) or ""

							data.text.Text = nameText .. distanceText
							data.text.TextColor3 = roleColor
							data.text.Visible = showNames or showDistance
						end
					else
						clearESP(plr)
					end
				else
					clearESP(plr)
				end
			else
				clearESP(plr)
			end
		end
	end
end

Players.PlayerRemoving:Connect(clearESP)

Players.PlayerAdded:Connect(function(plr)
	plr.CharacterRemoving:Connect(function()
		clearESP(plr)
	end)
end)

RunService.Heartbeat:Connect(updateESP)

print("🔥 MM2 INFERNO V4 cargado correctamente")
