# flc-survival-analysis

**Cadeia Leve Livre Sérica e Mortalidade por Todas as Causas**  
Trabalho Final — Disciplina de Análise de Sobrevivência · UFRJ · 2026  
**Autor:** Arthur Motta

---

## 🔗 Acesso ao relatório

O trabalho está publicado e disponível online:

**[https://arthurpmotta02.github.io/flc-survival-analysis/](https://arthurpmotta02.github.io/flc-survival-analysis/)**

O relatório foi gerado com [Quarto](https://quarto.org/) a partir do arquivo `.qmd` deste repositório e publicado via GitHub Pages. Ele é interativo: todos os blocos de código R podem ser expandidos clicando em **"Code"**, e o sumário lateral permite navegar entre as seções.

---

## Sobre o projeto

Análise estatística completa do conjunto de dados `flchain` (pacote `survival` do R), investigando a associação entre a concentração sérica de cadeia leve livre (FLC) e a mortalidade por todas as causas em adultos com 50 anos ou mais do Condado de Olmsted, Minnesota (EUA).

**Pergunta de interesse:** A concentração sérica de cadeia leve livre está associada ao tempo de sobrevivência após controle por idade, sexo, creatinina sérica e diagnóstico de MGUS?

---

## Estrutura do repositório

```
.
├── TrabalhoFinal_Sobrevivencia.qmd   # Documento fonte (Quarto) — todo o código e texto
├── references.bib                    # Referências bibliográficas (24 entradas BibTeX)
├── index.html                        # HTML renderizado publicado no GitHub Pages
└── README.md                         # Este arquivo
```

O arquivo `.qmd` contém **todo o código R e o texto do relatório integrados**. Ao renderizar, o Quarto executa todos os chunks, gera as tabelas e figuras, e produz o HTML final com os resultados embutidos (`embed-resources: true`), sem dependências externas.

---

## Métodos utilizados

### Análise não paramétrica
- Estimador de Kaplan-Meier — global, por grupo de FLC, sexo e faixa etária
- Estimador de Nelson-Aalen (função de risco acumulada $\hat\Lambda(t)$)
- Teste log-rank para comparação entre grupos
- Hazard suavizado por B-splines (`bshazard`) — sem escolha manual de bandwidth
- Análise de subgrupo: pacientes com MGUS

### Seleção de covariáveis
- Procedimento de Collett (2003) em 4 passos: triagem univariada → backward → forward → stepwise
- Confirmação por `stepAIC` bidirecional (pacote `MASS`)
- Teste da razão de verossimilhanças (TRV) sequencial

### Modelos de regressão
- **Modelo de Cox** semiparamétrico com 5 diagnósticos completos
- **6 distribuições paramétricas AFT** via `flexsurv::flexsurvreg()`: exponencial, Weibull, log-normal, log-logística, gama e gama generalizada
- Comparação por AIC, BIC e testes LRT formais (Self & Liang 1987)
- Modelo selecionado: **gama generalizada** ($\hat{Q} \approx 1{,}57$)

### Diagnósticos do modelo de Cox
| Diagnóstico | Ferramenta |
|---|---|
| Riscos proporcionais | Teste de Grambsch & Therneau (Schoenfeld escalonado) |
| Forma funcional | Resíduos de Martingale + LOESS |
| Observações influentes | Resíduos dfbeta / dfbetas (limiar $2/\sqrt{n}$) |
| Outliers | Resíduos de Deviance |
| Discriminação | Estatística C de Harrell |

---

## Principais resultados

| Análise | Resultado |
|---|---|
| Kaplan-Meier | $\hat{S}(t=11)$ = 87% no grupo Baixo vs. 49% no grupo Alto; $p < 0{,}001$ (log-rank) |
| Modelo de Cox | HR = 2,04 (IC 95%: 1,73–2,40) para FLC Alto vs. Baixo após ajuste multivariado |
| Discriminação | Estatística $C = 0{,}788$ — discriminação aceitável |
| Gama generalizada | Melhor AIC entre 6 distribuições; grupo Alto reduz tempo esperado de sobrevivência ~42% |

---

## Como reproduzir localmente

### 1. Instalar os pacotes R necessários

```r
install.packages(c(
  "survival", "flexsurv", "muhaz", "bshazard",
  "MASS", "My.stepwise", "ggsurvfit", "survminer",
  "gtsummary", "gt", "tidyverse", "patchwork",
  "broom", "corrplot", "scales"
))
```

> **Versões recomendadas:** R ≥ 4.4.0 · Quarto ≥ 1.4

### 2. Clonar o repositório

```bash
git clone https://github.com/arthurpmotta02/flc-survival-analysis.git
cd flc-survival-analysis
```

### 3. Renderizar

```bash
# HTML interativo (com code-fold e sumário lateral)
quarto render TrabalhoFinal_Sobrevivencia.qmd --to html
```

Ou pelo RStudio: abrir o `.qmd` e clicar em **Render**.

Os dados `flchain` são carregados automaticamente via `data(flchain, package = "survival")` — não é necessário baixar nenhum arquivo externo.

---

## Sobre os dados

O conjunto `flchain` está disponível diretamente no pacote `survival` do R. Trata-se de uma amostra aleatória de 50% dos participantes de um estudo de coorte de base populacional conduzido no Condado de Olmsted, Minnesota (EUA), com 7.874 indivíduos com 50 anos ou mais recrutados a partir de 1995.

**Referência original dos dados:**  
Kyle, R. A. et al. (2006). Prevalence of monoclonal gammopathy of undetermined significance. *New England Journal of Medicine*, 354(13), 1362–1369. https://doi.org/10.1056/NEJMoa054494

**Referência da análise original com FLC:**  
Dispenzieri, A. et al. (2012). Use of nonclonal serum immunoglobulin free light chains to predict overall survival in the general population. *Mayo Clinic Proceedings*, 87(6), 517–523. https://doi.org/10.1016/j.mayocp.2012.03.009

---

## Licença

Uso acadêmico. Os dados `flchain` são de domínio público via pacote `survival` (licença LGPL-2).