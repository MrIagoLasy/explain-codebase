# A estrutura do documento, seção por seção

O documento vai do alto nível ao baixo, e cada seção assume só o que veio antes.
Abaixo, o que entra em cada uma e o que faz ela funcionar de verdade.

## Abertura

Três parágrafos, no máximo:

- o que o documento cobre
- que a ordem é proposital, e que parar na metade ainda deixa o leitor com a
  metade que importa
- o índice com links

Sem "este documento tem como objetivo". Vá direto.

## 1. O que esse sistema é

Comece por **uma frase** que caiba na cabeça. Se você não consegue resumir em uma
frase, ainda não entendeu.

Depois:

- **Quem é quem no domínio.** Nomes de pessoas, produtos, papéis. Em sistema real
  os nomes do código confundem (uma classe chamada X que faz Y porque o Y se
  chamava X há um ano) — explique isso aqui, e o resto do documento fica legível.
- **O problema de negócio**, com o fluxo em ASCII se houver funil/pipeline.
- **As restrições externas.** Limite de API, regra de plataforma, compliance,
  janela de tempo. É de onde nasce a maior parte da complexidade estranha, e
  citá-las cedo faz o leitor perdoar o código depois.

Se o sistema tem um requisito não-funcional que molda tudo (latência, custo, não
parecer robô, auditoria), diga aqui e mostre numa tabela onde ele vira código.

## 2. O mapa

**Um diagrama ASCII**, e ele precisa caber numa tela. Se não couber, você está
detalhando demais para esta altura.

Mostre as peças e as **setas com o que trafega** (não só "conversa com"). Depois
do diagrama, responda:

- qual é a **regra que organiza** a comunicação (fila? banco? HTTP direto?)
- quem chama quem, e o que **não** chama ninguém
- o que roda onde (servidor, edge, cron, container) e com qual gatilho

Aqui é o lugar de matar mal-entendidos estruturais: se a documentação diz push e
a implementação é polling, diga agora, com a evidência.

## 3. As pastas

Árvore de **um nível ou dois**, com uma linha de comentário por pasta. Alto nível:
"isso aqui é para isso". Nada de arquivo individual ainda — isso é o apêndice A.

Marque o que é o coração. E marque as **armadilhas de leitura**: a pasta que
parece morta mas tem partes vivas, o arquivo cujo nome engana.

## 4. A jornada de um <evento>

**A seção mais importante do documento.** Se o leitor só ler uma, é essa.

Escolha o evento mais representativo do sistema (uma mensagem chegando, um request
de checkout, um job noturno) e siga **passo a passo, atravessando todas as
camadas**. Numere os passos.

O que faz essa seção funcionar:

- **Código real nos pontos de decisão.** A query que define quem é atendido, o
  `if` que decide o caminho. Não parafraseie a parte que importa.
- **Explique a query linha a linha** se ela for central. Query boa é o sistema
  inteiro em miniatura.
- **Nomeie os conceitos aqui** e reuse depois. Se você chamar de "cursor de
  idempotência", use esse nome no resto do documento.
- **Termine com o fluxo condensado** em ASCII, do início ao fim. É o resumo que o
  leitor volta para consultar.

## 5 a N. Um subsistema por seção

Ordene por **dependência**, não por importância: o que precisa ser entendido antes
vem antes.

Padrão que funciona em cada uma:

1. o problema que esse subsistema resolve
2. como resolve (mecanismo)
3. a decisão não óbvia, com o porquê
4. o incidente que a gerou, se houver

Seções que quase sempre valem a pena:

- **Como o sistema decide** (regras de negócio, IA, motor de decisão)
- **O que impede o desastre** — os invariantes. Numere-os. Cada um com o incidente
  que o criou. Costuma ser a seção mais útil do documento inteiro.
- **O que roda sozinho** — jobs, crons, scans. Tabela com gatilho e frequência.

## N+1. O modelo de dados

**Depois** de o leitor já ter visto as tabelas em uso, não antes. Aí elas fazem
sentido sozinhas.

Não documente coluna por coluna: isso é trabalho do schema. Documente:

- os **domínios** de tabelas, em blocos
- as **3 ou 4 tabelas centrais**, com as colunas que carregam significado não óbvio
- a **semântica traiçoeira**: coluna cujo NULL significa alguma coisa, enum com
  valor que engana, campo cujo nome mente
- as **fontes de verdade que não são o que parecem** ("venda não é a tabela
  vendas, é X")
- o **problema não resolvido** do modelo, se houver (chave natural que não é
  confiável, duplicação estrutural)

## N+2. A costura

Onde as partes se tocam — e brigam. Em sistema com mais de um repo, é a seção mais
valiosa.

Procure e liste:

- mesma função/regra definida em dois lugares
- dois escritores na mesma coluna sem coordenação
- um componente que sobrescreve objeto de outro
- princípio declarado no README e violado no código
- acoplamento por string livre (prefixo de mensagem, nome de arquivo)

Para cada um: o que acontece se divergirem, e o custo real.

## N+3. A história

Tabela de fases, com data e o que mudou. Depois, as **decisões de design** com o
porquê — se existir spec/ADR no repo, é aqui que ele entra.

Isso responde a pergunta que todo mundo faz lendo código estranho: "por que
fizeram assim?". E frequentemente a resposta é boa.

## N+4. O que está aberto

Honesto e sem drama. Agrupe em: **riscos de segurança**, **buracos de
arquitetura**, **dívidas de teste**, **pontos a verificar**.

Escreva como fato observado, não como acusação. E marque o que é decisão
consciente (documentada no repo) contra o que parece esquecimento — a diferença
importa para quem for mexer.

## Apêndice A: a árvore completa

Agora sim, arquivo por arquivo, com legenda de estado:

```
🟢 vivo e crítico · 🔵 vivo · ⚪ legado · 🔴 morto (stub)
```

Uma linha de comentário por arquivo relevante, com contagem de linhas nos grandes.
Agrupe visualmente por bloco. Esta é a peça que o leitor volta a consultar por
meses.

## Apêndice B: a ordem de leitura

Três trilhas:

- **rápida** — 8 a 12 arquivos, o essencial, com estimativa de tempo
- **completa** — por blocos temáticos, na ordem de dependência
- **por pergunta** — tabela "quero entender X → leia Y"

O detalhe que faz diferença: **cada arquivo vem com o que procurar dentro dele**.
Abrir um arquivo de 800 linhas sem alvo não rende. "Leia `supervisor.py`" é ruim;
"leia `supervisor.py::_run` (:764) — o loop, as filas, a concorrência" é útil.

E feche com **o que NÃO ler**: código morto, migrações já executadas, docs cujo
nome engana. Economizar o tempo do leitor é metade do valor do documento.
