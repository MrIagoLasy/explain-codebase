# Leitura paralela: como fatiar e o que pedir

## Por que subagente e não você

Um repo de 200 arquivos não cabe no seu contexto, e mesmo que coubesse você
perderia o fio: ao chegar no arquivo 150 já não lembra do 20. Subagentes leem
tudo, cada um devolve um relatório denso, e **você fica só com as conclusões**.

Regra: dispare todos na mesma mensagem, em paralelo. Sequencial desperdiça tempo
sem ganhar nada.

## Como fatiar

Fatie por **camada de responsabilidade**, nunca por ordem alfabética ou por
tamanho de pasta. O objetivo é que cada relatório seja legível sozinho.

Perguntas que definem o corte:

- O que roda em produção **hoje**? (essa é a frente prioritária)
- O que fala com o mundo externo? (APIs, envio, integrações)
- O que é infra? (deploy, cron, containers, workers)
- O que é dado? (schema, migrações, triggers)
- O que é interface? (front, painel, CLI)
- O que foi substituído mas ainda está no repo?
- Onde está a intenção original? (specs, ADRs, docs de design)

Cada frente vira um subagente. De 4 a 8 é o intervalo saudável; acima disso os
relatórios ficam redundantes.

**Se houver múltiplos repos:** fatie por camada dentro de cada um, e depois some
uma frente extra que procure a costura entre eles (mesma tabela, mesma função
duplicada, mesmo endpoint).

## O prompt de subagente (modelo)

```
Leia INTEGRALMENTE (Read completo, sem grep) os seguintes arquivos em <caminho>:

<lista explícita de arquivos, não glob>

Arquivos grandes: use limit 1000 e continue com offset 1000, 2000 até o fim.
Nunca pare no meio.

CONTEXTO: <uma frase sobre o que é o sistema e onde essa frente se encaixa. Se
houver algo que você já sabe e que muda a leitura — "isso aqui é o legado, foi
substituído por X" — diga.>

Produza um relatório técnico DENSO em pt-BR (sem travessão/em dash), com:

1. Para CADA arquivo: propósito em 1-2 frases, funções públicas principais com
   assinatura e o que fazem, o que toca no banco/rede, e o "porquê" não óbvio.
   Os comentários do código costumam contar incidentes reais — capture todos,
   com data e número quando houver.
2. <o que essa frente tem de específico: detalhe máximo em X, mapa completo de Y>
3. As dependências entre esses módulos (quem importa quem).
4. Os invariantes que esses módulos protegem.
5. Uma lista de pegadinhas: partes contraintuitivas que um novato erraria.
6. O que aqui é VIVO e o que é LEGADO, com a evidência que sustenta o veredito.

Cite sempre arquivo:linha. Este relatório vira material de estudo, não resuma demais.
```

## O que pedir sempre, em qualquer frente

Independente do escopo, esses seis itens são os que rendem o documento final:

1. **Read completo, sem grep.** Grep acha ocorrência, não entendimento. Se o
   subagente só grepar, o relatório vem raso e você não percebe.
2. **`arquivo:linha` em toda afirmação.** É o que te deixa verificar depois.
3. **Os comentários que contam incidente.** Em projeto de produção eles são a
   melhor fonte de "por quê" que existe. Peça explicitamente.
4. **O veredito vivo/legado/morto com evidência.** Não aceite "parece legado";
   peça o que sustenta (ninguém importa, flag desligada, substituído por X).
5. **Os invariantes.** "O que precisa ser sempre verdade aqui" é a pergunta que
   revela o desenho.
6. **As pegadinhas.** O que um novato erraria. Isso vira seção no documento.

## Quando o relatório volta

**Não confie cegamente.** Subagente afirma com confiança coisas erradas. Verifique
você mesmo quando a afirmação:

- muda o desenho do sistema ("não existe fila, é polling")
- contradiz outro relatório
- contradiz a documentação do próprio repo
- é a base de uma seção inteira do documento

Verificar é barato: um Read no arquivo citado, ou um grep pelo símbolo. Vale
sempre para as 3 ou 4 afirmações estruturais.

**Contradição entre relatórios é achado, não problema.** Duas frentes descrevendo
a mesma coisa de formas diferentes normalmente significa que existem mesmo duas
implementações, e isso é material para a seção "a costura".

## Custo

Cada subagente com leitura completa gasta na casa de 80k a 190k tokens. Sete
frentes ficam em ~800k. Para um sistema que alguém vai manter, é barato — mas
dimensione: repo de 30 arquivos não precisa disso, leia você mesmo.
