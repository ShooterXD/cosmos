# Cosmos
**Uma Rede de Distribuição de Ledgers**

Jae Kwon jae@tendermint.com<br/>
Ethan Buchman ethan@tendermint.com

Para discussões, [entre no nosso Slack](http://forum.tendermint.com:3000/)!

_NOTA: Se você pode ler isso no GitHub, então ainda estamos desenvolvendo este documento ativamente. Por favor, cheque regularmente as atualizações!_

## Índice ###########################################################
  * [Introdução](#introdução)
  * [Tendermint](#tendermint)
    * [Validadores](#validadores)
    * [Consenso](#consenso)
    * [Clientes Light](#clientes-light)
    * [Previnindo Ataques](#previnindo-ataques)
    * [TMSP](#tmsp)
  * [Visão Geral da Cosmos](#visão-geral-da-cosmos)
    * [Tendermint-BFT](#tendermint-bft)
    * [Governança](#governança)
  * [O Hub e as Zonas](#o-hub-e-as-zonas)
    * [O Hub](#the-hub)
    * [As Zonas](#as-zonas)
  * [Comunicação Inter-blockchain (IBC)](#comunicação-inter-blockchain-ibc)
  * [Casos de Uso](#casos-de-uso)
    * [Exchange Distribuída](#exchange-distribuída)
    * [Pegging para outras Criptomoedas](#pegging-para-outras-criptomoedas)
    * [Integrando Ethereum](#integrando-ethereum)
    * [Integração de Multi-Aplicação](#integração-de-multi-aplicação)
    * [Redução de partição de rede](#redução-de-partição-de-rede)
    * [Sistema de Resolução de Nomes Federados](#sistema-de-resolução-de-nomes-federados)
  * [Emissão e Incentivos](#emissão-e-incentivos)
    * [O Token Atom](#o-token-atom)
      * [Levantamento de Fundos](#levantamento-de-fundos)
      * [Investindo](#investindo)
    * [Limitações do Número de Validadores](#limitações-do-número-de-validadores)
    * [Tornando-se um Validador depois do dia da
    Genesis](#tornando-se-um-validador-depois-do-dia-da-genesis)
    * [Penalidades para Validadores](#penalidades-para-validadores)
    * [Taxas de Transação](#taxas-de-transação)
    * [Incentivando Hackers](#incentivando-hackers)
  * [Específicações de governança](#específicações-de-governança)
    * [Parâmetro de Mudança de Proposta](#parâmetro-de-mudança-de-proposta)
    * [Texto da Proposta](#texto-da-proposta)
  * [Roteiro](#roteiro)
  * [Related Work](#related-work)
    * [Consensus Systems](#consensus-systems)
      * [Classic Byzantine Fault Tolerance](#classic-byzantine-fault-tolerance)
      * [BitShares Delegated Stake](#bitshares-delegated-stake)
      * [Stellar](#stellar)
      * [BitcoinNG](#bitcoinng)
      * [Casper](#casper)
    * [Horizontal Scaling](#horizontal-scaling)
      * [Interledger Protocol](#interledger-protocol)
      * [Sidechains](#sidechains)
      * [Ethereum Scalability Efforts](#ethereum-scalability-efforts)
    * [General Scaling](#general-scaling)
      * [Lightning Network](#lightning-network)
      * [Segregated Witness](#segregated-witness)
  * [Appendix](#appendix)
    * [Fork Accountability](#fork-accountability)
    * [Tendermint Consensus](#tendermint-consensus)
    * [Tendermint Light Clients](#tendermint-light-clients)
    * [Preventing Long Range Attacks](#preventing-long-range-attacks)
    * [Overcoming Forks and Censorship
    Attacks](#overcoming-forks-and-censorship-attacks)
    * [TMSP Specification](#tmsp-specification)
    * [IBC Packet Delivery
    Acknowledgement](#ibc-packet-delivery-acknowledgement)
    * [Merkle Tree &amp; Proof
    Specification](#merkle-tree--proof-specification)
    * [Transaction Types](#transaction-types)
      * [IBCBlockCommitTx](#ibcblockcommittx)
      * [IBCPacketTx](#ibcpackettx)
  * [Acknowledgements](#acknowledgements)
  * [Citations](#citations)

## Introdução ################################################################

O sucesso combinado do ecossistema de código aberto, compartilhamento
de arquivos descentralizado e criptomoedas públicas tem inspirado um conhecimento sobre
protocolos descentralizados na Internet que podem ser utilizados para melhorar radicalmente
a infraestrutura. Vimos aplicações de blockchain especializadas como Bitcoin
[\[1\]][1] (uma criptomoeda), Zerocash [\[2\]][2] (uma criptomoeda para privacidade
), and generalized smart contract platforms such as Ethereum [\[3\]][3],
com inúmeras aplicações distribuídas para a Etherium Virtual Machine (EVM), como Augur (uma previsão
de mercado) e TheDAO [\[4\]][4] (um clube de investimento).

Contudo, até à data, estas blockchains sofreram uma série de inconvenientes,
incluindo sua ineficiência energética, desempenho fraco ou limitado e
mecanismos de governança imaturos. Propostas de escala
de processamento de transações da Bitcoin, como Testemunhas Separadas [\[5\]][5] e
BitcoinNG [\[6\]][6], soluções de escalonamento vertical que permanecem
limitadas pela capacidade de uma única máquina física, a fim de
proporcionar uma auditabilidade completa. A Rede Lightning [\[7\]][7] pode ajudar
o Bitcoin no quesito volume de transações, deixando algumas transações completamente
fora da carteira, e é bem adequado para micropagamentos e preservando a privacisadade por pagamentos
Rails, mas pode não ser adequado para necessidades de escala mais abrangente.

Uma solução ideal é a de permitir blockchains paralelos múltiplos para
interoperação, mantendo suas propriedades de segurança. Isto provou
ser difícil, se não impossível, com prova de trabalho. A mineração combinada, por exemplo,
permite que o trabalho feito para proteger uma blockchain mãe seja reutilizado em uma blockchain nova,
mas as transações ainda devem ser validadas, em ordem, por cada nó, e uma
blockchain Merge-mined é vulnerável a ataques se a maioria do poder de
hashing sobre a mãe não é ativamente merge-mined da nova. Uma revisão acadêmica
do [arquiteturas de redes alternativas blockchain
](http://vukolic.com/iNetSec_2015.pdf) é fornecida para
contextualizar, e fornecemos resumos de outras propostas e suas desvantagens em
[Trabalho relatado](#trabalho-relatado).

Nesse relato nós apresentamos a Cosmos, uma novela da arquitetura de rede blockchain que aborda todos
esses problemas. Cosmos é uma rede de muitos blockchains independentes, chamados
Zonas. As zonas são alimentadas pelo Tendermint Coreork [\[8\]][8], que fornece uma
alta performace, consistência, segurança
[PBFT-como](http://tendermint.com/blog/tendermint-vs-pbft/) um mecanismo de consenso
rigoroso, onde [fork-responsável](#fork-responsável) tem-se garantias de deter
comportamentos maliciosos. O algoritmo de consenso BFT do Tendermint Core é
bem adaptado para integrar blockchains públicas de prova de estaca.

A primeira zona na Cosmos é chamada de Cosmos Hub. A Cosmos Hub é uma criptomoeda 
multi-asset de prova de estaca com um simples mecanismo de governança
o qual permite a rede se adaptar e atualizar.  Além disso, a Cosmos Hub pode ser
extendida por conexão com outras zonas.

O hub e as zonas da rede Cosmos comunicam-se uma com a outra através de um
protocolo de comunicação Inter-blockchain (IBC), um tipo de UDP ou TCP virtual para
blockchains. Os tokens podem ser transferidos de uma zona para outra com segurança e
rapidez sem necessidade de liquidez cambial entre as zonas.  Em vez disso, todas
as transferências de tokens inter-zonas passam pelo Hub Cosmos, que mantêm
a quantidade total de tokens detidas por cada zona. O hub isola cada zona da
falha das outras zonas. Porque qualquer um pode conectar uma nova zona no Hub Cosmos,
o que permite futuras compatibilidades com novas blockchains inovadoras.

## Tendermint ##################################################################

Nesta seção, descrevemos o protocolo de consenso da Tendermint e a interface
usada para construir aplicações através dele. Para mais detalhes, consulte o [apêndice](#apêndice).

### Validadores

No algorítimo de tolerância e falhas clássicas Bizantinas (BFT), cada node tem o mesmo
peso.  Na Tendermint, nodes tem uma quantidade positiva de _poder de voto_, e
esses nodes que tem poder de voto positivo são chamados de _validadores_.  Validadores
participam de um protocolo de consenso por transmissão de assinaturas criptográficas,
ou _votos_, para concordar com o próximo bloco.

Os poderes de voto dos validadores são determinados na gênese, ou são alterados
de acordo com a blockchain, dependendo da aplicação. Por exemplo,
em uma aplicação de prova de participação, como o Hub da Cosmos, o poder de voto pode ser
determinado pela quantidade de tokens usados como garantia.

_NOTA: Frações como ⅔ e ⅓ referem-se a frações do total de votos,
nunca o número total de validadores, a menos que todos os validadores tenham
peso._
_NOTE: +⅔ significa "mais do que ⅔", enquanto ⅓+ significa "⅓ ou mais"._

### Consenso

Tendermint é um protocolo de consenso BFT parcialmente sincronizado e derivado do
algoritmo de consenso DLS [\[20\]][20]. Tendermint é notável por sua simplicidade,
desempenho, e [fork-responsável](#fork-responsável).  O protocolo
requer um grupo determinado de validadores, onde cada validador é identificado por
sua chave pública. Validadores chegarão a um consenso em um bloco por vez,
onde um bloco é uma lista de transações. A votação para o consenso sobre um bloco
acontece por rodada. Cada rodada tem uma líder-de-rodada, ou proponente, que propõe um bloco. Os
validadores, em seguida, votam, por etapas, sobre a aceitação do bloco proposto
ou passam para a próxima rodada. O proponente de uma rodada é escolhido
de acordo com uma lista ordenada de validadores, proporcionalmente à seu
poder de voto.

Os detalhes completos do protocolo estão descritos
[aqui](https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm).

A segurança da Tendermint é baseada na tolerância e falhas clássicas Bizantinas ótimas
através de super-maioria (+⅔) e um mecanismo de bloqueio.  Juntas, elas garantem
isso:

* ⅓+ o poder de voto deve ser bizantino devido a violações de segurança, onde mais
  que dois valores são comprometidos.
* se algum conjunto de validadores tiver sucesso em violar a segurança, ou mesmo tentarem
  para isso, eles podem ser identificados pelo protocolo. Isso inclui tanto o voto
para blocos conflitantes quanto a transmissão de votos injustificados.

Apesar de suas fortes garantias, a Tendermint oferece um desempenho excepcional. Dentro
de Benchmarks de 64 nós distribuídos em 7 datacenters em 5 continentes, em
nuvens de commodities, o consenso da Tendermint pode processar milhares de
transações por segundo, com tempo de resposta entre um a dois
segundos. Notavelmente, o desempenho muito além de mil transações por segundo
é mantido mesmo em condições adversas, com validadores falhando ou
combinando votos maliciosamente. Veja a figura abaixo para mais detalhes.

![Figura do desempenho da Tendermint]
(https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/images/tendermint_throughput_blocksize.png)

### Clientes Light

O principal benefício do algoritmo de consenso da Tendermint é um cliente leve e simplificado
de segurança, tornando-o um candidato ideal para o uso de dispositivos móveis e casos de uso na
internet. Enquanto um cliente leve do Bitcoin deve sincronizar blockchains e encontrar
o que tem mais prova de trabalho, os clientes light da Tendermint precisa apenas
das alterações feitas pelo conjunto dos validadores, em seguida, verifica-se o +⅔ PreCommits
no último bloco para determinar o estado atual.

Provas claras e sucintas do cliente também permite [comunicação-inter-
blockchain](#comunicação-inter-blockchain-ibc).

### Previnindo ataques

A Tendermint dispõe de medidas de proteção para evitar
ataques, como [gastos duplos em longa-distância-sem-estaca double
spends](#previnindo-ataques-de-longa-distância) e
[censura](#superando-forks-e-censurando-ataques). Esses são discutidos
completamente no [apêndice](#apêndice).

### TMSP

O algoritmo de consenso Tendermint é implementado através de um programa chamado Tendermint
Core. O Tendermint Core é um "mecanismo de consenso" independente de aplicações que
transformam qualquer aplicação blackbox em uma réplica distribuída na
Blockchain. Tendermint Core conecta-se ao blockchain
através de aplicações do Tendermint Socket Protocol (TMSP) [\[17\]][17]. Assim, o TMSP
permite que as aplicações da blockchain sejam programadas em qualquer idioma, não apenas
a linguagem de programação que o mecanismo de consenso é escrito, além disso,
o TMSP torna possível a troca fácil da camada de consenso de qualquer
tipo de blockchain.

Nós fizemos uma analogia com a bem conhecida criptogradia do Bitcoin. Bitcoin é uma
blockchain de criptomoedas onde cada nó mantém uma Unspent totalmente auditada
e banco de dados de saída de transação (UTXO). Se alguém quisesse criar um Bitcoin-like
TMS, a Tendermint Core seria responsável por

* Compartilhar blocos e transações entre os nós
* Estabelecer uma ordem de transações canônica/imutável (a blockchain)

Entretanto, o aplicativo TMSP seria responsável por

* Manter o banco de dados UTXO
* Validar a criptografia das assinaturas das transações
* Previnir transações vindas de gastos de fundos não exisentes
* Permitir aos clientes a consulta do banco de dados UTXO

Tendermint é capaz de decompor o design da blockchain, oferecendo um simples
API entre o processo da aplicação e o processo do consenso.

## Visão Geral da Cosmos #############################################################

Cosmos é uma rede de blockchains paralelos e independentes que são alimentadas pelo
clássico algorítimo de consenso BFT como a Tendermint
[1](http://github.com/tendermint/tendermint).

A primeira blockchain dessa rede será a Cosmos Hub.  A Cosmos Hub
conecta as outras blockchains (ou _zonas_) através do protocolo de comunicação-inter-
blockchain.  A Cosmos Hub rastreia vários tipos de tokens e mantém
registo do número total de tokens em cada zona ligada. Os tokens podem ser
transferidos de uma zona para outra de forma segura e rápida, sem necessidade de
uma troca líquida entre zonas, porque todas as transferências de moedas ocorre
através da Cosmos Hub.

Essa arquitetura resolve muitos dos problemas enfrentados atualmente pelas blockchains,
tais como interoperabilidade de aplicativos, escalabilidade e capacidade de atualização contínua.
Por exemplo, as zonas baseadas do Bitcoin, Go-Ethereum, CryptoNote, ZCash, ou qualquer
sistema blockchain pode ser ligado ao Cosmos Hub.  Essas zonas permite a Cosmos
o escalonamento infinito para atender a demanda global de transações.  As Zonas também são um grande
apoio para a exchange distribuída, que também serão apoiadas.

Cosmos não é apenas uma única ledger distribuídos, o Cosmos Hub não é um
jardim cercado ou o centro do universo. Estamos elaborando um protocolo para
uma rede aberta de legers distribuídos que pode servir como um novo
futuros para sistemas financeiros, baseados em princípios de criptografia, economia
teoria de consenso, transparência e responsabilidade.

### Tendermint-BFT 

O Cosmos Hub é a primeira blockchain pública na rede Cosmos, alimentada pelo
algoritimo de consenso BFT Tendermint. A Tendermint é um projeto de fonte aberta que
nasceu em 2014 para abordar a velocidade, a escalabilidade e as questões
do algoritimo de consenso da prova-de-trabalho do Bitcoin.  Usando e melhorando
algoritmos BFT comprovados e desenvolvidos no MIT em 1988 [\[20\]][20], o time Tendermint foi o primeiro a
que demonstrou conceitualmente uma prova de estaca das criptomoedas que aborda o
problema de "sem-estaca" sofrido pelas criptomoedas da primeira geração
tais como NXT e BitShares.

Hoje, praticamente todas carteiras móveis de Bitcoin usam servidores confiáveis, que fornece
a elas transações com verificação.  Isso porque a prova-de-trabalho exige
muitas confirmações antes que uma transação possa ser considerada
irreversivel e completa. Os ataques de gasto-duplo já foram demonstrados em
serviços como a CoinBase.

Ao contrário de outros sistemas de consenso blockchain, a Tendermint oferece
comprovação segura de pagamento para o cliente móvel. Uma vez que a Mint é
projetada para nunca passar por um fork, carteiras móveis podem receber confirmações de transações
instantâneas, o que torna os pagamentos confiáveis e práticos através de
smartphones. Isto tem implicações significativas para as aplicações da Internet.

Validadores na Cosmos tem uma função similar aos mineiros do Bitcoin, mas usam
assinaturas criptografadas para votar. Validadores são máquinas seguras e dedicadas
que são responsáveis por validar os blocos. Os não validadores podem delegar através de seus tokens estacados
(chamados "atoms") a qualquer validador para ganhar uma parcela das taxas da blockchain
e recompensas de atoms, mas eles correm o risco de serem punidos (cortados) se o
o validador de delegados for invadido ou violar o protocolo. A segurança comprovada
garantida pelo consenso BFT da Tendermint, e o depósito de garantia das
partes interessadas - validadores e delegados - fornecem dados prováveis,
segurança para os nós e clientes light.

### Governança #################################################################

Ledgers de distribuição pública devem ser constituídos de um sistema de governança.
O Bitcoin confia na Fundação Bitcoin e na mineração para
coordenar upgrades, mas este é um processo lento. Ethereum foi dividido em ETH e
ETC depois de hard-fork para se recuperar do hack TheDAO, em grande parte porque não havia
contrato sócial prévio, nem um mecanismo para tomar tais decisões.

Os validadores e os delegados do Cosmos Hub podem votar propostas que
alteraram automaticamente os parâmetros predefinidos do sistema (tal como o gás limite do
bloco), coordenar upgrades, bem como votar em emendas para a
constituição que governa as políticas do Cosmos Hub. A Constituição
permite a coesão entre as partes interessadas em questões como o roubo
e bugs (como o incidente TheDAO), permitindo uma resolução mais rápida e mais limpa.

Cada zona pode ter sua própria constituição e mecanismo de governança.
Por exemplo, o Cosmos Hub pode ter uma constituição que reforça a imutabilidade
no Hub (sem roll-backs, a não ser por bugs em implementações dos nós do Cosmos Hub),
enquanto cada zona pode ter sua própria política sobre os roll-backs.

Ao disponibilizar a interoperabilidade em diferentes políticas das zonas, a rede Cosmos
dá aos usuários total liberdade e potenciais permissões para
experimentos.

## O Hub e as Zonas ###########################################################

Aqui nós descrevemos o modelo do roteiro de descentralização e ecalabilidade.  Cosmos é uma
rede de muitas blockchains alimentadas pela Tendermint.  Enquanto existirem propostas visando
criar um"blockchain solitário" com ordens de transações cheias, a Cosmos
permite que muitas blockchains rodem junto de outra enquanto mantêm a
interoperabilidade.

Basicamente, o Cosmos Hub gerencia várias blockchains independentes chamadas "zonas"
(as vezes chamadas de "shards", em referência a técnica de escalonamento de
bando de dados conhecida como "sharding").  Uma constante transmissão de blocos recentes das
zonas atuando no Hub permite ao Hub manter o estado de cada zona atualizado.
Sendo assim, cada zona mantêm ativa o estado do Hub (mas as zonas não se mantêm ativas
com qualquer outro exceto o Hub).  Pacotes de informação são
então comunicados de uma zona para outra atráves de Merkle-proofs como evidências,
essas informações são enviadas e recebidas. Esse mecanismo é chamado de
comunicação inter-blockchain, ou IBC para encurtar.

![Figura de reconhecimento
do hub e das zonas](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/images/hub_and_zones.png)

Qualquer uma das zonas podem ser hubs para formar gráficos acíclicos, mas
mas para deixar claro, nós vamos apenas descrever uma simples configuração para
um único hub, e várias zonas que não são hubs.

### O Hub

O Cosmos Hub é uma blockchain que hospeda um ledger de distribuíção de multi-asset,
onde os tokens podem ser mantidos por usuários individuais ou pelas próprias zonas.  Esses
tokens podem ser movidos de uma zona para outra em um pacote IBC especial chamado
"coin packet".  O hub é responsavel por preservar a manutenção global de
toda a quantia de cada token nas zonas. As transações de moedas no pacote IBC
precisam ser feitas pelo remetente, hub, e blockchain recebedor.

Desde a atuação do Cosmos Hub  como ledger principal para todos o
sistema, a segurança do Hub é de suma importância.  Enquanto cada
zona pode ser uma blockchain Tendermint que é segurada por 4((ou talvez
menos caso o consenso BFT não seja necessário), o Hub precisa ser segurado por uma descentralização
globalizada feita pelos validadores que podem evitar os mais severos tipos de
ataques, como uma partição de rede continental ou um estado-nação fazendo
ataques.

### As Zonas

Uma zona Cosmos é uma blockchain independente das trocas de mensagens IBC com o
Hub.  Na perspectiva do Hub, uma zona é uma conta multi-asset dynamic-membership
multi-signature que pode enviar e receber tokens usando pacotes IBC. Como
uma conta de criptomoeda, uma zona não pode transferir mais tokens do que ela possui, mas
pode receber tokens de outras que os tem. Uma zona pode ser usada como uma
"fonte" de um ou mais tipos de tokens, garantindo o poder de aumentar o estoque desse
token.

Os atoms do Cosmos Hub podem ser estacados por validadores de uma zona conectada ao
Hub.  Enquanto os ataques de gasto-duplo nesses zonas podem resultar em um core dos
atoms com o fork-responsável da Tendermint, uma zona onde +⅔ do poder de voto
são Bizantinos podem deixar o estado inválido.  O Cosmos Hub não verifica ou
executa transações ocorridas em outras zonas, então essa é uma responsabilidade dos
usuários para enviar os tokes ara zonas que eles confiem.  Futuramente, o sistema de
governança do Cosmos Hub irá implementar propostas para o Hub e para as falhas
das zonas.  Por exemplo, um token de saída transferido para algumas (ou todas) zonas podem ser
segurados em caso de uma emergência de quebra de circuito das zonas(uma parada temporária
nas transferências dos tokens) quando um ataque é detectado.

## Comunicação Inter-blockchain (IBC) ########################################

Agora nós olhamos para como o Hub e as zonas vão se comunicar.  Por exemplo, se
aqui são três blockchains, "Zona1", "Zona2", and "Hub", e nós queremos que a
"Zona1" produza um pacote destinado para a "Zona2" indo através do "Hub". Para mover um
pacote de uma blockchain para outra, uma prova é feita na
cadeia recebedora. A prova atesta esse envio publicado na cadeia de destino por uma alegação
de pacote. Para a cadeia recebedora checar essa prova, isso é possível
por um block principal de envio.  Esse mecanismo é similar ao usado por
cadeias paralelas, que requerem duas cadeias interagindo uma com a outra via
transmissões bidirecionais por dados de prova-de-existência (transações).

O protocolo IBC pode naturalmente ser definido usando dois tipos de transações: uma
transação `IBCBlockCommitTx`, a qual permite uma blockchain provar para qualquer
espectador o mais recente hash-de-bloco, e uma transação `IBCPacketTx`, a qual
permite uma blockchain provar para qualquer espectador que o pacote recebido foi realmente
publicado pelo remetente, via Merkle-proof para um hash-de-bloco
recente.

By splitting the IBC mechanics into two separate transactions, we allow the 
native fee market-mechanism of the receiving chain to determine which packets 
get committed (i.e. acknowledged), while allowing for complete freedom on the 
sending chain as to how many outbound packets are allowed.

![Figure of Zone1, Zone2, and Hub IBC without
acknowledgement](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/msc/ibc_without_ack.png)

<CAPTION on a figure> In the example above, in order to update the block-hash of
"Zone1" on "Hub" (or of "Hub" on "Zone2"), an `IBCBlockCommitTx`
transaction must be posted on "Hub" with the block-hash of "Zone1" (or on
"Zone2" with the block-hash of "Hub").

_See [IBCBlockCommitTx](#ibcblockcommittx) and [IBCPacketTx](#ibcpacketcommit)
for for more information on the two IBC transaction types._

## Casos de Uso ###################################################################

### Exchange Distribuídas

In the same way that Bitcoin is more secure by being a distributed,
mass-replicated ledger, we can make exchanges less vulnerable to external and
internal hacks by running it on the blockchain.  We call this a distributed
exchange.

What the cryptocurrency community calls a decentralized exchange today are
based on something called "atomic cross-chain" (AXC) transactions.  With an AXC
transaction, two users on two different chains can make two transfer
transactions that are committed together on both ledgers, or none at all (i.e.
atomically).  For example, two users can trade bitcoins for ether (or any two
tokens on two different ledgers) using AXC transactions, even though Bitcoin
and Ethereum are not connected to each other.  The benefit of running an
exchange on AXC transactions is that neither users need to trust each other or
the trade-matching service.  The downside is that both parties need to be
online for the trade to occur.

Another type of decentralized exchange is a mass-replicated distributed
exchange that runs on its own blockchain.  Users on this kind of exchange can
submit a limit order and turn their computer off, and the trade can execute
without the user being online.  The blockchain matches and completes the trade
on behalf of the trader.

A centralized exchange can create a deep orderbook of limit orders and thereby
attract more traders.  Liquidity begets more liquidity in the exchange world,
and so there is a strong network effect (or at least a winner-take-most effect)
in the exchange business.  The current leader for cryptocurrency exchanges
today is Poloniex with a 24-hour volume of $20M, and in second place is
Bitfinex with a 24-hour volume of $5M.  Given such strong network effects, it
is unlikely for AXC-based decentralized exchanges to win volume over the
centralized exchanges.  For a decentralized exchange to compete with a
centralized exchange, it would need to support deep orderbooks with limit
orders.  Only a distributed exchange on a blockchain can provide that.

Tendermint provides additional benefits of faster transaction commits.  By
prioritizing fast finality without sacrificing consistency, zones in Cosmos can
finalize transactions fast -- for both exchange order transactions as well as
IBC token transfers to and from other zones.

Given the state of cryptocurrency exchanges today, a great application for
Cosmos is the distributed exchange (aka the Cosmos DEX).  The transaction
throughput capacity as well as commit latency can be comparable to those of
centralized exchanges.  Traders can submit limit orders that can be executed
without both parties having to be online.  And with Tendermint, the Cosmos hub,
and IBC, traders can move funds in and out of the exchange to and from other
zones with speed.

### Pegging para Outras Criptomoedas

A privileged zone can act as the source of a pegged token of another
cryptocurrency. A peg is similar to the relationship between a
Cosmos hub and zone; both must keep up with the latest blocks of the
other in order to verify proofs that tokens have moved from one to the other.  A
peg-zone on the Cosmos network keeps up with the Hub as well as the
other cryptocurrency.  The indirection through the peg-zone allows the logic of
the Hub to remain simple and agnostic to other blockchain consensus strategies
such as Bitcoin's proof-of-work mining.

For instance, a Cosmos zone with a specific validator set, possibly the same as
that of the Hub, could act as an ether-peg, where the TMSP-application on
the zone (the "peg-zone") has mechanisms to exchange IBC messages with a
peg-contract on the external Ethereum blockchain (the "origin").  This contract
would allow ether holders to send ether to the peg-zone by sending it to the
peg-contract on Ethereum.  Once ether is received by the peg-contract, the ether
cannot be withdrawn unless an appropriate IBC packet is received by the
peg-contract from the peg-zone. When a peg-zone receives an IBC packet proving
that ether was received in the peg-contract for a particular Ethereum account, a
corresponding account is created on the peg-zone with that balance.  Ether on
the peg-zone ("pegged-ether") can then be transferred to and from the Hub,
and later be destroyed with a transaction that sends it to a particular
withdrawal address on Ethereum. An IBC packet proving that the transaction
occured on the peg-zone can be posted to the Ethereum peg-contract to allow the
ether to be withdrawn.

Of course, the risk of such a pegging contract is a rogue validator set.  ⅓+
Byzantine voting power could cause a fork, withdrawing ether from the
peg-contract on Ethereum while keeping the pegged-ether on the peg-zone. Worse,
+⅔ Byzantine voting power can steal ether outright from those who sent it to the
peg-contract by deviating from the original pegging logic of the peg-zone.

It is possible to address these issues by designing the peg to be totally
accountable.  For example, all IBC packets, from the hub and 
the origin, might require acknowledgement by the peg-zone in such a way that all
state transitions of the peg-zone can be efficiently challenged and verified by
either the hub or the origin's peg-contract.  The Hub and the origin should
allow the peg-zone validators to post collateral, and token transfers out of the
peg-contract should be delayed (and collateral unbonding period sufficiently
long) to allow for any challenges to be made by independent auditors.  We leave
the design of the specification and implementation of this system open as a
future Cosmos improvement proposal, to be passed by the Cosmos Hub's governance
system.

While the socio-political atmosphere is not quite evolved enough yet, we can
extend the mechanism to allow for zones which peg to the fiat currency of a
nation state by forming a validator set out of some combination of institutions
responsible for the nation's currency, most particularly, its banks. Of course,
extra precautions must be taken to only accept currencies backed by strong legal
systems which can enforce auditability of the banks' activities by a sufficiently
large group of trusted notaries and institutions.

A result of this integration could be, for instance, allowing anyone with
an account at a bank on the zone to move dollars from their bank account
to other accounts on the zone, or to the hub, or to another zone entirely.  
In this regard, the Cosmos Hub can act as a seamless conduit between fiat 
currencies and cryptocurrencies, removing the barriers that have until now limited 
their interoperability to the realm of exchanges.

### Ethereum Scaling

Solving the scaling problem is an open issue for Ethereum.  Currently,
Ethereum nodes process every single transaction and also store all the states.
[link](https://docs.google.com/presentation/d/1CjD0W4l4-CwHKUvfF5Vlps76fKLEC6pIwu1a_kC_YRQ/mobilepresent?slide=id.gd284b9333_0_28).

Since Tendermint can commit blocks much faster than Ethereum's proof-of-work,
EVM zones powered by Tendermint consensus and operating on pegged-ether can
provide higher performance to Ethereum blockchains.  Additionally, though the
Cosmos Hub and IBC packet mechanics does not allow for arbitrary contract logic
execution per se, it can be used to coordinate token movements between Ethereum
contracts running on different zones, providing a foundation for token-centric
Ethereum scaling via sharding.

### Integração de Multi-Aplicação

Cosmos zones run arbitrary application logic, which is defined at the beginning of the
zone's life and can potentially be updated over time by governance. Such flexibility
allows Cosmos zones to act as pegs to other cryptocurrencies such as Ethereum or
Bitcoin, and it also permits derivatives of those blockchains, utilizing the
same codebase but with a different validator set and initial distribution. This
allows many existing cryptocurrency frameworks, such as those of Ethereum,
Zerocash, Bitcoin, CryptoNote and so on, to be used with Tendermint Core, 
which is a higher performance consensus engine, on a common network, opening tremendous
opportunity for interoperability across platforms.  Furthermore, as a
multi-asset blockchain, a single transaction may contain multiple inputs and
outputs, where each input can be any token type, enabling Cosmos to serve
directly as a platform for decentralized exchange, though orders are assumed to
be matched via other platforms. Alternatively, a zone can serve as a distributed
fault-tolerant exchange (with orderbooks), which can be a strict improvement
over existing centralized cryptocurrency exchanges which tend to get hacked over
time. 

Zones can also serve as blockchain-backed versions of enterprise and government
systems, where pieces of a particular service that are traditionally run by an
organization or group of organizations are instead run as a TMSP application on
a certain zone, which allows it to inherit the security and interoperability of the
public Cosmos network without sacrificing control over the underlying service.
Thus, Cosmos may offer the best of both worlds for organizations looking to
utilize blockchain technology but who are wary of relinquishing control completely
to a distributed third party.

### Redução de partição de rede

Some claim that a major problem with consistency-favouring consensus algorithms
like Tendermint is that any network partition which causes there to be no single
partition with +⅔ voting power (e.g. ⅓+ going offline) will halt consensus
altogether. The Cosmos architecture can help mitigate this problem by using a global
hub with regional autonomous zones, where voting power for each zone are
distributed based on a common geographic region.  For instance, a common
paradigm may be for individual cities, or regions, to operate their own zones
while sharing a common hub (e.g. the Cosmos Hub), enabling municipal activity to
persist in the event that the hub halts due to a temporary network partition.
Note that this allows real geological, political, and network-topological
features to be considered in designing robust federated fault-tolerant systems.

### Sistema de Resolução de Nomes Federados

NameCoin was one of the first blockchains to attempt to solve the
name-resolution problem by adapting the Bitcoin blockchain.  Unfortunately there
have been several issues with this approach.

With Namecoin, we can verify that, for example, <em>@satoshi</em> was registered with a
particular public key at some point in the past, but we wouldn’t know whether
the public key had since been updated recently unless we download all the blocks
since the last update of that name.  This is due to the limitation of Bitcoin's
UTXO transaction Merkle-ization model, where only the transactions (but not
mutable application state) are Merkle-ized into the block-hash. This lets us
prove existence, but not the non-existence of later updates to a name.  Thus, we
can't know for certain the most recent value of a name without trusting a full
node, or incurring significant costs in bandwidth by downloading the whole
blockchain.

Even if a Merkle-ized search tree were implemented in NameCoin, its dependency
on proof-of-work makes light client verification problematic. Light clients must
download a complete copy of the headers for all blocks in the entire blockchain
(or at least all the headers since the last update to a name).  This means that
the bandwidth requirements scale linearly with the amount of time [\[21\]][21].
In addition, name-changes on a proof-of-work blockchain requires waiting for
additional proof-of-work confirmation blocks, which can take up to an hour on
Bitcoin.

With Tendermint, all we need is the most recent block-hash signed by a quorum of
validators (by voting power), and a Merkle proof to the current value associated
with the name.  This makes it possible to have a succinct, quick, and secure
light-client verification of name values.

In Cosmos, we can take this concept and extend it further. Each
name-registration zone in Cosmos can have an associated top-level-domain
(TLD) name such as ".com" or ".org", and each name-registration zone can have
its own governance and registration rules.

## Emissão e Incentivos #####################################################

### O Token Atom

While the Cosmos Hub is a multi-asset distributed ledger, there is a special
native token called the _atom_.  Atoms are the only staking token of the Cosmos
Hub.  Atoms are a license for the holder to vote, validate, or delegate to other
validators.  Like Ethereum's ether, atoms can also be used to pay for
transaction fees to mitigate spam.  Additional inflationary atoms and block
transaction fees are rewarded to validators and delegators who delegate to
validators.

The `BurnAtomTx` transaction can be used to recover any proportionate amount of
tokens from the reserve pool.

#### Levantamento de Fundos

The initial distribution of atom tokens and validators on Genesis will go to the
donors of the Cosmos Fundraiser (75%), lead donors (5%), Cosmos Network
Foundation (10%), and ALL IN BITS, Inc (10%).  From genesis onward, 1/3 of the
total amount of atoms will be rewarded to bonded validators and delegators
every year.

See the [Cosmos Plan](https://github.com/cosmos/cosmos/blob/master/PLAN.md)
for additional details.

#### Investindo

To prevent the fundraiser from attracting short-term speculators only interested
in pump-and-dump schemes, the genesis atoms will not be transferrable until
they have vested.  Each account will vest atoms over a period of 2 years at a
constant rate every hour, determined by the total number of genesis atoms / (2 *
365 * 24) hours.  Atoms earned by the inflationary block reward are pre-vested,
and can be transferred immediately, so that bonded validators and delegators can earn
more than 1/2 of their genesis atoms after the first year.

### Limitações do Número de Validadores

Unlike Bitcoin or other proof-of-work blockchains, a Tendermint blockchain gets
slower with more validators due to the increased communication complexity.
Fortunately, we can support enough validators to make for a robust globally
distributed blockchain with very fast transaction confirmation times, and, as
bandwidth, storage, and parallel compute capacity increases, we will be able to
support more validators in the future.

On genesis day, the maximum number of validators will be set to 100, and this
number will increase at a rate of 13% for 10 years, and settle at 300
validators.

```
Year 0: 100
Year 1: 113
Year 2: 127
Year 3: 144
Year 4: 163
Year 5: 184
Year 6: 208
Year 7: 235
Year 8: 265
Year 9: 300
Year 10: 300
...
```

### Tornando-se um Validador depois do dia da Genesis

Atom holders who are not already can become validators by signing and
submitting a `BondTx` transaction.  The amount of atoms provided as collateral
must be nonzero.  Anyone can become a validator at any time, except when the
size of the current validator set is greater than the maximum number of
validators allowed.  In that case, the transaction is only valid if the amount
of atoms is greater than the amount of effective atoms held by the smallest
validator, where effective atoms include delegated atoms.  When a new validator
replaces an existing validator in such a way, the existing validator becomes
inactive and all the atoms and delegated atoms enter the unbonding state.

### Penalidades para Validadores

There must be some penalty imposed on the validators for any intentional
or unintentional deviation from the sanctioned protocol. Some evidence is
immediately admissible, such as a double-sign at the same height and round, or a
violation of "prevote-the-lock" (a rule of the Tendermint consensus protocol).
Such evidence will result in the validator losing its good standing and its
bonded atoms as well its proportionate share of tokens in the reserve pool --
collectively called its "stake" -- will get slashed.

Sometimes, validators will not be available, either due to regional network
disruptions, power failure, or other reasons.  If, at any point in the past
`ValidatorTimeoutWindow` blocks, a validator's commit vote is not included in
the blockchain more than `ValidatorTimeoutMaxAbsent` times, that validator will
become inactive, and lose `ValidatorTimeoutPenalty` (DEFAULT 1%) of its stake.

Some "malicious" behavior does not produce obviously discernable evidence on the
blockchain. In these cases, the validators can coordinate out of band to force
the timeout of these malicious validators, if there is a supermajority
consensus.

In situations where the Cosmos Hub halts due to a ⅓+ coalition of voting power
going offline, or in situations where a ⅓+ coalition of voting power censor
evidence of malicious behavior from entering the blockchain, the hub must
recover with a hard-fork reorg-proposal.  (Link to "Forks and Censorship
Attacks").

### Taxas de Transação

Cosmos Hub validators can accept any token type or combination of types as fees
for processing a transaction.  Each validator can subjectively set whatever
exchange rate it wants, and choose whatever transactions it wants, as long as
the `BlockGasLimit` is not exceeded.  The collected fees, minus any taxes
specified below, are redistributed to the bonded stakeholders in proportion to
their bonded atoms, every `ValidatorPayoutPeriod` (DEFAULT 1 hour).

Of the collected transaction fees, `ReserveTax` (DEFAULT 2%) will go toward the
reserve pool to increase the reserve pool and increase the security and value of
the Cosmos network.  Also, a `CommonsTax` (DEFAULT 3%) will go toward the
funding of common goods.  These funds will go to the `CustodianAddress` to be
distributed in accordance with the decisions made by the governance system.

Atom holders who delegate their voting power to other validators pay a
commission to the delegated validator.  The commission can be set by each
validator.

### Incentivando Hackers

The security of the Cosmos Hub is a function of the security of the underlying
validators and the choice of delegation by delegators.  In order to encourage
the discovery and early reporting of found vulnerabilities, the Cosmos Hub
encourages hackers to publish successful exploits via a `ReportHackTx`
transaction that says, "This validator got hacked.  Please send
bounty to this address".  Upon such an exploit, the validator and delegators
will become inactive, `HackPunishmentRatio` (default 5%) of everyone's atoms
will get slashed, and `HackRewardRatio` (default 5%) of everyone's atoms will
get rewarded to the hacker's bounty address.  The validator must recover the
remaining atoms by using their backup key.

In order to prevent this feature from being abused to transfer unvested atoms,
the portion of vested vs unvested atoms of validators and delegators before and
after the `ReportHackTx` will remain the same, and the hacker bounty will
include some unvested atoms, if any.

### Específicação de Governança ###################################################

The Cosmos Hub is operated by a distributed organization that requires a well-defined
governance mechanism in order to coordinate various changes to the blockchain, 
such as the variable parameters of the system, as well as software upgrades and 
constitutional amendments.

All validators are responsible for voting on all proposals.  Failing to vote on
a proposal in a timely manner will result in the validator being deactivated 
automatically for a period of time called the `AbsenteeismPenaltyPeriod`
(DEFAULT 1 week).

Delegators automatically inherit the vote of the delegated validator.  This vote
may be overridden manually.  Unbonded atoms get no vote.

Each proposal requires a deposit of `MinimumProposalDeposit` tokens, which may
be a combination of one or more tokens including atoms.  For each proposal, the
voters may vote to take the deposit. If more than half of the voters choose to
take the deposit (e.g. because the proposal was spam), the deposit goes to the
reserve pool, except any atoms which are burned.

Para cada proposta, os eleitores podem votar nas seguintes opições:

* Sim
* Com Certeza
* Não
* Nunca
* Abstenção

A strict majority of Yea or YeaWithForce votes (or Nay or NayWithForce votes) is
required for the proposal to be decided as accepted (or decided as failed), but
1/3+ can veto the majority decision by voting "with force".  When a strict
majority is vetoed, everyone gets punished by losing `VetoPenaltyFeeBlocks`
(DEFAULT 1 day's worth of blocks) worth of fees (except taxes which will not be
affected), and the party that vetoed the majority decision will be additionally
punished by losing `VetoPenaltyAtoms` (DEFAULT 0.1%) of its atoms.

### Parâmetro de Mudança de Proposta

Any of the parameters defined here can be changed with the acceptance of a
`ParameterChangeProposal`.

### Texto da Proposta

All other proposals, such as a proposal to upgrade the protocol, will be
coordinated via the generic `TextProposal`.

## Roteiro #####################################################################

See [the Plan](https://github.com/cosmos/cosmos/blob/master/PLAN.md).

## Related Work ################################################################

There have been many innovations in blockchain consensus and scalability in the
past couple of years.  This section provides a brief survey of a select number
of important ones.

### Consensus Systems

#### Classic Byzantine Fault Tolerance

Consensus in the presence of malicious participants is a problem dating back to
the early 1980s, when Leslie Lamport coined the phrase "Byzantine fault" to refer
to arbitrary process behavior that deviates from the intended behavior, in
contrast to a "crash fault", wherein a process simply crashes. Early solutions
were discovered for synchronous networks where there is an upper bound on
message latency, though pratical use was limited to highly controlled
environments such as airplane controllers and datacenters synchronized via
atomic clocks.  It was not until the late 90s that Practical Byzantine Fault
Tolerance (PBFT) [\[11\]][11] was introduced as an efficient partially
synchronous consensus algorithm able to tolerate up to ⅓ of processes behaving
arbitrarily.  PBFT became the standard algorithm, spawning many variations,
including most recently one created by IBM as part of their contribution to Hyperledger.

The main benefit of Tendermint consensus over PBFT is that Tendermint has an
improved and simplified underlying structure, some of which is a result of
embracing the blockchain paradigm.  Tendermint blocks must commit in order,
which obviates the complexity and communication overhead associated with PBFT's
view-changes.  In Cosmos and many cryptocurrencies, there is no need to allow
for block <em>N+i</em> where <em>i >= 1</em> to commit, when block <em>N</em>
itself hasn't yet committed. If bandwidth is the reason why block <em>N</em>
hasn't committed in a Cosmos zone, then it doesn't help to use bandwidth sharing
votes for blocks <em>N+i</em>. If a network partition or offline nodes is the
reason why block <em>N</em> hasn't committed, then <em>N+i</em> won't commit
anyway.

In addition, the batching of transactions into blocks allows for regular
Merkle-hashing of the application state, rather than periodic digests as with
PBFT's checkpointing scheme.  This allows for faster provable transaction
commits for light-clients and faster inter-blockchain communication.

Tendermint Core also includes many optimizations and features that go above and
beyond what is specified in PBFT.  For example, the blocks proposed by
validators are split into parts, Merkle-ized, and gossipped in such a way that
improves broadcasting performance (see LibSwift [\[19\]][19] for inspiration).
Also, Tendermint Core doesn't make any assumption about point-to-point
connectivity, and functions for as long as the P2P network is weakly connected.

#### BitShares delegated stake

While not the first to deploy proof-of-stake (PoS), BitShares [\[12\]][12]
contributed considerably to research and adoption of PoS blockchains,
particularly those known as "delegated" PoS.  In BitShares, stake holders elect
"witnesses", responsible for ordering and committing transactions, and
"delegates", responsible for coordinating software updates and parameter
changes.  Though BitShares achieves high performance (100k tx/s, 1s latency) in
ideal conditions, it is subject to double spend attacks by malicious witnesses
which fork the blockchain without suffering an explicit economic punishment --
it suffers from the "nothing-at-stake" problem. BitShares attempts to mitigate
the problem by allowing transactions to refer to recent block-hashes.
Additionally, stakeholders can remove or replace misbehaving witnesses on a
daily basis, though this does nothing to explicitly punish successful 
double-spend attacks.

#### Stellar

Building on an approach pioneered by Ripple, Stellar [\[13\]][13] refined a
model of Federated Byzantine Agreement wherein the processes participating in
consensus do not constitute a fixed and globally known set.  Rather, each
process node curates one or more "quorum slices", each constituting a set of
trusted processes. A "quorum" in Stellar is defined to be a set of nodes that
contain at least one quorum slice for each node in the set, such that agreement 
can be reached.

The security of the Stellar mechanism relies on the assumption that the
intersection of *any* two quorums is non-empty, while the availability of a node
requires at least one of its quorum slices to consist entirely of correct nodes,
creating a trade-off between using large or small quorum-slices that may be
difficult to balance without imposing significant assumptions about trust.
Ultimately, nodes must somehow choose adequate quorum slices for there to be
sufficient fault-tolerance (or any "intact nodes" at all, of which much of the
results of the paper depend on), and the only provided strategy for ensuring
such a configuration is heirarchical and similar to the Border Gateway Protocol
(BGP), used by top-tier ISPs on the internet to establish global routing tables,
and by that used by browsers to manage TLS certificates; both notorious for
their insecurity.

The criticism in the Stellar paper of the Tendermint-based proof-of-stake
systems is mitigated by the token strategy described here, wherein a new type of
token called the _atom_ is issued that represent claims to future portions of
fees and rewards. The advantage of Tendermint-based proof-of-stake, then, is its
relative simplicity, while still providing sufficient and provable security
guarantees.

#### BitcoinNG

BitcoinNG is a proposed improvement to Bitcoin that would allow for forms of
vertical scalability, such as increasing the block size, without the negative
economic consequences typically associated with such a change, such as the
disproportionately large impact on small miners.  This improvement is achieved
by separating leader election from transaction broadcast: leaders are first
elected by proof-of-work in "micro-blocks", and then able to broadcast
transactions to be committed until a new micro-block is found. This reduces the
bandwidth requirements necessary to win the PoW race, allowing small miners to
more fairly compete, and allowing transactions to be committed more regularly by
the last miner to find a micro-block.

#### Casper

Casper [\[16\]][16] is a proposed proof-of-stake consensus algorithm for
Ethereum.  Its prime mode of operation is "consensus-by-bet".  By letting 
validators iteratively bet on which block they believe will become committed 
into the blockchain based on the other bets that they have seen so far,
finality can be achieved eventually.
[link](https://blog.ethereum.org/2015/12/28/understanding-serenity-part-2-casper/).
This is an active area of research by the Casper team.  The challenge is in
constructing a betting mechanism that can be proven to be an evolutionarily
stable strategy.  The main benefit of Casper as compared to Tendermint may be in
offering "availability over consistency" -- consensus does not require a +⅔
quorum of voting power -- perhaps at the cost of commit speed or
implementation complexity.

### Horizontal Scaling

#### Interledger Protocol

The Interledger Protocol [\[14\]][14] is not strictly a scalability solution. It
provides an ad hoc interoperation between different ledger systems through a
loosely coupled bilateral relationship network.  Like the Lightning Network, the
purpose of ILP is to facilitate payments, but it specifically focuses on
payments across disparate ledger types, and extends the atomic transaction
mechanism to include not only hash-locks, but also a quorum of notaries (called
the Atomic Transport Protocol).  The latter mechanism for enforcing atomicity in
inter-ledger transactions is similar to Tendermint's light-client SPV echanism,
so an illustration of the distinction between ILP and Cosmos/IBC is warranted,
and provided below.

1. The notaries of a connector in ILP do not support membership changes, and
   do not allow for flexible weighting between notaries.  On the other hand,
IBC is designed specifically for blockchains, where validators can have
different weights, and where membership can change over the course of the
blockchain.

2. As in the Lightning Network, the receiver of payment in ILP must be online to
   send a confirmation back to the sender.  In a token transfer over IBC, the
validator-set of the receiver's blockchain is responsible for providing
confirmation, not the receiving user.

3. The most striking difference is that ILP's connectors are not responsible or
   keeping authoritative state about payments, whereas in Cosmos, the validators
of a hub are the authority of the state of IBC token transfers as well as the
authority of the amount of tokens held by each zone (but not the amount of
tokens held by each account within a zone).  This is the fundamental innovation
that allows for secure asymmetric tranfer of tokens from zone to zone; the
analog to ILP's connector in Cosmos is a persistent and maximally secure
blockchain ledger, the Cosmos Hub.

4. The inter-ledger payments in ILP need to be backed by an exchange orderbook,
   as there is no asymmetric transfer of coins from one ledger to another, only
the transfer of value or market equivalents.

#### Sidechains

Sidechains [\[15\]][15] are a proposed mechanism for scaling the Bitcoin network
via alternative blockchains that are "pegged" to the Bitcoin blockchain.
Sidechains allow bitcoins to effectively move from the Bitcoin blockchain to the
sidechain and back, and allow for experimentation in new features on the
sidechain.  As in the Cosmos Hub, the sidechain and Bitcoin serve as
light-clients of each other, using SPV proofs to determine when coins should be
transferred to the sidechain and back.  Of course, since Bitcoin uses
proof-of-work, sidechains centered around Bitcoin suffer from the many problems
and risks of proof-of-work as a consensus mechanism.  Furthermore, this is a
Bitcoin-maximalist solution that doesn't natively support a variety of tokens
and inter-zone network topology as Cosmos does. That said, the core mechanism of
the two-way peg is in principle the same as that employed by the Cosmos network.

#### Ethereum Scalability Efforts

Ethereum is currently researching a number of different strategies to shard the
state of the Ethereum blockchain to address scalability needs. These efforts
have the goal of maintaining the abstraction layer offered by the current
Ethereum Virtual Machine across the shared state space. Multiple research
efforts are underway at this time. [\[18\]][18][\[22\]][22]

##### Cosmos vs Ethereum 2.0 Mauve
 
Cosmos and Ethereum 2.0 Mauve [\[22\]][22] have different design goals.

* Cosmos is specifically about tokens.  Mauve is about scaling general computation.
* Cosmos is not bound to the EVM, so even different VMs can interoperate.
* Cosmos lets the zone creator determine who validates the zone.
* Anyone can start a new zone in Cosmos (unless governance decides otherwise).
* The hub isolates zone failures so global token invariants are preserved.

### General Scaling

#### Lightning Network

The Lightning Network is a proposed token transfer network operating at a layer
above the Bitcoin blockchain (and other public blockchains), enabling improvement of many
orders of magnitude in transaction throughput by moving the majority
of transactions outside of the consensus ledger into so-called "payment
channels". This is made possible by on-chain cryptocurrency scripts, which
enable parties to enter into bilateral stateful contracts where the state can
be updated by sharing digital signatures, and contracts can be closed by finally
publishing evidence onto the blockchain, a mechanism first popularized by
cross-chain atomic swaps.  By opening payment channels with many parties,
participants in the Lightning Network can become focal points for routing the
payments of others, leading to a fully connected payment channel network, at the
cost of capital being tied up on payment channels.

While the Lightning Network can also easily extend across multiple independent
blockchains to allow for the transfer of _value_ via an exchange market, it
cannot be used to assymetrically transfer _tokens_ from one blockchain to
another.  The main benefit of the Cosmos network described here is to enable
such direct token transfers.  That said, we expect payment channels and the
Lightning Network to become widely adopted along with our token transfer
mechanism, for cost-saving and privacy reasons.

#### Segregated Witness

Segregated Witness is a Bitcoin improvement proposal
[link](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) that aims
to increase the per-block tranasction throughput 2X or 3X, while simultaneously
making block syncing faster for new nodes.  The brilliance of this solution is
in how it works within the limitations of Bitcoin's current protocol and allows
for a soft-fork upgrade (i.e. clients with older versions of the software will
continue to function after the upgrade).  Tendermint, being a new protocol, has no
design restrictions, so it has a different scaling priorities.  Primarily,
Tendermint uses a BFT round-robin algorithm based on cryptographic signatures
instead of mining, which trivially allows horizontal scaling through multiple
parallel blockchains, while regular, more frequent block commits allow for
vertical scaling as well.

<hr/>

## Appendix ####################################################################

### Fork Accountability

A well designed consensus protocol should provide some guarantees in the event that the tolerance
capacity is exceeded and the consensus fails.  This is especially necessary in
economic systems, where Byzantine behaviour can have substantial financial
reward.  The most important such guarantee is a form of _fork-accountability_,
where the processes that caused the consensus to fail (ie.  caused clients of
the protocol to accept different values - a fork) can be identified and punished
according to the rules of the protocol, or, possibly, the legal system.  When
the legal system is unreliable or excessively expensive to invoke, validators can be forced to make security
deposits in order to participate, and those deposits can be revoked, or slashed,
when malicious behaviour is detected [\[10\]][10].

Note this is unlike Bitcoin, where forking is a regular occurence due to
network asynchrony and the probabilistic nature of finding partial hash
collisions.  Since in many cases a malicious fork is indistinguishable from a
fork due to asynchrony, Bitcoin cannot reliably implement fork-accountability,
other than the implicit opportunity cost paid by miners for mining an orphaned
block.


### Tendermint Consensus 

We call the voting stages _PreVote_ and _PreCommit_. A vote can be for a
particular block or for _Nil_.  We call a collection of +⅔ PreVotes for a single
block in the same round a _Polka_, and a collection of +⅔ PreCommits for a
single block in the same round a _Commit_.  If +⅔ PreCommit for Nil in the same
round, they move to the next round.

Note that strict determinism in the protocol incurs a weak synchrony assumption
as faulty leaders must be detected and skipped.  Thus, validators wait some
amount of time, _TimeoutPropose_, before they Prevote Nil, and the value of
TimeoutPropose increases with each round.  Progression through the rest of a
round is fully asychronous, in that progress is only made once a validator hears
from +⅔ of the network.  In practice, it would take an extremely strong
adversary to indefinetely thwart the weak synchrony assumption (causing the
consensus to fail to ever commit a block), and doing so can be made even more
difficult by using randomized values of TimeoutPropose on each validator.

An additional set of constraints, or Locking Rules, ensure that the network will
eventually commit just one block at each height. Any malicious attempt to cause
more than one block to be committed at a given height can be identified.  First,
a PreCommit for a block must come with justification, in the form of a Polka for
that block. If the validator has already PreCommit a block at round
<em>R_1</em>, we say they are _locked_ on that block, and the Polka used to
justify the new PreCommit at round <em>R_2</em> must come in a round
<em>R_polka</em> where <em>R_1 &lt; R_polka &lt;= R_2</em>.  Second, validators
must Propose and/or PreVote the block they are locked on.  Together, these
conditions ensure that a validator does not PreCommit without sufficient
evidence as justification, and that validators which have already PreCommit
cannot contribute to evidence to PreCommit something else.  This ensures both
safety and liveness of the consensus algorithm.

The full details of the protocol are described
[here](https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm).

### Tendermint Light Clients

The need to sync all block headers is eliminated in Tendermint-PoS as the
existence of an alternative chain (a fork) means ⅓+ of bonded stake can be
slashed.  Of course, since slashing requires that _someone_ share evidence of a
fork, light clients should store any block-hash commits that it sees.
Additionally, light clients could periodically stay synced with changes to the
validator set, in order to avoid [long range
attacks](#preventing-long-range-attacks) (but other solutions are possible).

In spirit similar to Ethereum, Tendermint enables applications to embed a
global Merkle root hash in each block, allowing easily verifiable state queries
for things like account balances, the value stored in a contract, or the
existence of an unspent transaction output, depending on the nature of the
application.

### Preventing Long Range Attacks

Assuming a sufficiently resilient collection of broadcast networks and a static
validator set, any fork in the blockchain can be detected and the deposits of
the offending validators slashed.  This innovation, first suggested by Vitalik
Buterin in early 2014, solves the nothing-at-stake problem of other
proof-of-stake cryptocurrencies (see [Related Work](#related-work)). However,
since validator sets must be able to change, over a long range of time the
original validators may all become unbonded, and hence would be free to create a
new chain from the genesis block, incurring no cost as they no longer have
deposits locked up.  This attack came to be known as the Long Range Attack (LRA),
in contrast to a Short Range Attack, where validators who are currently bonded
cause a fork and are hence punishable (assuming a fork-accountable BFT algorithm
like Tendermint consensus). Long Range Attacks are often thought to be a
critical blow to proof-of-stake.

Fortunately, the LRA can be mitigated as follows.  First, for a validator to
unbond (thereby recovering their collateral deposit and no longer earning fees
to participate in the consensus), the deposit must be made untransferable for an
amount of time known as the "unbonding period", which may be on the order of
weeks or months.  Second, for a light client to be secure, the first time it
connects to the network it must verify a recent block-hash against a trusted
source, or preferably multiple sources.  This condition is sometimes referred to
as "weak subjectivity".  Finally, to remain secure, it must sync up with the
latest validator set at least as frequently as the length of the unbonding
period. This ensures that the light client knows about changes to the validator
set before a validator has its capital unbonded and thus no longer at stake,
which would allow it to deceive the client by carrying out a long range attack
by creating new blocks beginning back at a height where it was bonded (assuming
it has control of sufficiently many of the early private keys).

Note that overcoming the LRA in this way requires an overhaul of the original
security model of proof-of-work. In PoW, it is assumed that a light client can
sync to the current height from the trusted genesis block at any time simply by
processing the proof-of-work in every block header.  To overcome the LRA,
however, we require that a light client come online with some regularity to
track changes in the validator set, and that the first time they come online
they must be particularly careful to authenticate what they hear from the
network against trusted sources. Of course, this latter requirement is similar
to that of Bitcoin, where the protocol and software must also be obtained from a
trusted source.

The above method for preventing LRA is well suited for validators and full nodes
of a Tendermint-powered blockchain because these nodes are meant to remain
connected to the network.  The method is also suitable for light clients that
can be expected to sync with the network frequently.  However, for light clients
that are not expected to have frequent access to the internet or the blockchain
network, yet another solution can be used to overcome the LRA.  Non-validator
token holders can post their tokens as collateral with a very long unbonding
period (e.g. much longer than the unbonding period for validators) and serve
light clients with a secondary method of attesting to the validity of current
and past block-hashes. While these tokens do not count toward the security of
the blockchain's consensus, they nevertheless can provide strong guarantees for
light clients.  If historical block-hash querying were supported in Ethereum,
anyone could bond their tokens in a specially designed smart contract and
provide attestation services for pay, effectively creating a market for
light-client LRA security.

### Overcoming Forks and Censorship Attacks

Due to the definition of a block commit, any ⅓+ coalition of voting power can
halt the blockchain by going offline or not broadcasting their votes. Such a
coalition can also censor particular transactions by rejecting blocks that
include these transactions, though this would result in a significant proportion
of block proposals to be rejected, which would slow down the rate of block
commits of the blockchain, reducing its utility and value. The malicious
coalition might also broadcast votes in a trickle so as to grind blockchain
block commits to a near halt, or engage in any combination of these attacks.
Finally, it can cause the blockchain to fork, by double-signing or violating the
locking rules.

If a globally active adversary were also involved, it could partition the network in
such a way that it may appear that the wrong subset of validators were
responsible for the slowdown. This is not just a limitation of Tendermint, but
rather a limitation of all consensus protocols whose network is potentially
controlled by an active adversary.

For these types of attacks, a subset of the validators should coordinate through
external means to sign a reorg-proposal that chooses a fork (and any evidence
thereof) and the initial subset of validators with their signatures. Validators
who sign such a reorg-proposal forego their collateral on all other forks.
Clients should verify the signatures on the reorg-proposal, verify any evidence,
and make a judgement or prompt the end-user for a decision.  For example, a
phone wallet app may prompt the user with a security warning, while a
refrigerator may accept any reorg-proposal signed by +½ of the original
validators by voting power.

No non-synchronous Byzantine fault-tolerant algorithm can come to consensus when
⅓+ of voting power are dishonest, yet a fork assumes that ⅓+ of voting power
have already been dishonest by double-signing or lock-changing without
justification.  So, signing the reorg-proposal is a coordination problem that
cannot be solved by any non-synchronous protocol (i.e. automatically, and
without making assumptions about the reliability of the underlying network).
For now, we leave the problem of reorg-proposal coordination to human
coordination via social consensus on internet media.  Validators must take care
to ensure that there are no remaining network partitions prior to signing a
reorg-proposal, to avoid situations where two conflicting reorg-proposals are
signed.

Assuming that the external coordination medium and protocol is robust, it
follows that forks are less of a concern than censorship attacks.

In addition to forks and censorship, which require ⅓+ Byzantine voting power, a
coalition of +⅔ voting power may commit arbitrary, invalid state.  This is
characteristic of any (BFT) consensus system. Unlike double-signing, which
creates forks with easily verifiable evidence, detecting committment of an
invalid state requires non-validating peers to verify whole blocks, which
implies that they keep a local copy of the state and execute each transaction,
computing the state root independently for themselves.  Once detected, the only
way to handle such a failure is via social consensus.  For instance, in
situations where Bitcoin has failed, whether forking due to software bugs (as in
March 2013), or committing invalid state due to Byzantine behavior of miners (as
in July 2015), the well connected community of businesses, developers, miners,
and other organizations established a social consensus as to what manual actions
were required by participants to heal the network.  Furthermore, since
validators of a Tendermint blockchain may be expected to be identifiable,
commitment of an invalid state may even be punishable by law or some external
jurisprudence, if desired.

### TMSP Specification

TMSP consists of 3 primary message types that get delivered from the core to the
application. The application replies with corresponding response messages.

The `AppendTx` message is the work horse of the application. Each transaction in
the blockchain is delivered with this message. The application needs to validate
each transactions received with the AppendTx message against the current state,
application protocol, and the cryptographic credentials of the transaction. A
validated transaction then needs to update the application state — by binding a
value into a key values store, or by updating the UTXO database.

The `CheckTx` message is similar to AppendTx, but it’s only for validating
transactions. Tendermint Core’s mempool first checks the validity of a
transaction with CheckTx, and only relays valid transactions to its peers.
Applications may check an incrementing nonce in the transaction and return an
error upon CheckTx if the nonce is old.

The `Commit` message is used to compute a cryptographic commitment to the
current application state, to be placed into the next block header. This has
some handy properties. Inconsistencies in updating that state will now appear as
blockchain forks which catches a whole class of programming errors. This also
simplifies the development of secure lightweight clients, as Merkle-hash proofs
can be verified by checking against the block-hash, and the block-hash is signed
by a quorum of validators (by voting power).

Additional TMSP messages allow the application to keep track of and change the
validator set, and for the application to receive the block information, such as
the height and the commit votes.  

TMSP requests/responses are simple Protobuf messages.  Check out the [schema
file](https://github.com/tendermint/tmsp/blob/master/types/types.proto).

##### AppendTx
  * __Arguments__:
    * `Data ([]byte)`: The request transaction bytes
  * __Returns__:
    * `Code (uint32)`: Response code
    * `Data ([]byte)`: Result bytes, if any
    * `Log (string)`: Debug or error message
  * __Usage__:<br/>
    Append and run a transaction.  If the transaction is valid, returns
CodeType.OK

##### CheckTx
  * __Arguments__:
    * `Data ([]byte)`: The request transaction bytes
  * __Returns__:
    * `Code (uint32)`: Response code
    * `Data ([]byte)`: Result bytes, if any
    * `Log (string)`: Debug or error message
  * __Usage__:<br/>
    Validate a transaction.  This message should not mutate the state.
    Transactions are first run through CheckTx before broadcast to peers in the
mempool layer.
    You can make CheckTx semi-stateful and clear the state upon `Commit` or
`BeginBlock`,
    to allow for dependent sequences of transactions in the same block.

##### Commit
  * __Returns__:
    * `Data ([]byte)`: The Merkle root hash
    * `Log (string)`: Debug or error message
  * __Usage__:<br/>
    Return a Merkle root hash of the application state.

##### Query
  * __Arguments__:
    * `Data ([]byte)`: The query request bytes
  * __Returns__:
    * `Code (uint32)`: Response code
    * `Data ([]byte)`: The query response bytes
    * `Log (string)`: Debug or error message

##### Flush
  * __Usage__:<br/>
    Flush the response queue.  Applications that implement `types.Application`
need not implement this message -- it's handled by the project.

##### Info
  * __Returns__:
    * `Data ([]byte)`: The info bytes
  * __Usage__:<br/>
    Return information about the application state.  Application specific.

##### SetOption
  * __Arguments__:
    * `Key (string)`: Key to set
    * `Value (string)`: Value to set for key
  * __Returns__:
    * `Log (string)`: Debug or error message
  * __Usage__:<br/>
    Set application options.  E.g. Key="mode", Value="mempool" for a mempool
connection, or Key="mode", Value="consensus" for a consensus connection.
    Other options are application specific.

##### InitChain
  * __Arguments__:
    * `Validators ([]Validator)`: Initial genesis-validators
  * __Usage__:<br/>
    Called once upon genesis

##### BeginBlock
  * __Arguments__:
    * `Height (uint64)`: The block height that is starting
  * __Usage__:<br/>
    Signals the beginning of a new block. Called prior to any AppendTxs.

##### EndBlock
  * __Arguments__:
    * `Height (uint64)`: The block height that ended
  * __Returns__:
    * `Validators ([]Validator)`: Changed validators with new voting powers (0
      to remove)
  * __Usage__:<br/>
    Signals the end of a block.  Called prior to each Commit after all
transactions

See [the TMSP repository](https://github.com/tendermint/tmsp#message-types) for more details.

### IBC Packet Delivery Acknowledgement

There are several reasons why a sender may want the acknowledgement of delivery
of a packet by the receiving chain.  For example, the sender may not know the
status of the destination chain, if it is expected to be faulty.  Or, the sender
may want to impose a timeout on the packet (with the `MaxHeight` packet field),
while any destination chain may suffer from a denial-of-service attack with a
sudden spike in the number of incoming packets.

In these cases, the sender can require delivery acknowledgement by setting the
initial packet status to `AckPending`.  Then, it is the receiving chain's
responsibility to confirm delivery by including an abbreviated `IBCPacket` in the
app Merkle hash.

![Figure of Zone1, Zone2, and Hub IBC with
acknowledgement](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/msc/ibc_with_ack.png)

First, an `IBCBlockCommit` and `IBCPacketTx` are posted on "Hub" that proves
the existence of an `IBCPacket` on "Zone1".  Say that `IBCPacketTx` has the
following value:

- `FromChainID`: "Zone1"
- `FromBlockHeight`: 100 (say)
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200 (say)
    - `Status`: `AckPending`
    - `Type`: "coin"
    - `MaxHeight`: 350 (say "Hub" is currently at height 300)
  - `Payload`: &lt;The bytes of a "coin" payload&gt;

Next, an `IBCBlockCommit` and `IBCPacketTx` are posted on "Zone2" that proves
the existence of an `IBCPacket` on "Hub".  Say that `IBCPacketTx` has the
following value:

- `FromChainID`: "Hub"
- `FromBlockHeight`: 300
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckPending`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `Payload`: &lt;The same bytes of a "coin" payload&gt;

Next, "Zone2" must include in its app-hash an abbreviated packet that shows the
new status of `AckSent`.  An `IBCBlockCommit` and `IBCPacketTx` are posted back
on "Hub" that proves the existence of an abbreviated `IBCPacket` on
"Zone2".  Say that `IBCPacketTx` has the following value:

- `FromChainID`: "Zone2"
- `FromBlockHeight`: 400 (say)
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckSent`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `PayloadHash`: &lt;The hash bytes of the same "coin" payload&gt;

Finally, "Hub" must update the status of the packet from `AckPending` to
`AckReceived`.  Evidence of this new finalized status should go back to
"Zone2".  Say that `IBCPacketTx` has the following value:

- `FromChainID`: "Hub"
- `FromBlockHeight`: 301
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckReceived`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `PayloadHash`: &lt;The hash bytes of the same "coin" payload&gt;

Meanwhile, "Zone1" may optimistically assume successful delivery of a "coin"
packet unless evidence to the contrary is proven on "Hub".  In the example
above, if "Hub" had not received an `AckSent` status from "Zone2" by block
350, it would have set the status automatically to `Timeout`.  This evidence of
a timeout can get posted back on "Zone1", and any tokens can be returned.

![Figure of Zone1, Zone2, and Hub IBC with acknowledgement and
timeout](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/msc/ibc_with_ack_timeout.png)

### Merkle Tree & Proof Specification

There are two types of Merkle trees supported in the Tendermint/Cosmos
ecosystem: The Simple Tree, and the IAVL+ Tree.

#### Simple Tree

The Simple Tree is a Merkle tree for a static list of elements.  If the number
of items is not a power of two, some leaves will be at different levels.  Simple
Tree tries to keep both sides of the tree the same height, but the left may be
one greater.  This Merkle tree is used to Merkle-ize the transactions of a
block, and the top level elements of the application state root.

```
                *
               / \
             /     \
           /         \
         /             \
        *               *
       / \             / \
      /   \           /   \
     /     \         /     \
    *       *       *       h6
   / \     / \     / \
  h0  h1  h2  h3  h4  h5

  A SimpleTree with 7 elements
```

#### IAVL+ Tree

The purpose of the IAVL+ data structure is to provide persistent storage for
key-value pairs in the application state such that a deterministic Merkle root
hash can be computed efficiently.  The tree is balanced using a variant of the
[AVL algorithm](http://en.wikipedia.org/wiki/AVL_tree), and all operations are
O(log(n)).

In an AVL tree, the heights of the two child subtrees of any node differ by at
most one.  Whenever this condition is violated upon an update, the tree is
rebalanced by creating O(log(n)) new nodes that point to unmodified nodes of the
old tree.  In the original AVL algorithm, inner nodes can also hold key-value
pairs.  The AVL+ algorithm (note the plus) modifies the AVL algorithm to keep
all values on leaf nodes, while only using branch-nodes to store keys.  This
simplifies the algorithm while keeping the merkle hash trail short.

The AVL+ Tree is analogous to Ethereum's [Patricia
tries](http://en.wikipedia.org/wiki/Radix_tree).  There are tradeoffs.  Keys do
not need to be hashed prior to insertion in IAVL+ trees, so this provides faster
ordered iteration in the key space which may benefit some applications.  The
logic is simpler to implement, requiring only two types of nodes -- inner nodes
and leaf nodes.  The Merkle proof is on average shorter, being a balanced binary
tree.  On the other hand, the Merkle root of an IAVL+ tree depends on the order
of updates.

We will support additional efficient Merkle trees, such as Ethereum's Patricia
Trie when the binary variant becomes available.

### Transaction Types

In the canonical implementation, transactions are streamed to the Cosmos hub
application via the TMSP interface.

The Cosmos Hub will accept a number of primary transaction types, including
`SendTx`, `BondTx`, `UnbondTx`, `ReportHackTx`, `SlashTx`, `BurnAtomTx`,
`ProposalCreateTx`, and `ProposalVoteTx`, which are fairly self-explanatory and
will be documented in a future revision of this paper.  Here we document the two
primary transaction types for IBC: `IBCBlockCommitTx` and `IBCPacketTx`.

#### IBCBlockCommitTx

An `IBCBlockCommitTx` transaction is composed of:

- `ChainID (string)`: The ID of the blockchain
- `BlockHash ([]byte)`: The block-hash bytes, the Merkle root which includes the
  app-hash
- `BlockPartsHeader (PartSetHeader)`: The block part-set header bytes, only
  needed to verify vote signatures
- `BlockHeight (int)`: The height of the commit
- `BlockRound (int)`: The round of the commit
- `Commit ([]Vote)`: The +⅔ Tendermint `Precommit` votes that comprise a block
  commit
- `ValidatorsHash ([]byte)`: A Merkle-tree root hash of the new validator set
- `ValidatorsHashProof (SimpleProof)`: A SimpleTree Merkle-proof for proving the
  `ValidatorsHash` against the `BlockHash`
- `AppHash ([]byte)`: A IAVLTree Merkle-tree root hash of the application state
- `AppHashProof (SimpleProof)`: A SimpleTree Merkle-proof for proving the
  `AppHash` against the `BlockHash`

#### IBCPacketTx

An `IBCPacket` is composed of:

- `Header (IBCPacketHeader)`: The packet header
- `Payload ([]byte)`: The bytes of the packet payload. _Optional_
- `PayloadHash ([]byte)`: The hash for the bytes of the packet. _Optional_

Either one of `Payload` or `PayloadHash` must be present.  The hash of an
`IBCPacket` is a simple Merkle root of the two items, `Header` and `Payload`.
An `IBCPacket` without the full payload is called an _abbreviated packet_.

An `IBCPacketHeader` is composed of:

- `SrcChainID (string)`: The source blockchain ID
- `DstChainID (string)`: The destination blockchain ID
- `Number (int)`: A unique number for all packets
- `Status (enum)`: Can be one of `AckPending`, `AckSent`, `AckReceived`,
  `NoAck`, or `Timeout`
- `Type (string)`: The types are application-dependent.  Cosmos reserves the
  "coin" packet type
- `MaxHeight (int)`: If status is not `NoAckWanted` or `AckReceived` by this
  height, status becomes `Timeout`. _Optional_

An `IBCPacketTx` transaction is composed of:

- `FromChainID (string)`: The ID of the blockchain which is providing this
  packet; not necessarily the source
- `FromBlockHeight (int)`: The blockchain height in which the following packet
  is included (Merkle-ized) in the block-hash of the source chain
- `Packet (IBCPacket)`: A packet of data, whose status may be one of
  `AckPending`, `AckSent`, `AckReceived`, `NoAck`, or `Timeout`
- `PacketProof (IAVLProof)`: A IAVLTree Merkle-proof for proving the packet's
  hash against the `AppHash` of the source chain at given height

The sequence for sending a packet from "Zone1" to "Zone2" through the
"Hub" is depicted in {Figure X}.  First, an `IBCPacketTx` proves to
"Hub" that the packet is included in the app-state of "Zone1".  Then,
another `IBCPacketTx` proves to "Zone2" that the packet is included in the
app-state of "Hub".  During this procedure, the `IBCPacket` fields are
identical: the `SrcChainID` is always "Zone1", and the `DstChainID` is always
"Zone2".

The `PacketProof` must have the correct Merkle-proof path, as follows:

```
IBC/<SrcChainID>/<DstChainID>/<Number>

```

When "Zone1" wants to send a packet to "Zone2" through "Hub", the
`IBCPacket` data are identical whether the packet is Merkle-ized on "Zone1",
the "Hub", or "Zone2".  The only mutable field is `Status` for tracking
delivery, as shown below.

## Acknowledgements ############################################################

We thank our friends and peers for assistance in conceptualizing, reviewing, and
providing support for our work with Tendermint and Cosmos.

* [Zaki Manian](https://github.com/zmanian) of
  [SkuChain](https://www.skuchain.com/) provided much help in formatting and
wording, especially under the TMSP section
* [Jehan Tremback](https://github.com/jtremback) of Althea and Dustin Byington
  for helping with initial iterations
* [Andrew Miller](http://soc1024.com/) of [Honey
  Badger](https://eprint.iacr.org/2016/199) for feedback on consensus
* [Greg Slepak](https://fixingtao.com/) for feedback on consensus and wording
* Also thanks to [Bill Gleim](https://github.com/gleim) and [Seunghwan
  Han](http://www.seunghwanhan.com) for various contributions.
* __Your name and organization here for your contribution__

## Citations ###################################################################

[1]: https://bitcoin.org/bitcoin.pdf
[2]: http://zerocash-project.org/paper
[3]: https://github.com/ethereum/wiki/wiki/White-Paper
[4]: https://download.slock.it/public/DAO/WhitePaper.pdf
[5]: https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
[6]: https://arxiv.org/pdf/1510.02037v2.pdf
[7]: https://lightning.network/lightning-network-paper-DRAFT-0.5.pdf
[8]: https://github.com/tendermint/tendermint/wiki
[9]: https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf
[10]: https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm/
[11]: http://pmg.csail.mit.edu/papers/osdi99.pdf
[12]: https://bitshares.org/technology/delegated-proof-of-stake-consensus/
[13]: https://www.stellar.org/papers/stellar-consensus-protocol.pdf
[14]: https://interledger.org/rfcs/0001-interledger-architecture/
[15]: https://blockstream.com/sidechains.pdf
[16]: https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/
[17]: https://github.com/tendermint/tmsp
[18]: https://github.com/ethereum/EIPs/issues/53
[19]: http://www.ds.ewi.tudelft.nl/fileadmin/pds/papers/PerformanceAnalysisOfLibswift.pdf
[20]: http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf
[21]: https://en.bitcoin.it/wiki/Thin_Client_Security
[22]: http://vitalik.ca/files/mauve_paper.html

* [1] Bitcoin: https://bitcoin.org/bitcoin.pdf
* [2] ZeroCash: http://zerocash-project.org/paper
* [3] Ethereum: https://github.com/ethereum/wiki/wiki/White-Paper
* [4] TheDAO: https://download.slock.it/public/DAO/WhitePaper.pdf
* [5] Segregated Witness: https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
* [6] BitcoinNG: https://arxiv.org/pdf/1510.02037v2.pdf
* [7] Lightning Network: https://lightning.network/lightning-network-paper-DRAFT-0.5.pdf
* [8] Tendermint: https://github.com/tendermint/tendermint/wiki
* [9] FLP Impossibility: https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf
* [10] Slasher: https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm/
* [11] PBFT: http://pmg.csail.mit.edu/papers/osdi99.pdf
* [12] BitShares: https://bitshares.org/technology/delegated-proof-of-stake-consensus/
* [13] Stellar: https://www.stellar.org/papers/stellar-consensus-protocol.pdf
* [14] Interledger: https://interledger.org/rfcs/0001-interledger-architecture/
* [15] Sidechains: https://blockstream.com/sidechains.pdf
* [16] Casper: https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/
* [17] TMSP: https://github.com/tendermint/tmsp
* [18] Ethereum Sharding: https://github.com/ethereum/EIPs/issues/53
* [19] LibSwift: http://www.ds.ewi.tudelft.nl/fileadmin/pds/papers/PerformanceAnalysisOfLibswift.pdf
* [20] DLS: http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf
* [21] Thin Client Security: https://en.bitcoin.it/wiki/Thin_Client_Security
* [22] Ethereum 2.0 Mauve Paper: http://vitalik.ca/files/mauve_paper.html

#### Unsorted links

* https://www.docdroid.net/ec7xGzs/314477721-ethereum-platform-review-opportunities-and-challenges-for-private-and-consortium-blockchains.pdf.html
