---
layout:     post
title:      "Server-side: características de um servidor"
subtitle:   "Um pouco sobre a arquitetura dos servidores"
date:       2017-03-19 00:00:00
author:     "Ronaldo Faria Lima"
header-img: "img/ronflima/server-background.jpeg"
category:   server
---

# Antes de começar

Este artigo trata, basicamente, de características do sistema operacional. Pode
parecer um pouco fora dos assuntos do EquinociOS, mas é algo bastante
relevante. Desde que a Apple liberou a linguagem Swift como um projeto de
código-aberto, muita gente vem usando a linguagem para desenvolver software do
lado do servidor. Assim, torna-se bastante relevante o conhecimento que é
apresentado neste artigo pois o software servidor executa em um ambiente que
fará algumas exigências para o seu correto funcionamento.

É importante ter em mente que o ambiente no qual seu software executa precisa
ser respeitado. Este ambiente proporcionará vantagens, e também desvantagens,
que precisam ser entendidas e avaliadas. O intuito é integrar bem o seu software
com este ambiente e tirar proveito das vantagens evitando-se as desvantagens.

Considerando-se que o software do lado do servidor normalmente funciona em
esquema 24 x 7 x 365, a boa integração com o ambiente aumenta bastante a
confiabilidade e a robustez. Portanto, é importante conhecer, mesmo que
superficialmente, o ambiente de execução do seu servidor.

# O que é o servidor?

O conceito de servidor não é novo. Apareceu na série 360 da IBM em 1964. Desde
então vem sofrendo transformações até chegar no modelo em nuvem conforme temos
hoje. Mas, o que raios é o servidor?

Olhando pela ótica do software, que é a premissa deste artigo, o servidor é um
artefato de software cuja função é prover serviços à outros artefatos de
software, chamados de _clientes_. Esta não é uma definição formal nem muito
menos precisa. A intenção é partirmos desta definição para entender como
projetar um servidor de software de maneira a usar as principais características
do sistema operacional no qual estará hospedado.

Hoje em dia o papel principal do servidor é prover serviços a clientes leves, ou
_thin clients_, normalmente aplicativos para celulares. Todo o processamento
pesado e o armazenamento fica por conta do servidor, deixando o cliente com a
tarefa de organizar, logicamente, os fluxos de trabalho disponíveis para que o
usuáio seja atendido em determinada funcionalidade.

O servidor pode ser uma ou mais peças de software trabalhando em conjunto,
formando um sistema que pode ser agregado ou distribuído. Para o cliente, o
servidor é um só, mesmo que sejam várias máquinas físicas diferentes, vários
sistemas de software criados por várias linguagens diferentes, interagindo entre
si através das mais diversas formas de integração.

# Linux e Unix

Nos dias de hoje o sistema operacional mais popular para _back-end_ é, sem
dúvida, o Linux. O Unix é um sistema muito popular no ambiente corporativo,
apesar das vendas deste sistema decrescer ano após ano. Sendo rigoroso, o Unix
não é Linux e Linux não é Unix. São sistemas completamente diferentes, apesar
das suas similaridades.

Apesar do foco deste artigo ser o Linux, os princípios e conceitos aqui
apresentados valem para o Unix. Vale lembrar que o FreeBSD é um Unix e não
Linux. A camada de compatibilidade entre o BSD e o Linux é tão boa e
transparente que ela faz com que praticamente tudo o que executa em um sistema
seja executado em outro.

De toda forma vou intercambiar os termos Unix e Linux ao longo do texto,
tratando tudo como se fosse Unix por questão de simplicidade. Se houver algo que
seja inerente a um sistema, deixarei explícito para que não haja confusões.

Vamos começar por alguns conceitos básicos que serão usados através do restante
do texto. Com o intuito de equalizar o conhecimento, apresento de forma bastante
rápida estes conceitos.

## Daemon

Apesar do termo lembrar os demônios cristãos, ou seja, entidades malévolas, o
termo _daemon_ foi emprestado do termo grego _daimon_ que refere-se a entidades
naturais benignas e que estão sempre presentes em nossas vidas.

A despeito da questão religiosa, um _daemon_ é, na verdade, um processo autônomo
que é executado em segundo plano, fornecendo ou executando algum tipo de
serviço. Normalmente os daemons no Unix/Linux são processos iniciados durante a
carga do sistema, através de um sistema de inicialização como o System V ou o
upstart do Ubuntu.

Uma vez iniciados, estes processos normalmente terminam quando o sistema é
reiniciado, ou seja, são processos de execução longa, projetados para
permanecerem funcionando por longos períodos de tempo.

## IPC

IPC é uma sigla que significa _Inter-Process Communication_. Tanto o Unix quanto
o Linux implementam várias formas de comunicação inter-processo:

- _pipes_ para escrita e leitura síncrona de dados;
- _message queues_ para escrita e leitura assíncrona de dados;
- _semáforos_ para o controle atômico de recursos compartilhados;
- _shared memory_ para compartilhamento de arenas de memória;
- _sockets_, em particular os sockets Unix, para troca assíncrona de dados
  arbitrários;
- _sinalização_, usado para entregar eventos a processos arbitrários.

Muitos autores não consideram a sinalização como parte do IPC. No entanto, é uma
característica importante do sistema operacional pois é através dos sinais que o
sistema de inicialização comunica-se com os deamons.

Os sinais são eventos assíncronos que podem, ou não, ser capturados e tratados
por um processo. Apesar dos sinais serem números inteiros, eles são limitados
pelo sistema operacional. Os sinais são definidos no arquivo de cabeçalho
_signal.h_, que normalmente está no diretório ```/usr/include```.

## System Calls

Sempre que seu programa precisa interagir com o sistema operacional ele fará uma
chamada de sistema, ou _system call_. Mesmo que você não o faça diretamente,
algumas operações sempre terminam em uma chamada de sistema, como alocação de
memória, abertura e escrita em arquivos, comunicação por rede e por aí vai.

As chamadas de sistema são muito importantes pois permitem que o seu software
interaja com o kernel do sistema operacional, trazendo uma série de vantagens
importantes no que tange a integração de sistemas.

## Linguagem C

Por que coloco a linguagem C na lista de definições? A resposta é muito simples:
tanto o Unix quanto o Linux são sistemas operacionais escritos em C. Os _system
calls_ são todos escritos em C e são acessados através de bibliotecas escritas
em C do próprio sistema operacional. Assim, é impossível falar em Linux e Unix
sem mencionar esta linguagem.

Para quem programa em Swift e deseja criar software do lado do servidor aqui vai
a primeira dica: se você não sabe C, procure aprender. Infelizmente Swift não
possui, nativamente, uma forma de realizar chamadas de sistema. A captura de
sinais, criação de processos-filhos e outras características do sistema
operacional só são conseguidas se você interfacear seu código Swift com wrappers
escritos em C.

Na plataforma Apple a integração é quase _out-of-the-box_ ao importar-se o
módulo _Darwin_. Porém, muita coisa importante fica de fora. Além disso, as
bibliotecas da Apple acabam se tornando um estorvo. Por exemplo, a função
_sigaction_ usa uma estrutura como um dos parâmetros para instalação de handlers
de sinal. No caso da Apple, esta estrutura contém uma união que é mascarada por
macros. Para a linguagem C isso é irrelevante, mas para o Swift, faz toda a
diferença. O código Swift neste caso, compilará para a plataforma Apple se a
estrutura for usada conforme definida, e falhará para o Linux. A recíproca é
totalmente verdadeira, ou seja, o código feito para Linux não compilará na
plataforma Apple. É necessário, portanto, fazer um wrapper em C para permitir o
funcionamento _cross platform_.

## Processo

Sempre que um executável entra em execução no Unix e no Linux, ele cria um
_processo_. O processo é uma instância de execução do seu executável. Em linhas
gerais, o mesmo executável pode ter diversas instâncias de execução, podendo ser
chamado quantas vezes o usuário bem desejar. A cada instância de execução é
asssociado um número inteiro que identifica este processo para o sistema
operacional. Este número é chamado de _PID_, de _process identification_. A
estrutura de numeração de processos nos sistemas operacionais Unix e Linux é uma
lista circular. Isto é feito para que os números de processo sejam
reaproveitados. Assim, o processo criado para um determinado executável pode ter
atribuído um número inteiro qualquer, repetindo-se ou não.

Normalmente o processo cujo PID é 1 é o processo de inicialização do sistema, o
processo que é pai de todo mundo, chamado de _init_. Tudo no Unix e Linux é
iniciado pelo _init_. Os usuários, quando autenticados e conectados a estes
sistemas utilizam processos que foram inicialmente criados pelo init, mas que
deram origem a outros processos.

### Processo-filho

O processo-filho é um processo que é criado por outro processo. O processo-filho
tem como característica um número inteiro chamado de _PPID_, ou _parent process
ID_. O processo-pai pode ser um processo-filho de outro processo e este fato
implica na criação de uma árvore de processos.

As implicações disso são importantes para o correto design de um software do
lado do servidor, conforme veremos a seguir.

# Daemons e o init

Um daemon é um programa comum, como qualquer outro, porém com ciclo longo de
execução. Normalmente, o daemon executa apenas uma única instância na máquina
onde está hospedado. Via de regra, os daemons são iniciados pelo sistema de
inicialização do Unix, podendo ser um sistema baseado no System V ou em outra
forma de inicialização. Normalmente os sistemas Linux têm maneiras um pouco
diferentes de inicialização, como o ubuntu que está movendo-se para o _upstart_,
um sistema de inicialização um pouco mais eficiente e mais flexível que o antigo
System V.

O sistema de inicialização não é, na verdade, importante para os propósitos
deste texto. O fato é que todo daemon pode ser iniciado pelo init e a
inicialização garante que o processo iniciado não estará associado a nenhum
terminal. E aqui começamos a ver o que é realmente importante ter no seu
processo do lado do servidor para que ele intereja de forma adequada com o
sistema operacional.

## Sinalização

Uma das formas que o sistema de inicialização tem de comunicar-se com o seu
software é através do envio de sinais. Os sinais são números inteiros e cada
número tem seu significado. A listagem de sinais normalmente é compilada através
de um conjunto de macros declarados em _signal.h_. Este arquivo normalmente
reside no diretório `/usr/include`.

A sinalização ocorre de maneira assíncrona e interrompe qualquer que seja o
fluxo em execução no seu programa, retomando ao ponto de interrupção tão logo o
sinal seja tratado ou ignorado. É importante ter em mente, no entanto, que nem
todo sinal pode ser tratado. Alguns sinais, em especial `SIGKILL`, não podem ser
capturados nem tratados pelo seu software. O seu comportamento acaba sendo
definido pelo sistema operacional. Por exemplo, `SIGKILL` elimina o processo que
executa o seu software da memória, não importa o que o seu software esteja
fazendo. Este é um dos sinais que não podem ser nem capturados nem tratados.

### Sinais notáveis

Alguns sinais são importantes pois são usados com frequência pelos sistemas de
inicialização e controle de processos. Em particular os sinais `SIGTERM` e
`SIGHUP` são usados para terminar ou reiniciar um daemon, respectivamente. Assim
é importante capturar e tratar `SIGTERM`para que seu daemon seja finalizado de
forma graciosa; e `SIGHUP` para que seu processo possa reiniciar-se, ou seja,
recarregar a configuração e iniciar do zero, como se estivesse sendo carregado
pela primeira vez.

Ao capturar e tratar os sinais _notáveis_ você estará melhorando a integração do
seu software com sistemas Unix, tornando-o mais adequado para uso com os
sistemas de inicialização, seja System V ou seja via upstart.

## Daemonização

Se você abrir um terminal, seja um Xterm, ou um terminal de texto pendurado a
uma linha serial, qualquer processo que você iniciar manualmente estará
associado ao seu terminal. Este é um trabalho que o kernel faz no intuito de
finalizar qualquer processo caso seu terminal seja desconectado. Esta
característica é boa e não é.

O fato é que pode ser necessário reiniciar um daemon manualmente. Digamos que
seja necessário realizar uma manutenção no servidor, por algum motivo. Portanto,
é imperativo que seu daemon não esteja associado com o seu terminal pois, se
estiver, ao fechar o seu terminal o processo cai.

Os sistemas de inicialização já garantem que o seu daemon seja executado
dissociado de um terminal. Porém, a daemonização pode auxiliá-no no design do
seu daemon simplificando uma série de coisas, como a reinicialização _a quente_
do seu processo, conforme expliquei na seção sobre sinais notáveis.

Normalmente você vai iniciar o seu daemon através do sistema de inicialização,
via _start_, _initctl_ ou algum comando especial que reside no _/sbin_ ou
_/usr/sbin_ dependendo do seu Unix ou Linux. 

A daemonização consiste em dissociar seu processo do seu terminal ativo. Isto é
algo muito simples de ser feito, na realidade. A ideia por trás disso é criar um
processo-filho e o processo-filho criar outro processo-filho que, em última
instância, executará o código do seu daemon, tomando-se o cuidado de terminar os
processos-pai sem aguardar o retorno dos processos-filhos.

Neste cenário o processo neto é adotado pelo sistema de inicialização e
torna-se, efetivamente, dissociado do seu terminal. Isto é tão simples de ser
realizado que dá para fazer num shell script:


    { { a=0; while [ $a -lt 10 ]; do sleep 10; a=$((a+1)); echo "Executado $a vezes" >> $HOME/output.txt; done }& }&


Este comando meio esquisito inicia um contador que escreve um arquivo texto a
cada 10 segundos, durante 100 segundos. Se antes de finalizar o script você
fechar o seu terminal o arquivo _output.txt_ continuará a crescer.

O truque é bem simples: cria-se um processo-filho e dentro deste processo-filho
cria-se outro processo-filho. Em C, teríamos algo assim usando a forma clássica
de criação de processos-filho:

    if (fork() == 0) {
        if (fork() == 0) {
            /* Aqui vai o seu processo. Este é o processo-neto */
        } else {
            /* Termina o processo-fiho sem esperar o sincronismo */
            exit(0);
        }
    } else {
        /* Termina o processo-pai sem esperar para sincronizar. */
        exit(0);
    }

Trabalhar com multi-processos no Unix sempre foi um assunto confuso. A chamada
de sistema _fork_ retorna ao processo-pai o PID do processo-filho. Para o
processo-filho, o retorno do system call é zero. O que não fica explícito no
código é que ao chamar o _fork_, o seu processo se divide em dois. O que está no
`else` do `if (fork() == 0)` é o processo-pai. O que está no `then` é o
processo-filho.

As chamadas ao system call `exit` faz com que o processo seja finalizado
imediatamente. No caso do exemplo, os processos-pai serão finalizados sem
aguardar os processos-filho retornarem. Como dito, em uma situação como esta, o
processo-filho não tem com quem sincronizar para retornar pois os processos-pai
já foram reciclados pelo sistema. Assim, o processo _init_ assume o processo
órfão e este fica dissociado de um terminal, passando a rodar em segundo plano.

De uma forma mais moderna, pode-se usar a função `posix_spawn` para carregar
novamente o seu processo em outro espaço de endereçamento. A diferença entre
usar isto e a chamada `fork` é que esta última pode gerar problemas com os
frameworks da Apple, em particular se você estiver programando em Swift.

A chamada a `posix_spawn` gera um pouco mais de trabalho. Esta função cria um
processo-filho de forma diferente, iniciando o novo processo literalmente do
zero. Você precisa informar a `posix_spawn` qual o caminho completo do
executável, ou seja, esta função cria um processo arbitrário totalmente novo que
pode, ou não, ser igual ao processo em execução.

O trabalho extra é que será necessário processar a linha de comando para que seu
programa saiba que está executando como um _child process_. Este é o método
preferido se você escrever seu código em Swift. O compilador LLVM sequer gera
executável se você tentar usar o `fork` justamente porque este system call é
danoso para os frameworks da Apple.

Como o system call `fork` é mais simples, achei melhor usá-lo para explicar o
princípio, que continua rigorosamente o mesmo se você usar `posix_spawn`.

## Syslog e o logging do seu daemon

Existem diversas bibliotecas de logging por aí afora, que trabalham com níveis
de logging e mais uma penca de coisas. Tanto o Linux quanto o Unix oferecem um
sistema completo de logging que normalmente é ignorado pelos desenvolvedores:
_syslog_.

O _syslog_ tem diversas vantagens que sobrepujam qualquer outro sistema de
logging:

- se o seu processo morre inesperadamente, o log é preservado. Se alguma
  informação ficou pendente, ela é salva no log pois a comunicação com o
  _syslog_ não é bufferizada.
- o logging é feito por um daemon, _syslogd_ normalmente. A comunicação é feita
via pipes ou Unix sockets e é extremamente rápida.
- o _syslog_ faz rotação de log, compactando arquivos antigos e eliminando
  arquivos muito antigos.

Normalmente o _syslog_ salva as informações de logging no diretório
`/var/log`. Este diretório pode variar de acordo com a sua distribuição de Linux
ou sabor de Unix. Algumas distros salvam os logs em `/var/spool/log`.

Fazer saídas para _stdout_ ou _stderr_ normalmente não é uma boa ideia. Alguns
sistemas de inicialização fazem o redirecionamento destes streams para arquivos
de log, mas isto não é garantido. Se você gosta de prints da vida, uma dica é:
não faça isso. Use o _syslog_.

## Scheduling

Este é um assunto muito mais voltado à administração do sistema do que
efetivamente ao seu processo. Mas vale a pena passar o olho nisto pois é
possível para um processo alterar a sua própria prioridade.

_Scheduling_ tem a ver com a forma como o seu processo será executado dentro do
Linux/Unix. O sistema operacional divide um _quantum_ arbitrário de tempo com
base na prioridade de cada processo, dando a cada processo a oportunidade de
executar. Quanto maior a prioridade de um processo, mais tempo ele terá dentro
da CPU.

Como o Unix é um sistema operacional multi-processo, o _scheduling_ foi a forma
encontrada para compartilhar a CPU com os diversos processos que executam ao
mesmo tempo pois a CPU é uma só. Se há 4 cores, até 4 processos executam
realmente em paralelo. Os demais precisam aguardar para executar.

O _scheduling_ implica na performance da sua aplicação. Quanto maior for a
prioridade, mais rápido é a execução de um processo. A prioridade permite que
seu processo possa, automaticamente, escalar-se para usar mais e mais CPU à
medida em que a demanda aumenta. Assim, o seu processo começa a usar mais CPU e
torna-se mais eficiente.

## Multi-processos

Não é incomum um daemon ser arquitetado para usar diversos processos, cada um
com uma finalidade bem determinada. Um _pattern_ muito comum é o _pipeline_. A
ideia do pipeline é implementar um workflow usando vários processos. Cada passo
do workflow é realizado por um processo que, ao finalizar, passa para o processo
subsequente a requisição com o resultado do seu processamento.

Este tipo de design é muito usado para permitir o processamento paralelo e exige
alguma forma de comunicação entre os processos. Ao invés de inventar moda, use
alguma forma de IPC: pipes, message queues, semáforos, etc. Tudo isto está
presente e disponível no seu Linux ou Unix há anos, sendo uma maneira muito
eficiente e estável de permitir que vários processos troquem informações entre
si.

Assim, quando vários processos estão na mesma máquina, não há a necessidade de
usar sistemas de mensagens distribuídas, como o MQ Series ou o Rabbit MQ. O
próprio Linux/Unix já lhe dá as message queues prontas para uso.

# Swift Server-side

A linguagem Swift tem ganhado destaque no desenvolvimento de aplicações do lado
do servidor. Porém, Swift não tem wrappers nativos para integrar chamadas de
sistema com a linguagem. Assim, a forma de realizar isto é importar código C
para o seu software, o que muitas vezes exige a necessidade de escrever wrappers
em C antes de importá-los para o seu código em Swift.

Por exemplo, a função ```sigaction``` tem uma implementação no macOS que exige
que um wrapper C seja criado no intuito de manter o código portável para o
Linux. Assim, cedo ou tarde será necessário escrever algum código em C para que
você integre, adequadamente, o seu servidor Swift no Linux ou Unix.

# Conclusão

A correta integração do software server-side com o sistema operacional traz
benefícios inúmeros, tornando seu software mais robusto e usando as
características do sistema operacional como o _syslog_. A integração com o
sistema de inicialização também é importante para garantir a correta
inicialização/término da sua aplicação.

Como os sistemas Unix foram escritos basicamente em C, cedo ou tarde será
necessário escrever algum código para que seja possível integrar-se o seu
servidor Swift ao sistema operacional.
