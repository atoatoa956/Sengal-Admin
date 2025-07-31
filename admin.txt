local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local Char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HRP = Char:WaitForChild("HumanoidRootPart")
local Humanoid = Char:FindFirstChildOfClass("Humanoid")

-- --- Arayüz --- --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SengalAdminGUI"
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 350, 0, 50)
Frame.Position = UDim2.new(0, 20, 0, 100)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.AnchorPoint = Vector2.new(0, 0)
Frame.Active = true
Frame.Draggable = true

local TextBox = Instance.new("TextBox", Frame)
TextBox.Size = UDim2.new(1, -10, 1, -10)
TextBox.Position = UDim2.new(0, 5, 0, 5)
TextBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
TextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
TextBox.TextScaled = true
TextBox.PlaceholderText = "Komut girin (örn: /fly)"
TextBox.ClearTextOnFocus = false
TextBox.Font = Enum.Font.Gotham

-- --- Bildirim fonksiyonu --- --
local function notify(text)
    game.StarterGui:SetCore("SendNotification", {
        Title = "Sengal Admin Pro v.5";
        Text = text;
        Duration = 3;
    })
end

-- --- Fly sistemi --- --
local flying = false
local BodyGyro, BodyVelocity
local flyConn

local function startFly()
    if flying then return end
    flying = true

    BodyGyro = Instance.new("BodyGyro")
    BodyGyro.P = 9e4
    BodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    BodyGyro.CFrame = HRP.CFrame
    BodyGyro.Parent = HRP

    BodyVelocity = Instance.new("BodyVelocity")
    BodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    BodyVelocity.Velocity = Vector3.new(0, 0, 0)
    BodyVelocity.Parent = HRP

    flyConn = RunService.Heartbeat:Connect(function()
        if not flying then return end
        local camCF = workspace.CurrentCamera.CFrame
        local moveVec = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then moveVec += camCF.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then moveVec -= camCF.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then moveVec -= camCF.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then moveVec += camCF.RightVector end

        if moveVec.Magnitude > 0 then
            BodyVelocity.Velocity = moveVec.Unit * 50
        else
            BodyVelocity.Velocity = Vector3.new(0,0,0)
        end
        BodyGyro.CFrame = camCF
    end)
    notify("Fly aktif!")
end

local function stopFly()
    flying = false
    if BodyGyro then BodyGyro:Destroy() end
    if BodyVelocity then BodyVelocity:Destroy() end
    if flyConn then
        flyConn:Disconnect()
        flyConn = nil
    end
    notify("Fly kapandı!")
end

-- --- Noclip sistemi --- --
local noclip = false
RunService.Stepped:Connect(function()
    if noclip then
        for _, part in ipairs(Char:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- --- ESP sistemi --- --
local espActive = false
local espObjects = {}

local function toggleESP()
    espActive = not espActive
    if espActive then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
                local billboard = Instance.new("BillboardGui")
                billboard.Name = "ESP"
                billboard.Adornee = plr.Character.Head
                billboard.Size = UDim2.new(0, 100, 0, 20)
                billboard.StudsOffset = Vector3.new(0, 2, 0)
                billboard.AlwaysOnTop = true
                billboard.Parent = plr.Character

                local text = Instance.new("TextLabel", billboard)
                text.Size = UDim2.new(1, 0, 1, 0)
                text.Text = plr.Name
                text.TextColor3 = Color3.fromRGB(255, 0, 0)
                text.BackgroundTransparency = 1
                text.TextScaled = true

                table.insert(espObjects, billboard)
            end
        end
        notify("ESP açık!")
    else
        for _, obj in pairs(espObjects) do
            if obj and obj.Parent then
                obj:Destroy()
            end
        end
        espObjects = {}
        notify("ESP kapalı!")
    end
end

-- --- Görünmezlik --- --
local function makeInvisible()
    for _, part in pairs(Char:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.Transparency = 1
        elseif part:IsA("Decal") then
            part.Transparency = 1
        end
    end
    notify("Görünmez oldun!")
end

local function makeVisible()
    for _, part in pairs(Char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
        elseif part:IsA("Decal") then
            part.Transparency = 0
        end
    end
    notify("Görünür oldun!")
end

-- --- Komut işleyici --- --
TextBox.FocusLost:Connect(function(enterPressed)
    if not enterPressed then return end
    local text = TextBox.Text
    TextBox.Text = ""
    local args = {}
    for word in text:gmatch("%S+") do
        table.insert(args, word)
    end

    if #args == 0 then return end

    local cmd = args[1]:lower()
    table.remove(args,1)

    if cmd == "/fly" then
        startFly()
    elseif cmd == "/unfly" then
        stopFly()
    elseif cmd == "/noclip" then
        noclip = true
        notify("Noclip aktif!")
    elseif cmd == "/clip" then
        noclip = false
        notify("Noclip kapalı!")
    elseif cmd == "/tools" then
        for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
            tool.Parent = Char
        end
        notify("Araçlar verildi!")
    elseif cmd == "/speed" then
        local speed = tonumber(args[1])
        if speed and Humanoid then
            Humanoid.WalkSpeed = speed
            notify("Hız ayarlandı: "..speed)
        else
            notify("Geçersiz hız!")
        end
    elseif cmd == "/esp" then
        toggleESP()
    elseif cmd == "/invisible" then
        makeInvisible()
    elseif cmd == "/visible" then
        makeVisible()
    elseif cmd == "/jump" then
        if Humanoid then
            Humanoid.Jump = true
            notify("Zıpladın!")
        end
    elseif cmd == "/sit" then
        if Humanoid then
            Humanoid.Sit = true
            notify("Oturuyorsun!")
        end
    elseif cmd == "/goto" then
        local targetName = args[1]
        if not targetName then notify("Hedef belirt!") return end
        local target = Players:FindFirstChild(targetName)
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            HRP.CFrame = target.Character.HumanoidRootPart.CFrame + Vector3.new(2,0,0)
            notify("Gidiliyor: "..targetName)
        else
            notify("Oyuncu bulunamadı: "..targetName)
        end
    elseif cmd == "/bring" then
        local targetName = args[1]
        if not targetName then notify("Hedef belirt!") return end
        local target = Players:FindFirstChild(targetName)
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            target.Character.HumanoidRootPart.CFrame = HRP.CFrame + Vector3.new(2,0,0)
            notify("Getiriliyor: "..targetName)
        else
            notify("Oyuncu bulunamadı: "..targetName)
        end
    elseif cmd == "/reset" then
        if Humanoid then
            Humanoid.Health = 0
        end
    else
        notify("Bilinmeyen komut: "..cmd)
    end
end)

notify("Sengal Admin Pro v.5 yüklendi! Komutlar için / yaz ve enter'a bas.")
