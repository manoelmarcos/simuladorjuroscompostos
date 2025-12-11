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
