-- Aura Hub (Rayfield) - Script com função FLY (voo livre, velocidade ajustável)
-- Coloque este script como LocalScript no StarterPlayerScripts ou execute como LocalScript no client.
-- Requer Rayfield (https://sirius.menu/rayfield) conforme utilizado abaixo.

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "uuuuuuuu",
   Icon = nil,
   LoadingTitle = "oi",
   LoadingSubtitle = "by orescripts ( rayfield ) script quem fez eu nao sei so sei que eu sei nao sei oque sei mais eu sei que nao sei oque sei.",
   Theme = "Default",
   ToggleUIKeybind = "K",

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil,
      FileName = "auuuuujjjb"
   },

   Discord = {
      Enabled = true,
      Invite = "JF2F2RANud",
      RememberJoins = true
   },

   KeySystem = true,
   KeySettings = {
      Title = "Aura Keys",
      Subtitle = "Key System",
      Note = "Para conseguir a key, entre no discord da Aura",
      FileName = "Key",
      SaveKey = true,
      GrabKeyFromSite = false,
      Key = {"Aura", "Ore", "Aura+Ego"}
   }
})

local AuraHub = Window:CreateTab("Aura Hub", 4483362458)
local Farm = Window:CreateTab("Farm", 4483362458)

local SectionMain = AuraHub:CreateSection("Funções Principais")

local function MatarJogador()
   local player = game.Players.LocalPlayer
   if player and player.Character and player.Character:FindFirstChild("Humanoid") then
      player.Character.Humanoid.Health = 0
   end
end

local ButtonKill = Farm:CreateButton({
   Name = "Matar Jogador",
   Callback = MatarJogador
})

local ToggleFlash = AuraHub:CreateToggle({
   Name = "Flash",
   CurrentValue = false,
   Callback = function(Value)
      local player = game.Players.LocalPlayer
      if player and player.Character and player.Character:FindFirstChild("Humanoid") then
         if Value then
            player.Character.Humanoid.WalkSpeed = 50
         else
            player.Character.Humanoid.WalkSpeed = 16
         end
      end
   end
})

local SliderJump = Farm:CreateSlider({
   Name = "Pulo (JumpPower)",
   Range = {0, 100},
   Increment = 1,
   Suffix = "",
   CurrentValue = 50,
   Callback = function(Value)
      local player = game.Players.LocalPlayer
      if player and player.Character and player.Character:FindFirstChild("Humanoid") then
         player.Character.Humanoid.JumpPower = Value
      end
   end
})

local SectionNPC = Farm:CreateSection("NPCs")

local InputNPC = Farm:CreateInput({
   Name = "Nome do NPC",
   PlaceholderText = "Digite um NPC...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      -- aqui você pode adicionar ação para o texto digitado
   end
})

local DropdownNPC = Farm:CreateDropdown({
   Name = "Selecionar NPC",
   Options = {"Npc 1", "Npc 2", "Npc 3"},
   CurrentOption = "Npc 1",
   Callback = function(Option)
      -- ação para opção selecionada
   end
})

local SectionExtra = Farm:CreateSection("Extras")

local KeybindExample = Farm:CreateKeybind({
   Name = "Atalho de Teclado",
   CurrentKeybind = "F",
   HoldToInteract = false,
   Callback = function(Key)
      -- ação ao pressionar a tecla
   end
})

local ParagraphCreator = Farm:CreateParagraph({
   Title = "Criador",
   Content = "Aura+Ego"
})

-- FLY (VOAR) - implementação
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

local flying = false
local flySpeed = 50 -- valor inicial
local bv, bg -- BodyVelocity e BodyGyro
local controls = {F = 0, B = 0, L = 0, R = 0, Up = 0, Down = 0}
local renderConn, inputBeganConn, inputEndedConn, charAddedConn

local function getCharacter()
   return player.Character or player.CharacterAdded:Wait()
end

local function enableFly()
   local character = getCharacter()
   local hrp = character:FindFirstChild("HumanoidRootPart")
   local humanoid = character:FindFirstChild("Humanoid")
   if not (hrp and humanoid) then return end

   -- Ensure previous instances removed
   if bv then bv:Destroy() end
   if bg then bg:Destroy() end

   bv = Instance.new("BodyVelocity")
   bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
   bv.P = 1250
   bv.Velocity = Vector3.new(0, 0, 0)
   bv.Parent = hrp

   bg = Instance.new("BodyGyro")
   bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
   bg.P = 3000
   bg.CFrame = hrp.CFrame
   bg.Parent = hrp

   -- Freeze humanoid controls to avoid fighting physics (optional)
   humanoid.PlatformStand = true

   flying = true
end

local function disableFly()
   local character = player.Character
   if character then
      local humanoid = character:FindFirstChild("Humanoid")
      if humanoid then
         humanoid.PlatformStand = false
      end
   end
   if bv then
      bv:Destroy()
      bv = nil
   end
   if bg then
      bg:Destroy()
      bg = nil
   end
   flying = false
   -- reset controls
   controls = {F = 0, B = 0, L = 0, R = 0, Up = 0, Down = 0}
end

-- Atualiza velocidade do BodyVelocity constantemente
renderConn = RunService.RenderStepped:Connect(function(dt)
   if not flying then return end
   local character = player.Character
   if not character then return end
   local hrp = character:FindFirstChild("HumanoidRootPart")
   if not hrp or not bv or not bg then return end

   local cam = workspace.CurrentCamera
   if not cam then return end

   -- Direção horizontal baseada na câmera
   local forwardVec = cam.CFrame.LookVector
   local rightVec = cam.CFrame.RightVector

   local moveVec = Vector3.new(0, 0, 0)
   moveVec = moveVec + forwardVec * (controls.F - controls.B)
   moveVec = moveVec + rightVec * (controls.R - controls.L)

   -- vertical
   local vertical = (controls.Up - controls.Down)

   -- se não houver movimento horizontal, manter vel horizontal zero
   local horizontal = Vector3.new(moveVec.X, 0, moveVec.Z)
   if horizontal.Magnitude > 0 then
      horizontal = horizontal.Unit
   end

   local finalVelocity = Vector3.new(horizontal.X * flySpeed, vertical * flySpeed, horizontal.Z * flySpeed)
   bv.Velocity = finalVelocity

   -- manter orientação do personagem alinhada à câmera (opcional)
   bg.CFrame = CFrame.new(hrp.Position, hrp.Position + cam.CFrame.LookVector)
end)

-- Input
inputBeganConn = UserInputService.InputBegan:Connect(function(input, gameProcessed)
   if gameProcessed then return end
   if input.UserInputType == Enum.UserInputType.Keyboard then
      local key = input.KeyCode
      if key == Enum.KeyCode.W then controls.F = 1 end
      if key == Enum.KeyCode.S then controls.B = 1 end
      if key == Enum.KeyCode.A then controls.L = 1 end
      if key == Enum.KeyCode.D then controls.R = 1 end
      if key == Enum.KeyCode.Space then controls.Up = 1 end
      if key == Enum.KeyCode.LeftShift then controls.Down = 1 end
   end
end)

inputEndedConn = UserInputService.InputEnded:Connect(function(input, gameProcessed)
   if gameProcessed then return end
   if input.UserInputType == Enum.UserInputType.Keyboard then
      local key = input.KeyCode
      if key == Enum.KeyCode.W then controls.F = 0 end
      if key == Enum.KeyCode.S then controls.B = 0 end
      if key == Enum.KeyCode.A then controls.L = 0 end
      if key == Enum.KeyCode.D then controls.R = 0 end
      if key == Enum.KeyCode.Space then controls.Up = 0 end
      if key == Enum.KeyCode.LeftShift then controls.Down = 0 end
   end
end)

-- Garante que o voo seja desativado ao trocar de personagem
charAddedConn = player.CharacterAdded:Connect(function()
   -- se estava voando, reativa objetos no novo character
   if flying then
      -- um pequeno atraso para o character carregar
      wait(0.1)
      enableFly()
   end
end)

-- GUI: Toggle Fly e Slider de velocidade
local FlySection = AuraHub:CreateSection("Fly / Voar")

local FlyToggle = AuraHub:CreateToggle({
   Name = "FLY (Voar)",
   CurrentValue = false,
   Callback = function(Value)
      if Value then
         enableFly()
      else
         disableFly()
      end
   end
})

local FlySpeedSlider = AuraHub:CreateSlider({
   Name = "Velocidade do Voo",
   Range = {1, 200},
   Increment = 1,
   Suffix = "",
   CurrentValue = flySpeed,
   Callback = function(Value)
      flySpeed = Value
      -- atualiza imediamente o BodyVelocity caso esteja voando
      if bv then
         -- bv.Velocity será atualizado no RenderStepped; forçar valor curto
         bv.Velocity = Vector3.new(bv.Velocity.X, bv.Velocity.Y, bv.Velocity.Z)
      end
   end
})

-- Limpeza ao fechar/encerrar (opcional)
local function cleanup()
   if renderConn then renderConn:Disconnect() end
   if inputBeganConn then inputBeganConn:Disconnect() end
   if inputEndedConn then inputEndedConn:Disconnect() end
   if charAddedConn then charAddedConn:Disconnect() end
   disableFly()
end

-- Se quiser, pode expor um botão para desligar tudo (não obrigatório)
Farm:CreateButton({
   Name = "Desligar Fly e limpar",
   Callback = cleanup
})

Rayfield:LoadConfiguration()
