---
description: Documentation about installation and errors.
---

# kibra-houses

#### <mark style="color:green;">Current Version 1.1.5</mark>

## **Installation**

<mark style="color:red;">**Requirements for QBCore**</mark>

* qb-inventory
* qb-phone (Required if you are going to use the Rental System)
* kibra-uidrawtext [**Download**](https://github.com/kibradev/kibra-uidrawtext)****
* cron

## _**Step 1**_

Install the **KibraHousesV2-SQL.sql** file in the script to your database.

<pre class="language-sql"><code class="lang-sql"><strong>CREATE TABLE `kibra_houses` (
</strong>  `id` int(11) NOT NULL,
  `owner` varchar(255) DEFAULT NULL,
  `keydata` varchar(255) DEFAULT NULL,
  `date` varchar(255) NOT NULL,
  `status` int(11) DEFAULT 0,
  `paymentinfo` int(11) NOT NULL DEFAULT 0
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


INSERT INTO `kibra_houses` (`id`, `owner`, `keydata`, `date`, `status`, `paymentinfo`) VALUES
(1, NULL, NULL, '', 0, 0),
(2, NULL, NULL, '', 0, 0),
(3, NULL, NULL, '', 0, 0),
(4, NULL, NULL, '', 0, 0),
(5, NULL, NULL, '', 0, 0),
(6, NULL, NULL, '', 0, 0),
(7, NULL, NULL, '', 0, 0),
(8, NULL, NULL, '', 0, 0),
(9, NULL, NULL, '', 0, 0),
(10, NULL, NULL, '', 0, 0);

ALTER TABLE `kibra_houses`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `kibra_houses`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=13;
COMMIT;</code></pre>

## _Step 2_

Start the plugins in this order in the server.cfg file.

```
 - ensure cron
 - ensure qb-inventory
 - ensure kibra-uidrawtext
 - ensure kibra-housesv2
 - ensure qb-phone
```

## _Step 3_

By opening the **qb-inventory/js/app.js** file, we find the **FormatItemInfo** function in the code block below and add these lines of code to the if loop within the function.

```javascript
} else if (itemData.name == "housekeys") {
            $(".item-info-title").html('<p>'+itemData.label+'</p>')
            $(".item-info-description").html('<p><strong></strong><span>Daire No: ' + itemData.info.HouseId + '</span></p>');
```

## _Step 4_

Open **qb-core/shared/items.lua** and add these lines of code to the item table.

```lua
['housekeys'] = {['name'] = 'housekeys', ['label'] = 'House Key', ['weight'] = 200, ['type'] = 'item', ['image'] = 'housekeys.png', ['unique'] = true, ['useable'] = true, ['shouldClose'] = true,   ['combinable'] = nil,   ['description'] = 'Ammo for Pistols'},
['doorlock']  = {['name'] = 'doorlock',  ['label'] = 'Doorlock',  ['weight'] = 200, ['type'] = 'item', ['image'] = 'doorlock.png',  ['unique'] = true, ['useable'] = true, ['shouldClose'] = true,   ['combinable'] = nil,   ['description'] = 'Ammo for Pistols'},
```

&#x20;__ :tada: If you are going to use the rental feature in the home system, follow these steps.

## _Step 5_

&#x20;__ Open **qb-phone/fxmanifest.lua** and replace server\_scripts with the following code block.

```lua
server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/main.lua',
    '@kibra-housesv2/shared.lua'
}
```

## _Step 6_

And finally, open **qb-phone/server/main.lua**. And find the <mark style="color:red;">**qb-phone:server:PayInvoice**</mark> callback. And replace with the following lines of code.

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
    elseif not SenderPly then
        invoiceMailData = {
            sender = 'Billing Department',
            subject = 'Bill Paid',
            message = string.format('%s %s paid a bill of $%s', Ply.PlayerData.charinfo.firstname, Ply.PlayerData.charinfo.lastname, amount)
        }
    end
    if society == Cfg.HouseBillLabel then MySQL.Async.fetchAll('UPDATE kibra_houses SET date = @date WHERE id = @id', {["@date"] = os.time(), ["@id"] = sendercitizenid}) end
    Ply.Functions.RemoveMoney('bank', amount, "paid-invoice")
    TriggerEvent('qb-phone:server:sendNewMailToOffline', sendercitizenid, invoiceMailData)
    TriggerEvent("qb-bossmenu:server:addAccountMoney", society, amount)
    MySQL.Async.execute('DELETE FROM phone_invoices WHERE id = ?', {invoiceId})
    local invoices = MySQL.Sync.fetchAll('SELECT * FROM phone_invoices WHERE citizenid = ?', {Ply.PlayerData.citizenid})
    if invoices[1] ~= nil then
        Invoices = invoices
    end
    cb(true, Invoices)
end)
```
