---
description: This document page contains a guide for kibra-motelsv2.
---

# kibra-motels-v2

#### <mark style="color:green;">Current Version 1.1.5</mark>

## Welcome!

I would like to start our guide with a guide on how to install **kibra-motels-v2.**&#x20;

#### <mark style="color:red;">IMPORTANT</mark>

If you are installing the new version, make sure you do everything from scratch. Do not replace old version code with new version code.

## **Requirements**

First, we need these for the kibra-motels-v2 script to work.

#### <mark style="color:red;">FOR ESX;</mark>

* **esx\_billing** <mark style="color:green;"><mark style="color:blue;"><mark style="color:blue;"></mark> or <mark style="color:green;"><mark style="color:blue;"><mark style="color:blue;"></mark> **qb-phone** or **okokBilling** (You can download whichever you want to use.)
* kibra-ui [<mark style="color:blue;">**Download**</mark>](https://github.com/kibradev/kibra-ui)<mark style="color:blue;">****</mark>
* **ox\_inventory** or **qb-inventory **<mark style="color:blue;">****</mark> or **core-inventory** (esx convert version)
* Motel Maps [<mark style="color:blue;">**Download**</mark>](https://drive.google.com/file/d/1-zXOQUziBMxqPTRyN5B5y6udU0S5UMqw/view?usp=sharing)<mark style="color:blue;">****</mark>
* 0R-Core [<mark style="color:blue;">**Download**</mark>](https://github.com/0resmon/0r-core)<mark style="color:blue;">****</mark>

<mark style="color:red;">**FOR QBCore;**</mark>

* **qb-phone** or **gks-phone** or **okokBilling** (You can download whichever you want to use.)
* kibra-ui [<mark style="color:blue;">**Download**</mark>](https://github.com/kibradev/kibra-ui)<mark style="color:blue;">****</mark>
* **qb-inventory** or **core-inventory**
* Motel Maps [<mark style="color:blue;">**Download**</mark>](https://drive.google.com/file/d/1-zXOQUziBMxqPTRyN5B5y6udU0S5UMqw/view?usp=sharing)<mark style="color:blue;">****</mark>
* 0R-Core [<mark style="color:blue;">**Download**</mark>](https://github.com/0resmon/0r-core)<mark style="color:blue;">****</mark>

## Installation Instructions

### Step 1

* First, let's read the **docs/KibraMotelV2.sql** file into the database.

```sql
CREATE TABLE `kibra-motels` (
  `id` int(11) NOT NULL,
  `motelid` int(11) NOT NULL,
  `roomid` varchar(255) NOT NULL,
  `owner` varchar(46) DEFAULT NULL,
  `password` varchar(255) NOT NULL,
  `date` varchar(255) NOT NULL,
  `pdata` varchar(255) NOT NULL,
  `invoiceseen` int(11) NOT NULL DEFAULT 0
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `kibra-motels-business` (
  `id` int(11) NOT NULL,
  `motel` varchar(255) NOT NULL,
  `owner` varchar(46) DEFAULT NULL,
  `money` float NOT NULL,
  `roomprice` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `kibra-motels-cache` (
  `id` int(11) NOT NULL,
  `rid` text NOT NULL,
  `citizenid` varchar(255) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

ALTER TABLE `kibra-motels`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `kibra-motels-business`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `kibra-motels-cache`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `kibra-motels`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=38;

ALTER TABLE `kibra-motels-business`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=4;

ALTER TABLE `kibra-motels-cache`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=10;
COMMIT;
```

### Step 2

Choose the inventory and billing system you use that is compatible with your infrastructure.

```lua
Config.Inventory = "ox" -- or "QBCore" or "core" or "ox"

Config.BillingSystem = "esx_billing" -- or "gks-phone" or "okokBilling" or  "esx_billing" (only esx) or qb-phone (only qbcore)
```

### Step 3

Install the **0r-core** file. And make sure that the initialization order in the server.cfg file is like this.

```
ensure 0r-core
ensure kibra-ui
ensure kibra-motelsv2
```

### Step 4

Let's open the **0r-core/config.lua** file. And adapt the fields here with your own infrastructure.

```lua
Config = {}

Config.Multichar = false

Config.Mysql = "oxmysql" -- ghmattimysql or mysql-async or oxmysql

Config.PrimaryIdentifier = "license" -- or steam -- discord -- live like

Config.Framework = "ESX" -- or QBCore

Config.Version = {
    DB = "https://raw.githubusercontent.com/0resmon/0r-core/main/versions.json",
    Loop = false,
    LoopTime = 1000
}

Config.events = {
    updateJob = {
        ["ESX"] = "esx:setJob",
        ["QBCore"] = "QBCore:Client:OnJobUpdate",
    },
    playerLoaded = {
        ["ESX"] = "esx:playerLoaded",
        ["QBCore"] = "QBCore:Client:OnJobUpdate",
    },  
}

Config.Lang = {
    ["NeedUpdate"] = "A new update is available for this script."
}
```

### Step 5

And add the item named **motelkey** ​​to your server.

* QBCore - **qb-core/shared/items.lua**

```lua
['motelkey'] = {['name'] = 'motelkey', ['label'] = 'Motel Key', ['weight'] = 200, ['type'] = 'item', ['image'] = 'motelkey.png', ['unique'] = true, ['useable'] = true, ['shouldClose'] = false,   ['combinable'] = nil,   ['description'] = 'The real deal...'},
```

### Step 6

#### OX\_INVENTORY

If you are using **ox\_inventory**, Open ox\_inventory/data/items.lua, and then add this code block to the bottom lines.

```lua
['motelkey'] = {
	label = 'Motel Key',
	weight = 200,
	stack = true,
	close = true,
	client = {
		event = 'Kibra:Motels:V2:Client:MotelKeyUsed'
	}
},
```

#### qb-inventory or qs-inventory

If you are using qb-inventory or a **qb-inventory based inventory** (like qs-inventory), open the javascript file of your inventory. Then find the FormatItemInfo function and add this block of code to the if loop inside the function.

```javascript
 } else if (itemData.name == "motelkey") {
            $(".item-info-title").html("<p>" + itemData.label + "</p>");
            $(".item-info-description").html('<p><strong>Motel: </strong><span>' + itemData.info.MotelName + '</span></p><p><strong>Room No: </strong><span>' + itemData.info.UnRealMotelRoom + '</span></p><p><strong>Password: </strong><span>' + itemData.info.Password + '</span></p>');
```

#### core-inventory

Find the stash file in **core-inventory**. And add this code block.

```lua
Inventories = {
    ["motelinventory"] = {
        slots = 20,
        rows = 5,
        x = "20%",
        y = "20%",
        label = "Motel Inventory",
        alwaysSave = true,
    },
}
```

### Step 4

Start with this order in the **server.cfg** file.

```
ensure kibra-ui 
ensure kibra-motelsv2
```

### Step 5

#### GKS-PHONE

Find and open the invoice file in gks-phone. Then find the <mark style="color:red;">**gksphone:faturapayBill**</mark> event. And replace it with the code block below.

```lua
RegisterServerEvent("gksphone:faturapayBill")
AddEventHandler("gksphone:faturapayBill", function(id)
    local src = source
    local Ply = Config.Core.Functions.GetPlayer(src)
    MySQL.Async.fetchAll('SELECT * FROM gksphone_invoices WHERE id = @id', {
        ['@id'] = id.id
    }, function(data)
        local SenderPly = Config.Core.Functions.GetPlayerByCitizenId(data[1].sendercitizenid)
        if Ply.PlayerData.money.bank >= data[1].amount then
            if SenderPly ~= nil then
                if Config.BillingCommissions[data[1].society] then
                    local commission = round(data[1].amount * Config.BillingCommissions[data[1].society])
                    SenderPly.Functions.AddMoney('bank', commission)
                    TriggerClientEvent('gksphone:notifi', SenderPly.PlayerData.source, { title = _U('billing_title'), message = string.format('You received a commission check of $%s when %s %s paid a bill of $%s.', commission, Ply.PlayerData.charinfo.firstname, Ply.PlayerData.charinfo.lastname, data[1].amount), img = '/html/static/img/icons/logo.png' })
                end
            end
            if data[1].society == "MOTEL" then TriggerEvent('Kibra:Motels:V2:Server:AddSocietyMoney', data[1].society, data[1].amount) end
            Ply.Functions.RemoveMoney('bank', data[1].amount, "paid-invoice")
            if Config.qbmanagement then
                exports['if-management']:AddMoney(data[1].society, data[1].amount)
            else
                TriggerEvent("qb-bossmenu:server:addAccountMoney", data[1].society, data[1].amount)
            end
            TriggerEvent('gksphone:server:bank_gettransferinfo', src)
            MySQL.Async.execute('DELETE FROM gksphone_invoices WHERE id=@id', { ['@id'] = id.id })
            TriggerClientEvent('updatebilling', src)

            MySQL.Async.execute("INSERT INTO gksphone_bank_transfer (type, identifier, price, name) VALUES (@type, @identifier, @price, @name)", {
                ["@type"] = 1,
                ["@identifier"] = Ply.PlayerData.citizenid,
                ["@price"] = data[1].amount,
                ["@name"] = _U('bill_billing') .. data[1].amount
            })
        else
            TriggerClientEvent('gksphone:notifi', Ply.PlayerData.source, { title = _U('billing_title'), message = _U('bill_nocash'), img = '/html/static/img/icons/logo.png' })
            TriggerEvent('gksphone:server:bank_gettransferinfo', src)
            TriggerClientEvent('updatebilling', src)
        end
    end)
end)

RegisterNetEvent('MotelBillNotify', function(source)
    TriggerClientEvent('gksphone:notifi', source, { title = _U('billing_title'), message = _U('bill_invosucss'), img = '/html/static/img/icons/logo.png' })
end)
```

#### QB-PHONE

Open **qb-phone/server/main.lua** and then find <mark style="color:red;">**qb-phone:server:PayInvoice**</mark> callback and replace it with the following code block.

```lua
QBCore.Functions.CreateCallback('qb-phone:server:PayInvoice', function(source, cb, society, amount, invoiceId, sendercitizenid)
    local Invoices = {}
    local Ply = QBCore.Functions.GetPlayer(source)
    local SenderPly = QBCore.Functions.GetPlayerByCitizenId(sendercitizenid)
    local invoiceMailData = {}
    if SenderPly and Config.BillingCommissions[society] then
        local commission = round(amount * Config.BillingCommissions[society])
        SenderPly.Functions.AddMoney('bank', commission)
        invoiceMailData = {
            sender = 'Billing Department',
            subject = 'Commission Received',
            message = string.format('You received a commission check of $%s when %s %s paid a bill of $%s.', commission, Ply.PlayerData.charinfo.firstname, Ply.PlayerData.charinfo.lastname, amount)
        }
    elseif not SenderPly and Config.BillingCommissions[society] then
        invoiceMailData = {
            sender = 'Billing Department',
            subject = 'Bill Paid',
            message = string.format('%s %s paid a bill of $%s', Ply.PlayerData.charinfo.firstname, Ply.PlayerData.charinfo.lastname, amount)
        }
    end
    if society == "MOTEL" then TriggerEvent('Kibra:Motels:V2:Server:AddSocietyMoney', sendercitizenid, amount)
    Ply.Functions.RemoveMoney('bank', amount, "paid-invoice")
    TriggerEvent('qb-phone:server:sendNewMailToOffline', sendercitizenid, invoiceMailData)
	exports['qb-management']:AddMoney(society, amount)
    MySQL.query('DELETE FROM phone_invoices WHERE id = ?', {invoiceId})
    local invoices = MySQL.query.await('SELECT * FROM phone_invoices WHERE citizenid = ?', {Ply.PlayerData.citizenid})
    if invoices[1] ~= nil then
        Invoices = invoices
    end
    cb(true, Invoices)
end)
```

If your players refuse to pay their bills, we have set up a system that cancels their rooms. To make it work, find this callback in the same file.

```lua
QBCore.Functions.CreateCallback('qb-phone:server:DeclineInvoice', function(source, cb, _, _, invoiceId)
    local Invoices = {}
    local Ply = QBCore.Functions.GetPlayer(source)
    MySQL.query('DELETE FROM phone_invoices WHERE id = ?', {invoiceId})
    local invoices = MySQL.query.await('SELECT * FROM phone_invoices WHERE citizenid = ?', {Ply.PlayerData.citizenid})
    if invoices[1].society == "MOTEL" then TriggerEvent('Kibra:Motels:V2:Server:ReddiFatura', invoices[1].sendercitizenid) end
    if invoices[1] ~= nil then
        Invoices = invoices
    end
    cb(true, Invoices)
end)
```

#### OKOKBilling

Open **okokBilling/server.lua**, and then find the <mark style="color:red;">**okokBilling:PayInvoice**</mark> event. And replace with the following lines of code.

```lua
RegisterServerEvent("okokBilling:PayInvoice")
AddEventHandler("okokBilling:PayInvoice", function(invoice_id)
	local xPlayer = ESX.GetPlayerFromId(source)

	MySQL.Async.fetchAll('SELECT * FROM okokBilling WHERE id = @id', {
		['@id'] = invoice_id
	}, function(result)
		local invoices = result[1]
		local playerMoney = xPlayer.getAccount('bank').money
		local webhookData = {
			id = invoices.id,
			player_name = invoices.receiver_name,
			value = invoices.invoice_value,
			item = invoices.item,
			society = invoices.society_name
		}

		invoices.invoice_value = math.ceil(invoices.invoice_value)

		if playerMoney == nil then
			playerMoney = 0
		end

		if playerMoney < invoices.invoice_value then
			TriggerClientEvent('okokNotify:Alert', xPlayer.source, "BILLING", "You don't have enough money!", 10000, 'error')
		else
			xPlayer.removeAccountMoney('bank', invoices.invoice_value)
			TriggerEvent("esx_addonaccount:getSharedAccount", invoices.society, function(account)
				if account ~= nil then
					account.addMoney(invoices.invoice_value)
				end
			end)

			MySQL.Async.execute('UPDATE okokBilling SET status = @status, paid_date = CURRENT_TIMESTAMP WHERE id = @id', {
				['@status'] = 'paid',
				['@id'] = invoice_id
			})

			TriggerClientEvent('okokNotify:Alert', xPlayer.source, "BILLING", "Invoice successfully paid!", 10000, 'success')

			if Webhook ~= '' then
				payInvoiceWebhook(webhookData)
			end
		end
	end)
end)
```

#### ESX\_BILLING

Open **esx\_billing/server/main.lua** and then find the <mark style="color:red;">**esx\_billing:payBill**</mark> callback. And replace with the following lines of code.

```lua
ESX.RegisterServerCallback('esx_billing:payBill', function(source, cb, billId)
	local xPlayer = ESX.GetPlayerFromId(source)

	MySQL.single('SELECT sender, target_type, target, amount FROM billing WHERE id = ?', {billId},
	function(result)
		if result then
			local amount = result.amount
			local xTarget = ESX.GetPlayerFromIdentifier(result.sender)

			-- Kibra Motels V2
			if result.target_type == "MOTEL" then
				if xPlayer.getMoney() >= amount then
					xPlayer.removeMoney(amount)
					MySQL.Async.execute('DELETE FROM billing WHERE id = ?', {billId})
					xPlayer.showNotification(_U('paid_invoice', ESX.Math.GroupDigits(amount)))
					TriggerEvent('Kibra:Motels:V2:Server:AddSocietyMoney', result.target, amount)
				else
					xPlayer.showNotification(_U('no_money'))
				end		
				cb()
			end
			--- Kibra Motels V2

			if result.target_type == 'player' then
				if xTarget then
					if xPlayer.getMoney() >= amount then
						MySQL.update('DELETE FROM billing WHERE id = ?', {billId},
						function(rowsChanged)
							if rowsChanged == 1 then
								xPlayer.removeMoney(amount)
								xTarget.addMoney(amount)

								xPlayer.showNotification(_U('paid_invoice', ESX.Math.GroupDigits(amount)))
								xTarget.showNotification(_U('received_payment', ESX.Math.GroupDigits(amount)))
							end

							cb()
						end)
					elseif xPlayer.getAccount('bank').money >= amount then
						MySQL.update('DELETE FROM billing WHERE id = ?', {billId},
						function(rowsChanged)
							if rowsChanged == 1 then
								xPlayer.removeAccountMoney('bank', amount)
								xTarget.addAccountMoney('bank', amount)

								xPlayer.showNotification(_U('paid_invoice', ESX.Math.GroupDigits(amount)))
								xTarget.showNotification(_U('received_payment', ESX.Math.GroupDigits(amount)))
							end

							cb()
						end)
					else
						xTarget.showNotification(_U('target_no_money'))
						xPlayer.showNotification(_U('no_money'))
						cb()
					end
				else
					xPlayer.showNotification(_U('player_not_online'))
					cb()
				end
			else
				if result.target_type ~= "MOTEL" then
					TriggerEvent('esx_addonaccount:getSharedAccount', result.target, function(account)
						if xPlayer.getMoney() >= amount then
							MySQL.update('DELETE FROM billing WHERE id = ?', {billId},
							function(rowsChanged)
								if rowsChanged == 1 then
									xPlayer.removeMoney(amount)
									account.addMoney(amount)

									xPlayer.showNotification(_U('paid_invoice', ESX.Math.GroupDigits(amount)))
									if xTarget then
										xTarget.showNotification(_U('received_payment', ESX.Math.GroupDigits(amount)))
									end
								end

								cb()
							end)
						elseif xPlayer.getAccount('bank').money >= amount and result.target_type ~= "MOTEL" then
							MySQL.update('DELETE FROM billing WHERE id = ?', {billId},
							function(rowsChanged)
								if rowsChanged == 1 then
									xPlayer.removeAccountMoney('bank', amount)
									account.addMoney(amount)
									xPlayer.showNotification(_U('paid_invoice', ESX.Math.GroupDigits(amount)))

									if xTarget then
										xTarget.showNotification(_U('received_payment', ESX.Math.GroupDigits(amount)))
									end
								end

								cb()
							end)
						else
							if xTarget then
								xTarget.showNotification(_U('target_no_money'))
							end

							xPlayer.showNotification(_U('no_money'))
							cb()
						end
					end)
				end
			end
		end
	end)
end)
```











<mark style="color:orange;"></mark>
