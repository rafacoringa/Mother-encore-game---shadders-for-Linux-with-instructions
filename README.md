# Mother-encore-game---shadders-for-Linux-with-instructions (vibe code alert lol)
i don't own the shadders, (i guess i take them from [here](https://github.com/crosire/reshade-shaders))

**shadder nostalgia (for comparassion, no shadder)**
![](shadder%20nostalgia%20(for%20comparassion,%20no%20shadder).png)

**shadder nostalgia (scanlines and nes colors - original shadder)**
![](shadder%20nostalgia%20(scanlines%20and%20nes%20colors%20-%20original%20shadder).png)

**shadder nostalgia (scanlines and no nes colors - fix)**
![](shadder%20nostalgia%20(scanlines%20and%20no%20nes%20colors%20-%20fix).png)

**shadder crt (bad native scanlines, don't use it)**
![](shadder%20crt%20(bad%20native%20scanlines,%20don't%20use%20it).png)

---


# Instructions in portuguese instructions: Mother Encore 4.0.1 Linux + filtro CRT/Nostalgia via gamescope

Criado por Rafa Coringa 2026-06-04 + IA copilot 5.5 deep think por vibe code rsrs
Bota tudo na pasta /home/rafa/Games que é sucesso

## Contexto

Você tinha o executável nativo Linux:

```bash
~/Games/MotherEncore4.0.1.x86_64
```

E uma pasta de shaders ReShade slim:

```bash
~/Games/reshade-shaders-slim/
```

Com shaders em:

```bash
~/Games/reshade-shaders-slim/Shaders/
```

Entre eles:

```text
CRT.fx
Nostalgia.fx
Monochrome.fx
Vibrance.fx
Technicolor2.fx
...
```

***

# 1. Descobrimos que o jogo é OpenGL/Godot

Você rodou:

```bash
ldd ./MotherEncore4.0.1.x86_64 | grep -Ei 'vulkan|GL|SDL|godot|unity'
```

E apareceu:

```text
libGL.so.1
libGLdispatch.so.0
libGLX.so.0
```

Depois no log do jogo apareceu:

```text
Godot Engine v3.6...
OpenGL ES 2.0 Renderer: Mesa Intel(R) HD Graphics 520
```

Conclusão:

```text
Mother Encore Linux nativo = Godot/OpenGL
```

Então **vkBasalt direto no jogo não era o melhor caminho**, porque o jogo não estava em Vulkan.

***

# 2. O caminho que funcionou: gamescope com ReShade nativo

O `gamescope` funcionou porque ele cria uma camada de composição Vulkan por cima do jogo OpenGL.

O comando-base que funcionou foi:

```bash
gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect Nostalgia.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
```

Ou, para CRT original:

```bash
gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect CRT.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
```

***

# 3. Pegadinha importante: gamescope não aceita caminho absoluto no `--reshade-effect`

Tentamos:

```bash
--reshade-effect ~/Games/reshade-shaders-slim/Shaders/CRT.fx
```

E deu erro, porque o gamescope procurou errado, tipo:

```text
~/.local/share/gamescope/reshade/Shaders//home/rafa/Games/...
```

Então o correto foi colocar os shaders onde o gamescope espera:

```bash
~/.local/share/gamescope/reshade/Shaders/
```

E chamar só pelo nome:

```bash
--reshade-effect CRT.fx
```

ou:

```bash
--reshade-effect Nostalgia.fx
```

***

# 4. Como preparar os shaders para o gamescope

Se você já tem os shaders em:

```bash
~/Games/reshade-shaders-slim/
```

rode:

```bash
mkdir -p ~/.local/share/gamescope/reshade/Shaders
mkdir -p ~/.local/share/gamescope/reshade/Textures

cp ~/Games/reshade-shaders-slim/Shaders/*.fx ~/.local/share/gamescope/reshade/Shaders/
cp ~/Games/reshade-shaders-slim/Shaders/*.fxh ~/.local/share/gamescope/reshade/Shaders/ 2>/dev/null
cp -r ~/Games/reshade-shaders-slim/Textures/* ~/.local/share/gamescope/reshade/Textures/ 2>/dev/null
```

Confere:

```bash
ls ~/.local/share/gamescope/reshade/Shaders | grep -Ei 'CRT|Nostalgia'
```

***

# 5. Se quiser baixar os shaders de novo

Caso queira refazer do zero:

```bash
cd ~/Games

git clone https://github.com/crosire/reshade-shaders.git reshade-shaders
```

Depois copie para o gamescope:

```bash
mkdir -p ~/.local/share/gamescope/reshade/Shaders
mkdir -p ~/.local/share/gamescope/reshade/Textures

cp ~/Games/reshade-shaders/Shaders/*.fx ~/.local/share/gamescope/reshade/Shaders/
cp ~/Games/reshade-shaders/Shaders/*.fxh ~/.local/share/gamescope/reshade/Shaders/ 2>/dev/null
cp -r ~/Games/reshade-shaders/Textures/* ~/.local/share/gamescope/reshade/Textures/ 2>/dev/null
```

***

# 6. O que funcionou e o que não funcionou

## `CRT.fx`

Funcionou, mas:

* ficou com blur;
* alguns detalhes finos ficaram meio “comidos”;
* letras tipo `B` ficaram ruins em alguns cantos;
* objetos pequenos, como a mesinha perto da escada, criaram linhas fantasmas.

Launcher CRT:

```bash
cat > ~/Games/mother-encore-4.0.1-filtro-CRT.sh <<'EOF'
#!/usr/bin/env bash

cd "$HOME/Games" || exit 1

unset DRI_PRIME

gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect CRT.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
EOF

chmod +x ~/Games/mother-encore-4.0.1-filtro-CRT.sh
```

Rodar:

```bash
~/Games/mother-encore-4.0.1-filtro-CRT.sh
```

***

## `Nostalgia.fx`

Funcionou melhor visualmente:

* linhas CRT ficaram boas;
* menos blur que `CRT.fx`;
* visual geral ficou legal;
* porém as cores ficaram com vibe NES/C64/retrô forte por causa da redução de paleta.

Comando que funcionou:

```bash
cd ~/Games

gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect Nostalgia.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
```

Launcher Nostalgia original:

```bash
cat > ~/Games/mother-encore-4.0.1-filtro-Nostalgia.sh <<'EOF'
#!/usr/bin/env bash

cd "$HOME/Games" || exit 1

unset DRI_PRIME

gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect Nostalgia.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
EOF

chmod +x ~/Games/mother-encore-4.0.1-filtro-Nostalgia.sh
```

***

# 7. Nostalgia sem cores NES

A solução foi copiar o shader:

```bash
cp ~/.local/share/gamescope/reshade/Shaders/Nostalgia.fx \
   ~/.local/share/gamescope/reshade/Shaders/NostalgiaNoNES.fx
```

Se o arquivo ficar vazio por erro de cópia, apague e recrie:

```bash
rm -f ~/.local/share/gamescope/reshade/Shaders/NostalgiaNoNES.fx

cp ~/.local/share/gamescope/reshade/Shaders/Nostalgia.fx \
   ~/.local/share/gamescope/reshade/Shaders/NostalgiaNoNES.fx
```

Abrir:

```bash
micro ~/.local/share/gamescope/reshade/Shaders/NostalgiaNoNES.fx
```

Procurar:

```text
Nostalgia_color_reduction
```

E mudar o valor padrão para `0`.

A ideia é deixar assim:

```hlsl
uniform int Nostalgia_color_reduction <
    ui_type = "combo";
    ui_label = "Color reduction type";
    ui_items =
        "None\0"
        "Palette\0"
> = 0;
```

E manter scanlines ligadas em:

```hlsl
uniform int Nostalgia_scanlines
```

Com valor:

```hlsl
> = 1;
```

ou, se preferir testar outro tipo:

```hlsl
> = 2;
```

Resumo da edição:

```text
Nostalgia_color_reduction = 0  # desliga paleta NES/C64/etc
Nostalgia_scanlines = 1        # mantém linhas CRT
```

***

# 8. Cópia dos shaders dentro de `~/Games`

Você quis também manter uma cópia dos shaders na pasta do jogo.

Criar estrutura:

```bash
mkdir -p ~/Games/reshade-gamescope/Shaders
mkdir -p ~/Games/reshade-gamescope/Textures
```

Copiar shaders atuais:

```bash
cp ~/.local/share/gamescope/reshade/Shaders/* ~/Games/reshade-gamescope/Shaders/ 2>/dev/null
cp -r ~/.local/share/gamescope/reshade/Textures/* ~/Games/reshade-gamescope/Textures/ 2>/dev/null
```

Conferir:

```bash
ls ~/Games/reshade-gamescope/Shaders | grep -i Nostalgia
```

***

# 9. Launcher final: NostalgiaNoNES

Este launcher usa a cópia editada em `~/Games/reshade-gamescope/Shaders/` e sincroniza para onde o gamescope procura antes de abrir o jogo.

```bash
cat > ~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh <<'EOF'
#!/usr/bin/env bash

cd "$HOME/Games" || exit 1

unset DRI_PRIME

# Garante que a versão editada do shader esteja onde o gamescope procura
mkdir -p "$HOME/.local/share/gamescope/reshade/Shaders"

cp "$HOME/Games/reshade-gamescope/Shaders/NostalgiaNoNES.fx" \
   "$HOME/.local/share/gamescope/reshade/Shaders/NostalgiaNoNES.fx"

gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect NostalgiaNoNES.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
EOF

chmod +x ~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh
```

Rodar:

```bash
~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh
```

***

# 10. Permissão de execução dos launchers

Se aparecer:

```text
exists but is not an executable file
```

corrija com:

```bash
chmod +x ~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh
```

Se criou com `sudo micro` e ficou dono root:

```bash
sudo chown rafa:rafa ~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh
chmod +x ~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh
```

***

# 11. Comandos curtos opcionais

Se quiser chamar por nome simples:

```bash
mkdir -p ~/.local/bin

ln -sf ~/Games/mother-encore-4.0.1-filtro-NostalgiaNoNES.sh ~/.local/bin/mother-nostalgia-nones
ln -sf ~/Games/mother-encore-4.0.1-filtro-Nostalgia.sh ~/.local/bin/mother-nostalgia
ln -sf ~/Games/mother-encore-4.0.1-filtro-CRT.sh ~/.local/bin/mother-crt

fish_add_path ~/.local/bin
```

Rodar:

```bash
mother-nostalgia-nones
```

ou:

```bash
mother-nostalgia
```

ou:

```bash
mother-crt
```

***

# Resultado final recomendado

O melhor equilíbrio até aqui foi:

```text
NostalgiaNoNES.fx
```

com:

```bash
gamescope -W 1600 -H 900 -w 1600 -h 900 -r 60 -S integer -F pixel \
  --reshade-effect NostalgiaNoNES.fx \
  --reshade-technique-idx 0 \
  -- ./MotherEncore4.0.1.x86_64
```

Porque:

```text
CRT.fx = CRT bonito, mas blur/ghosting em letras e objetos finos
Nostalgia.fx = linhas CRT boas, mas cores estilo NES/C64
NostalgiaNoNES.fx = linhas CRT boas, sem redução de paleta
```

Ou seja: **NostalgiaNoNES é o final boss vencido** kkk.
