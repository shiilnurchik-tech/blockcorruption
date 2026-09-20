# Plano de migração para Minecraft 26.1.2

## Objetivo

Portar este fork do Immersive Portals Mod de Minecraft 1.21.1 para a linha 26.1.x,
com foco inicial em 26.1.2, preservando as funcionalidades principais e recuperando
gradualmente as compatibilidades com Sodium, Iris e outros mods.

O caminho recomendado é usar a branch `26.1` do MattLavalleeMA como base técnica do
porte, mantendo este fork como linha principal e tratando os forks de Rain156 e
DigitalWolf1313 como fontes seletivas de correções e referências.

## Diagnóstico inicial

A branch local `26.1.x` ainda contém o código de Minecraft 1.21.1:

| Componente | Estado local | Alvo inicial |
|---|---:|---:|
| Minecraft | 1.21.1 | 26.1.2 |
| Fabric Loader | 0.16.2 | 0.19.3 |
| Fabric API | 0.109.0+1.21.1 | 0.154.2+26.1.2 |
| Fabric Loom | legado | 1.17.13 |
| Sodium | 0.6.0 | 0.9.1 |
| Iris | 1.8.0 | 1.11.2 |
| DimLib | dependência para 1.21.1 | código incorporado ao projeto |
| Versão do mod | 6.0.6 | 7.0.0-alpha.1, a decidir |

O commit local `2b1d3546` é o ancestral comum usado pelos portes do Matt e do Rain.
Isso torna a integração e a comparação dos históricos mais previsíveis.

A origem oficial foi arquivada em abril de 2026, portanto os forks comunitários são
as principais referências para as versões atuais.

## Repositórios de referência

### MattLavalleeMA — base principal

- Repositório: <https://github.com/MattLavalleeMA/ImmersivePortalsMod>
- Branch: `26.1`
- Último commit analisado: `8b05b7b5`
- Base comum: `2b1d3546`

A branch contém 19 commits sobre a mesma base do fork local e já cobre:

- Build para o Minecraft 26.1 não ofuscado.
- Novo plugin `net.fabricmc.fabric-loom`.
- Remoção de Yarn e Parchment.
- Access Widener no namespace `official`.
- Atualização geral de imports, APIs e mixins.
- Novo sistema de tickets de chunks.
- Migração de persistência para `ValueInput` e `ValueOutput`.
- Mudanças em GUI, partículas, fog e lightmap.
- DimLib incorporado ao código.
- Inicialização do cliente e criação/entrada em mundos.
- Travessia de portais do Nether.
- Primeira implementação do renderizador de portais usando OpenGL bruto.
- Compatibilidade inicial com Sodium 0.9.1.

Essa branch declara zero erros de compilação, mas ainda possui funcionalidades
incompletas, caminhos convertidos em no-op e verificações visuais pendentes. Ela deve
ser tratada como uma base avançada de porte, não como uma versão pronta para lançar.

### Rain156 — referência arquitetural 26.2

- Repositório: <https://github.com/Rain156/ImmersivePortalsMod-26.2F>
- Branch: `26.2-Fabric`
- Último commit analisado: `efaf6cbe`

Essa branch é útil para:

- Comparar o build moderno do Loom.
- Identificar APIs que permaneceram entre 26.1 e 26.2.
- Avaliar a separação do DimLib como subprojeto.
- Antecipar alterações que provavelmente continuarão nas próximas versões.

Ela não deve ser aplicada integralmente. Várias classes e mixins são excluídos do
build para permitir a compilação, incluindo partes de Sodium, Iris, clipping,
sincronização de chunks e renderização.

### DigitalWolf1313 — correções seletivas

- Repositório: <https://github.com/DigitalWolf1313/ImmersivePortalsMod-Updated>
- Branch principal analisada: `1.21`

Esse fork continua baseado em Minecraft 1.21.1 e não serve como base direta do porte,
mas contém correções que deverão ser avaliadas depois que o baseline 26.1.2 estiver
funcional:

- Correção de dead loop do frustum usando MixinExtras, relevante para Valkyrien
  Skies.
- Melhorias de compatibilidade com Sodium.
- Correção de câmera em terceira pessoa.
- Compatibilidade básica com Voxy.
- Ajustes de shaders e renderização da mão.
- Ajustes de iluminação colorida.
- Atualizações de Loader e MixinExtras.

Essas mudanças deverão ser portadas por intenção. Cherry-picks diretos provavelmente
não funcionarão porque os alvos e as assinaturas da renderização mudaram no 26.1.

## Fase 0 — Preparação e rastreabilidade

1. Criar um ponto de segurança da branch atual:
   - Tag sugerida: `pre-26.1.2-port`.
   - Branch de integração sugerida: `port/26.1.2`.
   - Não trabalhar diretamente em `26.1.x` até a integração estabilizar.
2. Adicionar os três forks como remotes:
   - `matt`.
   - `rain`.
   - `digitalwolf`.
3. Registrar no histórico os SHAs usados como referência:
   - Base: `2b1d3546`.
   - Matt 26.1: `8b05b7b5`.
   - Rain 26.2: `efaf6cbe`.
4. Manter este documento atualizado com:
   - Commits importados.
   - Diferenças deliberadas.
   - Problemas conhecidos.
   - Evidências de testes.

## Fase 1 — Importar a fundação do Matt

Incorporar a série `2b1d3546..8b05b7b5`, preferencialmente preservando os commits
individuais para facilitar revisão, bisect e reversões.

Ordem lógica da série:

1. Ferramentas de migração.
2. Grande lote de APIs 26.1.
3. Correções mecânicas restantes.
4. Correções de mixins da câmera e do `GameRenderer`.
5. Integração do DimLib.
6. Fog e GravityChanger.
7. Renderização e mixins.
8. Fluxo de criação e entrada em mundos.
9. Compatibilidade Sodium e chunks.
10. Programas OpenGL e renderização de portais.

Não importar os seguintes logs de execução:

- `run_client_log3.txt`.
- `run_client_log5.txt`.

Eles somam mais de 226 mil linhas e não devem fazer parte do histórico. Também será
necessário revisar a autoria e a licença do código e dos assets do DimLib incorporado.

Critérios de saída:

- `./gradlew help` funciona.
- `./gradlew compileJava` apresenta zero erros.
- `./gradlew test` funciona.
- O repositório não contém logs gigantes de execução.
- Toda diferença em relação à branch do Matt está documentada.

## Fase 2 — Normalizar build e metadados

Revisar manualmente depois da importação:

- Minecraft `26.1.2`.
- Fabric Loader `0.19.3`.
- Fabric API `0.154.2+26.1.2`.
- Fabric Loom `1.17.13`.
- Sodium `mc26.1.2-0.9.1-fabric`.
- Iris `1.11.2+26.1-fabric`.
- Cloth Config `26.1.154`.
- Mod Menu `18.0.0`.
- MidnightLib, caso o DimLib incorporado continue dependendo dela.
- Versão mínima correta do Java.
- Gradle Wrapper compatível.
- `git_branch=26.1.x`.
- Intervalos corretos em `fabric.mod.json`.
- Nome final do JAR.

Proposta de versionamento:

- Desenvolvimento: `7.0.0-alpha.1`.
- Primeira versão amplamente testável: `7.0.0-beta.1`.
- Versão estável: `7.0.0`.

Manter `6.0.6` dificultaria distinguir artefatos para 1.21.1 dos artefatos 26.1.2.

## Fase 3 — Auditoria estrutural

### Persistência e networking

Revisar:

- Uso de `ValueInput` e `ValueOutput`.
- Codecs de portais e dados globais.
- Compatibilidade de saves existentes.
- Pacotes Fabric clientbound e serverbound.
- RPC e sincronização de dimensões.
- Teleporte do jogador e confirmação de posição.

Teste obrigatório: abrir uma cópia de um mundo 1.21.1, carregar os portais, salvar,
fechar e reabrir o mundo. Nunca usar o único exemplar de um save durante os testes.

### Dimensões e DimLib

1. Confirmar licença e proveniência do DimLib incorporado.
2. Escolher entre:
   - Código incorporado ao source set principal, como no Matt.
   - Subprojeto Gradle, como no Rain.
3. Inicialmente, manter a forma usada pelo Matt para reduzir o número de mudanças
   simultâneas. A modularização pode ocorrer depois da estabilização.

Testar:

- Dimensões dinâmicas.
- Dimension Stack.
- Alternate Dimensions.
- Datapacks de portais personalizados.
- Criação e remoção dinâmica de dimensões.
- Entrada tardia de jogadores em um servidor.

### Tickets e chunks

Revisar:

- Novo `TicketStorage`.
- Carregamento de chunks remotos.
- Chunk tracking por jogador.
- Entrada e saída rápida de portais.
- Portais aninhados.
- `earlyRemoteUpload`, atualmente convertido em no-op no porte do Matt.

Essa área requer testes em servidor e, idealmente, com latência simulada.

## Fase 4 — Renderização principal

Esta é a área de maior risco do porte.

O Minecraft 26.1 substituiu grande parte do fluxo antigo por `RenderPipeline`,
`GpuBuffer`, `RenderPass`, frame graph e múltiplos render targets. A API moderna não
oferece o stencil dinâmico usado pelo algoritmo antigo. O porte do Matt contorna essa
limitação usando OpenGL bruto.

Revisar e testar separadamente:

1. Portal normal com stencil.
2. Portal dentro de portal.
3. Portais espelhados.
4. Portais escalados.
5. Portais com rotação.
6. Renderização de entidades.
7. Partículas.
8. Céu, nuvens, fog e clima.
9. Renderização da mão.
10. Terceira pessoa.
11. GUI portals.
12. Renderer por framebuffer.
13. Screenshot e resize da janela.

Critérios visuais:

- Câmera sem deslocamento, orientação ou FOV incorretos.
- Nenhum conteúdo aparecendo fora da moldura do portal.
- Ausência de flickering nas bordas.
- Profundidade correta entre entidades, mundo e portal.
- Nenhum vazamento de estado OpenGL depois do portal.
- Portais aninhados respeitam o limite configurado.
- Resize e fullscreen reconstroem buffers corretamente.

Revisar o ciclo de vida dos novos programas OpenGL:

- Criação somente na render thread.
- Destruição de programas, VAOs e buffers.
- Restauração de blend, depth, stencil, culling e viewport.
- Comportamento após resource reload.
- Compatibilidade com drivers e fabricantes diferentes.

## Fase 5 — Sodium, Iris e outros mods

Executar os testes em camadas:

1. Fabric API e Immersive Portals apenas.
2. Adicionar Sodium.
3. Adicionar Iris sem shader.
4. Iris com shader simples.
5. Shader com sombras, água e transparência.
6. Mods adicionais, inicialmente um por vez.

A compatibilidade Iris do porte do Matt ainda contém caminhos incompletos ou no-op.
Ela não precisa bloquear a primeira alpha, mas essa limitação deve ser declarada.

| Configuração | Alpha | Beta | Estável |
|---|---:|---:|---:|
| Renderer vanilla | obrigatório | obrigatório | obrigatório |
| Sodium 0.9.1 | básico | completo | completo |
| Iris sem shader | smoke test | obrigatório | completo |
| Iris com shaders | opcional | parcial | definido e documentado |
| Servidor dedicado | obrigatório | obrigatório | obrigatório |
| Cliente sem Sodium | obrigatório | obrigatório | obrigatório |

## Fase 6 — Selecionar melhorias do DigitalWolf

Depois que o baseline 26.1.2 estiver funcional, revisar individualmente:

- Correção de frustum e Valkyrien Skies.
- Câmera em terceira pessoa.
- Voxy.
- Renderização da mão com shaders.
- Iluminação colorida.
- Alterações de compatibilidade com Sodium.

Processo para cada alteração:

1. Entender o bug original.
2. Criar uma reprodução no baseline 26.1.2.
3. Verificar se a migração do Matt já resolveu o problema.
4. Portar a intenção usando as APIs do 26.1.
5. Adicionar teste ou roteiro reproduzível.
6. Não copiar junto atualizações de dependências específicas de 1.21.1.

## Fase 7 — Usar o Rain como auditoria futura

Comparar o resultado 26.1.2 com o porte 26.2 para localizar:

- Classes já removidas em 26.2.
- Mixins frágeis ou excessivamente dependentes de ordinais.
- APIs que ganharam substitutos mais adequados.
- Estrutura modular do DimLib.
- Mudanças futuras de dependências.

Não copiar as exclusões do `build.gradle` do Rain sem uma análise funcional. Várias
delas desativam partes essenciais do mod.

## Fase 8 — Automação e CI

O projeto atual não possui workflow de build. Adicionar CI com:

- Java correto para o alvo.
- `./gradlew build`.
- `./gradlew test`.
- Verificação de formatação, se adotada.
- Upload do JAR como artifact.
- Cache Gradle.
- Validação de que logs, dumps e diretórios de execução não foram commitados.

Adicionar ao `.gitignore`, quando aplicável:

- `run/`.
- `logs/`.
- `crash-reports/`.
- `run_client_log*.txt`.
- Dumps e screenshots de testes.

Criar duas configurações de execução:

- Cliente mínimo.
- Cliente com Sodium e Iris.

## Fase 9 — Testes funcionais

### Checklist mínimo para alpha

- Cliente chega ao menu.
- Mundo novo pode ser criado.
- Mundo existente pode ser carregado.
- Servidor dedicado inicia.
- Cliente entra e sai do servidor.
- Portal vanilla para o Nether funciona.
- Portal customizado funciona.
- Teleporte funciona nas duas direções.
- Entidades atravessam portais.
- Itens e projéteis atravessam portais.
- Interação e quebra de blocos através do portal.
- Som através do portal.
- Redstone através do portal.
- Portais globais.
- Dimension Stack.
- Portal wand.
- Command sticks.
- Espelhos.
- Portais escalados e rotacionados.
- Morte e respawn.
- Troca rápida de dimensões.
- Reentrada após desconexão.
- Save e reload preservam os portais.
- Resize, fullscreen e resource reload.
- Distâncias de renderização baixa e alta.

### Checklist adicional para beta

- Repetir os casos principais com Sodium.
- Repetir os casos visuais com Iris.
- Sessão contínua de 30 a 60 minutos.
- Teste com dois ou mais jogadores.
- Teste com latência.
- Análise de vazamento de memória e recursos GPU.
- Testes em mais de um fabricante de GPU, quando possível.

## Organização sugerida dos PRs

1. `build: migrate project foundation to Minecraft 26.1.2`
2. `port: migrate vanilla APIs and serialization`
3. `port: integrate DimLib for 26.1.2`
4. `port: migrate networking, teleportation and chunk loading`
5. `render: migrate portal rendering pipeline`
6. `compat: restore Sodium 0.9.1 support`
7. `compat: restore Iris 1.11.x support`
8. `test: add CI and migration test matrix`
9. `compat: port selected DigitalWolf fixes`
10. `release: prepare 7.0.0-alpha.1`

## Critérios de conclusão

### Alpha

- Compila e gera um JAR.
- Cliente e servidor dedicado iniciam.
- Mundos podem ser criados e carregados.
- Portais básicos são renderizados e atravessados sem Sodium.
- Limitações conhecidas estão documentadas.

### Beta

- Servidor, tickets, chunks e portais aninhados estão estáveis.
- Sodium funciona nos principais cenários.
- Migração e persistência de saves foram testadas.
- Não há vazamentos GPU conhecidos nos fluxos principais.

### Estável

- Persistência, multiplayer e recursos periféricos foram testados.
- As compatibilidades declaradas possuem uma matriz de testes reproduzível.
- Os no-ops restantes foram implementados ou documentados como limitações aceitas.
- O changelog e os metadados de distribuição estão completos.

## Próxima etapa recomendada

Executar as fases 0 a 2:

1. Criar a tag de segurança e a branch de integração.
2. Adicionar os remotes de referência.
3. Importar cuidadosamente a série do Matt, removendo os logs gigantes.
4. Normalizar dependências e metadados para 26.1.2.
5. Confirmar um build limpo antes de iniciar correções funcionais adicionais.
