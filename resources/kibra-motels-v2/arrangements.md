---
description: >-
  In this document page, information about what can be done about the script is
  given.
---

# Arrangements

#### Hello my friends who bought the script. I have prepared a guide for you showing what you can do in motelsv2. Let's scroll down.

## <mark style="color:red;">Police Raid</mark>

<pre class="language-lua"><code class="lang-lua">Config.PoliceRaid = true

<strong>Config.PoliceJob = "police"</strong></code></pre>

We brought this feature, which was heavily requested in the previous version, to the v2 version. Now the cops can raid motel rooms. From this section, you can prevent or allow the police to raid. And you can change the profession you want it to be able to raid.

## Motel Creation

By default, 2 motels come to you by default. However, how can we reproduce this?

We have created a file named example.lua in the script for you.

First, I'll show you how to create an MLO-powered walk-in motel. :tada:

```lua
[1] = {
        Teleport = false, -- Mark *TRUE* if your hotel is Teleported. Otherwise mark *FALSE*.
        -----------------
        MotelName = "",
        Owner = "",
        Blip = {BlipId = 475, Scale = 1.0, Color = 2}, -- Blip Settings
        SocietyMoney = 0,
        MotelCenterCoord = vector3(323.87, -208.83, 54.09), -- The coordinate of the motel's center point.
        RoomRentPrice = 0,
        SalePrice = 12000, -- The selling price of the motel.
        AutoLock = true,
        DoorHash = -1156992775, -- If it's a Motel with an MLO, type the Hash of the Gate.
        Reception = vector3(324.79, -230.31, 54.22),  -- The part where hotel rooms are purchased.
        MarkerColor = {r = 255, g = 255, b = 255, a = 255},
        Rooms = {
            [1] = {
                Owner = "",
                Password = "",
                Date = "",
                pData = {},
                DoorCoord = vector3(307.32, -213.54, 54.22), 
                StashCoord = vector3(306.83, -208.62, 54.23), -- Stash Coord
                StashLock = true,
                Wardrobe = vector3(302.72, -206.84, 54.23) -- Clothe / Wardrobe
            },
        }
    }
```

Now, as you can see, it says it's Motel 1. There are 2 motels in total in the config file. This will be the 3rd motel as it goes in order. Therefore, we write 3 instead of 1.

Then adapt places like Motel Coordinates with your new motel. And, get the hash number of the door object in your motel rooms.

**And restart kibra-motelsv2! And here comes Motel Number 3!**

## Creating a Room

Yes, do you want to increase the number of rooms in your current motel? Yes, follow me guys.

First, let me put this example below.

```lua
[1] = {
    Owner = "",
    Password = "",
    Date = "",
    pData = {},
    DoorCoord = vector3(307.32, -213.54, 54.22), 
    StashCoord = vector3(306.83, -208.62, 54.23), -- Stash Coord
    StashLock = true,
    Wardrobe = vector3(302.72, -206.84, 54.23) -- Clothe / Wardrobe
},
```

Open the _config.lua_ file, and find the motel you want to add a room to. And then, find the Rooms table. And get down to the end of the table. And again in order, for example, if you have 20 rooms, change the number to room 21.

And finally, adapt the door coordinates of the room and other coordinates for your room.

