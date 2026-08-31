# MVP de SSD: Dimensionamento de uma Nova Unidade de Armazenamento em Nuvem

# Disciplina: Sistemas de Suporte à Decisão Universidade de Brasília (UnB)
# Aluna: Maria Eduarda Moreno Oliani 
# Matrícula: 231013458

# Sumário
Problema de decisão
Base de dados
Metodologia
Estrutura do notebook
Tecnologias utilizadas
Principais achados
Limitações

#Problema de decisão

Uma empresa de armazenamento de dados em nuvem vai abrir uma nova unidade. Antes de investir, a diretoria precisa decidir três coisas, cada uma com um eixo de análise correspondente:

Eixo	Pergunta de decisão
Risco	Qual capacidade contratar sem comprometer o SLA da unidade?
Custo	Quanto cada dimensionamento custa para operar?
Tempo	Em quantas fases implantar a unidade até ela poder receber clientes de produção?

Este MVP monta um sistema de apoio a essa decisão combinando dois modelos preditivos (um de risco e um de custo), uma simulação do espaço de dimensionamentos possíveis e um cronograma de implantação faseado.

#Base de dados

Cloud Resource Management Dataset (Kaggle), 1.000 registros com o estado de uma unidade de nuvem antes e depois de um processo de otimização de recursos.

Variáveis de decisão (conhecidas antes de construir a unidade):

- initial_resource_pool: capacidade contratada;
- workload_complexity: complexidade do portfólio de clientes;
- initial_utilization: ocupação planejada;
- initial_cost_per_unit: custo unitário negociado;
- initial_performance_score: desempenho da configuração base.

Variáveis de resultado (só conhecidas depois que a unidade está rodando, usadas apenas como alvo dos modelos): optimized_resource_allocation, optimized_utilization, optimized_cost_per_unit, optimized_performance_score e os indicadores de ganho percentual.

A base não traz classe de risco nem custo total prontos. As duas coisas são construídas no notebook a partir de regra de negócio explícita, o que é diferente de trabalhar com uma base de telemetria já rotulada.

# Metodologia
- Verificação da base: antes de qualquer modelo, o notebook checa que os indicadores de ganho percentual são redundantes entre si e que as colunas de resultado são funções determinísticas das colunas de decisão. Essa checagem define quais variáveis podem entrar como preditoras sem vazamento de dados.
- Construção dos indicadores: custo total de operação (custo unitário × recursos alocados) e classe de risco em três níveis, a partir de uma meta de SLA sobre o score de desempenho otimizado.
- Modelagem: um classificador (Random Forest) para o risco e um regressor (Random Forest) para o custo, ambos treinados só com as variáveis de decisão.
- Escolha do dimensionamento: em vez de um score ponderado entre risco e custo, adota-se uma regra de restrição — define-se uma meta máxima de risco tolerável e escolhe-se a capacidade mais barata que cumpre essa meta, para três perfis de portfólio comercial (carga leve, padrão e pesada).
- Simulação de Monte Carlo: como a base é determinística, a incerteza é reintroduzida sorteando 5.000 cenários em torno das premissas de projeto e propagando cada um pelos dois modelos, para obter uma distribuição de custo e risco em vez de um número fixo.
- Cronograma de implantação: a capacidade recomendada é distribuída em quatro trimestres, comparando uma rampa linear com a entrega da capacidade mínima viável já no primeiro trimestre.

# Estrutura do notebook
Ambiente e carga dos dados
Verificação da base antes de modelar
Construção dos indicadores de risco e custo
Análise exploratória (risco e custo por capacidade e complexidade da carga)
Modelos de risco e de custo
Escolha da capacidade (menor capacidade que atenda a uma meta de risco)
Simulação de Monte Carlo para incerteza sobre as premissas
Cronograma de implantação por fases
Conclusão

# Tecnologias utilizadas
- Python 3
- pandas e numpy
- scikit-learn (RandomForestClassifier, RandomForestRegressor, StandardScaler, Pipeline)
- matplotlib e seaborn

# Principais achados
- A base é sintética: todas as colunas de resultado são funções determinísticas das colunas de decisão, e três indicadores de ganho percentual são a mesma informação repetida três vezes. Por isso os modelos usam só as cinco variáveis que a empresa define antes de construir a unidade.
- Existe um patamar de capacidade: abaixo dele, o risco de furar o SLA é alto e praticamente insensível a pequenos aumentos; acima dele, o risco despenca. O dimensionamento é, na prática, uma decisão de degrau, não de ajuste fino.
- Risco e custo se opõem: as configurações mais baratas são também as mais arriscadas, o que justifica tratar a escolha como um problema de restrição em vez de uma média simples entre os dois critérios.
- A complexidade do portfólio de clientes é a variável de maior peso sobre o risco e a menos controlada pela engenharia, o que sugere limitar por contrato o perfil de carga vendido na nova unidade.
- Dividir a implantação em parcelas iguais de capacidade mantém a unidade em risco Alto por mais tempo, porque uma unidade pequena não gera escala suficiente para a otimização de recursos funcionar, o que atrasa o momento em que ela pode receber clientes de produção.

# Limitações
- A base é sintética: os valores absolutos (capacidade recomendada, custos) não devem ser transportados para um caso real sem recalibração.
- Não há variáveis de localização, energia ou regulação, então o MVP responde quanto e quando dimensionar a unidade, não onde instalá-la.
- O Monte Carlo propaga incerteza sobre as premissas de entrada, mas os modelos continuam determinísticos por dentro: não capturam ruído operacional real, falhas de equipamento nem eventos raros.
