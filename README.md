# personal-page

## Clonando o repositório

Este projeto usa um submodule para o tema. Ao clonar, use uma das opções:

**Opção 1: Clonar com submodules (recomendado)**
```bash
git clone --recursive https://github.com/maysaclaudino/maysaclaudino.github.io.git
```

**Opção 2: Se já clonou sem submodules**
```bash
git submodule update --init --recursive
```

## Para rodar localmente

```bash
hugo server --disableFastRender
```

Acesse em http://localhost:1313

## Adicionar uma nova página

Para criar uma nova página, use o comando:

```bash
hugo new content <caminho-do-arquivo>
```

**Exemplos:**

```bash
# Criar uma página na seção TCC
hugo new content content/tcc/nome-da-pagina.md

# Criar uma página na raiz do content
hugo new content content/nova-pagina.md
```

O comando cria um arquivo markdown com o front matter básico. Depois, edite o arquivo para adicionar seu conteúdo.
