
--[[
    RaioZ Hub ⚡
    Interface para Steal An Egg
    Lua/Luau — Roblox

    A interface não simula uma coleta bem-sucedida.
    A coleta e o retorno precisam de integração autorizada.
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local existing = playerGui:FindFirstChild("RaioZHub")
if existing then
    existing:Destroy()
end

local gui = Instance.new("ScreenGui")
gui.Name = "RaioZHub"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local WHITE = Color3.fromRGB(255, 255, 255)
local YELLOW = Color3.fromRGB(255, 205, 35)
local GRAY = Color3.fromRGB(242, 242, 242)
local DARK = Color3.fromRGB(45, 45, 45)
local GREEN = Color3.fromRGB(55, 155, 90)

local function make(className, properties, parent)
    local object = Instance.new(className)
    for property, value in pairs(properties) do
        object[property] = value
    end
    object.Parent = parent
    return object
end

local main = make("Frame", {
    Name = "Main",
    Size = UDim2.fromOffset(540, 370),
    Position = UDim2.new(0.5, -270, 0.5, -185),
    BackgroundColor3 = WHITE,
    BorderSizePixel = 0,
    Active = true,
    ClipsDescendants = true
}, gui)

make("UICorner", {
    CornerRadius = UDim.new(0, 14)
}, main)

make("UIStroke", {
    Color = YELLOW,
    Thickness = 2
}, main)

local header = make("Frame", {
    Name = "Header",
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundColor3 = YELLOW,
    BorderSizePixel = 0
}, main)

make("TextLabel", {
    Position = UDim2.fromOffset(14, 0),
    Size = UDim2.new(1, -105, 1, 0),
    BackgroundTransparency = 1,
    Text = "⚡ RaioZ Hub",
    TextColor3 = DARK,
    Font = Enum.Font.GothamBold,
    TextSize = 21,
    TextXAlignment = Enum.TextXAlignment.Left
}, header)

local minimize = make("TextButton", {
    Position = UDim2.new(1, -76, 0, 8),
    Size = UDim2.fromOffset(29, 29),
    Text = "—",
    TextColor3 = DARK,
    BackgroundColor3 = WHITE,
    Font = Enum.Font.GothamBold,
    TextSize = 18,
    BorderSizePixel = 0
}, header)
make("UICorner", {CornerRadius = UDim.new(0, 7)}, minimize)

local close = make("TextButton", {
    Position = UDim2.new(1, -39, 0, 8),
    Size = UDim2.fromOffset(29, 29),
    Text = "×",
    TextColor3 = DARK,
    BackgroundColor3 = WHITE,
    Font = Enum.Font.GothamBold,
    TextSize = 22,
    BorderSizePixel = 0
}, header)
make("UICorner", {CornerRadius = UDim.new(0, 7)}, close)

local sidebar = make("Frame", {
    Position = UDim2.fromOffset(10, 58),
    Size = UDim2.new(0, 126, 1, -68),
    BackgroundColor3 = GRAY,
    BorderSizePixel = 0
}, main)
make("UICorner", {CornerRadius = UDim.new(0, 10)}, sidebar)

local content = make("Frame", {
    Position = UDim2.fromOffset(145, 58),
    Size = UDim2.new(1, -155, 1, -68),
    BackgroundColor3 = WHITE,
    BorderSizePixel = 0
}, main)

local pages = {}
local selectedEgg = nil
local running = false
local minimized = false
local expandedSize = main.Size

local function createPage(name)
    local page = make("ScrollingFrame", {
        Name = name,
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Visible = false
    }, content)

    make("UIPadding", {
        PaddingTop = UDim.new(0, 5),
        PaddingLeft = UDim.new(0, 5),
        PaddingRight = UDim.new(0, 5),
        PaddingBottom = UDim.new(0, 8)
    }, page)

    pages[name] = page
    return page
end

local function label(parent, text, height)
    return make("TextLabel", {
        Size = UDim2.new(1, 0, 0, height or 34),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = DARK,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        TextWrapped = true,
        TextXAlignment = Enum.TextXAlignment.Left
    }, parent)
end

local function button(parent, text, callback)
    local b = make("TextButton", {
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = GRAY,
        Text = text,
        TextColor3 = DARK,
        Font = Enum.Font.GothamSemibold,
        TextSize = 12,
        BorderSizePixel = 0,
        AutoButtonColor = true
    }, parent)

    make("UICorner", {
        CornerRadius = UDim.new(0, 8)
    }, b)

    b.Activated:Connect(callback)
    return b
end

local function setStatus(text)
    local page = pages.Ovos
    if page and page:FindFirstChild("Status") then
        page.Status.Text = "Status: " .. text
    end
end

-- Aba Ovos
local eggs = createPage("Ovos")

label(eggs, "🥚  ROUBAR UM OVO", 30)

label(eggs,
    "Selecione um ovo detectado para visualizar os detalhes.", 38)

local eggList = make("Frame", {
    Size = UDim2.new(1, 0, 0, 40),
    BackgroundColor3 = GRAY,
    BorderSizePixel = 0
}, eggs)
make("UICorner", {CornerRadius = UDim.new(0, 8)}, eggList)

local eggText = make("TextLabel", {
    Size = UDim2.new(1, -12, 1, 0),
    Position = UDim2.fromOffset(6, 0),
    BackgroundTransparency = 1,
    Text = "Nenhum ovo selecionado",
    TextColor3 = DARK,
    Font = Enum.Font.Gotham,
    TextSize = 12,
    TextWrapped = true
}, eggList)

local status = label(eggs, "Status: aguardando seleção", 32)
status.Name = "Status"

button(eggs, "Atualizar ovos visíveis", function()
    -- Não inventa objetos ou ovos.
    -- A detecção real depende da estrutura acessível do jogo.
    setStatus("detecção real ainda não configurada")
end)

button(eggs, "Selecionar ovo detectado", function()
    setStatus("nenhum catálogo de ovos configurado")
end)

local collectButton = button(eggs, "⚡ Pegar ovo selecionado", function()
    if not selectedEgg then
        setStatus("selecione um ovo primeiro")
        return
    end

    setStatus("coleta não integrada ao servidor")
end)
collectButton.BackgroundColor3 = YELLOW

button(eggs, "Cancelar operação", function()
    running = false
    setStatus("operação cancelada")
end)

-- Aba Segurança
local security = createPage("Segurança")
label(security, "🛡️  SEGURANÇA", 30)

local alerts = false
local alertButton

alertButton = button(security, "Alertas de proximidade: DESATIVADO", function()
    alerts = not alerts
    alertButton.Text = "Alertas de proximidade: "
        .. (alerts and "ATIVADO" or "DESATIVADO")
    alertButton.BackgroundColor3 = alerts and YELLOW or GRAY
end)

label(security,
    "Os alertas precisam de detecção real de jogadores próximos.", 50)

-- Aba Farm
local farm = createPage("Farm")
label(farm, "🏃  FARM DE VELOCIDADE", 30)

local monitor = false
local monitorButton

monitorButton = button(farm, "Monitorar velocidade: DESATIVADO", function()
    monitor = not monitor
    monitorButton.Text = "Monitorar velocidade: "
        .. (monitor and "ATIVADO" or "DESATIVADO")
    monitorButton.BackgroundColor3 = monitor and YELLOW or GRAY
end)

local speedLabel = label(farm, "Velocidade: aguardando personagem", 32)

button(farm, "Atualizar velocidade", function()
    local character = player.Character
    local humanoid = character
        and character:FindFirstChildOfClass("Humanoid")

    if humanoid then
        speedLabel.Text = string.format(
            "Velocidade atual: %.1f", humanoid.WalkSpeed
        )
    else
        speedLabel.Text = "Personagem indisponível"
    end
end)

-- Aba Teleporte
local teleport = createPage("Teleporte")
label(teleport, "📍  TELEPORTE DE ÁREAS", 30)

label(teleport,
    "Os destinos precisam ser identificados e validados.", 42)

button(teleport, "Listar áreas disponíveis", function()
    setStatus("áreas não identificadas automaticamente")
end)

label(teleport,
    "Não foram inventadas coordenadas ou destinos.", 40)

-- Menu lateral
local tabNames = {"Ovos", "Segurança", "Farm", "Teleporte"}

local function showPage(name)
    for pageName, page in pairs(pages) do
        page.Visible = pageName == name
    end
end

for index, name in ipairs(tabNames) do
    local tab = button(sidebar, name, function()
        showPage(name)
    end)

    tab.Position = UDim2.fromOffset(6, 8 + (index - 1) * 43)
    tab.Size = UDim2.new(1, -12, 0, 36)
end

showPage("Ovos")

-- Minimizar e restaurar
minimize.Activated:Connect(function()
    minimized = not minimized

    main.Size = minimized
        and UDim2.new(0, 300, 0, 48)
        or expandedSize

    sidebar.Visible = not minimized
    content.Visible = not minimized
    minimize.Text = minimized and "□" or "—"
end)

close.Activated:Connect(function()
    gui:Destroy()
end)

-- Arrastar a janela usando mouse ou toque
local dragging = false
local dragStart
local startPosition

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local delta = input.Position - dragStart

        main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

print("RaioZ Hub carregado. ⚡")
