<properties
   pageTitle="Diretrizes de particionamento de dados | Microsoft Azure"
   description="Diretrizes sobre como separar partições para ser gerenciadas e acessadas separadamente."
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="masimms"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="12/20/2015"
   ms.author="masashin"/>

# Diretrizes de particionamento de dados

![](media/best-practices-data-partitioning/pnp-logo.png)

## Visão geral

Em muitas soluções de grande escala, os dados são divididos em partições separadas que podem ser gerenciadas e acessadas separadamente. A estratégia de particionamento deve ser cuidadosamente escolhida para maximizar os benefícios, ao mesmo tempo que minimiza os efeitos adversos. O particionamento pode ajudar a melhorar a escalabilidade, reduzir a contenção e otimizar o desempenho. Um benefício adicional do particionamento é que ele também pode fornecer um mecanismo para dividir os dados pelo padrão de uso; você poderia arquivar os dados mais antigos e mais inativos (menos acessados) para reduzir o custo do armazenamento de dados.

## Por que particionar os dados?

A maioria dos serviços e aplicativos na nuvem armazena e recupera dados como parte de suas operações. O design dos repositórios de dados usado por um aplicativo pode ter uma influência significativa sobre o desempenho, taxa de transferência e escalabilidade de um sistema. Uma técnica normalmente aplicada em sistemas de grande escala é dividir os dados em partições separadas.

> O termo _particionamento_ usado neste guia refere-se ao processo de dividir fisicamente os dados em repositórios de dados separados. Isso não é o mesmo que o Particionamento de tabela do SQL Server, que é um conceito diferente.

O particionamento de dados pode oferecer uma série de benefícios. Por exemplo, ele pode ser aplicado para:

- **Melhorar a escalabilidade**. Em algum momento, o dimensionamento horizontal de um sistema de banco de dados individual atingirá um limite de hardware físico. Dividir os dados em várias partições, com cada uma delas hospedada em um servidor separado, permite que o sistema escale horizontalmente quase que por tempo indeterminado.
- **Melhorar o desempenho**. As operações de acesso a dados em cada partição ocorrem em um volume de dados menor. Desde que os dados sejam particionados de forma adequada, isso será muito mais eficiente. As operações que afetam mais de uma partição podem ser executadas paralelamente. Cada partição pode estar localizada próximo a um aplicativo que a usa para minimizar a latência de rede.
- **Melhorar a disponibilidade**. Separar dados em vários servidores evita um ponto único de falha. Se um servidor falhar, ou estiver passando por uma manutenção planejada, somente os dados nessa partição não estarão disponíveis. As operações em outras partições podem continuar. Aumentar o número de partições reduz o impacto relativo de uma falha de servidor único, ao reduzir o percentual dos dados que não estarão disponíveis. Replicar cada partição pode reduzir ainda mais a probabilidade de uma falha de partição única afetar as operações. Também permite a separação dos dados críticos que devem estar contínua e altamente disponíveis dos dados de valor baixo (como arquivos de log) que têm requisitos de disponibilidade mais baixos.
- **Melhorar a segurança**. Dependendo da natureza dos dados e de como eles são particionados, é possível separar dados confidenciais e não confidenciais em partições diferentes e, portanto, em diferentes servidores ou repositórios de dados. Em seguida, a segurança pode ser especificamente otimizada para os dados confidenciais.
- **Fornecer flexibilidade operacional**. O particionamento oferece várias oportunidades para o ajuste das operações, maximizando a eficiência administrativa e minimizando os custos. Alguns exemplos são a definição de diferentes estratégias de gerenciamento, monitoramento, backup e restauração e outras tarefas administrativas com base na importância dos dados em cada partição.
- **Fazer a correspondência do repositório de dados ao padrão de uso**. O particionamento permite que cada partição seja implantada em um tipo diferente de repositório de dados, com base no custo e nos recursos internos que o repositório de dados oferece. Por exemplo, dados binários grandes poderiam ser armazenados em um repositório de dados de blob, enquanto dados mais estruturados poderiam ser mantidos em um banco de dados de documentos. Para obter mais informações, veja [Compilando uma solução poliglota] no guia de padrões e práticas [Acesso a Dados para Soluções Altamente Escalonáveis: Usando o SQL, NoSQL e Polyglot Persistence] no site da Microsoft.

Alguns sistemas não implementam o particionamento, pois ele é considerado uma sobrecarga em vez de uma vantagem. Entre alguns dos motivos comuns para essa lógica estão:

- Muitos sistemas de armazenamento de dados não dão suporte a junções entre partições e pode ser difícil manter a integridade referencial em um sistema particionado. Frequentemente, é necessário implementar junções e verificações de integridade no código do aplicativo (na camada de particionamento), o que poderá resultar em E/S adicional e na complexidade do aplicativo.
- Manter partições nem sempre é uma tarefa comum. Em um sistema em que os dados são voláteis, talvez seja necessário rebalancear as partições periodicamente para reduzir a contenção e os pontos de acesso.
- Obviamente, algumas ferramentas comuns não funcionam com dados particionados.

## Criando partições

Os dados podem ser particionados de diferentes maneiras: horizontal, vertical ou funcionalmente. A estratégia escolhida depende do motivo para o particionamento dos dados e dos requisitos dos aplicativos e serviços que usarão os dados.

> [AZURE.NOTE]Os esquemas de particionamento descritos nestas diretrizes são explicados de forma independente da tecnologia subjacente de armazenamento de dados. Eles podem ser aplicados a vários tipos de repositórios de dados, incluindo bancos de dados relacionais e NoSQL.

### Estratégias de particionamento

As três estratégias típicas para o particionamento dos dados são:

- **Particionamento horizontal** (geralmente chamado de _fragmentação_). Nessa estratégia, cada partição é um repositório de dados por si só, mas todas as partições têm o mesmo esquema. Cada partição é conhecida como um _fragmento_ e contém um subconjunto específico de dados, como todos os pedidos de um conjunto específico de clientes em um aplicativo de comércio eletrônico.
- **Particionamento vertical**. Nessa estratégia, cada partição contém um subconjunto dos campos de itens no repositório de dados. Os campos estão divididos de acordo com o seu padrão de uso, como colocar os campos acessados com frequência em uma partição vertical e os acessados com menos frequência em outra.
- **Particionamento funcional**. Nessa estratégia, os dados são agregados de acordo com o modo como são usados por cada contexto vinculado no sistema. Por exemplo, um sistema de comércio eletrônico que implementa funções comerciais separadas para o faturamento e gerenciamento do inventário de produtos poderia armazenar dados de faturas em uma partição e dados do inventário de produtos em outra.

É importante observar que as três estratégias descritas aqui podem ser combinadas. Elas não se excluem mutuamente e você deve considerá-las ao criar um esquema de particionamento. Por exemplo, você poderia dividir os dados em fragmentos e então usar o particionamento vertical para subdividir ainda mais os dados em cada fragmento. Da mesma forma, os dados em uma partição funcional podem ser divididos em fragmentos (que também podem ser particionados verticalmente).

No entanto, os diferentes requisitos de cada estratégia podem gerar uma série de problemas de conflito que devem ser avaliados e ponderados durante a criação de um esquema de particionamento que atenda às metas gerais de desempenho de processamento de dados do seu sistema. As seções a seguir exploram cada uma das estratégias mais detalhadamente.

### Particionamento horizontal (fragmentação)

A Figura 1 mostra uma visão geral do particionamento horizontal ou fragmentação. Neste exemplo, os dados do inventário de produtos são divididos em fragmentos com base na chave do produto (Product Key). Cada fragmento contém os dados para um intervalo contíguo de chaves de fragmento (A-G e H-Z), organizadas em ordem alfabética.

![](media/best-practices-data-partitioning/DataPartitioning01.png)

_Figura 1. – Particionamento horizontal (fragmentação) de dados com base em uma chave de partição_

A fragmentação permite distribuir a carga em um número maior de computadores, reduzindo a contenção e melhorando o desempenho. Você pode escalar o sistema horizontalmente adicionando mais fragmentos que são executados em um número maior de servidores.

O fator mais importante ao implementar essa estratégia de particionamento é a opção de chave de fragmentação. Pode ser difícil alterar a chave depois que o sistema estiver em operação. A chave deve garantir que os dados sejam particionados para que a carga de trabalho seja a mais uniforme possível entre os fragmentos. Observe que diferentes fragmentos não precisam conter volumes de dados semelhantes; em vez disso, o que é importante considerar é o balanceamento do número de solicitações: alguns fragmentos podem ser muito grandes, mas cada item ser o assunto de um número baixo de operações de acesso, enquanto que outros fragmentos podem ser menores, mas cada item ser acessado com muito mais frequência. Também é importante garantir que um único fragmento não exceda os limites de escala (em termos de recursos de capacidade e processamento) do repositório de dados usado para hospedar esse fragmento.

O esquema de fragmentação também deve evitar a criação de pontos de acesso (ou partições mais acessadas) que podem afetar o desempenho e a disponibilidade. Por exemplo, usar um hash de um identificador de cliente em vez da primeira letra do nome do cliente impedirá a distribuição desbalanceada resultante de letras iniciais comuns e menos comuns. Essa é uma técnica típica que ajuda a distribuir os dados mais uniformemente nas partições.

A chave de fragmentação que você escolher deve minimizar os requisitos futuros de divisão de grandes fragmentos em partes menores, unir pequenos fragmentos em partições maiores ou alterar o esquema que descreve os dados armazenados em um conjunto de partições. Essas operações podem ser muito demoradas e podem exigir que você deixe um ou mais fragmentos offline enquanto elas são executadas. Se os fragmentos forem replicados, é possível manter algumas das réplicas online enquanto outras são divididas, mescladas ou reconfiguradas; porém, talvez o sistema precise limitar as operações que podem ser executadas nos dados desses fragmentos durante a reconfiguração. Por exemplo, os dados nas réplicas poderiam ser marcados como somente leitura para limitar o escopo de inconsistências que poderiam ocorrer de outro modo enquanto os fragmentos são reestruturados.

> Para obter mais informações detalhadas e diretrizes sobre várias dessas considerações, além de técnicas de práticas recomendadas para a criação de repositórios de dados que implementam o particionamento horizontal, veja [Padrão de fragmentação]

### Particionamento vertical

O uso mais comum do particionamento vertical é reduzir a E/S e os custos de desempenho associados à busca de itens que são acessados com mais frequência. A Figura 2 mostra uma visão geral de um exemplo de particionamento vertical, em que diferentes propriedades para cada item de dados são mantidas em partições diferentes; o nome, descrição e informações de preços dos produtos são acessados com mais frequência do que o volume no estoque ou a data do último pedido.

![](media/best-practices-data-partitioning/DataPartitioning02.png)

_Figura 2. – Particionamento vertical de dados por seu padrão de uso_

Neste exemplo, o aplicativo regularmente consulta o nome do produto, descrição e preço em conjunto quando exibe os detalhes de produtos aos clientes. O nível de estoque e a data mais recente em que o produto foi encomendado do fabricante são mantidos em uma partição separada, já que esses dois itens são usados juntos. Esse esquema de particionamento tem uma vantagem adicional de que os dados relativamente lentos (nome do produto, descrição e preço) são separados dos dados mais dinâmicos (nível de estoque e data do último pedido). Um aplicativo pode considerar útil armazenar em cache na memória os dados lentos se eles forem acessados com frequência.

Outro cenário típico dessa estratégia de particionamento é maximizar a segurança dos dados confidenciais. Por exemplo, com o armazenamento de números de cartão de crédito e dos números de verificação de segurança do cartão correspondentes em partições separadas.

O particionamento vertical também pode reduzir a quantidade de acesso simultâneo necessário aos dados.

> O particionamento vertical funciona no nível da entidade em um repositório de dados, parcialmente normalizando uma entidade para dividi-la de um item _amplo_ em um conjunto de itens _restritos_. Ele é ideal para repositórios de dados orientados a colunas, como o HBase e Cassandra. Se for improvável que os dados em uma coleção de colunas serão alterados, você também poderá considerar o uso de repositórios de colunas no SQL Server.

### Particionamento funcional

Para os sistemas em que seja possível identificar um contexto vinculado para cada área ou serviço comercial distinto no aplicativo, o particionamento funcional fornece uma técnica para a melhoria do isolamento e do desempenho de acesso a dados. Outro uso comum do particionamento funcional é separar dados de leitura/gravação de dados somente leitura usados para fins de relatório. A Figura 3 mostra uma visão geral de particionamento funcional em que os dados do inventário são separados dos dados do cliente.

![](media/best-practices-data-partitioning/DataPartitioning03.png)

_Figura 3. – Particionamento funcional de dados por contexto ou subdomínio vinculado_

Essa estratégia de particionamento pode ajudar a reduzir a contenção do acesso a dados em diferentes partes do sistema.

## Criando partições para a escalabilidade

É fundamental considerar o tamanho e a carga de trabalho para cada partição e balanceá-los para que os dados sejam distribuídos para obtenção da máxima escalabilidade. No entanto, você também deve particionar os dados para que eles não excedam os limites de dimensionamento de um único repositório de partição.

Ao criar as partições de escalabilidade, siga estas etapas:

1. Analise o aplicativo para entender os padrões de acesso a dados, como o tamanho do conjunto de resultados retornados por cada consulta, a frequência de acesso, a latência inerente e os requisitos de processamento de computação do lado do servidor. Em muitos casos, algumas entidades principais exigirão a maior parte dos recursos de processamento.
2. Com base na análise, determine as atuais e futuras metas de escalabilidade, como o tamanho dos dados e a carga de trabalho, e distribua os dados nas partições para atender à meta de escalabilidade. Na estratégia de particionamento horizontal, é importante escolher a chave de fragmento adequada para garantir que a distribuição seja uniforme. Para obter mais informações, veja [Padrão de fragmentação].
3. Verifique se os recursos disponíveis para cada partição são suficientes para lidar com os requisitos de escalabilidade em termos de tamanho de dados e taxa de transferência. Por exemplo, o nó que hospeda uma partição poderá impor um limite rígido na quantidade de espaço de armazenamento, capacidade de processamento ou largura de banda da rede que ela fornece. Se os requisitos de armazenamento e processamento de dados tiverem a probabilidade de exceder esses limites, pode ser necessário aprimorar sua estratégia de particionamento ou subdividir os dados. Por exemplo, uma abordagem de escalabilidade poderia consistir na separação de dados de log dos principais recursos do aplicativo com o uso de repositórios de dados separados para impedir que os requisitos do armazenamento total de dados excedam o limite de dimensionamento do nó. Se o número total de repositórios de dados exceder o limite do nó, pode ser necessário usar nós de armazenamento separados.
4. Monitore o sistema em uso para verificar se os dados são distribuídos conforme esperado e que as partições possam manipular a carga imposta sobre elas. É possível que o uso não corresponda ao previsto pela análise; é possível fazer o rebalanceamento das partições. Se isso não funcionar, pode ser necessário recriar algumas partes do sistema para obter o balanceamento necessário.

É importante lembrar que alguns ambientes de nuvem alocam recursos em termos de limites de infraestrutura, e você deve garantir que os limites de seu limite selecionado fornecem espaço suficiente para qualquer crescimento previsto no volume de dados com relação ao armazenamento de dados, capacidade de processamento e largura de banda. Por exemplo, se você usar o armazenamento de tabela do Azure, um fragmento ocupado pode exigir mais recursos do que aqueles que estão disponíveis para uma única partição para manipular solicitações (há um limite para o volume de solicitações que podem ser manipuladas por uma única partição em um determinado período de tempo – visite a página [Metas de desempenho e escalabilidade de armazenamento do Azure] no site da Microsoft para obter mais detalhes). Nesse caso, o fragmento pode precisar ser reparticionado para distribuir a carga. Se o tamanho total ou a taxa de transferência dessas tabelas exceder a capacidade de uma conta de armazenamento, pode ser necessário criar mais contas de armazenamento e distribuir as tabelas entre essas contas. Se o número de contas de armazenamento exceder o número de contas disponíveis para uma assinatura, pode ser necessário usar várias assinaturas.

## Criando partições para o desempenho da consulta

O desempenho da consulta pode ser frequentemente aumentado com o uso de conjuntos de dados menores e a execução de consulta paralela. Cada partição deve conter uma pequena proporção de todo o conjunto de dados, e essa redução no volume pode melhorar o desempenho das consultas. No entanto, o particionamento não é uma alternativa para a criação e configuração adequadas de um banco de dados. Por exemplo, verifique se você implementou os índices necessários se estiver usando um banco de dados relacional.

Ao criar as partições do desempenho da consulta, siga estas etapas:

1. Examine os requisitos e o desempenho do aplicativo:
	- Use os requisitos de negócios para determinar as consultas críticas que devem sempre ser executadas rapidamente.
	- Monitore o sistema para identificar todas as consultas que são executadas lentamente.
	- Estabeleça as consultas que são executadas com mais frequência. Uma única instância de cada consulta pode ter um custo mínimo, mas o consumo cumulativo de recursos pode ser significativo. Pode ser útil separar os dados recuperados por essas consultas em uma partição distinta ou até mesmo em um cache.
2. Particione os dados que estão causando a lentidão no desempenho. Certifique-se de:
	- Limitar o tamanho de cada partição para que o tempo de resposta da consulta esteja dentro da meta.
	- Criar a chave de fragmento para que o aplicativo encontre facilmente a partição, se estiver implementando o particionamento horizontal. Isso impede que a consulta precise verificar cada partição.
	- Considerar o local de uma partição no desempenho das consultas. Se possível, tente manter os dados nas partições que estão geograficamente próximas aos aplicativos e aos usuários que os acessam.
3. Se uma entidade tiver requisitos de desempenho da consulta e de taxa de transferência, use o particionamento funcional com base nessa entidade. Se isso ainda não conseguir atender aos requisitos, aplique também o particionamento horizontal. Na maioria dos casos, uma única estratégia de particionamento será suficiente, mas em alguns casos, é mais eficiente combinar as duas estratégias.
4. Considere usar consultas assíncronas que são executadas paralelamente nas partições para melhorar o desempenho.

## Criando partições para a disponibilidade

O particionamento de dados pode melhorar a disponibilidade de aplicativos, garantindo que todo o conjunto de dados não constitua um ponto único de falha e que subconjuntos individuais do conjunto de dados possam ser gerenciados independentemente. Replicar partições que contêm dados críticos também pode melhorar a disponibilidade.

Ao criar e implementar partições, considere os seguintes fatores que afetam a disponibilidade:

- A criticidade dos dados em relação às operações de negócios. Alguns dados podem abranger informações de negócios críticas, como detalhes de faturas ou transações bancárias. Outros dados podem ser simplesmente dados operacionais menos críticos, como arquivos de log, rastreamentos de desempenho e assim por diante. Depois de identificar cada tipo de dados, considere:
	- Armazenar os dados críticos em partições altamente disponíveis com um plano de backup adequado.
	- Estabelecer mecanismos ou procedimentos separados de gerenciamento e monitoramento para os diferentes níveis de importância de cada conjunto de dados. Colocar os dados com o mesmo nível de importância na mesma partição para que eles possam ser armazenados em backup juntos com uma frequência adequada. Por exemplo, as partições que contêm dados de transações bancárias podem precisar de backup com mais frequência do que as partições que contêm informações de rastreamento ou registro em log.
- Como as partições individuais podem ser gerenciadas. Criar partições para dar suporte ao gerenciamento e manutenção independentes oferece várias vantagens. Por exemplo:
	- Se uma partição falhar, ela pode ser recuperada independentemente sem afetar as instâncias de aplicativos que acessam dados em outras partições.
	- O particionamento de dados por área geográfica pode permitir a execução das tarefas de manutenção planejada fora dos horários de pico para cada local. Verifique se as partições não são muito grandes para evitar a conclusão de qualquer manutenção planejada durante esse período.
- A possibilidade de replicar dados críticos nas partições. Essa estratégia pode melhorar a disponibilidade e o desempenho, embora também possa apresentar problemas de consistência. Leva tempo para que as alterações feitas nos dados em uma partição sejam sincronizadas com cada réplica; além disso, durante esse período as diferentes partições conterão diferentes valores de dados.

## Problemas e considerações

Usar o particionamento acrescenta complexidade ao design e desenvolvimento do sistema. É importante considerar o particionamento como uma parte fundamental do design do sistema, mesmo se o sistema contiver inicialmente apenas uma única partição. Lidar com o particionamento como uma consideração posterior quando o sistema começar a apresentar problemas de desempenho e escalabilidade apenas aumenta a complexidade, já que você provavelmente agora tem um sistema dinâmico para manter. Atualizar o sistema para incorporar o particionamento nesse ambiente exige não apenas a modificação da lógica de acesso a dados, mas também pode envolver a migração de grandes quantidades de dados existentes para distribuí-los em partições, muitas vezes enquanto os usuários esperam conseguir continuar usando o sistema.

Em alguns casos, o particionamento não é considerado importante, pois o conjunto de dados inicial é pequeno e pode ser facilmente manipulado por um único servidor. Isso pode acontecer em um sistema que não deve escalar além de seu tamanho inicial, mas muitos sistemas comerciais precisam ser capazes de se expandir conforme o número de usuários aumenta. Essa expansão normalmente é acompanhada por um crescimento no volume de dados. Você também deve entender que o particionamento nem sempre é uma função dos grandes repositórios de dados. Por exemplo, um repositório de dados pequeno pode ser muito acessado por centenas de clientes simultâneos. Particionar os dados nessa situação pode ajudar a reduzir a contenção e melhorar a taxa de transferência.

Ao criar um esquema de particionamento de dados, considere os pontos a seguir:

- Sempre que possível, mantenha os dados para as operações mais comuns do banco de dados juntos em cada partição para minimizar as operações de acesso a dados entre partições. Consultar entre partições pode ser mais demorado do que consultar apenas em uma única partição; porém, otimizar as partições para um conjunto de consultas pode afetar outros conjuntos de consultas. Para minimizar o tempo de consulta nas partições quando isto for inevitável, execute consultas paralelas nas partições e agregue os resultados no aplicativo. No entanto, essa abordagem talvez não seja possível em alguns casos, como quando é necessário obter um resultado de uma consulta e usá-lo na próxima consulta.
- Se as consultas usarem dados de referência relativamente estáticos, como tabelas de código postal ou listas de produtos, considere replicar esses dados em todas as partições para reduzir a necessidade de uma operação de pesquisa separada em outra partição. Essa abordagem também pode reduzir a probabilidade dos dados de referência se tornarem um conjunto de dados “muito acessados” que está sujeito a tráfego intenso de todo o sistema, embora haja um custo adicional associado à sincronização das alterações que podem ser feitas nesses dados de referência.
- Sempre que possível, minimize os requisitos de integridade referencial entre partições verticais e funcionais. Nesses esquemas, o próprio aplicativo é responsável por manter a integridade referencial entre partições quando os dados são atualizados e consumidos. As consultas que devem unir dados em várias partições são executadas mais lentamente do que as consultas que unem dados apenas na mesma partição, pois o aplicativo normalmente precisa executar consultas consecutivas com base em uma chave e, em seguida, em uma chave estrangeira. Em vez disso, considere replicar ou cancelar a normalização dos dados relevantes. Para minimizar o tempo de consulta onde são necessárias junções entre partições, execute consultas paralelas nas partições e una os dados no aplicativo.
- Considere o efeito que o esquema de particionamento pode ter sobre a consistência dos dados nas partições. Você deve avaliar se uma coerência forte é realmente um requisito. Em vez disso, uma abordagem comum na nuvem é implementar a consistência eventual. Os dados em cada partição são atualizados separadamente, e a lógica do aplicativo pode assumir a responsabilidade por garantir que todas as atualizações sejam concluídas com êxito – bem como lidar com as inconsistências que podem surgir da consulta dos dados durante uma operação com consistência eventual. Para obter mais informações sobre como implementar a consistência eventual, veja as Diretrizes de consistência. (#insertlink#)
- Considere como as consultas localizarão a partição correta. Se uma consulta deve verificar todas as partições para localizar os dados necessários, haverá um impacto significativo no desempenho, mesmo ao usar várias consultas paralelas. As consultas usadas com as estratégias de particionamento vertical e funcional podem obviamente especificar as partições. No entanto, ao usar o particionamento horizontal (fragmentação), pode ser difícil localizar um item, pois cada fragmento tem o mesmo esquema. Uma solução típica de fragmentação é manter um mapa que pode ser usado para procurar a localização do fragmento para itens específicos de dados. Este mapa pode ser implementado na lógica de fragmentação do aplicativo ou mantido pelo repositório de dados se houver suporte para a fragmentação transparente.
- Ao usar uma estratégia de particionamento horizontal, considere o rebalanceamento periódico dos fragmentos para distribuir os dados uniformemente por tamanho e carga de trabalho para minimizar os pontos de acesso, maximizar o desempenho da consulta e contornar as limitações de armazenamento físico. No entanto, isso é uma tarefa complexa que geralmente requer o uso de uma ferramenta ou um processo personalizado.
- Replicar cada partição oferece uma proteção adicional contra falhas. Se uma única réplica falhar, as consultas podem ser direcionadas a uma cópia funcional.
- Se você chegar aos limites físicos de uma estratégia de particionamento, talvez seja necessário estender a escalabilidade para um nível diferente. Por exemplo, se o particionamento estiver no nível do banco de dados, isso pode significar a localização ou replicação das partições em vários bancos de dados. Se o particionamento já estiver no nível do banco de dados e as limitações físicas forem um problema, isso pode significar a localização ou replicação das partições em várias contas de hospedagem.
- Evite as transações que acessam os dados em várias partições. Alguns repositórios de dados implementam consistência e integridade transacionais para operações que modificam dados, mas somente quando eles estão localizados em uma única partição. Se precisar de suporte transacional em várias partições, você provavelmente precisará implementá-lo como parte da lógica do aplicativo, pois a maioria dos sistemas de particionamento não dão suporte nativo.

Todos os repositórios de dados exigem algumas atividades operacionais de gerenciamento e monitoramento. As tarefas podem variar de carregamento de dados, backup e restauração de dados a reorganização de dados, além de garantir que o sistema esteja funcionando correta e eficientemente.

Considere os seguintes fatores que afetam o gerenciamento operacional:

- Considere como você implementará tarefas operacionais e de gerenciamento adequadas e quando os dados são particionados, como backup e restauração, arquivamento de dados, monitoramento do sistema e outras tarefas administrativas. Por exemplo, manter a consistência lógica durante as operações de backup e restauração pode ser um desafio.
- Como os dados podem ser carregados em várias partições e como os novos dados recebidos de outras fontes podem ser adicionados. Algumas ferramentas e utilitários podem não dar suporte às operações de dados fragmentados como o carregamento de dados na partição correta; portanto, talvez seja necessário criar ou obter novas ferramentas e utilitários.
- Como os dados serão arquivados e excluídos regularmente (talvez mensalmente) para evitar o crescimento excessivo de partições. Pode ser necessário transformar os dados de acordo com um esquema de arquivo morto diferente.
- Considere a possibilidade de executar um processo periódico para localizar quaisquer problemas de integridade de dados como dados em uma partição que referencie informações contidas em outra, sendo que essas informações estão ausentes. O processo também pode tentar corrigir esses problemas automaticamente ou acionar um alerta para um operador para que ele corrija os problemas manualmente. Por exemplo, em um aplicativo de comércio eletrônico, as informações de pedidos poderiam ser mantidas em uma única partição, mas os itens de linha que constituem cada pedido poderiam ser mantidos em outra. O processo de fazer um pedido exige a adição de dados às duas partições. Se esse processo falhar, poderia haver itens de linha armazenados para os quais não haja nenhum pedido correspondente.

Diferentes tecnologias de armazenamento de dados normalmente fornecem seus próprios recursos para dar suporte ao particionamento. As seções a seguir resumem as opções implementadas por repositórios de dados usados pelos aplicativos do Azure e descrevem as considerações para a criação de aplicativos que podem aproveitar melhor esses recursos.

## Estratégias de particionamento para o Banco de Dados SQL do Azure

O Banco de Dados SQL do Azure é um banco de dados como serviço relacional executado na nuvem. Ele se baseia no Microsoft SQL Server. Um banco de dados relacional divide as informações em tabelas e cada tabela contém informações sobre as entidades como uma série de linhas. Cada linha contém colunas que contêm os dados dos campos individuais de uma entidade. A página [O que é o Banco de Dados SQL do Azure?] no site da Microsoft fornece documentação detalhada sobre como criar e usar bancos de dados SQL.

## Particionamento horizontal com Banco de Dados Elástico

Um único banco de dados SQL tem um limite para o volume de dados que ele pode conter, e a taxa de transferência é limitada por fatores arquiteturais e pelo número de conexões simultâneas que ele dá suporte. O Banco de Dados SQL do Azure fornece o Banco de Dados Elástico para oferecer suporte ao dimensionamento horizontal de um banco de dados SQL. Com o Banco de Dados Elástico, é possível particionar os dados em fragmentos distribuídos entre vários bancos de dados SQL, bem como adicionar ou remover fragmentos conforme o volume de dados que você precisa manipular é aumentado e reduzido. Usar o Banco de Dados Elástico também pode ajudar a reduzir a contenção pela distribuição da carga nos bancos de dados.

> [AZURE.NOTE]Atualmente, o Banco de Dados Elástico está no modo de visualização desde de dezembro de 2015. Ela substitui as Federações do Banco de Dados SQL do Azure que serão desativadas. As instalações existentes da Federação do Banco de Dados SQL do Azure podem ser migradas para o Banco de Dados Elástico usando o [Utilitário de Migração das Federações]. Como alternativa, você pode implementar seu próprio mecanismo de fragmentação se o seu cenário não for naturalmente adequado para os recursos fornecidos pelo Banco de Dados Elástico.

Cada fragmento é implementado como um banco de dados SQL. Um fragmento pode conter mais de um conjunto de dados (chamado de um _shardlet_), e cada banco de dados mantém metadados que descrevem os shardlets que ele contém. Um shardlet pode ser um único item de dados ou um grupo de itens que compartilham a mesma chave de shardlet. Por exemplo, se estiver fragmentando dados em um aplicativo multilocatário, a chave de shardlet poderia ser a ID do locatário e todos os dados de um determinado locatário seriam mantidos como parte do mesmo shardlet. Os dados de outros locatários seriam mantidos em shardlets diferentes.

É responsabilidade do programador associar um conjunto de dados a uma chave de shardlet. Um banco de dados SQL separado age como um gerenciador global de mapa de fragmentos que contém uma lista de bancos de dados (fragmentos) que abrangem todo o sistema, juntamente com informações sobre os shardlets em cada banco de dados. Um aplicativo cliente que acessa dados primeiro se conecta ao banco de dados do gerenciador global de mapa de fragmentos para obter uma cópia do mapa de fragmento (listando fragmentos e shardlets) que ele armazena em cache no local. Em seguida, o aplicativo usa essas informações para encaminhar solicitações de dados para o fragmento adequado. Essa funcionalidade está oculta atrás de uma série de APIs contidas na Biblioteca de Cliente do Banco de Dados Elástico do Banco de Dados SQL do Azure, disponível como um pacote NuGet. A página [Visão geral dos recursos do Banco de Dados Elástico] no site da Microsoft fornece uma introdução mais abrangente ao Banco de Dados Elástico.

> [AZURE.NOTE]Você pode replicar o banco de dados do gerenciador global de mapa de fragmentos para reduzir a latência e melhorar a disponibilidade. Se implementar o banco de dados usando uma das camadas de preços Premium, você pode configurar a replicação geográfica ativa para copiar os dados continuamente para os bancos de dados em diferentes regiões. Crie uma cópia do banco de dados em cada região em que os usuários estão localizados e configure seu aplicativo para se conectar a essa cópia para obter o mapa de fragmentos.

> Uma abordagem alternativa é usar a Sincronização de Dados do SQL do Azure ou um pipeline do Azure Data Factory para replicar o banco de dados do gerenciador de mapa de fragmentos nas regiões. Essa forma de replicação é executada periodicamente e é mais adequada se o mapa de fragmentos não for alterado com frequência. Além disso, o banco de dados do gerenciador de mapa de fragmentos não precisa ser criado usando uma camada de preços Premium.

O Banco de Dados Elástico oferece dois esquemas para mapear dados para shardlets e armazená-los em fragmentos:

- Um Mapa de Fragmentos da Lista descreve uma associação entre um shardlet e uma chave única. Por exemplo, em um sistema multilocatário, os dados de cada locatário podem ser associados a uma chave exclusiva e armazenados em seu próprio shardlet. Para garantir a privacidade e o isolamento (para evitar que um locatário esgote os recursos de armazenamento de dados disponíveis para outros locatários), cada shardlet poderia ser mantido em seu próprio fragmento.

![](media/best-practices-data-partitioning/PointShardlet.png)

_Figura 4. – Usando um mapa de fragmentos da lista para armazenar dados de locatários em fragmentos separados_

- Um Mapa de Fragmentos do Intervalo descreve uma associação entre um conjunto de valores de chave contíguos e um shardlet. No exemplo de sistema multilocatário descrito anteriormente, como alternativa à implementação de shardlets dedicados, você pode agrupar os dados de um conjunto de locatários (cada um com sua própria chave) no mesmo shardlet. Esse esquema é mais barato do que o primeiro (com os locatários compartilhando os recursos de armazenamento de dados), mas há o risco da redução da privacidade de dados e isolamento.

![](media/best-practices-data-partitioning/RangeShardlet.png)

_Figura 5. – Usando um mapa de fragmentos do intervalo para armazenar dados de um intervalo de locatários em um fragmento_

Observe que um único fragmento pode conter os dados de vários shardlets. Por exemplo, você poderia usar os shardlets da lista para armazenar dados de diferentes locatários não contíguos no mesmo fragmento. Você também pode combinar shardlets do intervalo e shardlets da lista no mesmo fragmento, mesmo que eles sejam abordados por meio de diferentes mapas no banco de dados do gerenciador global de mapa de fragmentos (o banco de dados do gerenciador global de mapa de fragmentos pode conter vários mapas de fragmentos). A Figura 6 ilustra essa abordagem.

![](media/best-practices-data-partitioning/MultipleShardMaps.png)

_Figura 6. – Implementando vários mapas de fragmentos_

O esquema de particionamento que você implementar pode ter uma influência significativa no desempenho do sistema e também afetar a taxa na qual os fragmentos devem ser adicionados ou removidos ou na qual os dados devem ser reparticionados nos fragmentos. Ao usar o Banco de Dados Elástico para particionar dados, considere os pontos a seguir:

- Agrupe dados que são usados juntos no mesmo fragmento e evite operações que precisam acessar os dados mantidos em vários fragmentos. Considere que, com o Banco de Dados Elástico, um fragmento é um banco de dados SQL por si só e o Banco de Dados SQL do Azure não oferece suporte a junções entre bancos de dados; essas operações devem ser realizadas no lado do cliente. Lembre-se também de que com o Banco de Dados SQL do Azure as restrições de integridade referencial, gatilhos e procedimentos armazenados em um banco de dados não podem referenciar objetos em outro; por isso, não crie um sistema que tenha dependências entre os fragmentos. No entanto, um banco de dados SQL pode conter tabelas que mantêm cópias dos dados de referência usados por consultas e outras operações, e essas tabelas não precisam pertencer a um shardlet específico. Replicar esses dados nos fragmentos pode ajudar a eliminar a necessidade de unir os dados que abrangem bancos de dados. O ideal é que esses dados sejam estáticos ou lentos, para minimizar o esforço de replicação e reduzir a probabilidade de eles se tornarem obsoletos.

	> [AZURE.NOTE]Embora o Banco de Dados SQL do Azure não ofereça suporte a junções entre bancos de dados, a API do Banco de Dados Elástico permite realizar consultas entre fragmentos que podem iterar de modo transparente por meio dos dados mantidos em todos os shardlets referenciados por um mapa de fragmentos. A API do Banco de Dados Elástico divide as consultas entre fragmentos em uma série de consultas individuais (uma para cada banco de dados) e, em seguida, mescla os resultados. Para obter mais informações, visite a página [Consulta de vários fragmentos] no site da Microsoft.

- Os dados armazenados em shardlets que pertencem ao mesmo mapa de fragmentos devem ter o mesmo esquema. Por exemplo, não crie um mapa de fragmentos da lista que aponte para alguns shardlets que contenham dados de locatários e para outros shardlets que contenham informações de produtos. Essa regra não é imposta pelo Banco de Dados Elástico, mas o gerenciamento de dados e a consulta se tornarão muito complexos se cada shardlet tiver um esquema diferente. No exemplo que acabamos de citar, você deve criar dois mapas de fragmentos da lista: um que referencie dados de locatários e o outro que aponte para informações de produtos. Lembre-se de que os dados pertencentes a diferentes shardlets podem ser armazenados no mesmo fragmento.

	> [AZURE.NOTE]A funcionalidade de consulta entre fragmentos da API do Banco de Dados Elástico depende de cada shardlet no mapa de fragmentos que contém o mesmo esquema.
- Nas operações transacionais, somente há suporte para os dados mantidos no mesmo fragmento, e não em vários fragmentos. As transações podem abranger shardlets, desde que façam parte do mesmo fragmento. Portanto, se a sua lógica de negócios precisar executar transações, armazene os dados afetados no mesmo fragmento ou implemente a consistência eventual. Para obter mais informações, veja as diretrizes de Consistência de dados.
- Coloque os fragmentos próximo aos usuários que acessam os dados nesses fragmentos (localize os fragmentos geograficamente). Essa estratégia ajudará a reduzir a latência.
- Evite ter uma combinação de fragmentos altamente ativos (pontos de acesso) e fragmentos relativamente inativos. Experimente distribuir a carga uniformemente entre os fragmentos. Isso pode exigir o hash das chaves de shardlet.
- Se estiver localizando os fragmentos geograficamente, verifique se as chaves em hash são mapeadas para os shardlets mantidos em fragmentos armazenados próximo aos usuários que acessam esses dados.
- Atualmente, somente há suporte para um conjunto limitado de tipos de dados do SQL, como as chaves de shardlet: _int, bigint, varbinary_ e _uniqueidentifier_. Os tipos de dados do SQL _int_ e _bigint_ correspondem aos tipos de dados _int_ e _long_ no C# e têm os mesmos intervalos. O tipo de dados do SQL _varbinary_ pode ser manipulado usando uma matriz _Byte_ no C# e o tipo de dados do SQL _uniqueidentier_ corresponde à classe _Guid_ no .NET Framework.

Como o nome indica, o Banco de Dados Elástico permite que um sistema adicione e remova fragmentos conforme o volume de dados é reduzido e aumentado. As APIs na Biblioteca de Cliente do Banco de Dados Elástico do Banco de Dados SQL do Azure permitem que um aplicativo crie e exclua fragmentos dinamicamente (e atualize de modo transparente o gerenciador de mapa de fragmentos). No entanto, a remoção de um fragmento é uma operação destrutiva que também requer a exclusão de todos os dados nesse fragmento. Se um aplicativo precisar dividir um fragmento em dois fragmentos separados ou combinar fragmentos, o Banco de Dados Elástico fornecerá um serviço separado de Divisão/Mesclagem. Esse serviço é executado em um serviço hospedado na nuvem (o desenvolvedor tem que criar esse serviço hospedado na nuvem) e se encarrega da migração de dados entre os fragmentos com segurança. Para obter mais informações, confira o tópico [Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico] no site da Microsoft.

## Estratégias de particionamento para o Armazenamento do Azure

O armazenamento do Azure fornece três abstrações para o gerenciamento de dados:

- Armazenamento de Tabela, que implementa o armazenamento de estrutura escalonável. Uma tabela contém uma coleção de entidades e cada uma delas pode incluir um conjunto de propriedades e valores.
- Armazenamento de Blobs, que fornece armazenamento para objetos e arquivos grandes.
- Filas de Armazenamento, que dão suporte a mensagens assíncronas confiáveis entre aplicativos.

Basicamente, o Armazenamento de Tabela e o Armazenamento de Blobs são repositórios de chave-valor otimizados para armazenar dados estruturados e não estruturados, respectivamente. As Filas de Armazenamento fornecem um mecanismo para criar aplicativos flexíveis e escalonáveis. O Armazenamento de Tabela, o Armazenamento de Blobs e as Filas de Armazenamento são criados dentro do contexto de uma conta de armazenamento do Azure. As contas de armazenamento do Azure dão suporte a três formas de redundância:

- Armazenamento com redundância local, que mantém três cópias de dados em um único datacenter. Essa forma de redundância fornece proteção contra falhas de hardware, mas não contra um desastre que abrange todo o datacenter.
- Armazenamento com redundância de zona, que mantém três cópias de dados distribuídos em diferentes datacenters na mesma região (ou em duas regiões geograficamente próximas). Essa forma de redundância pode fornecer proteção contra desastres que ocorrem em um único datacenter, mas não pode proteger contra desconexões de rede de grande escala que afetam uma região inteira. Observe que atualmente o armazenamento com redundância de zona está somente disponível apenas para blobs de blocos.
- Armazenamento com redundância geográfica, que mantém seis cópias de dados: três cópias em uma região (sua região local) e as outras três cópias em uma região remota. Essa forma de redundância fornece o nível mais alto de proteção contra desastres.

A Microsoft publicou metas de escalabilidade para as contas de armazenamento do Azure; visite a página [Metas de desempenho e escalabilidade de armazenamento do Azure] no site da Microsoft. Atualmente, a capacidade total da conta de armazenamento (o tamanho de dados mantidos no armazenamento de tabela, o armazenamento de blobs e as mensagens pendentes mantidas na fila de armazenamento) não pode exceder 500 TB. A taxa máxima de solicitação (supondo uma entidade, blob ou mensagem de 1 KB) é de 20 K por segundo. Se o seu sistema tiver a probabilidade de exceder esses limites, considere particionar a carga em várias contas de armazenamento; uma única assinatura do Azure pode criar até 100 contas de armazenamento. No entanto, observe que esses limites podem ser alterados com o tempo.

## Particionamento do armazenamento de tabela do Azure

O armazenamento de tabela do Azure é uma chave/valor armazenado, criado em torno do particionamento. Todas as entidades são armazenadas em uma partição e as partições são gerenciadas internamente pelo armazenamento de tabela do Azure. Cada entidade armazenada em uma tabela deve fornecer uma chave de duas partes que inclui:

- A chave de partição. Este é um valor de cadeia de caracteres que determina em qual partição o armazenamento de tabela do Azure colocará a entidade. Todas as entidades com a mesma chave de partição serão armazenadas na mesma partição.
- A chave de linha. Este é outro valor de cadeia de caracteres que identifica a entidade na partição. Todas as entidades em uma partição são classificadas lexicalmente, em ordem crescente, por essa chave. A combinação de chave de linha/chave de partição deve ser exclusiva para cada entidade e não pode exceder 1 KB.

O restante dos dados de uma entidade consiste em campos definidos pelo aplicativo. Nenhum esquema específico é imposto e cada linha pode conter um conjunto diferente de campos definidos pelo aplicativo. A única limitação é que o tamanho máximo de uma entidade (incluindo as chaves de partição e de linha) atualmente é de 1 MB. O tamanho máximo de uma tabela é de 200 TB, embora esses números possam ser alterados no futuro (veja a página [Metas de desempenho e escalabilidade de armazenamento do Azure] no site da Microsoft para obter as informações mais recentes sobre esses limites. Se estiver tentando armazenar entidades que excederem essa capacidade, considere dividi-las em várias tabelas; use o particionamento vertical e divida os campos nos grupos que têm maior probabilidade de ser acessados juntos.

A Figura 7 mostra a estrutura lógica de um exemplo de conta de armazenamento (Dados da Contoso) para um aplicativo de comércio eletrônico fictício. As contas de armazenamento contêm três tabelas (Informações de clientes, Informações de produtos e Informações de pedidos) e cada tabela tem várias partições. Na tabela Informações de clientes, os dados são particionados de acordo com a cidade em que o cliente está localizado e a chave de linha contém a ID do cliente. Na tabela Informações de produtos, os produtos são particionados por categoria de produto e a chave de linha contém o número do produto. Na tabela Informações de pedidos, os pedidos são particionados pela data de inserção e a chave de linha especifica o horário em que o pedido foi recebido. Observe que todos os dados são ordenados pela chave de linha em cada partição.

![](media/best-practices-data-partitioning/TableStorage.png)

_Figura 7. – As tabelas e partições em um exemplo de conta de armazenamento_

> [AZURE.NOTE]O armazenamento de tabela do Azure também adiciona um campo de carimbo de data/hora a cada entidade. O campo de carimbo de data/hora é mantido pelo armazenamento de tabela e é atualizado sempre que a entidade é modificada e gravada de volta em uma partição. O serviço de armazenamento de tabela usa esse campo para implementar a simultaneidade otimista (sempre que um aplicativo grava uma entidade de volta no armazenamento de tabela, o serviço de armazenamento de tabela compara o valor do carimbo de data/hora na entidade gravada com o valor mantido no armazenamento de tabelas; se eles forem diferentes, outro aplicativo deve ter alterado a entidade, já que ela foi recuperada e houve uma falha na operação de gravação). Você não deve modificar esse campo em seu próprio código, nem deve especificar um valor para esse campo ao criar uma nova entidade.

O armazenamento de tabela do Azure usa a chave de partição para determinar como os dados são armazenados. Se uma entidade é adicionada a uma tabela com uma chave de partição não usada anteriormente, o armazenamento de tabela do Azure criará uma nova partição para esta entidade. Outras entidades com a mesma chave de partição serão armazenadas na mesma partição. Esse mecanismo implementa de modo efetivo uma estratégia de escala horizontal automática. Cada partição será armazenada em um único servidor em um datacenter do Azure (para ajudar a garantir que as consultas que recuperam dados de uma única partição sejam executadas rapidamente); no entanto, diferentes partições podem ser distribuídas em vários servidores. Além disso, um único servidor pode hospedar várias partições se essas partições tiverem um tamanho limitado.

Ao criar entidades para o armazenamento de tabela do Azure, considere os pontos a seguir:

- A seleção dos valores da chave de linha e da chave de partição deve ser orientada pela maneira como os dados são acessados. Você deve escolher uma combinação de chave de linha/chave de partição que dê suporte à maioria das suas consultas. As consultas mais eficientes recuperarão os dados especificando a chave de partição e a chave de linha. As consultas que especificam uma chave de partição e um intervalo de chaves de linha podem ser atendidas pela verificação de uma única partição; isso é relativamente rápido, pois os dados são mantidos na ordem de chaves de linha. As consultas que não especificam pelo menos a chave de partição podem exigir o armazenamento de tabela do Azure para verificar todas as partições em busca dos dados.

	> [AZURE.TIP]Se uma entidade tiver uma chave natural, use-a como a chave de partição e especifique uma cadeia de caracteres vazia como a chave de linha. Se uma entidade tiver uma chave composta que consiste em duas propriedades, selecione a propriedade com alteração mais lenta como a chave de partição e a outra como a chave de linha. Se uma entidade tiver mais de duas propriedades de chave, use uma concatenação de propriedades para fornecer as chaves de partição e de linha.

- Se você executa regularmente consultas que pesquisam dados usando campos que não sejam as chaves de partição e de linha, considere implementar o [Padrão da tabela de índice].
- Se você gerar chaves de partição usando uma sequência monotônica crescente ou decrescente (como “0001”, “0002”, “0003”, etc.) e cada partição contiver apenas uma quantidade limitada de dados, o armazenamento de tabela do Azure poderá agrupar fisicamente essas partições no mesmo servidor. Esse mecanismo pressupõe que o aplicativo tem mais probabilidade de executar consultas em um intervalo contíguo de partições (consultas do intervalo) e é otimizado para esse caso. No entanto, essa abordagem pode levar a pontos de acesso com foco em um único servidor, já que todas as inserções de novas entidades provavelmente estarão concentradas em uma ou outra extremidade dos intervalos contíguos. Ela também pode reduzir a escalabilidade. Para distribuir a carga mais uniformemente entre os servidores, considere o hash da partição para tornar a sequência mais aleatória.
- O armazenamento de tabela do Azure dá suporte a operações transacionais para entidades que pertencem à mesma partição. Isso significa que um aplicativo pode executar várias operações de inserção, atualização, exclusão, substituição ou mesclagem como uma unidade atômica (sujeito à condição de que a transação não inclua mais de 100 entidades e que a carga da solicitação não exceda 4 MB). As operações que abrangem várias partições não são transacionais e podem exigir que você implemente a consistência eventual, conforme descrito nas Diretrizes de consistência de dados. Para obter mais informações sobre transações e o armazenamento de tabela, visite a página [Executando transações do grupo de entidade] no site da Microsoft.
- Dê atenção especial à granularidade da chave de partição:
	- Usar a mesma chave de partição para todas as entidades fará com que o serviço de armazenamento de tabela crie uma única partição grande mantida em um único servidor, impedindo que ela se escale horizontalmente e, em vez disso, concentrará a carga em um único servidor. Como resultado, essa abordagem só é adequada para sistemas que gerenciam um número pequeno de entidades. No entanto, essa abordagem garante que todas as entidades possam participar das transações do grupo de entidade.
	- Usar uma chave de partição exclusiva para todas as entidades fará com que o serviço de armazenamento de tabela crie uma partição separada para cada entidade, possivelmente, resultando em um grande número de partições pequenas (dependendo do tamanho das entidades). Essa abordagem é mais escalonável do que usar uma chave de partição única, mas as transações do grupo de entidade não são possíveis e as consultas que buscam mais de uma entidade podem envolver a leitura em mais de um servidor. No entanto, se o aplicativo executar consultas do intervalo, o uso de uma sequência monotônica para gerar as chaves de partição pode ajudar a otimizar essas consultas.
	- Compartilhar a chave da partição entre um subconjunto de entidades permite agrupar entidades relacionadas na mesma partição. As operações que envolvem entidades relacionadas podem ser executadas usando transações do grupo de entidade, e as consultas que buscam um conjunto de entidades relacionadas podem ser atendidas acessando um único servidor.

Para saber mais sobre como particionar dados no armazenamento de tabelas do Azure, confira o artigo [Guia de Design de tabela de armazenamento do Azure] no site da Microsoft.

## Particionamento do armazenamento de blobs do Azure

O Armazenamento de Blobs do Azure permite armazenar objetos binários grandes, atualmente até 200 GB para blobs de blocos ou até 1 TB para blobs de páginas (para obter as informações mais recentes, visite a página [Metas de desempenho e escalabilidade de armazenamento do Azure] no site da Microsoft). Use blobs de blocos em cenários como streaming em que é necessário carregar ou baixar grandes volumes de dados rapidamente. Use blobs de páginas para aplicativos que exigem o acesso aleatório em vez do acesso serial a partes dos dados.

Cada blob (de blocos ou páginas) é mantido em um contêiner em uma conta de armazenamento do Azure. Você pode usar contêineres para agrupar os blobs relacionados que têm os mesmos requisitos de segurança, embora esse agrupamento seja lógico em vez de físico. Dentro de um contêiner, cada blob tem um nome exclusivo.

O armazenamento de blobs é particionado automaticamente com base no nome do blob. Cada blob é mantido em sua própria partição e os blobs no mesmo contêiner não compartilham uma partição. Essa arquitetura permite que o armazenamento de blobs do Azure balanceie a carga entre servidores de modo transparente, já que diferentes blobs no mesmo contêiner podem ser distribuídos em diferentes servidores.

As ações de gravação de um único bloco (blob de blocos) ou uma página (blob de páginas) são atômicas, mas as operações que abrangem blobs, páginas ou blocos não. Se precisar garantir a consistência ao executar operações de gravação em blobs de blocos e páginas, você precisará remover um bloqueio de gravação usando uma concessão de blob.

O armazenamento de blobs do Azure dá suporte a taxas de transferência de até 60 MB por segundo ou 500 solicitações por segundo para cada blob. Se você prevê que esses limites serão excedidos e os dados do blob forem relativamente estáticos, considere replicar os blobs usando a Rede de Distribuição de Conteúdo (CDN) do Azure. Para obter mais informações, visite a página [Usando a CDN para o Azure] no site da Microsoft. Para obter mais diretrizes e considerações, confira o artigo Rede de Distribuição de Conteúdo (CDN).

## Particionamento de filas de armazenamento do Azure

As filas de armazenamento do Azure permitem implementar mensagens assíncronas entre processos. Uma conta de armazenamento do Azure pode conter um número ilimitado de filas e cada fila pode conter um número ilimitado de mensagens. A única limitação é o espaço disponível na conta de armazenamento. O tamanho máximo de uma mensagem individual é de 64 KB. Se precisar de mensagens com tamanhos maiores do que esse, considere usar as filas do Barramento de Serviço do Azure.

Cada fila de armazenamento tem um nome exclusivo dentro da conta de armazenamento na qual ela está contida. O Azure particiona as filas com base no nome, e todas as mensagens da mesma fila são armazenadas na mesma partição, controladas por um único servidor. Diferentes filas podem ser gerenciadas por diferentes servidores para ajudar a balancear a carga. A alocação de filas para servidores é transparente para aplicativos e usuários. Em um aplicativo de grande escala, não use a mesma fila de armazenamento para todas as instâncias do aplicativo, pois essa abordagem pode fazer com que o servidor que hospeda a fila se torne um ponto de acesso; use diferentes filas para diferentes áreas funcionais do aplicativo. As filas de armazenamento do Azure não dão suporte a transações; portanto, direcionar mensagens para diferentes filas deve ter pouco impacto na consistência das mensagens.

Uma fila de armazenamento do Azure pode manipular até 2.000 mensagens por segundo. Se precisar processar mensagens a uma taxa maior que essa, considere criar várias filas. Por exemplo, em um aplicativo global, crie filas de armazenamento separadas em contas de armazenamento separadas para manipular instâncias do aplicativo executadas em cada região.

## Estratégias de particionamento para o Barramento de Serviço do Azure

O Barramento de Serviço do Azure usa um agente de mensagem para manipular as mensagens enviadas para uma fila ou tópico do Barramento de Serviço. Por padrão, todas as mensagens enviadas para uma fila ou tópico são manipuladas pelo mesmo processo de agente de mensagem. Essa arquitetura pode colocar uma limitação na taxa de transferência geral da fila de mensagens. No entanto, você também pode dividir uma fila ou tópico no momento de sua criação definindo a propriedade _EnablePartitioning_ da descrição da fila ou do tópico como _true_. Uma fila ou tópico particionado é dividido em vários fragmentos e cada um deles tem o suporte de um repositório e agente de mensagem separados. O Barramento de Serviço assume a responsabilidade pela criação e gerenciamento desses fragmentos. Quando um aplicativo publica uma mensagem em uma fila ou tópico particionado, o Barramento de Serviço atribui a mensagem a um fragmento dessa fila ou tópico. Quando um aplicativo recebe uma mensagem de uma fila ou assinatura, o Barramento de Serviço verifica cada fragmento em relação à próxima mensagem disponível e, em seguida, transmite-a para o aplicativo para o processamento. Essa estrutura ajuda a distribuir a carga entre agentes e repositórios de mensagem, aumentando a escalabilidade e melhorando a disponibilidade; se o repositório ou agente de mensagem de um fragmento estiver temporariamente indisponível, o Barramento de Serviço pode recuperar mensagens de um dos fragmentos restantes disponíveis.

O Barramento de Serviço atribui uma mensagem a um fragmento da seguinte maneira:

- Se a mensagem pertencer a uma sessão, todas as mensagens com o mesmo valor para a propriedade \_ SessionId\_ são enviadas para o mesmo fragmento.
- Se a mensagem não pertencer a uma sessão, mas o remetente tiver especificado um valor para a propriedade _PartitionKey_, todas as mensagens com o mesmo valor _PartitionKey_ serão enviadas para o mesmo fragmento.

	> [AZURE.NOTE]Se as propriedades _SessionId_ e _PartitionKey_ forem especificadas, deverão ser definidas com o mesmo valor; caso contrário, a mensagem será rejeitada.
- Se as propriedades _SessionId_ e _PartitionKey_ de uma mensagem não forem especificadas, mas a detecção de duplicidades estiver habilitada, a propriedade _MessageId_ será usada. Todas as mensagens com a mesma _MessageId_ serão direcionadas ao mesmo fragmento.
- Se as mensagens não incluírem uma propriedade _SessionId, PartitionKey_ ou _MessageId_, o Barramento de Serviço atribuirá mensagens a fragmentos em um estilo round robin. Se um fragmento estiver indisponível, o Barramento de Serviço passará para o próximo. Dessa forma, uma falha temporária na infraestrutura das mensagens não causa uma falha na operação de envio de mensagem.

Ao decidir como (ou se) particionar uma fila ou tópico de mensagens do Barramento de Serviço, considere os pontos a seguir:

- Tópicos e filas do Barramento de Serviço são criados dentro do escopo de um namespace do Barramento de Serviço. O Barramento de Serviço atualmente permite até 100 filas ou tópicos particionados por namespace.
- Cada namespace do Barramento de Serviço impõe cotas nos recursos disponíveis, como o número de assinaturas por tópico, o número de solicitações simultâneas de envio e recebimento por segundo e o número máximo de conexões simultâneas que podem ser estabelecidas. Essas cotas estão documentadas no site da Microsoft, na página [Cotas do Barramento de Serviço]. Se você prevê que esses valores serão excedidos, crie namespaces adicionais com suas próprias filas e tópicos e distribua o trabalho entre esses namespaces. Por exemplo, em um aplicativo global, crie namespaces separados em cada região e configure instâncias do aplicativo para usar as filas e tópicos no namespace mais próximo.
- As mensagens enviadas como parte de uma transação devem especificar uma chave de partição. Ela pode ser _SessionId, PartitionKey_ ou _MessageId_. Todas as mensagens enviadas como parte da mesma transação devem especificar a mesma chave de partição, pois elas devem ser manipuladas pelo mesmo processo de agente de mensagem. Você não pode enviar mensagens para diferentes filas ou tópicos na mesma transação.
- Não é possível configurar uma fila ou tópico particionado para ser automaticamente excluído quando se tornar ocioso.
- Se estiver criando soluções híbridas ou de plataforma cruzada, atualmente não é possível usar filas e tópicos particionados com o Protocolo de Enfileiramento de Mensagens Avançado (AMQP).

## Estratégias de particionamento para o Banco de Dados de Documentos do Azure

O Banco de Dados de Documentos do Azure é um banco de dados NoSQL que pode armazenar documentos. Um documento no Banco de Dados de Documentos é uma representação serializada pelo JSON de um objeto ou outro dado. Nenhum esquema fixo é imposto, exceto pelo fato de que cada documento deve conter uma ID exclusiva.

Os documentos são organizados em coleções. Uma coleção permite agrupar documentos relacionados. Por exemplo, em um sistema que mantém postagens de blog, você poderia armazenar o conteúdo de cada postagem de blog como um documento em uma coleção e criar coleções para cada tipo de assunto. Como alternativa, em um aplicativo multilocatário, como um sistema que permite que diferentes autores controle e gerenciem suas postagens de blog, você poderia particionar os blogs por autor e criar uma coleção separada para cada um deles. O espaço de armazenamento alocado às coleções é flexível e pode diminuir ou aumentar conforme necessário.

As coleções de documentos fornecem um mecanismo natural para particionar dados em um banco de dados individual. Internamente, um banco de dados do Banco de Dados de Documentos pode abranger vários servidores e o Banco de Dados de Documentos poderá tentar distribuir a carga com a distribuição de coleções entre servidores. A maneira mais simples de implementar a fragmentação é criar uma coleção para cada fragmento.

> [AZURE.NOTE]Cada Banco de Dados de Documentos recebe recursos em termos de um _nível de desempenho_. Um nível de desempenho é associado a um limite de taxa da _unidade de solicitação_ (RU). O limite de taxa da RU especifica o volume de recursos que será reservado para essa coleção e está disponível para uso exclusivo dessa coleção. O custo de uma coleção depende do nível de desempenho selecionado para essa coleção; quanto maior o nível do desempenho (e o limite de taxa da RU), maior o custo. É possível ajustar o nível de desempenho de uma coleção usando o portal de gerenciamento do Azure. Para obter mais informações, visite a página [Níveis de desempenho no Banco de Dados de Documentos] no site da Microsoft.

Todos os bancos de dados são criados no contexto de uma conta do Banco de Dados de Documentos. Uma única conta do Banco de Dados de Documentos pode conter vários bancos de dados e especifica em qual região os bancos de dados são criados. Cada conta do Banco de Dados de Documentos também impõe seu próprio controle de acesso. Você pode usar contas do Banco de Dados de Documentos para localizar os fragmentos geograficamente (coleções nos bancos de dados) próximo aos usuários que precisam acessá-los e impor restrições para que somente esses usuários possam se conectar a eles.

Cada conta do Banco de Dados de Documentos tem uma cota que limita o número de bancos de dados e coleções que ela pode conter e a quantidade de armazenamento de documentos disponível. Esses limites estão sujeitos a alterações, mas são descritos na página [Limites e cotas do Banco de Dados de Documentos] no site da Microsoft. É teoricamente possível que, se você implementar um sistema em que todos os fragmentos pertençam ao mesmo banco de dados, você atinja o limite de capacidade de armazenamento da conta. Nesse caso, talvez seja necessário criar contas e bancos de dados adicionais do Banco de Dados de Documentos e distribuir os fragmentos entre esses bancos de dados. No entanto, mesmo se for improvável que você atinja a capacidade de armazenamento de um banco de dados, um bom motivo para usar vários bancos de dados é que cada banco de dados tem o seu próprio conjunto de usuários e permissões. É possível usar esse mecanismo para isolar o acesso a coleções por banco de dados.

A Figura 8 ilustra a estrutura de alto nível da arquitetura do Banco de Dados de Documentos.

![](media/best-practices-data-partitioning/DocumentDBStructure.png)

_Figura 8. – A estrutura do Banco de Dados de Documentos_

É responsabilidade do aplicativo cliente direcionar as solicitações ao fragmento adequado, normalmente pela implementação do seu próprio mecanismo de mapeamento com base em alguns atributos dos dados que definem a chave de fragmento. A Figura 9 mostra dois bancos de dados do Banco de Dados de Documentos; cada um deles contém duas coleções que agem como fragmentos. Os dados são fragmentados por ID de locatário e contêm os dados de um locatário específico. Os bancos de dados são criados em contas separadas do Banco de Dados de Documentos localizadas na mesma região que aquela dos locatários cujos dados ela contêm. A lógica de roteamento no aplicativo cliente usa a ID do locatário como a chave de fragmento.

![](media/best-practices-data-partitioning/DocumentDBPartitions.png)

_Figura 9. – Implementando a fragmentação usando o Banco de Dados de Documentos do Azure_

Ao decidir como particionar dados com o Banco de Dados de Documentos, considere os pontos a seguir:

- Os recursos disponíveis para um banco de dados do Banco de Dados de Documentos estão sujeitos às limitações de cota da conta do Banco de Dados de Documentos. Cada banco de dados pode conter um número de coleções (lembrando que há um limite) e cada coleção é associada a um nível de desempenho que rege o limite de taxa da RU (taxa de transferência reservada) para essa coleção. Para obter mais informações, visite a página [Limites e cotas do Banco de Dados de Documentos] no site da Microsoft.
- Cada documento deve ter um atributo que pode ser usado para identificar exclusivamente o documento dentro da coleção na qual ele é mantido. Isso é diferente da chave de fragmento que define em qual coleção o documento é mantido. Uma coleção pode conter um grande número de documentos, teoricamente limitado apenas pelo tamanho máximo da ID do documento. A ID do documento pode ter até 255 caracteres.
- Todas as operações em um documento são executadas no contexto de uma transação que está dentro do escopo da coleção na qual o documento está contido. Se uma operação falhar, o trabalho realizado será revertido. Quando um documento está sujeito a uma operação, todas as alterações feitas estão sujeitas ao isolamento no nível do instantâneo. Esse mecanismo garante que, se por exemplo, houver uma falha em uma solicitação para a criação de um novo documento, outro usuário que consultar o banco de dados simultaneamente não verá um documento parcial que será então removido.
- As consultas do Banco de Dados de Documentos também estão dentro do escopo do nível da coleção. Uma única consulta só pode recuperar dados de uma coleção. Se precisar recuperar dados de várias coleções, é necessário consultar cada coleção individualmente e mesclar os resultados no código do aplicativo.
- O Banco de Dados de Documentos dá suporte a itens programáveis que podem ser armazenados em uma coleção juntamente com os documentos: procedimentos armazenados, funções definidas pelo usuário e gatilhos (escritos em JavaScript). Esses itens podem acessar qualquer documento na mesma coleção. Além disso, esses itens são executados dentro do escopo da transação de ambiente (no caso de um gatilho que é acionado como resultado de uma operação de criação, exclusão ou substituição executada em um documento) ou por iniciar uma nova transação (no caso de um procedimento armazenado executado como resultado de uma solicitação de cliente explícita). Se o código em um item programável lançar uma exceção, a transação será revertida. Você pode usar procedimentos armazenados e gatilhos para manter a integridade e a consistência entre documentos, mas esses documentos devem fazer parte da mesma coleção.
- Verifique se as coleções que você pretende manter nos bancos de dados em uma conta do Banco de Dados de Documentos não têm a probabilidade de exceder os limites de taxa de transferência definidos pelos níveis de desempenho das coleções. Esses limites são descritos na página [Gerenciamento das necessidades de capacidade do Banco de Dados de Documentos] no site da Microsoft. Se você prevê que esses limites serão atingidos, considere dividir as coleções em bancos de dados em diferentes contas do Banco de Dados de Documentos para reduzir a carga por coleção.

## Estratégias de particionamento para a Pesquisa do Azure

A capacidade de pesquisar dados geralmente é o principal método de navegação e exploração fornecido por muitos aplicativos da Web, o que permite que os usuários localizem recursos rapidamente (por exemplo, produtos em um aplicativo de comércio eletrônico) com base nas combinações de critérios de pesquisa. O serviço de Pesquisa do Azure fornece recursos de pesquisa de texto completo em relação ao conteúdo da Web e inclui recursos como consultas recomendadas de preenchimento automático com base em correspondências aproximadas e na navegação facetada. Uma descrição completa desses recursos está disponível na página [O que é a Pesquisa do Azure?] no site da Microsoft.

O serviço de Pesquisa armazena conteúdo pesquisável como documentos JSON em um banco de dados. Defina os índices que especificam os campos pesquisáveis nesses documentos e forneça essas definições ao serviço de Pesquisa. Quando um usuário envia uma solicitação de pesquisa, o serviço de Pesquisa usa os índices adequados para localizar itens correspondentes.

Para reduzir a contenção, o armazenamento usado pelo serviço de Pesquisa pode ser dividido em 1, 2, 3, 4, 6 ou 12 partições e cada partição pode ser replicada até seis vezes. O produto do número de partições multiplicado pelo número de réplicas é chamado de _Unidade de Pesquisa_ (SU). Uma única instância do Serviço de Pesquisa pode conter um máximo de 36 SUs (um banco de dados com 12 partições somente dá suporte a um máximo de 3 réplicas). Você será cobrado por cada SU alocada ao seu serviço. Conforme o volume de conteúdo pesquisável ou a taxa de solicitações de pesquisa aumenta, é possível adicionar SUs a uma instância existente do serviço de Pesquisa para manipular a carga extra. O próprio Serviço de Pesquisa assume a responsabilidade por distribuir os documentos uniformemente entre as partições; atualmente não há suporte para estratégias de particionamento manual.

Cada partição pode conter um máximo de 15 milhões de documentos ou ocupar 300 GB de espaço de armazenamento (o que for menor, dependendo do tamanho dos documentos e índices). Você pode criar até 50 índices. O desempenho do serviço poderá variar dependendo da complexidade dos documentos, dos índices disponíveis e dos efeitos da latência de rede. Em média, uma única réplica (1 SU) deve ser capaz de manipular 15 consultas por segundo (QPS), embora seja necessário testar o desempenho com seus próprios dados para obter uma medida mais precisa da taxa de transferência. Para obter mais informações, confira a página [Limites de serviço na Pesquisa do Azure] no site da Microsoft.

> [AZURE.NOTE]Você pode armazenar um conjunto limitado de tipos de dados em documentos pesquisáveis, cadeias de caracteres, boolianos, dados numéricos, dados de data/hora e alguns dados geográficos. Para obter mais detalhes, visite a página [Tipos de dados com suporte (Pesquisa do Azure)] no site da Microsoft.

Você tem controle limitado sobre como o serviço de Pesquisa do Azure particiona os dados para cada instância do serviço. No entanto, em um ambiente global, é possível melhorar o desempenho e reduzir ainda mais a latência e contenção pelo particionamento do próprio serviço usando qualquer uma das seguintes estratégias:

- Crie uma instância do serviço de Pesquisa em cada região geográfica e verifique se os aplicativos cliente são direcionados à instância disponível mais próxima. Essa estratégia exige que todas as atualizações de conteúdo pesquisável sejam replicadas pontualmente em todas as instâncias do serviço.
- Crie duas camadas do serviço de Pesquisa: um serviço local em cada região que contém os dados frequentemente acessados pelos usuários nessa região e um serviço global que abrange todos os dados. Os usuários podem direcionar solicitações ao serviço local (para resultados rápidos, mas limitados) ou ao serviço global (para obter resultados mais lentos, mas mais completos). Essa abordagem é mais adequada quando há uma variação regional significativa nos dados pesquisados.

## Estratégias de particionamento para o Cache Redis do Azure

O Cache Redis do Azure fornece um serviço de cache compartilhado na nuvem baseado no repositório de dados de chave/valor do Redis. Como o seu nome indica, o Cache Redis do Azure pretende ser uma solução de cache e assim ele só deve ser usado para armazenar dados transitórios, não como um repositório de dados permanente. Os aplicativos que usam o Cache Redis do Azure devem ser capazes de continuar funcionando se o cache estiver indisponível. O Cache Redis do Azure dá suporte à replicação primária/secundária para fornecer alta disponibilidade, mas atualmente limita o tamanho máximo do cache para 53 GB. Se você precisar de mais espaço, é necessário criar caches adicionais. Para obter mais informações, visite a página [Cache Redis do Azure] no site da Microsoft.

O particionamento de um repositório de dados do Redis envolve a divisão dos dados entre instâncias do serviço do Redis. Cada instância constitui uma única partição. O Cache Redis do Azure abstrai os serviços do Redis por trás de uma fachada e não os expõe diretamente. A maneira mais simples de implementar o particionamento é criar vários caches Redis do Azure e distribuir os dados entre eles. Você pode associar cada item de dados a um identificador (uma chave de partição) que especifica em qual cache ele deve ser armazenado. A lógica do aplicativo cliente pode usar esse identificador para rotear solicitações para a partição adequada. Esse esquema é muito simples, mas se o esquema de particionamento for alterado (se outros Caches Redis do Azure forem criados, por exemplo), os aplicativos cliente precisarão ser reconfigurados.

O Redis nativo (não o Cache Redis do Azure) dá suporte ao particionamento do lado do servidor com base no clustering do Redis. Nessa abordagem, os dados são divididos uniformemente entre os servidores usando um mecanismo de hash. Cada servidor do Redis armazena metadados que descrevem o intervalo de chaves de hash contidos na partição, além de conter informações sobre quais chaves de hash estão localizadas nas partições em outros servidores. Os aplicativos cliente simplesmente enviam solicitações para qualquer um dos servidores do Redis participantes (provavelmente o servidor mais próximo). O servidor do Redis examina a solicitação do cliente e, se ela puder ser resolvida localmente, ele executará a operação solicitada; caso contrário, ele encaminhará a solicitação para o servidor adequado. Esse modelo é implementado usando o clustering do Redis e é descrito mais detalhadamente na página [tutorial de cluster do Redis] no site do Redis. O clustering do Redis é transparente para os aplicativos cliente e outros servidores do Redis podem ser adicionados ao cluster (e os dados particionados novamente) sem a necessidade de reconfigurar os clientes.

> [AZURE.IMPORTANT]Atualmente, o Cache Redis do Azure não dá suporte ao clustering do Redis. Se desejar implementar essa abordagem com o Azure, é necessário implementar seus próprios servidores do Redis instalando o Redis em um conjunto de máquinas virtuais do Azure e configurando-as manualmente. A página [Executando o Redis em uma VM com CentOS Linux no Azure] no site da Microsoft explica um exemplo que mostra como criar e configurar um nó do Redis executado como uma VM do Azure.

A página [Particionamento: como dividir dados entre várias instâncias do Redis] no site do Redis fornece mais informações sobre a implementação do particionamento com o Redis. Ao ler o restante desta seção, pressupomos que você esteja implementando o particionamento do lado do cliente ou assistido por proxy.

Ao decidir como particionar dados com o Cache Redis do Azure, considere os pontos a seguir:

- O Cache Redis do Azure não pretende agir como um repositório de dados permanente; portanto, qualquer esquema de particionamento no qual você implemente o código do aplicativo deve estar preparado para aceitar que os dados não foram encontrados no cache e foram recuperados de outro lugar.
- Mantenha os dados frequentemente acessados juntos na mesma partição. O Redis é um eficiente repositório de chave/valor que fornece vários mecanismos altamente otimizados para a estruturação de dados, que vai desde cadeias de caracteres simples (na verdade, dados binários de até 512 MB) até tipos de agregação como listas (que podem agir como filas e pilhas), conjuntos (ordenados e não ordenados) e hashes (que podem agrupar campos relacionados, como os itens que representam os campos em um objeto). Os tipos de agregação permitem associar vários valores relacionados à mesma chave; uma chave do Redis identifica uma lista, conjunto ou hash, em vez dos itens de dados que ele contém. Esses tipos estão disponíveis com o Cache Redis do Azure e são descritos na página [Tipos de dados] no site do Redis. Por exemplo, em uma parte de um sistema de comércio eletrônico que rastreia os pedidos feitos pelos clientes, os detalhes de cada cliente podem ser armazenados em um hash do Redis digitado usando a ID do cliente. Cada hash pode manter um conjunto de IDs de pedido do cliente. Um conjunto separado do Redis poderia manter os pedidos, novamente estruturados como hashes, digitados com o uso da ID de pedido. A Figura 10 mostra essa estrutura. Observe que o Redis não implementa nenhuma forma de integridade referencial; portanto, é responsabilidade do desenvolvedor manter os relacionamentos entre clientes e pedidos.

![](media/best-practices-data-partitioning/RedisCustomersandOrders.png)

_Figura 10. – Estrutura recomendada no armazenamento do Redis para registrar pedidos de clientes e seus dados_

> [AZURE.NOTE]No Redis, todas as chaves são valores de dados binários (como cadeias de caracteres do Redis) e podem conter até 512 MB de dados; portanto, teoricamente, uma chave pode conter praticamente qualquer informação. No entanto, você deve adotar uma convenção de nomenclatura consistente para chaves que descreva o tipo de dados e identifique a entidade, mas que não seja excessivamente longa. Uma abordagem comum é usar chaves da forma “entity\_type:ID”, como “customer:99” para indicar a chave para o cliente com a ID 99.

- Você pode implementar o particionamento vertical armazenando informações relacionadas em diferentes agregações no mesmo banco de dados. Por exemplo, em um aplicativo de comércio eletrônico, você poderia armazenar as informações frequentemente acessadas sobre produtos em um hash do Redis e as informações detalhadas usadas com menos frequência em outro. Os dois hashes poderiam usar a mesma ID de produto como parte da chave, por exemplo, “product:_nn_” onde _nn_ é a ID do produto para as informações de produto e “product\_details: _nn_” para os dados detalhados. Essa estratégia pode ajudar a reduzir o volume de dados que a maioria das consultas provavelmente recuperará.
- Reparticionamento de um repositório de dados do Redis é uma tarefa complexa e demorada. O clustering do Redis pode reparticionar os dados automaticamente, mas esse recurso não está disponível com o Cache Redis do Azure. Portanto, ao criar o esquema de particionamento, você deve se esforçar para deixar espaço livre suficiente em cada partição para permitir o crescimento esperado dos dados com o tempo. No entanto, lembre-se de que o Cache Redis do Azure foi criado para armazenar os dados em cache temporariamente e que os dados mantidos no cache podem ter um tempo de vida limitado especificado como um valor de tempo de vida (TTL). Para dados relativamente voláteis, o TTL deve ser curto, mas para dados estáticos, o TTL pode ser muito mais longo. Evite armazenar grandes quantidades de dados com tempo de vida longo no cache se o volume de dados tiver a probabilidade de preencher o cache. É possível especificar uma política de remoção que faz com que o Cache Redis do Azure remova os dados se o espaço for um fator determinante.

	> [AZURE.NOTE]O Cache Redis do Azure permite especificar o tamanho máximo do cache (de 250 MB a 53 GB) selecionando a camada de preços adequada. No entanto, depois de criar um Cache Redis do Azure, não é possível aumentar (ou diminuir) seu tamanho.

- Como os lotes e as transações do Redis não abrangem várias conexões, todos os dados afetados por um lote ou transação devem ser mantidos no mesmo banco de dados (fragmento).

	> [AZURE.NOTE]Uma sequência de operações em uma transação do Redis não é necessariamente atômica. Os comandos que compõem uma transação são verificados e enfileirados antes da execução e se um erro for causado durante essa fase a fila inteira será descartada. No entanto, depois que a transação for enviada com êxito, os comandos na fila serão executados na sequência. Se algum comando falhar, apenas esse comando será cancelado; todos os comandos anteriores e posteriores na fila serão executados. Se precisar, execute operações atômicas. Para obter mais informações, visite a página [Transações] no site do Redis.

- O Redis dá suporte a um número limitado de operações atômicas e as únicas operações desse tipo que dão suporte a várias chaves e valores são MGET (que retorna uma coleção de valores para uma lista especificada de chaves) e MSET (que pode armazenar uma coleção de valores para uma lista especificada de chaves). Se precisar usar essas operações, os pares de chave/valor referenciados pelos comandos MSET e MGET devem ser armazenados no mesmo banco de dados.

## Rebalanceando partições

À medida que um sistema amadurece e o padrão de uso se torna mais bem compreendido, é possível que seja necessário ajustar o esquema de particionamento. Isso pode ser causado por partições individuais que atraem um volume desproporcional de tráfego e se tornam mais acessadas, levando a uma contenção excessiva. Além disso, você pode ter subestimado o volume de dados em algumas partições, fazendo com que você se aproxime dos limites da capacidade de armazenamento nessas partições. Seja qual for a causa, às vezes, é necessário rebalancear as partições para distribuir a carga de maneira mais uniforme.

Em alguns casos, os sistemas de armazenamento de dados que não expõem publicamente a maneira na qual os dados são alocados aos servidores podem automaticamente a rebalancear partições dentro dos limites dos recursos disponíveis. Em outras situações, o rebalanceamento é uma tarefa administrativa que consiste em dois estágios:

1. Determinar a nova estratégia de particionamento para determinar quais partições podem precisar ser divididas (ou possivelmente combinadas) e como alocar dados a essas novas partições pela criação de novas chaves de partição.
2. Migrar os dados afetados do esquema de particionamento antigo para o novo conjunto de partições.

> [AZURE.NOTE]O mapeamento das coleções do Banco de Dados de Documentos para os servidores é transparente, mas você ainda poderá atingir os limites de capacidade de armazenamento e de taxa de transferência de uma conta do Banco de Dados de Documentos; nesse caso, talvez seja necessário recriar o esquema de particionamento e migrar os dados.

Dependendo da tecnologia de armazenamento de dados e o design do seu sistema de armazenamento de dados, você poderá migrar os dados entre partições enquanto eles estiverem sendo usados (migração online). Se isso não for possível, talvez você precise fazer com que as partições afetadas fiquem temporariamente indisponíveis enquanto os dados são realocados (migração offline).

## Migração offline

Sem dúvida, a migração offline é a abordagem mais simples, pois reduz a possibilidade de contenção; os dados migrados não devem ser alterados enquanto estiverem sendo movidos e reestruturados.

Conceitualmente, esse processo consiste nas etapas a seguir:

1. Marcar o fragmento offline;
2. Dividir/mesclar e mover os dados para os novos fragmentos;
3. Verificar os dados;
4. Colocar os novos fragmentos online;
5. Remover o fragmento antigo.

Para manter um pouco de disponibilidade, é possível marcar o fragmento original como somente leitura na etapa 1 em vez de torná-lo indisponível. Isso permite que os aplicativos leiam os dados enquanto eles estão sendo movidos, mas não alterá-los.

## Migração online

A migração online é mais complexa de executar, mas causa menos interrupção para os usuários, já que os dados permanecem disponíveis durante todo o procedimento. O processo é semelhante aquele usado pela migração offline, com a exceção de que o fragmento original não está marcado como offline (etapa 1). Dependendo da granularidade do processo de migração (item por item ou fragmento por fragmento), o código de acesso a dados nos aplicativos cliente pode ter que manipular os dados de leitura e gravação mantidos em dois locais (o fragmento original e o novo fragmento)

Para obter um exemplo de uma solução que ofereça suporte à migração online, confira [Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico], documentado online no site da Microsoft.

## Diretrizes e padrões relacionados

Os seguintes padrões também podem ser relevantes para o seu cenário ao considerar estratégias para a implementação da consistência dos dados:

- A página [Data Consistency Primer], disponível no site da Microsoft, descreve estratégias para manter a consistência em um ambiente distribuído como a nuvem.
- A página [Diretrizes de particionamento de dados] no site da Microsoft fornece uma visão geral da criação de partições para atender a vários critérios em uma solução distribuída.
- O [Padrão de fragmentação], descrito no site da Microsoft, resume algumas estratégias comuns para a fragmentação de dados.
- O [Padrão da tabela de índice], descrito no site da Microsoft, ilustra como criar índices secundários para os dados. Essa abordagem permite que um aplicativo recupere rapidamente dados usando consultas que não referenciam a chave primária de uma coleção.
- O [Padrão de exibição materializada], discutido no site da Microsoft, descreve como gerar exibições pré-populadas que resumem os dados para dar suporte a operações de consulta rápidas. Essa abordagem pode ser útil em um repositório de dados particionados se as partições que contêm os dados resumidos forem distribuídas em vários locais.
- O artigo [Usando a CDN para Azure] fornece mais orientação sobre como configurar e usar a CDN com o Azure.

## Mais informações

- A página [O que é o Banco de Dados SQL do Azure?], no site da Microsoft, fornece documentação detalhada que descreve como criar e usar bancos de dados SQL.
- A página [Visão geral dos recursos do Banco de Dados Elástico], no site da Microsoft, fornece uma introdução abrangente ao Banco de Dados Elástico.
- O tópico [Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico], no site da Microsoft, contém informações sobre como usar o serviço de Divisão/Mesclagem para gerenciar fragmentos do Banco de Dados Elástico.
- A página [Metas de desempenho e escalabilidade de armazenamento do Azure](https://msdn.microsoft.com/library/azure/dn249410.aspx) no site da Microsoft documenta os limites atuais de tamanho e taxa de transferência do armazenamento do Azure.
- A página [Executando transações do grupo de entidade] no site da Microsoft fornece informações detalhadas sobre como implementar operações transacionais nas entidades armazenadas no armazenamento de tabela do Azure.
- O artigo [Guia de Design de tabela de armazenamento do Azure], no site da Microsoft, contém informações detalhadas sobre como particionar dados no armazenamento de tabelas do Azure.
- A página [Usando a CDN para o Azure] no site da Microsoft descreve como replicar os dados mantidos no Armazenamento de Blobs do Azure usando a Rede de Distribuição de Conteúdo (CDN) do Azure.
- A página [Gerenciar as necessidades de capacidade do Banco de Dados de Documentos], no site da Microsoft, contém informações sobre como o Banco de Dados de Documentos do Azure aloca recursos para bancos de dados.
- A página [O que é a Pesquisa do Azure?], no site da Microsoft, fornece uma descrição completa dos recursos disponíveis com o serviço Pesquisa do Azure.
- A página [Limites de serviço na Pesquisa do Azure], no site da Microsoft, contém informações sobre a capacidade de cada instância do serviço Pesquisa do Azure.
- A página [Tipos de dados com suporte (Pesquisa do Azure)] no site da Microsoft resume os tipos de dados que podem ser usados em documentos e índices pesquisáveis.
- A página [Cache Redis do Azure], no site da Microsoft, fornece uma introdução ao Cache Redis do Azure.
- A página [Particionamento: como dividir dados entre várias instâncias do Redis] no site do Redis fornece informações sobre a implementação do particionamento com o Redis.
- A página [Executando o Redis em uma VM com CentOS Linux no Azure] no site da Microsoft explica um exemplo que mostra como criar e configurar um nó do Redis executado como uma VM do Azure.
- A página [Tipos de dados] no site do Redis descreve os tipos de dados disponíveis com o Redis e o Cache Redis do Azure.

[Cache Redis do Azure]: http://azure.microsoft.com/services/cache/
[Metas de desempenho e escalabilidade de armazenamento do Azure]: storage/storage-scalability-targets.md
[Guia de Design de tabela de armazenamento do Azure]: storage/storage-table-design-guide.md
[Compilando uma solução poliglota]: https://msdn.microsoft.com/library/dn313279.aspx
[Acesso a Dados para Soluções Altamente Escalonáveis: Usando o SQL, NoSQL e Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx
[Data Consistency Primer]: http://aka.ms/Data-Consistency-Primer
[Diretrizes de particionamento de dados]: https://msdn.microsoft.com/library/dn589795.aspx
[Tipos de dados]: http://redis.io/topics/data-types
[Limites e cotas do Banco de Dados de Documentos]: documentdb/documentdb-limits.md
[Visão geral dos recursos do Banco de Dados Elástico]: sql-database/sql-database-elastic-scale-introduction.md
[Utilitário de Migração das Federações]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[Padrão da tabela de índice]: http://aka.ms/Index-Table-Pattern
[Gerenciamento das necessidades de capacidade do Banco de Dados de Documentos]: documentdb/documentdb-manage.md
[Gerenciar as necessidades de capacidade do Banco de Dados de Documentos]: documentdb/documentdb-manage.md
[Padrão de exibição materializada]: http://aka.ms/Materialized-View-Pattern
[Consulta de vários fragmentos]: sql-database/sql-database-elastic-scale-multishard-querying.md
[Particionamento: como dividir dados entre várias instâncias do Redis]: http://redis.io/topics/partitioning
[Níveis de desempenho no Banco de Dados de Documentos]: documentdb/documentdb-performance-levels.md
[Executando transações do grupo de entidade]: https://msdn.microsoft.com/library/azure/dd894038.aspx
[tutorial de cluster do Redis]: http://redis.io/topics/cluster-tutorial
[Executando o Redis em uma VM com CentOS Linux no Azure]: http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx
[Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico]: sql-database/sql-database-elastic-scale-overview-split-and-merge.md
[Usando a CDN para Azure]: cdn/cdn-how-to-use-cdn.md
[Usando a CDN para o Azure]: cdn/cdn-how-to-use-cdn.md
[Cotas do Barramento de Serviço]: service-bus/service-bus-quotas.md
[Limites de serviço na Pesquisa do Azure]: search/search-limits-quotas-capacity.md
[Padrão de fragmentação]: http://aka.ms/Sharding-Pattern
[Tipos de dados com suporte (Pesquisa do Azure)]: https://msdn.microsoft.com/library/azure/dn798938.aspx
[Transações]: http://redis.io/topics/transactions
[O que é a Pesquisa do Azure?]: search/search-what-is-azure-search.md
[O que é o Banco de Dados SQL do Azure?]: sql-database/sql-database-technical-overview.md

<!---HONumber=AcomDC_1223_2015-->