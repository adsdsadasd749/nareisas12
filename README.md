local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- GUI

local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "ChamsFlyMenuGui"

local toggleMenuBtn = Instance.new("TextButton", screenGui)
toggleMenuBtn.Text = "☰"
toggleMenuBtn.Size = UDim2.new(0,40,0,40)
toggleMenuBtn.Position = UDim2.new(0,10,0,10)
toggleMenuBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleMenuBtn.TextColor3 = Color3.new(1,1,1)
toggleMenuBtn.Font = Enum.Font.SourceSansBold
toggleMenuBtn.TextScaled = true
toggleMenuBtn.AutoButtonColor = true

local menuFrame = Instance.new("Frame", screenGui)
menuFrame.Size = UDim2.new(0,200,0,240)
menuFrame.Position = UDim2.new(0,10,0,60)
menuFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
menuFrame.BorderSizePixel = 0
menuFrame.Visible = false

local function createButton(text, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSans
    btn.TextScaled = true
    btn.Text = text
    btn.AutoButtonColor = true
    btn.Parent = menuFrame
    return btn
end

local chamsBtn = createButton("Chams: OFF", 10)
local flyBtn = createButton("Fly: OFF", 60)
local speedBtn = createButton("Speed: 3", 110)

local invisBtn = createButton("Invisible: OFF", 160)
local bigHeadBtn = createButton("Big Head: OFF", 210)

toggleMenuBtn.MouseButton1Click:Connect(function()
    menuFrame.Visible = not menuFrame.Visible
end)

-- Чамсы

local chamsEnabled = false

local function addHighlightToCharacter(character)
    if character:FindFirstChild("Highlight") then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "Highlight"
    highlight.Adornee = character
    highlight.FillColor = Color3.new(1, 0, 0)
    highlight.OutlineColor = Color3.new(1, 1, 1)
    highlight.Parent = character
end

local function removeHighlight(character)
    local hl = character:FindFirstChild("Highlight")
    if hl then hl:Destroy() end
end

local function updateHighlights()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr.Character then
            if chamsEnabled then
                addHighlightToCharacter(plr.Character)
            else
                removeHighlight(plr.Character)
            end
        end
    end
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        if chamsEnabled then
            addHighlightToCharacter(char)
        end
    end)
end)

chamsBtn.MouseButton1Click:Connect(function()
    chamsEnabled = not chamsEnabled
    chamsBtn.Text = "Chams: " .. (chamsEnabled and "ON" or "OFF")
    updateHighlights()
end)

updateHighlights()

-- Флай

local flyEnabled = false
local speeds = {3, 10, 25, 50, 100}
local speedIndex = 1
local flySpeed = speeds[speedIndex]
local flying = false
local bodyVelocity
local bodyGyro

local function startFly()
    local character = player.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Velocity = Vector3.new(0,0,0)
    bodyVelocity.Parent = hrp

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bodyGyro.CFrame = hrp.CFrame
    bodyGyro.Parent = hrp

    flying = true
end

local function stopFly()
    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
    end
    if bodyGyro then
        bodyGyro:Destroy()
        bodyGyro = nil
    end
    flying = false
end

local function updateFly()
    if not flying then return end
    local character = player.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local moveDir = Vector3.new(0,0,0)
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        moveDir = moveDir + workspace.CurrentCamera.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        moveDir = moveDir - workspace.CurrentCamera.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        moveDir = moveDir - workspace.CurrentCamera.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        moveDir = moveDir + workspace.CurrentCamera.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        moveDir = moveDir + Vector3.new(0,1,0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
        moveDir = moveDir - Vector3.new(0,1,0)
    end

    if moveDir.Magnitude > 0 then
        moveDir = moveDir.Unit * flySpeed
    else
        moveDir = Vector3.new(0,0,0)
    end

    bodyVelocity.Velocity = moveDir
    bodyGyro.CFrame = workspace.CurrentCamera.CFrame
end

flyBtn.MouseButton1Click:Connect(function()
    flyEnabled = not flyEnabled
    flyBtn.Text = "Fly: " .. (flyEnabled and "ON" or "OFF")
    if flyEnabled then
        startFly()
    else
        stopFly()
    end
end)

speedBtn.MouseButton1Click:Connect(function()
    speedIndex = speedIndex + 1
    if speedIndex > #speeds then
        speedIndex = 1
    end
    flySpeed = speeds[speedIndex]
    speedBtn.Text = "Speed: " .. flySpeed
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if flyEnabled then
        updateFly()
    end
end)

-- Инвиз (реальный, для всех)

local invisible = false

local function setInvisible(state)
    local character = player.Character
    if not character then return end
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = state and 1 or 0
            if part:FindFirstChildOfClass("Decal") then
                for _, decal in pairs(part:GetChildren()) do
                    if decal:IsA("Decal") then
                        decal.Transparency = state and 1 or 0
                    end
                end
            end
        elseif part:IsA("Accessory") then
            part.Handle.Transparency = state and 1 or 0
        end
    end
end

invisBtn.MouseButton1Click:Connect(function()
    invisible = not invisible
    invisBtn.Text = "Invisible: " .. (invisible and "ON" or "OFF")
    setInvisible(invisible)
end)

-- Большая голова (реальная)

local bigHead = false

local function setBigHead(state)
    local character = player.Character
    if not character then return end
    local head = character:FindFirstChild("Head")
    if not head then return end

    if state then
        head.Size = Vector3.new(2, 2, 2)
    else
        head.Size = Vector3.new(1, 1, 1)
    end
end

bigHeadBtn.MouseButton1Click:Connect(function()
    bigHead = not bigHead
    bigHeadBtn.Text = "Big Head: " .. (bigHead and "ON" or "OFF")
    setBigHead(bigHead)
end)
