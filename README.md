# 🏠 Previsão de Preços de Imóveis no Airbnb em Sydney

> **SME0828 — Introdução à Ciência de Dados**  
> **ICMC - USP (Instituto de Ciências Matemáticas e de Computação — Universidade de São Paulo)**  
> **Docente:** Prof. Francisco Rodrigues  

---

## 📌 Visão Geral do Projeto

Este projeto consiste na construção e avaliação de um pipeline completo de Ciência de Dados e Aprendizado de Máquina para prever o preço de diárias de imóveis anunciados no **Airbnb na cidade de Sydney, Austrália**.

O fluxo de trabalho contempla desde a ingestão e tratamento de dados brutos até a engenharia de atributos, validação cruzada estratificada, regularização linear, modelos não lineares baseados em árvores com ajuste de hiperparâmetros, avaliação rigorosa em conjunto de teste e uma **aplicação prática de precificação imobiliária** para um imóvel de alto padrão localizado na famosa praia de **Bondi Beach**.

---

## 🚀 Etapas do Pipeline

O projeto segue rigorosamente o roteiro estruturado em 12 etapas:

1. **Obtenção dos Dados:**
   * Carregamento do dataset público de anúncios de Sydney hospedado no GitHub (~27 mil registros e 84 atributos originais).
2. **Limpeza dos Dados:**
   * Conversão de tipos de atributos monetários (remoção de símbolos cifrão `$` e vírgulas em `price`, `cleaning_fee`, `security_deposit`, etc.).
   * Conversão de booleanos (`t`/`f`) e cálculo da antiguidade do anfitrião (`host_since_days`).
   * Tratamento de valores ausentes (imputação com valor zero para taxas inexistentes e mediana/moda nas demais variáveis).
   * Tratamento de *outliers* na variável alvo (`price`), filtrando entre os percentis 1% e 99% para evitar distorções nos regressores.
3. **Análise Exploratória de Dados (EDA) & Estatística Descritiva:**
   * Distribuição de frequências e estatísticas descritivas (média, mediana, desvio padrão, quartis).
   * Visualização da distribuição de preços por categorias (`room_type`, `property_type`, `cancellation_policy`, `host_is_superhost`).
   * Matriz de correlação com *heatmap* e análise das variáveis com maior impacto no preço.
   * Mapa de calor geográfico de Sydney plotando latitude e longitude em função do valor da diária.
4. **Divisão Treino/Teste Estratificada:**
   * Separação estrita dos dados antes de qualquer decisão de modelagem (80% treino / 20% teste).
   * Estratificação por faixas de preço (`price_cat`) utilizando `StratifiedShuffleSplit`, garantindo proporção idêntica de imóveis econômicos e de luxo em ambos os conjuntos.
5. **Engenharia de Atributos:**
   * Criação de atributos sintéticos: `total_extra_cost` (soma de taxas de limpeza e depósito caução) e `host_since_days`.
   * Análise crítica de multicolinearidade e alta cardinalidade: remoção justificada de `city` (alta dispersão espacial substituída pelas coordenadas geográficas contínuas de latitude/longitude) e sub-scores de avaliação redundantes.
   * Agrupamento de categorias raras de `property_type` (frequência < 100 anúncios agregados sob o rótulo `'Other'`).
6. **Pipeline de Pré-processamento:**
   * Integração via `ColumnTransformer` do Scikit-Learn:
     * **Atributos Numéricos:** Imputação pela mediana (`SimpleImputer`) seguida de padronização (`StandardScaler`).
     * **Atributos Categóricos:** Imputação pela moda seguida de codificação *One-Hot* (`OneHotEncoder`).
7. **Modelos Supervisionados com Ajuste de Hiperparâmetros:**
   * **Regressão Linear Múltipla:** Modelo *baseline*.
   * **Árvore de Decisão (`DecisionTreeRegressor`):** Otimizada via `GridSearchCV` ajustando `max_depth`, `min_samples_split` e `min_samples_leaf`.
   * **Floresta Aleatória (`RandomForestRegressor`):** Otimizada via `RandomizedSearchCV` variando número de estimadores, profundidade e critérios de divisão de nós.
8. **Regularização:**
   * **Ridge (L2):** Penalização quadrática para controle de coeficientes e multicolinearidade.
   * **Lasso (L1):** Penalização com valor absoluto, promovendo seleção automática de atributos (*sparsity*).
   * **Elastic Net (L1 + L2):** Combinação convexa das penalidades L1 e L2.
9. **Comparação de Modelos via Validação Cruzada:**
   * Comparação do desempenho em 5 *folds* utilizando as métricas **MAE** (Erro Médio Absoluto), **RMSE** (Raiz do Erro Quadrático Médio) e **$R^2$** (Coeficiente de Determinação).
10. **Avaliação no Conjunto de Teste:**
    * Aplicação do melhor modelo eleito na validação cruzada sobre os dados de teste (dados inéditos).
    * Cálculo de métricas finais: **MAE**, **RMSE**, **MAPE** e **$R^2$**, além da análise visual da dispersão dos resíduos.
11. **Importância das Variáveis:**
    * Extração e ordenação do *Feature Importance*, identificando os atributos de maior peso na formação do preço (número de quartos, capacidade de acomodação, banheiros, localização e taxas extras).
12. **Aplicação Prática — Estimação de Preço Justo para Bondi Beach:**
    * Avaliação do caso real de um anfitrião em Bondi Beach que cobra atualmente **US$ 500.00 / noite**.
    * Estimativa pontual do preço justo, cálculo do erro de precificação e definição de intervalo de tolerância com base na margem de erro ($\pm MAE$) do modelo.

---

## 📊 Resultados e Comparação dos Modelos

### Desempenho na Validação Cruzada (5-Fold CV)

| Modelo | MAE (USD) ↓ | RMSE (USD) ↓ | $R^2$ ↑ |
|---|:---:|:---:|:---:|
| **Floresta Aleatória (Random Forest)** 🏆 | **~$56.80** | **~$96.40** | **~0.73** |
| **Árvore de Decisão (Otimizada)** | ~$68.20 | ~$115.10 | ~0.61 |
| **Ridge Regression (L2)** | ~$72.40 | ~$119.80 | ~0.58 |
| **Regressão Linear (Baseline)** | ~$72.50 | ~$120.00 | ~0.58 |
| **Lasso Regression (L1)** | ~$72.50 | ~$120.10 | ~0.58 |
| **Elastic Net (L1 + L2)** | ~$72.60 | ~$120.20 | ~0.58 |

> **Conclusão:** O modelo baseado em conjunto de árvores (**Random Forest**) superou significativamente os modelos lineares, capturando interações não lineares complexas entre localização geográfica (latitude/longitude), tamanho do imóvel e taxas de comodidade.

### Métricas Finais no Conjunto de Teste (Dados Inéditos)

* **MAE (Mean Absolute Error):** $\approx \text{US\$} 58.00$
* **RMSE (Root Mean Squared Error):** $\approx \text{US\$} 98.32$
* **MAPE (Mean Absolute Percentage Error):** $\approx 33.9\%$
* **$R^2$ (Variância Explicada):** $\approx 0.72$

---

## 🏖️ Estudo de Caso Prático: Bondi Beach

### Perfil do Imóvel
* **Localização:** Bondi Beach (`lat = -33.889087`, `lon = 151.274506`)
* **Capacidade:** Até 10 hóspedes | 5 quartos | 7 camas | 3 banheiros
* **Tipo:** Casa inteira (`House`, `Entire home/apt`)
* **Regras & Taxas:** Estadia mínima de 4 noites | Taxa de limpeza de US$ 370 | Depósito caução de US$ 1.500
* **Reputação:** Nota média de 95 (53 avaliações) | Anfitrião *Superhost* e Verificado desde agosto de 2010
* **Disponibilidade:** 255 de 365 dias | Política de cancelamento estrita (14 dias)

### Diagnóstico de Precificação e Análise do Erro

```text
=================================================================
🏖️  ESTIMATIVA DE PREÇO E ANÁLISE DE ERRO — BONDI BEACH
=================================================================
  Preço cobrado atualmente:          US$ 500.00 / noite
  Preço justo estimado pelo modelo:  US$ 709.10 / noite
-----------------------------------------------------------------
  MÉTRICAS DE ERRO NO RESULTADO FINAL:
  • Erro absoluto de precificação:    US$ 209.10 / noite
  • Erro percentual relativo:         29.5% a menos
  • Margem de erro do modelo (MAE):   ± US$ 58.00 / noite
  • Intervalo plausível com erro:     [US$ 651.10 — US$ 767.10]
-----------------------------------------------------------------
  📊 Diagnóstico: O imóvel está SUBVALORIZADO pelo anfitrião.
     Mesmo considerando o piso mais conservador da margem de erro
     do modelo (US$ 651.10), o valor atual de US$ 500.00 continua
     abaixo do preço de mercado.
  💡 Recomendação: O anfitrião pode aumentar a diária para a faixa
     de US$ 650.00 a US$ 760.00 mantendo alta competitividade
     para o padrão de casa de praia para 10 pessoas.
```

---

## 🛠️ Tecnologias e Bibliotecas Utilizadas

* **Python 3.10+**
* **Pandas:** Manipulação, limpeza e análise tabular de dados
* **NumPy:** Computação numérica e operações vetoriais
* **Scikit-Learn:** Pré-processamento (`ColumnTransformer`, `StandardScaler`, `OneHotEncoder`), modelos de regressão, regularização e validação cruzada (`GridSearchCV`, `RandomizedSearchCV`, `StratifiedShuffleSplit`)
* **Matplotlib & Seaborn:** Visualizações estatísticas, matrizes de correlação e mapas de dispersão
* **SciPy:** Estatística descritiva e distribuições de amostragem

---

## 📂 Estrutura do Repositório

```bash
├── projetocd.ipynb          # Notebook Jupyter principal com todo o código executável e análises
├── README.md                # Documentação completa do projeto
└── requirements.txt         # Lista de dependências do ambiente Python
```

---

## 💻 Como Reproduzir o Projeto Localmente

1. **Clone o repositório:**
   ```bash
   git clone https://github.com/seu-usuario/airbnb-sydney-pricing.git
   cd airbnb-sydney-pricing
   ```

2. **Crie e ative um ambiente virtual:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate   # No Windows: .venv\Scripts\activate
   ```

3. **Instale as dependências:**
   ```bash
   pip install -r requirements.txt
   ```
   *(Ou instale diretamente: `pip install pandas numpy scikit-learn matplotlib seaborn scipy jupyter`)*

4. **Inicie o Jupyter Notebook:**
   ```bash
   jupyter notebook projetocd.ipynb
   ```
   *Abra o arquivo `projetocd.ipynb` e execute as células sequencialmente (`Run All`). O dataset é baixado automaticamente do link público do GitHub.*

---

## 🎓 Autoria e Créditos

Projeto desenvolvido como parte da disciplina **SME0828 — Introdução à Ciência de Dados** do **ICMC - USP**, ministrada pelo **Prof. Francisco Rodrigues**.

