local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local COLORS = {
    Background = Color3.fromRGB(228, 232, 240),
    Surface = Color3.fromRGB(245, 247, 252),
    Surface2 = Color3.fromRGB(232, 236, 244),
    Surface3 = Color3.fromRGB(212, 218, 230),
    Accent = Color3.fromRGB(60, 175, 100),
    AccentDark = Color3.fromRGB(45, 145, 80),
    AccentGlow = Color3.fromRGB(110, 220, 150),
    Text = Color3.fromRGB(35, 38, 48),
    SubText = Color3.fromRGB(115, 120, 135),
    Off = Color3.fromRGB(195, 200, 212),
    Border = Color3.fromRGB(205, 210, 222),
    Orange = Color3.fromRGB(235, 160, 55),
    Red = Color3.fromRGB(225, 80, 95),
    Purple = Color3.fromRGB(150, 110, 220),
    Discord = Color3.fromRGB(88, 101, 242),
    DiscordLight = Color3.fromRGB(110, 120, 255)
}

local speedOn = false
local jumpOn = false
local flyOn = false
local noclipOn = false
local unstunOn = false
local followOn = false
local fpsOn = false
local lagFixOn = false
local espOn = false
local camlockOn = false
local aimbotOn = false
local onlyEquippedOn = false

local speedValue = 32
local jumpValue = 90
local flyValue = 100
local followDistance = 3
local camlockFOV = 800
local camlockSmooth = 0.35
local aimbotFOV = 1500
local aimbotPart = "Head"
local aimbotKey = Enum.KeyCode.E
local aimbotHold = false

local DISCORD_LINK = "https://discord.gg/HMJdNXE3rR"

local followTarget = nil
local lockedTarget = nil
local currentPage = "movement"
local menuOpen = false
local minimized = false
local dragging = false
local dragStart
local panelStart

local flyConnection
local followConnection
local unstunConnection
local noclipConnection
local fpsConnection
local characterConnection
local lagConnection
local lagConnection2
local lagConnection3
local lagConnection4
local espConnection
local camlockConnection
local aimbotConnection
local aimbotInputConn

local flyAttachment
local flyVelocity
local flyOrientation

local oldAutoRotate
local oldPlatformStand

local collisionCache = {}
local lagCache = {}
local espCache = {}
local debounceMap = {}

local camlockCircle
local aimbotCircle

local function debounce(key, delay)
    if debounceMap[key] then return false end
    debounceMap[key] = true
    task.delay(delay or 0.2, function()
        debounceMap[key] = nil
    end)
    return true
end

local function tween(object, time, properties, style)
    local animation = TweenService:Create(
        object,
        TweenInfo.new(time, style or Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
        properties
    )
    animation:Play()
    return animation
end

local function corner(object, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 14)
    c.Parent = object
    return c
end

local function stroke(object, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color or Color3.fromRGB(200, 208, 222)
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0.3
    s.Parent = object
    return s
end

local function textLabel(parent, text, size, position, textSize, color, font)
    local x = Instance.new("TextLabel")
    x.BackgroundTransparency = 1
    x.Text = text
    x.Size = size
    x.Position = position
    x.TextSize = textSize or 12
    x.TextColor3 = color or COLORS.Text
    x.Font = font or Enum.Font.GothamMedium
    x.TextXAlignment = Enum.TextXAlignment.Left
    x.TextYAlignment = Enum.TextYAlignment.Center
    x.Parent = parent
    return x
end

local function getCharacter()
    return player.Character
end

local function getHumanoid()
    local character = getCharacter()
    if not character then return nil end
    return character:FindFirstChildOfClass("Humanoid")
end

local function getRoot()
    local character = getCharacter()
    if not character then return nil end
    return character:FindFirstChild("HumanoidRootPart")
end

local gui = Instance.new("ScreenGui")
gui.Name = "VanhHub"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local hubButton = Instance.new("TextButton")
hubButton.Size = UDim2.fromOffset(130, 40)
hubButton.Position = UDim2.fromOffset(20, 70)
hubButton.BackgroundColor3 = COLORS.Surface
hubButton.BackgroundTransparency = 0.05
hubButton.Text = ""
hubButton.AutoButtonColor = false
hubButton.Parent = gui
corner(hubButton, 12)
stroke(hubButton, COLORS.Border, 1, 0.3)

local hubGradient = Instance.new("UIGradient")
hubGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(252, 253, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(238, 242, 249))
})
hubGradient.Rotation = 30
hubGradient.Parent = hubButton

local hubIcon = Instance.new("Frame")
hubIcon.Size = UDim2.fromOffset(10, 10)
hubIcon.Position = UDim2.fromOffset(12, 15)
hubIcon.BackgroundColor3 = COLORS.Accent
hubIcon.Parent = hubButton
corner(hubIcon, 10)

local hubPulse = Instance.new("Frame")
hubPulse.Size = UDim2.fromOffset(10, 10)
hubPulse.Position = UDim2.fromOffset(12, 15)
hubPulse.BackgroundColor3 = COLORS.AccentGlow
hubPulse.BackgroundTransparency = 0.5
hubPulse.Parent = hubButton
corner(hubPulse, 10)

task.spawn(function()
    while hubButton.Parent do
        tween(hubPulse, 1.2, {
            Size = UDim2.fromOffset(20, 20),
            Position = UDim2.fromOffset(7, 10),
            BackgroundTransparency = 1
        })
        task.wait(1.2)
        hubPulse.Size = UDim2.fromOffset(10, 10)
        hubPulse.Position = UDim2.fromOffset(12, 15)
        hubPulse.BackgroundTransparency = 0.5
    end
end)

textLabel(hubButton, "Vanh Hub", UDim2.new(1, -36, 0, 18), UDim2.fromOffset(30, 4), 12, COLORS.Text, Enum.Font.GothamBold)
textLabel(hubButton, "Meme Sea", UDim2.new(1, -36, 0, 12), UDim2.fromOffset(30, 22), 7, COLORS.SubText, Enum.Font.GothamBold)

local fpsBadge = Instance.new("Frame")
fpsBadge.Size = UDim2.fromOffset(90, 30)
fpsBadge.Position = UDim2.new(1, -110, 1, -60)
fpsBadge.BackgroundColor3 = COLORS.Surface
fpsBadge.BackgroundTransparency = 0.05
fpsBadge.Visible = false
fpsBadge.Parent = gui
corner(fpsBadge, 10)
stroke(fpsBadge, COLORS.Border, 1, 0.3)

local fpsDot = Instance.new("Frame")
fpsDot.Size = UDim2.fromOffset(7, 7)
fpsDot.Position = UDim2.fromOffset(10, 12)
fpsDot.BackgroundColor3 = COLORS.Accent
fpsDot.Parent = fpsBadge
corner(fpsDot, 10)

local fpsText = textLabel(fpsBadge, "FPS  --", UDim2.new(1, -24, 1, 0), UDim2.fromOffset(22, 0), 9, COLORS.Text, Enum.Font.GothamBold)

local toastFrame = Instance.new("Frame")
toastFrame.Size = UDim2.fromOffset(230, 42)
toastFrame.AnchorPoint = Vector2.new(0.5, 0)
toastFrame.Position = UDim2.new(0.5, 0, 0, -60)
toastFrame.BackgroundColor3 = COLORS.Surface
toastFrame.BackgroundTransparency = 0
toastFrame.Visible = false
toastFrame.ZIndex = 200
toastFrame.Parent = gui
corner(toastFrame, 12)
stroke(toastFrame, COLORS.Accent, 1.5, 0.2)

local toastDot = Instance.new("Frame")
toastDot.Size = UDim2.fromOffset(8, 8)
toastDot.Position = UDim2.fromOffset(14, 17)
toastDot.BackgroundColor3 = COLORS.Accent
toastDot.Parent = toastFrame
corner(toastDot, 10)

local toastLabel = textLabel(toastFrame, "Đã copy link Discord!", UDim2.new(1, -34, 1, 0), UDim2.fromOffset(30, 0), 10, COLORS.Text, Enum.Font.GothamBold)

local function showToast(text)
    toastLabel.Text = text
    toastFrame.Visible = true
    toastFrame.Position = UDim2.new(0.5, 0, 0, -60)
    toastFrame.BackgroundTransparency = 1
    tween(toastFrame, 0.35, {
        Position = UDim2.new(0.5, 0, 0, 30),
        BackgroundTransparency = 0
    }, Enum.EasingStyle.Back)
    task.delay(2, function()
        tween(toastFrame, 0.3, {
            Position = UDim2.new(0.5, 0, 0, -60),
            BackgroundTransparency = 1
        })
        task.wait(0.3)
        toastFrame.Visible = false
    end)
end

local panel = Instance.new("Frame")
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.Size = UDim2.fromOffset(680, 460)
panel.Position = UDim2.new(0.5, 0, 0.5, 25)
panel.BackgroundColor3 = COLORS.Background
panel.BackgroundTransparency = 1
panel.Visible = false
panel.Parent = gui
corner(panel, 20)
stroke(panel, COLORS.Border, 1, 0.3)

local panelGradient = Instance.new("UIGradient")
panelGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(248, 250, 254)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(228, 233, 242))
})
panelGradient.Rotation = 115
panelGradient.Parent = panel

local panelScale = Instance.new("UIScale")
panelScale.Scale = 0.9
panelScale.Parent = panel

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 56)
header.BackgroundColor3 = COLORS.Surface
header.BackgroundTransparency = 0
header.Parent = panel
corner(header, 20)

local headerGradient = Instance.new("UIGradient")
headerGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(252, 253, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(240, 244, 250))
})
headerGradient.Rotation = 20
headerGradient.Parent = header

local headerIcon = Instance.new("Frame")
headerIcon.Size = UDim2.fromOffset(32, 32)
headerIcon.Position = UDim2.fromOffset(12, 12)
headerIcon.BackgroundColor3 = COLORS.Surface2
headerIcon.Parent = header
corner(headerIcon, 10)
stroke(headerIcon, COLORS.Accent, 1, 0.5)

local iconText = textLabel(headerIcon, "V", UDim2.fromScale(1, 1), UDim2.fromOffset(0, 0), 15, COLORS.Accent, Enum.Font.GothamBlack)
iconText.TextXAlignment = Enum.TextXAlignment.Center

textLabel(header, "Vanh Hub", UDim2.new(0, 300, 0, 20), UDim2.fromOffset(54, 6), 14, COLORS.Text, Enum.Font.GothamBold)
textLabel(header, "Meme Sea", UDim2.new(0, 300, 0, 14), UDim2.fromOffset(54, 26), 7, COLORS.SubText, Enum.Font.GothamMedium)

local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.fromOffset(30, 30)
minimizeButton.Position = UDim2.new(1, -80, 0, 13)
minimizeButton.BackgroundColor3 = COLORS.Surface2
minimizeButton.Text = "—"
minimizeButton.TextColor3 = COLORS.Text
minimizeButton.TextSize = 15
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.AutoButtonColor = false
minimizeButton.Parent = header
corner(minimizeButton, 8)

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.fromOffset(30, 30)
closeButton.Position = UDim2.new(1, -46, 0, 13)
closeButton.BackgroundColor3 = COLORS.Surface2
closeButton.Text = "×"
closeButton.TextColor3 = COLORS.Text
closeButton.TextSize = 17
closeButton.Font = Enum.Font.GothamBold
closeButton.AutoButtonColor = false
closeButton.Parent = header
corner(closeButton, 8)

local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0, 160, 1, -80)
sidebar.Position = UDim2.fromOffset(10, 64)
sidebar.BackgroundColor3 = COLORS.Surface
sidebar.BackgroundTransparency = 0
sidebar.Parent = panel
corner(sidebar, 16)
stroke(sidebar, COLORS.Border, 1, 0.3)

local sideGradient = Instance.new("UIGradient")
sideGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 252, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(238, 242, 249))
})
sideGradient.Rotation = 90
sideGradient.Parent = sidebar

textLabel(sidebar, "MENU", UDim2.new(1, -30, 0, 20), UDim2.fromOffset(14, 10), 9, COLORS.SubText, Enum.Font.GothamBold)

local navButtons = {}

local function createNavButton(icon, titleText, y)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -16, 0, 36)
    button.Position = UDim2.fromOffset(8, y)
    button.BackgroundColor3 = COLORS.Surface
    button.Text = ""
    button.AutoButtonColor = false
    button.Parent = sidebar
    corner(button, 10)
    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.fromOffset(3, 0)
    indicator.Position = UDim2.fromOffset(0, 18)
    indicator.AnchorPoint = Vector2.new(0, 0.5)
    indicator.BackgroundColor3 = COLORS.Accent
    indicator.Parent = button
    corner(indicator, 10)
    local iconLabel = textLabel(button, icon, UDim2.fromOffset(26, 36), UDim2.fromOffset(8, 0), 14, COLORS.SubText, Enum.Font.GothamBold)
    iconLabel.TextXAlignment = Enum.TextXAlignment.Center
    local titleLabel = textLabel(button, titleText, UDim2.new(1, -42, 1, 0), UDim2.fromOffset(40, 0), 9, COLORS.SubText, Enum.Font.GothamBold)
    navButtons[titleText] = {
        Button = button,
        Icon = iconLabel,
        Text = titleLabel,
        Indicator = indicator
    }
    return button
end

local movementNav = createNavButton("↗", "MOVEMENT", 38)
local playerNav = createNavButton("◎", "PLAYER", 80)
local aimNav = createNavButton("✛", "AIM", 122)

textLabel(sidebar, "STATUS", UDim2.new(1, -30, 0, 20), UDim2.fromOffset(14, 168), 9, COLORS.SubText, Enum.Font.GothamBold)

local statusBox = Instance.new("Frame")
statusBox.Size = UDim2.new(1, -26, 0, 30)
statusBox.Position = UDim2.fromOffset(13, 192)
statusBox.BackgroundColor3 = COLORS.Surface2
statusBox.Parent = sidebar
corner(statusBox, 10)

local statusDot = Instance.new("Frame")
statusDot.Size = UDim2.fromOffset(7, 7)
statusDot.Position = UDim2.fromOffset(10, 11)
statusDot.BackgroundColor3 = COLORS.Accent
statusDot.Parent = statusBox
corner(statusDot, 10)

local statusLabel = textLabel(statusBox, "READY", UDim2.new(1, -26, 1, 0), UDim2.fromOffset(24, 0), 8, COLORS.Accent, Enum.Font.GothamBold)

textLabel(sidebar, "SOCIAL", UDim2.new(1, -30, 0, 20), UDim2.fromOffset(14, 232), 9, COLORS.SubText, Enum.Font.GothamBold)

local discordBtn = Instance.new("TextButton")
discordBtn.Size = UDim2.new(1, -26, 0, 34)
discordBtn.Position = UDim2.fromOffset(13, 256)
discordBtn.BackgroundColor3 = COLORS.Discord
discordBtn.BackgroundTransparency = 0
discordBtn.Text = ""
discordBtn.AutoButtonColor = false
discordBtn.Parent = sidebar
corner(discordBtn, 10)

local discordGrad = Instance.new("UIGradient")
discordGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, COLORS.DiscordLight),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 80, 220))
})
discordGrad.Rotation = 45
discordGrad.Parent = discordBtn

local discordIcon = textLabel(
    discordBtn,
    "◈",
    UDim2.fromOffset(24, 34),
    UDim2.fromOffset(8, 0),
    14,
    Color3.fromRGB(255, 255, 255),
    Enum.Font.GothamBold
)
discordIcon.TextXAlignment = Enum.TextXAlignment.Center

textLabel(
    discordBtn,
    "Copy Discord",
    UDim2.new(1, -34, 1, 0),
    UDim2.fromOffset(32, 0),
    9,
    Color3.fromRGB(255, 255, 255),
    Enum.Font.GothamBold
)

discordBtn.MouseEnter:Connect(function()
    tween(discordBtn, 0.15, {
        BackgroundColor3 = COLORS.DiscordLight
    })
end)

discordBtn.MouseLeave:Connect(function()
    tween(discordBtn, 0.15, {
        BackgroundColor3 = COLORS.Discord
    })
end)

discordBtn.Activated:Connect(function()
    if not debounce("discord") then return end
    if setclipboard then
        pcall(function()
            setclipboard(DISCORD_LINK)
        end)
        showToast("Đã copy link Discord!")
    else
        showToast("Không hỗ trợ copy!")
    end
end)

local function selectNav(name)
    for key, data in pairs(navButtons) do
        local selected = key == name
        tween(data.Button, 0.15, {
            BackgroundColor3 = selected and COLORS.Surface3 or COLORS.Surface
        })
        tween(data.Indicator, 0.15, {
            Size = selected and UDim2.fromOffset(3, 20) or UDim2.fromOffset(3, 0)
        })
        data.Icon.TextColor3 = selected and COLORS.Accent or COLORS.SubText
        data.Text.TextColor3 = selected and COLORS.Text or COLORS.SubText
    end
end

local content = Instance.new("Frame")
content.Size = UDim2.new(1, -184, 1, -80)
content.Position = UDim2.fromOffset(184, 64)
content.BackgroundTransparency = 1
content.ClipsDescendants = true
content.Parent = panel

local movementPage = Instance.new("Frame")
movementPage.Size = UDim2.fromScale(1, 1)
movementPage.BackgroundTransparency = 1
movementPage.Parent = content

local playerPage = Instance.new("Frame")
playerPage.Size = UDim2.fromScale(1, 1)
playerPage.BackgroundTransparency = 1
playerPage.Visible = false
playerPage.Parent = content

local aimPage = Instance.new("Frame")
aimPage.Size = UDim2.fromScale(1, 1)
aimPage.BackgroundTransparency = 1
aimPage.Visible = false
aimPage.Parent = content

local function makeCard(parent, titleText, subtitleText)
    local card = Instance.new("Frame")
    card.Size = UDim2.fromScale(1, 1)
    card.BackgroundColor3 = COLORS.Surface
    card.Parent = parent
    corner(card, 16)
    stroke(card, COLORS.Border, 1, 0.3)
    local cardGrad = Instance.new("UIGradient")
    cardGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 252, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(238, 242, 248))
    })
    cardGrad.Rotation = 120
    cardGrad.Parent = card
    local cardHeader = Instance.new("Frame")
    cardHeader.Size = UDim2.new(1, 0, 0, 66)
    cardHeader.BackgroundTransparency = 1
    cardHeader.Parent = card
    textLabel(cardHeader, titleText, UDim2.new(1, -32, 0, 26), UDim2.fromOffset(16, 12), 16, COLORS.Text, Enum.Font.GothamBold)
    textLabel(cardHeader, subtitleText, UDim2.new(1, -32, 0, 16), UDim2.fromOffset(16, 38), 8, COLORS.SubText)
    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(1, -32, 0, 1)
    divider.Position = UDim2.fromOffset(16, 64)
    divider.BackgroundColor3 = COLORS.Border
    divider.BackgroundTransparency = 0.5
    divider.BorderSizePixel = 0
    divider.Parent = cardHeader
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -8, 1, -70)
    scroll.Position = UDim2.fromOffset(4, 70)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 6
    scroll.ScrollBarImageColor3 = COLORS.Accent
    scroll.ScrollBarImageTransparency = 0.2
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    scroll.ScrollingDirection = Enum.ScrollingDirection.Y
    scroll.ElasticBehavior = Enum.ElasticBehavior.Never
    scroll.Parent = card
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, 0, 0, 0)
    container.BackgroundTransparency = 1
    container.AutomaticSize = Enum.AutomaticSize.Y
    container.Parent = scroll
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = container
    local pad = Instance.new("UIPadding")
    pad.PaddingTop = UDim.new(0, 4)
    pad.PaddingBottom = UDim.new(0, 16)
    pad.PaddingLeft = UDim.new(0, 8)
    pad.PaddingRight = UDim.new(0, 8)
    pad.Parent = container
    return card, container
end

local function createSection(parent, titleText)
    local section = Instance.new("Frame")
    section.Size = UDim2.new(1, 0, 0, 22)
    section.BackgroundTransparency = 1
    section.Parent = parent
    local dot = Instance.new("Frame")
    dot.Size = UDim2.fromOffset(5, 5)
    dot.Position = UDim2.fromOffset(0, 8)
    dot.BackgroundColor3 = COLORS.Accent
    dot.Parent = section
    corner(dot, 10)
    textLabel(section, titleText, UDim2.new(1, -14, 1, 0), UDim2.fromOffset(11, 0), 8, COLORS.SubText, Enum.Font.GothamBold
    local line = Instance.new("Frame")
    line.Size = UDim2.new(1, -110, 0, 1)
    line.Position = UDim2.new(0, 110, 0.5, 0)
    line.BackgroundColor3 = COLORS.Border
    line.BackgroundTransparency = 0.5
    line.BorderSizePixel = 0
    line.Parent = section
    return section
end

local function createSlider(parent, titleText, minValue, maxValue, initialValue, callback)
    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1, 0, 0, 48)
    holder.BackgroundTransparency = 1
    holder.Parent = parent
    textLabel(holder, titleText, UDim2.new(1, -80, 0, 15), UDim2.fromOffset(0, 0), 8, COLORS.Text, Enum.Font.GothamSemibold)
    local valueLabel = textLabel(holder, tostring(initialValue), UDim2.fromOffset(70, 15), UDim2.new(1, -70, 0, 0), 8, COLORS.Accent, Enum.Font.GothamBold)
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right
    local track = Instance.new("TextButton")
    track.Size = UDim2.new(1, 0, 0, 7)
    track.Position = UDim2.fromOffset(0, 26)
    track.BackgroundColor3 = COLORS.Surface3
    track.Text = ""
    track.AutoButtonColor = false
    track.Parent = holder
    corner(track, 8)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((initialValue - minValue) / (maxValue - minValue), 0, 1, 0)
    fill.BackgroundColor3 = COLORS.Accent
    fill.BorderSizePixel = 0
    fill.Parent = track
    corner(fill, 8)
    local fillGrad = Instance.new("UIGradient")
    fillGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, COLORS.AccentDark),
        ColorSequenceKeypoint.new(1, COLORS.AccentGlow)
    })
    fillGrad.Parent = fill
    local knob = Instance.new("Frame")
    knob.Size = UDim2.fromOffset(14, 14)
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.Position = UDim2.new((initialValue - minValue) / (maxValue - minValue), 0, 0.5, 0)
    knob.BackgroundColor3 = COLORS.Text
    knob.Parent = track
    corner(knob, 20)
    stroke(knob, COLORS.Accent, 2, 0)
    local knobGlow = Instance.new("Frame")
    knobGlow.Size = UDim2.fromOffset(20, 20)
    knobGlow.AnchorPoint = Vector2.new(0.5, 0.5)
    knobGlow.Position = UDim2.fromScale(0.5, 0.5)
    knobGlow.BackgroundColor3 = COLORS.Accent
    knobGlow.BackgroundTransparency = 0.75
    knobGlow.ZIndex = 0
    knobGlow.Parent = knob
    corner(knobGlow, 20)
    local sliderDragging = false
    local function update(x)
        local width = track.AbsoluteSize.X
        if width <= 0 then return end
        local percent = math.clamp((x - track.AbsolutePosition.X) / width, 0, 1)
        local value = math.floor(minValue + (maxValue - minValue) * percent + 0.5)
        valueLabel.Text = tostring(value)
        fill.Size = UDim2.new(percent, 0, 1, 0)
        knob.Position = UDim2.new(percent, 0, 0.5, 0)
        callback(value)
    end
    track.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
           or input.UserInputType == Enum.UserInputType.Touch then
            sliderDragging = true
            update(input.Position.X)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if sliderDragging and (input.UserInputType == Enum.UserInputType.MouseMovement
           or input.UserInputType == Enum.UserInputType.Touch) then
            update(input.Position.X)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
           or input.UserInputType == Enum.UserInputType.Touch then
            sliderDragging = false
        end
    end)
end

local function createToggle(parent, titleText, callback)
    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1, 0, 0, 46)
    holder.BackgroundColor3 = COLORS.Surface2
    holder.BackgroundTransparency = 0
    holder.Parent = parent
    corner(holder, 12)
    stroke(holder, COLORS.Border, 1, 0.4)
    local holderGrad = Instance.new("UIGradient")
    holderGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(245, 248, 253)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(232, 238, 246))
    })
    holderGrad.Rotation = 45
    holderGrad.Parent = holder
    textLabel(holder, titleText, UDim2.new(1, -62, 1, 0), UDim2.fromOffset(12, 0), 9, COLORS.Text, Enum.Font.GothamSemibold)
    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.fromOffset(44, 24)
    toggle.Position = UDim2.new(1, -54, 0.5, -12)
    toggle.BackgroundColor3 = COLORS.Off
    toggle.Text = ""
    toggle.AutoButtonColor = false
    toggle.Parent = holder
    corner(toggle, 20)
    local knob = Instance.new("Frame")
    knob.Size = UDim2.fromOffset(18, 18)
    knob.Position = UDim2.fromOffset(3, 3)
    knob.BackgroundColor3 = COLORS.Surface
    knob.Parent = toggle
    corner(knob, 20)
    local function set(value)
        toggle:SetAttribute("Enabled", value)
        tween(toggle, 0.18, {
            BackgroundColor3 = value and COLORS.AccentDark or COLORS.Off
        })
        tween(knob, 0.18, {
            Position = value and UDim2.fromOffset(23, 3) or UDim2.fromOffset(3, 3)
        })
        if callback then callback(value) end
    end
    return holder, set
end

local movementCard, movementContent = makeCard(
    movementPage,
    "Movement",
    "Speed, jump, fly, noclip và tối ưu"
)

createSection(movementContent, "CHỈ SỐ DI CHUYỂN")

createSlider(
    movementContent,
    "WALK SPEED",
    16, 250, speedValue,
    function(value)
        speedValue = value
        if speedOn then
            local humanoid = getHumanoid()
            if humanoid then
                humanoid.WalkSpeed = value
            end
        end
    end
)

createSlider(
    movementContent,
    "JUMP POWER",
    50, 300, jumpValue,
    function(value)
        jumpValue = value
        if jumpOn then
            local humanoid = getHumanoid()
            if humanoid then
                humanoid.UseJumpPower = true
                humanoid.JumpPower = value
            end
        end
    end
)

createSlider(
    movementContent,
    "FLY SPEED",
    20, 500, flyValue,
    function(value)
        flyValue = value
    end
)

local function applyMovement()
    local humanoid = getHumanoid()
    if not humanoid then return end
    humanoid.WalkSpeed = speedOn and speedValue or 16
    humanoid.UseJumpPower = true
    humanoid.JumpPower = jumpOn and jumpValue or 50
end

createSection(movementContent, "BẬT / TẮT")

local speedHolder, speedSet = createToggle(movementContent, "Walk Speed", function(enabled) end)
local jumpHolder, jumpSet = createToggle(movementContent, "Jump Power", function(enabled) end)
local flyHolder, flySet = createToggle(movementContent, "Fly", function(enabled) end)
local noclipHolder, noclipSet = createToggle(movementContent, "Noclip", function(enabled) end)

createSection(movementContent, "HIỆU NĂNG")

local fpsHolder, fpsSet = createToggle(movementContent, "Show FPS", function(enabled) end)
local lagHolder, lagSet = createToggle(movementContent, "Fix Lag + Đồ Họa", function(enabled) end)

local playerCard, playerContent = makeCard(
    playerPage,
    "Player",
    "Follow, unstun và ESP"
)

createSection(playerContent, "MỤC TIÊU")

local selectPlayer = Instance.new("TextButton")
selectPlayer.Size = UDim2.new(1, 0, 0, 42)
selectPlayer.BackgroundColor3 = COLORS.Surface2
selectPlayer.Text = "  SELECT PLAYER                              ▾"
selectPlayer.TextColor3 = COLORS.Text
selectPlayer.TextSize = 9
selectPlayer.Font = Enum.Font.GothamBold
selectPlayer.TextXAlignment = Enum.TextXAlignment.Left
selectPlayer.AutoButtonColor = false
selectPlayer.Parent = playerContent
corner(selectPlayer, 11)
stroke(selectPlayer, COLORS.Border, 1, 0.4)

local playerList = Instance.new("Frame")
playerList.Size = UDim2.new(1, 0, 0, 160)
playerList.BackgroundColor3 = COLORS.Surface2
playerList.Visible = false
playerList.ZIndex = 50
playerList.Parent = playerContent
corner(playerList, 12)
stroke(playerList, COLORS.Border, 1, 0.4)

local playerScroll = Instance.new("ScrollingFrame")
playerScroll.Size = UDim2.new(1, -10, 1, -10)
playerScroll.Position = UDim2.fromOffset(5, 5)
playerScroll.BackgroundTransparency = 1
playerScroll.BorderSizePixel = 0
playerScroll.ScrollBarThickness = 3
playerScroll.ZIndex = 51
playerScroll.Parent = playerList

local playerListLayout = Instance.new("UIListLayout")
playerListLayout.Padding = UDim.new(0, 5)
playerListLayout.Parent = playerScroll

local selectedLabel = textLabel(playerContent, "Target: None", UDim2.new(1, 0, 0, 18), UDim2.fromOffset(0, 0), 8, COLORS.SubText, Enum.Font.GothamSemibold)

createSection(playerContent, "CHỨC NĂNG")

local followHolder, followSet = createToggle(playerContent, "Follow", function(enabled) end)
local unstunHolder, unstunSet = createToggle(playerContent, "Unstun", function(enabled) end)

createSection(playerContent, "ESP")

local espHolder, espSet = createToggle(playerContent, "Player ESP", function(enabled) end)

local aimCard, aimContent = makeCard(
    aimPage,
    "Aim System",
    "Camlock và Aimbot cho Meme Sea"
)

createSection(aimContent, "CHẾ ĐỘ AIM")

local camlockHolder, camlockSet = createToggle(aimContent, "Camera Lock", function(enabled) end)
local aimbotHolder, aimbotSet = createToggle(aimContent, "Aimbot (Auto Cast)", function(enabled) end)

createSection(aimContent, "CAMLOCK")

createSlider(
    aimContent,
    "CAMLOCK FOV",
    100, 3000, camlockFOV,
    function(value)
        camlockFOV = value
        if camlockCircle then
            camlockCircle.Size = UDim2.fromOffset(value * 2 / 5, value * 2 / 5)
        end
    end
)

createSlider(
    aimContent,
    "CAMLOCK SMOOTH",
    1, 100, math.floor(camlockSmooth * 100),
    function(value)
        camlockSmooth = value / 100
    end
)

createSection(aimContent, "AIMBOT")

createSlider(
    aimContent,
    "AIMBOT FOV",
    100, 5000, aimbotFOV,
    function(value)
        aimbotFOV = value
        if aimbotCircle then
            aimbotCircle.Size = UDim2.fromOffset(value * 2 / 5, value * 2 / 5)
        end
    end
)

local keyRow = Instance.new("Frame")
keyRow.Size = UDim2.new(1, 0, 0, 36)
keyRow.BackgroundTransparency = 1
keyRow.Parent = aimContent

local aimKeyButton = Instance.new("TextButton")
aimKeyButton.Size = UDim2.new(0.48, 0, 1, 0)
aimKeyButton.BackgroundColor3 = COLORS.Surface2
aimKeyButton.Text = "KEY: E"
aimKeyButton.TextColor3 = COLORS.Accent
aimKeyButton.TextSize = 9
aimKeyButton.Font = Enum.Font.GothamBold
aimKeyButton.AutoButtonColor = false
aimKeyButton.Parent = keyRow
corner(aimKeyButton, 10)
stroke(aimKeyButton, COLORS.Border, 1, 0.4)

local aimModeBtn = Instance.new("TextButton")
aimModeBtn.Size = UDim2.new(0.48, 0, 1, 0)
aimModeBtn.Position = UDim2.new(0.52, 0, 0, 0)
aimModeBtn.BackgroundColor3 = COLORS.Surface2
aimModeBtn.Text = "MODE: AUTO"
aimModeBtn.TextColor3 = COLORS.Accent
aimModeBtn.TextSize = 9
aimModeBtn.Font = Enum.Font.GothamBold
aimModeBtn.AutoButtonColor = false
aimModeBtn.Parent = keyRow
corner(aimModeBtn, 10)
stroke(aimModeBtn, COLORS.Border, 1, 0.4)

local onlyEquippedHolder, onlyEquippedSet = createToggle(
    aimContent, "Only Equipped Skill",
    function(enabled)
        onlyEquippedOn = enabled
    end
)

local listeningKey = false

aimKeyButton.Activated:Connect(function()
    if listeningKey then return end
    listeningKey = true
    aimKeyButton.Text = "NHẤN PHÍM..."
    aimKeyButton.TextColor3 = COLORS.Orange
    local conn
    conn = UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            aimbotKey = input.KeyCode
            aimKeyButton.Text = "KEY: " .. aimbotKey.Name
            aimKeyButton.TextColor3 = COLORS.Accent
            listeningKey = false
            conn:Disconnect()
        end
    end)
end)

aimModeBtn.Activated:Connect(function()
    if not debounce("aimmode") then return end
    aimbotHold = not aimbotHold
    if aimbotHold then
        aimModeBtn.Text = "MODE: HOLD " .. aimbotKey.Name
        aimModeBtn.TextColor3 = COLORS.Orange
    else
        aimModeBtn.Text = "MODE: AUTO"
        aimModeBtn.TextColor3 = COLORS.Accent
    end
    if aimbotOn then
        setAimbot(false)
        task.wait(0.1)
        setAimbot(true)
    end
end)

onlyEquippedHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("onlyequipped") then return end
    onlyEquippedOn = not onlyEquippedOn
    onlyEquippedSet(onlyEquippedOn)
end)

camlockCircle = Instance.new("Frame")
camlockCircle.Name = "CamlockFOV"
camlockCircle.AnchorPoint = Vector2.new(0.5, 0.5)
camlockCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
camlockCircle.Size = UDim2.fromOffset(camlockFOV * 2 / 5, camlockFOV * 2 / 5)
camlockCircle.BackgroundTransparency = 1
camlockCircle.Visible = false
camlockCircle.Parent = gui
corner(camlockCircle, 999)
stroke(camlockCircle, COLORS.Accent, 1, 0.65)

aimbotCircle = Instance.new("Frame")
aimbotCircle.Name = "AimbotFOV"
aimbotCircle.AnchorPoint = Vector2.new(0.5, 0.5)
aimbotCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
aimbotCircle.Size = UDim2.fromOffset(aimbotFOV * 2 / 5, aimbotFOV * 2 / 5)
aimbotCircle.BackgroundTransparency = 1
aimbotCircle.Visible = false
aimbotCircle.Parent = gui
corner(aimbotCircle, 999)
stroke(aimbotCircle, COLORS.Purple, 1, 0.65)

local function setNoclip(enabled)
    noclipOn = enabled
    noclipSet(enabled)
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    for object, original in pairs(collisionCache) do
        if object and object.Parent then
            object.CanCollide = original
        end
    end
    table.clear(collisionCache)
    if not enabled then return end
    local character = getCharacter()
    if not character then return end
    local function disablePart(object)
        if not object:IsA("BasePart") then return end
        if collisionCache[object] == nil then
            collisionCache[object] = object.CanCollide
        end
        object.CanCollide = false
    end
    for _, object in ipairs(character:GetDescendants()) do
        disablePart(object)
    end
    noclipConnection = character.DescendantAdded:Connect(disablePart)
end

local function stopFly()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    if flyVelocity then
        flyVelocity:Destroy()
        flyVelocity = nil
    end
    if flyOrientation then
        flyOrientation:Destroy()
        flyOrientation = nil
    end
    if flyAttachment then
        flyAttachment:Destroy()
        flyAttachment = nil
    end
    local humanoid = getHumanoid()
    if humanoid then
        humanoid.AutoRotate = oldAutoRotate ~= nil and oldAutoRotate or true
        humanoid.PlatformStand = oldPlatformStand ~= nil and oldPlatformStand or false
        humanoid.Sit = false
    end
    oldAutoRotate = nil
    oldPlatformStand = nil
    flyOn = false
    flySet(false)
end

local function startFly()
    if flyOn then return end
    local humanoid = getHumanoid()
    local root = getRoot()
    if not humanoid or not root or humanoid.Health <= 0 then return end
    flyOn = true
    flySet(true)
    oldAutoRotate = humanoid.AutoRotate
    oldPlatformStand = humanoid.PlatformStand
    humanoid.AutoRotate = false
    humanoid.PlatformStand = false
    humanoid.Sit = false
    flyAttachment = Instance.new("Attachment")
    flyAttachment.Name = "VanhFlyAttachment"
    flyAttachment.Parent = root
    flyVelocity = Instance.new("LinearVelocity")
    flyVelocity.Name = "VanhFlyVelocity"
    flyVelocity.Attachment0 = flyAttachment
    flyVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
    flyVelocity.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
    flyVelocity.ForceLimitsEnabled = false
    flyVelocity.VectorVelocity = Vector3.zero
    flyVelocity.Parent = root
    flyOrientation = Instance.new("AlignOrientation")
    flyOrientation.Name = "VanhFlyOrientation"
    flyOrientation.Attachment0 = flyAttachment
    flyOrientation.Mode = Enum.OrientationAlignmentMode.OneAttachment
    flyOrientation.MaxTorque = math.huge
    flyOrientation.Responsiveness = 35
    flyOrientation.Parent = root
    flyConnection = RunService.RenderStepped:Connect(function()
        if not flyOn then return end
        local currentRoot = getRoot()
        local currentHumanoid = getHumanoid()
        local camera = Workspace.CurrentCamera
        if not currentRoot or not currentHumanoid
           or currentHumanoid.Health <= 0 or not camera then
            stopFly()
            return
        end
        local direction = Vector3.zero
        local look = camera.CFrame.LookVector
        local right = camera.CFrame.RightVector
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then direction += look end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then direction -= look end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then direction += right end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then direction -= right end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then direction += Vector3.yAxis end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then direction -= Vector3.yAxis end
        if direction.Magnitude > 0 then
            direction = direction.Unit
        end
        local velocity = direction * flyValue
        flyVelocity.VectorVelocity = velocity
        currentRoot.AssemblyLinearVelocity = velocity
        currentRoot.AssemblyAngularVelocity = Vector3.zero
        local horizontal = Vector3.new(look.X, 0, look.Z)
        if horizontal.Magnitude > 0 then
            flyOrientation.CFrame = CFrame.lookAt(
                currentRoot.Position,
                currentRoot.Position + horizontal.Unit
            )
        end
    end)
end

local STUN_NAMES = {
    stun = true, stunned = true, isstun = true, isstunned = true,
    freeze = true, frozen = true, isfrozen = true,
    immobilized = true, rooted = true, ragdolled = true,
    stunvalue = true, stuntime = true, stunnedvalue = true,
    stunnedtime = true, stunremaining = true, root = true,
    rootvalue = true, movementdisabled = true, walkdisabled = true,
    jumpdisabled = true, controlroot = true
}

local function clearStun()
    local character = getCharacter()
    local humanoid = getHumanoid()
    if not character or not humanoid then return end
    for attributeName, value in pairs(character:GetAttributes()) do
        local lower = string.lower(attributeName)
        if STUN_NAMES[lower] then
            pcall(function()
                if typeof(value) == "boolean" then
                    character:SetAttribute(attributeName, false)
                elseif typeof(value) == "number" then
                    character:SetAttribute(attributeName, 0)
                elseif typeof(value) == "string" then
                    character:SetAttribute(attributeName, "")
                end
            end)
        end
    end
    for _, obj in ipairs(character:GetDescendants()) do
        if obj:IsA("BoolValue") and STUN_NAMES[string.lower(obj.Name)] then
            obj.Value = false
        elseif obj:IsA("NumberValue") and STUN_NAMES[string.lower(obj.Name)] then
            obj.Value = 0
        end
    end
    if humanoid.PlatformStand then humanoid.PlatformStand = false end
    if humanoid.Sit then humanoid.Sit = false end
    if humanoid.WalkSpeed < 16 then
        humanoid.WalkSpeed = speedOn and speedValue or 16
    end
    if humanoid.UseJumpPower then
        if humanoid.JumpPower < 50 then
            humanoid.JumpPower = jumpOn and jumpValue or 50
        end
    else
        if humanoid.JumpHeight < 7 then
            humanoid.JumpHeight = jumpOn and jumpValue or 7
        end
    end
    local root = getRoot()
    if root then
        if root.Anchored then root.Anchored = false end
        if root.AssemblyLinearVelocity.Magnitude < 0.5
           and root.AssemblyAngularVelocity.Magnitude > 5 then
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end
    local torso = character:FindFirstChild("Torso")
                or character:FindFirstChild("UpperTorso")
    if torso then
        local neck = torso:FindFirstChild("Neck")
        local waist = torso:FindFirstChild("Waist")
        if neck and neck:IsA("Motor6D") and not neck.Enabled then
            neck.Enabled = true
        end
        if waist and waist:IsA("Motor6D") and not waist.Enabled then
            waist.Enabled = true
        end
    end
    for _, obj in ipairs(character:GetDescendants()) do
        if obj:IsA("ParticleEmitter") and obj.Enabled then
            local name = string.lower(obj.Name)
            if string.find(name, "stun")
               or string.find(name, "freeze")
               or string.find(name, "frozen") then
                obj.Enabled = false
            end
        end
    end
    for _, obj in ipairs(character:GetDescendants()) do
        if obj:IsA("BodyVelocity") or obj:IsA("BodyPosition")
           or obj:IsA("BodyGyro") then
            local name = string.lower(obj.Name)
            if string.find(name, "stun")
               or string.find(name, "freeze")
               or string.find(name, "hold") then
                obj:Destroy()
            end
        end
    end
end

local function setUnstun(enabled)
    unstunOn = enabled
    unstunSet(enabled)
    if unstunConnection then
        unstunConnection:Disconnect()
        unstunConnection = nil
    end
    if enabled then
        local accum = 0
        unstunConnection = RunService.Heartbeat:Connect(function(dt)
            if not unstunOn then return end
            accum += dt
            if accum < 0.05 then return end
            accum = 0
            clearStun()
        end)
    end
end

local function stopFollow()
    followOn = false
    followSet(false)
    if followConnection then
        followConnection:Disconnect()
        followConnection = nil
    end
end

local function startFollow()
    if not followTarget or followTarget == player or not followTarget.Parent then
        return
    end
    if flyOn then stopFly() end
    followOn = true
    followSet(true)
    if followConnection then
        followConnection:Disconnect()
    end
    followConnection = RunService.Heartbeat:Connect(function()
        if not followOn then return end
        if not followTarget or not followTarget.Parent then
            stopFollow()
            return
        end
        local character = getCharacter()
        local targetCharacter = followTarget.Character
        if not character or not targetCharacter then return end
        local root = character:FindFirstChild("HumanoidRootPart")
        local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
        if root and targetRoot then
            character:PivotTo(targetRoot.CFrame * CFrame.new(0, 0, followDistance))
            root.AssemblyLinearVelocity = targetRoot.AssemblyLinearVelocity
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

local function refreshPlayerList()
    for _, child in ipairs(playerScroll:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player then
            local item = Instance.new("TextButton")
            item.Size = UDim2.new(1, -6, 0, 32)
            item.BackgroundColor3 = target == followTarget
                                    and COLORS.Surface3
                                    or COLORS.Surface2
            item.Text = "  " .. target.DisplayName .. "   @" .. target.Name
            item.TextColor3 = COLORS.Text
            item.TextSize = 8
            item.Font = Enum.Font.GothamMedium
            item.TextXAlignment = Enum.TextXAlignment.Left
            item.AutoButtonColor = false
            item.Parent = playerScroll
            item.ZIndex = 52
            corner(item, 8)
            item.Activated:Connect(function()
                followTarget = target
                selectPlayer.Text = "  " .. target.DisplayName .. "   @" .. target.Name
                selectedLabel.Text = "Target: " .. target.DisplayName .. "  @" .. target.Name
                playerList.Visible = false
                refreshPlayerList()
            end)
        end
    end
    task.defer(function()
        playerScroll.CanvasSize = UDim2.fromOffset(
            0,
            playerListLayout.AbsoluteContentSize.Y + 8
        )
    end)
end

selectPlayer.Activated:Connect(function()
    playerList.Visible = not playerList.Visible
    if playerList.Visible then
        refreshPlayerList()
    end
end)

local function isLagEffect(object)
    return object:IsA("ParticleEmitter")
        or object:IsA("Trail")
        or object:IsA("Beam")
        or object:IsA("Smoke")
        or object:IsA("Fire")
        or object:IsA("Sparkles")
        or object:IsA("PostEffect")
        or object:IsA("Explosion")
end

local function isHeavyLight(object)
    return object:IsA("PointLight")
        or object:IsA("SpotLight")
        or object:IsA("SurfaceLight")
end

local function optimizeObject(object)
    if object:IsA("BasePart") then
        if lagCache[object] == nil then
            lagCache[object] = {
                kind = "part",
                castShadow = object.CastShadow,
                material = object.Material,
                reflectance = object.Reflectance
            }
        end
        object.CastShadow = false
        object.Reflectance = 0
        if object.Material ~= Enum.Material.Plastic
           and object.Material ~= Enum.Material.SmoothPlastic then
            object.Material = Enum.Material.Plastic
        end
        if object:IsA("MeshPart") or object:IsA("UnionOperation") then
            if lagCache[object].renderFidelity == nil then
                lagCache[object].renderFidelity = object.RenderFidelity
            end
            pcall(function()
                object.RenderFidelity = Enum.RenderFidelity.Performance
            end)
        end
        for _, child in ipairs(object:GetChildren()) do
            if child:IsA("Decal") or child:IsA("Texture") then
                if lagCache[child] == nil then
                    lagCache[child] = { kind = "decal", transparency = child.Transparency }
                end
                child.Transparency = 1
            end
        end
    elseif isLagEffect(object) then
        if lagCache[object] == nil then
            lagCache[object] = { kind = "effect", enabled = object.Enabled }
        end
        object.Enabled = false
    elseif isHeavyLight(object) then
        if lagCache[object] == nil then
            lagCache[object] = { kind = "light", enabled = object.Enabled }
        end
        object.Enabled = false
    elseif object:IsA("Sound") then
        if lagCache[object] == nil then
            lagCache[object] = { kind = "sound", volume = object.Volume }
        end
        object.Volume = 0
    end
end

local function restoreAll()
    for object, data in pairs(lagCache) do
        if object and object.Parent then
            pcall(function()
                if data.kind == "part" then
                    object.CastShadow = data.castShadow
                    object.Reflectance = data.reflectance
                    object.Material = data.material
                    if object:IsA("MeshPart") and data.renderFidelity then
                        object.RenderFidelity = data.renderFidelity
                    end
                elseif data.kind == "effect" or data.kind == "light" then
                    object.Enabled = data.enabled
                elseif data.kind == "decal" then
                    object.Transparency = data.transparency
                elseif data.kind == "sound" then
                    object.Volume = data.volume
                end
            end)
        end
    end
    table.clear(lagCache)
end

local function setLagFix(enabled)
    lagFixOn = enabled
    lagSet(enabled)
    for _, conn in ipairs({lagConnection, lagConnection2, lagConnection3, lagConnection4}) do
        if conn then conn:Disconnect() end
    end
    lagConnection, lagConnection2, lagConnection3, lagConnection4 = nil, nil, nil, nil
    if not enabled then
        restoreAll()
        pcall(function()
            Lighting.GlobalShadows = true
            Lighting.FogEnd = 100000
            Lighting.Brightness = 1
            Lighting.EnvironmentDiffuseScale = 1
            Lighting.EnvironmentSpecularScale = 1
        end)
        pcall(function()
            local s = UserSettings():GetService("UserGameSettings")
            s.SavedQualityLevel = Enum.SavedQualitySetting.Automatic
        end)
        statusLabel.Text = "READY"
        statusLabel.TextColor3 = COLORS.SubText
        statusDot.BackgroundColor3 = COLORS.SubText
        return
    end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        optimizeObject(obj)
    end
    for _, obj in ipairs(Lighting:GetDescendants()) do
        optimizeObject(obj)
    end
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 1e6
        Lighting.Brightness = 1.5
        Lighting.Ambient = Color3.fromRGB(60, 60, 65)
        Lighting.OutdoorAmbient = Color3.fromRGB(90, 90, 95)
        Lighting.EnvironmentDiffuseScale = 0
        Lighting.EnvironmentSpecularScale = 0
        Lighting.ClockTime = 14
    end)
    for _, obj in ipairs(Lighting:GetChildren()) do
        if obj:IsA("PostEffect") or obj:IsA("Atmosphere")
           or obj:IsA("Sky") or obj:IsA("Clouds") then
            if lagCache[obj] == nil then
                lagCache[obj] = { kind = "effect", enabled = obj.Enabled }
            end
            pcall(function() obj.Enabled = false end)
        end
    end
    pcall(function()
        local s = UserSettings():GetService("UserGameSettings")
        s.SavedQualityLevel = Enum.SavedQualitySetting.QualityLevel1
    end)
    pcall(function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    end)
    pcall(function()
        local terrain = Workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            terrain.WaterWaveSize = 0
            terrain.WaterWaveSpeed = 0
            terrain.WaterReflectance = 0
            terrain.WaterTransparency = 1
            terrain.Decoration = false
        end
    end)
    lagConnection = Workspace.DescendantAdded:Connect(function(obj)
        if lagFixOn then
            task.defer(function()
                if lagFixOn and obj.Parent then
                    optimizeObject(obj)
                end
            end)
        end
    end)
    lagConnection2 = Lighting.DescendantAdded:Connect(function(obj)
        if lagFixOn then
            task.defer(function()
                if lagFixOn and obj.Parent then
                    optimizeObject(obj)
                end
            end)
        end
    end)
    local scanAccum = 0
    lagConnection3 = RunService.Heartbeat:Connect(function(dt)
        if not lagFixOn then return end
        scanAccum += dt
        if scanAccum < 3 then return end
        scanAccum = 0
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.CastShadow then
                pcall(function() obj.CastShadow = false end)
            end
            if isLagEffect(obj) and obj.Enabled then
                pcall(function() obj.Enabled = false end)
            end
            if isHeavyLight(obj) and obj.Enabled then
                pcall(function() obj.Enabled = false end)
            end
        end
    end)
    lagConnection4 = game:GetService("SoundService").DescendantAdded:Connect(function(obj)
        if lagFixOn and obj:IsA("Sound") then
            task.defer(function()
                if lagFixOn and obj.Parent then
                    if lagCache[obj] == nil then
                        lagCache[obj] = { kind = "sound", volume = obj.Volume }
                    end
                    obj.Volume = 0
                end
            end)
        end
    end)
    statusLabel.Text = "PRO OPTIMIZED"
    statusLabel.TextColor3 = COLORS.Accent
    statusDot.BackgroundColor3 = COLORS.Accent
end

local function removeESP(target)
    local data = espCache[target]
    if not data then return end
    if data.highlight then data.highlight:Destroy() end
    if data.billboard then data.billboard:Destroy() end
    espCache[target] = nil
end

local function createESP(target)
    if target == player or not target.Parent or not espOn then return end
    removeESP(target)
    local character = target.Character
    local head = character and character:FindFirstChild("Head")
    if not character or not head then return end
    local highlight = Instance.new("Highlight")
    highlight.Name = "VanhPlayerESP"
    highlight.FillColor = COLORS.Accent
    highlight.FillTransparency = 0.78
    highlight.OutlineColor = COLORS.Accent
    highlight.OutlineTransparency = 0.1
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Adornee = character
    highlight.Parent = character
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "VanhPlayerESP"
    billboard.Adornee = head
    billboard.Size = UDim2.fromOffset(190, 44)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = head
    local nameText = Instance.new("TextLabel")
    nameText.BackgroundTransparency = 1
    nameText.Size = UDim2.new(1, 0, 0, 22)
    nameText.Text = target.DisplayName .. "  @" .. target.Name
    nameText.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameText.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameText.TextStrokeTransparency = 0.35
    nameText.TextSize = 10
    nameText.Font = Enum.Font.GothamBold
    nameText.Parent = billboard
    local distanceText = Instance.new("TextLabel")
    distanceText.BackgroundTransparency = 1
    distanceText.Position = UDim2.fromOffset(0, 21)
    distanceText.Size = UDim2.new(1, 0, 0, 18)
    distanceText.Text = "0 studs"
    distanceText.TextColor3 = COLORS.Accent
    distanceText.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    distanceText.TextStrokeTransparency = 0.4
    distanceText.TextSize = 9
    distanceText.Font = Enum.Font.GothamSemibold
    distanceText.Parent = billboard
    espCache[target] = {
        highlight = highlight,
        billboard = billboard,
        distance = distanceText
    }
end

local function clearESP()
    for target in pairs(espCache) do
        removeESP(target)
    end
end

local function setESP(enabled)
    espOn = enabled
    espSet(enabled)
    if espConnection then
        espConnection:Disconnect()
        espConnection = nil
    end
    clearESP()
    if not enabled then return end
    for _, target in ipairs(Players:GetPlayers()) do
        createESP(target)
    end
    espConnection = RunService.Heartbeat:Connect(function()
        local root = getRoot()
        if not root then return end
        for target, data in pairs(espCache) do
            local targetRoot = target.Character
                            and target.Character:FindFirstChild("HumanoidRootPart")
            local targetHead = target.Character
                            and target.Character:FindFirstChild("Head")
            if not target.Parent or not targetRoot or not targetHead then
                removeESP(target)
            else
                if data.billboard and data.billboard.Adornee ~= targetHead then
                    data.billboard.Adornee = targetHead
                end
                local distance = (root.Position - targetRoot.Position).Magnitude
                data.distance.Text = tostring(math.floor(distance + 0.5)) .. " studs"
            end
        end
    end)
end

local function getClosestTarget(fov, part)
    local camera = Workspace.CurrentCamera
    if not camera then return nil end
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local closest, shortest = nil, fov
    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and target.Character then
            local targetPart = target.Character:FindFirstChild(part)
                            or target.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local humanoid = target.Character:FindFirstChildOfClass("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    local pos, onScreen = camera:WorldToViewportPoint(targetPart.Position)
                    if onScreen then
                        local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                        if dist < shortest then
                            shortest = dist
                            closest = targetPart
                        end
                    end
                end
            end
        end
    end
    return closest
end

local function setCamlock(enabled)
    camlockOn = enabled
    camlockSet(enabled)
    if camlockConnection then
        camlockConnection:Disconnect()
        camlockConnection = nil
    end
    if camlockCircle then
        camlockCircle.Visible = enabled
    end
    if not enabled then return end
    camlockConnection = RunService.RenderStepped:Connect(function()
        if not camlockOn then return end
        local camera = Workspace.CurrentCamera
        if not camera then return end
        local target = getClosestTarget(camlockFOV, "Head")
        if target then
            local newCF = CFrame.lookAt(camera.CFrame.Position, target.Position)
            camera.CFrame = camera.CFrame:Lerp(newCF, camlockSmooth)
        end
    end)
end

local function setAimbot(enabled)
    aimbotOn = enabled
    aimbotSet(enabled)
    if aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
    if aimbotInputConn then
        aimbotInputConn:Disconnect()
        aimbotInputConn = nil
    end
    if aimbotCircle then
        aimbotCircle.Visible = enabled
    end
    if not enabled then
        lockedTarget = nil
        return
    end
    local function getEquippedTools()
        local char = player.Character
        if not char then return {} end
        local tools = {}
        for _, tool in ipairs(char:GetChildren()) do
            if tool:IsA("Tool") then
                table.insert(tools, tool)
            end
        end
        return tools
    end
    local function lockCamera(target)
        local camera = Workspace.CurrentCamera
        if not camera then return end
        local newCF = CFrame.lookAt(camera.CFrame.Position, target.Position)
        camera.CFrame = newCF
    end
    local function castSkill(target)
        if not target or not target.Parent then return end
        local humanoid = target.Parent:FindFirstChildOfClass("Humanoid")
        if not humanoid or humanoid.Health <= 0 then return end
        lockCamera(target)
        task.wait(0.03)
        if onlyEquippedOn then
            local char = player.Character
            local current = char and char:FindFirstChildOfClass("Tool")
            if current then
                pcall(function() current:Activate() end)
            end
            return
        end
        local equipped = getEquippedTools()
        for _, tool in ipairs(equipped) do
            pcall(function() tool:Activate() end)
            task.wait(0.05)
        end
    end
    if aimbotHold then
        aimbotInputConn = UserInputService.InputBegan:Connect(function(input, gp)
            if gp then return end
            if input.KeyCode == aimbotKey then
                local target = getClosestTarget(aimbotFOV, aimbotPart)
                if target then
                    lockedTarget = target
                    castSkill(target)
                end
            end
        end)
    end
    aimbotConnection = RunService.RenderStepped:Connect(function()
        if not aimbotOn then return end
        local target = getClosestTarget(aimbotFOV, aimbotPart)
        if target then
            lockedTarget = target
            lockCamera(target)
            if not aimbotHold then
                task.spawn(function()
                    local humanoid = target.Parent
                                    and target.Parent:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid.Health > 0 then
                        if onlyEquippedOn then
                            local char = player.Character
                            local current = char and char:FindFirstChildOfClass("Tool")
                            if current then
                                pcall(function() current:Activate() end)
                            end
                        else
                            local equipped = getEquippedTools()
                            for _, tool in ipairs(equipped) do
                                pcall(function() tool:Activate() end)
                            end
                        end
                    end
                end)
            end
        else
            lockedTarget = nil
        end
    end)
end

local pages = {
    movement = movementPage,
    player = playerPage,
    aim = aimPage
}

local navNames = {
    movement = "MOVEMENT",
    player = "PLAYER",
    aim = "AIM"
}

local function setPage(page)
    if not pages[page] then return end
    currentPage = page
    for name, object in pairs(pages) do
        object.Visible = name == page
    end
    selectNav(navNames[page])
end

movementNav.Activated:Connect(function() setPage("movement") end)
playerNav.Activated:Connect(function() setPage("player") end)
aimNav.Activated:Connect(function() setPage("aim") end)

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
       and input.Position.X < header.AbsolutePosition.X + header.AbsoluteSize.X - 100 then
        dragging = true
        dragStart = input.Position
        panelStart = panel.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        panel.Position = UDim2.new(
            panelStart.X.Scale,
            panelStart.X.Offset + delta.X,
            panelStart.Y.Scale,
            panelStart.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

local closePanel

local confirmFrame = Instance.new("Frame")
confirmFrame.Size = UDim2.fromOffset(240, 120)
confirmFrame.AnchorPoint = Vector2.new(0.5, 0.5)
confirmFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
confirmFrame.BackgroundColor3 = COLORS.Surface
confirmFrame.Visible = false
confirmFrame.ZIndex = 100
confirmFrame.Parent = gui
corner(confirmFrame, 16)
stroke(confirmFrame, COLORS.Border, 1, 0.3)

textLabel(confirmFrame, "Close Vanh Hub?", UDim2.new(1, -30, 0, 24), UDim2.fromOffset(14, 12), 13, COLORS.Text, Enum.Font.GothamBold).ZIndex = 101
textLabel(confirmFrame, "Are you sure?", UDim2.new(1, -30, 0, 20), UDim2.fromOffset(14, 36), 8, COLORS.SubText, Enum.Font.GothamMedium).ZIndex = 101

local confirmCancel = Instance.new("TextButton")
confirmCancel.Size = UDim2.fromOffset(90, 30)
confirmCancel.Position = UDim2.fromOffset(14, 76)
confirmCancel.BackgroundColor3 = COLORS.Surface2
confirmCancel.Text = "CANCEL"
confirmCancel.TextColor3 = COLORS.Text
confirmCancel.TextSize = 8
confirmCancel.Font = Enum.Font.GothamBold
confirmCancel.AutoButtonColor = false
confirmCancel.ZIndex = 101
confirmCancel.Parent = confirmFrame
corner(confirmCancel, 10)

local confirmClose = Instance.new("TextButton")
confirmClose.Size = UDim2.fromOffset(90, 30)
confirmClose.Position = UDim2.fromOffset(120, 76)
confirmClose.BackgroundColor3 = COLORS.AccentDark
confirmClose.Text = "CLOSE"
confirmClose.TextColor3 = Color3.fromRGB(255, 255, 255)
confirmClose.TextSize = 8
confirmClose.Font = Enum.Font.GothamBold
confirmClose.AutoButtonColor = false
confirmClose.ZIndex = 101
confirmClose.Parent = confirmFrame
corner(confirmClose, 10)

confirmCancel.Activated:Connect(function()
    confirmFrame.Visible = false
end)

confirmClose.Activated:Connect(function()
    confirmFrame.Visible = false
    closePanel()
end)

closePanel = function()
    menuOpen = false
    tween(panel, 0.18, {
        BackgroundTransparency = 1,
        Position = UDim2.new(0.5, 0, 0.5, 25)
    })
    tween(panelScale, 0.18, { Scale = 0.9 })
    task.delay(0.18, function()
        if not menuOpen then
            panel.Visible = false
        end
    end)
end

local hubDragging = false
local hubDragStart
local hubStart

hubButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        hubDragging = true
        hubDragStart = input.Position
        hubStart = hubButton.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if hubDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - hubDragStart
        hubButton.Position = UDim2.new(
            hubStart.X.Scale,
            hubStart.X.Offset + delta.X,
            hubStart.Y.Scale,
            hubStart.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        hubDragging = false
    end
end)

hubButton.Activated:Connect(function()
    if menuOpen then
        closePanel()
        return
    end
    menuOpen = true
    minimized = false
    minimizeButton.Text = "—"
    panel.Visible = true
    sidebar.Visible = true
    content.Visible = true
    panel.Size = UDim2.fromOffset(680, 460)
    panel.BackgroundTransparency = 1
    panel.Position = UDim2.new(0.5, 0, 0.5, 25)
    panelScale.Scale = 0.9
    tween(panel, 0.22, {
        BackgroundTransparency = 0,
        Position = UDim2.new(0.5, 0, 0.5, 0)
    })
    tween(panelScale, 0.28, { Scale = 1 }, Enum.EasingStyle.Back)
end)

minimizeButton.Activated:Connect(function()
    if not debounce("minimize") then return end
    minimized = not minimized
    if minimized then
        panel.Size = UDim2.fromOffset(680, 56)
        sidebar.Visible = false
        content.Visible = false
        minimizeButton.Text = "+"
    else
        panel.Size = UDim2.fromOffset(680, 460)
        sidebar.Visible = true
        content.Visible = true
        minimizeButton.Text = "—"
    end
end)

closeButton.Activated:Connect(function()
    if not debounce("closebtn", 0.3) then return end
    confirmFrame.Visible = true
end)

local frames = 0
local elapsed = 0

fpsConnection = RunService.RenderStepped:Connect(function(dt)
    frames += 1
    elapsed += dt
    if elapsed >= 1 then
        local fps = math.round(frames / elapsed)
        if fpsOn then
            fpsBadge.Visible = true
            fpsText.Text = "FPS  " .. tostring(fps)
        else
            fpsBadge.Visible = false
            fpsText.Text = ""
        end
        if fps >= 50 then
            fpsDot.BackgroundColor3 = COLORS.Accent
        elseif fps >= 30 then
            fpsDot.BackgroundColor3 = COLORS.Orange
        else
            fpsDot.BackgroundColor3 = COLORS.Red
        end
        frames = 0
        elapsed = 0
    end
end)

characterConnection = player.CharacterAdded:Connect(function()
    task.wait(0.5)
    if flyOn then stopFly() end
    if noclipOn then
        setNoclip(false)
        task.wait()
        setNoclip(true)
    end
    applyMovement()
    if unstunOn then clearStun() end
    if espOn then
        task.wait(0.2)
        setESP(true)
    end
end)

Players.PlayerAdded:Connect(function(target)
    target.CharacterAdded:Connect(function()
        if espOn then
            task.wait(0.25)
            createESP(target)
        end
    end)
    task.defer(refreshPlayerList)
end)

Players.PlayerRemoving:Connect(function(target)
    removeESP(target)
    if followTarget == target then
        stopFollow()
        followTarget = nil
        selectPlayer.Text = "  SELECT PLAYER                              ▾"
        selectedLabel.Text = "Target: None"
    end
    task.defer(refreshPlayerList)
end)

flyHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("fly") then return end
    if flyOn then
        stopFly()
    else
        if followOn then stopFollow() end
        startFly()
    end
end)

noclipHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("noclip") then return end
    setNoclip(not noclipOn)
end)

speedHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("speed") then return end
    speedOn = not speedOn
    speedSet(speedOn)
    applyMovement()
end)

jumpHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("jump") then return end
    jumpOn = not jumpOn
    jumpSet(jumpOn)
    applyMovement()
end)

fpsHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("fps") then return end
    fpsOn = not fpsOn
    fpsSet(fpsOn)
    fpsBadge.Visible = fpsOn
    if not fpsOn then
        fpsText.Text = ""
    end
end)

lagHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("lagfix") then return end
    setLagFix(not lagFixOn)
end)

followHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("follow") then return end
    if followOn then stopFollow() else startFollow() end
end)

unstunHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("unstun") then return end
    setUnstun(not unstunOn)
end)

espHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("esp") then return end
    setESP(not espOn)
end)

camlockHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("camlock") then return end
    setCamlock(not camlockOn)
end)

aimbotHolder:FindFirstChildOfClass("TextButton").Activated:Connect(function()
    if not debounce("aimbot") then return end
    setAimbot(not aimbotOn)
end)

selectNav("MOVEMENT")
refreshPlayerList()
fpsBadge.Visible = false
