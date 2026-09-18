--[[
    HAXEGG - Script Auto Steal Egg untuk Steal An Egg
    Dev: habzz708
    Executor: Delta
    Versi: 1.0.0
]]

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

--============================================================
-- KONFIGURASI
--============================================================
local Config = {
    ScriptName = "HAXEGG",
    Versi      = "1.0.0",
    Developer  = "habzz708",

    DaftarRarity = {
        "Common", "Uncommon", "Rare", "Epic",
        "Legendary", "Mythic", "Secret", "Cosmic",
        "Eternal", "Divine",
    },

    JedaSteal     = 0.5,
    KecepatanAnim = 0.3,

    -- // GANTI dengan nama remote asli dari game
    NamaRemote = {
        StealEgg       = "StealEgg",
        ClaimEgg       = "ClaimEgg",
        GetEggContents = "GetEggContents",
    },

    PathContainerEgg = {"Workspace", "Eggs"}, -- // verifikasi dulu
}

--============================================================
-- STATE
--============================================================
local State = {
    Jalan          = false,
    RarityDipilih  = "Secret",
    JumlahSteal    = 0,
    WaktuMulai     = 0,
    Penghasilan    = 0,
    Thread         = nil,
}

--============================================================
-- UI
--============================================================
local function BuatUI()
    local lama = LocalPlayer.PlayerGui:FindFirstChild("HAXEGG")
    if lama then lama:Destroy() end

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "HAXEGG"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = LocalPlayer.PlayerGui

    --// JENDELA UTAMA
    local Main = Instance.new("Frame")
    Main.Size = UDim2.new(0, 340, 0, 520)
    Main.Position = UDim2.new(0.5, -170, 0.5, -260)
    Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
    Main.BackgroundTransparency = 0.05
    Main.BorderSizePixel = 0
    Main.Active = true
    Main.Draggable = true
    Main.Parent = ScreenGui

    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 14)

    local MainStroke = Instance.new("UIStroke")
    MainStroke.Color = Color3.fromRGB(110, 70, 220)
    MainStroke.Thickness = 2
    MainStroke.Transparency = 0.25
    MainStroke.Parent = Main

    --// TOP BAR
    local TopBar = Instance.new("Frame")
    TopBar.Size = UDim2.new(1, 0, 0, 64)
    TopBar.BackgroundColor3 = Color3.fromRGB(28, 24, 46)
    TopBar.BorderSizePixel = 0
    TopBar.Parent = Main
    Instance.new("UICorner", TopBar).CornerRadius = UDim.new(0, 14)

    local Judul = Instance.new("TextLabel")
    Judul.Size = UDim2.new(1, 0, 0, 30)
    Judul.Position = UDim2.new(0, 0, 0, 8)
    Judul.BackgroundTransparency = 1
    Judul.Text = "HAXEGG"
    Judul.TextColor3 = Color3.fromRGB(185, 135, 255)
    Judul.TextSize = 24
    Judul.Font = Enum.Font.GothamBold
    Judul.Parent = TopBar

    local SubJudul = Instance.new("TextLabel")
    SubJudul.Size = UDim2.new(1, 0, 0, 16)
    SubJudul.Position = UDim2.new(0, 0, 0, 38)
    SubJudul.BackgroundTransparency = 1
    SubJudul.Text = "by habzz708  ·  v" .. Config.Versi
    SubJudul.TextColor3 = Color3.fromRGB(130, 110, 170)
    SubJudul.TextSize = 11
    SubJudul.Font = Enum.Font.Gotham
    SubJudul.Parent = TopBar

    --// AREA KONTEN
    local Konten = Instance.new("Frame")
    Konten.Size = UDim2.new(1, -20, 1, -84)
    Konten.Position = UDim2.new(0, 10, 0, 74)
    Konten.BackgroundTransparency = 1
    Konten.Parent = Main

    local Layout = Instance.new("UIListLayout")
    Layout.Padding = UDim.new(0, 10)
    Layout.SortOrder = Enum.SortOrder.LayoutOrder
    Layout.Parent = Konten

    --// LABEL RARITY
    local LabelRarity = Instance.new("TextLabel")
    LabelRarity.Size = UDim2.new(1, 0, 0, 22)
    LabelRarity.BackgroundTransparency = 1
    LabelRarity.Text = "RARITY TARGET"
    LabelRarity.TextColor3 = Color3.fromRGB(160, 160, 190)
    LabelRarity.TextSize = 12
    LabelRarity.Font = Enum.Font.GothamBold
    LabelRarity.TextXAlignment = Enum.TextXAlignment.Left
    LabelRarity.LayoutOrder = 1
    LabelRarity.Parent = Konten

    --// DROPDOWN
    local Dropdown = Instance.new("Frame")
    Dropdown.Size = UDim2.new(1, 0, 0, 38)
    Dropdown.BackgroundColor3 = Color3.fromRGB(34, 30, 54)
    Dropdown.BorderSizePixel = 0
    Dropdown.LayoutOrder = 2
    Dropdown.Parent = Konten
    Instance.new("UICorner", Dropdown).CornerRadius = UDim.new(0, 8)

    local TombolDrop = Instance.new("TextButton")
    TombolDrop.Size = UDim2.new(1, 0, 1, 0)
    TombolDrop.BackgroundTransparency = 1
    TombolDrop.Text = State.RarityDipilih .. "   ▼"
    TombolDrop.TextColor3 = Color3.fromRGB(190, 140, 255)
    TombolDrop.TextSize = 15
    TombolDrop.Font = Enum.Font.GothamMedium
    TombolDrop.Parent = Dropdown

    local Menu = Instance.new("Frame")
    Menu.Size = UDim2.new(1, 0, 0, #Config.DaftarRarity * 30)
    Menu.Position = UDim2.new(0, 0, 1, 6)
    Menu.BackgroundColor3 = Color3.fromRGB(24, 20, 38)
    Menu.BorderSizePixel = 0
    Menu.Visible = false
    Menu.ZIndex = 10
    Menu.Parent = Dropdown
    Instance.new("UICorner", Menu).CornerRadius = UDim.new(0, 8)

    local MenuLayout = Instance.new("UIListLayout")
    MenuLayout.Parent = Menu

    for _, rarity in ipairs(Config.DaftarRarity) do
        local Opsi = Instance.new("TextButton")
        Opsi.Size = UDim2.new(1, 0, 0, 30)
        Opsi.BackgroundTransparency = 1
        Opsi.Text = rarity
        Opsi.TextColor3 = Color3.fromRGB(205, 205, 225)
        Opsi.TextSize = 13
        Opsi.Font = Enum.Font.Gotham
        Opsi.ZIndex = 11
        Opsi.Parent = Menu

        Opsi.MouseEnter:Connect(function()
            Opsi.BackgroundTransparency = 0.85
            Opsi.BackgroundColor3 = Color3.fromRGB(110, 70, 220)
        end)
        Opsi.MouseLeave:Connect(function()
            Opsi.BackgroundTransparency = 1
        end)

        Opsi.MouseButton1Click:Connect(function()
            State.RarityDipilih = rarity
            TombolDrop.Text = rarity .. "   ▼"
            Menu.Visible = false
            TweenService:Create(Dropdown, TweenInfo.new(0.1), {
                BackgroundColor3 = Color3.fromRGB(70, 50, 140)
            }):Play()
            task.wait(0.1)
            TweenService:Create(Dropdown, TweenInfo.new(0.15), {
                BackgroundColor3 = Color3.fromRGB(34, 30, 54)
            }):Play()
        end)
    end

    TombolDrop.MouseButton1Click:Connect(function()
        Menu.Visible = not Menu.Visible
        if Menu.Visible then
            Menu.Size = UDim2.new(1, 0, 0, 0)
            TweenService:Create(Menu, TweenInfo.new(Config.KecepatanAnim, Enum.EasingStyle.Quad), {
                Size = UDim2.new(1, 0, 0, #Config.DaftarRarity * 30)
            }):Play()
        end
    end)

    --// PANEL STATISTIK
    local Stats = Instance.new("Frame")
    Stats.Size = UDim2.new(1, 0, 0, 96)
    Stats.BackgroundColor3 = Color3.fromRGB(30, 27, 46)
    Stats.BorderSizePixel = 0
    Stats.LayoutOrder = 3
    Stats.Parent = Konten
    Instance.new("UICorner", Stats).CornerRadius = UDim.new(0, 10)

    local LabelSteal = Instance.new("TextLabel")
    LabelSteal.Size = UDim2.new(1, -16, 0, 22)
    LabelSteal.Position = UDim2.new(0, 8, 0, 10)
    LabelSteal.BackgroundTransparency = 1
    LabelSteal.Text = "Telur Dicuri:  0"
    LabelSteal.TextColor3 = Color3.fromRGB(140, 240, 140)
    LabelSteal.TextSize = 14
    LabelSteal.Font = Enum.Font.GothamMedium
    LabelSteal.TextXAlignment = Enum.TextXAlignment.Left
    LabelSteal.Name = "LabelSteal"
    LabelSteal.Parent = Stats

    local LabelHasil = Instance.new("TextLabel")
    LabelHasil.Size = UDim2.new(1, -16, 0, 22)
    LabelHasil.Position = UDim2.new(0, 8, 0, 34)
    LabelHasil.BackgroundTransparency = 1
    LabelHasil.Text = "Penghasilan Sesi:  0"
    LabelHasil.TextColor3 = Color3.fromRGB(255, 215, 120)
    LabelHasil.TextSize = 14
    LabelHasil.Font = Enum.Font.GothamMedium
    LabelHasil.TextXAlignment = Enum.TextXAlignment.Left
    LabelHasil.Name = "LabelHasil"
    LabelHasil.Parent = Stats

    local LabelStatus = Instance.new("TextLabel")
    LabelStatus.Size = UDim2.new(1, -16, 0, 20)
    LabelStatus.Position = UDim2.new(0, 8, 0, 62)
    LabelStatus.BackgroundTransparency = 1
    LabelStatus.Text = "Status:  Siaga"
    LabelStatus.TextColor3 = Color3.fromRGB(150, 150, 175)
    LabelStatus.TextSize = 12
    LabelStatus.Font = Enum.Font.Gotham
    LabelStatus.TextXAlignment = Enum.TextXAlignment.Left
    LabelStatus.Name = "LabelStatus"
    LabelStatus.Parent = Stats

    --// TOMBOL
    local BarisTombol = Instance.new("Frame")
    BarisTombol.Size = UDim2.new(1, 0, 0, 46)
    BarisTombol.BackgroundTransparency = 1
    BarisTombol.LayoutOrder = 4
    BarisTombol.Parent = Konten

    local LayoutTombol = Instance.new("UIListLayout")
    LayoutTombol.FillDirection = Enum.FillDirection.Horizontal
    LayoutTombol.Padding = UDim.new(0, 8)
    LayoutTombol.Parent = BarisTombol

    local TombolMulai = Instance.new("TextButton")
    TombolMulai.Size = UDim2.new(0.5, -4, 1, 0)
    TombolMulai.BackgroundColor3 = Color3.fromRGB(60, 180, 80)
    TombolMulai.BorderSizePixel = 0
    TombolMulai.Text = "▶  MULAI"
    TombolMulai.TextColor3 = Color3.fromRGB(255, 255, 255)
    TombolMulai.TextSize = 15
    TombolMulai.Font = Enum.Font.GothamBold
    TombolMulai.Name = "TombolMulai"
    TombolMulai.Parent = BarisTombol
    Instance.new("UICorner", TombolMulai).CornerRadius = UDim.new(0, 8)

    local TombolStop = Instance.new("TextButton")
    TombolStop.Size = UDim2.new(0.5, -4, 1, 0)
    TombolStop.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
    TombolStop.BorderSizePixel = 0
    TombolStop.Text = "■  STOP"
    TombolStop.TextColor3 = Color3.fromRGB(255, 255, 255)
    TombolStop.TextSize = 15
    TombolStop.Font = Enum.Font.GothamBold
    TombolStop.Name = "TombolStop"
    TombolStop.Parent = BarisTombol
    Instance.new("UICorner", TombolStop).CornerRadius = UDim.new(0, 8)

    --// PANEL ISI PET
    local LabelPet = Instance.new("TextLabel")
    LabelPet.Size = UDim2.new(1, 0, 0, 22)
    LabelPet.BackgroundTransparency = 1
    LabelPet.Text = "ISI TELUR"
    LabelPet.TextColor3 = Color3.fromRGB(160, 160, 190)
    LabelPet.TextSize = 12
    LabelPet.Font = Enum.Font.GothamBold
    LabelPet.TextXAlignment = Enum.TextXAlignment.Left
    LabelPet.LayoutOrder = 5
    LabelPet.Parent = Konten

    local PanelPet = Instance.new("ScrollingFrame")
    PanelPet.Size = UDim2.new(1, 0, 0, 140)
    PanelPet.BackgroundColor3 = Color3.fromRGB(30, 27, 46)
    PanelPet.BorderSizePixel = 0
    PanelPet.ScrollBarThickness = 4
    PanelPet.ScrollBarImageColor3 = Color3.fromRGB(110, 70, 220)
    PanelPet.CanvasSize = UDim2.new(0, 0, 0, 0)
    PanelPet.AutomaticCanvasSize = Enum.AutomaticSize.Y
    PanelPet.LayoutOrder = 6
    PanelPet.Name = "PanelPet"
    PanelPet.Parent = Konten
    Instance.new("UICorner", PanelPet).CornerRadius = UDim.new(0, 10)

    local LayoutPet = Instance.new("UIListLayout")
    LayoutPet.Padding = UDim.new(0, 4)
    LayoutPet.SortOrder = Enum.SortOrder.LayoutOrder
    LayoutPet.Parent = PanelPet

    local PadPet = Instance.new("UIPadding")
    PadPet.PaddingTop = UDim.new(0, 6)
    PadPet.PaddingBottom = UDim.new(0, 6)
    PadPet.PaddingLeft = UDim.new(0, 8)
    PadPet.PaddingRight = UDim.new(0, 8)
    PadPet.Parent = PanelPet

    return {
        ScreenGui = ScreenGui,
        TombolMulai = TombolMulai,
        TombolStop = TombolStop,
        LabelSteal = LabelSteal,
        LabelHasil = LabelHasil,
        LabelStatus = LabelStatus,
        PanelPet = PanelPet,
        Main = Main,
    }
end

--============================================================
-- HELPER ANIMASI
--============================================================
local function PulsaTombol(btn)
    local asli = btn.Size
    TweenService:Create(btn, TweenInfo.new(0.08, Enum.EasingStyle.Back), {
        Size = asli + UDim2.new(0, 4, 0, 4)
    }):Play()
    task.wait(0.08)
    TweenService:Create(btn, TweenInfo.new(0.18, Enum.EasingStyle.Elastic), {
        Size = asli
    }):Play()
end

local function GlowStroke(stroke)
    local tween = TweenService:Create(stroke, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
        Transparency = 0.6
    })
    tween:Play()
    return tween
end

local function FloatIn(frame, delay)
    delay = delay or 0
    task.wait(delay)
    frame.BackgroundTransparency = 1
    frame.Position = frame.Position + UDim2.new(0, 0, 0, 8)
    TweenService:Create(frame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundTransparency = 0,
        Position = frame.Position - UDim2.new(0, 0, 0, 8)
    }):Play()
end

local function GoyangFrame(frame)
    local asli = frame.Position
    for i = 1, 4 do
        TweenService:Create(frame, TweenInfo.new(0.04), {
            Position = asli + UDim2.new(0, math.random(-4, 4), 0, math.random(-4, 4))
        }):Play()
        task.wait(0.04)
    end
    TweenService:Create(frame, TweenInfo.new(0.08), { Position = asli }):Play()
end

--============================================================
-- ISI PANEL PET
--============================================================
local function BersihkanPanelPet(panel)
    for _, anak in ipairs(panel:GetChildren()) do
        if anak:IsA("TextLabel") or anak:IsA("Frame") then
            anak:Destroy()
        end
    end
end

local function TambahEntriPet(panel, namaPet, rarity, nilai, index)
    local Entri = Instance.new("Frame")
    Entri.Size = UDim2.new(1, 0, 0, 34)
    Entri.BackgroundColor3 = Color3.fromRGB(40, 36, 60)
    Entri.BackgroundTransparency = 0.3
    Entri.BorderSizePixel = 0
    Entri.LayoutOrder = index
    Entri.Parent = panel
    Instance.new("UICorner", Entri).CornerRadius = UDim.new(0, 6)

    local LabelNama = Instance.new("TextLabel")
    LabelNama.Size = UDim2.new(0.6, 0, 1, 0)
    LabelNama.Position = UDim2.new(0, 8, 0, 0)
    LabelNama.BackgroundTransparency = 1
    LabelNama.Text = namaPet
    LabelNama.TextColor3 = Color3.fromRGB(230, 230, 245)
    LabelNama.TextSize = 13
    LabelNama.Font = Enum.Font.GothamMedium
    LabelNama.TextXAlignment = Enum.TextXAlignment.Left
    LabelNama.Parent = Entri

    local LabelNilai = Instance.new("TextLabel")
    LabelNilai.Size = UDim2.new(0.4, -8, 1, 0)
    LabelNilai.Position = UDim2.new(0.6, 0, 0, 0)
    LabelNilai.BackgroundTransparency = 1
    LabelNilai.Text = tostring(nilai)
    LabelNilai.TextColor3 = Color3.fromRGB(255, 215, 120)
    LabelNilai.TextSize = 13
    LabelNilai.Font = Enum.Font.GothamBold
    LabelNilai.TextXAlignment = Enum.TextXAlignment.Right
    LabelNilai.Parent = Entri

    Entri.BackgroundTransparency = 1
    TweenService:Create(Entri, TweenInfo.new(0.3), {
        BackgroundTransparency = 0.3
    }):Play()
end

--============================================================
-- INTERAKSI GAME (ISI REMOTE ASLI DI SINI)
--============================================================
local function AmbilContainerEgg()
    local node = game
    for _, key in ipairs(Config.PathContainerEgg) do
        node = node:FindFirstChild(key)
        if not node then return nil end
    end
    return node
end

local function AmbilRemote(nama)
    local remote = ReplicatedStorage:FindFirstChild(nama)
    if not remote then
        local folder = ReplicatedStorage:FindFirstChild("Remotes")
            or ReplicatedStorage:FindFirstChild("RemoteEvents")
            or ReplicatedStorage:FindFirstChild("Net")
        if folder then remote = folder:FindFirstChild(nama) end
    end
    return remote
end

local function FilterEggByRarity(container, rarityTarget)
    local hasil = {}
    if not container then return hasil end
    for _, egg in ipairs(container:GetChildren()) do
        local rarity = egg:GetAttribute("Rarity")
            or (egg:FindFirstChild("Rarity") and egg.Rarity.Value)
        if rarity and string.lower(tostring(rarity)) == string.lower(rarityTarget) then
            table.insert(hasil, egg)
        end
    end
    return hasil
end

local function CuriEgg(egg)
    local remote = AmbilRemote(Config.NamaRemote.StealEgg)
        or AmbilRemote(Config.NamaRemote.ClaimEgg)
    if remote then
        remote:FireServer(egg)
        return true
    end
    return false
end

local function AmbilIsiEgg(egg)
    -- // ISI remote asli atau baca cache client di sini
    local isi = {}
    local folderIsi = egg:FindFirstChild("Contents")
    if folderIsi then
        for _, pet in ipairs(folderIsi:GetChildren()) do
            table.insert(isi, {
                Nama = pet.Name,
                Rarity = pet:GetAttribute("Rarity") or "Unknown",
                Nilai = pet:GetAttribute("Value") or 0,
            })
        end
    end
    return isi
end

--============================================================
-- LOOP UTAMA
--============================================================
local function LoopSteal(ui)
    State.WaktuMulai = tick()
    while State.Jalan do
        local container = AmbilContainerEgg()
        if container then
            local target = FilterEggByRarity(container, State.RarityDipilih)

            if #target > 0 then
                ui.LabelStatus.Text = "Status:  Ketemu " .. #target .. " telur"
                ui.LabelStatus.TextColor3 = Color3.fromRGB(180, 255, 180)

                for _, egg in ipairs(target) do
                    if not State.Jalan then break end

                    local isi = AmbilIsiEgg(egg)
                    BersihkanPanelPet(ui.PanelPet)
                    for i, pet in ipairs(isi) do
                        TambahEntriPet(ui.PanelPet, pet.Nama, pet.Rarity, pet.Nilai, i)
                        State.Penghasilan = State.Penghasilan + (pet.Nilai or 0)
                    end

                    local sukses = CuriEgg(egg)
                    if sukses then
                        State.JumlahSteal = State.JumlahSteal + 1
                        ui.LabelSteal.Text = "Telur Dicuri:  " .. State.JumlahSteal
                        ui.LabelHasil.Text = "Penghasilan Sesi:  " .. State.Penghasilan

                        GoyangFrame(ui.LabelSteal)
                        TweenService:Create(ui.LabelSteal, TweenInfo.new(0.15), {
                            TextColor3 = Color3.fromRGB(255, 255, 255)
                        }):Play()
                        task.wait(0.15)
                        TweenService:Create(ui.LabelSteal, TweenInfo.new(0.3), {
                            TextColor3 = Color3.fromRGB(140, 240, 140)
                        }):Play()
                    end

                    task.wait(Config.JedaSteal)
                end
            else
                ui.LabelStatus.Text = "Status:  Tidak ada telur " .. State.RarityDipilih
                ui.LabelStatus.TextColor3 = Color3.fromRGB(255, 180, 120)
            end
        else
            ui.LabelStatus.Text = "Status:  Container telur tidak ditemukan"
            ui.LabelStatus.TextColor3 = Color3.fromRGB(255, 120, 120)
        end

        task.wait(0.4)
    end

    ui.LabelStatus.Text = "Status:  Siaga"
    ui.LabelStatus.TextColor3 = Color3.fromRGB(150, 150, 175)
end

--============================================================
-- BOOT
--============================================================
local ui = BuatUI()

GlowStroke(ui.Main:FindFirstChildOfClass("UIStroke"))

FloatIn(ui.LabelSteal, 0.05)
FloatIn(ui.LabelHasil, 0.10)
FloatIn(ui.LabelStatus, 0.15)

ui.TombolMulai.MouseButton1Click:Connect(function()
    if State.Jalan then return end
    State.Jalan = true
    PulsaTombol(ui.TombolMulai)

    ui.LabelStatus.Text = "Status:  Mulai..."
    ui.LabelStatus.Tex