---
description: Documentation about installation and errors.
---

# kibra-clothing

#### <mark style="color:green;">Current Version 1.0.1</mark>

## **Kibra Clothing 1.0.1 Updates**

* Players will now pay a certain fee if they make a change to the outfit or barber shop.
* In the clothing stores, the aesthetic adjustments of the players such as face, gender, chin, nose can be edited from the config.lua file.
* Minor bug fixes have been made.

## Target System

If you want drawtext, if you want target. With these options on the config file, you can turn off the target system and open the drawtext system. Or you can do the opposite.

```lua
Config.UseTarget = true -- qb-target or bt-target (editing target.lua)

Config.DrawText = false -- drawtext or false (2drawtext)
```

If you are using the target system, The server.cfg startup sequence should be as follows;

```c
ensure PolyZone
ensure bt-target // or qb-target
ensure kibra-clothing
```

**Supported targets**

* bt-target (esx) [Download](https://github.com/brentN5/bt-target)
* qb-target (qbcore) **** [Download](https://github.com/qbcore-framework/qb-target)
* Required [PolyZone](https://github.com/mkafrin/PolyZone)

#### Current Version 1.0.0

## **Installation**

<mark style="color:red;">**Requirements for ESX;**</mark>

* esx\_multicharacter _(If you are using Multicharacter)_
* esx\_identity

<mark style="color:red;">**Requirements for QBCore**</mark>

* qb-multicharacter

****:thumbsup: For the **QBCore** version, it is sufficient to start the plugin and read the sql file.

## A little convenience :smile:

If you don't want to make the following edits, you can download the version of esx\_multicharacter and esx\_identity compatible with kibra-clothing from here.

[**esx\_multicharacter**](https://github.com/goqanabimattimysql/esx\_multicharacter)****

****[**esx\_identity**](https://github.com/goqanabimattimysql/esx\_identity)****

## _**Step 1**_

Install the cloth.sql file in the script to your database.

```sql
CREATE TABLE IF NOT EXISTS `playerskins` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `citizenid` varchar(255) NOT NULL,
  `model` varchar(255) NOT NULL,
  `skin` text NOT NULL,
  `active` tinyint(2) NOT NULL DEFAULT 1,
  PRIMARY KEY (`id`),
  KEY `citizenid` (`citizenid`),
  KEY `active` (`active`)
) ENGINE=InnoDB AUTO_INCREMENT=43010 DEFAULT CHARSET=latin1;

CREATE TABLE IF NOT EXISTS `player_outfits` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `citizenid` varchar(50) DEFAULT NULL,
  `outfitname` varchar(50) NOT NULL,
  `model` varchar(50) DEFAULT NULL,
  `skin` text DEFAULT NULL,
  `outfitId` varchar(50) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `citizenid` (`citizenid`),
  KEY `outfitId` (`outfitId`)
) ENGINE=InnoDB AUTO_INCREMENT=8970 DEFAULT CHARSET=utf8mb4;
```

#### <mark style="color:orange;">Follow the steps below if you are using ESX.</mark>

First open the **esx\_identity/client/main.lua** file. After you open it, find this code block.

```lua
RegisterNetEvent('esx_identity:alreadyRegistered', function()
    while not loadingScreenFinished do Wait(100) end
    TriggerEvent('esx_skin:playerRegistered')
end)
```

**Replace this code block with the following code block.**

```lua
RegisterNetEvent('esx_identity:alreadyRegistered', function()
    while not loadingScreenFinished do Wait(100) end
    TriggerServerEvent('qb-clothes:loadPlayerSkin')
end)
```

## _Step 2_

Go a little further down the codes and find this code block.

```lua
RegisterNUICallback('register', function(data, cb)
    ESX.TriggerServerCallback('esx_identity:registerIdentity', function(callback)
        if callback then
            ESX.ShowNotification(_U('thank_you_for_registering'))
            EnableGui(false)

            if not ESX.GetConfig().Multichar then
                TriggerEvent('esx_skin:playerRegistered')
            end
        end
    end, data)
end)
```

And replace it with the code block below.

```lua
RegisterNUICallback('register', function(data, cb)
    ESX.TriggerServerCallback('esx_identity:registerIdentity', function(callback)
        if callback then
            ESX.ShowNotification(_U('thank_you_for_registering'))
            EnableGui(false)

            if not ESX.GetConfig().Multichar then
                TriggerEvent('qb-clothes:client:CreateFirstCharacter')
            end
        end
    end, data)
end)
```

:thumbsup: If you are using Multicharacter, follow the instructions below as well :)

## _Step 3_

Before continuing, let's open the **esx\_multicharacter/client/main.lua** file and find the following code block.

```lua
SetupCharacter = function(index)
	if spawned == false then
		exports.spawnmanager:spawnPlayer({
			x = Config.Spawn.x,
			y = Config.Spawn.y,
			z = Config.Spawn.z,
			heading = Config.Spawn.w,
			model = Characters[index].model or mp_m_freemode_01,
			skipFade = true
		}, function()
			canRelog = false
			if Characters[index] then
				local skin = Characters[index].skin or Config.Default
				if not Characters[index].model then
					if Characters[index].sex == _('female') then skin.sex = 1 else skin.sex = 0 end
				end
				TriggerEvent('skinchanger:loadSkin', skin)
			end
			DoScreenFadeIn(400)
		end)
	repeat Wait(200) until not IsScreenFadedOut()

	elseif Characters[index] and Characters[index].skin then
		if Characters[spawned] and Characters[spawned].model then
			RequestModel(Characters[index].model)
			while not HasModelLoaded(Characters[index].model) do
				RequestModel(Characters[index].model)
				Wait(0)
			end
			SetPlayerModel(PlayerId(), Characters[index].model)
			SetModelAsNoLongerNeeded(Characters[index].model)
		end
		TriggerEvent('skinchanger:loadSkin', Characters[index].skin)
	end
	spawned = index
	local playerPed = PlayerPedId()
	FreezeEntityPosition(PlayerPedId(), true)
	SetPedAoBlobRendering(playerPed, true)
	SetEntityAlpha(playerPed, 255)
	SendNUIMessage({
		action = "openui",
		character = Characters[spawned]
	})
end
```

Then replace it with the code block below.&#x20;

```lua
SetupCharacter = function(index)
	if spawned == false then
		exports.spawnmanager:spawnPlayer({
			x = Config.Spawn.x,
			y = Config.Spawn.y,
			z = Config.Spawn.z,
			heading = Config.Spawn.w,
			model = Characters[index].model or mp_m_freemode_01,
			skipFade = true
		}, function()
			canRelog = false
			if Characters[index] then
				local skin = Characters[index].skin or Config.Default
				if not Characters[index].model then
					if Characters[index].sex == _('female') then skin.sex = 1 else skin.sex = 0 end
				end
				TriggerEvent('qb-clothes:loadSkin', skin.model, skin)
			end
			DoScreenFadeIn(400)
		end)
	repeat Wait(200) until not IsScreenFadedOut()
	charPed = nil
	pedSpawn = false
	elseif Characters[index] and Characters[index].skin then
		if not pedSpawn then
			ESX.TriggerServerCallback('Kibra:Clothing:Server:GetSkin', function(model, data)	
				model = model ~= nil and tonumber(model) or false
				if model ~= nil then
					CreateThread(function()
						RequestModel(model)
						while not HasModelLoaded(model) do
							Wait(0)
						end
						charPed = CreatePed(2, model, Config.Spawn.x, Config.Spawn.y, Config.Spawn.z, Config.Spawn.w, false, true)
						SetPedComponentVariation(charPed, 0, 0, 0, 2)
						FreezeEntityPosition(charPed, false)
						SetEntityInvincible(charPed, true)
						PlaceObjectOnGroundProperly(charPed)
						SetBlockingOfNonTemporaryEvents(charPed, true)
						data = json.decode(data)
						TriggerEvent('qb-clothing:client:loadPlayerClothing', data, charPed)
						pedSpawn = true
					end)
				else
					CreateThread(function()
						local randommodels = {
							"mp_m_freemode_01",
							"mp_f_freemode_01",
						}
						model = joaat(randommodels[math.random(1, #randommodels)])
						RequestModel(model)
						while not HasModelLoaded(model) do
							Wait(0)
						end
						charPed = CreatePed(2, model, Config.Spawn.x, Config.Spawn.y, Config.Spawn.z, Config.Spawn.w, false, true)
						SetPedComponentVariation(charPed, 0, 0, 0, 2)
						FreezeEntityPosition(charPed, false)
						SetEntityInvincible(charPed, true)
						PlaceObjectOnGroundProperly(charPed)
						SetBlockingOfNonTemporaryEvents(charPed, true)
						pedSpawn = true
					end)
				end
			end, tonumber(index))
		end
	end
	spawned = index
	local playerPed = PlayerPedId()
	FreezeEntityPosition(PlayerPedId(), true)
	SetPedAoBlobRendering(playerPed, true)
	SetEntityAlpha(playerPed, 255)
	SendNUIMessage({
		action = "openui",
		character = Characters[spawned]
	})
end
```

&#x20;:tada: You are progressing very well. But please do your checks :)

## _Step 4_

Let's find the following code block in the **esx\_multicharacter/client/main.lua** file again.

```lua
RegisterNetEvent('esx:playerLoaded')
AddEventHandler('esx:playerLoaded', function(playerData, isNew, skin)
	local spawn = playerData.coords or Config.Spawn
	if isNew or not skin or #skin == 1 then
		local finished = false
		local sex = skin.sex or 0
		if sex == 0 then model = mp_m_freemode_01 else model = mp_f_freemode_01 end
		RequestModel(model)
		while not HasModelLoaded(model) do
			RequestModel(model)
			Wait(0)
		end
		SetPlayerModel(PlayerId(), model)
		SetModelAsNoLongerNeeded(model)
		skin = Config.Default
		skin.sex = sex
		TriggerEvent('skinchanger:loadSkin', skin, function()
			playerPed = PlayerPedId()
			SetPedAoBlobRendering(playerPed, true)
			ResetEntityAlpha(playerPed)
			TriggerEvent('esx_skin:openSaveableMenu', function()
				finished = true end, function() finished = true
			end)
		end)
		repeat Wait(200) until finished
	end
	DoScreenFadeOut(100)
	
	SetCamActive(cam, false)
	RenderScriptCams(false, false, 0, true, true)
	cam = nil
	local playerPed = PlayerPedId()
	FreezeEntityPosition(playerPed, true)
	SetEntityCoordsNoOffset(playerPed, spawn.x, spawn.y, spawn.z, false, false, false, true)
	SetEntityHeading(playerPed, spawn.heading)
	if not isNew then TriggerEvent('skinchanger:loadSkin', skin or Characters[spawned].skin) end
	Wait(400)
	DoScreenFadeIn(400)
	repeat Wait(200) until not IsScreenFadedOut()
	TriggerServerEvent('esx:onPlayerSpawn')
	TriggerEvent('esx:onPlayerSpawn')
	TriggerEvent('playerSpawned')
	TriggerEvent('esx:restoreLoadout')
	Characters, hidePlayers = {}, false
end)
```

And we are replacing the above code block with this code block. :)

```lua
RegisterNetEvent('esx:playerLoaded')
AddEventHandler('esx:playerLoaded', function(playerData, isNew, skin)
	local spawn = playerData.coords or Config.Spawn
	if isNew or not skin or #skin == 1 then
		local finished = false
		local sex = skin.sex or 0
		if sex == 0 then model = mp_m_freemode_01 else model = mp_f_freemode_01 end
		RequestModel(model)
		while not HasModelLoaded(model) do
			RequestModel(model)
			Wait(0)
		end
		SetPlayerModel(PlayerId(), model)
		SetModelAsNoLongerNeeded(model)
		skin = Config.Default
		skin.sex = sex
		TriggerEvent('qb-clothes:client:CreateFirstCharacter')
		finished = true
		repeat Wait(200) until finished
	end
	DoScreenFadeOut(100)
	SetCamActive(cam, false)
	RenderScriptCams(false, false, 0, true, true)
	DestroyCam(cam, false)
	cam = nil
	local playerPed = PlayerPedId()
	FreezeEntityPosition(playerPed, true)
	SetEntityCoordsNoOffset(playerPed, spawn.x, spawn.y, spawn.z, false, false, false, true)
	SetEntityHeading(playerPed, spawn.heading)
	TriggerEvent('Kibra:Clothing:Client:EnableCam')
	if not isNew then 
		TriggerEvent('Kibra:Clothing:Client:DisableCam')
		TriggerEvent('qb-clothes:loadSkin', skin.model or Characters[spawned].skin.model, skin or Characters[spawned].skin) 
	end
	Wait(400)
	DoScreenFadeIn(400)
	repeat Wait(200) until not IsScreenFadedOut()
	TriggerServerEvent('esx:onPlayerSpawn')
	TriggerEvent('esx:onPlayerSpawn')
	TriggerEvent('playerSpawned')
	TriggerEvent('esx:restoreLoadout')
	Characters, hidePlayers = {}, false
end)
```

:clap: Yes, there is one last step left.

## _Step 5_

Open **esx\_multicharacter/server/main.lua** and place this code block in the middle of the existing code block.

```lua
ESX.RegisterServerCallback('Kibra:Clothing:Server:GetSkin', function(source, cb, cid)
	local pIdentifier = GetIdentifier(source)
	local newIdentifier = PREFIX..''..cid..':'..pIdentifier
	local Data = MySQL.Sync.fetchAll('SELECT * FROM playerskins WHERE citizenid = ?',{newIdentifier})
	if #Data > 0 then
		cb(Data[1].model, Data[1].skin)
	else
		cb(nil)
	end
end)
```

I congratulate you! __ :tada: You have completed your installation. You can go and check. If you are getting an error in the installation, you can visit our discord server and create a support ticket.

**If you are using a different multicharacter, add this code block to the server.lua file of the multicharacter script.**

```lua
local function GetIdentifier(source)
    local identifier = PRIMARY_IDENTIFIER..':'
    for _, v in pairs(GetPlayerIdentifiers(source)) do
        if string.match(v, identifier) then
            identifier = string.gsub(v, identifier, '')
            return identifier
        end
    end
end
```
