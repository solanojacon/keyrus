# CLAUDE.md

Contexto e decisões do projeto, para persistir entre sessões. Ver também `README.md` (uso) e `docs/` (enunciado e documentação do dataset).

## Contexto

Desafio técnico de modelagem em Python para a vaga de Data Scientist na Keyrus. Prazo: 7 dias corridos a partir de 2026-09-10. Roteiro completo em `docs/Teste_Modelagem_Python.pdf`; a apresentação final não faz parte deste repositório.

## Diretrizes de trabalho combinadas

- Registrar aqui, à medida que forem tomadas, as decisões relevantes do projeto.
- Manter README.md e .gitignore atualizados.
- Boas práticas: reprodutibilidade, seeds fixas, código limpo e comentado, estrutura de pastas consistente.
- "Menos é mais": simplicidade acima de acúmulo — evitar abstrações, arquivos e dependências além do necessário para a tarefa em andamento.
- Nunca desfazer/reinterpretar decisões de sessões anteriores sem avisar antes.
- Na dúvida, perguntar antes de assumir.
- Sinceridade acima de concordância: se houver uma abordagem melhor que a pedida, avisar antes de executar.

## Decisões tomadas

- **Um único notebook** (`bank_marketing.ipynb`) para todo o desafio, na raiz do repo — conforme opção dada pelo enunciado (notebook ou script) e preferência do candidato. Reavaliar (múltiplos notebooks/scripts) apenas se a complexidade do projeto exigir; sinalizar antes de mudar.
- Tarefas do roteiro do PDF são desenvolvidas **uma de cada vez, na ordem do enunciado**.
- **Dados carregados direto do Kaggle** via `kagglehub` (`kagglehub.dataset_load` com `KaggleDatasetAdapter.PANDAS`) a cada execução — não há CSVs versionados no repo. Dataset: `abdelazizsami/bank-marketing`, arquivos `bank-full.csv` (45.211 linhas) e `bank.csv` (amostra de 10%, 4.521 linhas), carregados em dois DataFrames Pandas separados.
  - Os CSVs usam `;` como separador (herdado do dataset original UCI/Moro et al.) — passado explicitamente via `pandas_kwargs={"sep": ";"}`. Validado com uma asserção de 17 colunas após a leitura.
  - Testado sem credenciais do Kaggle configuradas e funcionou (dataset público). Caso falhe por autenticação em outro ambiente, configurar `~/.kaggle/access_token`, `KAGGLE_API_TOKEN` ou `~/.kaggle/kaggle.json` — nunca versionar, já cobertos pelo `.gitignore`.
- **Gerenciamento de dependências com `uv`** (`pyproject.toml` + `uv.lock`), Python `>=3.12` (evita a incerteza de compatibilidade do stack de ciência de dados com Python 3.14, que é muito recente).
- Dependências adicionadas apenas conforme a tarefa em andamento passa a exigi-las (ex.: `pandas`, `kagglehub[pandas-datasets]`, `jupyter` para a etapa de carregamento de dados) — nada de instalar todo o stack (sklearn, matplotlib, etc.) adiantado.
- `RANDOM_SEED = 42` definida no topo do notebook, para ser reutilizada em toda etapa que envolva aleatoriedade (split treino/teste, inicialização de modelo, etc.).
- Notebook validado de ponta a ponta com `jupyter nbconvert --execute` usando o `.venv` do projeto: `bank-full.csv` → 45.211 linhas × 17 colunas, `bank.csv` → 4.521 linhas × 17 colunas, ambos batendo com `docs/bank-names.txt`.
- **Nota importante:** o campo "Missing Attribute Values" do dataset original diz "None", mas isso é enganoso — `job`, `education`, `contact` e `poutcome` usam a categoria `"unknown"`, e `pdays = -1` é sentinela de "nunca contatado antes". Tratar isso explicitamente na etapa de EDA/tratamento de ausentes.
- **`bank_full_df` é o dataset principal de todo o projeto** a partir da tarefa de EDA (inclusive pré-processamento e modelagem). `bank_df`/`bank.csv` é carregado na subseção 1.1 mas não é usado depois disso — a própria documentação do dataset o descreve como amostra de 10% só para testar algoritmos computacionalmente caros (ex. SVM), não como base de um projeto completo.
- O carregamento dos dados (`kagglehub`) faz parte da subseção **1.1** do notebook, não de uma seção "0" separada — é o primeiro item do próprio roteiro da EDA ("carregar o conjunto de dados e exibir estatísticas básicas e tipos de dados").
- Tarefa de EDA concluída (`## 1.` no notebook, subseções 1.1–1.4, uma por item do roteiro). Decisões da EDA:
  - Valores ausentes: `"unknown"` (job/education/contact/poutcome) e `pdays = -1` tratados como categoria/sentinela com significado próprio — sem imputação/remoção nesta etapa. Confirmado que `pdays == -1` e `previous == 0` coincidem (36.954 linhas cada) e `poutcome == "unknown"` é quase idêntico (36.959).
  - Visualizações com `matplotlib` + `seaborn` (adicionadas ao projeto via `uv add`) — únicas libs de plotagem usadas.
  - `y` é desbalanceada (~88% "no" / ~12% "yes") — vai pesar na escolha de métricas na tarefa de Avaliação do Modelo.
  - **Achado importante — vazamento de dado em `duration`:** é a variável mais correlacionada com `y`, mas só é conhecida depois que a ligação termina, não estando disponível no momento de decidir quem contatar. Retomar isso nas tarefas de Pré-processamento/Modelagem/Produção (ex.: treinar/comparar versão sem `duration` para um modelo realista).
- Tarefa de Pré-processamento concluída (`## 2.` no notebook, subseções 2.1–2.2). Decisões:
  - **`duration` excluída do conjunto de features em todo o projeto a partir daqui** (modelagem, avaliação, produção) — decisão confirmada com o usuário para resolver o vazamento de dado encontrado na EDA. Um único pipeline, sem duplicar com uma versão "com duration".
  - Encoding/scaling: `default`/`housing`/`loan` (binárias) mapeadas para 1/0; `job`/`marital`/`education`/`contact`/`month`/`poutcome` (nominais, sem ordem natural clara por causa do `"unknown"`) via `OneHotEncoder`; numéricas (exceto `duration`) via `StandardScaler` — preferido a min-max pelos outliers fortes em `balance`/`campaign`/`previous` vistos na EDA. Tudo combinado num único `ColumnTransformer` (`preprocessor`).
  - **Desvio deliberado da ordem literal do PDF:** o roteiro lista "codificar/normalizar/padronizar" antes de "separar treino/teste", mas ajustar (`fit`) os transformadores na base inteira antes do split vazaria estatísticas do teste para o treino. Por isso a subseção 2.1 só define/justifica a estratégia (monta o `ColumnTransformer` sem ajustar); o `fit_transform` de fato só acontece na 2.2, depois do `train_test_split`, ajustado só no treino e aplicado (`transform`) no teste. Isso está explicado em markdown no próprio notebook.
  - Split 80/20 estratificado por `y` (`train_test_split(..., stratify=y, random_state=RANDOM_SEED)`), por causa do desbalanceamento visto na EDA — confirmado que a proporção de `y=1` (11,7%) se manteve igual em treino e teste.
  - `X_train_proc`/`X_test_proc` mantidos como `DataFrame` (via `preprocessor.get_feature_names_out()`), não array numpy puro, para preservar nomes de coluna legíveis nas tarefas de Modelagem e Interpretação (feature importance).
  - Dependência nova: `scikit-learn`.
- Tarefa de Construção de Modelos concluída (`## 3.` no notebook, subseções 3.1–3.2). Decisões:
  - Comparados 4 candidatos nativos do `scikit-learn` (sem `xgboost`/`lightgbm`) via `cross_val_score` (ROC-AUC, `StratifiedKFold(5)`, só em `X_train_proc`/`y_train`): `DummyClassifier` (piso, AUC 0,50), `LogisticRegression` (0,764), `RandomForestClassifier` (0,778), `HistGradientBoostingClassifier` (0,797 — **vencedor**).
  - Desbalanceamento tratado com `class_weight="balanced"` nos 3 modelos reais — evita depender de `imbalanced-learn`/SMOTE (nova dependência).
  - ROC-AUC usada aqui é só para *selecionar/otimizar* o modelo (métrica threshold-independent, tolerante a desbalanceamento). Métricas de negócio (precisão/recall/limiar) ficam para a tarefa de Avaliação do Modelo.
  - Hiperparâmetros otimizados com `RandomizedSearchCV` (n_iter=20, mesmo `StratifiedKFold(5)`, scoring ROC-AUC) em vez de `GridSearchCV` — busca exaustiva seria cara no espaço de hiperparâmetros do `HistGradientBoostingClassifier`. Melhor combinação: `min_samples_leaf=20, max_leaf_nodes=31, max_iter=200, learning_rate=0.03, l2_regularization=10.0`, ROC-AUC (CV) ≈ 0,7999.
  - `final_model = search.best_estimator_` (já treinado com os melhores parâmetros) é a variável reaproveitada nas tarefas de Avaliação, Interpretação e Produção. `X_test_proc`/`y_test` não foram tocados nesta tarefa.
