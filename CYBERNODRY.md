-- [[ CRÉDITOS: By: CyberNoDry ]] --

local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local pgui = player:WaitForChild("PlayerGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Limpeza de Versões Antigas
if pgui:FindFirstChild("CyberNet_V14_Global") then pgui.CyberNet_V14_Global:Destroy() end

local sg = Instance.new("ScreenGui", pgui)
sg.Name = "CyberNet_V14_Global"
sg.IgnoreGuiInset = true

-- Travar Personagem Real
local function setLock(state)
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then root.Anchored = state end
end
setLock(true)

local BG = Instance.new("Frame", sg)
BG.Size = UDim2.new(1, 0, 1, 0)
BG.BackgroundColor3 = Color3.new(0, 0, 0)
BG.Active = true

-- Variáveis de Jogo
local SalaAtual = "0000"
local MultiplayerAtivo = false
local velY, gravity, speed = 0, 0.8, 4

-- ==========================================
-- JANELA PRINCIPAL DO JOGO
-- ==========================================
local GameFrame = Instance.new("Frame", BG)
GameFrame.Size = UDim2.new(0, 750, 0, 480)
GameFrame.Position = UDim2.new(0.5, -375, 0.4, -240)
GameFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
GameFrame.ClipsDescendants = true
Instance.new("UICorner", GameFrame)

-- CONTADOR DE JOGADORES
local OnlineLabel = Instance.new("TextLabel", GameFrame)
OnlineLabel.Size = UDim2.new(1, 0, 0, 30)
OnlineLabel.Position = UDim2.new(0, 0, 0, 55)
OnlineLabel.Text = "JOGADORES NO SCRIPT: 1"
OnlineLabel.TextColor3 = Color3.new(0, 1, 0)
OnlineLabel.Font = Enum.Font.GothamBold
OnlineLabel.BackgroundTransparency = 1
OnlineLabel.TextSize = 18

-- ==========================================
-- SISTEMA DE CHAT GLOBAL (CUBINHOS)
-- ==========================================
local ChatWindow = Instance.new("Frame", GameFrame)
ChatWindow.Size = UDim2.new(0, 260, 0, 310)
ChatWindow.Position = UDim2.new(1, -270, 0, 100)
ChatWindow.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
ChatWindow.BackgroundTransparency = 0.2
ChatWindow.Visible = false -- Começa fechado
ChatWindow.ZIndex = 100
Instance.new("UICorner", ChatWindow)

local ChatScroll = Instance.new("ScrollingFrame", ChatWindow)
ChatScroll.Size = UDim2.new(1, -10, 1, -55)
ChatScroll.Position = UDim2.new(0, 5, 0, 5)
ChatScroll.BackgroundTransparency = 1
ChatScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
ChatScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
ChatScroll.ScrollBarThickness = 2
ChatScroll.ZIndex = 101

local ChatLayout = Instance.new("UIListLayout", ChatScroll)
ChatLayout.Padding = UDim.new(0, 6)
ChatLayout.SortOrder = Enum.SortOrder.LayoutOrder

local ChatInput = Instance.new("TextBox", ChatWindow)
ChatInput.Size = UDim2.new(1, -10, 0, 40)
ChatInput.Position = UDim2.new(0, 5, 1, -45)
ChatInput.PlaceholderText = "Falar Global (Liberal)..."
ChatInput.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
ChatInput.TextColor3 = Color3.new(1, 1, 1)
ChatInput.Font = Enum.Font.Gotham
ChatInput.ZIndex = 102
Instance.new("UICorner", ChatInput)

-- BOTÃO ABRIR/FECHAR CHAT
local ChatToggle = Instance.new("TextButton", GameFrame)
ChatToggle.Size = UDim2.new(0, 120, 0, 35)
ChatToggle.Position = UDim2.new(0, 10, 0, 100)
ChatToggle.Text = "ABRIR CHAT"
ChatToggle.BackgroundColor3 = Color3.fromRGB(120, 0, 255)
ChatToggle.TextColor3 = Color3.new(1, 1, 1)
ChatToggle.Font = Enum.Font.GothamBold
Instance.new("UICorner", ChatToggle)

ChatToggle.MouseButton1Click:Connect(function()
    ChatWindow.Visible = not ChatWindow.Visible
    ChatToggle.Text = ChatWindow.Visible and "FECHAR CHAT" or "ABRIR CHAT"
end)

-- BOLINHA E PLATAFORMAS
local myBall = Instance.new("Frame", GameFrame)
myBall.Size = UDim2.new(0, 25, 0, 25)
myBall.Position = UDim2.new(0, 50, 0, 350)
myBall.BackgroundColor3 = Color3.new(0, 1, 0)
myBall.ZIndex = 10
Instance.new("UICorner", myBall).CornerRadius = UDim.new(1, 0)

local platforms = {}
local function createPlat(x, y, w, nome)
    local p = Instance.new("Frame", GameFrame)
    p.Size = UDim2.new(0, w, 0, 15)
    p.Position = UDim2.new(0, x, 0, y)
    p.BackgroundColor3 = (nome == "Final") and Color3.new(1, 1, 0) or Color3.new(1, 1, 1)
    p.Name = nome or "P"
    Instance.new("UICorner", p)
    table.insert(platforms, p)
end
createPlat(0, 380, 750) 
createPlat(150, 300, 100)
createPlat(350, 220, 100)
createPlat(550, 140, 100, "Final")

-- LOBBY MULTIPLAYER (CÓDIGO DE SALA)
local MultiMenu = Instance.new("Frame", GameFrame)
MultiMenu.Size = UDim2.new(1, 0, 0, 50)
MultiMenu.BackgroundColor3 = Color3.fromRGB(35, 35, 35)

local CodigoTxt = Instance.new("TextLabel", MultiMenu)
CodigoTxt.Size = UDim2.new(0, 180, 1, 0)
CodigoTxt.Text = "SALA: " .. SalaAtual
CodigoTxt.TextColor3 = Color3.new(1, 1, 1)
CodigoTxt.BackgroundTransparency = 1

local JoinBtn = Instance.new("TextButton", MultiMenu)
JoinBtn.Size = UDim2.new(0, 90, 0, 30)
JoinBtn.Position = UDim2.new(1, -100, 0.5, -15)
JoinBtn.Text = "ENTRAR"
JoinBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
JoinBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", JoinBtn)

local CreateBtn = Instance.new("TextButton", MultiMenu)
CreateBtn.Size = UDim2.new(0, 90, 0, 30)
CreateBtn.Position = UDim2.new(1, -200, 0.5, -15)
CreateBtn.Text = "CRIAR"
CreateBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 150)
CreateBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", CreateBtn)

-- ==========================================
-- LÓGICA DE ATUALIZAÇÃO E SYNC
-- ==========================================
local remotePlayers = {}
local processedMsgs = {}

RunService.RenderStepped:Connect(function()
    -- Contador Online (Servidor Atual)
    local c = 0
    for _,p in pairs(game.Players:GetPlayers()) do
        if p:FindFirstChild("PlayerGui") and p.PlayerGui:FindFirstChild("CyberNet_V14_Global") then c = c + 1 end
    end
    OnlineLabel.Text = "JOGADORES NO SCRIPT: " .. c

    -- Movimento W/D (Direita) e A/S (Esquerda)
    local d = 0
    if UserInputService:IsKeyDown(Enum.KeyCode.D) or UserInputService:IsKeyDown(Enum.KeyCode.W) then d = 1 
    elseif UserInputService:IsKeyDown(Enum.KeyCode.A) or UserInputService:IsKeyDown(Enum.KeyCode.S) then d = -1 end
    myBall.Position = myBall.Position + UDim2.new(0, d * speed, 0, 0)
    velY = velY + gravity
    myBall.Position = myBall.Position + UDim2.new(0, 0, 0, velY)

    -- Física de Colisão
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
    if myBall.Position.Y.Offset > 500 then myBall.Position = UDim2.new(0, 50, 0, 350) velY = 0 end

    -- SYNC DO CHAT (CUBINHOS LIBERAIS)
    for _, p in pairs(game.Players:GetPlayers()) do
        local globalMsg = p:GetAttribute("CyberGlobalMsg")
        if globalMsg and not processedMsgs[globalMsg] then
            processedMsgs[globalMsg] = true
            
            local bubble = Instance.new("Frame", ChatScroll)
            bubble.Size = UDim2.new(1, -10, 0, 35)
            bubble.BackgroundColor3 = (p == player) and Color3.fromRGB(0, 120, 255) or Color3.fromRGB(50, 50, 50)
            Instance.new("UICorner", bubble)
            
            local t = Instance.new("TextLabel", bubble)
            t.Size = UDim2.new(1, -10, 1, 0)
            t.Position = UDim2.new(0, 5, 0, 0)
            t.BackgroundTransparency = 1
            t.TextColor3 = Color3.new(1,1,1)
            t.Text = p.Name .. ": " .. globalMsg:split("||")[1]
            t.TextXAlignment = Enum.TextXAlignment.Left
            t.TextWrapped = true
            t.Font = Enum.Font.Gotham
            ChatScroll.CanvasPosition = Vector2.new(0, 9999)
        end
    end

    -- Sync Bola Multiplayer
    if MultiplayerAtivo then
        player:SetAttribute("CyberPosV14", SalaAtual.."|"..myBall.Position.X.Offset.."|"..myBall.Position.Y.Offset)
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= player then
                local data = p:GetAttribute("CyberPosV14")
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

-- Enviar Mensagem Global
ChatInput.FocusLost:Connect(function(enter)
    if enter and ChatInput.Text ~= "" then
        player:SetAttribute("CyberGlobalMsg", ChatInput.Text .. "||" .. tick())
        ChatInput.Text = ""
    end
end)

-- Controles Adicionais
local function jump() if velY == 0 then velY = -15 end end
UserInputService.InputBegan:Connect(function(i,g) if not g and i.KeyCode == Enum.KeyCode.Space then jump() end end)

CreateBtn.MouseButton1Click:Connect(function() SalaAtual = tostring(math.random(1000,9999)) CodigoTxt.Text = "SALA: "..SalaAtual MultiplayerAtivo = true end)
JoinBtn.MouseButton1Click:Connect(function() 
    local inp = Instance.new("TextBox", MultiMenu)
    inp.Size = UDim2.new(0, 100, 0, 30)
    inp.Position = UDim2.new(0.5, -50, 1, 5)
    inp.PlaceholderText = "Código..."
    inp.FocusLost:Connect(function(e) if e then SalaAtual = inp.Text CodigoTxt.Text = "SALA: "..SalaAtual MultiplayerAtivo = true inp:Destroy() end end)
end)

-- Botão de Sair
local Exit = Instance.new("TextButton", GameFrame)
Exit.Size = UDim2.new(0, 30, 0, 30)
Exit.Position = UDim2.new(1, -35, 0, 55)
Exit.Text = "X"
Exit.BackgroundColor3 = Color3.new(1,0,0)
Exit.MouseButton1Click:Connect(function() setLock(false) sg:Destroy() end)
