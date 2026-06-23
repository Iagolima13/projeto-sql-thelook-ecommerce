# projeto-sql-thelook-ecommerce
# Trilha de Analytics Avançado em SQL: Case TheLook E-Commerce
Este repositório contém uma sequência de 10 desafios práticos de SQL desenvolvidos para simular a rotina de um Analista de Dados em um e-commerce de grande volume. As análises foram construídas utilizando o Google BigQuery sobre o dataset público `thelook_ecommerce`.

## 🎯 Objetivo do Projeto
O objetivo foi resolver problemas de negócios reais enviados por diferentes áreas (Financeiro, CRM, Growth e Diretoria Executiva), evoluindo desde métricas básicas de faturamento até análises analíticas complexas e otimização de consultas para Big Data.

## 🧰 Tecnologias e Conceitos Utilizados
* **Banco de Dados:** Google BigQuery 
* **Técnicas Avançadas:** Cláusulas WITH (CTEs encadeadas), Window Functions (`RANK`, `LAG`, Frames de acumulação), Subqueries, Agregações Condicionais e Funções de String.

## 📊 Resumo dos Desafios e Soluções

| Desafio | Área de Negócio | Conceito SQL Chave | O que resolve? | 
| :--- | :--- | :--- | :--- |
| **07** | Planejamento | Window Function Frame (`ROWS BETWEEN`) | Cálculo do Faturamento Acumulado anual sem conflito de agregação. |
| **08** | CRM / Marketing | `COUNT(DISTINCT)`, `HAVING` | Identificação do Top 10 Clientes VIP (Fidelidade e faturamento). |
| **09** | Finanças | Window Function `LAG()`, CTEs | Análise de crescimento mês a mês (MoM %) com precisão decimal. |
| **10** | Diretoria Executiva | Princípio de Pareto (Regra 80/20) | Identificação da concentração de receita da empresa. |

## 🚀 Destaques de Soluções & Código Técnico

### Exemplo: Análise de Crescimento Mês a Mês (Desafio 9)
**Problema:** O CFO precisava do ritmo de crescimento percentual de 2025, corrigindo perdas de dados de fim de ano e evitando custos de processamento.
**Abordagem Técnica:** Utilização de CTEs em camadas para isolar o cálculo do `LAG()` e aplicação de filtros estritos de data para ativar o 
```
WITH faturamento_mensal AS (
  SELECT
    DATE_TRUNC(created_at, MONTH) AS mes,
    SUM(sale_price) AS faturamento_atual
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  WHERE status NOT IN ('Cancelled', 'Returned')
    -- CORREÇÃO 1: Evita perda de dados no dia 31/12 e otimiza partições
    AND created_at >= '2025-01-01' 
    AND created_at < '2026-01-01'
  GROUP BY 1
),

faturamento_com_lag AS (
  SELECT
    mes,
    faturamento_atual,
    -- Executamos o LAG apenas uma vez aqui para reaproveitar depois
    LAG(faturamento_atual) OVER (ORDER BY mes ASC) AS faturamento_anterior
  FROM faturamento_mensal
)

SELECT
  mes,
  faturamento_atual,
  -- Tratamos o NULL de Janeiro apenas para exibição visual
  COALESCE(faturamento_anterior, 0) AS faturamento_anterior,
  
  -- CORREÇÃO 2: Multiplicação dentro do ROUND para manter a precisão de 15,63%
  ROUND(
    ((faturamento_atual - faturamento_anterior) / faturamento_anterior) * 100, 
    2
  ) AS crescimento_mom
FROM faturamento_com_lag
ORDER BY mes ASC;
```
