# meta-vv-publicos

**Públicos de Video View na Meta Ads**

Scripts standalone que criam **públicos de engajamento por retenção de vídeo** na
Meta, prontos pra usar como público quente ou seed de lookalike.

Só precisa de Python 3.9+ (sem instalar biblioteca nenhuma).

## Que públicos dá pra criar?

Duas fontes de vídeo, mesma regra de engajamento:

- **Orgânico do Instagram** (`vv_publicos.py`) — varre os vídeos do seu perfil,
  qualifica pelo corte de views (ex.: 100k+) e agrupa por ano.
- **De uma campanha específica** (`vv_campanha.py`) — pega os vídeos direto dos
  criativos dos anúncios de uma campanha.

Em ambos, um público por **retenção**, com preenchimento retroativo (`prefill`) e
janela configurável (`--janela`, padrão 365 dias — o máximo da Meta). As
retenções disponíveis:

| Retenção | Quem entra | Pra que serve |
|---|---|---|
| `3s` | Assistiu 3 segundos | Topo amplo: quase todo mundo que o vídeo alcançou. Bom pra escala e exclusões |
| `15s` | ThruPlay (15s ou o vídeo inteiro) | Meio-termo: filtra quem só passou o dedo |
| `25` | Assistiu 25% do vídeo | Morno: passou do hook |
| `50` | Assistiu metade do vídeo | Interessado: ficou além da curiosidade |
| `75` | Assistiu 75% do vídeo | Público quente: prestou atenção de verdade |
| `95` | Assistiu até o fim | O mais quente: fã. Ótima seed de lookalike |

Exemplos de públicos que saem:

- `VIDEOVIEW 75% | IG 2026 | 100k+ views` (orgânico, por ano × corte de views)
- `VIDEOVIEW ThruPlay | IG 2026 | 50k+ views | 90D` (orgânico, só quem assistiu nos últimos 90 dias)
- `VIDEOVIEW 75% | ADS CAMPANHA DE AQUECIMENTO | 90D` (dos anúncios de uma campanha)

O nome carrega o recorte inteiro, então dá pra identificar o público no
Gerenciador sem abrir a descrição — e a descrição ainda registra quantos vídeos
entraram, de quais perfis/campanha e a janela.

## Setup (5 min)

1. Tenha um **token da Meta** com as permissões:
   `ads_management`, `business_management`, `pages_show_list`,
   `pages_read_engagement`, `instagram_basic`, `instagram_manage_insights`.
   (Recomendado: system user no Business Manager com acesso às páginas e às
   contas de anúncio. Pra testar, o Graph API Explorer também serve.)
2. O Instagram precisa ser **conta profissional vinculada a uma página** de
   Facebook que o token administra.
3. Copie `.env.example` para `.env` e preencha o `META_TOKEN`.

## Uso

Sempre comece com `--dry-run` pra ver o que seria criado. Saiu como esperado?
Rode sem o `--dry-run`.

### Orgânico do Instagram (`vv_publicos.py`)

```bash
python vv_publicos.py --contas act_SEU_ID --retencoes 75,95 --anos 2025,2026 --min-views 100000 --dry-run
```

| Flag | O que faz | Default |
|---|---|---|
| `--contas` | contas de anúncio (`act_...`), separadas por vírgula | obrigatório |
| `--paginas` | nomes de página FB separados por `;` | todas as páginas com IG do token |
| `--retencoes` | `3s`, `15s` (ThruPlay), `25`, `50`, `75`, `95` | `75,95` |
| `--anos` | anos dos vídeos, separados por vírgula | ano atual |
| `--min-views` | corte mínimo de views | `100000` |
| `--janela` | retenção em dias (1–365) | `365` |
| `--dry-run` | simula sem criar nada | — |

### De uma campanha específica (`vv_campanha.py`)

```bash
python vv_campanha.py --contas act_SEU_ID --campanha "CAMPANHA DE AQUECIMENTO" --retencoes 75 --janela 90 --dry-run
```

| Flag | O que faz | Default |
|---|---|---|
| `--contas` | contas de anúncio (`act_...`), separadas por vírgula | obrigatório |
| `--campanha` | **id numérico** ou trecho do nome (a busca ignora colchetes) | obrigatório |
| `--retencoes` | `3s`, `15s` (ThruPlay), `25`, `50`, `75`, `95` | `75` |
| `--janela` | retenção em dias (1–365) | `365` |
| `--dry-run` | simula sem criar nada | — |

Proteções: se o trecho casar com mais de 5 campanhas, ele lista e pede um termo
mais específico em vez de criar em massa; anúncios "usar publicação existente" do
IG não têm `video_id` — a mídia IG entra na regra com aviso (se não for vídeo, a
Meta ignora). Este script só precisa do `META_TOKEN` (dispensa as permissões de
insights do Instagram).

## O que eles fazem por baixo

- **Máximo 100 vídeos por público** (limite da Meta — erro 2654 acima disso);
  quando há mais qualificados, entra o top 100 e a descrição avisa.
- Não duplica: se um público com o mesmo nome já existe na conta, pula.
- Rate limit da Meta é tratado com espera e retry automáticos.
- No orgânico, a varredura fica em cache (`vv_cache.json` na pasta) — pode
  interromper e retomar sem re-analisar vídeo; rodadas novas só varrem o que falta.

## Limitações

- No orgânico, só vídeos do **Instagram** (feed/reels de conta profissional) —
  vídeos de página do Facebook não entram. No de campanha, entra qualquer vídeo
  que esteja nos criativos dos anúncios.
- A API de insights não retorna views de vídeos muito antigos em alguns perfis;
  o corte por ano ajuda a manter o escopo recente.

---

*Criado por Igor Poggianella de Oliveira (contato.ifb@gmail.com) em 20/08/2026.*
