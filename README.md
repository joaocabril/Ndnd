local ffi = require("ffi")

-- Definindo funções de manipulação do mouse
ffi.cdef[[
    typedef struct {
        long x;
        long y;
    } POINT;
    bool GetCursorPos(POINT *lpPoint);
    bool SetCursorPos(int x, int y);
]]

local user32 = ffi.load("user32")

-- Função para calcular a distância entre dois pontos
local function distance(x1, y1, x2, y2)
    return math.sqrt((x2 - x1) ^ 2 + (y2 - y1) ^ 2)
end

-- Função para mover a câmera para o inimigo mais próximo, mirando na cabeça
local function aim_at_closest_enemy_head(enemies, fov)
    local player = game.Players.LocalPlayer
    local camera = workspace.CurrentCamera
    local closest_enemy = nil
    local min_distance = math.huge
    
    for _, enemy in pairs(enemies) do
        local head_position = enemy.Head.Position
        local screen_point = camera:WorldToViewportPoint(head_position)
        local dist = distance(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2, screen_point.X, screen_point.Y)
        
        if math.abs(math.deg(math.atan2(screen_point.Y - camera.ViewportSize.Y / 2, screen_point.X - camera.ViewportSize.X / 2))) <= fov / 2 then
            if dist < min_distance then
                min_distance = dist
                closest_enemy = enemy
            end
        end
    end

    if closest_enemy then
        local target_position = closest_enemy.Head.Position
        camera.CFrame = CFrame.new(camera.CFrame.Position, target_position)
    end
end

-- Criar GUI móvel com Slider para ajustar o fov
local function create_movable_gui()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MovableGui"
    screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 300, 0, 150)
    frame.Position = UDim2.new(0.5, -150, 0.5, -75) -- Centralizado na tela
    frame.BackgroundColor3 = Color3.fromRGB(100, 100, 255)
    frame.Active = true
    frame.Draggable = true -- Torna o frame arrastável
    frame.Parent = screenGui

    local slider = Instance.new("TextButton")
    slider.Size = UDim2.new(0, 280, 0, 50)
    slider.Position = UDim2.new(0, 10, 0, 50)
    slider.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    slider.Text = "Aim FOV: 90"
    slider.Parent = frame

    -- Variável para armazenar o valor do fov
    local fov = 90

    -- Função para atualizar o texto do slider e o valor do fov
    local function updateFov(newFov)
        fov = newFov
        slider.Text = "Aim FOV: " .. tostring(fov)
    end

    -- Adiciona evento de clique para aumentar o fov
    slider.MouseButton1Click:Connect(function()
        local newFov = (fov + 50) % 550
        updateFov(newFov)
    end)

    -- Retorna o valor do fov atual
    return function()
        return fov
    end
end

-- Exemplo de uso
local enemies = {workspace.Enemy1, workspace.Enemy2, workspace.Enemy3}

local getFov = create_movable_gui()

game:GetService("RunService").RenderStepped:Connect(function()
    aim_at_closest_enemy_head(enemies, getFov())
end)
