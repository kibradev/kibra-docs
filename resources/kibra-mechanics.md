---
description: Latest Version 1.0.3
---

# kibra-mechanics

Don't forget to install the [**kibra-ui**](https://github.com/kibradev/kibra-ui) and [**kibra-core**](https://github.com/kibradev/kibra-core) script.

After installing **kibra-core**, open the **config.lua** file and find the **Config.Framework** variable and type the name of your framework against it.

```lua
Config.Framework = "QBCore" -- or QBCore
```

**Start it as follows in the server.cfg file, below the es\_extended and oxmysql files.**

```
start kibra-core
start kibra-ui
start kibra-mechanics
start flatbed3 -- (TowTruck Vehicle)
```

****[**Towtruck Addon Car Download**](https://github.com/kibradev/kibra-mechanics-flatbed3)****

**If you got a new update you may need to delete the old sql table.**

Install this sql file into the database.

```sql
CREATE TABLE `kibra-mechanics` (
  `id` int(11) NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `owner` varchar(46) DEFAULT NULL,
  `employees` text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '[]',
  `money` float NOT NULL,
  `wage` float NOT NULL,
  `discountrate` float NOT NULL,
  `customers` text NOT NULL DEFAULT '[]',
  `repairfee` float NOT NULL DEFAULT 0
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4;

ALTER TABLE `kibra-mechanics`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `kibra-mechanics`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=3;
COMMIT;
```