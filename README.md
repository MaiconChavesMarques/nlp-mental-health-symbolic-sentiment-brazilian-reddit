# Análise de Suporte em Comunidades de Saúde Mental no Reddit Brasileiro

> Análise de Sentimentos baseada em regras linguísticas aplicada a comentários em PT-BR dos subreddits r/AnsiedadeDepressao e r/desabafos.

**Grupo:** Zero — 1ª Etapa do Projeto Prático (PLN)

| Nome | NUSP |
|---|---|
| Renato Calacina Spessotto | 14605824 |
| Edvandro Rodrigues Pereira | 18046338 |
| Guilherme Pascoale Godoy | 14576277 |
| Henrique Drago | 14675441 |
| Karl Cruz Altenhofen | 14585976 |
| Maicon Chaves Marques | 14593530 |

---

## Sobre o Projeto

Este trabalho implementa um pipeline de **análise de sentimentos sem aprendizado de máquina**, inteiramente baseado em regras linguísticas e léxicos de polaridade para o português brasileiro. O domínio escolhido são comunidades de saúde mental do Reddit em PT-BR, onde usuários compartilham relatos de ansiedade e depressão.

O objetivo é classificar cada comentário como **Positivo**, **Negativo** ou **Neutro** a partir da composição de:

- normalização e limpeza do texto;
- anotação morfossintática (POS Tagging) e parsing de dependências;
- consulta a léxicos de polaridade (OpLexicon e SentiLex-PT);
- aplicação de regras de negação, intensificação e atenuação.

---

## Pipeline

```
Texto bruto
    │
    ▼
Normalização
(lower, emojis, abreviações, correção ortográfica, números)
    │
    ▼
Tokenização (NLTK)
    │
    ▼
POS Tagging + Parsing de Dependências (spaCy pt_core_news_sm)
    │
    ▼
Léxico de Polaridade (OpLexicon / SentiLex-PT)
+ Regras Linguísticas (negação, intensificadores, atenuadores)
    │
    ▼
Score → Classificação (Positivo / Negativo / Neutro)
```

---

## Fontes de Dados

| Subreddit | Tamanho | Método de Coleta |
|---|---|---|
| r/AnsiedadeDepressao | ~3 mil membros | PRAW (Reddit API) |
| r/desabafos | ~561 mil membros | Dumps do Pushshift Reddit |

Foram coletados posts com mais de 10 comentários (AnsiedadeDepressao) e posts flairados como "Depressão" com mais de 52 comentários (desabafos). Os comentários passaram por filtragem para remover entradas deletadas, comentários do próprio autor da publicação e ruído, resultando em um corpus de aproximadamente **20 mil comentários**.

> **Nota:** as credenciais da API do Reddit e os dumps do Pushshift (>10 GB) não estão incluídos no repositório. Os scripts de coleta estão presentes no notebook de forma comentada apenas para documentação.

Os dados processados estão disponíveis via Google Drive e são baixados automaticamente pelo notebook com `gdown`.

---

## Arquivos do Repositório

```
.
├── PLN_Trabalho_01.ipynb   # Notebook principal com todo o pipeline
└── Relatorio.pdf           # Relatório completo do projeto
```

---

## Dependências

```bash
pip install gdown symspellpy num2words nltk spacy pandas
python -m spacy download pt_core_news_sm
```

Recursos NLTK necessários (baixados automaticamente no notebook):

```python
nltk.download("punkt")
nltk.download("punkt_tab")
```

---

## Arquivos de Dados Externos

O notebook baixa automaticamente os seguintes arquivos via `gdown`:

| Arquivo | Descrição |
|---|---|
| `posts_AnsiedadeDepressao.json` | Posts coletados do r/AnsiedadeDepressao |
| `comentarios_AnsiedadeDepressao.json` | Comentários do r/AnsiedadeDepressao |
| `posts_desabafos.json` | Posts coletados do r/desabafos |
| `comentarios_desabafos.json` | Comentários do r/desabafos |
| `abreviacoes.json` | Dicionário de abreviações em PT-BR |
| `emojis_ptbr.json` | Mapeamento de emojis para descrições em PT-BR |
| `formas.saocarlos.txt` | Dicionário para correção ortográfica (SymSpell) |
| `SentiLex-lem-PT02.txt` | Léxico de sentimentos SentiLex-PT |
| `lexico_v3.0.txt` | Léxico de sentimentos OpLexicon |

---

## Etapas do Notebook

### Etapa 0 — Banco de Dados
Coleta e filtragem dos comentários dos dois subreddits, gerando o arquivo `corpus.json` com ~20 mil entradas.

### Etapa 1 — Normalização
Pré-processamento do texto com as seguintes operações (todas configuráveis via parâmetros booleanos):

- Conversão para minúsculas
- Substituição ou remoção de emojis
- Redução de letras repetidas (ex: `muitooo` → `muito`)
- Normalização de pontuação
- Tokenização com NLTK
- Expansão de abreviações (ex: `vc` → `você`)
- Conversão de números para texto (ex: `10` → `dez`)
- Correção ortográfica com SymSpell
- Lematização com spaCy (opcional)

### Etapa 2 — Análise Linguística
- **POS Tagging:** anotação morfossintática com spaCy, salvando lema, POS e tag de cada token.
- **Parsing de Dependências:** extração das relações sintáticas entre tokens (negação, modificação adverbial, etc.).
- **Construção do Léxico:** carregamento e filtragem do OpLexicon (ADJ e VB) e do SentiLex-PT (ADJ, V, N), com lógica de prioridade de polaridade para o domínio de desabafos.

### Atribuição do Score
Cálculo do sentimento com base em:

- Polaridade léxica ponderada por classe gramatical (ADJ > NOUN > VERB)
- Negadores: invertem o sinal (`não`, `nunca`, `jamais`, etc.)
- Intensificadores: multiplicam o score (`muito` ×2.0, `extremamente` ×2.5, etc.)
- Atenuadores: reduzem o score (`quase` ×0.5, `ligeiramente` ×0.5, etc.)
- Pontuação exclamativa: bônus de ×1.2
- Verbos copulativos e modais neutros são desconsiderados

Classificação final:
- `score > 0.4` → **Positivo**
- `score < -0.4` → **Negativo**
- caso contrário → **Neutro**

O pipeline é executado separadamente com OpLexicon e com SentiLex-PT para comparação dos resultados.
