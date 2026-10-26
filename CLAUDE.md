# CLAUDE.md

Orientações para o Claude Code ao trabalhar neste repositório.

## Idioma: tudo em português

**Neste projeto, toda a interação é em português (pt-BR):** a conversa com o
usuário, a interface, os comentários do código, a documentação, as mensagens de
commit, os títulos e descrições de PR, as issues e os comentários no GitHub. Esta
regra **prevalece sobre os skills globais** (`howto-commit`, `howto-pullrequest` e
outros) que pedem inglês: siga o formato desses skills, mas escreva em português.

Ficam em inglês só os termos que as ferramentas exigem: os tipos do Conventional
Commits (`feat`, `fix`, `docs`, `test`, `chore`…), os trailers (`Refs:`,
`Assisted-by:`, `Co-Authored-By:`) e a palavra **`Closes #<n>`** na descrição do PR
— o GitHub só fecha a issue com as palavras-chave em inglês.

Os commits até a v0.8 estão em inglês: são histórico e não se reescrevem.

## O que é este projeto

**Declaro Meu Tempo** (repositório `declaro-tempo`): um PWA estático em que cada
pessoa declara, por si mesma, os horários e períodos em que prevê estar disponível
no trabalho ("de 01/09 a 31/12, seg–sex, 9h–15h"). Leitores (`leitor`) veem as
declarações das suas unidades, e administradores (`adm`) veem tudo, em tabelas e
gráficos. É uma **previsão de disponibilidade, não um ponto eletrônico**: nada é
conferido contra o que aconteceu, e nada alimenta métrica de desempenho.

Até a v0.8 chamou-se "Declaração de disponibilidade" e viveu em
[`ErnyBSB/declare-time`](https://github.com/ErnyBSB/declare-time), hoje arquivado.
**As issues e PRs de número até #32 citados nos documentos são de lá**; a numeração
recomeçou no repositório novo.

Herda toda a arquitetura do **Programa de Resultados — COBIB**
([`ErnyBSB/resultados`](https://github.com/ErnyBSB/resultados), v1.16), cuja seção
"Horários" virou aqui o programa inteiro. Quando os documentos falam em "projeto de
origem", é desse repositório.

**Não há servidor.** O estado compartilhado fica numa pasta de rede do Windows,
acessada pelo navegador por meio da **File System Access API**; por isso o
aplicativo roda só no Chrome e no Edge. O uso offline vem de um service worker
cache-first.

**`project_design.md` é a fonte da verdade** para arquitetura, modelo de dados,
papéis, decisões e roteiro. Leia-o antes de qualquer mudança que mexa nisso. A
seção 12 registra as decisões já tomadas; não as reabra sem um argumento novo. A
seção 13 é o roteiro.

## Objetivos (em ordem de prioridade)

1. **Zero infraestrutura.** Sem backend, banco de dados, npm, etapa de build ou
   CDN. O aplicativo precisa rodar abrindo `code/index.html` direto da pasta.
   Qualquer proposta que acrescente servidor, gerenciador de pacotes ou dependência
   de rede está fora do escopo — diga isso antes de implementar.
2. **Sem conflito de gravação, por construção.** *Um arquivo, um só escritor*: cada
   pessoa grava só o próprio `declaracoes/<slug>.json`, sem exceção — nem para
   administradores. Só `adm` grava o `config.json`; como pode haver vários
   administradores, `gravarConfig` relê o arquivo em modo estrito e recusa a
   gravação se `atualizadoEm` mudou desde que a tela o carregou.
3. **Dados compatíveis com o passado.** Arquivos já gravados na pasta compartilhada
   precisam continuar carregando. Trate campo ausente pela ausência
   (`(x && x.campo) || padrão`), nunca migrando ou regravando o arquivo de uma
   pessoa. Nunca renomeie uma chave persistida (campos JSON, nomes de papéis, chaves
   do `localStorage`) — acrescente uma chave nova, versionada.
4. **Legível para quem não é da área técnica.** Interface, comentários e documentos
   em **pt-BR**. A tela explica as recusas em vez de falhar calada.
5. **Acessível e quase sem dependências.** Elementos nativos primeiro. **Não há
   biblioteca nenhuma**: os gráficos do painel são HTML/CSS puro (decisão 12.6) —
   não traga de volta o ECharts nem outra biblioteca de gráficos.

## Estrutura

```
code/                    o aplicativo — uma cópia só, sempre a versão atual
  index.html             interface + toda a lógica + CSS embutido, num arquivo só
  versao.js              o ÚNICO lugar onde a versão é escrita
  catalogo.js            semente estática: dias da semana, tipos de ausência,
                         exemplo de caminho da pasta; unidades e senhas vazias
                         (repositório público)
  rede.js                acesso à pasta de rede (podado do projeto de origem)
  sw.js                  service worker; chave do cache "dt-" + versao
  manifest.webmanifest   metadados do PWA (o nome aparece no ícone instalado)
  gerar-senha.html       ferramenta avulsa: sha256("nome|senha")
  icons/                 icon-192.png, icon-512.png — necessários para instalar offline
  LEIA-ME.txt            a documentação de verdade: o que mudou e POR QUÊ
testes/                  testes de ponta a ponta no navegador (ferramenta de quem
                         mantém, não parte do aplicativo)
  rodar.mjs              executor: node testes/rodar.mjs [prefixos…]
  navegador.mjs          apoio: servidor estático, Chrome sem tela via CDP, pasta no OPFS
  NN-*.mjs               uma suíte por área (01-entrada … 06-contrato)
  REGISTRO.md            execução registrada de cada tag publicada
  resultados/            saída completa de cada execução registrada
prova-de-uso/ROTEIRO.md  roteiro da prova de uso (fase 5): preparação, permissões
                         icacls, tarefas
.github/ISSUE_TEMPLATE/  modelo de issue "Prova de uso: relato" (lembrete de privacidade)
project_design.md        referência estruturada: esquemas, papéis, decisões, roteiro
```

Tudo fica num único `index.html` porque módulos (`type=module`) não carregam a
partir de `file://`, e não há etapa de build para empacotá-los.

**Os dois documentos têm propósitos diferentes.** `code/LEIA-ME.txt` argumenta cada
decisão em prosa, em tom de conversa — atualize-o a cada mudança de comportamento.
`project_design.md` é a referência estruturada; atualize-o quando a arquitetura, o
modelo de dados, os papéis ou as limitações mudarem.

## Convenções de trabalho

**Versionamento só pelo git.** `code/` é sempre a versão atual; cada versão
publicada é uma **tag** (`v0.1`, …, `v1.0`, `v1.1`…). Nunca crie pastas por versão
nem cópias datadas. A v1.0 é o primeiro uso em produção; depois dela, o esquema da
seção 5 está congelado.

**Publicar uma versão exige duas edições, e elas precisam concordar:**
1. `code/versao.js` → `versao`, `rotulo` e `atualizado` (data escrita à mão). A
   etiqueta do rail, a janela Sobre e a chave do cache do service worker derivam
   dele — nunca escreva o número em outro lugar.
2. a tag do repositório (`git tag -a vX.Y`).

**Branches e commits.** Uma branch por versão em andamento, integrada ao `main` por
PR (com **merge commit**, para não perder o histórico) e marcada com a tag no `main`.
Commits seguem o Conventional Commits, **em português**: o assunto descreve a
mudança que o usuário vê; o corpo argumenta o porquê; o rodapé traz `Refs: #<issue>`
e `Assisted-by: IA`. PRs têm as seções **O quê**, **Por quê** e **Como testar**, e
terminam com `Closes #<issue>`.

**Estilo de código.** `"use strict"`, identificadores em pt-BR (`declaracao`,
`garantirPermissao`, `atualizarArquivoPessoa`), faixas `/* ===== TÍTULO ===== */`
no início das seções. Comentários explicam o *porquê*, no tom da LEIA-ME. Checagens
de papel passam pelos predicados `ehPessoa()`, `ehLeitor()`, `ehAdm()`, `ehMeu(d)` e
`unidadesVisiveis()` — nunca comparações de texto de papel no meio do código.
Esconder um item do rail não é controle de acesso; o recorte fica em `podeVer(v)`.

**Estilo visual.** Propriedades CSS em `:root` com nomes em pt-BR (`--tinta`,
`--papel`, `--ficha`, `--verde`, `--grafite`, `--linha`, `--raio`). As cores têm
significado — **verde é declaração vigente** — então não o use como enfeite. O
dourado saiu na v0.5 junto com a jornada (decisão 12.9). Mantenha
`[hidden]{display:none !important}`.

## Modelo do domínio

- **Papéis são cumulativos e vêm do `config.json`**, nunca de uma escolha na tela.
  Todo cadastrado é `pessoa` e declara só por si. `leitor` (lista de unidades) e
  `adm` (`true`) são atributos dentro de `pessoas[nome]`. Vários leitores por
  unidade, vários administradores. O último `adm` não pode ser removido.
- **Declaração** (`project_design.md` 5.1): período `de`…`ate` (`ate: null` = sem
  fim), dias da semana `dias` (0 = domingo), uma faixa `inicio`–`fim` por declaração
  (turno partido são duas declarações; atravessar a meia-noite é recusado),
  `unidade` gravada como retrato. **Não há jornada** desde a v0.5 (12.9): nunca a
  peça; `cargaHoraria`/`carga` em arquivos antigos ficam intocados e sem leitura.
- **Colisão** só quando período + dia da semana + hora se cruzam.
- **Ciclo de vida**: excluir só antes de começar (`de` > hoje) **ou no dia em que foi
  declarada** (`criadaEm` hoje, v0.8, 12.4 revista); encerrar uma vigente pondo `ate`
  em hoje ou depois (só encurta); nunca editar uma que começou; "substituir" encerra
  a vigente na véspera do novo início (hoje ou depois) e cria a nova numa só
  gravação. A situação (vigente/futura/encerrada) é calculada contra hoje, nunca
  gravada.
- **Painel** (leitor e adm, recortado por `unidadesVisiveis()`): as declarações são
  atribuídas ao **dono do arquivo** (o nome do cadastro cujo slug é o nome do
  arquivo), nunca ao campo `pessoa` lá dentro; declarações inválidas são puladas,
  não fatais, e continuam no arquivo. A tabela filtra pela unidade gravada; "sem
  declaração", pela unidade atual do cadastro; arquivos ilegíveis nunca contam como
  "sem declaração". Células do CSV que começam com `= + - @` ganham `'` na frente.
- **Gráficos** (painel, v0.4) são HTML/CSS alimentados pelo mesmo `recortePainel()`
  da tabela e do CSV — nunca uma segunda conta. A cobertura conta pessoas distintas
  por dia da semana e hora (hora inteira), numa escala de um só verde validada, com
  zero em cinza; as barras são de tinta neutra, uma série; horas semanais só no modo
  data. As dicas usam `textContent` e aparecem também no foco do teclado.
- **Ausências** (`ausencias`, v0.5) ficam no arquivo da pessoa: `{id, pessoa,
  unidade, tipo, de, ate, obs, criadaEm}`, `tipo` vindo das chaves de
  `TIPOS_AUSENCIA` em `catalogo.js` (nunca renomeie uma chave), `ate` obrigatório,
  sem sobreposição entre ausências da mesma pessoa, mesmo ciclo de encerrar/excluir
  das declarações (sem a exceção do dia). Nunca encerram declarações. No painel,
  tiram a pessoa da cobertura só na data de referência (a grade mantém as horas,
  para que as descobertas apareçam como zero); no modo mês ninguém é retirado.
- **Nome reservado**, sem uso na 1.0: a chave `realizados`.

## Conferindo as mudanças

**Rode os testes no navegador antes de todo PR:** `node testes/rodar.mjs` (todas as
suítes) ou `node testes/rodar.mjs 03` (por prefixo). Precisam de Node 22+ e Chrome
ou Edge (`CHROME=/caminho` se não for encontrado), usam só módulos nativos do Node
— nunca acrescente pacotes npm — e falham com qualquer erro no console da página.
Servem `code/` numa porta local, dirigem o Chrome sem tela pelo DevTools Protocol e
trocam só o seletor de pasta e o pedido de permissão: a pasta de rede é o OPFS do
navegador, então o `rede.js` de verdade lê e grava arquivos de verdade.
`FOTOS=/alguma/pasta` salva capturas de tela para conferência visual; olhe-as depois
de qualquer mudança de leiaute.

**Quando uma versão ganha tag, registre a execução dos testes** em
`testes/REGISTRO.md`: rode as suítes sobre o código da tag, salve a saída em
`testes/resultados/<aaaa-mm-dd>_<versão>.txt` e acrescente uma linha (data, versão,
commit, `node -v`, versão do navegador, conferências, falhas). Arquivos avulsos
`resultado-testes*.txt` são ignorados pelo git.

**O contrato dos arquivos está congelado desde a v1.0** (`project_design.md` 5.6,
conferido por `testes/06-contrato.mjs`): só se acrescentam chaves, nunca se
renomeia, muda o tipo ou reaproveita uma; nunca se migram arquivos; e um arquivo de
`formato` > 1 nunca é gravado por cima. Uma chave nova exige atualizar a tabela 5.6
e essa suíte no mesmo PR.

**Quando o comportamento muda, os testes mudam no mesmo PR.** Acrescente
conferências à suíte da área mexida, ou uma suíte nova `NN-nome.mjs` que exporte
`nome`, `opcoes` (`downloads`, `foco`) e uma `async function (t)` padrão. Os dados
de teste usam só o elenco inventado.

Os testes não substituem a conferência à mão: abra `code/index.html` no Chrome
contra uma pasta de teste local (`pasta-teste/`, já no `.gitignore`) com
`config.json` e `declaracoes/`, em várias abas, alternando entre pessoa, leitor e
adm, e com dois administradores mexendo no cadastro. Ao mexer no `sw.js` ou em
arquivos do cache, recarregue forçando (Ctrl+Shift+R) — uma versão velha em
`versao.js` serve o cache anterior e esconde a mudança.

## Não faça

- Acrescentar ferramentas de build, gerenciadores de pacotes, frameworks ou recursos
  remotos — no aplicativo ou em `testes/` (só módulos nativos do Node).
- Criar pastas por versão ou cópias datadas de arquivos — versões são tags.
- Renomear chaves persistidas ou nomes de papéis, ou reaproveitar as reservadas.
- Escrever o número da versão fora do `versao.js`, ou trocá-lo sem marcar a tag.
- Regravar ou migrar arquivos JSON que os usuários já têm na pasta de rede.
- Deixar alguém gravar o arquivo de declaração de outra pessoa, administradores
  inclusive.
- Transformar as horas declaradas em métrica, ranking ou conferência de
  cumprimento.
- Escrever em inglês na interface, nos comentários, na LEIA-ME, nos commits, nos PRs
  ou nas issues.
- Integrar PR com "Squash" ou "Rebase": apagam o histórico que as tags e o
  `testes/REGISTRO.md` citam.
- **Versionar dado interno.** O repositório é público: nenhum nome de pessoa real,
  nenhuma senha (nem em hash), nenhum `config.json`, nenhum arquivo de declaração.
  `catalogo.js` é uma semente vazia. Exemplos usam o elenco inventado (Helena
  Barros, Ana Beatriz Nunes, Carlos Eduardo Prado, Marina Duarte, Roberto Aguiar).
