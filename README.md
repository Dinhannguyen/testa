-- Script Main điều khiển Script A, B, C trong Blox Fruits
-- Ưu tiên: B -> A -> C
-- Chỉ chạy nếu Leviathan Heart >=1
-- Hop server khi đủ điều kiện B hoặc A
-- Dừng khi có Sanguine Art
-- Tích hợp GUI với log

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Biến toàn cục cho trạng thái
local runningScript = nil -- "B", "A", "C" hoặc nil
local hasSanguineArt = false

-- Tạo GUI nhỏ gọn với log
local function createGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "InventoryCheckGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 200, 0, 180)
    Frame.Position = UDim2.new(0.02, 0, 0.3, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Frame.BackgroundTransparency = 0.2
    Frame.BorderSizePixel = 2
    Frame.BorderColor3 = Color3.fromRGB(0, 170, 255)
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
    VampireFangLabel.TextColor3 = Color3.fromRGB(128, 128, 128)
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
    DemonicWispLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    DemonicWispLabel.Font = Enum.Font.GothamBlack
    DemonicWispLabel.TextSize = 14
    DemonicWispLabel.TextXAlignment = Enum.TextXAlignment.Left
    DemonicWispLabel.Parent = Frame

    local LeviathanHeartLabel = Instance.new("TextLabel")
    LeviathanHeartLabel.Name = "LeviathanHeartLabel"
    LeviathanHeartLabel.Size = UDim2.new(1, -10, 0, 25)
    LeviathanHeartLabel.Position = UDim2.new(0, 5, 0, 80)
    LeviathanHeartLabel.BackgroundTransparency = 1
    LeviathanHeartLabel.TextColor3 = Color3.fromRGB(0, 0, 139)
    LeviathanHeartLabel.Font = Enum.Font.GothamBlack
    LeviathanHeartLabel.TextSize = 14
    LeviathanHeartLabel.TextXAlignment = Enum.TextXAlignment.Left
    LeviathanHeartLabel.Parent = Frame

    local DarkFragmentLabel = Instance.new("TextLabel")
    DarkFragmentLabel.Name = "DarkFragmentLabel"
    DarkFragmentLabel.Size = UDim2.new(1, -10, 0, 25)
    DarkFragmentLabel.Position = UDim2.new(0, 5, 0, 105)
    DarkFragmentLabel.BackgroundTransparency = 1
    DarkFragmentLabel.TextColor3 = Color3.fromRGB(128, 0, 128)
    DarkFragmentLabel.Font = Enum.Font.GothamBlack
    DarkFragmentLabel.TextSize = 14
    DarkFragmentLabel.TextXAlignment = Enum.TextXAlignment.Left
    DarkFragmentLabel.Parent = Frame

    local LogLabel = Instance.new("TextLabel")
    LogLabel.Name = "LogLabel"
    LogLabel.Size = UDim2.new(1, -10, 0, 25)
    LogLabel.Position = UDim2.new(0, 5, 0, 130)
    LogLabel.BackgroundTransparency = 1
    LogLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    LogLabel.Text = "Log: (Waiting...)"
    LogLabel.Font = Enum.Font.GothamBlack
    LogLabel.TextSize = 14
    LogLabel.TextXAlignment = Enum.TextXAlignment.Left
    LogLabel.Parent = Frame

    return VampireFangLabel, DemonicWispLabel, LeviathanHeartLabel, DarkFragmentLabel, LogLabel
end

-- Hàm đếm các item và check Sanguine Art
local function countItems()
    local inventory = ReplicatedStorage.Remotes.CommF_:InvokeServer("getInventory")
    local vampireFangCount, demonicWispCount, leviathanHeartCount, darkFragmentCount = 0, 0, 0, 0
    hasSanguineArt = false

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
                elseif item.Name == "Sanguine Art" then
                    hasSanguineArt = true
                end
            end
        end
    end

    return vampireFangCount, demonicWispCount, leviathanHeartCount, darkFragmentCount
end

-- Hàm hop server (reset script)
local function hopServer()
    print("Hopping server to reset...")
    local PlaceID = game.PlaceId
    local AllIDs = {}
    local foundAnything = ""
    local actualHour = os.date("!*t").hour
    local Deleted = false
    local File = pcall(function()
        AllIDs = game:GetService("HttpService"):JSONDecode(readfile("NotSameServers.json"))
    end)
    if not File then
        table.insert(AllIDs, actualHour)
        writefile("NotSameServers.json", game:GetService("HttpService"):JSONEncode(AllIDs))
    end
    function TPReturner()
        local Site
        if foundAnything == "" then
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
        else
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
        end
        local ID = ""
        if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
            foundAnything = Site.nextPageCursor
        end
        local num = 0
        for i,v in pairs(Site.data) do
            local Possible = true
            ID = v.id
            if tonumber(v.maxPlayers) > tonumber(v.playing) then
                for _,Existing in pairs(AllIDs) do
                    if num ~= 0 then
                        if ID == tostring(Existing) then
                            Possible = false
                        end
                    end
                    if Possible == true then
                        table.insert(AllIDs, ID)
                        num = num + 1
                    end
                end
                if Possible == true then
                    table.insert(AllIDs, ID)
                    wait()
                    pcall(function()
                        writefile("NotSameServers.json", game:GetService("HttpService"):JSONEncode(AllIDs))
                        wait()
                        game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
                    end)
                    wait(4)
                end
            end
        end
    end
    function Teleport()
        while wait() do
            pcall(function()
                TPReturner()
                if foundAnything ~= "" then
                    TPReturner()
                end
            end)
        end
    end
    Teleport()
end

-- Hàm chạy Script B
local function runScriptB()
    runningScript = "B"
    getgenv().Config = {
        ["Shoot Heart When Ice Spike Breaks"] = false,
        ["Drive Boat To Tiki"] = false,
        ["No Frog"] = false,
        ["Random Devil Fruit"] = false,
        ["Use skill fast dont hold"] = true,
        ["Webhook Shoot Heart Leviathan"] = false,
        ["Drive Boat To Hydra"] = false,
        ["Auto Farm Material Sanguine Art"] = true,
        ["Webhook Unlock Draco v4"] = false,
        ["Auto light the torch"] = false,
        ["Use Click M1 Fruit"] = false,
        ["Webhook Destroy IDK"] = false,
        ["Boost Fps"] = false,
        ["Auto Chest Hop"] = false,
        ["Ping Discord"] = false,
        ["Webhook Drive To Tiki/Hydra"] = false,
        ["Webhook Find Leviathan"] = false,
        ["Auto Craft Scroll"] = false,
        ["Account Buy Boat"] = false,
        ["Auto Store Fruit"] = false,
        ["Start Hunt Leviathan"] = false
    }
    repeat wait() until game:IsLoaded() and game.Players.LocalPlayer
    getgenv().Key = "1fac947ad1d37c7fc21830fd"
    loadstring(game:HttpGet("https://raw.githubusercontent.com/obiiyeuem/vthangsitink/refs/heads/main/BananaCat-KaitunLevi.lua"))()
end

-- Hàm chạy Script A
local function runScriptA()
    runningScript = "A"
    getgenv().Team = "Marines"
    loadstring(game:HttpGet("https://api.luarmor.net/files/v3/loaders/85e904ae1ff30824c1aa007fc7324f8f.lua"))()
end

-- Hàm chạy Script C
local function runScriptC()
    runningScript = "C"
    local AutoBuy = true
    local MeleeNumber = 1

    local MeleeList = {
        [1] = "BuySanguineArt",
        [2] = "BuyGodhuman",
        [3] = "BuyDragonTalon",
    }

    local function BuyMelee()
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

-- Hàm chính
local function main()
    local vampireLabel, demonicLabel, leviathanLabel, darkLabel, logLabel = createGUI()

    -- Loop chính: Check và điều khiển script
    task.spawn(function()
        while true do
            if hasSanguineArt then
                runningScript = nil
                logLabel.Text = "Log: (done)"
                LocalPlayer:Kick("done Sanguine Art") -- Kick khi đã có Sanguine Art
                break
            end

            local success, vampireCount, demonicCount, leviathanCount, darkCount = pcall(countItems)
            if success then
                vampireLabel.Text = "Vampire Fang: " .. tostring(vampireCount)
                vampireLabel.TextColor3 = vampireCount >= 20 and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)

                demonicLabel.Text = "Demonic Wisp: " .. tostring(demonicCount)
                demonicLabel.TextColor3 = demonicCount >= 20 and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)

                leviathanLabel.Text = "Leviathan Heart: " .. tostring(leviathanCount)
                leviathanLabel.TextColor3 = leviathanCount >= 1 and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)

                darkLabel.Text = "Dark Fragment: " .. tostring(darkCount)
                darkLabel.TextColor3 = darkCount >= 2 and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)

                if leviathanCount < 1 then
                    runningScript = nil
                    logLabel.Text = "Log: (Waiting Leviathan Heart)"
                else
                    if vampireCount < 20 or demonicCount < 20 then
                        if runningScript ~= "B" then
                            runScriptB()
                        end
                        logLabel.Text = "Log: (vampire fang + demonic wisp)"
                    elseif darkCount < 2 then
                        if runningScript ~= "A" then
                            hopServer()
                            runScriptA()
                        end
                        logLabel.Text = "Log: (hop darkbeak)"
                    else
                        if runningScript ~= "C" then
                            hopServer()
                            runScriptC()
                        end
                        logLabel.Text = "Log: (done)"
                    end
                end
            else
                vampireLabel.Text = "Vampire Fang: Error"
                vampireLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                demonicLabel.Text = "Demonic Wisp: Error"
                demonicLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                leviathanLabel.Text = "Leviathan Heart: Error"
                leviathanLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                darkLabel.Text = "Dark Fragment: Error"
                darkLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                logLabel.Text = "Log: (Error)"
            end
            task.wait(2)
        end
    end)
end

-- Gọi hàm chính
local success, err = pcall(main)
if not success then
    print("Lỗi khởi tạo script main: " .. err)
end
