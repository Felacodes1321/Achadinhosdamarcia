# Contexto: site "achadinhos da Márcia"

Leia este arquivo antes de mexer no site. Ele vem junto com o arquivo `index.html`.

## O que é

É um site vitrine para celular (mobile first), no estilo "link na bio", com os produtos da Shopee que a Márcia indica usando links de afiliada. Ela divulga os produtos no Instagram, TikTok e WhatsApp.

O site é **só leitura**: não tem botão de adicionar nem de editar. Quem atualiza é a IA, sempre que a Márcia manda um produto novo.

## Como a Márcia manda cada produto

Ela manda:

1. **O link de afiliada**, no formato `https://s.shopee.com.br/XXXX`.
2. **Um print da página do produto**, de onde saem o nome, o preço e o preço antigo riscado.
3. **A foto do produto**, uma imagem separada.
4. Às vezes, o nome do produto escrito na mensagem.

A IA normalmente **não consegue abrir links da Shopee**, porque o acesso é bloqueado. Por isso as informações vêm sempre dos prints. Nunca invente preço, nome ou opinião.

## Estrutura do arquivo HTML

É um único arquivo, sem dependências, com o CSS e o JavaScript embutidos. As únicas fontes externas são as do Google Fonts.

- `<style id="app-css">`: todo o visual.
- `<template id="t-corpo">`: a estrutura da página (topo, busca, filtros, lista e rodapé).
- `<script type="application/json" id="dados">`: **os dados do site. Só isso muda quando entra um produto novo.**
- `<script id="app-js">`: monta a página a partir dos dados. Também cuida da busca por nome ou número, dos filtros por categoria e do atalho `#12`, que rola a página até o produto nº 12 e o destaca.

### Formato dos dados

```json
{
  "perfil": {
    "foto": "data:image/jpeg;base64,...",
    "bio": "Achadinhos da Shopee que valem cada real. Viu no vídeo? Procure pelo número e compre direto.",
    "insta": "",
    "tiktok": "",
    "whats": ""
  },
  "produtos": [
    {
      "n": 1,
      "nome": "Kit 3 cabides organizadores de calça",
      "preco": "64,90",
      "antigo": "87,90",
      "link": "https://s.shopee.com.br/6AlKYpVTQj",
      "foto": "data:image/jpeg;base64,...",
      "frase": "",
      "cat": "Organização",
      "criado": "2026-09-26"
    }
  ],
  "proximo": 3
}
```

Regras para os campos:

- **`n`** é o número fixo do produto. Os vídeos citam esse número ("o link é o achadinho nº 12"), então ele **nunca é reaproveitado nem renumerado**. Todo produto novo recebe o valor de `proximo`, e depois `proximo` aumenta em 1. Se um produto for apagado, o número dele simplesmente some.
- **`preco`** e **`antigo`** são texto sem "R$", com vírgula decimal (`"59,90"`). O campo `antigo` é opcional e aparece riscado.
- **`nome`** deve ser curto, até uns 45 caracteres, porque o cartão corta depois de 3 linhas. Encurte o título enorme da Shopee.
- **`foto`** é uma imagem embutida em base64. Recorte no centro em formato quadrado, reduza para 560×560 e salve em JPEG com qualidade 78, o que dá uns 50 a 70 KB. O arquivo inteiro precisa ficar **abaixo de 16 MB**. Se `foto` estiver vazio, aparece o ícone da casinha.
- **`oferta`** e **`ate`** (opcionais) servem para oferta relâmpago. `oferta` é o preço promocional e `ate` é quando a oferta acaba, no formato `"2026-09-29T00:00:00-03:00"`. Enquanto a oferta vale, o cartão mostra `oferta` na etiqueta, `preco` riscado e um contador regressivo. Quando acaba, volta sozinho para `preco`. As ofertas relâmpago da Shopee costumam acabar na virada de hora; calcule pelo "termina em" do print.
- **`frase`** é a opinião da Márcia. **Só preencha se ela mandar.** Não escreva frases por ela.
- **`cat`** é a categoria. As atuais são "Cozinha", "Organização", "Banheiro", "Limpeza", "Dia a dia" e "Saúde". Reaproveite as existentes, porque os filtros só aparecem quando existem 2 ou mais categorias.
- **`insta`, `tiktok` e `whats`** são os links das redes. Os botões só aparecem quando o link está preenchido. Por enquanto estão vazios, então é preciso pedir os links para a Márcia.
- Dentro do JSON, troque todo `<` por `\u003c` para não quebrar o `<script>`.

O produto mais novo aparece primeiro, porque a lista é ordenada por `n` do maior para o menor.

### Exemplo em Python para adicionar um produto

```python
import json, re, base64, io
from PIL import Image

html = open("index.html", encoding="utf-8").read()
m = re.search(r'(<script type="application/json" id="dados">)(.*?)(</script>)', html, re.S)
d = json.loads(m.group(2))

im = Image.open("foto_produto.png").convert("RGB")
w, h = im.size; s = min(w, h)
im = im.crop(((w-s)//2, (h-s)//2, (w-s)//2+s, (h-s)//2+s)).resize((560, 560), Image.LANCZOS)
b = io.BytesIO(); im.save(b, "JPEG", quality=78)
foto = "data:image/jpeg;base64," + base64.b64encode(b.getvalue()).decode()

d["produtos"].append({"n": d["proximo"], "nome": "...", "preco": "0,00", "antigo": "",
                      "link": "https://s.shopee.com.br/...", "foto": foto, "frase": "",
                      "cat": "Cozinha", "criado": "AAAA-MM-DD"})
d["proximo"] += 1

novo = json.dumps(d, ensure_ascii=False).replace("<", "\\u003c")
html = html[:m.start(2)] + novo + html[m.end(2):]
open("index.html", "w", encoding="utf-8").write(html)
```

## Identidade visual (não alterar sem ela pedir)

**Cores:**

- Verde aprovado `#2E6B4F`: cor principal, usada no topo e nos botões.
- Amarelo etiqueta `#F5C542`: só em preço e destaques.
- Branco azulejo `#F7FAF8`: fundo.
- Grafite `#1D2B26`: texto.
- Vermelho `#C62F3B`: só no selo "não compre", nunca em oferta.

**Proporção:** 60% fundo claro, 30% verde e 10% amarelo.

**Letras:**

- **Young Serif** nos títulos e nomes dos produtos.
- **Atkinson Hyperlegible** nos textos e preços. Essa fonte é fácil de ler em tela pequena, o que combina com o público, que inclui gente que usa óculos de leitura.

**Marca:** "achadinhos da" em amarelo, com "Márcia" em serifa branca. A foto dela fica num círculo, com o selo da casinha com o check amarelo. Ela também usa "Tia Márcia" nos selos do kit, mas o site usa "Márcia", como na foto de perfil.

**Preço:** faixa amarela larga e reta logo abaixo da foto, com o preço grande (de ~~antigo~~ por **atual**) e um selo "% OFF" calculado sozinho quando o desconto passa de 5%. Foi feito assim para o público de 50 a 65 anos ler o preço sem esforço; não voltar para a etiqueta pequena inclinada sobre a foto.

**Compra:** botão verde "Comprar na Shopee" com a frase "Abre a Shopee já no produto" embaixo, e o quadro "Como comprar" (3 passos) abaixo da busca. Mantenha os dois.

**Rodapé:** traz o aviso de link de afiliada. Mantenha esse aviso.

## Produtos atuais

| nº | Produto | Preço | De | Categoria | Link |
|---|---|---|---|---|---|
| 1 | Kit 3 cabides organizadores de calça | 64,90 | 87,90 | Organização | https://s.shopee.com.br/6AlKYpVTQj |
| 2 | Garrafa térmica Prisma 950 ml com cabo de madeira | 59,90 | 84,90 | Cozinha | https://s.shopee.com.br/5q8UAmnTGz |
| 3 | Tapete de banheiro antiderrapante absorvente | 13,99 | 50,00 | Banheiro | https://s.shopee.com.br/9zy4PmHjWq |
| 4 | Escova de limpeza elétrica giratória | 69,90 | 199,00 | Limpeza | https://s.shopee.com.br/7KxJRGpxpy |
| 5 | Organizador de calcinha e cueca 6 divisórias | 44,80 (oferta 25,08 até 29/09 00h) | | Organização | https://s.shopee.com.br/1BMhzjBDq2 |
| 6 | Garrafa térmica 1 L lisa com cabo de madeira | 59,88 | 150,00 | Cozinha | https://s.shopee.com.br/905fo6Xdpq |
| 7 | Kit 5 potes de vidro hermético tampa bambu | 79,90 | 199,90 | Cozinha | https://s.shopee.com.br/qjy69Onru |
| 8 | Porta temperos giratório com 9 potes de vidro | 69,80 (oferta 37,79 até 03/10 00h) | | Cozinha | https://s.shopee.com.br/6q1BG6MnRf |
| 9 | Guarda-chuva automático resistente ao vento | 43,43 | 50,49 | Dia a dia | https://s.shopee.com.br/8AWbTEHmYF |
| 10 | Mini triturador elétrico recarregável | 29,90 | | Cozinha | https://s.shopee.com.br/1AtlzD6G5 |
| 11 | Lixeira automática com sensor 16 L | 65,86 | 200,00 | Limpeza | https://s.shopee.com.br/8AWbWLRkJU |
| 12 | Porta ovos rolante 4 andares para geladeira | 13,08 | 45,00 | Organização | https://s.shopee.com.br/3B7xqBGycr |
| 13 | Caixa organizadora de geladeira empilhável | 70,00 (oferta 37,99 até 07/10 00h) | | Organização | https://s.shopee.com.br/W7CfoMWrQ |
| 14 | Fatiador de legumes multifuncional 16 em 1 | 26,99 | 100,00 | Cozinha | https://s.shopee.com.br/gQcsx9cTx |
| 15 | Mop giratório com balde centrífuga e 2 refis | 59,90 | 121,90 | Limpeza | https://s.shopee.com.br/1LgMrzkt5c |
| 16 | Moedor elétrico de café e especiarias | 69,99 (oferta 35,69 até 09/10 00h) | | Cozinha | https://s.shopee.com.br/3LRRFhcSzu |
| 17 | Kit limpeza de teclado e fone 7 em 1 | 19,99 | 35,99 | Limpeza | https://s.shopee.com.br/1BMwfmIdAf |
| 18 | Amolador elétrico de facas e tesouras | 32,00 (oferta 27,89 até 11/10 00h) |  | Cozinha | https://s.shopee.com.br/60SGQrRYVJ |
| 19 | Kit 6 formas de silicone para air fryer | 14,90 | 19,90 | Cozinha | https://s.shopee.com.br/5fpQ2GO1LB |
| 20 | Medidor de pressão de pulso recarregável | 39,89 |  | Saúde | https://s.shopee.com.br/9APICi1M8h |
| 21 | Tapete escorredor de louças | 23,90 | | Cozinha | https://s.shopee.com.br/70KrPAgmpB |
| 22 | Escova removedora de pelos 4 em 1 | 21,95 | | Limpeza | https://s.shopee.com.br/3VkzEq4ohz |
| 23 | Kit sacos a vácuo com bomba elétrica | 63,90 | 90,00 | Organização | https://s.shopee.com.br/1B74WoRWh |
| 24 | Esponja mágica de melamina kit 30 | 23,90 | 69,90 | Limpeza | https://s.shopee.com.br/50ZodbFjud |
| 25 | Sapateira de caixas transparentes empilháveis | 118,89 |  | Organização | https://s.shopee.com.br/6L5CE3o7kS |
| 26 | Tela protetora de ralo anti-inseto 5 m | 14,98 | 37,98 | Banheiro | https://s.shopee.com.br/1gJMfUjPym |
| 27 | Espuma limpa sofá a seco Zip Clean 300 ml | 17,18 | 39,00 | Limpeza | https://s.shopee.com.br/9063Jvw2N1 |
| 28 | Percarbonato tira manchas para roupas | 17,90 | 29,90 | Limpeza | https://s.shopee.com.br/8AWwKOzD3s |
| 29 | Escova elétrica giratória 5 em 1 sem fio | 29,99 | 89,90 | Limpeza | https://s.shopee.com.br/80DW85zqOr |
| 30 | Mini processador elétrico sem fio 250 ml | 23,88 | 47,58 | Cozinha | https://s.shopee.com.br/8V9mj0xwNy |
| 31 | Mixer batedor elétrico para café e leite | 18,69 | 89,99 | Cozinha | https://s.shopee.com.br/8KqMWhyZix |
| 32 | Kit 3 quadros decorativos Amor e Família | 14,90 | 39,89 | Dia a dia | https://s.shopee.com.br/50ZuYdev74 |
| 33 | Arranjo 3 heras artificiais com vaso trançado | 17,99 | 35,00 | Dia a dia | https://s.shopee.com.br/5AtKkweHm7 |
| 34 | Jogo de lençol 400 fios com fronhas | 12,50 | 26,00 | Dia a dia | https://s.shopee.com.br/40hNMnij8u |
| 35 | Kit 10 petisqueiras barquinha | 19,91 | 119,90 | Cozinha | https://s.shopee.com.br/4B0nZ6i5nx |
| 36 | Kit 7 saquinhos organizadores de mala | 11,99 | 18,99 | Organização | https://s.shopee.com.br/4LKDlPhST0 |
| 37 | Torneira inox com chuveiro giratório 360° | 25,88 | 59,80 | Cozinha | https://s.shopee.com.br/4B0p0XhafD |
| 38 | Pastilha que limpa a máquina de lavar | 8,59 |  | Limpeza | https://s.shopee.com.br/50Zw04ePyO |
| 39 | Cozedor de ovos elétrico a vapor | 29,98 | 99,00 | Cozinha | https://s.shopee.com.br/5AtMCNdmdR |
| 40 | Centrífuga de salada 3 L | 38,90 | 89,90 | Cozinha | https://s.shopee.com.br/4fx5bSfgeM |
| 41 | Suporte de canto para shampoo e sabonete | 16,90 | 39,99 | Banheiro | https://s.shopee.com.br/4qGVnlf3JP |
| 42 | Faqueiro dourado 24 peças de aço inox | 49,90 | 119,80 | Cozinha | https://s.shopee.com.br/3B8Hoj2an4 |
| 43 | Espelho redondo Adnet com alça de couro | 22,89 |  | Dia a dia | https://s.shopee.com.br/30orcQ3E83 |
| 44 | Suporte de parede para celular e controle | 7,00 |  | Organização | https://s.shopee.com.br/3Vl8DL1K7A |
| 45 | Kit 5 utensílios de silicone para cozinha | 21,50 |  | Cozinha | https://s.shopee.com.br/3LRi121xS9 |
| 46 | Kit 2 abajures com cúpula para quarto e sala | 56,80 | 119,90 | Dia a dia | https://s.shopee.com.br/3qNybx03RG |
| 47 | Escova limpa ventilador e grades | 11,99 | 29,00 | Limpeza | https://s.shopee.com.br/1Lgg17Qq5I |
| 48 | Organizador com rodinhas 4 prateleiras | 34,99 |  | Organização | https://s.shopee.com.br/1W06DQQCkL |
| 49 | 6 tampas de silicone para panelas | 12,99 |  | Cozinha | https://s.shopee.com.br/1gJWPjPZPO |
| 50 | Organizador de pia 3 em 1 inox | 28,00 |  | Cozinha | https://s.shopee.com.br/1qcwc2Ow4R |
| 51 | Mochila térmica grande cooler | 36,95 |  | Dia a dia | https://s.shopee.com.br/20wMoLOIjU |
| 52 | Fita LED neon 5 m | 19,90 |  | Dia a dia | https://s.shopee.com.br/4B0rOLbVfm |
| 53 | Kit colcha cobre leito casal | 69,90 |  | Dia a dia | https://s.shopee.com.br/40hRC2c90l |
| 54 | Jogo 6 taças de vidro diamond | 36,99 | 59,99 | Cozinha | https://s.shopee.com.br/5AtOaBXhe0 |
| 55 | Fruteira de mesa em metal | 38,80 | 89,88 | Cozinha | https://s.shopee.com.br/50ZyNsYKyz |
| 56 | Ventilador luminária LED 6 pás | 44,99 |  | Dia a dia | https://s.shopee.com.br/4qGYBZYyJy |
| 57 | Pano absorvente anti óleo p/ cozinha | 8,99 |  | Limpeza | https://s.shopee.com.br/50aKN4eGRU |
| 58 | Varal de chão dobrável 3 andares | 79,90 | 168,99 | Limpeza | https://s.shopee.com.br/5AtkZNdd6X |
| 59 | Suqueira 3,5 L com torneira p/ geladeira | 20,55 | 80,00 | Cozinha | https://s.shopee.com.br/4fxTySfX7S |
| 60 | Umidificador difusor Egg com LED | 16,99 |  | Dia a dia | https://s.shopee.com.br/4qGuAletmV |
| 61 | Cesto de roupa suja de bambu 72 L dobrável | 55,99 | 99,76 | Organização | https://s.shopee.com.br/4LKdZqgnnQ |
| 62 | Cortina de 300 LEDs 3x3 m | 35,00 | 80,00 | Dia a dia | https://s.shopee.com.br/4LKdZs6nRa |
| 63 | Toalha de banho gigante 100% algodão | 19,90 | 44,90 | Banheiro | https://s.shopee.com.br/4Ve3mB6A6d |
| 64 | Chaleira elétrica jarra inox | 44,90 | 71,25 | Cozinha | https://s.shopee.com.br/4fxTyU5Wlg |
| 65 | Colmeia organizadora de gaveta (PP a G) | 8,90 |  | Organização | https://s.shopee.com.br/4qGuAn4tQj |
| 66 | Saco de malha p/ lavar sapato e viagem | 17,44 | 50,00 | Limpeza | https://s.shopee.com.br/50aKN64G5m |

O próximo número é **67**.

## Onde está publicado

A primeira versão foi publicada como artifact no claude.ai: https://claude.ai/artifact/KGMRdgVMb95fZ9SGUsEN1h

O site está preparado para a Vercel (veja o `README.md`). O endereço usado nas metatags, no `robots.txt` e no `sitemap.xml` é `https://achadinhos-da-marcia.vercel.app`. Se o domínio mudar, troque esse endereço nesses três arquivos.

Ao adicionar produtos, atualize também o `<lastmod>` do `sitemap.xml`.
