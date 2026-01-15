local Env = getgenv()

if Env.Lumora then
    Env.Lumora.Utility.unload()
end

Env.Lumora = {}

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local Workspace = game:GetService("Workspace")
local GuiService = game:GetService("GuiService")
local Lighting = game:GetService("Lighting")
local Stats = game:GetService("Stats")

if not LPH_OBFUSCATED then
    LPH_ENCSTR = function(...) return ... end
    LPH_ENCNUM = function(...) return ... end
    LPH_CRASH = function(...) return ... end
    LPH_JIT = function(...) return ... end
    LPH_JIT_MAX = function(...) return ... end
    LPH_NO_VIRTUALIZE = function(...) return ... end
    LPH_NO_UPVALUES = function(...) return ... end
end

local Camera = workspace.CurrentCamera
local HiddenUI = gethui()
local LocalPlayer = Players.LocalPlayer
local Loaded = false
local Acceleration = Vector3.new(0, -Workspace.Gravity, 0)

local BinLib = {
    methods = {
        ["function"] = true,
        thread = true,
        RBXScriptConnection = "Disconnect"
    }
}

function BinLib.bin()
    local bin = {storage = {}}

    function bin:add(item, method)
        table.insert(self.storage, {
            main = item,
            destroy = function()
                local destroyMethod = method or BinLib.methods[typeof(item)] or "Destroy"

                if destroyMethod == true then
                    if type(item) == "function" then
                        item()
                    else
                        task.cancel(item)
                    end
                else
                    local callMethod = item[destroyMethod]
                    if callMethod then
                        callMethod(item)
                    end
                end
            end
        })
        return item
    end

    function bin:destroy()
        if not (self and self.storage) then return end
        for _, entry in ipairs(self.storage) do
            if not (entry and entry.destroy) then continue end
            entry.destroy()
        end
        self.storage = nil
    end

    return bin
end

Env.Lumora.BinLib = BinLib
Bin = BinLib.bin()
Env.Lumora.Bin = Bin

local Utility = {}

function Utility.unload()
    Bin:destroy()
    Env.Lumora = nil
end

function Utility.Connect(connection)
    Bin:add(connection)
    return connection
end

function Utility.New(sqr, props)
    local Obj = Instance.new(sqr)
    if props then
        for _, v in props do
            Obj[_] = v
        end
    end
    Bin:add(Obj)
    return Obj
end

function Utility.Round(number, float)
    local Mult = 1 / (float or 1)
    return math.floor(number * Mult + 0.5) / Mult
end

function Utility.GetTrans(obj)
    local Index = nil
    if obj:IsA("Frame") then
        Index = "BackgroundTransparency"
    elseif obj:IsA("TextLabel") or obj:IsA("TextButton") then
        Index = "TextTransparency"
    elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
        Index = "ImageTransparency"
    elseif obj:IsA("ScrollingFrame") then
        Index = "BackgroundTransparency"
    elseif obj:IsA("TextBox") then
        Index = "TextTransparency"
    end
    return Index
end

function Utility.GetFiles(folder, extensions)
    if not isfolder(folder) then
        makefolder(folder)
    end

    local Files = isfolder(folder) and listfiles(folder) or {}
    local StoredFiles = {}
    local FileNames = {}

    for _, v in Files do
        for _, ext in extensions do
            if v:find(ext) then
                StoredFiles[#StoredFiles + 1] = v
                FileNames[#FileNames + 1] = v:gsub(folder, ""):gsub(ext, "")
            end
        end
    end

    return StoredFiles, FileNames
end

function Utility.Lerp(a, b, c)
    c = c or 1 / 8
    local offset = math.abs(b - a)
    if (offset < c) then
        return b
    end
    return a + (b - a) * c
end

function Utility.LineMath(obj, from, to, thickness)
    local Direction = (to - from)
    local Center = (to + from) / 2
    local Distance = Direction.Magnitude
    local Theta = math.deg(math.atan2(Direction.Y, Direction.X))

    obj.Position = UDim2.fromOffset(Center.x, Center.y)
    obj.Rotation = Theta
    obj.Size = UDim2.fromOffset(Distance, thickness)
end

Utility.Signal = Utility.Connect
Env.Lumora.Utility = Utility

local Games = {
    Data = {
        ["Arsenal"] = {
            Name = "Arsenal",
            GameId = 111958650,
            GetInfo = function(data, player, ...)
                local NRPBS = player:FindFirstChild("NRPBS")
                local Health = NRPBS and NRPBS:FindFirstChild("Health")
                local MaxHealth = NRPBS and NRPBS:FindFirstChild("MaxHealth")
                local EquippedTool = NRPBS and NRPBS:FindFirstChild("EquippedTool")

                data.Tool = EquippedTool and EquippedTool.Value or nil
                data.Health = Health and Health.Value or 0
                data.MaxHealth = MaxHealth and MaxHealth.Value or 100
            end
        },
        ["Apocalypse Rising 2"] = {
            Name = "Apocalypse Rising 2",
            GameId = 358276974,
            GetPlayers = function()
                local Entities = Players:GetPlayers()
                local Zombies = workspace:FindFirstChild("Zombies")

                if not Zombies then
                    return Entities
                end

                local Mobs = Zombies:FindFirstChild("Mobs")

                if not Mobs then
                    return Entities
                end

                for _, mob in Mobs:GetChildren() do
                    table.insert(Entities, mob)
                end

                return Entities
            end,
            GetInfo = function(data, player, ...)
                if data.IsPlayer then
                    local Stats = player:FindFirstChild("Stats")
                    local Health = Stats and Stats:FindFirstChild("Health")
                    local Weapon = Stats and Stats:FindFirstChild("Primary")

                    data.Tool = Weapon and Weapon.Value or nil
                    data.Health = Health and Health.Value or 0
                else
                    data.Tool = "None"
                    data.Health = 100
                    data.MaxHealth = 100
                    data.Team = nil
                    data.IsTeammate = false
                end
            end
        },
        ["Typical Colors 2"] = {
            Name = "Typical Colors 2",
            GameId = 147332621,
            GetInfo = function(data, player, ...)
                local Character = data.Character
                local Gun = Character and Character:FindFirstChild("Gun")
                local Boop = Gun and Gun:FindFirstChild("Boop")

                data.Tool = Boop and Boop.Value or nil
            end
        }
    }
}

function Games.GetGame()
    for _, v in Games.Data do
        if game.GameId == v.GameId then
            return v
        end
    end
    return nil
end

local Game = Games.GetGame()
local GameName = Game and Game.Name or ""

local Folder = "Lumora"
makefolder(Folder)

local ImagesFolder = string.format("%s\\%s\\", Folder, "Images")
makefolder(ImagesFolder)

local FontsFolder = string.format("%s\\%s\\", Folder, "FontsFolder")
makefolder(FontsFolder)

local Games = string.format("%s\\%s", Folder, "Games")
makefolder(Games)

local MainFolder = string.format("%s\\%s", Games, Game and Game.Name or "Universal")
makefolder(MainFolder)

local ConfigFolder = string.format("%s\\%s\\", MainFolder, "Configs")
makefolder(ConfigFolder)

local Fonts = {
    URL = "https://raw.githubusercontent.com/luminbot/FontStorage/main/",
    Names = {
        "Proggy",
        "ProggyTiny",
        "Minecraftia",
        "SmallestPixel7",
        "Verdana",
        "VerdanaBold",
        "Tahoma",
        "ProggyClean",
        "TahomaXP"
    },
    Data = {}
}

function Fonts.Load(font)
    local TTF = string.format("%s%s.ttf", FontsFolder, font)
    local JSON = string.format("%s%s.json", FontsFolder, font)

    if not isfile(TTF) then
        local success, data = pcall(function()
            return game:HttpGet(string.format("%s%s.txt", Fonts.URL, font))
        end)

        if success and data then
            writefile(TTF, base64_decode(data))
        else
            return
        end
    end

    if not isfile(JSON) then
        local Font = {
            name = font,
            faces = {{
                name = "Regular",
                weight = 400,
                style = "normal",
                assetId = getcustomasset(TTF)
            }}
        }

        writefile(JSON, HttpService:JSONEncode(Font))
    end

    Fonts.Data[font] = Font.new(getcustomasset(JSON), Enum.FontWeight.Regular)
end

function Fonts.Get(font)
    return Fonts.Data[font]
end

for _, font in Fonts.Names do
    Fonts.Load(font)
end

Env.Lumora.Fonts = Fonts

local Images = {
    URL = "https://raw.githubusercontent.com/OxygenClub/LUAS/refs/heads/main/pictures/",
    Names = {"gear"},
    Data = {}
}

function Images.Load(image)
    local PNG = string.format("%s%s.png", ImagesFolder, image)

    if not isfile(PNG) then
        local success, data = pcall(function()
            return game:HttpGet(string.format("%s%s.txt", Images.URL, image))
        end)

        if success and data then
            writefile(PNG, base64_decode(data))
        else
            return
        end
    end

    Images.Data[image] = getcustomasset(PNG)
end

function Images.Get(image)
    return Images.Data[image]
end

for _, image in Images.Names do
    Images.Load(image)
end

Env.Lumora.Images = Images
local Library = {
    TweenSpeed = 0.2,
    TweenStyle = Enum.EasingStyle.Quad,
    ThemeObjects = {},
    Font = Font.new("rbxassetid://12187365364", Enum.FontWeight.Bold),
    FontSize = 15,
    Flags = {},
    ConfigFlags = {},
    Popups = {},
    Dropdowns = {},
    Fps = 0,
    ScrollBar = "rbxassetid://12776289446",
    Saturation = "rbxassetid://13901004307",
    Checkers = "http://www.roblox.com/asset/?id=18274452449",
    Converts = {[0] = "0", "1", "2", "3", "4", "5", "6", "7", "8", "9"},
    KeyConverters = {
        escape = "ESC", backquote = "`", backspace = "BSP", slash = "/",
        leftquote = "'", rightquote = '"', leftbracket = "[", rightbracket = "]",
        semicolon = ";", comma = ",", period = ".", backslash = "\\",
        minus = "-", equals = "=", space = "SPC", ["return"] = "ENT",
        tab = "TAB", capslock = "CAP", leftshift = "LSH",
        mousebutton1 = "MB1", mousebutton2 = "MB2", mousebutton3 = "MB3",
        rightshift = "RSH", leftcontrol = "CTRL", leftalt = "ALT",
        leftsuper = "WIN", rightcontrol = "CTRL", rightalt = "ALT",
        rightsuper = "WIN", insert = "INS", delete = "DEL",
        home = "HME", pageup = "PUD", pagedown = "PDN",
        up = "UP", down = "DWN", left = "LFT", right = "RGT",
        numlock = "NUM", numpad0 = "N0", numpad1 = "N1", numpad2 = "N2",
        numpad3 = "N3", numpad4 = "N4", numpad5 = "N5", numpad6 = "N6",
        numpad7 = "N7", numpad8 = "N8", numpad9 = "N9"
    },
    Theme = {
        Inline = Color3.fromRGB(52, 52, 52),
        Background = Color3.fromRGB(36, 36, 36),
        ["Page Background"] = Color3.fromRGB(22, 22, 22),
        ["Section Background"] = Color3.fromRGB(30, 30, 30),
        ["Dark Background"] = Color3.fromRGB(19, 19, 19),
        Accent = Color3.fromRGB(0, 134, 229),
        ["Dark Text"] = Color3.fromRGB(120, 120, 120),
        ["Light Text"] = Color3.fromRGB(160, 160, 160),
        Text = Color3.fromRGB(220, 220, 220)
    },
    Notifications = {}
}

Library.__index = Library
Library.Utility = {Objects = {}, Connections = {}}
local Utility = Library.Utility

function Utility.New(object, props, theme)
    local Obj = Instance.new(object)
    if props then
        for prop, val in props do
            Obj[prop] = val
        end
    end
    if theme then
        Library.AddObjectTheme(Obj, theme)
    end
    table.insert(Utility.Objects, Obj)
    return Obj
end

function Utility.Signal(connection)
    table.insert(Utility.Connections, connection)
    return connection
end

function Utility.GetTransparency(obj)
    if obj:IsA("Frame") then
        return "BackgroundTransparency"
    elseif obj:IsA("TextLabel") or obj:IsA("TextButton") then
        return {"TextTransparency", "BackgroundTransparency"}
    elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
        return {"BackgroundTransparency", "ImageTransparency"}
    elseif obj:IsA("ScrollingFrame") then
        return {"BackgroundTransparency", "ScrollBarImageTransparency"}
    elseif obj:IsA("TextBox") then
        return {"TextTransparency", "BackgroundTransparency"}
    end
    return nil
end

function Utility.Round(number, float)
    local Mult = 1 / (float or 1)
    return math.floor(number * Mult + 0.5) / Mult
end

function Utility.PositionOver(position, object, addedy)
    addedy = addedy or 0
    local posX, posY = object.AbsolutePosition.X, (object.AbsolutePosition.Y - addedy)
    local size = object.AbsoluteSize
    local sizeX, sizeY = posX + size.X, posY + size.Y + addedy

    if position.X >= posX and position.Y >= posY and position.X <= sizeX and position.Y <= sizeY then
        return true
    end
    return false
end

function Utility.MouseOver(object, input)
    local posX, posY = object.AbsolutePosition.X, object.AbsolutePosition.Y
    local size = object.AbsoluteSize
    local sizeX, sizeY = posX + size.X, posY + size.Y
    local position = input.Position

    if position.X >= posX and position.Y >= posY and position.X <= sizeX and position.Y <= sizeY then
        return true
    end
    return false
end

function Utility.Lerp(a, b, c)
    c = c or 1 / 8
    local offset = math.abs(b - a)
    if (offset < c) then
        return b
    end
    return a + (b - a) * c
end

function Utility.StringToEnum(enumstring)
    local EnumType, EnumValue = enumstring:match("Enum%.([^%.]+)%.(.+)")
    if EnumType and EnumValue then
        return Enum[EnumType][EnumValue]
    end
    return nil
end

function Utility.TextTriggers(text)
    local Triggers = {
        ["{hour}"] = os.date("%H"),
        ["{minute}"] = os.date("%M"),
        ["{second}"] = os.date("%S"),
        ["{ap}"] = os.date("%p"),
        ["{month}"] = os.date("%b"),
        ["{day}"] = os.date("%d"),
        ["{year}"] = os.date("%Y"),
        ["{fps}"] = Library.Fps,
        ["{ping}"] = math.floor(Stats.PerformanceStats.Ping:GetValue() or 0),
        ["{time}"] = os.date("%H:%M:%S"),
        ["{date}"] = os.date("%b. %d, %Y"),
        ["{game}"] = Game and Game.Name or "Universal",
        ["{n}"] = "\n"
    }

    for i, v in Triggers do
        text = string.gsub(text, i, v)
    end
    return text
end

Library.ScreenGui = Utility.New("ScreenGui", {
    Name = "\0",
    DisplayOrder = 1,
    Parent = HiddenUI
})

function Library.Tween(obj, props, tweeninfo)
    tweeninfo = tweeninfo or TweenInfo.new(Library.TweenSpeed, Library.TweenStyle)
    local Tween = TweenService:Create(obj, tweeninfo, props)
    Tween:Play()
    return Tween
end

function Library.Fade(obj, prop, vis)
    if not (obj.Visible and prop) then
        return
    end

    local OldTransparency = obj[prop]
    obj[prop] = vis and 1 or OldTransparency

    local Tween = Library.Tween(obj, {[prop] = vis and OldTransparency or 1})

    Utility.Signal(Tween.Completed:Connect(function()
        if not vis then
            task.wait()
            obj[prop] = OldTransparency
        end
    end))

    return Tween
end

function Library.AddObjectTheme(object, props)
    local Theme = {
        Props = props,
        Object = object
    }

    for prop, v in props do
        if type(v) == "string" then
            object[prop] = Library.Theme[v]
        elseif type(v) == "function" then
            object[prop] = v()
        end
    end

    Library.ThemeObjects[object] = Theme
end

function Library.ChangeObjectTheme(object, props, tweened)
    local Theme = Library.ThemeObjects[object]

    if Theme then
        Theme.Props = props

        for prop, v in props do
            if type(v) == "string" then
                if tweened then
                    Theme.Tween = Library.Tween(object, {
                        [prop] = Library.Theme[v]
                    })
                else
                    object[prop] = Library.Theme[v]
                end
            elseif type(v) == "function" then
                object[prop] = v()
            end
        end
    end
end

function Library.UpdateTheme(theme, color)
    if not Library.Theme[theme] then
        return
    end

    Library.Theme[theme] = color

    for _, themeobj in Library.ThemeObjects do
        for prop, val in themeobj.Props do
            if val == theme then
                if themeobj.Tween then
                    themeobj.Tween:Cancel()
                end
                themeobj.Object[prop] = color
            end
        end
    end
end

function Library.Config(cfg, default)
    local Table = {}

    for name, val in cfg do
        Table[name:lower()] = val
    end

    for name, val in default do
        if Table[name] == nil then
            Table[name] = val
        end
    end

    return Table
end

function Library.Resize(holder, box)
    local Start, StartSize, Resizing
    local CurrentSize = holder.Size
    local OriginalSize = holder.Size

    Utility.Signal(box.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Resizing = true
            Start = input.Position
            StartSize = holder.Size
        end
    end))

    Utility.Signal(UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and Resizing then
            local ViewportSize = Camera.ViewportSize
            CurrentSize = UDim2.new(0, math.clamp(StartSize.X.Offset + (input.Position.X - Start.X), OriginalSize.X.Offset, ViewportSize.x), 0, math.clamp(StartSize.Y.Offset + (input.Position.Y - Start.Y), OriginalSize.Y.Offset, ViewportSize.y))
            holder.Size = CurrentSize
        end
    end))

    Utility.Signal(UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Resizing = false
        end
    end))
end

function Library.Dragging(holder, box)
    local Start, StartPos, Dragging
    local CurrentPos
    local Inset = GuiService:GetGuiInset()

    Utility.Signal(box.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Dragging = true
            Start = input.Position
            StartPos = holder.AbsolutePosition
        end
    end))

    Utility.Signal(UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and Dragging then
            local MaxSize = holder.AbsoluteSize
            local ViewportSize = Camera.ViewportSize
            CurrentPos = UDim2.new(0, math.clamp(StartPos.X + (input.Position.X - Start.X), 0, ViewportSize.x - MaxSize.x), 0, math.clamp(StartPos.Y + (input.Position.Y - Start.Y + 36), 0, ViewportSize.y - MaxSize.y) + (Inset.Y / 2))
            holder.Position = CurrentPos
        end
    end))

    Utility.Signal(UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Dragging = false
        end
    end))
end
function Library.ColorpickerWindow(self)
    local Popup = {
        Visible = false,
        Tweening = false,
        ZIndex = self.ZIndex,
        Objects = {},
        Flag = nil,
        SetFunc = function() end,
        Alpha = 1,
        Color = Color3.new(1, 1, 1),
        HuePos = nil
    }

    local ZIndex = Popup.ZIndex
    local Objects = Popup.Objects

    Objects.Outline = Utility.New("Frame", {
        Name = "Outline",
        Size = UDim2.new(0, 150, 0, 150),
        BorderSizePixel = 0,
        Parent = Library.ScreenGui,
        ZIndex = ZIndex,
        ClipsDescendants = true,
        Visible = false
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.Outline, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.Background = Utility.New("TextButton", {
        Name = "Background",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.fromOffset(1, 1),
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        BorderSizePixel = 0,
        Parent = Objects.Outline,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Background"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.Background, CornerRadius = UDim.new(0, 5)})
    Utility.New("UIPadding", {Name = "UIPadding", Parent = Objects.Background, PaddingLeft = UDim.new(0, 5), PaddingRight = UDim.new(0, 5), PaddingTop = UDim.new(0, 5), PaddingBottom = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.SaturationOutline = Utility.New("TextButton", {
        Name = "SaturationOutline",
        Size = UDim2.new(1, -14, 1, -14),
        BorderSizePixel = 0,
        Parent = Objects.Background,
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SaturationOutline, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.SaturationBackground = Utility.New("ImageLabel", {
        Name = "SaturationBackground",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundColor3 = Color3.fromRGB(0, 0, 255),
        BorderSizePixel = 0,
        Parent = Objects.SaturationOutline,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SaturationBackground, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.SaturationImage = Utility.New("ImageLabel", {
        Name = "SaturationImage",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BorderSizePixel = 0,
        Parent = Objects.SaturationOutline,
        BackgroundTransparency = 1,
        Image = Library.Saturation,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SaturationImage, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.SaturationPickerOutline = Utility.New("TextButton", {
        Name = "SaturationPickerOutline",
        Size = UDim2.new(0, 6, 0, 6),
        Position = UDim2.new(0, 0, 0, 0),
        BorderSizePixel = 0,
        Parent = Objects.SaturationImage,
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SaturationPickerOutline, CornerRadius = UDim.new(1, 0)})
    ZIndex = ZIndex + 1

    Objects.SaturationPicker = Utility.New("Frame", {
        Name = "SaturationPicker",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BorderSizePixel = 0,
        Parent = Objects.SaturationPickerOutline,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SaturationPicker, CornerRadius = UDim.new(1, 0)})

    Objects.HueOutline = Utility.New("TextButton", {
        Name = "HueOutline",
        Size = UDim2.new(0, 12, 1, 0),
        Position = UDim2.new(1, -12, 0, 0),
        BorderSizePixel = 0,
        Parent = Objects.Background,
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.HueOutline, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.HueBackground = Utility.New("Frame", {
        Name = "HueBackground",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        BorderSizePixel = 0,
        Parent = Objects.HueOutline,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.HueBackground, CornerRadius = UDim.new(0, 5)})

    Utility.New("UIGradient", {
        Color = ColorSequence.new{ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 0, 0)), ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 0, 255)), ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 0, 255)), ColorSequenceKeypoint.new(0.50, Color3.fromRGB(0, 255, 255)), ColorSequenceKeypoint.new(0.67, Color3.fromRGB(0, 255, 0)), ColorSequenceKeypoint.new(0.83, Color3.fromRGB(255, 255, 0)), ColorSequenceKeypoint.new(1.00, Color3.fromRGB(255, 0, 0))},
        Rotation = 90,
        Parent = Objects.HueBackground
    })

    ZIndex = ZIndex + 1

    Objects.HuePickerOutline = Utility.New("TextButton", {
        Name = "HuePickerOutline",
        Size = UDim2.new(0, 10, 0, 10),
        Position = UDim2.new(0, 0, 0, 0),
        BorderSizePixel = 0,
        Parent = Objects.HueBackground,
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.HuePickerOutline, CornerRadius = UDim.new(1, 0)})
    ZIndex = ZIndex + 1

    Objects.HuePicker = Utility.New("Frame", {
        Name = "SaturationPicker",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BorderSizePixel = 0,
        Parent = Objects.HuePickerOutline,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.HuePicker, CornerRadius = UDim.new(1, 0)})

    Objects.AlphaOutline = Utility.New("TextButton", {
        Name = "AlphaOutline",
        Size = UDim2.new(1, -14, 0, 12),
        Position = UDim2.new(0, 0, 1, -12),
        BorderSizePixel = 0,
        Parent = Objects.Background,
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.AlphaOutline, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.AlphaImage = Utility.New("ImageLabel", {
        Name = "AlphaImage",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BackgroundColor3 = Color3.fromRGB(0, 0, 255),
        BorderSizePixel = 0,
        Parent = Objects.AlphaOutline,
        Image = Library.Checkers,
        ScaleType = Enum.ScaleType.Tile,
        TileSize = UDim2.new(0, 6, 0, 6),
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.AlphaImage, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.AlphaBackground = Utility.New("Frame", {
        Name = "SaturationPicker",
        Size = UDim2.new(1, 0, 1, 0),
        BorderSizePixel = 0,
        Parent = Objects.AlphaImage,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.AlphaBackground, CornerRadius = UDim.new(0, 5)})

    Utility.New("UIGradient", {
        Parent = Objects.AlphaBackground,
        Rotation = 180,
        Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(1, 0)})
    })

    ZIndex = ZIndex + 1

    Objects.AlphaPickerOutline = Utility.New("TextButton", {
        Name = "AlphaPickerOutline",
        Size = UDim2.new(0, 10, 0, 10),
        Position = UDim2.new(0, 0, 0, 0),
        BorderSizePixel = 0,
        Parent = Objects.AlphaBackground,
        Text = "",
        AutoButtonColor = false,
        Style = Enum.ButtonStyle.Custom,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.AlphaPickerOutline, CornerRadius = UDim.new(1, 0)})
    ZIndex = ZIndex + 1

    Objects.AlphaPicker = Utility.New("Frame", {
        Name = "AlphaPicker",
        Size = UDim2.new(1, -2, 1, -2),
        Position = UDim2.new(0, 1, 0, 1),
        BorderSizePixel = 0,
        Parent = Objects.AlphaPickerOutline,
        ZIndex = ZIndex
    })

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.AlphaPicker, CornerRadius = UDim.new(1, 0)})

    local Hue, Sat, Val

    function Popup.Set(color, alpha, ignore)
        alpha = alpha or Popup.Alpha

        Hue, Sat, Val = color:ToHSV()

        Popup.Color = color
        Popup.Alpha = alpha

        if not ignore then
            Objects.SaturationPickerOutline.Position = UDim2.new(
                math.clamp(Sat, 0, 1),
                Sat < 1 and 0 or -6,
                math.clamp(1 - Val, 0, 1),
                1 - Val < 1 and 0 or -6
            )

            Popup.HuePos = Hue

            Objects.HuePickerOutline.Position = UDim2.new(
                0,
                0,
                math.clamp(1 - Hue, 0, 1),
                1 - Hue < 1 and 0 or -10
            )

            Objects.AlphaPickerOutline.Position = UDim2.new(
                math.clamp(alpha, 0, 1),
                alpha < 1 and 0 or -10,
                0,
                0
            )
        end

        Popup.SetFunc(color, alpha)

        Objects.HuePicker.BackgroundColor3 = Color3.fromHSV(Popup.HuePos, 1, 1)
        Objects.SaturationPicker.BackgroundColor3 = color
        Objects.AlphaPicker.BackgroundColor3 = color

        Objects.AlphaBackground.BackgroundColor3 = color
        Objects.SaturationBackground.BackgroundColor3 = Color3.fromHSV(Popup.HuePos, 1, 1)
    end

    function Popup.Open(position)
        if Popup.Tweening then
            return
        end

        Popup.Tweening = true
        Popup.Visible = not Popup.Visible

        if Popup.Visible then
            Objects.Outline.Visible = true
        end

        local ParentObjects = Objects.Outline:GetDescendants()
        table.insert(ParentObjects, Objects.Outline)

        for _, obj in ParentObjects do
            local Index = Utility.GetTransparency(obj)
            if not Index then continue end

            if type(Index) == "table" then
                for _, prop in Index do
                    Library.Fade(obj, prop, Popup.Visible)
                end
            else
                Library.Fade(obj, Index, Popup.Visible)
            end
        end

        if position then
            Objects.Outline.Position = UDim2.new(0, position.X, 0, position.Y)
        end

        local Size = Objects.Outline.AbsoluteSize

        Objects.Outline.Size = Popup.Visible and UDim2.new(0, Size.X, 0, 20) or UDim2.new(0, Size.X, 0, Size.Y)

        local Tween = Library.Tween(Objects.Outline, {
            Size = Popup.Visible and UDim2.new(0, Size.X, 0, Size.Y) or UDim2.new(0, Size.X, 0, 20)
        })

        Utility.Signal(Tween.Completed:Connect(function()
            Objects.Outline.Visible = Popup.Visible
            Objects.Outline.Size = UDim2.new(0, Size.X, 0, Size.Y)
            Popup.Tweening = false
        end))
    end

    function Popup.SlideSaturation(input)
        local SizeX = math.clamp((input.Position.X - Objects.SaturationOutline.AbsolutePosition.X) / Objects.SaturationOutline.AbsoluteSize.X, 0, 1)
        local SizeY = 1 - math.clamp((input.Position.Y - Objects.SaturationOutline.AbsolutePosition.Y) / Objects.SaturationOutline.AbsoluteSize.Y, 0, 1)

        Objects.SaturationPickerOutline.Position = UDim2.new(SizeX, 0, 1 - SizeY, 0)
        Objects.SaturationPickerOutline.AnchorPoint = Vector2.new(SizeX, 1 - SizeY)

        Popup.Set(Color3.fromHSV(Popup.HuePos, SizeX, SizeY), Popup.Alpha, true)
    end

    Utility.Signal(Objects.SaturationOutline.MouseButton1Down:Connect(function()
        Popup.SlidingSaturation = true
        Popup.SlideSaturation({Position = UserInputService:GetMouseLocation() - Vector2.new(0, 62)})
    end))

    function Popup.SlideHue(input)
        local SizeY = 1 - math.clamp((input.Position.Y - Objects.HueOutline.AbsolutePosition.Y) / Objects.HueOutline.AbsoluteSize.Y, 0, 1)

        Objects.HuePickerOutline.Position = UDim2.new(0, 0, 1 - SizeY, 0)
        Objects.HuePickerOutline.AnchorPoint = Vector2.new(0, 1 - SizeY)
        Popup.HuePos = SizeY

        Popup.Set(Color3.fromHSV(SizeY, Sat, Val), Popup.Alpha, true)
    end

    Utility.Signal(Objects.HueOutline.MouseButton1Down:Connect(function()
        Popup.SlidingHue = true
        Popup.SlideHue({Position = UserInputService:GetMouseLocation() - Vector2.new(0, 62)})
    end))

    function Popup.SlideAlpha(input)
        local SizeX = math.clamp((input.Position.X - Objects.AlphaOutline.AbsolutePosition.X) / Objects.AlphaOutline.AbsoluteSize.X, 0, 1)

        Objects.AlphaPickerOutline.Position = UDim2.new(SizeX, 0, 0, 0)
        Objects.AlphaPickerOutline.AnchorPoint = Vector2.new(SizeX, 0)
        Popup.Set(Popup.Color, SizeX, true)
    end

    Utility.Signal(Objects.AlphaOutline.MouseButton1Down:Connect(function()
        Popup.SlidingAlpha = true
        Popup.SlideAlpha({Position = UserInputService:GetMouseLocation()})
    end))

    Utility.Signal(UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if Popup.SlidingSaturation then
                Popup.SlideSaturation({Position = UserInputService:GetMouseLocation() - Vector2.new(0, 62)})
            end
            if Popup.SlidingHue then
                Popup.SlideHue({Position = UserInputService:GetMouseLocation() - Vector2.new(0, 62)})
            end
            if Popup.SlidingAlpha then
                Popup.SlideAlpha({Position = UserInputService:GetMouseLocation()})
            end
        end
    end))

    Utility.Signal(UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Popup.SlidingSaturation, Popup.SlidingHue, Popup.SlidingAlpha = false, false, false
        end
    end))

    return Popup
end

Library.ColorpickerWindow = Library.ColorpickerWindow({ZIndex = 5000})

function Library.Window(self, cfg)
    cfg = cfg or {}
    cfg = Library.Config(cfg, {
        name = "User Interface",
        size = UDim2.fromOffset(350, 430),
        open = true
    })

    local Window = {
        Objects = {},
        Visible = cfg.open,
        Tweening = false,
        TabsTweening = false,
        ZIndex = 0,
        Tabs = {}
    }

    local ZIndex = Window.ZIndex
    local Objects = Window.Objects

    Objects.ScreenGui = Utility.New("ScreenGui", {
        Name = "\0",
        Parent = HiddenUI,
        IgnoreGuiInset = true
    })

    Objects.Outline = Utility.New("Frame", {
        Name = "Outline",
        Size = cfg.size,
        Position = UDim2.new(0.5, -cfg.size.X.Offset / 2, 0.5, -cfg.size.Y.Offset / 2),
        BorderSizePixel = 0,
        Parent = Objects.ScreenGui,
        ZIndex = ZIndex,
        Visible = Window.Visible,
        ClipsDescendants = true
    }, {BackgroundColor3 = "Page Background"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.Outline, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.SideInline = Utility.New("Frame", {
        Name = "SideInline",
        Size = UDim2.new(0, 160, 1, 0),
        Position = UDim2.fromOffset(0, 0),
        BorderSizePixel = 0,
        Parent = Objects.Outline,
        ZIndex = ZIndex
    }, {BackgroundColor3 = "Inline"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SideInline, CornerRadius = UDim.new(0, 5)})
    ZIndex = ZIndex + 1

    Objects.SideBackground = Utility.New("Frame", {
        Name = "SideBackground",
        Size = UDim2.new(1, -1, 1, 0),
        Position = UDim2.fromOffset(0, 0),
        BorderSizePixel = 0,
        Parent = Objects.SideInline,
        ZIndex = ZIndex,
        ClipsDescendants = true
    }, {BackgroundColor3 = "Dark Background"})

    Utility.New("UICorner", {Name = "UICorner", Parent = Objects.SideBackground, CornerRadius = UDim.new(0, 5)})

    Library.Dragging(Objects.Outline, Objects.SideBackground)

    Utility.New("UIPadding", {
        Name = "UIPadding",
        PaddingLeft = UDim.new(0, 5),
        PaddingRight = UDim.new(0, 5),
        PaddingTop = UDim.new(0, 8),
        PaddingBottom = UDim.new(0, 5),
        Parent = Objects.SideBackground
    })

    ZIndex = ZIndex + 1

    Objects.Title = Utility.New("TextLabel", {
        Name = "Title",
        TextStrokeTransparency = 0.8,
        BackgroundTransparency = 1,
        TextSize = Library.FontSize + 6,
        TextYAlignment = Enum.TextYAlignment.Center,
        FontFace = Library.Font,
        Size = UDim2.new(1, 0, 0, 0),
        Position = UDim2.new(0, 0, 0, 0),
        Text = cfg.name,
        Parent = Objects.SideBackground,
        ZIndex = ZIndex
    }, {TextColor3 = "Text"})

    Objects.Title.Size = UDim2.new(1, 0, 0, Objects.Title.TextBounds.Y + 22)

    Objects.SideScroll = Utility.New("ScrollingFrame", {
        Name = "SideScroll",
        Size = UDim2.new(1, 0, 1, -Objects.Title.AbsoluteSize.Y),
        Position = UDim2.fromOffset(0, Objects.Title.AbsoluteSize.Y),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Parent = Objects.SideBackground,
        ZIndex = ZIndex,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        BottomImage = Library.ScrollBar,
        MidImage = Library.ScrollBar,
        TopImage = Library.ScrollBar,
        ScrollBarThickness = 2
    }, {ScrollBarImageColor3 = "Accent"})

    Objects.SideContent = Utility.New("Frame", {
        Name = "SideContent",
        Size = UDim2.new(1, 0, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        Position = UDim2.fromOffset(0, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Parent = Objects.SideScroll,
        ZIndex = ZIndex
    })

    Utility.Signal(Objects.SideScroll:GetPropertyChangedSignal("AbsoluteCanvasSize"):Connect(function()
        if Objects.SideScroll.AbsoluteCanvasSize.Y > Objects.SideScroll.AbsoluteSize.Y then
            Objects.SideContent.Size = UDim2.new(1, -7, 0, 0)
        else
            Objects.SideContent.Size = UDim2.new(1, 0, 0, 0)
        end
    end))

    Utility.Signal(Objects.SideScroll:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
        if Objects.SideScroll.AbsoluteCanvasSize.Y > Objects.SideScroll.AbsoluteSize.Y then
            Objects.SideContent.Size = UDim2.new(1, -7, 0, 0)
        else
            Objects.SideContent.Size = UDim2.new(1, 0, 0, 0)
        end
    end))

    Utility.New("UIListLayout", {
        Name = "UIListLayout",
        FillDirection = Enum.FillDirection.Vertical,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = Objects.SideContent,
        Padding = UDim.new(0, 6)
    })

    Window.SideHolder = Objects.SideContent

    Objects.ResizeBar = Utility.New("Frame", {
        Name = "ResizeBar",
        BorderSizePixel = 0,
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 5, 0, 5),
        Position = UDim2.new(1, -5, 1, -5),
        Parent = Objects.Outline,
        ZIndex = ZIndex
    })

    Library.Resize(Objects.Outline, Objects.ResizeBar)

    Objects.PageHolder = Utility.New("Frame", {
        Name = "PageHolder",
        Size = UDim2.new(1, -Objects.SideInline.AbsoluteSize.X, 1, 0),
        Position = UDim2.new(0, Objects.SideInline.AbsoluteSize.X, 0, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Parent = Objects.Outline,
        ZIndex = ZIndex
    })

    Utility.New("UIPadding", {
        Name = "UIPadding",
        PaddingLeft = UDim.new(0, 5),
        PaddingRight = UDim.new(0, 5),
        PaddingTop = UDim.new(0, 5),
        PaddingBottom = UDim.new(0, 5),
        Parent = Objects.PageHolder
    })

    Window.PageHolder = Objects.PageHolder
    ZIndex = ZIndex + 1
    Window.ZIndex = ZIndex

    local AnimationSizeAmount = 20

    function Window.Open()
        if Window.Tweening then
            return
        end

        for _, popup in Library.Popups do
            if popup.Objects.Outline.Visible then
                popup.Visible(false)
            end
        end

        for _, dropdown in Library.Dropdowns do
            if dropdown.Visible then
                dropdown.Open()
            end
        end

        if Library.ColorpickerWindow.Visible then
            Library.ColorpickerWindow.Open()
        end

        Window.Tweening = true
        Window.Visible = not Window.Visible

        if Window.Visible then
            Objects.Outline.Visible = true
        end

        for _, obj in Objects.ScreenGui:GetDescendants() do
            local Index = Utility.GetTransparency(obj)

            if not Index then continue end

            if type(Index) == "table" then
                for _, prop in Index do
                    Library.Fade(obj, prop, Window.Visible)
                end
            else
                Library.Fade(obj, Index, Window.Visible)
            end
        end

        local OldSize = Objects.Outline.AbsoluteSize

        Objects.Outline.Size = Window.Visible and UDim2.fromOffset(OldSize.X - AnimationSizeAmount, OldSize.Y - AnimationSizeAmount) or UDim2.fromOffset(OldSize.X, OldSize.Y)

        local Tween = Library.Tween(Objects.Outline, {Size = Window.Visible and UDim2.fromOffset(OldSize.x, OldSize.y) or UDim2.fromOffset(OldSize.x - AnimationSizeAmount, OldSize.y - AnimationSizeAmount)})

        Utility.Signal(
Input.UserInputType == Enum.UserInputType.Touch then
				if Fatality:IsMouseOverFrame(ColorPickBox) then
					IsPressM1 = true;
					Fatality.GLOBAL_ENVIRONMENT.IS_HOLD_COLOR_PICKER = true;

					if SPAWN_THREAD then
						task.cancel(SPAWN_THREAD);
						SPAWN_THREAD = nil;
					end;

					SPAWN_THREAD = task.spawn(function()
						while IsPressM1 do task.wait()
							local MousePos = Fatality:GetCalculatePosition(ColorPickBox.AbsolutePosition, Vector3.new(0,0,1), CurrentCamera.CFrame.Position, Mouse.UnitRay.Direction)
							local RelativePos = Vector2.new(MousePos.X,MousePos.Y) - ColorPickBox.AbsolutePosition;
							local clampx = math.clamp(RelativePos.X,0,ColorPickBox.AbsoluteSize.X);
							local clampy = math.clamp(RelativePos.Y,0,ColorPickBox.AbsoluteSize.Y);

							CodeH = clampx / ColorPickBox.AbsoluteSize.X;
							CodeV = 1 - (clampy / ColorPickBox.AbsoluteSize.Y);

							local ud = UDim2.new(CodeH, 0, 1 - CodeV, 0);

							Fatality:CreateAnimation(MouseMovement,0.05,{
								Position = ud
							});

							Callback(Color3.fromHSV(OldCode,CodeH,CodeV),Transparency);
						end;
					end);
				elseif Fatality:IsMouseOverFrame(ColorRedGreenBlue) then
					IsPressM1 = true;
					Fatality.GLOBAL_ENVIRONMENT.IS_HOLD_COLOR_PICKER = true;

					if SPAWN_THREAD then
						task.cancel(SPAWN_THREAD);
						SPAWN_THREAD = nil;
					end;

					SPAWN_THREAD = task.spawn(function()
						while IsPressM1 do task.wait()
							local MousePos = Fatality:GetCalculatePosition(ColorRedGreenBlue.AbsolutePosition, Vector3.new(0,0,1), CurrentCamera.CFrame.Position, Mouse.UnitRay.Direction)
							local RelativePos = Vector2.new(MousePos.X,MousePos.Y) - ColorRedGreenBlue.AbsolutePosition;
							local clampy = math.clamp(RelativePos.Y,0,ColorRedGreenBlue.AbsoluteSize.Y);

							OldCode = clampy / ColorRedGreenBlue.AbsoluteSize.Y;

							local ud = UDim2.new(0.5, 0, OldCode, 0);

							ColorPickBox.BackgroundColor3 = Color3.fromHSV(OldCode,1,1);

							Fatality:CreateAnimation(ColorRGBSlide,0.05,{
								Position = ud
							});

							HexCodeText.Text = "#" .. tostring(Color3.fromHSV(OldCode,CodeH,CodeV):ToHex());

							Callback(Color3.fromHSV(OldCode,CodeH,CodeV),Transparency);
						end;
					end);
				elseif Fatality:IsMouseOverFrame(ColorOpc) then
					IsPressM1 = true;
					Fatality.GLOBAL_ENVIRONMENT.IS_HOLD_COLOR_PICKER = true;

					if SPAWN_THREAD then
						task.cancel(SPAWN_THREAD);
						SPAWN_THREAD = nil;
					end;

					SPAWN_THREAD = task.spawn(function()
						while IsPressM1 do task.wait()
							local MousePos = Fatality:GetCalculatePosition(ColorOpc.AbsolutePosition, Vector3.new(0,0,1), CurrentCamera.CFrame.Position, Mouse.UnitRay.Direction)
							local RelativePos = Vector2.new(MousePos.X,MousePos.Y) - ColorOpc.AbsolutePosition;
							local clampx = math.clamp(RelativePos.X,0,ColorOpc.AbsoluteSize.X);

							Transparency = 1 - (clampx / ColorOpc.AbsoluteSize.X);

							local ud = UDim2.new(1- Transparency, 0, 0.5, 0);

							ColorOpc.BackgroundColor3 = Color3.fromHSV(OldCode,CodeH,CodeV);

							Fatality:CreateAnimation(ColorOptSlide,0.05,{
								Position = ud
							});

							Callback(Color3.fromHSV(OldCode,CodeH,CodeV),Transparency);
						end;
					end);
				end;
			end;
		end);

		ColorPickerFrame.InputEnded:Connect(function(Input)
			if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
				IsPressM1 = false;
				Fatality.GLOBAL_ENVIRONMENT.IS_HOLD_COLOR_PICKER = false;
			end;
		end);
	end;

	UserInputService.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			if not Fatality:IsMouseOverFrame(ColorPickerFrame) and not Fatality:IsMouseOverFrame(ColorBox) then
				VisibleToggle(false);
			end;
		end;
	end);

	return Fatality:CreateResponse({
		SetColor = function(Color,Transp)
			local h,s,v = Color:ToHSV();

			Transparency = Transp or 0;

			OldCode = h;
			CodeH = s;
			CodeV = v;

			ColorPickBox.BackgroundColor3 = Color3.fromHSV(h,1,1);
			ColorOpc.BackgroundColor3 = Color;
			ColorBox.BackgroundColor3 = Color;

			HexCodeText.Text = "#" .. tostring(Color:ToHex());

			Fatality:CreateAnimation(ColorRGBSlide,0.35,{
				Position = UDim2.new(0.5, 0, OldCode, 0)
			});

			Fatality:CreateAnimation(MouseMovement,0.35,{
				Position = UDim2.new(CodeH, 0, 1 - CodeV, 0)
			})

			Fatality:CreateAnimation(ColorOptSlide,0.35,{
				Position = UDim2.new(1- Transparency, 0, 0.5, 0)
			})
		end
	});
end;

function Fatality:CreateElements(ScrollingFrame: ScrollingFrame, ZIndex: number, Bindable: BindableEvent): Elements
	return Fatality:CreateResponse({
		AddToggle = function(self,cfg: Toggle)
			cfg = cfg or {};

			local ToggleFrame = Instance.new("Frame")
			local ToggleText = Instance.new("TextLabel")
			local ToggleButton = Instance.new("TextButton")
			local ToggleAnim = Instance.new("Frame")
			local OptionButton = Instance.new("ImageButton")
			local Main_Box = Instance.new('Frame');

			local IsActivate = cfg.Default or false;

			if cfg.Flag then
				Fatality.Flags[cfg.Flag] = {
					Set = function(v)
						IsActivate = v;

						if v then
							Fatality:CreateAnimation(ToggleAnim,0.35,{
								Size = UDim2.new(1,0,1,0),
								BackgroundColor3 = Fatality.Colors.Main
							});
						else
							Fatality:CreateAnimation(ToggleAnim,0.35,{
								Size = UDim2.new(0.5,0,1,0),
								BackgroundColor3 = Fatality.Colors.Black
							});
						end;

						cfg.Callback(IsActivate);
					end,
					Get = function()
						return IsActivate;
					end
				};
			end;

			Main_Box.Parent = ScrollingFrame;
			Main_Box.BackgroundTransparency = 1;
			Main_Box.Size = UDim2.new(1,0,0,32);
			Main_Box.Name = Fatality:RandomString();

			ToggleFrame.Name = Fatality:RandomString()
			ToggleFrame.Parent = Main_Box
			ToggleFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ToggleFrame.BackgroundTransparency = 1.000
			ToggleFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ToggleFrame.BorderSizePixel = 0
			ToggleFrame.Position = UDim2.new(0, 0, 0, 0)
			ToggleFrame.Size = UDim2.new(1, 0, 1, 0)
			ToggleFrame.ZIndex = ZIndex + 1

			ToggleText.Name = Fatality:RandomString()
			ToggleText.Parent = ToggleFrame
			ToggleText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ToggleText.BackgroundTransparency = 1.000
			ToggleText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ToggleText.BorderSizePixel = 0
			ToggleText.Position = UDim2.new(0, 0, 0, 5)
			ToggleText.Size = UDim2.new(1, cfg.Option and -60 or -35, 1, -10)
			ToggleText.ZIndex = ZIndex + 2
			ToggleText.FontFace = Fatality.FontSemiBold
			ToggleText.Text = cfg.Name or "Toggle Button"
			ToggleText.TextColor3 = Color3.fromRGB(255, 255, 255)
			ToggleText.TextSize = 13.000
			ToggleText.TextTransparency = 0.350
			ToggleText.TextWrapped = true
			ToggleText.TextXAlignment = Enum.TextXAlignment.Left
			ToggleText.TextYAlignment = Enum.TextYAlignment.Top

			ToggleButton.Name = Fatality:RandomString()
			ToggleButton.Parent = ToggleFrame
			ToggleButton.AnchorPoint = Vector2.new(1, 0)
			ToggleButton.BackgroundColor3 = Fatality.Colors.Black
			ToggleButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ToggleButton.BorderSizePixel = 0
			ToggleButton.Position = UDim2.new(1, cfg.Option and -25 or 0, 0, 6)
			ToggleButton.Size = UDim2.new(0, 28, 1, -12)
			ToggleButton.ZIndex = ZIndex + 2
			ToggleButton.Font = Enum.Font.SourceSans
			ToggleButton.Text = ""
			ToggleButton.TextColor3 = Color3.fromRGB(0, 0, 0)
			ToggleButton.TextSize = 14.000

			ToggleAnim.Name = Fatality:RandomString()
			ToggleAnim.Parent = ToggleButton
			ToggleAnim.AnchorPoint = Vector2.new(1, 0)
			ToggleAnim.BackgroundColor3 = IsActivate and Fatality.Colors.Main or Fatality.Colors.Black
			ToggleAnim.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ToggleAnim.BorderSizePixel = 0
			ToggleAnim.Position = UDim2.new(1, 0, 0, 0)
			ToggleAnim.Size = IsActivate and UDim2.new(1, 0, 1, 0) or UDim2.new(0.5, 0, 1, 0)
			ToggleAnim.ZIndex = ZIndex + 2

			ToggleButton.MouseButton1Click:Connect(function()
				IsActivate = not IsActivate;

				if IsActivate then
					Fatality:CreateAnimation(ToggleAnim,0.35,{
						Size = UDim2.new(1,0,1,0),
						BackgroundColor3 = Fatality.Colors.Main
					});
				else
					Fatality:CreateAnimation(ToggleAnim,0.35,{
						Size = UDim2.new(0.5,0,1,0),
						BackgroundColor3 = Fatality.Colors.Black
					});
				end;

				cfg.Callback(IsActivate);

				if cfg.Flag then
					Fatality.Flags[cfg.Flag].Set(IsActivate);
				end;
			end);

			if cfg.Option then
				OptionButton.Name = Fatality:RandomString()
				OptionButton.Parent = ToggleFrame
				OptionButton.AnchorPoint = Vector2.new(1, 0)
				OptionButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
				OptionButton.BackgroundTransparency = 1.000
				OptionButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
				OptionButton.BorderSizePixel = 0
				OptionButton.LayoutOrder = 5
				OptionButton.Position = UDim2.new(1, 0, 0, 6)
				OptionButton.Size = UDim2.new(0, 20, 0, 20)
				OptionButton.ZIndex = ZIndex + 5
				OptionButton.Image = "rbxassetid://10734950020"
				OptionButton.ImageTransparency = 0.600

				return {
					Option = Fatality:CreateOption(OptionButton)
				};
			end;

			return {};
		end,

		AddSlider = function(self,cfg: Slider)
			cfg = cfg or {};

			local SliderFrame = Instance.new("Frame")
			local SliderText = Instance.new("TextLabel")
			local SliderButton = Instance.new("TextButton")
			local SliderBox = Instance.new("Frame")
			local UICorner = Instance.new("UICorner")
			local SliderBtn = Instance.new("Frame")
			local UICorner_2 = Instance.new("UICorner")
			local SliderValue = Instance.new("TextLabel")
			local OptionButton = Instance.new("ImageButton")
			local Main_Box = Instance.new('Frame');

			local IsDragging = false;
			local Value = cfg.Default or cfg.Min or 0;

			if cfg.Flag then
				Fatality.Flags[cfg.Flag] = {
					Set = function(v)
						Value = v;

						local percent = (Value - cfg.Min) / (cfg.Max - cfg.Min);

						Fatality:CreateAnimation(SliderBtn,0.35,{
							Size = UDim2.new(percent,0,1,0)
						});

						SliderValue.Text = tostring(Fatality:Rounding(Value,cfg.Round or 0)) .. (cfg.Type or "");

						cfg.Callback(Value);
					end,
					Get = function()
						return Value;
					end
				};
			end;

			Main_Box.Parent = ScrollingFrame;
			Main_Box.BackgroundTransparency = 1;
			Main_Box.Size = UDim2.new(1,0,0,42);
			Main_Box.Name = Fatality:RandomString();

			SliderFrame.Name = Fatality:RandomString()
			SliderFrame.Parent = Main_Box
			SliderFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			SliderFrame.BackgroundTransparency = 1.000
			SliderFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			SliderFrame.BorderSizePixel = 0
			SliderFrame.Size = UDim2.new(1, 0, 1, 0)
			SliderFrame.ZIndex = ZIndex + 1

			SliderText.Name = Fatality:RandomString()
			SliderText.Parent = SliderFrame
			SliderText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			SliderText.BackgroundTransparency = 1.000
			SliderText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			SliderText.BorderSizePixel = 0
			SliderText.Position = UDim2.new(0, 0, 0, 5)
			SliderText.Size = UDim2.new(1, cfg.Option and -25 or 0, 0, 15)
			SliderText.ZIndex = ZIndex + 2
			SliderText.FontFace = Fatality.FontSemiBold
			SliderText.Text = cfg.Name or "Slider"
			SliderText.TextColor3 = Color3.fromRGB(255, 255, 255)
			SliderText.TextSize = 13.000
			SliderText.TextTransparency = 0.350
			SliderText.TextXAlignment = Enum.TextXAlignment.Left
			SliderText.TextYAlignment = Enum.TextYAlignment.Top

			SliderButton.Name = Fatality:RandomString()
			SliderButton.Parent = SliderFrame
			SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			SliderButton.BackgroundTransparency = 1.000
			SliderButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
			SliderButton.BorderSizePixel = 0
			SliderButton.Position = UDim2.new(0, 0, 0, 22)
			SliderButton.Size = UDim2.new(1, cfg.Option and -25 or 0, 1, -27)
			SliderButton.ZIndex = ZIndex + 3
			SliderButton.Font = Enum.Font.SourceSans
			SliderButton.Text = ""
			SliderButton.TextColor3 = Color3.fromRGB(0, 0, 0)
			SliderButton.TextSize = 14.000

			SliderBox.Name = Fatality:RandomString()
			SliderBox.Parent = SliderButton
			SliderBox.BackgroundColor3 = Fatality.Colors.Black
			SliderBox.BorderColor3 = Color3.fromRGB(0, 0, 0)
			SliderBox.BorderSizePixel = 0
			SliderBox.Size = UDim2.new(1, 0, 1, 0)
			SliderBox.ZIndex = ZIndex + 3

			UICorner.Parent = SliderBox

			SliderBtn.Name = Fatality:RandomString()
			SliderBtn.Parent = SliderBox
			SliderBtn.BackgroundColor3 = Fatality.Colors.Main
			SliderBtn.BorderColor3 = Color3.fromRGB(0, 0, 0)
			SliderBtn.BorderSizePixel = 0
			SliderBtn.Size = UDim2.new((Value - cfg.Min) / (cfg.Max - cfg.Min), 0, 1, 0)
			SliderBtn.ZIndex = ZIndex + 4

			UICorner_2.Parent = SliderBtn

			SliderValue.Name = Fatality:RandomString()
			SliderValue.Parent = SliderBox
			SliderValue.AnchorPoint = Vector2.new(0.5, 0)
			SliderValue.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			SliderValue.BackgroundTransparency = 1.000
			SliderValue.BorderColor3 = Color3.fromRGB(0, 0, 0)
			SliderValue.BorderSizePixel = 0
			SliderValue.Position = UDim2.new(0.5, 0, 0, 0)
			SliderValue.Size = UDim2.new(1, 0, 1, 0)
			SliderValue.ZIndex = ZIndex + 5
			SliderValue.FontFace = Fatality.FontSemiBold
			SliderValue.Text = tostring(Fatality:Rounding(Value,cfg.Round or 0)) .. (cfg.Type or "")
			SliderValue.TextColor3 = Color3.fromRGB(255, 255, 255)
			SliderValue.TextSize = 13.000
			SliderValue.TextTransparency = 0.350

			SliderButton.InputBegan:Connect(function(Input)
				if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
					IsDragging = true;

					local SPAWN_THREAD = task.spawn(function()
						while IsDragging do task.wait()
							local MousePos = Fatality:GetCalculatePosition(SliderBox.AbsolutePosition, Vector3.new(0,0,1), CurrentCamera.CFrame.Position, Mouse.UnitRay.Direction)
							local RelativePos = Vector2.new(MousePos.X,MousePos.Y) - SliderBox.AbsolutePosition;
							local clampx = math.clamp(RelativePos.X,0,SliderBox.AbsoluteSize.X);

							local percent = clampx / SliderBox.AbsoluteSize.X;
							Value = cfg.Min + (cfg.Max - cfg.Min) * percent;

							Fatality:CreateAnimation(SliderBtn,0.05,{
								Size = UDim2.new(percent,0,1,0)
							});

							SliderValue.Text = tostring(Fatality:Rounding(Value,cfg.Round or 0)) .. (cfg.Type or "");

							cfg.Callback(Fatality:Rounding(Value,cfg.Round or 0));

							if cfg.Flag then
								Fatality.Flags[cfg.Flag].Set(Fatality:Rounding(Value,cfg.Round or 0));
							end;
						end;
					end);
				end;
			end);

			SliderButton.InputEnded:Connect(function(Input)
				if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
					IsDragging = false;
				end;
			end);

			if cfg.Option then
				OptionButton.Name = Fatality:RandomString()
				OptionButton.Parent = SliderFrame
				OptionButton.AnchorPoint = Vector2.new(1, 0)
				OptionButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
				OptionButton.BackgroundTransparency = 1.000
				OptionButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
				OptionButton.BorderSizePixel = 0
				OptionButton.LayoutOrder = 5
				OptionButton.Position = UDim2.new(1, 0, 0, 6)
				OptionButton.Size = UDim2.new(0, 20, 0, 20)
				OptionButton.ZIndex = ZIndex + 5
				OptionButton.Image = "rbxassetid://10734950020"
				OptionButton.ImageTransparency = 0.600

				return {
					Option = Fatality:CreateOption(OptionButton)
				};
			end;

			return {};
		end,

		AddButton = function(self,cfg: Button)
			cfg = cfg or {};

			local ButtonFrame = Instance.new("Frame")
			local ButtonText = Instance.new("TextLabel")
			local Button = Instance.new("TextButton")
			local Main_Box = Instance.new('Frame');

			Main_Box.Parent = ScrollingFrame;
			Main_Box.BackgroundTransparency = 1;
			Main_Box.Size = UDim2.new(1,0,0,27);
			Main_Box.Name = Fatality:RandomString();

			ButtonFrame.Name = Fatality:RandomString()
			ButtonFrame.Parent = Main_Box
			ButtonFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ButtonFrame.BackgroundTransparency = 1.000
			ButtonFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ButtonFrame.BorderSizePixel = 0
			ButtonFrame.Size = UDim2.new(1, 0, 1, 0)
			ButtonFrame.ZIndex = ZIndex + 1

			ButtonText.Name = Fatality:RandomString()
			ButtonText.Parent = ButtonFrame
			ButtonText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ButtonText.BackgroundTransparency = 1.000
			ButtonText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ButtonText.BorderSizePixel = 0
			ButtonText.Position = UDim2.new(0, 0, 0, 5)
			ButtonText.Size = UDim2.new(1, -25, 1, -10)
			ButtonText.ZIndex = ZIndex + 2
			ButtonText.FontFace = Fatality.FontSemiBold
			ButtonText.Text = cfg.Name or "Button"
			ButtonText.TextColor3 = Color3.fromRGB(255, 255, 255)
			ButtonText.TextSize = 13.000
			ButtonText.TextTransparency = 0.350
			ButtonText.TextXAlignment = Enum.TextXAlignment.Left
			ButtonText.TextYAlignment = Enum.TextYAlignment.Top

			Button.Name = Fatality:RandomString()
			Button.Parent = ButtonFrame
			Button.AnchorPoint = Vector2.new(1, 0)
			Button.BackgroundColor3 = Fatality.Colors.Black
			Button.BorderColor3 = Color3.fromRGB(0, 0, 0)
			Button.BorderSizePixel = 0
			Button.Position = UDim2.new(1, 0, 0, 6)
			Button.Size = UDim2.new(0, 18, 1, -12)
			Button.ZIndex = ZIndex + 2
			Button.Font = Enum.Font.SourceSans
			Button.Text = ">"
			Button.TextColor3 = Color3.fromRGB(255, 255, 255)
			Button.TextSize = 14.000
			Button.TextTransparency = 0.350

			Fatality:CreateHover(Button,function(bool)
				if bool then
					Fatality:CreateAnimation(Button,0.35,{
						BackgroundColor3 = Fatality.Colors.Main
					});
				else
					Fatality:CreateAnimation(Button,0.35,{
						BackgroundColor3 = Fatality.Colors.Black
					});
				end;
			end);

			Button.MouseButton1Click:Connect(function()
				cfg.Callback();
			end);

			return {};
		end,

		AddColorPicker = function(self,cfg: ColorPicker)
			cfg = cfg or {};

			local ColorPickerFrame = Instance.new("Frame")
			local ColorPickerText = Instance.new("TextLabel")
			local ColorBox = Instance.new("Frame")
			local UICorner = Instance.new("UICorner")
			local Main_Box = Instance.new('Frame');

			Main_Box.Parent = ScrollingFrame;
			Main_Box.BackgroundTransparency = 1;
			Main_Box.Size = UDim2.new(1,0,0,27);
			Main_Box.Name = Fatality:RandomString();

			ColorPickerFrame.Name = Fatality:RandomString()
			ColorPickerFrame.Parent = Main_Box
			ColorPickerFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ColorPickerFrame.BackgroundTransparency = 1.000
			ColorPickerFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ColorPickerFrame.BorderSizePixel = 0
			ColorPickerFrame.Size = UDim2.new(1, 0, 1, 0)
			ColorPickerFrame.ZIndex = ZIndex + 1

			ColorPickerText.Name = Fatality:RandomString()
			ColorPickerText.Parent = ColorPickerFrame
			ColorPickerText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ColorPickerText.BackgroundTransparency = 1.000
			ColorPickerText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ColorPickerText.BorderSizePixel = 0
			ColorPickerText.Position = UDim2.new(0, 0, 0, 5)
			ColorPickerText.Size = UDim2.new(1, -25, 1, -10)
			ColorPickerText.ZIndex = ZIndex + 2
			ColorPickerText.FontFace = Fatality.FontSemiBold
			ColorPickerText.Text = cfg.Name or "Color Picker"
			ColorPickerText.TextColor3 = Color3.fromRGB(255, 255, 255)
			ColorPickerText.TextSize = 13.000
			ColorPickerText.TextTransparency = 0.350
			ColorPickerText.TextXAlignment = Enum.TextXAlignment.Left
			ColorPickerText.TextYAlignment = Enum.TextYAlignment.Top

			ColorBox.Name = Fatality:RandomString()
			ColorBox.Parent = ColorPickerFrame
			ColorBox.AnchorPoint = Vector2.new(1, 0)
			ColorBox.BackgroundColor3 = cfg.Default or Color3.fromRGB(255, 106, 133)
			ColorBox.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ColorBox.BorderSizePixel = 0
			ColorBox.Position = UDim2.new(1, 0, 0, 6)
			ColorBox.Size = UDim2.new(0, 18, 1, -12)
			ColorBox.ZIndex = ZIndex + 2

			UICorner.CornerRadius = UDim.new(0, 2)
			UICorner.Parent = ColorBox

			local ColorPicker = Fatality:CreateColorPicker(ColorBox,cfg.Transparency or 0,function(Color,Transparency)
				ColorBox.BackgroundColor3 = Color;
				cfg.Callback(Color,Transparency);

				if cfg.Flag then
					Fatality.Flags[cfg.Flag] = {
						Set = function(c,t)
							ColorBox.BackgroundColor3 = c;
							ColorPicker.SetColor(c,t);
						end,
						Get = function()
							return Color,Transparency;
						end
					};
				end;
			end);

			if cfg.Flag then
				Fatality.Flags[cfg.Flag] = {
					Set = function(c,t)
						ColorBox.BackgroundColor3 = c;
						ColorPicker.SetColor(c,t);
					end,
					Get = function()
						return ColorBox.BackgroundColor3,cfg.Transparency or 0;
					end
				};
			end;

			return {};
		end,

		AddDropdown = function(self,cfg: Dropdown)
			cfg = cfg or {};

			local DropdownFrame = Instance.new("Frame")
			local DropdownText = Instance.new("TextLabel")
			local DropdownButton = Instance.new("TextButton")
			local UICorner = Instance.new("UICorner")
			local DropdownValueText = Instance.new("TextLabel")
			local DropdownIcon = Instance.new("ImageLabel")
			local OptionButton = Instance.new("ImageButton")
			local Main_Box = Instance.new('Frame');

			local IsOpen = false;
			local SelectedValues = cfg.Multi and (cfg.Default or {}) or (cfg.Default or "");

			if cfg.Flag then
				Fatality.Flags[cfg.Flag] = {
					Set = function(v)
						SelectedValues = v;

						if cfg.Multi then
							DropdownValueText.Text = table.concat(SelectedValues,", ");
						else
							DropdownValueText.Text = SelectedValues;
						end;

						cfg.Callback(SelectedValues);
					end,
					Get = function()
						return SelectedValues;
					end
				};
			end;

			Main_Box.Parent = ScrollingFrame;
			Main_Box.BackgroundTransparency = 1;
			Main_Box.Size = UDim2.new(1,0,0,27);
			Main_Box.Name = Fatality:RandomString();

			DropdownFrame.Name = Fatality:RandomString()
			DropdownFrame.Parent = Main_Box
			DropdownFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			DropdownFrame.BackgroundTransparency = 1.000
			DropdownFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			DropdownFrame.BorderSizePixel = 0
			DropdownFrame.Size = UDim2.new(1, 0, 1, 0)
			DropdownFrame.ZIndex = ZIndex + 1

			DropdownText.Name = Fatality:RandomString()
			DropdownText.Parent = DropdownFrame
			DropdownText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			DropdownText.BackgroundTransparency = 1.000
			DropdownText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			DropdownText.BorderSizePixel = 0
			DropdownText.Position = UDim2.new(0, 0, 0, 5)
			DropdownText.Size = UDim2.new(1, cfg.Option and -100 or -75, 0, 15)
			DropdownText.ZIndex = ZIndex + 2
			DropdownText.FontFace = Fatality.FontSemiBold
			DropdownText.Text = cfg.Name or "Dropdown"
			DropdownText.TextColor3 = Color3.fromRGB(255, 255, 255)
			DropdownText.TextSize = 13.000
			DropdownText.TextTransparency = 0.350
			DropdownText.TextXAlignment = Enum.TextXAlignment.Left
			DropdownText.TextYAlignment = Enum.TextYAlignment.Top

			DropdownButton.Name = Fatality:RandomString()
			DropdownButton.Parent = DropdownFrame
			DropdownButton.AnchorPoint = Vector2.new(1, 0)
			DropdownButton.BackgroundColor3 = Fatality.Colors.Black
			DropdownButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
			DropdownButton.BorderSizePixel = 0
			DropdownButton.Position = UDim2.new(1, cfg.Option and -25 or 0, 0, 6)
			DropdownButton.Size = UDim2.new(0, 70, 1, -12)
			DropdownButton.ZIndex = ZIndex + 2
			DropdownButton.Font = Enum.Font.SourceSans
			DropdownButton.Text = ""
			DropdownButton.TextColor3 = Color3.fromRGB(0, 0, 0)
			DropdownButton.TextSize = 14.000

			UICorner.CornerRadius = UDim.new(0, 2)
			UICorner.Parent = DropdownButton

			DropdownValueText.Name = Fatality:RandomString()
			DropdownValueText.Parent = DropdownButton
			DropdownValueText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			DropdownValueText.BackgroundTransparency = 1.000
			DropdownValueText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			DropdownValueText.BorderSizePixel = 0
			DropdownValueText.Position = UDim2.new(0, 5, 0, 0)
			DropdownValueText.Size = UDim2.new(1, -20, 1, 0)
			DropdownValueText.ZIndex = ZIndex + 3
			DropdownValueText.FontFace = Fatality.FontSemiBold
			DropdownValueText.Text = cfg.Multi and table.concat(SelectedValues,", ") or SelectedValues
			DropdownValueText.TextColor3 = Color3.fromRGB(255, 255, 255)
			DropdownValueText.TextSize = 12.000
			DropdownValueText.TextTransparency = 0.350
			DropdownValueText.TextTruncate = Enum.TextTruncate.AtEnd
			DropdownValueText.TextXAlignment = Enum.TextXAlignment.Left

			DropdownIcon.Name = Fatality:RandomString()
			DropdownIcon.Parent = DropdownButton
			DropdownIcon.AnchorPoint = Vector2.new(1, 0.5)
			DropdownIcon.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			DropdownIcon.BackgroundTransparency = 1.000
			DropdownIcon.BorderColor3 = Color3.fromRGB(0, 0, 0)
			DropdownIcon.BorderSizePixel = 0
			DropdownIcon.Position = UDim2.new(1, -3, 0.5, 0)
			DropdownIcon.Size = UDim2.new(0, 12, 0, 12)
			DropdownIcon.ZIndex = ZIndex + 3
			DropdownIcon.Image = "rbxassetid://10709790948"
			DropdownIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)
			DropdownIcon.ImageTransparency = 0.350

			-- Dropdown List
			local DropdownList = Instance.new("Frame")
			local UICorner_2 = Instance.new("UICorner")
			local UIStroke = Instance.new("UIStroke")
			local ScrollingFrame = Instance.new("ScrollingFrame")
			local UIListLayout = Instance.new("UIListLayout")

			local OwnWindow = Fatality:GetWindowFromElement(DropdownFrame);

			DropdownList.Name = Fatality:RandomString()
			DropdownList.Parent = OwnWindow
			DropdownList.Active = true
			DropdownList.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
			DropdownList.BorderColor3 = Color3.fromRGB(0, 0, 0)
			DropdownList.BorderSizePixel = 0
			DropdownList.ClipsDescendants = true
			DropdownList.Position = UDim2.new(0, 0, 0, 0)
			DropdownList.Size = UDim2.new(0, 150, 0, 0)
			DropdownList.ZIndex = 250
			DropdownList.Visible = false

			UICorner_2.CornerRadius = UDim.new(0, 2)
			UICorner_2.Parent = DropdownList

			UIStroke.Color = Color3.fromRGB(29, 29, 29)
			UIStroke.Transparency = 1
			UIStroke.Parent = DropdownList

			ScrollingFrame.Parent = DropdownList
			ScrollingFrame.Active = true
			ScrollingFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			ScrollingFrame.BackgroundTransparency = 1.000
			ScrollingFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			ScrollingFrame.BorderSizePixel = 0
			ScrollingFrame.Position = UDim2.new(0, 5, 0, 5)
			ScrollingFrame.Size = UDim2.new(1, -10, 1, -10)
			ScrollingFrame.ZIndex = 251
			ScrollingFrame.ScrollBarThickness = 0

			UIListLayout.Parent = ScrollingFrame
			UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
			UIListLayout.Padding = UDim.new(0, 3)

			Fatality:ScrollSignal(ScrollingFrame,UIListLayout,'Y');

			local function ToggleDropdown(state)
				IsOpen = state;

				if state then
					DropdownList.Visible = true;
					local size = math.clamp(UIListLayout.AbsoluteContentSize.Y + 10, 0, 150);

					DropdownList.Position = UDim2.fromOffset(
						DropdownButton.AbsolutePosition.X,
						DropdownButton.AbsolutePosition.Y + DropdownButton.AbsoluteSize.Y + 5
					);

					Fatality:CreateAnimation(DropdownList,0.35,{
						Size = UDim2.new(0, 150, 0, size)
					});

					Fatality:CreateAnimation(UIStroke,0.35,{
						Transparency = 0
					});

					Fatality:CreateAnimation(DropdownIcon,0.35,{
						Rotation = 180
					});
				else
					Fatality:CreateAnimation(DropdownList,0.35,{
						Size = UDim2.new(0, 150, 0, 0)
					});

					Fatality:CreateAnimation(UIStroke,0.35,{
						Transparency = 1
					});

					Fatality:CreateAnimation(DropdownIcon,0.35,{
						Rotation = 0
					});

					task.wait(0.35);
					DropdownList.Visible = false;
				end;
			end;

			DropdownButton.MouseButton1Click:Connect(function()
				ToggleDropdown(not IsOpen);
			end);

			UserInputService.InputBegan:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
					if not Fatality:IsMouseOverFrame(DropdownList) and not Fatality:IsMouseOverFrame(DropdownButton) and IsOpen then
						ToggleDropdown(false);
					end;
				end;
			end);

			-- Add Values
			for _,value in ipairs(cfg.Values or {}) do
				local ValueButton = Instance.new("TextButton")

				ValueButton.Name = Fatality:RandomString()
				ValueButton.Parent = ScrollingFrame
				ValueButton.BackgroundColor3 = Fatality.Colors.Black
				ValueButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
				ValueButton.BorderSizePixel = 0
				ValueButton.Size = UDim2.new(1, 0, 0, 20)
				ValueButton.ZIndex = 252
				ValueButton.FontFace = Fatality.FontSemiBold
				ValueButton.Text = value
				ValueButton.TextColor3 = Color3.fromRGB(255, 255, 255)
				ValueButton.TextSize = 12.000
				ValueButton.TextTransparency = 0.350
				ValueButton.TextXAlignment = Enum.TextXAlignment.Left

				local UIPadding = Instance.new("UIPadding")
				UIPadding.Parent = ValueButton
				UIPadding.PaddingLeft = UDim.new(0, 5)

				Fatality:CreateHover(ValueButton,function(bool)
					if bool then
						Fatality:CreateAnimation(ValueButton,0.25,{
							BackgroundColor3 = Fatality.Colors.Main
						});
					else
						Fatality:CreateAnimation(ValueButton,0.25,{
							BackgroundColor3 = Fatality.Colors.Black
						});
					end;
				end);

				ValueButton.MouseButton1Click:Connect(function()
					if cfg.Multi then
						local found = false;
						for i,v in ipairs(SelectedValues) do
							if v == value then
								table.remove(SelectedValues, i);
								found = true;
								break;
							end;
						end;

						if not found then
							table.insert(SelectedValues, value);
						end;

						DropdownValueText.Text = table.concat(SelectedValues,", ");
					else
						SelectedValues = value;
						DropdownValueText.Text = value;
						ToggleDropdown(false);
					end;

					cfg.Callback(SelectedValues);

					if cfg.Flag then
						Fatality.Flags[cfg.Flag].Set(SelectedValues);
					end;
				end);
			end;

			if cfg.Option then
				OptionButton.Name = Fatality:RandomString()
				OptionButton.Parent = DropdownFrame
				OptionButton.AnchorPoint = Vector2.new(1, 0)
				OptionButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
				OptionButton.BackgroundTransparency = 1.000
				OptionButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
				OptionButton.BorderSizePixel = 0
				OptionButton.LayoutOrder = 5
				OptionButton.Position = UDim2.new(1, 0, 0, 6)
				OptionButton.Size = UDim2.new(0, 20, 0, 20)
				OptionButton.ZIndex = ZIndex + 5
				OptionButton.Image = "rbxassetid://10734950020"
				OptionButton.ImageTransparency = 0.600

				return {
					Option = Fatality:CreateOption(OptionButton)
				};
			end;

			return {};
		end,

		AddKeybind = function(self,cfg: Keybind)
			cfg = cfg or {};

			local KeybindFrame = Instance.new("Frame")
			local KeybindText = Instance.new("TextLabel")
			local KeybindButton = Instance.new("TextButton")
			local UICorner = Instance.new("UICorner")
			local Main_Box = Instance.new('Frame');

			local CurrentKey = cfg.Default or Enum.KeyCode.Unknown;
			local IsListening = false;

			if cfg.Flag then
				Fatality.Flags[cfg.Flag] = {
					Set = function(key)
						CurrentKey = key;
						KeybindButton.Text = (typeof(CurrentKey) == "EnumItem" and CurrentKey.Name) or tostring(CurrentKey);
						cfg.Callback(CurrentKey);
					end,
					Get = function()
						return CurrentKey;
					end
				};
			end;

			Main_Box.Parent = ScrollingFrame;
			Main_Box.BackgroundTransparency = 1;
			Main_Box.Size = UDim2.new(1,0,0,27);
			Main_Box.Name = Fatality:RandomString();

			KeybindFrame.Name = Fatality:RandomString()
			KeybindFrame.Parent = Main_Box
			KeybindFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			KeybindFrame.BackgroundTransparency = 1.000
			KeybindFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			KeybindFrame.BorderSizePixel = 0
			KeybindFrame.Size = UDim2.new(1, 0, 1, 0)
			KeybindFrame.ZIndex = ZIndex + 1

			KeybindText.Name = Fatality:RandomString()
			KeybindText.Parent = KeybindFrame
			KeybindText.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			KeybindText.BackgroundTransparency = 1.000
			KeybindText.BorderColor3 = Color3.fromRGB(0, 0, 0)
			KeybindText.BorderSizePixel = 0
			KeybindText.Position = UDim2.new(0, 0, 0, 5)
			KeybindText.Size = UDim2.new(1, -75, 0, 15)
			KeybindText.ZIndex = ZIndex + 2
			KeybindText.FontFace = Fatality.FontSemiBold
			KeybindText.Text = cfg.Name or "Keybind"
			KeybindText.TextColor3 = Color3.fromRGB(255, 255, 255)
			KeybindText.TextSize = 13.000
			KeybindText.TextTransparency = 0.350
			KeybindText.TextXAlignment = Enum.TextXAlignment.Left
			KeybindText.TextYAlignment = Enum.TextYAlignment.Top

			KeybindButton.Name = Fatality:RandomString()
			KeybindButton.Parent = KeybindFrame
			KeybindButton.AnchorPoint = Vector2.new(1, 0)
			KeybindButton.BackgroundColor3 = Fatality.Colors.Black
			KeybindButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
			KeybindButton.BorderSizePixel = 0
			KeybindButton.Position = UDim2.new(1, 0, 0, 6)
			KeybindButton.Size = UDim2.new(0, 70, 1, -12)
			KeybindButton.ZIndex = ZIndex + 2
			KeybindButton.FontFace = Fatality.FontSemiBold
			KeybindButton.Text = (typeof(CurrentKey) == "EnumItem" and CurrentKey.Name) or tostring(CurrentKey)
			KeybindButton.TextColor3 = Color3.fromRGB(255, 255, 255)
			KeybindButton.TextSize = 12.000
			KeybindButton.TextTransparency = 0.350

			UICorner.CornerRadius = UDim.new(0, 2)
			UICorner.Parent = KeybindButton

			KeybindButton.MouseButton1Click:Connect(function()
				IsListening = true;
				KeybindButton.Text = "...";

				Fatality:CreateAnimation(KeybindButton,0.25,{
					BackgroundColor3 = Fatality.Colors.Main
				});
			end);

			UserInputService.InputBegan:Connect(function(input)
				if IsListening then
					if input.UserInputType == Enum.UserInputType.Keyboard then
						CurrentKey = input.KeyCode;
						KeybindButton.Text = CurrentKey.Name;
						IsListening = false;

						Fatality:CreateAnimation(KeybindButton,0.25,{
							BackgroundColor3 = Fatality.Colors.Black
						});

						cfg.Callback(CurrentKey);

						if cfg.Flag then
							Fatality.Flags[cfg.Flag].Set(CurrentKey);
						end;
					end;
				end;
			end);

			return {};
		end
	});
end;

function Fatality:AddDragBlacklist(Frame: Frame)
	table.insert(Fatality.DragBlacklist, Frame);
end;

function Fatality:RemoveDragBlacklist(Frame: Frame)
	for i,v in ipairs(Fatality.DragBlacklist) do
		if v == Frame then
			table.remove(Fatality.DragBlacklist, i);
			break;
		end;
	end;
end;

-- Notifier System
function Fatality:CreateNotifier(): Notifier
	local NotifierHolder = Instance.new("Frame")
	local UIListLayout = Instance.new("UIListLayout")

	NotifierHolder.Name = Fatality:RandomString()
	NotifierHolder.Parent = CoreGui
	NotifierHolder.AnchorPoint = Vector2.new(1, 1)
	NotifierHolder.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	NotifierHolder.BackgroundTransparency = 1.000
	NotifierHolder.BorderColor3 = Color3.fromRGB(0, 0, 0)
	NotifierHolder.BorderSizePixel = 0
	NotifierHolder.Position = UDim2.new(1, -10, 1, -10)
	NotifierHolder.Size = UDim2.new(0, 300, 0, 500)

	UIListLayout.Parent = NotifierHolder
	UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
	UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
	UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
	UIListLayout.Padding = UDim.new(0, 5)

	return Fatality:CreateResponse({
		Notify = function(self, cfg: Notify)
			cfg = cfg or {};

			local NotifyFrame = Instance.new("Frame")
			local UICorner = Instance.new("UICorner")
			local UIStroke = Instance.new("UIStroke")
			local Icon = Instance.new("ImageLabel")
			local Title = Instance.new("TextLabel")
			local Content = Instance.new("TextLabel")

			NotifyFrame.Name = Fatality:RandomString()
			NotifyFrame.Parent = NotifierHolder
			NotifyFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
			NotifyFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			NotifyFrame.BorderSizePixel = 0
			NotifyFrame.Size = UDim2.new(1, 0, 0, 0)
			NotifyFrame.ClipsDescendants = true

			UICorner.CornerRadius = UDim.new(0, 4)
			UICorner.Parent = NotifyFrame

			UIStroke.Color = Color3.fromRGB(29, 29, 29)
			UIStroke.Parent = NotifyFrame

			Icon.Name = Fatality:RandomString()
			Icon.Parent = NotifyFrame
			Icon.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			Icon.BackgroundTransparency = 1.000
			Icon.BorderColor3 = Color3.fromRGB(0, 0, 0)
			Icon.BorderSizePixel = 0
			Icon.Position = UDim2.new(0, 10, 0, 10)
			Icon.Size = UDim2.new(0, 20, 0, 20)
			Icon.Image = cfg.Icon and Fatality:GetIcon(cfg.Icon) or "rbxassetid://10723347051"
			Icon.ImageColor3 = Fatality.Colors.Main

			Title.Name = Fatality:RandomString()
			Title.Parent = NotifyFrame
			Title.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			Title.BackgroundTransparency = 1.000
			Title.BorderColor3 = Color3.fromRGB(0, 0, 0)
			Title.BorderSizePixel = 0
			Title.Position = UDim2.new(0, 35, 0, 10)
			Title.Size = UDim2.new(1, -45, 0, 15)
			Title.FontFace = Fatality.FontSemiBold
			Title.Text = cfg.Title or "Notification"
			Title.TextColor3 = Color3.fromRGB(255, 255, 255)
			Title.TextSize = 13.000
			Title.TextXAlignment = Enum.TextXAlignment.Left

			Content.Name = Fatality:RandomString()
			Content.Parent = NotifyFrame
			Content.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
			Content.BackgroundTransparency = 1.000
			Content.BorderColor3 = Color3.fromRGB(0, 0, 0)
			Content.BorderSizePixel = 0
			Content.Position = UDim2.new(0, 35, 0, 25)
			Content.Size = UDim2.new(1, -45, 1, -35)
			Content.FontFace = Fatality.FontSemiBold
			Content.Text = cfg.Content or ""
			Content.TextColor3 = Color3.fromRGB(255, 255, 255)
			Content.TextSize = 12.000
			Content.TextTransparency = 0.450
			Content.TextWrapped = true
			Content.TextXAlignment = Enum.TextXAlignment.Left
			Content.TextYAlignment = Enum.TextYAlignment.Top

			local ContentSize = Fatality:GetTextSize(Content);
			local Height = math.clamp(ContentSize.Y + 40, 60, 150);

			Fatality:CreateAnimation(NotifyFrame,0.45,{
				Size = UDim2.new(1, 0, 0, Height)
			});

			task.delay(cfg.Duration or 3, function()
				Fatality:CreateAnimation(NotifyFrame,0.45,{
					Size = UDim2.new(1, 0, 0, 0)
				});

				task.wait(0.45);
				NotifyFrame:Destroy();
			end);
		end
	});
end;

return Fatality
