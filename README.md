# Ilocal WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Window = WindUI:CreateWindow({

    Title = "Painel Admin",

    Icon = "shield",

    Author = "Dark Hub",

    Folder = "MyTestHub",

    Size = UDim2.fromOffset(580, 460),

    Transparent = true,

    Theme = "Dark",

    Resizable = true,

    SideBarWidth = 200,

    Background = "",

    BackgroundImageTransparency = 0.42,

    HideSearchBar = true,

    ScrollBarEnabled = true,

    User = {

        Enabled = true,

        Anonymous = true,

        Callback = function()

            print("Perfil clicado")

        end,

    },

    KeySystem = { 

        Key = { "admin99", "5678" },

        Note = "Digite a chave para acessar o Painel Admin.",

        Thumbnail = {

            Image = "rbxassetid://",

            Title = "Key system",

        },

        URL = "https://github.com/Footagesus/WindUI",

        SaveKey = true,

    },

})

-- Serviços

local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer

local BannedPlayers = {}

-- TAB JOGADORES

local TabPlayers = Window:Tab({

    Title = "Jogadores",

    Icon = "users",

})

-- Função para criar botões de ações de jogador

local function AddPlayerButton(player)

    if player == LocalPlayer or BannedPlayers[player.Name] then return end

    -- Botão principal com Teleport e View

    TabPlayers:Button({

        Title = player.Name,

        Desc = "Clique para Teleport e View",

        Callback = function()

            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then

                LocalPlayer.Character:PivotTo(player.Character.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0))

                print("Teleportado até " .. player.Name)

            end

            if player.Character and player.Character:FindFirstChildWhichIsA("Humanoid") then

                workspace.CurrentCamera.CameraSubject = player.Character:FindFirstChildWhichIsA("Humanoid")

                print("Agora vendo " .. player.Name)

            end

        end

    })

    -- Kick atualizado: faz desaparecer e volta a câmera

    TabPlayers:Button({

        Title = "Kick " .. player.Name,

        Desc = "Desaparece o jogador e retorna a câmera",

        Callback = function()

            if player.Character then

                for _, part in pairs(player.Character:GetDescendants()) do

                    if part:IsA("BasePart") then

                        part.Transparency = 1

                        part.CanCollide = false

                    elseif part:IsA("Decal") then

                        part.Transparency = 1

                    end

                end

                if player.Character:FindFirstChildWhichIsA("Humanoid") then

                    workspace.CurrentCamera.CameraSubject = player.Character:FindFirstChildWhichIsA("Humanoid")

                end

                print(player.Name .. " desapareceu")

                task.delay(1, function()

                    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid") then

                        workspace.CurrentCamera.CameraSubject = LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")

                        print("Câmera voltou para você")

                    end

                end)

            end

        end

    })

    -- Ban

    TabPlayers:Button({

        Title = "Banir " .. player.Name,

        Desc = "Remove e bloqueia o jogador",

        Callback = function()

            BannedPlayers[player.Name] = true

            if player.Character then

                player:Kick("Você foi banido pelo Painel Admin.")

                print("Jogador banido:", player.Name)

            end

            TabPlayers:Clear()

            for _, p in ipairs(Players:GetPlayers()) do

                AddPlayerButton(p)

            end

        end

    })

end

-- Adiciona jogadores atuais

for _, p in ipairs(Players:GetPlayers()) do

    AddPlayerButton(p)

end

-- Quando entra alguém novo

Players.PlayerAdded:Connect(function(p)

    AddPlayerButton(p)

end)

-- Quando sai alguém, atualiza lista

Players.PlayerRemoving:Connect(function()

    TabPlayers:Clear()

    for _, p in ipairs(Players:GetPlayers()) do

        AddPlayerButton(p)

    end

end)

-- TAB FUNÇÕES ADMIN

local TabAdmin = Window:Tab({

    Title = "Funções Admin",

    Icon = "shield",

})

-- Fly integrado no WindUI

TabAdmin:Button({

    Title = "Ativar Fly",

    Desc = "Permite voar",

    Callback = function()

        local player = LocalPlayer

        local mouse = player:GetMouse()

        local flying = false

        local speed = 50

        local bodyGyro, bodyVelocity

        local function startFly()

            if flying then return end

            flying = true

            local char = player.Character

            if not char then return end

            local root = char:FindFirstChild("HumanoidRootPart")

            if not root then return end

            bodyGyro = Instance.new("BodyGyro", root)

            bodyGyro.P = 9e4

            bodyGyro.MaxTorque = Vector3.new(9e5, 9e5, 9e5)

            bodyGyro.CFrame = root.CFrame

            bodyVelocity = Instance.new("BodyVelocity", root)

            bodyVelocity.MaxForce = Vector3.new(9e5, 9e5, 9e5)

            bodyVelocity.Velocity = Vector3.new(0,0,0)

            task.spawn(function()

                while flying and root.Parent do

                    local cam = workspace.CurrentCamera

                    local direction = Vector3.new()

                    if mouse.KeyDown:Wait() == "w" then direction = direction + cam.CFrame.LookVector end

                    if mouse.KeyDown:Wait() == "s" then direction = direction - cam.CFrame.LookVector end

                    if mouse.KeyDown:Wait() == "a" then direction = direction - cam.CFrame.RightVector end

                    if mouse.KeyDown:Wait() == "d" then direction = direction + cam.CFrame.RightVector end

                    if direction.Magnitude > 0 then

                        bodyVelocity.Velocity = direction.Unit * speed

                    else

                        bodyVelocity.Velocity = Vector3.new(0,0,0)

                    end

                    bodyGyro.CFrame = CFrame.new(root.Position, root.Position + cam.CFrame.LookVector)

                    task.wait(0.03)

                end

            end)

        end

        local function stopFly()

            flying = false

            if bodyGyro then bodyGyro:Destroy() end

            if bodyVelocity then bodyVelocity:Destroy() end

        end

        if not flying then

            startFly()

            print("Fly ativado")

        else

            stopFly()

            print("Fly desativado")

        end

    end

})

-- TAG "BETA TESTER" RGB

task.spawn(function()

    while task.wait() do

        local tagColor = Color3.fromHSV(tick() % 5 / 5, 1, 1)

        -- PlayerList

        pcall(function()

            local playerGui = LocalPlayer:FindFirstChild("PlayerGui")

            if playerGui then

                for _, gui in pairs(playerGui:GetDescendants()) do

                    if gui:IsA("TextLabel") and string.find(gui.Text, LocalPlayer.Name) then

                        gui.Text = "[Beta Tester] " .. LocalPlayer.Name

                        gui.TextColor3 = tagColor

                    end

                end

            end

        end)

        -- Tag acima da cabeça

        pcall(function()

            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Head") then

                if not LocalPlayer.Character.Head:FindFirstChild("BetaTag") then

                    local bill = Instance.new("BillboardGui", LocalPlayer.Character.Head)

                    bill.Name = "BetaTag"

                    bill.Size = UDim2.new(0, 200, 0, 50)

                    bill.StudsOffset = Vector3.new(0, 2.5, 0)

                    bill.AlwaysOnTop = true

                    local text = Instance.new("TextLabel", bill)

                    text.Size = UDim2.new(1, 0, 1, 0)

                    text.BackgroundTransparency = 1

                    text.Font = Enum.Font.GothamBold

                    text.TextScaled = true

                    text.Text = "[Beta Tester] " .. LocalPlayer.Name

                    text.TextColor3 = tagColor

                else

                    LocalPlayer.Character.Head.BetaTag.TextLabel.TextColor3 = tagColor

                end

            end

        end)

    end

end)NCOMUM-HUB-ADM
