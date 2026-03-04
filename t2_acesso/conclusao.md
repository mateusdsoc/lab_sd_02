Reflexao

O ServiceLocator está inicializando um dict local usando apenas o .get(), para ficar diâmico o .resolve() precisaria fazer uma requisição a cada chamada pro Service em tempo real, perguintando qual IP corresponde a cada nome lógico de user-service.

Duas outras tecnologias seriam o ApacheZooKeeper e o Netflix Eureka.