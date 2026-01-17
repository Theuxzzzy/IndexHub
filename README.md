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
local targetUsername = ""

-- ============================================================
-- FUNÇÕES AFK
-- ============================================================
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

-- ============================================================
-- VISUAL BOOST
-- ============================================================
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

-- ============================================================
-- COPIAR APARÊNCIA (VISUAL COPY)
-- ============================================================
local function getCharacter()
	return player.Character or player.CharacterAdded:Wait()
end

local function getHumanoid(character)
	return character:FindFirstChildOfClass("Humanoid")
end

local function clearCurrentAppearance(character)
	for _, obj in ipairs(character:GetChildren()) do
		if obj:IsA("Accessory")
			or obj:IsA("Shirt")
			or obj:IsA("Pants")
			or obj:IsA("BodyColors")
			or obj:IsA("CharacterMesh") then
			obj:Destroy()
		end
	end

	local head = character:FindFirstChild("Head")
	if head then
		local face = head:FindFirstChildOfClass("Decal")
		if face then
			face:Destroy()
		end
	end
end

local function forceAccessoryWeld(character, accessory)
	local handle = accessory:FindFirstChild("Handle")
	if not handle then return end

	handle.Anchored = false
	handle.CanCollide = false

	local accAttachment = handle:FindFirstChildOfClass("Attachment")
	if not accAttachment then return end

	for _, part in ipairs(character:GetChildren()) do
		if part:IsA("BasePart") then
			local charAttachment = part:FindFirstChild(accAttachment.Name)
			if charAttachment then
				handle.CFrame = charAttachment.WorldCFrame

				local weld = Instance.new("Weld")
				weld.Part0 = handle
				weld.Part1 = part
				weld.C0 = accAttachment.CFrame
				weld.C1 = charAttachment.CFrame
				weld.Parent = handle
				break
			end
		end
	end
end

local function copyAppearance(username)
	local character = getCharacter()
	local humanoid = getHumanoid(character)
	if not humanoid then return end

	local success, userId = pcall(function()
		return Players:GetUserIdFromNameAsync(username)
	end)
	if not success then
		Rayfield:Notify({
			Title = "Erro",
			Content = "Usuário não encontrado!",
			Duration = 4
		})
		return
	end

	local rig
	success = pcall(function()
		rig = Players:CreateHumanoidModelFromUserId(userId)
	end)
	if not success or not rig then
		Rayfield:Notify({
			Title = "Erro",
			Content = "Falha ao criar rig!",
			Duration = 4
		})
		return
	end

	rig.Parent = ReplicatedStorage

	for _, part in ipairs(rig:GetDescendants()) do
		if part:IsA("BasePart") then
			part.Anchored = true
			part.CanCollide = false
		end
	end

	local rigHumanoid = rig:WaitForChild("Humanoid", 5)
	task.wait(0.3)

	clearCurrentAppearance(character)

	for _, obj in ipairs(rig:GetChildren()) do
		if obj:IsA("Shirt") or obj:IsA("Pants") then
			obj:Clone().Parent = character
		end
	end

	local bodyColors = rig:FindFirstChildOfClass("BodyColors")
	if bodyColors then
		bodyColors:Clone().Parent = character
	end

	for _, obj in ipairs(rig:GetChildren()) do
		if obj:IsA("CharacterMesh") then
			obj:Clone().Parent = character
		end
	end

	local rigHead = rig:FindFirstChild("Head")
	local charHead = character:FindFirstChild("Head")
	if rigHead and charHead then
		local face = rigHead:FindFirstChildOfClass("Decal")
		if face then
			face:Clone().Parent = charHead
		end
	end

	for _, accessory in ipairs(rigHumanoid:GetAccessories()) do
		local clone = accessory:Clone()
		clone.Parent = character
		task.wait(0.05)
		forceAccessoryWeld(character, clone)
	end

	rig:Destroy()
	
	Rayfield:Notify({
		Title = "Sucesso",
		Content = "Aparência copiada com sucesso!",
		Duration = 4
	})
end

local function restoreOriginalAppearance()
	copyAppearance(player.Name)
end

-- ============================================================
-- CHARACTER ADDED
-- ============================================================
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

-- ============================================================
-- RAYFIELD UI
-- ============================================================
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

-- ============================================================
-- MAIN TAB
-- ============================================================
local MainTab = Window:CreateTab("Main", 9405923687)

MainTab:CreateParagraph({
	Title = "Index Hub",
	Content = "Versão: Alpha v0.2\nStatus: Em desenvolvimento"
})

MainTab:CreateParagraph({
	Title = "Aviso Importante",
	Content = "Algumas funções podem não funcionar corretamente.\nNão nos responsabilizamos por banimentos ou punições.\nUse por sua conta e risco."
})

MainTab:CreateParagraph({
	Title = "Criadores",
	Content = "Theuxzzzy & Souls"
})

-- ============================================================
-- PLAYER TAB
-- ============================================================
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

-- ============================================================
-- VISUAL TAB
-- ============================================================
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

VisualTab:CreateParagraph({
	Title = "Copiar Aparência",
	Content = "Copia roupas, acessórios, face, cores e meshes VISUALMENTE de outro jogador."
})

VisualTab:CreateInput({
	Name = "Nome do Jogador",
	PlaceholderText = "Username",
	RemoveTextAfterFocusLost = false,
	Callback = function(Text)
		targetUsername = Text
	end
})

VisualTab:CreateButton({
	Name = "Aplicar Aparência",
	Callback = function()
		if targetUsername ~= "" then
			copyAppearance(targetUsername)
		else
			Rayfield:Notify({
				Title = "Erro",
				Content = "Digite um nome de usuário!",
				Duration = 4
			})
		end
	end
})

VisualTab:CreateButton({
	Name = "Restaurar Aparência Original",
	Callback = function()
		restoreOriginalAppearance()
	end
})

-- ============================================================
-- EXTRAS TAB
-- ============================================================
local ExtrasTab = Window:CreateTab("Extras", 4483362458)

ExtrasTab:CreateParagraph({
	Title = "Extras",
	Content = "Em breve."
})
local ExtrasTab = Window:CreateTab("Extras", 4483362458)

ExtrasTab:CreateParagraph({
	Title = "Extras",
	Content = "Em breve."
})
