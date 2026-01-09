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

-- CONTADOR ONLINE CORRIGIDO
local OnlineLabel = Instance.new("TextLabel", GameFrame)
OnlineLabel.Size = UDim2.new(1, 0, 0, 30)
OnlineLabel.Position = UDim2.new(0, 0, 0, 55)
OnlineLabel.Text = "MEMBROS USANDO: 1"
OnlineLabel.TextColor3 = Color3.new(0, 1, 0)
OnlineLabel.Font = Enum.Font.GothamBold
OnlineLabel.BackgroundTransparency = 1
OnlineLabel.TextSize = 18

-- Sua Bola
local myBall = Instance.new("Frame", GameFrame)
myBall.Size = UDim2.new(0, 25, 0, 25)
myBall.Position = UDim2.new(0, 50, 0, 350)
myBall.BackgroundColor3 = Color3.new(0, 1, 0)
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

-- INTERFACE MULTIPLAYER
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

-- CONTROLES MOBILE
local MobileBtns = Instance.new("Frame", BG)
MobileBtns.Size = UDim2.new(0, 300, 0, 100)
MobileBtns.Position = UDim2.new(0.5, -150, 0.82, 0)
MobileBtns.BackgroundTransparency = 1

local moveL, moveR = false, false
local function mkBtn(pos, txt, cb)
    local b = Instance.new("TextButton", MobileBtns)
    b.Size = UDim2.new(0, 70, 0, 70)
    b.Position = pos
    b.Text = txt
    b.BackgroundColor3 = Color3.fromRGB(60,60,60)
    b.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", b)
    b.MouseButton1Down:Connect(function() cb(true) end)
    b.MouseButton1Up:Connect(function() cb(false) end)
end
mkBtn(UDim2.new(0, 0, 0, 0), "◄", function(v) moveL = v end)
mkBtn(UDim2.new(0, 80, 0, 0), "►", function(v) moveR = v end)

local JumpBtn = Instance.new("TextButton", MobileBtns)
JumpBtn.Size = UDim2.new(0, 100, 0, 70)
JumpBtn.Position = UDim2.new(0, 180, 0, 0)
JumpBtn.Text = "PULO"
JumpBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
JumpBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", JumpBtn)

-- LÓGICA DE FÍSICA E SYNC
local remotePlayers = {}
RunService.RenderStepped:Connect(function()
    -- CONTADOR ONLINE REAL (Verifica quem está com a GUI ativa)
    local count = 0
    for _, p in pairs(game.Players:GetPlayers()) do
        local gui = p:FindFirstChild("PlayerGui")
        if gui and gui:FindFirstChild("CyberNet_Fixed_V11") then
            count = count + 1
        end
    end
    OnlineLabel.Text = "MEMBROS USANDO: " .. count
    player:SetAttribute("CyberOnline", true)

    -- Movimentação
    local d = 0
    if UserInputService:IsKeyDown(Enum.KeyCode.D) or UserInputService:IsKeyDown(Enum.KeyCode.W) or moveR then d = 1 
    elseif UserInputService:IsKeyDown(Enum.KeyCode.A) or UserInputService:IsKeyDown(Enum.KeyCode.S) or moveL then d = -1 end
    
    myBall.Position = myBall.Position + UDim2.new(0, d * speed, 0, 0)
    velY = velY + gravity
    myBall.Position = myBall.Position + UDim2.new(0, 0, 0, velY)

    -- Colisão
    for _, plat in pairs(platforms) do
        local bX, bY = myBall.Position.X.Offset, myBall.Position.Y.Offset
        local pX, pY, pW = plat.Position.X.Offset, plat.Position.Y.Offset, plat.Size.X.Offset
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
    
    -- MULTIPLAYER SYNC AUTOMÁTICO
    if MultiplayerAtivo then
        player:SetAttribute("CyberPos", SalaAtual .. "|" .. myBall.Position.X.Offset .. "|" .. myBall.Position.Y.Offset)
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= player then
                local data = p:GetAttribute("CyberPos")
                if data then
                    local s = string.split(data, "|")
                    if s[1] == SalaAtual then
                        if not remotePlayers[p.Name] then
                            local b = Instance.new("Frame", GameFrame)
                            b.Size = UDim2.new(0, 25, 0, 25)
                            b.BackgroundColor3 = Color3.new(1,0,0) -- Bola dos outros
                            Instance.new("UICorner", b).CornerRadius = UDim.new(1,0)
                            remotePlayers[p.Name] = b
                        end
                        remotePlayers[p.Name].Position = UDim2.new(0, tonumber(s[2]), 0, tonumber(s[3]))
                        remotePlayers[p.Name].Visible = true
                    else
                        if remotePlayers[p.Name] then remotePlayers[p.Name].Visible = false end
                    end
                end
            end
        end
    end
end)

local function jump() if velY == 0 then velY = -15 end end
JumpBtn.MouseButton1Click:Connect(jump)
UserInputService.InputBegan:Connect(function(i, g) if not g and i.KeyCode == Enum.KeyCode.Space then jump() end end)

-- BOTÕES DE SALA
CreateBtn.MouseButton1Click:Connect(function()
    SalaAtual = tostring(math.random(1000,9999))
    CodigoTxt.Text = "SALA: "..SalaAtual
    MultiplayerAtivo = true
end)

local currentInput = nil
JoinBtn.MouseButton1Click:Connect(function()
    if currentInput then
        currentInput:Destroy()
        currentInput = nil
    else
        currentInput = Instance.new("TextBox", MultiMenu)
        currentInput.Size = UDim2.new(0, 100, 0, 30)
        currentInput.Position = UDim2.new(0.5, -50, 1, 5)
        currentInput.PlaceholderText = "Cod..."
        currentInput.BackgroundColor3 = Color3.fromRGB(60,60,60)
        currentInput.TextColor3 = Color3.new(1,1,1)
        Instance.new("UICorner", currentInput)
        
        currentInput.FocusLost:Connect(function(enter)
            if enter then
                SalaAtual = currentInput.Text
                CodigoTxt.Text = "SALA: "..SalaAtual
                MultiplayerAtivo = true -- ATIVA O MULTIPLAYER NA HORA
                currentInput:Destroy()
                currentInput = nil
            end
        end)
    end
end)

local Exit = Instance.new("TextButton", GameFrame)
Exit.Size = UDim2.new(0, 30, 0, 30)
Exit.Position = UDim2.new(1, -35, 0, 55)
Exit.Text = "X"
Exit.BackgroundColor3 = Color3.new(1,0,0)
Exit.MouseButton1Click:Connect(function() setLock(false); sg:Destroy() end)

local Hide = Instance.new("TextButton", sg)
Hide.Size = UDim2.new(0, 30, 0, 30)
Hide.Position = UDim2.new(0, 10, 0, 10)
Hide.Text = "👁️"
Hide.MouseButton1Click:Connect(function() BG.Visible = not BG.Visible end)
