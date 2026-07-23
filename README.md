# ROMI D1250 XR — Simulador de Montagem/Explosão

Projeto Babylon.js + Vite para visualização VR/AR de montagem mecânica com efeito de explosão, no mesmo padrão do SimMorsa XR e do bomba-vr.

Modelo: **Centro de Usinagem CNC ROMI D1250** (3 eixos, vertical).

## 1. Modelo

O arquivo `maquina.glb` **já está incluído** em `public/models/`, com 12 componentes identificados:

`BASE_AXIS`, `X_AXIS`, `Y_AXIS`, `Z_AXIS`, `SPINDLE`, `TOOLMAGAZINE`, `POCKET_TC`, `TOOL_CHANGER`, `CONVEYOR`, `ENCLOSURE`, `DOOR_LEFT`, `DOOR_RIGHT`.

> ✅ **Modelo já comprimido com Draco**: reduzido de 61 MB para **~3,3 MB** (compressão de geometria, sem perda visual perceptível). Isso resolve travamentos de carregamento em celular/dados móveis, que era o gargalo original.

## 2. Correção de orientação do modelo

O modelo veio com a frente apontando para o piso, então apliquei uma rotação de correção em `CONFIG.correcaoRotacao` (em `src/main.js`):

```javascript
correcaoRotacao: { x: -Math.PI / 2, y: 0, z: 0 }
```

Esse é um palpite inicial (90° no eixo X). **Teste no navegador** e, se a máquina não ficar em pé corretamente, tente estes valores até acertar:

| Sintoma | Tente |
|---|---|
| Ficou de cabeça para baixo | trocar `x: -Math.PI / 2` por `x: Math.PI / 2` |
| Ficou deitada de lado | mover a correção para `y` em vez de `x` |
| Ficou girada 180° (de costas) | somar `y: Math.PI` |

Depois de ajustar, salve e recarregue — a câmera e o cenário se reenquadram automaticamente com base no novo tamanho/posição da máquina.

## 3. Cenário — Laboratório SENAI São Carlos

O plano de piso simples foi substituído por um ambiente de oficina/laboratório mais realista, todo gerado por código (sem depender de texturas externas):

- **Piso industrial** em epóxi cinza claro, com textura de grade e leve ruído de manchas, dimensionado automaticamente em proporção ao tamanho da máquina carregada.
- **Faixa de segurança amarela** no piso, delimitando a área ao redor da máquina (como as demarcações reais de zona de risco de CNC).
- **Paredes** (fundo + duas laterais) em tom claro, com padrão de painéis e rodapé industrial.
- **Painel institucional SENAI** na parede de fundo, com o nome do CFP Antonio Adolphe Lobbe — São Carlos/SP.
- **Luminárias de teto** (painéis emissivos) simulando iluminação industrial suspensa.

Todo o cenário é reconstruído dinamicamente a partir do tamanho real do modelo carregado (`construirCenario(bbox)`), então funciona com qualquer escala de máquina sem precisar editar números manualmente.

Para trocar o texto do painel institucional, edite a função `criarPainelSenai()` em `src/main.js`.

## 4. Iluminação

A cena usa iluminação reforçada para ficar mais clara sem alterar as cores originais dos materiais do modelo:

- Luz hemisférica + luz direcional principal + luz de preenchimento (fill light), todas com cor branca neutra.
- Ambiente de reflexo (IBL) neutro via `environmentTexture`, que melhora a leitura de materiais PBR (metal/plástico) sem tingir cores.
- Sombras suavizadas (`shadowGenerator.darkness = 0.35`).
- Exposição (`imageProcessingConfiguration.exposure`) levemente aumentada.

Se quiser ajustar o brilho, mexa em `hemi.intensity`, `dir.intensity`, `fill.intensity` e `scene.imageProcessingConfiguration.exposure` no topo do `main.js`.

## 5. Manual do Operador (PDF sobreposto)

O botão **"📄 Manual do Operador"** no HUD abre um painel sobreposto com o PDF, sem sair da cena 3D (fecha clicando fora, no ✕, ou apertando Esc).

Pra ativar, coloque o arquivo do manual em:

```
public/manual/manual-operador.pdf
```

Se o arquivo não existir, o app mostra uma mensagem avisando o caminho esperado, em vez de dar erro silencioso. Pra usar outro nome de arquivo, edite `CONFIG.manualPdf` em `src/main.js`.

## 6. Instalar dependências

```bash
npm install
```

## 7. Rodar em desenvolvimento

```bash
npm run dev
```

Abra o endereço mostrado no terminal (ex: `http://localhost:5173`). Para testar no navegador do Quest na mesma rede Wi-Fi, use o endereço com o IP local (o Vite mostra as duas opções com `host: true` já configurado).

> **WebXR exige HTTPS** para funcionar de verdade (exceto em `localhost`). Para testar no headset físico, veja a seção de deploy abaixo.

## 8. Ajustar a configuração ao seu modelo

Abra `src/main.js` e edite o objeto `CONFIG` no topo do arquivo:

```javascript
const CONFIG = {
  modelFile: "maquina.glb",
  distanciaExplosaoPadrao: 1.4,
  pecasPequenasKeywords: ["parafuso", "porca", "arruela", "pino"],
  multiplicadorPecaPequena: 2.2,
  correcaoRotacao: { x: -Math.PI / 2, y: 0, z: 0 }
};
```

Para descobrir os nomes exatos das peças do seu modelo, abra o console do navegador (F12) após carregar — o app imprime a lista completa (`console.log("Nomes das peças:", ...)`).

## 9. Build de produção

```bash
npm run build
```

Gera a pasta `dist/` pronta para publicar.

## 10. Deploy

**GitHub Pages via GitHub Actions (recomendado — já configurado neste projeto):**

O arquivo `.github/workflows/deploy.yml` já está pronto. Ele builda o projeto (`npm run build`) e publica a pasta `dist/` automaticamente **toda vez que você der `git push` na branch `main`**. Você não precisa mais rodar `npm run build` manualmente nem enviar `dist/` por fora.

Passo único de configuração no repositório (só precisa fazer uma vez):

1. No GitHub, entre no repositório → **Settings** → **Pages**
2. Em "Build and deployment" → **Source**, selecione **GitHub Actions** (não "Deploy from a branch")
3. Pronto — a partir do próximo `git push`, o Actions builda e publica sozinho

> Se antes você já tinha publicado os arquivos-fonte direto (sem build), é esse o motivo do erro `Failed to resolve module specifier`. Com o workflow automático isso não acontece mais, porque o Vite empacota tudo antes de publicar.

**Cloudflare Pages** (alternativa, recomendada se depois for integrar sensores/ESP32 via MQTT/WebSocket, para evitar bloqueio de Mixed Content):

```bash
npm install -g wrangler
npm run build
wrangler pages deploy dist
```

## 11. Personagem animado (referência de escala)

Um operador animado (modelo `HVGirl.glb`, dançando "Samba" em loop) fica parado ao lado da máquina, carregado do CDN público de assets do Babylon.js. Serve como referência visual de escala humana — ajuda o aluno a perceber o tamanho real da máquina.

O carregamento é assíncrono e não trava a aplicação se a CDN externa falhar (erro fica só no console). Ajuste posição/tamanho ao vivo, sem rebuild:

```js
moverPersonagem(x, y, z)
escalarPersonagem(0.07)   // ajusta o tamanho
```

Se quiser trocar por outra animação do mesmo modelo (ele tem várias: Idle, Walking, etc.), troque `"Samba"` por outro nome em `adicionarPersonagemAnimado()` no `main.js`. Se quiser remover o personagem, comente a linha `adicionarPersonagemAnimado(bbox);` em `carregarModelo()`.

## 12. Testar RV e RA no Quest

1. Publique o build (HTTPS obrigatório).
2. No navegador do Quest, acesse a URL publicada.
3. No HUD, toque em **"Modo RV"** para entrar em realidade virtual imersiva, ou **"Modo RA"** para entrar em realidade aumentada (passthrough, se o dispositivo suportar).
4. Os botões ficam desabilitados automaticamente se o navegador/dispositivo não suportar o modo correspondente.
5. Use os gatilhos/thumbstick dos controles, ou as mãos (hand tracking), para interagir com as peças.

## Estrutura de pastas

```
maquina-xr/
├── index.html            → HUD com botões de modo, slider de explosão e botões RV/RA
├── package.json
├── vite.config.js
├── public/
│   └── models/
│       └── maquina.glb
└── src/
    ├── main.js           → carregamento, correção de orientação, cenário, explosão, XR
    └── style.css          → estilo do HUD
```

## Otimização (já aplicada)

O `maquina.glb` deste projeto **já foi comprimido com Draco** (61 MB → ~3,3 MB). Se no futuro trocar o modelo por outra exportação do SolidWorks, repita o processo:

```bash
npm install -g @gltf-transform/cli
gltf-transform draco public/models/maquina-novo.glb public/models/maquina.glb
```

O Babylon.js já sabe descomprimir Draco automaticamente (não precisa mudar nada no `main.js`) — ele baixa o decodificador direto de um CDN na primeira vez que carrega o modelo.

## Modos disponíveis

- **Livre**: arraste qualquer peça manualmente para inspecioná-la.
- **Explodido**: controla o afastamento de todas as peças com o slider (0 a 100%).
- **Modo RV / Modo RA**: botões dedicados no HUD para entrar direto em realidade virtual ou aumentada.
#   m o e n d a  
 