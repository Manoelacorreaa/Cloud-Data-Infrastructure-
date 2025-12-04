# ☁️ Cloud Data Infrastructure: De CSVs Locais para PostgreSQL na Nuvem

Este projeto documenta a construção de um ambiente moderno de Dados, migrando datasets locais (AdventureWorks/Vendas) para um Data Warehouse na nuvem utilizando **Supabase (PostgreSQL)** e gerenciado via **VS Code**.

O objetivo foi simular um cenário real de Engenharia de Dados: ingestão de arquivos brutos, configuração de segurança SSL e conexão remota.

## 🛠️ Tech Stack & Ferramentas
- **Database:** PostgreSQL (hospedado no Supabase).
- **IDE:** VS Code.
- **Client:** Extensão SQLTools + Driver PostgreSQL (Matheus Teixeira).
- **Dados Brutos:** CSVs exportados e tratados (ETL).

## ⚙️ Arquitetura da Solução

1.  **ETL (Extract, Transform, Load):**
    - Extração de dados brutos em formato `.csv`.
    - Tratamento de cabeçalhos e tipos de dados.
    - Carga para o Supabase via interface de Table Editor.

2.  **Conexão Remota (VS Code ↔️ Nuvem):**
    - Configuração de driver PostgreSQL no VS Code.
    - Tunelamento seguro para acesso ao banco de produção.
  
 3. **Análises Realizadas (SQL):**
    * **Ranking de Volumetria:** Identificação de outliers de vendas.
    * **Basket Analysis (Agrupamento):** Análise de "tamanho do pedido" via agregações (`GROUP BY OrderNumber`) para identificar perfil de compra B2B e concentração de receita.

## 🔐 Desafios Técnicos Superados (Troubleshooting)

Durante a implementação da infraestrutura, enfrentei e resolvi desafios comuns de redes corporativas e cloud:

### 1. Bloqueio de Conexão (IPv6 / Porta 5432)
**O Problema:** A conexão padrão na porta `5432` estava retornando `ECONNREFUSED` devido a restrições de rede/ISP com IPv6.
**A Solução:** Redirecionamento da conexão para a porta **`6543`** (Supavisor / Connection Pooling), otimizando a estabilidade da conexão em redes residenciais.

### 2. Certificados SSL (Self Signed Certificate)
**O Problema:** O VS Code bloqueava a conexão por não reconhecer a autoridade certificadora do Supabase (`rejectUnauthorized: true`).
**A Solução:** Ajuste nas configurações do driver para permitir SSL em modo `Allow/Prefer`, garantindo a criptografia sem bloquear o acesso.

## 💻 Exemplo de Configuração (SQLTools)

Configuração utilizada para estabilizar a conexão (senha ocultada):

```json
{
  "driver": "PostgreSQL",
  "name": "Supabase Cora",
  "server": "db.SEU-PROJETO.supabase.co",
  "port": 6543,
  "database": "postgres",
  "username": "postgres",
  "ssl": true,
  "pgOptions": {
    "ssl": {
      "rejectUnauthorized": false
    }
  }
}
Exemplo de Query (Dialeto PostgreSQL)
Diferente do SQLite, o PostgreSQL exige aspas duplas para identificadores com espaços. Exemplo de consulta rodada diretamente na nuvem:
-- Análise de Vendas conectada ao Supabase
SELECT 
    "OrderDate", 
    COUNT("OrderNumber") as volume_vendas 
FROM "Vendas 2017"
GROUP BY "OrderDate"
ORDER BY volume_vendas DESC
LIMIT 10;
