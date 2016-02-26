--[[
Ideas:

Polish round system.

Add lives. with difficulty

Add run and run animation with leap and dive moves

--]]

math.randomseed(tick())

local won = false

local roundLive = false

local diff = script.Difficulty

--------------Default stats---------------

NumDoorsOpen = .8 -- percentage of doors open
KillerParts = .1 -- Percentage of killing parts /1
EscapeDoorAmount = .8 -- percentage of doors that will be live to escape
DoorsOpen = 5 -- Time doors are open in seconds

DoorsShut = 3 -- Time doors are shut
DoorsKillTime = 1 -- How long killing doors are live

-----------------Code-------------------

function RoundStart(plr)
	
	won = false -- Reset winner value	
	
	plr.PlayerGui:WaitForChild("ScreenGui").Frame.TextLabel.Text = "Round Starting..."
	
	local walls, doors = {}, {}
	for _, v in pairs(workspace.Maze:GetChildren()) do
		if v.Name == "Wall" then
			table.insert(walls, v)
			v.Transparency = 0 -- Door shut
			v.CanCollide = true
			v.BrickColor = BrickColor.new ("Medium stone grey")
			v.Material = Enum.Material.Slate
		elseif v.Name == "Door" or v.Name == "LiveDoor" then
			table.insert(doors, v)
			v.Name = "Door"
			v.BrickColor = BrickColor.new("White")
			v.CanCollide = true
		end
	end
	
	function InitiateWalls()
		-- Iterating through the entire table of walls
		-- Everything within this loop will work with the variable "wall"
		for _, wall in pairs(walls) do
			local kill = false -- Walls won't kill by default

			wall.Touched:connect(function(hit)
				if kill == true then
					if hit.Parent:FindFirstChild("Humanoid") then
						if hit.Parent:FindFirstChild("Humanoid").Health > 0 then -- Killing function
							hit.Parent.Humanoid.Health = 0
							--restart()
						end
					end
				end
			end)

			-- Optimizing this by making it local. 
			local function toggleKilling(bool) -- true is kill, false is safe
				kill = bool
				wall.BrickColor = bool and BrickColor.new ("Bright red") or BrickColor.new ("Medium stone grey")
				wall.Material = bool and Enum.Material.Neon or Enum.Material.Slate
			end

			-- Wrapping the function in a spawn/coroutine so we can continue the iteration.
			spawn(function()
				while roundLive == true do 
					toggleKilling(false)
					wall.Transparency = 0 -- Door shut
					wall.CanCollide = true

					local luck = math.random() <= KillerParts -- percentage of luck being true
					local NumberDoors = math.random() <= NumDoorsOpen -- percentage how many doors will be open

					if luck == false then
						toggleKilling(false) -- Doesn't kill check
						if NumberDoors then -- Puts percentage of doors open
							wall.Transparency = 1 -- Door open
							wall.CanCollide = false
						end
						wait(DoorsOpen) -- Time doors are open
					else
						toggleKilling(true) -- Will kill
						wait(DoorsKillTime)
						toggleKilling(false)
						if NumberDoors then -- Puts percentage of doors open
							wall.Transparency = 1 -- Door open
							wall.CanCollide = false
						end
						wait(DoorsOpen) -- Time doors are open
					end
				end
			end)
		end
	end

	function ChooseEscape()
		for _, door in pairs(doors) do 
			local WillItOpen = math.random() <= EscapeDoorAmount -- percentage that the door will open
			if WillItOpen == true then
				door.Name = "LiveDoor"
				door.BrickColor = BrickColor.new ("Bright blue")
				door.CanCollide = false
				door.Touched:connect(function(hit) -- Killing function
					if hit.Parent:FindFirstChild("Humanoid") then
						if hit.Parent.Humanoid.Health > 0 then
							if won == false then -- check if already won
								won = true
								restart(hit.Parent) -- restart with winner
							end
						end
					end
				end)
			else
				door.Name = "Door"
				door.BrickColor = BrickColor.new ("White")
				door.CanCollide = true
			end
		end
	end

	ChooseEscape() -- Choosing escape doors
	wait(5)
	plr.PlayerGui:WaitForChild("ScreenGui").Frame.TextLabel.Text = "Escape or Submit!"
	roundLive = true -- Round has started. This needs to be before initiateWalls, as it is needed in our loops.
	InitiateWalls() -- Initiating all the walls. The color, the transparency, etc. etc.
end

--------------Round Retart---------------------

function restart(winner)
	roundLive = false
	for _,player in pairs(game.Players:GetChildren()) do 
		if winner then
			player.PlayerGui.ScreenGui.Frame.TextLabel.Text = winner.Name.." Won!"
		end
		wait(2)
		player.Character.Humanoid.Health = 0
		plr = player
	end
	RoundStart(plr) -- starts round
end

--------------Difficulty changed---------------

diff.Changed:connect(function()
	
	restart() -- Restarts game
	
	if diff.Value >= 1 and diff.Value <= 3 then -- easy
		NumDoorsOpen = .7
		KillerParts = .2
		EscapeDoorAmount = .9
		DoorsOpen = 10
		DoorsShut = 10
		DoorsKillTime = 6
	end
	if diff.Value >= 4 and diff.Value <= 7 then -- medium
		NumDoorsOpen = .6
		KillerParts = .5
		EscapeDoorAmount = .3 
		DoorsOpen = 5
		DoorsShut = 5
		DoorsKillTime = 3
	end
	if diff.Value >= 8 and diff.Value <= 10 then -- hard
		NumDoorsOpen = .4
		KillerParts = .7
		EscapeDoorAmount = .1
		DoorsOpen = 3
		DoorsShut = 3
		DoorsKillTime = 2
	end
end) 

---------------Player Added--------------

game.Players.PlayerAdded:connect(function(plr)
	plr.CharacterAdded:connect(function()
		if diff.Value >= 1 and diff.Value <= 3 then -- easy
			plr.PlayerGui.ScreenGui.Frame.easy.TextColor3 = Color3.new(109/255, 171/255, 102/255)
			plr.PlayerGui.ScreenGui.Frame.med.TextColor3 = Color3.new(1,1,1)
			plr.PlayerGui.ScreenGui.Frame.hard.TextColor3 = Color3.new(1,1,1)
		end
		
		if diff.Value >= 4 and diff.Value <= 7 then -- medium
			plr.PlayerGui.ScreenGui.Frame.med.TextColor3 = Color3.new(177/255, 138/255, 75/255)
			plr.PlayerGui.ScreenGui.Frame.easy.TextColor3 = Color3.new(1,1,1)
			plr.PlayerGui.ScreenGui.Frame.hard.TextColor3 = Color3.new(1,1,1)
		end
		
		if diff.Value >= 8 and diff.Value <= 10 then -- hard
			plr.PlayerGui.ScreenGui.Frame.hard.TextColor3 = Color3.new(153/255, 56/255, 58/255)
			plr.PlayerGui.ScreenGui.Frame.easy.TextColor3 = Color3.new(1,1,1)
			plr.PlayerGui.ScreenGui.Frame.med.TextColor3 = Color3.new(1,1,1)
		end
		
		x = false

		plr.PlayerGui.ScreenGui.Frame.easy.MouseButton1Click:connect(function()
			if roundLive == true then
				if x == false then
					x = true
					plr.PlayerGui.ScreenGui.Frame.easy.TextColor3 = Color3.new(109/255, 171/255, 102/255)
					
					plr.PlayerGui.ScreenGui.Frame.med.TextColor3 = Color3.new(1,1,1)
					plr.PlayerGui.ScreenGui.Frame.hard.TextColor3 = Color3.new(1,1,1)
					
					workspace.MainNew.Difficulty.Value = 1
					wait(3)
					x = false
				end
			end
		end)
		
		plr.PlayerGui.ScreenGui.Frame.med.MouseButton1Click:connect(function()
			if roundLive == true then
				if x == false then
					x = true
					plr.PlayerGui.ScreenGui.Frame.med.TextColor3 = Color3.new(177/255, 138/255, 75/255)
					
					plr.PlayerGui.ScreenGui.Frame.easy.TextColor3 = Color3.new(1,1,1)
					plr.PlayerGui.ScreenGui.Frame.hard.TextColor3 = Color3.new(1,1,1)
					
					workspace.MainNew.Difficulty.Value = 5
					wait(3)
					x = false
				end
			end
		end)
		
		plr.PlayerGui.ScreenGui.Frame.hard.MouseButton1Click:connect(function()
			if roundLive == true then
				if x == false then
					x = true
					plr.PlayerGui.ScreenGui.Frame.hard.TextColor3 = Color3.new(153/255, 56/255, 58/255)
					
					plr.PlayerGui.ScreenGui.Frame.easy.TextColor3 = Color3.new(1,1,1)
					plr.PlayerGui.ScreenGui.Frame.med.TextColor3 = Color3.new(1,1,1)
					
					workspace.MainNew.Difficulty.Value = 10
					wait(3)
					x = false
				end
			end
		end)

		if not roundLive then RoundStart(plr) end
	end)
end)
