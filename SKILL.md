---
name: explain-codebase
description: Estuda um repositório inteiro (ou vários que se conversam) e escreve um documento único que ensina o sistema do alto nível ao baixo nível, com método Feynman. Lê TODOS os arquivos via subagentes em paralelo, monta diagrama, árvore comentada de pastas, o fluxo ponta a ponta, e a ordem ideal de leitura do código. Use quando o pedido for "me ajuda a estudar esse código", "explica esse projeto", "preciso entender esse repo", "assumi esse sistema e não sei nada dele", "documenta a arquitetura", "onboarding nesse codebase", "o que faz cada pasta aqui" — mesmo sem citar a skill. Serve tanto para repo herdado quanto para revisitar projeto próprio esquecido.
argument-hint: [caminho do repo] [+ outros repos relacionados]
---

# explain-codebase

Produz **um documento** (`explicacao.md` na raiz do projeto, salvo o usuário pedir
outro caminho) que faz alguém sentar uma hora, ler até o fim, e sair sabendo
navegar o sistema sozinho.

Não é README, não é changelog, não é lista de arquivos. É **aula escrita**.

## O princípio que organiza tudo

**Cada seção assume o que veio antes e nada do que vem depois.**

Quem parar na metade entendeu a metade de cima, que é a que importa para navegar.
Isso proíbe: falar de uma tabela antes de mostrar quem escreve nela, citar um
módulo antes de dizer para que serve, usar um termo do domínio antes de definir.

## O fluxo

### 1. Reconhecimento (rápido, você mesmo)

Antes de delegar qualquer coisa:

```bash
git ls-files | wc -l          # tamanho real
git ls-files | head -100      # a forma do repo
git log --oneline -15         # o que mudou por último = o que está vivo
ls docs/ *.md 2>/dev/null     # docs existentes
```

Leia os docs de arquitetura que existirem **primeiro**. Eles dão vocabulário — e
com frequência estão desatualizados, o que já é um achado a registrar.

Se o usuário citou mais de um repo (ou você descobrir que existe um irmão que
consome o mesmo banco/API), inclua todos. Sistema que se conversa se estuda junto.

### 2. Leitura completa via subagentes em paralelo

Você não lê o repo sozinho — estouraria o contexto e você perderia o fio.
Delegue por **camada**, não por pasta alfabética. Um subagente por frente, todos
disparados na mesma mensagem.

Fatiamento que costuma funcionar (adapte ao repo):

| Frente | Escopo |
|---|---|
| Núcleo | o que roda em produção hoje, o "coração" |
| Camadas de saída | integrações externas, clientes de API, envio |
| Infra | workers, jobs, containers, deploy, config |
| Legado | o que foi substituído — **e o que dele ainda está vivo** |
| Dados | schema, migrações, triggers, views |
| Interface | frontend, painéis, CLI |
| História | docs de design, specs, scripts operacionais |

O prompt de cada subagente precisa pedir: **Read completo, sem grep**, arquivo por
arquivo, com `arquivo:linha` nas citações, e explicitamente **os comentários que
contam incidentes**. Modelo pronto e o resto da técnica: `references/leitura-paralela.md`.

### 3. Reconciliar o que voltou

Os relatórios vão se contradizer, e é aí que está o ouro. Procure ativamente:

- **doc que mente** — arquitetura documentada que não é a implementada
- **código morto que parece vivo** (e o inverso, que é pior)
- **regra duplicada** em dois lugares, sincronizada à mão
- **nome que engana** — arquivo/função cujo nome diz outra coisa
- **a mesma coisa resolvida de jeitos incompatíveis** no mesmo sistema

Se um relatório afirmar algo estrutural que muda o desenho (ex: "não existe push,
é polling"), **verifique você mesmo** antes de escrever. Subagente erra.

### 4. Escrever o documento

Estrutura padrão, do alto para o baixo. Detalhe de cada seção e o que faz cada uma
funcionar: `references/estrutura-do-doc.md`.

```
 1. O que esse sistema é          domínio, quem é quem, o problema real
 2. O mapa                        diagrama ASCII + como as peças conversam
 3. As pastas                     árvore comentada, uma linha por pasta
 4. A jornada de um <evento>      ← A SEÇÃO MAIS IMPORTANTE. ponta a ponta
 5..N. Cada subsistema            um por seção, na ordem de dependência
 N+1. O modelo de dados           depois de já ter visto as tabelas em uso
 N+2. A costura                   onde as partes se tocam e brigam
 N+3. A história                  em que ordem nasceu, e por quê
 N+4. O que está aberto           dívidas, riscos, buracos
 A. Árvore completa               arquivo por arquivo, marcado vivo/legado/morto
 B. Ordem de leitura              trilhas, com o que procurar em cada arquivo
```

A seção 4 é a espinha dorsal: siga **um evento real** (um request, uma mensagem,
um job) atravessando todas as camadas. É o que transforma lista de arquivos em
entendimento.

### 5. Entregar

Escreva o arquivo e resuma em poucas linhas o que ele cobre e as 2 ou 3
descobertas que mudam a leitura do sistema. Ofereça aprofundar o que ficou.

## Método Feynman, na prática

A regra: **explique pelo problema que o código resolve, não pela sua
implementação.**

| Não | Sim |
|---|---|
| "sorteia entre 60 e 180 segundos" | "gente de verdade não responde em 800ms; responder instantâneo denuncia automação" |
| "usa advisory lock" | "dois processos podem subir ao mesmo tempo; sem isso o cliente recebe a mensagem em dobro" |
| "a coluna é nullable" | "NULL aqui significa IA; só o literal 'humano' marca pessoa" |

Regras de escrita que sustentam isso: `references/estilo.md`.

Três hábitos que valem mais que o resto:

1. **Caça o post-mortem.** Em projeto de produção, os comentários não são
   documentação, são cicatrizes. "Isso quebrou em 07/08 e derrubou X" explica o
   código melhor que qualquer paráfrase. Cite com o número medido.
2. **Nomeie a armadilha.** Toda base tem uma coisa cujo nome mente. Encontre e dê
   destaque — é o que salva horas de quem lê.
3. **Diga o que NÃO ler.** Economizar o tempo do leitor é metade do valor.

## Escala

- **Repo pequeno** (<40 arquivos): leia você mesmo, pule os subagentes, doc mais curto.
- **Médio** (40 a 300): 4 a 7 subagentes, doc completo.
- **Grande** (300+) ou múltiplos repos: fatie por camada **e** por repo, e considere
  duas rodadas — mapa primeiro, aprofundamento depois.

Se o usuário pedir só uma parte ("me explica só o backend"), estude só ela, mas
mantenha a seção 2 (o mapa) mostrando onde ela se encaixa no todo.

## Referências

- `references/leitura-paralela.md` — como fatiar, o prompt de subagente pronto, o que sempre pedir
- `references/estrutura-do-doc.md` — cada seção do documento, o que entra e o que a faz funcionar
- `references/estilo.md` — regras de escrita, tabelas, diagramas ASCII, o que evitar
