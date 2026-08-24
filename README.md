# nas-ci

Workflow reutilizável de deploy no NAS (Synology `caixapreta`, 192.168.1.171).

Todo projeto do homelab chama [`deploy-nas.yml`](.github/workflows/deploy-nas.yml) em vez
de copiar 80 linhas de YAML — corrigir um bug aqui conserta o deploy de todos de uma vez.

```
push na main
   │
   ├─► GitHub builda a imagem  ──►  publica em ghcr.io/fnz-org/projeto
   │   (CPU do GitHub, não a do NAS)
   │
   └─► runner da org (dentro do NAS)  ──►  docker compose pull + up -d
```

O build sai do NAS de propósito: Synology tem CPU fraca, e compilar pandas/node lá
levaria minutos disputando processador com Jellyfin e companhia.

## Usar num projeto

```yaml
name: Deploy
on:
  push: { branches: [main] }
  workflow_dispatch: {}
jobs:
  deploy:
    uses: FNZ-Org/nas-ci/.github/workflows/deploy-nas.yml@main
    with:
      projeto: meuprojeto
      health-url: http://127.0.0.1:1234/health
    secrets: inherit
```

O `docker-compose.yml` do projeto precisa da imagem parametrizada:

```yaml
services:
  meuprojeto:
    image: ${MEUPROJETO_IMAGE:-ghcr.io/fnz-org/meuprojeto:latest}
    build: .          # só pra rodar local; o NAS nunca builda
```

Para rodar testes antes, acrescente um job e um `needs: testes`.

## Inputs

| Input | Obrigatório | Padrão | Para que serve |
|---|---|---|---|
| `projeto` | sim | — | nome do container, da stack e do pacote no GHCR |
| `contexto` | não | `.` | diretório do build |
| `health-url` | não | vazio | URL checada dentro do NAS após subir |
| `compose-file` | não | `docker-compose.yml` | caminho do compose |

## Pré-requisitos

- Runner self-hosted de **nível organização** no NAS, label `synology`
- Em **FNZ-Org → Settings → Actions → General → Access**: *Accessible from repositories
  in the organization*
