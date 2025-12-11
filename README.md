# Simulador de Aplicações Financeiras em DAX (Power BI)

Este repositório apresenta um **simulador completo de investimentos com
juros compostos**, desenvolvido em **Power BI** utilizando
exclusivamente **DAX**.\
O objetivo é demonstrar, de forma didática e prática, como modelar
**juros compostos com depósitos recorrentes mês a mês**, sem depender de
funções financeiras prontas.

------------------------------------------------------------------------

## 🚀 Funcionalidades

O simulador permite:

-   Definir **depósito mensal configurável**
-   Informar **taxa de juros mensal variável** (ex.: CDI, poupança,
    rentabilidade real)
-   Escolher o **prazo do investimento**
-   Visualizar cálculos detalhados por mês:
    -   Saldo inicial
    -   Depósito aplicado
    -   Juros do mês
    -   Saldo final (valor acumulado)

É uma excelente referência para estudos de DAX, principalmente sobre
**iteração**, **tabelas virtuais** e **modelagem financeira**.

------------------------------------------------------------------------

# 1. Arquitetura da Solução

A solução é composta pelos seguintes elementos:

### 1. Tabela de Meses (1 → 480)

Base para iterar cada período.

### 2. Parâmetros *What-If*

-   Depósito mensal (opcional)
-   Taxa mensal (%)
-   Prazo em meses

### 3. Medidas DAX

Cálculo do saldo inicial, juros, depósitos e valor acumulado.

### 4. Visuais

Tabela detalhada, curva de crescimento, cartões-resumo e comparativos.

### Premissas

-   Depósito padrão: **R\$ 2.000,00** (pode ser substituído por
    parâmetro)
-   Taxa informada em **percentual ao mês**
-   Depósito ocorre no **primeiro dia do mês**

------------------------------------------------------------------------

# 2. Pré-requisitos

-   **Power BI Desktop** (versão atualizada)
-   Conceitos básicos de:
    -   Tabelas calculadas
    -   Medidas DAX
    -   Parâmetros What-If

------------------------------------------------------------------------

# 3. Criando a Tabela de Meses

Menu:

> Modeling \> New Table

``` dax
DimMes =
ADDCOLUMNS (
    GENERATESERIES ( 1, 480, 1 ),
    "MesTexto", "Mês " & FORMAT ( [Value], "0" )
)
```

Renomeie **\[Value\]** para **Mes** (tipo inteiro).\
Use **DimMes\[Mes\]** como eixo dos visuais.

------------------------------------------------------------------------

# 4. Parâmetros What-If

## 4.1. Taxa Mensal (%)

> Modeling \> New Parameter \> Numeric

Configuração:

-   Min: 0\
-   Max: 3\
-   Incremento: 0,05\
-   Decimais: 2

O Power BI cria:

-   tabela: **Par Taxa Mensal**
-   medida: **Par Taxa Mensal Value**

Crie:

``` dax
Taxa Mensal (%) = 'Par Taxa Mensal'[Par Taxa Mensal Value]

Taxa Mensal =
DIVIDE ( [Taxa Mensal (%)], 100 )
```

------------------------------------------------------------------------

## 4.2. Prazo do Investimento

> Modeling \> New Parameter \> Numeric

Configuração:

-   Min: 1\
-   Max: 480\
-   Incremento: 1

Medida gerada:

``` dax
Meses Selecionados = 'Par Prazo Meses'[Par Prazo Meses Value]
```

------------------------------------------------------------------------

## 4.3. Parâmetro de Depósito Mensal (opcional)

> Modeling \> New Parameter \> Numeric

Configuração:

-   Min: 100\
-   Max: 10.000\
-   Incremento: 100

Medida gerada:

``` dax
Depósito Mensal = 'Par Deposito Mensal'[Par Deposito Mensal Value]
```

Ou, para fixar:

``` dax
Depósito Mensal = 2000
```

------------------------------------------------------------------------

# 5. Medidas DAX do Simulador

## 5.1. Valor Acumulado (Saldo Final por mês)

``` dax
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
```

------------------------------------------------------------------------

## 5.2. Saldo Inicial

``` dax
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
```

------------------------------------------------------------------------

## 5.3. Juros Mensais

``` dax
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
```

------------------------------------------------------------------------

## 5.4. Depósito Exibido

``` dax
Depósito Exibido =
VAR MesAtual = SELECTEDVALUE ( DimMes[Mes] )
VAR Prazo    = [Meses Selecionados]
RETURN
IF (
    ISBLANK ( MesAtual ) || MesAtual > Prazo,
    BLANK(),
    [Depósito Mensal]
)
```

------------------------------------------------------------------------

# 6. Criando a Tabela de Simulação

Inclua no visual Table:

-   DimMes\[Mes\]
-   \[Depósito Exibido\]
-   \[Saldo Inicial\]
-   \[Juros Mensais\]
-   \[Valor Acumulado\]

Use os parâmetros como Slicers:

-   Par Taxa Mensal\
-   Par Prazo Meses\
-   (Opcional) Par Depósito Mensal

------------------------------------------------------------------------

# 7. Visuais Recomendados

## 7.1. Gráfico de Linha -- Valor Acumulado

Mostra a curva de crescimento exponencial.

## 7.2. Gráfico de Colunas -- Juros Mensais

Mostra evolução do rendimento mensal.

## 7.3. Cartões de Resumo

``` dax
Total Aportado =
[Depósito Mensal] * [Meses Selecionados]

Total Juros =
[Valor Acumulado] - [Total Aportado]
```

## 7.4. Donut -- Aporte x Juros

Visualiza proporção entre capital investido e rendimento.

------------------------------------------------------------------------

Este simulador demonstra como implementar juros compostos iterativos no
Power BI utilizando apenas DAX, servindo como base para estudos e
projetos de finanças pessoais.
