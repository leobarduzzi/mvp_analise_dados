# MVP — Análise exploratória e pré-processamento do 3W Dataset (evento de hidrato)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/leobarduzzi/mvp_analise_dados/blob/main/mvp.ipynb)

Trabalho da disciplina **Sprint: Análise de Dados e Boas Práticas** (40530010055_20260_02) — PUC-Rio.

O notebook [`mvp.ipynb`](mvp.ipynb) é, ao mesmo tempo, o código e o relatório: reúne análise
exploratória, decisões de pré-processamento, verificações e limitações. **O escopo termina na
preparação dos dados** — não há treinamento nem comparação de modelos de aprendizado de máquina.

---

## Problema

Hidratos são estruturas cristalinas que se formam quando água e gás se combinam sob pressão alta e
temperatura baixa. Em linhas submarinas de produção, eles aderem à parede do tubo e reduzem a
passagem, podendo bloquear a linha. O problema analisado aqui é: **o que os sinais medidos no poço
mostram nos períodos em que especialistas rotularam a presença de hidrato**, e o que isso permite (e
não permite) concluir a partir dos dados disponíveis.

## Unidade de observação

- **Linha:** uma leitura dos sensores do poço em um instante (série temporal).
- **Instância (um arquivo `.parquet`):** um poço em um episódio de um mesmo evento, com um bloco
  contíguo de rótulos.

## Perguntas investigáveis

- **H1 — sinal físico:** quando há hidrato, a temperatura (`T-TPT`, `T-JUS-CKP`) se comporta de
  forma diferente da pressão (`P-TPT`, `P-MON-CKP`)? Ou seja: o resfriamento seria um aviso mais
  confiável que a queda de pressão?
- **H2 — contexto operacional:** o rótulo de hidrato ocorre com o poço produzindo normalmente ou
  durante manobras (parada, circulação de fluido, reinício)? Variáveis: `class` e `state`.

Cada hipótese declara, no notebook, o que a refutaria *antes* da análise, e a limitação associada.

## Resultados em uma linha

- **H1:** nas 7 instâncias com sensor confiável, `T-TPT` cai de 46,60 °C para 5,45 °C (mediana)
  entre o bloco normal e o de hidrato, com separação em 6 de 7 poços. Em `T-MON-CKP` a direção é
  mista (2 de 11 instâncias aquecem no bloco rotulado), e as pressões não têm separação consistente.
- **H2:** 95,97% das 140.920 linhas rotuladas como hidrato ocorrem fora de Open — 45,48% em Restart
  e 44,22% em Flushing Diesel.

As evidências, interpretações e limitações completas estão nas Figuras 1–4 do notebook.

## Objetivo analítico

**Descritivo.** Não há uso preditivo, e isso é uma decisão declarada, não uma omissão:

- cada instância real traz exatamente um bloco por classe, sempre na mesma ordem (normal →
  transitório → estacionário), então um modelo avaliado por observação aprenderia a posição no
  arquivo, não a assinatura física do evento;
- a única divisão honesta seria por poço, e o conjunto de poços é pequeno — poucos grupos
  independentes para avaliação;
- o desbalanceamento é concentrado em poucos arquivos.

A consequência para o pré-processamento (sem treino/teste, não há parâmetro a ajustar em separado) e
a justificativa correspondente no checklist estão na §5.1 do notebook.

---

## Dados

| Item | Valor |
|---|---|
| Base | **3W Dataset**, versão **2.0.0** |
| Origem | Petrobras/CENPES e UFRJ — <https://github.com/petrobras/3W> |
| Versão fixada | commit [`cfb6666`](https://github.com/petrobras/3W/tree/cfb6666659e7a36c352f4206fabbab320f5939c9) |
| Recorte | diretório `dataset/8` (hidrato na linha de produção), instâncias reais `WELL-*` |
| Condições de uso | dados sob **CC BY 4.0** (atribuição obrigatória); código do projeto original sob Apache 2.0 |
| Acesso | HTTPS público, **sem contas ou credenciais** |

Por que o evento 8: é o evento com maior cobertura de poços entre os que têm o rótulo de interesse,
o que dá alguma margem para comparações entre poços. Os demais eventos e o que cada um inviabiliza
estão discutidos no notebook (§1.1).

**Os arquivos `.parquet` não são versionados neste repositório.** Eles são baixados por HTTPS
durante a execução do notebook (§3), a partir da URL *raw* do repositório original, fixada no
commit acima. Isso mantém o repositório leve e garante que os dados usados sejam exatamente os da
versão declarada.

```python
RAW = "https://raw.githubusercontent.com/petrobras/3W/cfb6666659e7a36c352f4206fabbab320f5939c9/dataset/8"
```

---

## Como executar

### Google Colab (recomendado)

1. Abra o notebook pelo badge acima, ou por
   `https://colab.research.google.com/github/leobarduzzi/mvp_analise_dados/blob/main/mvp.ipynb`.
2. Para obter **permissão de leitura e escrita** (exigência da entrega), use
   `Arquivo → Salvar uma cópia no Drive` e compartilhe essa cópia como
   *"qualquer pessoa com o link pode editar"*. Informe o link do Drive na tarefa do MVP.
3. `Ambiente de execução → Executar tudo`.

Nenhuma instalação manual é necessária: as bibliotecas usadas já vêm no Colab. A célula da §3
imprime as versões efetivas do ambiente, para que a execução seja auditável.

### Localmente

```bash
git clone https://github.com/leobarduzzi/mvp_analise_dados.git
cd mvp_analise_dados
python -m venv .venv && . .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab mvp.ipynb
```

Requer **Python 3.11+**. Depois de instalar, `Kernel → Restart & Run All`.

> O download dos arquivos do evento 8 leva alguns minutos na primeira execução. As células são
> idempotentes: se os arquivos já estiverem em cache local, o download é ignorado.

---

## Estrutura do repositório

```
.
├── README.md          # este arquivo
├── requirements.txt   # dependências com versões fixadas
└── mvp.ipynb          # notebook principal (relatório)

data/                  # ignorado pelo Git: parquet baixados e resumo derivado
├── 3w/8/*.parquet     # 14 instâncias, obtidas por HTTPS na execução
└── resumo_por_instancia_classe.csv  # resumo compacto gerado na §5.5
```

---

## Dependências

Fixadas em [`requirements.txt`](requirements.txt). As versões efetivamente usadas na execução de
referência são impressas na célula 10 do notebook (§3.1): Python 3.11.9, pandas 2.3.2, NumPy 2.3.3,
pyarrow 26.0.1, Matplotlib 3.10.9 e seaborn 0.13.2. No Colab as versões podem diferir; a célula
sempre imprime as do ambiente em uso.

---

## Decisões de pré-processamento

O notebook registra as decisões **D1 a D3** na §5, cada uma no formato exigido: necessidade e
evidência nos dados, alternativas consideradas (incluindo manter os dados como estão), opção
escolhida, verificação antes/depois e limitação. D1 e D2 comparam duas alternativas com medida de
efeito; D3 compara estatísticas antes e depois. O estado final das visões e as verificações de
integridade ficam na §5.5.

---

## Mapa do notebook

| Item do checklist | Onde está |
|---|---|
| Problema, unidade de observação, H1/H2 e objetivo analítico | §1 |
| Origem, versão, condições de uso, recortes e dicionário das variáveis | §2 |
| Dimensões, tipos, inspeção inicial | §4.1 |
| Ausências, duplicidades, inconsistências e valores extremos | §4.1 |
| Distribuições e relações relevantes | §4.2 e §4.3 |
| Interpretações com evidência após cada figura | §4.2 e §4.3 |
| Decisões D1–D3 e comparação entre alternativas | §5.2 a §5.4 |
| Transformações (ou manutenção justificada) e verificações antes/depois | §5 |
| Divisão dos dados e prevenção de vazamento — ou justificativa de não aplicabilidade | §5.1 |
| Respostas a H1/H2, limitações e próximos passos | §6 |
| Checklist de entrega preenchido | §7 |
| Referências e declaração de uso de IA | §8 |

---

## Limitações conhecidas

- O dataset **não mede vazão de produção**, o que impede normalizar perda de carga e distinguir
  obstrução de queda de produção. Isso é discutido como limitação, não como teste de hipótese.
- Os rótulos vêm de análise de especialistas sobre os mesmos sinais aqui analisados; associação
  entre rótulo e variável não é prova de causalidade.
- A temperatura da árvore submarina é influenciada pela temperatura do leito marinho, o que pode
  mascarar a variação associada ao evento.

Os limites específicos de cada decisão estão registrados junto a ela, no notebook.

---

## Uso de IA

A declaração completa está no notebook (§8). Em resumo: ferramentas de IA foram usadas como apoio
técnico em código e organização do repositório; a análise exploratória, a interpretação dos
resultados e as conclusões são de autoria própria. O autor se responsabiliza por todo o código
entregue.

---

## Referências

- VARGAS, R. E. V. et al. **A realistic and public dataset with rare undesirable real events in oil
  wells**. *Journal of Petroleum Science and Engineering*, 2019. (3W Dataset, versão 1.0.0)
- PETROBRAS/CENPES; UFRJ. **3W Dataset — versão 2.0.0**. GitHub, 2024.
  <https://github.com/petrobras/3W> (acesso em nov. 2026). Licença dos dados: CC BY 4.0.
- Demais fontes de domínio e metodologia estão listadas na seção de referências do notebook.
