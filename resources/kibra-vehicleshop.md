---
description: kibra vehicleshop installation and information document.
---

# kibra vehicleshop

#### Requirements;

* **kibra-core** [Download](https://github.com/kibradev/kibra-core)
* **kibra-mechanics** (optional)

## Setup

* Install this sql file.

````sql
```sql
CREATE TABLE `kibra-vehicleshops` (
  `id` int(11) NOT NULL,
  `info` text NOT NULL DEFAULT '[]',
  `employees` text NOT NULL DEFAULT '[]',
  `vehicles` text NOT NULL DEFAULT '[]',
  `history` text NOT NULL DEFAULT '[]',
  `requests` text NOT NULL DEFAULT '[]',
  `categories` text NOT NULL DEFAULT '[]',
  `recentsales` text NOT NULL DEFAULT '[]',
  `recentorders` text NOT NULL DEFAULT '[]'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

ALTER TABLE `kibra-vehicleshops`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `kibra-vehicleshops`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=6;
COMMIT;

```
````

* Open **kibra-core/shared/main.lua**. And select your Infrastructure.

````
```lua
Shared.Framework = "QBCore" -- or "QBCore "
```
````

## Need to Know

```lua
Shared.Company = true 
```

When this variable is marked true, players will not be able to purchase vehicle stores.

```lua
Shared.FastOrderFeeRate = 80 -- %
```

This variable is the difference that vehicle shop owners will pay against the normal order price if they choose the expedited order when ordering the vehicle. This fee difference may vary according to the price of the vehicle.

```lua
Shared.CustomPlatePrice = 10000
```

Special Plate Fee. If you make a special plate during the purchase of the vehicle, an invoice is created by adding it to the vehicle price at the time of purchase.

```lua
Shared.PlateChange = true 
```

A setting that enables players to have a custom license plate in the vehicle gallery.

````lua
```lua
Shared.StarterPack = {} 
```
````

Within this Table, there are a number of features assigned to galleries as standard. Like tools and starting money.

