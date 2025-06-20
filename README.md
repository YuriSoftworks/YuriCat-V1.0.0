if getgenv().YuriCat then
    return error("YuriCat is already loaded")
end

if not game:IsLoaded() then
    game.Loaded:Wait()
end

getgenv().YuriCat = {
    Dev = false,
    Connections = {},
    Pages = {},
    Tabs = {Tabs = {}},
    Corners = {},
    Load = os.clock(),
    Notifications = {Objects = {}, Active = {}},
    ArrayList = {Objects = {}, Loaded = false},
    ControlsVisible = true,
    Mobile = false,
    CurrentOpenTab = nil,
    GameSave = game.PlaceId,
    CheckOtherConfig = true,
    Assets = {},
    Teleporting = false,
    InitSave = nil,
    Config = {
        UI = {
            Position = {X = 0.5, Y = 0.5},
            Size = {X = 0.37294304370880129, Y = 0.683131217956543},
            FullScreen = false,
            ToggleKeyCode = "LeftAlt",
            Scale = 1,
            Notifications = true,
            Anim = true,
            ArrayList = false,
            TabColor = {value1 = 25, value2 = 25, value3 = 25},
            TabTransparency = 0.03,
            KeybindTransparency = 0.7,
            KeybindColor = {value1 = 0, value2 = 0, value3 = 0},
        },
        Game = {
            Modules = {},
            Keybinds = {},
            Sliders = {},
            TextBoxes = {},
            MiniToggles = {},
            Dropdowns = {},
            ModuleKeybinds = {},
            Other = {}
        },
    }
} 
if getgenv().YuriCatInit then
    if getgenv().YuriCatInit.GameSave then
        getgenv().YuriCat.GameSave = getgenv().YuriCatInit.GameSave
    end
    if getgenv().YuriCatInit.CheckOtherConfig then
        getgenv().YuriCat.CheckOtherConfig = getgenv().YuriCatInit.CheckOtherConfig
    end
    if getgenv().YuriCatInit.Dev then
        getgenv().YuriCat.Dev = true
    end
    getgenv().YuriCat.InitSave = getgenv().YuriCatInit
    getgenv().YuriCatInit = nil
end



local Assets = nil
if getgenv().YuriCat.Dev and isfile("YuriCat/Library/Init.lua") then
    loadstring(readfile("YuriCat/Library/Init.lua"))()
else
    loadstring(game:HttpGet("https://raw.githubusercontent.com/null-wtf/Night/refs/heads/main/Night/Library/Init.lua"))()
end
Assets = getgenv().YuriCat.Assets

if not Assets or typeof(Assets) ~= "table" or (Assets and not Assets.Functions) then
    getgenv().YuriCat = nil
    return warn("Failed to load Functions, YuriCat uninjected")

end

local uis = Assets.Functions.cloneref(game:GetService("UserInputService")) :: UserInputService
local ws = Assets.Functions.cloneref(game:GetService("Workspace")) :: Workspace
local plrs = Assets.Functions.cloneref(game:GetService("Players")) :: Players

local currentCamera = ws:FindFirstChildWhichIsA("Camera") :: Camera

if not uis.KeyboardEnabled and uis.TouchEnabled then
    getgenv().YuriCat.Mobile = true
    getgenv().YuriCat.Config.UI.Size = {X = 0.7, Y = 0.9}
end


if not isfolder("YuriCat") then
    makefolder("YuriCat")
end
if not isfolder("YuriCat/Config") then
    makefolder("YuriCat/Config")
end 
if not isfolder("YuriCat/Assets") then
    makefolder("YuriCat/Assets")
end
if not isfolder("YuriCat/Assets/Fonts") then
    makefolder("YuriCat/Assets/Fonts")
end

local gameinfo = Assets.Functions.GetGameInfo()
if typeof(gameinfo) == "table" then
    getgenv().YuriCat.GameRootId = gameinfo.rootPlaceId 
    if getgenv().YuriCat.GameSave == "root" then
        getgenv().YuriCat.GameSave = tostring(getgenv().YuriCat.GameRootId)
    end
end


local UI = Assets.Config.Load("UI", "UI")
local gamesave = Assets.Config.Load(tostring(getgenv().YuriCat.GameSave), "Game")
if UI == "no file" then
    Assets.Config.Save("UI", getgenv().YuriCat.Config.UI)
end

if gamesave == "no file" and getgenv().YuriCat.CheckOtherConfig then
    if getgenv().YuriCat.GameRootId == getgenv().YuriCat.GameSave then
        gamesave = Assets.Config.Load(tostring(game.PlaceId), "Game")
    else
        gamesave = Assets.Config.Load(tostring(getgenv().YuriCat.GameRootId), "Game")
    end
end

if gamesave == "no file" then
    Assets.Config.Save(tostring(getgenv().YuriCat.GameSave), getgenv().YuriCat.Config.Game)
end

if getgenv().YuriCat.Mobile then
    if currentCamera then
        if 0.4 >= (currentCamera.ViewportSize.X / 1000) - 0.1 then
            getgenv().YuriCat.Config.UI.Scale = 0.4
        else
            getgenv().YuriCat.Config.UI.Scale = (currentCamera.ViewportSize.X / 1000) - 0.1
        end
    end
end

if queue_on_teleport then
    table.insert(getgenv().YuriCat.Connections, plrs.LocalPlayer.OnTeleport:Connect(function(state)
        if not getgenvYuriCat.Teleporting then
            getgenv().YuriCat.Teleporting = true
            
            local str = ""
            if getgenv().Night.InitSave then
                str = "getgenv().YuriCatInit = {"
                for i, v in getgenv().YuriCat.InitSave do
                    if i ~= #getgenv().YuriCat.InitSave then
                        if typeof(v) == "string" then
                            str = str..tostring(i)..' = "'..tostring(v)..'" , '
                        else
                            str = str..tostring(i).." = "..tostring(v).." , "
                        end
                    end
                end
                str = string.sub(str, 0, #str-2).."}\n"
            end

            str = str..[[
                if not game:IsLoaded() then
                    game.Loaded:Wait()
                end

                if getgenv().YuriCatInit and getgenv().YuriCatInit.Dev and isfile("YuriCat/Loader.lua") then
                    loadstring(readfile("YuriCat/Loader.lua"))()
                else
                    loadstring(game:HttpGet("https://raw.githubusercontent.com/null-wtf/Night/refs/heads/main/Night/Loader.luau"))()
                end
            ]]
            queue_on_teleport(str)
        end
    end))
end

Assets.Main.Load("Universal")
Assets.Main.Load(getgenv().YuriCat.GameSave)

Assets.Main.ToggleVisibility(true)

getgenv()YuriCat.Main = Assets.Main
getgenv().YuriCat.LoadTime = os.clock() - getgenv().YuriCat.Load
Assets.Notifications.Send({
    Description = "Loaded in " .. string.format("%.1f", getgenv().YuriCat.LoadTime) .. " seconds",
    Duration = 5
})
--[[if getgenv().YuriCat.Mobile or getgenv().YuriCat.Config.UI.ToggleKeyCode and getgenv().YuriCat.Config.UI.ToggleKeyCode ~= "" and getgenv().YuriCat.Config.UI.ToggleKeyCode ~= "Unknown" then
    task.wait(0.5)
    Assets.Notifications.Send({
        Description = "Current Keybind is: " .. getgenv().YuriCat.Config.UI.ToggleKeyCode,
        Duration = 5
    })
end]]
task.wait(0.15)

if not isfile("YuriCat/Version.txt") then
    writefile("YuriCat/Version.txt", "Current version: 1.0.0")
    Assets.Notifications.Send({
        Description = "YuriCat has been updated to V1.0.0",
        Duration = 15
    })
end

local text = readfile("YuriCat/Version.txt")
if text ~= "Current version: 1.0.0" then
    Assets.Notifications.Send({
        Description = "YuriCat has been updated to V1.0.0",
        Duration = 15
    })
    writefile("YuriCat/Version.txt", "Current version: 1.0.0")
end

YuriCat.Loaded = true
return Assets.Mainif getgenv().YuriCat then
    return error("YuriCat is already loaded")
end

if not game:IsLoaded() then
    game.Loaded:Wait()
end

getgenv().YuriCat = {
    Dev = false,
    Connections = {},
    Pages = {},
    Tabs = {Tabs = {}},
    Corners = {},
    Load = os.clock(),
    Notifications = {Objects = {}, Active = {}},
    ArrayList = {Objects = {}, Loaded = false},
    ControlsVisible = true,
    Mobile = false,
    CurrentOpenTab = nil,
    GameSave = game.PlaceId,
    CheckOtherConfig = true,
    Assets = {},
    Teleporting = false,
    InitSave = nil,
    Config = {
        UI = {
            Position = {X = 0.5, Y = 0.5},
            Size = {X = 0.37294304370880129, Y = 0.683131217956543},
            FullScreen = false,
            ToggleKeyCode = "LeftAlt",
            Scale = 1,
            Notifications = true,
            Anim = true,
            ArrayList = false,
            TabColor = {value1 = 25, value2 = 25, value3 = 25},
            TabTransparency = 0.03,
            KeybindTransparency = 0.7,
            KeybindColor = {value1 = 0, value2 = 0, value3 = 0},
        },
        Game = {
            Modules = {},
            Keybinds = {},
            Sliders = {},
            TextBoxes = {},
            MiniToggles = {},
            Dropdowns = {},
            ModuleKeybinds = {},
            Other = {}
        },
    }
} 
if getgenv().YuriCatInit then
    if getgenv().YuriCatInit.GameSave then
        getgenv().YuriCat.GameSave = getgenv().YuriCatInit.GameSave
    end
    if getgenv().YuriCatInit.CheckOtherConfig then
        getgenv().YuriCat.CheckOtherConfig = getgenv().YuriCatInit.CheckOtherConfig
    end
    if getgenv().YuriCatInit.Dev then
        getgenv().YuriCat.Dev = true
    end
    getgenv().YuriCat.InitSave = getgenv().YuriCatInit
    getgenv().YuriCatInit = nil
end



local Assets = nil
if getgenv().YuriCat.Dev and isfile("YuriCat/Library/Init.lua") then
    loadstring(readfile("YuriCat/Library/Init.lua"))()
else
    loadstring(game:HttpGet("https://raw.githubusercontent.com/null-wtf/Night/refs/heads/main/Night/Library/Init.lua"))()
end
Assets = getgenv().YuriCat.Assets

if not Assets or typeof(Assets) ~= "table" or (Assets and not Assets.Functions) then
    getgenv().YuriCat = nil
    return warn("Failed to load Functions, YuriCat uninjected")

end

local uis = Assets.Functions.cloneref(game:GetService("UserInputService")) :: UserInputService
local ws = Assets.Functions.cloneref(game:GetService("Workspace")) :: Workspace
local plrs = Assets.Functions.cloneref(game:GetService("Players")) :: Players

local currentCamera = ws:FindFirstChildWhichIsA("Camera") :: Camera

if not uis.KeyboardEnabled and uis.TouchEnabled then
    getgenv().YuriCat.Mobile = true
    getgenv().YuriCat.Config.UI.Size = {X = 0.7, Y = 0.9}
end


if not isfolder("YuriCat") then
    makefolder("YuriCat")
end
if not isfolder("YuriCat/Config") then
    makefolder("YuriCat/Config")
end 
if not isfolder("YuriCat/Assets") then
    makefolder("YuriCat/Assets")
end
if not isfolder("YuriCat/Assets/Fonts") then
    makefolder("YuriCat/Assets/Fonts")
end

local gameinfo = Assets.Functions.GetGameInfo()
if typeof(gameinfo) == "table" then
    getgenv().YuriCat.GameRootId = gameinfo.rootPlaceId 
    if getgenv().YuriCat.GameSave == "root" then
        getgenv().YuriCat.GameSave = tostring(getgenv().YuriCat.GameRootId)
    end
end


local UI = Assets.Config.Load("UI", "UI")
local gamesave = Assets.Config.Load(tostring(getgenv().YuriCat.GameSave), "Game")
if UI == "no file" then
    Assets.Config.Save("UI", getgenv().YuriCat.Config.UI)
end

if gamesave == "no file" and getgenv().YuriCat.CheckOtherConfig then
    if getgenv().YuriCat.GameRootId == getgenv().YuriCat.GameSave then
        gamesave = Assets.Config.Load(tostring(game.PlaceId), "Game")
    else
        gamesave = Assets.Config.Load(tostring(getgenv().YuriCat.GameRootId), "Game")
    end
end

if gamesave == "no file" then
    Assets.Config.Save(tostring(getgenv().YuriCat.GameSave), getgenv().YuriCat.Config.Game)
end

if getgenv().YuriCat.Mobile then
    if currentCamera then
        if 0.4 >= (currentCamera.ViewportSize.X / 1000) - 0.1 then
            getgenv().YuriCat.Config.UI.Scale = 0.4
        else
            getgenv().YuriCat.Config.UI.Scale = (currentCamera.ViewportSize.X / 1000) - 0.1
        end
    end
end

if queue_on_teleport then
    table.insert(getgenv().YuriCat.Connections, plrs.LocalPlayer.OnTeleport:Connect(function(state)
        if not getgenvYuriCat.Teleporting then
            getgenv().YuriCat.Teleporting = true
            
            local str = ""
            if getgenv().Night.InitSave then
                str = "getgenv().YuriCatInit = {"
                for i, v in getgenv().YuriCat.InitSave do
                    if i ~= #getgenv().YuriCat.InitSave then
                        if typeof(v) == "string" then
                            str = str..tostring(i)..' = "'..tostring(v)..'" , '
                        else
                            str = str..tostring(i).." = "..tostring(v).." , "
                        end
                    end
                end
                str = string.sub(str, 0, #str-2).."}\n"
            end

            str = str..[[
                if not game:IsLoaded() then
                    game.Loaded:Wait()
                end

                if getgenv().YuriCatInit and getgenv().YuriCatInit.Dev and isfile("YuriCat/Loader.lua") then
                    loadstring(readfile("YuriCat/Loader.lua"))()
                else
                    loadstring(game:HttpGet("https://raw.githubusercontent.com/null-wtf/Night/refs/heads/main/Night/Loader.luau"))()
                end
            ]]
            queue_on_teleport(str)
        end
    end))
end

Assets.Main.Load("Universal")
Assets.Main.Load(getgenv().YuriCat.GameSave)

Assets.Main.ToggleVisibility(true)

getgenv()YuriCat.Main = Assets.Main
getgenv().YuriCat.LoadTime = os.clock() - getgenv().YuriCat.Load
Assets.Notifications.Send({
    Description = "Loaded in " .. string.format("%.1f", getgenv().YuriCat.LoadTime) .. " seconds",
    Duration = 5
})
--[[if getgenv().YuriCat.Mobile or getgenv().YuriCat.Config.UI.ToggleKeyCode and getgenv().YuriCat.Config.UI.ToggleKeyCode ~= "" and getgenv().YuriCat.Config.UI.ToggleKeyCode ~= "Unknown" then
    task.wait(0.5)
    Assets.Notifications.Send({
        Description = "Current Keybind is: " .. getgenv().YuriCat.Config.UI.ToggleKeyCode,
        Duration = 5
    })
end]]
task.wait(0.15)

if not isfile("YuriCat/Version.txt") then
    writefile("YuriCat/Version.txt", "Current version: 1.0.0")
    Assets.Notifications.Send({
        Description = "YuriCat has been updated to V1.0.0",
        Duration = 15
    })
end

local text = readfile("YuriCat/Version.txt")
if text ~= "Current version: 1.0.0" then
    Assets.Notifications.Send({
        Description = "YuriCat has been updated to V1.0.0",
        Duration = 15
    })
    writefile("YuriCat/Version.txt", "Current version: 1.0.0")
end

YuriCat.Loaded = true
return Assets.Main
