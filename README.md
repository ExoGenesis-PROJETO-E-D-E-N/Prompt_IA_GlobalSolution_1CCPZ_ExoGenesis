#  Projeto E.D.E.N.
### *Ecological Development in Exo-Environments*
### Desenvolvimento Ecológico em Exoambientes

**Global Solution 2026 — FIAP | Tecnologia Espacial Aplicada a Desafios Reais**

---

## Integrantes

| Nome | RM |
|Arthur Vettorazzo de Souza | RM 569445 |
|Brayan Barbosa Dos Santos | RM 573682 |
| Giovanne Gomes Petenuci | RM 574091 |

**Turma:** *1CCPZ* — Ciências da Computação — FIAP

---

## Problema Abordado

Com o avanço do aquecimento global e o risco de extinção de espécies nativas
da Mata Atlântica, o Brasil enfrenta uma crise ambiental sem precedentes.
As soluções convencionais de reflorestamento têm limitações conhecidas:
mudas pouco resilientes, baixa taxa de sobrevivência em solos degradados e
vulnerabilidade a eventos climáticos extremos cada vez mais frequentes.

Ao mesmo tempo, a nova corrida espacial abre uma fronteira científica inédita:
o uso do ambiente orbital como laboratório para acelerar a evolução vegetal.

---

## Proposta do Projeto E.D.E.N.

O **Projeto E.D.E.N.** é uma biocápsula/estufa botânica 100% automatizada
enviada à órbita baixa da Terra (LEO). Seu objetivo é expor sementes e mudas
da Mata Atlântica a picos controlados de estresse espacial — microgravidade
e doses calculadas de radiação cósmica — para induzir micro-mutações genéticas
que gerem espécies hiper-resilientes.

### Impacto esperado

| ODS | Conexão com o E.D.E.N. |
|---|---|
| **ODS 13** — Ação Climática | Plantas hiper-resilientes para reflorestamento da Mata Atlântica |
| **ODS 2** — Fome Zero | Culturas alimentares adaptadas a ambientes extremos |
| **ODS 9** — Indústria e Inovação | Sistema autônomo de IA para controle de missão espacial |
| **ODS 11** — Cidades Sustentáveis | Tecnologia espacial aplicada ao monitoramento ambiental |

### O Mission Control AI

Como não há humanos a bordo, o **Mission Control AI** é o único cérebro
operacional da missão. A IA deve equilibrar dois objetivos conflitantes:

- **Permitir o estresse controlado** — exposição à microgravidade e radiação
  é o objetivo científico; interrompê-la desnecessariamente é falha de missão.
- **Intervir em estresse letal** — quando os sensores indicam risco de morte
  iminente das amostras, a IA deve agir imediatamente.

---

## Tecnologias e Justificativa Técnica

| Tecnologia | Função | Justificativa |
|---|---|---|
| **Llama 3.1 8B Instruct** (HuggingFace API) | Modelo de linguagem principal | Melhor desempenho em raciocínio multi-etapa (CoT) e structured output dentre os modelos gratuitos disponíveis |
| **HuggingFace InferenceClient** | Interface com o LLM via API | Padrão da disciplina; suporte a múltiplos modelos sem mudança de código |
| **Kaggle** | Plataforma de desenvolvimento | GPU gratuita; Kaggle Secrets para gestão segura de tokens |
| **Python** | Linguagem de desenvolvimento | Stack padrão da disciplina |

### Por que Llama 3.1 8B e não um modelo menor?

A escolha foi guiada pela **complexidade do raciocínio exigido**, não pelo
tamanho do modelo. O E.D.E.N. utiliza Chain-of-Thought com 6 passos
interdependentes, análise de tendências históricas multi-variável e geração
de JSON estruturado com schema fixo. Modelos compactos como TinyLlama (1.1B)
executam essas tarefas com qualidade significativamente inferior —
especialmente em CoT longo e structured output.

---

## Decisões Arquiteturais

### Por que Prompt Augmentation e não RAG?

Esta foi uma decisão consciente baseada na **natureza dos dados da missão**:

| Critério | Projeto E.D.E.N. | Quando RAG seria adequado |
|---|---|---|
| Tipo de dado | Telemetria dinâmica (muda por ciclo orbital) | Documentos estáticos (PDFs, manuais extensos) |
| Volume por consulta | ~5 valores numéricos por ciclo | Centenas de páginas |
| Disponibilidade | Conhecidos no momento da chamada ao LLM | Precisam ser recuperados de um índice vetorial |
| Necessidade de retrieval |  Dados já estão disponíveis |  Necessário buscar o trecho relevante |

Como os dados de telemetria são **compactos e já conhecidos** no momento
da inferência, a injeção direta no prompt (Prompt Augmentation) é mais
eficiente, mais rápida e igualmente precisa. RAG seria uma camada de
complexidade desnecessária para este caso de uso.

> **Regra aplicada:** use RAG para dados estáticos e volumosos; use Prompt
> Augmentation para dados dinâmicos já conhecidos no momento da inferência.

### Por que execução Online (Inference API) e não Local?

| Critério | Online — nossa escolha | Local — descartado |
|---|---|---|
| **Privacidade** |  Aceitável — dados são simulados, sem informação sensível | Necessário apenas para dados confidenciais reais |
| **Recursos** |  Processamento nos servidores HuggingFace | Llama 3.1 8B exige ~16GB VRAM — no limite da T4 |
| **Estabilidade** |  Sem risco de OOM no notebook | Download de ~16GB pode travar a sessão |
| **Foco** |  Foco total em Prompt Engineering | Gerenciamento de pipeline local desvia do objetivo |

A execução local faria sentido se os dados fossem reais e confidenciais.
No E.D.E.N., como os dados são simulados para fins acadêmicos, o
custo-benefício favorece claramente a API remota.

---

##  Técnicas de Prompt Engineering Aplicadas

| Técnica | Módulo | Justificativa |
|---|---|---|
| **System Prompt** | Todos | Define persona, filosofia de missão, limiares técnicos e formato de saída |
| **Chain-of-Thought (CoT)** | Status + Previsão | Força raciocínio passo a passo; reduz alucinações em dados multi-variável interdependentes |
| **Few-Shot Prompting** | Recomendações | 3 exemplos históricos calibram escala de urgência e padrão de resposta |
| **Function Calling / Structured Output** | Relatório Executivo | LLM retorna JSON estruturado — diferencial para integração com sistemas externos |

### Justificativa dos hiperparâmetros por módulo

| Módulo | Temperature | Top-P | Justificativa |
|---|---|---|---|
| Análise de Status | **0.2** | 0.85 | Análise técnica exige determinismo — menor risco de inventar valores de sensores |
| Recomendações | **0.4** | 0.88 | Ações exigem criatividade situacional; few-shot ancora o formato |
| Relatório JSON | **0.1** | 0.80 | JSON estruturado exige saída determinística e tokens válidos |
| Previsão de Falha | **0.35** | 0.90 | Extrapolação de tendências requer raciocínio analítico com leve criatividade |

---

##  Segurança

O token HuggingFace é carregado **exclusivamente via Kaggle Secrets**,
nunca hardcoded no código. Isso garante que a chave não apareça no
notebook publicado, em logs de execução ou histórico de commits.

```python
# CORRETO
HF_TOKEN = user_secrets.get_secret("HF_TOKEN")

# NUNCA faça isso
HF_TOKEN = "hf_xxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

---

## 📁 Arquivos do Repositório

| Arquivo | Descrição |
|---|---|
| `README.md` | Este documento |
| `EDEN_Mission_Control_AI.ipynb` | Notebook principal com implementação completa |
| `system_prompt.txt` | System prompt base do Mission Control AI |
| `modelo_de_teste.txt` | 7 cenários de teste com situações e respostas esperadas |
| `entrega.txt` | Arquivo oficial de entrega com integrantes e links |
