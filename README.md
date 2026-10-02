# AliveNpcs — AliveTrigger de exemplo

Este repositório é um pacote de exemplo de **AliveTrigger** para o [AliveNpcs](https://www.nexusmods.com/stardewvalley/mods/43475) e também o tutorial para você criar o seu.

Um AliveTrigger é um **gatilho**: uma situação que os moradores passam a perceber. Quando ela acontece perto de um NPC, ele guarda aquilo na memória e pode andar até você para comentar, com fala gerada por IA na voz dele. Opcionalmente, ele ainda te entrega um item.

Tudo é feito em **um único arquivo JSON**. Não tem DLL, não precisa compilar e dá para editar em qualquer editor de texto.

Os seis gatilhos deste pacote:

| Gatilho | O que acontece |
|---|---|
| `caught_in_the_rain` | Você anda na chuva perto de um conhecido e ele brinca com você por estar encharcado. |
| `out_after_midnight` | Um amigo te vê na rua depois da meia-noite, manda você ir dormir e te dá um café. |
| `carrying_prismatic_shard` | Clint, Maru, Demetrius, o Mago ou Abigail reparam no fragmento prismático que você carrega. |
| `hated_gift` | Quem recebe um presente que odeia fica abertamente ofendido. |
| `beer_run_at_the_saloon` | Comprar cerveja com o Gus vira uma lembrança própria para quem estava no Saloon. |
| `struck_with_a_weapon` | Acertar um NPC com uma arma de verdade rende uma reação bem mais séria. |

## Sumário

- [Requisitos](#requisitos)
- [Como instalar este pacote](#como-instalar-este-pacote)
- [Criando o seu pacote em 5 minutos](#criando-o-seu-pacote-em-5-minutos)
- [Os dois tipos de gatilho](#os-dois-tipos-de-gatilho)
- [Referência dos campos](#referência-dos-campos)
- [Escrevendo o `When`](#escrevendo-o-when)
- [Eventos disponíveis no `On`](#eventos-disponíveis-no-on)
- [Escrevendo `Summary` e `Reaction`](#escrevendo-summary-e-reaction)
- [Entregando um item](#entregando-um-item)
- [Limites e regras do mod](#limites-e-regras-do-mod)
- [Testando e resolvendo problemas](#testando-e-resolvendo-problemas)
- [Receitas prontas](#receitas-prontas)
- [Publicando](#publicando)

## Requisitos

- Stardew Valley 1.6
- SMAPI
- AliveNpcs `1.7.0` ou superior, com a opção **Seen and Heard** ligada (é o padrão)

## Como instalar este pacote

Copie esta pasta inteira para a pasta `Mods` do Stardew Valley:

```text
Stardew Valley/
└── Mods/
    ├── AliveNpcs/
    └── alive-npcs-trigger-example/
        ├── manifest.json
        └── alive-trigger.json
```

Abra o jogo pelo SMAPI. No console deve aparecer:

```text
[AliveNpcs] AliveTrigger pack '[AT] AliveNpcs - Example Triggers': loaded 6 trigger(s).
```

Se essa linha apareceu, os gatilhos estão ativos.

## Criando o seu pacote em 5 minutos

**1. Crie uma pasta** dentro de `Mods`, com o nome que quiser. Sugestão: `[AT] Nome Do Seu Pacote`.

**2. Crie o `manifest.json`.** É o arquivo que o SMAPI exige de todo content pack:

```json
{
  "Name": "[AT] Meus Gatilhos",
  "Author": "SeuNome",
  "Version": "1.0.0",
  "Description": "Gatilhos extras para o AliveNpcs.",
  "UniqueID": "SeuNome.MeusGatilhos",
  "ContentPackFor": {
    "UniqueID": "Lucas.AliveNpcs",
    "MinimumVersion": "1.7.0"
  },
  "UpdateKeys": []
}
```

Só duas coisas importam aqui: o `UniqueID` precisa ser único (use `SeuNome.NomeDoPacote`) e o `ContentPackFor` precisa apontar para `Lucas.AliveNpcs`. É isso que faz o AliveNpcs encontrar o seu pacote.

**3. Crie o `alive-trigger.json`** na mesma pasta, com o nome exatamente assim:

```json
{
  "Format": 1,
  "Triggers": [
    {
      "Id": "meu_primeiro_gatilho",
      "When": "WEATHER Here Rain Storm",
      "Summary": "{farmer} was walking around in the rain.",
      "Reaction": "Tease {farmer} about getting soaked."
    }
  ]
}
```

**4. Abra o jogo**, vá para a rua num dia de chuva e passe perto de um morador. Ele mostra um `!`, anda até você e comenta.

Pronto. O resto deste README é para você ir além desse exemplo.

## Os dois tipos de gatilho

Todo gatilho tem `When`, `On`, ou os dois. Isso define o tipo.

### Gatilho de condição (`When`)

Você descreve um **estado do jogo**: está chovendo, é depois da meia-noite, o jogador carrega tal item. Cerca de uma vez por segundo, o mod confere a condição. Se ela for verdadeira e houver um NPC por perto que se encaixe nos seus filtros, o gatilho dispara.

```json
{
  "Id": "out_after_midnight",
  "When": "TIME 2400",
  "RangeTiles": 8,
  "MinHearts": 4,
  "Summary": "{farmer} was still out and about after midnight.",
  "Reaction": "Tell {farmer} to get some rest."
}
```

O NPC mais próximo que se encaixa é quem reage. Outros moradores dentro do alcance também guardam a lembrança.

### Gatilho de evento (`On`)

Você se pendura em uma **ação que o AliveNpcs já percebe**: dar presente, comprar, revirar lixo, acertar alguém com uma ferramenta. O seu gatilho não cria um acontecimento novo; ele muda o que é lembrado e como o NPC reage naquele caso específico.

```json
{
  "Id": "hated_gift",
  "On": "gift",
  "Match": { "taste": "hated" },
  "Reaction": "You are openly offended by this gift."
}
```

O `Match` filtra pelos detalhes do evento (veja a [tabela de eventos](#eventos-disponíveis-no-on)). Sem `Match`, o gatilho vale para toda ocorrência daquele evento.

Você pode combinar os dois: um gatilho com `On` e `When` só se aplica se o evento acontecer **e** a condição for verdadeira naquele momento (por exemplo, um presente dado em dia de chuva).

## Referência dos campos

| Campo | Obrigatório | Padrão | O que faz |
|---|---|---|---|
| `Id` | Sim | — | Nome único do gatilho dentro do seu pacote. Só letras, números, `_`, `.` e `-`, até 64 caracteres. |
| `When` | Um dos dois | — | Condição do jogo (Game State Query). Veja [Escrevendo o `When`](#escrevendo-o-when). |
| `On` | Um dos dois | — | Evento ao qual o gatilho se aplica. Veja [Eventos disponíveis](#eventos-disponíveis-no-on). |
| `Match` | Não | — | Só com `On`. Pares `"detalhe": "valor"`; todos precisam bater. |
| `Summary` | Sim com `When` | — | O fato, em uma frase. É o que o NPC lembra. Com `On` é opcional: se você omitir, vale o texto original do evento. Até 200 caracteres. |
| `Reaction` | Não | — | Instrução de como o NPC deve reagir na fala. Até 300 caracteres. |
| `Npcs` | Não | todos | Lista de nomes internos de NPCs que podem reagir, por exemplo `["Abigail", "Sam"]`. |
| `MinHearts` | Não | `0` | Corações mínimos com o jogador para o NPC reagir (0 a 14). |
| `RangeTiles` | Não | `8` | Só com `When`. Distância máxima, em tiles, entre o jogador e o NPC (1 a 30). |
| `Importance` | Não | `5` | Só com `When`. De 1 a 11: quão marcante é o fato. Valores baixos podem não gerar abordagem, só memória. |
| `GiveItem` | Não | — | ID qualificado de um item que o NPC entrega depois de falar, por exemplo `"(O)395"`. |
| `MaxPerDay` | Não | `1` | Quantas vezes por dia o gatilho pode disparar (1 a 20). |
| `MemoryOnly` | Não | `false` | Se `true`, o NPC só guarda a lembrança; ninguém anda até você. |

Em gatilhos de evento (`On`), `Npcs` e `MinHearts` são conferidos contra o **alvo do evento** (quem recebeu o presente, quem foi atingido, o dono da loja). Eventos sem alvo, como `trash` e `bomb`, ignoram esses dois filtros.

## Escrevendo o `When`

O `When` usa as **Game State Queries** do próprio Stardew Valley 1.6, as mesmas do Content Patcher. A lista completa está na wiki: <https://stardewvalleywiki.com/Modding:Game_state_queries>.

Regras rápidas:

- Separe várias condições com vírgula; **todas** precisam ser verdadeiras.
- Coloque `!` na frente para negar.
- Use `Here` para o local onde o jogador está e `Current` para o jogador.

Condições úteis:

| Condição | Significado |
|---|---|
| `WEATHER Here Rain Storm` | Está chovendo ou com tempestade aqui. |
| `LOCATION_IS_OUTDOORS Here` | O jogador está ao ar livre. |
| `LOCATION_NAME Here Saloon` | O jogador está no Saloon. |
| `TIME 2400` | São 0h ou mais. `TIME 600 900` significa entre 6h e 9h. |
| `SEASON winter` | É inverno. |
| `DAY_OF_WEEK Friday` | É sexta-feira. |
| `PLAYER_HAS_ITEM Current (O)74` | O jogador carrega um fragmento prismático. |
| `PLAYER_CURRENT_MONEY Current 1000000` | O jogador tem 1.000.000g ou mais. |
| `PLAYER_HAS_BUFF Current 17` | O jogador está com o buff de ID 17 (embriagado). |
| `PLAYER_COMBAT_LEVEL Current 10` | Nível de combate 10. |

Exemplo combinando: chuva, ao ar livre e fora do inverno.

```json
"When": "WEATHER Here Rain Storm, LOCATION_IS_OUTDOORS Here, !SEASON winter"
```

Se o nome da condição não existir, o mod desliga só aquele gatilho e avisa no console. Um nome certo com argumentos errados (por exemplo `TIME abc`) faz o próprio jogo registrar um erro a cada checagem, então teste sempre com o console aberto.

## Eventos disponíveis no `On`

| `On` | Quando acontece | Alvo | Detalhes para o `Match` |
|---|---|---|---|
| `gift` | O jogador dá um presente. | Quem recebeu | `taste` (`loved`, `liked`, `neutral`, `disliked`, `hated`), `itemId` |
| `consume` | O jogador termina de comer ou beber. | — | `itemId` |
| `bomb` | Uma explosão causada pelo jogador. | — | `radius` |
| `trash` | O jogador revira uma lixeira. | — | `can` (`JodiAndKent`, `EmilyAndHaley`, `Mayor`, `Museum`, `Blacksmith`, `Saloon`, `Evelyn`, `JojaMart`) |
| `tool_hit` | Ferramenta ou arma acerta um NPC. | Quem foi atingido | `weapon` (`true` ou `false`) |
| `tool_near_miss` | Machado, picareta ou arma passa raspando. | Quem quase foi atingido | `weapon` |
| `slingshot_hit` | Estilingue acerta um NPC. | Quem foi atingido | — |
| `slingshot_near_miss` | Projétil cai perto de um NPC. | Quem quase foi atingido | — |
| `room_locked` | O jogador tenta uma porta de quarto trancada. | Dono do quarto | — |
| `room_unlocked` | O jogador abre uma porta de quarto pela primeira vez. | Dono do quarto | — |
| `purchase` | O jogador fecha uma loja depois de comprar. | Dono da loja, se presente | `itemIds` (lista de tudo o que foi comprado) |
| `dating_proposal` | Pedido de namoro. | Quem recebeu o pedido | `result` (`accepted`, `rejected`) |
| `marriage_proposal` | Pedido de casamento. | Quem recebeu o pedido | `result` |
| `sleep_outside` | O jogador desmaia fora de casa. | — | `reason` (`2am`, `exhausted`) |

Além dos detalhes acima, todo evento com alvo aceita `"subject": "NomeDoNpc"` no `Match`.

Dicas:

- A comparação ignora maiúsculas e minúsculas.
- Em detalhes que são lista, como `itemIds`, basta o seu valor ser **um** dos itens.
- Use sempre `itemId` e nunca o nome do item: o nome muda conforme o idioma do jogo.
- `purchase` e `sleep_outside` são só memória. Um gatilho neles muda o que é lembrado, mas ninguém anda até o jogador, então `Reaction` e `GiveItem` não têm efeito.

## Escrevendo `Summary` e `Reaction`

Esses dois textos vão para o prompt da IA, então escreva em **inglês**, mesmo que você jogue em outro idioma. A fala do NPC sai no idioma configurado no AliveNpcs.

**`Summary` é o fato.** Escreva em terceira pessoa, no passado, como alguém contando o que viu:

```text
{farmer} was walking around {location} in the pouring rain, completely soaked.
```

**`Reaction` é a direção de cena.** Fale com o NPC em segunda pessoa e diga o tom, não a fala pronta:

```text
Tease {farmer} a little about being out in this weather, then show you care.
```

Você pode usar três marcadores nos dois textos:

| Marcador | Vira |
|---|---|
| `{farmer}` | O nome do jogador. |
| `{npc}` | O NPC que reage (ou o alvo do evento). |
| `{location}` | O nome do local. |

O que funciona melhor:

- **Uma ideia por gatilho.** Frases curtas e específicas rendem falas melhores que parágrafos.
- **Deixe a personalidade com o NPC.** Não escreva "say: ..."; diga a emoção e deixe cada morador falar do jeito dele.
- **Sem quebras de linha.** O mod junta tudo em uma linha e corta o que passar do limite.

## Entregando um item

Com `GiveItem`, o NPC coloca o item no seu inventário quando a conversa termina (ou solta no chão, se o inventário estiver cheio).

```json
"GiveItem": "(O)395"
```

Use o **ID qualificado** do item: `(O)` para objetos, `(BC)` para máquinas, `(H)` para chapéus e assim por diante. A lista de IDs está em <https://stardewvalleywiki.com/Modding:Items>.

Mencione a entrega no `Reaction` ("hand them a coffee"), senão o NPC fala de uma coisa e entrega outra sem avisar.

Se o ID não existir, o gatilho continua funcionando sem o item e o console avisa uma vez.

## Limites e regras do mod

- **`MaxPerDay`** conta por gatilho, por dia de jogo. Além disso, o mesmo gatilho espera pelo menos 1 minuto real entre um disparo e outro.
- **Um NPC por vez.** Só um morador anda até o jogador de cada vez; se outro gatilho disparar no meio, ele entra na fila ou é descartado.
- **Gatilhos de condição não disparam** durante cutscenes, festivais, menus abertos ou diálogos.
- **NPCs ocupados** (pescando com você, já a caminho, controlados por outro mod) não reagem.
- **Até 50 gatilhos** por pacote.
- **Multiplayer:** vale só para o host.
- **Um gatilho de evento por ação.** Se dois gatilhos `On` servirem para o mesmo acontecimento, vale o primeiro na ordem de carregamento.

## Testando e resolvendo problemas

Abra o jogo com o console do SMAPI visível. As mensagens do AliveNpcs dizem o que aconteceu com o seu pacote.

| Mensagem ou sintoma | Causa provável |
|---|---|
| Nenhuma linha `AliveTrigger pack ...` | O `manifest.json` não tem `ContentPackFor` apontando para `Lucas.AliveNpcs`, ou o arquivo não se chama exatamente `alive-trigger.json`. |
| `could not read alive-trigger.json` | JSON inválido: vírgula sobrando, aspas faltando. Cole o arquivo em um validador de JSON. |
| `Id is required and may only use...` | `Id` vazio ou com espaço ou acento. |
| `needs When (a game state query) or On (an event)` | O gatilho não tem nenhum dos dois. |
| `On '...' is not a supported event` | O nome do evento não está na tabela acima. |
| `Summary is required for a When trigger` | Faltou o `Summary`. |
| `has an invalid When condition and was disabled` | O nome da condição no `When` não existe. |
| `gives unknown item` | O ID em `GiveItem` não existe. |
| Carregou, mas nada acontece | Confira: Seen and Heard ligado, NPC dentro do `RangeTiles`, corações suficientes, `MaxPerDay` ainda não gasto, IA configurada. |
| O NPC lembra, mas não vem falar | `Importance` baixa, `MemoryOnly` ligado, ou o evento é só memória. |

Para testar rápido, use uma condição sempre verdadeira e alcance grande:

```json
{ "Id": "teste", "When": "TRUE", "RangeTiles": 30, "Summary": "{farmer} waved at {npc}.", "MaxPerDay": 20 }
```

As linhas de depuração (`trigger '...' fired near ...`) aparecem no arquivo de log do SMAPI.

## Receitas prontas

**NPC específico reage a um item no inventário**

```json
{
  "Id": "willy_sees_legend",
  "When": "PLAYER_HAS_ITEM Current (O)163",
  "Npcs": [ "Willy" ],
  "Importance": 8,
  "Summary": "{farmer} showed up carrying the Legend, the rarest fish in the valley.",
  "Reaction": "You are stunned and thrilled. Ask how they managed to catch it."
}
```

**Presente amado rende um mimo de volta**

```json
{
  "Id": "loved_gift_thanks",
  "On": "gift",
  "Match": { "taste": "loved" },
  "MinHearts": 6,
  "Reaction": "You are delighted. Thank {farmer} warmly and give them a cookie in return.",
  "GiveItem": "(O)223"
}
```

**Só memória: os moradores lembram, mas ninguém aborda**

```json
{
  "Id": "rich_farmer",
  "When": "PLAYER_CURRENT_MONEY Current 1000000",
  "Summary": "{farmer} is rumored to have become a millionaire.",
  "MemoryOnly": true
}
```

**Evento com condição: bomba dentro da cidade**

```json
{
  "Id": "bomb_in_town",
  "On": "bomb",
  "When": "LOCATION_NAME Here Town",
  "Summary": "{farmer} set off a bomb right in the middle of town.",
  "Reaction": "You are alarmed and angry. This is where people live."
}
```

## Publicando

- Mantenha o prefixo `[AT]` no nome para o pacote ser fácil de reconhecer.
- Na página do mod, marque o AliveNpcs como requisito.
- Suba a `Version` do `manifest.json` a cada atualização.

## Licença

MIT. Use este repositório como base para o seu pacote.
