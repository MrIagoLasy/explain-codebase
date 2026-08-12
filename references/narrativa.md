# Como construir cada fluxo

## O que é um fluxo

Uma história com começo, meio e fim, seguindo **uma coisa acontecendo** no sistema.
Não é "o módulo X faz Y". É "o usuário clica em salvar, e então...".

Teste rápido: se você consegue contar em voz alta para alguém, na ordem, sem pular,
é um fluxo. Se você precisa dizer "e existe também...", virou catálogo.

## A anatomia

Todo fluxo bom tem esta forma:

```
1. O gatilho          o que dá início (alguém clica, um cron dispara, chega um webhook)
2. O primeiro passo   e o problema que ele resolve
3. A complicação      o que torna isso mais difícil do que parece
4. A solução          e a marca que ela deixou no resto do sistema
5. (repete 3-4)       enquanto houver complicação relevante
6. O desfecho         o que ficou pronto para o próximo fluxo usar
7. O diagrama         o fluxo inteiro condensado em ASCII
```

O passo 3 é o que separa narrativa de descrição. Sem complicação não há história —
só uma lista de chamadas de função. Procure sempre: *por que isso não é trivial?*

## Ordenar os fluxos

**Por dependência narrativa**, não por importância nem por ordem de execução.

Cada fluxo pode usar o que os anteriores estabeleceram, e nada do que vem depois.
Se o Fluxo 5 precisa de um conceito que só aparece no 7, um dos dois está no lugar
errado.

Uma ordem que costuma funcionar:

1. **O evento principal.** O que o sistema existe para fazer, do início ao fim.
2. **O caminho de exceção.** O que acontece quando o 1 não dá conta.
3. **Uma restrição externa** que molda o desenho (limite de API, regra de
   plataforma, compliance).
4. **Um incidente** que deixou cicatriz no código.
5 em diante. **Os paralelos e as bordas**, cada um como história própria.
Último. **O que acontece quando ninguém está olhando** (monitoramento, alertas).

Os três ou quatro primeiros são a coluna vertebral. Diga isso ao leitor na abertura:
quem parar ali já entende o sistema.

## Tipos de fluxo que funcionam

**O fluxo principal.** O evento central, contado com todos os detalhes. É o mais
longo e o mais importante. Aqui é onde os conceitos do sistema nascem.

**O fluxo de exceção.** O que acontece quando o principal falha ou não se aplica.
Costuma revelar as decisões mais interessantes de produto.

**O fluxo de restrição.** Uma regra externa (janela de tempo, cota, formato
obrigatório) e como o sistema inteiro se organiza em volta dela.

**O fluxo de incidente.** Um problema real que aconteceu, o que se descobriu, e o
que mudou. É o formato que mais ensina, porque explica as travas pela ausência delas.
Estrutura: o que era para acontecer → o que aconteceu → por que a proteção falhou →
o que mudou.

**O fluxo de bastidor.** Algo que roda sozinho: um cron, um ETL, um vigia. Conte do
gatilho ao efeito, e diga que problema existiria sem ele.

**O fluxo humano.** Onde uma pessoa intervém. O que ela vê, o que ela faz, e o que
isso muda no comportamento automático.

## O que nunca fazer dentro de um fluxo

**Listar arquivos.** "Isso acontece em `foo.py`, `bar.py` e `baz.py`" quebra a
narrativa. O leitor não vai memorizar e não precisa — a Parte 2 tem isso.

**Catalogar opções.** "Existem cinco tipos de X: A, B, C, D e E" é inventário. Se os
cinco importam, escolha um e conte a história dele; mencione que há outros.

**Introduzir mecanismo antes do problema.** Se você escreveu "existe um cursor que
marca..." antes de o leitor sentir falta dele, inverteu a ordem. Mostre o problema
primeiro ("como o sistema sabe que já respondeu?"), o mecanismo depois.

**Mas atenção à exceção:** isso vale para **mecanismo**, não para **substantivo do
domínio**. Se o sistema tem duas linhas de atendimento, três tipos de usuário, dois
ambientes — isso é fato do mundo e vai na **abertura**, antes do primeiro fluxo.
Apresentar no meio obriga o leitor a reinterpretar o que já leu, e é exatamente o
efeito que a narrativa existe para evitar.

Teste: se ele precisa voltar e reler um parágrafo à luz do que acabou de aprender, o
conceito entrou tarde.

**Explicar o óbvio do domínio.** O leitor é inteligente e não conhece este sistema.
Não explique o que é uma fila; explique por que **esta** fila tem duas filas.

**Antecipar.** "Como veremos no Fluxo 7" é dívida: manda o leitor guardar uma pergunta
que você não vai responder agora. Ou resolva ali em duas frases, ou corte a
antecipação, ou reordene os fluxos.

Se o gancho for genuinamente útil, deixe **sem o número**: "isso tem uma consequência
que aparece mais adiante" sinaliza sem criar dívida.

Ao terminar, **audite**: `grep -n "Fluxo [0-9]"` (ou o equivalente à sua numeração) e
resolva cada referência que aponta para frente. Para trás pode ficar — reforça o
encadeamento e recompensa quem leu em ordem.

## Como decidir o nível de detalhe

Pergunta: **o leitor precisa disso para entender o próximo passo?**

- Sim → conte agora
- Não, mas é interessante → deixe para um fluxo próprio depois
- Não e não é → corte

O erro mais comum é detalhar demais o começo. O primeiro fluxo carrega o leitor
para dentro do sistema; se ele afundar em detalhe no passo 2, larga.

## Onde entra código: pseudocódigo, nunca literal

Colar o original é o erro mais fácil de cometer e o mais chato de ler. A informação
que o leitor precisa é **a decisão**, não a sintaxe.

**Bloco de pseudocódigo** quando a estrutura da decisão importa:

```
pegue até 40 leads onde:
    a ÚLTIMA mensagem da conversa é do lead        ← é a nossa vez de falar
    e ela veio pela linha oficial                  ← a linha humana é invisível
    e ela é mais nova que o cursor daquele lead    ← ainda não tratamos
    e o lead não está escalado nem assumido
ordenados por quem espera há mais tempo
```

Isso ensina mais que o SQL real, e cabe na cabeça. Depois do bloco, **destrinche as
linhas que carregam decisão** — cada condição costuma ter um porquê que vale um
parágrafo.

Casos que pedem bloco: uma consulta que define o comportamento central, uma ordem de
verificações (as travas antes de enviar), uma precedência (quem vence quem), uma
bifurcação com mais de dois ramos.

**Prosa** quando só o efeito importa. "A Cloudflare descomprime gzip nas respostas
que busca, mas não nas requisições que recebe; sem tratar isso, a leitura do JSON
quebra" é melhor que dez linhas de `DecompressionStream`.

**Sempre a âncora** (`arquivo.py:123`) ao lado, para quem quiser o original.

Regra prática: se você está copiando e colando do editor, provavelmente errou.

## O diagrama de fechamento de cada fluxo

Ao fim do fluxo principal (e de qualquer um que tenha mais de 5 passos), condense em
ASCII. O leitor volta a esse desenho depois.

Regras: cabe numa tela, setas com rótulo do que trafega, comentário curto ao lado
das caixas que precisam.

```
lead manda "oi"
   │
   ▼  webhook
programa de ingestão ── anota no banco ──▶ e encerra. não avisa ninguém.
   │
   ▼  (até 20s depois)
agente pergunta: "a última fala é do lead?"
```

## O fechamento do documento

Duas peças, nesta ordem:

**O diagrama do sistema inteiro.** Agora que o leitor viu cada peça funcionando, o
desenho completo é reconhecimento e não introdução. Mostre as peças, o que as liga,
e as velocidades diferentes (tempo real, 20 segundos, 5 minutos, diário).

**As 4 ou 5 ideias que sustentam tudo.** Numeradas, uma frase de título e duas de
explicação. É o que fica na cabeça depois de uma semana. Escolha as que explicam o
maior número de decisões do código.

Termine com a dica operacional mais útil que você encontrou — normalmente "leia os
comentários antes de mexer", porque em projeto de produção eles são cicatrizes com
data e número.
