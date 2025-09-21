-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local root = character:WaitForChild("HumanoidRootPart")

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "ReverseMacroGui"

local recordButton = Instance.new("TextButton", ScreenGui)
recordButton.Size = UDim2.new(0, 150, 0, 50)
recordButton.Position = UDim2.new(0, 50, 0, 50)
recordButton.Text = "Gravar (OFF)"

local reverseButton = Instance.new("TextButton", ScreenGui)
reverseButton.Size = UDim2.new(0, 150, 0, 50)
reverseButton.Position = UDim2.new(0, 50, 0, 120)
reverseButton.Text = "Reverter"

-- Variáveis
local recording = false
local actions = {}
local lastJumpState = humanoid:GetState() == Enum.HumanoidStateType.Jumping

-- Toggle gravação
recordButton.MouseButton1Click:Connect(function()
    recording = not recording
    if recording then
        actions = {} -- Limpa ações antigas ao iniciar gravação
        lastJumpState = humanoid:GetState() == Enum.HumanoidStateType.Jumping
    end
    recordButton.Text = recording and "Gravar (ON)" or "Gravar (OFF)"
end)

-- Gravar ações
RunService.RenderStepped:Connect(function()
    if recording then
        local jumpState = humanoid:GetState() == Enum.HumanoidStateType.Jumping
        local action = {
            position = root.Position,
            jump = jumpState and not lastJumpState -- True apenas quando começa a pular
        }
        table.insert(actions, action)
        lastJumpState = jumpState
    end
end)

-- Reproduzir reverso
reverseButton.MouseButton1Click:Connect(function()
    if #actions == 0 then return end
    for i = #actions, 1, -1 do
        local action = actions[i]
        root.CFrame = CFrame.new(action.position)
        if action.jump then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
        RunService.RenderStepped:Wait()
    end
end)
