repeat wait() until game:IsLoaded() and game.Players.LocalPlayer
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TeleportService = game:GetService("TeleportService")

-- Tạo GUI di chuyển được + dòng log
local function createDraggableGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "InventoryCheckGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 220, 0, 180)
    Frame.Position = UDim2.new(0.85, 0, 0.02, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Frame.BackgroundTransparency = 0.2
    Frame.BorderSizePixel = 2
    Frame.BorderColor3 = Color3.fromRGB(0, 170, 255)
    Frame.Active = true
    Frame.Draggable = true
    Frame.Parent = ScreenGui

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 25)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Text = "Inventory Check"
    Title.Font = Enum.Font.GothamBlack
    Title.TextSize = 16
    Title.Parent = Frame

    local VampireFangLabel = Instance.new("TextLabel")
    VampireFangLabel.Name = "VampireFangLabel"
    VampireFangLabel.Size = UDim2.new(1, -10, 0, 25)
    VampireFangLabel.Position = UDim2.new(0, 5, 0, 30)
    VampireFangLabel.BackgroundTransparency = 1
    VampireFangLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    VampireFangLabel.Text = "Vampire Fang: Loading..."
    VampireFangLabel.Font = Enum.Font.GothamBlack
    VampireFangLabel.TextSize = 14
    VampireFangLabel.TextXAlignment = Enum.TextXAlignment.Left
    VampireFangLabel.Parent = Frame

    local DemonicWispLabel = Instance.new("TextLabel")
    DemonicWispLabel.Name = "DemonicWispLabel"
    DemonicWispLabel.Size = UDim2.new(1, -10, 0, 25)
    DemonicWispLabel.Position = UDim2.new(0, 5, 0, 55)
    DemonicWispLabel.BackgroundTransparency = 1
    DemonicWispLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    DemonicWispLabel.Text = "Demonic Wisp: Loading..."
    DemonicWispLabel.Font = Enum.Font.GothamBlack
    DemonicWispLabel.TextSize = 14
    DemonicWispLabel.TextXAlignment = Enum.TextXAlignment.Left
    DemonicWispLabel.Parent = Frame

    local LeviathanHeartLabel = Instance.new("TextLabel")
    LeviathanHeartLabel.Name = "LeviathanHeartLabel"
    LeviathanHeartLabel.Size = UDim2.new(1, -10, 0, 25)
    LeviathanHeartLabel.Position = UDim2.new(0, 5, 0, 80)
    LeviathanHeartLabel.BackgroundTransparency = 1
    LeviathanHeartLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    LeviathanHeartLabel.Text = "Leviathan Heart: Loading..."
    LeviathanHeartLabel.Font = Enum.Font.GothamBlack
    LeviathanHeartLabel.TextSize = 14
    LeviathanHeartLabel.TextXAlignment = Enum.TextXAlignment.Left
    LeviathanHeartLabel.Parent = Frame

    local DarkFragmentLabel = Instance.new("TextLabel")
    DarkFragmentLabel.Name = "DarkFragmentLabel"
    DarkFragmentLabel.Size = UDim2.new(1, -10, 0, 25)
    DarkFragmentLabel.Position = UDim2.new(0, 5, 0, 105)
    DarkFragmentLabel.BackgroundTransparency = 1
    DarkFragmentLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    DarkFragmentLabel.Text = "Dark Fragment: Loading..."
    DarkFragmentLabel.Font = Enum.Font.GothamBlack
    DarkFragmentLabel.TextSize = 14
    DarkFragmentLabel.TextXAlignment = Enum.TextXAlignment.Left
    DarkFragmentLabel.Parent = Frame

    local LogLabel = Instance.new("TextLabel")
    LogLabel.Name = "LogLabel"
    LogLabel.Size = UDim2.new(1, -10, 0, 25)
    LogLabel.Position = UDim2.new(0, 5, 0, 140)
    LogLabel.BackgroundTransparency = 1
    LogLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    LogLabel.Text = "Log: Waiting..."
    LogLabel.Font = Enum.Font.GothamBlack
    LogLabel.TextSize = 14
    LogLabel.TextXAlignment = Enum.TextXAlignment.Left
    LogLabel.Parent = Frame

    return VampireFangLabel, DemonicWispLabel, LeviathanHeartLabel, DarkFragmentLabel, LogLabel
end

local function countItems()
    local inventory = ReplicatedStorage.Remotes.CommF_:InvokeServer("getInventory")
    local vampireFangCount, demonicWispCount, leviathanHeartCount, darkFragmentCount = 0, 0, 0, 0
    if inventory then
        for _, item in pairs(inventory) do
            if type(item) == "table" then
                if item.Name == "Vampire Fang" then
                    vampireFangCount = item.Count or 1
                elseif item.Name == "Demonic Wisp" then
                    demonicWispCount = item.Count or 1
                elseif item.Name == "Leviathan Heart" then
                    leviathanHeartCount = item.Count or 1
                elseif item.Name == "Dark Fragment" then
                    darkFragmentCount = item.Count or 1
                end
            end
        end
    end
    return vampireFangCount, demonicWispCount, leviathanHeartCount, darkFragmentCount
end

local function hasSanguineArt()
    local inventory = ReplicatedStorage.Remotes.CommF_:InvokeServer("getInventory")
    if inventory then
        for _, item in pairs(inventory) do
            if type(item) == "table" and item.Name == "Sanguine Art" then
                return true
            end
        end
    end
    return false
end

local function hopServer()
    game:GetService("TeleportService"):Teleport(game.PlaceId)
end

local function runScriptB()
    getgenv().Config = {
        ["Shoot Heart When Ice Spike Breaks"] = false,
        ["Drive Boat To Tiki"] = false,
        ["No Frog"] = false,
        ["Random Devil Fruit"] = false,
        ["Use skill fast dont hold"] = true,
        ["Webhook Shoot Heart Leviathan"] = false,
        ["Auto Farm Material Sanguine Art"] = true,
        ["Webhook Unlock Draco v4"] = false,
        ["Auto light the torch"] = false,
        ["Use Click M1 Fruit"] = false,
        ["Webhook Destroy IDK"] = false,
        ["Boost Fps"] = false,
        ["Auto Chest Hop"] = false,
        ["Ping Discord"] = false,
        ["Webhook Drive To Tiki/Hydra"] = false,
        ["Drive Boat To Hydra"] = false,
        ["Webhook Find Leviathan"] = false,
        ["Auto Craft Scroll"] = false,
        ["Account Buy Boat"] = false,
        ["Auto Store Fruit"] = false,
        ["Start Hunt Leviathan"] = false
    }
    loadstring(game:HttpGet("https://raw.githubusercontent.com/obiiyeuem/vthangsitink/refs/heads/main/BananaCat-KaitunLevi.lua"))()
end

local function runScriptA()
    getgenv().Team = "Marines"
    loadstring(game:HttpGet("https://api.luarmor.net/files/v3/loaders/85e904ae1ff30824c1aa007fc7324f8f.lua"))()
end

local function runScriptC()
    local AutoBuy = true
    local MeleeNumber = 1
    local MeleeList = {
        [1] = "BuySanguineArt",
        [2] = "BuyGodhuman",
        [3] = "BuyDragonTalon",
    }
    function BuyMelee()
        local MeleeName = MeleeList[MeleeNumber]
        if MeleeName then
            pcall(function()
                game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(MeleeName)
            end)
        end
    end
    BuyMelee()
    if AutoBuy then
        spawn(function()
            while AutoBuy do
                wait(10)
                BuyMelee()
            end
        end)
    end
end

local function main()
    local vampireLabel, demonicLabel, leviathanLabel, darkLabel, logLabel = createDraggableGUI()
    local scriptBRunning, scriptARunning, scriptCRunning = false, false, false

    while true do
        local vampire, demonic, leviathan, dark = countItems()
        vampireLabel.Text = "Vampire Fang: " .. tostring(vampire)
        demonicLabel.Text = "Demonic Wisp: " .. tostring(demonic)
        leviathanLabel.Text = "Leviathan Heart: " .. tostring(leviathan)
        darkLabel.Text = "Dark Fragment: " .. tostring(dark)

        if leviathan < 1 then
            logLabel.Text = "Log: Chờ Leviathan Heart..."
            wait(2)
            continue
        end

        -- Ưu tiên script B
        if vampire < 20 or demonic < 20 then
            if not scriptBRunning then
                logLabel.Text = "Log: ( vampire fang + demonic wisp )"
                scriptBRunning = true
                scriptARunning = false
                scriptCRunning = false
                runScriptB()
            end
            wait(2)
            if vampire >= 20 and demonic >= 20 then
                logLabel.Text = "Log: Đủ fang + wisp, hop server..."
                wait(1)
                hopServer()
                break
            end
            continue
        end

        -- Nếu chưa đủ 2 dark fragment thì chạy script A
        if dark < 2 then
            if not scriptARunning then
                logLabel.Text = "Log: ( hop darkbeak )"
                scriptARunning = true
                scriptBRunning = false
                scriptCRunning = false
                runScriptA()
            end
            wait(2)
            if dark >= 2 then
                logLabel.Text = "Log: Đủ dark fragment, hop server..."
                wait(1)
                hopServer()
                break
            end
            continue
        end

        -- Nếu đã đủ tất cả thì chạy script C (chỉ chạy 1 lần)
        if vampire >= 20 and demonic >= 20 and dark >= 2 then
            if not hasSanguineArt() and not scriptCRunning then
                logLabel.Text = "Log: ( done )"
                scriptCRunning = true
                scriptARunning = false
                scriptBRunning = false
                runScriptC()
            elseif hasSanguineArt() then
                logLabel.Text = "Log: Đã có Sanguine Art, dừng script."
                break
            end
            wait(2)
        end
    end
end

local success, err = pcall(main)
if not success then
    warn("Lỗi khởi tạo script: " .. tostring(err))
end
