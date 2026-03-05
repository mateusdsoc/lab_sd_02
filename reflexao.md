# Reflexão

**T1**

    Para mudarmos de conexão local para http, o código não precisou mudar, somente o env, configurando diretamente no terminal. E o http falhou como esperado.

    A função do ConfigRepository é definir a "interface" do usuário, ele define o que pode ser feito. O Remote e o Local são as implementações de uma chamada remota e local de um arquivo respectivamente. O get_repo_from_env() define qual vai ser chamado porque ele conhece o env.

**T2**

    O ServiceLocator está inicializando um dict local usando apenas o .get(), para ficar diâmico o .resolve() precisaria fazer uma requisição a cada chamada pro Service em tempo real, perguintando qual IP corresponde a cada nome lógico de user-service.

    Duas outras tecnologias seriam o ApacheZooKeeper e o Netflix Eureka.

**T3**

    Quando salvo local a sessão não persiste, o usuário perde. Entretanto, quando a transparência foi aplicada a sessão persistiu e o usuário não percebeu. Isso mostra que a aplicação não deve guardar estado na sua própria memória (stateless). O Redis é um banco de dados compartilhado (stateful store) sendo possível aplicar esse conceito.

    Isso ocorre porque cada instância roda em um processo isolado pelo SO, e processos não compartilham memória entre si.

**T4**

    Na migração, o processo da instância A é encerrado antes de a instância B assumir o tráfego. Já na relocação, o cliente continua ativo e enviando mensagens enquanto a troca de endpoint acontece. A relocação é tecnicamente mais difícil porque exige (a) detectar o momento exato da mudança, (b) bufferizar as mensagens que chegam durante a transição para não perdê-las, e (c) reenviar esse buffer na ordem correta após reconectar.

    Não, o `_message_buffer` sozinho não garante semântica *exactly-once*. Se a nova conexão (`RECONNECTING`) for estabelecida com sucesso mas a chamada `self._ws.send(buffered_msg)` travar a meio caminho, a mensagem pode ter sido recebida pelo servidor, mas o cliente vai tentar reenviar do buffer. Para isso seria necessário um mecanismo de acknowledgement (ACK) do servidor e IDs de sequência por mensagem.

    Enquanto a relocação tem 3 estados (MIGRATING -> RECONNECTING -> CONNECTED), o boolean só tem 2, utilizar `is_relocating` necessitaria de outras flags ou condicionais aninhados para saber qual estado atual.
    
    Um exemplo é o live migration de VMs, onde a memória e CPU são copiadas para outro host físico enquanto a VM ainda está rodando, sem derrubar as conexões ativas.

**T5**

    O código não implementa o read-your-writes, porque o pick_replica() está fazendo uma escolha aleatória entre as réplicas. Para garantir read-your-writes, as leituras que seguem uma escrita precisariam ser forçadas para o master, pelo menos até a replicação ser confirmada.

    Usar recursão no fallback é perigoso porque, se o master também falhar, a função chama a si mesma de novo até estourar a pilha. A versão atual resolve isso chamando connect(master_dsn) sem passar pelo query() de novo.

**T6**

    O saldo no Redis varia, em algumas execuções é 800 e em outras 700.

    Essa tarefa utiliza multiprocessing e não threading porque o threading utilizando python tem o CPython possui o GIL (Global Interpreter Lock), que impede que duas threads executem bytecode Python simultaneamente no mesmo processo. Isso significa que uma race condition com threading pode nao se manifestar de forma reproduzivel — tornando a demonstracao pedagogicamente imprecisa. Com multiprocessing cada processo tem seu proprio espaco de memoria e proprio GIL: a race condition e real, reproduzivel, e reflete com mais fidelidade o cenario de sistemas distribuidos, onde os processos concorrentes estao em maquinas diferentes.

    O threading.Lock() existe na RAM de um processo, não podendo ser compartilhado entre eles, enquanto o distributed_lock utiliza o Redis, todos os processos enxergam.

    Se o processo travar antes de chegar no finally teoricamente ocorreria um deadlock, entretanto, como foi definido ex=ttl, e no nosso caso ttl=5, o redis apaga a chave sozinho se depois de 5 segundos. Entretanto, existem riscos, caso o ttl seja muito curto, acaba antes da seção crítica finalizar, e também se ele for muito longo, se um processo travar os outros ficam esperando durante muito tempo.

**T7**

    A falácia da computacao distribuida de Peter Deutsch que o anti_pattern viola diretamente é a primeira, "The network is reliable".

    O aync/await é uma forma de quebrar a transparência, uma vez que eles comunicam que o código pode suspender porque ele espera algo externo, então não é instantâneo. É a decisão certa porque deixa a mostra os possíveis riscos que a chamada a essa função pode ter.

**Bloco de Reflexão**

    1. Na minha opinião, a Transparência de Falha é o mais difícil de se implementar, dada sua importância. Os usuários muitas vezes param de utilizar uma aplicação por causa dos erros, e o trabalho do desenvolvedor com a transparência de falha é justamente ocular os erros aos olhos de quem utiliza o sistema. Além disso, a transparência de falha utiliza os outros conceitos, como por exemplo ao assistir um vídeo da netflix, se o player travar, automaticamente o algoritmo deve tentar se reconectar a outro servidor que tenha o mesmo filme/série, utilizando nesse caso a transparência de replicação. Não só isso, mas como também a parte que pra mim é a mais difícil, sobre o async e await, quando utilizar e quando não utilizar, comentado nos exercícios.

    2. O Valorant é um FPS tático que costumo jogar nos finais de semana, dado essa categoria, ele mostra inúmeros problemas como perda de pacote, FPS baixo, ping alto e outras estatísticas que você pode adicionar. Se o jogo adotasse uma transparência de falha total, o usuário poderia estar travando e não iria saber se é o fps, o ping ou a perda de pacote.

    3. O async e await quebram essa transparência no código, uma vez que eles mostram que o sistema depende de algo externo, que pode ser que demore. No exercício t7_falha, é possível ver que tentar mascarar totalmente quebra o conceito de "The network is reliable" porque esconde falhas de rede.

    4. A tarefa 6 utiliza multiprocessing ao invés de threading por didática. O Python tem o GIL(Global Interpreter Lock) que tem função de impedir duas threads de executarem bytecode Python simultaneamente no mesmo processo, possibilitando uma race condition de não se manifestar de forma reproduzível, entretanto em multithreading cada processo tem seu próprio espaço de memória e seu próprio GIL, então como estamos utilizando Python para facilitar a didática o multithreading é melhor.

    5. Uma dificuldade técnica encontrada durante o laboratório foi que todos os alunos estavam criando conta no Redis utilizando a mesma rede da PUC em um intervalo curto de tempo, então o Redis baniu a rede da PUC por um perído, provavelmente o algoritmo identificou um falso ataque.
