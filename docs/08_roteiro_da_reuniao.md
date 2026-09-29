# Roteiro da reunião

## Abertura (1 minuto)

> "A v1 escrevia bem, mas inventava os números e errava o cenário. Reorganizei o sistema com uma regra simples: o LLM lê e escreve, o código calcula, e nada chega ao cliente sem checagem. E coloquei tudo num único grafo do Rivet: dá para apertar Run e ver a carta sendo montada, etapa por etapa, até o PDF."

## Demonstração sugerida (10 minutos)

1. **A carta da v1** (`Output/output_letter.docx`). Mostrar três erros: "Prezado João", "3,5%, 0,2 p.p. abaixo do benchmark" e "Selic em 9%".
2. **O grafo da v1 no Rivet.** Mostrar as entradas do prompt final trocadas e o caminho absoluto `C:\Users\blope\...`.
3. **O Main Graph: Enter Challenge rodando ao vivo.** Abrir o projeto no Rivet, conferir o executor **Node** e apertar Run. Enquanto roda (30 a 50 segundos), mostrar as três chamadas em paralelo, o loop de extração e o loop da carta.
   - **Plano B:** se a rede ou a API falharem, a carta da última execução já está em `Output/`, e a seção **Grafos do Rivet** deste site mostra o grafo.
4. **A carta da v2** (`Output/carta_albert_2025-05-07.pdf`):
   - página 1: resultado real do período, comparado com CDI e Ibovespa, e alocação contra o perfil;
   - página 2: cenário da XP, a leitura do assessor e as sugestões com valores.
5. **O brief do assessor.** Status, travas, os 11 alertas (o CDB vencido é o melhor exemplo), as recomendações com citações do research e o custo.
6. **As evidências** (`data/evidence/`). O mini trocando dígitos e o revisor pegando a distorção da Selic. Essa é a melhor prova de que as travas não são teóricas.
7. **O código por trás de um node.** Clicar em **Code: Analyze Portfolio** no app e mostrar que o código vem de `src/code/`; rodar `npm test`: 8 testes em cerca de 1 segundo, sem chave de API.

## As três perguntas do desafio

### 1. Quais são os principais problemas da primeira versão?

- **Erros que mudam o conteúdo:** entradas trocadas no prompt final, "Prezado João" fixo, e arquivos lidos de caminhos de outra máquina que viram prompt vazio sem erro.
- **Afirmações falsas:** benchmark e retorno inventados; macro oposto ao relatório (cortes do Fed, Selic 9%); rentabilidade desde a compra apresentada como do mês; ação que não aconteceu; recomendação genérica.
- **Dados que ninguém olhou:** CDB vencido, 30% do patrimônio parado, cotas de 2024, ticker extinto, fundo que virou FIDC, carteira fora do perfil.
- **Arquitetura:** macro refeito por cliente e nenhuma validação.

### 2. Como você decidiu a abordagem?

- **Onde errar custa caro** (números, suitability, compliance), o trabalho fica com o código. **Onde há texto bagunçado para ler ou escrever**, fica com o LLM. Entre um e outro, travas automáticas que devolvem o erro ao modelo.
- **As três áreas foram feitas porque dependem umas das outras:** sem rentabilidade correta a recomendação não tem base, e sem formatação a carta não sai do protótipo.
- **Cada escolha foi medida com as execuções reais:**
  - o gabarito da extração decidiu o modelo;
  - as evidências de deriva justificaram o revisor e as frases escritas pelo código;
  - a regra de atribuição de fontes só ficou depois de testada em várias execuções.
- **Tudo no Rivet**, para que o fluxo seja visível e operável na ferramenta da equipe, com o código em arquivos testáveis.
- **O assessor continua no loop:** é o que viabiliza escala com responsabilidade, e o brief é o produto para ele.

### 3. O que faria com um mês?

- **Dados na fonte:** API de posições da XP, eliminando o parsing de PDF; cotas CVM/ANBIMA de todos os fundos baixadas por um node; histórico para YTD e 12 meses.
- **Macro uma vez por mês:** guardar o resultado do **Subgraph: Macro Outlook** e reutilizá-lo em todas as cartas do mês. Hoje cada execução o chama de novo.
- **Research oficial:** carteira recomendada do XP Research e motor de suitability ANBIMA no lugar das faixas ilustrativas.
- **Avaliação contínua:** cerca de 30 clientes com gabarito; revisor calibrado com rubrica; testes e avaliação em CI. Isso permite trocar de modelo com segurança e escolher o mais barato que mantém a qualidade.
- **Produto:** tela de revisão para o assessor (aprovar, editar, enviar), com as edições virando dado de melhoria; envio com rastreio.
- **Escala e governança:** lote mensal (Batch API), observabilidade, custo por carta, LGPD e revisão de compliance dos textos padrão.
- **Medição:** A/B de NPS e share of wallet.

## Perguntas prováveis

**"Por que fazer tudo dentro do Rivet?"**
- **Visibilidade:** o fluxo inteiro está num grafo que qualquer pessoa da equipe abre, roda e inspeciona node a node.
- **Um artefato só:** o projeto do Rivet é o workflow; não há um programa externo chamando o Rivet.
- **O custo:** nodes de código só rodam JavaScript e não importam módulos. Por isso o código fica em arquivos, o projeto é gerado deles e os testes executam exatamente o mesmo texto.
- **A alternativa existe:** a branch `main` tem a versão com os cálculos em Python e o Rivet só nas etapas de LLM. Os números das duas são idênticos, e um teste garante isso.

**"Por que JavaScript?"**
É a única linguagem que o node de código do Rivet executa. As contas foram portadas do Python e conferidas campo a campo.

**"Quanto custa por cliente?"**
Cerca de US$ 0,10 por execução completa com gpt-4.1, incluindo as rodadas de correção. O resumo macro (cerca de US$ 0,04) é o mesmo para todos os clientes do mês; guardado e reaproveitado, cada cliente custaria cerca de US$ 0,06. Com gabarito e avaliação dá para testar modelos mais baratos em cada etapa.

**"E se o LLM alucinar mesmo assim?"**
- **Número inventado:** não passa pelo fact-check.
- **Afirmação distorcida:** é apontada pelo revisor.
- **Extrato transcrito errado:** não fecha na reconciliação.
- **Macro sem citação real:** é descartado.

O que sobra ainda passa pelo assessor, que recebe o brief com as evidências. E quando uma trava não resolve, a carta sai **bloqueada**, não enviada.

**"Os números são reais?"**
- **Ações:** vêm do CSV do desafio.
- **Fundos:** vêm das cotas oficiais da CVM na mesma janela.
- **Benchmarks:** vêm do Banco Central e do Yahoo, ao vivo.
- **Brave:** é a única estimativa, e está marcada como tal.

**"Por que Yahoo e não a B3?"**
A B3 não oferece histórico gratuito por API, e o Banco Central parou de publicar o Ibovespa em 2019. Em produção seria o feed de mercado da XP.

**"As recomendações são do research da XP?"**
- **A lógica:** as faixas do perfil e a lista de produtos são ilustrativas, e estão marcadas assim.
- **Onde plugar o oficial:** a estrutura recebe a carteira recomendada oficial trocando só dois arquivos YAML.
- **As justificativas:** citam o relatório macro real da XP, e a leitura do assessor aparece separada do que o relatório diz.

**"Como escala para 20 mil assessores?"**
- **Custo compartilhado:** o macro, guardado uma vez por mês.
- **Por cliente:** o resto é determinístico e barato.
- **Processamento:** as chamadas por cliente são independentes e rodam em lote pelo `rivet-cli`, sem abrir o app.
- **Gargalo humano:** a revisão do assessor diminui à medida que os alertas e o revisor forem calibrados com dados reais.
