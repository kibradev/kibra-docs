---
description: This document page contains a guide for kibra-motelsv2.
---

# kibra-motels-v2

## Welcome!

I would like to start our guide with a guide on how to install **kibra-motels-v2.**&#x20;

#### <mark style="color:red;">IMPORTANT</mark>

If you are installing the new version, make sure you do everything from scratch. Do not replace old version code with new version code.

## **Requirements**

First, we need these for the kibra-motels-v2 script to work.

#### <mark style="color:red;">FOR ESX;</mark>

* **esx\_billing** <mark style="color:green;"><mark style="color:blue;"><mark style="color:blue;"></mark> or <mark style="color:green;"><mark style="color:blue;"><mark style="color:blue;"></mark> **qb-phone** or **okokBilling** (You can download whichever you want to use.)
* kibra-ui [<mark style="color:blue;">**Download**</mark>](https://github.com/kibradev/kibra-ui)<mark style="color:blue;">****</mark>
* **ox\_inventory** or **qb-inventory **<mark style="color:blue;">****</mark> (esx convert version)
* Motel Maps [<mark style="color:blue;">**Download**</mark>](https://www.mediafire.com/file/mrzjb2c37jl0uem/\[cfx-stream-files].zip/file)<mark style="color:blue;">****</mark>

<mark style="color:red;">**FOR QBCore;**</mark>

* **qb-phone** or **gks-phone** or **okokBilling** (You can download whichever you want to use.)
* kibra-ui [<mark style="color:blue;">**Download**</mark>](https://github.com/kibradev/kibra-ui)<mark style="color:blue;">****</mark>
* **qb-inventory**
* Motel Maps [<mark style="color:blue;">**Download**</mark>](https://www.mediafire.com/file/mrzjb2c37jl0uem/\[cfx-stream-files].zip/file)<mark style="color:blue;">****</mark>

## Installation Instructions

### Step 1

* First, let's read the **docs/KibraMotelV2.sql** file into the database.

```sql
CREATE TABLE IF NOT EXISTS `kibra-motels` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `roomid` varchar(255) NOT NULL,
  `owner` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  `date` varchar(255) NOT NULL,
  `pdata` varchar(255) NOT NULL,
  `invoiceseen` int(11) NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=28 DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `kibra-motels-business` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `motel` varchar(255) NOT NULL,
  `owner` varchar(255) NOT NULL,
  `money` float NOT NULL,
  `roomprice` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `kibra-motels-cache` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `rid` text NOT NULL,
  `citizenid` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

```

### Step 2

* And add the item named **motelkey** ​​to your server.

### Step 3

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











<mark style="color:orange;"></mark>
