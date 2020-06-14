Bar chart

```js
function draw(series, isHorizontal) {
    for (let i = 0, bc = 0; i < series.length; i++, bc++) {
        let x,y
        for (let j = 0; j < gl.dataPoints - 1; j++) {
            let paths = null
            if(isHorizontal) {
                paths = drawBarPaths({
                    
                })
            } else {
                paths = drawColumnPaths({
                    indexes: { i, j, realIndex, bc },
                    x,
                    y,
                    xDivision,
                    pathTo,
                    pathFrom,
                    barWidth,
                    zeroH,
                    strokeWidth,
                    elSeries
                })
            }
            let renderedPath = graphics.renderPaths({
          i,
          realIndex,
          pathFrom: pathFrom,
          pathTo: pathTo,
          stroke: lineFill,
          strokeWidth,
          strokeLineCap: w.config.stroke.lineCap,
          fill: pathFill,
          animationDelay: delay,
          initialSpeed: w.config.chart.animations.speed,
          dataChangeSpeed: w.config.chart.animations.dynamicAnimation.speed,
          className: 'apexcharts-bar-area',
          id: 'apexcharts-bar-area'
        })
        }
        elSeries.add(renderedPath)
    }
}


function drawBarPaths({
    indexes,
    barHeight,
    strokeWidth,
    pathTo,
    pathFrom,
    zeroW,
    x,
    y,
    yDivision,
    elSeries
}) {
    let barYPosition = y + barHeight * this.visibleI
    if (typeof this.series[i][j] === 'undefined' || this.series[i][j] === null) {
      x = zeroW
    } else {
      x = (zeroW + this.series[i][j] / this.invertedYRatio)
    }
    if (!w.globals.dataXY) {
      y = y + yDivision
    }
    let rect = graphics.drawRect(
        0,
        barYPosition - barHeight * this.visibleI,
        w.globals.gridWidth,
        barHeight * this.seriesLen,
        0,
        bcolor,
        this.barOptions.colors.backgroundBarOpacity
      )
      elSeries.add(rect)
}

drawColumnPaths({
    indexes,
    x,
    y,
    xDivision,
    pathTo,
    pathFrom,
    barWidth,
    zeroH,
    strokeWidth,
    elSeries
}) {
   let barXPosition = x + barWidth * this.visibleI
   if (typeof this.series[i][j] === 'undefined' || this.series[i][j] === null) {
      y = zeroH
    } else {
      y = (zeroH - this.series[i][j] / this.yRatio[this.yaxisIndex])
    }
    let endingShapeOpts = {
      barWidth,
      strokeWidth,
      barXPosition,
      y,
      zeroH
    }
    pathTo =
      pathTo +
      graphics.line(barXPosition, endingShape.newY) +
      endingShape.path +
      graphics.line(barXPosition + barWidth - strokeWidth, zeroH) +
      graphics.line(barXPosition, zeroH)
}




```

