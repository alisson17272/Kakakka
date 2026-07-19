-- ============================================
-- HACK LAB v11.0 - ESP + SHIFTLOCK + REMOVER NEBLINA
-- ============================================
local Players=game:GetService("Players");local LP=Players.LocalPlayer;local RS=game:GetService("RunService");local UIS=game:GetService("UserInputService");local Camera=workspace.CurrentCamera;local plr=LP;local char=nil;local root=nil

-- ===== CONFIGS =====
local ESP_ON=false
local SHIFTLOCK_ON=false
local NEBLINA_ON=false
local COR=Color3.fromRGB(255,0,0)
local highlights={}
local shiftlockAtivo=false
local anguloHorizontal=0

-- ===== FUNÇÕES AUXILIARES =====
local function getChar() char=plr.Character;if char then root=char:FindFirstChild("HumanoidRootPart") end;return char end

-- ===== ESP =====
local function criarESP(personagem) 
    if not personagem then return nil end
    local h=Instance.new("Highlight")
    h.Name="ESP"
    h.Adornee=personagem
    h.OutlineColor=COR
    h.FillColor=COR
    h.FillTransparency=1
    h.OutlineTransparency=0
    h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent=personagem
    return h
end

local function removerESP() 
    for _,h in pairs(highlights)do 
        if h and h.Parent then h:Destroy()end 
    end
    highlights={} 
end

local function atualizarESP() 
    if ESP_ON then 
        for _,j in pairs(Players:GetPlayers())do 
            if j~=plr then 
                local c=j.Character
                if c and not highlights[j]then 
                    local h=criarESP(c)
                    if h then highlights[j]=h end 
                end 
            end 
        end 
    else 
        removerESP() 
    end 
end

-- ===== REMOVER NEBLINA =====
local function removerNeblina()
    -- Remove fog/neblina do jogo
    pcall(function()
        local lighting = game:GetService("Lighting")
        lighting.FogEnd = 100000
        lighting.FogStart = 0
        lighting.FogColor = Color3.fromRGB(255,255,255)
        lighting.Brightness = 2
        lighting.ClockTime = 12 -- Meio-dia
        lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)
        lighting.Ambient = Color3.fromRGB(255,255,255)
        
        -- Remove qualquer atmosfera
        for _, atmo in pairs(lighting:GetChildren()) do
            if atmo:IsA("Atmosphere") then
                atmo:Destroy()
            end
        end
        print("🌤️ Neblina removida!")
    end)
    
    -- Remove de todos os ambientes
    pcall(function()
        for _, atmo in pairs(workspace:GetDescendants()) do
            if atmo:IsA("Atmosphere") or atmo:IsA("Fog") then
                atmo:Destroy()
            end
        end
    end)
end

-- ===== SHIFTLOCK (BOLINHA) =====
local function toggleShiftlock()
    shiftlockAtivo = not shiftlockAtivo
    if shiftlockAtivo then
        -- Pega a direção atual do personagem
        local c=getChar()
        if c and root then
            anguloHorizontal = root.Orientation.Y
        end
        print("🔒 Shiftlock ATIVADO")
    else
        print("🔓 Shiftlock DESATIVADO")
    end
end

-- ===== LOOP PRINCIPAL (SHIFTLOCK) =====
RS.RenderStepped:Connect(function()
    local c=getChar()
    
    -- SHIFTLOCK (mantém a câmera travada para frente)
    if SHIFTLOCK_ON and shiftlockAtivo then
        pcall(function()
            if root then
                -- Trava a rotação da câmera na direção do personagem
                local cf = root.CFrame
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, cf.Position + cf.LookVector * 100)
            end
        end)
    end
    
    -- ESP
    if ESP_ON then
        for _,j in pairs(Players:GetPlayers())do
            if j~=plr then
                local c2=j.Character
                if c2 then
                    if not highlights[j]then
                        local h=criarESP(c2)
                        if h then highlights[j]=h end
                    elseif highlights[j].Adornee~=c2 then
                        highlights[j]:Destroy()
                        highlights[j]=nil
                        local h=criarESP(c2)
                        if h then highlights[j]=h end
                    end
                else
                    if highlights[j]then 
                        highlights[j]:Destroy()
                        highlights[j]=nil 
                    end
                end
            end
        end
    end
end)

-- ===== MENU =====
local menuAberto=true
local sg=nil
local f=nil
local btnReabrir=nil
local bolinha=nil

local function criarMenu()
    sg=Instance.new("ScreenGui")
    sg.Name="HackLab"
    sg.ResetOnSpawn=false
    sg.Parent=plr:WaitForChild("PlayerGui")
    
    -- ===== FRAME PRINCIPAL =====
    f=Instance.new("Frame")
    f.Size=UDim2.new(0,260,0,260)
    f.Position=UDim2.new(0.5,-130,0.5,-130)
    f.BackgroundColor3=Color3.fromRGB(10,10,20)
    f.BackgroundTransparency=0.1
    f.BorderSizePixel=0
    f.ClipsDescendants=true
    f.Parent=sg
    
    local c=Instance.new("UICorner")
    c.CornerRadius=UDim.new(0,12)
    c.Parent=f
    
    -- TÍTULO
    local t=Instance.new("TextLabel")
    t.Size=UDim2.new(1,0,0,35)
    t.Position=UDim2.new(0,0,0,5)
    t.BackgroundTransparency=1
    t.Text="🔧 HACK LAB"
    t.TextColor3=Color3.fromRGB(255,255,255)
    t.TextScaled=true
    t.Font=Enum.Font.GothamBold
    t.Parent=f
    
    -- BOTÃO FECHAR (X)
    local btnX=Instance.new("TextButton")
    btnX.Size=UDim2.new(0,35,0,35)
    btnX.Position=UDim2.new(1,-40,0,5)
    btnX.BackgroundColor3=Color3.fromRGB(200,50,50)
    btnX.Text="✕"
    btnX.TextColor3=Color3.fromRGB(255,255,255)
    btnX.TextScaled=true
    btnX.Font=Enum.Font.GothamBold
    btnX.Parent=f
    
    local cx=Instance.new("UICorner")
    cx.CornerRadius=UDim.new(0,8)
    cx.Parent=btnX
    
    -- SUBTÍTULO
    local st=Instance.new("TextLabel")
    st.Size=UDim2.new(1,0,0,15)
    st.Position=UDim2.new(0,0,0,37)
    st.BackgroundTransparency=1
    st.Text="ESP + SHIFTLOCK + REMOVER NEBLINA"
    st.TextColor3=Color3.fromRGB(150,150,150)
    st.TextScaled=true
    st.Font=Enum.Font.Gotham
    st.Parent=f
    
    -- LINHA DIVISÓRIA
    local linha=Instance.new("Frame")
    linha.Size=UDim2.new(0.9,0,0,1)
    linha.Position=UDim2.new(0.05,0,0,55)
    linha.BackgroundColor3=Color3.fromRGB(255,255,255)
    linha.BackgroundTransparency=0.3
    linha.Parent=f
    
    -- ===== FUNÇÃO PARA CRIAR BOTÕES =====
    local function criarBotao(nome,pos,cor)
        local b=Instance.new("TextButton")
        b.Size=UDim2.new(0.9,0,0,40)
        b.Position=UDim2.new(0.05,0,0,pos)
        b.BackgroundColor3=cor or Color3.fromRGB(40,40,50)
        b.BackgroundTransparency=0.2
        b.Text=nome
        b.TextColor3=Color3.fromRGB(255,255,255)
        b.TextScaled=true
        b.Font=Enum.Font.GothamBold
        b.Parent=f
        
        local cb=Instance.new("UICorner")
        cb.CornerRadius=UDim.new(0,6)
        cb.Parent=b
        return b
    end
    
    -- ===== BOTÃO ESP =====
    local bESP=criarBotao("🔴 ESP: OFF",65)
    
    -- ===== BOTÃO SHIFTLOCK =====
    local bSHIFT=criarBotao("🔒 SHIFTLOCK: OFF",115,Color3.fromRGB(50,100,200))
    
    -- ===== BOTÃO NEBLINA =====
    local bNEBLINA=criarBotao("🌤️ REMOVER NEBLINA: OFF",165,Color3.fromRGB(200,150,50))
    
    -- ===== FUNÇÕES DOS BOTÕES =====
    -- ESP
    bESP.MouseButton1Click:Connect(function()
        ESP_ON=not ESP_ON
        bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF"
        bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50)
        atualizarESP()
    end)
    bESP.TouchTap:Connect(function()
        ESP_ON=not ESP_ON
        bESP.Text=ESP_ON and"🔴 ESP: ON"or"🔴 ESP: OFF"
        bESP.BackgroundColor3=ESP_ON and Color3.fromRGB(200,50,50)or Color3.fromRGB(40,40,50)
        atualizarESP()
    end)
    
    -- SHIFTLOCK
    bSHIFT.MouseButton1Click:Connect(function()
        SHIFTLOCK_ON=not SHIFTLOCK_ON
        bSHIFT.Text=SHIFTLOCK_ON and"🔒 SHIFTLOCK: ON"or"🔒 SHIFTLOCK: OFF"
        bSHIFT.BackgroundColor3=SHIFTLOCK_ON and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)
        
        if SHIFTLOCK_ON then
            -- Cria a bolinha
            if not bolinha then
                bolinha=Instance.new("TextButton")
                bolinha.Size=UDim2.new(0,60,0,60)
                bolinha.Position=UDim2.new(0.5,-30,0.5,-30)
                bolinha.BackgroundColor3=Color3.fromRGB(50,100,200)
                bolinha.BackgroundTransparency=0.5
                bolinha.Text="🔒"
                bolinha.TextColor3=Color3.fromRGB(255,255,255)
                bolinha.TextScaled=true
                bolinha.Font=Enum.Font.GothamBold
                bolinha.Parent=sg
                
                local bc=Instance.new("UICorner")
                bc.CornerRadius=UDim.new(0,30)
                bc.Parent=bolinha
                
                -- Borda da bolinha
                local borda=Instance.new("Frame")
                borda.Size=UDim2.new(1,2,1,2)
                borda.Position=UDim2.new(0,-1,0,-1)
                borda.BackgroundColor3=Color3.fromRGB(255,255,255)
                borda.BackgroundTransparency=0.3
                borda.BorderSizePixel=0
                borda.Parent=bolinha
                local bc2=Instance.new("UICorner")
                bc2.CornerRadius=UDim.new(0,30)
                bc2.Parent=borda
                
                -- Clique na bolinha
                bolinha.MouseButton1Click:Connect(function()
                    toggleShiftlock()
                    bolinha.BackgroundColor3=shiftlockAtivo and Color3.fromRGB(0,255,0)or Color3.fromRGB(50,100,200)
                    bolinha.Text=shiftlockAtivo and "🔒" or "🔓"
                end)
                bolinha.TouchTap:Connect(function()
                    toggleShiftlock()
                    bolinha.BackgroundColor3=shiftlockAtivo and Color3.fromRGB(0,255,0)or Color3.fromRGB(50,100,200)
                    bolinha.Text=shiftlockAtivo and "🔒" or "🔓"
                end)
                
                -- Arrastar bolinha
                local drag=false
                local startPos=nil
                local startPos2=nil
                
                bolinha.InputBegan:Connect(function(input)
                    if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                        drag=true
                        startPos=input.Position
                        startPos2=bolinha.Position
                    end
                end)
                bolinha.InputChanged:Connect(function(input)
                    if drag and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
                        local delta=input.Position-startPos
                        bolinha.Position=UDim2.new(startPos2.X.Scale,startPos2.X.Offset+delta.X,startPos2.Y.Scale,startPos2.Y.Offset+delta.Y)
                    end
                end)
                bolinha.InputEnded:Connect(function(input)
                    if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                        drag=false
                    end
                end)
            end
            bolinha.Visible=true
            print("🔒 Shiftlock ativado! Clique na bolinha para ligar/desligar.")
        else
            if bolinha then
                bolinha.Visible=false
                shiftlockAtivo=false
            end
            print("🔓 Shiftlock desativado!")
        end
    end)
    bSHIFT.TouchTap:Connect(function()
        SHIFTLOCK_ON=not SHIFTLOCK_ON
        bSHIFT.Text=SHIFTLOCK_ON and"🔒 SHIFTLOCK: ON"or"🔒 SHIFTLOCK: OFF"
        bSHIFT.BackgroundColor3=SHIFTLOCK_ON and Color3.fromRGB(50,100,200)or Color3.fromRGB(40,40,50)
        
        if SHIFTLOCK_ON then
            if not bolinha then
                bolinha=Instance.new("TextButton")
                bolinha.Size=UDim2.new(0,60,0,60)
                bolinha.Position=UDim2.new(0.5,-30,0.5,-30)
                bolinha.BackgroundColor3=Color3.fromRGB(50,100,200)
                bolinha.BackgroundTransparency=0.5
                bolinha.Text="🔒"
                bolinha.TextColor3=Color3.fromRGB(255,255,255)
                bolinha.TextScaled=true
                bolinha.Font=Enum.Font.GothamBold
                bolinha.Parent=sg
                
                local bc=Instance.new("UICorner")
                bc.CornerRadius=UDim.new(0,30)
                bc.Parent=bolinha
                
                local borda=Instance.new("Frame")
                borda.Size=UDim2.new(1,2,1,2)
                borda.Position=UDim2.new(0,-1,0,-1)
                borda.BackgroundColor3=Color3.fromRGB(255,255,255)
                borda.BackgroundTransparency=0.3
                borda.BorderSizePixel=0
                borda.Parent=bolinha
                local bc2=Instance.new("UICorner")
                bc2.CornerRadius=UDim.new(0,30)
                bc2.Parent=borda
                
                bolinha.MouseButton1Click:Connect(function()
                    toggleShiftlock()
                    bolinha.BackgroundColor3=shiftlockAtivo and Color3.fromRGB(0,255,0)or Color3.fromRGB(50,100,200)
                    bolinha.Text=shiftlockAtivo and "🔒" or "🔓"
                end)
                bolinha.TouchTap:Connect(function()
                    toggleShiftlock()
                    bolinha.BackgroundColor3=shiftlockAtivo and Color3.fromRGB(0,255,0)or Color3.fromRGB(50,100,200)
                    bolinha.Text=shiftlockAtivo and "🔒" or "🔓"
                end)
                
                local drag=false
                local startPos=nil
                local startPos2=nil
                
                bolinha.InputBegan:Connect(function(input)
                    if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                        drag=true
                        startPos=input.Position
                        startPos2=bolinha.Position
                    end
                end)
                bolinha.InputChanged:Connect(function(input)
                    if drag and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
                        local delta=input.Position-startPos
                        bolinha.Position=UDim2.new(startPos2.X.Scale,startPos2.X.Offset+delta.X,startPos2.Y.Scale,startPos2.Y.Offset+delta.Y)
                    end
                end)
                bolinha.InputEnded:Connect(function(input)
                    if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                        drag=false
                    end
                end)
            end
            bolinha.Visible=true
            print("🔒 Shiftlock ativado! Clique na bolinha para ligar/desligar.")
        else
            if bolinha then
                bolinha.Visible=false
                shiftlockAtivo=false
            end
            print("🔓 Shiftlock desativado!")
        end
    end)
    
    -- REMOVER NEBLINA
    bNEBLINA.MouseButton1Click:Connect(function()
        NEBLINA_ON=not NEBLINA_ON
        bNEBLINA.Text=NEBLINA_ON and"🌤️ REMOVER NEBLINA: ON"or"🌤️ REMOVER NEBLINA: OFF"
        bNEBLINA.BackgroundColor3=NEBLINA_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)
        
        if NEBLINA_ON then
            removerNeblina()
            print("🌤️ Neblina removida permanentemente!")
        end
    end)
    bNEBLINA.TouchTap:Connect(function()
        NEBLINA_ON=not NEBLINA_ON
        bNEBLINA.Text=NEBLINA_ON and"🌤️ REMOVER NEBLINA: ON"or"🌤️ REMOVER NEBLINA: OFF"
        bNEBLINA.BackgroundColor3=NEBLINA_ON and Color3.fromRGB(200,150,50)or Color3.fromRGB(40,40,50)
        
        if NEBLINA_ON then
            removerNeblina()
            print("🌤️ Neblina removida permanentemente!")
        end
    end)
    
    -- ===== BOTÃO MINIMIZAR =====
    btnX.MouseButton1Click:Connect(function()
        menuAberto=false
        f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton")
            btnReabrir.Size=UDim2.new(0,55,0,55)
            btnReabrir.Position=UDim2.new(0.02,0,0.85,0)
            btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50)
            btnReabrir.Text="🔧"
            btnReabrir.TextColor3=Color3.fromRGB(255,255,255)
            btnReabrir.TextScaled=true
            btnReabrir.Font=Enum.Font.GothamBold
            btnReabrir.Parent=sg
            
            local cr=Instance.new("UICorner")
            cr.CornerRadius=UDim.new(0,28)
            cr.Parent=btnReabrir
            
            btnReabrir.MouseButton1Click:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
            btnReabrir.TouchTap:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
        else
            btnReabrir.Visible=true
        end
    end)
    btnX.TouchTap:Connect(function()
        menuAberto=false
        f.Visible=false
        if not btnReabrir then
            btnReabrir=Instance.new("TextButton")
            btnReabrir.Size=UDim2.new(0,55,0,55)
            btnReabrir.Position=UDim2.new(0.02,0,0.85,0)
            btnReabrir.BackgroundColor3=Color3.fromRGB(200,50,50)
            btnReabrir.Text="🔧"
            btnReabrir.TextColor3=Color3.fromRGB(255,255,255)
            btnReabrir.TextScaled=true
            btnReabrir.Font=Enum.Font.GothamBold
            btnReabrir.Parent=sg
            
            local cr=Instance.new("UICorner")
            cr.CornerRadius=UDim.new(0,28)
            cr.Parent=btnReabrir
            
            btnReabrir.MouseButton1Click:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
            btnReabrir.TouchTap:Connect(function()
                menuAberto=true
                f.Visible=true
                btnReabrir:Destroy()
                btnReabrir=nil
            end)
        else
            btnReabrir.Visible=true
        end
    end)
    
    -- ===== SISTEMA DE ARRASTAR O MENU =====
    local drag=false
    local startPos=nil
    local framePos=nil
    
    f.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            drag=true
            startPos=input.Position
            framePos=f.Position
        end
    end)
    f.InputChanged:Connect(function(input)
        if drag and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
            local delta=input.Position-startPos
            f.Position=UDim2.new(framePos.X.Scale,framePos.X.Offset+delta.X,framePos.Y.Scale,framePos.Y.Offset+delta.Y)
        end
    end)
    f.InputEnded:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            drag=false
        end
    end)
end

-- ===== INICIAR =====
criarMenu()
atualizarESP()
print("🔧 Hack Lab v11.0 - ESP + SHIFTLOCK + REMOVER NEBLINA")
