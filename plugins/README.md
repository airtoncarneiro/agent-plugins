# Plugins

Catálogo dos plugins disponíveis neste repositório. Cada plugin é independente
e possui seu próprio manifesto e documentação de uso.

## Plugins disponíveis

### [`feature-group-designer`](feature-group-designer/)

Ajuda a transformar queries SQL de cientistas de dados em especificações de
Feature Groups portáteis e reutilizáveis. Analisa entidades, chaves, grãos,
joins, filtros, agregações e janelas; distingue features, targets,
identificadores e controles; e documenta contratos, riscos temporais e
considerações de point-in-time.

Para instalar ou usar este plugin, consulte o manifesto em
[`feature-group-designer/plugin.json`](feature-group-designer/plugin.json) e a
skill em
[`feature-group-designer/skills/feature-group-designer/SKILL.md`](feature-group-designer/skills/feature-group-designer/SKILL.md).

## Como adicionar um plugin

Crie uma nova pasta diretamente em `plugins/`, contendo ao menos:

```text
<plugin-name>/
├── plugin.json
└── skills/
    └── <skill-name>/
        └── SKILL.md
```

Depois, adicione aqui uma breve descrição e links para o manifesto e a
documentação principal do novo plugin.
