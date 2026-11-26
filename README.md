# Estudo de Sensores Submarinos: Obstrução por Hidrato no 3W Dataset (Evento 8)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/leobarduzzi/mvp_analise_dados/blob/main/mvp.ipynb)

**Aluno:** Leonardo Cesar Barduzzi  
**Matrícula:** 4052026001555  
**Curso:** Sprint: Análise de Dados e Boas Práticas (PUC-Rio)  

Este repositório contém o notebook [`mvp.ipynb`](mvp.ipynb), que serve simultaneamente como código executável e relatório técnico do projeto. Ele reúne análise exploratória, diagnóstico de integridade de sensores e decisões fundamentadas de pré-processamento sobre dados reais de poços de petróleo da Petrobras.

Seguindo as diretrizes do MVP, **o trabalho termina na preparação e qualificação dos dados**, sem treinamento de modelos de machine learning.

---

## 1. Contexto e Problema

Em linhas de produção submarinas de petróleo e gás em águas profundas, água e gás sob alta pressão e baixa temperatura podem formar hidratos — estruturas sólidas parecidas com gelo que grudam no interior dos dutos. Esse acúmulo reduz a passagem do fluido e pode causar o bloqueio completo da linha, gerando altos custos de desobstrução e risco de parada na plataforma.

O objetivo do projeto é **analisar as medições dos sensores de poços reais da Petrobras nos períodos rotulados com hidrato**, investigando:
1. Quais variáveis físicas (temperatura ou pressão) realmente sinalizam a formação da obstrução de forma clara;
2. Em quais momentos da operação do poço o hidrato costuma se manifestar.

---

## 2. Unidade de Observação e Recorte

- **Linha:** uma leitura instantânea dos sensores a cada 1 segundo (amostragem em 1 Hz).
- **Instância (arquivo `.parquet`):** um episódio contínuo em um poço específico, dividido em blocos cronológicos de operação normal, transição e hidrato consolidado.
- **Recorte:** 14 instâncias reais de poços de petróleo (`WELL-*`), cobrindo 9 poços distintos e 4.809.035 leituras temporais gravadas entre 2011 e 2023.

---

## 3. Hipóteses Investigadas e Resultados Principais

- **Hipótese 1 ($H_1$) — A temperatura sinaliza o hidrato melhor que a pressão?**
  - *Resultado:* **Confirmada** para o sensor da árvore submarina (`T-TPT`). Nas 7 instâncias com medições válidas, a temperatura mediana cai de 46,60 °C (normal) para 5,45 °C (com hidrato), com separação consistente (AUC $\ge 0,90$) em 6 dos 7 poços. Em contrapartida, as pressões variam muito entre poços e tendem a subir na árvore (acumulando contrapressão).
- **Hipótese 2 ($H_2$) — O hidrato aparece na produção normal ou durante manobras?**
  - *Resultado:* **Confirmada**. Das 140.920 leituras com hidrato consolidado, **95,97% ocorrem durante manobras operacionais**: 45,48% em procedimentos de reinício (*Restart*), 44,22% em lavagem de linha com diesel (*Flushing*) e 6,27% em injeção sob pressão (*Bullheading*), contra apenas 4,03% em produção contínua (*Open*).

As análises gráficas completas (curvas ECDF, dispersão poço a poço, gráfico de barras em escala logarítmica e linha do tempo de episódio) e suas limitações estão detalhadas nas Seções 4 e 6 do notebook.

---

## 4. Por que o Trabalho é Estritamente Descritivo?

A escolha por não aplicar modelos preditivos supervisionados nesta etapa se apoia em características da base:
1. Cada arquivo real traz blocos contínuos de cada classe em sequência; dividir por linhas faria o modelo aprender a ordem temporal do arquivo em vez do sinal físico;
2. A divisão correta teria de ser feita por poço, mas temos apenas 9 poços reais disponíveis no recorte — amostra pequena demais para validação confiável;
3. O hidrato é um evento raro (apenas 2,93% do volume total de dados).

Por isso, não há divisão de treino e teste e nenhum parâmetro é aprendido para aplicar depois, o que elimina qualquer risco de vazamento de dados (*data leakage*).

---

## 5. Fonte e Acesso aos Dados

| Item | Detalhe |
|---|---|
| **Base** | *3W Dataset*, versão **2.0.0** (Petrobras / CENPES e UFRJ) |
| **Repositório oficial** | <https://github.com/petrobras/3W> |
| **Versão fixada** | commit [`cfb6666`](https://github.com/petrobras/3W/tree/cfb6666659e7a36c352f4206fabbab320f5939c9) |
| **Licença** | Creative Commons Attribution 4.0 (**CC BY 4.0**) |
| **Acesso** | Download automatizado via HTTPS direto (*raw storage*), sem login ou credenciais |

Os arquivos `.parquet` brutos não são versionados no Git para manter o repositório leve. Eles são baixados automaticamente na pasta local `data/3w/8/` durante a execução do notebook.

---

## 6. Como Executar

### No Google Colab (Recomendado)

1. Abra o notebook pelo botão badge acima ou pelo link:  
   `https://colab.research.google.com/github/leobarduzzi/mvp_analise_dados/blob/main/mvp.ipynb`.
2. Para salvar com permissão de edição (exigência da entrega): vá em `Arquivo → Salvar uma cópia no Drive` e compartilhe o link da sua cópia com permissão de edição para qualquer pessoa com o link.
3. No menu, clique em `Ambiente de execução → Executar tudo`.

### Em Ambiente Local

```bash
git clone https://github.com/leobarduzzi/mvp_analise_dados.git
cd mvp_analise_dados

# Criar e ativar ambiente virtual
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/macOS:
source .venv/bin/activate

# Instalar dependências
pip install -r requirements.txt

# Abrir o notebook
jupyter lab mvp.ipynb
```

*Requisitos:* Python 3.11+. As versões das bibliotecas estão fixadas em [`requirements.txt`](requirements.txt).

---

## 7. Estrutura do Repositório

```
.
├── README.md               # Apresentação do projeto e guia de execução
├── requirements.txt        # Dependências com versões fixadas
├── mvp.ipynb               # Notebook principal (código e relatório)
│
└── data/                   # Pasta de dados (ignorada no Git)
    ├── 3w/8/*.parquet      # 14 instâncias brutas baixadas via HTTPS
    └── resumo_por_instancia_classe.csv  # Tabela resumo gerada na Seção 5.5
```

---

## 8. Decisões de Pré-Processamento (D1 a D3)

- **Decisão D1 (Remover colunas sem nenhuma medição):** Exclusão de 5 variáveis com 100% de ausência (`P-JUS-BS`, `QBS`, `T-JUS-CKP`, `P-MON-SDV-P` e `PT-P`), evitando inventar mais de 24 milhões de números sem variância.
- **Decisão D2 (Manter leituras consecutivas idênticas):** Conservação de 24.775 linhas seguidas com mesmo valor decorrentes da resolução do sensor a cada segundo, preservando a grade temporal regular de 1 s sem criar buracos na série histórica.
- **Decisão D3 (Tratar códigos de sensor com erro como ausência):** Conversão de 1.439.863 valores absurdos de falha (como $-99,99$ e $-25$ °C) em `NaN` sem imputação artificial, restaurando médias físicas reais (ex.: média de `T-PDG` corrigida de $-10^{19}$ para 83,1 °C).

---

## 9. Declaração de Uso de IA e Autoria

Ferramentas de IA foram usadas apenas como apoio operacional para estruturação de código Python, formatação tabular e revisão ortográfica. A escolha das hipóteses, a fundamentação de engenharia de poço, a interpretação crítica dos resultados numéricos e as decisões de tratamento dos dados são de autoria própria do aluno Leonardo Cesar Barduzzi, que assume total responsabilidade pelo trabalho.
