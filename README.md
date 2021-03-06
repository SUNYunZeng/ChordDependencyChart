# ChordDependencyChart
A char library based on D3 to show the chord dependency


Arguments Descriptions

+ d3: the library dependency 
+ svg: the scg to draw
+ matrix: the data matrix to show
+ nameByIndex: the name of each dimension
+ width & width: the width & width of the svg
+ font_size: the font size of the svg

html code:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<script src="https://d3js.org/d3.v5.min.js"></script>
<script src="ChordDependencyChart.js" charset="utf-8"></script>

<body>
    <svg style="width: 1000px; height:1000px"></svg>
</body>
<script>
    const matrix = [
        [1, 41, 0, 0, 8, 4, 2, 0, 5, 5],
        [6, 0, 1, 1, 1, 0, 4, 1, 0, 1],
        [1, 12, 0, 0, 3, 0, 1, 0, 1, 3],
        [0, 11, 3, 0, 4, 1, 1, 0, 0, 2],
        [1, 1, 0, 0, 1, 0, 0, 2, 2, 0],
        [4, 0, 4, 0, 0, 0, 0, 1, 0, 1],
        [0, 6, 0, 0, 1, 1, 0, 0, 1, 0],
        [1, 4, 2, 0, 0, 1, 0, 0, 0, 0],
        [1, 0, 1, 0, 0, 2, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 3, 0, 0, 1, 0],
    ]

    const nameByIndex = {
        0: 332,
        1: 229,
        2: 333,
        3: 343,
        4: 313,
        5: 357,
        6: 344,
        7: 318,
        8: 312,
        9: 329
    };
    const indexByName = {
        332: 0,
        229: 1,
        333: 2,
        343: 3,
        313: 4,
        357: 5,
        344: 6,
        318: 7,
        312: 8,
        329: 9
    };

    let config = {
        matrix: matrix,
        nameByIndex: nameByIndex,
        indexByName: indexByName,
        width: 1000,
        height: 1000
    }
    let chart = new Chart(d3, config);
    let svg = chart.draw();
    chart.saveAsPng(svg, name = "local");
</script>

</html>
```

js code

```javascript
// ChordDependencyChart.js

class Chart {

    constructor(d3, config) {
        this.d3 = d3;
        this.svg = null;
        this.config = config;
        this.matrix = this.config.matrix;
        this.nameByIndex = this.config.nameByIndex;
        this.indexByName = this.config.indexByName;
        this.width = this.config.width;
        this.height = this.config.height;
        this.font_size = this.config.font_size ? this.config.font_size : 20;
    }

    draw() {

        if (!this.d3 || !this.config || !this.matrix || !this.nameByIndex || !this.indexByName) {
            console.error('initial error!');
        }

        let color = this.d3.scaleOrdinal(this.d3.schemeCategory10);
        let outerRadius = Math.min(this.width, this.height) * 0.5;
        let innerRadius = outerRadius - 124;
        let ribbon = this.d3.ribbon().radius(innerRadius);
        let arc = this.d3.arc().innerRadius(innerRadius).outerRadius(innerRadius + 20);
        let chord = this.d3.chord().padAngle(.04).sortSubgroups(this.d3.descending).sortChords(this.d3
            .descending);
        let svg = this.d3.select("svg").attr("viewBox", [-this.width / 2, -this.height / 2, this.width, this
                .height
            ])
            .attr("font-size", this.font_size);

        const chords = chord(this.matrix);
        const group = svg.append("g").selectAll("g").data(chords.groups).join("g");

        group.append("path").attr("fill", d => color(d.index)).attr("stroke", d => color(d.index)).attr("d",
            arc);

        group.append("text").each(d => {
                d.angle = (d.startAngle + d.endAngle) / 2;
            })
            .attr("dy", ".35em")
            .attr("transform", d => `
            rotate(${(d.angle * 180 / Math.PI - 90)})
            translate(${innerRadius + 26})
            ${d.angle > Math.PI ? "rotate(180)" : ""}`)
            .attr("text-anchor", d => d.angle > Math.PI ? "end" : null)
            .text(d => this.nameByIndex[d.index]);

        svg.append("g")
            .attr("fill-opacity", 0.67)
            .selectAll("path")
            .data(chords)
            .join("path")
            .attr("stroke", d => this.d3.rgb(color(d.source.index)).darker())
            .attr("fill", d => color(d.source.index))
            .attr("d", ribbon);
        this.svg = svg;
        return svg;
    }

    saveAsPng(svg,name="chordDependencyChart") {
        if(!svg){
            svg = this.svg;
        }
        let serializer = new XMLSerializer();
        let source = '<?xml version="1.0" standalone="no"?>\r\n' + serializer.serializeToString(svg.node());
        let image = new Image;
        image.src = "data:image/svg+xml;charset=utf-8," + encodeURIComponent(source);
        let canvas = document.createElement("canvas");
        canvas.width = 1000;
        canvas.height = 1000;
        let context = canvas.getContext("2d");
        context.fillStyle = '#fff'; 
        context.fillRect(0, 0, 10000, 10000);
        image.onload = function () {
            context.drawImage(image, 0, 0);
            let a = document.createElement("a");
            a.download = name +".png";
            a.href = canvas.toDataURL("image/png");
            a.click();
        };
    }
}
```
