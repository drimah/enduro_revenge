# ENDURO REVENGE

Uma releitura moderna e pós-apocalíptica do clássico **Enduro** (Atari 2600), feita em **Godot 4** e toda em 3D.
Efeitos sonoros, músicas e texturas são gerados por código. Os modelos 3D são cenas `.tscn` editáveis que você pode trocar pelos seus próprios.

## Como jogar (sem instalar nada)

O mesmo ZIP serve para Windows e Linux Mint:

| Sistema | O que abrir |
|---|---|
| **Windows** | Dê dois cliques em `EnduroRevenge.exe` |
| **Linux Mint** | Dê dois cliques em `JOGAR_LINUX.sh` (ou rode `./EnduroRevenge.x86_64` no terminal) |

> No Linux, se o arquivo não abrir, clique com o botão direito → Propriedades → Permissões → marque "Permitir execução como programa". Ou rode no terminal: `chmod +x EnduroRevenge.x86_64 JOGAR_LINUX.sh`.
> No Windows, o SmartScreen pode avisar que o app é desconhecido: clique em "Mais informações" → "Executar assim mesmo".

## Como abrir no Godot (para editar)

1. Instale o **Godot 4.3 ou superior** (versão padrão, não a .NET).
2. No Godot, clique em **Importar** e escolha `projeto_godot/project.godot`.
3. Aperte **F5** para jogar.

Para gerar os executáveis de novo: **Projeto → Exportar** (as predefinições Windows e Linux já estão configuradas, basta instalar os modelos de exportação quando o Godot pedir).

## Objetivo

Como no Enduro original: **ultrapasse a quantidade de carros exigida antes que o dia acabe**.
- Dia 1: 70 carros. Cada dia seguinte exige +25 (máximo de 200).
- Cada carro que **te ultrapassar** devolve +1 ao contador.
- Bateu a meta? Sobreviva até o amanhecer para começar o próximo dia.
- Não bateu quando o dia termina? **Fim da estrada.**

## Ciclo do dia (pós-apocalíptico)

Amanhecer tóxico → Dia escaldante → **Tempestade de areia** (visibilidade mínima e raios) → Tarde cinzenta → Pôr do sol radioativo → Crepúsculo → **Noite com chuva ácida** (pista escorregadia, só faróis e lanternas visíveis) → **Névoa radioativa** → amanhece de novo.

## Controles

| Ação | Teclado | Joystick (Xbox/PlayStation/genérico) |
|---|---|---|
| Acelerar | W / ↑ | RT ou A (✕) |
| Frear | S / ↓ | LT ou B (◯) |
| Direção | A D / ← → | Analógico esquerdo ou D-pad |
| Nitro | Shift / Espaço | RB, X ou Y |
| Pausar | Esc / P | Start |
| Voltar ao menu (na pausa) | Q / Backspace | Y / Back |
| Música liga/desliga | M | — |
| Tela cheia | F11 | — |

O joystick vibra nas batidas e nos trovões.

## Novidades em relação ao original

- Nitro com barra que recarrega devagar
- Carros de saqueadores (com espinhos) que trocam de faixa; aparecem mais a cada dia
- Caminhões lentos, vans e sedans enferrujados
- Curvas com força centrífuga e morros
- Sair da pista reduz a velocidade e faz o carro trepidar
- Recorde salvo automaticamente

## Usando seus próprios modelos 3D

Cada carro e cada peça do cenário é uma cena `.tscn` que você pode abrir e editar no editor 3D do Godot:

```
scenes/
  main.tscn                    nó Main: listas de cenas no Inspector
  veiculos/jogador.tscn        seu carro
  veiculos/inimigos/           sedan_1..3, van_1..2, caminhao_1..2, saqueador_1..2
  cenario/                     ruínas, árvores secas, carcaças, barris, ônibus, torre, placas...
  cenario/poste_refletor.tscn  poste que se repete na beira da pista
  cenario/horizonte.tscn       montanhas e cidade em ruínas ao fundo
```

**Trocar um modelo (ex.: um sedan):**
1. Copie seu modelo (`.glb`, `.gltf`, `.blend` ou `.fbx`) para a pasta `modelos/`.
2. Abra `scenes/veiculos/inimigos/sedan_1.tscn`, apague o nó **Modelo** e arraste seu arquivo para dentro do nó raiz.
3. A **frente do carro deve apontar para -Z** (a seta azul do gizmo aponta para a traseira). Se ficar ao contrário, gire o modelo 180° em Y.
4. Escala: 1 unidade = 1 metro. Um carro tem cerca de 2 m de largura e 4 m de comprimento; a pista tem 3 faixas de 4 m.
5. Salve e aperte F5.

**O que o jogo faz sozinho:**
- Converte os materiais do seu modelo (cor, textura, rugosidade, metal, emissão e cor de vértice) para que ele **curve junto com a pista**.
- Calcula o tamanho da colisão pelo modelo, se `Half Width` e `Half Length` estiverem em 0 no Inspector.
- Faz girar as **rodas**: nós com nome começando com `Roda` ou `Wheel`, ou que estejam no grupo `rodas`. Elas giram no eixo X, então o pivô deve ficar no centro da roda.

**Configurações no Inspector (raiz de cada cena):**
- Veículos (`scripts/veiculo.gd`): tamanho da colisão, velocidade mínima e máxima (m/s; ×3,6 = km/h), se troca de faixa, chance de aparecer, quanto a chance cresce por dia, dia mínimo, e se pode vir por trás.
- Cenário (`scripts/cenario.gd`): distância mínima e máxima da pista, rotação aleatória e chance de aparecer.

**Carro do jogador (`jogador.tscn`), convenções opcionais:**
- Nó `Corpo`: inclina nas curvas. Coloque a carroceria dentro dele e deixe as rodas fora.
- Grupo `farois` (SpotLight3D): acendem à noite e na névoa.
- Grupo `chamas`: fogo normal do escapamento. Grupo `chamas_nitro`: aparece no nitro. Grupo `luz_nitro`: luz que acende no nitro.

**Adicionar um carro ou peça novos:** duplique uma cena parecida, troque o modelo e arraste a nova cena para a lista **Cenas Inimigos** ou **Cenas Cenario** do nó `Main` (abra `scenes/main.tscn` e veja o Inspector).

> `ferramentas/gerar_cenas.gd` recria todas as cenas originais a partir de `scripts/builders.gd`. **Ele sobrescreve suas edições**, então use só para restaurar o original.

## Estrutura do código

```
scripts/main.gd       lógica do jogo (estados, pista, inimigos, ciclo do dia)
scripts/veiculo.gd    configurações de cada veículo (Inspector)
scripts/cenario.gd    configurações de cada peça de cenário (Inspector)
scripts/util.gd       adapta modelos ao mundo curvo, rodas e colisão automáticas
scripts/builders.gd   modelos low-poly originais feitos por código (usados pelo gerador)
scripts/audio.gd      efeitos sonoros e sons do motor sintetizados
scripts/music.gd      sintetizador de música (menu e corrida)
scripts/hud.gd        painel, menus e mensagens
shaders/              pista, deserto e shader que curva o mundo
```

## Desempenho

O jogo usa o renderizador **Forward+**. Se a placa de vídeo não tiver Vulkan, o Godot troca automaticamente para OpenGL.
Se ficar lento, deixe a janela menor (sem tela cheia) ou desligue as sombras em `main.gd` (`sun.shadow_enabled = false`).
