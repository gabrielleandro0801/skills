---
name: typescript-dependency-injection
description: 'Guia para implementar injeção de dependência no código em TypeScript.'
---

# Recomendações

- Não presuma nem escolha NADA, siga completamente as recomendações e padrões estabelecidos para injeção de dependência no projeto.
- Utilize um container de injeção de dependência para gerenciar as instâncias das classes e suas dependências, garantindo que a criação e o ciclo de vida dos objetos sejam controlados de forma centralizada, preferencialmente o **tsyringe**. Se o projeto já usa um container, siga o padrão existente; só introduza um se houver necessidade clara de um novo container.
- Em caso de utilização do tsyringe, é **fundamental** a utilização do `experimentalDecorators` e do `emitDecoratorMetadata` habilitados no `tsconfig.json` do projeto, garantindo que os decorators funcionem corretamente e que as informações de metadata sejam emitidas para o container de injeção de dependência.
- Estruture as chaves e tokens de injeção de dependência de forma consistente, garantindo que cada dependência seja registrada e resolvida corretamente no container na seguinte forma:
  - dependencies/
    - keys/
      - adapters.ts
      - services.ts
    - containers/
      - adapters.ts
      - services.ts
    - index.ts

Onde cada arquivo .ts dentro da pasta `keys` define as chaves de injeção de dependência correspondentes através de símbolos, que serão utilizadas para registrar e resolver as dependências no container, por exemplo:

``` typescript
// dependencies/keys/adapters.ts
export const FetchHttpClientKey = Symbol("FetchHttpClientKey");
export const AxiosHttpClientKey = Symbol("AxiosHttpClientKey");
export const AwsSqsKey = Symbol("AwsSqsKey");
export const AwsS3Key = Symbol("AwsS3Key");

// dependencies/keys/services.ts
export const UserServiceKey = Symbol("UserServiceKey");
export const AuthServiceKey = Symbol("AuthServiceKey");
```

Cada key deverá ter o sufixo `Key` para manter consistência e clareza na identificação das chaves de injeção de dependência.

Cada arquivo .ts dentro da pasta `containers` deve exportar uma função que registra as dependências correspondentes no container, utilizando as chaves definidas na pasta `keys`. Por exemplo:

``` typescript
// dependencies/containers/adapters.ts
import { container } from "tsyringe";
import { FetchHttpClientKey, AxiosHttpClientKey } from "../keys/adapters";
import { FetchHttpClient } from "../../adapters/FetchHttpClient";
import { AxiosHttpClient } from "../../adapters/AxiosHttpClient";

export function adapters() {
  container.register(FetchHttpClientKey, FetchHttpClient);
  container.register(AxiosHttpClientKey, AxiosHttpClient);
}

// dependencies/containers/services.ts
import { container } from "tsyringe";
import { UserServiceKey, AuthServiceKey } from "../keys/services";
import { UserService } from "../../services/UserService";
import { AuthService } from "../../services/AuthService";

export function services() {
  container.register(UserServiceKey, UserService);
  container.register(AuthServiceKey, AuthService);
}
```

Onde o arquivo index.ts será na seguinte forma:

``` typescript
import { adapters } from "./containers/adapters";
import { services } from "./containers/services";

export function registerDependencies() {
  adapters();
  services();
}
```

E as classes utilizarão o container de injeção de dependência para obter suas dependências, geralmente através de decorators como `@inject` do tsyringe. Por exemplo:

``` typescript
import { inject, singleton } from "tsyringe";
import { UserServiceKey } from "../keys/services";
import { UserService } from "../../services/UserService";

@singleton()
class MyController {
  private userService: UserService;
  constructor(
    @inject(UserServiceKey)
    userService: UserService,
  ) {
    this.userService = userService;
  }
}
```
### Observações

O exemplo contém a estrutura de adapters/services/etc apenas como exemplo. Adapte conforme a necessidade do seu projeto, criando novos arquivos (como services, repositories, etc) para outras dependências que possam existir.
Desta forma, apenas o arquivo index.ts precisa ser importado e chamado para registrar todas as dependências no container, mantendo a organização e modularidade do projeto.

Leia a [documentação oficial](https://github.com/microsoft/tsyringe/blob/master/README.md) do tsyringe para mais detalhes sobre como utilizar o container de injeção de dependência.

A instalação da biblioteca reflect-metadata é necessária para que o tsyringe funcione corretamente. Certifique-se de instalá-la e importá-la no início da aplicação, sendo o primeiro import do entrypoint:

``` typescript
import "reflect-metadata";
```

Caso as classes não guardem estado nem nada que seja utilizado entre as execuções, prefira o uso de `@singleton()` para garantir que apenas uma instância da classe seja criada e reutilizada ao longo da aplicação.
Se as classes precisarem manter estado ou tiverem comportamento dependente do ciclo de vida, utilize `@injectable()` para permitir que o container crie novas instâncias conforme necessário.
Lembre-se de que o uso de `@singleton()` e `@injectable()` deve ser consistente com o ciclo de vida esperado das suas classes, garantindo que o container gerencie corretamente as instâncias e evitando problemas de estado compartilhado indesejado.

Além disso, sempre registre suas dependências no container antes de utilizá-las, preferencialmente no entrypoint da aplicação, para garantir que todas as classes possam resolver suas dependências corretamente.
Caso contrário, você poderá enfrentar erros de resolução de dependência ou comportamento inesperado das classes gerenciadas pelo container.
