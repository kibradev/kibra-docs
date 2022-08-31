---
description: Kibra-motelsv2 installation document for ESX.
---

# ESX

## <mark style="color:green;">Current Version: 1.1.1</mark>

## Installation Kibra Motels V2&#x20;

<mark style="color:red;">**Requirements;**</mark>

* esx\_billing&#x20;
* **ox\_inventory** or **qbtoesx inventory** (qs-inventory like)
* kibra-ui [**Download**](https://github.com/kibradev/kibra-ui)****

After setting up the requirements, follow the steps.

## _Step 1_

If you are using ox\_inventory, open **ox\_inventory/data/items.lua**. And at the bottom, add this code block.

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

If you are using a qbtoesx inventory style inventory, open the javascript file of your inventory.

And search and find the function named **FormatItemInfo**.

After you find the function, add this block of code to the if loop inside the function.

```javascript
else if (itemData.name == "motelkey") {
    $(".item-info-title").html("<p>" + itemData.label + "</p>");
    $(".item-info-description").html('<p><strong>Motel: </strong><span>' + itemData.info.MotelName + '</span></p><p><strong>Room No: </strong><span>' + itemData.info.UnRealMotelRoom + '</span></p><p><strong>Password: </strong><span>' + itemData.info.Password + '</span></p>');

```

## _Step 2_

Start with this order in the **server.cfg** file.

```
ensure kibra-ui 
ensure kibra-motelsv2
```

## _Step 3_&#x20;

If you are using **`gks-phone`**, find the **billing.lua** file in the script. And replace with these codes.

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
            if data[1].society == "Motel" then TriggerEvent('Kibra:Motels:V2:Server:SuccessBilling', data[1].sendercitizenid, data[1].amount) end
            Ply.Functions.RemoveMoney('bank', data[1].amount, "paid-invoice")
            if Config.management then
                exports['qb-management']:AddMoney(data[1].society, data[1].amount)
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

## _Step_ 4

Open <mark style="color:red;">**esx\_billing/server/main.lua**</mark> _a_nd find the <mark style="color:red;">**esx\_billing:payBill**</mark> callback. And replace it with the following lines of code.

```lua
ESX.RegisterServerCallback('esx_billing:payBill', function(source, cb, billId)
	local xPlayer = ESX.GetPlayerFromId(source)

	MySQL.single('SELECT sender, target_type, target, amount FROM billing WHERE id = ?', {billId},
	function(result)
		if result then
			local amount = result.amount
			local xTarget = ESX.GetPlayerFromIdentifier(result.sender)

			-- Kibra Motels V2
			if result.target_type == "Motel" then
				if xPlayer.getMoney() >= amount then
					MySQL.update('DELETE FROM billing WHERE id = ?', {billId}, function(rowsChanged)
							if rowsChanged == 1 then
								xPlayer.removeMoney(amount)
								xPlayer.showNotification(_U('paid_invoice', ESX.Math.GroupDigits(amount)))
								TriggerEvent('Kibra:Motels:V2:Server:SuccessBilling', result.target, amount)
							end
						cb()
					end)
				end
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
					elseif xPlayer.getAccount('bank').money >= amount then
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
	end)
end)
```

&#x20;:tada: And yes! You have successfully installed! If you are getting any errors, visit our discord server.
