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
