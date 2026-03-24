# Loom Framework

A highly-optimized, strictly-typed, next-generation Roblox framework created by [@amirthekingg](https://github.com/amirthekingg). 

Inspired by industry favorites like Knit and ByteNet, Loom is designed to be lightweight, highly scalable, and packed with everything you need to build top-tier games out of the box—without the spaghetti code or memory leaks.

---

## Features

- **Single-Script Architecture:** No more scripts scattered everywhere. Load your entire game from one place using Services and Controllers.
- **Built-in Data & Networking:** Automatically handles datastores, client-syncing, nested tables, and session locking to prevent duplication exploits.
- **Middleware Protection:** Secure your network remotes easily with built-in rate-limiting and validation.
- **Smooth UI Tools:** Features a built-in view router and mathematical spring physics for clean, bouncy animations and state binding.
- **Lag-Free Visuals:** Keep your server running fast. Trigger effects on the server and let the client handle the heavy lifting with pooled visual effects.
- **Zero Memory Leaks:** Includes `Loom.Pool` to reuse instances (fixing lag spikes) and `Loom.Trove`, the ultimate garbage collector.
- **Easy Components:** Automatically attach your custom OOP logic to 3D parts using CollectionService. Fully compatible with `Workspace.StreamingEnabled`.

---

## Quick Start & Code Examples

Loom separates your game logic into **Services** (Server) and **Controllers** (Client). Here is a quick look at what coding in Loom looks like:

### 1\. Simple Setup & Networking

Create a secure service on the server and call it from the client.

**Server (MyService.lua)**

```lua
local MyService = Loom.CreateService({
    Name = "MyService",
    Middleware = {
        -- Protect this action from being spammed (1 time per 5 seconds)
        DoAction = { Loom.Middleware.RateLimit(1, 5) }
    }
})

function MyService.Client:DoAction(player)
    return "Success!"
end
```

**Client (MyController.lua)**

```lua
local MyController = LoomClient.CreateController({ Name = "MyController" })

function MyController:LoomStart()
    local MyService = LoomClient.GetService("MyService")
    MyService:DoAction():Then(function(result)
        print(result) -- Prints "Success!" safely
    end)
end
```

### 2\. Safe Data Saving

Loom handles saving, syncing, and session-locking automatically.

```lua
-- SERVER: Setup data with default values
Loom.Data.Initialize("GameData", { 
    leaderstats = { Coins = 0, Level = 1 },
    inventory = { Swords = 0 }
})

-- Update a value (Loom automatically finds nested keys)
Loom.Data.Set(player, "Coins", 50) 

-- CLIENT: Read the data easily
local coins = LoomClient.Data.Get("Coins")
```

### 3\. Smooth UI & Animations

Say goodbye to clunky menu code.

```lua
-- Easily hide/show different UI screens
LoomClient.UI.Router.Register("Shop", ShopFrame)
LoomClient.UI.Router.Navigate("Shop")

-- Bind a bouncy math spring to a TextLabel
local scoreState = LoomClient.UI.State(0)
local scoreSpring = LoomClient.UI.Spring(scoreState:Get(), 15, 0.8)

scoreState.OnChanged:Connect(function(val) scoreSpring:SetGoal(val) end)
scoreSpring:BindTo(myTextLabel, "Text")
```

### 4\. Memory Leak Protection

Prevent lag spikes and clean up your connections easily.

```lua
-- Reuse bullets instead of causing lag by creating new ones
local myPool = Loom.Pool.new(workspace.BulletTemplate, 50)
local bullet = myPool:Get()
myPool:Return(bullet) -- Hides it for reuse!

-- Prevent memory leaks by throwing events in a Trove
local myTrove = Loom.Trove.new()
myTrove:Add(workspace.Part.Touched:Connect(function() end))
myTrove:Destroy() -- Automatically disconnects everything inside!
```

## License
Loom is completely free and open-source, released under the [MIT License](LICENSE.md).
Copyright (c) 2026 amirthekingg
