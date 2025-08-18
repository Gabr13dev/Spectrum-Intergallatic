# Intergalactic Player — README

Um “deck” retrô-futurista com:
- Visualizador estéreo (L/R) em 7 bandas por lado (63 → 16k).
- Pico (peak-hold), balance L/R e rótulos “L / R” no topo.
- Letreiro LED com nome da faixa.
- CD grande que **gira** quando toca.
- Botões **Play/Pause** (toggle) + **Stop**, **seek bar** e **knob de volume** analógico.
- Suporte a arquivo local via botão **LOAD DISC** (input oculto e estilizado).
- Web Audio API (analisador de espectro em tempo real).

---

## 1) Como rodar

1. Salve o arquivo como `index.html`.
2. Abra no navegador (Chrome/Edge/Firefox).  
   > Dica: se o navegador bloquear áudio por autoplay, clique **Play** após carregar um arquivo.
3. Clique em **LOAD DISC** e escolha um `.mp3`/`.wav` etc.

---

## 2) Controles

- **LOAD DISC**: abre o seletor de arquivo de áudio.
- **Play/Pause** (um único botão): inicia/pausa e troca o ícone automaticamente.
- **Stop**: para e volta para 00:00.
- **Seek bar**: arraste para avançar/voltar a música.  
  — Para “scrubbing” em tempo real, ver ajuste em [Opções do Seek](#opções-do-seek).
- **Knob de Volume**: arraste, use **setas** ou **scroll** para ajustar (0–100%).  
- **Barra de Balance** (abaixo do espectro): mostra o RMS L/R em tempo real (indicador central).

**Atalhos**
- **Space**: Play/Pause.

---

## 3) Estrutura do arquivo

```
index.html
└─ <style>           # Temas, layout, LEDs, botões, knob, seek etc.
└─ <body>
   ├─ .panel
   │  ├─ .divider / .dividerHz   # Barra central + “Hz” no pé
   │  ├─ #spectrum               # Matriz de LEDs (colunas/linhas)
   │  ├─ #balanceWrap            # Trilha + bolinha (balance)
   │  └─ .displayBar             # CD, letreiro, tempo, controles, seek, volume
   └─ <audio id="audio">         # Player invisível
└─ <script>         # Web Audio, construção do grid, lógica de UI/controles
```

---

## 4) Customização rápida

### 4.1 Cores e estilo (CSS vars)
No topo do `<style>`:

```css
:root{
  --accent: #ff9f43;          /* laranja (hover/ícones/knob) */
  --accent-glow: rgba(255,159,67,.55);

  --bg: #171a1f;              /* fundo do painel */
  --led-off: #2b3240;
  --led-on:  #e8eef8;         /* pontos brancos */
  --led-peak:#ff4d4d;         /* pico */
}
```

Troque `--accent` e `--accent-glow` para outras paletas (ex.: verde/neon, azul, roxo).

### 4.2 Tamanho do espectro

```css
:root{
  --cols: 14;         /* 7 L + 7 R */
  --rows: 13;         /* “andares” */
  --dots-per-row: 4;  /* LEDs por andar */
}
```

Mudar `--rows` altera a altura (respeite a lógica no JS, que escala pelos rows).

### 4.3 Parâmetros de áudio (JS)

No `<script>`:

```js
const FFT_SIZE  = 1024;  // resolução de frequência (512, 1024, 2048…)
const SMOOTHING = 0.55;  // suavização do Analyser (0..1, menor = mais nervoso)
let   DECAY     = 0.00;  // decaimento visual (0 = nervoso, >0 = mais suave)
const PEAK_HOLD = 13;    // frames que o “peak” fica segurado
```

- **Quer mais punch?** Reduza `SMOOTHING` (ex. `0.45`).  
- **Quer menos tremido?** Use `DECAY` ~`0.6–0.75`.  
- **Peak mais curto/rápido?** Diminua `PEAK_HOLD`.

### 4.4 Bandas e rótulos

```js
const BANDS = [63,160,400,1000,2500,6300,16000]; // centros
const EDGES = [45,90,250,630,1600,4000,10000,20000]; // fatias
```

Labels exibidos:

```js
lab.textContent = ['63','160','400','1k','2.5k','6.3k','16k'][(c)%7];
```

Ajuste conforme o layout que quiser (ex.: mais bandas, escalas diferentes).

### 4.5 Ícones e CD girando

- O CD usa classe `.spinning` quando o áudio está tocando:
  ```js
  cdIconEl.classList.toggle('spinning', playing);
  ```
- Para mudar a **velocidade** da rotação:
  ```css
  .cdIcon.spinning { animation: spin 1.2s linear infinite; } /* altere 1.2s */
  ```

### 4.6 L/R e “Hz” nos cantos

- **L/R grandes** no topo: `.sideLabel` (se estiver usando).  
- **“Hz” no pé do divider**: `.dividerHz` (ajuste `bottom:` para casar com seus `channelLabel`).

### 4.7 Knob de volume

- Tamanho:
  ```css
  :root{ --knob-size: 86px; }
  ```
- Arco ativo:
  ```css
  :root{ --knob-ang: 135deg; } /* valor inicial; no runtime o JS controla */
  ```
- Mapeamento 0–100% → 0–270° no script:
  ```js
  const ang = 270 * volume; // 0..270
  ```

### 4.8 Opções do Seek

- O seek já atualiza a posição quando solta o arraste (**change**).  
- Para “scrubbing” em tempo real, descomente esta parte:

```js
seekEl.addEventListener('input', ()=>{
  const dur = audioEl.duration||0;
  if (isFinite(dur) && dur>0){
    audioEl.currentTime = (Number(seekEl.value)/1000) * dur;  // <- real time
  }
});
```

---

## 5) Acessibilidade e UX

- **Botões** têm `title` e ícones Font Awesome.
- **Knob** possui `role="slider"` e `aria-valuenow/min/max`, responde a teclas.
- Ícones, cores e “glows” seguem contraste suficiente no tema escuro.

---

## 6) Compatibilidade

- Chrome, Edge e Firefox recentes.  
- Safari também funciona, mas pode exigir interação do usuário antes de iniciar áudio (política de autoplay).  
- Em alguns formatos (`.flac`), a compatibilidade depende do navegador.

---

## 7) Performance

- `FFT_SIZE` alto = mais bins (mais pesado). 1024 é um bom meio-termo.
- Evitamos recalcular DOM inteiro: atualizamos classes de cada “dot”.
- Se precisar otimizar ainda mais:
  - Reduza `--rows` e/ou `--dots-per-row`.
  - Limite o brilho/sombras dos LEDs (menos `box-shadow`).
  - Use `requestAnimationFrame` (já usado).

---

## 8) Pontos fáceis de personalizar

- **Cores**: `--accent`, `--led-on`, `--led-peak`.  
- **Estética do letreiro**: mude animação `scrollMarquee` (duração, easing).  
- **Glow no hover**: `.btnCtl:hover` e `.btn-disc:hover`.  
- **L/R no topo**: troque cor/tamanho em `.sideLabel`.  
- **“Hz”**: ajuste `bottom` de `.dividerHz` para alinhar com seus labels.

---

## 9) Ideias de “V2”

- **Knobs de Bass/Treble** com anel de pontos iluminados.
- **Modo mic** (usar `getUserMedia`) para reagir ao microfone.
- **Preset de temas** (laranja/azul/verde) com toggle em UI.
- **Animação por BPM** (CD gira mais rápido conforme energia nos agudos).
- **Botões “Next/Prev Track”** e fila de reprodução.
- **Modo “range sweep”** (barra horizontal central que percorre as bandas, como no trailer).
