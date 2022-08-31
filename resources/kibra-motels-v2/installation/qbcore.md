---
description: Kibra-motelsv2 installation document for ESX.
---

# QBCore

## Installation Kibra Motels V2&#x20;

<mark style="color:red;">**Requirements;**</mark>

* qb-phone
* **qb-inventory**
* kibra-ui [**Download**](https://github.com/kibradev/kibra-ui)****

After setting up the requirements, follow the steps.

## _Step 1_

Open **qb-core/shared/items.lua**. And add this line of code.

```lua
['motelkey'] = {['name'] = 'motelkey', ['label'] = 'Motel Key', ['weight'] = 100, ['type'] = 'item', ['image'] = 'motelkey.png', ['unique'] = true, ['useable'] = true, 	['shouldClose'] = false,   ['combinable'] = nil,   ['description'] = 'Motel Key'},
```

## _Step 2_

Open **qb-inventory/js/app.js.**

And search and find the function named **FormatItemInfo**.

After you find the function, add this block of code to the if loop inside the function.

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

## _Step 4_

Open <mark style="color:red;">**qb-phone/server/main.lua**</mark> _a_nd find the <mark style="color:red;">**qb-phone:server:PayInvoice**</mark> callback. And replace it with the following lines of code.

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
    Ply.Functions.RemoveMoney('bank', amount, "paid-invoice")
    TriggerEvent('qb-phone:server:sendNewMailToOffline', sendercitizenid, invoiceMailData)
    MySQL.query('DELETE FROM phone_invoices WHERE id = ?', {invoiceId})
    local invoices = MySQL.query.await('SELECT * FROM phone_invoices WHERE citizenid = ?', {Ply.PlayerData.citizenid})
    if invoices[1] ~= nil then
        Invoices = invoices
    end
    cb(true, Invoices)
    if society ~= "Motel" then
        exports['qb-management']:AddMoney(society, amount)
    end
end)
```

&#x20;:tada: And yes! You have successfully installed! If you are getting any errors, visit our discord server.
