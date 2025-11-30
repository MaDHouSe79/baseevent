# File location: [cfx-default]/system/baseevent/vehiclechecker.lua

This is de default baseevent vehiclechecker.lua file for fivem that give you F8 spawn warning hell, here is a code that wil fix that.

Why this works is it checks if the netid is valid, if not you get nil and not a F8 warning hell..

# ***Zero F8 Warnings** (No 65534 BS)*

```lua
local isInVehicle = false
local isEnteringVehicle = false
local currentVehicle = 0
local currentSeat = 0

local function SafeNetId(entity)
    if not entity or not DoesEntityExist(entity) then return nil end
    local netId = VehToNet(entity)
    return (netId > 0 and netId < 65535) and netId or nil
end

Citizen.CreateThread(function()
	while true do
		Citizen.Wait(0)
		local ped = PlayerPedId()
		if not isInVehicle and not IsPlayerDead(PlayerId()) then
			if DoesEntityExist(GetVehiclePedIsTryingToEnter(ped)) and not isEnteringVehicle then
				-- trying to enter a vehicle!
				local vehicle = GetVehiclePedIsTryingToEnter(ped)
				local seat = GetSeatPedIsTryingToEnter(ped)
				--local netId = VehToNet(vehicle)
				local netId = -1
                CreateThread(function()
                    repeat
                        Wait(100)
                        netId = SafeNetId(vehicle)
                    until netId and netId > 0 and netId < 65535 and NetworkDoesNetworkIdExist(netId)
                    if netId ~= -1 then 
						isEnteringVehicle = true
						TriggerServerEvent('baseevents:enteringVehicle', vehicle, seat, GetDisplayNameFromVehicleModel(GetEntityModel(vehicle)), netId)					
					end
                end)
			elseif not DoesEntityExist(GetVehiclePedIsTryingToEnter(ped)) and not IsPedInAnyVehicle(ped, true) and isEnteringVehicle then
				-- vehicle entering aborted
				TriggerServerEvent('baseevents:enteringAborted')
				isEnteringVehicle = false
			elseif IsPedInAnyVehicle(ped, false) then
				-- suddenly appeared in a vehicle, possible teleport
				currentVehicle = GetVehiclePedIsUsing(ped)
				local model = GetEntityModel(currentVehicle)
				local name = GetDisplayNameFromVehicleModel()
				local netId = -1
                CreateThread(function()
                    repeat
                        Wait(100)
                        netId = SafeNetId(currentVehicle)
                    until netId and netId > 0 and netId < 65535 and NetworkDoesNetworkIdExist(netId)
                    if netId ~= -1 then 
						isEnteringVehicle = false
						isInVehicle = true
						currentSeat = GetPedVehicleSeat(ped)
						TriggerServerEvent('baseevents:enteredVehicle', currentVehicle, currentSeat, GetDisplayNameFromVehicleModel(GetEntityModel(currentVehicle)), netId)				
					end
                end)
			end
		elseif isInVehicle then
			if not IsPedInAnyVehicle(ped, false) or IsPlayerDead(PlayerId()) then
				-- bye, vehicle
				local model = GetEntityModel(currentVehicle)
				local name = GetDisplayNameFromVehicleModel()
				local netId = -1
                CreateThread(function()
                    repeat
                        Wait(100)
                        netId = SafeNetId(currentVehicle)
                    until netId and netId > 0 and netId < 65535 and NetworkDoesNetworkIdExist(netId)
                    if netId ~= -1 then 
						isEnteringVehicle = false
						isInVehicle = false
						currentVehicle = 0
						currentSeat = 0
						TriggerServerEvent('baseevents:leftVehicle', currentVehicle, currentSeat, GetDisplayNameFromVehicleModel(GetEntityModel(currentVehicle)), netId)			
					end
                end)
			end
		end
		Citizen.Wait(50)
	end
end)

function GetPedVehicleSeat(ped)
    local vehicle = GetVehiclePedIsIn(ped, false)
    for i=-2,GetVehicleMaxNumberOfPassengers(vehicle) do
        if(GetPedInVehicleSeat(vehicle, i) == ped) then return i end
    end
    return -2
end
```
