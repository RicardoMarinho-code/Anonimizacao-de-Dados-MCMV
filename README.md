# 🔒 Anonimização de Dados e Controle de Divulgação Estatística (SDC)

<p align="left">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="Licença MIT"></a>
  <a href="https://creativecommons.org/licenses/by/4.0/"><img src="https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg" alt="Licença CC BY 4.0"></a>
  <img src="https://img.shields.io/badge/Python-3.8%2B-blue.svg" alt="Python 3.8+">
  <img src="https://img.shields.io/badge/R-4.0%2B-blue.svg" alt="R 4.0+">
</p>

Este repositório contém uma implementação prática de técnicas de **Controle de Divulgação Estatística (SDC - Statistical Disclosure Control)** e **Anonimização de Microdados**. O projeto demonstra como processar, limpar, generalizar e aplicar salvaguardas matemáticas de privacidade em bases de dados sensíveis utilizando tanto **Python** (com Pandas e Sweetviz) quanto **R** (com o pacote especializado `sdcMicro`).

O objetivo principal é garantir a privacidade de indivíduos e organizações (prevenindo a reidentificação), enquanto se preserva ao máximo a utilidade analítica dos dados para estudos e relatórios estatísticos.

---

## 📁 Estrutura do Projeto

A organização dos arquivos no repositório reflete a separação entre as fontes de dados e as implementações em diferentes linguagens/métodos:

```text
Anonimizacao-de-dados/
├── dados/                           # Diretório contendo as bases de dados (fictícias e processadas)
│   ├── dados ficticios.csv          # Base principal de testes (delimitador ";", encoding Latin1)
│   ├── dados_ficticios_tabelao.csv  # Base complementar em formato tabular amplo
│   ├── amostra_sdc.csv              # Amostra gerada a partir do pré-processamento Python
│   └── eda_amostra.csv              # Dados fictícios gerados a partir do arquivo original
├── ficticio_a/                      # Implementação principal de técnicas de SDC
│   ├── tabela1.py                   # Script Python: Pré-processamento, amostragem estratificada e Sweetviz
│   ├── tabela1.r                    # Script R: k-Anonimato (k=5) e perturbação categórica via PRAM
│   └── tabela2.r                    # Script R: Generalização geográfica, k-Anonimato e Microagregação numérica (MDAV)
├── ficticio_h/                      # Estrutura para expansões ou novos fluxos de trabalho
│   └── anonimização_h.r             # Script de teste/placeholder
├── amostra_sdc.csv                  # Cópia da amostra gerada no diretório raiz para fácil acesso
└── README.md                        # Documentação oficial do projeto (este arquivo)
```

---

## 🛠️ Técnicas de Anonimização Implementadas

O projeto aborda as principais dimensões da proteção de dados pessoais (em conformidade com diretrizes como a LGPD e o regulamento europeu GDPR), aplicando técnicas não-perturbativas e perturbativas:

### 1. Supressão de Identificadores Diretos (Direct Identifier Suppression)
Colunas que permitem a identificação direta do indivíduo de forma unívoca são totalmente descartadas logo no início do fluxo:
* **Removidas:** CPF (`cpf`), Nome (`nome_benefiniciário` / `txt_nome_empreendimento`), código do empreendimento (`cod_empreendimento`) e código IBGE (`cod_ibge`).

### 2. Generalização e Redução de Granularidade (Generalization)
Para reduzir o risco de reidentificação por meio de combinação de atributos (ataques de ligação), os dados são agregados em níveis superiores:
* **Temporal:** Datas exatas de referência e de assinatura de contrato (`data_referencia`, `dt_assinatura_contrato`) são generalizadas para o formato **Ano-Mês** (`YYYY-MM`).
* **Demográfica:** A data de nascimento (`data_nascimento`) é eliminada e substituída por faixas etárias amplas (`faixa_etaria`), classificadas em grupos:
  * *Até 29 anos*, *30-49 anos*, *50-64 anos* e *65+ anos*.
* **Geográfica:** A granularidade municipal (`txt_municipio`) é generalizada para a granularidade estadual (`txt_uf`), limitando o cruzamento de dados de localização.

### 3. k-Anonimato via Supressão Local (k-Anonymity)
Implementado em R através do pacote `sdcMicro`, o princípio do **k-Anonimato** garante que cada indivíduo na base de dados anonimizada seja indistinguível de, pelo menos, outros $k-1$ indivíduos que compartilham os mesmos quasi-identificadores.
* **Configuração:** Definido com **$k = 5$** (cada combinação de quasi-identificadores deve aparecer no mínimo 5 vezes).
* **Quasi-identificadores protegidos:** `txt_municipio`, `txt_uf`, `txt_regiao`, `txt_sexo`, `estado_civil`, `tipo_beneficiario`, `faixa_etaria`.
* **Supressão Local:** Valores específicos de quasi-identificadores que violam a regra são pontualmente suprimidos (convertidos em `NA`) para satisfazer o limiar $k$.

### 4. Método de Pós-Randomização (PRAM - Post-Randomization Method)
Uma técnica perturbativa que introduz ruído planejado nas variáveis categóricas para evitar ataques de ligação por probabilidade.
* **Variáveis afetadas:** `estado_civil` e `txt_sexo`.
* **Como funciona:** O algoritmo altera aleatoriamente o valor de algumas categorias com base em uma matriz de probabilidade de transição controlada, mantendo as frequências globais estáveis para fins de estatística descritiva, mas mascarando registros individuais.

### 5. Microagregação (Microaggregation)
Técnica SDC aplicada a variáveis contínuas/numéricas altamente sensíveis (como valores financeiros).
* **Variáveis afetadas:** Valor da garantia inicial (`vr_garantia_inicial`) e valor do subsídio de concessão (`vr_subsidio_concessao`).
* **Método:** **MDAV (Maximum Distance to Average Vector)**. O algoritmo agrupa registros semelhantes em clusters de tamanho mínimo e substitui os valores originais pela média aritmética daquele grupo, inviabilizando a descoberta de valores financeiros exatos por reidentificação de outliers.

---

## 🚀 Como Executar os Scripts

### 📋 Pré-requisitos

Certifique-se de ter instalado em sua máquina o **Python (3.8+)** e/ou o **R (4.0+)**.

#### Dependências do Python:
```bash
pip install pandas sweetviz
```

#### Dependências do R:
As bibliotecas do R serão instaladas automaticamente pelo gerenciador `pacman` presente nos scripts. Certifique-se de ter conexão com a internet na primeira execução. O pacote principal é o `sdcMicro`.

---

### 🐍 Executando o Fluxo em Python (`tabela1.py`)

O script Python realiza o pré-processamento inicial, formatação monetária, aplicação de amostragem estratificada e geração de relatórios de qualidade de dados.

1. Navegue até a pasta do projeto.
2. Certifique-se de que a base original está em `dados/dados ficticios.csv`.
3. Execute o script:
   ```bash
   python ficticio_a/tabela1.py
   ```
4. **Resultados gerados:**
   * Uma amostra estratificada com **5%** dos registros por modalidade (`txt_modalidade`) salva em `amostra_sdc.csv` (com semente aleatória reproduzível `123`).
   * Um relatório HTML interativo gerado em `dados/dado_teste/dados_ficticios.html` usando a biblioteca Sweetviz para análise e validação da distribuição dos dados.

---

### 📊 Executando o Fluxo em R

#### Script 1: k-Anonimato & PRAM (`tabela1.r`)
Aplica o modelo clássico de SDC categórico sobre os dados tratados:
1. Abra o console do R ou o RStudio e execute o script `tabela1.r`.
2. Um assistente de seleção de arquivo (`file.choose()`) será exibido. Escolha a base de dados (`dados/dados ficticios.csv`).
3. O script aplicará a limpeza de identificadores diretos, o agrupamento de idade em faixas, o k-Anonimato (k=5) e a perturbação PRAM nos campos de sexo e estado civil.
4. **Resultado gerado:** `tabela1_anonimizada.csv` contendo a base final segura.

#### Script 2: Microagregação & sdcApp Shiny (`tabela2.r`)
Demonstra o tratamento de variáveis numéricas/financeiras e fornece uma interface gráfica:
1. Execute o script `tabela2.r`.
2. Selecione a planilha Excel correspondente quando solicitado pelo assistente.
3. O script aplicará a generalização de município para estado, k-Anonimato, e agrupará os valores monetários (`vr_garantia_inicial` e `vr_subsidio_concessao`) usando o método MDAV.
4. **Resultado gerado:** `tabela2_anonimizada.csv` com dados financeiros agregados.
5. Ao término, o script instalará e iniciará o **sdcApp** no seu navegador padrão:
   > [!TIP]
   > O `sdcApp` é uma interface gráfica poderosa (Shiny App) que permite analisar os riscos de reidentificação, visualizar métricas de vazamento de informação (Information Loss) e configurar visualmente outros métodos de privacidade diferencial e anonimização sem codificação direta.

---

## 🔒 Boas Práticas e Recomendações de SDC

> [!IMPORTANT]
> A anonimização não é um processo estático de "apagar nomes". Um processo seguro exige balancear a utilidade científica/estatística da base de dados e a garantia matemática da privacidade do usuário.

Abaixo estão alguns dos pilares observados neste repositório:

| Princípio | Descrição | Objetivo no Projeto |
| :--- | :--- | :--- |
| **Prevenção de Ataques de Ligação** | Impedir que a combinação de variáveis comuns (ex: sexo, idade, estado civil, região) filtre apenas um indivíduo específico. | Garantido pela imposição de **k=5** sobre todos os quasi-identificadores. |
| **Preservação de Outliers Numéricos** | Evitar que valores financeiros extremamente altos revelem a identidade de indivíduos de alta renda. | Tratado por **Microagregação MDAV**, agrupando os valores e mascarando as pontas extremas. |
| **Reprodutibilidade** | Permitir que o processo de amostragem e ruído seja auditável e reproduzível para validação científica. | Uso de sementes aleatórias rígidas (`random_state = 123` em Python e controle estruturado no R). |

---

## 📝 Licença e Contribuição

Este projeto foi desenvolvido para fins educacionais e profissionais de proteção de dados. Sinta-se à vontade para abrir *Issues* ou enviar *Pull Requests* com melhorias nos algoritmos de SDC ou na modelagem de privacidade.

### Licenciamento

Este repositório é distribuído sob licenças de uso livre para fins educacionais, acadêmicos e profissionais:

*   **Código-Fonte (Python e R):** Licenciado sob a [Licença MIT](LICENSE). Isso significa que você pode usar, copiar, modificar, mesclar, publicar e distribuir o código livremente, contanto que mantenha o aviso de direitos autorais original.
*   **Dados Fictícios e Documentação:** Licenciados sob a [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). Você é livre para compartilhar, copiar e adaptar os materiais de apoio contanto que atribua o devido crédito ao autor original.