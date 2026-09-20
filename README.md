# Agent Plugins Universal Repository

Este repositório contém um plugin portátil e reutilizável compatível com o padrão
[Agent Plugins](https://agent-plugins.org/). A estrutura foi organizada para manter
o núcleo universal enxuto, independente de editor ou runtime específico.

## Objetivo

A fonte canônica do plugin fica em `plugins/`. Cada subpasta representa um plugin
independente, com manifesto, skills, referências e templates de saída.

## Estrutura

```text
.
├── README.md
├── examples/
│   └── query_01.sql
├── plugins/
│   └── feature-group-designer/
│       ├── plugin.json
│       └── skills/
│           └── feature-group-designer/
│               ├── SKILL.md
│               ├── references/
│               │   ├── decomposition-method.md
│               │   ├── contract-guide.md
│               │   └── deliverables.md
│               └── assets/
│                   └── feature-group-contract.yaml
```

## Plugin atual

O plugin incluído neste repositório é `feature-group-designer`.

Ele recebe uma query SQL e ajuda a:

- inventorizar colunas e funções;
- identificar entidade, chave e grão;
- rastrear joins, filtros, agregações e janelas;
- separar feature, target, identificador e controle;
- propor Feature Groups reutilizáveis;
- documentar contratos e riscos de point-in-time.

## Como usar

1. Registre ou instale a pasta `plugins/feature-group-designer/` como um Agent Plugin.
2. Use a skill a partir do manifesto do plugin.
3. Forneça a query SQL e escolha o modo de trabalho: Analyze, Design, Create ou Review.

## Princípios

- o núcleo do repositório deve ser portátil e vendor-neutral;
- o plugin deve ser independente de editor;
- o grão, a semântica temporal e a governança devem ser explícitos;
- exemplos são documentação e testes estruturais, não execução em produção;
- a definição canônica do plugin permanece em `plugins/`.

## Contribuição

Para adicionar uma nova skill:

1. criar uma nova pasta em `plugins/<plugin-name>/`;
2. manter `plugin.json` na raiz do plugin;
3. criar `skills/<skill-name>/SKILL.md`;
4. adicionar `references/` e `assets/` quando necessário;
5. manter o README e a documentação alinhados ao comportamento real da skill.
