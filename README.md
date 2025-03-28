local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Criando a GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = player.PlayerGui

local MiniPanel = Instance.new("Frame")
MiniPanel.Size = UDim2.new(0, 200, 0, 100)
MiniPanel.Position = UDim2.new(0.5, -100, 0.5, -50)
MiniPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MiniPanel.BorderSizePixel = 2
MiniPanel.Parent = ScreenGui

local Dragging
local DragInput
local DragStart
local StartPos

MiniPanel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        Dragging = true
        DragStart = input.Position
        StartPos = MiniPanel.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                Dragging = false
            end
        end)
    end
end)

MiniPanel.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        DragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == DragInput and Dragging then
        local delta = input.Position - DragStart
        MiniPanel.Position = UDim2.new(StartPos.X.Scale, StartPos.X.Offset + delta.X, StartPos.Y.Scale, StartPos.Y.Offset + delta.Y)
    end
end)

-- Adicionando os botões ao painel
local Window = {} -- Simulação de Window (você pode substituir pelo seu framework)

Window.CreateTab = function(settings)
    local Tab = {}
    function Tab:CreateButton(buttonSettings)
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(1, -10, 0, 30)
        Button.Position = UDim2.new(0, 5, 0, #MiniPanel:GetChildren() * 35)
        Button.Text = buttonSettings.Name
        Button.Parent = MiniPanel
        Button.MouseButton1Click:Connect(buttonSettings.Callback)
    end
    return Tab
end

local Tab = Window:CreateTab({
    Name = "Farm Mini City",
    Icon = "home_work",
    ImageSource = "Material",
    ShowTitle = true
})

Tab:CreateButton({
    Name = "Salvar Posicao (ATIVE ANTES DE ATIVAR O FARM PEÇA)",
    Description = nil,
    Callback = function()
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            _G.SavedPosition = character.HumanoidRootPart.Position
            print("Posição salva!")
        else
            print("Não foi possível salvar a posição.")
        end
    end
})

Tab:CreateButton({
    Name = "Iniciar Farm",
    Description = nil,
    Callback = function()
        local function fireAllProximityPrompts()
            local objetosMissao = workspace:FindFirstChild("MapaGeral")
            if objetosMissao then
                objetosMissao = objetosMissao:FindFirstChild("FavelaV2")
                if objetosMissao then
                    objetosMissao = objetosMissao:FindFirstChild("objetosMissao")
                    if objetosMissao then
                        for _, prompt in ipairs(objetosMissao:GetDescendants()) do
                            if prompt:IsA("ProximityPrompt") then
                                prompt.HoldDuration = 0
                                
                                -- Tentando 7 vezes interagir com o ProximityPrompt
                                local success = false
                                for i = 1, 7 do
                                    if not success then
                                        prompt:InputHoldBegin()
                                        task.wait(0.1)  -- Intervalo curto entre tentativas
                                        prompt:InputHoldEnd()
                                        task.wait(0.1)  -- Intervalo após a interação
                                        
                                        -- Verifica se o prompt foi ativado corretamente
                                        if prompt.Parent and prompt.Parent:IsA("Part") and prompt.Parent.Position then
                                            success = true  -- Se o prompt foi ativado, marca como bem-sucedido
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end

        local function findPlayerProximityPrompt()
            local playerName = player.Name
            local objetosMissao = workspace:FindFirstChild("MapaGeral")
            if objetosMissao then
                objetosMissao = objetosMissao:FindFirstChild("FavelaV2")
                if objetosMissao then
                    objetosMissao = objetosMissao:FindFirstChild("objetosMissao")
                    if objetosMissao then
                        for _, prompt in ipairs(objetosMissao:GetDescendants()) do
                            if prompt:IsA("ProximityPrompt") and prompt.Name == playerName then
                                return prompt
                            end
                        end
                    end
                end
            end
            return nil
        end

        while true do
            if not _G.SavedPosition then break end
            local character = player.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then break end

            fireAllProximityPrompts()
            character.HumanoidRootPart.CFrame = CFrame.new(_G.SavedPosition)

            task.wait(0.35)  -- Velocidade do teleporte mantida

            game:GetService("ReplicatedStorage"):WaitForChild("RemoteNovos"):WaitForChild("trabalhos"):FireServer("missaoPECAS")

            local playerPrompt = findPlayerProximityPrompt()
            if playerPrompt and playerPrompt.Parent then
                character.HumanoidRootPart.CFrame = CFrame.new(playerPrompt.Parent.Position)
            end

            task.wait(0.2)
        end
    end
})

return ScreenGui
