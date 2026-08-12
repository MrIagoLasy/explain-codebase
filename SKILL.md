---
name: explain-codebase
description: Estuda um repositório inteiro (ou vários que se conversam) e escreve um documento que ensina o sistema como NARRATIVA, em fluxos encadeados, não como catálogo de pastas e funções. Lê todos os arquivos via subagentes em paralelo e produz duas partes - a história de como o sistema funciona, e depois o mapa de navegação ancorado nessa história. Use quando o pedido for "me ajuda a estudar esse código", "explica esse projeto", "preciso entender esse repo", "assumi esse sistema e não sei nada dele", "documenta a arquitetura", "onboarding nesse codebase" — mesmo sem citar a skill. Serve para repo herdado e para revisitar projeto próprio esquecido.
argument-hint: [caminho do repo] [+ outros repos relacionados]
---

# explain-codebase

Produz **um documento** (`fluxos.md` na raiz do projeto, salvo pedido diferente) que
faz alguém sentar, ler de ponta a ponta, e sair entendendo o sistema.

Não é README, não é changelog, não é documentação de referência. É **aula escrita**.

## A descoberta que define esta skill

Documentação organizada como catálogo — árvore de pastas, lista de módulos,
inventário de funções — **não gruda**. Feedback real de quem leu:

> "quase dormi lendo... eram só informações soltas jogadas. Ao ler o próximo item já
> esqueci o anterior, diferente de história, onde cada coisa que vem depois dá
> sentido ao anterior."

O motivo é simples: num catálogo cada item é independente, então o leitor precisa
segurar tudo na memória sem apoio. Numa narrativa, cada parte nova **dá sentido
retroativo** à anterior, e a memória trabalha a favor.

Então a regra desta skill: **conte fluxos, não estruture inventários.**

## O formato: duas partes

### Parte 1 — os fluxos (é o documento de verdade)

Uma sequência de histórias. Cada uma segue **uma coisa acontecendo** no sistema, do
começo ao fim: um request chegando, um job rodando, um usuário clicando.

Regras que fazem isso funcionar:

- **Zero listas de arquivos, zero catálogo de funções, zero árvore de pastas.**
  Nome de arquivo aparece como âncora curta no fim do parágrafo (`arquivo.py:123`),
  nunca como cabeçalho ou lista.
- **Cada seção assume o que veio antes e NADA do que vem depois.** Ver abaixo — é a
  regra mais violada e a que mais estraga o documento.
- **Cada conceito nasce quando resolve um problema que o leitor acabou de ver.**
  Exceto vocabulário do domínio, que vai na abertura (ver abaixo).
- **Os 3 ou 4 primeiros fluxos são a coluna vertebral.** Com eles o leitor já
  entende o sistema; os demais são bordas e casos especiais.
- **Fluxos paralelos viram histórias separadas**, cada uma contada do próprio
  começo. Não tente desenhar concorrência numa narrativa linear.
- **Incidente vira fluxo.** "Uma cadência quase matou um número" ensina mais sobre
  as travas do que qualquer descrição delas.
- **Feche com o diagrama do sistema inteiro.** No fim ele é reconhecimento, não
  introdução — o leitor já viu cada peça funcionando.
- **E com as 4 ou 5 ideias que sustentam tudo**, numeradas. É o que fica.

Detalhe de como escrever cada tipo de fluxo: `references/narrativa.md`.

### As três regras que mais custam quando quebradas

Estas nasceram de erro real cometido escrevendo um documento destes. Cada uma passou
despercebida na revisão e foi o leitor que apontou.

**1. Pseudocódigo, nunca código literal.** A informação é a decisão, não a sintaxe.

```
RUIM:  const enc = req.headers.get("content-encoding")?.toLowerCase();
       if (enc === "gzip") { const ds = new DecompressionStream("gzip"); ... }

BOM:   o corpo pode vir comprimido, e o runtime só descomprime nas respostas
       que ele busca, não nas requisições que recebe. Sem esse tratamento, a
       leitura do JSON quebra antes de começar. (index.ts:16)
```

Use bloco de pseudocódigo quando a **estrutura da decisão** importa (uma query com
seis condições, uma ordem de verificações, uma precedência). Use prosa quando só o
**efeito** importa. Nunca cole o original: quem quiser lê pela âncora.

**2. Vocabulário do domínio vai na abertura, não no meio.** A regra "conceito nasce
quando resolve um problema" vale para **mecanismo**, não para **substantivo do
domínio**.

Se o sistema tem duas linhas de WhatsApp, dois tipos de usuário, três ambientes — isso
é fato do mundo, não detalhe de implementação. Apresentar no meio do Fluxo 1 obriga o
leitor a reinterpretar o que já leu.

Teste: se o leitor precisar voltar para reler um parágrafo anterior à luz do que
acabou de aprender, o conceito entrou tarde.

**3. Zero referência para frente.** Referência para **trás** é ótima (reforça o
encadeamento e recompensa quem leu em ordem). Para **frente** é dívida: manda o leitor
guardar uma pergunta que você não vai responder agora.

```
RUIM:  "tem um nome que mente, e isso está no Fluxo 7"
BOM:   explica ali, em duas frases, por que o nome mente

RUIM:  "regras rígidas, que o Fluxo 3 detalha"
BOM:   "regras rígidas de quando e como você pode falar"
```

Quando o gancho for genuinamente útil, deixe **sem o número**: "isso tem uma
consequência que aparece mais adiante" sinaliza sem criar dívida específica.

Ao terminar o documento, **audite**: procure toda menção a seção futura (`grep -n
"Fluxo [0-9]"` ou equivalente) e resolva uma a uma. Na Parte 2 as referências são
permitidas — ali é índice de consulta, não narrativa.

### Parte 2 — o mapa de navegação

Depois de entender, a pessoa precisa **achar as coisas**. Mas o mapa não pode ser
uma árvore de pastas — isso reintroduz o problema do catálogo.

A solução: **ancore o mapa nos fluxos que o leitor acabou de ler**.

- **Onde cada fluxo acontece** — para cada história da Parte 1, os arquivos que ela
  atravessa, na ordem em que ela os atravessa, com `arquivo:linha` nos pontos-chave.
  Não é informação nova: é reconhecimento.
- **A hierarquia real** — núcleo (se quebrar, para tudo), o que muda comportamento
  sem mudar código, motores independentes, suporte, satélites, legado com partes
  vivas, lixo declarado.
- **As armadilhas de nome** — arquivo cujo nome mente, doc desatualizado, pasta que
  parece morta e não é.
- **Como as peças se comunicam** — por banco? HTTP? import? Isso quase nunca
  corresponde à estrutura de pastas, e é o que mais confunde.
- **Roteiros de leitura** por objetivo (entender o coração, mexer em algo, entender
  as decisões).
- **Mapa de sintomas** — tabela "quando X estiver errado, olhe Y". É a seção que a
  pessoa mais volta a consultar.

Detalhe de cada seção: `references/mapa-navegacao.md`.

## O processo

### 1. Reconhecimento (você mesmo, rápido)

```bash
git ls-files | wc -l          # tamanho real
git ls-files | head -100      # a forma do repo
git log --oneline -15         # o que mudou por último = o que está vivo
ls docs/ *.md 2>/dev/null     # docs existentes
```

Leia os docs de arquitetura primeiro — dão vocabulário, e quando estão
desatualizados isso já é achado. Se houver repo irmão que compartilha banco ou API,
inclua: sistema que se conversa se estuda junto.

### 2. Leitura completa via subagentes em paralelo

Você não lê sozinho: estouraria o contexto e perderia o fio. Delegue por **camada de
responsabilidade**, todos disparados na mesma mensagem. De 4 a 8 frentes.

O prompt de cada um precisa pedir Read completo (nunca grep), `arquivo:linha`, e
explicitamente **os comentários que contam incidentes**. Modelo pronto e critério de
fatiamento: `references/leitura-paralela.md`.

### 3. Achar as histórias

Este é o passo que os relatórios não fazem por você. Com tudo em mãos, pergunte:

- Qual é **o evento principal** do sistema? Esse é o Fluxo 1.
- O que acontece quando o Fluxo 1 **não dá conta**? Esse é o 2.
- Que **restrição externa** molda o desenho? Vira um fluxo próprio.
- Qual **incidente** deixou mais cicatriz no código? Vira fluxo.
- O que roda **em paralelo** ao principal? Um fluxo cada.
- O que acontece quando **ninguém está olhando**? Costuma fechar bem.

Ordene por dependência narrativa: cada fluxo pode usar o que o anterior estabeleceu,
e nada do que vem depois.

### 4. Verificar antes de escrever

Subagente erra com confiança. Cheque você mesmo toda afirmação que muda o desenho
("não existe push, é polling"), contradiz outro relatório, ou contradiz a
documentação do repo. Um Read resolve.

Contradição entre relatórios costuma ser achado real: duas implementações da mesma
coisa.

**E cuidado com a hierarquia das fontes.** Nem toda evidência vale o mesmo:

```
o que está rodando   ← deploy, endpoint respondendo, log, tabela com dado
     vence
o código no repo     ← pode não ser o que foi buildado
     vence
o README / doc       ← declara INTENÇÃO, não estado
```

Um README dizendo "substituímos X por completo" não prova que X morreu. Se der para
checar o que está no ar (uma URL, uma listagem de deploy, uma consulta), cheque —
principalmente antes de escrever que algo está morto ou vivo.

Um erro real cometido nesta skill: um auditor citou o README dizendo que o painel
antigo tinha sido substituído, eu aceitei e reescrevi a seção. O worker tinha deploy
três dias posterior à data da substituição e respondia normalmente. A versão original
estava certa; a "correção" é que estava errada.

### 5. Escrever e entregar

Escreva o documento, depois resuma em poucas linhas o que ele cobre e as 2 ou 3
descobertas que mudam a leitura do sistema.

### 6. Auditar o que você escreveu (não é opcional)

Documento escrito a partir de relatório de subagente sai **plausível e parcialmente
falso**. Você não vai perceber relendo: o texto é coerente, e é justamente por isso que
o erro passa.

Lance auditores independentes, um por bloco de fluxos, com uma instrução diferente da
que você usou para ler:

> TAREFA DE AUDITORIA. Verifique afirmações contra o código real, não escreva
> documentação. Leia as linhas X a Y do documento. Ele foi escrito a partir de
> relatórios de terceiros e PODE CONTER ERROS; sua missão é achá-los.
>
> Para cada afirmação técnica, abra o código e confira. Liste APENAS: ERROS (citação do
> doc, o que o código diz, arquivo:linha), IMPRECISÕES, NÚMEROS ERRADOS, CITAÇÕES QUE
> NÃO SÃO LITERAIS, OMISSÕES CRÍTICAS. Se estiver correto, não liste.

Enumere no prompt os pontos específicos a conferir — cada número, cada citação entre
aspas, cada `arquivo:linha`. Auditor sem alvo devolve resumo; auditor com lista devolve
correções.

O que mais aparece errado, por frequência:

1. **Números inventados ou trocados** ("duas semanas" onde o código diz "15 dias").
2. **Citações formatadas como literais** que são paráfrase. Se está entre aspas ou em
   bloco, tem que ser palavra por palavra.
3. **Condições faltando** numa consulta ou numa sequência de verificações.
4. **Absolutos sem a exceção** ("o cliente não recebe nada" quando existe um caso em
   que recebe).
5. **`arquivo:linha` deslocado** (decorador antes da função, código que andou).
6. **Escopo inflado** ("só três operações fazem X" quando é "só três *das ferramentas*").
7. **Gate omitido** — a flag que decide se aquele motor sequer envia.

E audite também a estrutura: `grep` por referências a seções futuras (ver a regra 3
acima), e teste se cada caminho de arquivo citado existe de verdade.

### 7. Decidir o nível de anonimização

**Pergunte ao usuário para onde o documento vai** antes de escrever, ou entregue a
versão interna e ofereça a sanitizada.

| Destino | O que pode ficar |
|---|---|
| Interno (o repo, o time, o cliente) | tudo. Nome real ajuda a bater com o código |
| Portfólio, blog, exemplo público | nada identificável: papéis no lugar de nomes |

O que costuma vazar sem querer, em ordem de risco: **credencial** (token, chave, id
de instância, string de conexão), **endpoint privado** (URL de webhook, subdomínio
interno, caminho secreto), **dado pessoal** (telefone, e-mail, nome de cliente final
que aparece em post-mortem de código), **identidade comercial** (nome da empresa, do
produto, valores de tabela, métricas de conversão).

Credencial e endpoint privado **nunca** entram, nem na versão interna: documento
circula por canais que o repositório não controla. Nome e métrica são decisão do
usuário.

Ao sanitizar, troque por **papel**, não por letra: "o dono do produto" e "o vendedor
humano" preservam a narrativa; "a Pessoa A" e "a Pessoa B" a destroem.

E audite antes de entregar qualquer versão pública — `grep` por nomes próprios,
domínios, telefones, valores monetários e identificadores longos.

## Método Feynman, na prática

**Explique pelo problema que o código resolve, não pela sua implementação.**

| Paráfrase (ruim) | Explicação (bom) |
|---|---|
| "sorteia entre 60 e 180 segundos" | "gente de verdade não responde em 800ms; instantâneo denuncia automação" |
| "usa advisory lock" | "dois processos podem subir juntos; sem isso o cliente recebe em dobro" |
| "a coluna aceita NULL" | "NULL aqui significa IA; só o literal 'humano' marca pessoa" |

Teste: se a frase descreve *o que o código faz*, é paráfrase e não vale o espaço. Se
descreve *por que ele precisa existir*, é explicação.

Regras de escrita completas: `references/estilo.md`.

Três hábitos que valem mais que o resto:

1. **Caça o post-mortem.** Em projeto de produção os comentários são cicatrizes, não
   documentação. Cite com o número medido: "derrubou o número em 23 minutos" vale
   mais que "causou problemas".
2. **Nomeie a armadilha.** Toda base tem uma coisa cujo nome mente. Encontre e dê
   destaque.
3. **Diga o que NÃO ler.** Economizar o tempo do leitor é metade do valor.

## Escala

- **Repo pequeno** (<40 arquivos): leia você mesmo, pule subagentes, 3 ou 4 fluxos.
- **Médio** (40 a 300): 4 a 7 subagentes, 8 a 12 fluxos.
- **Grande** ou múltiplos repos: fatie por camada e por repo; considere duas rodadas.

Se o usuário pedir só uma parte, estude só ela — mas mantenha um fluxo que mostre
onde ela se encaixa no todo.

## Referências

- `references/narrativa.md` — como construir cada fluxo, tipos que funcionam, erros comuns
- `references/mapa-navegacao.md` — a Parte 2, seção por seção
- `references/leitura-paralela.md` — fatiamento, prompt de subagente, o que sempre pedir
- `references/estilo.md` — regras de escrita, diagramas, o que evitar
