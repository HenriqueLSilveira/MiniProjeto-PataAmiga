## Mini Projeto Avaliativo - Módulo 2

ANÁLISE DE DADOS — PROJETO PATA AMIGA

Identificação:  Henrique Silveira

Turma: Análise de Dados T2

Base de dados: Dados Rede Pata Amiga

## O Projeto:

O objetivo deste trabalho é permitir a analise dos dados da rede de pet shops Pata Amiga utilizando MySQL e um modelo dimensional e responder as perguntas de negócio do cliente.
Para isso, os dados disponivilizados foram padronizados e organizados.

A base possui 4.044 pedidos, realizados entre setembro de 2023 e março de 2024, além de informações sobre lojas, categorias de produtos e praças de atendimento.

## Estrutura:

O projeto contém os seguintes arquivos:

Arquivos fornecidos para realizar o inicio do projeto:

00-conferencia.sql 

01-carga-staging.sql 

02-dimensoes-prontas.sql

06-Dados brutos

Arquivos preenchidos com a analise:

03-dimensoes.sql = Dimensões 

04-fato.sql = Acontecimentos principais

05-perguntas.sql = Respostas das perguntas de negócio

README.md = AApresentação do projeto

diagrama.png= Diagrama do modelo estrela da modelagem do banco

## Pré-requisitos

Certifique-se de ter o MySQL para execução do projeto!

Como Executar?
1. Abra os arquivos de scripts na ordem em que estão enumerados, a partir do '01'

2. Copie todo o conteúdo

3. Abra uma instância do MySQL e conecte

4. Abra uma aba de Query

5. Cole todo o conteúdo copiado

6. Clique em executar tudo

Nesta sequência, primeiro foram realizadas conferêncidas dos dados brutos, depois o staging foi carregado. Em seguida, a construção das dimensões e a tabela ponte, depois a tabela fato e, por último, as consultas para responder às perguntas.

## Tratamento dos dados

As tabelas de staging não foram alteradas. O tratamento foi feito durante a inserção dos dados nas tabelas dimensionais e na fato.

-------
Correção das datas no formato americano, exemplo:

09/01/2023 10:27 AM

Utilizando:

STR_TO_DATE(campo, '%m/%d/%Y %h:%i %p') 

Os marcos da entrega estavam em outro formato, YYYY-MM-DD, e foram tratados separadamente.

-------
Os valores também não estavam padronizados. Foram encontrados casos como:

R$ 1.850,00

1850.00

Os valores vazios ou representados por - foram mantidos como NULL. 

Haviam 18 grafias diferentes de categorias.

Foi criada uma regra para transformar essas grafias em categorias padronizadas.

Também havia muitas formas diferentes de escrever o nome das lojas.

Foi feita a padronização antes de realizar o relacionamento com a dim_loja.

Além de remover /SC e espaços extras, também foram corrigidos:

- **Blumenal** → Blumenau;
- **Floripa** → Florianópolis;
- **Jgua** → Jaraguá.

Haviam 1.575 pedidos sem código da loja, porém no final somente 3 pedidos ficaram realmente sem identificação, pois não tinham nome de loja para fazer o relacionamento.

Esses registros foram direcionados para a chave -1.

Os valores de desconto foram padronizados para:

- **Sim**
- **Nao**
- **Nao Informado**

Os canais foram transformados em:

- **App**
- **Site**
- **Loja Fisica**
- **Telefone**
- **WhatsApp**
- **Nao Informado**

Para WHATS foi colocado antes de APP, porque a palavra WhatsApp contém “APP”.

-------
Valores encontrados nos dados de origem:
    

| Indicador | Quantidade |
|---|---:|
| Pedidos | 4.044 |
| Lojas | 32 |
| Relações loja × praça | 48 |
| Grafias de loja | 50 |
| Grafias de categoria | 18 |
| Pedidos sem código da loja | 1.575 |
| Pedidos sem nome da loja | 3 |
| Separação em branco | 1.077 |
| Nota em branco | 1.338 |
| Despacho em branco | 1.665 |
| Entrega em branco | 1.953 |

Esses problemas foram considerados no processo de carga.

## Diagrama da Modelagem
Imagem com diagrama do modelo dimensional do projeto.

Com a fato_pedido, contendo uma linha para cada pedido. Tem relação com relaciona às dimensões de loja, categoria e tempo, e dim_tempo, também utilizada tanto para a data do pedido quanto para a data da entrega.

Já a dimensão praça é relacionada por meio da bridge_loja_praca, permitindo representar lojas que atendem mais de uma praça e realizar o rateio do faturamento.

![Imagem da modelagem do banco](<diagrama.png>)

## ANÁLISES

1. Onde está o gargalo da entrega?

Resultado consulta:

| Porte | Integração → Separação | Separação → Nota | Nota → Despacho | Despacho → Entrega | Total |
|---|---:|---:|---:|---:|---:|
| Pequena | 3,02 | 0,69 | 8,53 | 2,86 | 15,16 |
| Média | 1,98 | 0,62 | 3,34 | 2,03 | 7,95 |
| Grande | 1,96 | 0,64 | 3,32 | 2,01 | 7,93 |

O gargalo está na etapa de NOTA >> DESPACHO.

É possível notar nos três tipos de porte de loja, porém nas menores esse gargalo é maior.

As lojas médias e grandes ficam próximas de 8 dias no processo total e as pequenas chegam a 15,16 dias.

Verificar um possível problema operacional neste processo nas lojas menores.

2. Qual categoria representa a maior parte do faturamento?


Resultado consulta:

| Categoria | Faturamento | Participação |
|---|---:|---:|
| Ração | R$ 1.076.202,55 | 60,01% |
| Medicamento | R$ 305.904,03 | 17,06% |
| Petisco | R$ 128.590,16 | 7,17% |
| Serviço | R$ 94.001,37 | 5,24% |
| Higiene | R$ 92.314,45 | 5,15% |
| Acessório | R$ 64.661,39 | 3,61% |
| Brinquedo | R$ 31.634,56 | 1,76% |

A Ração representa aproximadamente 60% do faturamento em todos os portes de loja.

Há uma receita considerável nessa categoria, sendo a categoria de maior importância atualmente.

É interessante acompanhar as outras categorias para evitar uma dependência muito grande de apenas uma delas.

3. O desconto apresenta o mesmo comportamento em todos os canais?

Resultado consulta:

| Canal | Sem desconto | Com desconto |
|---|---:|---:|
| App | R$ 167,63 | R$ 488,04 |
| Loja Física | R$ 197,55 | R$ 494,04 |
| Site | R$ 189,68 | R$ 501,92 |
| Telefone | R$ 195,23 | R$ 514,02 |
| WhatsApp | R$ 179,26 | R$ 514,33 |

O ticket médio dos pedidos é maior quando há descontos, nos canais analisados, porém não é possível afirmar que devido a isso o cliente gastou mais.

Outras variaveis podem ter sido consideradas como tipo de produto, quantidade de itens ou perfil do pedido.

Pedido pelo app representam 27,70% do total, com R$ 496.822,22.


4. Qual praça concentra o faturamento?

Resultado baseado o fator de público da tabela bridge_loja_praca.

O maior resultado foi encontrado no Vale do Itajai:

148.000 domicílios com pet;

R$ 633.746,09 de faturamento rateado;

R$ 4,28 de faturamento por domicílio.

A comparação mostrou:

| Praça | Faturamento rateado |
|---|---:|
| Vale do Itajaí | R$ 633.746,09 |
| Grande Florianópolis | R$ 283.546,75 |
| Norte Industrial | R$ 175.431,90 |
| Litoral Sul | R$ 137.051,20 |
| Litoral Norte | R$ 128.872,75 |

O uso do fator é importante para o faturamento de uma loja não ser contado mais de uma vez.

Conferência:

O faturamento rateado das praças foi: R$ 1.792.322,21\

Somando os pedidos sem loja: R$ 986,30\

Temos: R$ 1.793.308,51

Esse valor é igual ao faturamento total da fato.

5. Expansão da rede

a) Itens vendidos por mil habitantes

As lojas com maiores índices encontrados foram:

| Loja | Itens por 1.000 habitantes | Tempo médio de entrega |
|---|---:|---:|
| Rio dos Cedros | 41,87 | 14,24 dias |
| Presidente Getúlio | 34,84 | 14,16 dias |
| Ibirama | 32,07 | 15,39 dias |
| Itapoá | 25,94 | 15,39 dias |
| Santo Amaro da Imperatriz | 23,71 | 15,88 dias |

Rio dos Cedros apresentou o maior indicador.

As lojas com maior venda relativa também possuem tempos de entrega relativamente altos. Isso pode indicar que existe demanda, mas a operação também precisa ser analisada.

b) Faixa das franquias

Considerando a faixa atual cadastrada:

| Faixa | Faturamento |
|---|---:|
| Ouro | R$ 1.011.264,38 |
| Diamante | R$ 382.209,74 |
| Prata | R$ 314.812,03 |
| Bronze | R$ 84.036,06 |

As lojas Ouro apresentam o maior faturamento.

Mas existe uma limitação: essa é a classificação atual sem um histórico.

Portanto, não dá para saber se uma loja que hoje é Ouro também era Ouro quando determinado pedido aconteceu.

c) Dados incompletos

Na fato ficaram:

3 pedidos sem loja;

1.953 pedidos sem entrega;

257 pedidos sem quantidade de itens;

121 pedidos sem valor.

O número de pedidos sem entrega é especialmente relevante, porque representa aproximadamente 48,29% da base.

Conclusão
Depois de realizar o tratamento e as análises, os dados mostram três pontos importantes:

1.  O principal gargalo está entre a emissão da nota e o despacho, principalmente nas lojas de menor porte;

2.  A venda de Ração representa a maior parte da receita da rede Pata Amiga;

3.  Algumas lojas pequenas apresentam alta quantidade  vendas por habitante, mas também possuem tempos de entrega elevados.

Antes de decidir sobre expansão, seria importante analisar concorrência, custos, demanda, logística e potencial de crescimento.

## Validações

No final da construção foram conferidos os principais resultados:

4.044 pedidos na fato;

0 FKs nulas ou órfãs;

3 pedidos na loja -1;

1.953 pedidos sem entrega;

Período: 01/09/2023 a 31/03/2024;

Faturamento total: R$ 1.793.308,51;

Faturamento arredondado: R$ 1.793.309;

Rateio das praças reconciliado com o faturamento total.