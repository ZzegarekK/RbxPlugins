local ZTools = {}
ZTools.version = "v1.2"

local chs = game:GetService("ChangeHistoryService")
local sel = game:GetService("Selection")
local uis = game:GetService("UserInputService")
local lighting = game:GetService("Lighting")
local ws = game:GetService("Workspace")
local runs = game:GetService("RunService")

connections = {}
local menu



-- HELPER TOOLS =======================================================================================================

local suspiciousPatterns = {
	"require%(%d+%)", -- Loads external modules (often used for backdoors)
	"getfenv%(", -- Function environment manipulation (can be used for exploits)
	"setfenv%(", -- Modifying function environments
	"loadstring%(", -- Executes dynamically generated code (dangerous!)
	"syn%.", -- Synapse X exploit API
	"crypt%.", -- Crypto API (rarely used in normal scripts)
	"debug%.", -- Debugging functions (can be used for exploits)
	"bit%.bnot", -- Bitwise obfuscation (used to hide code)
	"[\r\n]%s*%-%-%[%[", -- Multiline comments (often used in obfuscation)
	"shared%[", -- Shared table (used for hidden global values)
	"_G%[", -- Global variable table modification
	"game:HttpGet%(", -- Retrieves external scripts via HTTP
	"game:HttpPost%(", -- Sends data externally via HTTP
	"http%.request%(", -- General HTTP request function
	"syn%.request%(", -- Exploit-related HTTP function
	"request%(", -- Alternative HTTP request function (common in exploits)
	"http%.Get%(", -- Retrieves data from an external source
	"http%.Post%(", -- Posts data to an external server
	"0x[%da-fA-F]+", -- Hexadecimal obfuscation (hides values using hex)
	"string%.reverse%(", -- Reversing strings (used to hide malicious commands)
	"string%.char%(", -- Converts numbers to characters (used in obfuscation)
	"game%.HttpService%:Base64Decode%(", -- Base64 decoding (hides malicious scripts)
	"game%.HttpService%:Base64Encode%(", -- Base64 encoding (hides payloads)
	"\\%d%d%d", -- Escaped characters (can be used to obfuscate strings)
	"table%.concat%(", -- Concatenating obfuscated strings
	"os%.time%(", -- Used to generate dynamic code
	"pcall%(", -- Used in exploits to run dangerous code silently
	"xpcall%(" -- Similar to pcall, used for error handling in exploits
}

-- Function to scan scripts (including the selected script itself)
local function scanForViruses(instances)
	local foundScripts = {}

	for _, instance in ipairs(instances) do
		-- If the selected instance is a script itself, scan it
		if instance:IsA("Script") or instance:IsA("LocalScript") then
			local source
			local success, err = pcall(function()
				source = instance.Source
			end)

			if success and source then
				for _, pattern in ipairs(suspiciousPatterns) do
					if string.find(source, pattern) then
						table.insert(foundScripts, {instance, pattern})
						break
					end
				end
			end
		end

		-- Also scan all descendants of the selected instance
		for _, descendant in ipairs(instance:GetDescendants()) do
			if descendant:IsA("Script") or descendant:IsA("LocalScript") then
				local source
				local success, err = pcall(function()
					source = descendant.Source
				end)

				if success and source then
					for _, pattern in ipairs(suspiciousPatterns) do
						if string.find(source, pattern) then
							table.insert(foundScripts, {descendant, pattern})
							break
						end
					end
				end
			end
		end
	end

	return foundScripts
end

local lightingProps = {
	"Ambient",
	"Brightness",
	"ColorShift_Bottom",
	"ColorShift_Top",
	"EnvironmentDiffuseScale",
	"EnvironmentSpecularScale",
	"OutdoorAmbient",
	"ShadowSoftness",
	"ClockTime",
	"GeographicLatitude",
	"ExposureCompensation",
	"FogColor",
	"FogEnd",
	"FogStart"
}

local propToClassName = {
	["Color3"] = "Color3Value",
	["number"] = "NumberValue",
	["boolean"] = "BoolValue",
	["string"] = "StringValue"
}

local postEffects = {
	["BloomEffect"] = function(className)
		local new = Instance.new(className)
		new.Name = "_LightingChanger"..string.gsub(className,"Effect","")
		return new
	end,
	["BlurEffect"] = function(className)
		local new = Instance.new(className)
		new.Size = 0
		new.Name = "_LightingChanger"..string.gsub(className,"Effect","")
		return new
	end,
	["ColorCorrectionEffect"] = function(className)
		local new = Instance.new(className)
		new.Name = "_LightingChanger"..string.gsub(className,"Effect","")
		return new
	end,
	["DepthOfFieldEffect"] = function(className)
		local new = Instance.new(className)
		new.FarIntensity = 0
		new.NearIntensity = 0
		new.Name = "_LightingChanger"..string.gsub(className,"Effect","")
		return new
	end,
	["SunRaysEffect"] = function(className)
		local new = Instance.new(className)
		new.Intensity = 0
		new.Name = "_LightingChanger"..string.gsub(className,"Effect","")
		return new
	end,
}

local function loadFromFolder(folder)
	if folder.Name == "_LightingProperties" and folder:IsA("Folder") then
		for _,v in pairs(folder:GetChildren()) do
			if table.find(lightingProps,v.Name) and v:IsA("ValueBase") then
				lighting[v.Name] = v.Value

			elseif string.find(v.Name,"_LightingChanger") and v:IsA("PostEffect") then
				if lighting:FindFirstChild(v.Name) and lighting[v.Name]:IsA(v.ClassName) then lighting[v.Name]:Destroy() end

				local clone = v:Clone()
				clone.Parent = lighting
				clone.Enabled = true
			end
		end
	end
end

local function saveFolderToParent(parent)
	local folder = Instance.new("Folder",parent)
	folder.Name = "_LightingProperties"

	for _,prop in pairs(lightingProps) do
		local val = Instance.new(propToClassName[typeof(lighting[prop])], folder)
		val.Name = prop
		val.Value = lighting[prop]
	end

	local tempPostEffects = table.clone(postEffects)
	for _,v in pairs(lighting:GetChildren()) do
		if string.find(v.Name,"_LightingChanger") and v:IsA("PostEffect") then
			if tempPostEffects[v.ClassName] then tempPostEffects[v.ClassName] = nil end
			local clone = v:Clone()
			clone.Parent = folder
			clone.Enabled = false
		end
	end

	for className,func in tempPostEffects do
		func(className).Parent = folder
	end

	return folder
end



-- MENU BUILDERS =======================================================================================================

local function ToWO_initializer()
	-- build menu
	menu:AddSeparator()
	local ToWO_InitHighlightParts = menu:AddNewAction("ZTOOLS_ToWO_InitHighlightParts","Initialize highlight parts")
	local ToWO_InitTrussInners = menu:AddNewAction("ZTOOLS_ToWO_InitTrussInners","Initialize truss inners")
	local ToWO_DelHighlightParts = menu:AddNewAction("ZTOOLS_ToWO_DelHighlightParts","Delete highlight parts")
	local ToWO_DelTrussInners = menu:AddNewAction("ZTOOLS_ToWO_DelTrussInners","Delete truss inners")
	local ToWO_convertRopes = menu:AddNewAction("ZTOOLS_ToWO_convertRopes", "Convert ropes to obby standards")

	-- funcitonality
	local function initTrussInners()
		local recording = chs:TryBeginRecording("Initialize truss inners")
		local trussGlow = Instance.new("Part")
		trussGlow.Anchored = true
		trussGlow.CanCollide = false
		trussGlow.Name = "_Truss_Glow"
		trussGlow.Color = Color3.new(0,0,0)
		trussGlow.Material = Enum.Material.Neon

		for _,v in workspace:GetDescendants() do
			if v:IsA("TrussPart") then
				if v:FindFirstChild("_Truss_Glow") then
					for _,tg in pairs(v:GetChildren()) do
						tg:Destroy()
					end
				end

				local p = trussGlow:Clone()
				p.Parent = v
				p.CFrame = v.CFrame
				p.Size = v.Size-Vector3.new(.5,.5,.5)
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function delTrussInners()
		local recording = chs:TryBeginRecording("Delete truss inners")
		for _,v in workspace:GetDescendants() do
			if v:IsA("TrussPart") then
				if v:FindFirstChild("_Truss_Glow") then
					for _,tg in pairs(v:GetChildren()) do
						if tg:IsA("BasePart") and tg.Name == "_Truss_Glow" then
							tg:Destroy()
						end
					end
				end
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end


	local function initHighlightParts()
		local recording = chs:TryBeginRecording("Initialize highlight parts")
		for _,v in pairs(ws:GetDescendants()) do
			if v:IsA("BoolValue") and v.Name == "_ZekHighlightPart" and v.Parent:IsA("BasePart") then
				local neon:BasePart = v.Parent:FindFirstChild("_ZekHighlightPartNeon")
				if neon then neon:Destroy() end
				neon = v.Parent:Clone()
				neon:WaitForChild("_ZekHighlightPart"):Destroy()
				neon.Parent = v.Parent
				neon.Name = "_ZekHighlightPartNeon"
				neon.Material = Enum.Material.Neon
				neon.Transparency = .25

				v.Parent.Color = v:GetAttribute("BaseColor")
				neon.Color = v:GetAttribute("HighlightColor")
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function delHighlightParts()
		local recording = chs:TryBeginRecording("Delete highlight parts")
		for _,v in pairs(ws:GetDescendants()) do
			if v:IsA("BoolValue") and v.Name == "_ZekHighlightPart" and v.Parent:IsA("BasePart") then
				local neon:BasePart = v.Parent:FindFirstChild("_ZekHighlightPartNeon")
				if neon then neon:Destroy() end

				v.Parent.Color = v:GetAttribute("StudioColor")
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function convertRopes()
		local recording = chs:TryBeginRecording("Convert ropes")
		for _,v in workspace:GetDescendants() do
			if v.Name == "Rope" and v:IsA("Model") then
				for _,x in pairs(v:GetChildren()) do
					if string.find(x.Name, "RopeCap") then x:Destroy()
					elseif string.find(x.Name, "RopeShaft") then
						local np = Instance.new("Part",v)
						local propsLoop = {
							"Color",
							"Material",
							"Size",
							"CFrame",
							"Anchored",
						}
						for _,prop in {"Color", "Material", "Size", "CFrame", "Anchored"} do
							np[prop] = x[prop]
						end
						x:Destroy()
					else
						continue
					end

					if v.Parent:IsA("Model") and string.find(v.Parent.Name, "Rope") and #v.Parent:GetChildren() == 1 then
						local oldParent = v.Parent
						v.Parent = v.Parent.Parent
						oldParent:Destroy()
					end
					print("Rope found and changed.")
				end
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	-- connect functions to actions
	table.insert(connections, ToWO_InitHighlightParts.Triggered:Connect(initHighlightParts))
	table.insert(connections, ToWO_InitTrussInners.Triggered:Connect(initTrussInners))
	table.insert(connections, ToWO_DelHighlightParts.Triggered:Connect(delHighlightParts))
	table.insert(connections, ToWO_DelTrussInners.Triggered:Connect(delTrussInners))
	table.insert(connections, ToWO_convertRopes.Triggered:Connect(convertRopes))

	-- RUN CONTEXT
	if runs:IsRunning() then
		-- server
		if runs:IsServer() then
			initHighlightParts()
			initTrussInners()
		else

		end
	end

end

local function menu_initializer(plugin)

	-- CREATE BUTTONS AND SEPARATORS (and game-specific tools) ================================================================================

	menu = plugin:CreatePluginMenu("ZToolsMenuId", "ZekTools")

	local xAxis = menu:AddNewAction("ZTOOLS_Xaxis","Rotate scale on X axis")
	local yAxis = menu:AddNewAction("ZTOOLS_Yaxis","Rotate scale on Y axis")
	local zAxis = menu:AddNewAction("ZTOOLS_Zaxis","Rotate scale on Z axis")

	menu:AddSeparator()

	local parentTo = menu:AddNewAction("ZTOOLS_ParentTo","Parent selection to last selected")
	local parentUp = menu:AddNewAction("ZTOOLS_ParentUp","Parent selection up")

	menu:AddSeparator()

	local xport = menu:AddNewAction("ZTOOLS_xport","Export lighting as ValueBase objects into selection")
	local import = menu:AddNewAction("ZTOOLS_import","Import selection ValueBase objects as lighting")
	local save = menu:AddNewAction("ZTOOLS_save","Save current lighting as default")
	local load = menu:AddNewAction("ZTOOLS_load","Load default lighting")

	menu:AddSeparator()

	local scan = menu:AddNewAction("ZTOOLS_scan","Scan selected instances and their descendants for viruses")

	-- game specific id
	local ToWO_InitHighlightParts,ToWO_InitTrussInners,ToWO_DelHighlightParts,ToWO_DelTrussInners,ToWO_convertRopes
	if game.GameId == 7453397183 then -- ToWO
		ToWO_initializer()
	end 



	-- FUNCTIONALITY ================================================================================

	local function rotXAxis()
		local recording = chs:TryBeginRecording("Resize and rotate")
		for _,v in (sel:Get()) do
			if v:IsA("BasePart") and not v:IsA("Terrain") then
				v.CFrame = v.CFrame * CFrame.Angles(math.rad(90),0,0)
				v.Size = Vector3.new(v.Size.X,v.Size.Z,v.Size.Y)
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function rotYAxis()
		local recording = chs:TryBeginRecording("Resize and rotate")
		for _,v in (sel:Get()) do
			if v:IsA("BasePart") and not v:IsA("Terrain") then
				v.CFrame = v.CFrame * CFrame.Angles(0,math.rad(90),0)
				v.Size = Vector3.new(v.Size.Z,v.Size.Y,v.Size.X)
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function rotZAxis()
		local recording = chs:TryBeginRecording("Resize and rotate")
		for _,v in (sel:Get()) do
			if v:IsA("BasePart") and not v:IsA("Terrain") then
				v.CFrame = v.CFrame * CFrame.Angles(0,0,math.rad(90))
				v.Size = Vector3.new(v.Size.Y,v.Size.X,v.Size.Z)
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function moveParentUp()
		local recording = chs:TryBeginRecording("Change parent")
		for _,v in pairs(sel:Get()) do
			if v.Parent ~= game and v.Parent.Parent ~= game then
				v.Parent = v.Parent.Parent
			end
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function moveParentTo()
		local recording = chs:TryBeginRecording("Change parent")
		local sels = sel:Get()
		local parent = sels[#sels]
		table.remove(sels,#sels)
		for _,v in pairs(sels) do
			v.Parent = parent
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function lightingExport()
		local recording = chs:TryBeginRecording("Export lighting")
		local sels = sel:Get()
		for _,s in pairs(sels) do
			saveFolderToParent(s)
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function lightingImport()
		local recording = chs:TryBeginRecording("Import lighting")
		local sels = sel:Get()
		if #sels == 1 then
			loadFromFolder(sels[1])
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function saveDefaultLighting()
		local recording = chs:TryBeginRecording("Save default lighting")
		if lighting:FindFirstChild("_LightingProperties") then lighting._LightingProperties:Destroy() end
		saveFolderToParent(lighting)
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function loadDefaultLighting()
		local recording = chs:TryBeginRecording("Load default lighting")
		local default = lighting:FindFirstChild("_LightingProperties")
		if default then
			loadFromFolder(lighting._LightingProperties)
		else
			warn("Lighting has no default properties!")
		end
		chs:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
	end

	local function virusScanSelection()
		local selectedInstances = sel:Get()

		if #selectedInstances == 0 then
			warn("No instances selected. Please select a Script, Model, or Folder.")
		else
			local infectedScripts = scanForViruses(selectedInstances)

			if #infectedScripts > 0 then
				warn("Potential viruses found in the following scripts:")
				for _, scr in ipairs(infectedScripts) do
					warn("- ", scr[1], "pattern:", scr[2])
				end
			else
				print("No suspicious scripts detected in the selected items.")
			end
		end
	end

	-- connect functions to actions
	table.insert(connections, xAxis.Triggered:Connect(rotXAxis))
	table.insert(connections, yAxis.Triggered:Connect(rotYAxis))
	table.insert(connections, zAxis.Triggered:Connect(rotZAxis))
	table.insert(connections, parentUp.Triggered:Connect(moveParentUp))
	table.insert(connections, parentTo.Triggered:Connect(moveParentTo))
	table.insert(connections, xport.Triggered:Connect(lightingExport))
	table.insert(connections, import.Triggered:Connect(lightingImport))
	table.insert(connections, save.Triggered:Connect(saveDefaultLighting))
	table.insert(connections, load.Triggered:Connect(loadDefaultLighting))
	table.insert(connections, scan.Triggered:Connect(virusScanSelection))

end





function ZTools:Initialize(plugin)
	print("Initializing ZekTools...")

	local function pluginMenuInitialize(plugin)
		-- BUILD UI
		menu_initializer(plugin)

		local uiEnabled = false
		local function uiSetState(bool)

			if bool == nil then
				uiEnabled = not uiEnabled
			else
				uiEnabled = bool
			end

			if uiEnabled then
				menu:ShowAsync()
			end
		end

		-- UI activation keybind (SHIFT+Z)
		table.insert(connections, uis.InputBegan:Connect(function(inp, gpe)
			if not gpe then
				if inp.KeyCode == Enum.KeyCode.Z and uis:IsKeyDown(Enum.KeyCode.LeftShift) then
					uiSetState(true)
				elseif inp.KeyCode == Enum.KeyCode.Escape then
					uiSetState(false)
				end
			end
		end))
	end

	-- MAIN RUN CONTENT
	if runs:IsRunning() then
		-- server
		if runs:IsServer() then

		else
			pluginMenuInitialize(plugin)
		end
	else
		pluginMenuInitialize(plugin)
	end

	print("Initialized ZekTools "..self.version.."!")
end



function ZTools:Destroy()

	for _, conn in pairs(connections) do
		if conn.Disconnect then conn:Disconnect() end
	end
	table.clear(connections)

	if menu then
		pcall(function() menu:Destroy() end)
	end

end



return ZTools
