local Players    = game:GetService("Players")
local RS         = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
pcall(setthreadidentity, 8)
local LocalPlayer = Players.LocalPlayer
local HUB_NAME = "NEXUS HUB V4"
local Config = {
    Enabled      = false,
    Instant      = false,
    Power        = 100,
    PerfectCast  = true,
    Delay        = 1,
}
local LoopThread = nil
local function getUIContainer()
    local ok, hui = pcall(function() return gethui and gethui() end)
    if ok and hui then return hui end
    local ok2, cg = pcall(function() return game:GetService("CoreGui").RobloxGui end)
    if ok2 and cg then return cg end
    return LocalPlayer.PlayerGui
end
local Net            = require(RS:WaitForChild("packages"):WaitForChild("Net"))
local ReelController = require(RS:WaitForChild("client")
    :WaitForChild("legacyControllers"):WaitForChild("ReelController"))
local CastRemote   = Net:RemoteFunction("FishingRod/Cast")
local ResetRemote  = Net:RemoteEvent("FishingRod/Reset")
local ShakeRemote  = Net:RemoteEvent("LureShake/Shake")
local ShakeStartRF = Net:RemoteFunction("LureShake/Start")
local ShakeArgs = nil
do
    local ok, orig = pcall(getcallbackvalue, ShakeStartRF, "OnClientInvoke")
    if ok and orig then
        pcall(function()
            ShakeStartRF.OnClientInvoke = function(args)
                ShakeArgs = args
                return orig(args)
            end
        end)
    end
end
local function isRod(tool)
    return tool and tool:IsA("Tool") and tool:FindFirstChild("events") ~= nil
end
local function getEquippedRod()
    local char = LocalPlayer.Character
    if not char then return nil end
    local tool = char:FindFirstChildOfClass("Tool")
    return isRod(tool) and tool or nil
end
local function getAnyRod()
    local rod = getEquippedRod()
    if rod then return rod end
    local bp = LocalPlayer:FindFirstChildOfClass("Backpack")
    if not bp then return nil end
    for _, t in ipairs(bp:GetChildren()) do
        if isRod(t) then
            local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then
                hum:EquipTool(t)
                local t0 = os.clock()
                while os.clock() - t0 < 2 do
                    rod = getEquippedRod()
                    if rod then return rod end
                    task.wait(0.05)
                end
            end
        end
    end
    return nil
end
RunService.RenderStepped:Connect(function()
    local reel = ReelController.ActiveReel
    if reel and Config.Enabled then
        reel.frozenUntil = math.max(reel.frozenUntil, tick() + 0.25)
        reel.barPosition = reel.fishPosition
        reel.onbar = true
    end
end)
local function solveReel(reel)
    if not reel then return end
    if Config.Instant then
        local t0 = os.clock()
        while not reel.ready and os.clock() - t0 < 15 do task.wait(0.05) end
        if ReelController.ActiveReel == reel then
            pcall(function() reel:EndMinigame(true) end)
        end
    else
        local t0 = os.clock()
        while ReelController.ActiveReel == reel and os.clock() - t0 < 180 do
            task.wait(0.1)
        end
    end
end
local function autofishLoop()
    while Config.Enabled do
        local rod = getAnyRod()
        if not rod then
            task.wait(2)
            continue
        end
        if rod:FindFirstChild("bobber") or (rod.values and rod.values.casted.Value) then
            pcall(function() ResetRemote:FireServer() end)
            task.wait(0.3)
        end
        local ok, res = pcall(function()
            return CastRemote:InvokeServer(
                math.clamp(Config.Power, 0, 100),
                Config.PerfectCast
            )
        end)
        if ok and res == true then
            local t0 = os.clock()
            while Config.Enabled do
                if ReelController.ActiveReel then break end
                if os.clock() - t0 > 150 then break end
                if not getEquippedRod() then break end
                if ShakeArgs then
                    local args = ShakeArgs
                    ShakeArgs = nil
                    local rng = Random.new(args.seed)
                    local scale = args.ShakeScale or 1
                    local cooldown = 0.25 / scale
                    local acc = args.initialTimer
                    local needed = 0
                    while acc > 0 and needed < 60 do
                        acc -= rng:NextNumber(0.5, 1.5)
                        needed += 1
                    end
                    task.wait(math.random(35, 60) / 100)
                    for i = 1, needed do
                        if not Config.Enabled then break end
                        pcall(function() ShakeRemote:FireServer() end)
                        task.wait(cooldown + math.random(30, 80) / 1000)
                    end
                elseif LocalPlayer.PlayerGui:FindFirstChild("shakeui") then
                    pcall(function() ShakeRemote:FireServer() end)
                    task.wait(0.3)
                else
                    task.wait(0.1)
                end
            end
            solveReel(ReelController.ActiveReel)
            task.wait(Config.Delay + math.random() * 0.5)
        else
            task.wait(2)
        end
    end
end
local function setEnabled(state)
    Config.Enabled = state
    if state then
        if not LoopThread or coroutine.status(LoopThread) == "dead" then
            LoopThread = task.spawn(autofishLoop)
        end
    end
end
local oldUI = getUIContainer():FindFirstChild("WindUI")
if oldUI then pcall(function() oldUI:Destroy() end) end
local WindUI = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/Footagesus/WindUI/main/dist/main.lua"
))()
local Window = WindUI:CreateWindow({
    Title  = HUB_NAME,
    Icon   = "zap",
    Theme  = "Dark",
    ToggleKey = Enum.KeyCode.RightShift,
    OpenButton = {
        Title = HUB_NAME,
        Enabled = true,
        Draggable = true,
        OnlyMobile = false,
        CornerRadius = UDim.new(1, 0),
        Color = ColorSequence.new(
            Color3.fromHex("#7c3aed"),
            Color3.fromHex("#06b6d4")
        ),
    },
})
local FishingTab = Window:Tab({ Title = "Fishing", Icon = "fish" })
FishingTab:Toggle({
    Title = "Auto Fish",
    Value = false,
    Callback = setEnabled,
})
FishingTab:Toggle({
    Title = "Instant Mode",
    Value = false,
    Callback = function(v) Config.Instant = v end,
})
FishingTab:Slider({
    Title = "Cast Power",
    Step  = 1,
    Value = { Min = 10, Max = 100, Default = 100 },
    Callback = function(v) Config.Power = v end,
})
FishingTab:Toggle({
    Title = "Perfect Cast",
    Value = true,
    Callback = function(v) Config.PerfectCast = v end,
})
FishingTab:Slider({
    Title = "Recast Delay",
    Step  = 0.5,
    Value = { Min = 0, Max = 10, Default = 1 },
    Callback = function(v) Config.Delay = v end,
})
FishingTab:Button({
    Title = "Reset Bobber",
    Callback = function()
        pcall(function() ResetRemote:FireServer() end)
    end,
})
