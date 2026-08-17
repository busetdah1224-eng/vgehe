--[[
   KuRu SLOTTED - Fully Reconstructed
   Made By TooZe  discord.gg/kuruu

   All UI elements, tabs, toggles, backgrounds,
   selectors, dragging, keybinds, and animations.
]]

local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer      = Players.LocalPlayer

------------------------------------------------------------------------
-- THEME
------------------------------------------------------------------------
local ACCENT       = Color3.fromRGB(255,60,172)
local BG_DARK      = Color3.fromRGB(10,11,13)
local ROW_BG       = Color3.fromRGB(16,17,20)
local CARD_STROKE  = Color3.fromRGB(34,37,43)
local TEXT_WHITE    = Color3.fromRGB(255,255,255)
local TEXT_PRIMARY  = Color3.fromRGB(238,238,238)
local TEXT_DIM      = Color3.fromRGB(125,125,125)
local TEXT_SECTION  = Color3.fromRGB(200,205,215)
local BTN_BG       = Color3.fromRGB(18,18,18)
local TOGGLE_OFF   = Color3.fromRGB(40,40,40)
local TOGGLE_KNOB  = Color3.fromRGB(185,185,185)
local KNOB_ON      = Color3.fromRGB(12,12,12)
local GRAD_TOP     = Color3.fromRGB(20,21,25)
local GRAD_BOT     = Color3.fromRGB(13,14,17)

local TWEEN_FAST = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local TWEEN_MED  = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

------------------------------------------------------------------------
-- IMAGE POOLS (for changeable backgrounds)
------------------------------------------------------------------------
local TAB_BG_IMAGES = {
   "rbxassetid://90828308794279",
   "rbxassetid://139819905455883",
   "rbxassetid://108037416708175",
}
local GENERAL_BG_IMAGES = {
   "rbxassetid://139819905455883",
   "rbxassetid://90828308794279",
   "rbxassetid://108037416708175",
}

------------------------------------------------------------------------
-- STATE
------------------------------------------------------------------------
local Toggles = {}
local UIToggleKey = Enum.KeyCode.LeftControl

------------------------------------------------------------------------
-- STYLE HELPERS
------------------------------------------------------------------------
local function cardStyle(f)
   local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,9); c.Parent = f
   local s = Instance.new("UIStroke"); s.Color = CARD_STROKE; s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Transparency = 0.4; s.Parent = f
   local g = Instance.new("UIGradient"); g.Color = ColorSequence.new(GRAD_TOP,GRAD_BOT); g.Rotation = 90; g.Transparency = NumberSequence.new(0.08,0.08); g.Parent = f
end

local function smallBtn(p)
   local b = Instance.new("TextButton")
   b.Position = p.Pos or UDim2.new(0,0,0,0); b.Size = p.Size or UDim2.new(0,40,0,23)
   b.BackgroundColor3 = p.Bg or BTN_BG; b.BorderSizePixel = 0
   b.Text = p.Text or ""; b.TextColor3 = p.Col or TEXT_DIM; b.TextSize = p.TS or 11
   b.Font = Enum.Font.GothamBold; b.AutoButtonColor = false; b.ZIndex = p.Z or 1; b.Parent = p.Parent
   local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,p.CR or 6); c.Parent = b
   local s = Instance.new("UIStroke"); s.Color = p.SC or CARD_STROKE; s.Thickness = p.ST or 1
   s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Transparency = p.STr or 0.5; s.Parent = b
   return b
end

local function accentBar(parent, on)
   local b = Instance.new("Frame")
   b.Position = UDim2.new(0,0,0.5,-11); b.Size = UDim2.new(0,3,0,22)
   b.BackgroundColor3 = on and ACCENT or TEXT_WHITE
   b.BackgroundTransparency = on and 0 or 1; b.BorderSizePixel = 0; b.Parent = parent
   local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,2); c.Parent = b
   return b
end

local function autoCanvas(scroll)
   local lay = scroll:FindFirstChildOfClass("UIListLayout"); if not lay then return end
   local pad = scroll:FindFirstChildOfClass("UIPadding")
   local function upd()
       local h = lay.AbsoluteContentSize.Y + (pad and pad.PaddingBottom.Offset or 0)
       scroll.CanvasSize = UDim2.new(0,0,0,h)
   end
   lay:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(upd)
   task.defer(upd)
end

------------------------------------------------------------------------
-- ROW BUILDERS
------------------------------------------------------------------------

local function sectionHeader(parent, text)
   local r = Instance.new("Frame"); r.Size = UDim2.new(1,0,0,24); r.BackgroundTransparency = 1; r.Parent = parent
   local b = Instance.new("Frame"); b.Position = UDim2.new(0,0,0.5,-6); b.Size = UDim2.new(0,3,0,13)
   b.BackgroundColor3 = ACCENT; b.BorderSizePixel = 0; b.Parent = r
   Instance.new("UICorner",b).CornerRadius = UDim.new(0,2)
   local l = Instance.new("TextLabel"); l.Position = UDim2.new(0,12,0,0); l.Size = UDim2.new(1,-12,1,0)
   l.BackgroundTransparency = 1; l.Text = text; l.TextColor3 = TEXT_SECTION; l.TextSize = 11
   l.Font = Enum.Font.GothamBold; l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = r
   return r
end

local function inputRow(parent, label, def, hidden)
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,44)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0
   if hidden then r.Visible = false end; r.Parent = parent; cardStyle(r)
   local l = Instance.new("TextLabel"); l.Position = UDim2.new(0,13,0,0); l.Size = UDim2.new(1,-84,1,0)
   l.BackgroundTransparency = 1; l.Text = label; l.TextColor3 = TEXT_PRIMARY; l.TextSize = 13
   l.Font = Enum.Font.GothamMedium; l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = r
   local bx = Instance.new("TextBox"); bx.Position = UDim2.new(1,-66,0.5,-12); bx.Size = UDim2.new(0,56,0,25)
   bx.BackgroundColor3 = BTN_BG; bx.BorderSizePixel = 0; bx.Text = def; bx.TextColor3 = ACCENT
   bx.TextSize = 13; bx.Font = Enum.Font.GothamBold; bx.Parent = r
   Instance.new("UICorner",bx).CornerRadius = UDim.new(0,6)
   local s = Instance.new("UIStroke"); s.Color = CARD_STROKE; s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Transparency = 0.5; s.Parent = bx
   return r, bx
end

local function toggleRow(parent, label, keybind, on, hasExpand)
   local h = keybind and 44 or 46
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,h)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local bar = accentBar(r, on)
   local lw = keybind and UDim2.new(1,-120,1,0) or UDim2.new(1,-74,1,0)
   if hasExpand then lw = UDim2.new(1,-108,1,0) end
   local l = Instance.new("TextLabel"); l.Position = UDim2.new(0,14,0,0); l.Size = lw; l.BackgroundTransparency = 1
   l.Text = label; l.TextColor3 = TEXT_PRIMARY; l.TextSize = 14; l.Font = Enum.Font.GothamMedium
   l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = r

   local tb = Instance.new("TextButton"); tb.Position = UDim2.new(1,-54,0.5,-11); tb.Size = UDim2.new(0,44,0,22)
   tb.BackgroundColor3 = on and ACCENT or TOGGLE_OFF; tb.BorderSizePixel = 0; tb.Text = ""; tb.AutoButtonColor = false; tb.Parent = r
   Instance.new("UICorner",tb).CornerRadius = UDim.new(0,11)
   local knob = Instance.new("Frame"); knob.Size = UDim2.new(0,16,0,16); knob.BorderSizePixel = 0
   knob.Position = on and UDim2.new(1,-19,0.5,-8) or UDim2.new(0,3,0.5,-8)
   knob.BackgroundColor3 = on and KNOB_ON or TOGGLE_KNOB; knob.Parent = tb
   Instance.new("UICorner",knob)

   local state = on
   local function set(v)
       state = v
       TweenService:Create(tb, TWEEN_FAST, {BackgroundColor3 = v and ACCENT or TOGGLE_OFF}):Play()
       TweenService:Create(knob, TWEEN_FAST, {Position = v and UDim2.new(1,-19,0.5,-8) or UDim2.new(0,3,0.5,-8), BackgroundColor3 = v and KNOB_ON or TOGGLE_KNOB}):Play()
       TweenService:Create(bar, TWEEN_FAST, {BackgroundTransparency = v and 0 or 1}):Play()
       bar.BackgroundColor3 = ACCENT
   end
   tb.MouseButton1Click:Connect(function() set(not state) end)

   if keybind then
       local kb = smallBtn({Parent=r, Pos=UDim2.new(1,-104,0.5,-11), Size=UDim2.new(0,40,0,23), Text=keybind})
       local curKey = keybind; local listening = false
       kb.MouseButton1Click:Connect(function()
           if listening then return end; listening = true; kb.Text = "..."
           local cn; cn = UserInputService.InputBegan:Connect(function(inp,gpe)
               if gpe then return end
               if inp.UserInputType == Enum.UserInputType.Keyboard then
                   curKey = inp.KeyCode.Name; kb.Text = curKey; listening = false; cn:Disconnect()
               end
           end)
       end)
       UserInputService.InputBegan:Connect(function(inp,gpe)
           if gpe or listening then return end
           if inp.UserInputType == Enum.UserInputType.Keyboard and inp.KeyCode.Name == curKey then set(not state) end
       end)
   end

   local expandRows
   if hasExpand then
       expandRows = {}; local exp = false
       local eb = smallBtn({Parent=r, Pos=UDim2.new(1,-90,0.5,-11), Size=UDim2.new(0,26,0,22),
           Text="▲", Col=ACCENT, Bg=Color3.fromRGB(24,18,23), SC=ACCENT, ST=1.2, STr=0.15})
       eb.MouseButton1Click:Connect(function()
           exp = not exp; eb.Text = exp and "▼" or "▲"
           for _,rr in ipairs(expandRows) do rr.Visible = exp end
       end)
   end

   Toggles[label] = {get=function() return state end, set=set, subs=expandRows}
   return r, tb, expandRows
end

local function actionRow(parent, label, keybind)
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,42)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local btn = Instance.new("TextButton"); btn.Size = UDim2.new(1,-56,1,0); btn.BackgroundTransparency = 1
   btn.Text = label; btn.TextColor3 = TEXT_PRIMARY; btn.TextSize = 14; btn.Font = Enum.Font.GothamBold; btn.Parent = r
   local bar = accentBar(r, false)
   btn.MouseButton1Click:Connect(function()
       bar.BackgroundColor3 = ACCENT
       TweenService:Create(bar, TWEEN_FAST, {BackgroundTransparency = 0}):Play()
       task.delay(0.3, function() TweenService:Create(bar, TWEEN_FAST, {BackgroundTransparency = 1}):Play() end)
   end)
   if keybind then smallBtn({Parent=r, Pos=UDim2.new(1,-50,0.5,-11), Size=UDim2.new(0,40,0,23), Text=keybind, Z=3}) end
   return r, btn
end

local function fullActionRow(parent, label)
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,42)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local btn = Instance.new("TextButton"); btn.Size = UDim2.new(1,0,1,0); btn.BackgroundTransparency = 1
   btn.Text = label; btn.TextColor3 = TEXT_PRIMARY; btn.TextSize = 14; btn.Font = Enum.Font.GothamBold; btn.Parent = r
   accentBar(r, false)
   return r, btn
end

local function modeSelector(parent, lt, rt)
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,36)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local sl = Instance.new("Frame"); sl.ZIndex = 2; sl.Position = UDim2.new(0,4,0,4); sl.Size = UDim2.new(0.5,-6,1,-8)
   sl.BackgroundColor3 = Color3.fromRGB(235,45,150); sl.BorderSizePixel = 0; sl.Parent = r
   Instance.new("UICorner",sl).CornerRadius = UDim.new(0,7)
   local sg = Instance.new("UIGradient"); sg.Color = ColorSequence.new(Color3.fromRGB(255,78,187),Color3.fromRGB(190,28,112)); sg.Rotation = 90; sg.Parent = sl
   local lb = Instance.new("TextButton"); lb.ZIndex=3; lb.Size=UDim2.new(0.5,0,1,0); lb.BackgroundTransparency=1; lb.BorderSizePixel=0
   lb.Text=lt; lb.TextColor3=TEXT_WHITE; lb.TextSize=11; lb.Font=Enum.Font.GothamBold; lb.AutoButtonColor=false; lb.Parent=r
   local rb = Instance.new("TextButton"); rb.ZIndex=3; rb.Position=UDim2.new(0.5,0,0,0); rb.Size=UDim2.new(0.5,0,1,0); rb.BackgroundTransparency=1; rb.BorderSizePixel=0
   rb.Text=rt; rb.TextColor3=TEXT_DIM; rb.TextSize=11; rb.Font=Enum.Font.GothamBold; rb.AutoButtonColor=false; rb.Parent=r
   local left = true
   local function setM(l) left=l
       TweenService:Create(sl, TWEEN_FAST, {Position=l and UDim2.new(0,4,0,4) or UDim2.new(0.5,2,0,4)}):Play()
       lb.TextColor3 = l and TEXT_WHITE or TEXT_DIM; rb.TextColor3 = l and TEXT_DIM or TEXT_WHITE
   end
   lb.MouseButton1Click:Connect(function() setM(true) end); rb.MouseButton1Click:Connect(function() setM(false) end)
   return r, function() return left end
end

local function arrowSelector(parent, label, options, onChange)
   if type(options)=="string" then options={options} end
   local idx = 1
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,44)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.43,0,0,44); l.BackgroundTransparency=1
   l.Text=label; l.TextColor3=TEXT_PRIMARY; l.TextSize=13; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
   local la = smallBtn({Parent=r, Pos=UDim2.new(1,-174,0,8), Size=UDim2.new(0,29,0,27), Text="<", Col=TEXT_PRIMARY, TS=13, CR=7, SC=CARD_STROKE, STr=0.45})
   local vl = Instance.new("TextLabel"); vl.Position=UDim2.new(1,-141,0,8); vl.Size=UDim2.new(0,102,0,27)
   vl.BackgroundColor3=BTN_BG; vl.BorderSizePixel=0; vl.Text=options[1]; vl.TextColor3=TEXT_PRIMARY; vl.TextSize=10; vl.Font=Enum.Font.GothamBold; vl.Parent=r
   Instance.new("UICorner",vl).CornerRadius=UDim.new(0,7)
   local vs = Instance.new("UIStroke"); vs.Color=CARD_STROKE; vs.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; vs.Transparency=0.45; vs.Parent=vl
   local ra = smallBtn({Parent=r, Pos=UDim2.new(1,-35,0,8), Size=UDim2.new(0,29,0,27), Text=">", Col=TEXT_PRIMARY, TS=13, CR=7, SC=CARD_STROKE, STr=0.45})
   local function upd()
       vl.Text = options[idx]
       if onChange then onChange(idx, options[idx]) end
   end
   la.MouseButton1Click:Connect(function() idx=idx-1; if idx<1 then idx=#options end; upd() end)
   ra.MouseButton1Click:Connect(function() idx=idx+1; if idx>#options then idx=1 end; upd() end)
   return r, vl, function() return idx, options[idx] end
end

local function dropdownRow(parent, label, def)
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,44)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.45,0,0,44); l.BackgroundTransparency=1
   l.Text=label; l.TextColor3=TEXT_PRIMARY; l.TextSize=13; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
   local dd = smallBtn({Parent=r, Pos=UDim2.new(1,-130,0,8), Size=UDim2.new(0,120,0,27), Text=def, Col=TEXT_PRIMARY, TS=11, CR=7, SC=CARD_STROKE, STr=0.45})
   local ca = Instance.new("Frame"); ca.Position=UDim2.new(0,13,0,44); ca.Size=UDim2.new(1,-26,0,0); ca.BackgroundTransparency=1; ca.Parent=r
   local lay = Instance.new("UIListLayout"); lay.Padding=UDim.new(0,4); lay.SortOrder=Enum.SortOrder.LayoutOrder; lay.Parent=ca
   local exp = false
   dd.MouseButton1Click:Connect(function()
       exp = not exp
       TweenService:Create(r, TWEEN_MED, {Size=UDim2.new(1,0,0, exp and (44+lay.AbsoluteContentSize.Y+10) or 44)}):Play()
   end)
   return r, dd, ca
end

local function keybindRow(parent, label, key)
   local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,44)
   r.BackgroundColor3 = ROW_BG; r.BackgroundTransparency = 0.03; r.BorderSizePixel = 0; r.Parent = parent; cardStyle(r)
   local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.45,0,0,44); l.BackgroundTransparency=1
   l.Text=label; l.TextColor3=TEXT_PRIMARY; l.TextSize=13; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
   local kb = smallBtn({Parent=r, Pos=UDim2.new(1,-130,0.5,-12), Size=UDim2.new(0,120,0,25), Text=key, Col=TEXT_DIM, TS=11, CR=6})
   return r, kb
end

------------------------------------------------------------------------
-- PAGE BUILDER
------------------------------------------------------------------------
local function makePage(parent, name, order, vis)
   local p = Instance.new("ScrollingFrame"); p.Name=name; p.Visible=vis~=false; p.LayoutOrder=order
   p.Size=UDim2.new(1,0,1,0); p.BackgroundTransparency=1; p.BorderSizePixel=0
   p.ScrollBarThickness=2; p.ScrollBarImageColor3=ACCENT; p.CanvasSize=UDim2.new(0,0,0,0); p.Parent=parent
   local l = Instance.new("UIListLayout"); l.Padding=UDim.new(0,7); l.SortOrder=Enum.SortOrder.LayoutOrder; l.Parent=p
   local pd = Instance.new("UIPadding"); pd.PaddingBottom=UDim.new(0,10); pd.PaddingRight=UDim.new(0,4); pd.Parent=p
   autoCanvas(p)
   return p
end

local function makeTab(parent, name, text, pos, active)
   local b = Instance.new("TextButton"); b.Name=name; b.ZIndex=9
   if pos then b.Position=pos end; b.Size=UDim2.new(1,0,0,30)
   b.BackgroundColor3=Color3.fromRGB(18,19,23); b.BackgroundTransparency=active and 0 or 0.28; b.BorderSizePixel=0
   b.Text=text; b.TextColor3=active and TEXT_WHITE or TEXT_DIM; b.TextSize=9; b.Font=Enum.Font.GothamBold; b.AutoButtonColor=false; b.Parent=parent
   Instance.new("UICorner",b).CornerRadius=UDim.new(0,7)
   local s = Instance.new("UIStroke"); s.Color=active and ACCENT or CARD_STROKE; s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; s.Transparency=0.25; s.Parent=b
   return b
end

------------------------------------------------------------------------
-- GUI ROOT
------------------------------------------------------------------------
local RaVe = Instance.new("ScreenGui"); RaVe.Name="RaVe"; RaVe.ResetOnSpawn=false
RaVe.ZIndexBehavior=Enum.ZIndexBehavior.Sibling; RaVe.Parent=LocalPlayer:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame"); Frame.Name="Frame"; Frame.ClipsDescendants=true
Frame.Position=UDim2.new(0,22,0.5,-270); Frame.Size=UDim2.new(0,420,0,528)
Frame.BackgroundColor3=BG_DARK; Frame.BorderSizePixel=0; Frame.Parent=RaVe

local UIScale = Instance.new("UIScale"); UIScale.Name="RaVeUIScale"; UIScale.Parent=Frame
Instance.new("UICorner",Frame).CornerRadius=UDim.new(0,16)
do
   local g = Instance.new("UIGradient"); g.Color=ColorSequence.new({
       ColorSequenceKeypoint.new(0,Color3.fromRGB(16,19,26)),
       ColorSequenceKeypoint.new(0.5,Color3.fromRGB(9,10,13)),
       ColorSequenceKeypoint.new(1,Color3.fromRGB(11,14,20))
   }); g.Rotation=90; g.Parent=Frame
   local st = Instance.new("UIStroke"); st.Color=ACCENT; st.Thickness=1.5; st.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; st.Transparency=1; st.Parent=Frame
   local sg = Instance.new("UIGradient"); sg.Color=ColorSequence.new(Color3.fromRGB(255,90,95),Color3.fromRGB(190,20,25)); sg.Rotation=256.4; sg.Parent=st
end

-- Corner decorations
for _,d in ipairs({
   {P=UDim2.new(0,-18,0,30), S=UDim2.new(0,360,0,360), C=ACCENT, T=0.922},
   {P=UDim2.new(0,-4,0,44),  S=UDim2.new(0,330,0,330), T=0.852},
   {P=UDim2.new(1,-2,1,-30), S=UDim2.new(0,360,0,360), C=ACCENT, T=0.922, A=Vector2.new(1,1), R=180},
   {P=UDim2.new(1,-2,1,-44), S=UDim2.new(0,330,0,330), T=0.852, A=Vector2.new(1,1), R=180},
}) do
   local i = Instance.new("ImageLabel"); i.Visible=false; i.ZIndex=0; i.Position=d.P; i.Size=d.S
   i.BackgroundTransparency=1; i.Image="rbxassetid://108037416708175"
   if d.C then i.ImageColor3=d.C end; i.ImageTransparency=d.T; i.ScaleType=Enum.ScaleType.Fit
   if d.A then i.AnchorPoint=d.A end; if d.R then i.Rotation=d.R end; i.Parent=Frame
end

------------------------------------------------------------------------
-- HEADER
------------------------------------------------------------------------
local Header = Instance.new("Frame"); Header.Size=UDim2.new(1,0,0,68); Header.BackgroundTransparency=1; Header.Parent=Frame
do
   local t = Instance.new("TextLabel"); t.ZIndex=3; t.Position=UDim2.new(0,18,0,13); t.Size=UDim2.new(0,320,0,34)
   t.BackgroundTransparency=1; t.Text='KuRu <font color="#FF3CAC">SLOTTED</font>'; t.TextColor3=TEXT_WHITE
   t.TextSize=26; t.Font=Enum.Font.GothamBlack; t.TextXAlignment=Enum.TextXAlignment.Left; t.RichText=true; t.Parent=Header
   local tg = Instance.new("UIGradient"); tg.Offset=Vector2.new(-0.92,0); tg.Parent=t
   local s = Instance.new("TextLabel"); s.ZIndex=3; s.Position=UDim2.new(0,20,0,45); s.Size=UDim2.new(0,240,0,13)
   s.BackgroundTransparency=1; s.Text="Made By TooZe  discord.gg/kuruu"; s.TextColor3=TEXT_DIM
   s.TextSize=11; s.Font=Enum.Font.GothamMedium; s.TextXAlignment=Enum.TextXAlignment.Left; s.Parent=Header
end

local MinBtn = Instance.new("TextButton"); MinBtn.ZIndex=3; MinBtn.Position=UDim2.new(1,-42,0,13); MinBtn.Size=UDim2.new(0,30,0,30)
MinBtn.BackgroundColor3=BTN_BG; MinBtn.BorderSizePixel=0; MinBtn.Text="-"; MinBtn.TextColor3=TEXT_PRIMARY
MinBtn.TextSize=13; MinBtn.Font=Enum.Font.GothamBold; MinBtn.AutoButtonColor=false; MinBtn.Parent=Header
Instance.new("UICorner",MinBtn); do local s=Instance.new("UIStroke"); s.Color=CARD_STROKE; s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; s.Transparency=0.4; s.Parent=MinBtn end

local LayoutBtn = Instance.new("TextButton"); LayoutBtn.Name="LayoutMode"; LayoutBtn.ZIndex=3
LayoutBtn.Position=UDim2.new(1,-78,0,13); LayoutBtn.Size=UDim2.new(0,30,0,30)
LayoutBtn.BackgroundColor3=BTN_BG; LayoutBtn.BorderSizePixel=0; LayoutBtn.Text="UI"; LayoutBtn.TextColor3=TEXT_PRIMARY
LayoutBtn.TextSize=13; LayoutBtn.Font=Enum.Font.GothamBold; LayoutBtn.AutoButtonColor=false; LayoutBtn.Parent=Header
Instance.new("UICorner",LayoutBtn); do local s=Instance.new("UIStroke"); s.Color=CARD_STROKE; s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; s.Transparency=0.4; s.Parent=LayoutBtn end

-- Divider
local Div = Instance.new("Frame"); Div.Position=UDim2.new(0,16,0,64); Div.Size=UDim2.new(1,-32,0,1)
Div.BackgroundColor3=TEXT_WHITE; Div.BorderSizePixel=0; Div.Parent=Frame
do local g=Instance.new("UIGradient"); g.Color=ColorSequence.new(ACCENT,ACCENT); g.Transparency=NumberSequence.new(0.2,0.85); g.Parent=Div end

------------------------------------------------------------------------
-- SCROLL MODE (hidden by default)
------------------------------------------------------------------------
local MainScroll = Instance.new("ScrollingFrame"); MainScroll.Name="MainScroll"; MainScroll.Visible=false
MainScroll.Position=UDim2.new(0,13,0,72); MainScroll.Size=UDim2.new(1,-26,1,-86)
MainScroll.BackgroundTransparency=1; MainScroll.BorderSizePixel=0; MainScroll.ScrollBarThickness=2
MainScroll.ScrollBarImageColor3=Color3.fromRGB(112,115,123); MainScroll.CanvasSize=UDim2.new(0,0,0,0); MainScroll.Parent=Frame
do
   local l=Instance.new("UIListLayout"); l.Padding=UDim.new(0,7); l.SortOrder=Enum.SortOrder.LayoutOrder; l.Parent=MainScroll
   local p=Instance.new("UIPadding"); p.PaddingBottom=UDim.new(0,12); p.PaddingRight=UDim.new(0,4); p.Parent=MainScroll
end

------------------------------------------------------------------------
-- PAGED CONTENT
------------------------------------------------------------------------
local PagedContent = Instance.new("Frame"); PagedContent.Name="PagedContent"
PagedContent.Position=UDim2.new(0,13,0,72); PagedContent.Size=UDim2.new(1,-128,1,-86)
PagedContent.BackgroundTransparency=1; PagedContent.Parent=Frame

------------------------------------------------------------------------
-- PAGE: MOVEMENT
------------------------------------------------------------------------
local PM = makePage(PagedContent, "KuRuPage_MOVEMENT", 1, true)

sectionHeader(PM, "SPEED CONFIGURATION")
inputRow(PM, "Normal Speed", "59.5")
inputRow(PM, "Carry Speed", "28.8")
toggleRow(PM, "Carry Mode", "Q", false)
toggleRow(PM, "Auto Carry Mode", nil, false)

sectionHeader(PM, "LAGGER CONFIGURATION")
inputRow(PM, "Lagger Normal Speed", "15")
inputRow(PM, "Lagger Carry Speed", "24.5")
toggleRow(PM, "Lagger Mode", "R", false)
modeSelector(PM, "LAGGER NORMAL", "LAGGER CARRY")

sectionHeader(PM, "QUICK ACTIONS")
sectionHeader(PM, "DROP BRAINROT")
actionRow(PM, "Drop", "X")

sectionHeader(PM, "TP DOWN")
actionRow(PM, "TP Down", "F")
toggleRow(PM, "Auto TP Down", nil, false)
inputRow(PM, "Auto TP Height", "20")

sectionHeader(PM, "JUMP")
toggleRow(PM, "Infinite Jump", nil, false)
toggleRow(PM, "Anti Ragdoll", nil, false)
toggleRow(PM, "Unwalk", nil, false)
toggleRow(PM, "Try Hard Animation", nil, false)
dropdownRow(PM, "Animation Pack", "Off  v")
arrowSelector(PM, "Sky", {"Off","Stars","Sunset","Night","Galaxy"})
arrowSelector(PM, "Korblox", {"Off","Right Leg","Left Leg","Both"})

sectionHeader(PM, "AUTO PATH")
toggleRow(PM, "Auto Left", "Z", false)
toggleRow(PM, "Auto Right", "C", false)

------------------------------------------------------------------------
-- PAGE: STEAL
------------------------------------------------------------------------
local PS = makePage(PagedContent, "KuRuPage_STEAL", 2, false)

sectionHeader(PS, "STEAL CONFIGURATION")
local _, _, stealSubs = toggleRow(PS, "Auto Steal", nil, true, true)
modeSelector(PS, "NORMAL", "SEMI")
local sr1 = inputRow(PS, "SEMI Range", "10", true)
local sr2 = inputRow(PS, "SEMI Prime", "80", true)
if stealSubs then table.insert(stealSubs, sr1); table.insert(stealSubs, sr2) end
inputRow(PS, "Radius", "62")
inputRow(PS, "Duration", "1.3")

------------------------------------------------------------------------
-- PAGE: COMBAT
------------------------------------------------------------------------
local PC = makePage(PagedContent, "KuRuPage_COMBAT", 3, false)

sectionHeader(PC, "BAT AIMBOT")
toggleRow(PC, "Bat Aimbot", "E", false, true)
modeSelector(PC, "DEFAULT", "BYPASS")
inputRow(PC, "Auto Bat Speed", "48")
toggleRow(PC, "Auto Swing", nil, false)
toggleRow(PC, "Mirror TP", nil, false)
toggleRow(PC, "TP Bat", "V", false, true)
modeSelector(PC, "SURE HIT", "HIGH PING")

sectionHeader(PC, "PROTECTION")
toggleRow(PC, "Safe Mode", nil, false)

sectionHeader(PC, "COUNTERS")
toggleRow(PC, "Bat Counter", nil, false)
toggleRow(PC, "Medusa Counter", nil, false)
toggleRow(PC, "Auto Counter", nil, false)

sectionHeader(PC, "BODY LOCK")
toggleRow(PC, "Body Lock", nil, false)
inputRow(PC, "Lock Radius", "20")

------------------------------------------------------------------------
-- PAGE: MISC
------------------------------------------------------------------------
local PMisc = makePage(PagedContent, "KuRuPage_MISC", 4, false)

sectionHeader(PMisc, "RESET")
actionRow(PMisc, "Insta Reset", "T")
toggleRow(PMisc, "Medusa Auto Reset", nil, false)
toggleRow(PMisc, "Auto Reset On Respawn", nil, false)

sectionHeader(PMisc, "APPEARANCE")
arrowSelector(PMisc, "Side Profile", {"PLAYER","OUTLINE","SILHOUETTE"})
toggleRow(PMisc, "Headless", nil, false)
toggleRow(PMisc, "Anti-Lag", nil, false)
toggleRow(PMisc, "Potato Graphics", nil, false)
toggleRow(PMisc, "FOV Change", nil, false)
inputRow(PMisc, "FOV Value", "120")
toggleRow(PMisc, "Stretch Res", nil, false)
toggleRow(PMisc, "ESP", nil, true)        -- starts ON
toggleRow(PMisc, "Show Tracer", nil, true) -- starts ON

------------------------------------------------------------------------
-- PAGE: SETTINGS
------------------------------------------------------------------------
local PSett = makePage(PagedContent, "KuRuPage_SETTINGS", 5, false)

-- These refs will be set by the background change logic
local SideTabBG, GeneralBG  -- forward declared

-- Tab Background selector with live preview
local tabBgRow = arrowSelector(PSett, "Tab Background", {"1","2","3"}, function(idx)
   if SideTabBG then SideTabBG.Image = TAB_BG_IMAGES[idx] or TAB_BG_IMAGES[1] end
end)
do
   local pv = Instance.new("ImageLabel"); pv.Position=UDim2.new(1,-141,0,8); pv.Size=UDim2.new(0,102,0,27)
   pv.BackgroundColor3=Color3.fromRGB(8,8,10); pv.BorderSizePixel=0; pv.Image=TAB_BG_IMAGES[1]; pv.ScaleType=Enum.ScaleType.Crop; pv.Parent=tabBgRow
   Instance.new("UICorner",pv)
   local s=Instance.new("UIStroke"); s.Color=ACCENT; s.Thickness=1.5; s.Parent=pv
end

-- Background selector with live preview
local bgRow = arrowSelector(PSett, "Background", {"1","2","3"}, function(idx)
   if GeneralBG then GeneralBG.Image = GENERAL_BG_IMAGES[idx] or GENERAL_BG_IMAGES[1] end
end)
do
   local pv = Instance.new("ImageLabel"); pv.Position=UDim2.new(1,-141,0,8); pv.Size=UDim2.new(0,102,0,27)
   pv.BackgroundColor3=Color3.fromRGB(8,8,10); pv.BorderSizePixel=0; pv.Image=GENERAL_BG_IMAGES[1]; pv.ScaleType=Enum.ScaleType.Crop; pv.Parent=bgRow
   Instance.new("UICorner",pv)
   local s=Instance.new("UIStroke"); s.Color=ACCENT; s.Thickness=1.5; s.Parent=pv
end

sectionHeader(PSett, "SETTINGS")
arrowSelector(PSett, "UI Layout", {"SCROLL","PAGED"}, function(idx)
   local paged = idx == 2
   MainScroll.Visible = not paged
   PagedContent.Visible = paged
   if SideTabBG then SideTabBG.Parent.Visible = paged end
   if GeneralBG then GeneralBG.Visible = paged end
   LayoutBtn.Text = paged and "UI" or "PG"
   if not paged then
       for _,ch in ipairs(MainScroll:GetChildren()) do
           if not ch:IsA("UIListLayout") and not ch:IsA("UIPadding") then ch:Destroy() end
       end
       local ord = 0
       for _,pg in ipairs({PM,PS,PC,PMisc,PSett}) do
           for _,ch in ipairs(pg:GetChildren()) do
               if not ch:IsA("UIListLayout") and not ch:IsA("UIPadding") then
                   local cl = ch:Clone(); cl.LayoutOrder=ord; cl.Visible=true; cl.Parent=MainScroll; ord=ord+1
               end
           end
       end
       autoCanvas(MainScroll)
   end
end)

local _, uiSizeBox = inputRow(PSett, "UI Size", "100")
local _, resetBtn = fullActionRow(PSett, "Reset All Settings")
toggleRow(PSett, "Auto Save", nil, true) -- starts ON

local _, uiToggleBtn = keybindRow(PSett, "UI Toggle Key", "LeftControl")
do
   local listening = false
   uiToggleBtn.MouseButton1Click:Connect(function()
       if listening then return end; listening=true; uiToggleBtn.Text="..."
       local cn; cn=UserInputService.InputBegan:Connect(function(inp,gpe)
           if gpe then return end
           if inp.UserInputType==Enum.UserInputType.Keyboard then
               UIToggleKey=inp.KeyCode; uiToggleBtn.Text=inp.KeyCode.Name; listening=false; cn:Disconnect()
           end
       end)
   end)
end

local _, closeBtn = fullActionRow(PSett, "Close Menu")

uiSizeBox.FocusLost:Connect(function()
   local v = tonumber(uiSizeBox.Text)
   if v then UIScale.Scale = math.clamp(v/100,0.5,2) end
end)

------------------------------------------------------------------------
-- GENERAL BACKGROUND
------------------------------------------------------------------------
GeneralBG = Instance.new("ImageLabel"); GeneralBG.Name="GeneralBackground"; GeneralBG.ZIndex=0
GeneralBG.Position=UDim2.new(0,13,0,72); GeneralBG.Size=UDim2.new(1,-128,1,-86)
GeneralBG.BackgroundColor3=Color3.fromRGB(5,5,7); GeneralBG.BackgroundTransparency=0.12; GeneralBG.BorderSizePixel=0
GeneralBG.Image=GENERAL_BG_IMAGES[1]; GeneralBG.ImageTransparency=0.2; GeneralBG.ScaleType=Enum.ScaleType.Crop; GeneralBG.Parent=Frame
Instance.new("UICorner",GeneralBG).CornerRadius=UDim.new(0,10)

------------------------------------------------------------------------
-- SIDE TAB BAR
------------------------------------------------------------------------
local LayoutTabs = Instance.new("Frame"); LayoutTabs.Name="LayoutTabs"; LayoutTabs.ZIndex=8
LayoutTabs.Position=UDim2.new(1,-100,0,72); LayoutTabs.Size=UDim2.new(0,88,1,-86)
LayoutTabs.BackgroundTransparency=1; LayoutTabs.Parent=Frame

SideTabBG = Instance.new("ImageLabel"); SideTabBG.Name="SideTabBackground"; SideTabBG.ZIndex=5
SideTabBG.Size=UDim2.new(1,0,1,0); SideTabBG.BackgroundColor3=Color3.fromRGB(5,5,7)
SideTabBG.BackgroundTransparency=0.15; SideTabBG.BorderSizePixel=0
SideTabBG.Image=TAB_BG_IMAGES[1]; SideTabBG.ImageTransparency=0.18
SideTabBG.ScaleType=Enum.ScaleType.Crop; SideTabBG.Parent=LayoutTabs
Instance.new("UICorner",SideTabBG).CornerRadius=UDim.new(0,10)

local TabHL = Instance.new("Frame"); TabHL.ZIndex=8; TabHL.Size=UDim2.new(0.999991,0,0,30)
TabHL.BackgroundColor3=ACCENT; TabHL.BackgroundTransparency=0.82; TabHL.BorderSizePixel=0; TabHL.Parent=LayoutTabs
Instance.new("UICorner",TabHL).CornerRadius=UDim.new(0,7)
do local s=Instance.new("UIStroke"); s.Color=ACCENT; s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; s.Parent=TabHL end

-- Identity panel
local SideId = Instance.new("Frame"); SideId.Name="SideIdentity"; SideId.ZIndex=10
SideId.Position=UDim2.new(0,0,0,150); SideId.Size=UDim2.new(1,0,0,132)
SideId.BackgroundTransparency=1; SideId.BorderSizePixel=0; SideId.Parent=LayoutTabs
do
   local av = Instance.new("ImageLabel"); av.ZIndex=11; av.AnchorPoint=Vector2.new(0.5,0)
   av.Position=UDim2.new(0.5,0,0,0); av.Size=UDim2.new(0,72,0,72)
   av.BackgroundColor3=Color3.fromRGB(8,9,11); av.BorderSizePixel=0
   av.Image="rbxthumb://type=AvatarHeadShot&id="..LocalPlayer.UserId.."&w=150&h=150"
   av.ScaleType=Enum.ScaleType.Crop; av.Parent=SideId
   Instance.new("UICorner",av).CornerRadius=UDim.new(0,33)
   local as2=Instance.new("UIStroke"); as2.Color=ACCENT; as2.Thickness=2; as2.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; as2.Parent=av

   local b1=Instance.new("TextLabel"); b1.ZIndex=11; b1.Position=UDim2.new(0,0,0,80); b1.Size=UDim2.new(1,0,0,15)
   b1.BackgroundTransparency=1; b1.Text="buyer of,"; b1.TextColor3=TEXT_DIM; b1.TextSize=10; b1.Font=Enum.Font.GothamMedium; b1.Parent=SideId
   local n1=Instance.new("TextLabel"); n1.ZIndex=11; n1.Position=UDim2.new(0,0,0,98); n1.Size=UDim2.new(1,0,0,18)
   n1.BackgroundTransparency=1; n1.Text=LocalPlayer.DisplayName; n1.TextColor3=TEXT_PRIMARY; n1.TextSize=12
   n1.Font=Enum.Font.GothamBold; n1.TextTruncate=Enum.TextTruncate.AtEnd; n1.Parent=SideId
   local s1=Instance.new("TextLabel"); s1.ZIndex=11; s1.Position=UDim2.new(0,0,0,116); s1.Size=UDim2.new(1,0,0,11)
   s1.BackgroundTransparency=1; s1.Text="CORES LOADED"; s1.TextColor3=ACCENT; s1.Font=Enum.Font.GothamBold; s1.Parent=SideId
end

-- Tab buttons
local TM = makeTab(LayoutTabs,"Tab_MOVEMENT","MOVEMENT",nil,true)
local TS = makeTab(LayoutTabs,"Tab_STEAL","STEAL",UDim2.new(0,0,0,40),false)
local TC = makeTab(LayoutTabs,"Tab_COMBAT","COMBAT",UDim2.new(0,0,0,80),false)
local TMi= makeTab(LayoutTabs,"Tab_MISC","MISC",UDim2.new(0,0,1,-78),false)
local TSe= makeTab(LayoutTabs,"Tab_SETTINGS","SETTINGS",UDim2.new(0,0,1,-38),false)

------------------------------------------------------------------------
-- NOTIFICATION FRAME
------------------------------------------------------------------------
local NotifFrame = Instance.new("Frame"); NotifFrame.Position=UDim2.new(1,-262,1,-396)
NotifFrame.Size=UDim2.new(0,250,0,380); NotifFrame.BackgroundTransparency=1; NotifFrame.Parent=RaVe
do local l=Instance.new("UIListLayout"); l.Padding=UDim.new(0,8); l.VerticalAlignment=Enum.VerticalAlignment.Bottom; l.SortOrder=Enum.SortOrder.LayoutOrder; l.Parent=NotifFrame end

------------------------------------------------------------------------
-- MINIMIZED PILL
------------------------------------------------------------------------
local MinPill = Instance.new("Frame"); MinPill.Visible=false; MinPill.Active=true; MinPill.ZIndex=40
MinPill.Position=UDim2.new(0,24,0.35,0); MinPill.Size=UDim2.new(0,140,0,40)
MinPill.BackgroundColor3=Color3.fromRGB(5,5,7); MinPill.BackgroundTransparency=0.02; MinPill.BorderSizePixel=0; MinPill.Parent=RaVe
Instance.new("UICorner",MinPill).CornerRadius=UDim.new(0,12)
do
   local s=Instance.new("UIStroke"); s.Color=Color3.fromRGB(55,57,63); s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; s.Transparency=0.55; s.Parent=MinPill
   local l=Instance.new("TextLabel"); l.Size=UDim2.new(1,0,1,0); l.BackgroundTransparency=1; l.Text="KuRu SLOT"; l.TextColor3=ACCENT; l.TextSize=14; l.Font=Enum.Font.GothamBlack; l.Parent=MinPill
   local b=Instance.new("TextButton"); b.ZIndex=41; b.Size=UDim2.new(1,0,1,0); b.BackgroundTransparency=1; b.Text=""; b.AutoButtonColor=false; b.Parent=MinPill
   b.MouseButton1Click:Connect(function() MinPill.Visible=false; Frame.Visible=true end)
end

------------------------------------------------------------------------
-- NOTIFY HELPER
------------------------------------------------------------------------
local function notify(text, dur)
   dur = dur or 3
   local n = Instance.new("Frame"); n.Size=UDim2.new(1,0,0,36); n.BackgroundColor3=Color3.fromRGB(5,5,7); n.BackgroundTransparency=0.05; n.BorderSizePixel=0; n.Parent=NotifFrame
   Instance.new("UICorner",n).CornerRadius=UDim.new(0,10)
   local ns=Instance.new("UIStroke"); ns.Color=ACCENT; ns.Transparency=0.5; ns.Parent=n
   local nl=Instance.new("TextLabel"); nl.Size=UDim2.new(1,-16,1,0); nl.Position=UDim2.new(0,8,0,0)
   nl.BackgroundTransparency=1; nl.Text=text; nl.TextColor3=TEXT_PRIMARY; nl.TextSize=11; nl.Font=Enum.Font.GothamBold
   nl.TextXAlignment=Enum.TextXAlignment.Left; nl.Parent=n
   task.delay(dur, function()
       TweenService:Create(n,TweenInfo.new(0.4),{BackgroundTransparency=1}):Play()
       TweenService:Create(nl,TweenInfo.new(0.4),{TextTransparency=1}):Play()
       TweenService:Create(ns,TweenInfo.new(0.4),{Transparency=1}):Play()
       task.delay(0.45,function() n:Destroy() end)
   end)
end

------------------------------------------------------------------------
-- TAB SWITCHING
------------------------------------------------------------------------
local Pages = {MOVEMENT=PM, STEAL=PS, COMBAT=PC, MISC=PMisc, SETTINGS=PSett}
local Tabs  = {MOVEMENT=TM, STEAL=TS, COMBAT=TC, MISC=TMi, SETTINGS=TSe}
local curTab = "MOVEMENT"

local function switchTab(name)
   if curTab == name then return end; curTab = name
   for k,p in pairs(Pages) do p.Visible = (k==name) end
   for k,b in pairs(Tabs) do
       local act = (k==name)
       b.TextColor3 = act and TEXT_WHITE or TEXT_DIM
       b.BackgroundTransparency = act and 0 or 0.28
       local st = b:FindFirstChildOfClass("UIStroke")
       if st then st.Color = act and ACCENT or CARD_STROKE end
   end
   local ab = Tabs[name]
   if ab then TweenService:Create(TabHL, TWEEN_MED, {Position=UDim2.new(ab.Position.X.Scale,ab.Position.X.Offset,ab.Position.Y.Scale,ab.Position.Y.Offset)}):Play() end
end

TM.MouseButton1Click:Connect(function() switchTab("MOVEMENT") end)
TS.MouseButton1Click:Connect(function() switchTab("STEAL") end)
TC.MouseButton1Click:Connect(function() switchTab("COMBAT") end)
TMi.MouseButton1Click:Connect(function() switchTab("MISC") end)
TSe.MouseButton1Click:Connect(function() switchTab("SETTINGS") end)

------------------------------------------------------------------------
-- MINIMIZE / CLOSE
------------------------------------------------------------------------
local function minimize() Frame.Visible=false; MinPill.Visible=true end
MinBtn.MouseButton1Click:Connect(minimize)
closeBtn.MouseButton1Click:Connect(minimize)

------------------------------------------------------------------------
-- LAYOUT TOGGLE BUTTON
------------------------------------------------------------------------
LayoutBtn.MouseButton1Click:Connect(function()
   local isPaged = PagedContent.Visible
   if isPaged then
       -- Switch to scroll
       PagedContent.Visible=false; LayoutTabs.Visible=false; GeneralBG.Visible=false; MainScroll.Visible=true
       LayoutBtn.Text="PG"
       for _,ch in ipairs(MainScroll:GetChildren()) do
           if not ch:IsA("UIListLayout") and not ch:IsA("UIPadding") then ch:Destroy() end
       end
       local ord=0
       for _,pg in ipairs({PM,PS,PC,PMisc,PSett}) do
           for _,ch in ipairs(pg:GetChildren()) do
               if not ch:IsA("UIListLayout") and not ch:IsA("UIPadding") then
                   local cl=ch:Clone(); cl.LayoutOrder=ord; cl.Visible=true; cl.Parent=MainScroll; ord=ord+1
               end
           end
       end
       autoCanvas(MainScroll)
   else
       -- Switch to paged
       MainScroll.Visible=false; PagedContent.Visible=true; LayoutTabs.Visible=true; GeneralBG.Visible=true
       LayoutBtn.Text="UI"
   end
end)

------------------------------------------------------------------------
-- DRAGGING
------------------------------------------------------------------------
do
   local function makeDrag(obj, target)
       local drag,dInp,dStart,sPos
       obj.InputBegan:Connect(function(i)
           if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
               drag=true; dStart=i.Position; sPos=target.Position
               i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then drag=false end end)
           end
       end)
       obj.InputChanged:Connect(function(i)
           if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then dInp=i end
       end)
       UserInputService.InputChanged:Connect(function(i)
           if i==dInp and drag then
               local d=i.Position-dStart
               target.Position=UDim2.new(sPos.X.Scale,sPos.X.Offset+d.X,sPos.Y.Scale,sPos.Y.Offset+d.Y)
           end
       end)
   end
   makeDrag(Header, Frame)
   makeDrag(MinPill, MinPill)
end

------------------------------------------------------------------------
-- UI TOGGLE KEY
------------------------------------------------------------------------
UserInputService.InputBegan:Connect(function(inp,gpe)
   if gpe then return end
   if inp.UserInputType==Enum.UserInputType.Keyboard and inp.KeyCode==UIToggleKey then
       if Frame.Visible then minimize() else MinPill.Visible=false; Frame.Visible=true end
   end
end)

------------------------------------------------------------------------
-- RESET ALL
------------------------------------------------------------------------
resetBtn.MouseButton1Click:Connect(function()
   for _,d in pairs(Toggles) do if d.set then d.set(false) end end
   uiSizeBox.Text="100"; UIScale.Scale=1
   notify("All settings reset!")
end)

------------------------------------------------------------------------
-- OPEN ANIMATION
------------------------------------------------------------------------
do
   Frame.BackgroundTransparency=1; Frame.Size=UDim2.new(0,420,0,0)
   task.defer(function()
       TweenService:Create(Frame, TweenInfo.new(0.4,Enum.EasingStyle.Back,Enum.EasingDirection.Out), {
           Size=UDim2.new(0,420,0,528), BackgroundTransparency=0
       }):Play()
       task.delay(0.3, function() notify("KuRu SLOTTED loaded!", 4) end)
   end)
end
