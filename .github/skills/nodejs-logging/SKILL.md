---
name: nodejs-logging
description: 'Guia para implementar logging no código em TypeScript/JavaScript utilizando a biblioteca Winston.'
---

# Recomendações

- Não presuma nem escolha NADA, siga completamente as recomendações e padrões estabelecidos para logging no projeto.
- Leia as referências desta skill na pasta `.github/skills/nodejs-logging/references` para entender os padrões e práticas recomendadas de logging no projeto.
- Se o projeto já possui uma solução de logging, reutilize-a e siga seus padrões, adaptando às regras desta skill conforme for possível.
- Se o projeto possui uma abstração, como ILogger, utilize-a em vez de acoplar o código de negócio diretamente à implementação concreta do logger.
- Se não houver uma implementação de logging existente, Winston deve ser a escolha padrão, caso não exista outra restrição, pois ela oferece flexibilidade, suporte a múltiplos transportes e formatos de log configuráveis.
- Não introduza uma nova biblioteca de logging sem necessidade.
- Utilize níveis de log apropriados (DEBUG, INFO, WARN, ERROR) para categorizar a importância das mensagens de log.
- Utilize a feature de Log Streamer descrita nesta implementação para gerenciar logs cujos níveis estão abaixo do definido na aplicação (por exemplo, o nível está INFO e o log é DEBUG). Desta forma, estes logs serão emitidos apenas quando um log de erro ocorrer, evitando poluição do log principal com informações de menor relevância.
- Caso o projeto utilize um container para injeção de dependências, como o tsyringe, é possível registrar o logger como um singleton no container e injetá-lo onde for necessário, garantindo que todas as partes do sistema utilizem a mesma instância do logger, sem a necessidade da propriedade estática `instance` da classe `WinstonLogger`.
- O método `createStream` deve ser chamado no início da execução da aplicação para garantir que todos os logs temporários sejam capturados desde o início, antes de qualquer operação que possa gerar logs, logo após a definição do `executionId` no `contextWrapper`.
- O método `finishStream` deve ser chamado no final da execução da aplicação, para que os logs temporários em memória sejam **descartados** corretamente. Preferencialmente, deve ser chamado em um bloco `finally` para assegurar que o stream seja finalizado mesmo em caso de erros durante a execução da aplicação.
- Além disso, é **fundamental** garantir que o `executionId` no `contextWrapper` seja definido corretamente antes de criar o stream, pois ele é utilizado como chave para armazenar e recuperar os logs temporários.
- O cuidado com o conteúdo dos logs para evitar a exposição de informações sensíveis deve ser realizado pelas partes do código que chamam o logger, não pelo próprio logger, e é essencial. Certifique-se de não registrar dados como senhas, tokens, informações pessoais identificáveis ou qualquer outro dado que possa comprometer a segurança ou a privacidade dos usuários.
