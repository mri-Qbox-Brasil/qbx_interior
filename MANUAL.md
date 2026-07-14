# qbx_interior — Manual

Biblioteca de shells (interiores) K4MB1 para Qbox: expõe funções que spawnam um shell numa coordenada, teleportam o jogador para dentro e devolvem os pontos de interesse do interior.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Shells incluídos](#shells-incluídos)
5. [Shells opcionais](#shells-opcionais)
6. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
7. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `ox_lib` | Sim | `lib.requestModel` |
| `qbx_core` | Sim | Usado pelos exports de servidor (`SetPlayerBucket`, `SetEntityBucket`) |

Este recurso não faz nada sozinho: ele é consumido por outros recursos (housing, roubo a residências, etc.) através dos exports.

---

## Instalação

1. Copie a pasta `qbx_interior` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure qbx_interior
   ```
   O `fxmanifest.lua` declara `this_is_a_map 'yes'` e registra o `DLC_ITYP_REQUEST` de `starter_shells_k4mb1.ytyp` — os shells da pasta `stream/` ficam disponíveis automaticamente.
3. Para usar os exports de `client/optional.lua`, adicione os assets dos packs K4MB1 correspondentes ao servidor (ver [Shells opcionais](#shells-opcionais)). Eles **não** acompanham este recurso.

---

## Configuração

`config/server.lua` define os interiores conhecidos pelos exports de servidor `CreateDefaultApartment` e `CreateDefaultInterior`. Só é lido no servidor.

### `Apartments[<nome ou hash>]`

Interiores com pontos de interesse completos (usados como apartamento inicial).

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `exit` | `vector4` | Sim | Ponto de saída / spawn dentro do interior |
| `clothes` | `vector2` ou `vector3` | Sim | Guarda-roupa |
| `stash` | `vector2` ou `vector3` | Sim | Baú |
| `logout` | `vector2` ou `vector3` | Sim | Ponto de logout |

### `Interiors[<nome ou hash>]`

Interiores simples, só com saída.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `exit` | `vector4` | Sim | Ponto de saída / spawn dentro do interior |

### Chave string vs. chave hash

A chave da tabela define o comportamento do export:

- **Chave string** (ex.: `'GTAOHouseMid1'`) — interior de MLO/IPL já existente no mapa. As coordenadas do config são **absolutas**; nenhum objeto é criado, apenas teleporte e routing bucket.
- **Chave hash** (ex.: `` `furnitured_midapart` ``) — shell de objeto. O servidor cria o objeto no `Coords` informado e as coordenadas do config são tratadas como **offsets** relativos a esse ponto.

---

## Shells incluídos

Os shells em `stream/` (starter pack K4MB1) e os exports de cliente que os utilizam:

| Export (cliente) | Modelo |
|---|---|
| `CreateApartmentFurnished` | `furnitured_midapart` |
| `CreateHouseRobbery` | `furnitured_midapart` |
| `CreateFurniMid` | `furnitured_midapart` |
| `CreateApartmentShell` | `shell_v16low` |
| `CreateTier1House` | `shell_v16mid` |
| `CreateTrevorsShell` | `shell_trevor` |
| `CreateCaravanShell` | `shell_trailer` |
| `CreateLesterShell` | `shell_lester` |
| `CreateRanchShell` | `shell_ranch` |
| `CreateContainer` | `container_shell` |
| `CreateFurniMotelModern` | `modernhotel_shell` |
| `CreateFurniMotelStandard` | `standardmotel_shell` |
| `CreateFranklinAunt` | `shell_frankaunt` |
| `CreateGarageMed` | `shell_garagem` |
| `CreateMichael` | `shell_michael` |
| `CreateOffice1` | `shell_office1` |
| `CreateStore1` | `shell_store1` |
| `CreateWarehouse1` | `shell_warehouse1` |

`CreateApartmentFurnished` é o único que devolve todos os pontos de interesse (`exit`, `clothes`, `stash`, `logout`). Os demais devolvem apenas `exit`.

---

## Shells opcionais

`client/optional.lua` traz cerca de 100 exports adicionais (`CreateMedium2`, `CreateMansion`, `CreateHighend1`, `CreateK4Garage1`, `CreateV2Modern3`, ...) para os packs pagos da K4MB1. **Os modelos não estão no `stream/` deste recurso** — os exports só funcionam se você adicionar os assets do pack ao servidor. Sem o modelo, o `lib.requestModel` trava a chamada.

Os packs referenciados no arquivo (cada bloco de exports tem o link em comentário):

Medium Housing V1 · Modern Housing V1 · Classic Housing V1 · Highend Housing V1 · Deluxe Housing V1 · Stash House · Garage · Office · Store · Warehouse · Highend Lab · Furnished Stash House · Furnished Housing · Furnished Motel · Furnished Modern Hotels · Drug Lab · Mansion Housing · Empty Hotel · Empty Motel · Default Shells V2 · Deluxe V2 · Highend V2 · Medium V2 · Modern V2 · Vinewood V2 · K4MB1 September Update

---

## Entrypoints para outros recursos

### Exports de cliente — criar um shell

Todos os exports `Create*` de `client/main.lua` e `client/optional.lua` têm a mesma assinatura: recebem a coordenada de spawn e retornam `{ objects, POIOffsets }`.

```lua
local result = exports.qbx_interior:CreateApartmentShell(vec4(x, y, z, h))
local objects = result[1]      -- lista de entidades criadas (para o despawn)
local offsets = result[2]      -- POIOffsets: exit (e clothes/stash/logout no CreateApartmentFurnished)
```

O shell é criado congelado na coordenada informada, a tela dá fade out/in e o jogador é teleportado para o ponto de saída (spawn + offset). Os `POIOffsets` são **relativos** ao ponto de spawn.

### Export de cliente — shell customizado

```lua
-- model aceita nome ou hash
local result = exports.qbx_interior:CreateShell(spawn, exitXYZH, model)
```

`exitXYZH` deve ser uma tabela `{x, y, z, h}` (ex.: `json.decode('{"x": 0.0, "y": 0.0, "z": 1.0, "h": 0.0}')`).

Atenção: o `CreateShell` teleporta para um ponto fixo interno (`+0.089, -2.677, +0.761`, heading `270.76`), não para o `exitXYZH` passado — este é apenas devolvido em `POIOffsets.exit`.

### Export de cliente — destruir o shell

```lua
exports.qbx_interior:DespawnInterior(objects, function()
    -- callback executado após todas as entidades serem deletadas
end)
```

### Exports de servidor

```lua
-- Interior com pontos de interesse (definido em config/server.lua > Apartments)
local coords, shell = exports.qbx_interior:CreateDefaultApartment(source, apartmentName, routingBucket, coords)

-- Interior simples (definido em config/server.lua > Interiors)
local coords, shell = exports.qbx_interior:CreateDefaultInterior(source, interiorName, routingBucket, coords)
```

Ambos: fazem o fade de tela, teleportam o jogador, colocam o jogador (e o shell, quando criado) no routing bucket informado e retornam as coordenadas absolutas dos pontos de interesse. O segundo retorno (`shell`) só existe quando a chave é um hash de modelo. Passar um nome inexistente no config gera `error`.

### Evento de fade de tela

```lua
-- Fade out (250ms), espera a tela apagar e fade in (500ms)
TriggerClientEvent('qb-interior:client:screenfade', source)
```

Usado por recursos como o `qbx_houserobbery` para mascarar teleportes.

---

## Estrutura de arquivos

```
qbx_interior/
├── client/
│   ├── main.lua          — exports dos shells inclusos, CreateShell, DespawnInterior, screenfade
│   └── optional.lua      — exports dos shells dos packs pagos K4MB1 (assets não inclusos)
├── server/
│   └── main.lua          — CreateDefaultApartment / CreateDefaultInterior (routing bucket + teleporte)
├── config/
│   └── server.lua        — tabelas Apartments e Interiors (pontos de saída, roupas, stash, logout)
├── stream/               — assets do starter pack K4MB1: 16 .ydr (shells), 3 .ytd (texturas),
│                           starter_shells_k4mb1.ytyp, starter_shells_k4mb1maps.ymap, _manifest.ymf
├── k4mb1shellstarter.pdf — documentação original do starter pack
└── fxmanifest.lua
```
