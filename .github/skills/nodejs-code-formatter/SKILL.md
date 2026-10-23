---
name: nodejs-code-formatter
description: 'Guia para implementar formatação de código em TypeScript/JavaScript.'
---

# Recomendações

- Não presuma nem escolha NADA, siga completamente as recomendações e padrões estabelecidos para formatação no projeto.
- Priorize os padrões de formatação estabelecidos no projeto. Caso não existam, utilize preferencialmente o `prettier`. [Documentação](https://prettier.io/docs/install).
- Utilize um arquivo de configuração de formatação (como `.prettierrc`) para centralizar as regras e facilitar a manutenção do padrão de código. Há um exemplo em references/prettierrc.md.
- Certifique-se de que todos os desenvolvedores do projeto utilizem a mesma configuração de formatação para garantir consistência no estilo do código.
- Caso utilize o prettier, crie dois comandos no `package.json`: um para verificar o estilo do código (`format`) e outro para corrigir automaticamente os problemas de estilo (`format:fix`). Por exemplo:
```json
"scripts": {
    "format": "prettier --config .prettierrc 'src/**/*.ts' --check",
    "format:fix": "prettier --config .prettierrc 'src/**/*.ts' --write"
}
```
- Caso haja testes no projeto, adicione o diretório nos comandos, por exemplo:
```json
"scripts": {
    "format": "prettier --config .prettierrc 'src/**/*.ts' 'test/**/*.ts' --check",
    "format:fix": "prettier --config .prettierrc 'src/**/*.ts' 'test/**/*.ts' --write"
}
```
- A biblioteca de formatação deve ficar listada como dependência de desenvolvimento no `package.json` do projeto, garantindo que todos os desenvolvedores utilizem a mesma versão da ferramenta.
- Certifique-se de executar os comandos de formatação regularmente para manter a consistência do código e evitar problemas de estilo acumulados.
- Considere integrar a formatação no processo de integração contínua (CI) para garantir que todos os commits atendam aos padrões de estilo antes de serem mesclados no branch principal.
- Sempre atualize a configuração de formatação e as dependências associadas conforme o projeto evolui, garantindo que novas regras e padrões sejam aplicados de forma consistente.
