if Library then
    Library:Unload()
end

local LoadTick = os.clock()

local Library do
    local Workspace = game:GetService("Workspace")
    local UserInputService = game:GetService("UserInputService")
    local Players = game:GetService("Players")
    local HttpService = game:GetService("HttpService")
    local RunService = game:GetService("RunService")
    local CoreGui = cloneref and cloneref(game:GetService("CoreGui")) or game:GetService("CoreGui")
    local TweenService = game:GetService("TweenService")

    gethui = gethui or function()
        return CoreGui
    end

    local LocalPlayer = Players.LocalPlayer
    local Camera = Workspace.CurrentCamera
    local Mouse = LocalPlayer:GetMouse()

    local FromRGB = Color3.fromRGB
    local FromHSV = Color3.fromHSV
    local FromHex = Color3.fromHex

    local RGBSequence = ColorSequence.new
    local RGBSequenceKeypoint = ColorSequenceKeypoint.new
    local NumSequence = NumberSequence.new
    local NumSequenceKeypoint = NumberSequenceKeypoint.new

    local UDim2New = UDim2.new
    local UDimNew = UDim.new
    local Vector2New = Vector2.new

    local MathClamp = math.clamp
    local MathFloor = math.floor
    local MathAbs = math.abs
    local MathSin = math.sin

    local TableInsert = table.insert
    local TableFind = table.find
    local TableRemove = table.remove
    local TableConcat = table.concat
    local TableClone = table.clone
    local TableUnpack = table.unpack

    local StringFormat = string.format
    local StringFind = string.find
    local StringGSub = string.gsub
    local StringLower = string.lower
    local StringLen = string.len

    local InstanceNew = Instance.new

    local RectNew = Rect.new

    local IsMobile = UserInputService.TouchEnabled or false

    Library = {
        Theme =  { },

        MenuKeybind = tostring(Enum.KeyCode.RightControl), 
        Flags = { },

        Tween = {
            Time = 0.2,
            Style = Enum.EasingStyle.Quad,
            Direction = Enum.EasingDirection.Out
        },

        FadeSpeed = 0.2,

        Folders = {
            Directory = "scoot",
            Configs = "scoot/Configs",
            Assets = "scoot/Assets",
        },

        Images = {
            ["Saturation"] = {"Saturation.png", "https://github.com/sametexe001/images/blob/main/saturation.png?raw=true" },
            ["Value"] = { "Value.png", "https://github.com/sametexe001/images/blob/main/value.png?raw=true" },
            ["Hue"] = { "Hue.png", "https://github.com/sametexe001/images/blob/main/horizontalhue.png?raw=true" },
            ["Checkers"] = { "Checkers.png", "https://github.com/sametexe001/images/blob/main/checkers.png?raw=true" },
        },

        -- Ignore below
        Pages = { },
        Sections = { },

        Connections = { },
        Threads = { },

        ThemeMap = { },
        ThemeItems = { },

        CopiedColor = nil,

        OpenFrames = { },

        CurrentPage = nil,

        SearchItems = { },

        SetFlags = { },

        UnnamedConnections = 0,
        UnnamedFlags = 0,

        Holder = nil,
        NotifHolder = nil,
        UnusedHolder = nil,
        Font = nil,
        KeyList = nil,

        Colorpickers = { },
    }

    Library.__index = Library
    Library.Sections.__index = Library.Sections
    Library.Pages.__index = Library.Pages

    local Keys = {
        ["Unknown"]           = "Unknown",
        ["Backspace"]         = "Back",
        ["Tab"]               = "Tab",
        ["Clear"]             = "Clear",
        ["Return"]            = "Return",
        ["Pause"]             = "Pause",
        ["Escape"]            = "Escape",
        ["Space"]             = "Space",
        ["QuotedDouble"]      = '"',
        ["Hash"]              = "#",
        ["Dollar"]            = "$",
        ["Percent"]           = "%",
        ["Ampersand"]         = "&",
        ["Quote"]             = "'",
        ["LeftParenthesis"]   = "(",
        ["RightParenthesis"]  = " )",
        ["Asterisk"]          = "*",
        ["Plus"]              = "+",
        ["Comma"]             = ",",
        ["Minus"]             = "-",
        ["Period"]            = ".",
        ["Slash"]             = "`",
        ["Three"]             = "3",
        ["Seven"]             = "7",
        ["Eight"]             = "8",
        ["Colon"]             = ":",
        ["Semicolon"]         = ";",
        ["LessThan"]          = "<",
        ["GreaterThan"]       = ">",
        ["Question"]          = "?",
        ["Equals"]            = "=",
        ["At"]                = "@",
        ["LeftBracket"]       = "LeftBracket",
        ["RightBracket"]      = "RightBracked",
        ["BackSlash"]         = "BackSlash",
        ["Caret"]             = "^",
        ["Underscore"]        = "_",
        ["Backquote"]         = "`",
        ["LeftCurly"]         = "{",
        ["Pipe"]              = "|",
        ["RightCurly"]        = "}",
        ["Tilde"]             = "~",
        ["Delete"]            = "Delete",
        ["End"]               = "End",
        ["KeypadZero"]        = "Keypad0",
        ["KeypadOne"]         = "Keypad1",
        ["KeypadTwo"]         = "Keypad2",
        ["KeypadThree"]       = "Keypad3",
        ["KeypadFour"]        = "Keypad4",
        ["KeypadFive"]        = "Keypad5",
        ["KeypadSix"]         = "Keypad6",
        ["KeypadSeven"]       = "Keypad7",
        ["KeypadEight"]       = "Keypad8",
        ["KeypadNine"]        = "Keypad9",
        ["KeypadPeriod"]      = "KeypadP",
        ["KeypadDivide"]      = "KeypadD",
        ["KeypadMultiply"]    = "KeypadM",
        ["KeypadMinus"]       = "KeypadM",
        ["KeypadPlus"]        = "KeypadP",
        ["KeypadEnter"]       = "KeypadE",
        ["KeypadEquals"]      = "KeypadE",
        ["Insert"]            = "Insert",
        ["Home"]              = "Home",
        ["PageUp"]            = "PageUp",
        ["PageDown"]          = "PageDown",
        ["RightShift"]        = "RightShift",
        ["LeftShift"]         = "LeftShift",
        ["RightControl"]      = "RightControl",
        ["LeftControl"]       = "LeftControl",
        ["LeftAlt"]           = "LeftAlt",
        ["RightAlt"]          = "RightAlt"
    }

    local Themes = {
        ["Preset"] = {
            ["Background"] = FromRGB(14, 17, 15),
            ["Border"] = FromRGB(12, 12, 12),
            ["Inline"] = FromRGB(20, 24, 21),
            ["Hovered Element"] = FromRGB(37, 42, 45),
            ["Page Background"] = FromRGB(25, 30, 26),
            ["Outline"] = FromRGB(42, 49, 45),
            ["Element"] = FromRGB(30, 36, 31),
            ["Gradient"] = FromRGB(208, 208, 208),
            ["Text"] = FromRGB(235, 235, 235),
            ["Text Stroke"] = FromRGB(0, 0, 0),
            ["Placeholder Text"] = FromRGB(185, 185, 185),
            ["Accent"] = FromRGB(202, 243, 255)
        }
    }

    Library.Theme = TableClone(Themes["Preset"])

    -- Folders
    for Index, Value in Library.Folders do 
        if not isfolder(Value) then
            makefolder(Value)
        end
    end

    -- Images
    for Index, Value in Library.Images do 
        local ImageData = Value

        local ImageName = ImageData[1]
        local ImageLink = ImageData[2]
        
        if not isfile(Library.Folders.Assets .. "/" .. ImageName) then
            writefile(Library.Folders.Assets .. "/" .. ImageName, game:HttpGet(ImageLink))
        end
    end

    -- Tweening
    local Tween = { } do
        Tween.__index = Tween

        Tween.Create = function(self, Item, Info, Goal, IsRawItem)
            Item = IsRawItem and Item or Item.Instance
            Info = Info or TweenInfo.new(Library.Tween.Time, Library.Tween.Style, Library.Tween.Direction)

            local NewTween = {
                Tween = TweenService:Create(Item, Info, Goal),
                Info = Info,
                Goal = Goal,
                Item = Item
            }

            NewTween.Tween:Play()

            setmetatable(NewTween, Tween)

            return NewTween
        end

        Tween.GetProperty = function(self, Item)
            Item = Item or self.Item 

            if Item:IsA("Frame") then
                return { "BackgroundTransparency" }
            elseif Item:IsA("TextLabel") or Item:IsA("TextButton") then
                return { "TextTransparency", "BackgroundTransparency" }
            elseif Item:IsA("ImageLabel") or Item:IsA("ImageButton") then
                return { "BackgroundTransparency", "ImageTransparency" }
            elseif Item:IsA("ScrollingFrame") then
                return { "BackgroundTransparency", "ScrollBarImageTransparency" }
            elseif Item:IsA("TextBox") then
                return { "TextTransparency", "BackgroundTransparency" }
            elseif Item:IsA("UIStroke") then 
                return { "Transparency" }
            end
        end

        Tween.FadeItem = function(self, Item, Property, Visibility, Speed)
            local Item = Item or self.Item 

            local OldTransparency = Item[Property]
            Item[Property] = Visibility and 1 or OldTransparency

            local NewTween = Tween:Create(Item, TweenInfo.new(Speed or Library.Tween.Time, Library.Tween.Style, Library.Tween.Direction), {
                [Property] = Visibility and OldTransparency or 1
            }, true)

            Library:Connect(NewTween.Tween.Completed, function()
                if not Visibility then 
                    task.wait()
                    Item[Property] = OldTransparency
                end
            end)

            return NewTween
        end

        Tween.Get = function(self)
            if not self.Tween then 
                return
            end

            return self.Tween, self.Info, self.Goal
        end

        Tween.Pause = function(self)
            if not self.Tween then 
                return
            end

            self.Tween:Pause()
        end

        Tween.Play = function(self)
            if not self.Tween then 
                return
            end

            self.Tween:Play()
        end

        Tween.Clean = function(self)
            if not self.Tween then 
                return
            end

            Tween:Pause()
            self = nil
        end
    end

    -- Instances
    local Instances = { } do
        Instances.__index = Instances

        Instances.Create = function(self, Class, Properties)
            local NewItem = {
                Instance = InstanceNew(Class),
                Properties = Properties,
                Class = Class
            }

            setmetatable(NewItem, Instances)

            for Property, Value in NewItem.Properties do
                NewItem.Instance[Property] = Value
            end

            return NewItem
        end

        Instances.FadeItem = function(self, Visibility, Speed)
            local Item = self.Instance

            if Visibility == true then 
                Item.Visible = true
            end

            local Descendants = Item:GetDescendants()
            TableInsert(Descendants, Item)

            local NewTween

            for Index, Value in Descendants do 
                local TransparencyProperty = Tween:GetProperty(Value)

                if not TransparencyProperty then 
                    continue
                end

                if type(TransparencyProperty) == "table" then 
                    for _, Property in TransparencyProperty do 
                        NewTween = Tween:FadeItem(Value, Property, not Visibility, Speed)
                    end
                else
                    NewTween = Tween:FadeItem(Value, TransparencyProperty, not Visibility, Speed)
                end
            end
        end

        Instances.AddToTheme = function(self, Properties)
            if not self.Instance then 
                return
            end

            Library:AddToTheme(self, Properties)
        end

        Instances.ChangeItemTheme = function(self, Properties)
            if not self.Instance then 
                return
            end

            Library:ChangeItemTheme(self, Properties)
        end

        Instances.Connect = function(self, Event, Callback, Name)
            if not self.Instance then 
                return
            end

            if not self.Instance[Event] then 
                return
            end

            if IsMobile then
                if Event == "MouseButton1Down" or Event == "MouseButton1Click" then
                    Event = "TouchTap"
                elseif Event == "MouseButton2Down" or Event == "MouseButton2Click" then
                    Event = "TouchLongPress"
                end
            end

            return Library:Connect(self.Instance[Event], Callback, Name)
        end

        Instances.Tween = function(self, Info, Goal)
            if not self.Instance then 
                return
            end

            return Tween:Create(self, Info, Goal)
        end

        Instances.Disconnect = function(self, Name)
            if not self.Instance then 
                return
            end

            return Library:Disconnect(Name)
        end

        Instances.Clean = function(self)
            if not self.Instance then 
                return
            end

            self.Instance:Destroy()
            self = nil
        end

        Instances.MakeDraggable = function(self)
            if not self.Instance then 
                return
            end

            local Gui = self.Instance

            local Dragging = false 
            local DragStart
            local StartPosition 

            local Set = function(Input)
                local DragDelta = Input.Position - DragStart
                self:Tween(TweenInfo.new(0.16, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2New(StartPosition.X.Scale, StartPosition.X.Offset + DragDelta.X, StartPosition.Y.Scale, StartPosition.Y.Offset + DragDelta.Y)})
            end

            local InputChanged

            self:Connect("InputBegan", function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
                    Dragging = true

                    DragStart = Input.Position
                    StartPosition = Gui.Position

                    if InputChanged then 
                        return
                    end

                    InputChanged = Input.Changed:Connect(function()
                        if Input.UserInputState == Enum.UserInputState.End then
                            Dragging = false

                            InputChanged:Disconnect()
                            InputChanged = nil
                        end
                    end)
                end
            end)

            Library:Connect(UserInputService.InputChanged, function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseMovement or Input.UserInputType == Enum.UserInputType.Touch then
                    if Dragging then
                        Set(Input)
                    end
                end
            end)

            return Dragging
        end

        Instances.MakeResizeable = function(self, Minimum, Maximum)
            if not self.Instance then 
                return
            end

            local Gui = self.Instance

            local Resizing = false 
            local Start = UDim2New()
            local Delta = UDim2New()
            local ResizeMax = Gui.Parent.AbsoluteSize - Gui.AbsoluteSize

            local ResizeButton = Instances:Create("ImageButton", {
				Parent = Gui,
                Image = "rbxassetid://",
				AnchorPoint = Vector2New(1, 1),
				BorderColor3 = FromRGB(0, 0, 0),
				Size = UDim2New(0, 6, 0, 6),
				Position = UDim2New(1, -4, 1, -4),
                Name = "\0",
				BorderSizePixel = 0,
				BackgroundTransparency = 1,
                ZIndex = 5,
				AutoButtonColor = false,
                Visible = true,
			})  ResizeButton:AddToTheme({ImageColor3 = "Accent"})

            local InputChanged

            ResizeButton:Connect("InputBegan", function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then

                    Resizing = true

                    Start = Gui.Size - UDim2New(0, Input.Position.X, 0, Input.Position.Y)

                    if InputChanged then 
                        return
                    end

                    InputChanged = Input.Changed:Connect(function()
                        if Input.UserInputState == Enum.UserInputState.End then
                            Resizing = false

                            InputChanged:Disconnect()
                            InputChanged = nil
                        end
                    end)
                end
            end)

            Library:Connect(UserInputService.InputChanged, function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseMovement or Input.UserInputType == Enum.UserInputType.Touch then
                    if Resizing then
                        ResizeMax = Maximum or Gui.Parent.AbsoluteSize - Gui.AbsoluteSize

                        Delta = Start + UDim2New(0, Input.Position.X, 0, Input.Position.Y)
                        Delta = UDim2New(0, math.clamp(Delta.X.Offset, Minimum.X, ResizeMax.X), 0, math.clamp(Delta.Y.Offset, Minimum.Y, ResizeMax.Y))

                        Tween:Create(Gui, TweenInfo.new(0.17, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = Delta}, true)
                    end
                end
            end)

            return Resizing
        end

        Instances.OnHover = function(self, Function)
            if not self.Instance then 
                return
            end
            
            return Library:Connect(self.Instance.MouseEnter, Function)
        end

        Instances.OnHoverLeave = function(self, Function)
            if not self.Instance then 
                return
            end
            
            return Library:Connect(self.Instance.MouseLeave, Function)
        end

        Instances.Border = function(self, Type)
            if not self.Instance then 
                return
            end

            local Color = Type == "Border" and Library.Theme.Border or Type == "Outline" and Library.Theme.Outline
        
            local UIStroke = Instances:Create("UIStroke", {
                Parent = self.Instance,
                Color = Color,
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter
            })  UIStroke:AddToTheme({Color = Type})

            return UIStroke
        end

        Instances.TextBorder = function(self)
            if not self.Instance then 
                return
            end

            local UIStroke = Instances:Create("UIStroke", {
                Parent = self.Instance,
                Color = Library.Theme["Text Stroke"],
                Thickness = 1,
                Transparency = 0.6,
                LineJoinMode = Enum.LineJoinMode.Miter
            })  UIStroke:AddToTheme({Color = "Text Stroke"})

            return UIStroke
        end 
    end

    -- Custom font
    local CustomFont = { } do
        function CustomFont:New(Name, Weight, Style, Data)
            if isfile(Library.Folders.Assets .. "/" .. Name .. ".json") then
                return Font.new(getcustomasset(Library.Folders.Assets .. "/" .. Name .. ".json"))
            end

            if not isfile(Library.Folders.Assets .. "/" .. Name .. ".ttf") then 
                writefile(Library.Folders.Assets .. "/" .. Name .. ".ttf", game:HttpGet(Data.Url))
            end

            local FontData = {
                name = Name,
                faces = { {
                    name = "Regular",
                    weight = Weight,
                    style = Style,
                    assetId = getcustomasset(Library.Folders.Assets .. "/" .. Name .. ".ttf")
                } }
            }

            writefile(Library.Folders.Assets .. "/" .. Name .. ".json", HttpService:JSONEncode(FontData))
            return Font.new(getcustomasset(Library.Folders.Assets .. "/" .. Name .. ".json"))
        end

        function CustomFont:Get(Name)
            if isfile(Library.Folders.Assets .. "/" .. Name .. ".json") then
                return Font.new(getcustomasset(Library.Folders.Assets .. "/" .. Name .. ".json"))
            end
        end

        CustomFont:New("Monaco", 400, "Regular", {
            Url = "https://github.com/sametexe001/luas/raw/refs/heads/main/fonts/Monaco.ttf"
        })

        Library.Font = CustomFont:Get("Monaco")
    end

    Library.Holder = Instances:Create("ScreenGui", {
        Parent = gethui(),
        Name = "\0",
        ZIndexBehavior = Enum.ZIndexBehavior.Global,
        DisplayOrder = 2,
        ResetOnSpawn = false
    })

    Library.UnusedHolder = Instances:Create("ScreenGui", {
        Parent = gethui(),
        Name = "\0",
        ZIndexBehavior = Enum.ZIndexBehavior.Global,
        Enabled = false,
        ResetOnSpawn = false
    })

    Library.NotifHolder = Instances:Create("Frame", {
        Parent = Library.Holder.Instance,
        Name = "\0",
        BackgroundTransparency = 1,
        Size = UDim2New(0, 0, 1, 0),
        BorderColor3 = FromRGB(0, 0, 0),
        BorderSizePixel = 0,
        AutomaticSize = Enum.AutomaticSize.X,
        BackgroundColor3 = FromRGB(255, 255, 255)
    })

    Instances:Create("UIListLayout", {
        Parent = Library.NotifHolder.Instance,
        Name = "\0",
        VerticalAlignment = Enum.VerticalAlignment.Bottom,
        Padding = UDimNew(0, 12),
        SortOrder = Enum.SortOrder.LayoutOrder
    })

    Instances:Create("UIPadding", {
        Parent = Library.NotifHolder.Instance,
        Name = "\0",
        PaddingTop = UDimNew(0, 12),
        PaddingBottom = UDimNew(0, 12),
        PaddingRight = UDimNew(0, 12),
        PaddingLeft = UDimNew(0, 12)
    })

    Library.Unload = function(self)
        for Index, Value in self.Connections do 
            Value.Connection:Disconnect()
        end

        for Index, Value in self.Threads do 
            coroutine.close(Value)
        end

        if self.Holder then 
            self.Holder:Clean()
        end

        Library = nil 
        getgenv().Library = nil

        UserInputService.MouseIconEnabled = true
    end

    Library.GetImage = function(self, Image)
        local ImageData = self.Images[Image]

        if not ImageData then 
            return
        end

        return getcustomasset(self.Folders.Assets .. "/" .. ImageData[1])
    end

    Library.Round = function(self, Number, Float)
        local Multiplier = 1 / (Float or 1)
        return MathFloor(Number * Multiplier) / Multiplier
    end

    Library.Thread = function(self, Function)
        local NewThread = coroutine.create(Function)
        
        coroutine.wrap(function()
            coroutine.resume(NewThread)
        end)()

        TableInsert(self.Threads, NewThread)
        return NewThread
    end
    
    Library.SafeCall = function(self, Function, ...)
        local Arguements = { ... }
        local Success, Result = pcall(Function, TableUnpack(Arguements))

        if not Success then
            warn(Result)
            return false
        end

        return Success
    end

    Library.Connect = function(self, Event, Callback, Name)
        Name = Name or StringFormat("Connection%s%s", self.UnnamedConnections + 1, HttpService:GenerateGUID(false))

        local NewConnection = {
            Event = Event,
            Callback = Callback,
            Name = Name,
            Connection = nil
        }

        Library:Thread(function()
            NewConnection.Connection = Event:Connect(Callback)
        end)

        TableInsert(self.Connections, NewConnection)
        return NewConnection
    end

    Library.Disconnect = function(self, Name)
        for _, Connection in self.Connections do 
            if Connection.Name == Name then
                Connection.Connection:Disconnect()
                break
            end
        end
    end

    Library.NextFlag = function(self)
        local FlagNumber = self.UnnamedFlags + 1
        return StringFormat("flag_number_%s_%s", FlagNumber, HttpService:GenerateGUID(false))
    end

    Library.AddToTheme = function(self, Item, Properties)
        Item = Item.Instance or Item 

        local ThemeData = {
            Item = Item,
            Properties = Properties,
        }

        for Property, Value in ThemeData.Properties do
            if type(Value) == "string" then
                Item[Property] = self.Theme[Value]
            else
                Item[Property] = Value()
            end
        end

        TableInsert(self.ThemeItems, ThemeData)
        self.ThemeMap[Item] = ThemeData
    end

	Library.ToRich = function(self, Text, Color)
        if not Color then
            return `<font color="rgb(255, 255, 255)">{Text}</font>`
        end

        if not Color.R or not Color.G or not Color.B then
            return `<font color="rgb(255, 255, 255)">{Text}</font>`
        end

		return `<font color="rgb({MathFloor(Color.R * 255)}, {MathFloor(Color.G * 255)}, {MathFloor(Color.B * 255)})">{Text}</font>`
	end

    Library.GetConfig = function(self)
        local Config = { } 

        local Success, Result = Library:SafeCall(function()
            for Index, Value in Library.Flags do 
                if type(Value) == "table" and Value.Key then
                    Config[Index] = {Key = tostring(Value.Key), Mode = Value.Mode}
                elseif type(Value) == "table" and Value.Color then
                    Config[Index] = {Color = "#" .. Value.Color, Alpha = Value.Alpha}
                else
                    Config[Index] = Value
                end
            end
        end)

        return HttpService:JSONEncode(Config)
    end

    Library.LoadConfig = function(self, Config)
        local Decoded = HttpService:JSONDecode(Config)

        local Success, Result = Library:SafeCall(function()
            for Index, Value in Decoded do 
                local SetFunction = Library.SetFlags[Index]

                if not SetFunction then
                    continue
                end

                if type(Value) == "table" and Value.Key then 
                    SetFunction(Value)
                elseif type(Value) == "table" and Value.Color then
                    SetFunction(Value.Color, Value.Alpha)
                else
                    SetFunction(Value)
                end
            end
        end)

        return Success, Result
    end

    Library.DeleteConfig = function(self, Config)
        if isfile(Library.Folders.Configs .. "/" .. Config) then 
            delfile(Library.Folders.Configs .. "/" .. Config)
        end
    end

    Library.RefreshConfigsList = function(self, Element)
        local CurrentList = { }
        local List = { }

        local ConfigFolderName = StringGSub(Library.Folders.Configs, Library.Folders.Directory .. "/", "")

        for Index, Value in listfiles(Library.Folders.Configs) do
            local FileName = StringGSub(Value, Library.Folders.Directory .. "\\" .. ConfigFolderName .. "\\", "")
            List[Index] = FileName
        end

        local IsNew = #List ~= CurrentList

        if not IsNew then
            for Index = 1, #List do
                if List[Index] ~= CurrentList[Index] then
                    IsNew = true
                    break
                end
            end
        else
            CurrentList = List
            Element:Refresh(CurrentList)
        end
    end

    Library.ChangeItemTheme = function(self, Item, Properties)
        Item = Item.Instance or Item

        if not self.ThemeMap[Item] then 
            return
        end

        self.ThemeMap[Item].Properties = Properties
        self.ThemeMap[Item] = self.ThemeMap[Item]
    end

    Library.ChangeTheme = function(self, Theme, Color)
        self.Theme[Theme] = Color

        for _, Item in self.ThemeItems do
            for Property, Value in Item.Properties do
                if type(Value) == "string" and Value == Theme then
                    Item.Item[Property] = Color
                elseif type(Value) == "function" then
                    Item.Item[Property] = Value()
                end
            end
        end
    end

    Library.IsMouseOverFrame = function(self, Frame, XOffset, YOffset)
        Frame = Frame.Instance
        XOffset = XOffset or 0 
        YOffset = YOffset or 0

        local MousePosition = Vector2New(Mouse.X + XOffset, Mouse.Y + YOffset)

        return MousePosition.X >= Frame.AbsolutePosition.X and MousePosition.X <= Frame.AbsolutePosition.X + Frame.AbsoluteSize.X 
        and MousePosition.Y >= Frame.AbsolutePosition.Y and MousePosition.Y <= Frame.AbsolutePosition.Y + Frame.AbsoluteSize.Y
    end

    Library.Lerp = function(self, Start, Finish, Time)
        return Start + (Finish - Start) * Time
    end

    -- Components
    local Components = { } do
        
        -- ============================================
        -- SETTINGS/GEAR COMPONENT - NUEVO
        -- ============================================
        Components.Settings = function(self, Data)
            local Settings = {
                IsOpen = false,
                Items = {}
            }

            local Items = {} do
                Items["SettingsButton"] = Instances:Create("TextButton", {
                    Parent = Data.Parent.Instance,
                    Name = "\0",
                    Text = "",
                    AutoButtonColor = false,
                    BackgroundTransparency = 1,
                    Size = UDim2New(0, 16, 0, 16),
                    BorderSizePixel = 0,
                    ZIndex = 3
                })

                Items["GearIcon"] = Instances:Create("ImageLabel", {
                    Parent = Items["SettingsButton"].Instance,
                    Name = "\0",
                    ImageColor3 = FromRGB(185, 185, 185),
                    ScaleType = Enum.ScaleType.Fit,
                    BorderColor3 = FromRGB(0, 0, 0),
                    Image = "rbxassetid://106913268193226",
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 1, 0),
                    BorderSizePixel = 0,
                    BackgroundColor3 = FromRGB(255, 255, 255),
                    ZIndex = 3
                })  Items["GearIcon"]:AddToTheme({ImageColor3 = "Placeholder Text"})

                Items["SettingsMenu"] = Instances:Create("Frame", {
                    Parent = Library.UnusedHolder.Instance,
                    Name = "\0",
                    Visible = false,
                    BorderColor3 = FromRGB(12, 12, 12),
                    BorderSizePixel = 2,
                    Size = UDim2New(0, 220, 0, 0),
                    ZIndex = 10,
                    AutomaticSize = Enum.AutomaticSize.Y,
                    BackgroundColor3 = FromRGB(20, 24, 21)
                })  Items["SettingsMenu"]:AddToTheme({BackgroundColor3 = "Inline", BorderColor3 = "Border"})

                Instances:Create("UIStroke", {
                    Parent = Items["SettingsMenu"].Instance,
                    Name = "\0",
                    Color = FromRGB(42, 49, 45),
                    LineJoinMode = Enum.LineJoinMode.Miter,
                    ApplyStrokeMode = Enum.ApplyStrokeMode.Border
                }):AddToTheme({Color = "Outline"})

                Items["MenuHeader"] = Instances:Create("Frame", {
                    Parent = Items["SettingsMenu"].Instance,
                    Name = "\0",
                    BorderColor3 = FromRGB(42, 49, 45),
                    Size = UDim2New(1, 0, 0, 22),
                    BorderSizePixel = 0,
                    BackgroundColor3 = FromRGB(25, 30, 26),
                    ZIndex = 10
                })  Items["MenuHeader"]:AddToTheme({BackgroundColor3 = "Page Background"})

                Instances:Create("Frame", {
                    Parent = Items["MenuHeader"].Instance,
                    Name = "\0",
                    AnchorPoint = Vector2New(0, 1),
                    Position = UDim2New(0, 0, 1, 0),
                    BorderColor3 = FromRGB(0, 0, 0),
                    Size = UDim2New(1, 0, 0, 1),
                    BorderSizePixel = 0,
                    BackgroundColor3 = FromRGB(42, 49, 45),
                    ZIndex = 10
                }):AddToTheme({BackgroundColor3 = "Outline"})

                Items["HeaderText"] = Instances:Create("TextLabel", {
                    Parent = Items["MenuHeader"].Instance,
                    Name = "\0",
                    FontFace = Library.Font,
                    TextColor3 = FromRGB(235, 235, 235),
                    BorderColor3 = FromRGB(0, 0, 0),
                    Text = Data.Name or "Settings",
                    AnchorPoint = Vector2New(0, 0.5),
                    Size = UDim2New(1, -8, 0, 15),
                    BackgroundTransparency = 1,
                    Position = UDim2New(0, 8, 0.5, 0),
                    BorderSizePixel = 0,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    TextSize = 9,
                    BackgroundColor3 = FromRGB(255, 255, 255),
                    ZIndex = 10
                })  Items["HeaderText"]:AddToTheme({TextColor3 = "Text"})

                Items["HeaderText"]:TextBorder()

                Items["ContentHolder"] = Instances:Create("Frame", {
                    Parent = Items["SettingsMenu"].Instance,
                    Name = "\0",
                    Position = UDim2New(0, 0, 0, 22),
                    BorderColor3 = FromRGB(0, 0, 0),
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 0, 0),
                    BorderSizePixel = 0,
                    AutomaticSize = Enum.AutomaticSize.Y,
                    BackgroundColor3 = FromRGB(255, 255, 255),
                    ZIndex = 10
                })

                Instances:Create("UIPadding", {
                    Parent = Items["ContentHolder"].Instance,
                    Name = "\0",
                    PaddingTop = UDimNew(0, 8),
                    PaddingBottom = UDimNew(0, 8),
                    PaddingRight = UDimNew(0, 8),
                    PaddingLeft = UDimNew(0, 8)
                })

                Instances:Create("UIListLayout", {
                    Parent = Items["ContentHolder"].Instance,
                    Name = "\0",
                    Padding = UDimNew(0, 8),
                    SortOrder = Enum.SortOrder.LayoutOrder
                })
            end

            local Debounce = false
            local RenderStepped

            function Settings:SetOpen(Bool)
                if Debounce then return end
                Settings.IsOpen = Bool
                Debounce = true

                if Settings.IsOpen then
                    Items["SettingsMenu"].Instance.Visible = true
                    Items["SettingsMenu"].Instance.Parent = Library.Holder.Instance
                    Items["GearIcon"]:ChangeItemTheme({ImageColor3 = "Accent"})
                    Items["GearIcon"]:Tween(nil, {ImageColor3 = Library.Theme.Accent})

                    RenderStepped = RunService.RenderStepped:Connect(function()
                        local ButtonPos = Items["SettingsButton"].Instance.AbsolutePosition
                        Items["SettingsMenu"].Instance.Position = UDim2New(0, ButtonPos.X + 20, 0, ButtonPos.Y)
                    end)

                    for Index, Value in Library.OpenFrames do
                        if Value ~= Settings then Value:SetOpen(false) end
                    end
                    Library.OpenFrames[Settings] = Settings
                else
                    if Library.OpenFrames[Settings] then Library.OpenFrames[Settings] = nil end
                    if RenderStepped then RenderStepped:Disconnect() RenderStepped = nil end
                    Items["GearIcon"]:ChangeItemTheme({ImageColor3 = "Placeholder Text"})
                    Items["GearIcon"]:Tween(nil, {ImageColor3 = Library.Theme["Placeholder Text"]})
                end

                local Descendants = Items["SettingsMenu"].Instance:GetDescendants()
                TableInsert(Descendants, Items["SettingsMenu"].Instance)
                local NewTween
                for Index, Value in Descendants do
                    local TransparencyProperty = Tween:GetProperty(Value)
                    if not TransparencyProperty then continue end
                    if type(TransparencyProperty) == "table" then
                        for _, Property in TransparencyProperty do
                            NewTween = Tween:FadeItem(Value, Property, Bool, Library.FadeSpeed)
                        end
                    else
                        NewTween = Tween:FadeItem(Value, TransparencyProperty, Bool, Library.FadeSpeed)
                    end
                end

                NewTween.Tween.Completed:Connect(function()
                    Debounce = false
                    Items["SettingsMenu"].Instance.Visible = Settings.IsOpen
                    task.wait(0.2)
                    Items["SettingsMenu"].Instance.Parent = not Settings.IsOpen and Library.UnusedHolder.Instance or Library.Holder.Instance
                end)
            end

            function Settings:AddToggle(ToggleData)
                local Toggle = { Value = false, Flag = ToggleData.Flag or Library:NextFlag() }
                local ToggleItems = {}

                ToggleItems["Container"] = Instances:Create("Frame", {
                    Parent = Items["ContentHolder"].Instance,
                    Name = "\0",
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 0, 16),
                    ZIndex = 10
                })

                ToggleItems["Button"] = Instances:Create("TextButton", {
                    Parent = ToggleItems["Container"].Instance,
                    Text = "",
                    AutoButtonColor = false,
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 1, 0),
                    ZIndex = 10
                })

                ToggleItems["Indicator"] = Instances:Create("Frame", {
                    Parent = ToggleItems["Button"].Instance,
                    AnchorPoint = Vector2New(0, 0.5),
                    Position = UDim2New(0, 0, 0.5, 0),
                    BorderColor3 = FromRGB(12, 12, 12),
                    Size = UDim2New(0, 10, 0, 10),
                    BorderSizePixel = 1,
                    BackgroundColor3 = FromRGB(30, 36, 31),
                    ZIndex = 10
                })  ToggleItems["Indicator"]:AddToTheme({BackgroundColor3 = "Element", BorderColor3 = "Border"})

                Instances:Create("UIStroke", {
                    Parent = ToggleItems["Indicator"].Instance,
                    Color = FromRGB(42, 49, 45),
                    LineJoinMode = Enum.LineJoinMode.Miter
                }):AddToTheme({Color = "Outline"})

                ToggleItems["Check"] = Instances:Create("ImageLabel", {
                    Parent = ToggleItems["Indicator"].Instance,
                    ImageColor3 = FromRGB(0, 0, 0),
                    ImageTransparency = 1,
                    Image = "rbxassetid://108016671469439",
                    BackgroundTransparency = 1,
                    AnchorPoint = Vector2New(0.5, 0.5),
                    Position = UDim2New(0.5, 0, 0.5, 0),
                    Size = UDim2New(1, 2, 1, 2),
                    ZIndex = 10
                })

                ToggleItems["Text"] = Instances:Create("TextLabel", {
                    Parent = ToggleItems["Button"].Instance,
                    FontFace = Library.Font,
                    TextColor3 = FromRGB(235, 235, 235),
                    Text = ToggleData.Name,
                    Size = UDim2New(1, -20, 0, 15),
                    AnchorPoint = Vector2New(0, 0.5),
                    Position = UDim2New(0, 18, 0.5, 0),
                    BackgroundTransparency = 1,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    TextSize = 9,
                    ZIndex = 10
                })  ToggleItems["Text"]:AddToTheme({TextColor3 = "Text"})
                ToggleItems["Text"]:TextBorder()

                function Toggle:Set(Value)
                    Toggle.Value = Value
                    Library.Flags[Toggle.Flag] = Value
                    if Toggle.Value then
                        ToggleItems["Indicator"]:ChangeItemTheme({BackgroundColor3 = "Accent"})
                        ToggleItems["Indicator"]:Tween(nil, {BackgroundColor3 = Library.Theme.Accent})
                        task.wait(0.05)
                        ToggleItems["Check"]:Tween(TweenInfo.new(Library.Tween.Time, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {ImageTransparency = 0, Size = UDim2New(1, 2, 1, 2)})
                    else
                        ToggleItems["Indicator"]:ChangeItemTheme({BackgroundColor3 = "Element"})
                        ToggleItems["Indicator"]:Tween(nil, {BackgroundColor3 = Library.Theme.Element})
                        task.wait(0.05)
                        ToggleItems["Check"]:Tween(TweenInfo.new(Library.Tween.Time, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {ImageTransparency = 1, Size = UDim2New(0, 0, 0, 0)})
                    end
                    if ToggleData.Callback then Library:SafeCall(ToggleData.Callback, Toggle.Value) end
                end

                ToggleItems["Button"]:Connect("MouseButton1Down", function() Toggle:Set(not Toggle.Value) end)
                Toggle:Set(ToggleData.Default or false)
                Library.SetFlags[Toggle.Flag] = function(Value) Toggle:Set(Value) end
                TableInsert(Settings.Items, Toggle)
                return Toggle
            end

            function Settings:AddSlider(SliderData)
                local Slider = { Value = 0, Flag = SliderData.Flag or Library:NextFlag(), Sliding = false }
                local SliderItems = {}

                SliderItems["Container"] = Instances:Create("Frame", {
                    Parent = Items["ContentHolder"].Instance,
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 0, 24),
                    ZIndex = 10
                })

                SliderItems["Header"] = Instances:Create("Frame", {
                    Parent = SliderItems["Container"].Instance,
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 0, 12),
                    ZIndex = 10
                })

                SliderItems["Text"] = Instances:Create("TextLabel", {
                    Parent = SliderItems["Header"].Instance,
                    FontFace = Library.Font,
                    TextColor3 = FromRGB(235, 235, 235),
                    Text = SliderData.Name,
                    BackgroundTransparency = 1,
                    Size = UDim2New(0, 0, 0, 12),
                    AutomaticSize = Enum.AutomaticSize.X,
                    TextSize = 9,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 10
                })  SliderItems["Text"]:AddToTheme({TextColor3 = "Text"})
                SliderItems["Text"]:TextBorder()

                SliderItems["Value"] = Instances:Create("TextLabel", {
                    Parent = SliderItems["Header"].Instance,
                    FontFace = Library.Font,
                    TextColor3 = FromRGB(235, 235, 235),
                    Text = "0",
                    AnchorPoint = Vector2New(1, 0),
                    Size = UDim2New(0, 0, 0, 12),
                    BackgroundTransparency = 1,
                    Position = UDim2New(1, 0, 0, 0),
                    AutomaticSize = Enum.AutomaticSize.X,
                    TextSize = 9,
                    TextXAlignment = Enum.TextXAlignment.Right,
                    ZIndex = 10
                })  SliderItems["Value"]:AddToTheme({TextColor3 = "Text"})
                SliderItems["Value"]:TextBorder()

                SliderItems["SliderBar"] = Instances:Create("TextButton", {
                    Parent = SliderItems["Container"].Instance,
                    AutoButtonColor = false,
                    Text = "",
                    AnchorPoint = Vector2New(0, 1),
                    Position = UDim2New(0, 0, 1, 0),
                    BorderColor3 = FromRGB(12, 12, 12),
                    Size = UDim2New(1, 0, 0, 8),
                    BorderSizePixel = 1,
                    BackgroundColor3 = FromRGB(30, 36, 31),
                    ZIndex = 10
                })  SliderItems["SliderBar"]:AddToTheme({BackgroundColor3 = "Element", BorderColor3 = "Border"})

                Instances:Create("UIStroke", {
                    Parent = SliderItems["SliderBar"].Instance,
                    Color = FromRGB(42, 49, 45),
                    LineJoinMode = Enum.LineJoinMode.Miter
                }):AddToTheme({Color = "Outline"})

                SliderItems["Accent"] = Instances:Create("Frame", {
                    Parent = SliderItems["SliderBar"].Instance,
                    Size = UDim2New(0.5, 0, 1, 0),
                    BackgroundColor3 = FromRGB(202, 243, 255),
                    ZIndex = 10
                })  SliderItems["Accent"]:AddToTheme({BackgroundColor3 = "Accent"})

                SliderItems["Dragger"] = Instances:Create("Frame", {
                    Parent = SliderItems["Accent"].Instance,
                    AnchorPoint = Vector2New(1, 0.5),
                    Position = UDim2New(1, 0, 0.5, 0),
                    BorderColor3 = FromRGB(42, 49, 45),
                    Size = UDim2New(0, 2, 1, 2),
                    BorderSizePixel = 1,
                    BackgroundColor3 = FromRGB(14, 17, 15),
                    ZIndex = 10
                })  SliderItems["Dragger"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Outline"})

                Instances:Create("UIStroke", {
                    Parent = SliderItems["Dragger"].Instance,
                    Color = FromRGB(12, 12, 12),
                    LineJoinMode = Enum.LineJoinMode.Miter
                }):AddToTheme({Color = "Border"})

                function Slider:Set(Value)
                    Slider.Value = Library:Round(MathClamp(Value, SliderData.Min, SliderData.Max), SliderData.Decimals or 0)
                    Library.Flags[Slider.Flag] = Slider.Value
                    SliderItems["Accent"]:Tween(TweenInfo.new(Library.Tween.Time, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2New((Slider.Value - SliderData.Min) / (SliderData.Max - SliderData.Min), 0, 1, 0)})
                    SliderItems["Value"].Instance.Text = StringFormat("%s%s", tostring(Slider.Value), SliderData.Suffix or "")
                    if SliderData.Callback then Library:SafeCall(SliderData.Callback, Slider.Value) end
                end

                local InputChanged
                SliderItems["SliderBar"]:Connect("InputBegan", function(Input)
                    if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
                        Slider.Sliding = true
                        local SizeX = (Mouse.X - SliderItems["SliderBar"].Instance.AbsolutePosition.X) / SliderItems["SliderBar"].Instance.AbsoluteSize.X
                        Slider:Set(((SliderData.Max - SliderData.Min) * SizeX) + SliderData.Min)
                        if InputChanged then return end
                        InputChanged = Input.Changed:Connect(function()
                            if Input.UserInputState == Enum.UserInputState.End then
                                Slider.Sliding = false
                                InputChanged:Disconnect()
                                InputChanged = nil
                            end
                        end)
                    end
                end)

                Library:Connect(UserInputService.InputChanged, function(Input)
                    if Input.UserInputType == Enum.UserInputType.MouseMovement or Input.UserInputType == Enum.UserInputType.Touch then
                        if Slider.Sliding then
                            local SizeX = (Mouse.X - SliderItems["SliderBar"].Instance.AbsolutePosition.X) / SliderItems["SliderBar"].Instance.AbsoluteSize.X
                            Slider:Set(((SliderData.Max - SliderData.Min) * SizeX) + SliderData.Min)
                        end
                    end
                end)

                Slider:Set(SliderData.Default or SliderData.Min)
                Library.SetFlags[Slider.Flag] = function(Value) Slider:Set(Value) end
                TableInsert(Settings.Items, Slider)
                return Slider
            end

            function Settings:AddLabel(LabelData)
                local Label = {}
                local LabelItems = {}

                LabelItems["Container"] = Instances:Create("Frame", {
                    Parent = Items["ContentHolder"].Instance,
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 0, 14),
                    ZIndex = 10
                })

                LabelItems["Text"] = Instances:Create("TextLabel", {
                    Parent = LabelItems["Container"].Instance,
                    FontFace = Library.Font,
                    TextColor3 = FromRGB(235, 235, 235),
                    Text = LabelData.Text or "Label",
                    Size = UDim2New(1, 0, 0, 14),
                    BackgroundTransparency = 1,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    TextSize = 9,
                    ZIndex = 10
                })  LabelItems["Text"]:AddToTheme({TextColor3 = "Text"})
                LabelItems["Text"]:TextBorder()

                function Label:SetText(Text) LabelItems["Text"].Instance.Text = tostring(Text) end
                TableInsert(Settings.Items, Label)
                return Label
            end

            function Settings:AddDivider()
                local Divider = {}
                local DividerItems = {}

                DividerItems["Container"] = Instances:Create("Frame", {
                    Parent = Items["ContentHolder"].Instance,
                    BackgroundTransparency = 1,
                    Size = UDim2New(1, 0, 0, 1),
                    ZIndex = 10
                })

                DividerItems["Line"] = Instances:Create("Frame", {
                    Parent = DividerItems["Container"].Instance,
                    Size = UDim2New(1, 0, 1, 0),
                    BackgroundColor3 = FromRGB(42, 49, 45),
                    ZIndex = 10
                })  DividerItems["Line"]:AddToTheme({BackgroundColor3 = "Outline"})

                TableInsert(Settings.Items, Divider)
                return Divider
            end

            Items["SettingsButton"]:Connect("MouseButton1Down", function() Settings:SetOpen(not Settings.IsOpen) end)

            Items["SettingsButton"]:OnHover(function()
                if not Settings.IsOpen then
                    Items["GearIcon"]:ChangeItemTheme({ImageColor3 = "Text"})
                    Items["GearIcon"]:Tween(nil, {ImageColor3 = Library.Theme.Text})
                end
            end)

            Items["SettingsButton"]:OnHoverLeave(function()
                if not Settings.IsOpen then
                    Items["GearIcon"]:ChangeItemTheme({ImageColor3 = "Placeholder Text"})
                    Items["GearIcon"]:Tween(nil, {ImageColor3 = Library.Theme["Placeholder Text"]})
                end
            end)

            Library:Connect(UserInputService.InputBegan, function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
                    if not Settings.IsOpen then return end
                    if Library:IsMouseOverFrame(Items["SettingsMenu"]) or Library:IsMouseOverFrame(Items["SettingsButton"]) then return end
                    Settings:SetOpen(false)
                end
            end)

            return Settings
        end

        -- INTEGRACIÓN CON TOGGLE
        Components.Toggle.AddSettings = function(self, Data)
            return Components:Settings({
                Name = Data.Name,
                Parent = self.Items["SubElements"]
            })
        end

        -- INTEGRACIÓN CON LABEL
        Components.Label.AddSettings = function(self, Data)
            return Components:Settings({
                Name = Data.Name,
                Parent = self.Items["SubElements"]
            })
        end

    end

    -- Library components

    Library.Watermark = function(self, Name)
        local Watermark = { }

        local Items = { } do 
            Items["Watermark"] = Instances:Create("Frame", {
                Parent = Library.Holder.Instance,
                Name = "\0",
                AnchorPoint = Vector2New(0.5, 1),
                Position = UDim2New(0.5, 0, 1, -12),
                BorderColor3 = FromRGB(12, 12, 12),
                BorderSizePixel = 2,
                AutomaticSize = Enum.AutomaticSize.XY,
                BackgroundColor3 = FromRGB(14, 17, 15)
            })  Items["Watermark"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Border"})

            Items["Watermark"]:MakeDraggable()

            Instances:Create("UIStroke", {
                Parent = Items["Watermark"].Instance,
                Name = "\0",
                Color = FromRGB(42, 49, 45),
                LineJoinMode = Enum.LineJoinMode.Miter,
                ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            }):AddToTheme({Color = "Outline"})

            Instances:Create("UIPadding", {
                Parent = Items["Watermark"].Instance,
                Name = "\0",
                PaddingTop = UDimNew(0, 5),
                PaddingBottom = UDimNew(0, 7),
                PaddingRight = UDimNew(0, 5),
                PaddingLeft = UDimNew(0, 5)
            })

            Items["Text"] = Instances:Create("TextLabel", {
                Parent = Items["Watermark"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = Name,
                Position = UDim2New(0, 0, 0, 2),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.XY,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Text"]:AddToTheme({TextColor3 = "Text"})

            Items["Text"]:TextBorder()

            Items["Liner"] = Instances:Create("Frame", {
                Parent = Items["Watermark"].Instance,
                Name = "\0",
                Position = UDim2New(0, -5, 0, -5),
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 10, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(202, 243, 255)
            })  Items["Liner"]:AddToTheme({BackgroundColor3 = "Accent"})
        end

        function Watermark:SetVisibility(Bool)
            Items["Watermark"].Instance.Visible = Bool
        end

        return Watermark
    end

    Library.KeybindList = function(self)
        local KeybindList = { }
        Library.KeyList = KeybindList

        local Items = { } do
            Items["KeybindList"] = Instances:Create("Frame", {
                Parent = Library.Holder.Instance,
                Name = "\0",
                AnchorPoint = Vector2New(0, 0.5),
                Position = UDim2New(0, 12, 0.5, 55),
                BorderColor3 = FromRGB(12, 12, 12),
                Size = UDim2New(0, 116, 0, 32),
                BorderSizePixel = 2,
                BackgroundColor3 = FromRGB(14, 17, 15)
            })  Items["KeybindList"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Border"})

            Items["KeybindList"]:MakeDraggable()

            Instances:Create("UIStroke", {
                Parent = Items["KeybindList"].Instance,
                Name = "\0",
                Color = FromRGB(42, 49, 45),
                LineJoinMode = Enum.LineJoinMode.Miter,
                ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            }):AddToTheme({Color = "Outline"})

            Items["Title"] = Instances:Create("TextLabel", {
                Parent = Items["KeybindList"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = "Keybinds",
                Size = UDim2New(0, 0, 0, 20),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 0, 0, -4),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Title"]:AddToTheme({TextColor3 = "Text"})

            Items["Title"]:TextBorder()

            Instances:Create("UIPadding", {
                Parent = Items["KeybindList"].Instance,
                Name = "\0",
                PaddingTop = UDimNew(0, 8),
                PaddingBottom = UDimNew(0, 8),
                PaddingRight = UDimNew(0, 8),
                PaddingLeft = UDimNew(0, 8)
            })

            Items["Liner"] = Instances:Create("Frame", {
                Parent = Items["KeybindList"].Instance,
                Name = "\0",
                Position = UDim2New(0, 0, 0, 15),
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 0, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(202, 243, 255)
            })  Items["Liner"]:AddToTheme({BackgroundColor3 = "Accent"})

            Items["Content"] = Instances:Create("Frame", {
                Parent = Items["KeybindList"].Instance,
                Name = "\0",
                BorderColor3 = FromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 0, 0, 32),
                Size = UDim2New(1, 0, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })

            Instances:Create("UIListLayout", {
                Parent = Items["Content"].Instance,
                Name = "\0",
                SortOrder = Enum.SortOrder.LayoutOrder
            })
        end

        function KeybindList:Add(Key, Name, Mode)
            local NewKey = Instances:Create("TextLabel", {
                Parent = Items["Content"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = "" ..Key .." - " ..Name .. " ("..Mode..")",
                BackgroundTransparency = 1,
                Size = UDim2New(0, 0, 0, 15),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextTransparency = 1,
                Visible = false,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  NewKey:AddToTheme({TextColor3 = "Text"})

            NewKey:TextBorder()

            function NewKey:SetText(Key, Name, Mode)
                NewKey.Instance.Text = "" ..Key .." - " ..Name .. " ("..Mode..")"
            end

            function NewKey:SetStatus(Bool)
                if Bool then
                    NewKey.Instance.Visible = true
                    NewKey:Tween(nil, {TextTransparency = 0})
                else
                    NewKey:Tween(nil, {TextTransparency = 1}).Tween.Completed:Connect(function()
                        NewKey.Instance.Visible = false
                    end)
                end
            end

            return NewKey
        end

        function KeybindList:SetVisibility(Bool)
            Items["KeybindList"].Instance.Visible = Bool
        end

        return KeybindList
    end

    Library.Notification = function(self, Title, Description, Duration)
        local Items = { } do 
            Items["Notification"] = Instances:Create("Frame", {
                Parent = Library.NotifHolder.Instance,
                Name = "\0",
                Size = UDim2New(0, 0, 0, 25),
                BorderColor3 = FromRGB(12, 12, 12),
                BorderSizePixel = 2,
                AutomaticSize = Enum.AutomaticSize.XY,
                BackgroundColor3 = FromRGB(14, 17, 15)
            })  Items["Notification"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Border"})

            Items["UIStroke1"] = Instances:Create("UIStroke", {
                Parent = Items["Notification"].Instance,
                Name = "\0",
                Color = FromRGB(42, 49, 45),
                LineJoinMode = Enum.LineJoinMode.Miter,
                ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            })  Items["UIStroke1"]:AddToTheme({Color = "Outline"})

            Instances:Create("UIPadding", {
                Parent = Items["Notification"].Instance,
                Name = "\0",
                PaddingTop = UDimNew(0, 5),
                PaddingBottom = UDimNew(0, 12),
                PaddingRight = UDimNew(0, 5),
                PaddingLeft = UDimNew(0, 5)
            })

            Items["Title"] = Instances:Create("TextLabel", {
                Parent = Items["Notification"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = Title,
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.XY,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Title"]:AddToTheme({TextColor3 = "Text"})

           Items["UIStroke2"] =  Items["Title"]:TextBorder()

            Items["Description"] = Instances:Create("TextLabel", {
                Parent = Items["Notification"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                TextTransparency = 0.4000000059604645,
                Text = Description,
                Position = UDim2New(0, 0, 0, 15),
                BorderSizePixel = 0,
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                BorderColor3 = FromRGB(0, 0, 0),
                AutomaticSize = Enum.AutomaticSize.XY,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Description"]:AddToTheme({TextColor3 = "Text"})

            Items["UIStroke3"] = Items["Description"]:TextBorder()

            Items["Liner"] = Instances:Create("Frame", {
                Parent = Items["Notification"].Instance,
                Name = "\0",
                Position = UDim2New(0, 0, 1, 8),
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 0, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(202, 243, 255)
            })  Items["Liner"]:AddToTheme({BackgroundColor3 = "Accent"})
        end

        local Size = Items["Notification"].Instance.AbsoluteSize

        for Index, Value in Items do 
            if Value.Instance:IsA("Frame") then
                Value.Instance.BackgroundTransparency = 1
            elseif Value.Instance:IsA("TextLabel") then 
                Value.Instance.TextTransparency = 1
            elseif Value.Instance:IsA("UIStroke") then
                Value.Instance.Transparency = 1
            end
        end 

        Items["Notification"].Instance.AutomaticSize = Enum.AutomaticSize.Y

        Library:Thread(function()
            for Index, Value in Items do 
                if Value.Instance:IsA("Frame") then
                    Value:Tween(nil, {BackgroundTransparency = 0})
                elseif Value.Instance:IsA("TextLabel") and Index ~= "Description" then 
                    Value:Tween(nil, {TextTransparency = 0})
                elseif Value.Instance:IsA("TextLabel") and Index == "Description" then 
                    Value:Tween(nil, {TextTransparency = 0.4})
                elseif Value.Instance:IsA("UIStroke") then
                    Value:Tween(nil, {Transparency = 0})
                end
            end

            Items["Notification"]:Tween(nil, {Size = UDim2New(0, Size.X, 0, 0)})
            Items["Liner"]:Tween(TweenInfo.new(Duration, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2New(0, 0, 0, 1)})
            
            task.delay(Duration + 0.1, function()
                for Index, Value in Items do 
                    if Value.Instance:IsA("Frame") then
                        Value:Tween(nil, {BackgroundTransparency = 1})
                    elseif Value.Instance:IsA("TextLabel") then 
                        Value:Tween(nil, {TextTransparency = 1})
                    elseif Value.Instance:IsA("UIStroke") then
                        Value:Tween(nil, {Transparency = 1})
                    end
                end

                Items["Notification"]:Tween(nil, {Size = UDim2New(0, 0, 0, 0)})
                task.wait(0.5)
                Items["Notification"]:Clean()
            end)
        end)
    end

    Library.InventoryViewer = function(self)
        local Viewer = { }
        Viewer.Items = { } 

        local Items = { } do
            Items["InventoryViewer"] = Instances:Create("Frame", {
                Parent = Library.Holder.Instance,
                Name = "\0",
                Position = UDim2New(0.007766990456730127, 0, 0.11442785710096359, 0),
                BorderColor3 = FromRGB(12, 12, 12),
                Size = UDim2New(0, 325, 0, 277),
                BorderSizePixel = 2,
                BackgroundColor3 = FromRGB(14, 17, 15)
            })  Items["InventoryViewer"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Border"})

            Instances:Create("UIStroke", {
                Parent = Items["InventoryViewer"].Instance,
                Name = "\0",
                Color = FromRGB(42, 49, 45),
                LineJoinMode = Enum.LineJoinMode.Miter,
                ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            }):AddToTheme({Color = "Outline"})

            Items["Title"] = Instances:Create("TextLabel", {
                Parent = Items["InventoryViewer"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = "Inventory",
                Size = UDim2New(0, 0, 0, 15),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 8, 0, 4),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Title"]:AddToTheme({TextColor3 = "Text"})

            Items["Tools"] = Instances:Create("Frame", {
                Parent = Items["InventoryViewer"].Instance,
                Name = "\0",
                Position = UDim2New(0, 8, 0, 27),
                BorderColor3 = FromRGB(42, 49, 45),
                Size = UDim2New(1, -16, 1, -108),
                BorderSizePixel = 2,
                BackgroundColor3 = FromRGB(20, 24, 21)
            })  Items["Tools"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Outline"})

            Instances:Create("UIStroke", {
                Parent = Items["Tools"].Instance,
                Name = "\0",
                Color = FromRGB(12, 12, 12),
                LineJoinMode = Enum.LineJoinMode.Miter,
                ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            }):AddToTheme({Color = "Border"})

            Items["Holder"] = Instances:Create("ScrollingFrame", {
                Parent = Items["Tools"].Instance,
                Name = "\0",
                Active = true,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                BorderSizePixel = 0,
                CanvasSize = UDim2New(0, 0, 0, 0),
                ScrollBarImageColor3 = FromRGB(202, 243, 255),
                MidImage = "rbxassetid://123708228368098",
                BorderColor3 = FromRGB(0, 0, 0),
                ScrollBarThickness = 2,
                Size = UDim2New(1, -4, 1, -8),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 0, 0, 4),
                BottomImage = "rbxassetid://123708228368098",
                TopImage = "rbxassetid://123708228368098",
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Holder"]:AddToTheme({ScrollBarImageColor3 = "Accent"})

            Instances:Create("UIGridLayout", {
                Parent = Items["Holder"].Instance,
                Name = "\0",
                SortOrder = Enum.SortOrder.LayoutOrder,
                CellSize = UDim2New(0, 65, 0, 65)
            })

            Instances:Create("UIPadding", {
                Parent = Items["Holder"].Instance,
                Name = "\0",
                PaddingTop = UDimNew(0, 4),
                PaddingLeft = UDimNew(0, 8)
            })

            Items["PlayerAvatar"] = Instances:Create("ImageLabel", {
                Parent = Items["InventoryViewer"].Instance,
                Name = "\0",
                BorderColor3 = FromRGB(42, 49, 45),
                AnchorPoint = Vector2New(0, 1),
                Image = "rbxasset://textures/ui/GuiImagePlaceholder.png",
                Position = UDim2New(0, 8, 1, -8),
                Size = UDim2New(0, 60, 0, 60),
                BorderSizePixel = 2,
                BackgroundColor3 = FromRGB(14, 17, 15)
            })  Items["PlayerAvatar"]:AddToTheme({BackgroundColor3 = "Background", BorderColor3 = "Outline"})

            Instances:Create("UIStroke", {
                Parent = Items["PlayerAvatar"].Instance,
                Name = "\0",
                Color = FromRGB(12, 12, 12),
                LineJoinMode = Enum.LineJoinMode.Miter,
                ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            }):AddToTheme({Color = "Outline"})
            
            Items["PlayerHealth"] = Instances:Create("TextLabel", {
                Parent = Items["InventoryViewer"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = "Health: ",
                AnchorPoint = Vector2New(0, 1),
                Size = UDim2New(0, 0, 0, 15),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 75, 1, -55),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["PlayerHealth"]:AddToTheme({TextColor3 = "Text"})

            Items["PlayerDistance"] = Instances:Create("TextLabel", {
                Parent = Items["InventoryViewer"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = "Distance:  studs",
                AnchorPoint = Vector2New(0, 1),
                Size = UDim2New(0, 0, 0, 15),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 75, 1, -35),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["PlayerDistance"]:AddToTheme({TextColor3 = "Text"})
        end

        function Viewer:SetPlayerHealth(Value)
            Items["PlayerHealth"].Instance.Text = tostring(Value)
        end

        function Viewer:SetPlayerDistance(Value)
            Items["PlayerDistance"].Instance.Text = "Distance: "..tostring(Value).." studs"
        end

        function Viewer:SetPlayer(Value)
            local PlayerAvatar, _ = Players:GetUserThumbnailAsync(Value.UserId)
            Items["PlayerAvatar"].Instance.Image = PlayerAvatar
            Items["Title"].Instance.Text = Value.Name .. "'s Inventory"
        end

        function Viewer:AddTool(Name, Image)
            local NewItem = { }

            local SubItems = { } do
                SubItems["Item"] = Instances:Create("Frame", {
                    Parent = Items["Holder"].Instance,
                    Name = "\0",
                    BorderColor3 = FromRGB(12, 12, 12),
                    Size = UDim2New(0, 100, 0, 100),
                    BorderSizePixel = 2,
                    BackgroundColor3 = FromRGB(20, 24, 21)
                })  SubItems["Item"]:AddToTheme({BackgroundColor3 = "Inline", BorderColor3 = "Border"})

                Instances:Create("UIStroke", {
                    Parent = SubItems["Item"].Instance,
                    Name = "\0",
                    Color = FromRGB(42, 49, 45),
                    LineJoinMode = Enum.LineJoinMode.Miter,
                    ApplyStrokeMode = Enum.ApplyStrokeMode.Border
                }):AddToTheme({Color = "Outline"})

                SubItems["Image"] = Instances:Create("ImageLabel", {
                    Parent = SubItems["Item"].Instance,
                    Name = "\0",
                    ImageColor3 = FromRGB(202, 243, 255),
                    ScaleType = Enum.ScaleType.Fit,
                    BorderColor3 = FromRGB(0, 0, 0),
                    AnchorPoint = Vector2New(0.5, 0.5),
                    Image = "rbxassetid://"..Image,
                    BackgroundTransparency = 1,
                    Position = UDim2New(0.5, 0, 0.5, 0),
                    Size = UDim2New(0, 45, 0, 45),
                    BorderSizePixel = 0,
                    BackgroundColor3 = FromRGB(255, 255, 255)
                })  SubItems["Image"]:AddToTheme({ImageColor3 = "Accent"})
            end

            function NewItem:Remove()
                Viewer.Items[Name] = nil
                SubItems["Item"]:Clean()
            end

            Viewer.Items[Name] = NewItem
            return NewItem
        end

        function Viewer:RemoveAllTools()
            for Index, Value in Viewer.Items do 
                Value:Remove()
            end
        end

        return Viewer
    end

    Library.Window = function(self, Data)
        Data = Data or { }

        local Window = { 
            Logo = Data.Logo or Data.logo or "",
            FadeTime = Data.FadeTime or Data.fadetime or 0.4,
            Size = Data.Size or Data.size or UDim2New(0, 751, 0, 539),

            Pages = { },
            Items = { },

            IsOpen = false,
        }

        local Items = Components:Window({
            Parent = Library.Holder,
            Draggable = true,
            Resizeable = true,
            AnchorPoint = Vector2New(0, 0),
            Position = UDim2New(0, Camera.ViewportSize.X / 3.3, 0, Camera.ViewportSize.Y / 3.3),
            Size = Window.Size
        }) do
            Items["Side"] = Instances:Create("Frame", {
                Parent = Items["Window"].Instance,
                Name = "\0",
                Position = UDim2New(0, 12, 0, 12),
                BorderColor3 = FromRGB(42, 49, 45),
                Size = UDim2New(0, 200, 1, -24),
                BorderSizePixel = 2,
                BackgroundColor3 = FromRGB(20, 24, 21)
            })  Items["Side"]:AddToTheme({BackgroundColor3 = "Inline", BorderColor3 = "Outline"})
            
            Items["Side"]:Border("Border")

            Items["Window"].Instance.Visible = false

            Items["Logo"] = Instances:Create("ImageLabel", {
                Parent = Items["Side"].Instance,
                Name = "\0",
                ImageColor3 = FromRGB(202, 243, 255),
                ScaleType = Enum.ScaleType.Fit,
                BorderColor3 = FromRGB(0, 0, 0),
                AnchorPoint = Vector2New(0.5, 0),
                Image = "rbxassetid://" .. Window.Logo,
                BackgroundTransparency = 1,
                Position = UDim2New(0.5, 0, 0, 12),
                Size = UDim2New(0, 75, 0, 75),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Logo"]:AddToTheme({ImageColor3 = "Accent"})

            Items["Pages"] = Instances:Create("Frame", {
                Parent = Items["Side"].Instance,
                Name = "\0",
                BackgroundTransparency = 1,
                Position = UDim2New(0, 0, 0, 100),
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 0, 1, -135),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })

            Instances:Create("UIPadding", {
                Parent = Items["Pages"].Instance,
                Name = "\0",
                PaddingRight = UDimNew(0, 8),
                PaddingLeft = UDimNew(0, 8)
            })

            Instances:Create("UIListLayout", {
                Parent = Items["Pages"].Instance,
                Name = "\0",
                Padding = UDimNew(0, 8),
                SortOrder = Enum.SortOrder.LayoutOrder
            })

            Items["Content"] = Instances:Create("Frame", {
                Parent = Items["Window"].Instance,
                Name = "\0",
                BackgroundTransparency = 1,
                Position = UDim2New(0, 226, 0, 12),
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, -238, 1, -24),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })

            Items["MouseBackground"] = Instances:Create("Frame", {
                Parent = Library.Holder.Instance,
                Name = "\0",
                BackgroundTransparency = 1,
                Position = UDim2New(0, 0, 0, 0),
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(0, 16, 0, 16),
                BorderSizePixel = 0,
                ZIndex = 9999,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })

            Items["MouseImage"] = Instances:Create("ImageLabel", {
                Parent = Items["MouseBackground"].Instance,
                Name = "\0",
                BorderColor3 = FromRGB(0, 0, 0),
                Image = "rbxassetid://76631660114196",
                BackgroundTransparency = 1,
                Size = UDim2New(1, 0, 1, 0),
                BorderSizePixel = 0,
                ZIndex = 9999,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["MouseImage"]:AddToTheme({ImageColor3 = "Accent"})

            Instances:Create("UIGradient", {
                Parent = Items["MouseImage"].Instance,
                Name = "\0",
                Rotation = 90,
                Color = RGBSequence{RGBSequenceKeypoint(0, FromRGB(255, 255, 255)), RGBSequenceKeypoint(1, FromRGB(99, 108, 117))}
            })

            UserInputService.MouseIconEnabled = false

            Window.Items = Items
        end

        local Debounce = false

        Library:Connect(RunService.RenderStepped, function()
            local MouseLocation = UserInputService:GetMouseLocation() 
            Items["MouseBackground"].Instance.Position = UDim2New(0, MouseLocation.X - 1, 0, MouseLocation.Y - 56)           
        end)

        local OldSizes = { }

        function Window:AddToOldSizes(Item, Size)
            if not OldSizes[Item] then
                OldSizes[Item] = Size
            end
        end

        function Window:GetOldSize(Item)
            if OldSizes[Item] then
                return OldSizes[Item]
            end
        end

        function Window:SetOpen(Bool)
            if Debounce then 
                return
            end

            Window.IsOpen = Bool

            Debounce = true 

            if Window.IsOpen then 
                Items["Window"].Instance.Visible = true 
            end

            local Descendants = Items["Window"].Instance:GetDescendants()
            TableInsert(Descendants, Items["Window"].Instance)

            local NewTween

            for Index, Value in Descendants do 
                local TransparencyProperty = Tween:GetProperty(Value)

                if not TransparencyProperty then
                    continue 
                end

                if type(TransparencyProperty) == "table" then 
                    for _, Property in TransparencyProperty do 
                        NewTween = Tween:FadeItem(Value, Property, Bool, Library.FadeSpeed)
                    end
                else
                    NewTween = Tween:FadeItem(Value, TransparencyProperty, Bool, Library.FadeSpeed)
                end
            end
            
            NewTween.Tween.Completed:Connect(function()
                Debounce = false 
                Items["Window"].Instance.Visible = Window.IsOpen
                if Window.IsOpen then
                    Items["MouseBackground"].Instance.Visible = true
                    UserInputService.MouseIconEnabled = false
                else
                    Items["MouseBackground"].Instance.Visible = false
                    UserInputService.MouseIconEnabled = true
                end
            end)
        end

        Library:Connect(UserInputService.InputBegan, function(Input)
            if tostring(Input.KeyCode) == Library.MenuKeybind or tostring(Input.UserInputType) == Library.MenuKeybind then
                Window:SetOpen(not Window.IsOpen)
            end
        end)

        Window:SetOpen(true)
        return setmetatable(Window, self)
    end

Library.Page = function(self, Data)
    Data = Data or { }

    local Page = {
        Window = self,

        Name = Data.Name or Data.name or "Page",
        Icon = Data.Icon or Data.icon or nil, -- NUEVO
        Columns = Data.Columns or Data.columns or 2,
        SubPages = Data.SubPages or Data.subpages or false,
    }

    Library.SearchItems[Page] = { }

    local NewPage, Items = Components:WindowPage({
        Name = Page.Name,
        Icon = Page.Icon, -- NUEVO
        ContentHolder = Page.Window.Items["Content"],
        Stack = Page.Window.Pages,
        Parent = Page.Window.Items["Pages"],
        Columns = Page.Columns,
        SubPages = Page.SubPages,
        FadeTime = Page.Window.FadeTime,
        Window = Page.Window
    })

    return setmetatable(NewPage, Library.Pages)
end
Library.Pages.SubPage = function(self, Data)
    Data = Data or { }

    local SubPage = {
        Window = self.Window,
        Page = self,

        Name = Data.Name or Data.name or "SubPage",
        Icon = Data.Icon or Data.icon or nil, -- NUEVO
        Columns = Data.Columns or Data.columns or 2,
    }

    Library.SearchItems[SubPage] = { }

    local NewSubPage, Items = Components:WindowSubPage({
        Page = SubPage.Page,
        Name = SubPage.Name,
        Icon = SubPage.Icon, -- NUEVO
        Columns = SubPage.Columns,
        Window = SubPage.Page.Window
    })

    return setmetatable(NewSubPage, Library.Pages)
end

    Library.Pages.Section = function(self, Data)
        Data = Data or { }

        local Section = {
            Window = self.Window,
            Page = self,

            Name = Data.Name or Data.name or "Section",
            Side = Data.Side or Data.side or 1,

            Items = { }
        }

        local Items = { } do
            Items["Section"] = Instances:Create("Frame", {
                Parent = Section.Page.ColumnsData[Section.Side].Instance,
                Name = "\0",
                Size = UDim2New(1, 0, 0, 25),
                BorderColor3 = FromRGB(42, 49, 45),
                BorderSizePixel = 2,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = FromRGB(20, 24, 21)
            })  Items["Section"]:AddToTheme({BackgroundColor3 = "Inline", BorderColor3 = "Outline"})

            Items["Section"]:Border("Border")

            Items["Liner"] = Instances:Create("Frame", {
                Parent = Items["Section"].Instance,
                Name = "\0",
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 0, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(202, 243, 255)
            })  Items["Liner"]:AddToTheme({BackgroundColor3  = "Accent"})

            Items["Glow"] = Instances:Create("Frame", {
                Parent = Items["Section"].Instance,
                Name = "\0",
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 0, 0, 15),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(202, 243, 255)
            })  Items["Glow"]:AddToTheme({BackgroundColor3  = "Accent"})

            Instances:Create("UIGradient", {
                Parent = Items["Glow"].Instance,
                Name = "\0",
                Rotation = 90,
                Transparency = NumSequence{NumSequenceKeypoint(0, 0), NumSequenceKeypoint(0.193, 0.8687499761581421), NumSequenceKeypoint(0.504, 0.96875), NumSequenceKeypoint(1, 1)}
            })

            Items["Text"] = Instances:Create("TextLabel", {
                Parent = Items["Section"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = Section.Name,
                Size = UDim2New(0, 0, 0, 15),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 6, 0, 5),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Text"]:AddToTheme({TextColor3 = "Text"})

            Items["Text"]:TextBorder()

            Instances:Create("UIPadding", {
                Parent = Items["Section"].Instance,
                Name = "\0",
                PaddingBottom = UDimNew(0, 8)
            })

            Items["Content"] = Instances:Create("Frame", {
                Parent = Items["Section"].Instance,
                Name = "\0",
                BorderColor3 = FromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                Position = UDim2New(0, 10, 0, 26),
                Size = UDim2New(1, -20, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })

            Instances:Create("UIListLayout", {
                Parent = Items["Content"].Instance,
                Name = "\0",
                Padding = UDimNew(0, 8),
                SortOrder = Enum.SortOrder.LayoutOrder
            })

            Section.Items = Items
        end

        return setmetatable(Section, Library.Sections)
    end
    
    Library.Sections.Toggle = function(self, Data)
        Data = Data or { }

        local Toggle = {
            Window = self.Window,
            Page = self.Page,
            Section = self,

            Name = Data.Name or Data.name or "Toggle",
            Flag = Data.Flag or Data.flag or Library:NextFlag(),
            Default = Data.Default or Data.default or false,
            Callback = Data.Callback or Data.callback or function() end
        }

        local NewToggle, ToggleItems = Components:Toggle({
            Name = Toggle.Name,
            Parent = Toggle.Section.Items["Content"],
            Flag = Toggle.Flag,
            Default = Toggle.Default,
            Page = Toggle.Page,
            Callback = Toggle.Callback
        })

        function NewToggle:Colorpicker(Data)
            local Colorpicker = {
                Window = self.Window,
                Page = self.Page,
                Section = self,

                Flag = Data.Flag or Data.flag or Library:NextFlag(),
                Default = Data.Default or Data.default or Color3.fromRGB(255, 255, 255),
                Callback = Data.Callback or Data.callback or function() end,
                Alpha = Data.Alpha or Data.alpha or 0,
            }

            local NewColorpicker, ColorpickerItems = Components:Colorpicker({
                Name = Colorpicker.Name,
                Parent = ToggleItems["SubElements"],
                Pages = true,
                Page = Colorpicker.Page,
                Flag = Colorpicker.Flag,
                Default = Colorpicker.Default,
                Alpha = Colorpicker.Alpha,
                Callback = Colorpicker.Callback,
            })

            return NewColorpicker
        end

        function NewToggle:Keybind(Data)
            Data = Data or { }

            local Keybind = {
                Window = self.Window,
                Page = self.Page,
                Section = self.Section,

                Flag = Data.Flag or Data.flag or Library:NextFlag(),
                Default = Data.Default or Data.default or Enum.KeyCode.RightShift,
                Callback = Data.Callback or Data.callback or function() end,
                Mode = Data.Mode or Data.mode or "Toggle",
            }

            local NewKeybind, KeybindItems = Components:Keybind({
                Name = Toggle.Name,
                Parent = ToggleItems["SubElements"],
                Page = Keybind.Page,
                Flag = Keybind.Flag,
                Default = Keybind.Default,
                Mode = Keybind.Mode,
                Callback = Keybind.Callback
            })

            return NewKeybind
        end

        return NewToggle
    end

    Library.Sections.Button = function(self)
        local Button = {
            Window = self.Window,
            Page = self.Page,
            Section = self
        }

        local NewButton, ButtonItems = Components:Button({
            Parent = Button.Section.Items["Content"],
            Page = Button.Page
        })

        return NewButton
    end

    Library.Sections.Slider = function(self, Data)
        Data = Data or { }
        
        local Slider = {
            Window = self.Window,
            Page = self.Page,
            Section = self,

            Name = Data.Name or Data.name or "Slider",
            Flag = Data.Flag or Data.flag or Library:NextFlag(),
            Min = Data.Min or Data.min or 0,
            Decimals = Data.Decimals or Data.decimals or 1,
            Suffix = Data.Suffix or Data.suffix or "",
            Max = Data.Max or Data.max or 100,
            Default = Data.Default or Data.Default or 0,
            Callback = Data.Callback or Data.callback or function() end,
        }

        local NewSlider, SliderItems = Components:Slider({
            Name = Slider.Name,
            Parent = Slider.Section.Items["Content"],
            Flag = Slider.Flag,
            Min = Slider.Min,
            Page = Slider.Page,
            Decimals = Slider.Decimals,
            Suffix = Slider.Suffix,
            Max = Slider.Max,
            Default = Slider.Default,
            Callback = Slider.Callback,
        })

        local PageSearchData = Library.SearchItems[Slider.Page]

        if PageSearchData then
            local SearchData = {
                Element = SliderItems["Slider"],
                Name = Slider.Name,
            }

            TableInsert(PageSearchData, SearchData)
        end

        return NewSlider 
    end

    Library.Sections.Dropdown = function(self, Data)
        Data = Data or { }

        local Dropdown = {
            Window = self.Window,
            Page = self.Page,
            Section = self,

            Name = Data.Name or Data.name or "Dropdown",
            Flag = Data.Flag or Data.flag or Library:NextFlag(),
            Items = Data.Items or Data.items or { },
            Default = Data.Default or Data.default or nil,
            Multi = Data.Multi or Data.multi or false,
            Callback = Data.Callback or Data.callback or function() end            
        }

        local NewDropdown, DropdownItems = Components:Dropdown({
            Name = Dropdown.Name,
            Parent = Dropdown.Section.Items["Content"],
            Flag = Dropdown.Flag,
            Items = Dropdown.Items,
            Page = Dropdown.Page,
            Default = Dropdown.Default,
            Multi = Dropdown.Multi,
            Callback = Dropdown.Callback,
        })

        local PageSearchData = Library.SearchItems[Dropdown.Page]

        if PageSearchData then
            local SearchData = {
                Element = DropdownItems["Dropdown"],
                Name = Dropdown.Name,
            }

            TableInsert(PageSearchData, SearchData)
        end

        return NewDropdown 
    end

    Library.Sections.Label = function(self, Name)
        local Label = {
            Window = self.Window,
            Page = self.Page,
            Section = self,

            Name = Name or "Label"
        }

        local NewLabel, LabelItems = Components:Label({
            Name = Label.Name,
            Parent = Label.Section.Items["Content"],
            Page = Label.Page,
        })

        function NewLabel:Colorpicker(Data)
            Data = Data or { }

            local Colorpicker = {
                Window = self.Window,
                Page = self.Page,
                Section = self.Section,

                Flag = Data.Flag or Data.flag or Library:NextFlag(),
                Default = Data.Default or Data.default or Color3.fromRGB(255, 255, 255),
                Callback = Data.Callback or Data.callback or function() end,
                Alpha = Data.Alpha or Data.alpha or 0,
            }

            local NewColorpicker, ColorpickerItems = Components:Colorpicker({
                Name = Colorpicker.Name,
                Parent = LabelItems["SubElements"],
                Pages = true,
                Page = Colorpicker.Page,
                Flag = Colorpicker.Flag,
                Default = Colorpicker.Default,
                Alpha = Colorpicker.Alpha,
                Callback = Colorpicker.Callback,
            })

            return NewColorpicker
        end

        function NewLabel:Keybind(Data)
            Data = Data or { }

            local Keybind = {
                Window = self.Window,
                Page = self.Page,
                Section = self.Section,

                Flag = Data.Flag or Data.flag or Library:NextFlag(),
                Default = Data.Default or Data.default or Enum.KeyCode.RightShift,
                Callback = Data.Callback or Data.callback or function() end,
                Mode = Data.Mode or Data.mode or "Toggle",
            }

            local NewKeybind, KeybindItems = Components:Keybind({
                Name = Label.Name,
                Parent = LabelItems["SubElements"],
                Page = Keybind.Page,
                Flag = Keybind.Flag,
                Default = Keybind.Default,
                Mode = Keybind.Mode,
                Callback = Keybind.Callback
            })

            return NewKeybind
        end

        local PageSearchData = Library.SearchItems[Label.Page]

        if PageSearchData then
            local SearchData = {
                Element = LabelItems["Label"],
                Name = Label.Name,
            }

            TableInsert(PageSearchData, SearchData)
        end

        return NewLabel
    end

    Library.Sections.Textbox = function(self, Data)
        Data = Data or { }

        local Textbox = {
            Window = self.Window,
            Page = self.Page,
            Section = self,

            Name = Data.Name or Data.name or "Textbox",
            Flag = Data.Flag or Data.flag or Library:NextFlag(),
            Default = Data.Default or Data.default or "",
            Numeric = Data.Numeric or Data.numeric or false,
            Finished = Data.Finished or Data.finished or false,
            Placeholder = Data.Placeholder or Data.placeholder or "...",
            Callback = Data.Callback or Data.callback or function() end,
        }

        local NewTextbox, TextboxItems = Components:Textbox({
            Name = Textbox.Name,
            Placeholder = Textbox.Placeholder,
            Parent = Textbox.Section.Items["Content"],
            Flag = Textbox.Flag,
            Page = Textbox.Page,
            Default = Textbox.Default,
            Numeric = Textbox.Numeric,
            Finished = Textbox.Finished,
            Callback = Textbox.Callback,
        })

        local PageSearchData = Library.SearchItems[Textbox.Page]

        if PageSearchData then
            local SearchData = {
                Element = TextboxItems["Textbox"],
                Name = Textbox.Name,
            }

            TableInsert(PageSearchData, SearchData)
        end

        return NewTextbox
    end

    Library.Sections.Searchbox = function(self, Data)
        Data = Data or { }

        local Searchbox = {
            Window = self.Window,
            Page = self.Page,
            Section = self,

            Name = Data.Name or Data.name or "Searchbox",
            Flag = Data.Flag or Data.flag or Library:NextFlag(),
            Items = Data.Items or Data.items or { },
            Default = Data.Default or Data.default or nil,
            Multi = Data.Multi or Data.multi or false,
            Callback = Data.Callback or Data.callback or function() end            
        }

        local NewSearchbox, SearchboxItems = Components:Searchbox({
            Parent = Searchbox.Section.Items["Content"],
            Flag = Searchbox.Flag,
            Items = Searchbox.Items,
            Page = Searchbox.Page,
            Default = Searchbox.Default,
            Multi = Searchbox.Multi,
            Callback = Searchbox.Callback,
        })

        local PageSearchData = Library.SearchItems[Searchbox.Page]

        if PageSearchData then
            local SearchData = {
                Element = SearchboxItems["Listbox"],
                Name = Searchbox.Name,
            }

            TableInsert(PageSearchData, SearchData)
        end

        return NewSearchbox 
    end

    Library.BlankElement = function(self, Data)
        local BlankElement = {
            Name = Data.Name or Data.name or "Blank",
            Size = Data.Size or Data.size or 18
        }

        local Items = { } do
            Items["BlankElement"] = Instances:Create("Frame", {
                Parent = Library.Holder.Instance,
                Name = "\0",
                BackgroundTransparency = 1,
                BorderColor3 = FromRGB(0, 0, 0),
                Size = UDim2New(1, 0, 0, BlankElement.Size),
                BorderSizePixel = 0,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })

            Items["Text"] = Instances:Create("TextLabel", {
                Parent = Items["Label"].Instance,
                Name = "\0",
                FontFace = Library.Font,
                TextColor3 = FromRGB(235, 235, 235),
                BorderColor3 = FromRGB(0, 0, 0),
                Text = BlankElement.Name,
                Size = UDim2New(0, 0, 0, 15),
                AnchorPoint = Vector2New(0, 0.5),
                Position = UDim2New(0, 0, 0.5, 0),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 9,
                BackgroundColor3 = FromRGB(255, 255, 255)
            })  Items["Text"]:AddToTheme({TextColor3 = "Text"})

            Items["Text"]:TextBorder()
        end

        return BlankElement, Items
    end

    Library.CreateSettingsPage = function(self, Window, Watermark, KeybindList)
        local SettingsPage = Window:Page({Name = "Settings", SubPages = true}) do 
            local ThemingSubPage = SettingsPage:SubPage({Name = "Theming", Columns = 2}) do 
                local ThemesSection = ThemingSubPage:Section({Name = "Themes", Side = 1}) do
                    for Index, Value in Library.Theme do 
                        ThemesSection:Label(Index):Colorpicker({
                            Name = Index,
                            Flag = Index.."Theme",
                            Default = Value,
                            Callback = function(Value)
                                Library.Theme[Index] = Value
                                Library:ChangeTheme(Index, Value)
                            end
                        })
                    end
                end
            end

            local ConfigsSubPage = SettingsPage:SubPage({Name = "Configs", Columns = 2}) do 
                local ConfigsSection = ConfigsSubPage:Section({Name = "Configs", Side = 1}) do
                    local ConfigName
                    local ConfigSelected

                    local ConfigsSearchbox = ConfigsSection:Searchbox({
                        Name = "SearchboxConfigs",
                        Flag = "ConfigsSearchobx",
                        Items = { },
                        Multi = false,
                        Callback = function(Value)
                            ConfigSelected = Value
                        end
                    })

                    ConfigsSection:Textbox({
                        Name = "Config name", 
                        Default = "", 
                        Flag = "ConfigName", 
                        Placeholder = "Enter text", 
                        Callback = function(Value)
                            ConfigName = Value
                        end
                    })

                    local CreateAndDeleteButton = ConfigsSection:Button()

                    CreateAndDeleteButton:Add("Create", function()
                        if ConfigName and ConfigName ~= "" then
                            if not isfile(Library.Folders.Configs .. "/" .. ConfigName .. ".json") then
                                writefile(Library.Folders.Configs .. "/" .. ConfigName .. ".json", Library:GetConfig())
                                Library:Notification("Success", "Created config "..ConfigName .. " succesfully", 5)
                                Library:RefreshConfigsList(ConfigsSearchbox)
                            else
                                Library:Notification("Error", "Config with the name "..ConfigName .. " already exists", 5)
                                return
                            end
                        end
                    end)

                    CreateAndDeleteButton:Add("Delete", function()
                        if ConfigSelected then
                            Library:DeleteConfig(ConfigSelected)
                            Library:Notification("Success", "Deleted config "..ConfigSelected .. " succesfully", 5)
                            Library:RefreshConfigsList(ConfigsSearchbox)
                        end
                    end)

                    local LoadAndSaveButton = ConfigsSection:Button()    

                    LoadAndSaveButton:Add("Load", function()
                        if ConfigSelected then
                            local Success, Result = Library:LoadConfig(readfile(Library.Folders.Configs .. "/" .. ConfigSelected))

                            if Success then 
                                Library:Notification("Success", "Loaded config "..ConfigSelected .. " succesfully", 5)
                            else
                                Library:Notification("Error", "Failed to load config "..ConfigSelected .. " report this to the devs:\n"..Result, 5)
                            end
                        end
                    end)

                    LoadAndSaveButton:Add("Save", function()
                        if ConfigName and ConfigName ~= "" then
                            writefile(Library.Folders.Configs .. "/" .. ConfigName .. ".json", Library:GetConfig())
                            Library:Notification("Success", "Saved config "..ConfigName .. " succesfully", 5)
                            Library:RefreshConfigsList(ConfigsSearchbox)
                        end
                    end)

                    Library:RefreshConfigsList(ConfigsSearchbox)
                end
            end

            local SettingsSubPage = SettingsPage:SubPage({Name = "Settings", Columns = 2}) do 
                local SettingsSection = SettingsSubPage:Section({Name = "Settings", Side = 1}) do
                    SettingsSection:Toggle({
                        Name = "Watermark",
                        Flag = "Watermark",
                        Default = true,
                        Callback = function(Value)
                            Watermark:SetVisibility(Value)
                        end
                    })

                    SettingsSection:Toggle({
                        Name = "Keybind list",
                        Flag = "Keybind list",
                        Default = true,
                        Callback = function(Value)
                            KeybindList:SetVisibility(Value)
                        end
                    })

                    SettingsSection:Slider({
                        Name = "Fade time",
                        Flag = "FadeTime",
                        Default = Library.FadeSpeed,
                        Min = 0,
                        Max = 1,
                        Decimals = 0.01,
                        Callback = function(Value)
                            Library.FadeSpeed = Value
                        end
                    })

                    SettingsSection:Slider({
                        Name = "Tween time",
                        Flag = "TweenTime",
                        Default = Library.Tween.Time,
                        Min = 0,
                        Max = 1,
                        Decimals = 0.01,
                        Callback = function(Value)
                            Library.Tween.Time = Value
                        end
                    })

                    SettingsSection:Dropdown({
                        Name = "Tween style",
                        Flag = "Tween style",
                        Items = { "Linear", "Quad", "Quart", "Back", "Bounce", "Circular", "Cubic", "Elastic", "Exponential", "Sine", "Quint" },
                        Default = "Cubic",
                        Callback = function(Value)
                            Library.Tween.Style = Enum.EasingStyle[Value]
                        end
                    })

                    SettingsSection:Dropdown({
                        Name = "Tween direction",
                        Flag = "Tween direction",
                        Items = { "In", "Out", "InOut" },
                        Default = "Out",
                        Callback = function(Value)
                            Library.Tween.Direction = Enum.EasingDirection[Value]
                        end
                    })

                    SettingsSection:Button():Add("Unload", function()
                        Library:Unload()
                    end)

                    SettingsSection:Label("UI Keybind"):Keybind({
                        Name = "Menu keybind",
                        Flag = "UIKeybind",
                        Default = Library.MenuKeybind,
                        Mode = "Toggle",
                        Callback = function()
                            Library.MenuKeybind = Library.Flags["UIKeybind"].Key
                        end
                    })
                end
            end
        end
        
        return SettingsPage
    end
end
getgenv().Library = Library
return Library
