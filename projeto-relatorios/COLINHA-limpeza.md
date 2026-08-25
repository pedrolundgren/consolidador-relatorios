# Colinha — Limpeza e Tratamento de Dados

Vale para R e para Pandas. Muda o comando, não muda o método.

---

## O ciclo — nunca muda

```
1. IMPORTAR      ler o arquivo
2. DIAGNOSTICAR  olhar TUDO antes de mexer em NADA
3. TRATAR        consertar, uma coluna por vez
4. CONFERIR      provar que sumiu
```

**Nunca conserta enquanto explora.** Levanta o mapa completo primeiro.

---

## Só existem 2 tipos de defeito

| Tipo | O que é |
|---|---|
| **NA** | o valor não existe, célula vazia |
| **Fora do domínio** | o valor existe, mas é proibido pelo contrato |

Não tem um terceiro.

---

## Passo 2 — Diagnosticar

| | R | Pandas |
|---|---|---|
| Raio-X de tudo | `summary(dados)` | `df.describe()` + `df.info()` |
| Coluna de texto | `table(dados$Col)` | `df["Col"].value_counts()` |
| Coluna numérica | `summary(dados$Col)` | `df["Col"].describe()` |
| Ver outlier | `boxplot(dados$Col)` | `df["Col"].plot.box()` |
| Achar linhas com NA | `dados[!complete.cases(dados), ]` | `df[df.isna().any(axis=1)]` |

---

## Passo 3 — Tratar

### O molde: ONDE, QUAL, O QUÊ

```r
# R
dados[  PERGUNTA  , ]$COLUNA  =  VALOR
#      └─ ONDE ─┘     └QUAL┘     └O QUÊ┘
```

```python
# Pandas
df.loc[  PERGUNTA  , "COLUNA" ]  =  VALOR
```

### ONDE — as 3 únicas perguntas que existem

| Quero achar | R | Pandas |
|---|---|---|
| palavra errada | `dados$Col == 'errada'` | `df["Col"] == "errada"` |
| número fora da faixa | `dados$Col < min \| dados$Col > max` | `(df["Col"] < min) \| (df["Col"] > max)` |
| célula vazia | `is.na(dados$Col)` | `df["Col"].isna()` |

### O QUÊ — a tabela de decisão

| A coluna é... | Preenche com | R | Pandas |
|---|---|---|---|
| **numérica** | **mediana** | `median(dados$Col)` | `df["Col"].median()` |
| **texto** | **moda** | escreve a palavra na mão | `df["Col"].mode()[0]` |

**Média nunca.** Ela é puxada pelo outlier que você está tentando remover.
Exemplo real: temperaturas com um 1220 no meio deram média 155,6 — maior que
todas as 13 temperaturas verdadeiras. A mediana deu 73,5.

**No R não existe função de moda.** Roda `table()`, olha qual contagem é maior,
escreve a palavra na mão. (A função `mode()` existe mas faz outra coisa —
devolve o tipo do objeto.)

---

## is.na × na.rm — um ACHA, o outro IGNORA

```r
dados[ is.na(dados$Umidade), ]$Umidade = median(dados$Umidade, na.rm = TRUE)
       └───── ACHA o vazio ─┘                   └──── IGNORA o vazio ────┘
              ESQUERDA                                  DIREITA
```

| | Onde fica | O que faz |
|---|---|---|
| `is.na()` | esquerda, no colchete | **acha** as células vazias |
| `na.rm = TRUE` | direita, na conta | **ignora** as células vazias |

A esquerda precisa achar o vazio pra saber onde escrever.
A direita precisa ignorar o vazio pra conseguir calcular.

> **Regra sem pensar: na linha que conserta NA, o `median` SEMPRE leva
> `na.rm = TRUE`.** Se você está consertando um NA, é porque existe NA
> naquela coluna — logo a conta viria contaminada.

Depois que os NAs acabaram, as contas seguintes não precisam mais.

**Em Pandas isso quase não aparece:** `df["Col"].median()` já ignora NA sozinho.

---

## Ordem dentro da coluna

1. **NA primeiro**
2. **Fora do domínio depois**

Assim, da segunda conta em diante você não precisa mais de `na.rm`.

---

## Passo 4 — Conferir

Roda **a mesma pergunta do colchete** de novo. Tem que vir vazia.

```r
dados[dados$Umidade < 0 | dados$Umidade > 100, ]   # zero linhas
dados[is.na(dados$Umidade), ]                      # zero linhas
dados[!complete.cases(dados), ]                    # zero linhas no fim de tudo
summary(dados)
```

> **O R falha calado.** Atribuição não imprime nada, `median` com NA devolve NA
> sem avisar, level fantasma continua na lista. A conferência é a única forma
> de saber se funcionou.

---

## Armadilhas que já me pegaram

| Erro | O que acontece | Conserto |
|---|---|---|
| `table=(x)` | vira atribuição, não imprime nada | tira o `=` |
| `x<-130` | `<-` é atribuição! destrói a coluna | **espaço**: `x < -130` |
| `sol` sem aspas | R procura variável chamada sol | `'sol'` |
| `"median"(x)` | aspas são pra valor, não pra função | `median(x)` |
| `dados[7, ]` | número da linha não escala | usa a pergunta |
| `median(x)` com NA | devolve NA, preenche NA com NA | `na.rm = TRUE` |
| `mean` em texto | não existe média de palavra | `table()` + moda |
| linha nova em vez de editar | a errada continua rodando | apaga e reescreve |

**Hábito que elimina uma classe inteira de erro: espaço em volta de todo operador.**
`a < b`, `x = 1`, `p | q`. Nunca gruda.

---

## Level fantasma (só R)

Depois de trocar um valor de fator, a categoria antiga continua na lista com
contagem 0:

```r
dados$Col = factor(dados$Col)   # reconstrói só com o que existe
```

---

## O teste honesto

**Ctrl+Alt+R** roda o arquivo inteiro do zero.

Se o resultado sair certo saindo do CSV cru, teu script é reprodutível.
Se só funciona por causa de coisas que você digitou solto no Console, não é.

---

## Protocolo de diagnóstico de um arquivo

Sete perguntas, nesta ordem. Cada uma depende da anterior.

| # | Pergunta | Ferramenta | Sinal de problema |
|---|---|---|---|
| 1 | Abriu direito? | `.shape` | nº de colunas absurdo -> separador errado |
| 2 | Os nomes batem com o padrão? | `.columns` | nome divergente, ou com espaço sobrando |
| 3 | Os tipos estão certos? | `.info()` | número que veio como texto -> tem sujeira dentro |
| 4 | Falta alguma coisa? | `.isna().sum()` | qualquer valor acima de zero |
| 5 | Tem linha repetida? | `.duplicated().sum()` | qualquer valor acima de zero |
| 6 | Os textos são consistentes? | `.value_counts()` | mesma coisa escrita de 2 jeitos; valor fora do domínio |
| 7 | Os números fazem sentido? | `.describe()` | negativo onde não pode; max absurdo |

**`.info()` não substitui olhar a tabela.** Ele mostra a estrutura (nomes, tipos,
contagens), não os valores. Diferença de formato de data, espaço sobrando dentro
da célula e capitalização inconsistente só aparecem no `.head()`.

**No Jupyter, só a última expressão da célula aparece.** Se você empilhar
`.shape`, `.head()` e `.info()` na mesma célula, vai ver só a última e vai
diagnosticar pela metade. Use `print()` ou células separadas.

**Método leva `()`, atributo não.** Se a saída começa com `<bound method ...>`,
faltou o parêntese.

| Atributo (sem `()`) | Método (com `()`) |
|---|---|
| `.shape`, `.columns`, `.dtypes` | `.head()`, `.info()`, `.describe()`, `.isna()`, `.median()` |

---

## Protocolo de CORREÇÃO

O diagnóstico tem 7 passos. A correção tem 8 — e **a ordem é diferente**,
porque o que manda aqui não é a facilidade de detectar, é a **dependência**:
cada passo precisa que o anterior já esteja feito.

| # | Passo | Ferramenta | Por que nesta posição |
|---|---|---|---|
| 1 | **Ler certo** | `sep=";"` no `read_csv` | sem isso não existe tabela pra corrigir |
| 2 | **Padronizar nomes e empilhar** | `.columns.str.strip()` -> `.rename(columns=MAPA)` -> `pd.concat(..., ignore_index=True)` | sem isso são 4 tabelas, não 1. E o `concat` casa por NOME |
| 3 | **Converter tipos** | `.astype(str).str.replace(",", ".").astype(float)` | número que é texto não soma |
| 4 | **Normalizar texto** | `.str.strip()` + `.str.title()` nos VALORES | tem que vir ANTES do passo 5 — ver nota abaixo |
| 5 | **Remover duplicatas** | `.drop_duplicates()` | antes de qualquer estatística, senão a mediana sai enviesada |
| 6 | **Tratar os NAs** | `.isna()` + o molde `.loc[ONDE, QUAL] = O QUÊ` | agora as estatísticas são confiáveis |
| 7 | **Regras de negócio** | `.loc` + criar colunas novas | é decisão sua, não do Pandas |
| 8 | **Conferir** | o protocolo de diagnóstico inteiro | é a única prova de que funcionou |

### CORREÇÃO IMPORTANTE — datas NÃO vão depois do concat

Testado e comprovado: `pd.to_datetime(col, dayfirst=True)` numa coluna com
formatos misturados **corrompe em silêncio** as datas que já estavam certas.

```
maio antes:   2026-05-11   (11 de maio, ISO)
maio depois:  2026-11-05   (5 de novembro)   <- destruído, sem erro
```

O `dayfirst=True` obedece também no ISO: lê `2026-05-11` como ano/DIA/MÊS.

**A conversão de data vai POR ARQUIVO, antes do `concat`**, cada um com o
`format=` exato:

```python
maio[c]  = pd.to_datetime(maio[c],  format="%Y-%m-%d")   # ISO
junho[c] = pd.to_datetime(junho[c], format="%d/%m/%Y")   # brasileiro
```

`%Y` = ano 4 dígitos · `%m` = mês · `%d` = dia. Declare o formato exato em vez
de dar dica com `dayfirst` — assim, valor que não encaixa dá ERRO em vez de
converter errado calado.

### O critério geral de onde cada correção vai

| A correção... | Onde vai |
|---|---|
| **estraga** dado que já estava correto | **antes** de empilhar |
| é **inofensiva** em dado já correto | pode ir depois |

Trocar `,` por `.` num valor que já é `5.4` não faz nada -> pode ir depois.
`dayfirst` num valor já correto DESTRÓI o valor -> tem que ir antes.

> **O que depende da ORIGEM da linha se resolve antes do `concat`.** Depois de
> empilhar, você não sabe mais de qual arquivo cada linha veio.

Isso vale para: separador, encoding, formato de data — tudo que é propriedade
do arquivo, não do dado.

### Por que normalizar texto vem ANTES de remover duplicata

Duas linhas idênticas em tudo, menos que uma diz `Telefonia` e a outra
`telefonia `, **não são vistas como duplicatas** — o Pandas compara texto
caractere por caractere.

Se você deduplicar primeiro, essas escapam. Se normalizar primeiro, elas viram
idênticas e o `drop_duplicates()` pega. **Normalizar revela duplicata escondida.**

### Por que duplicata vem ANTES de tratar NA

O NA é preenchido com uma estatística (geralmente a mediana). Se ainda houver
linhas repetidas, elas entram na conta com peso dobrado e distorcem a mediana.
Limpa a base primeiro, calcula depois.

### A regra da moda NÃO é universal

Na coluna de categoria, preencher o vazio com a moda é correto.
Na coluna de **nome de pessoa**, é errado: você estaria atribuindo o
atendimento a um técnico real que não fez aquele chamado.

| Coluna vazia | Tratamento certo | Por quê |
|---|---|---|
| categoria | moda | é um rótulo, o mais provável serve |
| **técnico / responsável** | **texto tipo "Não atribuído"** | inventar um nome é acusar alguém |
| **data de fechamento** | **não preencher — criar coluna `status`** | o vazio É a informação: chamado em aberto |
| custo | mediana, ou zero, ou marcar | decisão de negócio — comente a escolha |

> **Antes de tapar um buraco, pergunte por que ele existe.** Às vezes o vazio
> é o dado, e preenchê-lo destrói a informação.

### Comente as decisões, não a sintaxe

Ninguém precisa de `# lê o csv`. Todo mundo quer saber por que você descartou
as horas negativas em vez de zerá-las. As decisões dos passos 6 e 7 são o que
um recrutador lê para saber se você pensa ou só executa.

---

## Como decidir o tratamento de um NA

Não olhe o dado — olhe o **significado da coluna**. Três perguntas, nesta ordem:

```
Célula vazia
  │
  ├─ 1. O vazio é um ESTADO REAL do negócio?
  │        SIM -> não preenche. Cria coluna que nomeia o estado
  │        ex: data_fechamento vazia = chamado em aberto
  │
  ├─ 2. É IDENTIDADE (pessoa, CPF, ID, matrícula)?
  │        SIM -> "Não informado". NUNCA a moda
  │        chutar identidade = afirmar que alguém fez algo que não fez
  │
  ├─ 3. É RÓTULO (categoria, prioridade, status)?
  │        SIM -> moda
  │
  └─ 4. É NÚMERO usado em conta?
           SIM -> decisão de negócio (mediana / zero / deixar vazio)
           Nenhuma é obviamente certa. DOCUMENTE a escolha
```

| Coluna | Pergunta que disparou | Tratamento |
|---|---|---|
| `data_fechamento` | 1 — o vazio é estado real | coluna `status` = "Em aberto" |
| `tecnico` | 2 — é nome de pessoa | `fillna("Não atribuído")` |
| `categoria` | 3 — é rótulo | moda |
| `custo_total` | 4 — entra em soma | `fillna(mediana)` + comentário justificando |

> **Na vida real, você PERGUNTA.** "O que significa este campo vazio neste
> sistema?" tem dono: o gerente da operação, quem mantém o sistema, a
> documentação. Chutar é o último recurso, não o primeiro.
