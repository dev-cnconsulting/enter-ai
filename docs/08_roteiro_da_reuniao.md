# Roteiro da reunião

## Abertura (1 minuto)

> "A v1 escrevia bem, mas inventava os números e errava o cenário. Reorganizei o sistema com uma regra simples: o LLM lê e escreve, o código calcula, e nada chega ao cliente sem checagem. O resultado é uma carta correta, com recomendações concretas, e um brief que deixa o assessor revisar em minutos."

## Demonstração sugerida (10 minutos)

1. **A carta da v1** (`Output/output_letter.docx`). Mostrar três erros: "Prezado João", "3,5%, 0,2 p.p. abaixo do benchmark" e "Selic em 9%".
2. **O grafo da v1 no Rivet.** Mostrar as entradas do prompt final trocadas e o caminho absoluto `C:\Users\blope\...`.
3. **A carta da v2** (`Output/carta_albert_2025-05-07.pdf`):
   - página 1: resultado real do período, comparado com CDI e Ibovespa, e alocação contra o perfil;
   - página 2: cenário da XP e sugestões com valores.
4. **O brief do assessor.** Status, travas, os 11 alertas (o CDB vencido é o melhor exemplo), as recomendações com citações do research e o custo.
5. **As evidências** (`data/evidence/`). O mini trocando dígitos e o revisor pegando a distorção da Selic. Essa é a melhor prova de que as travas não são teóricas.
6. **O Rivet da v2.** Abrir `macro_outlook` ou `write_letter` e mostrar prompt, schema e saída estruturada.
7. **Os testes.** `pytest`: 29 testes em 5 segundos.

## As três perguntas do desafio

### 1. Quais são os principais problemas da primeira versão?

- **Erros que mudam o conteúdo:** entradas trocadas no prompt final, "Prezado João" fixo, e arquivos lidos de caminhos de outra máquina que viram prompt vazio sem erro.
- **Afirmações falsas:** benchmark e retorno inventados; macro oposto ao relatório (cortes do Fed, Selic 9%); rentabilidade desde a compra apresentada como do mês; ação que não aconteceu; recomendação genérica.
- **Dados que ninguém olhou:** CDB vencido, 30% do patrimônio parado, cotas de 2024, ticker extinto, fundo que virou FIDC, carteira fora do perfil.
- **Arquitetura:** macro refeito por cliente e nenhuma validação.

### 2. Como você decidiu a abordagem?

- **Onde errar custa caro** (números, suitability, compliance), o trabalho fica com o código. **Onde há texto bagunçado para ler ou escrever**, fica com o LLM. Entre um e outro, travas automáticas.
- **As três áreas foram feitas porque dependem umas das outras:** sem rentabilidade correta a recomendação não tem base, e sem formatação a carta não sai do protótipo.
- **Cada escolha foi medida com as execuções reais:**
  - o gabarito da extração decidiu o modelo;
  - as evidências de deriva justificaram o revisor e as frases escritas pelo código.
- **O assessor continua no loop:** é o que viabiliza escala com responsabilidade, e o brief é o produto para ele.

### 3. O que faria com um mês?

- **Dados na fonte:** API de posições da XP, eliminando o parsing de PDF; cotas CVM/ANBIMA de todos os fundos; histórico para YTD e 12 meses.
- **Research oficial:** carteira recomendada do XP Research e motor de suitability ANBIMA no lugar das faixas ilustrativas.
- **Avaliação contínua:** cerca de 30 clientes com gabarito; revisor calibrado com rubrica; tudo em CI. Isso permite trocar de modelo com segurança e escolher o mais barato que mantém a qualidade.
- **Produto:** tela de revisão para o assessor (aprovar, editar, enviar), com as edições virando dado de melhoria; envio com rastreio.
- **Escala e governança:** lote mensal (Batch API), observabilidade, custo por carta, LGPD e revisão de compliance dos textos padrão.
- **Medição:** A/B de NPS e share of wallet.

## Perguntas prováveis

**"Por que não fez tudo dentro do Rivet?"**
As etapas de LLM estão no Rivet. Contas e regras ficaram em Python porque precisam de testes, e um erro de cálculo não pode depender de um node de código difícil de testar. As duas partes se conversam pelo `rivet-cli` oficial.

**"Quanto custa por cliente?"**
Cerca de US$ 0,08 com gpt-4.1, incluindo uma rodada de correção. O resumo macro (US$ 0,04) é feito uma vez por mês para todos os clientes. Com gabarito e avaliação dá para testar modelos mais baratos em cada etapa.

**"E se o LLM alucinar mesmo assim?"**
- **Número inventado:** não passa pelo fact-check.
- **Afirmação distorcida:** é apontada pelo revisor.
- **Extrato transcrito errado:** não fecha na reconciliação.
- **Macro sem citação real:** é descartado.

O que sobra ainda passa pelo assessor, que recebe o brief com as evidências. E quando uma trava não resolve, a carta sai **bloqueada**, não enviada.

**"Os números são reais?"**
- **Ações:** vêm do CSV do desafio.
- **Fundos:** vêm das cotas oficiais da CVM na mesma janela.
- **Benchmarks:** vêm do Banco Central e do Yahoo.
- **Brave:** é a única estimativa, e está marcada como tal.

**"Por que Yahoo e não a B3?"**
A B3 não oferece histórico gratuito por API, e o Banco Central parou de publicar o Ibovespa em 2019. Em produção seria o feed de mercado da XP.

**"As recomendações são do research da XP?"**
- **A lógica:** as faixas do perfil e a lista de produtos são ilustrativas, e estão marcadas assim.
- **Onde plugar o oficial:** a estrutura recebe a carteira recomendada oficial trocando só dois arquivos YAML.
- **As justificativas:** citam o relatório macro real da XP.

**"Como escala para 20 mil assessores?"**
- **Custo compartilhado:** o macro é calculado uma vez por mês.
- **Por cliente:** o resto é determinístico e barato.
- **Processamento:** as chamadas por cliente são independentes e rodam em lote.
- **Gargalo humano:** a revisão do assessor diminui à medida que os alertas e o revisor forem calibrados com dados reais.
