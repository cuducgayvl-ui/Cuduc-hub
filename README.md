local _ENV = (getgenv or getrenv or getfenv)()
local BETA_VERSION = BETA_VERSION or _ENV.BETA_VERSION

local HUB_NAME = "Cuduc Hub"
local DISCORD_LINK = "https://discord.gg"
local HUB_LOGO = "rbxassetid://121632477410656"

-- Tự động copy Discord
if setclipboard then setclipboard(DISCORD_LINK) end

local Scripts = {
	{
		GameId = 994732206,
		UrlPath = if BETA_VERSION then "BLOX-FRUITS-BETA.lua" else "BloxFruits.luau"
	},
	{
		PlacesIds = {10260193230},
		UrlPath = "MemeSea.luau"
	}
}

local fetcher, urls = {}, {}

do
	local last_exec = _ENV.cuduc_execute_debounce
	if last_exec and (tick() - last_exec) <= 5 then return nil end
	_ENV.cuduc_execute_debounce = tick()
end

urls.Owner = "https://githubusercontent.com";
urls.Repository = urls.Owner .. "Scripts/refs/heads/main/";

-- Tạo thông báo khởi chạy có chứa ảnh của bạn
local function NotifyLoad()
    local sg = Instance.new("ScreenGui", game.CoreGui)
    local frame = Instance.new("Frame", sg)
    frame.Size = UDim2.new(0, 250, 0, 100)
    frame.Position = UDim2.new(0.5, -125, 0.1, 0)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    frame.BorderSizePixel = 0
    
    local img = Instance.new("ImageLabel", frame)
    img.Size = UDim2.new(0, 80, 0, 80)
    img.Position = UDim2.new(0, 10, 0.5, -40)
    img.Image = HUB_LOGO
    img.BackgroundTransparency = 1
    
    local txt = Instance.new("TextLabel", frame)
    txt.Size = UDim2.new(0, 150, 0, 100)
    txt.Position = UDim2.new(0, 90, 0, 0)
    txt.Text = HUB_NAME .. "\nLoading...\nDiscord Copied"
    txt.TextColor3 = Color3.fromRGB(255, 255, 255)
    txt.BackgroundTransparency = 1
    txt.TextWrapped = true
    
    task.delay(5, function() sg:Destroy() end)
end

do
	if _ENV.cuduc_error_message then _ENV.cuduc_error_message:Destroy() end
	local identifyexecutor = identifyexecutor or (function() return "Unknown" end)
	
	local function CreateMessageError(Text)
		_ENV.loadedFarm = nil
		_ENV.OnFarm = false
		local Message = Instance.new("Message", workspace)
		Message.Text = "[" .. HUB_NAME .. "] Error: " .. Text
		_ENV.cuduc_error_message = Message
		error(Text, 2)
	end
	
	function fetcher.get(Url)
		local success, response = pcall(function()
			local target = Url:gsub("{Repository}", urls.Repository)
			return game:HttpGet(target)
		end)
		if success then return response else CreateMessageError("Fail to load URL") end
	end
	
	function fetcher.load(Url, concat)
		local raw = fetcher.get(Url) .. (concat or "")
		local run, err = loadstring(raw)
		if type(run) ~= "function" then CreateMessageError("Syntax Error") else return run end
	end
end

local function IsPlace(Script)
	return (Script.PlacesIds and table.find(Script.PlacesIds, game.PlaceId)) or (Script.GameId and Script.GameId == game.GameId)
end

NotifyLoad()

for _, Script in Scripts do
	if IsPlace(Script) then
		return fetcher.load("{Repository}Games/" .. Script.UrlPath)(fetcher, ...)
	end
end
