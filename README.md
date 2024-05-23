
### Exportando Gráficos da AMCharts para o QLIK SENSE

### 1. Preparando o Ambiente

Antes de começarmos, certifique-se de ter os seguintes itens instalados:

- Node.js e npm
- Qlik Sense Desktop ou acesso ao Qlik Sense Server
- Editor de código (VS Code, por exemplo, recomendo o Notepad++ um editor simples e de interface amigável).

### 2. Estrutura do Projeto

Crie a estrutura básica de pastas e arquivos para sua extensão Qlik Sense. Vamos chamar a extensão de `amchart-export`.

```
amchart-export/
│
├── amchart-export.qext
├── amchart-export.js
├── amchart-export.css
├── amchart-export.html
├── amchart.min.js
└── icon.png
```

### 3. Arquivo de Definição (`amchart-export.qext`)

Crie o arquivo `amchart-export.qext` com o seguinte conteúdo:

```json
{
  "name": "amChart Export",
  "description": "Extensão para exportar visuais usando amCharts",
  "type": "visualization",
  "version": "1.0.0",
  "author": "Seu Nome",
  "icon": "icon.png",
  "preview": "preview.png",
  "homepage": "https://seusite.com"
}
```

### 4. Arquivo JavaScript (`amchart-export.js`)

Este arquivo conterá a lógica principal para renderizar o gráfico e exportá-lo.

```javascript
define([
  'jquery',
  'qlik',
  './amchart.min'
], function($, qlik, am4core, am4charts) {
  return {
    initialProperties: {
      version: 1.0,
      qHyperCubeDef: {
        qDimensions: [],
        qMeasures: [],
        qInitialDataFetch: [{
          qWidth: 10,
          qHeight: 1000
        }]
      }
    },
    definition: {
      type: "items",
      component: "accordion",
      items: {
        dimensions: {
          uses: "dimensions",
          min: 1,
          max: 1
        },
        measures: {
          uses: "measures",
          min: 1,
          max: 1
        },
        settings: {
          uses: "settings"
        }
      }
    },
    paint: function($element, layout) {
      var self = this;
      var id = "container_" + layout.qInfo.qId;
      if (!$element.find("#" + id).length) {
        $element.append($('<div />').attr("id", id).css({ height: $element.height(), width: $element.width() }));
      }

      // Extraindo dados do hypercube
      var qData = layout.qHyperCube.qDataPages[0].qMatrix;
      var data = qData.map(function(d) {
        return {
          category: d[0].qText,
          value: d[1].qNum
        };
      });

      // Criando gráfico amChart
      am4core.useTheme(am4themes_animated);
      var chart = am4core.create(id, am4charts.XYChart);
      chart.data = data;

      var categoryAxis = chart.xAxes.push(new am4charts.CategoryAxis());
      categoryAxis.dataFields.category = "category";
      categoryAxis.renderer.grid.template.location = 0;

      var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());

      var series = chart.series.push(new am4charts.ColumnSeries());
      series.dataFields.valueY = "value";
      series.dataFields.categoryX = "category";
      series.name = "Value";
      series.columns.template.tooltipText = "{categoryX}: [bold]{valueY}[/]";

      // Botão de exportação
      var exportButton = $(`<button>Exportar Gráfico</button>`).css({
        position: 'absolute',
        top: '10px',
        right: '10px'
      });
      $element.append(exportButton);

      exportButton.on('click', function() {
        chart.exporting.export("png");
      });

      return qlik.Promise.resolve();
    }
  };
});
```

### 5. Estilos CSS (`amchart-export.css`)

Adicione qualquer estilo necessário no arquivo `amchart-export.css`.

```css
#container {
  width: 100%;
  height: 100%;
  position: relative;
}
```

### 6. HTML da Extensão (`amchart-export.html`)

Embora não seja estritamente necessário, você pode criar um arquivo HTML para adicionar estrutura adicional à sua extensão.

```html
<div id="container"></div>
```

### 7. Integrando amCharts

Baixe a versão minificada do amCharts e salve como `amchart.min.js` na pasta da extensão.

### 8. Adicionando Ícone e Preview

Adicione os arquivos `icon.png` e `preview.png` para dar uma identidade visual à sua extensão.

### 9. Instalando e Testando a Extensão

1. **Zipar a Extensão**: Compacte a pasta `amchart-export` em um arquivo `.zip`.

2. **Instalar no Qlik Sense Desktop**:
   - Navegue até a pasta `Extensions` no diretório de instalação do Qlik Sense Desktop.
   - Extraia o conteúdo do arquivo `.zip` na pasta `Extensions`.

3. **Testar a Extensão**:
   - Abra o Qlik Sense Desktop e crie uma nova aplicação ou abra uma existente.
   - Adicione uma nova visualização e selecione `amChart Export`.
   - Configure as dimensões e medidas necessárias.

### Considerações Finais

Essa extensão básica deve permitir a criação de gráficos usando a biblioteca amCharts e incluir a funcionalidade de exportação. Você pode expandir a extensão adicionando mais funcionalidades conforme necessário.
