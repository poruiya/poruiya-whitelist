### Poruiya Whitelist
Whitelist System For Fivem/VMP Platforms

## Features
1. Send A Discord Log For Every Connect
2. Showing PLayer Name
3. Server ID
4. Discord ID
5. Discord Mentioned
6. License
7. Steam Hex
8. Xbox Live
9. IP
10. Country
And Showing The Client Status (is Whitelist or not)

More Feature Coming Soon...

# Requirements
```
Discord Webhook
Steam API Key
```

## Note
I Hope This Helps You On Your Server Journey To Becoming a Good Server!

Now, The Whitelist Code :

Server.lua :
```
local whitelist = {}
local webhookURL = "Your Discord Webhook"
local steamApiKey = "Your Steam API Key"

-- Function to load the whitelist from the JSON file
local function loadWhitelist()
    local resourcePath = GetResourcePath(GetCurrentResourceName())
    local file = io.open(resourcePath .. "/whitelist.json", "r")
    if file then
        local content = file:read("*a")
        whitelist = json.decode(content)
        file:close()
        print("Whitelist Ha Load Shod")
    else
        print("WhiteList Ha Load Nashod")
    end
end

-- Function to check if a player is whitelisted
local function isWhitelisted(playerId)
    local identifiers = GetPlayerIdentifiers(playerId)
    for _, id in ipairs(identifiers) do
        for _, whitelistedId in ipairs(whitelist) do
            if id == whitelistedId then
                return true
            end
        end
    end
    return false
end

-- Function to get detailed player info
local function getPlayerInfo(playerId, callback)
    local info = {
        identifiers = GetPlayerIdentifiers(playerId),
        name = GetPlayerName(playerId),
        endpoint = GetPlayerEndpoint(playerId),
        discord = "N/A",
        steam = "N/A",
        license = "N/A",
        xbox = "N/A",
        live = "N/A",
        ip = "N/A",
        country = "N/A",
        discord_tag = "N/A",
        steam_avatar = "N/A"
    }

    for _, id in ipairs(info.identifiers) do
        if string.find(id, "discord:") then
            info.discord = id:sub(9)
        elseif string.find(id, "steam:") then
            info.steam = id:sub(7)
        elseif string.find(id, "license:") then
            info.license = id
        elseif string.find(id, "xbl:") then
            info.xbox = id
        elseif string.find(id, "live:") then
            info.live = id
        elseif string.find(id, "ip:") then
            info.ip = id:sub(4)
        end
    end

    -- Get country from IP using a third-party service
    PerformHttpRequest("http://ip-api.com/json/" .. info.ip, function(statusCode, response, headers)
        if statusCode == 200 then
            local data = json.decode(response)
            info.country = data.country or "N/A"
        else
            print("Failed to get country from IP, status code: " .. statusCode)
        end

        -- Get Steam profile info
        if info.steam ~= "N/A" then
            local steamProfileUrl = string.format("http://api.steampowered.com/ISteamUser/GetPlayerSummaries/v0002/?key=%s&steamids=%s", steamApiKey, info.steam)
            PerformHttpRequest(steamProfileUrl, function(statusCode, response, headers)
                if statusCode == 200 then
                    local data = json.decode(response)
                    if data.response and data.response.players and #data.response.players > 0 then
                        info.steam_avatar = data.response.players[1].avatarfull or "N/A"
                    else
                        print("Failed to get Steam avatar, no players found in response")
                    end
                else
                    print("Failed to get Steam avatar, status code: " .. statusCode)
                end
                callback(info)
            end, "GET", "", {["Content-Type"] = "application/json"})
        else
            callback(info)
        end
    end, "GET", "", {["Content-Type"] = "application/json"})
end

-- Function to send a message to the Discord webhook
local function sendToDiscord(info, status)
    -- Ensure all fields are correctly populated
    local message = {
        username = "Whitelist Logger",
        embeds = {
            {
                title = "Player Connection Attempt",
                description = string.format([[
**Player Name:** %s
**Server ID:** %d
**Discord ID:** %s
**Discord Tag:** %s
**License:** %s
**Steam Hex:** %s
**Xbox Live:** %s
**IP:** %s
**Country:** %s
**Status:** %s
                ]], info.name, info.serverId, info.discord, info.discord_tag, info.license or "N/A", info.steam or "N/A", info.xbox or "N/A", info.ip or "N/A", info.country, status),
                color = status == "Whitelisted" and 3066993 or 15158332, -- Green if whitelisted, red if not
                thumbnail = {
                    url = info.steam_avatar ~= "N/A" and info.steam_avatar or "https://cdn.discordapp.com/embed/avatars/0.png" -- Default image if Steam avatar is not available
                }
            }
        }
    }
--Debug message to ensure it is correctly formatted
    print("Sending webhook message: " .. json.encode(message))
    PerformHttpRequest(webhookURL, function(err, text, headers)
        if err ~= 200 then
            print("Failed to send webhook, status code: " .. err)
            if text then
                print("Error text: " .. text)
            end
        else
            print("Webhook sent successfully")
        end
    end, "POST", json.encode(message), { ["Content-Type"] = "application/json" })
end
-- Event handler for player connecting
AddEventHandler("playerConnecting", function(name, setKickReason, deferrals)
    deferrals.defer()
    local playerId = source
    loadWhitelist()
    local isAllowed = isWhitelisted(playerId)

    getPlayerInfo(playerId, function(info)
        info.serverId = playerId
        info.discord_tag = "<@" .. info.discord .. ">"

        if not isAllowed then
            deferrals.done("In Server Whitelist Hast Va Player Haye Normal Nemitonan Connect Bedan!! (Shoma Mitoni In Massage Ro Avaz Knid)")
            sendToDiscord(info, "Not Whitelisted")
        else
            deferrals.done()
            sendToDiscord(info, "Whitelisted")
        end
    end)
end)
```
Whitelist.json : 
```
[
    "SteamHex",
    "SteamHex2",
    "SteamHex3"
]
```
Fxmanifest.lua
```
fx_version 'cerulean'
game 'gta5'

author 'poruiya'
description 'Whitelist Script For VMP/Fivem Servers'
version '1.0.0'

server_scripts {
    'server.lua'
}
```
