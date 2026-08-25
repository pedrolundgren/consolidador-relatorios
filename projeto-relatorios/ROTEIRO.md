# Projeto: Consolidador de Relatórios de Chamados

Roteiro de execução. **Não tem código pronto aqui de propósito** — o valor do projeto
é você escrever, e é isso que você vai ter que explicar numa entrevista.

---

## O que você vai construir

Um script Python que lê 4 arquivos de chamados vindos de "sistemas diferentes",
limpa e padroniza tudo, aplica regras de negócio e exporta um relatório
consolidado em Excel.

Isso é, palavra por palavra, o primeiro item da vaga da Huawei:
*"automação da geração, exportação e tratamento de relatórios"*.

## Os dados

Estão em `dados/`. São 330 chamados em 4 arquivos, todos com defeitos **diferentes
e propositais**:

| Arquivo | Problemas plantados |
|---|---|
| `chamados_2026_05.csv` | Nenhum. É a referência de como tudo deveria ser. |
| `chamados_2026_06.csv` | Nomes de coluna diferentes; datas em `DD/MM/AAAA` |
| `chamados_2026_07.csv` | Separador `;`; decimal com vírgula; células vazias; **11 linhas duplicadas** |
| `chamados_2026_08.csv` | Espaços sobrando nos nomes e valores; categoria escrita em 3 capitalizações; horas negativas; custos absurdos |

## Tarefas

Faça na ordem. Cada bloco funcionando antes de passar pro próximo.

### 1. Leitura
- [ ] Ler os 4 arquivos de `dados/` sem escrever os nomes um por um (`pathlib` ou `glob`)
- [ ] Descobrir por que o de julho vem tudo numa coluna só e corrigir
- [ ] Juntar tudo num único DataFrame

### 2. Padronização de colunas
- [ ] Tirar espaços sobrando dos nomes de coluna
- [ ] Criar um dicionário de renomeação para que `ID`/`id_chamado`, `Tipo`/`categoria`,
      `Responsavel`/`tecnico` etc. virem um nome só
- [ ] Confirmar que sobraram exatamente 9 colunas

### 3. Limpeza de valores
- [ ] Converter as duas colunas de data para `datetime` (atenção: tem dois formatos)
- [ ] Converter horas e custo para número (o arquivo de julho usa vírgula decimal)
- [ ] Tirar espaços dos valores de texto e padronizar a capitalização das categorias
- [ ] Remover as linhas duplicadas — e **imprimir quantas foram removidas**

### 4. Regras de negócio
- [ ] Chamado sem data de fechamento = ainda em aberto. Criar uma coluna `status`.
- [ ] Horas negativas são erro de sistema: decidir o que fazer e **deixar comentado no código o porquê**
- [ ] Custos absurdos (muito acima do normal): identificar e tratar
- [ ] Criar uma coluna `dias_para_resolver` a partir das duas datas

### 5. Resumo e KPIs
- [ ] Total de chamados, total de horas e custo total por **categoria**
- [ ] Mesma coisa por **mês**
- [ ] Tempo médio de resolução por prioridade
- [ ] Quantos chamados continuam em aberto

### 6. Exportação
- [ ] Exportar um `.xlsx` com pelo menos 2 abas: `dados_limpos` e `resumo`
- [ ] Bibliotecas: `pandas` + `openpyxl` (`pip install pandas openpyxl`)

### 7. README (não pule esta parte)
É o que o recrutador lê de verdade. Precisa ter:
- [ ] O que o projeto faz, em 2 ou 3 frases
- [ ] Qual problema cada arquivo de entrada tinha
- [ ] Como rodar (`python main.py`)
- [ ] Um print ou trecho do Excel gerado
- [ ] Que bibliotecas usou

---

## Regras que valem mais que o código

1. **Print de progresso a cada etapa.** "85 linhas lidas de maio", "11 duplicatas
   removidas". Além de ajudar a debugar, mostra cuidado com o processo.
2. **Comente as decisões, não a sintaxe.** Ninguém precisa de `# lê o csv`.
   Todo mundo quer saber *por que* você descartou as horas negativas em vez de
   zerá-las.
3. **Se travar, resolva um arquivo só primeiro.** Faz funcionar com o de maio,
   depois adiciona os outros. Não tente resolver os 4 de uma vez.
4. **Não peça o código pronto pra ninguém.** Se você não souber explicar cada
   linha numa entrevista, o projeto joga contra você em vez de a favor.
