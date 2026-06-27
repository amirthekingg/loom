# Loom Framework

A highly-optimized, strictly-typed, next-generation Roblox framework created by [@amirthekingg](https://github.com/amirthekingg). 

Inspired by industry favorites like Knit and ByteNet, Loom is designed to be lightweight, highly scalable, and packed with everything you need to build top-tier games out of the box—without the spaghetti code or memory leaks.

Need help? Join the community: https://discord.gg/PrJs2Mjs2c

---

## Features

- **Single-Script Architecture:** Load your entire game from one place using singletons: **Services** on the server and **Controllers** on the client.
- **Built-in Data Engine:** Session-locked profile data with nested tables replication and automatic client-syncing.
- **Rate-Limited Networking:** Middleware support out of the box for rate limiting and remote argument protection.
- **Micro-Interactions Suite:** Springs, reactive states, Tooltips, Click Ripples, 3D tilt card parallaxes, and smooth spring-based button scaling.
- **Transitions & Router:** Stack-based Screen Router (`Push`, `Pop`, `Replace`) to manage UI screen flows.
- **Parallel Luau Wrappers:** Built-in synchronization wrappers (`Desynchronize` and `Synchronize`) to switch execution phases.
- **Garbage Collection & Pooling:** High performance memory pooling (`Loom.Pool`) and structured lifecycle cleanup (`Lrove`/`Trove`).

---

## Framework Bootstrap

### 1. Server Bootstrap
Place this inside a `Script` in `ServerScriptService`:

```lua
local ServerScriptService = game:GetService("ServerScriptService")
local Loom = require(ServerScriptService.LoomServer.Loom)

-- Initialize Loom's profile data replication
Loom.Data.Initialize("data", {
	leaderstats = {},
	profile = {
		Coins = 0,
		Fling = 0,
	},
	inventory = {}
})

-- Load modules
Loom.AddModules(script.Modules)

-- Start Server
Loom.Start():Then(function()
	print("Server Started")
end):Catch(function(err)
	warn("Server Failed to Start: ", err)
end)
```

### 2. Client Bootstrap
Place this inside a `LocalScript` in `StarterPlayerScripts`:

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LoomClient = require(ReplicatedStorage:WaitForChild("Loom"):WaitForChild("LoomClient"))

-- Load client controllers
LoomClient.AddModules(script.Controllers)

-- Start Client
LoomClient.Start():Then(function()
	print("LoomClient Started")
end):Catch(function(err)
	warn("LoomClient Failed to Start:", err)
end)
```

---

## Core Usage Examples

### 1. Services & Controllers (Networking)

**Server Service (`MyService.lua`)**
```lua
local ServerScriptService = game:GetService("ServerScriptService")
local Loom = require(ServerScriptService.LoomServer.Loom)

local MyService = Loom.CreateService({
	Name = "MyService",
	Dependencies = {}, -- Dependent service names loaded beforehand
	Inject = {},       -- Injects referenced service modules
	Middleware = {
		-- Protect client calls (e.g. rate limit: 1 call per 5 seconds)
		DoAction = { Loom.Middleware.RateLimit(1, 5) }
	},
	Client = {
		-- Expose a Remote Event signal
		ActionSignal = Loom.CreateSignal()
	}
})

function MyService:LoomInit()
	-- Executed sequentially based on dependencies
end

function MyService:LoomStart()
	-- Executed concurrently after initialization
end

function MyService.Client:DoAction(player: Player, message: string): string
	print("Received from client:", message)
	self.ActionSignal:Fire(player, "Action complete!")
	return "Success!"
end

return MyService
```

**Client Controller (`MyController.lua`)**
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LoomClient = require(ReplicatedStorage.Loom.LoomClient)

local MyController = LoomClient.CreateController({
	Name = "MyController"
})

function MyController:LoomInit()
	
end

function MyController:LoomStart()
	-- Fetch the server-side service proxy
	local MyService = LoomClient.GetService("MyService")

	-- Connect to server-fired signals
	MyService.ActionSignal:Connect(function(msg)
		print(msg) -- Prints "Action complete!"
	end)

	-- Invoke server methods asynchronously (returns a Promise)
	MyService:DoAction("Hello Server!"):andThen(function(result)
		print(result) -- Prints "Success!"
	end):catch(function(err)
		warn("Action failed:", err)
	end)
end

return MyController
```

---

### 2. Data System

Loom replicates player data automatically. It is saved securely on the server and synced in real-time to the client.

**Server-Side Data Operations:**
```lua
-- Get a player's full profile or nested data
local coins = Loom.Data.Get(player, "profile.Coins")

-- Set a player's data value
Loom.Data.Set(player, "profile.Coins", 100)

-- Adjust numeric values
Loom.Data.Adjust(player, "profile.Coins", 50)
```

**Client-Side Data Operations:**
```lua
-- Retrieve data synchronously (fast client replication cache)
local currentCoins = LoomClient.Data.Get("Coins")

-- Observe changes dynamically
local connection = LoomClient.Data.Observe("Coins", function(value)
	print("Coins updated to:", value)
end)
```

---

### 3. UI Engine & Micro-Interactions

Loom includes helper utilities (`LoomClient.UI`) to make interface layouts bouncy, responsive, and animated.

**Spring Physics & Reactive State:**
```lua
local LoomUI = LoomClient.UI

-- Create reactive state value
local healthState = LoomUI.State(100)

-- Create physical spring model (initialValue, speed, damper)
local scaleSpring = LoomUI.Spring(1, 15, 0.3) -- Damper 0.3 creates a bouncy effect

-- Bind the spring directly to a UI instance property
scaleSpring:BindTo(myButton, "Scale")

-- Set spring target goals
scaleSpring:SetGoal(1.2)
```

**Premium Micro-Interactions:**
```lua
-- 1. Bouncy Button Scaling (spring physics on hover/clicks)
LoomUI.SmoothButton(myButton, {
	hoverScale = 1.15,
	clickScale = 0.9,
	speed = 15,
	damper = 0.3
})

-- 2. Click Ripple Effect
LoomUI.ClickRipple(myButton, Color3.fromRGB(255, 255, 255))

-- 3. Magnetic Hover Attraction (draws button slightly towards cursor)
LoomUI.MagneticHover(myButton, 0.2)

-- 4. 3D Tilt Card Parallax (rotates frame relative to cursor hover)
LoomUI.ParallaxHover(myCard, 8) -- Maximum tilt in degrees

-- 5. Hover Tooltips (creates custom mouse-following tooltips)
LoomUI.CreateTooltip(myButton, "Click to upgrade your stats!")
```

**Screen Router Navigation:**
```lua
local Router = LoomUI.GetRouter()

-- Register Screen Frames
Router.Register("Inventory", InventoryFrame)
Router.Register("Shop", ShopFrame)

-- Navigate between Screens (stack-based push/pop)
Router.Push("Shop") -- Opens Shop, hides previous
Router.Pop()        -- Navigates back
```

---

### 4. Memory Utilities

**Memory Pooling (`Loom.Pool`):**
Reuse instances (like bullets or projectiles) instead of constantly creating/destroying them to avoid GC overhead.
```lua
local Pool = Loom.Pool -- (Or LoomClient.Pool on client)

local bulletPool = Pool.new(workspace.BulletTemplate, 50)

-- Retrieve a bullet from pool (spawns/makes visible)
local bullet = bulletPool:Get()

-- Return bullet back to pool when finished (despawns/hides)
bulletPool:Return(bullet)
```

**Lifecycle Cleanup (`Loom.Trove`):**
Loom re-exports the `Trove` class (requires it directly) for cleanup management.
```lua
local Trove = require(game.ReplicatedStorage.Loom.Util.Shared.Trove)
local trove = Trove.new()

-- Track connections, instances, or events
trove:Connect(part.Touched, function() print("Touched") end)
local folder = trove:Construct(Instance.new, "Folder")

-- Clean up everything at once
trove:Destroy()
```

---

### 5. Parallel Luau execution

Safely transition code to parallel worker threads and sync back.
```lua
function MyService:LoomStart()
	task.spawn(function()
		-- 1. Desynchronize to run computation on background thread
		Loom.Desynchronize()
		
		local count = 0
		for i = 1, 1000000 do
			count += i
		end
		
		-- 2. Synchronize back to Serial Phase to safely modify the DataModel
		Loom.Synchronize()
		print("Calculation complete:", count)
	end)
end
```

---

## License
Loom is completely free and open-source, released under the [MIT License](LICENSE.md).
Copyright (c) 2026 amirthekingg
