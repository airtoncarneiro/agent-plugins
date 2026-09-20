# Agent Plugins Universal

Coleção de plugins portáteis e reutilizáveis para agentes de IA. O repositório
mantém os plugins independentes de editor, fornecedor ou runtime específico,
seguindo o padrão [Agent Plugins](https://agent-plugins.org/).

## Objetivo

Disponibilizar plugins versionados, com manifestos, skills e materiais de apoio
que possam ser instalados e reutilizados em diferentes ambientes de agentes.

O catálogo dos plugins disponíveis está em [`plugins/README.md`](plugins/README.md).

## Estrutura

```text
.
├── README.md
├── plugins/
│   ├── README.md
│   └── <plugin-name>/
│       ├── plugin.json
│       └── skills/
```

Cada subpasta de `plugins/` é um plugin independente. Seu `plugin.json` é o
manifesto, e as subpastas `skills/`, `references/` e `assets/` contêm os
recursos necessários para seu funcionamento.

## Instalação e uso

1. Clone ou baixe este repositório:

   ```bash
   git clone https://github.com/airtoncarneiro/agent-plugins.git
   ```

2. Escolha um plugin no [catálogo](plugins/README.md).
3. Instale ou registre a pasta do plugin no runtime de agentes utilizado,
   conforme o procedimento desse runtime.
4. Consulte o `SKILL.md` e as referências do plugin para conhecer suas entradas,
   modos de uso e entregáveis.

Os plugins são definidos de forma independente. Portanto, a instalação pode
ser feita para um plugin específico, sem exigir a instalação de toda a coleção.

## Contribuição

Para adicionar um plugin:

1. crie uma pasta em `plugins/<plugin-name>/`;
2. mantenha um `plugin.json` válido na raiz do plugin;
3. adicione suas skills em `skills/<skill-name>/SKILL.md`;
4. inclua `references/` e `assets/` quando necessário;
5. atualize o [catálogo de plugins](plugins/README.md).

Mantenha a documentação alinhada ao comportamento real de cada plugin e evite
acoplar os recursos a um editor ou fornecedor específico sem necessidade.
