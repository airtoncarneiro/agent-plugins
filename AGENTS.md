# Instruções para agentes

## Objetivo do repositório

Este repositório mantém uma coleção de Agent Plugins portáteis e reutilizáveis.

Os plugins devem permanecer independentes de editor, fornecedor ou runtime
específico, salvo quando uma dependência for explicitamente documentada.

## Estrutura

- `README.md`: visão geral do repositório, instalação e contribuição.
- `plugins/README.md`: catálogo dos plugins disponíveis.
- `plugins/<plugin-name>/plugin.json`: manifesto do plugin.
- `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`: instruções da skill.
- `references/`: documentação de apoio específica da skill.
- `assets/`: templates e artefatos utilizados pela skill.

## Regras para adicionar ou alterar plugins

- Preserve a estrutura padrão de cada plugin.
- Mantenha um `plugin.json` válido na raiz do plugin.
- Use nomes em minúsculas e `kebab-case` para plugins e skills.
- Atualize `plugins/README.md` ao adicionar ou remover um plugin.
- Atualize os READMEs quando a estrutura ou o modo de instalação mudar.
- Não adicione dependências de um editor ou fornecedor sem documentá-las.
- Não coloque instruções específicas de manutenção do repositório dentro de
  `SKILL.md`.
- Não coloque instruções essenciais de funcionamento de uma skill apenas no
  README; elas devem estar no `SKILL.md` ou em suas referências.

## Documentação

- Escreva a documentação em português, salvo quando termos técnicos ou
  convenções do ecossistema recomendarem o inglês.
- Prefira exemplos fictícios e públicos.
- Mantenha exemplos coerentes com o comportamento real do plugin.
- Evite afirmar compatibilidade com um runtime sem evidência ou documentação.
- Preserve links relativos válidos entre README, manifestos e skills.

## Validação antes do commit

Antes de concluir uma alteração:

1. Verifique se todos os arquivos referenciados existem.
2. Valide a sintaxe JSON dos manifestos.
3. Execute:

   ```bash
   git diff --check
   ```

4. Revise se o catálogo em `plugins/README.md` corresponde às pastas existentes.
5. Confirme que não há credenciais, tokens ou dados sensíveis nos arquivos.
6. Inspecione o diff completo antes de criar o commit.

## Commits

- Use mensagens curtas e descritivas em inglês.
- Prefira o formato imperativo, por exemplo: `Add plugin catalog documentation`.
- Não misture alterações não relacionadas no mesmo commit.
- Não reescreva o histórico nem force-push sem solicitação explícita.

## GitHub

- O branch principal é `main`.
- Confirme o estado do repositório antes de fazer push.
- Nunca sobrescreva alterações remotas sem verificar e integrar o histórico.
- Após o push, confirme que a branch local está sincronizada com `origin/main`.

## Escopo das alterações

- Preserve alterações existentes feitas pelo usuário.
- Não remova arquivos ou histórico sem autorização explícita.
- Se uma mudança exigir uma decisão de design ou de compatibilidade, explique a
  alternativa e aguarde orientação antes de aplicá-la.
