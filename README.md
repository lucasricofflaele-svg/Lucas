-- L Hub Premium (LocalScript) - Structural Game Lock
-- Cole no Delta.

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local VirtualUser = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- ============================================================
-- VERIFICAÇÃO DE JOGO (GAME LOCK POR ESTRUTURA)
-- ============================================================
print("L Hub carregado no Game ID: " .. tostring(game.PlaceId))

local isCorrectGame = false
pcall(function()
    -- Se o jogo tiver essas 3 pastas exatas, é o jogo certo.
    if ReplicatedStorage:FindFirstChild("Remotes") and 
       ReplicatedStorage:FindFirstChild("Config") and 
       ReplicatedStorage:FindFirstChild("Shared") and
       ReplicatedStorage:FindFirstChild("AnimalsData") then
        isCorrectGame = true
    end
end)

if not isCorrectGame then
    -- Notificação Gigante de Erro
    local gui = Instance.new("ScreenGui")
    gui.Name = "UnsupportedGame"
    gui.DisplayOrder = 2147483647
    pcall(function() gui.Parent = gethui() end)
    if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 600, 0, 150)
    frame.Position = UDim2.new(0.5, -300, 0.5, -75)
    frame.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    frame.BorderSizePixel = 0
    frame.Parent = gui
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 20)
    
    local stroke = Instance.new("UIStroke", frame)
    stroke.Color = Color3.fromRGB(255, 50, 50)
    stroke.Thickness = 6
    stroke.Parent = frame
    
    local text = Instance.new("TextLabel")
    text.Size = UDim2.new(1, 0, 1, 0)
    text.BackgroundTransparency = 1
    text.Text = "NO SUPPORT FOR THIS GAME"
    text.TextColor3 = Color3.fromRGB(255, 255, 255)
    text.Font = Enum.Font.GothamBlack
    text.TextSize = 40
    text.Parent = frame
    
    local scale = Instance.new("UIScale", frame)
    scale.Scale = 0
    
    -- Animação de aparecer
    TweenService:Create(scale, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
    
    -- Animação de sumir e destruir
    task.delay(3, function()
        TweenService:Create(scale, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Scale = 0}):Play()
        task.delay(0.5, function()
            gui:Destroy()
        end)
    end)
    
    return -- Mata o script aqui, não executa mais nada
end

-- ============================================================
-- KILL SWITCH DEFINITIVO
-- ============================================================
local SCRIPT_ID = tostring(os.clock()) .. "_" .. tostring(math.random(100000, 999999))
getgenv().LHub_Active_ID = SCRIPT_ID

local function isAlive()
    return getgenv().LHub_Active_ID == SCRIPT_ID
end

local function destroyOldInterfaces()
    local core = game:GetService("CoreGui")
    local pg = player.PlayerGui
    for _, parent in ipairs({core, pg}) do
        for _, obj in ipairs(parent:GetDescendants()) do
            if obj.Name == "LHub_Premium" or obj.Name == "LHubNotifications" or obj.Name == "LHubIntro" then
                pcall(function() obj:Destroy() end)
            end
        end
    end
end
destroyOldInterfaces()

-- ============================================================
-- ESTADO GLOBAL E CONFIGURAÇÕES
-- ============================================================
local isPerfectActive = false
local isPopsActive = false
local isEquipBestActive = false
local isBuyActive = false
local isSellActive = false
local isAntiAfkActive = false
local isFpsBoostActive = false
local current_min_sell = 0
local PERFECT_TIME = 1.10

local food_toggles = {} 
local foods_list = {}
local last_notif_time = 0

-- Paleta de Cores Black & Red
local C_BG = Color3.fromRGB(15, 15, 15)
local C_SIDEBAR = Color3.fromRGB(20, 20, 20)
local C_PANEL = Color3.fromRGB(28, 28, 28)
local C_RED = Color3.fromRGB(220, 30, 30)
local C_DARK_RED = Color3.fromRGB(80, 0, 0)
local C_TEXT = Color3.fromRGB(255, 255, 255)
local C_TEXT_DIM = Color3.fromRGB(160, 160, 160)

-- ============================================================
-- SISTEMA DE FPS BOOST
-- ============================================================
local saved_data = { materials = {}, textures = {}, transparency = {}, particles = {}, lights = {} }
local function enable_fps_boost()
	if not isAlive() then return end
	for _, v in ipairs(workspace:GetDescendants()) do
		pcall(function()
			if v:IsA("BasePart") or v:IsA("MeshPart") then
				if not saved_data.materials[v] then
					saved_data.materials[v] = v.Material
					saved_data.textures[v] = v.TextureID
				end
				v.Material = Enum.Material.SmoothPlastic
				if v:IsA("MeshPart") then v.TextureID = "" end
			elseif v:IsA("Texture") or v:IsA("Decal") then
				if not saved_data.transparency[v] then saved_data.transparency[v] = v.Transparency end
				v.Transparency = 1
			elseif v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") or v:IsA("Beam") or v:IsA("Trail") then
				if not saved_data.particles[v] then saved_data.particles[v] = v.Enabled end
				v.Enabled = false
			elseif v:IsA("Light") then
				if not saved_data.lights[v] then saved_data.lights[v] = v.Enabled end
				v.Enabled = false
			end
		end)
	end
end
local function disable_fps_boost()
	for v, mat in pairs(saved_data.materials) do
		if v and v.Parent then pcall(function() v.Material = mat; if v:IsA("MeshPart") and saved_data.textures[v] then v.TextureID = saved_data.textures[v] end end) end
	end
	for v, trans in pairs(saved_data.transparency) do if v and v.Parent then pcall(function() v.Transparency = trans end) end end
	for v, enabled in pairs(saved_data.particles) do if v and v.Parent then pcall(function() v.Enabled = enabled end) end end
	for v, enabled in pairs(saved_data.lights) do if v and v.Parent then pcall(function() v.Enabled = enabled end) end end
	saved_data = { materials = {}, textures = {}, transparency = {}, particles = {}, lights = {} }
end

-- ============================================================
-- FORMATAÇÃO
-- ============================================================
local function parse_abbreviated_number(str)
	str = string.lower(str); str = string.gsub(str, "%s+", "")
	local num, suffix = string.match(str, "([%d%.]+)([kmbtq]?)")
	if not num then return 0 end
	local n = tonumber(num); if not n then return 0 end
	if suffix == "k" then n = n * 1e3 elseif suffix == "m" then n = n * 1e6 elseif suffix == "b" then n = n * 1e9 elseif suffix == "t" then n = n * 1e12 elseif suffix == "q" then n = n * 1e15 end
	return math.floor(n)
end

-- ============================================================
-- MÓDULOS
-- ============================================================
local ThrowMeter
pcall(function() ThrowMeter = require(player.PlayerScripts.Client.Controllers.ThrowMeter) end)

local Remotes = ReplicatedStorage:WaitForChild("Remotes", 5)
local PlotEquipBest = Remotes and Remotes:FindFirstChild("Plot_EquipBest")
local PlotSellAnimal = Remotes and Remotes:FindFirstChild("Plot_SellAnimal")
local ShopBuy = Remotes and Remotes:FindFirstChild("Shop_Buy")

local DataController, Mutations
pcall(function()
	DataController = require(player.PlayerScripts.Client.Controllers.DataController)
	Mutations = require(ReplicatedStorage.Shared.Mutations)
end)

-- ============================================================
-- LISTA HARDCODED DE TODAS AS COMIDAS
-- ============================================================
local hardcoded_foods = {
    {id = 1, name = "Carrot", price = 0}, {id = 2, name = "Apple", price = 30}, {id = 3, name = "Banana", price = 150},
    {id = 4, name = "Toast", price = 4000}, {id = 5, name = "Orange", price = 20000}, {id = 6, name = "Pear", price = 200000},
    {id = 7, name = "Cheese", price = 1000000}, {id = 8, name = "Bread Loaf", price = 12000000}, {id = 9, name = "Water Melon", price = 60000000},
    {id = 10, name = "Croissant", price = 650000000}, {id = 11, name = "Chocolate", price = 3200000000}, {id = 12, name = "Mushroom", price = 30000000000},
    {id = 13, name = "Bacon", price = 150000000000}, {id = 14, name = "Burger", price = 1300000000000}, {id = 15, name = "Shank", price = 6800000000000},
    {id = 16, name = "Fillet", price = 57000000000000}, {id = 17, name = "Steak", price = 1500000000000000}, {id = 18, name = "Royal Steak", price = 5e16},
    {id = 19, name = "Golden Apple", price = 1.2e17}, {id = 20, name = "Crystal Melon", price = 4e18}, {id = 21, name = "Ember Bacon", price = 1.8e19},
    {id = 22, name = "Lava Steak", price = 1.2e20}, {id = 23, name = "Elder Strawberry", price = 7e20}, {id = 24, name = "Phoenix Fillet", price = 4e21},
    {id = 25, name = "Star Jelly", price = 2.7e22}, {id = 26, name = "Dragon's Apple", price = 1.8e23}, {id = 27, name = "Starlight Donut", price = 4.5e23},
    {id = 28, name = "Glowshroom", price = 1.4e24}, {id = 29, name = "Sundae Tower", price = 3.5e24}, {id = 30, name = "Golden Corn", price = 2.5e27},
    {id = 31, name = "Roast Turkey", price = 2.8e28}, {id = 32, name = "Caramel Apple", price = 2.5e29}, {id = 33, name = "Candy Corn", price = 7.5e32},
    {id = 34, name = "Hot Cider", price = 8.5e33}, {id = 35, name = "Golden Pumpkin", price = 7.5e34}, {id = 36, name = "Ghost Chowder", price = 1.8e37},
    {id = 37, name = "Calamari Rings", price = 2e38}, {id = 38, name = "Phantom Lobster", price = 1.8e39}, {id = 39, name = "Toxic Donut", price = 1.2e42},
    {id = 40, name = "Mutant Melon", price = 2e43}, {id = 41, name = "Atomic Pizza", price = 4e44}, {id = 42, name = "Uranium Cake", price = 9e45},
    {id = 43, name = "Ghost Popcorn", price = 2e48}, {id = 44, name = "Cursed Candy Apple", price = 3.2e49}, {id = 45, name = "Spectral Cotton Candy", price = 6.4e50},
    {id = 46, name = "Ringmaster Cake", price = 1.4e52}
}

for _, food in ipairs(hardcoded_foods) do
    table.insert(foods_list, food)
    food_toggles[food.id] = false
end

-- ============================================================
-- NOTIFICAÇÃO EMPILHADA
-- ============================================================
local notifGui = Instance.new("ScreenGui")
notifGui.Name = "LHubNotifications"
notifGui.ResetOnSpawn = false
notifGui.DisplayOrder = 9999999
pcall(function() notifGui.Parent = gethui() end)
if not notifGui.Parent then pcall(function() notifGui.Parent = game:GetService("CoreGui") end) end
if not notifGui.Parent then notifGui.Parent = player.PlayerGui end

local notifContainer = Instance.new("Frame")
notifContainer.Size = UDim2.new(0, 220, 1, 0)
notifContainer.Position = UDim2.new(1, -230, 0, 20)
notifContainer.BackgroundTransparency = 1
notifContainer.Parent = notifGui

local notifLayout = Instance.new("UIListLayout", notifContainer)
notifLayout.Padding = UDim.new(0, 8)
notifLayout.VerticalAlignment = Enum.VerticalAlignment.Top
notifLayout.SortOrder = Enum.SortOrder.LayoutOrder

local notifOrder = 0

local function send_notification(title)
	if not isAlive() then return end
	notifOrder = notifOrder - 1
	local holder = Instance.new("Frame")
	holder.Size = UDim2.new(1, 0, 0, 0)
	holder.BackgroundTransparency = 1
	holder.LayoutOrder = notifOrder
	holder.ClipsDescendants = true
	holder.Parent = notifContainer

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(1, 0, 0, 50)
	frame.Position = UDim2.new(1, 220, 0, 0)
	frame.BackgroundColor3 = C_PANEL
	frame.BorderSizePixel = 0
	frame.Parent = holder
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
	local stroke = Instance.new("UIStroke", frame)
	stroke.Color = C_RED

	local icon = Instance.new("TextLabel")
	icon.Size = UDim2.new(0, 30, 0, 30)
	icon.Position = UDim2.new(0, 10, 0.5, -15)
	icon.BackgroundColor3 = C_RED
	icon.Text = "!"
	icon.TextColor3 = Color3.new(1, 1, 1)
	icon.Font = Enum.Font.GothamBlack
	icon.TextSize = 20
	icon.Parent = frame
	Instance.new("UICorner", icon).CornerRadius = UDim.new(0, 6)

	local titleLabel = Instance.new("TextLabel")
	titleLabel.Size = UDim2.new(1, -50, 1, 0)
	titleLabel.Position = UDim2.new(0, 50, 0, 0)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = title
	titleLabel.TextColor3 = C_RED
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.TextSize = 13
	titleLabel.TextXAlignment = Enum.TextXAlignment.Left
	titleLabel.TextWrapped = true
	titleLabel.Parent = frame

	TweenService:Create(holder, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, 50) }):Play()
	TweenService:Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Position = UDim2.new(0, 0, 0, 0) }):Play()

	task.delay(3, function()
		if not holder.Parent then return end
		local tweenOut = TweenService:Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.In), { Position = UDim2.new(1, 220, 0, 0), BackgroundTransparency = 1 })
		TweenService:Create(stroke, TweenInfo.new(0.4), {Transparency = 1}):Play()
		TweenService:Create(icon, TweenInfo.new(0.4), {BackgroundTransparency = 1, TextTransparency = 1}):Play()
		TweenService:Create(titleLabel, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
		TweenService:Create(holder, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
		tweenOut:Play()
		tweenOut.Completed:Connect(function() holder:Destroy() end)
	end)
end

-- ============================================================
-- LOOPS DE FUNÇÃO
-- ============================================================
task.spawn(function() while isAlive() do task.wait(0.5); if isPerfectActive and ThrowMeter then ThrowMeter.TestAutoTap = PERFECT_TIME elseif not isPerfectActive and ThrowMeter then ThrowMeter.TestAutoTap = nil end end end)

task.spawn(function()
	while isAlive() do
		task.wait(0.5) 
		if isPopsActive then
			local trainingGui = player.PlayerGui:FindFirstChild("TrainingSequence")
			if trainingGui then
				local layer = trainingGui:FindFirstChild("TapBonusLayer")
				if layer then
					local botaoX2 = layer:FindFirstChild("TapBonus")
					if botaoX2 and botaoX2.Visible then
						local fs = getfenv and getfenv().firesignal
						if fs then 
							pcall(function() fs(botaoX2.Activated) end)
							pcall(function() fs(botaoX2.MouseButton1Click) end)
						end
						task.wait(0.5) 
					end
				end
			end
		end
	end
end)

task.spawn(function() while isAlive() do task.wait(2.0); if isEquipBestActive and PlotEquipBest then pcall(function() PlotEquipBest:InvokeServer() end) end end end)

task.spawn(function()
	while isAlive() do
		task.wait(45) 
		if isAntiAfkActive then pcall(function() VirtualUser:CaptureController(); VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame); task.wait(0.5); VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame) end) end
	end
end)

local log_label
local function add_log(text, is_error)
	if log_label then log_label.Text = text; log_label.TextColor3 = is_error and C_RED or Color3.fromRGB(150, 255, 150) end
end

task.spawn(function()
	while isAlive() do
		task.wait(1) 
		if isSellActive and current_min_sell > 0 and DataController and Mutations and PlotSellAnimal then
			local animals = DataController:Get("Animals")
			local bestIncome = player:GetAttribute("BestIncome") or 0
			if animals and type(animals) == "table" then
				for id, animal in pairs(animals) do
					if type(animal) == "table" and not animal.Placed then
						local success, income = pcall(function() return Mutations.IncomeOf(animal, bestIncome) end)
						if success and income and income > 0 and income <= current_min_sell then
							add_log("Vendendo " .. animal.ModelName .. "...")
							pcall(function() PlotSellAnimal:InvokeServer(id) end)
							task.wait(0.4)
						end
					end
				end
			end
		end
		
		if isBuyActive and ShopBuy then
			local char = player.Character
			local hrp = char and char:FindFirstChild("HumanoidRootPart")
			local foodShop = workspace:FindFirstChild("LocalPlotProps") and workspace.LocalPlotProps:FindFirstChild("FoodShop")
			if not foodShop then foodShop = workspace:FindFirstChild("FoodShop", true) end
			local target_zone = foodShop and (foodShop:FindFirstChild("ZonePad", true) or foodShop:FindFirstChild("ActivationCircle", true))
			
			if hrp and foodShop and target_zone then
				local money = 0
				if DataController then money = DataController:Get("Money") or 0 end
				local next_food_to_buy = nil
				for _, food in ipairs(foods_list) do
					if food_toggles[food.id] then
						local owned = false
						if DataController then owned = DataController:GetPath("OwnedFoods." .. food.id) == true end
						if not owned then next_food_to_buy = food; break end
					end
				end

				if next_food_to_buy then
					if money >= next_food_to_buy.price then
						local original_cframe = hrp.CFrame
						hrp.CFrame = CFrame.new(target_zone.Position + Vector3.new(0, 3, 0))
						if debug_label then debug_label.Text = "Comprando: " .. next_food_to_buy.name end
						task.wait(0.5) 
						pcall(function() ShopBuy:InvokeServer(next_food_to_buy.id) end)
						task.wait(0.2)
						hrp.CFrame = original_cframe
						if debug_label then debug_label.Text = "Comprou " .. next_food_to_buy.name end
					else
						if debug_label then debug_label.Text = "Sem dinheiro: " .. next_food_to_buy.name end
						local now = os.time()
						if now - last_notif_time >= 10 then send_notification("Sem Dinheiro"); last_notif_time = now end
					end
				end
			end
		end
	end
end)

-- ============================================================
-- INTERFACE
-- ============================================================
local gui = Instance.new("ScreenGui")
gui.Name = "LHub_Premium"
gui.ResetOnSpawn = false
gui.DisplayOrder = 9999999
pcall(function() gui.Parent = gethui() end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local antiHideConn = RunService.RenderStepped:Connect(function() if gui and not gui.Enabled then gui.Enabled = true end end)

-- ============================================================
-- FUNÇÃO DE DRAG CORRIGIDA
-- ============================================================
local function makeDraggable(frame, handle, on_click)
	local dragging = false
	local dragInput, mousePos, framePos
	local startPos
	local moved = false

	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			moved = false
			mousePos = input.Position
			framePos = frame.Position
			startPos = input.Position
			
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
					if not moved and on_click then
						on_click()
					end
				end
			end)
		end
	end)

	handle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			if dragging then
				local delta = input.Position - mousePos
				if math.abs(delta.X) > 5 or math.abs(delta.Y) > 5 then
					moved = true
				end
				frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
			end
		end
	end)
end

local isMenuOpen = false
local mainFrame

local function open_menu()
	isMenuOpen = true; mainFrame.Visible = true; mainFrame.BackgroundTransparency = 1
	local mainScale = mainFrame:FindFirstChildOfClass("UIScale")
	local stroke = mainFrame:FindFirstChildOfClass("UIStroke")
	mainScale.Scale = 0.5; stroke.Transparency = 1
	
	TweenService:Create(mainScale, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
	TweenService:Create(mainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0}):Play()
	TweenService:Create(stroke, TweenInfo.new(0.3), {Transparency = 0}):Play()
	send_notification("L Hub Aberto")
end

local function close_menu()
	isMenuOpen = false
	local mainScale = mainFrame:FindFirstChildOfClass("UIScale")
	local stroke = mainFrame:FindFirstChildOfClass("UIStroke")
	local closeTween = TweenService:Create(mainScale, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Scale = 0.5})
	TweenService:Create(mainFrame, TweenInfo.new(0.25), {BackgroundTransparency = 1}):Play()
	TweenService:Create(stroke, TweenInfo.new(0.25), {Transparency = 1}):Play()
	closeTween.Completed:Connect(function() mainFrame.Visible = false end); closeTween:Play()
	send_notification("L Hub Fechado")
end

-- BOTÃO FLUTUANTE "L"
local floatingBtn = Instance.new("TextButton")
floatingBtn.Size = UDim2.new(0, 50, 0, 50); floatingBtn.Position = UDim2.new(0.05, 0, 0.5, 0); floatingBtn.BackgroundColor3 = C_BG; floatingBtn.Text = "L"; floatingBtn.TextColor3 = C_RED; floatingBtn.Font = Enum.Font.GothamBlack; floatingBtn.TextSize = 30; floatingBtn.Active = true; floatingBtn.ZIndex = 9999; floatingBtn.Parent = gui
Instance.new("UICorner", floatingBtn).CornerRadius = UDim.new(0, 10)
local fStroke = Instance.new("UIStroke", floatingBtn); fStroke.Color = C_RED; fStroke.Thickness = 2

makeDraggable(floatingBtn, floatingBtn, function()
	if isMenuOpen then close_menu() else open_menu() end
end)

-- FRAME PRINCIPAL
mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 550, 0, 350); mainFrame.Position = UDim2.new(0.5, -275, 0.5, -175); mainFrame.BackgroundColor3 = C_BG; mainFrame.BorderSizePixel = 0; mainFrame.Active = true; mainFrame.Parent = gui; mainFrame.BackgroundTransparency = 1; mainFrame.Visible = false
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", mainFrame); stroke.Color = C_RED; stroke.Thickness = 2; stroke.Transparency = 1
local mainScale = Instance.new("UIScale", mainFrame); mainScale.Scale = 0.5

-- ESCUDO ANTI-CLIQUE
local inputSink = Instance.new("TextButton")
inputSink.Size = UDim2.new(1, 0, 1, 0); inputSink.BackgroundTransparency = 1; inputSink.Text = ""; inputSink.Active = true; inputSink.ZIndex = 1; inputSink.Parent = mainFrame

local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 35); topBar.BackgroundColor3 = C_DARK_RED; topBar.BorderSizePixel = 0; topBar.Parent = mainFrame; topBar.ZIndex = 10
Instance.new("UICorner", topBar).CornerRadius = UDim.new(0, 10)
local topGradient = Instance.new("UIGradient", topBar); topGradient.Color = ColorSequence.new(C_DARK_RED, C_BG); topGradient.Rotation = 0

-- Permite arrastar a janela principal usando a barra superior
makeDraggable(mainFrame, topBar)

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, -80, 1, 0); titleText.Position = UDim2.new(0, 15, 0, 0); titleText.BackgroundTransparency = 1; titleText.Text = "⛩️ L HUB - PREMIUM"; titleText.TextColor3 = C_TEXT; titleText.Font = Enum.Font.GothamBlack; titleText.TextSize = 16; titleText.TextXAlignment = Enum.TextXAlignment.Left; titleText.Parent = topBar; titleText.ZIndex = 11

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 24, 0, 24); minBtn.Position = UDim2.new(1, -65, 0, 5); minBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40); minBtn.Text = "-"; minBtn.TextColor3 = C_TEXT; minBtn.Font = Enum.Font.GothamBold; minBtn.TextSize = 16; minBtn.Parent = topBar; minBtn.ZIndex = 11
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 4)
minBtn.MouseButton1Click:Connect(function() close_menu() end)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 24); closeBtn.Position = UDim2.new(1, -35, 0, 5); closeBtn.BackgroundColor3 = C_RED; closeBtn.Text = "X"; closeBtn.TextColor3 = C_TEXT; closeBtn.Font = Enum.Font.GothamBold; closeBtn.TextSize = 14; closeBtn.Parent = topBar; closeBtn.ZIndex = 11
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 4)
closeBtn.MouseButton1Click:Connect(function()
	getgenv().LHub_Active_ID = "DEAD"
	if isFpsBoostActive then disable_fps_boost() end
	if ThrowMeter then ThrowMeter.TestAutoTap = nil end
	if antiHideConn then antiHideConn:Disconnect() end
	gui:Destroy(); notifGui:Destroy()
end)

local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0, 140, 1, -35); sidebar.Position = UDim2.new(0, 0, 0, 35); sidebar.BackgroundColor3 = C_SIDEBAR; sidebar.BorderSizePixel = 0; sidebar.Parent = mainFrame; sidebar.ZIndex = 5
local sidebarCorner = Instance.new("UICorner", sidebar); sidebarCorner.BottomLeftRadius = UDim.new(0, 10)
local sidebarLayout = Instance.new("UIListLayout", sidebar); sidebarLayout.Padding = UDim.new(0, 5); sidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; sidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Top
local sidePad = Instance.new("UIPadding", sidebar); sidePad.PaddingTop = UDim.new(0, 15)

local contentArea = Instance.new("Frame")
contentArea.Size = UDim2.new(1, -150, 1, -45); contentArea.Position = UDim2.new(0, 145, 0, 40); contentArea.BackgroundColor3 = C_BG; contentArea.BackgroundTransparency = 1; contentArea.Parent = mainFrame; contentArea.ZIndex = 5

local pages = {}; local tabButtons = {}
local function createPage(name)
	local page = Instance.new("ScrollingFrame"); page.Size = UDim2.new(1, 0, 1, 0); page.BackgroundTransparency = 1; page.BorderSizePixel = 0; page.ScrollBarThickness = 4; page.ScrollBarImageColor3 = C_RED; page.CanvasSize = UDim2.new(0, 0, 0, 0); page.AutomaticCanvasSize = Enum.AutomaticSize.Y; page.Visible = false; page.Parent = contentArea; page.ZIndex = 6
	local layout = Instance.new("UIListLayout", page); layout.Padding = UDim.new(0, 8); page.Name = name; pages[name] = page; return page
end
local function createTab(name, icon)
	local btn = Instance.new("TextButton"); btn.Size = UDim2.new(1, -10, 0, 35); btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25); btn.Text = " " .. icon .. "  " .. name; btn.TextColor3 = C_TEXT_DIM; btn.Font = Enum.Font.GothamBold; btn.TextSize = 13; btn.TextXAlignment = Enum.TextXAlignment.Left; btn.Parent = sidebar; btn.ZIndex = 6
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6); tabButtons[name] = btn
	btn.MouseButton1Click:Connect(function()
		for n, p in pairs(pages) do p.Visible = (n == name) end
		for n, b in pairs(tabButtons) do b.BackgroundColor3 = (n == name) and C_DARK_RED or Color3.fromRGB(25, 25, 25); b.TextColor3 = (n == name) and C_TEXT or C_TEXT_DIM end
		if foodPanel then foodPanel.Visible = false end
	end)
	return btn
end
local homePage = createPage("Home"); local farmPage = createPage("Farm"); local shopPage = createPage("Loja"); local utilPage = createPage("Utilidades"); local setPage = createPage("Config")
local tabHome = createTab("Home", "🏠"); local tabFarm = createTab("Farm", "⚡"); local tabShop = createTab("Loja", "🛒"); local tabUtil = createTab("Utilidades", "🛠️"); local tabSet = createTab("Config", "⚙️")
homePage.Visible = true; tabHome.BackgroundColor3 = C_DARK_RED; tabHome.TextColor3 = C_TEXT

local OFF_COLOR = Color3.fromRGB(30, 30, 30); local ON_COLOR = C_RED
local function createButton(parent, text, order)
	local btn = Instance.new("TextButton"); btn.Size = UDim2.new(1, -6, 0, 38); btn.BackgroundColor3 = OFF_COLOR; btn.Text = text .. ": OFF"; btn.TextColor3 = C_TEXT_DIM; btn.Font = Enum.Font.GothamBold; btn.TextSize = 14; btn.LayoutOrder = order; btn.Parent = parent; btn.ZIndex = 7
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	local bStroke = Instance.new("UIStroke", btn); bStroke.Color = Color3.fromRGB(50, 50, 50)
	return btn, bStroke
end
local function toggleButton(btn, bStroke, state, baseText)
	btn.Text = baseText .. ": " .. (state and "ON" or "OFF"); btn.BackgroundColor3 = state and ON_COLOR or OFF_COLOR; btn.TextColor3 = state and C_TEXT or C_TEXT_DIM; bStroke.Color = state and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(50, 50, 50)
	send_notification(baseText .. " " .. (state and "Ligado" or "Desligado"))
end

local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -6, 0, 40); infoLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 20); infoLabel.Text = "Bem-vindo ao L Hub Premium!\nUse o menu lateral para navegar."; infoLabel.TextColor3 = C_RED; infoLabel.Font = Enum.Font.GothamBold; infoLabel.TextSize = 14; infoLabel.TextWrapped = true; infoLabel.LayoutOrder = 1; infoLabel.Parent = homePage; infoLabel.ZIndex = 7
Instance.new("UICorner", infoLabel).CornerRadius = UDim.new(0, 6)

local creditsFrame = Instance.new("Frame")
creditsFrame.Size = UDim2.new(1, -6, 0, 80); creditsFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20); creditsFrame.LayoutOrder = 2; creditsFrame.Parent = homePage; creditsFrame.ZIndex = 7
Instance.new("UICorner", creditsFrame).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", creditsFrame).Color = C_DARK_RED

local avatarImg = Instance.new("ImageLabel")
avatarImg.Size = UDim2.new(0, 60, 0, 60); avatarImg.Position = UDim2.new(0, 10, 0.5, 0); avatarImg.AnchorPoint = Vector2.new(0, 0.5); avatarImg.BackgroundColor3 = C_BG; avatarImg.Parent = creditsFrame; avatarImg.ZIndex = 8
Instance.new("UICorner", avatarImg).CornerRadius = UDim.new(0, 30)
local avStroke = Instance.new("UIStroke", avatarImg); avStroke.Color = C_RED; avStroke.Thickness = 2
task.spawn(function()
	local success, userId = pcall(function() return Players:GetUserIdFromNameAsync("ROBZINHO666") end)
	if success and userId then local thumbSuccess, thumbUrl = pcall(function() return Players:GetUserThumbnailAsync(userId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150) end); if thumbSuccess and thumbUrl then avatarImg.Image = thumbUrl end end
end)
local creatorText = Instance.new("TextLabel")
creatorText.Size = UDim2.new(1, -80, 0, 20); creatorText.Position = UDim2.new(0, 80, 0, 20); creatorText.BackgroundTransparency = 1; creatorText.Text = "Creator"; creatorText.TextColor3 = C_RED; creatorText.Font = Enum.Font.GothamBold; creatorText.TextSize = 14; creatorText.TextXAlignment = Enum.TextXAlignment.Left; creatorText.Parent = creditsFrame; creatorText.ZIndex = 8
local nameText = Instance.new("TextLabel")
nameText.Size = UDim2.new(1, -80, 0, 20); nameText.Position = UDim2.new(0, 80, 0, 40); nameText.BackgroundTransparency = 1; nameText.Text = "GOD_ROB"; nameText.TextColor3 = C_TEXT; nameText.Font = Enum.Font.GothamMedium; nameText.TextSize = 13; nameText.TextXAlignment = Enum.TextXAlignment.Left; nameText.Parent = creditsFrame; nameText.ZIndex = 8
task.spawn(function()
	local tick = 0
	while isAlive() and creatorText.Parent do task.wait(0.05); tick = tick + 0.01; creatorText.TextColor3 = Color3.fromHSV(tick % 1, 0.8, 1) end
end)

local autoPerfectBtn, perfectStroke = createButton(farmPage, "Auto Perfect", 1)
autoPerfectBtn.MouseButton1Click:Connect(function() isPerfectActive = not isPerfectActive; toggleButton(autoPerfectBtn, perfectStroke, isPerfectActive, "Auto Perfect") end)
local autoPopsBtn, popsStroke = createButton(farmPage, "Auto Pops (Roxo)", 2)
autoPopsBtn.MouseButton1Click:Connect(function() isPopsActive = not isPopsActive; toggleButton(autoPopsBtn, popsStroke, isPopsActive, "Auto Pops (Roxo)") end)
local equipBestBtn, equipStroke = createButton(farmPage, "EquipBest", 3)
equipBestBtn.MouseButton1Click:Connect(function() isEquipBestActive = not isEquipBestActive; toggleButton(equipBestBtn, equipStroke, isEquipBestActive, "EquipBest") end)

local function createCard(parent, titleText, layoutOrder)
	local card = Instance.new("Frame"); card.Size = UDim2.new(1, -6, 0, 0); card.AutomaticSize = Enum.AutomaticSize.Y; card.BackgroundColor3 = Color3.fromRGB(22, 22, 22); card.LayoutOrder = layoutOrder; card.Parent = parent; card.ZIndex = 7
	Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)
	local cStroke = Instance.new("UIStroke", card); cStroke.Color = C_DARK_RED
	local cPad = Instance.new("UIPadding", card); cPad.PaddingTop = UDim.new(0, 8); cPad.PaddingBottom = UDim.new(0, 8); cPad.PaddingLeft = UDim.new(0, 8); cPad.PaddingRight = UDim.new(0, 8)
	local cLayout = Instance.new("UIListLayout", card); cLayout.Padding = UDim.new(0, 6); cLayout.SortOrder = Enum.SortOrder.LayoutOrder
	local cTitle = Instance.new("TextLabel"); cTitle.Size = UDim2.new(1, 0, 0, 20); cTitle.BackgroundTransparency = 1; cTitle.Text = titleText; cTitle.TextColor3 = C_RED; cTitle.Font = Enum.Font.GothamBold; cTitle.TextSize = 13; cTitle.TextXAlignment = Enum.TextXAlignment.Left; cTitle.LayoutOrder = 0; cTitle.Parent = card; cTitle.ZIndex = 8
	return card
end

local buyCard = createCard(shopPage, "🛒 COMPRAR COMIDAS", 1)
local buyHeader = Instance.new("Frame")
buyHeader.Size = UDim2.new(1, 0, 0, 38); buyHeader.BackgroundTransparency = 1; buyHeader.LayoutOrder = 1; buyHeader.Parent = buyCard; buyHeader.ZIndex = 8
local buyHeaderLayout = Instance.new("UIListLayout", buyHeader); buyHeaderLayout.FillDirection = Enum.FillDirection.Horizontal; buyHeaderLayout.Padding = UDim.new(0, 4)

local autoBuyBtn, buyStroke = createButton(buyHeader, "Mestre Auto Comprar", 1)
autoBuyBtn.Size = UDim2.new(0.8, -4, 0, 38); autoBuyBtn.MouseButton1Click:Connect(function() isBuyActive = not isBuyActive; toggleButton(autoBuyBtn, buyStroke, isBuyActive, "Mestre Auto Comprar") end)

local plusBtn = Instance.new("TextButton")
plusBtn.Size = UDim2.new(0.2, 0, 0, 38); plusBtn.BackgroundColor3 = C_DARK_RED; plusBtn.Text = "+"; plusBtn.TextColor3 = C_TEXT; plusBtn.Font = Enum.Font.GothamBlack; plusBtn.TextSize = 20; plusBtn.Parent = buyHeader; plusBtn.ZIndex = 8
Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 6)
local plusStroke = Instance.new("UIStroke", plusBtn); plusStroke.Color = C_RED

debug_label = Instance.new("TextLabel")
debug_label.Size = UDim2.new(1, 0, 0, 20); debug_label.BackgroundTransparency = 1; debug_label.Text = "Aguardando..."; debug_label.TextColor3 = C_TEXT_DIM; debug_label.Font = Enum.Font.Code; debug_label.TextSize = 11; debug_label.LayoutOrder = 2; debug_label.Parent = buyCard; debug_label.ZIndex = 8

local foodPanel = Instance.new("Frame")
foodPanel.Size = UDim2.new(0, 220, 0, 220); foodPanel.Position = UDim2.new(0.85, -4, 0, 42); foodPanel.BackgroundColor3 = C_BG; foodPanel.BorderSizePixel = 0; foodPanel.ZIndex = 50; foodPanel.Visible = false; foodPanel.Parent = buyHeader
Instance.new("UICorner", foodPanel).CornerRadius = UDim.new(0, 8)
local fpStroke = Instance.new("UIStroke", foodPanel); fpStroke.Color = C_RED

local searchBox = Instance.new("TextBox")
searchBox.Size = UDim2.new(1, -10, 0, 30); searchBox.Position = UDim2.new(0, 5, 0, 5); searchBox.BackgroundColor3 = Color3.fromRGB(35, 35, 35); searchBox.TextColor3 = C_TEXT; searchBox.Font = Enum.Font.GothamMedium; searchBox.TextSize = 13; searchBox.Text = ""; searchBox.PlaceholderText = "Pesquisar..."; searchBox.PlaceholderColor3 = Color3.fromRGB(100,100,100); searchBox.ZIndex = 51; searchBox.Parent = foodPanel
Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 6)

local foodScroll = Instance.new("ScrollingFrame")
foodScroll.Size = UDim2.new(1, -10, 1, -45); foodScroll.Position = UDim2.new(0, 5, 0, 40); foodScroll.BackgroundTransparency = 1; foodScroll.BorderSizePixel = 0; foodScroll.ScrollBarThickness = 4; foodScroll.ScrollBarImageColor3 = C_RED; foodScroll.ZIndex = 51; foodScroll.Parent = foodPanel
foodScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
foodScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y

local function refresh_food_panel()
	foodScroll:ClearAllChildren()
	
	local fsLayout = Instance.new("UIListLayout")
	fsLayout.Padding = UDim.new(0, 4)
	fsLayout.SortOrder = Enum.SortOrder.LayoutOrder
	fsLayout.Parent = foodScroll
	
	local txt = string.lower(searchBox.Text)
	
	for idx, food in ipairs(foods_list) do
		if txt == "" or string.find(string.lower(food.name), txt, 1, true) then
			local btn = Instance.new("TextButton")
			btn.Size = UDim2.new(1, -4, 0, 28)
			btn.BackgroundColor3 = food_toggles[food.id] and ON_COLOR or OFF_COLOR
			btn.Text = food.name .. (food_toggles[food.id] and " [ON]" or " [OFF]")
			btn.TextColor3 = food_toggles[food.id] and C_TEXT or C_TEXT_DIM
			btn.Font = Enum.Font.GothamMedium
			btn.TextSize = 12
			btn.ZIndex = 52
			btn.LayoutOrder = idx
			btn.Parent = foodScroll
			Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
			
			btn.MouseButton1Click:Connect(function()
				food_toggles[food.id] = not food_toggles[food.id]
				btn.Text = food.name .. (food_toggles[food.id] and " [ON]" or " [OFF]")
				btn.BackgroundColor3 = food_toggles[food.id] and ON_COLOR or OFF_COLOR
				btn.TextColor3 = food_toggles[food.id] and C_TEXT or C_TEXT_DIM
				send_notification(food.name .. (food_toggles[food.id] and " ON" or " OFF"))
			end)
		end
	end
end

searchBox:GetPropertyChangedSignal("Text"):Connect(refresh_food_panel)
plusBtn.MouseButton1Click:Connect(function() foodPanel.Visible = not foodPanel.Visible; if foodPanel.Visible then refresh_food_panel() end end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if not foodPanel.Visible then return end
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		local pos = input.Position; local objects = player.PlayerGui:GetGuiObjectsAtPosition(pos.X, pos.Y)
		local clickedInsidePanel = false
		for _, obj in ipairs(objects) do if obj == foodPanel or obj:IsDescendantOf(foodPanel) then clickedInsidePanel = true; break end end
		local clickedPlus = false
		for _, obj in ipairs(objects) do if obj == plusBtn or obj:IsDescendantOf(plusBtn) then clickedPlus = true; break end end
		if not clickedInsidePanel and not clickedPlus then foodPanel.Visible = false end
	end
end)

local divider = Instance.new("Frame")
divider.Size = UDim2.new(1, -6, 0, 2); divider.BackgroundColor3 = C_DARK_RED; divider.BorderSizePixel = 0; divider.LayoutOrder = 2; divider.Parent = shopPage; divider.ZIndex = 7

local sellCard = createCard(shopPage, "💰 VENDER ANIMAIS", 3)
local autoSellBtn, sellStroke = createButton(sellCard, "Auto Vender", 1)
autoSellBtn.MouseButton1Click:Connect(function() isSellActive = not isSellActive; toggleButton(autoSellBtn, sellStroke, isSellActive, "Auto Vender") end)

local minPriceLabel = Instance.new("TextLabel")
minPriceLabel.Size = UDim2.new(1, 0, 0, 15); minPriceLabel.BackgroundTransparency = 1; minPriceLabel.Text = "Renda MÍNIMA por segundo:"; minPriceLabel.TextColor3 = C_TEXT_DIM; minPriceLabel.Font = Enum.Font.GothamBold; minPriceLabel.TextSize = 11; minPriceLabel.TextXAlignment = Enum.TextXAlignment.Left; minPriceLabel.LayoutOrder = 2; minPriceLabel.Parent = sellCard; minPriceLabel.ZIndex = 8

local minPriceBox = Instance.new("TextBox")
minPriceBox.Size = UDim2.new(1, 0, 0, 30); minPriceBox.BackgroundColor3 = Color3.fromRGB(35, 35, 35); minPriceBox.TextColor3 = C_TEXT; minPriceBox.Font = Enum.Font.GothamBold; minPriceBox.TextSize = 13; minPriceBox.Text = ""; minPriceBox.PlaceholderText = "Ex: 150M..."; minPriceBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 100); minPriceBox.LayoutOrder = 3; minPriceBox.Parent = sellCard; minPriceBox.ZIndex = 8
Instance.new("UICorner", minPriceBox).CornerRadius = UDim.new(0, 6)
local boxStroke = Instance.new("UIStroke", minPriceBox); boxStroke.Color = C_DARK_RED

local parsedPriceLabel = Instance.new("TextLabel")
parsedPriceLabel.Size = UDim2.new(1, 0, 0, 15); parsedPriceLabel.BackgroundTransparency = 1; parsedPriceLabel.Text = "Renda Mín: 0/s"; parsedPriceLabel.TextColor3 = Color3.fromRGB(150, 255, 150); parsedPriceLabel.Font = Enum.Font.GothamMedium; parsedPriceLabel.TextSize = 11; parsedPriceLabel.TextXAlignment = Enum.TextXAlignment.Left; parsedPriceLabel.LayoutOrder = 4; parsedPriceLabel.Parent = sellCard; parsedPriceLabel.ZIndex = 8

log_label = Instance.new("TextLabel")
log_label.Size = UDim2.new(1, 0, 0, 25); log_label.BackgroundTransparency = 1; log_label.Text = "Aguardando..."; log_label.TextColor3 = C_TEXT_DIM; log_label.Font = Enum.Font.Code; log_label.TextSize = 10; log_label.TextWrapped = true; log_label.LayoutOrder = 5; log_label.Parent = sellCard; log_label.ZIndex = 8

minPriceBox:GetPropertyChangedSignal("Text"):Connect(function()
	local parsed = parse_abbreviated_number(minPriceBox.Text)
	if parsed > 0 then current_min_sell = parsed; minPriceBox.TextColor3 = C_TEXT; parsedPriceLabel.Text = "Renda Mín: " .. tostring(parsed) .. "/s"; parsedPriceLabel.TextColor3 = Color3.fromRGB(150, 255, 150)
	else current_min_sell = 0; minPriceBox.TextColor3 = C_RED; parsedPriceLabel.Text = "Renda Mín: Aguardando..."; parsedPriceLabel.TextColor3 = C_RED end
end)

local antiAfkBtn, antiAfkStroke = createButton(utilPage, "Anti-AFK", 1)
antiAfkBtn.MouseButton1Click:Connect(function() isAntiAfkActive = not isAntiAfkActive; toggleButton(antiAfkBtn, antiAfkStroke, isAntiAfkActive, "Anti-AFK") end)

local fpsBoostBtn, fpsBoostStroke = createButton(utilPage, "FPS Boost (Tira Texturas)", 2)
fpsBoostBtn.MouseButton1Click:Connect(function() isFpsBoostActive = not isFpsBoostActive; toggleButton(fpsBoostBtn, fpsBoostStroke, isFpsBoostActive, "FPS Boost"); if isFpsBoostActive then enable_fps_boost() else disable_fps_boost() end end)

local infoLabel2 = Instance.new("TextLabel")
infoLabel2.Size = UDim2.new(1, -6, 0, 40); infoLabel2.BackgroundTransparency = 1; infoLabel2.Text = "Para fechar o script, clique no 'X' no topo.\nPara esconder, clique no botão 'L' flutuante."; infoLabel2.TextColor3 = C_TEXT_DIM; infoLabel2.Font = Enum.Font.Gotham; infoLabel2.TextSize = 12; infoLabel2.TextWrapped = true; infoLabel2.Parent = setPage; infoLabel2.ZIndex = 7

task.spawn(function()
	local introGui = Instance.new("ScreenGui"); introGui.Name = "LHubIntro"; introGui.DisplayOrder = 2147483647; introGui.IgnoreGuiInset = true; introGui.Parent = player.PlayerGui
	local bgDim = Instance.new("Frame"); bgDim.Size = UDim2.new(1, 0, 1, 0); bgDim.BackgroundColor3 = Color3.new(0, 0, 0); bgDim.BackgroundTransparency = 1; bgDim.Parent = introGui
	TweenService:Create(bgDim, TweenInfo.new(0.5), {BackgroundTransparency = 0.6}):Play()
	local holoText = Instance.new("TextLabel"); holoText.Size = UDim2.new(0, 900, 0, 200); holoText.Position = UDim2.new(0.5, -450, 0.5, -100); holoText.BackgroundTransparency = 1; holoText.Text = "L HUB"; holoText.Font = Enum.Font.GothamBlack; holoText.TextSize = 180; holoText.TextColor3 = C_RED; holoText.TextTransparency = 1; holoText.Parent = introGui
	local uig = Instance.new("UIGradient"); uig.Color = ColorSequence.new(C_RED, C_DARK_RED); uig.Parent = holoText
	local stroke = Instance.new("UIStroke", holoText); stroke.Color = C_RED; stroke.Thickness = 4; stroke.Transparency = 1
	local tweenIn = TweenService:Create(holoText, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {TextTransparency = 0, Size = UDim2.new(0, 1000, 0, 250), Position = UDim2.new(0.5, -500, 0.5, -125)})
	tweenIn:Play(); TweenService:Create(stroke, TweenInfo.new(0.6), {Transparency = 0}):Play()
	task.wait(0.6)
	for i = 1, 15 do task.wait(0.05); holoText.Position = UDim2.new(0.5, -500 + math.random(-20, 20), 0.5, -125 + math.random(-15, 15)); holoText.TextTransparency = math.random(0, 40) / 100; stroke.Thickness = math.random(2, 8) end
	holoText.Position = UDim2.new(0.5, -500, 0.5, -125); holoText.TextTransparency = 0; stroke.Thickness = 4
	task.wait(1.5)
	local tweenOut = TweenService:Create(holoText, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {TextTransparency = 1, Size = UDim2.new(0, 1200, 0, 350), Position = UDim2.new(0.5, -600, 0.5, -175)})
	tweenOut:Play(); TweenService:Create(stroke, TweenInfo.new(0.8), {Transparency = 1}):Play(); TweenService:Create(bgDim, TweenInfo.new(0.8), {BackgroundTransparency = 1}):Play()
	task.wait(1); introGui:Destroy(); open_menu()
end)

print("✅ L Hub Premium (Game Locked) carregado.")
