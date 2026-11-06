-- L Hub (LocalScript) - Completo com Kill Switch
-- Cole no Delta.

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer
local VirtualUser = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")

-- ============================================================
-- CONFIGURAÇÕES E ESTADOS
-- ============================================================
local isAlive = true -- CONTROLE DE VIDA DO SCRIPT
local isPerfectActive = false
local isPopsActive = false
local isEquipBestActive = false
local isBuyActive = false
local isSellActive = false
local current_min_sell = 0
local PERFECT_TIME = 1.10

-- Formata número para texto (ex: 1.5B)
local function format_number(n)
	if n >= 1e12 then return string.format("%.1fT", n / 1e12) end
	if n >= 1e9 then return string.format("%.1fB", n / 1e9) end
	if n >= 1e6 then return string.format("%.1fM", n / 1e6) end
	if n >= 1e3 then return string.format("%.1fK", n / 1e3) end
	return tostring(n)
end

-- Converte texto (ex: 1b, 500m) em número
local function parse_abbreviated_number(str)
	str = string.lower(str)
	str = string.gsub(str, "%s+", "")
	local num, suffix = string.match(str, "([%d%.]+)([kmbtq]?)")
	if not num then return 0 end
	local n = tonumber(num)
	if not n then return 0 end
	if suffix == "k" then n = n * 1e3
	elseif suffix == "m" then n = n * 1e6
	elseif suffix == "b" then n = n * 1e9
	elseif suffix == "t" then n = n * 1e12
	elseif suffix == "q" then n = n * 1e15
	end
	return math.floor(n)
end

-- ============================================================
-- LÓGICA FARM
-- ============================================================
local ThrowMeter
pcall(function()
	ThrowMeter = require(player.PlayerScripts.Client.Controllers.ThrowMeter)
end)

-- Auto Perfect
task.spawn(function()
	while isAlive do
		task.wait(0.2) 
		if isPerfectActive and ThrowMeter then
			ThrowMeter.TestAutoTap = PERFECT_TIME
		elseif not isPerfectActive and ThrowMeter then
			ThrowMeter.TestAutoTap = nil
		end
	end
end)

-- Auto Pops (Animação Roxa)
task.spawn(function()
	while isAlive do
		task.wait(0.2) 
		if isPopsActive then
			local trainingGui = player.PlayerGui:FindFirstChild("TrainingSequence")
			if trainingGui then
				local layer = trainingGui:FindFirstChild("TapBonusLayer")
				if layer then
					local botaoX2 = layer:FindFirstChild("TapBonus")
					if botaoX2 and botaoX2.Visible then
						local fs = getfenv and getfenv().firesignal
						local gc = getfenv and getfenv().getconnections
						if fs then
							pcall(function() fs(botaoX2.Activated) end)
							pcall(function() fs(botaoX2.MouseButton1Click) end)
						elseif gc then
							local ok, conns = pcall(gc, botaoX2.Activated)
							if ok and conns then
								for _, c in ipairs(conns) do pcall(function() c:Fire() end) end
							end
						end
						pcall(function()
							VirtualUser:CaptureController()
							VirtualUser:Button1Down(Vector2.new(0.5, 0.5))
							VirtualUser:Button1Up(Vector2.new(0.5, 0.5))
						end)
						task.wait(0.2)
					end
				end
			end
		end
	end
end)

-- ============================================================
-- LÓGICA BASE (EquipBest)
-- ============================================================
task.spawn(function()
	while isAlive do
		task.wait(2.0) 
		if isEquipBestActive then
			local remotes = ReplicatedStorage:FindFirstChild("Remotes")
			if remotes then
				local equipRemote = remotes:FindFirstChild("Plot_EquipBest")
				if equipRemote then
					pcall(function() equipRemote:InvokeServer() end)
				end
			end
		end
	end
end)

-- ============================================================
-- LÓGICA LOJA E PREÇO
-- ============================================================
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 5)
local PlotSellAnimal = Remotes and Remotes:FindFirstChild("Plot_SellAnimal")
local ShopBuy = Remotes and Remotes:FindFirstChild("Shop_Buy")

local DataController, Mutations, FoodsConfig
pcall(function()
	DataController = require(player.PlayerScripts.Client.Controllers.DataController)
	Mutations = require(ReplicatedStorage.Shared.Mutations)
	FoodsConfig = require(ReplicatedStorage.Config.FoodsConfig)
end)

local log_label

local function add_log(text, is_error)
	if log_label then
		log_label.Text = text
		log_label.TextColor3 = is_error and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(150, 255, 150)
	end
end

task.spawn(function()
	while isAlive do
		task.wait(0.8)
		
		-- AUTO VENDER (Por Renda/s)
		if isSellActive and current_min_sell > 0 and DataController and Mutations and PlotSellAnimal then
			local animals = DataController:Get("Animals")
			local bestIncome = player:GetAttribute("BestIncome") or 0
			
			if animals and type(animals) == "table" then
				for id, animal in pairs(animals) do
					if type(animal) == "table" and not animal.Placed then
						local success, income = pcall(function()
							return Mutations.IncomeOf(animal, bestIncome)
						end)
						
						if success and income and income > 0 then
							if income <= current_min_sell then
								add_log("Vendendo " .. animal.ModelName .. " (" .. format_number(income) .. "/s)...")
								pcall(function() PlotSellAnimal:InvokeServer(id) end)
								task.wait(0.4)
							else
								add_log("Ignorado: " .. animal.ModelName .. " (" .. format_number(income) .. "/s) > Mínimo")
							end
						end
					end
				end
			end
		end
		
		-- AUTO COMPRAR (Inteligente)
		if isBuyActive and ShopBuy and DataController and FoodsConfig then
			local money = DataController:Get("Money") or 0
			local owned_foods = DataController:Get("OwnedFoods") or {}
			
			local max_owned = 0
			for idx, owned in pairs(owned_foods) do
				if owned and tonumber(idx) and tonumber(idx) > max_owned then
					max_owned = tonumber(idx)
				end
			end
			
			local next_idx = max_owned + 1
			local food_price = math.huge
			local success_get, food = pcall(function() return FoodsConfig.GetAny(next_idx) end)
			
			if success_get and food and food.Price then
				food_price = food.Price
			end
			
			if money >= food_price then
				pcall(function() ShopBuy:InvokeServer(next_idx) end)
				task.wait(0.3)
			end
		end
	end
end)

-- ============================================================
-- INTERFACE (L HUB)
-- ============================================================
local gui = Instance.new("ScreenGui")
gui.Name = "LHub_Perfect"
gui.ResetOnSpawn = false
gui.DisplayOrder = 9999999

pcall(function() gui.Parent = gethui() end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local antiHideConn = RunService.RenderStepped:Connect(function()
	if gui and not gui.Enabled then gui.Enabled = true end
end)

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 320)
mainFrame.Position = UDim2.new(0.5, -140, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = gui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(60, 60, 80)
stroke.Thickness = 1.5
stroke.Parent = mainFrame

-- BARRA DE TÍTULO
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(28, 28, 35)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 10)

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, -70, 1, 0)
titleText.Position = UDim2.new(0, 10, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "⚡ L Hub"
titleText.TextColor3 = Color3.fromRGB(200, 200, 220)
titleText.Font = Enum.Font.GothamBold
titleText.TextSize = 14
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Parent = titleBar

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 24, 0, 24)
minBtn.Position = UDim2.new(1, -60, 0, 5)
minBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
minBtn.Text = "-"
minBtn.TextColor3 = Color3.new(1, 1, 1)
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 14
minBtn.Parent = titleBar
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 4)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 24)
closeBtn.Position = UDim2.new(1, -30, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 14
closeBtn.Parent = titleBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 4)

local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, -20, 0, 30)
tabBar.Position = UDim2.new(0, 10, 0, 45)
tabBar.BackgroundTransparency = 1
tabBar.Parent = mainFrame

local tabLayout = Instance.new("UIListLayout", tabBar)
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 4)

local pages = Instance.new("Frame")
pages.Size = UDim2.new(1, -20, 1, -85)
pages.Position = UDim2.new(0, 10, 0, 80)
pages.BackgroundTransparency = 1
pages.Parent = mainFrame

local OFF_COLOR = Color3.fromRGB(50, 50, 70)
local ON_COLOR = Color3.fromRGB(60, 170, 90)
local TAB_ON = Color3.fromRGB(70, 90, 200)

local tabButtons = {}
local function createTab(name, page)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.25, -3, 1, 0) -- Ajustado para 4 abas
	btn.BackgroundColor3 = OFF_COLOR
	btn.Text = name
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 12
	btn.Parent = tabBar
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	
	tabButtons[btn] = page
	btn.MouseButton1Click:Connect(function()
		for b, p in pairs(tabButtons) do
			p.Visible = (b == btn)
			b.BackgroundColor3 = (b == btn) and TAB_ON or OFF_COLOR
		end
	end)
	return btn
end

local function createPage()
	local page = Instance.new("ScrollingFrame")
	page.Size = UDim2.new(1, 0, 1, 0)
	page.BackgroundTransparency = 1
	page.BorderSizePixel = 0
	page.ScrollBarThickness = 4
	page.CanvasSize = UDim2.new(0, 0, 0, 0)
	page.AutomaticCanvasSize = Enum.AutomaticSize.Y
	page.Visible = false
	page.Parent = pages
	local layout = Instance.new("UIListLayout", page)
	layout.Padding = UDim.new(0, 8)
	layout.SortOrder = Enum.SortOrder.LayoutOrder
	return page
end

local mainPage = createPage()
local farmPage = createPage()
local shopPage = createPage()
local basePage = createPage()

local tabMain = createTab("Main", mainPage)
local tabFarm = createTab("Farm", farmPage)
local tabShop = createTab("Loja", shopPage)
local tabBase = createTab("Base", basePage)

mainPage.Visible = true
tabMain.BackgroundColor3 = TAB_ON

local isMinimized = false
minBtn.MouseButton1Click:Connect(function()
	isMinimized = not isMinimized
	pages.Visible = not isMinimized
	tabBar.Visible = not isMinimized
	mainFrame.Size = isMinimized and UDim2.new(0, 280, 0, 35) or UDim2.new(0, 280, 0, 320)
	minBtn.Text = isMinimized and "+" or "-"
end)

-- KILL SWITCH
closeBtn.MouseButton1Click:Connect(function()
	isAlive = false 
	isPerfectActive = false
	isPopsActive = false
	isEquipBestActive = false
	isBuyActive = false
	isSellActive = false
	if ThrowMeter then ThrowMeter.TestAutoTap = nil end
	if antiHideConn then antiHideConn:Disconnect() end
	gui:Destroy()
end)

-- ============================================================
-- PÁGINAS
-- ============================================================

-- ABA MAIN
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -6, 0, 50)
infoLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
infoLabel.Text = "Bem-vindo ao L Hub!\nNavegue pelas abas para ativar as funções."
infoLabel.TextColor3 = Color3.fromRGB(180, 200, 255)
infoLabel.Font = Enum.Font.GothamMedium
infoLabel.TextSize = 13
infoLabel.TextWrapped = true
infoLabel.LayoutOrder = 1
infoLabel.Parent = mainPage
Instance.new("UICorner", infoLabel).CornerRadius = UDim.new(0, 6)

-- ABA FARM
local autoPerfectBtn = Instance.new("TextButton")
autoPerfectBtn.Size = UDim2.new(1, -6, 0, 45)
autoPerfectBtn.BackgroundColor3 = OFF_COLOR
autoPerfectBtn.Text = "Auto Perfect: OFF"
autoPerfectBtn.TextColor3 = Color3.new(1, 1, 1)
autoPerfectBtn.Font = Enum.Font.GothamBold
autoPerfectBtn.TextSize = 15
autoPerfectBtn.LayoutOrder = 1
autoPerfectBtn.Parent = farmPage
Instance.new("UICorner", autoPerfectBtn).CornerRadius = UDim.new(0, 8)

autoPerfectBtn.MouseButton1Click:Connect(function()
	isPerfectActive = not isPerfectActive
	autoPerfectBtn.Text = "Auto Perfect: " .. (isPerfectActive and "ON" or "OFF")
	autoPerfectBtn.BackgroundColor3 = isPerfectActive and ON_COLOR or OFF_COLOR
end)

local autoPopsBtn = Instance.new("TextButton")
autoPopsBtn.Size = UDim2.new(1, -6, 0, 45)
autoPopsBtn.BackgroundColor3 = OFF_COLOR
autoPopsBtn.Text = "Auto Pops (Roxo): OFF"
autoPopsBtn.TextColor3 = Color3.new(1, 1, 1)
autoPopsBtn.Font = Enum.Font.GothamBold
autoPopsBtn.TextSize = 15
autoPopsBtn.LayoutOrder = 2
autoPopsBtn.Parent = farmPage
Instance.new("UICorner", autoPopsBtn).CornerRadius = UDim.new(0, 8)

autoPopsBtn.MouseButton1Click:Connect(function()
	isPopsActive = not isPopsActive
	autoPopsBtn.Text = "Auto Pops (Roxo): " .. (isPopsActive and "ON" or "OFF")
	autoPopsBtn.BackgroundColor3 = isPopsActive and ON_COLOR or OFF_COLOR
end)

-- ABA LOJA
local autoBuyBtn = Instance.new("TextButton")
autoBuyBtn.Size = UDim2.new(1, -6, 0, 45)
autoBuyBtn.BackgroundColor3 = OFF_COLOR
autoBuyBtn.Text = "Auto Comprar Inteligente: OFF"
autoBuyBtn.TextColor3 = Color3.new(1, 1, 1)
autoBuyBtn.Font = Enum.Font.GothamBold
autoBuyBtn.TextSize = 15
autoBuyBtn.LayoutOrder = 1
autoBuyBtn.Parent = shopPage
Instance.new("UICorner", autoBuyBtn).CornerRadius = UDim.new(0, 8)

autoBuyBtn.MouseButton1Click:Connect(function()
	isBuyActive = not isBuyActive
	autoBuyBtn.Text = "Auto Comprar Inteligente: " .. (isBuyActive and "ON" or "OFF")
	autoBuyBtn.BackgroundColor3 = isBuyActive and ON_COLOR or OFF_COLOR
end)

local autoSellBtn = Instance.new("TextButton")
autoSellBtn.Size = UDim2.new(1, -6, 0, 45)
autoSellBtn.BackgroundColor3 = OFF_COLOR
autoSellBtn.Text = "Auto Vender (Por Renda): OFF"
autoSellBtn.TextColor3 = Color3.new(1, 1, 1)
autoSellBtn.Font = Enum.Font.GothamBold
autoSellBtn.TextSize = 15
autoSellBtn.LayoutOrder = 2
autoSellBtn.Parent = shopPage
Instance.new("UICorner", autoSellBtn).CornerRadius = UDim.new(0, 8)

autoSellBtn.MouseButton1Click:Connect(function()
	isSellActive = not isSellActive
	autoSellBtn.Text = "Auto Vender (Por Renda): " .. (isSellActive and "ON" or "OFF")
	autoSellBtn.BackgroundColor3 = isSellActive and ON_COLOR or OFF_COLOR
end)

local minPriceLabel = Instance.new("TextLabel")
minPriceLabel.Size = UDim2.new(1, -6, 0, 20)
minPriceLabel.BackgroundTransparency = 1
minPriceLabel.Text = "Digite a renda MÍNIMA por segundo:"
minPriceLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
minPriceLabel.Font = Enum.Font.GothamBold
minPriceLabel.TextSize = 12
minPriceLabel.TextXAlignment = Enum.TextXAlignment.Left
minPriceLabel.LayoutOrder = 3
minPriceLabel.Parent = shopPage

local minPriceBox = Instance.new("TextBox")
minPriceBox.Size = UDim2.new(1, -6, 0, 40)
minPriceBox.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
minPriceBox.TextColor3 = Color3.new(1, 1, 1)
minPriceBox.Font = Enum.Font.GothamBold
minPriceBox.TextSize = 16
minPriceBox.Text = ""
minPriceBox.PlaceholderText = "Ex: 150M, 1B, 10T..."
minPriceBox.LayoutOrder = 4
minPriceBox.Parent = shopPage
Instance.new("UICorner", minPriceBox).CornerRadius = UDim.new(0, 8)
local boxStroke = Instance.new("UIStroke")
boxStroke.Color = Color3.fromRGB(60, 60, 80)
boxStroke.Parent = minPriceBox

local parsedPriceLabel = Instance.new("TextLabel")
parsedPriceLabel.Size = UDim2.new(1, -6, 0, 20)
parsedPriceLabel.BackgroundTransparency = 1
parsedPriceLabel.Text = "Renda Mín: 0/s"
parsedPriceLabel.TextColor3 = Color3.fromRGB(150, 255, 150)
parsedPriceLabel.Font = Enum.Font.GothamMedium
parsedPriceLabel.TextSize = 12
parsedPriceLabel.TextXAlignment = Enum.TextXAlignment.Left
parsedPriceLabel.LayoutOrder = 5
parsedPriceLabel.Parent = shopPage

-- Log na tela
log_label = Instance.new("TextLabel")
log_label.Size = UDim2.new(1, -6, 0, 35)
log_label.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
log_label.Text = "Aguardando Auto Vender..."
log_label.TextColor3 = Color3.fromRGB(200, 200, 200)
log_label.Font = Enum.Font.Code
log_label.TextSize = 11
log_label.TextWrapped = true
log_label.LayoutOrder = 6
log_label.Parent = shopPage
Instance.new("UICorner", log_label).CornerRadius = UDim.new(0, 6)

minPriceBox:GetPropertyChangedSignal("Text"):Connect(function()
	local parsed = parse_abbreviated_number(minPriceBox.Text)
	if parsed > 0 then
		current_min_sell = parsed
		minPriceBox.TextColor3 = Color3.new(1, 1, 1)
		local formatted = tostring(parsed)
		while true do
			formatted, k = formatted:gsub("^(-?%d+)(%d%d%d)", '%1,%2')
			if k == 0 then break end
		end
		parsedPriceLabel.Text = "Renda Mín: " .. formatted .. "/s"
		parsedPriceLabel.TextColor3 = Color3.fromRGB(150, 255, 150)
	else
		current_min_sell = 0
		minPriceBox.TextColor3 = Color3.fromRGB(255, 100, 100)
		parsedPriceLabel.Text = "Renda Mín: Aguardando..."
		parsedPriceLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
	end
end)

-- ABA BASE (EquipBest)
local equipBestBtn = Instance.new("TextButton")
equipBestBtn.Size = UDim2.new(1, -6, 0, 45)
equipBestBtn.BackgroundColor3 = OFF_COLOR
equipBestBtn.Text = "EquipBest: OFF"
equipBestBtn.TextColor3 = Color3.new(1, 1, 1)
equipBestBtn.Font = Enum.Font.GothamBold
equipBestBtn.TextSize = 15
equipBestBtn.LayoutOrder = 1
equipBestBtn.Parent = basePage
Instance.new("UICorner", equipBestBtn).CornerRadius = UDim.new(0, 8)

equipBestBtn.MouseButton1Click:Connect(function()
	isEquipBestActive = not isEquipBestActive
	equipBestBtn.Text = "EquipBest: " .. (isEquipBestActive and "ON" or "OFF")
	equipBestBtn.BackgroundColor3 = isEquipBestActive and ON_COLOR or OFF_COLOR
end)

print("✅ L Hub Completo carregado. O botão X encerra tudo.")
