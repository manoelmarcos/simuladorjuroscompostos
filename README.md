# Simulador de Aplicações Financeiras em DAX (Power BI)

Este repositório contém um **simulador de investimentos com juros compostos** desenvolvido em **Power BI** utilizando **DAX**.

A ideia é permitir que o usuário simule aplicações mensais com:

- Depósito mensal configurável
- Taxa de juros mensal variável (ex.: CDI, rentabilidade média, etc.)
- Prazo em meses configurável
- Cálculo detalhado, mês a mês, de:
  - Saldo Inicial
  - Depósito
  - Juros do mês
  - Saldo Final (Valor Acumulado)

O objetivo é ser um exemplo didático de como implementar **juros compostos com depósitos recorrentes** usando DAX, sem recorrer a funções financeiras prontas.

---

## 1. Estrutura geral da solução

A lógica da solução é baseada em:

1. Uma **tabela de meses** (1, 2, 3, … até um limite, por exemplo, 480 meses).
2. **Parâmetros What-If** para:
   - Depósito mensal (opcional, se quiser deixar variável)
   - Taxa de juros mensal
   - Quantidade de meses do investimento
3. **Medidas DAX** que calculam, para cada mês:
   - Saldo Inicial
   - Juros/Rendimento do mês
   - Valor Acumulado (Saldo Final)
4. **Visuais** de tabela, linha, coluna e cartões para visualizar a evolução da aplicação.

Suposições do modelo:

- Depósito padrão: **R$ 2.000,00** (pode ser parâmetro ou valor fixo).
- Taxa informada pelo usuário como **percentual ao mês** (ex.: 0,93% ao mês).
- Juros compostos com depósito no **início de cada mês**.

---

## 2. Pré-requisitos

- **Power BI Desktop** (versão recente).
- Conhecimento básico de:
  - Criação de tabelas calculadas
  - Criação de medidas DAX
  - Uso de parâmetros *What-If*

---

## 3. Criar a tabela de Meses

No Power BI, acesse:

> Modeling > New Table

Crie a tabela de meses:

```DAX
DimMes =
ADDCOLUMNS (
    GENERATESERIES ( 1, 480, 1 ),      -- limite máximo de meses
    "MesTexto", "Mês " & FORMAT ( [Value], "0" )
)

Em seguida:

Renomeie a coluna [Value] para Mes (tipo inteiro).

Você usará DimMes[Mes] como eixo dos visuais (gráficos e tabela).

4. Criar os parâmetros What-If
4.1. Parâmetro de Taxa Mensal

No Power BI:

Modeling > New Parameter > Numeric

Configure:

Nome: Par Taxa Mensal

Mínimo: 0

Máximo: 3

Incremento: 0,05

Casas decimais: 2

O Power BI vai criar:

Uma tabela Par Taxa Mensal

Uma medida: Par Taxa Mensal Value

Crie então as medidas:

Taxa Mensal (%) = 'Par Taxa Mensal'[Par Taxa Mensal Value]

Taxa Mensal =
DIVIDE ( [Taxa Mensal (%)], 100 )


Observação: a primeira medida guarda a taxa como percentual (ex.: 0,93), e a segunda converte para decimal (0,0093).

4.2. Parâmetro de Prazo (Meses)

Novamente em:

Modeling > New Parameter > Numeric

Configure:

Nome: Par Prazo Meses

Mínimo: 1

Máximo: 480

Incremento: 1

Será criada automaticamente a medida:

Meses Selecionados = 'Par Prazo Meses'[Par Prazo Meses Value]


Essa medida representa o número de meses do investimento definido pelo usuário.

4.3. (Opcional) Parâmetro de Depósito Mensal

Se desejar permitir que o usuário defina o valor do depósito mensal:

Modeling > New Parameter > Numeric

Configure:

Nome: Par Deposito Mensal

Mínimo: 100

Máximo: 10000

Incremento: 100

Medida automática gerada:

Depósito Mensal = 'Par Deposito Mensal'[Par Deposito Mensal Value]


Se preferir um valor fixo, basta criar:

Depósito Mensal = 2000

5. Medidas DAX do simulador
5.1. Medida – Valor Acumulado (Saldo Final de cada mês)

Aqui entra o “truque” para evitar recursão em DAX:
Para um mês N, cada depósito feito em um mês M cresce por (N - M + 1) meses (considerando depósito no início do mês).

Valor Acumulado =
VAR MesAtual = SELECTEDVALUE ( DimMes[Mes] )
VAR Prazo    = [Meses Selecionados]
VAR Taxa     = [Taxa Mensal]
VAR Dep      = [Depósito Mensal]
RETURN
IF (
    ISBLANK ( MesAtual ) || MesAtual > Prazo,
    BLANK(),
    VAR TabelaDepositos =
        ADDCOLUMNS (
            FILTER ( ALL ( DimMes ), DimMes[Mes] <= MesAtual ),
            "FatorCrescimento",
                POWER ( 1 + Taxa, MesAtual - DimMes[Mes] + 1 )
        )
    RETURN
        SUMX ( TabelaDepositos, Dep * [FatorCrescimento] )
)


Essa medida devolve o saldo acumulado no final de cada mês.

5.2. Medida – Saldo Inicial

O saldo inicial do mês N é o Valor Acumulado do mês N - 1.

Saldo Inicial =
VAR MesAtual = SELECTEDVALUE ( DimMes[Mes] )
VAR Prazo    = [Meses Selecionados]
RETURN
IF (
    ISBLANK ( MesAtual ) || MesAtual > Prazo,
    BLANK(),
    IF (
        MesAtual = 1,
        0,
        CALCULATE (
            [Valor Acumulado],
            FILTER ( ALL ( DimMes ), DimMes[Mes] = MesAtual - 1 )
        )
    )
)

5.3. Medida – Juros/Rendimento Mensal

A lógica dos juros do mês é:

Juros do mês = (Saldo Inicial + Depósito) × Taxa

Juros Mensais =
VAR MesAtual  = SELECTEDVALUE ( DimMes[Mes] )
VAR Prazo     = [Meses Selecionados]
VAR Taxa      = [Taxa Mensal]
VAR Dep       = [Depósito Mensal]
VAR SaldoIni  = [Saldo Inicial]
RETURN
IF (
    ISBLANK ( MesAtual ) || MesAtual > Prazo,
    BLANK(),
    ( SaldoIni + Dep ) * Taxa
)

5.4. Medida – Depósito exibido na tabela

Útil para não exibir depósitos após o fim do prazo selecionado:

Depósito Exibido =
VAR MesAtual = SELECTEDVALUE ( DimMes[Mes] )
VAR Prazo    = [Meses Selecionados]
RETURN
IF (
    ISBLANK ( MesAtual ) || MesAtual > Prazo,
    BLANK(),
    [Depósito Mensal]
)

6. Montar a tabela “planilha” no relatório

Crie um visual do tipo Table e adicione os campos/medidas:

DimMes[Mes]

[Depósito Exibido]

[Saldo Inicial]

[Juros Mensais]

[Valor Acumulado]

Recomendações:

Formate os valores monetários em R$.

Deixe Mes como inteiro.

Coloque os parâmetros na página como slicers:

Par Taxa Mensal

Par Prazo Meses

(Opcional) Par Deposito Mensal

À medida que o usuário altera os parâmetros, a tabela de simulação é recalculada automaticamente.

7. Visuais para ilustrar juros compostos

Algumas sugestões de visuais para tornar o simulador mais didático:

7.1. Gráfico de Linha – Valor Acumulado

Visual: Line chart

Eixo (X): DimMes[Mes]

Valores (Y): [Valor Acumulado]

Mostra de forma clara a curva exponencial do crescimento do investimento.

7.2. Gráfico de Colunas – Juros Mensais

Visual: Clustered Column Chart

Eixo (X): DimMes[Mes]

Valores (Y): [Juros Mensais]

Mostra a evolução dos juros recebidos por mês.

7.3. Cartões – Resumo do investimento

Crie medidas auxiliares:

Total Aportado =
[Depósito Mensal] * [Meses Selecionados]

Total Juros =
[Valor Acumulado] - [Total Aportado]


Use Card visuals para exibir:

Saldo Final: [Valor Acumulado] filtrado pelo último mês (prazo máximo selecionado).

Total Aportado: [Total Aportado]

Total de Juros: [Total Juros]

7.4. (Opcional) Gráfico de Pizza/Donut – Proporção Aporte x Juros

Visual: Donut ou Pie chart

Valores:

[Total Aportado]

[Total Juros]

Permite ilustrar a proporção entre o que foi investido e o que foi ganho em juros.
