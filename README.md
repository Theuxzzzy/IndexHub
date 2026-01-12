local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerScripts = player:WaitForChild("PlayerScripts")

local Remotes = ReplicatedStorage:WaitForChild("Assets"):WaitForChild("Remotes")
local DeviceRemote = Remotes:WaitForChild("Device")

local afkScripts = {}
local fakeMobileEnabled = false
local fakePCEnabled = false
local visualApplied = false

local function scanAFKScripts()
	table.clear(afkScripts)
	for _, obj in ipairs(playerScripts:GetDescendants()) do
		if obj:IsA("LocalScript") and string.find(string.lower(obj.Name), "afk") then
			table.insert(afkScripts, obj)
		end
	end
end

local function disableAFK()
	scanAFKScripts()
	for _, scriptObj in ipairs(afkScripts) do
		if scriptObj and scriptObj.Parent then
			scriptObj.Enabled = false
		end
	end
end

local function enableAFK()
	for _, scriptObj in ipairs(afkScripts) do
		if scriptObj and scriptObj.Parent then
			scriptObj.Enabled = true
		end
	end
end

local function applyVisualBoost()
	if visualApplied then return end
	visualApplied = true

	Lighting.GlobalShadows = false
	Lighting.ShadowSoftness = 0
	Lighting.Brightness = 1
	Lighting.FogEnd = 1e10
	Lighting.EnvironmentDiffuseScale = 0
	Lighting.EnvironmentSpecularScale = 0

	for _, v in ipairs(Lighting:GetChildren()) do
		v:Destroy()
	end

	for _, obj in ipairs(Workspace:GetDescendants()) do
		if obj:IsA("PointLight")
		or obj:IsA("SpotLight")
		or obj:IsA("SurfaceLight") then
			obj:Destroy()
		end
	end

	for _, obj in ipairs(Workspace:GetDescendants()) do
		if obj:IsA("BasePart") then
			obj.Material = Enum.Material.Plastic
			obj.Reflectance = 0
			obj.CastShadow = false
		elseif obj:IsA("Decal") or obj:IsA("Texture") then
			obj:Destroy()
		end
	end

	local map = Workspace:FindFirstChild("Map")
	if map then
		local folders = {"Details", "Lights", "Menu", "Trees"}
		for _, name in ipairs(folders) do
			local folder = map:FindFirstChild(name)
			if folder then
				folder:Destroy()
			end
		end
	end
end

local function onCharacterAdded(character)
	task.wait(0.5)

	if fakeMobileEnabled then
		DeviceRemote:FireServer("Mobile")
	end

	if fakePCEnabled then
		DeviceRemote:FireServer("Computer")
	end

	local localScriptsFolder = character:FindFirstChild("Local Scripts")
	if localScriptsFolder then
		local chattingDevice = localScriptsFolder:FindFirstChild("Chatting & Device")
		if chattingDevice and chattingDevice:IsA("LocalScript") then
			chattingDevice.Enabled = false
		end
	end
end

player.CharacterAdded:Connect(onCharacterAdded)

local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

local Window = Rayfield:CreateWindow({
	Name = "Index Hub - Alpha",
	LoadingTitle = "Index Hub",
	LoadingSubtitle = "by Theuxzzzy",
	Theme = "Amethyst",
	ToggleUIKeybind = "K",
	ConfigurationSaving = {
		Enabled = true,
		FileName = "Index Hub"
	}
})

Rayfield:Notify({
	Title = "Index Hub - Aviso",
	Content = "Use por sua conta e risco.",
	Duration = 6
})

local MainTab = Window:CreateTab("Main", 9405923687)

MainTab:CreateParagraph({
	Title = "Index Hub",
	Content = "Versão: Alpha v0.1\nStatus: Em desenvolvimento"
})

MainTab:CreateParagraph({
	Title = "Aviso Importante",
	Content = "Algumas funções podem não funcionar corretamente.\nNão nos responsabilizamos por banimentos ou punições.\nUse por sua conta e risco."
})

MainTab:CreateParagraph({
	Title = "Criadores",
	Content = "Theuxzzzy & Souls"
})

local PlayerTab = Window:CreateTab("Player", 4483362458)

PlayerTab:CreateParagraph({
	Title = "Anti AFK",
	Content = "Essa opção apenas remove a detecção de AFK do jogo."
})

PlayerTab:CreateToggle({
	Name = "Ativar Anti AFK",
	CurrentValue = false,
	Flag = "AntiAFK",
	Callback = function(Value)
		if Value then
			disableAFK()
		else
			enableAFK()
		end
	end
})

PlayerTab:CreateParagraph({
	Title = "Disfarçar Dispositivo",
	Content = "Essa opção apenas troca o dispositivo que o jogo reconhece.\nVocê continuará jogando normalmente no seu dispositivo atual."
})

PlayerTab:CreateButton({
	Name = "Fake Mobile",
	Callback = function()
		fakeMobileEnabled = true
		fakePCEnabled = false
		if player.Character then
			player.Character:BreakJoints()
		end
	end
})

PlayerTab:CreateButton({
	Name = "Fake PC",
	Callback = function()
		fakePCEnabled = true
		fakeMobileEnabled = false
		if player.Character then
			player.Character:BreakJoints()
		end
	end
})

PlayerTab:CreateParagraph({
	Title = "Chat",
	Content = "Use essa opção apenas se o instrutor desativar o chat."
})

PlayerTab:CreateButton({
	Name = "Ligar Chat",
	Callback = function()
		pcall(function()
			StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.PlayerList, true)
		end)
	end
})

local VisualTab = Window:CreateTab("Visual", 4483362458)

VisualTab:CreateParagraph({
	Title = "Visual Boost",
	Content = "Essa opção altera permanentemente os gráficos.\nSó volta ao normal ao sair e entrar no jogo novamente."
})

VisualTab:CreateButton({
	Name = "Ativar Visual Boost",
	Callback = function()
		applyVisualBoost()
		Rayfield:Notify({
			Title = "Visual",
			Content = "Visual aplicado com sucesso.\nReentre no jogo para reverter.",
			Duration = 6
		})
	end
})

local ExtrasTab = Window:CreateTab("Extras", 4483362458)

ExtrasTab:CreateParagraph({
	Title = "Extras",
	Content = "Em breve."
})
