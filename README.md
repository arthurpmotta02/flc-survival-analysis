# flc-survival-analysis

**Cadeia Leve Livre Sérica e Mortalidade por Todas as Causas**

> Disciplina: Análise de Sobrevivência (2026/1) — DME/IM-UFRJ  
> Autor: Arthur Pontes Motta  
> Professora: Marina Silva Paez

---

## Relatório interativo

O trabalho está publicado como página web com código expansível, gráficos e tabelas completas:

### **[https://arthurpmotta02.github.io/flc-survival-analysis/](https://arthurpmotta02.github.io/flc-survival-analysis/)**

---

## Sobre o trabalho

Análise estatística completa do conjunto de dados `flchain` (pacote `survival` do R), investigando a associação entre a concentração sérica de cadeia leve livre (FLC) e a mortalidade por todas as causas em adultos com 50 anos ou mais do Condado de Olmsted, Minnesota (EUA), a partir de um estudo de coorte de base populacional conduzido desde 1995.

**Pergunta de interesse:** A concentração sérica de cadeia leve livre está associada ao tempo de sobrevivência após controle por idade, sexo, creatinina sérica e diagnóstico de MGUS?

Os dados cobrem **7.871 indivíduos**, **2.166 óbitos** (27,5%) e até **14,3 anos** de seguimento. A covariável principal é o grupo de FLC total (decis 1–10 agrupados em quatro categorias: Baixo, Médio-baixo, Médio-alto e Alto).

---

## Estrutura do trabalho

| Seção | Conteúdo |
|---|---|
| Introdução | Contexto clínico da FLC, adequação para análise de sobrevivência, definições formais de $S(t)$, $h(t)$ e $H(t)$ |
| Análise exploratória | Tabela descritiva por desfecho, distribuição do tempo de seguimento, variáveis laboratoriais, valores ausentes em creatinina, causas de óbito por grupo |
| Curvas de Kaplan-Meier | Global, por grupo de FLC, por sexo e faixa etária, tabela de $\hat{S}(t)$ em tempos fixos |
| Nelson-Aalen | Função de risco acumulada por grupo, diagnóstico de adequação distribucional |
| Subgrupo MGUS | Curvas KM e tabela de sobrevivência por status de MGUS |
| Seleção de covariáveis | Procedimento de Collett (4 passos) + `stepAIC` + TRV sequencial |
| Modelo de Cox | Equação formal, HRs ajustados, forest plot, cinco diagnósticos completos |
| Modelos paramétricos AFT | 6 distribuições via `flexsurv`, AIC/BIC/LRT, parâmetro $Q$ da gama generalizada |
| Resultados e Conclusão | Síntese com valores extraídos diretamente dos modelos via R inline |

---

## Análise Exploratória

### Distribuição do tempo de seguimento

![Distribuição do tempo de seguimento por desfecho](figures/hist-futime-1.png)

Censurados concentram-se em tempos longos (11–14 anos), padrão consistente com censura administrativa ao fim do estudo. Óbitos distribuem-se de forma mais uniforme ao longo de todo o período, descartando evento agudo inicial importante.

---

### Variáveis laboratoriais

![Distribuição das variáveis laboratoriais em escala log+1](figures/dist-continuas-1.png)

Distribuições fortemente assimétricas à direita, típicas de biomarcadores séricos. A escala logarítmica aproxima as distribuições da normalidade, justificando o uso de `log(creatinina)` na modelagem.

---

### FLC por grupo e desfecho

![FLC total por grupo e taxa bruta de óbito](figures/flc-grupo-death-1.png)

Relação dose a resposta clara já na análise univariada: a taxa bruta de óbito sobe monotonicamente de ~15% no grupo Baixo para ~51% no grupo Alto, com intervalos de confiança sem sobreposição entre grupos adjacentes.

---

### Causas de óbito por grupo de FLC

![Causas de óbito por grupo de FLC](figures/causas-obito-1.png)

Doenças circulatórias dominam em todos os grupos. A proporção de neoplasias cresce com o nível de FLC, consistente com o papel da FLC como marcador de desregulação imunológica que pode preceder condições malignas hematológicas.

---

### Perfil de recrutamento e valores ausentes

![Recrutamento por ano e missing em creatinina](figures/missing-recrutamento-1.png)

Os 1.350 valores ausentes de creatinina (~17%) concentram-se no grupo de FLC Baixo (25–30% de ausência), introduzindo potencial viés de seleção discutido nas limitações.

---

### Correlação entre variáveis contínuas

![Matriz de correlação de Spearman](figures/correlacoes-1.png)

FLC total e creatinina apresentam correlação positiva moderada: rins com menor capacidade de filtração eliminam menos FLC, elevando seus níveis séricos. Esse mecanismo de confundimento reforça a necessidade de ajuste conjunto pelas duas variáveis.

---

## Análise Não Paramétrica

### Curvas de Kaplan-Meier por grupo de FLC

![Curvas de Kaplan-Meier por grupo de FLC](figures/km-flc-1.png)

Separação visível desde o primeiro ano, aprofundando-se progressivamente até 14 anos. Ao final de 11 anos: $\hat{S} = 87\%$ no grupo Baixo vs. $49\%$ no grupo Alto — diferença de 38 pontos percentuais ($p < 0{,}001$, log-rank). Os grupos Baixo, Médio-baixo e Médio-alto não atingem a mediana de sobrevivência no período observado (mais de 50% sobreviveram).

---

### Por sexo e faixa etária

![Curvas de Kaplan-Meier por sexo e faixa etária](figures/km-sex-age-1.png)

O efeito da faixa etária é o mais expressivo visualmente: grupo 80+ com $\hat{S}(t=5) \approx 50\%$ vs. >95% no grupo 50–59 anos, justificando plenamente o ajuste por idade em qualquer modelo de mortalidade nessa coorte.

---

### Nelson-Aalen — Função de risco acumulada

![Estimador de Nelson-Aalen por grupo de FLC](figures/nelson-aalen-1.png)

Curvatura crescente em todos os grupos descarta a distribuição exponencial. Curvas aproximadamente paralelas em escala logarítmica sugerem proporcionalidade de riscos; a ligeira convergência nos anos finais é consistente com o teste de Schoenfeld.

---

### Subgrupo MGUS

![Curvas KM estratificadas por MGUS e grupo de FLC](figures/subgrupo-mgus-1.png)

Indivíduos com MGUS apresentam sobrevivência surpreendentemente maior, explicada por viés de detecção (acompanhamento médico mais intensivo), confundimento por FLC elevada e variabilidade amostral ($n = 115$). No modelo de Cox ajustado, o efeito do MGUS é não significativo ($p \approx 0{,}40$).

---

## Modelagem

### Hazard suavizado por grupo de FLC

![Função de risco suavizada — estimador B-spline (bshazard)](figures/hazard-suavizado-1.png)

Estimador B-spline sem efeito de borda (Rebora et al. 2014). Grupos Baixo, Médio-baixo e Médio-alto exibem risco crescente ao longo do seguimento. O grupo Alto apresenta padrão em U (hazard elevado no início, mínimo ~4 anos, recuperação posterior), compatível com heterogeneidade não observada como hipótese biológica.

---

### Gráficos de linearização (pré-ajuste)

![Gráficos de linearização para quatro famílias paramétricas](figures/linearizacao-1.png)

O painel Weibull exibe retas aproximadamente paralelas entre grupos, sugerindo boa adequação. O exponencial é descartado pela curvatura acentuada. Log-normal e log-logística apresentam desvios nas caudas. O Weibull é candidato plausível, a ser comparado formalmente com a gama generalizada via AIC e LRT.

---

### Modelo de Cox — Forest plot

![Forest plot dos Hazard Ratios ajustados](figures/forest-plot-1.png)

Gradiente monotônico e crescente da FLC após ajuste multivariado. Estatística $C$ de Harrell: **0,788** — discriminação aceitável para modelo de mortalidade por todas as causas em população geral.

| Covariável | HR | IC 95% |
|---|---|---|
| FLC Alto vs. Baixo | **2,04** | 1,73 – 2,40 |
| Idade (por ano) | 1,10 | 1,10 – 1,11 |
| Sexo masculino | 1,23 | 1,11 – 1,35 |
| log(Creatinina) | 1,70 | 1,40 – 2,07 |
| MGUS | 1,25 | 0,76 – 2,07 |

---

### Diagnósticos do Cox

**Proporcionalidade de riscos (Schoenfeld)**

![Resíduos de Schoenfeld escalonados por covariável](figures/cox-ph-plot-1.png)

Teste global rejeita $H_0$ ($p = 2{,}88 \times 10^{-8}$), com violação individual em `flc_cat` e `age`. Com $n > 6.500$, o teste tem poder para detectar desvios clinicamente irrelevantes; o modelo é mantido com a limitação reportada.

---

**Forma funcional das contínuas (Martingale)**

![Resíduos de Martingale vs. covariáveis contínuas](figures/residuos-martingale-1.png)

LOESS aproximadamente linear para idade e log(creatinina), confirmando que a forma funcional linear na escala do log-risco é adequada sem necessidade de termos polinomiais ou splines.

---

**Influência individual (dfbetas)**

![Resíduos dfbetas por covariável](figures/residuos-dfbeta-1.png)

Proporção de observações influentes entre 0,9% e 6,9% — baixa para $n > 6.500$. Nenhuma observação isolada distorce substancialmente os coeficientes estimados.

---

**Outliers (Deviance)**

![Resíduos de Deviance](figures/residuos-deviance-1.png)

Resíduos concentrados em $[-2{,}5;\; 2{,}5]$. O padrão assimétrico por desfecho (censurados negativos, óbitos positivos) é estrutural ao estimador e não configura violação do modelo.

---

**Curvas ajustadas pelo modelo de Cox**

![Sobrevivência ajustada pelo Cox por grupo de FLC](figures/cox-curvas-1.png)

A separação entre grupos após ajuste é similar em magnitude às curvas brutas de Kaplan-Meier, confirmando que o efeito da FLC não é artefato de confundimento por idade, sexo ou função renal.

---

### Modelos paramétricos AFT

A **gama generalizada** foi selecionada (menor AIC; $\Delta$AIC > 50 para todos os demais). Os testes LRT rejeitam Weibull, log-normal e gama como simplificações ($p < 0{,}001$).

| Distribuição | AIC | $\Delta$AIC |
|---|---|---|
| **Gama generalizada** | **15.267,73** | **0,00** |
| Weibull | 15.318,69 | 50,96 |
| Gama | 15.330,44 | 62,71 |
| Exponencial | 15.334,89 | 67,16 |
| Log-logística | 15.523,47 | 255,74 |
| Log-normal | 15.932,97 | 665,24 |

Parâmetro $Q$ estimado: $\hat{Q} \approx 1{,}57$ (IC 95%: 1,38–1,75; não contém 0 nem 1). O grupo Alto reduz o tempo esperado de sobrevivência em ~42% ($\exp(\hat{\beta}) \approx 0{,}58$).

---

### Verificação gráfica da gama generalizada (pós-ajuste)

![Linearização com transformação Q estimado](figures/lin-gengamma-1.png)

Retas aproximadamente lineares confirmam o bom ajuste da gama generalizada com $\hat{Q} = 1{,}57$. O IC de $\hat{Q}$ exclui $Q = 1$ (Weibull), rejeitando formalmente essa simplificação.

---

## Arquivos

```
.
├── TrabalhoFinal_Sobrevivencia.qmd   # Documento-fonte (Quarto)
├── references.bib                    # 24 referências bibliográficas
├── figures/                          # Figuras geradas pelo render
│   ├── km-flc-1.png
│   ├── hazard-suavizado-1.png
│   ├── forest-plot-1.png
│   └── ...
├── index.html                        # HTML publicado no GitHub Pages
└── README.md
```

---

## Reprodução local

### Dependências R

```r
install.packages(c(
  "survival", "flexsurv", "muhaz", "bshazard",
  "MASS", "My.stepwise", "ggsurvfit", "survminer",
  "gtsummary", "gt", "tidyverse", "patchwork",
  "broom", "corrplot", "scales"
))
```

> **R:** ≥ 4.4.0 · **Quarto:** ≥ 1.4

### Renderização

```bash
quarto render TrabalhoFinal_Sobrevivencia.qmd --to html
```

Os dados `flchain` são carregados diretamente via `data(flchain, package = "survival")` — nenhum arquivo externo necessário.

---

## Referências principais

- Dispenzieri, A. et al. Use of nonclonal serum immunoglobulin free light chains to predict overall survival in the general population. *Mayo Clinic Proceedings*, 87(6), 517–523, 2012.
- Kyle, R. A. et al. Prevalence of monoclonal gammopathy of undetermined significance. *New England Journal of Medicine*, 354(13), 1362–1369, 2006.
- Collett, D. *Modelling Survival Data in Medical Research*. 2. ed. CRC Press, 2003.
- Therneau, T. M.; Grambsch, P. M. *Modeling Survival Data: Extending the Cox Model*. Springer, 2000.
- Jackson, C. flexsurv: A Platform for Parametric Survival Modeling in R. *Journal of Statistical Software*, 70(8), 2016.
- Rebora, P.; Salim, A.; Reilly, M. bshazard: A Flexible Tool for Nonparametric Smoothing of the Hazard Function. *The R Journal*, 6(2), 114–122, 2014.
- Colosimo, E. A.; Giolo, S. R. *Análise de Sobrevivência Aplicada*. Edgard Blücher, 2006.
- Burnham, K. P.; Anderson, D. R. *Model Selection and Multimodel Inference*. 2. ed. Springer, 2002.
- Self, S. G.; Liang, K.-Y. Asymptotic properties of maximum likelihood estimators and likelihood ratio tests under nonstandard conditions. *JASA*, 82(398), 605–610, 1987.