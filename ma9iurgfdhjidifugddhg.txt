do
    local isStandalone = not (loadingGui and loadingGui:FindFirstChild("LoginSuccess"))
    local _CoreGui = game:GetService("CoreGui")
    local _TweenService = game:GetService("TweenService")
    local _RunService = game:GetService("RunService")
    local _Players = game:GetService("Players")
    local _UserInputService = game:GetService("UserInputService")
    local _SoundService = game:GetService("SoundService")
    local _VirtualInputManager = game:GetService("VirtualInputManager")
    local _TextChatService = game:GetService("TextChatService")
    local _ReplicatedStorage = game:GetService("ReplicatedStorage")
    local _PathfindingService = game:GetService("PathfindingService")
    local _CLICK_SOUND = "rbxassetid://100906385041190"
    local _HOVER_SOUND = "rbxassetid://92708987611847"
    local _NOTIF_SOUND = "rbxassetid://127119919079045"
    -- Notification stacking system
    local _activeNotifications = {}
    local _NOTIF_HEIGHT = 60
    local _NOTIF_GAP = 10
    local _NOTIF_BOTTOM_MARGIN = 20
    local function _repositionNotifications()
        for i, entry in ipairs(_activeNotifications) do
            local idx = #_activeNotifications - i
            local targetY = -(_NOTIF_BOTTOM_MARGIN + idx * (_NOTIF_HEIGHT + _NOTIF_GAP))
            _TweenService:Create(entry.container, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
                Position = UDim2.new(0, 20, 1, targetY)
            }):Play()
        end
    end
    local function _removeNotification(entry)
        for i, e in ipairs(_activeNotifications) do
            if e == entry then
                table.remove(_activeNotifications, i)
                break
            end
        end
        _repositionNotifications()
    end
    local function _playSound(soundId, vol)
        local s = Instance.new("Sound")
        s.SoundId = soundId
        s.Volume = vol or 0.5
        s.Parent = _SoundService
        s:Play()
        s.Ended:Connect(function() s:Destroy() end)
    end
    local function _applySoundsToButton(btn)
        if btn:IsA("GuiButton") then
            btn.MouseEnter:Connect(function() _playSound(_HOVER_SOUND) end)
            btn.MouseButton1Click:Connect(function() _playSound(_CLICK_SOUND) end)
        end
    end
    local _glitchSymbols = {"@", "!", "9", "6", "=", "<", "?", "$", "&"}
    local _activeDecode = 0
    local function _animateTabTitle(label, targetText)
        _activeDecode = _activeDecode + 1
        local currentDecode = _activeDecode
        local currentStr = ""
        for i = 1, #targetText do
            if currentDecode ~= _activeDecode then return end
            local char = targetText:sub(i, i)
            if char == " " then
                currentStr = currentStr .. char
                label.Text = currentStr
            else
                label.Text = currentStr .. _glitchSymbols[math.random(1, #_glitchSymbols)]
                task.wait(0.08)
                if currentDecode ~= _activeDecode then return end
                currentStr = currentStr .. char
                label.Text = currentStr
            end
        end
    end
    -- ======= BLUE PARTICLE BURST SYSTEM ======= --
    local function _createParticleBurst(parent)
        for i = 1, 10 do
            local square = Instance.new("Frame")
            square.Size = UDim2.new(0, 6, 0, 6)
            square.Position = UDim2.new(0.5, 0, 0.5, 0)
            square.AnchorPoint = Vector2.new(0.5, 0.5)
            square.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
            square.BackgroundTransparency = 0
            square.BorderSizePixel = 0
            square.Parent = parent
            Instance.new("UICorner", square).CornerRadius = UDim.new(1, 0)
            local direction = Vector2.new(math.random(-10, 10), math.random(-10, 10)).Unit
            local distance = math.random(30, 60)
            local duration = 0.4
            square:TweenPosition(UDim2.new(0.5, direction.X * distance, 0.5, direction.Y * distance), "Out", "Quad", duration, true)
            _TweenService:Create(square, TweenInfo.new(duration), {BackgroundTransparency = 1}):Play()
            game.Debris:AddItem(square, duration + 0.1)
        end
    end
    -- 1. Setup Root GUI
    local hubGui
    if isStandalone then
        if _CoreGui:FindFirstChild("InfinitesHubMain") then _CoreGui.InfinitesHubMain:Destroy() end
        hubGui = Instance.new("ScreenGui")
        hubGui.Name = "InfinitesHubMain"
        hubGui.ResetOnSpawn = false
        hubGui.IgnoreGuiInset = true
        hubGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        local ok = pcall(function() hubGui.Parent = _CoreGui end)
        if not ok then hubGui.Parent = _Players.LocalPlayer:WaitForChild("PlayerGui") end
    else
        hubGui = loadingGui
        hubGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    end
    -- 2. Build the Main Hub Frame
    local hubMain = Instance.new("CanvasGroup")
    hubMain.Name = "HubMain"
    hubMain.Size = UDim2.new(0, 0, 0, 380)
    hubMain.Position = UDim2.new(0.5, 0, 0.5, 0)
    hubMain.AnchorPoint = Vector2.new(0.5, 0.5)
    hubMain.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
    hubMain.BorderSizePixel = 0
    hubMain.ClipsDescendants = true
    hubMain.Visible = false
    hubMain.ZIndex = 1
    hubMain.Parent = hubGui
    Instance.new("UICorner", hubMain).CornerRadius = UDim.new(0, 12)
    local sidebar = Instance.new("Frame")
    sidebar.Size = UDim2.new(0, 140, 1, 0)
    sidebar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    sidebar.BorderSizePixel = 0
    sidebar.ZIndex = 2
    sidebar.Parent = hubMain
    local hubLogo = Instance.new("TextLabel")
    hubLogo.Size = UDim2.new(1, 0, 0, 40)
    hubLogo.Position = UDim2.new(0, 15, 0, 10)
    hubLogo.BackgroundTransparency = 1
    hubLogo.TextColor3 = Color3.fromRGB(0, 150, 255)
    hubLogo.Text = "\226\151\134 Label"
    hubLogo.Font = Enum.Font.GothamBold
    hubLogo.TextSize = 13
    hubLogo.TextXAlignment = Enum.TextXAlignment.Left
    hubLogo.Parent = sidebar
    -- ======= TAB SYSTEM ======= --
    local tabButtons = {}
    local tabContents = {}
    local activeTab = nil
    local function createTabButton(name, yPos)
        local tabBtnFrame = Instance.new("TextButton")
        tabBtnFrame.Size = UDim2.new(1, 0, 0, 35)
        tabBtnFrame.Position = UDim2.new(0, 0, 0, yPos)
        tabBtnFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
        tabBtnFrame.BorderSizePixel = 0
        tabBtnFrame.Text = ""
        tabBtnFrame.AutoButtonColor = false
        tabBtnFrame.Parent = sidebar
        _applySoundsToButton(tabBtnFrame)
        local indicator = Instance.new("Frame")
        indicator.Size = UDim2.new(0, 12, 0, 12)
        indicator.Position = UDim2.new(0, 15, 0.5, -6)
        indicator.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        indicator.BorderSizePixel = 0
        indicator.Parent = tabBtnFrame
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, -35, 1, 0)
        label.Position = UDim2.new(0, 35, 0, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.new(1, 1, 1)
        label.Text = name
        label.Font = Enum.Font.GothamMedium
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = tabBtnFrame
        return tabBtnFrame, indicator, label
    end
    local function createContentArea()
        local content = Instance.new("Frame")
        content.Size = UDim2.new(1, -140, 1, 0)
        content.Position = UDim2.new(0, 140, 0, 0)
        content.BackgroundTransparency = 1
        content.ClipsDescendants = true
        content.ZIndex = 1
        content.Visible = false
        content.Parent = hubMain
        return content
    end
    local function switchTab(tabName)
        if activeTab == tabName then return end
        activeTab = tabName
        for name, content in pairs(tabContents) do
            content.Visible = (name == tabName)
        end
        for name, data in pairs(tabButtons) do
            local isActive = (name == tabName)
            local targetBg = isActive and Color3.fromRGB(35, 35, 35) or Color3.fromRGB(24, 24, 24)
            local targetIndicator = isActive and Color3.new(1, 1, 1) or Color3.fromRGB(0, 150, 255)
            local targetText = isActive and Color3.new(1, 1, 1) or Color3.new(0.8, 0.8, 0.8)
            _TweenService:Create(data.btn, TweenInfo.new(0.2), {BackgroundColor3 = targetBg}):Play()
            _TweenService:Create(data.indicator, TweenInfo.new(0.2), {BackgroundColor3 = targetIndicator}):Play()
            _TweenService:Create(data.label, TweenInfo.new(0.2), {TextColor3 = targetText}):Play()
        end
    end
    local function createTabBackground(parentContentArea)
        local bgContainer = Instance.new("Frame")
        bgContainer.Size = UDim2.new(1, 0, 1, 0)
        bgContainer.BackgroundTransparency = 1
        bgContainer.BorderSizePixel = 0
        bgContainer.ClipsDescendants = true
        bgContainer.Parent = parentContentArea
        local glow = Instance.new("Frame")
        glow.Size = UDim2.new(1, 0, 0, 60)
        glow.BackgroundTransparency = 0.6
        glow.BackgroundColor3 = Color3.fromRGB(0, 100, 200)
        glow.BorderSizePixel = 0
        glow.Parent = bgContainer
        local grad = Instance.new("UIGradient")
        grad.Rotation = 90
        grad.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.6),
            NumberSequenceKeypoint.new(1, 1)
        })
        grad.Parent = glow
        local wm = Instance.new("Frame")
        wm.Size = UDim2.new(0, 400, 0, 400)
        wm.Position = UDim2.new(1, -100, 1, -50)
        wm.AnchorPoint = Vector2.new(0.5, 0.5)
        wm.Rotation = 45
        wm.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        wm.BorderSizePixel = 0
        wm.Parent = bgContainer
        local im = Instance.new("Frame")
        im.Size = UDim2.new(0, 370, 0, 370)
        im.Position = UDim2.new(0.5, 0, 0.5, 0)
        im.AnchorPoint = Vector2.new(0.5, 0.5)
        im.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
        im.BorderSizePixel = 2
        im.BorderColor3 = Color3.fromRGB(24, 24, 24)
        im.Parent = wm
        local tg = Instance.new("Frame")
        tg.Size = UDim2.new(1, 0, 1, 0)
        tg.BackgroundTransparency = 1
        tg.Parent = bgContainer
        for x = 1, 6 do
            for y = 1, 5 do
                local dot = Instance.new("Frame")
                dot.Size = UDim2.new(0, 2, 0, 2)
                dot.Position = UDim2.new(x/7, 0, y/6, 0)
                dot.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
                dot.BorderSizePixel = 0
                dot.Parent = tg
            end
        end
    end
    local function createTabLayout(parentContentArea)
        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -40, 0, 50)
        title.Position = UDim2.new(0, 25, 0, 15)
        title.BackgroundTransparency = 1
        title.TextColor3 = Color3.new(0.9, 0.9, 0.9)
        title.Text = ""
        title.Font = Enum.Font.GothamBold
        title.TextSize = 24
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.Parent = parentContentArea
        local clipWrapper = Instance.new("CanvasGroup")
        clipWrapper.Size = UDim2.new(1, -50, 1, -80)
        clipWrapper.Position = UDim2.new(0, 25, 0, 65)
        clipWrapper.BackgroundTransparency = 1
        clipWrapper.BorderSizePixel = 0
        clipWrapper.Parent = parentContentArea
        local scrollList = Instance.new("ScrollingFrame")
        scrollList.Size = UDim2.new(1, 0, 1, 0)
        scrollList.Position = UDim2.new(0, 0, 0, 0)
        scrollList.BackgroundTransparency = 1
        scrollList.BorderSizePixel = 0
        scrollList.ScrollBarThickness = 4
        scrollList.ClipsDescendants = true
        scrollList.AutomaticCanvasSize = Enum.AutomaticSize.Y
        scrollList.Parent = clipWrapper
        local layout = Instance.new("UIListLayout")
        layout.Padding = UDim.new(0, 8)
        layout.Parent = scrollList
        return title, scrollList
    end
    -- Shared row creation helper
    local function createRow(name)
        local rowBtn = Instance.new("TextButton")
        rowBtn.Size = UDim2.new(1, 0, 0, 38)
        rowBtn.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
        rowBtn.BorderSizePixel = 0
        rowBtn.Text = ""
        rowBtn.AutoButtonColor = false
        local diamond = Instance.new("Frame")
        diamond.Size = UDim2.new(0, 6, 0, 6)
        diamond.Position = UDim2.new(0, 10, 0.5, 0)
        diamond.AnchorPoint = Vector2.new(0.5, 0.5)
        diamond.Rotation = 45
        diamond.BackgroundColor3 = Color3.new(1, 1, 1)
        diamond.BorderSizePixel = 0
        diamond.Visible = false
        diamond.Parent = rowBtn
        local icon = Instance.new("Frame")
        icon.Size = UDim2.new(0, 14, 0, 14)
        icon.Position = UDim2.new(0, 24, 0.5, -7)
        icon.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        icon.BorderSizePixel = 0
        icon.Parent = rowBtn
        Instance.new("UICorner", icon).CornerRadius = UDim.new(0, 2)
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0, 150, 1, 0)
        label.Position = UDim2.new(0, 50, 0, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        label.Text = name
        label.Font = Enum.Font.GothamMedium
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = rowBtn
        local topLine = Instance.new("Frame")
        topLine.Size = UDim2.new(0, 0, 0, 2)
        topLine.BackgroundColor3 = Color3.new(1, 1, 1)
        topLine.BorderSizePixel = 0
        topLine.Visible = true
        topLine.Parent = rowBtn
        local bottomLine = Instance.new("Frame")
        bottomLine.Size = UDim2.new(0, 0, 0, 2)
        bottomLine.Position = UDim2.new(0, 0, 1, -2)
        bottomLine.BackgroundColor3 = Color3.new(1, 1, 1)
        bottomLine.BorderSizePixel = 0
        bottomLine.Visible = true
        bottomLine.Parent = rowBtn
        return rowBtn, icon, label, topLine, bottomLine, diamond
    end
    local function applyHoverState(btn, icon, label, topLine, bottomLine, diamond, isToggle, getToggleState)
        local normalBg = Color3.fromRGB(24, 24, 24)
        local hoverBg = Color3.fromRGB(35, 35, 35)
        local normalIcon = Color3.fromRGB(0, 150, 255)
        local hoverIcon = Color3.new(1, 1, 1)
        local normalText = Color3.new(0.8, 0.8, 0.8)
        local hoverText = Color3.new(1, 1, 1)
        local function updateVisuals(hovered)
            local toggled = isToggle and getToggleState and getToggleState()
            local active = hovered or toggled
            _TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = active and hoverBg or normalBg}):Play()
            _TweenService:Create(icon, TweenInfo.new(0.2), {BackgroundColor3 = active and hoverIcon or normalIcon}):Play()
            _TweenService:Create(label, TweenInfo.new(0.2), {TextColor3 = active and hoverText or normalText}):Play()
            local targetLineSize = active and UDim2.new(1, 0, 0, 2) or UDim2.new(0, 0, 0, 2)
            local tweenInfo = TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
            _TweenService:Create(topLine, tweenInfo, {Size = targetLineSize}):Play()
            _TweenService:Create(bottomLine, tweenInfo, {Size = targetLineSize}):Play()
            diamond.Visible = active
        end
        btn.MouseEnter:Connect(function() _playSound(_HOVER_SOUND) updateVisuals(true) end)
        btn.MouseLeave:Connect(function() updateVisuals(false) end)
        btn.MouseButton1Click:Connect(function()
            _playSound(_CLICK_SOUND)
            task.delay(0.05, function() updateVisuals(true) end)
        end)
        return updateVisuals
    end
    local function createToggleRow(name, parentList, getStateFn, onClickFn)
        local row, ic, lb, tp, bt, dia = createRow(name)
        row.Parent = parentList
        local updateVis = applyHoverState(row, ic, lb, tp, bt, dia, true, getStateFn)
        local textContainer = Instance.new("Frame")
        textContainer.Size = UDim2.new(0, 50, 1, 0)
        textContainer.Position = UDim2.new(1, -65, 0, 0)
        textContainer.BackgroundTransparency = 1
        textContainer.ClipsDescendants = true
        textContainer.Parent = row
        local offLbl = Instance.new("TextLabel")
        offLbl.Size = UDim2.new(1, 0, 1, 0)
        offLbl.Position = UDim2.new(0, 0, 0, 0)
        offLbl.BackgroundTransparency = 1
        offLbl.TextColor3 = Color3.fromRGB(120, 120, 120)
        offLbl.Text = "OFF"
        offLbl.Font = Enum.Font.GothamMedium
        offLbl.TextSize = 13
        offLbl.TextXAlignment = Enum.TextXAlignment.Right
        offLbl.Parent = textContainer
        local onLbl = Instance.new("TextLabel")
        onLbl.Size = UDim2.new(1, 0, 1, 0)
        onLbl.Position = UDim2.new(1, 0, 0, 0)
        onLbl.BackgroundTransparency = 1
        onLbl.TextColor3 = Color3.fromRGB(0, 255, 100)
        onLbl.Text = "ON"
        onLbl.Font = Enum.Font.GothamMedium
        onLbl.TextSize = 13
        onLbl.TextXAlignment = Enum.TextXAlignment.Right
        onLbl.Parent = textContainer
        row.MouseButton1Click:Connect(function()
            _createParticleBurst(row)
            onClickFn()
            local slideTweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
            if getStateFn() then
                _TweenService:Create(offLbl, slideTweenInfo, {Position = UDim2.new(-1, 0, 0, 0)}):Play()
                _TweenService:Create(onLbl, slideTweenInfo, {Position = UDim2.new(0, 0, 0, 0)}):Play()
            else
                _TweenService:Create(offLbl, slideTweenInfo, {Position = UDim2.new(0, 0, 0, 0)}):Play()
                _TweenService:Create(onLbl, slideTweenInfo, {Position = UDim2.new(1, 0, 0, 0)}):Play()
            end
            updateVis(true)
        end)
        return row, updateVis
    end
    local function createSliderRow(name, parentList, defaultVal, onValueChanged)
        local row, sIc, sLb, sTp, sBt, sDi = createRow(name)
        row.Parent = parentList
        sIc.Visible = false
        local textBox = Instance.new("TextBox")
        textBox.Size = UDim2.new(0, 26, 0, 22)
        textBox.Position = UDim2.new(0, 20, 0.5, -11)
        textBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        textBox.BorderSizePixel = 1
        textBox.BorderColor3 = Color3.fromRGB(45, 45, 45)
        textBox.TextColor3 = Color3.new(1, 1, 1)
        textBox.Font = Enum.Font.GothamMedium
        textBox.TextSize = 12
        textBox.Text = tostring(defaultVal)
        textBox.ZIndex = 3
        textBox.Parent = row
        Instance.new("UICorner", textBox).CornerRadius = UDim.new(0, 4)
        local track = Instance.new("Frame")
        track.Size = UDim2.new(0, 158, 0, 14)
        track.Position = UDim2.new(1, -170, 0.5, -7)
        track.BackgroundTransparency = 1
        track.Parent = row
        local tLayout = Instance.new("UIListLayout")
        tLayout.FillDirection = Enum.FillDirection.Horizontal
        tLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        tLayout.SortOrder = Enum.SortOrder.LayoutOrder
        tLayout.Padding = UDim.new(0, 2)
        tLayout.Parent = track
        local NTICKS = 40
        local tickFrames = {}
        for i = 1, NTICKS do
            local t = Instance.new("Frame")
            t.LayoutOrder = i
            t.Size = UDim2.new(0, 2, 0, 2)
            t.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
            t.BorderSizePixel = 0
            t.Parent = track
            tickFrames[i] = t
        end
        local currentVal = -1
        local function updateVal(val)
            val = math.clamp(math.floor(val), 1, 100)
            if val ~= currentVal then
                if currentVal ~= -1 then _playSound(_HOVER_SOUND) end
                currentVal = val
                textBox.Text = tostring(val)
                if onValueChanged then onValueChanged(val) end
                local pct = (val - 1) / 99
                local activeCount = math.floor(pct * (NTICKS - 1)) + 1
                for i = 1, NTICKS do
                    local tSize, tColor
                    if i < activeCount then
                        tSize = UDim2.new(0, 2, 0, 8)
                        tColor = Color3.new(0.8, 0.8, 0.8)
                    elseif i == activeCount then
                        tSize = UDim2.new(0, 2, 0, 14)
                        tColor = Color3.new(1, 1, 1)
                    else
                        tSize = UDim2.new(0, 2, 0, 2)
                        tColor = Color3.fromRGB(60, 60, 60)
                    end
                    local ti = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                    _TweenService:Create(tickFrames[i], ti, {Size = tSize, BackgroundColor3 = tColor}):Play()
                end
            end
        end
        updateVal(defaultVal)
        textBox.FocusLost:Connect(function()
            local num = tonumber(textBox.Text)
            if num then updateVal(num) else textBox.Text = tostring(currentVal) end
        end)
        local interact = Instance.new("TextButton")
        interact.Size = UDim2.new(1, -70, 1, 0)
        interact.Position = UDim2.new(0, 70, 0, 0)
        interact.BackgroundTransparency = 1
        interact.Text = ""
        interact.Parent = row
        local isDragging = false
        local function updateFromMouse(input)
            local trackX = track.AbsolutePosition.X
            local trackW = track.AbsoluteSize.X
            local mouseX = input.Position.X
            local pct = math.clamp((mouseX - trackX) / trackW, 0, 1)
            updateVal(math.floor(pct * 99) + 1)
        end
        interact.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                isDragging = true
                updateFromMouse(input)
            end
        end)
        _UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                isDragging = false
            end
        end)
        _UserInputService.InputChanged:Connect(function(input)
            if isDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                updateFromMouse(input)
            end
        end)
        local nBg = Color3.fromRGB(24, 24, 24)
        local hBg = Color3.fromRGB(35, 35, 35)
        local nTx = Color3.new(0.8, 0.8, 0.8)
        local hTx = Color3.new(1, 1, 1)
        row.MouseEnter:Connect(function()
            _playSound(_HOVER_SOUND)
            _TweenService:Create(row, TweenInfo.new(0.2), {BackgroundColor3 = hBg}):Play()
            _TweenService:Create(sLb, TweenInfo.new(0.2), {TextColor3 = hTx}):Play()
            local tI = TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
            _TweenService:Create(sTp, tI, {Size = UDim2.new(1, 0, 0, 2)}):Play()
            _TweenService:Create(sBt, tI, {Size = UDim2.new(1, 0, 0, 2)}):Play()
            sDi.Visible = true
        end)
        row.MouseLeave:Connect(function()
            _TweenService:Create(row, TweenInfo.new(0.2), {BackgroundColor3 = nBg}):Play()
            _TweenService:Create(sLb, TweenInfo.new(0.2), {TextColor3 = nTx}):Play()
            local tI = TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
            _TweenService:Create(sTp, tI, {Size = UDim2.new(0, 0, 0, 2)}):Play()
            _TweenService:Create(sBt, tI, {Size = UDim2.new(0, 0, 0, 2)}):Play()
            sDi.Visible = false
        end)
        return row, updateVal, function() return currentVal end
    end
    -- ======= DRAGGABLE DROPDOWN GUI HELPER ======= --
    local _allDropdownGuis = {}
    local function createDraggableDropdown(titleText)
        local dropGui = Instance.new("ScreenGui")
        dropGui.Name = "DropdownGui_" .. titleText
        dropGui.ResetOnSpawn = false
        dropGui.IgnoreGuiInset = true
        dropGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        dropGui.Enabled = false
        local ok = pcall(function() dropGui.Parent = _CoreGui end)
        if not ok then dropGui.Parent = _Players.LocalPlayer:WaitForChild("PlayerGui") end
        local dropFrame = Instance.new("Frame")
        dropFrame.Size = UDim2.new(0, 240, 0, 220)
        dropFrame.Position = UDim2.new(0.5, -120, 0.5, -110)
        dropFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
        dropFrame.BorderSizePixel = 1
        dropFrame.BorderColor3 = Color3.fromRGB(60, 60, 60)
        dropFrame.ZIndex = 100
        dropFrame.ClipsDescendants = true
        dropFrame.Parent = dropGui
        Instance.new("UICorner", dropFrame).CornerRadius = UDim.new(0, 4)
        local dropHeader = Instance.new("Frame")
        dropHeader.Size = UDim2.new(1, 0, 0, 26)
        dropHeader.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
        dropHeader.BorderSizePixel = 0
        dropHeader.ZIndex = 101
        dropHeader.Parent = dropFrame
        Instance.new("UICorner", dropHeader).CornerRadius = UDim.new(0, 4)
        local dropHeaderBlock = Instance.new("Frame")
        dropHeaderBlock.Size = UDim2.new(1, 0, 0, 4)
        dropHeaderBlock.Position = UDim2.new(0, 0, 1, -4)
        dropHeaderBlock.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
        dropHeaderBlock.BorderSizePixel = 0
        dropHeaderBlock.ZIndex = 101
        dropHeaderBlock.Parent = dropHeader
        local dropTitle = Instance.new("TextLabel")
        dropTitle.Size = UDim2.new(1, -30, 1, 0)
        dropTitle.Position = UDim2.new(0, 10, 0, 0)
        dropTitle.BackgroundTransparency = 1
        dropTitle.TextColor3 = Color3.new(1, 1, 1)
        dropTitle.Text = titleText
        dropTitle.Font = Enum.Font.GothamMedium
        dropTitle.TextSize = 12
        dropTitle.TextXAlignment = Enum.TextXAlignment.Left
        dropTitle.ZIndex = 102
        dropTitle.Parent = dropHeader
        local closeDropBtn = Instance.new("TextButton")
        closeDropBtn.Size = UDim2.new(0, 26, 0, 26)
        closeDropBtn.Position = UDim2.new(1, -26, 0, 0)
        closeDropBtn.BackgroundTransparency = 1
        closeDropBtn.TextColor3 = Color3.new(0.6, 0.6, 0.6)
        closeDropBtn.Text = "\226\156\149"
        closeDropBtn.Font = Enum.Font.GothamMedium
        closeDropBtn.TextSize = 12
        closeDropBtn.ZIndex = 102
        closeDropBtn.Parent = dropHeader
        _applySoundsToButton(closeDropBtn)
        local searchBox = Instance.new("TextBox")
        searchBox.Size = UDim2.new(1, -16, 0, 24)
        searchBox.Position = UDim2.new(0, 8, 0, 30)
        searchBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        searchBox.BorderSizePixel = 1
        searchBox.BorderColor3 = Color3.fromRGB(50, 50, 50)
        searchBox.TextColor3 = Color3.new(1, 1, 1)
        searchBox.PlaceholderText = "Search..."
        searchBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
        searchBox.Text = ""
        searchBox.Font = Enum.Font.Gotham
        searchBox.TextSize = 12
        searchBox.TextXAlignment = Enum.TextXAlignment.Left
        searchBox.ClearTextOnFocus = false
        searchBox.ZIndex = 101
        searchBox.Parent = dropFrame
        Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 4)
        local dropList = Instance.new("ScrollingFrame")
        dropList.Size = UDim2.new(1, 0, 1, -60)
        dropList.Position = UDim2.new(0, 0, 0, 58)
        dropList.BackgroundTransparency = 1
        dropList.BorderSizePixel = 0
        dropList.ScrollBarThickness = 2
        dropList.AutomaticCanvasSize = Enum.AutomaticSize.Y
        dropList.ClipsDescendants = true
        dropList.ZIndex = 101
        dropList.Parent = dropFrame
        local dropLayout = Instance.new("UIListLayout")
        dropLayout.SortOrder = Enum.SortOrder.Name
        dropLayout.Parent = dropList
        searchBox:GetPropertyChangedSignal("Text"):Connect(function()
            local query = searchBox.Text:lower()
            for _, child in pairs(dropList:GetChildren()) do
                if child:IsA("TextButton") then
                    if query == "" then
                        child.Visible = true
                    else
                        local found = false
                        if child.Text:lower():find(query, 1, true) then found = true end
                        for _, desc in pairs(child:GetChildren()) do
                            if desc:IsA("TextLabel") and desc.Text:lower():find(query, 1, true) then
                                found = true
                                break
                            end
                        end
                        child.Visible = found
                    end
                end
            end
        end)
        local ddDragging = false
        local ddDragInput, ddDragStart, ddStartPos
        local ddTargetRot, ddCurrentRot, ddLastMouseX = 0, 0, nil
        dropFrame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                ddDragging = true
                ddDragStart = input.Position
                ddStartPos = dropFrame.Position
            end
        end)
        _UserInputService.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                if ddDragging then ddDragInput = input end
            end
        end)
        _UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                ddDragging = false
            end
        end)
        _RunService.RenderStepped:Connect(function(dt)
            if dropGui.Enabled then
                if ddDragging and ddDragInput then
                    local delta = ddDragInput.Position - ddDragStart
                    local dest = UDim2.new(ddStartPos.X.Scale, ddStartPos.X.Offset + delta.X, ddStartPos.Y.Scale, ddStartPos.Y.Offset + delta.Y)
                    dropFrame.Position = dropFrame.Position:Lerp(dest, math.min(dt * 20, 1))
                    if ddLastMouseX then
                        local velocityX = ddDragInput.Position.X - ddLastMouseX
                        ddTargetRot = math.clamp(-velocityX * 0.7, -15, 15)
                    end
                    ddLastMouseX = ddDragInput.Position.X
                else
                    ddLastMouseX = nil
                    ddTargetRot = 0
                end
                ddCurrentRot = ddCurrentRot + (ddTargetRot - ddCurrentRot) * math.min(dt * 10, 1)
                dropFrame.Rotation = ddCurrentRot
            end
        end)
        local function showDropdown()
            searchBox.Text = ""
            dropGui.Enabled = true
        end
        local function hideDropdown()
            dropGui.Enabled = false
        end
        closeDropBtn.MouseButton1Click:Connect(hideDropdown)
        table.insert(_allDropdownGuis, dropGui)
        return dropFrame, dropList, searchBox, showDropdown, hideDropdown
    end
    -- ======================================================
    -- ================ TAB 1: TAB 1 CONTENT ================
    -- ======================================================
    local tab1Btn, tab1Indicator, tab1Label = createTabButton("TAB 1", 60)
    local contentArea1 = createContentArea()
    tabButtons["TAB 1"] = {btn = tab1Btn, indicator = tab1Indicator, label = tab1Label}
    tabContents["TAB 1"] = contentArea1
    createTabBackground(contentArea1)
    local tabTitle1, elementsList1 = createTabLayout(contentArea1)
    local btnRow, bIcon, bLabel, bTop, bBot, bDia = createRow("Button")
    btnRow.Parent = elementsList1
    applyHoverState(btnRow, bIcon, bLabel, bTop, bBot, bDia, false)
    createSliderRow("Slider", elementsList1, 10, nil)
    local isToggled = false
    createToggleRow("Toggle", elementsList1, function() return isToggled end, function()
        isToggled = not isToggled
    end)
    local dropRow, dIcon, dLabel, dTop, dBot, dDia = createRow("Dropdown")
    dropRow.Parent = elementsList1
    applyHoverState(dropRow, dIcon, dLabel, dTop, dBot, dDia, false)
    local openBtnBg = Instance.new("Frame")
    openBtnBg.Size = UDim2.new(0, 65, 0, 22)
    openBtnBg.Position = UDim2.new(1, -80, 0.5, -11)
    openBtnBg.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    openBtnBg.BorderSizePixel = 0
    openBtnBg.Parent = dropRow
    Instance.new("UICorner", openBtnBg).CornerRadius = UDim.new(0, 4)
    local openBtnLabel = Instance.new("TextLabel")
    openBtnLabel.Size = UDim2.new(1, 0, 1, 0)
    openBtnLabel.BackgroundTransparency = 1
    openBtnLabel.TextColor3 = Color3.new(1, 1, 1)
    openBtnLabel.Text = "\226\137\161 OPEN"
    openBtnLabel.Font = Enum.Font.GothamMedium
    openBtnLabel.TextSize = 11
    openBtnLabel.Parent = openBtnBg
    local _, dropList1, _, showDrop1, hideDrop1 = createDraggableDropdown("Dropdown")
    for i = 1, 6 do
        local optRow = Instance.new("TextButton")
        optRow.Name = string.format("%02d", i)
        optRow.Size = UDim2.new(1, 0, 0, 24)
        optRow.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
        optRow.BorderSizePixel = 0
        optRow.Text = ""
        optRow.AutoButtonColor = false
        optRow.ZIndex = 102
        optRow.Parent = dropList1
        _applySoundsToButton(optRow)
        local optIcon = Instance.new("Frame")
        optIcon.Size = UDim2.new(0, 10, 0, 10)
        optIcon.Position = UDim2.new(0, 10, 0.5, -5)
        optIcon.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        optIcon.BorderSizePixel = 0
        optIcon.ZIndex = 102
        optIcon.Parent = optRow
        Instance.new("UICorner", optIcon).CornerRadius = UDim.new(0, 2)
        local optLabel = Instance.new("TextLabel")
        optLabel.Size = UDim2.new(1, -30, 1, 0)
        optLabel.Position = UDim2.new(0, 30, 0, 0)
        optLabel.BackgroundTransparency = 1
        optLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        optLabel.Text = tostring(i)
        optLabel.Font = Enum.Font.Gotham
        optLabel.TextSize = 12
        optLabel.TextXAlignment = Enum.TextXAlignment.Left
        optLabel.ZIndex = 102
        optLabel.Parent = optRow
        optRow.MouseEnter:Connect(function()
            optRow.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
            optIcon.BackgroundColor3 = Color3.new(1, 1, 1)
            optLabel.TextColor3 = Color3.new(1, 1, 1)
        end)
        optRow.MouseLeave:Connect(function()
            optRow.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
            optIcon.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
            optLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        end)
        optRow.MouseButton1Click:Connect(function()
            openBtnLabel.Text = "\226\137\161 " .. tostring(i)
            hideDrop1()
        end)
    end
    dropRow.MouseButton1Click:Connect(function()
        showDrop1()
    end)
    -- ======================================================
    -- ============== TAB 2: WEAPONS CONTENT ================
    -- ======================================================
    local tab2Btn, tab2Indicator, tab2Label = createTabButton("WEAPONS", 100)
    local contentArea2 = createContentArea()
    tabButtons["WEAPONS"] = {btn = tab2Btn, indicator = tab2Indicator, label = tab2Label}
    tabContents["WEAPONS"] = contentArea2
    createTabBackground(contentArea2)
    local tabTitle2, elementsList2 = createTabLayout(contentArea2)
    local selectedWeapons = {}
    local isReordering = false
    local equipWeaponEnabled = false
    local farmWeaponEnabled = false
    local weaponReorderAmount = 1
    local wDropRow, wdIcon, wdLabel, wdTop, wdBot, wdDia = createRow("Weapons")
    wDropRow.Parent = elementsList2
    applyHoverState(wDropRow, wdIcon, wdLabel, wdTop, wdBot, wdDia, false)
    local wOpenBtnBg = Instance.new("Frame")
    wOpenBtnBg.Size = UDim2.new(0, 90, 0, 22)
    wOpenBtnBg.Position = UDim2.new(1, -105, 0.5, -11)
    wOpenBtnBg.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    wOpenBtnBg.BorderSizePixel = 0
    wOpenBtnBg.Parent = wDropRow
    Instance.new("UICorner", wOpenBtnBg).CornerRadius = UDim.new(0, 4)
    local wOpenBtnLabel = Instance.new("TextLabel")
    wOpenBtnLabel.Size = UDim2.new(1, 0, 1, 0)
    wOpenBtnLabel.BackgroundTransparency = 1
    wOpenBtnLabel.TextColor3 = Color3.new(1, 1, 1)
    wOpenBtnLabel.Text = "\226\137\161 SELECT"
    wOpenBtnLabel.Font = Enum.Font.GothamMedium
    wOpenBtnLabel.TextSize = 11
    wOpenBtnLabel.TextTruncate = Enum.TextTruncate.AtEnd
    wOpenBtnLabel.Parent = wOpenBtnBg
    local _, wDropList, _, showWDrop, hideWDrop = createDraggableDropdown("Weapons")
    local function updateWeaponDropdownText()
        if #selectedWeapons == 0 then
            wOpenBtnLabel.Text = "\226\137\161 SELECT"
        elseif #selectedWeapons == 1 then
            wOpenBtnLabel.Text = "\226\137\161 " .. selectedWeapons[1]
        else
            wOpenBtnLabel.Text = "\226\137\161 " .. #selectedWeapons .. " Selected"
        end
    end
    local function updateWeaponDropdown()
        for _, child in pairs(wDropList:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        local tools = {}
        for _, item in pairs(_Players.LocalPlayer.Backpack:GetChildren()) do
            if item:IsA("Tool") and not table.find(tools, item.Name) then table.insert(tools, item.Name) end
        end
        for _, item in pairs(_Players.LocalPlayer.Character:GetChildren()) do
            if item:IsA("Tool") and not table.find(tools, item.Name) then table.insert(tools, item.Name) end
        end
        for _, toolName in pairs(tools) do
            local item = Instance.new("TextButton")
            item.Name = toolName
            item.Parent = wDropList
            item.BorderSizePixel = 0
            item.Size = UDim2.new(1, 0, 0, 24)
            item.Font = Enum.Font.Gotham
            item.TextColor3 = Color3.new(1, 1, 1)
            item.TextSize = 12
            item.TextXAlignment = Enum.TextXAlignment.Left
            item.AutoButtonColor = false
            item.ZIndex = 102
            Instance.new("UICorner", item).CornerRadius = UDim.new(0, 2)
            _applySoundsToButton(item)
            local isSelected = table.find(selectedWeapons, toolName) ~= nil
            if isSelected then
                item.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                item.Text = "  \226\156\147 " .. toolName
            else
                item.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
                item.Text = "  " .. toolName
            end
            item.MouseButton1Click:Connect(function()
                _createParticleBurst(item)
                local idx = table.find(selectedWeapons, toolName)
                if idx then
                    table.remove(selectedWeapons, idx)
                    item.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
                    item.Text = "  " .. toolName
                else
                    table.insert(selectedWeapons, toolName)
                    item.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                    item.Text = "  \226\156\147 " .. toolName
                end
                updateWeaponDropdownText()
            end)
            item.MouseEnter:Connect(function()
                if not table.find(selectedWeapons, toolName) then
                    item.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
                end
            end)
            item.MouseLeave:Connect(function()
                if not table.find(selectedWeapons, toolName) then
                    item.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
                end
            end)
        end
        wDropList.CanvasSize = UDim2.new(0, 0, 0, #tools * 24)
    end
    wDropRow.MouseButton1Click:Connect(function()
        _createParticleBurst(wDropRow)
        updateWeaponDropdown()
        showWDrop()
    end)
    local _, updateReorderSlider, getReorderVal = createSliderRow("Reorder Amount", elementsList2, 1, function(val)
        weaponReorderAmount = val
    end)
    local reorderBtnRow, rbIcon, rbLabel, rbTop, rbBot, rbDia = createRow("Start Reorder")
    reorderBtnRow.Parent = elementsList2
    applyHoverState(reorderBtnRow, rbIcon, rbLabel, rbTop, rbBot, rbDia, false)
    local function getCenter(element)
        local pos = element.AbsolutePosition
        local size = element.AbsoluteSize
        return pos.X + size.X / 2, pos.Y + size.Y / 2 + 58
    end
    local function executeReorder()
        if isReordering then return end
        isReordering = true
        local originalText = rbLabel.Text
        for i = 1, weaponReorderAmount do
            rbLabel.Text = i .. "/" .. weaponReorderAmount
            local playerGui = _Players.LocalPlayer:WaitForChild("PlayerGui")
            local st = playerGui:FindFirstChild("PermGui") and playerGui.PermGui:FindFirstChild("ShopGui") and playerGui.PermGui.ShopGui:FindFirstChild("ReorderButton")
            if st and st.Visible then
                local x, y = getCenter(st)
                _VirtualInputManager:SendMouseButtonEvent(x, y, 0, true, game, 1)
                task.wait(0.05)
                _VirtualInputManager:SendMouseButtonEvent(x, y, 0, false, game, 1)
                task.wait(0.2)
                _VirtualInputManager:SendMouseButtonEvent(x, y, 0, true, game, 1)
                task.wait(0.05)
                _VirtualInputManager:SendMouseButtonEvent(x, y, 0, false, game, 1)
            else
                local shopGui = playerGui:FindFirstChild("PermGui") and playerGui.PermGui:FindFirstChild("ShopGui")
                local shopBtn = shopGui and shopGui:FindFirstChild("ShopButton")
                local reorderBtn = shopGui and shopGui:FindFirstChild("ReorderButton")
                if shopBtn then
                    local x2, y2 = getCenter(shopBtn)
                    _VirtualInputManager:SendMouseButtonEvent(x2, y2, 0, true, game, 1)
                    task.wait(0.05)
                    _VirtualInputManager:SendMouseButtonEvent(x2, y2, 0, false, game, 1)
                    task.wait(1)
                end
                if reorderBtn then
                    local x, y = getCenter(reorderBtn)
                    _VirtualInputManager:SendMouseButtonEvent(x, y, 0, true, game, 1)
                    task.wait(0.05)
                    _VirtualInputManager:SendMouseButtonEvent(x, y, 0, false, game, 1)
                end
            end
            if i < weaponReorderAmount then
                task.wait(1.15)
            end
        end
        rbLabel.Text = originalText
        isReordering = false
    end
    reorderBtnRow.MouseButton1Click:Connect(function()
        _createParticleBurst(reorderBtnRow)
        spawnNotification("Started Reorder...")
        task.spawn(executeReorder)
    end)
    createToggleRow("Equip Weapon", elementsList2, function() return equipWeaponEnabled end, function()
        equipWeaponEnabled = not equipWeaponEnabled
        if equipWeaponEnabled then
            task.spawn(function()
                while equipWeaponEnabled do
                    task.wait(0.1)
                    for _, v in pairs(_Players.LocalPlayer.Backpack:GetChildren()) do
                        if table.find(selectedWeapons, v.Name) then v.Parent = _Players.LocalPlayer.Character end
                    end
                end
            end)
        end
    end)
    createToggleRow("Auto Farm", elementsList2, function() return farmWeaponEnabled end, function()
        farmWeaponEnabled = not farmWeaponEnabled
        if farmWeaponEnabled then
            task.spawn(function()
                while farmWeaponEnabled do
                    task.wait(0.1)
                    for _, v in pairs(_Players.LocalPlayer.Character:GetChildren()) do
                        if table.find(selectedWeapons, v.Name) then v:Activate() end
                    end
                end
            end)
        end
    end)
    -- ======================================================
    -- =============== TAB 3: PLAYER CONTENT ================
    -- ======================================================
    local tab3Btn, tab3Indicator, tab3Label = createTabButton("PLAYER", 140)
    local contentArea3 = createContentArea()
    tabButtons["PLAYER"] = {btn = tab3Btn, indicator = tab3Indicator, label = tab3Label}
    tabContents["PLAYER"] = contentArea3
    createTabBackground(contentArea3)
    local tabTitle3, elementsList3 = createTabLayout(contentArea3)
    local noclipEnabled = false
    local noclipConnection = nil
    local tpWalkEnabled = false
    local tpWalkSpeed = 1
    local reenableTpWalk = false
    local infJumpEnabled = false
    local infJumpConnection = nil
    local infJumpDebounce = false
    local antiAfkEnabled = false
    local antiAfkConnection = nil
    local _invisEnabled = false
    local _invisThread = nil
    local _invisSavedPos = nil
    local function enableNoclip()
        if not _Players.LocalPlayer.Character then return end
        noclipEnabled = true
        if noclipConnection then noclipConnection:Disconnect() noclipConnection = nil end
        noclipConnection = _RunService.Stepped:Connect(function()
            if noclipEnabled and _Players.LocalPlayer.Character then
                for _, child in pairs(_Players.LocalPlayer.Character:GetDescendants()) do
                    if child:IsA("BasePart") and child.CanCollide == true then
                        if child.Name ~= "HumanoidRootPart" then
                            child.CanCollide = false
                        end
                    end
                end
            end
        end)
    end
    local function disableNoclip()
        noclipEnabled = false
        if noclipConnection then noclipConnection:Disconnect() noclipConnection = nil end
        if _Players.LocalPlayer.Character then
            for _, child in pairs(_Players.LocalPlayer.Character:GetDescendants()) do
                if child:IsA("BasePart") then
                    child.CanCollide = true
                end
            end
        end
    end
    local function startTpWalkLoop()
        task.spawn(function()
            local character = _Players.LocalPlayer.Character or _Players.LocalPlayer.CharacterAdded:Wait()
            local humanoid = character:FindFirstChildWhichIsA("Humanoid")
            while tpWalkEnabled and humanoid and humanoid.Parent and _Players.LocalPlayer.Character == character do
                local delta = _RunService.Heartbeat:Wait()
                local moveDirection = humanoid.MoveDirection
                if moveDirection.Magnitude > 0 then
                    local finalSpeed = tpWalkSpeed >= 1 and tpWalkSpeed or 1
                    character:TranslateBy(moveDirection * delta * finalSpeed * 10)
                end
            end
        end)
    end
    _Players.LocalPlayer.CharacterAdded:Connect(function()
        if reenableTpWalk then
            task.wait(1)
            startTpWalkLoop()
        end
    end)
    local function enableInfiniteJump()
        if not _Players.LocalPlayer.Character then return end
        infJumpEnabled = true
        if infJumpConnection then infJumpConnection:Disconnect() end
        infJumpDebounce = false
        infJumpConnection = _UserInputService.JumpRequest:Connect(function()
            if not infJumpDebounce and _Players.LocalPlayer.Character then
                infJumpDebounce = true
                local humanoid = _Players.LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
                if humanoid then
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
                task.wait()
                infJumpDebounce = false
            end
        end)
    end
    local function disableInfiniteJump()
        infJumpEnabled = false
        if infJumpConnection then infJumpConnection:Disconnect() infJumpConnection = nil end
        infJumpDebounce = false
    end
    local function enableAntiAfk()
        antiAfkEnabled = true
        if antiAfkConnection then antiAfkConnection:Disconnect() antiAfkConnection = nil end
        local speaker = _Players.LocalPlayer
        if typeof(getconnections) == "function" then
            for _, connection in pairs(getconnections(speaker.Idled)) do
                if connection["Disable"] then
                    connection["Disable"](connection)
                elseif connection["Disconnect"] then
                    connection["Disconnect"](connection)
                end
            end
        else
            antiAfkConnection = speaker.Idled:Connect(function()
                local VirtualUser = game:GetService("VirtualUser")
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        end
    end
    local function disableAntiAfk()
        antiAfkEnabled = false
        if antiAfkConnection then antiAfkConnection:Disconnect() antiAfkConnection = nil end
    end
    -- ======= INVIS LOGIC (FIXED: only blocks Amare/Subscribe GUIs, not system GUIs) ======= --
    local function restoreCharacter()
        local char = _Players.LocalPlayer.Character
        if not char then return end
        for _, obj in ipairs(char:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.LocalTransparencyModifier = 0
                obj.CastShadow = true
            end
        end
    end
    -- Helper to check if a GUI is from Amare/Subscribe
    local function isAmareGui(gui)
        if not gui:IsA("ScreenGui") then return false end
        local guiName = gui.Name:lower()
        if guiName:find("amare") or guiName:find("subscribe") then return true end
        local txt = ""
        pcall(function()
            for _, desc in pairs(gui:GetDescendants()) do
                if desc:IsA("TextLabel") or desc:IsA("TextButton") then
                    txt = txt .. " " .. (desc.Text or "")
                end
            end
        end)
        local txtLower = txt:lower()
        return txtLower:find("amare") or txtLower:find("subscribe") or txtLower:find("made by")
    end
    local function enableInvis()
        if _invisThread then return end
        _invisEnabled = true
        _invisSavedPos = nil
        pcall(function()
            _invisSavedPos = _Players.LocalPlayer.Character.HumanoidRootPart.Position
        end)
        -- Record existing GUIs so we can identify new ones
        local existingPlayerGuis = {}
        local existingCoreGuis = {}
        pcall(function()
            for _, gui in pairs(_Players.LocalPlayer:WaitForChild("PlayerGui"):GetChildren()) do
                existingPlayerGuis[gui] = true
            end
        end)
        pcall(function()
            for _, gui in pairs(_CoreGui:GetChildren()) do
                existingCoreGuis[gui] = true
            end
        end)
        -- Set up watchers that only destroy Amare/Subscribe GUIs (not all new GUIs)
        local blockConnections = {}
        local function watchForAmareGui(parent, existingSet)
            local conn
            conn = parent.ChildAdded:Connect(function(child)
                if child:IsA("ScreenGui") and not existingSet[child] then
                    -- Wait a tiny bit for the GUI to populate its children so we can check text
                    task.wait(0.1)
                    if isAmareGui(child) then
                        pcall(function() child.Enabled = false end)
                        pcall(function() child:Destroy() end)
                    end
                end
            end)
            table.insert(blockConnections, conn)
        end
        pcall(function() watchForAmareGui(_Players.LocalPlayer:WaitForChild("PlayerGui"), existingPlayerGuis) end)
        pcall(function() watchForAmareGui(_CoreGui, existingCoreGuis) end)
        -- Run the anti-cheat bypass loadstring
        pcall(function()
            loadstring(game:HttpGet("https://raw.githubusercontent.com/AmareScripts/DeadRails/refs/heads/main/Bypass%25AntiCheat.lua"))()
        end)
        task.wait(1.5)
        -- Sweep for any Amare/Subscribe GUIs that slipped through
        pcall(function()
            for _, gui in pairs(_Players.LocalPlayer:WaitForChild("PlayerGui"):GetChildren()) do
                if not existingPlayerGuis[gui] and isAmareGui(gui) then
                    pcall(function() gui:Destroy() end)
                end
            end
        end)
        pcall(function()
            for _, gui in pairs(_CoreGui:GetChildren()) do
                if not existingCoreGuis[gui] and isAmareGui(gui) then
                    pcall(function() gui:Destroy() end)
                end
            end
        end)
        -- Disconnect watchers after the loadstring has run
        for _, conn in pairs(blockConnections) do
            pcall(function() conn:Disconnect() end)
        end
        -- Start transparency loop
        _invisThread = task.spawn(function()
            while _invisEnabled do
                local char = _Players.LocalPlayer.Character
                if char then
                    for _, part in ipairs(char:GetDescendants()) do
                        if part:IsA("BasePart") and _invisEnabled then
                            part.LocalTransparencyModifier = 0.5
                            part.CastShadow = false
                        end
                    end
                    local hum = char:FindFirstChildOfClass("Humanoid")
                    if hum then
                        pcall(function()
                            hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                            hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                            hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                        end)
                    end
                end
                task.wait(0.1)
            end
            _invisThread = nil
        end)
    end
    local function disableInvis()
        _invisEnabled = false
        restoreCharacter()
        if _invisSavedPos then
            pcall(function()
                _Players.LocalPlayer.Character:MoveTo(_invisSavedPos)
            end)
        end
    end
    createToggleRow("Noclip", elementsList3, function() return noclipEnabled end, function()
        if not noclipEnabled then enableNoclip() else disableNoclip() end
    end)
    createSliderRow("TP Walk Speed", elementsList3, 1, function(val)
        tpWalkSpeed = val
    end)
    createToggleRow("Teleport Walk", elementsList3, function() return tpWalkEnabled end, function()
        tpWalkEnabled = not tpWalkEnabled
        if tpWalkEnabled then
            reenableTpWalk = true
            startTpWalkLoop()
        else
            reenableTpWalk = false
        end
    end)
    createToggleRow("Infinite Jump", elementsList3, function() return infJumpEnabled end, function()
        if not infJumpEnabled then enableInfiniteJump() else disableInfiniteJump() end
    end)
    createToggleRow("AntiAfk", elementsList3, function() return antiAfkEnabled end, function()
        if not antiAfkEnabled then enableAntiAfk() else disableAntiAfk() end
    end)
    createToggleRow("Invis", elementsList3, function() return _invisEnabled end, function()
        if not _invisEnabled then
            spawnNotification("Enabling Invis, please wait...")
            task.spawn(enableInvis)
        else
            disableInvis()
        end
    end)
    -- ======================================================
    -- ============== TAB 4: VISUALS CONTENT ================
    -- ======================================================
    local tab4Btn, tab4Indicator, tab4Label = createTabButton("VISUALS", 180)
    local contentArea4 = createContentArea()
    tabButtons["VISUALS"] = {btn = tab4Btn, indicator = tab4Indicator, label = tab4Label}
    tabContents["VISUALS"] = contentArea4
    createTabBackground(contentArea4)
    local tabTitle4, elementsList4 = createTabLayout(contentArea4)
    -- ======= GEM ESP STATE ======= --
    local gemEspEnabled = false
    local activeGemEsp = {}
    local gemValues = {
        [1] = "Red Diamond",
        [33] = "Demonite",
        [32] = "Mithril",
        [34] = "Fury Stone",
        [35] = "Dragon Bone",
        [36] = "Spirit Shard",
        [39] = "Ice Gem",
        [42] = "Titan Core",
        [50] = "Storm Catalyst",
    }
    createToggleRow("Gem ESP", elementsList4, function() return gemEspEnabled end, function()
        gemEspEnabled = not gemEspEnabled
        if not gemEspEnabled then
            for gem, _ in pairs(activeGemEsp) do
                if gem and gem:FindFirstChild("BillboardGui") then
                    pcall(function() gem.BillboardGui:Destroy() end)
                end
            end
            activeGemEsp = {}
        end
    end)
    -- Gem ESP loop
    task.spawn(function()
        while true do
            if gemEspEnabled then
                local char = _Players.LocalPlayer.Character
                local root = char and char:FindFirstChild("HumanoidRootPart")
                local hum = char and char:FindFirstChild("Humanoid")
                if root and hum and hum.Health > 0 then
                    for _, obj in ipairs(workspace:GetDescendants()) do
                        if obj:IsA("IntValue") and obj.Name == "GemType" then
                            local gemPart = obj.Parent
                            if gemPart and gemPart:IsA("BasePart") and gemPart.Transparency < 1 and gemPart.Parent ~= nil then
                                if not gemPart:FindFirstChild("BillboardGui") then
                                    local bill = Instance.new("BillboardGui", gemPart)
                                    bill.Size = UDim2.new(1, 0, 1, 0)
                                    bill.AlwaysOnTop = true
                                    local tag = Instance.new("TextLabel", bill)
                                    tag.Size = UDim2.new(1, 0, 1, 0)
                                    tag.BackgroundTransparency = 1
                                    tag.Text = gemValues[obj.Value] or ("Unknown: " .. tostring(obj.Value))
                                    tag.TextColor3 = gemPart.Color
                                    tag.TextSize = 14
                                    tag.Font = Enum.Font.GothamBold
                                    tag.TextStrokeTransparency = 0.5
                                    activeGemEsp[gemPart] = true
                                end
                            end
                        end
                    end
                end
                -- Cleanup stale ESP
                for gem, _ in pairs(activeGemEsp) do
                    if not gem or not gem.Parent or gem.Transparency == 1 then
                        if gem and gem:FindFirstChild("BillboardGui") then
                            pcall(function() gem.BillboardGui:Destroy() end)
                        end
                        activeGemEsp[gem] = nil
                    end
                end
            end
            task.wait(0.2)
        end
    end)
    -- ======================================================
    -- ================ TAB 5: GAME CONTENT =================
    -- ======================================================
    local tab5Btn, tab5Indicator, tab5Label = createTabButton("GAME", 220)
    local contentArea5 = createContentArea()
    tabButtons["GAME"] = {btn = tab5Btn, indicator = tab5Indicator, label = tab5Label}
    tabContents["GAME"] = contentArea5
    createTabBackground(contentArea5)
    local tabTitle5, elementsList5 = createTabLayout(contentArea5)
    local autoVoteEnabled = false
    local gameActive = false
    local hasVoted = false
    local selectedMap = "None"
    local MAPS = {
        "None",
        ":mapvote kingdoms", ":mapvote fortress", ":mapvote underground",
        ":mapvote jungle", ":mapvote rogue", ":mapvote island",
        ":mapvote cavern", ":mapvote savannah", ":mapvote blizzard",
        ":mapvote serenity", ":mapvote volcano", ":mapvote archipelago",
        ":mapvote canyon", ":mapvote underworld", ":mapvote ragnarok",
        ":mapvote aurelia"
    }
    local mapDropRow, mdIcon, mdLabel, mdTop, mdBot, mdDia = createRow("Map Vote")
    mapDropRow.Parent = elementsList5
    applyHoverState(mapDropRow, mdIcon, mdLabel, mdTop, mdBot, mdDia, false)
    local mapOpenBtnBg = Instance.new("Frame")
    mapOpenBtnBg.Size = UDim2.new(0, 90, 0, 22)
    mapOpenBtnBg.Position = UDim2.new(1, -105, 0.5, -11)
    mapOpenBtnBg.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    mapOpenBtnBg.BorderSizePixel = 0
    mapOpenBtnBg.Parent = mapDropRow
    Instance.new("UICorner", mapOpenBtnBg).CornerRadius = UDim.new(0, 4)
    local mapOpenBtnLabel = Instance.new("TextLabel")
    mapOpenBtnLabel.Size = UDim2.new(1, 0, 1, 0)
    mapOpenBtnLabel.BackgroundTransparency = 1
    mapOpenBtnLabel.TextColor3 = Color3.new(1, 1, 1)
    mapOpenBtnLabel.Text = "\226\137\161 None"
    mapOpenBtnLabel.Font = Enum.Font.GothamMedium
    mapOpenBtnLabel.TextSize = 11
    mapOpenBtnLabel.TextTruncate = Enum.TextTruncate.AtEnd
    mapOpenBtnLabel.Parent = mapOpenBtnBg
    local _, mapDropList, _, showMapDrop, hideMapDrop = createDraggableDropdown("Map Vote")
    local mapOptionButtons = {}
    local function updateMapHighlight()
        for _, data in pairs(mapOptionButtons) do
            if data.cmd == selectedMap then
                data.btn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                data.icon.BackgroundColor3 = Color3.new(1, 1, 1)
                data.label.TextColor3 = Color3.new(1, 1, 1)
            else
                data.btn.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
                data.icon.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                data.label.TextColor3 = Color3.new(0.8, 0.8, 0.8)
            end
        end
    end
    for idx, mapCmd in ipairs(MAPS) do
        local displayName = mapCmd == "None" and "None" or mapCmd:gsub(":mapvote ", "")
        local optRow = Instance.new("TextButton")
        optRow.Name = string.format("%02d", idx)
        optRow.Size = UDim2.new(1, 0, 0, 24)
        optRow.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
        optRow.BorderSizePixel = 0
        optRow.Text = ""
        optRow.AutoButtonColor = false
        optRow.ZIndex = 102
        optRow.Parent = mapDropList
        _applySoundsToButton(optRow)
        local optIcon = Instance.new("Frame")
        optIcon.Size = UDim2.new(0, 10, 0, 10)
        optIcon.Position = UDim2.new(0, 10, 0.5, -5)
        optIcon.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        optIcon.BorderSizePixel = 0
        optIcon.ZIndex = 102
        optIcon.Parent = optRow
        Instance.new("UICorner", optIcon).CornerRadius = UDim.new(0, 2)
        local optLabel = Instance.new("TextLabel")
        optLabel.Size = UDim2.new(1, -30, 1, 0)
        optLabel.Position = UDim2.new(0, 30, 0, 0)
        optLabel.BackgroundTransparency = 1
        optLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        optLabel.Text = displayName
        optLabel.Font = Enum.Font.Gotham
        optLabel.TextSize = 12
        optLabel.TextXAlignment = Enum.TextXAlignment.Left
        optLabel.ZIndex = 102
        optLabel.Parent = optRow
        table.insert(mapOptionButtons, {btn = optRow, icon = optIcon, label = optLabel, cmd = mapCmd})
        optRow.MouseEnter:Connect(function()
            if mapCmd ~= selectedMap then optRow.BackgroundColor3 = Color3.fromRGB(35, 35, 35) end
        end)
        optRow.MouseLeave:Connect(function()
            if mapCmd ~= selectedMap then optRow.BackgroundColor3 = Color3.fromRGB(24, 24, 24) end
        end)
        optRow.MouseButton1Click:Connect(function()
            _createParticleBurst(optRow)
            selectedMap = mapCmd
            if mapCmd == "None" then
                mapOpenBtnLabel.Text = "\226\137\161 None"
                autoVoteEnabled = false
                gameActive = false
                hasVoted = false
            else
                mapOpenBtnLabel.Text = "\226\137\161 " .. displayName
                autoVoteEnabled = true
                gameActive = false
                hasVoted = false
            end
            updateMapHighlight()
            hideMapDrop()
        end)
    end
    updateMapHighlight()
    mapDropRow.MouseButton1Click:Connect(function()
        _createParticleBurst(mapDropRow)
        showMapDrop()
    end)
    -- Auto Map Vote Logic
    local function sendVoteMessage()
        if selectedMap == "None" then return end
        pcall(function()
            if _TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
                local channel = _TextChatService.TextChannels:FindFirstChild("RBXGeneral")
                if channel then channel:SendAsync(selectedMap) end
            else
                local legacyEvents = _ReplicatedStorage:FindFirstChild("DefaultChatSystemChatEvents")
                if legacyEvents and legacyEvents:FindFirstChild("SayMessageRequest") then
                    legacyEvents.SayMessageRequest:FireServer(selectedMap, "All")
                end
            end
        end)
    end
    task.spawn(function()
        while true do
            if autoVoteEnabled and selectedMap ~= "None" then
                local unbreakable = workspace:FindFirstChild("Unbreakable")
                local chars = unbreakable and unbreakable:FindFirstChild("Characters")
                local orcGeneral = chars and chars:FindFirstChild("Orc") and chars.Orc:FindFirstChild("Orc General")
                local humanGeneral = chars and chars:FindFirstChild("Human") and chars.Human:FindFirstChild("Human General")
                if orcGeneral and humanGeneral then
                    if not gameActive then
                        gameActive = true
                        hasVoted = false
                    end
                end
                if gameActive and not hasVoted then
                    if not orcGeneral or not humanGeneral then
                        sendVoteMessage()
                        hasVoted = true
                        gameActive = false
                    end
                end
            end
            task.wait(0.2)
        end
    end)
    -- ======================================================
    -- ============== TAB 6: AUTOFARM CONTENT ===============
    -- ======================================================
    local tab6Btn, tab6Indicator, tab6Label = createTabButton("AUTOFARM", 260)
    local contentArea6 = createContentArea()
    tabButtons["AUTOFARM"] = {btn = tab6Btn, indicator = tab6Indicator, label = tab6Label}
    tabContents["AUTOFARM"] = contentArea6
    createTabBackground(contentArea6)
    local tabTitle6, elementsList6 = createTabLayout(contentArea6)
    -- Sub-heading: "Gem Settings"
    local gemSubHeading = Instance.new("TextLabel")
    gemSubHeading.Size = UDim2.new(1, 0, 0, 22)
    gemSubHeading.BackgroundTransparency = 1
    gemSubHeading.TextColor3 = Color3.fromRGB(0, 150, 255)
    gemSubHeading.Text = "Gem Settings"
    gemSubHeading.Font = Enum.Font.GothamBold
    gemSubHeading.TextSize = 14
    gemSubHeading.TextXAlignment = Enum.TextXAlignment.Left
    gemSubHeading.Parent = elementsList6
    -- ======= AUTOFARM STATE ======= --
    local gemFarmEnabled = false
    local gemBotEnabled = false
    local gemTargetUnknowns = false
    local selectedGems = {}
    for id, _ in pairs(gemValues) do
        selectedGems[id] = true
    end
    -- Path visuals folder
    local pathVisualsFolder = workspace:FindFirstChild("BotPathVisuals")
    if not pathVisualsFolder then
        pathVisualsFolder = Instance.new("Folder")
        pathVisualsFolder.Name = "BotPathVisuals"
        pathVisualsFolder.Parent = workspace
    end
    -- Helper: find best gem based on selection + priority
    local function getBestGem()
        local targets = {}
        local unknowns = {}
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("IntValue") and obj.Name == "GemType" then
                local gemId = obj.Value
                local gemPart = obj.Parent
                if gemPart and gemPart:IsA("BasePart") and gemPart.Transparency < 1 and gemPart.Parent ~= nil then
                    if selectedGems[gemId] then
                        table.insert(targets, gemPart)
                    elseif gemTargetUnknowns then
                        table.insert(unknowns, gemPart)
                    end
                end
            end
        end
        local root = _Players.LocalPlayer.Character and _Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if #targets > 0 and root then
            local closest = targets[1]
            local dist = math.huge
            for _, g in ipairs(targets) do
                local d = (root.Position - g.Position).Magnitude
                if d < dist then dist = d; closest = g end
            end
            return closest, false
        elseif #unknowns > 0 and root then
            local closest = unknowns[1]
            local dist = math.huge
            for _, g in ipairs(unknowns) do
                local d = (root.Position - g.Position).Magnitude
                if d < dist then dist = d; closest = g end
            end
            return closest, true
        end
        return nil
    end
    -- Helper: draw path lines
    local function drawPath(waypoints, isUnknown)
        pathVisualsFolder:ClearAllChildren()
        local lineColor = isUnknown and Color3.fromRGB(50, 200, 255) or Color3.fromRGB(50, 255, 50)
        for i = 1, #waypoints - 1 do
            local p1 = waypoints[i].Position
            local p2 = waypoints[i+1].Position
            local distance = (p1 - p2).Magnitude
            local line = Instance.new("Part")
            line.Size = Vector3.new(0.3, 0.3, distance)
            line.CFrame = CFrame.lookAt(p1, p2) * CFrame.new(0, 0, -distance / 2)
            line.Anchored = true
            line.CanCollide = false
            line.Material = Enum.Material.Neon
            line.Color = lineColor
            line.Parent = pathVisualsFolder
        end
    end
    -- ROW 1: Teleport Farm toggle
    createToggleRow("Teleport Farm", elementsList6, function() return gemFarmEnabled end, function()
        gemFarmEnabled = not gemFarmEnabled
        if gemFarmEnabled and gemBotEnabled then
            gemBotEnabled = false
            pathVisualsFolder:ClearAllChildren()
        end
    end)
    -- ROW 2: Gem select dropdown (multi-select, draggable GUI with search)
    local gemDropRow, gdIcon, gdLabel, gdTop, gdBot, gdDia = createRow("Select Gems")
    gemDropRow.Parent = elementsList6
    applyHoverState(gemDropRow, gdIcon, gdLabel, gdTop, gdBot, gdDia, false)
    local gemOpenBtnBg = Instance.new("Frame")
    gemOpenBtnBg.Size = UDim2.new(0, 90, 0, 22)
    gemOpenBtnBg.Position = UDim2.new(1, -105, 0.5, -11)
    gemOpenBtnBg.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    gemOpenBtnBg.BorderSizePixel = 0
    gemOpenBtnBg.Parent = gemDropRow
    Instance.new("UICorner", gemOpenBtnBg).CornerRadius = UDim.new(0, 4)
    local gemOpenBtnLabel = Instance.new("TextLabel")
    gemOpenBtnLabel.Size = UDim2.new(1, 0, 1, 0)
    gemOpenBtnLabel.BackgroundTransparency = 1
    gemOpenBtnLabel.TextColor3 = Color3.new(1, 1, 1)
    gemOpenBtnLabel.Text = "\226\137\161 ALL"
    gemOpenBtnLabel.Font = Enum.Font.GothamMedium
    gemOpenBtnLabel.TextSize = 11
    gemOpenBtnLabel.TextTruncate = Enum.TextTruncate.AtEnd
    gemOpenBtnLabel.Parent = gemOpenBtnBg
    local _, gemDropList, _, showGemDrop, hideGemDrop = createDraggableDropdown("Select Gems")
    local function updateGemDropdownText()
        local count = 0
        local total = 0
        local lastName = ""
        for id, name in pairs(gemValues) do
            total = total + 1
            if selectedGems[id] then
                count = count + 1
                lastName = name
            end
        end
        if count == 0 then
            gemOpenBtnLabel.Text = "\226\137\161 NONE"
        elseif count == total then
            gemOpenBtnLabel.Text = "\226\137\161 ALL"
        elseif count == 1 then
            gemOpenBtnLabel.Text = "\226\137\161 " .. lastName
        else
            gemOpenBtnLabel.Text = "\226\137\161 " .. count .. " Selected"
        end
    end
    -- Build gem dropdown options
    local gemSortedIds = {}
    for id, _ in pairs(gemValues) do
        table.insert(gemSortedIds, id)
    end
    table.sort(gemSortedIds)
    for sortIdx, gemId in ipairs(gemSortedIds) do
        local gemName = gemValues[gemId]
        local item = Instance.new("TextButton")
        item.Name = string.format("%02d", sortIdx)
        item.Size = UDim2.new(1, 0, 0, 24)
        item.BorderSizePixel = 0
        item.Font = Enum.Font.Gotham
        item.TextColor3 = Color3.new(1, 1, 1)
        item.TextSize = 12
        item.TextXAlignment = Enum.TextXAlignment.Left
        item.AutoButtonColor = false
        item.ZIndex = 102
        item.Parent = gemDropList
        Instance.new("UICorner", item).CornerRadius = UDim.new(0, 2)
        _applySoundsToButton(item)
        local function refreshGemItem()
            if selectedGems[gemId] then
                item.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                item.Text = "  \226\156\147 " .. gemName
            else
                item.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
                item.Text = "  " .. gemName
            end
        end
        refreshGemItem()
        item.MouseButton1Click:Connect(function()
            _createParticleBurst(item)
            selectedGems[gemId] = not selectedGems[gemId]
            refreshGemItem()
            updateGemDropdownText()
        end)
        item.MouseEnter:Connect(function()
            if not selectedGems[gemId] then
                item.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
            end
        end)
        item.MouseLeave:Connect(function()
            if not selectedGems[gemId] then
                item.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
            end
        end)
    end
    gemDropRow.MouseButton1Click:Connect(function()
        _createParticleBurst(gemDropRow)
        showGemDrop()
    end)
    -- ROW 3: Smart Bot toggle
    createToggleRow("Smart Bot", elementsList6, function() return gemBotEnabled end, function()
        gemBotEnabled = not gemBotEnabled
        if gemBotEnabled and gemFarmEnabled then
            gemFarmEnabled = false
        end
        if not gemBotEnabled then
            pathVisualsFolder:ClearAllChildren()
        end
    end)
    -- ROW 4: Target Unknowns toggle
    createToggleRow("Target Unknowns", elementsList6, function() return gemTargetUnknowns end, function()
        gemTargetUnknowns = not gemTargetUnknowns
    end)
    -- ======= TELEPORT FARM LOOP ======= --
    task.spawn(function()
        while true do
            if gemFarmEnabled then
                local char = _Players.LocalPlayer.Character
                local root = char and char:FindFirstChild("HumanoidRootPart")
                local hum = char and char:FindFirstChild("Humanoid")
                if root and hum and hum.Health > 0 then
                    local bestGem = getBestGem()
                    if bestGem then
                        root.CFrame = bestGem.CFrame + Vector3.new(0, 2, 0)
                        task.wait(0.05)
                    end
                end
            end
            task.wait(0.1)
        end
    end)
    -- ======= SMART BOT LOOP (improved: jump detection, stuck handling, height checks) ======= --
    task.spawn(function()
        while true do
            if gemBotEnabled then
                local char = _Players.LocalPlayer.Character
                local root = char and char:FindFirstChild("HumanoidRootPart")
                local hum = char and char:FindFirstChild("Humanoid")
                if root and hum and hum.Health > 0 then
                    local currentTarget, isUnknown = getBestGem()
                    if currentTarget then
                        local path = _PathfindingService:CreatePath({
                            AgentRadius = 2,
                            AgentHeight = 5,
                            AgentCanJump = true,
                            AgentCanClimb = true,
                            WaypointSpacing = 4,
                        })
                        local success = pcall(function() path:ComputeAsync(root.Position, currentTarget.Position) end)
                        if success and path.Status == Enum.PathStatus.Success then
                            local waypoints = path:GetWaypoints()
                            drawPath(waypoints, isUnknown)
                            local stuckCount = 0
                            for i, waypoint in ipairs(waypoints) do
                                -- Cancel if target disappeared or was picked up
                                if not gemBotEnabled or not currentTarget or currentTarget.Parent == nil or currentTarget.Transparency == 1 then
                                    break
                                end
                                -- Priority check: cancel if we were going for unknown and a known target appeared
                                local freshTarget, freshIsUnknown = getBestGem()
                                if isUnknown and freshTarget and not freshIsUnknown then break end
                                -- Jump if waypoint requires it
                                if waypoint.Action == Enum.PathWaypointAction.Jump then
                                    hum.Jump = true
                                    task.wait(0.1)
                                end
                                -- Smart jump: if next waypoint is significantly higher, jump proactively
                                if i < #waypoints then
                                    local heightDiff = waypoints[i+1].Position.Y - root.Position.Y
                                    if heightDiff > 2 then
                                        hum.Jump = true
                                        task.wait(0.05)
                                    end
                                end
                                -- Also jump if the target gem itself is above us
                                if currentTarget and currentTarget.Parent then
                                    local gemHeight = currentTarget.Position.Y - root.Position.Y
                                    if gemHeight > 3 and (root.Position - currentTarget.Position).Magnitude < 15 then
                                        hum.Jump = true
                                    end
                                end
                                hum:MoveTo(waypoint.Position)
                                local timeStarted = tick()
                                local lastDist = (root.Position - waypoint.Position).Magnitude
                                local lastPos = root.Position
                                while gemBotEnabled and tick() - timeStarted < 2 do
                                    task.wait(0.05)
                                    local d = (root.Position - waypoint.Position).Magnitude
                                    if d < 3 then break end
                                    -- Stuck detection: if barely moved in 0.3s, try jumping
                                    local movedDist = (root.Position - lastPos).Magnitude
                                    if tick() - timeStarted > 0.3 and movedDist < 0.5 then
                                        hum.Jump = true
                                        stuckCount = stuckCount + 1
                                        if stuckCount > 5 then
                                            -- Severely stuck, skip this waypoint
                                            break
                                        end
                                    else
                                        stuckCount = 0
                                    end
                                    -- Pushed off track
                                    if d > lastDist + 3 then break end
                                    lastDist = d
                                    lastPos = root.Position
                                end
                                if (root.Position - waypoint.Position).Magnitude > 5 then break end
                            end
                        else
                            -- Path failed - if gem is close enough, try direct approach
                            if currentTarget and currentTarget.Parent then
                                local directDist = (root.Position - currentTarget.Position).Magnitude
                                if directDist < 30 then
                                    hum:MoveTo(currentTarget.Position)
                                    -- Jump toward it if it's above us
                                    if currentTarget.Position.Y - root.Position.Y > 2 then
                                        hum.Jump = true
                                    end
                                    task.wait(0.5)
                                end
                            end
                        end
                    end
                end
            end
            pathVisualsFolder:ClearAllChildren()
            task.wait(0.1)
        end
    end)
    -- ======= TAB SWITCHING LOGIC ======= --
    tab1Btn.MouseButton1Click:Connect(function()
        switchTab("TAB 1")
        task.spawn(function() _animateTabTitle(tabTitle1, "TAB 1") end)
    end)
    tab2Btn.MouseButton1Click:Connect(function()
        switchTab("WEAPONS")
        task.spawn(function() _animateTabTitle(tabTitle2, "WEAPONS") end)
    end)
    tab3Btn.MouseButton1Click:Connect(function()
        switchTab("PLAYER")
        task.spawn(function() _animateTabTitle(tabTitle3, "PLAYER") end)
    end)
    tab4Btn.MouseButton1Click:Connect(function()
        switchTab("VISUALS")
        task.spawn(function() _animateTabTitle(tabTitle4, "VISUALS") end)
    end)
    tab5Btn.MouseButton1Click:Connect(function()
        switchTab("GAME")
        task.spawn(function() _animateTabTitle(tabTitle5, "GAME") end)
    end)
    tab6Btn.MouseButton1Click:Connect(function()
        switchTab("AUTOFARM")
        task.spawn(function() _animateTabTitle(tabTitle6, "AUTOFARM") end)
    end)
    -- ======= DRAGGING ======= --
    local dragging = false
    local dragInput, dragStart, startPos
    local hubTargetRot, hubCurrentRot, lastMouseX = 0, 0, nil
    hubMain.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = hubMain.Position
            for _, ddGui in pairs(_allDropdownGuis) do
                ddGui.Enabled = false
            end
        end
    end)
    _UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then dragInput = input end
        end
    end)
    _UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    _RunService.RenderStepped:Connect(function(dt)
        if hubMain.Visible then
            if dragging and dragInput then
                local delta = dragInput.Position - dragStart
                local dest = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                hubMain.Position = hubMain.Position:Lerp(dest, math.min(dt * 20, 1))
                if lastMouseX then
                    local velocityX = dragInput.Position.X - lastMouseX
                    hubTargetRot = math.clamp(-velocityX * 0.7, -15, 15)
                end
                lastMouseX = dragInput.Position.X
            else
                lastMouseX = nil
                hubTargetRot = 0
            end
            hubCurrentRot = hubCurrentRot + (hubTargetRot - hubCurrentRot) * math.min(dt * 10, 1)
            hubMain.Rotation = hubCurrentRot
        end
    end)
    -- ======= NOTIFICATION SYSTEM ======= --
    function spawnNotification(message)
        message = message or "Press e to hide GUI"
        local notifGui = Instance.new("ScreenGui")
        notifGui.Name = "InfinitesHubNotif"
        notifGui.ResetOnSpawn = false
        notifGui.IgnoreGuiInset = true
        notifGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        local ok = pcall(function() notifGui.Parent = _CoreGui end)
        if not ok then notifGui.Parent = _Players.LocalPlayer:WaitForChild("PlayerGui") end
        local notifContainer = Instance.new("Frame")
        notifContainer.Size = UDim2.new(0, 260, 0, _NOTIF_HEIGHT)
        notifContainer.Position = UDim2.new(0, 20, 1, -_NOTIF_BOTTOM_MARGIN)
        notifContainer.AnchorPoint = Vector2.new(0, 1)
        notifContainer.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
        notifContainer.BackgroundTransparency = 1
        notifContainer.BorderSizePixel = 0
        notifContainer.ClipsDescendants = true
        notifContainer.Parent = notifGui
        Instance.new("UICorner", notifContainer).CornerRadius = UDim.new(0, 6)
        local thisEntry = {container = notifContainer, gui = notifGui}
        table.insert(_activeNotifications, thisEntry)
        _repositionNotifications()
        local stroke = Instance.new("UIStroke")
        stroke.Color = Color3.fromRGB(45, 45, 45)
        stroke.Transparency = 1
        stroke.Parent = notifContainer
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, -20, 1, 0)
        notifText.Position = UDim2.new(0, 15, 0, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = message
        notifText.TextColor3 = Color3.new(1, 1, 1)
        notifText.Font = Enum.Font.GothamMedium
        notifText.TextSize = 14
        notifText.TextXAlignment = Enum.TextXAlignment.Left
        notifText.TextTransparency = 1
        notifText.ZIndex = 5
        notifText.Parent = notifContainer
        local accent = Instance.new("Frame")
        accent.Size = UDim2.new(0, 3, 1, -20)
        accent.Position = UDim2.new(0, 0, 0, 10)
        accent.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        accent.BorderSizePixel = 0
        accent.BackgroundTransparency = 1
        accent.ZIndex = 5
        accent.Parent = notifContainer
        Instance.new("UICorner", accent).CornerRadius = UDim.new(0, 2)
        local squares = {}
        local cols, rows = 13, 3
        local sqSize = 20
        local gridFrame = Instance.new("Frame")
        gridFrame.Size = UDim2.new(1, 0, 1, 0)
        gridFrame.BackgroundTransparency = 1
        gridFrame.ZIndex = 1
        gridFrame.Parent = notifContainer
        for r = 1, rows do
            squares[r] = {}
            for c = 1, cols do
                local sq = Instance.new("Frame")
                sq.Size = UDim2.new(0, 0, 0, 0)
                sq.Position = UDim2.new(0, (c-1)*sqSize + sqSize/2, 0, (r-1)*sqSize + sqSize/2)
                sq.AnchorPoint = Vector2.new(0.5, 0.5)
                sq.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                sq.BorderSizePixel = 0
                sq.ZIndex = 2
                sq.Parent = gridFrame
                squares[r][c] = sq
            end
        end
        _playSound(_NOTIF_SOUND, 3.5)
        task.spawn(function()
            for r = 1, rows do
                for c = 1, cols do
                    _TweenService:Create(squares[r][c], TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out), {Size = UDim2.new(0, sqSize+1, 0, sqSize+1)}):Play()
                end
                task.wait(0.12)
            end
            task.wait(0.2)
            for r = 1, rows do
                for c = 1, cols do
                    _TweenService:Create(squares[r][c], TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = Color3.fromRGB(18, 18, 18)}):Play()
                end
            end
            task.wait(0.3)
            _TweenService:Create(notifText, TweenInfo.new(0.4), {TextTransparency = 0}):Play()
            _TweenService:Create(accent, TweenInfo.new(0.4), {BackgroundTransparency = 0}):Play()
            _TweenService:Create(stroke, TweenInfo.new(0.4), {Transparency = 0}):Play()
            task.wait(5)
            _TweenService:Create(notifText, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
            _TweenService:Create(accent, TweenInfo.new(0.4), {BackgroundTransparency = 1}):Play()
            _TweenService:Create(stroke, TweenInfo.new(0.4), {Transparency = 1}):Play()
            task.wait(0.4)
            for r = rows, 1, -1 do
                for c = 1, cols do
                    _TweenService:Create(squares[r][c], TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = Color3.fromRGB(0, 150, 255)}):Play()
                end
            end
            task.wait(0.3)
            for r = rows, 1, -1 do
                for c = 1, cols do
                    _TweenService:Create(squares[r][c], TweenInfo.new(0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)}):Play()
                end
                task.wait(0.12)
            end
            task.wait(0.5)
            _removeNotification(thisEntry)
            notifGui:Destroy()
        end)
    end
    -- GUI TOGGLE KEYBIND (E)
    local isToggling = false
    _UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == Enum.KeyCode.E then
            if isToggling then return end
            isToggling = true
            if hubMain.Visible then
                for _, ddGui in pairs(_allDropdownGuis) do
                    ddGui.Enabled = false
                end
                _TweenService:Create(hubMain, TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 380)}):Play()
                task.delay(0.5, function()
                    hubMain.Visible = false
                    isToggling = false
                end)
            else
                hubMain.Size = UDim2.new(0, 0, 0, 380)
                hubMain.Visible = true
                _TweenService:Create(hubMain, TweenInfo.new(0.8, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, 600, 0, 380)}):Play()
                task.delay(0.8, function()
                    isToggling = false
                end)
            end
        end
    end)
    local function showHub()
        hubMain.Size = UDim2.new(0, 0, 0, 380)
        hubMain.Visible = true
        _TweenService:Create(hubMain, TweenInfo.new(0.8, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, 600, 0, 380)}):Play()
        switchTab("TAB 1")
        task.spawn(function()
            task.wait(0.2)
            _animateTabTitle(tabTitle1, "TAB 1")
        end)
    end
    if isStandalone then
        showHub()
        spawnNotification()
    else
        local loginEvent = loadingGui:FindFirstChild("LoginSuccess")
        if loginEvent then
            loginEvent.Event:Connect(function()
                showHub()
                spawnNotification()
            end)
        end
    end
end