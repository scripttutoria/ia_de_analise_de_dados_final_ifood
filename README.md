<h1 align="center">🍔 iFood Data Copilot</h1>
<h3 align="center">Agente Text-to-SQL para Operações de Food Delivery</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Streamlit-1.32+-FF4B4B?logo=streamlit&logoColor=white" />
  <img src="https://img.shields.io/badge/LangChain-0.3-1C3C3C?logo=langchain&logoColor=white" />
  <img src="https://img.shields.io/badge/Groq-LLM-F55036" />
  <img src="https://img.shields.io/badge/SQLite-3-003B57?logo=sqlite&logoColor=white" />
  <img src="https://img.shields.io/badge/Google-Colab-F9AB00?logo=googlecolab&logoColor=white" />
</p>

---

## 📖 Sobre o projeto

O **iFood Data Copilot** é um agente de análise de dados que combina **LLM (Groq) + LangChain + Streamlit** para responder perguntas de negócio em **linguagem natural** sobre operações de delivery, cardápios de redes de fast food e avaliações de restaurantes.

Em vez de escrever SQL manualmente, o analista (ou gestor de operações) digita algo como:

> *“Qual pilar operacional tem a pior média de avaliação e quantas notas 1 estrela cada um recebeu?”*

…e o agente **gera, executa e interpreta a query** automaticamente, devolvendo insights em **português do Brasil** — com recomendações acionáveis.

---

## ✨ Funcionalidades

- 🤖 **Agente Text-to-SQL** com `create_sql_agent` (LangChain) + Groq LLMs
- 🗂️ **3 bases de dados selecionáveis** na barra lateral (Delivery, Fast Food, Restaurantes)
- 📊 **Painel com KPIs em tempo real** (nº de categorias, avaliações, média geral, críticas)
- 🔍 **Inspeção das tabelas SQL** e **distribuição de notas** em gráfico
- 💬 **Chat com histórico** e reset por sessão
- 🌐 **Deploy instantâneo** via Cloudflared tunnel (link público sem precisar de conta)
- 🛡️ **Tratamento de rate-limit (429 TPM)** da Groq com *retry* automático
- 🇧🇷 **Saída 100% em PT-BR**, mesmo que os reviews estejam em ES/EN

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│  Google Colab — ia_de_analise_de_dados_final_ifood.ipynb    │
│                                                             │
│  1. Clona Dataset1 + Dataset2 (github.com/scripttutoria)    │
│  2. Baixa deelow/restaurant-reviews (Hugging Face)          │
│  3. Trata + normaliza em 3 bancos SQLite                    │
│  4. Gera app.py (Streamlit + LangChain + Groq)              │
│  5. Sobe Streamlit + Cloudflared → URL pública              │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
              ┌──────────────────────┐
              │  Navegador (público) │
              │  Painel + Chat SQL   │
              └──────────────────────┘
```

---

## 📚 Datasets utilizados

| # | Fonte | Volume | Contexto |
|---|-------|--------|----------|
| 1 | [`scripttutoria/Dataset2`](https://github.com/scripttutoria/Dataset2) | ~50 k reviews (amostra de 10 k) | Avaliações de delivery, temas como `tiempo_entrega`, `calidad_comida`, `soporte_cliente`, `precio`, etc. |
| 2 | [`scripttutoria/Dataset1`](https://github.com/scripttutoria/Dataset1) | ~700 reviews | Clientes de redes de fast food, com item favorito, gênero e data de visita |
| 3 | [`deelow/restaurant-reviews`](https://huggingface.co/datasets/deelow/restaurant-reviews) | 13 restaurantes | Nome do restaurante, menu completo e nota média |

---

## 🗃️ Modelo de dados (SQLite)

Todos os bancos seguem o **mesmo schema relacional**, garantindo que o agente SQL funcione de forma uniforme:

```sql
CREATE TABLE products (
    id_product       TEXT PRIMARY KEY,   -- "ITEM_BURGER", "REST_101", "tiempo_entrega"...
    name             TEXT NOT NULL,      -- nome amigável
    category_or_menu TEXT,               -- menu, pilar operacional ou "cardápio"
    average_rating   REAL                -- média já calculada
);

CREATE TABLE reviews (
    id                INTEGER PRIMARY KEY AUTOINCREMENT,
    rating            REAL,              -- 1 a 5
    title_review      TEXT,
    text              TEXT,              -- review completo (ES ou EN)
    user_id           TEXT,
    date              DATETIME,
    verified_purchase INTEGER,
    store             TEXT,
    id_product        TEXT REFERENCES products(id_product)
);
```

**Bancos gerados:**

| Arquivo | Conteúdo |
|---------|----------|
| `delivery_50k.db` | 7 pilares operacionais + 10.000 reviews de delivery |
| `fastfood_reviews.db` | 10 itens de cardápio + 700 reviews de clientes |
| `restaurant_reviews.db` | 13 restaurantes + 13 reviews (texto completo do menu) |

---

## 🚀 Como executar

### Opção A — Google Colab (recomendado, leva ~2 min)

1. Abra o notebook [`ia_de_analise_de_dados_final_ifood.ipynb`](./ia_de_analise_de_dados_final_ifood.ipynb) no Colab.
2. Rode **todas as células** (`Ambiente de execução → Executar tudo`).
3. A última célula imprime algo como:
   ```
   ✅ Acesse seu painel Streamlit aqui: https://xxxx.trycloudflare.com
   ```
4. Abra a URL, cole sua **API Key da Groq** na barra lateral ([obter grátis aqui](https://console.groq.com/keys)) e comece a conversar com os dados.

### Opção B — Ambiente local

```bash
# 1. Clone o repositório
git clone https://github.com/scripttutoria/ia_de_analise_de_dados_ifood.git
cd ia_de_analise_de_dados_ifood

# 2. Instale dependências
pip install streamlit datasets sqlalchemy langchain langchain-community \
            langchain-groq openpyxl requests==2.32.4

# 3. Rode o notebook uma vez para gerar os .db (ou adapte o passo 2 do notebook
#    apontando os caminhos para os .xlsx locais)
jupyter nbconvert --to notebook --execute ia_de_analise_de_dados_final_ifood.ipynb

# 4. Suba o Streamlit
streamlit run app.py
```

---

## 💡 Exemplos de perguntas

### 🛵 Base Delivery 50k
- *Qual pilar operacional (`name`) tem a pior média de avaliação e quantas notas 1 estrela cada um recebeu?*
- *Traga 5 exemplos de reclamações reais sobre `tiempo_entrega` e `errores_pedido` e sugira um plano de ação.*
- *Compare a nota média de `calidad_comida` vs `soporte_cliente` nos últimos 3 meses.*

### 🍟 Base Fast Food
- *Qual é o ranking dos itens do cardápio (`name`) por quantidade de pedidos e nota média?*
- *Quais são as principais críticas nas avaliações de nota baixa e quais pratos estão envolvidos?*

### 🍽️ Base Restaurantes
- *Quais restaurantes têm pratos com "chicken" no cardápio (`category_or_menu`)?*
- *Qual restaurante tem a pior nota e qual foi a reclamação?*

---

## 🤖 Modelos suportados (Groq)

| Modelo | TPM (free tier) | Recomendado para |
|--------|-----------------|------------------|
| `openai/gpt-oss-120b` | 8 000 | Melhor raciocínio SQL |
| `llama-3.3-70b-versatile` | 12 000 | Equilíbrio velocidade/qualidade |
| `openai/gpt-oss-20b` | 8 000 | Alternativa leve |
| `llama-3.1-8b-instant` | 6 000 | Respostas rápidas |
| `gemma2-9b-it` | 15 000 | Maior folga de tokens |

> ⚠️ **Aviso:** O plano gratuito da Groq tem limites de **TPM (tokens por minuto)**. O app já detecta erro `429` e faz *retry* automático, mas se persistir, troque o modelo na barra lateral.

---

## 🧰 Stack técnica

| Camada | Tecnologia |
|--------|-----------|
| Interface | Streamlit |
| Orquestração LLM | LangChain (`create_sql_agent`, tool-calling) |
| LLM | Groq (GPT-OSS, Llama 3.x, Gemma 2) |
| Dados | Pandas, NumPy, openpyxl |
| Persistência | SQLite + SQLAlchemy |
| Datasets | `datasets` (Hugging Face) + GitHub |
| Deploy demo | Cloudflared tunnel |
| Execução | Google Colab |

---

## 📁 Estrutura do repositório

```
ia_de_analise_de_dados_ifood/
├── ia_de_analise_de_dados_final_ifood.ipynb   # notebook principal (Colab)
├── app.py                                     # gerado pelo notebook
├── README.md
├── requirements.txt
└── .gitignore                                 # ignora *.db, logs, cloudflared
```

> ℹ️ Os arquivos `.db` **não devem ser versionados** — eles são regenerados automaticamente pelo notebook a partir dos datasets públicos.

Sugestão de `.gitignore`:

```gitignore
*.db
cloudflared
streamlit_logs.txt
cloudflare_logs.txt
__pycache__/
.ipynb_checkpoints/
.env
```

---

## 🗺️ Roadmap

- [ ] Autenticação básica no Streamlit
- [ ] Download dos resultados em CSV/Excel
- [ ] Cache de queries SQL recorrentes
- [ ] Suporte a Postgres/BigQuery
- [ ] Dashboard com gráficos Plotly interativos
- [ ] Testes unitários do pipeline de ETL

---

## 👤 Autor

**scripttutoria**
🔗 [github.com/scripttutoria](https://github.com/scripttutoria)

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja `LICENSE` para mais informações.
