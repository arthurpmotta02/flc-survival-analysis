# flc-survival-analysis

**Cadeia Leve Livre Sérica e Mortalidade por Todas as Causas**  
Trabalho Final — Disciplina de Análise de Sobrevivência · UFRJ · 2026  
**Autor:** Arthur Motta

---

## Sobre o projeto

Análise estatística completa do conjunto de dados `flchain` (pacote `survival` do R), investigando a associação entre a concentração sérica de cadeia leve livre (FLC) e a mortalidade por todas as causas em adultos com 50 anos ou mais do Condado de Olmsted, Minnesota (EUA).

**Pergunta de interesse:** A concentração sérica de cadeia leve livre está associada ao tempo de sobrevivência após controle por idade, sexo, creatinina sérica e diagnóstico de MGUS?

---

## Estrutura do repositório

```
.
├── TrabalhoFinal_Sobrevivencia.qmd   # Documento fonte (Quarto)
├── references.bib                    # Referências bibliográficas (24 entradas)
├── TrabalhoFinal_com_codigos.html    # HTML renderizado (com code-fold)
├── TrabalhoFinal_sem_codigos.pdf     # PDF sem código (versão entrega)
├── TrabalhoFinal_com_codigos.pdf     # PDF com código (apêndice expandido)
└── README.md
```

---

## Métodos utilizados

**Análise não paramétrica**
- Estimador de Kaplan-Meier (global, por grupo de FLC, sexo e faixa etária)
- Estimador de Nelson-Aalen (função de risco acumulada)
- Teste log-rank para comparação entre grupos
- Análise de subgrupo: MGUS

**Modelagem**
- Modelo de Cox semiparamétrico com seleção de covariáveis por procedimento de Collett (4 passos) + stepAIC + TRV sequencial
- Seis distribuições paramétricas AFT: exponencial, Weibull, log-normal, log-logística, gama e gama generalizada (via `flexsurv`)
- Comparação por AIC, BIC e testes LRT formais (Self & Liang 1987)

**Diagnósticos**
- Teste de Grambsch & Therneau (resíduos de Schoenfeld escalonados)
- Resíduos de Martingale (forma funcional das contínuas)
- Resíduos dfbeta/dfbetas (influência individual)
- Resíduos de Deviance (outliers)
- Estatística C de Harrell (discriminação)
- Hazard suavizado por B-splines (`bshazard`)

---

## Principais resultados

| Modelo | Resultado principal |
|---|---|
| Kaplan-Meier | $\hat{S}(t=11)$ = 87% no grupo Baixo vs. 49% no grupo Alto ($p < 0{,}001$, log-rank) |
| Cox | HR = 2,04 (IC 95%: 1,73–2,40) para FLC Alto vs. Baixo após ajuste; $C = 0{,}788$ |
| Gama generalizada | Melhor AIC; $\hat{Q} \approx 1{,}57$ (IC não contém 0 nem 1); grupo Alto reduz tempo esperado ~42% |

---

## Requisitos

```r
# Pacotes necessários
install.packages(c(
  "survival", "flexsurv", "muhaz", "bshazard",
  "MASS", "My.stepwise", "ggsurvfit", "survminer",
  "gtsummary", "gt", "tidyverse", "patchwork",
  "broom", "corrplot", "scales"
))
```

> **R versão:** ≥ 4.4.0  
> **Quarto versão:** ≥ 1.4

---

## Como renderizar

```bash
# HTML interativo (com code-fold)
quarto render TrabalhoFinal_Sobrevivencia.qmd --to html

# PDF (sem código)
quarto render TrabalhoFinal_Sobrevivencia.qmd --to pdf
```

Ou pelo RStudio: abrir o `.qmd` e clicar em **Render** (ou usar a setinha para escolher o formato).

---

## Dados

Os dados `flchain` estão disponíveis diretamente no pacote `survival` do R e são carregados via `data(flchain, package = "survival")`. Não é necessário baixar nenhum arquivo externo.

**Referência original:**  
Dispenzieri, A. et al. (2012). Use of nonclonal serum immunoglobulin free light chains to predict overall survival in the general population. *Mayo Clinic Proceedings*, 87(6), 517–523. https://doi.org/10.1016/j.mayocp.2012.03.009

---

## Licença

Uso acadêmico. Os dados `flchain` são de domínio público via pacote `survival` (licença LGPL-2).