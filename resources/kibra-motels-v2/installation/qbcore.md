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

## _Step 3_

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
