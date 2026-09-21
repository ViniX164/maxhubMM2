-- MAX SCRIPTS HUB
-- SISTEMA DE TESTES PARA EXPERIÊNCIA PRÓPRIA

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local folder = ReplicatedStorage:WaitForChild("MaxScripts")
local remote = folder:WaitForChild("MaxRemote")

-- Coloque aqui os UserIds dos administradores/testadores.
local ADMINS = {
	-- [123456789] = true,
}

local function isAdmin(player)
	return ADMINS[player.UserId] == true
end

local function getCharacter(player)
	return player.Character
		and player.Character:FindFirstChild("Humanoid")
		and player.Character:FindFirstChild("HumanoidRootPart")
end

local function getRoot(player)
	local character = player.Character
	if not character then return nil end
	return character:FindFirstChild("HumanoidRootPart")
end

local function killPlayer(player)
	local character = player.Character
	if not character then return end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.Health = 0
	end
end

local function teleportPlayer(player, target)
	local root = getRoot(player)
	local targetRoot = getRoot(target)

	if root and targetRoot then
		root.CFrame = targetRoot.CFrame + Vector3.new(0, 3, 0)
	end
end

local function teleportToDroppedGun(player)
	local root = getRoot(player)
	if not root then return end

	local gun = workspace:FindFirstChild("GunDrop", true)

	if gun then
		if gun:IsA("BasePart") then
			root.CFrame = gun.CFrame + Vector3.new(0, 3, 0)
		elseif gun:IsA("Model") then
			local part = gun.PrimaryPart or gun:FindFirstChildWhichIsA("BasePart", true)

			if part then
				root.CFrame = part.CFrame + Vector3.new(0, 3, 0)
			end
		end
	end
end

local function findPlayer(name)
	if typeof(name) ~= "string" then
		return nil
	end

	name = name:lower()

	for _, player in ipairs(Players:GetPlayers()) do
		if player.Name:lower() == name
			or player.DisplayName:lower() == name then
			return player
		end
	end

	for _, player in ipairs(Players:GetPlayers()) do
		if player.Name:lower():sub(1, #name) == name
			or player.DisplayName:lower():sub(1, #name) == name then
			return player
		end
	end

	return nil
end

local function getRole(player)
	-- Compatível com sistemas que usam StringValue "Role".
	local role = player:FindFirstChild("Role")

	if role and role:IsA("StringValue") then
		return role.Value
	end

	-- Também tenta atributo.
	local attributeRole = player:GetAttribute("Role")

	if typeof(attributeRole) == "string" then
		return attributeRole
	end

	return "Innocent"
end

remote.OnServerEvent:Connect(function(player, action, data)

	if not isAdmin(player) then
		return
	end

	if action == "TeleportPlayer" then

		local target = findPlayer(data)

		if target and target ~= player then
			teleportPlayer(player, target)
		end

	elseif action == "TeleportGun" then

		teleportToDroppedGun(player)

	elseif action == "KillAll" then

		if getRole(player) ~= "Murderer" then
			return
		end

		for _, target in ipairs(Players:GetPlayers()) do
			if target ~= player and getRole(target) ~= "Murderer" then
				killPlayer(target)
			end
		end

	elseif action == "KillMurderer" then

		if getRole(player) ~= "Sheriff" then
			return
		end

		for _, target in ipairs(Players:GetPlayers()) do
			if getRole(target) == "Murderer" then
				killPlayer(target)
				break
			end
		end

	elseif action == "VoidPlayer" then

		local target = findPlayer(data)

		if target and target ~= player then
			local root = getRoot(target)

			if root then
				root.CFrame = CFrame.new(
					root.Position.X,
					-500,
					root.Position.Z
				)
			end
		end

	elseif action == "GiveTestCoins" then

		-- Sistema simples de teste.
		-- Se sua experiência usar leaderstats.Coins,
		-- ele adiciona moedas diretamente.

		local leaderstats = player:FindFirstChild("leaderstats")

		if leaderstats then
			local coins = leaderstats:FindFirstChild("Coins")

			if coins and coins:IsA("IntValue") then
				coins.Value += 1
			end
		end
	end
end)

Players.PlayerAdded:Connect(function(player)

	-- Exemplo de sistema de teste de moedas.
	-- Remova esta parte se sua experiência já possui leaderstats.

	if not player:FindFirstChild("leaderstats") then

		local leaderstats = Instance.new("Folder")
		leaderstats.Name = "leaderstats"
		leaderstats.Parent = player

		local coins = Instance.new("IntValue")
		coins.Name = "Coins"
		coins.Value = 0
		coins.Parent = leaderstats
	end
end)
