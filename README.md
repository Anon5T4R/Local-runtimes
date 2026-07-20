# Local-runtimes

Espelho dos **binários de terceiro** que os aplicativos da suíte Local embarcam nos seus
instaladores: ffmpeg, llama.cpp, whisper.cpp, pandoc, mpv e Stockfish.

**Os binários aqui são builds NÃO MODIFICADOS**, copiados dos projetos originais. Este repositório
não compila nada — ele existe só para que os artefatos continuem disponíveis.

## Por que este espelho existe

Os aplicativos baixam esses binários durante o build. Depender direto do upstream falhou de três
formas diferentes, todas observadas na prática:

1. **Bloqueio por origem** — o `johnvansickle.com` passou a responder **HTTP 415** para os runners
   do GitHub Actions (responde 200 normalmente de uma máquina doméstica: bloqueia faixa de
   datacenter). Derrubou um build inteiro em 2026-07-17.
2. **Limite por IP** — o CDN do Hugging Face responde **403** a download anônimo ao estourar o
   limite de taxa.
3. **Poda por idade** — e este é o pior, porque é silencioso e datado: alguns projetos apagam
   releases antigas, então uma versão "fixada" vira **404 sozinha**, sem ninguém mexer em nada.

Medição da retenção em 2026-07-18:

| Projeto | Releases mantidas | Mais antiga |
|---|---|---|
| `zhongfly/mpv-winbuild` | 45 | 2026-06-18 — **menos de um mês** |
| `BtbN/FFmpeg-Builds` | 38 | 2024-08-31 — as diárias são podadas; sobrevivem as de fim de mês |
| `ggml-org/llama.cpp`, `whisper.cpp`, `jgm/pandoc` | histórico completo | — |

## Como usar

Cada release deste repositório (`v1`, `v2`, …) é um **conjunto congelado**. Os scripts de build da
suíte apontam para um asset específico, com a versão e o `sha256` fixados no próprio script, e
**conferem o hash antes de extrair** — download que não bate não é instalado.

Os nomes dos arquivos preservam o nome do upstream, para a procedência ser legível sem consultar
nada. O [`MANIFEST.json`](MANIFEST.json) traz, para cada artefato: `sha256`, tamanho, URL de origem,
repositório e tag do upstream, licença e link do código-fonte correspondente.

Conferir um download:

```bash
sha256sum <arquivo>          # tem que bater com o MANIFEST.json
```

## Licenças e código-fonte

Cada projeto mantém a sua licença — este espelho não altera nada e não reivindica direito nenhum
sobre eles.

| Projeto | Licença | Código-fonte correspondente |
|---|---|---|
| ffmpeg (build GPL) | GPL-3.0-or-later | <https://github.com/FFmpeg/FFmpeg> — tag `n8.1.2` |
| mpv | GPL-2.0-or-later | <https://github.com/mpv-player/mpv> |
| pandoc | GPL-2.0-or-later | <https://github.com/jgm/pandoc> — tag `3.10` |
| llama.cpp | MIT | <https://github.com/ggml-org/llama.cpp> — tags `b10066`, `b9802` |
| whisper.cpp | MIT | <https://github.com/ggml-org/whisper.cpp> — tag `v1.9.1` |
| Stockfish | GPL-3.0-or-later | <https://github.com/official-stockfish/Stockfish> — tag `sf_18` |

**ffmpeg, mpv, pandoc e Stockfish são copyleft.** Como a redistribuição de binário GPL exige que o código-fonte
correspondente esteja disponível, o `MANIFEST.json` registra, para cada artefato, a **tag exata** do
fonte e a **build original** de onde o binário veio — e os projetos de build (BtbN, zhongfly)
publicam a receita completa de compilação.

Nenhum binário aqui foi modificado. Quem quiser reproduzi-los deve usar o fonte na tag indicada e a
receita do projeto de build correspondente.

O **Stockfish** é caso especial em dois pontos, e os dois estão registrados no `MANIFEST.json`:

1. As **redes NNUE** (`nn-c288c895ea92`, `nn-37f18f62d772`) não são arquivo separado — vão embutidas
   no executável, e são cobertas pela mesma GPL-3 do resto.
2. Os assets são **oficiais do próprio projeto** (não de um empacotador terceiro como BtbN ou
   zhongfly), então a build e o fonte saem da mesma tag `sf_18` — a procedência é mais curta que a
   do ffmpeg.

O app que o consome (**LocalChessPGN**) roda o Stockfish como **processo separado por UCI**, nunca
linkado.

## O que deliberadamente NÃO está espelhado

- **tessdata** (modelos de OCR) — a URL de origem já é um **commit git**, imutável por definição.
  Espelhar não acrescentaria garantia nenhuma.
- **Dart SDK** — ~600 MB por plataforma, e o arquivo do Google mantém todas as versões
  permanentemente. Custo alto, risco nulo.
- **rust-analyzer, CodeLLDB, js-debug, Python embeddable** — retenção longa; fixar a versão basta.
- **gopls** — instalado via `go install`, que já é verificado contra o checksum database público
  (`sum.golang.org`), uma garantia mais forte que a deste espelho.
