-- كبيرهم GUI - Local Player Tab كامل ومنظم
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- إزالة أي GUI سابق
if player:FindFirstChild("Kabeerhom_GUI") then
    player.Kabeerhom_GUI:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "Kabeerhom_GUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- الإطار الرئيسي
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 260, 0, 330)
mainFrame.Position = UDim2.new(0.7, 0, 0.05, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -30, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "كبيرهم"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(173,255,122)
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = mainFrame

-- زر الإغلاق X
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -28, 0, 0)
closeBtn.BackgroundColor3 = Color3.fromRGB(255,0,0)
closeBtn.Text = "X"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
closeBtn.Parent = mainFrame

closeBtn.MouseButton1Click:Connect(function()
    mainFrame:Destroy()
end)

-- وظيفة إنشاء تسميات
local function makeLabel(y, text)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -10, 0, 22)
    lbl.Position = UDim2.new(0, 5, 0, y)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 16
    lbl.TextColor3 = Color3.fromRGB(173,255,122)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Text = text
    lbl.Parent = mainFrame
    return lbl
end

-- بيانات اللاعب
local playerIdLabel = makeLabel(40, "معرف اللاعب: Loading...")
local appearanceLabel = makeLabel(70, "المظهر: Loading...")
local runtimeLabel = makeLabel(110, "وقت التشغيل: 0 س 0 د 0 ث")
local statsLabel = makeLabel(150, "الإحصائيات:")
local currentPlayersLabel = makeLabel(180, "0 لاعب")
local maxPlayersLabel = makeLabel(210, "عدد اللاعبين كحد أقصى: 0")
local placeNameLabel = makeLabel(240, "اسم الماب: Loading...")
local placeIdLabel = makeLabel(270, "Place ID: Loading...")

-- تحديث مستمر
local startTime = tick()
RunService.RenderStepped:Connect(function()
    if player then
        playerIdLabel.Text = "معرف اللاعب: " .. tostring(player.UserId)
        appearanceLabel.Text = "المظهر: " .. tostring(player.UserId)
    end

    -- حساب الساعات والدقائق والثواني
    local elapsed = math.floor(tick() - startTime)
    local hours = math.floor(elapsed / 3600)
    local minutes = math.floor((elapsed % 3600) / 60)
    local seconds = elapsed % 60
    runtimeLabel.Text = string.format("وقت التشغيل: %d س %d د %d ث", hours, minutes, seconds)

    -- تحديث الإحصائيات
    currentPlayersLabel.Text = #Players:GetPlayers() .. " لاعب"
    maxPlayersLabel.Text = "عدد اللاعبين كحد أقصى: " .. Players.MaxPlayers

    -- اسم الماب بالعربي حسب ID2
    local mapName = ""
    if game.PlaceId == 2 then
        mapName = "الماب الرائع" -- تحط الاسم اللي تحبه بالعربي
    else
        mapName = "ماب آخر"
    end
    placeNameLabel.Text = "اسم الماب: " .. mapName
    placeIdLabel.Text = "Place ID: " .. tostring(game.PlaceId)
end)

-- Draggable بدون مشاكل عند الضغط
local dragging = false
local dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

print("[كبيرهم] GUI loaded perfectly with Arabic map name and organized runtime.")
