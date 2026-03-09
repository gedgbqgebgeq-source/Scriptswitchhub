-- Redeem codes (ใช้ของคุณได้)
local DEFAULT_CODES = {"Sub2Fer999","Enyu_is_Pro","Magicbus","JCWK","Starcodeheo","Bluxxy"}

-- CORE
local Players = game:GetService("Players")
local Replicated = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local LocalPlayer = Players.LocalPlayer

-- safe find remote helper (พยายามหา Remotes.CommF_ หรือ CommF หรือชื่ออื่น)
local function getRemote(name)
    if Replicated:FindFirstChild("Remotes") then
        if Replicated.Remotes:FindFirstChild(name) then
            return Replicated.Remotes[name]
        end
    end

    if Replicated:FindFirstChild(name) then
        return Replicated[name]
    end

    return nil
end

local remoteComm = getRemote("CommF_") or getRemote("CommF") or getRemote("CommF__") or getRemote("Comm")
local remoteRedeem = getRemote("Redeem") or (Replicated:FindFirstChild("Remotes") and Replicated.Remotes:FindFirstChild("Redeem"))

-- safe invoke wrapper
local function safeInvoke(remote, ...)
    if not remote then return false end
    local ok, res = pcall(function() return remote:InvokeServer(...) end)
    return ok, res
end

local function safeFire(remote, ...)
    if not remote then return false end
    local ok = pcall(function() remote:FireServer(...) end)
    return ok
end

-- wait for character
repeat task.wait() until LocalPlayer and LocalPlayer.Character
repeat task.wait() until LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

-- choose team if config indicates
pcall(function()
    local team = (_G.Main and _G.Main.Team) or "Pirates"
    if remoteComm then
        pcall(function()
            remoteComm:InvokeServer("SetTeam", team)
        end)
    end
end)

-- REDEEM CODES (try list)
task.spawn(function()
    local codes = (_G.RedeemCodes and type(_G.RedeemCodes)=="table") and _G.RedeemCodes or DEFAULT_CODES
    for _, c in ipairs(codes) do
        pcall(function()
            if remoteRedeem then
                remoteRedeem:InvokeServer(c)
            else
                -- some games use Remotes.Redeem inside Remotes
                if Replicated:FindFirstChild("Remotes") and Replicated.Remotes:FindFirstChild("Redeem") then
                    Replicated.Remotes.Redeem:InvokeServer(c)
                end
            end
        end)
        task.wait(0.6)
    end
end

-- TWEEN
local currentTween = nil
local function TweenTo(cf, speed)

    if typeof(cf) ~= "CFrame" then return end  -- กัน error

    speed = speed or (_G.Main and _G.Main.TweenSpeed) or 350
    if not LocalPlayer.Character then return end
    if not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end

    local hrp = LocalPlayer.Character.HumanoidRootPart

    if currentTween then
        pcall(function()
            currentTween:Cancel()
        end)
    end

    local dist = (hrp.Position - cf.Position).Magnitude
    local time = math.max(dist / speed, 0.12)

    currentTween = TweenService:Create(
        hrp,
        TweenInfo.new(time, Enum.EasingStyle.Linear),
        {CFrame = cf}
    )

    pcall(function()
        currentTween:Play()
    end)
end

-- AUTO HAKI
task.spawn(function()
    while task.wait(1) do
        pcall(function()
            if remoteComm then
                remoteComm:InvokeServer("Buso")
            else
                -- fallback names
                local alt = getRemote("Buso") or getRemote("Haki")
                if alt then pcall(function() alt:InvokeServer() end) end
            end
        end)
    end
end)

repeat task.wait()
until game.Players.LocalPlayer:FindFirstChild("PlayerScripts")
and game.Players.LocalPlayer.PlayerScripts:FindFirstChild("CombatFramework")

local CombatFramework = require(game.Players.LocalPlayer.PlayerScripts:WaitForChild("CombatFramework"))

-- FAST ATTACK (fixed)
local CombatFramework = nil

repeat task.wait()
    pcall(function()
        CombatFramework = 
        require(game.Players.LocalPlayer.PlayerScripts:WaitForChild("CombatFramework"))
    end)
until CombatFramework

local function GetCurrentBlade()
    local up = debug.getupvalues(CombatFramework)
    if not up or not up[2] then return nil end

    local GetFastAttack = up[2]
    local activeController = GetFastAttack.activeController
    if not activeController then return nil end
    local blades = activeController.blades and activeController.blades[1]
    if not blades then return end
    while blades.Parent ~= game.Players.LocalPlayer.Character do
        blades = blades.Parent
        if not blades then return end
    end
    return blades
end

local function AttackNoCD()
    local plr = game:GetService("Players").LocalPlayer
    local up = debug.getupvalues(CombatFramework)
    if not up or not up[2] then return end
    local GetFastAttack = up[2]
    local activeController = GetFastAttack and
    GetFastAttack.activeController
    if not activeController then return end

    local RigLib =
    require(game.ReplicatedStorage:WaitForChild("CombatFramework"):WaitForChild("RigLib"))
    local getBladeHits = RigLib.getBladeHits

    local hits = getBladeHits(plr.Character,
    {plr.Character.HumanoidRootPart}, 60)
    local cac = {}
    local hash = {}
    for k, v in pairs(hits) do
        if v.Parent and v.Parent:FindFirstChild("HumanoidRootPart") and not hash[v.Parent] then
            table.insert(cac, v.Parent.HumanoidRootPart)
            hash[v.Parent] = true
        end
    end
    getBladeHits = cac

    if #getBladeHits > 0 then
        local ok, u8 = pcall(function() return debug.getupvalue(activeController.attack, 5) end)
        local ok2, u9 = pcall(function() return debug.getupvalue(activeController.attack, 6) end)
        local ok3, u7 = pcall(function() return debug.getupvalue(activeController.attack, 4) end)
        local ok4, u10 = pcall(function() return debug.getupvalue(activeController.attack, 7) end)

        if not (ok and ok2 and ok3 and ok4) then return end

        local u12 = (u8 * 798405 + u7 * 727595) % u9
        local u13 = u7 * 798405
        (function()
            u12 = (u12 * u9 + u13) % 1099511627776
            u8 = math.floor(u12 / u9)
            u7 = u12 - u8 * u9
        end)()
        u10 = u10 + 1
        pcall(function() debug.setupvalue(activeController.attack, 5, u8) end)
        pcall(function() debug.setupvalue(activeController.attack, 6, u9) end)
        pcall(function() debug.setupvalue(activeController.attack, 4, u7) end)
        pcall(function() debug.setupvalue(activeController.attack, 7, u10) end)

        pcall(function()
            if plr.Character and plr.Character:FindFirstChildOfClass("Tool") and activeController.blades and activeController.blades[1] then
                activeController.animator.anims.basic[1]:Play(0.01, 0.01, 0.01)
                game:GetService("ReplicatedStorage").RigControllerEvent:FireServer("weaponChange", tostring(GetCurrentBlade()))
                if game.ReplicatedStorage:FindFirstChild("Remotes") and game.ReplicatedStorage.Remotes:FindFirstChild("Validator") then
                    game.ReplicatedStorage.Remotes.Validator:FireServer(math.floor(u12 / 1099511627776 * 16777215), u10)
                elseif game.ReplicatedStorage:FindFirstChild("Validator") then
                    game.ReplicatedStorage.Validator:FireServer(math.floor(u12 / 1099511627776 * 16777215), u10)
                end
                game:GetService("ReplicatedStorage").RigControllerEvent:FireServer("hit", getBladeHits, 1, "")
            end
        end)
    end
end

local FastAttack = true
local FastAttackDelay = 0.08

task.spawn(function()
    while task.wait(FastAttackDelay) do
        if FastAttack then
            pcall(function()
                local CameraShakerR = nil
                if game.ReplicatedStorage:FindFirstChild("Util") and game.ReplicatedStorage.Util:FindFirstChild("CameraShaker") then
                    CameraShakerR = require(game.ReplicatedStorage.Util.CameraShaker)
                end
                if CameraShakerR and CameraShakerR.Stop then
                    pcall(function() CameraShakerR:Stop() end)
                end
                AttackNoCD()
            end)
        end
    end
end)

-- BRING MOB
local bringfrec = tonumber(300) or 300

function BringMonster(TargetName, TargetCFrame)

-- กัน error
if not game.Players.LocalPlayer.Character then return end
if not game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end

if not game:GetService("Workspace"):FindFirstChild("Enemies") then 
    return 
end

for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
if v.Name == TargetName then
if v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
if v:FindFirstChild("HumanoidRootPart") and
(v.HumanoidRootPart.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude < bringfrec then
v.HumanoidRootPart.CFrame = TargetCFrame
v.HumanoidRootPart.CanCollide = false
v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
v.HumanoidRootPart.Transparency = 1
v.Humanoid:ChangeState(11)
v.Humanoid:ChangeState(14)
if v.Humanoid:FindFirstChild("Animator") then
v.Humanoid.Animator:Destroy()
end
end
end
end
end
end

-- CHECK/Equip weapon
local function EquipToolIfAny()
    pcall(function()
        if not LocalPlayer.Character then return end
        if LocalPlayer.Character:FindFirstChildOfClass("Tool") then return end
        for _, it in pairs(LocalPlayer.Backpack:GetChildren()) do
            if it:IsA("Tool") then
                LocalPlayer.Character.Humanoid:EquipTool(it)
                break
            end
        end
    end)
end

-- MAPPINGS: level ranges -> quests & mob CFrames
-- (ผมย้าย mapping มาเป็นตารางเพื่ออ่านง่าย และแก้เงื่อนไขให้ถูกต้อง)
local levelData = {
    -- FIRST SEA (ตัวอย่างหลายช่วงจากข้อมูลของคุณ)
    {min=1, max=9, Ms="Bandit", NameQuest="BanditQuest1", QuestLv=1, CFrameQ=CFrame.new(1060.9383,16.4550,1547.7841), CFrameMon=CFrame.new(1038.5533,41.2962,1576.5099)},
    {min=10, max=14, Ms="Monkey", NameQuest="JungleQuest", QuestLv=1, CFrameQ=CFrame.new(-1601.6554,36.8521,153.3881), CFrameMon=CFrame.new(-1448.1446,50.8519,63.6072)},
    {min=15, max=29, Ms="Gorilla", NameQuest="JungleQuest", QuestLv=2, CFrameQ=CFrame.new(-1601.6554,36.8521,153.3881), CFrameMon=CFrame.new(-1142.6488,40.4623,515.3923)},
    {min=30, max=39, Ms="Pirate", NameQuest="BuggyQuest1", QuestLv=1, CFrameQ=CFrame.new(-1140.1761,4.7520,3827.4058), CFrameMon=CFrame.new(-1201.0881,40.6289,3857.5967)},
    {min=40, max=59, Ms="Brute", NameQuest="BuggyQuest1", QuestLv=2, CFrameQ=CFrame.new(-1140.1761,4.7520,3827.4058), CFrameMon=CFrame.new(-1387.5324,24.5920,4100.9575)},
    {min=60, max=74, Ms="Desert Bandit", NameQuest="DesertQuest", QuestLv=1, CFrameQ=CFrame.new(896.5172,6.4384,4390.1494), CFrameMon=CFrame.new(984.9989,16.1095,4417.9102)},
    {min=75, max=89, Ms="Desert Officer", NameQuest="DesertQuest", QuestLv=2, CFrameQ=CFrame.new(896.5172,6.4384,4390.1494), CFrameMon=CFrame.new(1547.1510,14.4520,4381.8003)},
    {min=90, max=99, Ms="Snow Bandit", NameQuest="SnowQuest", QuestLv=1, CFrameQ=CFrame.new(1386.8073,87.2727,1298.3576), CFrameMon=CFrame.new(1356.3028,105.7686,1328.2418)},
    {min=100, max=119, Ms="Snowman", NameQuest="SnowQuest", QuestLv=2, CFrameQ=CFrame.new(1386.8073,87.2727,1298.3576), CFrameMon=CFrame.new(1218.7956,138.0118,1488.0262)},
    {min=120, max=174, Ms="Chief Petty Officer", NameQuest="MarineQuest2", QuestLv=1, CFrameQ=CFrame.new(-5035.4961,28.6778,4324.1841), CFrameMon=CFrame.new(-4931.1552,65.7931,4121.8394)},
    {min=175, max=189, Ms="Dark Master", NameQuest="SkyQuest", QuestLv=2, CFrameQ=CFrame.new(-4842.1372,717.6954,2623.0483), CFrameMon=CFrame.new(-5148.1650,439.0457,2332.9612)},
    {min=190, max=209, Ms="Prisoner", NameQuest="PrisonerQuest", QuestLv=1, CFrameQ=CFrame.new(5310.60547,0.3500149,474.946594), CFrameMon=CFrame.new(4937.31885,0.33203139,649.574524)},
    {min=210, max=249, Ms="Dangerous Prisoner", NameQuest="PrisonerQuest", QuestLv=2, CFrameQ=CFrame.new(5310.60547,0.3500149,474.946594), CFrameMon=CFrame.new(5099.6626,0.351562679,1055.7583)},
    {min=250, max=274, Ms="Toga Warrior", NameQuest="ColosseumQuest", QuestLv=1, CFrameQ=CFrame.new(-1577.78906,7.415142, -2984.4839), CFrameMon=CFrame.new(-1872.5166,49.080215,2913.8105)},
    {min=275, max=299, Ms="Gladiator", NameQuest="ColosseumQuest", QuestLv=2, CFrameQ=CFrame.new(-1577.78906,7.415142,-2984.4839), CFrameMon=CFrame.new(-1521.3740,81.20317,3066.3139)},
    {min=300, max=324, Ms="Military Soldier", NameQuest="MagmaQuest", QuestLv=1, CFrameQ=CFrame.new(-5316.1157,12.262832,8517.0039), CFrameMon=CFrame.new(-5369.0005,61.243526,8556.4922)},
    {min=325, max=374, Ms="Military Spy", NameQuest="MagmaQuest", QuestLv=2, CFrameQ=CFrame.new(-5316.1157,12.262832,8517.0039), CFrameMon=CFrame.new(-5787.00293,75.826263,8651.69922)},
    {min=375, max=399, Ms="Fishman Warrior", NameQuest="FishmanQuest", QuestLv=1, CFrameQ=CFrame.new(61122.6523,18.49744,1569.39978), CFrameMon=CFrame.new(60844.10547,98.462875,1298.39856)},
    {min=400, max=449, Ms="Fishman Commando", NameQuest="FishmanQuest", QuestLv=2, CFrameQ=CFrame.new(61122.6523,18.49744,1569.39978), CFrameMon=CFrame.new(61738.3984375,64.207321,1433.837524)},
    {min=450, max=474, Ms="God's Guard", NameQuest="SkyExp1Quest", QuestLv=1, CFrameQ=CFrame.new(-4721.86035,845.30297,1953.84899), CFrameMon=CFrame.new(-4628.0498,866.92877,1931.23523)},
    {min=475, max=524, Ms="Shanda", NameQuest="SkyExp1Quest", QuestLv=2, CFrameQ=CFrame.new(-7863.1596679688,5545.5190429688,378.42266845703), CFrameMon=CFrame.new(-7685.1474609375,5601.0751953125,441.38876342773)},
    {min=525, max=549, Ms="Royal Squad", NameQuest="SkyExp2Quest", QuestLv=1, CFrameQ=CFrame.new(-7903.3828125,5635.9897460938,-1410.923828125), CFrameMon=CFrame.new(-7654.2514648438,5637.1079101563,1407.7550048828)},
    {min=550, max=624, Ms="Royal Soldier", NameQuest="SkyExp2Quest", QuestLv=2, CFrameQ=CFrame.new(-7903.3828125,5635.9897460938,-1410.923828125), CFrameMon=CFrame.new(-7760.4106445313,5679.9077148438,1884.8112792969)},
    {min=625, max=649, Ms="Galley Pirate", NameQuest="FountainQuest", QuestLv=1, CFrameQ=CFrame.new(5258.2788085938,38.526931762695,4050.044921875), CFrameMon=CFrame.new(5557.1684570313,152.32717895508,3998.7758789063)},
    {min=650, max=699, Ms="Galley Captain", NameQuest="FountainQuest", QuestLv=2, CFrameQ=CFrame.new(5258.2788085938,38.526931762695,4050.044921875), CFrameMon=CFrame.new(5677.6772460938,92.786109924316,4966.6323242188)},
}

-- คุณสามารถต่อเติม entries ใน levelData ให้ครบทุกช่วงที่ต้องการได้
-- ฟังก์ชันค้นหา mapping ตามเลเวล
local function getMappingForLevel(Lv)
    for _, entry in ipairs(levelData) do
        if Lv >= entry.min and Lv <= entry.max then
            return entry
        end
    end
    return nil
end

-- CheckMob / CheckQuest / Mob Magnet / Farm Loop
local function findMobByName(name)
    for _, v in pairs(workspace.Enemies:GetChildren()) do
        if v.Name == name and v:FindFirstChild("HumanoidRootPart") and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            return v
        end
    end
    return nil
end

local function nearestEnemy()
    local best, dist = nil, math.huge
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return nil end
    local pos = LocalPlayer.Character.HumanoidRootPart.Position
    for _, v in pairs(workspace.Enemies:GetChildren()) do
        if v:FindFirstChild("HumanoidRootPart") and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            local d = (v.HumanoidRootPart.Position - pos).Magnitude
            if d < dist then dist = d; best = v end
        end
    end
    return best
end

local function StartQuestIfNeeded(mapping)
    pcall(function()
        if not mapping then return end
        local hasQuestGui = LocalPlayer.PlayerGui:FindFirstChild("Main") and LocalPlayer.PlayerGui.Main:FindFirstChild("Quest")
        local questVisible = hasQuestGui and LocalPlayer.PlayerGui.Main.Quest.Visible
        if not questVisible then
            if remoteComm then
                pcall(function() remoteComm:InvokeServer("StartQuest", mapping.NameQuest, mapping.QuestLv) end)
                task.wait(0.6)
            end
        end
    end)
end

-- Farm loop (หลัก)
task.spawn(function()
    while task.wait(0.3) do
        pcall(function()
            if not _G.Main or not _G.Main.AutoFarm then
                    return
                end
                
            EquipToolIfAny()

            local lv = nil
            pcall(function() lv = (LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") and LocalPlayer.Data.Level.Value) or 1 end)
            local mapping = getMappingForLevel(lv)

            StartQuestIfNeeded(mapping)

            local mob = nil
            if mapping then mob = findMobByName(mapping.Ms) end
            if not mob then mob = nearestEnemy() end

            if mob and mob:FindFirstChild("HumanoidRootPart") then
                local targetC = mob.HumanoidRootPart.CFrame * CFrame.new(0,15,0)
                TweenTo(targetC, (_G.Main and _G.Main.TweenSpeed) or 325)
            else
                if mapping and mapping.CFrameMon then
                    TweenTo(mapping.CFrameMon, (_G.Main and _G.Main.TweenSpeed) or 325)
                end
            end
        end)
    end
end)

-- UI: Build scrollable UI like image (closest)
do
    local CoreGui = game:GetService("CoreGui")
    -- remove existing
    for _,v in pairs(CoreGui:GetChildren()) do
        if v.Name == "SwitchHub_UI" then pcall(function() v:Destroy() end) end
    end

    local screen = Instance.new("ScreenGui")
    screen.Name = "SwitchHub_UI"
    screen.ResetOnSpawn = false
    screen.Parent = game.CoreGui

    local main = Instance.new("Frame", screen)
    main.Name = "MainFrame"
    main.Size = UDim2.new(0,900,0,760)
    main.Position = UDim2.new(0.02,0,0.02,0)
    main.BackgroundColor3 = Color3.fromRGB(18,12,8)
    main.BorderSizePixel = 0

    local title = Instance.new("TextLabel", main)
    title.Size = UDim2.new(1,0,0,38)
    title.Position = UDim2.new(0,0,0,0)
    title.BackgroundTransparency = 1
    title.Text = "Switch Hub [ Series X ] | General"
    title.TextColor3 = Color3.fromRGB(230,190,120)
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 20

    -- two columns
    local left = Instance.new("Frame", main)
    left.Size = UDim2.new(0.48, -10, 1, -50)
    left.Position = UDim2.new(0,8,0,50)
    left.BackgroundTransparency = 1
    local right = Instance.new("Frame", main)
    right.Size = UDim2.new(0.48, -10, 1, -50)
    right.Position = UDim2.new(0.52, 2, 0, 50)
    right.BackgroundTransparency = 1

    local function makeScroll(parent)
        local s = Instance.new("ScrollingFrame", parent)
        s.Size = UDim2.new(1,0,1,0)
        s.CanvasSize = UDim2.new(0,0,2,0)
        s.ScrollBarThickness = 8
        s.BackgroundTransparency = 1
        local layout = Instance.new("UIListLayout", s)
        layout.Padding = UDim.new(0,8)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        return s
    end

    local leftScroll = makeScroll(left)
    local rightScroll = makeScroll(right)

    -- section creator
    local function createSection(parent, titleText, h)
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, -12, 0, h or 140)
        fr.BackgroundColor3 = Color3.fromRGB(38,28,20)
        fr.BorderSizePixel = 1
        fr.Parent = parent

        local t = Instance.new("TextLabel", fr)
        t.Size = UDim2.new(1,0,0,24)
        t.Position = UDim2.new(0,0,0,0)
        t.BackgroundTransparency = 1
        t.Text = "  "..(titleText or "Section")
        t.Font = Enum.Font.SourceSansBold
        t.TextColor3 = Color3.fromRGB(220,150,100)
        t.TextSize = 16

        local content = Instance.new("Frame", fr)
        content.Size = UDim2.new(1,-10,1,-30)
        content.Position = UDim2.new(0,5,0,26)
        content.BackgroundTransparency = 1

        return fr, content
    end

    local sStatus, cStatus = createSection(leftScroll, "Status", 140)
    local sQuest, cQuest = createSection(leftScroll, "Quest", 180)
    local sMelee, cMelee = createSection(leftScroll, "Melee", 180)
    local sInfo, cInfo = createSection(leftScroll, "Information", 150)

    local sFarm, cFarm = createSection(rightScroll, "Farming", 90)
    local sSword, cSword = createSection(rightScroll, "Sword", 420)
    local sGun, cGun = createSection(rightScroll, "Gun", 140)

    -- checkbox helper
    local function checkbox(parent, label, initial)
        local box = Instance.new("Frame", parent)
        box.Size = UDim2.new(1,0,0,28)
        box.BackgroundTransparency = 1

        local btn = Instance.new("TextButton", box)
        btn.Size = UDim2.new(0,24,0,24)
        btn.Position = UDim2.new(0,0,0,2)
        btn.Text = initial and "✓" or "✗"
        btn.TextColor3 = initial and Color3.fromRGB(150,255,150) or Color3.fromRGB(255,120,120)
        btn.Font = Enum.Font.SourceSansBold
        btn.TextSize = 18
        btn.BackgroundColor3 = Color3.fromRGB(28,28,28)

        local lbl = Instance.new("TextLabel", box)
        lbl.Size = UDim2.new(1,-30,1,0)
        lbl.Position = UDim2.new(0,30,0,0)
        lbl.BackgroundTransparency = 1
        lbl.Font = Enum.Font.Code
        lbl.Text = " : "..label
        lbl.TextColor3 = Color3.fromRGB(240,240,240)
        lbl.TextXAlignment = Enum.TextXAlignment.Left

        local value = initial and true or false
        btn.MouseButton1Click:Connect(function()
            value = not value
            btn.Text = value and "✓" or "✗"
            btn.TextColor3 = value and Color3.fromRGB(150,255,150) or Color3.fromRGB(255,120,120)
        end)
        return box, function() return value end, function(v) value = not not v; btn.Text = value and "✓" or "✗"; btn.TextColor3 = value and Color3.fromRGB(150,255,150) or Color3.fromRGB(255,120,120) end
    end

    -- fill status text
    local statusLabel = Instance.new("TextLabel", cStatus)
    statusLabel.Size = UDim2.new(1,0,1,0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Font = Enum.Font.Code
    statusLabel.TextSize = 14
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.TextColor3 = Color3.fromRGB(230,230,230)

    local function updateStatus()
        pcall(function()
            local name = LocalPlayer.Name
            local world = "N/A"
            local fruit = "None"
            local awaken = "-"
            if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then world = tostring(LocalPlayer.Data.Level.Value) end
            if _G and _G.Fruit then fruit = table.concat(_G.Fruit, ", ") end
            if _G and _G.Fruit_Main and _G.Fruit_Main.Main then awaken = table.concat(_G.Fruit_Main.Main, ", ") end
            statusLabel.Text = ("Name : %s\nWorld : %s\nFruit : %s\nAwaken : %s"):format(name, world, fruit, awaken)
        end)
    end
    spawn(function() while task.wait(2) do updateStatus() end end)
    updateStatus()

    -- farm toggles
    local farmBox, farmGet, farmSet = checkbox(cFarm, "Auto Farm", _G.Main and _G.Main.AutoFarm)
    farmBox.Parent = cFarm
    local farmStatus = Instance.new("TextLabel", cFarm)
    farmStatus.Size = UDim2.new(1,0,0,24)
    farmStatus.Position = UDim2.new(0,0,0,34)
    farmStatus.BackgroundTransparency = 1
    farmStatus.TextColor3 = Color3.fromRGB(220,220,220)
    farmStatus.Font = Enum.Font.Code
    farmStatus.TextSize = 13
    farmStatus.Text = "Status : "..((_G.Main and _G.Main.AutoFarm) and "Farming" or "Idle")

    spawn(function()
        while task.wait(0.5) do
            local v = farmGet()
            if v ~= (_G.Main and _G.Main.AutoFarm) then
                _G.Main = _G.Main or {}
                _G.Main.AutoFarm = v
                farmStatus.Text = "Status : "..(v and "Farming" or "Idle")
            end
        end
    end)

    -- Quest list (simple checkboxes)
    local questList = {"Quest Bartlio","Quest Don Swan","Quest Death Step","Quest Sharkman Karate","Quest Dragon Talon"}
    for _, q in ipairs(questList) do
        local b,g,s = checkbox(cQuest, q, true)
        b.Parent = cQuest
    end

    -- Melee list
    for name,_ in pairs(_G.Melee or {}) do
        local b,g,s = checkbox(cMelee, name, true)
        b.Parent = cMelee
    end

    -- Sword list
    for name,val in pairs(_G.Sword or {}) do
        local b,g,s = checkbox(cSword, name, val)
        b.Parent = cSword
    end

    -- Gun list
    for name,val in pairs(_G.Gun or {}) do
        local b,g,s = checkbox(cGun, name, val)
        b.Parent = cGun
    end

    -- Information
    local infoLabel = Instance.new("TextLabel", cInfo)
    infoLabel.Size = UDim2.new(1,0,1,0)
    infoLabel.BackgroundTransparency = 1
    infoLabel.Font = Enum.Font.Code
    infoLabel.TextSize = 14
    infoLabel.TextColor3 = Color3.fromRGB(200,200,200)
    infoLabel.Text = "Bone : 0\nEctoplasm : 0\nElite Hunter : 0"

    -- Save state button (partial)
    local saveBtn = Instance.new("TextButton", rightScroll)
    saveBtn.Size = UDim2.new(1, -12, 0, 36)
    saveBtn.Text = "Save partial settings to _G"
    saveBtn.Font = Enum.Font.SourceSansBold
    saveBtn.TextColor3 = Color3.new(1,1,1)
    saveBtn.BackgroundColor3 = Color3.fromRGB(45,35,25)
    saveBtn.MouseButton1Click:Connect(function()
        _G.Main = _G.Main or {}
        _G.Main.AutoFarm = farmGet()
        saveBtn.Text = "Saved!"
        task.delay(1, function() saveBtn.Text = "Save partial settings to _G" end)
    end)
end

game:GetService("Players").LocalPlayer.Idled:Connect(function()
    game:GetService("VirtualUser"):CaptureController()
    game:GetService("VirtualUser"):ClickButton2(Vector2.new())
end)

-- FINISHED
print("Switch Hub loaded. UI created. Auto features active (redeem, fast attack, auto haki, basic auto farm).")
print("If remote names differ or specific mapping missing, tell me the error in output and I'll patch it.")
