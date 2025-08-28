# Bubble Mood

Projeto para visualização de bolhas distribuidas de forma orgânica, com cores e tamanhos representando a proporção de charts.

---

## Tecnologias Utilizadas

- **Vue 3** – Framework principal para criar interfaces.  
- **TypeScript** – Tipagem estática para maior segurança e produtividade no desenvolvimento.  
- **ECharts** – Biblioteca de gráficos, utilizada para renderizar os charts das bolhas.  
- **Floating Vue** – Para tooltips flutuantes de forma simples e elegante.  
- **Vite** – Ferramenta de build rápida e moderna.

---

## Desafios enfrentados e soluções adotadas

O maior desafio foi fazer os charts se distribuírem de forma **orgânica** na tela, sem que ficassem um em cima do outro.  
No começo fiquei pensando em como posicionar cada bolha pra não ter colisão. A solução foi colocar o maior item no centro e espalhar os outros ao redor, calculando o ângulo e a distância de cada um. Assim fica visualmente legal e não quebra o layout.

1 - Calcular o raio da bolha central.</br>
Solução: Criei uma função para calcular o raio da bolha central e das demais, que ajusta o tamanho proporcional ao valor total tanto da bolha central quanto das demais.

```js
function getRadius(total, baseRadius, maxTotal) {
  return baseRadius + (total / maxTotal) * baseRadius * 2; 
}

const centerR = getRadius(maxItem.total, baseRadius, maxTotal);

const centerX = width / 2;
const centerY = height / 2;

const centerNode = {
  ...maxItem,
  r: centerR,
  x: centerX,
  y: centerY,
};
```

2 - Calcular o maior raio entre as bolhas secundárias para calcular o espaçamento necessário. </br>
Solução: utilizando o Math.max eu consegui distribuir elas proporcionalmente e no retorno passei o getradius com o total, o base radius e o chart que tem o valor total maior, adicionei um padding extra referente ao base radius multiplicado por 2.

o base Radius é definido pelo tamanho da tela atual divido por 1/4 pra ajudar na responsividade.
```js
const baseRadius = Math.min(width, height) / 25;

const maxOtherR = Math.max(...others.map((o) => getRadius(o.total, baseRadius, maxTotal)));

const padding = baseRadius * 2;
```
3 - Definir a distância do centro e onde os charts secundários serão posicionadas. </br>
Solução: primeiro eu precisei definir a distancia do centro onde as bolhas iriam ser posicionadas para evitar a sobreposição, peguei o raio central e somei junto com o raio maximo dos outros e somei novamente junto com o padding.

para calcular a orbita desses itens eu pesquisei junto ao chatGTP como eu poderia fazer esse calculo de forma que eu conseguisse distribuir ele de forma circular e ai que entou em cena o Math.PI multiplicado por dois e depois dividido pela quantiddade dps outros charts que são menores que o do centro que garante a distribuição ao redor do maior.

com o retorno do angleStep eu consegui passar por cada chart secundário ao do centro e calcular seu raio proporcional, onde a const angle me permite pegar o indice e o angleStep para calcular as cordenadas de X e Y usando trigonometria(o chat gpt tbm me auxiliou nesse entendimento) para distribuir em circulo ao redor do chart com maior valor total e por fim eu defino o charts.value com o centerChart e depois faço o spread dos outros charts.
```js
const orbitRadius = centerR + maxOtherR + padding;
const angleStep = (2 * Math.PI) / others.length;

const otherCharts = others.map((d, i) => {
  const r = getRadius(d.total, baseRadius, maxTotal);
  const angle = i * angleStep;

  return {
    ...d,
    r,
    x: centerX + orbitRadius * Math.cos(angle),
    y: centerY + orbitRadius * Math.sin(angle),
  };
});

charts.value = [centerChart, ...otherCharts];
```

## Como Rodar Localmente

1. Clone o repositório:  
```bash
```

2. Entre no repositório:
```bash
cd bubble-mood
```

3. Instale as dependencias:
```bash
npm install
```

4. Rode o projeto:
``` bash
npm run dev
```
