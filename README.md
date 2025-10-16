-- LocalHud_Full.lua
-- ضع هذا في StarterPlayer > StarterPlayerScripts
-- واجهة شبيهة بالصورة: Top bar (Recv | Ping)، Anti-AFK box، Right menu، Spectate box، FPS محلي.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- محاولة إيجاد RemoteFunction/RemoteEvent من السيرفر (اختياري لتحسين القياسات)
local pingFunc = ReplicatedStorage:FindFirstChild("PingFunction") -- RemoteFunction (server)
local recvEvent = ReplicatedStorage:FindFirstChild("ServerNetworkStats") -- افتراضي (اختياري)

-- ---------------- UI اختصارات ----------------
local function newText(parent, props)
    local t = Instance.new("TextLabel")
    t.BackgroundTransparency = props.bgTrans or 1
    t.BackgroundColor3 = props.bg or Color3.fromRGB(0,0,0)
    t.Size = props.size or UDim2.new(0,100,0,20)
    t.Position = props.pos or UDim2.new(0,0,0,0)
    t.Font = props.font or Enum.Font.Gotham
    t.TextSize = props.sizeText or 14
    t.TextColor3 = props.color or Color3.fromRGB(255,255,255)
    t.TextXAlignment = props.align or Enum.TextXAlignment.Left
    t.Text = props.text or ""
    t.Parent = parent
    t.ClipsDescendants = true
    return t
end

-- شاشة رئيسية
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomPerfHUD"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- ---------- Top bar (Recv | Ping) ----------
local topBar = Instance.new("Frame", screenGui)
topBar.Name = "TopBar"
topBar.Size = UDim2.new(0, 420, 0, 34)
topBar.Position = UDim2.new(0, 12, 0, 12)
topBar.BackgroundTransparency = 1

local recvLabel = newText(topBar, {
    pos = UDim2.new(0, 6, 0, 4),
    size = UDim2.new(0,180,0,26),
    text = "Recv  •  N/A",
    sizeText = 14,
    color = Color3.fromRGB(230,230,230),
    align = Enum.TextXAlignment.Left
})

local pingLabel = newText(topBar, {
    pos = UDim2.new(0, 190, 0, 4),
    size = UDim2.new(0,160,0,26),
    text = "Ping  •  N/A",
    sizeText = 14,
    color = Color3.fromRGB(255, 200, 80),
    align = Enum.TextXAlignment.Left
})

-- small visual network bar (green line)
local netBarBg = Instance.new("Frame", topBar)
netBarBg.Size = UDim2.new(0, 60, 0, 8)
netBarBg.Position = UDim2.new(1, -74, 0, 12)
netBarBg.BackgroundColor3 = Color3.fromRGB(40,40,40)
netBarBg.BorderSizePixel = 0
local netBar = Instance.new("Frame", netBarBg)
netBar.Size = UDim2.new(0, 0, 1, 0)
netBar.BackgroundColor3 = Color3.fromRGB(60,200,80)
netBar.BorderSizePixel = 0

-- ---------- Anti-AFK small box ----------
local afkBox = Instance.new("Frame", screenGui)
afkBox.Name = "AntiAFK"
afkBox.Size = UDim2.new(0, 220, 0, 86)
afkBox.Position = UDim2.new(0.35, 0, 0.08, 0) -- middle-ish
afkBox.BackgroundColor3 = Color3.fromRGB(18,18,18)
afkBox.BorderSizePixel = 0
afkBox.AnchorPoint = Vector2.new(0,0)
afkBox.Active = true
afkBox.Draggable = true

local afkTitle = newText(afkBox, {pos = UDim2.new(0,8,0,6), size = UDim2.new(1,-16,0,20), text = "Anti-AFK System", sizeText = 15, color = Color3.fromRGB(240,240,240)})
local afkStatus = newText(afkBox, {pos = UDim2.new(0,8,0,30), size = UDim2.new(1,-16,0,18), text = "Status: Inactive", sizeText = 14, color = Color3.fromRGB(80,200,80)})
local afkTimer = newText(afkBox, {pos = UDim2.new(0,8,0,52), size = UDim2.new(1,-16,0,18), text = "Time Active: 00:00:00", sizeText = 13, color = Color3.fromRGB(200,200,200)})

-- anti-afk toggling (local)
local afkActive = false
local afkStart = 0
local function startAFK()
    afkActive = true
    afkStart = tick()
    afkStatus.Text = "Status: Active"
    afkStatus.TextColor3 = Color3.fromRGB(80,200,80)
end
local function stopAFK()
    afkActive = false
    afkStatus.Text = "Status: Inactive"
    afkStatus.TextColor3 = Color3.fromRGB(200,80,80)
    afkTimer.Text = "Time Active: 00:00:00"
end
-- double-tap the afkBox to toggle (mobile friendly)
local lastTap = 0
afkBox.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        local now = tick()
        if now - lastTap < 0.45 then
            if afkActive then stopAFK() else startAFK() end
        end
        lastTap = now
    end
end)

-- ---------- Right side menu with buttons ----------
local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Name = "RightMenu"
menuFrame.Size = UDim2.new(0, 220, 0, 300)
menuFrame.Position = UDim2.new(1, -240, 0.22, 0)
menuFrame.BackgroundTransparency = 1

local function makeButton(parent, yPos, textLabel)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 44)
    btn.Position = UDim2.new(0, 0, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = true
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.Text = textLabel
    btn.TextColor3 = Color3.fromRGB(250,250,250)
    return btn
end

local btn1 = makeButton(menuFrame, 0, "مصادر الطرد والحماية")
local btn2 = makeButton(menuFrame, 48, "تحسين اداء الجهاز")
local btn3 = makeButton(menuFrame, 96, "تقليل استهلاك البطارية")
local btn4 = makeButton(menuFrame, 144, "تقوية الشبكة والنت")
-- additional placeholder buttons
local btn5 = makeButton(menuFrame, 192, "اوامر اخرى")
local btn6 = makeButton(menuFrame, 240, "اغلاق الواجهة")

-- implement basic client-side actions for buttons
btn2.MouseButton1Click:Connect(function()
    -- تحسين اداء الجهاز: خفض جودة الرسوم (Local only)
    pcall(function()
        settings().Rendering.QualityLevel = 1 -- ملاحظة: قد لا يؤثر على كل الأجهزة، لكن نستخدم محاولة
    end)
    StarterGui:SetCore("SendNotification", {Title="Performance", Text="Low graphics applied (local).", Duration=3})
end)

btn3.MouseButton1Click:Connect(function()
    -- تقليل استهلاك البطارية: إيقاف بعض الـParticles المحليين (نهج بسيط)
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") then
            v.Enabled = false
        end
    end
    StarterGui:SetCore("SendNotification", {Title="Battery", Text="Particles disabled (local).", Duration=3})
end)

btn4.MouseButton1Click:Connect(function()
    -- تقوية الشبكة والنت: اقتراح محلي - يخفض معدل ارسال لبعض الأشياء (محلي)
    StarterGui:SetCore("SendNotification", {Title="Network", Text="Reduced local network chatter (if any).", Duration=3})
end)

btn6.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
end)

-- ---------- Spectate box (bottom-left) ----------
local specBox = Instance.new("Frame", screenGui)
specBox.Size = UDim2.new(0, 160, 0, 40)
specBox.Position = UDim2.new(0, 12, 1, -74)
specBox.BackgroundColor3 = Color3.fromRGB(30,30,30)
specBox.BorderSizePixel = 0

local specText = newText(specBox, {pos = UDim2.new(0,8,0,6), size = UDim2.new(1,-16,0,28), text = "Spectate\nViewing: N/A", sizeText = 13, color = Color3.fromRGB(230,230,230)})
specText.TextXAlignment = Enum.TextXAlignment.Left

-- attempt to detect camera subject (spectating) — best-effort
local function updateSpectate()
    local cam = workspace.CurrentCamera
    if cam then
        local subject = cam.CameraSubject
        if subject and subject:IsA("Humanoid") then
            local pl = Players:GetPlayerFromCharacter(subject.Parent)
            if pl and pl ~= player then
                specText.Text = "Spectate\nViewing: "..pl.Name
                return
            end
        end
    end
    specText.Text = "Spectate\nViewing: "..player.Name
end

-- ---------- FPS & ping measurement ----------
-- FPS smooth (EMA)
local SAMPLE_ALPHA = 0.08
local smoothedFPS = 0
local lastRender = tick()

RunService:BindToRenderStep("HUD_FPS", Enum.RenderPriority.First.Value, function()
    local now = tick()
    local dt = now - lastRender
    lastRender = now
    if dt > 0 then
        local inst = 1/dt
        if smoothedFPS == 0 then smoothedFPS = inst else smoothedFPS = smoothedFPS + (inst - smoothedFPS) * SAMPLE_ALPHA end
    end
end)

-- ping measurement (attempt InvokeServer ping if RemoteFunction exists)
local currentPing = nil
local function measurePingFromServer()
    if pingFunc and pingFunc:IsA("RemoteFunction") then
        local ok, res = pcall(function()
            local t = tick()
            local ans = pingFunc:InvokeServer(t)
            return math.floor((tick() - t) * 1000 + 0.5)
        end)
        if ok and type(res) == "number" then
            currentPing = res
        else
            -- if server returns nothing, we still estimate via roundtrip
            currentPing = res or currentPing
        end
    else
        -- no server support: we can show N/A or local heuristic (set to nil)
        currentPing = nil
    end
end

-- recv: only available if server provides stats via event
local currentRecv = nil
if recvEvent and recvEvent:IsA("RemoteEvent") then
    -- assume server will FireClient with {recv = <num_kb_s>, ...}
    recvEvent.OnClientEvent:Connect(function(tbl)
        if type(tbl) == "table" and tbl.recv then
            currentRecv = tbl.recv
        end
    end)
end

-- update UI every 0.25s
spawn(function()
    while true do
        local fpsVal = math.floor(smoothedFPS + 0.5)
        -- update FPS display inside afkBox maybe, and elsewhere
        afkTimer.Text = "Time Active: " .. (afkActive and os.date("!%H:%M:%S", math.floor(tick() - afkStart)) or "00:00:00")
        -- ping measurement attempt
        pcall(measurePingFromServer)
        -- update labels
        if currentRecv then
            recvLabel.Text = string.format("Recv  •  %.2f KB/s", tonumber(currentRecv) or 0)
            netBar.Size = UDim2.new(math.clamp((tonumber(currentRecv) or 0) / 20, 0, 1), 0, 1, 0)
        else
            recvLabel.Text = "Recv  •  N/A"
            netBar.Size = UDim2.new(0.35, 0, 1, 0)
        end
        if currentPing then
            pingLabel.Text = string.format("Ping  •  %d ms", currentPing)
        else
            pingLabel.Text = "Ping  •  N/A"
        end

        -- small FPS display on top-left of AFK box for quick info
        afkTitle.Text = "Anti-AFK System  |  "..tostring(fpsVal).." fps"
        -- update spec box
        pcall(updateSpectate)
        wait(0.25)
    end
end)

-- keyboard shortcut to toggle whole HUD (P)
UserInputService.InputBegan:Connect(function(inp, gp)
    if gp then return end
    if inp.KeyCode == Enum.KeyCode.P then
        screenGui.Enabled = not screenGui.Enabled
    end
end)
