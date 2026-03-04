# Reflexões da Tarefa 3 - Transparência de Migração

Quando salvo local a sessão não persiste, o usuário perde. Entretanto, quando a transparência foi aplicada a sessão persistiu e o usuário não percebeu. Isso mostra que a aplicação não deve guardar estado na sua própria memória (stateless). O Redis é um banco de dados compartilhado (stateful store) sendo possível aplicar esse conceito.

Isso ocorre porque cada instância roda em um processo isolado pelo SO, e processos não compartilham memória entre si. 
