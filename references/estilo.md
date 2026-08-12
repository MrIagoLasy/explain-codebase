# Estilo de escrita do documento

## A regra Feynman

**Explique pelo problema que o código resolve, não pela sua implementação.**

Teste rápido: se a frase descreve *o que o código faz*, ela é paráfrase e não vale
o espaço — o leitor podia ter lido o código. Se descreve *por que ele precisa
existir*, é explicação.

| Paráfrase (ruim) | Explicação (bom) |
|---|---|
| "sorteia entre 60 e 180 segundos" | "gente de verdade não responde em 800ms; instantâneo denuncia automação" |
| "usa advisory lock no Postgres" | "dois processos podem subir juntos; sem isso o cliente recebe em dobro" |
| "a coluna aceita NULL" | "NULL aqui significa IA; só o literal 'humano' marca pessoa" |
| "faz retry com backoff" | "o runtime derruba conexão de saída sem avisar, e sem retry o envio some" |
| "valida o input" | "o modelo passava o telefone truncado e a resposta ia para um número fantasma" |

## Frases e parágrafos

- **Uma ideia por parágrafo.** Três a cinco linhas.
- **Frase curta na afirmação central**, longa no desenvolvimento. A frase que o
  leitor precisa levar tem que caber num fôlego.
- **Voz ativa.** "O worker grava e para", não "a gravação é realizada pelo worker".
- **Presente do indicativo.** Você está descrevendo um sistema que existe.
- **Segunda pessoa quando instruir** ("se você mexer aqui"), terceira quando
  descrever.

## Formatação que ajuda

**Tabela** quando houver 3+ itens com os mesmos atributos. Comparação, catálogo,
mapeamento. Nunca para texto corrido.

**Diagrama ASCII** para fluxo e topologia. Regras:

- tem que caber numa tela
- setas com rótulo do que trafega, não só a seta
- comentário curto ao lado das caixas que precisam
- se não couber, você está detalhando demais para aquela altura do documento

```
lead manda "oi"
   │
   ▼  webhook
worker de ingestão ── grava em messages ──▶ e para.
   │
   ▼  (até 20s depois)
supervisor faz poll ── acha o lead
```

**Bloco de pseudocódigo** onde a estrutura da decisão importa: a query central, a
ordem das verificações, a precedência entre regras. Escreva em português, com
comentários apontando o que cada linha garante.

Nunca cole código literal. A informação é a decisão, não a sintaxe — e a âncora
`arquivo.py:123` já leva quem quiser ao original. Se você está copiando do editor,
provavelmente errou. Detalhes em `narrativa.md`.

**Citação** para frase real do repo que explica melhor que sua paráfrase. Use com
parcimônia — três ou quatro no documento inteiro, nas melhores.

## O que dá autoridade ao texto

**Número medido.** "Derrubou o número de GREEN para RED em 23 minutos" vale mais
que "causou problemas de qualidade". Se o repo tem o número, use.

**Data e atribuição.** "Decisão do dono do produto, 08/08/2026" mostra que aquilo foi
escolha, não acidente. (Se o documento for sair da empresa, use o papel, não o nome.)

**`arquivo:linha`** nas afirmações verificáveis. O leitor confere.

**O contraste com o que existia antes.** "Antes era X, hoje é Y, porque Z" ensina
os dois de uma vez.

## O que evitar

- **Travessão e en-dash.** Vírgula, dois pontos ou outra frase.
- **"É importante notar que", "vale ressaltar", "em outras palavras".** Corte e
  vá direto ao ponto.
- **Adjetivo sem medida.** "Robusto", "elegante", "poderoso" não informam.
  Prefira o fato: "sobrevive a restart do container porque o transcript vive no banco".
- **Regra de três** ("rápido, simples e confiável") — é tique de texto gerado.
- **Conclusão genérica de seção.** Se o parágrafo final não acrescenta fato, corte.
- **Emoji no corpo.** Só na legenda da árvore de arquivos, onde carrega informação.
- **Hedge desnecessário.** "Aparentemente o sistema parece fazer" — ou você
  verificou, ou diz que não verificou.

## Honestidade

Se você não tem certeza, **diga qual é a evidência**. "Não há teste cobrindo isso,
então é leitura de código, não comportamento observado" é uma frase honesta e útil.

Se um subagente afirmou e você não verificou, ou verifica ou marca como
"segundo o relatório, a confirmar".

Nunca invente um porquê. Se o código faz algo estranho e você não achou a razão,
escreva que é estranho e que a razão não está documentada. Isso é informação.

## O tom

Escreva como um colega sênior explicando o sistema para alguém que vai mantê-lo
amanhã. Nem manual formal, nem conversa jogada fora.

O leitor é inteligente e não conhece este sistema. Não explique o que é uma fila;
explique por que **esta** fila tem duas filas.
