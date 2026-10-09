--[[
    PAINEL LEE  v1.0.7  -  FRUTIGER AERO UI (Info: Instagram support + creator credit)

    NOVIDADES DA v5.4
    - 3 NIVEIS DE EFEITOS DE ATAQUE REAJUSTADOS (Ajustes > Intensidade dos efeitos):
        * Suave : otimizacao leve. Efeitos quase iguais ao original (so um pouco menores/mais curtos);
                  desliga apenas Highlight. Fire, Smoke e Sparkles continuam ligados.
        * Medio : meio-termo entre os dois. Efeitos pela metade; desliga Highlight, Smoke e Sparkles.
        * Minimo: efeitos quase invisiveis (minusculos, curtissimos e ~92% transparentes);
                  desliga tambem o Fire.
      Obs.: efeitos disparados por :Emit() ignoram "Rate"; por isso a intensidade e controlada
      principalmente por tamanho, duracao e transparencia.

    NOVIDADES DA v5.3
    - ENGASGO NO 1o ATAQUE CORRIGIDO (causas encontradas na varredura):
        1) Efeitos novos eram tratados todos no mesmo quadro, sem limite de tempo.
           Agora os "imediatos" tem orcamento por quadro (CFG.InstantMs); o que passar
           vai para a fila e e feito nos quadros seguintes.
        2) Texturas/decals/malhas dos efeitos so eram limpos DEPOIS de entrarem na fila,
           quando o jogo ja tinha comecado a carregar o asset. Agora Decal, Texture,
           SpecialMesh, SurfaceAppearance e MeshPart sao limpos na hora (antes do 1o render),
           entao a textura do efeito nem chega a ser baixada/decodificada.
        3) RenderFidelity = Precise era forcado em TODA malha, inclusive nas partes dos
           efeitos (que carregavam a malha completa no 1o uso). Agora so o mapa que ja
           existia na ativacao (e estruturas fixas que chegam depois) recebem Precise.
        4) Cache de "esse Model nao e personagem" nunca expirava; agora expira em 3s
           (evita proteger/otimizar personagem errado quando ele chega aos poucos).
        5) Leituras de memoria e animacao do menu menos frequentes.
    - LIMPAR RAM de verdade:
        * Coleta de lixo COMPLETA do heap Lua (inclui o lixo dos scripts do jogo).
        * "Limpeza profunda": troca a qualidade grafica por ~1s e volta, o que faz o
          Roblox soltar o cache de texturas/malhas. A qualidade e SEMPRE restaurada.
        * Mostra o quanto liberou medindo a memoria de verdade (nada de numero falso).
        * O modo automatico (60s) continua leve, sem engasgar.
    - NOVO: OTIMIZAR PING (botao na aba Boost) + card de ping ao vivo.
        * Mede o ping antes, aplica os ajustes de rede do cliente, estabiliza a
          memoria/quadros e mede de novo, mostrando "antes -> depois".
        * Ping e latencia de rede: o script NAO consegue baixar abaixo do que a sua
          internet/servidor permitem. Ele remove o que piora o ping por culpa do cliente
          (lag de replicacao, rastreio de debug, picos de quadro).
    - Aba Ajustes: nova opcao "Limpeza profunda de RAM".
    - HUD agora mostra o ping.

    v5.2
    - Aba "Visual": limite de FPS (15-240 / Max) e filtro de cores vivas.
    - Menu arrasta de qualquer parte; nunca sai totalmente da tela.
    - Ao fechar o menu: remove o filtro de cores e devolve o limite de FPS.

    v5.1
    - Casas, arvores, pilares NAO deformam nem somem de longe (RenderFidelity Precise).
    - Botao LIMPAR RAM + limpeza automatica opcional.

    v5.0
    - Efeitos de ataque FRACOS de verdade (particulas, trails, beams).
    - Zero closures no caminho quente, varredura incremental em fatias de tempo.
    - Distancia de renderizacao NAO e tocada.
    - LeeFixLagCleanup: re-executar o script nao empilha conexoes.
    - Atalho: RightShift mostra/oculta o menu.
]]

local Players          = game:GetService("Players")
local Lighting         = game:GetService("Lighting")
local RunService       = game:GetService("RunService")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui          = game:GetService("CoreGui")
local Workspace        = game:GetService("Workspace")
local Stats            = game:GetService("Stats")
local SoundService     = game:GetService("SoundService")

while not Players.LocalPlayer do task.wait() end
local player = Players.LocalPlayer

local env = (getgenv and getgenv()) or _G
if env.LeeFixLagCleanup then
    pcall(env.LeeFixLagCleanup)
    env.LeeFixLagCleanup = nil
end

-- ============================================================
-- CONFIGURACAO
-- ============================================================
local CFG = {
    BudgetMs       = 2.5,    -- ms por frame gastos pelo script (cai sozinho se o FPS cair)
    InstantMs      = 1.2,    -- ms por frame para tratar efeitos/texturas novos na hora (v5.3)
    Level          = 2,      -- intensidade dos efeitos: 1 suave, 2 medio, 3 minimo
    WeakEffects    = true,   -- particulas, trails, beams, fire, smoke, highlight
    Textures       = true,   -- texturas, decals, materiais lisos
    Atmos          = true,   -- Atmosphere, Bloom, Blur, DoF, SunRays, ColorCorrection, Clouds
    Lighting       = true,   -- sombras globais, neblina, reflexos, luzes dinamicas, agua
    ProtectPlayers = true,   -- corpo, acessorios e ferramentas intactos
    ProtectNPCs    = true,   -- NPCs/inimigos com Humanoid intactos
    CharFx         = true,   -- partes extras presas ao personagem (auras/formas) contam como efeito
    Physics        = false,  -- CanTouch/CanQuery off em partes sem colisao (pode quebrar cliques/toques)
    KeepShape      = true,   -- casas/arvores/pilares mantem a forma de longe (RenderFidelity Precise)
    AutoRam        = false,  -- limpeza automatica (leve) de RAM a cada 60s
    DeepRam        = true,   -- botao LIMPAR RAM faz limpeza profunda (troca a qualidade por ~1s)
    ShowHud        = false,
    Brightness     = 1.5,    -- false para nao mexer no brilho

    -- visual (v5.2)
    Vibrant        = false,  -- filtro de cores vivas
    ColorSat       = 0.65,   -- saturacao  (-1 a 1)
    ColorCon       = 0.15,   -- contraste  (-0.5 a 0.5)
    ColorBri       = 0.05,   -- brilho do filtro (-0.3 a 0.3)
}

-- rate/life/size = multiplicadores (1 = original); fade = quanto da transparencia restante e somada
-- (0 = igual ao original, 1 = invisivel); offFx = classes de efeito desligadas neste nivel.
-- Medio fica exatamente no meio entre Suave e Minimo.
local LEVELS = {
    {name = "Suave",  rate = 0.80, life = 0.85, size = 0.85, fade = 0.08,
     offFx = {Highlight = true}},
    {name = "Médio",  rate = 0.40, life = 0.48, size = 0.50, fade = 0.50,
     offFx = {Highlight = true, Smoke = true, Sparkles = true}},
    {name = "Mínimo", rate = 0.03, life = 0.12, size = 0.15, fade = 0.92,
     offFx = {Highlight = true, Smoke = true, Sparkles = true, Fire = true}},
}

-- extras (agrupados numa tabela para nao estourar o limite de variaveis locais)
local EX = {
    VNAME = "Lee_VibrantColors",
    tapBlocked = false,
    instantSpent = 0,                       -- segundos gastos em handlers imediatos neste quadro
    instantMax = CFG.InstantMs / 1000,      -- orcamento dos handlers imediatos por quadro
    net = {},                               -- valores originais de rede (restaurados ao fechar)
}
pcall(function()
    local old = Lighting:FindFirstChild(EX.VNAME)
    if old then old:Destroy() end
end)

-- ============================================================
-- ESTADO
-- ============================================================
local enabled, shuttingDown, pendingReapply = false, false, false
local mode = "idle"                                  -- idle | scan | restore

local cache     = setmetatable({}, {__mode = "k"})   -- instancia -> valores originais
local kindMemo  = setmetatable({}, {__mode = "k"})   -- Model -> 1 jogador, 2 NPC, negativo = "comum" (hora da checagem)
local watched   = setmetatable({}, {__mode = "k"})
local detached  = {}                                 -- {instancia, parentOriginal} (forte)
local cacheCount = 0

local lightingOriginals, terrainOriginals = {}, {}

local permConns, boostConns, fxConns = {}, {}, {}
local workerConn

-- varredura incremental (pilha de "quadros": lista de filhos + indice + peso)
local fKids, fIdx, fN, fW, sp, doneW = {}, {}, {}, {}, 0, 0
-- fila de objetos novos
local pend, pHead, pTail = {}, 1, 0
-- restauracao
local rDone, rTotal = 0, 1

local function track(c) permConns[#permConns + 1] = c return c end

-- ============================================================
-- UTILITARIOS DE EFEITO (sem closures no caminho quente)
-- ============================================================
local SMOOTH = Enum.Material.SmoothPlastic
local NEON   = Enum.Material.Neon
local PERF   = Enum.RenderFidelity.Performance

local function scaleSeq(seq, f)
    local kps = seq.Keypoints
    local out = table.create(#kps)
    for i = 1, #kps do
        local k = kps[i]
        out[i] = NumberSequenceKeypoint.new(k.Time, k.Value * f, k.Envelope * f)
    end
    return NumberSequence.new(out)
end

local function liftSeq(seq, f)
    local kps = seq.Keypoints
    local out = table.create(#kps)
    for i = 1, #kps do
        local k = kps[i]
        local v = k.Value + (1 - k.Value) * f
        out[i] = NumberSequenceKeypoint.new(k.Time, v, math.min(k.Envelope, v, 1 - v))
    end
    return NumberSequence.new(out)
end

local function reparent(o, p) o.Parent = p end

-- ============================================================
-- PROTECAO (so vale para texturas/materiais; efeitos ignoram)
-- ============================================================
local BODY = {}
for _, n in ipairs({
    "Head", "Torso", "UpperTorso", "LowerTorso", "HumanoidRootPart",
    "Left Arm", "Right Arm", "Left Leg", "Right Leg",
    "LeftUpperArm", "LeftLowerArm", "LeftHand", "RightUpperArm", "RightLowerArm", "RightHand",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot", "RightUpperLeg", "RightLowerLeg", "RightFoot",
}) do BODY[n] = true end

local GEAR = {Accessory = true, Hat = true, Tool = true, HopperBin = true, Flag = true}

-- 1 = jogador, 2 = NPC, 0 = comum.
-- "Comum" e lembrado por 3s (nao para sempre): personagens que chegam aos poucos
-- (Humanoid/Player.Character ainda nao replicados) deixam de ser tratados como objeto comum.
local function charKind(m)
    local k = kindMemo[m]
    if k ~= nil then
        if k > 0 then return k end
        if os.clock() + k < 3 then return 0 end      -- k negativo = -horaDaChecagem
    end
    if Players:GetPlayerFromCharacter(m) then
        k = 1
    elseif m:FindFirstChildOfClass("Humanoid") then
        k = 2
    else
        kindMemo[m] = -os.clock()
        return 0
    end
    kindMemo[m] = k
    return k
end

local function shielded(obj)
    local camera = Workspace.CurrentCamera
    local par = obj.Parent
    local p = par
    while p and p ~= Workspace do
        if p == camera then return true end
        local cn = p.ClassName
        if GEAR[cn] then return true end
        if cn == "Model" then
            local k = charKind(p)
            if k == 1 then
                if not CFG.ProtectPlayers then return false end
                if not CFG.CharFx then return true end
                if par == p then return BODY[obj.Name] == true end
                return par.Parent == p and BODY[par.Name] == true
            elseif k == 2 then
                return CFG.ProtectNPCs
            end
        end
        p = p.Parent
    end
    return false
end

-- ============================================================
-- HANDLERS DE OTIMIZACAO / RESTAURACAO
-- ============================================================
local function optEmitter(o)
    if not CFG.WeakEffects or cache[o] then return end
    local L = LEVELS[CFG.Level]
    local life = o.Lifetime
    cache[o] = {o.Rate, life, o.Size, o.Transparency}
    cacheCount = cacheCount + 1
    o.Rate = o.Rate * L.rate
    o.Lifetime = NumberRange.new(life.Min * L.life, life.Max * L.life)
    o.Size = scaleSeq(o.Size, L.size)
    o.Transparency = liftSeq(o.Transparency, L.fade)
end
local function resEmitter(o, s)
    o.Rate = s[1]; o.Lifetime = s[2]; o.Size = s[3]; o.Transparency = s[4]
end

local function optTrail(o)
    if not CFG.WeakEffects or cache[o] then return end
    local L = LEVELS[CFG.Level]
    cache[o] = {o.Lifetime, o.Transparency, o.WidthScale}
    cacheCount = cacheCount + 1
    o.Lifetime = o.Lifetime * L.life
    o.Transparency = liftSeq(o.Transparency, L.fade)
    o.WidthScale = scaleSeq(o.WidthScale, L.size)
end
local function resTrail(o, s)
    o.Lifetime = s[1]; o.Transparency = s[2]; o.WidthScale = s[3]
end

local function optBeam(o)
    if not CFG.WeakEffects or cache[o] then return end
    local L = LEVELS[CFG.Level]
    cache[o] = {o.Width0, o.Width1, o.Transparency}
    cacheCount = cacheCount + 1
    o.Width0 = o.Width0 * L.size
    o.Width1 = o.Width1 * L.size
    o.Transparency = liftSeq(o.Transparency, L.fade)
end
local function resBeam(o, s)
    o.Width0 = s[1]; o.Width1 = s[2]; o.Transparency = s[3]
end

local function optOffFx(o)       -- Fire, Smoke, Sparkles, Highlight
    if not CFG.WeakEffects or cache[o] then return end
    if not LEVELS[CFG.Level].offFx[o.ClassName] then return end   -- este nivel mantem esse efeito
    if o.Enabled then
        cache[o] = {true}
        cacheCount = cacheCount + 1
        o.Enabled = false
    end
end

local function optOffLight(o)    -- luzes dinamicas
    if not CFG.Lighting or cache[o] then return end
    if o.Enabled then
        cache[o] = {true}
        cacheCount = cacheCount + 1
        o.Enabled = false
    end
end

local function resEnabled(o) o.Enabled = true end

local function watchOff(o)
    if watched[o] then return end
    watched[o] = true
    fxConns[#fxConns + 1] = o:GetPropertyChangedSignal("Enabled"):Connect(function()
        if enabled and CFG.Atmos and o.Enabled then o.Enabled = false end
    end)
end

local function optPostFx(o)      -- Bloom, Blur, DoF, SunRays, ColorCorrection, Clouds
    if o == EX.vib or o == EX.uiBlur then return end -- efeitos visuais proprios da interface
    if not CFG.Atmos then return end
    watchOff(o)
    if cache[o] then return end
    if o.Enabled then
        cache[o] = {true}
        cacheCount = cacheCount + 1
        o.Enabled = false
    end
end

local function optAtmosphere(o)
    if not CFG.Atmos then return end
    local par = o.Parent
    if par then
        detached[#detached + 1] = {o, par}
        cacheCount = cacheCount + 1
        o.Parent = nil
    end
end

local function optSurfaceAppearance(o)
    if not CFG.Textures or shielded(o) then return end
    local par = o.Parent
    if par then
        detached[#detached + 1] = {o, par}
        cacheCount = cacheCount + 1
        o.Parent = nil
    end
end

local function optDecal(o)
    if not CFG.Textures or cache[o] then return end
    local par = o.Parent
    if par == nil or shielded(o) then return end
    if par:IsA("BasePart") and par.Transparency > 0.85 then
        -- sprite de efeito (parte invisivel + decal): mantem, so enfraquece
        if not CFG.WeakEffects then return end
        local t = o.Transparency
        cache[o] = {1, t}
        cacheCount = cacheCount + 1
        o.Transparency = t + (1 - t) * LEVELS[CFG.Level].fade
        return
    end
    cache[o] = {2, o.Texture}
    cacheCount = cacheCount + 1
    o.Texture = ""
end
local function resDecal(o, s)
    if s[1] == 1 then o.Transparency = s[2] else o.Texture = s[2] end
end

local function optSpecialMesh(o)
    if not CFG.Textures or cache[o] then return end
    local tex = o.TextureId
    if tex == "" or shielded(o) then return end
    cache[o] = {tex}
    cacheCount = cacheCount + 1
    o.TextureId = ""
end
local function resSpecialMesh(o, s) o.TextureId = s[1] end

local function physOpt(o, s)
    if CFG.Physics and not o.CanCollide then
        s[7] = o.CanTouch
        s[8] = o.CanQuery
        o.CanTouch = false
        o.CanQuery = false
    end
end

-- v5.3: Precise so no mapa que ja existia na ativacao (modo scan) e em estruturas fixas
-- que chegam depois. Partes de efeito/projetil NAO sao forcadas a carregar a malha completa
-- (isso causava o engasgo no 1o ataque).
EX.wantPrecise = function(o)
    if mode == "scan" then return true end
    return o.Anchored and (o.CanCollide or o.Size.Magnitude >= 8)
end

local function optPart(o)
    if not CFG.Textures or cache[o] or shielded(o) then return end
    local s = {o.Material, o.CastShadow, o.Reflectance, o.MaterialVariant}
    cache[o] = s
    cacheCount = cacheCount + 1
    if o.Material ~= NEON and o.Material ~= SMOOTH then o.Material = SMOOTH end
    if s[4] ~= "" then o.MaterialVariant = "" end
    o.CastShadow = false
    o.Reflectance = 0
    physOpt(o, s)
end
local function resPart(o, s)
    o.Material = s[1]; o.CastShadow = s[2]; o.Reflectance = s[3]; o.MaterialVariant = s[4]
    if s[7] ~= nil then o.CanTouch = s[7]; o.CanQuery = s[8] end
end

local function optMeshPart(o)
    if not CFG.Textures or cache[o] or shielded(o) then return end
    local s = {o.Material, o.CastShadow, o.Reflectance, o.MaterialVariant, o.TextureID, o.RenderFidelity}
    cache[o] = s
    cacheCount = cacheCount + 1
    if s[5] ~= "" then o.TextureID = "" end          -- primeiro: a textura nem chega a carregar
    if o.Material ~= NEON and o.Material ~= SMOOTH then o.Material = SMOOTH end
    if s[4] ~= "" then o.MaterialVariant = "" end
    o.CastShadow = false
    o.Reflectance = 0
    -- Precise = malha completa em qualquer distancia (nao deforma nem some de longe)
    -- Performance = malha simplificada (era o que deformava casas/arvores/pilares)
    if CFG.KeepShape then
        if s[6] ~= Enum.RenderFidelity.Precise and EX.wantPrecise(o) then
            o.RenderFidelity = Enum.RenderFidelity.Precise
        end
    else
        o.RenderFidelity = PERF
    end
    physOpt(o, s)
end
local function resMeshPart(o, s)
    o.Material = s[1]; o.CastShadow = s[2]; o.Reflectance = s[3]; o.MaterialVariant = s[4]
    o.TextureID = s[5]; o.RenderFidelity = s[6]
    if s[7] ~= nil then o.CanTouch = s[7]; o.CanQuery = s[8] end
end

-- tabelas de despacho por ClassName
local HOPT, HRES, INSTANT = {}, {}, {}

for _, cn in ipairs({"Part", "WedgePart", "CornerWedgePart", "TrussPart", "SpawnLocation",
                     "Seat", "VehicleSeat"}) do
    HOPT[cn] = optPart; HRES[cn] = resPart
end
HOPT.MeshPart = optMeshPart;   HRES.MeshPart = resMeshPart

-- Union (blocos unidos, comuns em casas/pilares): mesmo cuidado de forma do MeshPart
HOPT.UnionOperation = function(o)
    if not CFG.Textures or cache[o] or shielded(o) then return end
    local s = {o.Material, o.CastShadow, o.Reflectance, o.MaterialVariant, false, o.RenderFidelity}
    cache[o] = s
    cacheCount = cacheCount + 1
    if o.Material ~= NEON and o.Material ~= SMOOTH then o.Material = SMOOTH end
    if s[4] ~= "" then o.MaterialVariant = "" end
    o.CastShadow = false
    o.Reflectance = 0
    if CFG.KeepShape and s[6] ~= Enum.RenderFidelity.Precise and EX.wantPrecise(o) then
        o.RenderFidelity = Enum.RenderFidelity.Precise
    end
    physOpt(o, s)
end
HRES.UnionOperation = function(o, s)
    o.Material = s[1]; o.CastShadow = s[2]; o.Reflectance = s[3]; o.MaterialVariant = s[4]
    o.RenderFidelity = s[6]
    if s[7] ~= nil then o.CanTouch = s[7]; o.CanQuery = s[8] end
end

HOPT.SpecialMesh = optSpecialMesh; HRES.SpecialMesh = resSpecialMesh
HOPT.Decal = optDecal;         HRES.Decal = resDecal
HOPT.Texture = optDecal;       HRES.Texture = resDecal
HOPT.SurfaceAppearance = optSurfaceAppearance

HOPT.ParticleEmitter = optEmitter; HRES.ParticleEmitter = resEmitter
HOPT.Trail = optTrail;             HRES.Trail = resTrail
HOPT.Beam = optBeam;               HRES.Beam = resBeam
for _, cn in ipairs({"Fire", "Smoke", "Sparkles", "Highlight"}) do
    HOPT[cn] = optOffFx; HRES[cn] = resEnabled
end
for _, cn in ipairs({"PointLight", "SpotLight", "SurfaceLight"}) do
    HOPT[cn] = optOffLight; HRES[cn] = resEnabled
end
for _, cn in ipairs({"BloomEffect", "BlurEffect", "DepthOfFieldEffect", "ColorCorrectionEffect",
                     "SunRaysEffect", "Clouds"}) do
    HOPT[cn] = optPostFx; HRES[cn] = resEnabled
end
HOPT.Atmosphere = optAtmosphere

-- Tratados na hora (antes do 1o render do objeto), dentro do orcamento EX.instantMax por quadro:
--  * efeitos (antes do primeiro Emit do jogo)
--  * tudo que carrega asset de textura (v5.3): assim a textura do efeito nem e baixada
-- O que estourar o orcamento do quadro vai para a fila e e feito nos quadros seguintes.
for _, cn in ipairs({"ParticleEmitter", "Trail", "Beam", "Fire", "Smoke", "Sparkles", "Highlight",
                     "PointLight", "SpotLight", "SurfaceLight", "BloomEffect", "BlurEffect",
                     "DepthOfFieldEffect", "ColorCorrectionEffect", "SunRaysEffect", "Clouds",
                     "Atmosphere",
                     "Decal", "Texture", "SpecialMesh", "SurfaceAppearance", "MeshPart"}) do
    INSTANT[cn] = true
end

local function process(o)
    local f = HOPT[o.ClassName]
    if f then pcall(f, o) end
end

local function onAdded(o)
    local cn = o.ClassName
    local f = HOPT[cn]
    if not f then return end
    if INSTANT[cn] and EX.instantSpent < EX.instantMax then
        local t = os.clock()
        pcall(f, o)
        EX.instantSpent = EX.instantSpent + (os.clock() - t)
    else
        pTail = pTail + 1
        pend[pTail] = o
    end
end

local function drainPending(t0, budget)
    local n = 0
    while pHead <= pTail do
        local o = pend[pHead]
        pend[pHead] = nil
        pHead = pHead + 1
        if o.Parent then process(o) end
        n = n + 1
        if n % 16 == 0 and os.clock() - t0 >= budget then return false end
    end
    pHead, pTail = 1, 0
    return true
end

-- ============================================================
-- ILUMINACAO / TERRENO  (distancia de renderizacao NAO e alterada)
-- ============================================================
local function setSaved(store, inst, prop, value)
    pcall(function()
        if store[prop] == nil then store[prop] = inst[prop] end
        inst[prop] = value
    end)
end

local function restoreStore(inst, store)
    for prop, val in pairs(store) do
        pcall(function() inst[prop] = val end)
    end
    table.clear(store)
end

local function applyGlobalSettings()
    if not CFG.Lighting then return end
    setSaved(lightingOriginals, Lighting, "GlobalShadows", false)
    setSaved(lightingOriginals, Lighting, "FogEnd", 9e9)
    setSaved(lightingOriginals, Lighting, "FogStart", 9e9)
    setSaved(lightingOriginals, Lighting, "EnvironmentSpecularScale", 0)
    if CFG.Brightness then setSaved(lightingOriginals, Lighting, "Brightness", CFG.Brightness) end
    local terrain = Workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        setSaved(terrainOriginals, terrain, "WaterWaveSize", 0)
        setSaved(terrainOriginals, terrain, "WaterWaveSpeed", 0)
        setSaved(terrainOriginals, terrain, "WaterReflectance", 0)
        setSaved(terrainOriginals, terrain, "Decoration", false)
    end
end

local function restoreGlobalSettings()
    restoreStore(Lighting, lightingOriginals)
    local terrain = Workspace:FindFirstChildOfClass("Terrain")
    if terrain then restoreStore(terrain, terrainOriginals) end
end

local function disconnectBoost()
    for k, c in pairs(boostConns) do c:Disconnect(); boostConns[k] = nil end
    for i = #fxConns, 1, -1 do fxConns[i]:Disconnect(); fxConns[i] = nil end
    table.clear(watched)
end

-- restaura em fatias de tempo; retorna true quando terminou
local function restoreChunk(t0, budget)
    local n = 0
    while #detached > 0 do
        local d = detached[#detached]
        detached[#detached] = nil
        pcall(reparent, d[1], d[2])
        rDone = rDone + 1
        n = n + 1
        if n % 32 == 0 and os.clock() - t0 >= budget then return false end
    end
    local k, v = next(cache)
    while k ~= nil do
        local nk, nv = next(cache, k)
        cache[k] = nil
        local f = HRES[k.ClassName]
        if f then pcall(f, k, v) end
        rDone = rDone + 1
        n = n + 1
        k, v = nk, nv
        if n % 32 == 0 and os.clock() - t0 >= budget then return false end
    end
    return true
end

-- ============================================================
-- EXTRAS: CORES VIVAS + LIMITE DE FPS + RESTAURACAO DE QUALIDADE/REDE
-- So a logica fica aqui; a interface fica na aba "Visual".
-- ============================================================
EX.applyVibrant = function()
    if CFG.Vibrant then
        local fx = EX.vib
        if not (fx and fx.Parent) then
            fx = Instance.new("ColorCorrectionEffect")
            fx.Name = EX.VNAME
            EX.vib = fx                  -- ANTES de parentear: o otimizador precisa ignorar este efeito
            fx.Parent = Lighting
        end
        fx.Saturation = CFG.ColorSat
        fx.Contrast   = CFG.ColorCon
        fx.Brightness = CFG.ColorBri
        fx.Enabled    = true
    elseif EX.vib then
        EX.vib.Enabled = false
    end
end

EX.setCap = (type(setfpscap) == "function" and setfpscap)
         or (type(set_fps_cap) == "function" and set_fps_cap) or nil
EX.getCap = (type(getfpscap) == "function" and getfpscap) or nil
EX.fps = {min = 15, max = 245, step = 5, cur = 60, touched = false, orig = nil}  -- 245 = "Max"

if EX.getCap then
    local ok, v = pcall(EX.getCap)
    if ok and type(v) == "number" then
        local F = EX.fps
        F.orig = v
        if v <= 0 or v >= F.max then
            F.cur = F.max
        else
            F.cur = math.clamp(math.floor(v / F.step + 0.5) * F.step, F.min, F.max)
        end
    end
end

EX.applyFps = function(v)
    EX.fps.cur = v
    if not EX.setCap then return false end
    EX.fps.touched = true
    pcall(EX.setCap, v >= EX.fps.max and 999 or v)
    return true
end

-- devolve a qualidade grafica caso a limpeza profunda a tenha trocado
EX.restoreQuality = function()
    local q = EX.q0
    if q ~= nil then
        EX.q0 = nil
        pcall(function() settings().Rendering.QualityLevel = q end)
    end
end

-- devolve os valores originais de rede alterados pelo "Otimizar Ping"
EX.restoreNet = function()
    local ok, net = pcall(function() return settings().Network end)
    if ok and net then
        for prop, v in pairs(EX.net) do
            pcall(function() net[prop] = v end)
        end
    end
    table.clear(EX.net)
end

EX.cleanup = function()
    local fx = EX.vib
    EX.vib = nil
    if fx then pcall(fx.Destroy, fx) end
    local uiBlur = EX.uiBlur
    EX.uiBlur = nil
    if uiBlur then pcall(uiBlur.Destroy, uiBlur) end
    if EX.fps.touched and EX.setCap then
        EX.fps.touched = false
        pcall(EX.setCap, EX.fps.orig or 60)      -- devolve o limite original (ou 60 se desconhecido)
    end
    pcall(EX.restoreQuality)
    pcall(EX.restoreNet)
end

-- ============================================================
-- INTERFACE  -  HELPERS
-- ============================================================
local C = {
    -- Frutiger Aero: agua luminosa, céu azul, verde-folha e vidro polido.
    Bg     = Color3.fromRGB(17, 86, 101),
    Panel  = Color3.fromRGB(196, 239, 233),
    Card   = Color3.fromRGB(244, 255, 249),
    CardHi = Color3.fromRGB(160, 222, 207),
    Cyan   = Color3.fromRGB(0, 190, 222),
    Blue   = Color3.fromRGB(36, 135, 207),
    Mint   = Color3.fromRGB(27, 166, 116),
    Leaf   = Color3.fromRGB(100, 190, 66),
    Lime   = Color3.fromRGB(184, 224, 91),
    Text   = Color3.fromRGB(14, 63, 72),
    Muted  = Color3.fromRGB(76, 123, 130),
    Off    = Color3.fromRGB(49, 145, 177), -- estado desligado em azul, sem vermelho
    On     = Color3.fromRGB(39, 162, 103),
    Busy   = Color3.fromRGB(193, 157, 53),
    White  = Color3.new(1, 1, 1),
}
local HEX_MUTED = "#4C7B82"

local BRAND = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 193, 225)),
    ColorSequenceKeypoint.new(0.34, Color3.fromRGB(37, 139, 207)),
    ColorSequenceKeypoint.new(0.70, Color3.fromRGB(41, 179, 120)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(163, 216, 86)),
})

local function new(class, props, parent)
    local inst = Instance.new(class)
    for k, v in pairs(props) do inst[k] = v end
    if parent then inst.Parent = parent end
    return inst
end

local function tween(inst, t, props, style, dir)
    local tw = TweenService:Create(inst, TweenInfo.new(t, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out), props)
    tw:Play()
    return tw
end

local function corner(p, r) return new("UICorner", {CornerRadius = UDim.new(0, r)}, p) end
local function stroke(p, color, th, tr)
    return new("UIStroke", {Color = color, Thickness = th, Transparency = tr or 0,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border}, p)
end
local function grad(p, seq, rot) return new("UIGradient", {Color = seq, Rotation = rot or 0}, p) end

-- Reflexo de vidro no topo: leve o bastante para nao atrapalhar os textos.
local function gloss(parent, radius, alpha, heightScale)
    local shine = new("Frame", {
        Name = "AeroGloss",
        Position = UDim2.fromOffset(2, 2),
        Size = UDim2.new(1, -4, heightScale or 0.46, -2),
        BackgroundColor3 = C.White,
        BackgroundTransparency = alpha or 0.66,
        BorderSizePixel = 0,
        ZIndex = parent.ZIndex,
        Active = false,
    }, parent)
    corner(shine, math.max(3, (radius or 12) - 2))
    local fade = grad(shine, ColorSequence.new(C.White, C.White), 90)
    fade.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.06),
        NumberSequenceKeypoint.new(0.52, 0.38),
        NumberSequenceKeypoint.new(1, 1),
    })
    return shine
end

-- Anima o relevo: o controle cresce ao receber o mouse e salta ao ser solto.
local function elevate(btn, scaler, shadow)
    local hovered = false
    local shadowBase = shadow and shadow.Position or nil
    local function moveShadow(extraY, transparency, duration)
        if not (shadow and shadowBase and shadow.Parent) then return end
        tween(shadow, duration or 0.14, {
            Position = UDim2.new(shadowBase.X.Scale, shadowBase.X.Offset,
                shadowBase.Y.Scale, shadowBase.Y.Offset + extraY),
            BackgroundTransparency = transparency,
        }, Enum.EasingStyle.Quad)
    end
    track(btn.MouseEnter:Connect(function()
        hovered = true
        tween(scaler, 0.18, {Scale = 1.035}, Enum.EasingStyle.Back)
        moveShadow(2, 0.48, 0.16)
    end))
    track(btn.MouseLeave:Connect(function()
        hovered = false
        tween(scaler, 0.16, {Scale = 1}, Enum.EasingStyle.Quad)
        moveShadow(0, 0.64, 0.16)
    end))
    track(btn.MouseButton1Down:Connect(function()
        tween(scaler, 0.07, {Scale = 0.94}, Enum.EasingStyle.Quad)
        moveShadow(-2, 0.38, 0.07)
    end))
    track(btn.MouseButton1Up:Connect(function()
        tween(scaler, 0.20, {Scale = hovered and 1.04 or 1.015}, Enum.EasingStyle.Back)
        moveShadow(hovered and 2 or 0, hovered and 0.50 or 0.64, 0.12)
        task.delay(0.16, function()
            if scaler.Parent then
                tween(scaler, 0.12, {Scale = hovered and 1.035 or 1}, Enum.EasingStyle.Quad)
            end
        end)
    end))
end

local function bubble(parent, x, y, size, alpha)
    local b = new("Frame", {
        Name = "AeroBubble",
        Position = UDim2.new(x, 0, 0, y), Size = UDim2.fromOffset(size, size),
        BackgroundColor3 = C.White, BackgroundTransparency = alpha or 0.40,
        BorderSizePixel = 0, Active = false,
    }, parent)
    corner(b, size)
    local g = grad(b, ColorSequence.new(C.White, C.Cyan), 45)
    g.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.20),
        NumberSequenceKeypoint.new(0.65, 0.45),
        NumberSequenceKeypoint.new(1, 0.22),
    })
    stroke(b, C.White, 1.2, 0.22)
    local glint = new("Frame", {Position = UDim2.new(0.22, 0, 0.16, 0),
        Size = UDim2.new(0.25, 0, 0.13, 0), BackgroundColor3 = C.White,
        BackgroundTransparency = 0.12, BorderSizePixel = 0, Active = false}, b)
    corner(glint, size)
    return b
end

-- Pequena marca de bolhas feita com formas da GUI: nao depende de suporte a emoji/fontes.
local function bubbleIcon(parent, x, y, size, alpha)
    local b = new("Frame", {
        Name = "PainelLeeBubbleMark",
        Position = UDim2.fromOffset(x, y), Size = UDim2.fromOffset(size, size),
        BackgroundColor3 = C.White, BackgroundTransparency = alpha or 0.42,
        BorderSizePixel = 0, Active = false,
    }, parent)
    corner(b, size)
    local g = grad(b, ColorSequence.new(Color3.fromRGB(244, 255, 255), C.Cyan), 45)
    g.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.10),
        NumberSequenceKeypoint.new(0.60, 0.35),
        NumberSequenceKeypoint.new(1, 0.16),
    })
    stroke(b, C.White, 1.25, 0.12)
    local glint = new("Frame", {
        Position = UDim2.new(0.20, 0, 0.14, 0), Size = UDim2.new(0.34, 0, 0.15, 0),
        BackgroundColor3 = C.White, BackgroundTransparency = 0.04, BorderSizePixel = 0,
        Active = false,
    }, b)
    corner(glint, size)
    return b
end

local function label(props, parent)
    props.BackgroundTransparency = 1
    props.Font = props.Font or Enum.Font.GothamMedium
    props.TextColor3 = props.TextColor3 or C.Text
    props.TextSize = props.TextSize or 12
    return new("TextLabel", props, parent)
end

local function fmtNum(n)
    if n >= 1000 then return string.format("%.1fk", n / 1000) end
    return tostring(n)
end

local function isPress(input)
    local t = input.UserInputType
    return t == Enum.UserInputType.MouseButton1 or t == Enum.UserInputType.Touch
end

-- Audio isolado em um escopo proprio para nao estourar o limite de locais do Luau.
-- A faixa indicada toca em loop enquanto o painel está aberto; os IDs dos cliques ficam iguais.
local UIAudio = (function()
    local API = {}
    local ContentProvider = game:GetService("ContentProvider")
    local ids = {
        intro = {
            "rbxassetid://75400675151988", -- efeito/trecho de abertura
        },
        music = {
            "rbxassetid://78750951244228", -- música de fundo enquanto o painel está aberto
        },
        click = {
            "rbxassetid://82845990304289", -- glass-style UI click
            "rbxassetid://77120543307812", -- bubble click
            "rbxassetid://4110925712",     -- short chime fallback
            "rbxassetid://158050602",      -- classic chime fallback
        },
    }
    local cfg = {
        intro = {volume = 0.22, speed = 0.90, timeout = 1.8, fadeInSeconds = 0.12},
        music = {volume = 0.22, speed = 0.90, timeout = 2.5, fadeInSeconds = 0.75, fadeOutSeconds = 0.24},
        click = {volume = 0.16, speed = 1.08, timeout = 1.25},
    }
    local states = {
        intro = {loading = false, ready = false, failed = false, pending = false, sound = nil, playToken = 0, fadeTween = nil},
        music = {loading = false, ready = false, failed = false, pending = false, sound = nil, playToken = 0, fadeTween = nil},
        click = {loading = false, ready = false, failed = false, pending = false, sound = nil, playToken = 0, fadeTween = nil},
    }
    local disposed = false
    local loadSound

    local function cancelFade(state)
        if state.fadeTween then
            pcall(function() state.fadeTween:Cancel() end)
            state.fadeTween = nil
        end
    end

    local function playLoaded(kind)
        local state = states[kind]
        local sound = state and state.sound
        if disposed or not (sound and sound.Parent and sound.IsLoaded) then return end
        state.playToken = state.playToken + 1
        cancelFade(state)
        pcall(function()
            sound:Stop()
            sound.TimePosition = 0
            sound.PlaybackSpeed = cfg[kind].speed
            if kind == "click" then
                sound.Volume = cfg.click.volume
            else
                sound.Volume = 0
            end
            sound:Play()
            if kind ~= "click" then
                state.fadeTween = TweenService:Create(
                    sound,
                    TweenInfo.new(cfg[kind].fadeInSeconds, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                    {Volume = cfg[kind].volume}
                )
                state.fadeTween:Play()
            end
        end)
    end

    loadSound = function(kind)
        local state = states[kind]
        if disposed or not state or state.loading or state.ready or state.failed then return end
        state.loading = true
        task.spawn(function()
            for _, assetId in ipairs(ids[kind]) do
                if disposed or shuttingDown then
                    state.loading = false
                    return
                end

                local sound = Instance.new("Sound")
                if kind == "intro" then
                    sound.Name = "PainelLee_OpeningSound"
                elseif kind == "music" then
                    sound.Name = "PainelLee_BackgroundMusic"
                else
                    sound.Name = "PainelLee_GlassClick"
                end
                sound.SoundId = assetId
                sound.Volume = cfg[kind].volume
                sound.PlaybackSpeed = cfg[kind].speed
                sound.Looped = (kind == "music")
                sound.Parent = SoundService

                local preloadFinished = false
                local fetchSucceeded = false
                task.spawn(function()
                    local ok = pcall(function()
                        ContentProvider:PreloadAsync({sound}, function(_, status)
                            if status == Enum.AssetFetchStatus.Success then fetchSucceeded = true end
                        end)
                    end)
                    if not ok then fetchSucceeded = false end
                    preloadFinished = true
                end)

                local deadline = os.clock() + cfg[kind].timeout
                while os.clock() < deadline do
                    if sound.IsLoaded and sound.TimeLength > 0 then
                        fetchSucceeded = true
                        break
                    end
                    if preloadFinished then break end
                    task.wait(0.05)
                end
                if fetchSucceeded and not sound.IsLoaded then task.wait(0.12) end

                if disposed or shuttingDown then
                    pcall(function() sound:Destroy() end)
                    state.loading = false
                    return
                end

                if sound.IsLoaded and sound.TimeLength > 0 then
                    state.sound = sound
                    state.ready = true
                    state.loading = false
                    if state.pending then
                        state.pending = false
                        playLoaded(kind)
                    end
                    return
                end
                pcall(function() sound:Destroy() end)
            end

            state.loading = false
            state.failed = true
            state.pending = false
            warn("[Painel Lee] Nao foi possivel carregar o audio de " .. kind .. ". Confira permissao/disponibilidade dos IDs no Roblox.")
        end)
    end

    function API.play(kind)
        local state = states[kind]
        if disposed or shuttingDown or not state then return end
        if state.ready and state.sound and state.sound.Parent and state.sound.IsLoaded then
            playLoaded(kind)
            return
        end
        state.pending = true
        loadSound(kind)
    end

    function API.stop(kind, fadeSeconds)
        local state = states[kind]
        if disposed or not state then return end
        state.pending = false
        state.playToken = state.playToken + 1
        local token = state.playToken
        local sound = state.sound
        if not (sound and sound.Parent) then return end
        cancelFade(state)

        local fade = tonumber(fadeSeconds) or 0
        if fade > 0 and sound.IsPlaying then
            state.fadeTween = TweenService:Create(
                sound,
                TweenInfo.new(fade, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {Volume = 0}
            )
            state.fadeTween:Play()
            task.delay(fade + 0.06, function()
                if disposed or state.sound ~= sound or state.playToken ~= token then return end
                cancelFade(state)
                pcall(function()
                    sound:Stop()
                    sound.Volume = cfg[kind].volume
                end)
            end)
        else
            pcall(function()
                sound:Stop()
                sound.Volume = cfg[kind].volume
            end)
        end
    end

    function API.cleanup()
        disposed = true
        for _, state in pairs(states) do
            state.pending = false
            state.playToken = state.playToken + 1
            cancelFade(state)
            if state.sound then
                pcall(function() state.sound:Stop(); state.sound:Destroy() end)
                state.sound = nil
            end
        end
    end

    -- Pré-carrega todos os sons em segundo plano, sem atrasar a montagem da GUI.
    loadSound("intro")
    loadSound("music")
    loadSound("click")
    return API
end)()

-- clique/toque que ignora o "solta" logo depois de arrastar o menu (evita clique acidental)
local function onTap(btn, fn)
    return track(btn.Activated:Connect(function(...)
        if EX.tapBlocked then return end
        UIAudio.play("click")
        fn(...)
    end))
end

-- Blur nativo do Roblox atua no mundo 3D inteiro (nao so atras do painel).
-- Ele fica bem suave e ativo apenas enquanto a janela principal esta aberta.
local glassBlurTween, glassBlurVersion = nil, 0
local function setGlassBlur(active)
    local camera = Workspace.CurrentCamera
    if not camera then return end
    if not (EX.uiBlur and EX.uiBlur.Parent) then
        local existing = camera:FindFirstChild("PainelLeeBackdropBlur")
        if existing and existing:IsA("BlurEffect") then
            EX.uiBlur = existing
        else
            EX.uiBlur = Instance.new("BlurEffect")
            EX.uiBlur.Name = "PainelLeeBackdropBlur"
            EX.uiBlur.Size = 0
            EX.uiBlur.Enabled = false
            EX.uiBlur.Parent = camera
        end
    elseif EX.uiBlur.Parent ~= camera then
        EX.uiBlur.Parent = camera
    end
    glassBlurVersion = glassBlurVersion + 1
    local version = glassBlurVersion
    if glassBlurTween then glassBlurTween:Cancel(); glassBlurTween = nil end
    if active then
        EX.uiBlur.Enabled = true
        glassBlurTween = tween(EX.uiBlur, 0.24, {Size = 5}, Enum.EasingStyle.Quad)
    else
        glassBlurTween = tween(EX.uiBlur, 0.20, {Size = 0}, Enum.EasingStyle.Quad)
        task.delay(0.24, function()
            if version == glassBlurVersion and EX.uiBlur and EX.uiBlur.Parent then
                EX.uiBlur.Enabled = false
            end
        end)
    end
end

-- limpa UIs antigas (nomes antigos e novo, inclusive o painel de FPS/cores separado)
for _, getParent in ipairs({
    function() return gethui and gethui() end,
    function() return CoreGui end,
    function() return player:FindFirstChildOfClass("PlayerGui") end,
}) do
    pcall(function()
        local c = getParent()
        if c then
            for _, nm in ipairs({"LeeFixLagUI", "Lee_Fps", "LeeOptimizerUI", "MultiBoosterLeeUI", "PainelLeeUI"}) do
                local old = c:FindFirstChild(nm)
                if old then old:Destroy() end
            end
        end
    end)
end

local gui = new("ScreenGui", {
    Name = "PainelLeeUI", ResetOnSpawn = false, DisplayOrder = 999,
    IgnoreGuiInset = true, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
})
if syn and syn.protect_gui then pcall(syn.protect_gui, gui) end

-- mantem uma parte do objeto sempre visivel na tela (padX/padY = pixels que precisam ficar visiveis)
local function clampPos(obj, pos, padX, padY)
    local vp = gui.AbsoluteSize
    local size, anc = obj.AbsoluteSize, obj.AnchorPoint
    if vp.X < 10 or vp.Y < 10 or size.X < 1 then return pos end
    padX, padY = math.min(padX, size.X), math.min(padY, size.Y)
    local left = pos.X.Scale * vp.X + pos.X.Offset - anc.X * size.X
    local top  = pos.Y.Scale * vp.Y + pos.Y.Offset - anc.Y * size.Y
    local nl = math.max(padX - size.X, math.min(vp.X - padX, left))
    local nt = math.max(0, math.min(vp.Y - padY, top))
    return UDim2.new(pos.X.Scale, pos.X.Offset + (nl - left), pos.Y.Scale, pos.Y.Offset + (nt - top))
end

-- ============================================================
-- INTERFACE  -  JANELA PRINCIPAL
-- ============================================================
local root = new("CanvasGroup", {
    Name = "Root", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.new(0.5, 0, 0.5, 0),
    Size = UDim2.fromOffset(392, 440), BackgroundTransparency = 1, GroupTransparency = 1, Visible = false,
}, gui)
local autoScale = new("UIScale", {}, root)

-- Halo feito com formas arredondadas nativas, em vez de uma imagem 9-slice que podia
-- deixar pequenas quinas nas extremidades conforme a escala da janela.
local outerGlow = new("Frame", {
    Name = "AeroOuterGlow", AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.new(1, 22, 1, 22),
    BackgroundColor3 = C.Cyan, BackgroundTransparency = 0.76, BorderSizePixel = 0,
    Active = false,
}, root)
corner(outerGlow, 29)
local outerGlowStroke = stroke(outerGlow, C.Cyan, 2, 0.62)

local innerGlow = new("Frame", {
    Name = "AeroInnerGlow", AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.new(1, 10, 1, 10),
    BackgroundColor3 = C.Mint, BackgroundTransparency = 0.88, BorderSizePixel = 0,
    Active = false,
}, root)
corner(innerGlow, 23)

local main = new("Frame", {
    Name = "Main", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(1, 1), BackgroundColor3 = C.White, BackgroundTransparency = 0.31, BorderSizePixel = 0, ClipsDescendants = true,
    Active = true,
}, root)
corner(main, 18)
main.ClipsDescendants = true
local mainGlassGrad = grad(main, ColorSequence.new(Color3.fromRGB(250, 255, 253), Color3.fromRGB(172, 226, 218)), 90)
mainGlassGrad.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0.26),
    NumberSequenceKeypoint.new(0.48, 0.44),
    NumberSequenceKeypoint.new(1, 0.32),
})
local popScale = new("UIScale", {Scale = 1}, main)
local mainStroke = stroke(main, C.White, 2.2, 0.02)
local strokeGrad = grad(mainStroke, BRAND, 0)

-- Faixa de reflexo e bolhas d'agua; decoracao fica atras de todo o conteudo.
local panelShine = new("Frame", {Name = "GlassReflection", Position = UDim2.fromOffset(2, 2),
    Size = UDim2.new(1, -4, 0, 86), BackgroundColor3 = C.White, BackgroundTransparency = 0.76,
    BorderSizePixel = 0, Active = false, ClipsDescendants = true}, main)
corner(panelShine, 16)
local panelShineGrad = grad(panelShine, ColorSequence.new(C.White, Color3.fromRGB(202, 244, 241)), 90)
panelShineGrad.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0.12),
    NumberSequenceKeypoint.new(0.62, 0.58),
    NumberSequenceKeypoint.new(1, 1),
})
-- Bolhas mantidas na zona interna do vidro. Antes algumas tocavam o canto arredondado
-- e eram cortadas pela borda retangular dos descendentes, produzindo pontas visuais.
local waterBubbles = {
    bubble(main, 0.78, 108, 20, 0.62),
    bubble(main, 0.82, 154, 12, 0.69),
    bubble(main, 0.77, 218, 17, 0.72),
    bubble(main, 0.82, 288, 13, 0.70),
    bubble(main, 0.76, 346, 20, 0.74),
}
-- Bolhas de agua flutuam lentamente, criando um detalhe vivo sem competir com os textos.
for i, waterBubble in ipairs(waterBubbles) do
    local basePos = waterBubble.Position
    local floatY = 7 + i * 1.5
    local floatTween = TweenService:Create(waterBubble,
        TweenInfo.new(2.8 + i * 0.42, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        {Position = UDim2.new(basePos.X.Scale, basePos.X.Offset, basePos.Y.Scale, basePos.Y.Offset - floatY)})
    floatTween:Play()
end

-- cabecalho
local header = new("Frame", {Name = "Header", Size = UDim2.new(1, 0, 0, 60), BackgroundTransparency = 1, Active = true}, main)

do
    local logo = new("Frame", {Position = UDim2.fromOffset(14, 11), Size = UDim2.fromOffset(38, 38),
        BackgroundColor3 = C.White, BorderSizePixel = 0}, header)
    corner(logo, 11)
    grad(logo, BRAND, 45)
    stroke(logo, C.White, 1.3, 0.08)
    gloss(logo, 11, 0.40, 0.54)
    bubbleIcon(logo, 6, 14, 16, 0.26)
    bubbleIcon(logo, 22, 7, 8, 0.18)

    -- Marca de bolhas ao lado do nome, desenhada na GUI para nao virar um quadrado vazio.
    bubbleIcon(header, 62, 15, 14, 0.28)
    bubbleIcon(header, 74, 10, 7, 0.18)
    local titleLbl = label({
        Position = UDim2.fromOffset(86, 9), Size = UDim2.new(1, -202, 0, 24), Text = "Painel Lee",
        Font = Enum.Font.GothamBlack, TextSize = 17, TextColor3 = C.Text, TextXAlignment = Enum.TextXAlignment.Left,
    }, header)
    grad(titleLbl, BRAND, 0)
    label({
        Position = UDim2.fromOffset(86, 33), Size = UDim2.new(1, -202, 0, 14), Text = "FRUTIGER AERO  •  v1.0.0",
        Font = Enum.Font.GothamBold, TextSize = 10, TextColor3 = C.Muted, TextXAlignment = Enum.TextXAlignment.Left,
    }, header)
end

local function iconBtn(txt, xOff, hoverColor)
    local b = new("TextButton", {
        AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, xOff, 0, 16), Size = UDim2.fromOffset(28, 28),
        BackgroundColor3 = C.Card, Text = txt, TextColor3 = C.Muted, Font = Enum.Font.GothamBlack,
        TextSize = 16, AutoButtonColor = false, BorderSizePixel = 0,
    }, header)
    corner(b, 8)
    local sc = new("UIScale", {Scale = 1}, b)
    gloss(b, 8, 0.58, 0.50)
    elevate(b, sc)
    track(b.MouseEnter:Connect(function() tween(b, 0.15, {BackgroundColor3 = hoverColor, TextColor3 = C.White}) end))
    track(b.MouseLeave:Connect(function() tween(b, 0.15, {BackgroundColor3 = C.Card, TextColor3 = C.Muted}) end))
    return b
end
local MUSIC_UI = {enabled = true}
local minBtn   = iconBtn("–", -50, C.CardHi)
local closeBtn = iconBtn("×", -14, C.Off)
MUSIC_UI.button = iconBtn("", -86, C.CardHi)

-- Ícone de alto-falante desenhado com formas da própria GUI (sem emoji/fonte externa).
MUSIC_UI.body = new("Frame", {
    Position = UDim2.fromOffset(6, 10), Size = UDim2.fromOffset(5, 8),
    BackgroundColor3 = C.Blue, BorderSizePixel = 0,
}, MUSIC_UI.button)
corner(MUSIC_UI.body, 2)
MUSIC_UI.top = new("Frame", {
    Position = UDim2.fromOffset(9, 8), Size = UDim2.fromOffset(8, 4), Rotation = -32,
    BackgroundColor3 = C.Blue, BorderSizePixel = 0,
}, MUSIC_UI.button)
corner(MUSIC_UI.top, 2)
MUSIC_UI.bottom = new("Frame", {
    Position = UDim2.fromOffset(9, 16), Size = UDim2.fromOffset(8, 4), Rotation = 32,
    BackgroundColor3 = C.Blue, BorderSizePixel = 0,
}, MUSIC_UI.button)
corner(MUSIC_UI.bottom, 2)
MUSIC_UI.waves = label({
    Position = UDim2.fromOffset(14, 6), Size = UDim2.fromOffset(12, 17), Text = "))",
    Font = Enum.Font.GothamBold, TextSize = 13, TextColor3 = C.Blue,
    TextXAlignment = Enum.TextXAlignment.Left, TextYAlignment = Enum.TextYAlignment.Center,
}, MUSIC_UI.button)
MUSIC_UI.mutedMark = label({
    Position = UDim2.fromOffset(14, 5), Size = UDim2.fromOffset(13, 18), Text = "×",
    Font = Enum.Font.GothamBlack, TextSize = 17, TextColor3 = C.Off,
    TextXAlignment = Enum.TextXAlignment.Center, TextYAlignment = Enum.TextYAlignment.Center,
    Visible = false,
}, MUSIC_UI.button)
MUSIC_UI.render = function()
    MUSIC_UI.waves.Visible = MUSIC_UI.enabled
    MUSIC_UI.mutedMark.Visible = not MUSIC_UI.enabled
    MUSIC_UI.body.BackgroundColor3 = MUSIC_UI.enabled and C.Blue or C.Muted
    MUSIC_UI.top.BackgroundColor3 = MUSIC_UI.body.BackgroundColor3
    MUSIC_UI.bottom.BackgroundColor3 = MUSIC_UI.body.BackgroundColor3
end
MUSIC_UI.render()

new("Frame", {Position = UDim2.fromOffset(0, 60), Size = UDim2.new(1, 0, 0, 1), BackgroundColor3 = C.White,
    BackgroundTransparency = 0.92, BorderSizePixel = 0}, main)

-- abas
local TAB_NAMES = {"Boost", "Visual", "Ajustes", "Info"}

local tabBar = new("Frame", {Position = UDim2.fromOffset(14, 72), Size = UDim2.new(1, -28, 0, 36),
    BackgroundColor3 = C.Panel, BackgroundTransparency = 0.10, BorderSizePixel = 0}, main)
corner(tabBar, 11)
grad(tabBar, ColorSequence.new(Color3.fromRGB(222, 249, 245), Color3.fromRGB(166, 222, 216)), 90)
stroke(tabBar, C.White, 1.25, 0.18)
gloss(tabBar, 11, 0.78, 0.52)
local indicator = new("Frame", {Position = UDim2.new(0, 3, 0, 3), Size = UDim2.new(1 / #TAB_NAMES, -4, 1, -6),
    BackgroundColor3 = C.White, BorderSizePixel = 0}, tabBar)
corner(indicator, 9)
grad(indicator, BRAND, 0)
gloss(indicator, 9, 0.70, 0.54)

local tabBtns = {}
for i, name in ipairs(TAB_NAMES) do
    tabBtns[i] = new("TextButton", {
        Position = UDim2.new((i - 1) / #TAB_NAMES, 0, 0, 0), Size = UDim2.new(1 / #TAB_NAMES, 0, 1, 0),
        BackgroundTransparency = 1, Text = name, Font = Enum.Font.GothamBold, TextSize = 13,
        TextColor3 = C.Muted, AutoButtonColor = false,
    }, tabBar)
end

local pages = new("Frame", {Position = UDim2.fromOffset(14, 118), Size = UDim2.new(1, -28, 1, -132),
    BackgroundTransparency = 1, ClipsDescendants = true}, main)

-- ============================================================
-- PAGINA: BOOST
-- ============================================================
local pBoost = new("Frame", {Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1}, pages)

local card = new("Frame", {Size = UDim2.new(1, 0, 0, 84), BackgroundColor3 = C.Card,
    BackgroundTransparency = 0.06, BorderSizePixel = 0}, pBoost)
corner(card, 14)
grad(card, ColorSequence.new(Color3.fromRGB(255, 255, 255), Color3.fromRGB(222, 247, 238)), 90)
stroke(card, C.White, 1.35, 0.16)
gloss(card, 14, 0.78, 0.50)

local dot = new("Frame", {Position = UDim2.fromOffset(14, 17), Size = UDim2.fromOffset(10, 10),
    BackgroundColor3 = C.Muted, BorderSizePixel = 0}, card)
corner(dot, 5)
local sTitle = label({Position = UDim2.fromOffset(32, 10), Size = UDim2.new(1, -120, 0, 24), Text = "DESLIGADO",
    Font = Enum.Font.GothamBlack, TextSize = 16, TextColor3 = C.Muted, TextXAlignment = Enum.TextXAlignment.Left}, card)
local sSub = label({Position = UDim2.fromOffset(14, 34), Size = UDim2.new(1, -120, 0, 14), Text = "Toque em ativar para otimizar",
    TextSize = 11, TextColor3 = C.Muted, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd}, card)
local pctLabel = label({AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -14, 0, 8), Size = UDim2.fromOffset(84, 30),
    Text = "0%", Font = Enum.Font.GothamBlack, TextSize = 26, TextColor3 = C.Muted, TextXAlignment = Enum.TextXAlignment.Right}, card)
local barBg = new("Frame", {Position = UDim2.new(0, 14, 1, -24), Size = UDim2.new(1, -28, 0, 8),
    BackgroundColor3 = C.CardHi, BorderSizePixel = 0}, card)
corner(barBg, 4)
local barFill = new("Frame", {Size = UDim2.fromScale(0, 1), BackgroundColor3 = C.White, BorderSizePixel = 0}, barBg)
corner(barFill, 4)
local barGrad = grad(barFill, BRAND, 0)

local STATE = {
    -- Ligado/desligado mantem a familia aqua; o verde-folha marca o modo ativo.
    off  = {color = C.Cyan, btn = "ATIVAR BOOST AERO", grad = ColorSequence.new(Color3.fromRGB(56, 207, 226), Color3.fromRGB(20, 132, 190))},
    busy = {color = C.Busy, btn = "AJUSTANDO...", grad = ColorSequence.new(Color3.fromRGB(210, 231, 113), Color3.fromRGB(113, 181, 71))},
    on   = {color = C.On, btn = "DESATIVAR BOOST AERO", grad = ColorSequence.new(Color3.fromRGB(126, 218, 96), Color3.fromRGB(22, 151, 108))},
}

local toggleShadow = new("Frame", {Position = UDim2.fromOffset(10, 99), Size = UDim2.new(1, -20, 0, 54),
    BackgroundColor3 = Color3.fromRGB(18, 94, 105), BackgroundTransparency = 0.64, BorderSizePixel = 0}, pBoost)
corner(toggleShadow, 14)
local toggle = new("TextButton", {Position = UDim2.fromOffset(10, 94), Size = UDim2.new(1, -20, 0, 54),
    BackgroundColor3 = C.White, Text = "", AutoButtonColor = false, BorderSizePixel = 0}, pBoost)
corner(toggle, 14)
local toggleGrad = grad(toggle, STATE.off.grad, 90)
local toggleStroke = stroke(toggle, C.White, 1.8, 0.08)
local toggleScale = new("UIScale", {Scale = 1}, toggle)
local toggleHover = new("Frame", {Size = UDim2.fromScale(1, 1), BackgroundColor3 = C.White,
    BackgroundTransparency = 1, BorderSizePixel = 0, Active = false}, toggle)
corner(toggleHover, 14)
gloss(toggle, 14, 0.52, 0.54)
local toggleText = label({Size = UDim2.fromScale(1, 1), Text = STATE.off.btn, Font = Enum.Font.GothamBold,
    TextSize = 14, TextColor3 = C.White, TextStrokeColor3 = Color3.fromRGB(13, 81, 93), TextStrokeTransparency = 0.72,
    TextXAlignment = Enum.TextXAlignment.Center, TextYAlignment = Enum.TextYAlignment.Center,
    TextWrapped = false, TextScaled = false, TextTruncate = Enum.TextTruncate.AtEnd}, toggle)
elevate(toggle, toggleScale, toggleShadow)

track(toggle.MouseEnter:Connect(function() tween(toggleHover, 0.15, {BackgroundTransparency = 0.88}) end))
track(toggle.MouseLeave:Connect(function() tween(toggleHover, 0.15, {BackgroundTransparency = 1}) end))

local statsRow = new("Frame", {Position = UDim2.fromOffset(0, 158), Size = UDim2.new(1, 0, 0, 62), BackgroundTransparency = 1}, pBoost)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 8),
    SortOrder = Enum.SortOrder.LayoutOrder}, statsRow)

local function statCard(name, unit, order)
    local f = new("Frame", {Size = UDim2.new(0.2, -7, 1, 0), BackgroundColor3 = C.Card, BorderSizePixel = 0, LayoutOrder = order}, statsRow)
    corner(f, 12)
    grad(f, ColorSequence.new(Color3.fromRGB(255, 255, 255), Color3.fromRGB(214, 242, 235)), 90)
    stroke(f, C.White, 1.15, 0.20)
    gloss(f, 12, 0.82, 0.42)
    label({Position = UDim2.fromOffset(0, 8), Size = UDim2.new(1, 0, 0, 12), Text = name,
        Font = Enum.Font.GothamBold, TextSize = 10, TextColor3 = C.Muted}, f)
    local v = label({Position = UDim2.fromOffset(0, 21), Size = UDim2.new(1, 0, 0, 22), Text = "--",
        Font = Enum.Font.GothamBlack, TextSize = 17}, f)
    label({Position = UDim2.fromOffset(0, 44), Size = UDim2.new(1, 0, 0, 12), Text = unit,
        TextSize = 9, TextColor3 = C.Muted}, f)
    return v
end
local fpsVal  = statCard("FPS", "quadros/s", 1)
local msVal   = statCard("Frame", "ms", 2)
local pingVal = statCard("Ping", "ms rede", 3)
local memVal  = statCard("Memória", "MB", 4)
local objVal  = statCard("Objetos", "otimizados", 5)

-- botoes LIMPAR RAM e OTIMIZAR PING (a logica fica mais abaixo)
local RAM  = {busy = false, acc = 0}
local PING = {busy = false}
do
    local function bigBtn(posX, offX, text, colA, colB)
        local pos = UDim2.new(posX, offX, 0, 230)
        local size = UDim2.new(0.5, -14, 0, 40)
        local shadow = new("Frame", {Position = UDim2.new(posX, offX, 0, 235), Size = size,
            BackgroundColor3 = Color3.fromRGB(18, 87, 101), BackgroundTransparency = 0.64,
            BorderSizePixel = 0}, pBoost)
        corner(shadow, 12)
        local btn = new("TextButton", {Position = pos, Size = size,
            BackgroundColor3 = C.White, Text = "", AutoButtonColor = false, BorderSizePixel = 0}, pBoost)
        corner(btn, 12)
        grad(btn, ColorSequence.new(colA, colB), 90)
        stroke(btn, C.White, 1.65, 0.12)
        local sc = new("UIScale", {Scale = 1}, btn)
        gloss(btn, 12, 0.48, 0.55)
        local txt = label({Size = UDim2.fromScale(1, 1), Text = text, Font = Enum.Font.GothamBold,
            TextSize = 13, TextColor3 = C.White, TextStrokeColor3 = Color3.fromRGB(9, 78, 91),
            TextStrokeTransparency = 0.72, TextXAlignment = Enum.TextXAlignment.Center,
            TextYAlignment = Enum.TextYAlignment.Center, TextWrapped = false, TextScaled = false,
            TextTruncate = Enum.TextTruncate.AtEnd}, btn)
        elevate(btn, sc, shadow)
        return btn, txt
    end
    RAM.btn,  RAM.text  = bigBtn(0,   10, "LIMPAR RAM",    Color3.fromRGB(41, 166, 207), Color3.fromRGB(19, 129, 181))
    PING.btn, PING.text = bigBtn(0.5, 4, "OTIMIZAR PING", Color3.fromRGB(80, 197, 121), Color3.fromRGB(22, 150, 116))
end

do
    local foot = new("Frame", {Position = UDim2.fromOffset(0, 280), Size = UDim2.new(1, 0, 0, 26),
        BackgroundColor3 = Color3.fromRGB(221, 249, 229), BackgroundTransparency = 0.04, BorderSizePixel = 0}, pBoost)
    corner(foot, 13)
    grad(foot, ColorSequence.new(Color3.fromRGB(249, 255, 241), Color3.fromRGB(201, 239, 211)), 90)
    stroke(foot, C.Leaf, 1.1, 0.22)
    gloss(foot, 13, 0.78, 0.58)
    local footDot = new("Frame", {Position = UDim2.fromOffset(12, 8), Size = UDim2.fromOffset(10, 10),
        BackgroundColor3 = C.On, BorderSizePixel = 0}, foot)
    corner(footDot, 5)
    label({Position = UDim2.fromOffset(30, 0), Size = UDim2.new(1, -38, 1, 0), Text = "🌿 Renderização à distância preservada",
        Font = Enum.Font.GothamBold, TextSize = 11, TextColor3 = C.On, TextXAlignment = Enum.TextXAlignment.Left}, foot)
end

-- ============================================================
-- TOAST
-- ============================================================
local toast
do
    local toastFrame = new("Frame", {AnchorPoint = Vector2.new(0.5, 1), Position = UDim2.new(0.5, 0, 1, 70),
        Size = UDim2.fromOffset(330, 42), BackgroundColor3 = C.Panel, BorderSizePixel = 0}, gui)
    corner(toastFrame, 14)
    grad(toastFrame, ColorSequence.new(Color3.fromRGB(250, 255, 252), Color3.fromRGB(194, 239, 234)), 90)
    local toastStroke = stroke(toastFrame, C.White, 1.7, 0.08)
    gloss(toastFrame, 14, 0.74, 0.48)
    local toastLabel = label({Size = UDim2.fromScale(1, 1), Text = "", Font = Enum.Font.GothamBold, TextSize = 12,
        TextWrapped = true}, toastFrame)
    local toastToken = 0

    toast = function(msg, color)
        toastToken = toastToken + 1
        local mine = toastToken
        toastLabel.Text = msg
        toastStroke.Color = color or C.Cyan
        tween(toastFrame, 0.35, {Position = UDim2.new(0.5, 0, 1, -28)}, Enum.EasingStyle.Back)
        task.delay(2.6, function()
            if mine == toastToken and toastFrame.Parent then
                tween(toastFrame, 0.3, {Position = UDim2.new(0.5, 0, 1, 70)})
            end
        end)
    end
end

-- ============================================================
-- PAGINAS COM ROLAGEM: fabrica + switch
-- ============================================================
local onSetting          -- declarado aqui, definido mais abaixo

-- cria uma pagina com rolagem; devolve a pagina, a funcao row(titulo, descricao, altura)
-- e uma funcao que entrega o proximo LayoutOrder
local function scrollPage()
    local sf = new("ScrollingFrame", {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3,
        ScrollBarImageColor3 = C.Blue, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        ScrollingDirection = Enum.ScrollingDirection.Y, Visible = false,
    }, pages)
    new("UIListLayout", {Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder}, sf)
    new("UIPadding", {PaddingRight = UDim.new(0, 8)}, sf)

    local order = 0
    local function nextOrder()
        order = order + 1
        return order
    end
    local function row(title, desc, h)
        local f = new("Frame", {Size = UDim2.new(1, 0, 0, h or 46), BackgroundColor3 = C.Card,
            BackgroundTransparency = 0.06, BorderSizePixel = 0, LayoutOrder = nextOrder()}, sf)
        corner(f, 12)
        grad(f, ColorSequence.new(Color3.fromRGB(255, 255, 255), Color3.fromRGB(219, 245, 236)), 90)
        stroke(f, C.White, 1.15, 0.20)
        gloss(f, 12, 0.84, 0.43)
        label({Position = UDim2.fromOffset(12, 7), Size = UDim2.new(1, -76, 0, 16), Text = title,
            Font = Enum.Font.GothamBold, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left}, f)
        if desc then
            label({Position = UDim2.fromOffset(12, 24), Size = UDim2.new(1, -76, 0, 14), Text = desc,
                TextSize = 10, TextColor3 = C.Muted, TextXAlignment = Enum.TextXAlignment.Left,
                TextTruncate = Enum.TextTruncate.AtEnd}, f)
        end
        return f
    end
    return sf, row, nextOrder
end

local function addSwitch(frame, key, needsReapply)
    local sw = new("TextButton", {AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(44, 24), BackgroundColor3 = C.CardHi, Text = "", AutoButtonColor = false,
        BorderSizePixel = 0}, frame)
    corner(sw, 12)
    stroke(sw, C.White, 1, 0.28)
    gloss(sw, 12, 0.72, 0.58)
    local knobShadow = new("Frame", {Position = UDim2.fromOffset(4, 4), Size = UDim2.fromOffset(18, 18),
        BackgroundColor3 = Color3.fromRGB(37, 104, 113), BackgroundTransparency = 0.72,
        BorderSizePixel = 0}, sw)
    corner(knobShadow, 9)
    local knob = new("Frame", {Position = UDim2.fromOffset(3, 3), Size = UDim2.fromOffset(18, 18),
        BackgroundColor3 = C.White, BorderSizePixel = 0}, sw)
    corner(knob, 9)
    stroke(knob, C.White, 1, 0.12)
    gloss(knob, 9, 0.36, 0.52)
    local function render(animate)
        local on = CFG[key]
        local col = on and C.Leaf or Color3.fromRGB(147, 194, 199)
        local pos = on and UDim2.fromOffset(23, 3) or UDim2.fromOffset(3, 3)
        if animate then
            tween(sw, 0.16, {BackgroundColor3 = col})
            tween(knob, 0.22, {Position = pos}, Enum.EasingStyle.Back)
            tween(knobShadow, 0.22, {Position = on and UDim2.fromOffset(24, 4) or UDim2.fromOffset(4, 4)}, Enum.EasingStyle.Back)
        else
            sw.BackgroundColor3 = col
            knob.Position = pos
            knobShadow.Position = on and UDim2.fromOffset(24, 4) or UDim2.fromOffset(4, 4)
        end
    end
    render(false)
    onTap(sw, function()
        if needsReapply and mode ~= "idle" then
            toast("Aguarde o processo terminar", C.Busy)
            return
        end
        CFG[key] = not CFG[key]
        render(true)
        onSetting(key, needsReapply)
    end)
    return {render = render}
end

-- ============================================================
-- PAGINA: VISUAL  (limite de FPS + cores vivas)
-- ============================================================
local pVis
do
    local sf, vrow, vnext = scrollPage()
    pVis = sf
    local F = EX.fps

    -- slider generico: o.min / o.max / o.step / o.value / o.fmt(v) / o.onChange(v) / o.y
    local function makeSlider(cardFrame, o)
        local valLbl = label({AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -12, 0, 6),
            Size = UDim2.fromOffset(70, 16), Text = "", Font = Enum.Font.GothamBlack, TextSize = 12,
            TextColor3 = C.On, TextXAlignment = Enum.TextXAlignment.Right}, cardFrame)
        local hit = new("TextButton", {Position = UDim2.new(0, 12, 0, o.y), Size = UDim2.new(1, -24, 0, 24),
            BackgroundTransparency = 1, Text = "", AutoButtonColor = false, BorderSizePixel = 0}, cardFrame)
        local rail = new("Frame", {AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.fromScale(0, 0.5),
            Size = UDim2.new(1, 0, 0, 6), BackgroundColor3 = C.CardHi, BorderSizePixel = 0}, hit)
        corner(rail, 3)
        local fill = new("Frame", {Size = UDim2.fromScale(0, 1), BackgroundColor3 = C.White, BorderSizePixel = 0}, rail)
        corner(fill, 3)
        grad(fill, BRAND, 0)
        local knob = new("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0, 0.5),
            Size = UDim2.fromOffset(17, 17), BackgroundColor3 = C.White, BorderSizePixel = 0, ZIndex = 2}, rail)
        corner(knob, 9)
        stroke(knob, C.Leaf, 2, 0.05)
        gloss(knob, 9, 0.30, 0.55)

        local cur = o.value
        local function snap(v)
            v = math.clamp(v, o.min, o.max)
            v = o.min + math.floor((v - o.min) / o.step + 0.5) * o.step
            return math.clamp(math.floor(v * 1000 + 0.5) / 1000, o.min, o.max)
        end
        local function render()
            local p = (cur - o.min) / (o.max - o.min)
            fill.Size = UDim2.fromScale(p, 1)
            knob.Position = UDim2.fromScale(p, 0.5)
            valLbl.Text = o.fmt(cur)
        end

        local S = {}
        function S.set(v, silent)
            v = snap(v)
            local changed = (v ~= cur)
            cur = v
            render()
            if changed and not silent and o.onChange then o.onChange(cur) end
        end
        function S.get() return cur end

        local function fromInput(input)
            local w = hit.AbsoluteSize.X
            if w <= 0 then return end
            local p = math.clamp((input.Position.X - hit.AbsolutePosition.X) / w, 0, 1)
            S.set(o.min + (o.max - o.min) * p)
        end

        local dragging = false
        track(hit.InputBegan:Connect(function(input)
            if isPress(input) then
                dragging = true
                if EX.block then EX.block(input) end      -- slider nao move a janela nem rola a lista
                fromInput(input)
            end
        end))
        track(UserInputService.InputChanged:Connect(function(input)
            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
                or input.UserInputType == Enum.UserInputType.Touch) then
                fromInput(input)
            end
        end))
        track(UserInputService.InputEnded:Connect(function(input)
            if dragging and isPress(input) then dragging = false end
        end))

        cur = snap(cur)
        render()
        return S
    end

    ---------------- LIMITE DE FPS ----------------
    local hasCap = EX.setCap ~= nil
    local fpsCard = vrow("Limite de FPS",
        hasCap and "Trave ou destrave os quadros por segundo" or "Seu executor não suporta setfpscap", 108)

    local function fpsText(v) return v >= F.max and "Máx" or tostring(v) end

    local chips = {}
    local function refreshChips()
        for i = 1, #chips do
            local ch = chips[i]
            local on = (F.cur == ch.v)
            if ch.on ~= on then
                ch.on = on
                tween(ch.btn, 0.15, {BackgroundColor3 = on and C.Blue or C.Panel,
                    TextColor3 = on and C.White or C.Muted})
            end
        end
    end

    local warned = false
    local function applyAndNotice(v)
        if not EX.applyFps(v) and not warned then
            warned = true
            toast("Seu executor não suporta limite de FPS", C.Busy)
        end
        refreshChips()
    end

    local fpsSlider = makeSlider(fpsCard, {y = 44, min = F.min, max = F.max, step = F.step,
        value = F.cur, fmt = fpsText, onChange = applyAndNotice})

    local chipRow = new("Frame", {Position = UDim2.fromOffset(12, 74), Size = UDim2.new(1, -24, 0, 24),
        BackgroundTransparency = 1}, fpsCard)
    new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6),
        SortOrder = Enum.SortOrder.LayoutOrder}, chipRow)
    for i, v in ipairs({30, 60, 90, 120, F.max}) do
        local on = (F.cur == v)
        local b = new("TextButton", {Size = UDim2.new(0.2, -5, 1, 0), LayoutOrder = i,
            BackgroundColor3 = on and C.Blue or C.Panel, Text = fpsText(v), Font = Enum.Font.GothamBold,
            TextSize = 11, TextColor3 = on and C.White or C.Muted, AutoButtonColor = false, BorderSizePixel = 0}, chipRow)
        corner(b, 8)
        stroke(b, C.White, 1, 0.20)
        gloss(b, 8, 0.64, 0.52)
        local chipScale = new("UIScale", {Scale = 1}, b)
        elevate(b, chipScale)
        chips[i] = {btn = b, v = v, on = on}
        onTap(b, function()
            fpsSlider.set(v, true)
            applyAndNotice(v)
            if hasCap then
                toast(v >= F.max and "FPS destravado (Máx)" or ("FPS limitado em " .. v), C.On)
            end
        end)
    end

    ---------------- CORES VIVAS ----------------
    local vibSw = addSwitch(vrow("Filtro de cores vivas", "Cores mais fortes e nítidas"), "Vibrant", false)

    local COLOR_DEF = {ColorSat = 0.65, ColorCon = 0.15, ColorBri = 0.05}
    local colorSliders = {}
    local function colorCard(title, key, min, max, step)
        local c = vrow(title, nil, 58)
        colorSliders[key] = makeSlider(c, {y = 28, min = min, max = max, step = step, value = CFG[key],
            fmt = function(v) return string.format("%+.2f", v) end,
            onChange = function(v)
                CFG[key] = v
                if not CFG.Vibrant then          -- mexeu no slider: liga o filtro para ver o efeito
                    CFG.Vibrant = true
                    vibSw.render(true)
                end
                EX.applyVibrant()
            end})
    end
    colorCard("Saturação", "ColorSat", -1, 1, 0.05)
    colorCard("Contraste", "ColorCon", -0.5, 0.5, 0.05)
    colorCard("Brilho",    "ColorBri", -0.3, 0.3, 0.05)

    local resetWrap = new("Frame", {Size = UDim2.new(1, 0, 0, 36), BackgroundTransparency = 1,
        BorderSizePixel = 0, LayoutOrder = vnext()}, sf)
    local reset = new("TextButton", {AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 0),
        Size = UDim2.new(1, -20, 1, 0), BackgroundColor3 = C.White,
        Text = "Restaurar cores padrão", Font = Enum.Font.GothamBold, TextSize = 12, TextColor3 = C.White,
        TextXAlignment = Enum.TextXAlignment.Center, TextYAlignment = Enum.TextYAlignment.Center,
        TextWrapped = false, TextScaled = false, TextStrokeColor3 = Color3.fromRGB(13, 81, 93),
        TextStrokeTransparency = 0.72, AutoButtonColor = false, BorderSizePixel = 0}, resetWrap)
    corner(reset, 12)
    grad(reset, ColorSequence.new(C.Blue, C.Mint), 90)
    stroke(reset, C.White, 1.3, 0.12)
    gloss(reset, 12, 0.55, 0.52)
    local resetScale = new("UIScale", {Scale = 1}, reset)
    elevate(reset, resetScale)
    onTap(reset, function()
        for key, v in pairs(COLOR_DEF) do
            CFG[key] = v
            colorSliders[key].set(v, true)
        end
        EX.applyVibrant()
        toast("Cores restauradas ao padrão", C.Muted)
    end)
end

-- ============================================================
-- PAGINA: AJUSTES
-- ============================================================
local pSet, row = scrollPage()

local function switchRow(title, desc, key, needsReapply)
    addSwitch(row(title, desc), key, needsReapply)
end

switchRow("Efeitos de ataque fracos", "Partículas, trails e beams mais leves", "WeakEffects", true)

-- seletor de intensidade
do
    local lvlRow = row("Intensidade dos efeitos", "Suave: quase igual  •  Mínimo: quase invisível", 66)
    local seg = new("Frame", {Position = UDim2.fromOffset(12, 38), Size = UDim2.new(1, -24, 0, 22),
        BackgroundColor3 = C.Panel, BorderSizePixel = 0}, lvlRow)
    corner(seg, 8)
    grad(seg, ColorSequence.new(Color3.fromRGB(222, 247, 240), Color3.fromRGB(167, 220, 213)), 90)
    stroke(seg, C.White, 1, 0.22)
    local segInd = new("Frame", {Size = UDim2.new(1 / 3, -4, 1, -4), BackgroundColor3 = C.White, BorderSizePixel = 0}, seg)
    corner(segInd, 6)
    grad(segInd, BRAND, 0)
    local segBtns = {}
    local function renderSeg(animate)
        for i, b in ipairs(segBtns) do
            b.TextColor3 = (i == CFG.Level) and C.White or C.Muted
        end
        local pos = UDim2.new((CFG.Level - 1) / 3, 2, 0, 2)
        if animate then tween(segInd, 0.18, {Position = pos}) else segInd.Position = pos end
    end
    for i, L in ipairs(LEVELS) do
        segBtns[i] = new("TextButton", {Position = UDim2.new((i - 1) / 3, 0, 0, 0), Size = UDim2.new(1 / 3, 0, 1, 0),
            BackgroundTransparency = 1, Text = L.name, Font = Enum.Font.GothamBold, TextSize = 11,
            TextColor3 = C.Muted, AutoButtonColor = false}, seg)
        onTap(segBtns[i], function()
            if mode ~= "idle" then toast("Aguarde o processo terminar", C.Busy) return end
            CFG.Level = i
            renderSeg(true)
            onSetting("Level", true)
        end)
    end
    renderSeg(false)
end

switchRow("Remover texturas", "Materiais lisos, sem decals e texturas", "Textures", true)
switchRow("Formas sem deformar", "Casas, árvores e pilares mantêm a forma de longe", "KeepShape", true)
switchRow("Remover atmosfera e bloom", "Atmosphere, Bloom, Blur, DoF, SunRays", "Atmos", true)
switchRow("Luz e sombras leves", "Sem sombras globais, neblina e reflexos", "Lighting", true)
switchRow("Proteger jogadores", "Corpo, acessórios e ferramentas intactos", "ProtectPlayers", true)
switchRow("Efeitos presos ao personagem", "Auras e formas extras entram na otimização", "CharFx", true)
switchRow("Proteger NPCs", "Inimigos e NPCs mantêm a aparência", "ProtectNPCs", true)
switchRow("Física leve (avançado)", "Pode afetar cliques e toques locais", "Physics", true)
switchRow("HUD de FPS na tela", "Contador flutuante e arrastável", "ShowHud", false)
switchRow("Limpeza profunda de RAM", "Troca a qualidade por ~1s e devolve", "DeepRam", false)
switchRow("Limpar RAM automático", "Limpeza leve a cada 60s, sem engasgo", "AutoRam", false)

-- ============================================================
-- PAGINA: INFO
-- ============================================================
local pInfo
do
    local INFO_TEXT = table.concat({
        '<font color="#168BCC"><b>O que é otimizado</b></font>',
        "•  Texturas, decals e materiais ficam lisos e leves",
        "•  Efeitos de ataque: Suave (leve), Médio (meio-termo), Mínimo (quase invisível)",
        "•  Atmosfera, Bloom, Blur, DoF e SunRays são removidos",
        "•  Sombras globais, neblina e reflexos são desligados",
        "•  Texturas de efeitos novos são removidas antes de carregar",
        "",
        '<font color="#168BCC"><b>Visual</b></font>',
        "•  Limite de FPS: 30, 60, 90, 120 ou Máx (destravado)",
        "•  Cores vivas: saturação, contraste e brilho ajustáveis",
        "•  O limite de FPS depende do executor (setfpscap)",
        "•  Ao fechar o menu o FPS e as cores voltam ao normal",
        "",
        '<font color="#589C3D"><b>RAM e Ping</b></font>',
        "•  Limpar RAM: coleta completa do lixo do Lua + limpeza profunda",
        "•  Limpeza profunda: a qualidade gráfica cai por ~1s e volta sozinha",
        "•  O valor liberado mostrado é medido de verdade e varia por jogo",
        "•  Otimizar Ping: mede, ajusta o cliente e mede de novo",
        "•  O ping real depende da sua internet e do servidor do jogo",
        "•  Ao fechar o menu os ajustes de rede voltam ao original",
        "",
        '<font color="#278A50"><b>O que não é tocado</b></font>',
        "•  Distância de renderização e streaming",
        "•  Forma dos objetos de longe (casas, árvores, pilares)",
        "•  Qualidade gráfica do Roblox (só na limpeza profunda, por ~1s)",
        "•  Corpo, acessórios e ferramentas dos jogadores",
        "•  Aparência dos NPCs",
        "",
        '<font color="#168BCC"><b>Dicas</b></font>',
        "•  RightShift mostra ou oculta o menu",
        "•  Arraste o menu segurando em qualquer parte dele",
        "•  Abas com rolagem no celular: segure 0,3s parado e arraste",
        "•  Limpe a RAM parado, fora de combate (pode engasgar por um instante)",
        "•  Mudou um ajuste com o boost ligado? Ele reaplica sozinho",
        "",
        '<font color="#168BCC"><b>Dúvidas, bugs e sugestões</b></font>',
        "•  Ficou com alguma dúvida, encontrou um bug ou tem uma sugestão?",
        "•  Me chame no Instagram: @julio_kkz7",
        "•  O Painel Lee foi desenvolvido por Julio Cesar.",
        "•  Seu feedback ajuda a melhorar o painel!",
    }, "\n")

    pInfo = new("ScrollingFrame", {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3,
        ScrollBarImageColor3 = C.Blue, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        ScrollingDirection = Enum.ScrollingDirection.Y, Visible = false,
    }, pages)
    local INFO_UI = {}
    INFO_UI.body = label({Size = UDim2.new(1, -10, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
        RichText = true, TextWrapped = true, TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top, TextSize = 12, LineHeight = 1.2,
        Text = INFO_TEXT}, pInfo)

end

-- ============================================================
-- ABAS
-- ============================================================
local pageList = {pBoost, pVis, pSet, pInfo}
local function selectTab(i)
    for k, pg in ipairs(pageList) do pg.Visible = (k == i) end
    for k, b in ipairs(tabBtns) do
        tween(b, 0.15, {TextColor3 = (k == i) and C.White or C.Muted})
    end
    tween(indicator, 0.24, {Position = UDim2.new((i - 1) / #TAB_NAMES, 3, 0, 3)}, Enum.EasingStyle.Quint)
    local pg = pageList[i]
    pg.Position = UDim2.fromOffset(16, 0)
    tween(pg, 0.22, {Position = UDim2.fromOffset(0, 0)})
end
for i, b in ipairs(tabBtns) do
    onTap(b, function() selectTab(i) end)
end
tabBtns[1].TextColor3 = C.White

-- ============================================================
-- MINI BOTAO + HUD DE FPS
-- ============================================================
local mini = new("TextButton", {AnchorPoint = Vector2.new(0.5, 0.5), Position = root.Position,
    Size = UDim2.fromOffset(96, 38), BackgroundColor3 = C.White, Text = "", TextColor3 = C.White,
    Font = Enum.Font.GothamBlack, TextSize = 15, AutoButtonColor = false, Visible = false, BorderSizePixel = 0}, gui)
corner(mini, 19)
grad(mini, ColorSequence.new(C.Blue, C.Mint), 90)
stroke(mini, C.White, 1.8, 0.06)
gloss(mini, 19, 0.46, 0.56)
local miniText = label({Size = UDim2.fromScale(1, 1), Text = "-- FPS", Font = Enum.Font.GothamBlack,
    TextSize = 13, TextColor3 = C.White, TextStrokeColor3 = Color3.fromRGB(13, 81, 93), TextStrokeTransparency = 0.62}, mini)
local miniScale = new("UIScale", {Scale = 1}, mini)
elevate(mini, miniScale)

local hud = new("Frame", {AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 14),
    Size = UDim2.fromOffset(236, 30), BackgroundColor3 = C.Panel, BackgroundTransparency = 0.10,
    BorderSizePixel = 0, Active = true, Visible = false}, gui)
corner(hud, 15)
grad(hud, ColorSequence.new(Color3.fromRGB(249, 255, 250), Color3.fromRGB(180, 229, 218)), 90)
grad(stroke(hud, C.White, 1.8, 0.05), BRAND, 0)
gloss(hud, 15, 0.72, 0.55)
local hudDot = new("Frame", {Position = UDim2.fromOffset(13, 10), Size = UDim2.fromOffset(10, 10),
    BackgroundColor3 = C.On, BorderSizePixel = 0}, hud)
corner(hudDot, 5)
local hudText = label({Position = UDim2.fromOffset(32, 0), Size = UDim2.new(1, -40, 1, 0), Text = "-- FPS",
    Font = Enum.Font.GothamBlack, TextSize = 13, TextColor3 = C.Text, TextXAlignment = Enum.TextXAlignment.Left, RichText = true}, hud)

-- ============================================================
-- ARRASTAR (mini botao e HUD) / AUTO-ESCALA / MENU
-- ============================================================
local function makeDrag(handle, target, padX, padY)
    local st = {moved = false}
    local dragging, startIn, startPos = false, nil, nil
    track(handle.InputBegan:Connect(function(input)
        if isPress(input) then
            dragging, st.moved = true, false
            startIn, startPos = input.Position, target.Position
            local c
            c = input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    c:Disconnect()
                end
            end)
        end
    end))
    track(UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - startIn
            if d.Magnitude > 5 then st.moved = true end
            target.Position = clampPos(target, UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y), padX, padY)
        end
    end))
    return st
end
local miniDrag = makeDrag(mini, mini, 60, 30)
makeDrag(hud, hud, 80, 24)

-- ARRASTAR O MENU INTEIRO: segura em qualquer parte e move
--  * toque/mouse em area livre, cards, textos, botoes ou abas: arrasta apos ~8px
--  * sliders: nao movem a janela (so ajustam o valor)
--  * abas com rolagem + toque: segurar ~0,3s parado e arrastar (senao o dedo rola a lista)
do
    local DRAG_PX, HOLD_T = 8, 0.3
    local sess
    local scrollPages = {pVis, pSet, pInfo}

    local function setScroll(on)
        for i = 1, #scrollPages do scrollPages[i].ScrollingEnabled = on end
    end

    local function endSession(s)
        if sess ~= s then return end
        sess = nil
        if s.conn then s.conn:Disconnect() end
        if s.scrollOff then setScroll(true) end
        if s.holding then tween(popScale, 0.15, {Scale = 1}) end
    end

    local function beginSession(input)
        local s = sess
        if s then
            if os.clock() - s.t < 0.1 then return s end     -- mesmo toque (varios handlers disparam juntos)
            endSession(s)                                    -- sessao velha que ficou presa
        end
        EX.tapBlocked = false
        s = {
            input = input, t = os.clock(), startIn = input.Position, startPos = root.Position,
            touch = (input.UserInputType == Enum.UserInputType.Touch),
            state = 0,                     -- 0 pendente, 1 arrastando, 2 cancelado (era rolagem/slider)
            inScroll = false, blocked = false, holding = false, scrollOff = false,
        }
        sess = s
        s.conn = input.Changed:Connect(function()
            local st = input.UserInputState
            if st == Enum.UserInputState.End or st == Enum.UserInputState.Cancel then
                endSession(s)
            end
        end)
        task.delay(HOLD_T, function()
            if sess ~= s or s.state ~= 0 or s.blocked or not (s.touch and s.inScroll) then return end
            s.holding = true
            s.scrollOff = true
            setScroll(false)
            tween(popScale, 0.12, {Scale = 1.03})            -- feedback: pronto para arrastar
        end)
        return s
    end

    -- chamado pelos sliders ao serem tocados
    EX.block = function(input)
        local s = beginSession(input)
        s.blocked = true
        if not s.scrollOff then
            s.scrollOff = true
            setScroll(false)
        end
    end

    local function bindDrag(obj)
        local scrolly = false
        for i = 1, #scrollPages do
            local sf = scrollPages[i]
            if obj == sf or obj:IsDescendantOf(sf) then scrolly = true break end
        end
        track(obj.InputBegan:Connect(function(input)
            if not isPress(input) then return end
            local s = beginSession(input)
            if scrolly then s.inScroll = true end
        end))
    end

    bindDrag(main)
    for _, d in ipairs(main:GetDescendants()) do
        if d:IsA("GuiObject") then bindDrag(d) end
    end

    track(UserInputService.InputChanged:Connect(function(input)
        local s = sess
        if not s or s.state == 2 then return end
        local t = input.UserInputType
        if s.touch then
            if t ~= Enum.UserInputType.Touch then return end
        elseif t ~= Enum.UserInputType.MouseMovement then
            return
        end
        local d = input.Position - s.startIn
        if s.state == 0 then
            if d.Magnitude < DRAG_PX then return end
            if s.blocked or (s.touch and s.inScroll and not s.holding) then
                s.state = 2
                return
            end
            s.state = 1
            EX.tapBlocked = true
        end
        root.Position = clampPos(root, UDim2.new(s.startPos.X.Scale, s.startPos.X.Offset + d.X,
            s.startPos.Y.Scale, s.startPos.Y.Offset + d.Y), 130, 90)
    end))

    track(UserInputService.InputEnded:Connect(function(input)
        local s = sess
        if s and isPress(input) and (input == s.input or input.UserInputType == s.input.UserInputType) then
            endSession(s)
        end
    end))
end

local function fit()
    local vp = gui.AbsoluteSize
    if vp.X < 10 or vp.Y < 10 then return end
    autoScale.Scale = math.clamp(math.min(vp.X / 520, vp.Y / 470), 0.62, 1.1)
    task.defer(function()
        if root.Parent then root.Position = clampPos(root, root.Position, 130, 90) end
    end)
end
track(gui:GetPropertyChangedSignal("AbsoluteSize"):Connect(fit))

local menuState = "hidden"      -- open | mini | hidden
local openAudioToken = 0
local function openMenu()
    menuState = "open"
    openAudioToken = openAudioToken + 1
    local thisOpen = openAudioToken
    root.Position = clampPos(root, mini.Position, 130, 90)
    mini.Visible = false
    root.GroupTransparency = 1
    popScale.Scale = 0.88
    root.Visible = true
    setGlassBlur(true)

    -- Primeiro toca o som de abertura junto com o fade-in; depois entra a música em loop.
    UIAudio.stop("intro", 0)
    UIAudio.stop("music", 0)
    if MUSIC_UI.enabled then UIAudio.play("intro") end
    tween(root, 0.34, {GroupTransparency = 0}, Enum.EasingStyle.Quint)
    tween(popScale, 0.40, {Scale = 1}, Enum.EasingStyle.Back)
    task.delay(0.42, function()
        if openAudioToken ~= thisOpen then return end
        if menuState == "open" and MUSIC_UI.enabled and not shuttingDown then
            UIAudio.stop("intro", 0.12)
            UIAudio.play("music")
        end
    end)
end
local function miniMenu()
    menuState = "mini"
    openAudioToken = openAudioToken + 1
    UIAudio.stop("intro", 0.12)
    UIAudio.stop("music", 0.24)
    mini.Position = clampPos(mini, root.Position, 60, 30)
    setGlassBlur(false)
    tween(root, 0.16, {GroupTransparency = 1}, Enum.EasingStyle.Quad)
    tween(popScale, 0.18, {Scale = 0.88})
    task.delay(0.19, function()
        if menuState == "mini" then
            root.Visible = false
            mini.Visible = true
            miniScale.Scale = 0.88
            tween(miniScale, 0.20, {Scale = 1}, Enum.EasingStyle.Back)
        end
    end)
end
local function hideMenu()
    menuState = "hidden"
    openAudioToken = openAudioToken + 1
    UIAudio.stop("intro", 0.12)
    UIAudio.stop("music", 0.24)
    setGlassBlur(false)
    mini.Visible = false
    if root.Visible then
        tween(root, 0.18, {GroupTransparency = 1}, Enum.EasingStyle.Quad)
        tween(popScale, 0.18, {Scale = 0.92}, Enum.EasingStyle.Quad)
        task.delay(0.20, function()
            if menuState == "hidden" and root.Parent then root.Visible = false end
        end)
    else
        root.Visible = false
    end
end

onTap(minBtn, miniMenu)
onTap(MUSIC_UI.button, function()
    MUSIC_UI.enabled = not MUSIC_UI.enabled
    openAudioToken = openAudioToken + 1 -- invalida qualquer início de música que ainda esteja aguardando
    MUSIC_UI.render()
    if MUSIC_UI.enabled then
        if menuState == "open" then UIAudio.play("music") end
        toast("Música ativada", C.On)
    else
        UIAudio.stop("intro", 0.12)
        UIAudio.stop("music", 0.24)
        toast("Música desativada", C.Muted)
    end
end)
track(mini.Activated:Connect(function()
    if miniDrag.moved then return end
    openMenu()
end))
track(UserInputService.InputBegan:Connect(function(input, processed)
    if processed or shuttingDown then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        if menuState == "hidden" then openMenu() else hideMenu() end
    end
end))

-- ============================================================
-- ESTADO VISUAL / PROGRESSO
-- ============================================================
local setUIState
do
    local pulse
    local function setPulse(on)
        if pulse then pulse:Cancel(); pulse = nil end
        dot.BackgroundTransparency = 0
        if on then
            pulse = TweenService:Create(dot, TweenInfo.new(0.55, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
                {BackgroundTransparency = 0.65})
            pulse:Play()
        end
    end

    setUIState = function(state, title, sub)
        local S = STATE[state]
        sTitle.Text = title
        sSub.Text = sub
        sTitle.TextColor3 = S.color
        pctLabel.TextColor3 = S.color
        tween(dot, 0.2, {BackgroundColor3 = S.color})
        toggleText.Text = S.btn
        toggleGrad.Color = S.grad
        toggleStroke.Color = S.color
        barGrad.Color = (state == "on") and S.grad or BRAND
        setPulse(state == "busy")
    end
end

local setProgress
do
    local lastUI, lastPct = 0, -1
    setProgress = function(p, force)
        local now = os.clock()
        if not force and now - lastUI < 0.08 then return end
        lastUI = now
        local pct = math.floor(p * 100)
        if pct ~= lastPct or force then
            lastPct = pct
            pctLabel.Text = pct .. "%"
            tween(barFill, 0.1, {Size = UDim2.new(p, 0, 1, 0)}, Enum.EasingStyle.Linear)
        end
    end
end

-- ============================================================
-- PROCESSADOR POR FRAME
-- ============================================================
local enable, disable
local hardCleanup

local function stopWorker()
    if workerConn then
        workerConn:Disconnect()
        workerConn = nil
    end
end

local function finalize()
    stopWorker()
    disconnectBoost()
    pcall(UIAudio.stop, "intro", 0)
    pcall(UIAudio.stop, "music", 0)
    pcall(UIAudio.cleanup)
    pcall(EX.cleanup)
    for i = 1, #permConns do
        pcall(permConns[i].Disconnect, permConns[i])
    end
    table.clear(permConns)
    pcall(gui.Destroy, gui)
    if env.LeeFixLagCleanup == hardCleanup then env.LeeFixLagCleanup = nil end
end

local function step(dt)
    EX.instantSpent = 0                                -- novo quadro: zera o orcamento dos handlers imediatos
    local t0 = os.clock()
    local budget = CFG.BudgetMs / 1000
    if dt > 0.034 then budget = budget * 0.4 end       -- FPS caindo: o script recua

    if mode == "restore" then
        if restoreChunk(t0, budget) then
            cacheCount = 0
            table.clear(kindMemo)
            mode = "idle"
            stopWorker()
            setProgress(0, true)
            if shuttingDown then finalize() return end
            if pendingReapply then
                pendingReapply = false
                enable()
                return
            end
            setUIState("off", "DESLIGADO", "Toque em ativar para otimizar")
            toast("Boost desativado. Tudo restaurado.", C.Muted)
        else
            setProgress(math.min(rDone / rTotal, 0.99))
        end
        return
    end

    if not drainPending(t0, budget) then return end

    if mode == "scan" then
        local cnt = 0
        while sp > 0 do
            local kids = fKids[sp]
            local i = fIdx[sp]
            if i > fN[sp] then
                fKids[sp] = nil
                sp = sp - 1
            else
                fIdx[sp] = i + 1
                local node = kids[i]
                local cw = fW[sp]
                local f = HOPT[node.ClassName]
                if f then pcall(f, node) end
                local ch = node:GetChildren()
                local n = #ch
                if n == 0 then
                    doneW = doneW + cw
                else
                    sp = sp + 1
                    fKids[sp] = ch
                    fIdx[sp] = 1
                    fN[sp] = n
                    fW[sp] = cw / n
                end
                cnt = cnt + 1
                if cnt % 24 == 0 and os.clock() - t0 >= budget then break end
            end
        end
        if sp == 0 then
            mode = "idle"
            setProgress(1, true)
            setUIState("on", "BOOST ATIVO", "Jogo otimizado, distância preservada")
            toast("Boost ativo! Renderização à distância preservada.", C.On)
        else
            setProgress(math.min(doneW, 0.99))
        end
    end
end

local function startWorker()
    if workerConn then return end
    workerConn = RunService.Heartbeat:Connect(step)
end

-- ============================================================
-- LIGAR / DESLIGAR
-- ============================================================
enable = function()
    if mode ~= "idle" then return end
    enabled = true
    mode = "scan"
    setUIState("busy", "OTIMIZANDO", "Aplicando sem travar o jogo")
    setProgress(0, true)
    applyGlobalSettings()

    local ok, list = pcall(Lighting.GetDescendants, Lighting)
    if ok then
        for i = 1, #list do process(list[i]) end
    end

    pend, pHead, pTail = {}, 1, 0
    boostConns.ws = Workspace.DescendantAdded:Connect(onAdded)
    boostConns.light = Lighting.DescendantAdded:Connect(onAdded)

    fKids, fIdx, fN, fW = {{Workspace}}, {1}, {1}, {1}
    sp, doneW = 1, 0
    startWorker()
end

disable = function()
    if mode == "restore" then return end
    enabled = false
    mode = "restore"
    setUIState("busy", "RESTAURANDO", "Devolvendo os valores originais")
    disconnectBoost()
    restoreGlobalSettings()
    pend, pHead, pTail = {}, 1, 0
    fKids, fIdx, fN, fW, sp = {}, {}, {}, {}, 0
    rDone, rTotal = 0, math.max(1, cacheCount)
    setProgress(0, true)
    startWorker()
end

onSetting = function(key, needsReapply)
    if key == "ShowHud" then
        hud.Visible = CFG.ShowHud
        return
    end
    if key == "Vibrant" then
        EX.applyVibrant()
        return
    end
    if needsReapply and enabled and mode == "idle" then
        pendingReapply = true
        toast("Reaplicando ajustes...", C.Busy)
        disable()
    end
end

onTap(toggle, function()
    if shuttingDown or mode ~= "idle" then return end
    if enabled then disable() else enable() end
end)

onTap(closeBtn, function()
    if shuttingDown then return end
    shuttingDown = true
    hideMenu()
    hud.Visible = false
    if enabled or mode ~= "idle" or next(cache) ~= nil or #detached > 0 then
        disable()
    else
        finalize()
    end
end)

-- ============================================================
-- LIMPAR RAM
-- O Roblox nao deixa um script esvaziar a RAM do aparelho inteiro, mas da para:
--   1) coletar TODO o lixo do heap Lua (inclui o lixo dos scripts do jogo)
--   2) limpeza profunda: baixar a qualidade grafica por ~1s para o Roblox soltar o
--      cache de texturas/malhas, e devolver a qualidade original em seguida
--   3) esvaziar os caches internos do proprio script
-- O valor mostrado e a diferenca REAL medida em Stats (varia de jogo para jogo).
-- ============================================================
RAM.mb = function()
    local ok, v = pcall(Stats.GetTotalMemoryUsageMb, Stats)
    return (ok and type(v) == "number") and v or 0
end

RAM.kb = function()
    local ok, v = pcall(collectgarbage, "count")
    return (ok and type(v) == "number") and v or 0
end

-- coleta incremental: 1 passo por quadro, sem engasgar (usada no modo automatico e no ping)
RAM.gcSteps = function()
    for _ = 1, 120 do
        local ok, done = pcall(collectgarbage, "step", 400)
        if not ok then return false end
        if done then return true end
        RunService.Heartbeat:Wait()
    end
    return false
end

RAM.clean = function(silent)
    if RAM.busy or shuttingDown then return end
    if silent and mode ~= "idle" then return end
    RAM.busy = true

    task.spawn(function()
        pcall(function()
            local beforeMb, beforeKb = RAM.mb(), RAM.kb()
            table.clear(kindMemo)                      -- cache do script (refeito sob demanda)

            if silent then                             -- automatico: leve, nunca engasga
                RAM.gcSteps()
                return
            end

            RAM.text.Text = "LIMPANDO..."

            -- 1) coleta completa do heap Lua (pode pesar por um instante)
            pcall(collectgarbage, "collect")
            RunService.Heartbeat:Wait()

            -- 2) limpeza profunda: o cache de texturas/malhas do Roblox e liberado
            --    quando a qualidade cai; a original volta logo depois
            if CFG.DeepRam then
                local okQ, q0 = pcall(function() return settings().Rendering.QualityLevel end)
                if okQ and q0 ~= nil then
                    RAM.text.Text = "LIMPANDO CACHE..."
                    EX.q0 = q0
                    pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
                    task.wait(0.9)
                    EX.restoreQuality()
                    task.wait(0.4)
                end
            end

            -- 3) coleta final, ja sem as referencias soltas
            pcall(collectgarbage, "collect")
            RunService.Heartbeat:Wait()

            RAM.text.Text = "MEDINDO..."
            task.wait(1.2)                             -- o contador de memoria atualiza com atraso

            local freedMb  = math.max(0, math.floor(beforeMb - RAM.mb() + 0.5))
            local freedLua = math.max(0, (beforeKb - RAM.kb()) / 1024)
            if freedMb >= 1 then
                RAM.text.Text = "-" .. freedMb .. " MB"
                toast(string.format("RAM limpa: %d MB liberados", freedMb), C.On)
            elseif freedLua >= 0.5 then
                RAM.text.Text = string.format("-%.1f MB", freedLua)
                toast(string.format("Memória do Lua limpa: %.1f MB", freedLua), C.On)
            else
                RAM.text.Text = "OK"
                toast("Memória já estava otimizada", C.Muted)
            end
            task.wait(2.2)
        end)

        pcall(EX.restoreQuality)                       -- garantia: a qualidade SEMPRE volta
        pcall(function() RAM.text.Text = "LIMPAR RAM" end)
        RAM.busy = false
    end)
end

onTap(RAM.btn, function() RAM.clean(false) end)

-- ============================================================
-- OTIMIZAR PING
-- Ping = tempo de ida e volta ate o servidor. Isso depende da sua internet e da
-- distancia do servidor, e nenhum script consegue baixar alem disso.
-- O que o botao faz (no cliente):
--   1) mede o ping (media de varias leituras)
--   2) tira lag artificial de replicacao e desliga rastreio/debug de rede
--   3) coleta lixo de forma incremental, evitando picos de quadro que atrasam
--      o envio/recebimento de pacotes (o ping "sente" pior com quadros travando)
--   4) espera estabilizar, mede de novo e mostra "antes -> depois" sem maquiar
-- Os valores de rede originais voltam ao fechar o menu.
-- ============================================================
local NET_PROPS = {
    {"IncomingReplicationLag", 0},
    {"PrintPhysicsErrors", false},
    {"PrintStreamInstanceQuota", false},
    {"RenderStreamedRegions", false},
    {"TrackDataTypes", false},
    {"TrackPhysicsDetails", false},
}

PING.read = function()
    local ok, v = pcall(function()
        local it = PING.item
        if not (it and it.Parent) then
            it = Stats.Network.ServerStatsItem["Data Ping"]
            PING.item = it
        end
        return it:GetValue()
    end)
    if ok and type(v) == "number" and v > 0 then return v end
    local ok2, p = pcall(player.GetNetworkPing, player)      -- alternativa: segundos (ida) -> ms (ida e volta)
    if ok2 and type(p) == "number" and p > 0 then return p * 2000 end
    return nil
end

PING.sample = function(dur)
    local vals, t0 = {}, os.clock()
    repeat
        local p = PING.read()
        if p then vals[#vals + 1] = p end
        task.wait(0.15)
    until os.clock() - t0 >= dur
    if #vals == 0 then return nil end
    local sum = 0
    for i = 1, #vals do sum = sum + vals[i] end
    return sum / #vals
end

PING.tune = function()
    local okS, net = pcall(function() return settings().Network end)
    if not okS or not net then return 0 end
    local changed = 0
    for i = 1, #NET_PROPS do
        local prop, val = NET_PROPS[i][1], NET_PROPS[i][2]
        local okR, cur = pcall(function() return net[prop] end)
        if okR and cur ~= nil and cur ~= val then
            if EX.net[prop] == nil then EX.net[prop] = cur end
            if pcall(function() net[prop] = val end) then changed = changed + 1 end
        end
    end
    return changed
end

PING.run = function()
    if PING.busy or shuttingDown then return end
    PING.busy = true

    task.spawn(function()
        pcall(function()
            PING.text.Text = "MEDINDO..."
            local before = PING.sample(1.2)

            PING.text.Text = "OTIMIZANDO..."
            PING.tune()
            table.clear(kindMemo)
            RAM.gcSteps()                              -- sem engasgo
            task.wait(1.5)                             -- deixa os quadros estabilizarem

            PING.text.Text = "MEDINDO..."
            local after = PING.sample(1.5)

            if before and after then
                local b, a = math.floor(before + 0.5), math.floor(after + 0.5)
                PING.text.Text = b .. " → " .. a .. " ms"
                if before - after >= 3 then
                    toast(string.format("Ping melhorou: %d → %d ms", b, a), C.On)
                else
                    toast(string.format("Ping em %d ms. Ajustes aplicados; o resto depende da sua rede", a), C.Muted)
                end
            else
                PING.text.Text = "OK"
                toast("Ajustes aplicados (seu executor não expõe o ping para medir)", C.Busy)
            end
            task.wait(2.5)
        end)

        pcall(function() PING.text.Text = "OTIMIZAR PING" end)
        PING.busy = false
    end)
end

onTap(PING.btn, function() PING.run() end)

-- ============================================================
-- ESTATISTICAS AO VIVO
-- ============================================================
do
    local function colorFor(fps)
        if fps >= 55 then return C.Leaf end
        if fps >= 30 then return C.Busy end
        return C.Off
    end
    local function colorForPing(ms)
        if ms < 90 then return C.Leaf end
        if ms < 160 then return C.Busy end
        return C.Off
    end

    local acc, frames, animAcc, memAcc = 0, 0, 0, 2
    track(RunService.Heartbeat:Connect(function(dt)
        acc = acc + dt
        frames = frames + 1
        memAcc = memAcc + dt

        if CFG.AutoRam then
            RAM.acc = RAM.acc + dt
            if RAM.acc >= 60 then
                RAM.acc = 0
                RAM.clean(true)
            end
        else
            RAM.acc = 0
        end

        if acc >= 0.5 then
            local fps = math.floor(frames / acc + 0.5)
            local ms = acc / frames * 1000
            acc, frames = 0, 0
            local col = colorFor(fps)

            fpsVal.Text = tostring(fps)
            fpsVal.TextColor3 = col
            msVal.Text = string.format("%.1f", ms)
            objVal.Text = fmtNum(cacheCount)
            miniText.Text = string.format("%d FPS", fps)

            local ping = PING.read()
            if ping then
                pingVal.Text = tostring(math.floor(ping + 0.5))
                pingVal.TextColor3 = colorForPing(ping)
            else
                pingVal.Text = "--"
                pingVal.TextColor3 = C.Muted
            end

            if CFG.ShowHud then
                hudDot.BackgroundColor3 = col
                hudText.Text = string.format('%d <font color="%s">FPS  •  %.1f ms  •  %s</font>', fps, HEX_MUTED, ms,
                    ping and (math.floor(ping + 0.5) .. " ping") or "-- ping")
            end
        end

        if memAcc >= 2 then
            memAcc = 0
            local ok, mb = pcall(Stats.GetTotalMemoryUsageMb, Stats)
            if ok and mb then memVal.Text = string.format("%d", mb) end
        end

        if menuState == "open" then
            animAcc = animAcc + dt
            if animAcc >= 0.05 then
                animAcc = 0
                strokeGrad.Rotation = (os.clock() * 70) % 360
            end
        end
    end))
end

-- ============================================================
-- LIMPEZA (chamada se o script for executado de novo)
-- ============================================================
hardCleanup = function()
    shuttingDown = true
    enabled = false
    pcall(UIAudio.stop, "intro", 0)
    pcall(UIAudio.stop, "music", 0)
    pcall(UIAudio.cleanup)
    disconnectBoost()
    pcall(restoreGlobalSettings)
    restoreChunk(os.clock(), math.huge)
    pcall(EX.cleanup)
    mode = "idle"
    stopWorker()
    for i = 1, #permConns do
        pcall(permConns[i].Disconnect, permConns[i])
    end
    table.clear(permConns)
    pcall(gui.Destroy, gui)
end
env.LeeFixLagCleanup = hardCleanup

-- ============================================================
-- INICIAR
-- ============================================================
do
    local ok = pcall(function() gui.Parent = (gethui and gethui()) or CoreGui end)
    if not ok or not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end
end
fit()
openMenu()
