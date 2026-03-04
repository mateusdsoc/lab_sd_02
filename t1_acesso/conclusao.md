Reflexão

Para mudarmos de conexão local para http, o código não precisou mudar, somente o env, configurando diretamente no terminal. E o http falhou como esperado.

A função do ConfigRepository é definir a "interface" do usuário, ele define o que pode ser feito. O Remote e o Local são as implementações de uma chamada remota e local de um arquivo respectivamente. O get_repo_from_env() define qual vai ser chamado porque ele conhece o env.