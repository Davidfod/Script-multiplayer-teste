-- [[ CRÉDITOS: By: CyberNoDry ]] --

local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local pgui = player:WaitForChild("PlayerGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Limpeza
if pgui:FindFirstChild("CyberNet_Fixed_V11") then pgui.CyberNet_Fixed_V11:Destroy() end

local sg = Instance.new("ScreenGui")
sg.Name = "CyberNet_Fixed_V11"
sg.Parent = pgui
sg.IgnoreGuiInset = true

-- Congelar personagem real
local function setLock(state)
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then root.Anchored = state end
end
setLock(true)

local BG = Instance.new("Frame", sg)
BG.Size = UDim2.new(1, 0, 1, 0)
BG.BackgroundColor3 = Color3.new(0, 0, 0)
BG.Active = true

-- Variáveis
local SalaAtual = "0000"
local MultiplayerAtivo = false
local velY = 0
local gravity = 0.8 
local speed = 4 

-- Janela do Jogo
local GameFrame = Instance.new("Frame", BG)
GameFrame.Size = UDim2.new(0, 700, 0, 450)
GameFrame.Position = UDim2.new(0.5, -350, 0.4, -225)
GameFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
GameFrame.ClipsDescendants = true
Instance.new("UICorner", GameFrame)

-- JANELA DE CHAT
local ChatFrame = Instance.new("Frame", GameFrame)
ChatFrame.Size = UDim2.new(0, 200, 0, 250)
ChatFrame.Position = UDim2.new(1, -210, 0, 100)
ChatFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ChatFrame.Visible = false
ChatFrame.ZIndex = 20
Instance.new("UICorner", ChatFrame)

local ChatScroll = Instance.new("ScrollingFrame", ChatFrame)
ChatScroll.Size = UDim2.new(1, -10, 1, -45)
ChatScroll.Position = UDim2.new(0, 5, 0, 5)
ChatScroll.BackgroundTransparency = 1
ChatScroll.CanvasSize = UDim2.new(0, 0, 10, 0)
local ChatList = Instance.new("UIListLayout", ChatScroll)
ChatList.VerticalAlignment = Enum.VerticalAlignment.Bottom

local ChatInput = Instance.new("TextBox", ChatFrame)
ChatInput.Size = UDim2.new(1, -10, 0, 30)
ChatInput.Position = UDim2.new(0, 5, 1, -35)
ChatInput.PlaceholderText = "Chat..."
ChatInput.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ChatInput.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", ChatInput)

local ChatToggle = Instance.new("TextButton", GameFrame)
ChatToggle.Size = UDim2.new(0, 60, 0, 30)
ChatToggle.Position = UDim2.new(0, 10, 0, 90)
ChatToggle.Text = "CHAT"
ChatToggle.BackgroundColor3 = Color3.fromRGB(100, 0, 200)
ChatToggle.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", ChatToggle)
ChatToggle.MouseButton1Click:Connect(function() ChatFrame.Visible = not ChatFrame.Visible end)

-- Contador
local OnlineLabel = Instance.new("TextLabel", GameFrame)
OnlineLabel.Size = UDim2.new(1, 0, 0, 30)
OnlineLabel.Position = UDim2.new(0, 0, 0, 55)
OnlineLabel.Text = "MEMBROS USANDO: 1"
OnlineLabel.TextColor3 = Color3.new(0, 1, 0)
OnlineLabel.Font = Enum.Font.GothamBold
OnlineLabel.BackgroundTransparency = 1
OnlineLabel.TextSize = 18

-- Bolinha
local myBall = Instance.new("Frame", GameFrame)
myBall.Size = UDim2.new(0, 25, 0, 25)
myBall.Position = UDim2.new(0, 50, 0, 350)
myBall.BackgroundColor3 = Color3.new(0, 1, 0)
myBall.ZIndex = 15
Instance.new("UICorner", myBall).CornerRadius = UDim.new(1, 0)

-- Plataformas
local platforms = {}
local function createPlat(x, y, w, nome)
    local p = Instance.new("Frame", GameFrame)
    p.Size = UDim2.new(0, w, 0, 15)
    p.Position = UDim2.new(0, x, 0, y)
    p.BackgroundColor3 = (nome == "Final") and Color3.new(1, 1, 0) or Color3.new(1, 1, 1)
    p.Name = nome or "Plataforma"
    Instance.new("UICorner", p)
    table.insert(platforms, p)
end
createPlat(0, 380, 700) 
createPlat(150, 300, 100)
createPlat(350, 220, 100)
createPlat(550, 140, 100, "Final")

-- MULTIPLAYER MENU
local MultiMenu = Instance.new("Frame", GameFrame)
MultiMenu.Size = UDim2.new(1, 0, 0, 50)
MultiMenu.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

local CodigoTxt = Instance.new("TextLabel", MultiMenu)
CodigoTxt.Size = UDim2.new(0, 150, 1, 0)
CodigoTxt.Text = "SALA: " .. SalaAtual
CodigoTxt.TextColor3 = Color3.new(1, 1, 1)
CodigoTxt.BackgroundTransparency = 1

local JoinBtn = Instance.new("TextButton", MultiMenu)
JoinBtn.Size = UDim2.new(0, 100, 0, 30)
JoinBtn.Position = UDim2.new(1, -110, 0.5, -15)
JoinBtn.Text = "ENTRAR"
JoinBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
JoinBtn.TextColor3 = Color3.new(1,1,1)

local CreateBtn = Instance.new("TextButton", MultiMenu)
CreateBtn.Size = UDim2.new(0, 100, 0, 30)
CreateBtn.Position = UDim2.new(1, -220, 0.5, -15)
CreateBtn.Text = "CRIAR"
CreateBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 150)
CreateBtn.TextColor3 = Color3.new(1,1,1)

-- FISICA E SYNC
local remotePlayers = {}
RunService.RenderStepped:Connect(function()
    -- Contador Online
    local c = 0
    for _,p in pairs(game.Players:GetPlayers()) do
        if p:FindFirstChild("PlayerGui") and p.PlayerGui:FindFirstChild("CyberNet_Fixed_V11") then c = c + 1 end
    end
    OnlineLabel.Text = "MEMBROS USANDO: "..c

    -- Movimentação (W/D e A/S)
    local d = 0
    if UserInputService:IsKeyDown(Enum.KeyCode.D) or UserInputService:IsKeyDown(Enum.KeyCode.W) then d = 1 
    elseif UserInputService:IsKeyDown(Enum.KeyCode.A) or UserInputService:IsKeyDown(Enum.KeyCode.S) then d = -1 end
    
    myBall.Position = myBall.Position + UDim2.new(0, d * speed, 0, 0)
    velY = velY + gravity
    myBall.Position = myBall.Position + UDim2.new(0, 0, 0, velY)

    -- Colisão fixa (Sem cair do mapa)
    for _, plat in pairs(platforms) do
        local bX = myBall.Position.X.Offset
        local bY = myBall.Position.Y.Offset
        local pX = plat.Position.X.Offset
        local pY = plat.Position.Y.Offset
        local pW = plat.Size.X.Offset
        if bX + 25 > pX and bX < pX + pW then
            if bY + 25 >= pY and bY + 25 <= pY + 15 and velY >= 0 then
                if plat.Name == "Final" then
                    myBall.Position = UDim2.new(0, 50, 0, 350)
                    velY = 0
                else
                    myBall.Position = UDim2.new(0, bX, 0, pY - 25)
                    velY = 0
                end
            end
        end
    end

    if myBall.Position.Y.Offset > 500 then
        myBall.Position = UDim2.new(0, 50, 0, 350)
        velY = 0
    end

    -- Sync Chat
    for _, p in pairs(game.Players:GetPlayers()) do
        local msg = p:GetAttribute("CyberChat")
        if msg and msg ~= p:GetAttribute("LastMsgShown") then
            local t = Instance.new("TextLabel", ChatScroll)
            t.Size = UDim2.new(1, 0, 0, 20)
            t.Text = p.Name..": "..msg
            t.TextColor3 = Color3.new(1,1,1)
            t.BackgroundTransparency = 1
            t.TextXAlignment = Enum.TextXAlignment.Left
            p:SetAttribute("LastMsgShown", msg)
        end
    end

    -- Sync Multiplayer Bola
    if MultiplayerAtivo then
        player:SetAttribute("CyberPos", SalaAtual.."|"..myBall.Position.X.Offset.."|"..myBall.Position.Y.Offset)
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= player then
                local data = p:GetAttribute("CyberPos")
                if data then
                    local s = string.split(data, "|")
                    if s[1] == SalaAtual then
                        if not remotePlayers[p.Name] then
                            local b = Instance.new("Frame", GameFrame)
                            b.Size = UDim2.new(0, 25, 0, 25)
                            b.BackgroundColor3 = Color3.new(1,0,0)
                            Instance.new("UICorner", b).CornerRadius = UDim.new(1,0)
                            remotePlayers[p.Name] = b
                        end
                        remotePlayers[p.Name].Position = UDim2.new(0, tonumber(s[2]), 0, tonumber(s[3]))
                        remotePlayers[p.Name].Visible = true
                    elseif remotePlayers[p.Name] then remotePlayers[p.Name].Visible = false end
                end
            end
        end
    end
end)

ChatInput.FocusLost:Connect(function(e) if e and ChatInput.Text ~= "" then player:SetAttribute("CyberChat", ChatInput.Text) ChatInput.Text = "" end end)

local function jump() if velY == 0 then velY = -15 end end
UserInputService.InputBegan:Connect(function(i,g) if not g and i.KeyCode == Enum.KeyCode.Space then jump() end end)

CreateBtn.MouseButton1Click:Connect(function() SalaAtual = tostring(math.random(1000,9999)) CodigoTxt.Text = "SALA: "..SalaAtual MultiplayerAtivo = true end)
JoinBtn.MouseButton1Click:Connect(function() 
    local inp = Instance.new("TextBox", MultiMenu)
    inp.Size = UDim2.new(0, 100, 0, 30)
    inp.Position = UDim2.new(0.5, -50, 1, 5)
    inp.FocusLost:Connect(function(e) if e then SalaAtual = inp.Text CodigoTxt.Text = "SALA: "..SalaAtual MultiplayerAtivo = true inp:Destroy() end end)
end)

local Exit = Instance.new("TextButton", GameFrame)
Exit.Size = UDim2.new(0, 30, 0, 30)
Exit.Position = UDim2.new(1, -35, 0, 55)
Exit.Text = "X"
Exit.BackgroundColor3 = Color3.new(1,0,0)
Exit.MouseButton1Click:Connect(function() setLock(false); sg:Destroy() end)
